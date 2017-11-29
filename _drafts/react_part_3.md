---
layout: post
title: Entendendo o Redux
thumbnail: /images/react-part-1.png
short_url: https://goo.gl/nJmg14
ref: react_3
lang: pt
---

Redux é um container de estado previsível para apps javascript. Essa é a forma como o framework é descrito em sua [documentação](http://redux.js.org/).

O Redux é simples. Isso é ressaltado em todo site, blog, vídeo que fala sobre Redux. No meu caso pessoal, achei que a terminologia utilizada pode assustar um pouco. Nomes como **actions**, **reducers** e **store**, por exemplo.

O criador do Redux escreveu em seu Twitter que a revolução do javascript não é sobre os frameworks, mas sim sobre como gerenciamos dados.

Descomplicando conceitos
------------------------

No Redux, o estado da aplicação é representado por um objeto javascript simples:

``` javascript
{
  user: {
    email: 'user@gmail.com',
    displayName: 'User'
  },
  posts: [{
    content: 'World Cup Classifiers has ended',
    category: 'sport'
  }, {
    content: 'New JS framework is launched',
    category: 'tech'
  }]
}
```

Esse estado precisa ser armazenado em algum lugar, certo? Quem armazena esse estado, pela terminologia do Redux, é a **store** (ou Loja, em tradução livre).

Cada aplicação Redux possui somente uma **store** e através dela conseguimos acessar todo o estado da aplicação. Pense no estado como um banco de dados, onde armazenamos tudo o que é importante para a aplicação.

Observe que o objeto javascript que representa o estado da aplicação possui duas chaves: ```user``` e ```posts```.

O Redux precisa saber como atribuir valores para essas chaves, certo?

E como ele faz isso?

Através de **reducers**! Um **reducer** é uma função que sabe como atualizar o estado. Mais na frente vamos escrever o ```userReducer``` e o ```postReducer```, que serão responsáveis por atualizar ```user``` e ```posts```.

Os reducers sabem *como* o estado deve ser atualizado. Para saber *o que* deve ser atualizado, precisamos de uma  ação, ou **action**.

**Actions** são objetos javascript que descrevem o que aconteceu na aplicação. Por exemplo, se o usuário logou com sucesso no sistema, podemos escrever a seguinte ação, que possui tipo e dados:


``` javascript
{
  type: 'login_successfully',
  payload: {
    email: 'user@gmail.com',
    displayName: 'User'
  }
}
```

Esses são os conceitos básicos do Redux! Para modificarmos o estado da aplicação, que fica armazenado na **store**, devemos despachar uma ação, ou seja, uma **action**, que é processada por uma função que sabe como modificar o estado, o **reducer**.

Vamos aprender agora como utilizar a API do Redux para colocar tudo isso para funcionar. Perceba que até agora ainda não escrevemos nenhum código além de objetos javascript simples.

Próximos Passos
---------------

Nosso código funciona, mas parece que o componente ```Feed``` está fazendo coisas demais. Além de ter a responsabilidade de cuidar de seu estado e de suas propriedades, o componente se preocupa em fazer requisições a um servidor externo.

No próximo post vamos aprender como gerenciar os dados de nossa aplicação React com [redux](http://redux.js.org/). Até lá!
