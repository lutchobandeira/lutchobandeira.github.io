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

Mas como alterar o estado do Redux?

Cada atributo do estado é atualizado através um **reducer**! O **reducer** uma função que sabe como atualizar o estado.

Se para alterar o estado precisamos de um **reducer**, para rodar um um **reducer** precisamos despachar uma **action**.

**Actions** descrevem o que aconteceu na aplicação. Por exemplo, se o usuário logou com sucesso no sistema, podemos escrever a seguinte ação, um objeto que  possui os atributos tipo e dados:

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

Utilizando a API do Redux
-------------------------

Como falei anteriormente, o **reducer** sabe como atualizar o estado. Uma coisa interessante é que cada reducer é responsável por atualizar uma parte do estado!

Precisamos de um reducer para atualizar ```user``` e outro para atualizar ```posts```.

A primeira função do Redux que vamos utilizar é a ```combineReducers```, uma função auxiliar do Redux que faz o link entre os reducers da aplicação com partes independentes do estado.

Segue o arquivo ```reducers/index.js```:

``` javascript
import { combineReducers } from 'redux';

export default combineReducers({
  user: UserReducer,
  posts: PostsReducer,
});
```

Um rascunho do ```UserReducer``` fica mais ou menos assim, no arquivo ```reducers/UserReducer.js```:

``` javascript
export default (state, action) => {
  // logic to create newState here
  return newState
};
```

O reducer recebe a parte do estado que ele gerencia e uma ação, e retorna um novo estado.

O ```state``` corresponde ao último estado que a função retornou. Na primeira execução o ```state``` não vai possuir um valor. Então é interessante criar um valor inicial:

``` javascript
const initialState = {
  email: null,
  displayName: 'Anonimous User',
};

export default (state = initialState, action) => {
  // logic to create newState here
  return newState
};
```

Uma coisa que nunca podemos fazer no reducer é modificar seus argumentos. Logo devemos realmente retornar um novo objeto, ao invés de reaproveitar ```state```:

``` javascript
import {
  LOGIN_SUCCESSFULLY,
} from '../actions/types';

const initialState = {
  email: '',
  displayName: 'Anonimous User',
  error: '',
};

export default (state = initialState, action) => {
  switch (action.type) {
    case LOGIN_SUCCESSFULLY:
      return {
          ...state,
          email: action.payload.email,
          displayName: action.payload.displayName
        };
    default:
      return state;
  }
};
```

Utilizamos o spread operator do ES6 para criar um novo objeto a partir de ```state``` e sobrescrever alguns atributos.

E agora vamos para as ações.

