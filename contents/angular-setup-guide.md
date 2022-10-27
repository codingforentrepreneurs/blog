---
title: Angular Setup, Install, &amp; Build Guide
slug: angular-setup-guide

publish_timestamp: Dec. 3, 2017
url: https://www.codingforentrepreneurs.com/blog/angular-setup-guide/

---


This is a step-by-step setup guide to setup Angular v.5, the latest version of Angular 2, on your computer. Adapted from docs on [Angular CLI](https://angular.io/docs/ts/latest/cli-quickstart.html).

### 1. Install Node.js from [here](https://nodejs.org/en/download/)
After downloading Node.js, the node package manager (`npm`) should automatically be installed.  Test it out by doing:

```
$ npm --version
```

### 2. Install Angluar CLI via [npm](https://docs.npmjs.com/):
```
$ npm install -g @angular/cli
```


### 3.  Navigate to project directory:
```
$ cd ~/Dev/
$ mkdir appDir && cd appDir 
$ ng new cfe-app
```
*`ng new` takes a minute to run.*

### 4. Navigate to project & run local server
```
$ cd /path/to/your/newly/created/app/
```
like

```
$ cd ~/Dev/appDir/cfe-app/
$ ng serve --open
```
This will automatically open [http://localhost:4200/](http://localhost:4200/)

***note**: `ng serve` command launches the server, watches your files for changes, and rebuilds the app as you save changes*

### 5. Edit project:
1. Open file in `appDir/cfe-app/src/app/app.component.ts`:
    ```
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.css']
    })
    export class AppComponent {
      title = 'Hello world!';
    }
    ```
2. Save `app.component.ts`
3. Within app root, Run:
    ```
    ng serve --open
    ```
    Or just navigate to [http://localhost:4200/](http://localhost:4200/)

    **app root** is in `/appDir/cfe-app/` for our app, 

4. With `ng serve` still running, make and save a new change in `app.component.ts`:
    ```
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.css']
    })
    export class AppComponent {
      title = 'Hi there again!';
    }
    ```
5. Open [http://localhost:4200/](http://localhost:4200/) -- see how it changed automatically? Yup it's almost like magic. Learn how to do this in plain [typescript here](https://www.codingforentrepreneurs.com/blog/typescript-setup-guide/) using webpack.


### 6. Test build
```
ng build
```
This creates a new folder called "/dist/" in your **app root** which is your entire app compiled and ready to ship.

Great optional flags:
- `--prod` this creates a production-ready version of your app
- `--output-path /to/your/path/` this changes the default path for where your Angular files should be built to
- `--output-hashing none` Adding this will remove the additional hash on your file name
- More options are [here](https://github.com/angular/angular-cli/wiki/build#options)

**Now, run production code locally** 
```
$ cd path/to/your/ng/proj
$ ng build --prod --output-hashing none
$ npm install -g http-server
$ cd dist
$ ls 
3rdpartylicenses.txt	inline.bundle.js	styles.bundle.css
favicon.ico		main.bundle.js
index.html		polyfills.bundle.js
$ http-server
Starting up http-server, serving ./
Available on:
  http://127.0.0.1:8080
  http://192.168.0.9:8080
Hit CTRL-C to stop the server
```
Now you can open [http://127.0.0.1:8080](http://127.0.0.1:8080) or [http://192.168.0.9:8080](http://192.168.0.9:8080)


====

#### Watch
Below covers v.4 but it's the same process for v5

<div style="position:relative;height:0;padding-bottom:56.25%"><iframe src="https://www.youtube.com/embed/1hyjLD7pk10?list=PLEsfXFp6DpzSUvTvnKaN8xmu4bRZIaawC?ecver=2" width="640" height="360" frameborder="0" style="position:absolute;width:100%;height:100%;left:0" allowfullscreen></iframe></div>
