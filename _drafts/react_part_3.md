---
layout: post
title: Entendendo o Redux
thumbnail: /images/react-part-1.png
short_url: https://goo.gl/nJmg14
ref: react_3
lang: pt
---

Redux é um container de estado previsível para apps javascript. O Redux é simples. Todo site, blog, vídeo que fala sobre Redux ressalta sua simplicidade. No meu caso pessoal, achei que a terminologia utilizada pode confudir um pouco. Nomes como **actions**, **reducers** e **store**, por exemplo.

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

Reducers para alterar o estado
------------------------------

Como falei anteriormente, o **reducer** sabe como atualizar o estado. Uma coisa interessante é que cada reducer é responsável por atualizar uma parte do estado!

Precisamos de um reducer para atualizar ```user``` e outro para atualizar ```posts```.

A primeira função da API do Redux que vamos utilizar é a ```combineReducers```, uma função auxiliar que faz o link entre os reducers da aplicação com partes independentes do estado.

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

O argumento ```state``` contém último estado que a função retornou, ou seja, o estado anterior. Na primeira execução o ```state``` não vai possuir um valor. Então é interessante criar um valor inicial:

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

Uma coisa que nunca podemos fazer no reducer é modificar seus argumentos. Os reducers devem ser bem sucintos e diretos, sem surpresas. Logo, nada de fazer chamadas a API ou funções não puras como ```Date.now()``` e ```Math.random()```.

Logo devemos realmente retornar um novo objeto, ao invés de reaproveitar ```state```:

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

Utilizamos o [spread operator](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Operators/Spread_operator) do ES6 para criar uma cópia do objeto ```state``` e popular alguns atributos. Atributos novos são adicionados e atributos existentes são sobrescritos.

