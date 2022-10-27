---
title: Setup &amp; Install Ionic
slug: setup-install-ionic

publish_timestamp: Dec. 3, 2017
url: https://www.codingforentrepreneurs.com/blog/setup-install-ionic/

---

Ionic is a framework for building native mobile apps using web technologies. The related docs are [here](https://ionicframework.com/docs/).

Ionic is built on Angular so you'll definitely want to know [some Angular](https://www.codingforentrepreneurs.com/tags/angular/) and [TypeScript](https://www.codingforentrepreneurs.com/projects/getting-started-typescript/) before jumping in. 

This guide is to help you install Ionic and even run a basic build. If you know Angular, you can probably already build a solid Ionic app with little trouble. So after you do some of the basic installs, you might play around with the code generated.

==================

### 1. Install Node.js from [here](https://nodejs.org/en/download/)
After downloading Node.js, the node package manager (`npm`) should automatically be installed.  Test it out by doing:

```
$ npm --version
```


### 2. Install Ionic & Cordova 

```
npm install -g cordova ionic
```


#### 3. Create a new App

```
$ cd ~/Dev # or whatever you'd liek
$ mkdir new-proj
$ cd new-proj 
$ ionic start cfeApp sidemenu
```
Start options include:
- ionic start <appname> blank
- ionic start <appname> tabs
- ionic start <appname> sidemenu


#### 4. Run Newly Created App
```
$ ionic serve --open
```


#### 5. Build App
```
$ ionic build --prod 
```

#### 6. Test Web App

```
$ npm install -g http-server

$ http-server www/

```

#### 7. Add Platform (optional step)

**iOS**
- Install [XCode](https://developer.apple.com/xcode/)
- Run
```
$ xcode-select install
```

Now, add to your project
```
$ cd path/to/your/ionic/proj 
$ ionic cordova platform add ios
```

Emulate
```
$ ionic cordova emulate ios
```

Build & Release
```
$ ionic cordova build ios --prod --release
```

**Android** 
- Follow [these requirements](https://cordova.apache.org/docs/en/latest/guide/platforms/android/#installing-the-requirements)
Then:

Add
```
$ ionic cordova platform add andriod
```

Emulate
```
$ ionic cordova emulate andriod
```

Build & Release
```
$ ionic cordova build andriod --prod --release
```