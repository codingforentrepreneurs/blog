---
title: Setup React
slug: setup-react

publish_timestamp: March 29, 2021
url: https://www.codingforentrepreneurs.com/blog/setup-react/

---


> Watch [this series](https://www.codingforentrepreneurs.com/projects/try-reactjs-2021) to build a React.js project from scratch.

Below is a reference to setup a React.js project on your local machine. We'll be updating it from time to time so please let us know in the comments if you have anything we should add.  For more React.js projects, posts, courses and more, check out [this link](https://www.codingforentrepreneurs.com/topics/react/).


## 1. Install Node.js

Download & Install [Node.js from the source](https://nodejs.org/en/download/). I recommend downloading the LTS unless you have a strong reason not to. At the time of this writing, it's version `14.16.20`,

After you run the standard installation your command line (Terminal or PowerShell) will have the following commands: 
- npm
- npx 


Open (or restart) Terminal/PowerShell and run:

```console
node --version
npm --version
npx --version
```

**node** is the command for using the `node` runtime or executing a file like `node project.js` 

**npm** is the **n**ode **p**ackage **m**anager. This makes it easy to install dependancies for any of your node-based projects. (Remember Node.js is a JavaScript runtime so Node.js is JavaScript).

**npx** is a tool for running Node packages. Personally, I use `npm` (& `yarn`) far more than `npx` even when building React.js projects.



## 2. Recommended. Install Yarn
Yarn is a popular dependency manager for creating Node.js (JavaScript) packages.

```
npm install --global yarn
```


## 3. Choose Project Parent Location
I create a `Dev` folder in my Users' root folder. It goes something like:
```
$ mkdir ~/Dev
$ cd ~/Dev
```
I like using this folder for my projects on my local system but you can choose where you'd like. Next, decide your project's name. For this, I'll call my project `try-reactjs` and it's final home will be `~/Dev/try-reactjs`



## 4. Optional. Use `npx` to create a React.js Project.
This an [official way](https://reactjs.org/docs/create-a-new-react-app.html#create-react-app) to bootstrap a new react application with some boilerplate code. Personally, I prefer using Parcel.js as discussed in section 5 below.

```
npx create-react-app try-reactjs
```
> This will take a few minutes the first time you run it.

##### Review the project

```
cd try-reactjs
ls -a
```

You should see:
```
.		.git		README.md	package.json	src
..		.gitignore	node_modules	public		yarn.lock
```

`package.json` is all of the project's requirements/dependencies as well as some commands for running things. Some default commands are: 
- `npm run start` / `npm start` -> Development build/watch
- `npm run build` / `npm build` -> Production build
- `npm install` -> Install dependencies in `package.json`


If `yarn` is installed, you can use:

- `yarn start` -> Development build/watch
- `yarn build` -> Production Build
- `yarn install` -> Install dependencies in `package.json`


> Troubleshooting installs. Every once and a while your installs will fail. When this happens, it's often great to just delete the `node_modules` directory and run `yarn install`/`npm install` again. This may save you some headache in the future. Lookout for `.cache` folders as well.


## 5. Recommended. Use [Parcel.js](https://parceljs.org/) to create a React.js project.


#### 1. Install Parcel.js
Following the [docs](https://v1.parceljs.org/) go ahead and:
```
yarn global add parcel-bundler
```
> If using `npm`, run: `npm install -g parcel-bundler`


#### 2. Navigate to project dir:
```
cd ~/Dev/
mkdir try-reactjs
cd try-reactjs
```
> this assumes you did not use `npx` above


#### 3. Initialize Project
```
yarn init -y
```
> You can also use `npm init`


#### 4. Create `index.html`

```html
<!DOCTYPE html>
<html>
<body>
   <div id='app'></div>
  <script src="./index.js"></script>

</body>
</html>
```
Note `index.js` in this html file as our entrypoint for our project.

#### 5. Create `index.js`
```javascript
import React from 'react';
import ReactDOM from 'react-dom';


const appDivEl = document.getElementById('app')

const App = props => {
   return <h1>Hello World</h1>
}
ReactDOM.render(<App />, appDivEl)
```

#### 6. Run `parcel watch`

```
parcel watch index.html
```

> This will install any dependancies your `index.js` file has and that includes any javascript modules you import. Cool huh?

As you can see, Parcel.js is a minimal version of a React.js app with incredible flexibility to add packages at will. `parcel watch` will automatically install packages for you.


## 6. Adding a Linter

A linter analyze our code and offers improvements, bug checks, and often reformats it for us. 

- VSCode [Official Linter](https://code.visualstudio.com/docs/nodejs/reactjs-tutorial#_linting)
- SublimeText [SublimeLinter](https://github.com/Flet/SublimeLinter-contrib-standard)
