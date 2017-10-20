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
- Fazer a aplicação React se comunicar com a aplicação Rails.

Código fonte do projeto React que escrevemos no post da parte 1 [está nesse repositório](https://github.com/lutchobandeira/news-feed-react)

As alterações que fizemos no projeto React neste post estão [no mesmo repositório mas em outra branch](https://github.com/lutchobandeira/news-feed-react/commits/backend_integration).

Diferentes maneiras de integrar React e Rails
---------------------------------------------

Uma possibilidade seria escrever o código React dentro de um projeto Rails. Com o [lançamento do Rails 5.1](http://guides.rubyonrails.org/5_1_release_notes.html), fica mais fácil trabalhar com frameworks javascript dentro de uma aplicação Rails.

Uma das novidades da versão 5.1 do Rails é o suporte ao [Yarn](https://yarnpkg.com/en/) e o suporte ao Webpack através da gem [webpacker](https://github.com/rails/webpacker).

Caso você queira usar essa abordagem em um projeto Rails 4, você pode utilizar as gems [react-rails](https://github.com/reactjs/react-rails) e [react_on_rails](https://github.com/shakacode/react_on_rails).

Outra possibilidade é manter os códigos React e Rails em projetos separados e fazer eles se comunicarem através de uma API. Esta é a abordagem que vamos adotar neste post.

Criando uma aplicação API com o Rails
-------------------------------------

A partir do Rails 5 é possível criar aplicações específicas para API:


``` bash
$ rails new news_feed_rails --api
```

Com este comando, o Rails cria uma aplicação mais magra, sem os recursos que seriam utilizados por aplicações tradicionais com views em HTML.

A aplicação é configurada para utilizar somente os middlewares necessários. O ```ApplicationController``` da aplicação herda de ```ActionController::API```, que por sua vez importa um número menor de módulos.

A aplicação Rails que vamos criar é muito simples. Ela tem apenas o modelo ```Post```:

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

Agora vamos para o controller:

``` bash
$ rails g controller posts
```

Precisamos de duas ações em nosso controller: uma para listar os posts e outra para criar um novo post:

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

Nossa API está pronta! Mas como vamos fazer as requisições a partir da aplicação React, devemos habilitar requisições de outro servidor.

O Rails 5 já deixa as coisas fáceis para permitir o CORS (cross-origin HTTP request). Basta descomentar a gem [rack-cors](https://github.com/cyu/rack-cors) no ```Gemfile```:

``` ruby
gem 'rack-cors'
```

E descomentar o conteúdo do arquivo ```config/initializers/cors.rb```:

``` ruby
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins '*'

    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```

Para deixar as coisas mais simples, em ```origins '*'``` permitimos requisições de todas as origens.

Vamos iniciar a aplicação na porta 3001 (para deixar a aplicação React rodar na porta 3000):

``` bash
$ rails s -p 3001
```

Pronto!

Como consumir uma API a partir do React
---------------------------------------

Antes de começar, vamos navegar para a pasta do projeto React e iniciar o servidor:

``` bash
$ npm start
```

No mundo javascript há inúmeras maneiras de fazer requisições HTTP, [como você pode ver neste artigo](https://hashnode.com/post/5-best-libraries-for-making-ajax-calls-in-react-cis8x5f7k0jl7th53z68s41k1).

Neste post vamos utilizar a [Fetch API](https://developer.mozilla.org/pt-BR/docs/Web/API/Fetch_API/Using_Fetch), que já está disponível a partir do Firefox 39 e do Chrome 42.

A Fetch API fornece o método ```fetch()```, que possui somente um argumento obrigatório, a URL do recurso que queremos acessar, e retorna uma [Promisse](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Promise) com a resposta.

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

Os componentes do React tem vários [métodos de ciclo de vida](https://reactjs.org/docs/react-component.html) (lifecycle methods) que podemos sobrescrever quando queremos rodar um código específico, como uma chamada a uma API, em determinado momento.

Vamos sobrescrever ```componentWillMount()``` para fazer nossa requisição. Esse método roda quando o componente está sendo montado, antes da chamada a ```render()```:

``` jsx
componentWillMount() {
  this.fetchPosts();
}
```

Vamos alterar o código para inicializar a lista de posts com um array vazio (a implementação anterior carregava a lista de posts do localStorage):

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

Dessa forma, o fluxo de chamadas fica assim: ```componentWillMount()``` -> ```fetchPosts()``` -> ```startPolling()``` -> ```fetchPosts()``` -> (...).

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

Aqui fazemos uma requisição POST e atualizamos o Feed otimisticamente através da chamada ```setState()```, para deixar a UI mais fluida.

Para cadastrar a categoria corretamente, temos que modificar a lista de categorias para ficar exatamente igual ao ```enum``` do modelo ```Post``` da aplicação Rails. No código que escrevemos no post anterior, a primeira letra de cada categoria estava maiúscula. Se deixarmos tudo minúsculo já ficamos bem:

<pre class="line-numbers" data-start="4"><code class="language-jsx">
const categories = ['world', 'business', 'tech', 'sport'];
</code></pre>

Exibindo erros de validação
---------------------------

Mas e se tivermos erros de validação? Nesse caso, seria interessante:

- exibir o erro de validação ao lado do campo correspondente;
- retirar da lista de posts o novo post que adicionamos otimisticamente.

Vamos começar alterando o método ```handleNewPost()```:

<pre class="line-numbers" data-start="51" data-line="2-3,15-24"><code class="language-jsx">
  handleNewPost(post) {
    const currentPosts = this.state.posts;
    const context = this;

    var posts = this.state.posts.concat([post]);
    this.setState({ posts });

    fetch(`http://localhost:3001/posts`, {
      method: 'post',
      body: JSON.stringify(post),
      headers: { 'Content-Type': 'application/json' }
    }).then(function(response) {
      return response.json();
    }).then(function(data) {
      if (data.errors) {
        context.setState({
          errors: data.errors,
          posts: currentPosts
        });
      } else {
        context.setState({
          errors: {}
        });
      }
    });
  }
</code></pre>

Observe as linhas 65 a 74. Nelas escrevemos o código que trata os erros. Lembre-se que a API Rails retorna uma lista de erros caso o modelo tenha erros de validação. Se tivermos erros de validação, atualizamos o estado com a lista de erros e resetamos a lista de posts (linhas 67 e 68).

Nas linhas 52 e 53 guardamos a lista corrente de posts, para ser utilizada em caso de erros da validação, e passamos ```this``` para a variável ```context```. Esse é um truque para usar a referência ```this``` de ```Feed``` dentro do callback.

Vamos inicializar o objeto de erros no estado:

<pre class="line-numbers" data-start="1" data-line="7"><code class="language-jsx">
class Feed extends Component {
  constructor(props) {
    super(props);
    this.state = {
      posts: [],
      filteredPosts: [],
      errors: {}
    }
</code></pre>

Antes de alterar o componente ```PostForm```, vamos passar a lista de erros para ele através de uma prop. Segue a modificação no componente ```Feed```:

<pre class="line-numbers" data-start="88" data-line="12"><code class="language-jsx">
  render() {
    const posts = this.state.posts.map((post, index) =&gt;
      &lt;Post key={index} value={post} /&gt;
    );
    const filteredPosts = this.state.filteredPosts.map((post, index) =&gt;
      &lt;Post key={index} value={post} /&gt;
    );
    return (
      &lt;div className="feed"&gt;
        &lt;Filter onFilter={this.handleFilter} /&gt;
        {filteredPosts.length &gt; 0 ? filteredPosts : posts}
        &lt;PostForm onSubmit={this.handleNewPost} errors={this.state.errors} /&gt;
      &lt;/div&gt;
    )
  }
<!-- render() {
    const posts = this.state.posts.map((post, index) =>
      <Post key={index} value={post} />
    );
    const filteredPosts = this.state.filteredPosts.map((post, index) =>
      <Post key={index} value={post} />
    );
    return (
      <div className="feed">
        <Filter onFilter={this.handleFilter} />
        {filteredPosts.length > 0 ? filteredPosts : posts}
        <PostForm onSubmit={this.handleNewPost} errors={this.state.errors} />
      </div>
    )
  } -->
</code></pre>


E por último atualizamos o componente ```PostForm``` para exibir os erros de validação:

<pre class="line-numbers" data-start="132" data-line="2-5,11,20"><code class="language-jsx">
 render() {
    let errors = {};
    Object.keys(this.props.errors).forEach((key) =&gt; {
      errors[key] = this.props.errors[key] ? this.props.errors[key][0] : null;
    });
    return (
      &lt;div className="post-form"&gt;
        &lt;form onSubmit={this.handleSubmit}&gt;
          &lt;label&gt;
            Category:
            &lt;small className="error"&gt;{errors.category}&lt;/small&gt;
            &lt;select ref={(input) =&gt; this.category = input}&gt;
              {categories.map((category, index) =&gt;
                &lt;option key={category} value={category}&gt;{category}&lt;/option&gt;
              )}
            &lt;/select&gt;
          &lt;/label&gt;
          &lt;label&gt;
            Content:
            &lt;small className="error"&gt;{errors.content}&lt;/small&gt;
            &lt;input type="text" ref={(input) =&gt; this.content = input} /&gt;
          &lt;/label&gt;
          &lt;button className="button"&gt;Submit&lt;/button&gt;
        &lt;/form&gt;
      &lt;/div&gt;
    )
  }

<!--
  render() {
    let errors = {};
    Object.keys(this.props.errors).forEach((key) => {
      errors[key] = this.props.errors[key] ? this.props.errors[key][0] : null;
    });
    return (
      <div className="post-form">
        <form onSubmit={this.handleSubmit}>
          <label>
            Category:
            <small className="error">{errors.category}</small>
            <select ref={(input) => this.category = input}>
              {categories.map((category, index) =>
                <option key={category} value={category}>{category}</option>
              )}
            </select>
          </label>
          <label>
            Content:
            <small className="error">{errors.content}</small>
            <input type="text" ref={(input) => this.content = input} />
          </label>
          <button className="button">Submit</button>
        </form>
      </div>
    )
  }
 -->
</code></pre>

Nas linhas 133 a 136, colocamos no objeto ```errors``` o primeiro erro de cada atributo, se houver algum.

Com isso, finalizamos nossa implementação! Você pode ver o código funcionando neste [CodePen](https://codepen.io/lutchobandeira/pen/gGEXZz){:target="_blank"} aqui.

Próximos Passos
---------------

Nosso código funciona, mas parece que o componente ```Feed``` está fazendo coisas demais. Além de ter a responsabilidade de cuidar de seu estado e de suas propriedades, o componente se preocupa em fazer requisições a um servidor externo.

No próximo post vamos aprender como gerenciar os dados de nossa aplicação React com [redux](http://redux.js.org/). Até lá!
