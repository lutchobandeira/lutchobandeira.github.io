---
layout: post
title: Como fazer um feed de notícias em React.js
ref: first
lang: pt
---

Este é o início de uma série de posts sobre React.js. O objetivo é contruir um feed de notícias com atualizações em tempo real. Além do front-end, vamos implementar também um back-end em Rails. Este primeiro post é uma introdução ao React.js. Então mesmo que você não seja um programador Rails, este post pode ser útil pra você!

Para quem está iniciando em React pode parecer complicado o número de tecnologias e ferramentas disponíveis. Então vamos deixar as coisas bem simples aqui para facilitar o aprendizado.

Vamos construir a aplicação passo a passo e vamos introduzindo alguns conceitos teóricos do React no momento mais oportuno.

Criando o projeto React.js
--------------------------

A maneira mais fácil de criar um projeto React é utilizando o [create-react-app](https://github.com/facebookincubator/create-react-app). O ```create-react-app``` traz tudo que precisamos para nosso projecto React:

- webpack - um module bundler que trabalha muito bem com o npm. Ele também inclui um web server e um file watcher.
- Babel - um compilador javascript que oferece suporte a última versão do javascript mesmo que os browsers ainda não ofereçam.
- Autoprefixer - um plugin que adiciona os prefixos de CSS dos browsers automaticamante.
- ESLint - uma ferramenta que analisa o código e aponta eventuais problemas.

Vamos criar o projeto:

{% highlight bash %}
$ npm install -g create-react-app
$ create-react-app news-feed
{% endhighlight %}

Feito isso já podemos entrar na pasta e iniciar nosso servidor:
{% highlight bash %}
$ cd news-feed/
$ npm start
{% endhighlight %}

Pronto! Nosso servidor já está disponível em [http://localhost:3000](http://localhost:3000) (ou em outra porta se a porta 3000 já estiver em uso).

Componentes React
-----------------

O ```create-react-app``` criou um componente raíz chamado ```App```. Vamos aproveitar para introduzir alguns conceitos de React. Dê uma rápida olhada no código abaixo:

{% highlight console %}
src/App.js
{% endhighlight %}

{% highlight typescript linenos %}
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  render() {
    return (
      <div className="App">
        <div className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h2>Welcome to React</h2>
        </div>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>
      </div>
    );
  }
}

export default App;
{% endhighlight %}

**O que essa sintaxe estilo HTML/XML está fazendo dentro do javascript?** A primeira vista isso pode parecer assustador.

Esse é o ```JSX```, uma sintaxe que o React recomenda para descrever como a UI deve ser exibida. O ```JSX``` é uma extensão ao javascript, mas no estilo ```XML```.

Então podemos usar expressões javascript dentro do ```JSX```. No trecho ```<img src={logo} className="App-logo" alt="logo" />``` vemos que expressões javascript são passadas entre chaves e strings entre aspas duplas.

Poderíamos ter escrito esse mesmo código usando somente javascript. E na verdade é isso que o ```Babel``` faz, ele converte os elementos ```JSX``` em chamadas ```React.createElement()```.

Nosso código ficaria assim:

{% highlight typescript linenos %}
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  render() {
    return React.createElement(
      'div',
      { className: 'App' },
      React.createElement('div', { className: 'App-header' },
        React.createElement('img', { src: logo, className: 'App-logo', alt: 'logo' }),
        React.createElement('h2', null, 'Welcome to React')
      ),
      React.createElement('p', { className: 'App-intro' },
        'To get started, edit ',
        React.createElement('code', null, 'src/App.js'),
        ' and save to reload.'
      )
    );
  }
}

export default App;
{% endhighlight %}

