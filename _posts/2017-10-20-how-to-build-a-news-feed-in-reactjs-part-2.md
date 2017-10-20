---
layout: post
title: How to implement a news feed in React.js - Part 2
thumbnail: /images/react-part-1.png
short_url: https://goo.gl/66sykq
ref: react_2
lang: en
---

This is the second post in a series of posts about React.js. In the [first post](http://lutchobandeira.com/how-to-build-a-news-feed-in-reactjs-part-1/), we have developed a news feed in React. In this post we will:

- Create a back-end in Rails (source code available at this [repository on GitHub](https://github.com/lutchobandeira/news_feed_rails));
- Build the communication beetween the React app and the Rails app.

The source code of the React project we have developed on part 1 [is available in this repo](https://github.com/lutchobandeira/news-feed-react).

The changes we made at the React project are [in the same repo but in another branch](https://github.com/lutchobandeira/news-feed-react/commits/backend_integration).

Different ways of integrating React and Rails
---------------------------------------------

One possibility is to write React code inside a Rails project. As of [Rails 5.1 release](http://guides.rubyonrails.org/5_1_release_notes.html), it's easier to work with javascript frameworks within a Rails application.

One of the new features of Rails 5.1 is the [Yarn](https://yarnpkg.com/en/) support and the Webpack support through gem [webpacker](https://github.com/rails/webpacker).

If you want to use this approach in a Rails 4 project, you can use the gems [react-rails](https://github.com/reactjs/react-rails) and [react_on_rails](https://github.com/ shakacode / react_on_rails).

Another possibility is to keep React code and Rails code in separate projects and make them talk to each other through an API. This is the approach we are going to use in this post.

Creating a Rails API application
--------------------------------

As of Rails 5 it's possible to create API-only applications:

``` bash
$ rails new news_feed_rails --api
```

When this command runs, Rails creates a lighter application, without the resources that would be used by traditional applications with HTML views.

The application is configured to use only the necessary middlewares. With this setup,  ```ApplicationController``` inherits from ```ActionController::API```, which in turn imports a smaller number of modules.

The Rails application we are going to create is very simple. It has only the ```Post``` model:

``` bash
$ rails g model post category:integer content:string
```

We will add an ```enum``` for the categories attribute and leave category and content as required fields:

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

Let's take care of the controller:

``` bash
$ rails g controller posts
```

We need two actions in our controller: one to list the posts and another one to create a new post:

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

Finnally we update ```routes.rb```:

<pre class="line-numbers "><code class="language-ruby">
Rails.application.routes.draw do
  resources :posts, only: [:index, :create]
end
</code></pre>

Our API is done! But since we are going to make the requests from the React application, we must enable requests from other domains.

Rails 5 makes things easier to allow CORS (cross-origin HTTP request). Just uncomment the gem [rack-cors](https://github.com/cyu/rack-cors) in ```Gemfile```:

``` ruby
gem 'rack-cors'
```

And uncomment the contents of the file ```config/initializers/cors.rb```:

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

To make things simpler, in ```origins '*'``` we allow requests from all domains.

Let's start the app at the port 3001 (in order to let the React app running at port 3000):

``` bash
$ rails s -p 3001
```

Done!

How to consume an API from a React app
--------------------------------------

Before we begin, let's navigate to the React project folder and start the server:

``` bash
$ npm start
```

In the javascript world there are a bunch of ways of sending HTTP requests, [as you can see in this article](https://hashnode.com/post/5-best-libraries-for-making-ajax-calls-in-react-cis8x5f7k0jl7th53z68s41k1).

In this post we will use the [Fetch API](https://developer.mozilla.org/pt-BR/docs/Web/API/Fetch_API/Using_Fetch), which is available as of Firefox 39 and Chrome 42.

The Fetch API provides the method ```fetch()```, it has only one required argument, the URL of the resource we want to access, and returns a [Promisse](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Promise) with the response.

Let's start by adding the method ```fetchPosts()``` to the component ```Feed```:

``` jsx
fetchPosts() {
  fetch('http://localhost:3001/posts').then((response) => {
    return response.json();
  }).then((posts) => {
    this.setState({ posts });
  });
}
```

Note the we use ```fetch()``` to make the request. We deal with the Promisse in order to get the response in the JSON format through the call to ```response.json()``` and finally we have an array of posts, which we use in order to update the state of the ```Feed``` component.

Using React lifecycle methods
-----------------------------

React components have a lot of [lifecycle methods](https://reactjs.org/docs/react-component.html) we can override when we want to run a specific code, as an API call, at particular times.

Let's override ```componentWillMount()``` in order to make our request. This method runs before mounting occurs, before the call to ```render()```:

``` jsx
componentWillMount() {
  this.fetchPosts();
}
```

Let's change the code to initialize the list of posts with an empty array (the former implementation loads the list of posts from the localStorage):

<pre class="line-numbers" data-start="17" data-line="5"><code class="language-jsx">
class Feed extends Component {
  constructor(props) {
    super(props);
    this.state = {
      posts: [],
      filteredPosts: []
    }
</code></pre>

Our News Feed already loads posts from the API!

However, it would be interesting if we repeat the request from time to time in order to load new posts without the need to refesh the page. Let's create the method ```startPolling``` then:

``` jsx
startPolling() {
  this.timeout = setTimeout(() => this.fetchPosts(), 10000);
}
```

Note that we store the timer used by the method [setTimeout()](https://developer.mozilla.org/en-US/docs/Web/API/Window/setTimeout) in ```this.timeout```. We will use this reference to cancel the timer when component unmounts:

``` jsx
componentWillUnmount() {
  clearTimeout(this.timeout);
}
```

We need to change the method ```fetchPosts()``` to call ```startPolling()``` when the first request finished:

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

This way, the flux of calls is something like this: ```componentWillMount()``` -> ```fetchPosts()``` -> ```startPolling()``` -> ```fetchPosts()``` -> (...).

Making POST requests
--------------------

In order to make a POST request, we pass a second argument to the method ```fetch()```:

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

Here we make a POST request and update Feed optimistically though the call to ```setState()```, in order to leave the UI more fluid.

In order to add category correctly, we must change the list of categories to match the ```enum``` of the model ```Post``` from the Rails app. In the code we wrote in the previous post, the first character of each category name was uppercase. We will be fine if we change everything to downcase:

<pre class="line-numbers" data-start="4"><code class="language-jsx">
const categories = ['world', 'business', 'tech', 'sport'];
</code></pre>

Displaying validation errors
----------------------------

But what if we have validation errors? In this case, it would be nice to:

- display validation error next to the corresponding field;
- remove from the list of posts the new post we have added optimistically.

Let's start by changing the method ```handleNewPost()```:

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

Take a look at the lines 65-74. Here we write the code that treats the errors. Remember that the Rails API returns a list of errors if the model has validation errors. If we have validation errors, we update the state with the list of errors and we reset the list of posts (lines 67 and 68).

At the lines 52 and 53 we store the current list of posts, it will be used if the model has validatin errors, and we passed ```this``` to the variable ```context```. This is a trick to use the reference ```this``` of ```Feed``` inside the callback.

Let's initialize the errors object at the state:

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

Before changing ```PostForm```, let's pass the list of errors to it via a prop. The change to the  ```Feed``` component is following:

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


Finally we update the component ```PostForm``` in order to display validation errors:

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

At the lines 133-136, we put in the object ```errors``` the first error of each attribute, if there are any.

Our implementation is finished! You can see the code running in this [CodePen](https://codepen.io/lutchobandeira/pen/gGEXZz){:target="_blank"} here.

Next Steps
----------

Our code works, but it looks like the ```Feed``` component is doing too many things. In addition to having the responsibility to take care of its state and props, the component is concerned with making requests to an external server.

In the next post we will learn how to manage the data of our React app using [redux](http://redux.js.org/). See you there!