A imutabilidade é uma característica importante para alcançar a previsibilidade do Redux. Se você se encontrar em um cenário mais complexo, recomendo utilizar uma lib como o [Immutable.js](https://facebook.github.io/immutable-js/) para isso.

Já discutimos anteriormente sobre a ação que deve ser despachada para atualizar o estado ```user```.

Como criar ações
----------------

O que escrevemos abaixo é uma função que retorna a ação que queremos despachar, também conhecida como **action creator**:


``` javascript
import {
  LOGIN_SUCCESSFULLY,
} from 'types';

export const login = (email, password) => {
  // we should replace this by an API call
  const user = { email, displayName: 'Jhon Doe' }
  return {
    type: LOGIN_SUCCESSFULLY,
    payload: user
  };
};
```

Mais na frente vamos voltar a esse código para implementar uma chamada de API assíncrona. Vamos deixar assim por enquanto.

Se a gente quiser implementar o log out, a lógica é bem parecida:

``` javascript
import {
  LOGIN_SUCCESSFULLY,
  LOGED_OUT
} from 'types';

export const login = (email, password) => {
  // we should replace this by an API call
  const user = { email, displayName: 'Jhon Doe' }
  return {
    type: LOGIN_SUCCESSFULLY,
    payload: user
  };
};

export const logout = () => {
  return { type: LOGED_OUT };
};
```

E o reducer:

``` javascript
import {
  LOGIN_SUCCESSFULLY,
  LOGED_OUT
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
    case LOGED_OUT:
      return {
        ...state,
        email: initialState.email,
        displayName: initialState.displayName
      };
    default:
      return state;
  }
};
```

Colocando o Redux para funcionar
--------------------------------

Vamos conhecer mais sobre a API do Redux.

A função que cria a **store**, que é responsável por manter o estado, é chamada **createStore** e tem a seguinte assinatura:

``` javascript
createStore(reducer, [preloadedState], [enhancer])
```

Por enquanto, vamos utilizar somente o primeiro argumento:

``` javascript
import { createStore } from 'redux';
import reducers from '.reducers';

const store = createStore(reducers)

// integration with some framework here
```

Com a variável ```store``` podemos fazer a integração do Redux com o framework javascript que estivermos 
utilizando.

Para integrar com o React, por exemplo, utilizamos a lib [react-redux](https://github.com/reactjs/react-redux).

Depois de feita a integração, não lidamos mais diretamente com a ```store```. O lib que faz a integração, como o **react-redux**, fica responsável por lidar com a ```store```, então não precisamos nos preocupar com isso.

Integrando com o ```redux-thunk``` para fazer chamadas assíncronas
--------------------------------------------------------------

A função ```createStore``` também é utilizada para integrar com middlewares de terceiros.

Por exemplo, para implementar chamadas de API assíncronas, uma lib muito interessante é a [redux-thunk](https://github.com/gaearon/redux-thunk), que é um middleware utilizado para atrasar o despacho de uma ação. Segue a configuração:

``` javascript
import { createStore, applyMiddleware } from 'redux';
import reducers from '.reducers';
import ReduxThunk from 'redux-thunk';

const store = createStore(
  reducers,
  {},
  applyMiddleware(ReduxThunk)
);

// integration with some framework here
```

```applyMiddleware``` é um enchancer que vem com o Redux, utilizado exatamente para fazer integrações com libs de terceiros.

O próximo passo é alterar a maneira de como retornamos a ação lá no action creator.

A sacada do **redux-thuk** é fazer o action creator retornar uma função que cria uma ação, ao invés de retornar a ação diretamente.

O rascunho de action creator de ```login``` com redux-thuk fica assim:

``` javascript
export const login = () => {
  return (dispatch) => {
    loginWithApi().then(() => dispatch({ type: LOGIN_USER_SUCCESS }));
  };
};
```

A função ```login``` agora retorna uma função que tem ```dispatch``` como argumento. Utilizamos a função ```dispatch``` para despachar a ação quando estivermos prontos. Isso nos dá muita flexibilidade.

As ações comunicam o que acontece no sistema, lembra? Poderíamos depachar ações para informar quando o usuário deu início ao login e para falar que o login foi concluído com sucesso ou falha.

O código fica assim:

``` javascript
export const login = (email, password) => {
  return (dispatch) => {
    dispatch({ type: LOGIN_USER });

    loginWithApi(email, password)
      .then(user => {
        dispatch({ type: LOGIN_USER_SUCCESS, payload: user});
      })
      .catch((e) => {
        dispatch({ type: LOGIN_USER_FAIL });
      });
  };
};
```

No código acima, comunicamos quando o usuário resolveu fazer o login, com a ação do tipo ```LOGIN_USER``` e depois despachamos ```LOGIN_USER_SUCCESS``` ou ```LOGIN_USER_FAIL```, de acordo com a resposta da API.

Com isso podemos fazer coisas legais, como mostrar um "carragando" entre o início e o fim da chamada de API.

Precisamos modificar o reducer para isso:

``` javascript
import {
  LOGIN_SUCCESSFULLY,
  LOGED_OUT
} from '../actions/types';

const initialState = {
  email: '',
  displayName: 'Anonimous User',
  error: '',
  loading: false,
};

export default (state = initialState, action) => {
  switch (action.type) {
    case LOGIN_USER:
      return {...state, loading: true };
    case LOGIN_SUCCESSFULLY:
      return {
          ...state,
          email: action.payload.email,
          displayName: action.payload.displayName,
          loading: false,
        };
    case LOGIN_FAILED:
      return {
        ...state,
        error: action.payload.errorMessage,
        loading: false,
      };
    case LOGED_OUT:
      return {
        ...state,
        email: initialState.email,
        displayName: initialState.displayName
      };
    default:
      return state;
  }
};
```

Passamos a utilizar os atributos ```error``` e ```loading```.

Conclusão
---------

Vimos a visão geral de como utilizar o Redux. Uma dúvida que tive quando comecei a utilizar Redux foi: "Por que mesmo utilizar o Redux? Poderia modificar o estado diretamente. Para que criar ações, reducers, etc?".

A resposta para isso vem quando integramos o Redux ao nosso framework javascript. Os componentes ficam muito mais limpos, com código muito mais fácil de ser mantido. O Redux traz ao nosso projeto mais previsibilidade, e com isso conseguimos diminuir a quantidade de bugs.

No próximo post vou explicar em mais detalhes a integração do Redux com React e disponibilizar o código fonte. Vamos ver na prática as vantagens do Redux, e explorar ainda mais sua API. Até lá!
