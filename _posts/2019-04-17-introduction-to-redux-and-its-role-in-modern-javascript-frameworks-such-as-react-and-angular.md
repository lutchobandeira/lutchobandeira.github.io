---
layout: post
title: Introduction to Redux and its role in modern javascript frameworks such as React and Angular
thumbnail: /images/react-part-1.png
short_url: https://goo.gl/nJmg14
ref: react_3
lang: en
---

Redux is a predictable state container for javascript apps. Redux is simple, however I found that its terminology may be a bit confusing. Names like **actions**, **reducers** and **store**, for example.

The co-founder of Ionic and a polyglot in frameworks and programming languages, Max Lynch, quoted in his blog that the biggest innovation in frontend development isn't at the component level, but at the state management level. He wrote [an interesting post about Redux](https://medium.com/@maxlynch/redux-is-the-pivotal-frontend-innovation-a406736552cb), mentioning some of its benefits, like bugs reduction by centralizing state management and the possibility of writing pure components.

Making concepts simpler
-----------------------

Think of state as a database, where we store everything that is important for the application.

In a Redux app, the state is represented by a plain old javascript object:

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

This state needs to be saved somewhere, right? Who holds this state, by Redux terminology, is the **store**.

Every Redux application has only one **store** and we can access all the application state through it.

But how to update Redux state?

The first step is to dispatch an **action**. Redux actions are used to communicate that something happened at the application, such as clicks on buttons, form submissions, receiving API response,  and so on.

The action is simply a javascript object that has an attribute ```type``` to represent the type of the action and an attribute ```payload``` to hold the data we want to introduce in the application state.

For example, if the user has successfully logged in the system, we can dispatch the following action:


``` javascript
{
  type: 'login_successfully',
  payload: {
    email: 'user@gmail.com',
    displayName: 'User'
  }
}
```

Upon dispatching this action, we make clear our intent to update the application state. So in order to materialize this desire we need a **reducer**, a function that knows how to update the state.

Putting the words in the same sentence: In order to update the application state, which is hold by the **store**, we need to dispatch an **action**, that is processed by a function that knows how to update state, the **reducer**.

Let's learn how to use the Redux API in order to put all that to work. Note that we haven't written any code rather than plain old javascript objects until now.

Reducers to update state
------------------------

As I said previously, the **reducer** knows how to update state. It's interesting to note that every reducer is responsible to update a specific part of the state!

In our example, we need a reducer to update ```user``` and another one to update ```posts```.

The first function of Redux API that we will use is ```combineReducers```, a helper function that binds each reducer to the part of state it is accountable for.

The file ```reducers/index.js``` is following:

``` javascript
import { combineReducers } from 'redux';

export default combineReducers({
  user: UserReducer,
  posts: PostsReducer,
});
```

A draft of ```UserReducer``` looks like that, in the file ```reducers/UserReducer.js```:

``` javascript
export default (state, action) => {
  // logic to create newState here
  return newState
};
```

The reducer is a function that takes two arguments (the state it manages and an action), and returns a **new state**. This is very important! We will always return a new object rather than changing ```state```.

The argument ```state``` holds the last state returned by the function, in other words, the previous state. It is interesting then to assign an initial value:

<pre class="line-numbers" data-start="1" data-line="1-4,6"><code class="language-jsx">
const initialState = {
  email: null,
  displayName: 'Anonymous User',
};

export default (state = initialState, action) => {
  // logic to create newState here
  return newState
};
</code></pre>

A thing we must never do is to change its arguments. Reducers should be very succinct and direct, no surprises. Therefore, we must avoid making API calls or use non-pure functions like ```Date.now()``` and ```Math.random()``` inside reducers.

Therefore, we really must to return a new object rather than reuse the variable ```state```:

<pre class="line-numbers" data-start="1" data-line="14-18"><code class="language-jsx">
import {
  LOGIN_SUCCESSFULLY,
} from '../actions/types';

