---
layout: post
title: Entendendo o Redux
thumbnail: /images/react-part-1.png
short_url: https://goo.gl/nJmg14
ref: react_3
lang: pt
---

Redux é um container de estado previsível para apps javascript. O Redux é simples. Todo site, blog, vídeo que fala sobre Redux ressalta sua simplicidade. No meu caso pessoal, achei que a terminologia utilizada pode confundir um pouco. Nomes como **actions**, **reducers** e **store**, por exemplo.

A grande inovação no desenvolvimento frontend não está no nível de componentes, mas no nível de gerenciamento de estado, segundo Max Lynch, cofundador do Ionic e poliglota em frameworks e linguagens de programação. Ele escreveu [um post bem interessante sobre Redux]() sobre Redux, citando também alguns benefícios, como a redução da quantidade de bugs por centralizar o gerenciamento de estado e a possibilidade de escrever componentes puros.

Descomplicando conceitos
------------------------

Pense no estado como um banco de dados, onde armazenamos tudo o que é importante para a aplicação.

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

Esse estado precisa ser armazenado em algum lugar, certo? Quem armazena esse estado, pela terminologia do Redux, é a **store**.

Cada aplicação Redux possui somente uma **store** e através dela conseguimos acessar todo o estado da aplicação.

Mas como alterar o estado do Redux?

Para demonstrar a intenção de alterar o estado devemos despachar uma **ação**. No geral depachamos ações quando alguma coisa acontece na aplicação, como um clique em um botão ou quando obtemos a resposta de uma chamada de API.

Por exemplo, se o usuário logou com sucesso no sistema, podemos despachar a seguinte ação, um objeto que possui um atributo ```type``` para representar o tipo da ação e um atributo ```payload``` para armazenar os dados que queremos introduzir no estado da aplicação:

``` javascript
{
  type: 'login_successfully',
  payload: {
    email: 'user@gmail.com',
    displayName: 'User'
  }
}
```

Ao despachar essa ação explicitamos a intenção de alterar o estado da aplicação. Então, para materializar esse nosso desejo, precisamos de um **reducer**, uma função que sabe como atualizar o estado.

Colocando tudo na mesma frase: Para modificarmos o estado da aplicação, que fica armazenado na **store**, devemos despachar uma **ação**, que é processada por uma função que sabe como modificar o estado, o **reducer**.

Vamos aprender agora como utilizar a API do Redux para colocar tudo isso para funcionar. Perceba que até agora ainda não escrevemos nenhum código além de objetos javascript simples.

Reducers para alterar o estado
------------------------------

Como falei anteriormente, o **reducer** sabe como atualizar o estado. Uma coisa interessante é que cada reducer é responsável por atualizar uma parte do estado!

No nosso exemplo, precisamos de um reducer para atualizar ```user``` e outro para atualizar ```posts```.

A primeira função da API do Redux que vamos utilizar é a ```combineReducers```, uma função auxiliar que liga cada reducer ao pedaço de estado que ele é responsável por gerenciar.

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

O reducer é uma função que recebe dois argumentos (a parte do estado que ele gerencia e uma ação), e retorna um novo estado.

O argumento ```state``` contém último estado que a função retornou, ou seja, o estado anterior. Na primeira execução o ```state``` não vai possuir um valor. Então é interessante criar um valor inicial:

<pre class="line-numbers" data-start="1" data-line="1-4,6"><code class="language-jsx">
const initialState = {
  email: null,
  displayName: 'Anonimous User',
};

export default (state = initialState, action) => {
  // logic to create newState here
  return newState
};
</code></pre>

Uma coisa que nunca podemos fazer no reducer é modificar seus argumentos. Os reducers devem ser bem sucintos e diretos, sem surpresas. Logo, nada de fazer chamadas a API ou utilizar funções não puras como ```Date.now()``` e ```Math.random()```.

Logo devemos realmente retornar um novo objeto, ao invés de reaproveitar a variável ```state```:

<pre class="line-numbers" data-start="1" data-line="14-18"><code class="language-jsx">
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
</code></pre>

Utilizamos o [spread operator](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Operators/Spread_operator) do ES6 para criar uma cópia do objeto ```state``` e popular alguns atributos. Atributos novos são adicionados e atributos existentes são sobrescritos.

A imutabilidade é uma característica importante para alcançar a previsibilidade do Redux. Se você se encontrar em um cenário mais complexo, recomendo utilizar uma lib como o [Immutable.js](https://facebook.github.io/immutable-js/) para isso.

Vamos falar agora sobe como criar ações.

Como criar ações
----------------

Vamos escrever uma função que retorna a ação que queremos despachar, também conhecida como **action creator**:


<pre class="line-numbers" data-start="1" data-line=""><code class="language-jsx">
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
</code></pre>

Mais na frente vamos voltar a esse código para implementar uma chamada de API assíncrona. Vamos deixar assim por enquanto.

Se a gente quiser implementar o log out, a lógica é bem parecida:

<pre class="line-numbers" data-start="1" data-line="15-17"><code class="language-jsx">
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
</code></pre>

E o reducer:

<pre class="line-numbers" data-start="1" data-line="20-25"><code class="language-jsx">
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
</code></pre>

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

Depois de feita a integração, não lidamos mais diretamente com a ```store```. A lib que faz a integração, como o **react-redux**, fica responsável por lidar com a ```store```, então não precisamos nos preocupar com isso.

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

<pre class="line-numbers" data-start="1" data-line=""><code class="language-jsx">
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
</code></pre>

Passamos a utilizar os atributos ```error``` e ```loading```.

Conclusão
---------

Neste post vimos os principais conceitos do Redux. Uma dúvida que tive quando comecei a utilizar Redux foi: "Por que mesmo utilizar o Redux? Poderia modificar o estado diretamente. Para que criar ações, reducers, etc?".

A resposta para isso vem quando integramos o Redux ao nosso framework javascript. Os componentes ficam muito mais limpos, com código muito mais fácil de ser mantido. O Redux traz ao nosso projeto mais previsibilidade, e com isso conseguimos diminuir a quantidade de bugs.

No próximo post vou explicar em mais detalhes a integração do Redux com React e disponibilizar o código fonte. Vamos ver na prática as vantagens do Redux, e explorar ainda mais sua API. Até lá!
