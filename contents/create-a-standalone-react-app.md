---
title: Create a Standalone React App
slug: create-a-standalone-react-app

publish_timestamp: March 19, 2019
url: https://www.codingforentrepreneurs.com/blog/create-a-standalone-react-app/

---


This post will answer the following questions:
- How do we build a standalone React app without installing a bunch of npm packages?
- How do we ensure it's minified?
- How do we make a stand alone component (e.g. 'like' button) that can be embedded nearly anywhere and over and over again?


We'll do this by creating a standalone react app using `React` and `Babel`.


So our react imports will be this:

```html
<script src="https://unpkg.com/react@16/umd/react.production.min.js" crossorigin></script>
<script src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js" crossorigin></script>
```

Instead of:

```console
npx create-react-app
```
### [Code](https://github.com/codingforentrepreneurs/Create-a-Standalone-React-App)
The [code](https://github.com/codingforentrepreneurs/Create-a-Standalone-React-App) is exactly what we do in this post with everything working as expected.

### Watch
<iframe width="560" height="315" src="https://www.youtube.com/embed/jWUlvAv0LOA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### 1. Package.json
Navigate to a blank directory so we can create our app. I'm calling mine `standalone_react` (riveting name I know)

```
cd /path/to/your/local/dev
mkdir standalone_react
cd standalone_react
touch package.json
```

In your `package.json` you'll want to add the following (at bare minimum):

```json
{
  "name": "standalone-react",
  "version": "0.0.1",
  "main": "app.js",
  "scripts": {
    "start": "open http://localhost:8888/ && python3 -m http.server 8888",
    "build": "npx babel app.js --out-file app.min.js --presets minify --watch"
  },
  "author": "",
  "license": "ISC",
  "keywords": [],
  "description": "",
  "devDependencies": {
    "@babel/cli": "^7.2.3",
    "@babel/core": "^7.2.2",
    "@babel/plugin-proposal-class-properties": "^7.3.4",
    "@babel/plugin-transform-react-jsx": "^7.2.0",
    "babel-minify": "^0.5.0",
    "babel-preset-minify": "^0.5.0"
  }
}
```
> You'll see `python3 -m http.server` in there because I'm testing this with a basic Python server to keep down the 'development package bloat'. If you want a npm http server consider using `npm install --save-dev http-server` and replacing `python3 -m http.server 8888` with `http-server -P 8888`


### 2. `.babelrc` Configuration

```
cd /path/to/your/local/dev/standalone_react
touch .babelrc
```

In `.babelrc` we're going to add:

```
{
  "plugins": ["@babel/plugin-transform-react-jsx", "@babel/plugin-proposal-class-properties"],
  "env": {
    "production": {
      "presets": ["minify"]
    }
  }
}
```

This gives us the ability to use arrow functions `myFunc = () => {}` as well as create a minified version of our app's code (aka `app.min.js`) like `create-react-app`'s build does.



### 3. Index.html

This is production-ready html. I'll continue to test with this. If you need a development version of `react` just replace `production.min` with `development` in the react urls below.

```
cd /path/to/your/local/dev/standalone_react
touch index.html
```

In `index.html` add:
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Standing Alone React</title>
  </head>
  <body>


    <div class="cfe-app" data-userid='1'></div>

    <!-- Load React. -->
    <script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
    <script src="https://unpkg.com/react@16/umd/react.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js" crossorigin></script>

    <!-- Our React App (notice the type is babel) -->
    <script src="./app.min.js" type='text/babel'></script>

  </body>
</html>
```


### 4. Our React App.
This, of course, is a simple app file. But it has all of the features as you'd expect building a v16 React.js app. 

```
cd /path/to/your/local/dev/standalone_react
touch app.js
```

In `app.js` add:
```
'use strict'

const e = React.createElement;

class HelloWorld extends React.Component {

    handleClick = () => {
        alert("Hello world")
    }

    render () {
        const {userid} = this.props;
        return <h1>Hello World <span onClick={this.handleClick}>{userid}</span></h1>
    }
}

// Find all DOM containers, and render our component into them.
var containers = document.querySelectorAll('.cfe-app')
containers.forEach(domContainer => {
    // Read the user ID from a data-* attribute.
    const userid = domContainer.dataset.userid
    // render the component into the DOM
    ReactDOM.render(
      e(HelloWorld, { userid: userid}),
      domContainer
    )
});
```


### 5. Build and Test
Our build command will create the `app.min.js` file for us (as referenced in `index.html`) and it will watch for any changes by default. You'll run two commands at once both `npm run build` and `npm run start` to both develop and view our app.


```
cd /path/to/your/local/dev/standalone_react
```

```console
npm run build
```
This will also watch for changes. Open another terminal window and run:

```
cd /path/to/your/local/dev/standalone_react
```

```
npm run start
```