const initialState = {
  email: '',
  displayName: 'Anonymous User',
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

We have used the [spread operator](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Operators/Spread_operator) of ES6 to create a copy of the object ```state``` and populate some of its attributes. New attributes are added and existing attributes are overwritten.

Immutability is an important feature in order to achieve the predictability of Redux.

Let's talk now about how to create actions.

How to create actions
---------------------

Let's write a function that returns the action we want to dispatch, also known as **action creator**.


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

We will come back to this code later on to implement an asynchronous API call. Let's leave the code this way for a while.

If we want to implement logout functionality, the logic is very similar:

<pre class="line-numbers" data-start="1" data-line="15-17"><code class="language-jsx">
import {
  LOGIN_SUCCESSFULLY,
  LOGGED_OUT
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
  return { type: LOGGED_OUT };
};
</code></pre>

And the reducer:

<pre class="line-numbers" data-start="1" data-line="20-25"><code class="language-jsx">
import {
  LOGIN_SUCCESSFULLY,
  LOGGED_OUT
} from '../actions/types';

const initialState = {
  email: '',
  displayName: 'Anonymous User',
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
    case LOGGED_OUT:
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

Putting Redux to work
---------------------

Let's learn more about Redux API.

The function that creates a **store**, which is responsible for holding the state, is called **createStore** and has the following structure:

``` javascript
createStore(reducer, [preloadedState], [enhancer])
```

Let's use only the first argument for a while:

``` javascript
import { createStore } from 'redux';
import reducers from '.reducers';

const store = createStore(reducers)

// integration with some framework here
```

Having the variable ```store``` we can implement the integration between Redux and the javascript framework we use.

To integrate with React, for example, we use the lib [react-redux](https://github.com/reactjs/react-redux).

Once the integration is done, we don't handle ```store``` directly. The library we used for the integration, like **react-redux**, becomes responsible for dealing with the **store**, so we don't need to be concerned about that.

Integrating with ```redux-thunk``` to make asynchronous calls
-------------------------------------------------------------

The function ```createStore``` is also used to integrate with third-party middlewares.

For example, to implement asynchronous API calls, we can use an interesting lib called [redux-thunk](https://github.com/gaearon/redux-thunk), a middleware used to delay the dispatch of an action. The setup is following:

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

```applyMiddleware``` is an enhancer built in Redux, used exactly to implement integrations with third-party libs.

The next step to to change the way we return the action from the action creator.

The idea of using **redux-thuk** is to **make action creator return a function that creates an action**, rather than returning the action directly.

The draft of the action creator of ```login``` with redux-thuk looks like that:

``` javascript
export const login = () => {
  return (dispatch) => {
    loginWithApi().then(() => dispatch({ type: LOGIN_USER_SUCCESS }));
  };
};
```

The function ```login``` returns now a function that receives ```dispatch``` as argument. We use the function ```dispatch``` to dispatch the action when we are done. This give us a lot of flexibility.

Actions are used to communicate what is going on the system, remember? We could dispatch an action to tell when an user has started his login process and another one to tell when the login process has finished with success or with failure.

The code looks like that:

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

At the above code, we communicate when the user decided to login, with the action of type ```LOGIN_USER``` and afterwards we dispatch ```LOGIN_USER_SUCCESS``` or ```LOGIN_USER_FAIL```, depending on the response from the API call.

This way we can do cool things, like display a loading spinner between the API call and its response.

We need to change the reducer to implement this:

<pre class="line-numbers" data-start="1" data-line=""><code class="language-jsx">
import {
  LOGIN_SUCCESSFULLY,
  LOGGED_OUT
} from '../actions/types';

const initialState = {
  email: '',
  displayName: 'Anonymous User',
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
    case LOGGED_OUT:
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

We have introduced the attributes ```error``` and ```loading```.

Conclusion
----------

In this post we talked about the main concepts of Redux. A question I had when I started using Redux was: "Do I really need to use Redux? I could change state directly. Why create actions, reducers, etc?".

The answer to that comes when we integrate Redux with the javascript framework we use. Components become cleaner, with code much more easier to be maintained. Redux brings more predictability to our project, and this way we manage to reduce the amount of bugs.

In the next blog post I will show how to integrate Redux and React and share the source code. Let's take a look at the advantages of Redux, and explore its API. See you there!