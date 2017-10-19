---
layout: post
title: Como fazer um feed de notícias em React.js - Parte 2
thumbnail: /images/react-part-1.png
short_url: https://goo.gl/nJmg14
ref: react_2
lang: pt
---

Este é o segundo de uma série de posts sobre React.js. No [primeiro post](http://lutchobandeira.com/como-fazer-um-feed-de-noticias-em-reactjs-parte-1/), desenvolvemos um feed de notícias somente em React. Neste post vamos:

- Criar um back-end em Rails (código fonte disponível neste [repositório do GitHub](https://github.com/lutchobandeira/news_feed_rails));
- Fazer a aplicação React se comunicar com a aplicação Rails. (Código fonte disponível neste [outro repositório](https://github.com/lutchobandeira/news-feed-rails))

Diferentes maneiras de integrar React e Rails
---------------------------------------------

Uma possibilidade seria escrever o código React dentro de um projeto Rails. Com o [lançamento do Rails 5.1](http://guides.rubyonrails.org/5_1_release_notes.html), fica mais fácil trabalhar com frameworks javascript dentro de uma aplicação Rails.

Uma das novidades da versão 5.1 do Rails é o suporte ao [Yarn](https://yarnpkg.com/en/) e o suporte ao Webpack através da gem [webpacker](https://github.com/rails/webpacker).

Outra possibidade é manter os códigos React e Rails em projetos separados e fazer eles se comunicarem através de uma API. Esta é a abordagem que vamos adotar neste post.

Criando uma aplicação API com o Rails
-------------------------------------

A partir do Rails 5 é possível criar aplicações específicas para API:


``` bash
$ rails new news_feed_rails --api
```

Com este comando, o Rails cria uma aplicação mais magra, sem os recursos que seriam utilizados por aplicações tradicionais com views em HTML.

A aplicação é configurada para utilizar somente os middlewares necessários. Os controllers da aplicação extendem ```ActionController::API```, que por sua vez importam um número menor de módulos.

A aplicação Rails que vamos criar é muito simples. Ela tem apenas um modelo:

``` bash
$ rails g model post category:integer content:string
```

Com o modelo criado, vamos adicionar um ```enum``` para as categorias e vamos deixar categoria e conteúdo como campos obrigatórios:

<pre class="line-numbers "><code class="language-ruby">
class Post < ApplicationRecord
  validates :category, :content, presence: true

  enum category: {
    world: 1,
    business: 2,
    tech: 3,
    sport: 4
  }
end
</code></pre>

Precisamos de duas ações em nosso controller: uma para listar os posts e outra para criar um novo post:

``` bash
$ rails g controller posts
```

Nosso controller fica assim:

<pre class="line-numbers "><code class="language-ruby">
class PostsController < ApplicationController
  def index
    posts = Post.all
    render json: posts, status: :ok
  end

  def create
    post = Post.new(post_params)
    if post.save
      render json: post, status: :created
    else
      render json: { errors: post.errors }, status: :bad_request
    end
  end

  private

  def post_params
    params.permit(:category, :content)
  end
end
</code></pre>

E finalmente atualizamos o ```routes.rb```:

<pre class="line-numbers "><code class="language-ruby">
Rails.application.routes.draw do
  resources :posts, only: [:index, :create]
end
</code></pre>

Vamos iniciar a aplicação na porta 3001 (para deixar a aplicação React rodar na porta 3000):

``` bash
$ rails s -p 3001
```

Pronto!

Como consumir uma API a partir do React
---------------------------------------

No mundo javascript há inúmeras maneiras de fazer requisições HTTP, [como você pode ver neste artigo](https://hashnode.com/post/5-best-libraries-for-making-ajax-calls-in-react-cis8x5f7k0jl7th53z68s41k1).

Neste post vamos utilizar a [Fetch API](https://developer.mozilla.org/pt-BR/docs/Web/API/Fetch_API/Using_Fetch), que já disponível a pardir do Firefox 39 e do Chrome 42.

A Fetch API fornece o método ```fetch()```, que tem um argumento obrigatório, a URL do recurso que queremos acessar, e retorna uma [Promisse](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Promise).

Vamos começar adicionando o método ```fetchPosts()``` ao component ```Feed```:

``` jsx
fetchPosts() {
  fetch('http://localhost:3001/posts').then((response) => {
    return response.json();
  }).then((posts) => {
    this.setState({ posts });
  });
}
```

Observe que utilizamos o ```fetch()``` para fazer a requisição. Tratamos a Promisse para receber a resposta no formato JSON através da chamada ```response.json()``` e finalmente temos um array de posts, que utilizamos para atualizar o estado do componente ```Feed```.

Utilizando os lifecycle methods do React
----------------------------------------

Os componentes do React tem vários [métodos de ciclo de vida](https://reactjs.org/docs/react-component.html) (lifecycle methods) que podemos sobrescrever quando queremos rodar um código espefífico, como uma chamada a uma API, em determinado momento.

Vamos sobrescrever ```componentWillMount()``` para fazer nossa requisição. Esse método roda quando o componente está sendo montado, antes da chamada a ```render()```:

``` jsx
componentWillMount() {
  this.fetchPosts();
}
```

Vamos alterar nossa antiga implementação, que carregava a lista de posts do localStorage:

<pre class="line-numbers" data-start="17" data-line="5"><code class="language-jsx">
class Feed extends Component {
  constructor(props) {
    super(props);
    this.state = {
      posts: [],
      filteredPosts: []
    }
</code></pre>

Nosso Feed de notícias já carrega os posts a partir da API!

No entanto, seria interessante que a gente repetisse essa requisição de tempos em tempos para carregar novos posts sem recarregar a página. Então vamos criar o método ```startPolling```:

``` jsx
startPolling() {
  this.timeout = setTimeout(() => this.fetchPosts(), 10000);
}
```

Perceba que armazenamos o timer criado pelo método [setTimeout()](https://developer.mozilla.org/en-US/docs/Web/API/Window/setTimeout) em ```this.timeout```. Vamos utilizar essa referência para cancelar o timer quando o componente for desmontado:

``` jsx
componentWillUnmount() {
  clearTimeout(this.timeout);
}
```

Precisamos modificar o método ```fetchPosts()``` para chamar ```startPolling()``` quando a primeira requisição terminar:

<pre class="line-numbers" data-start="41" data-line="5,6"><code class="language-jsx">
  fetchPosts() {
    fetch(`${apiUrl}/posts`).then((response) => {
      return response.json();
    }).then((posts) => {
      clearTimeout(this.timeout);
      this.startPolling();
      this.setState({ posts });
    });
  }
</code></pre>

Fazendo requisições POST
------------------------

Para fazer uma requisição POST, passamos um segundo argumento ao método ```fetch()```:

<pre class="line-numbers" data-start="51" data-line="3-5"><code class="language-jsx">
  handleNewPost(post) {
    fetch(`http://localhost:3001/posts`, {
      method: 'post',
      body: JSON.stringify(post),
      headers: { 'Content-Type': 'application/json' }
    }).then(function(response) {
      return response.json();
    }).then(function(data) {
      console.log('server response', data);
    });

    var posts = this.state.posts.concat([post]);
    this.setState({ posts });
  }
</code></pre>

Aqui fazemos uma requisição POST mas atualizamos o Feed otimisticamente através da chamada ```setState()```, para deixar a UI mais fluida. Poderíamos tratar a resposta para verificar se o post foi criado com sucesso ou se ele contém algum erro de validaçao, por exemplo.

Com isso, finalizamos nossa implementação! Yay!

Próximos Passos
---------------

Nosso código funciona, mas parece que o componente ```Feed``` está fazendo coisas demais. Além de ter a responsabilidade de cuidar de seu estado e de suas propriedades, o componente se preocupa em fazer requisições a um servidor externo.

No próximo post vamos aprender como geranciar os dados de nossa aplicação React com [redux](http://redux.js.org/). Até lá!
