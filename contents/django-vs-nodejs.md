---
title: Django vs Node.js
slug: django-vs-nodejs

publish_timestamp: April 5, 2021
url: https://www.codingforentrepreneurs.com/blog/django-vs-nodejs/

---


In this post, I'll address a common question I get:

***Which is better Django or Node.js?***

On the surface, this question may seem legitimate but it is actually a clear indication that whomever is posing the question lacks the fundamental understanding of what each is.


### Node.js is a runtime, Python is a runtime

Consider the two files:


`hello.js`
```javascript
console.log('hello world')
```

`hello.py`
```python
print('hello world')
```

If you run
```
node hello.js
```

and
```
python hello.py
```

You should get the same result: 

```console
hello world
```

This is what I mean by runtime. You can 'run' a file (or module) with **Node.js** or **Python** (and many other languages).

Let's try something a bit different:

```
python hello.js
```
```
node hello.py
```

What happens? Do you still get the same result as above? 


### Django is a Web Framework
By design, Django has limited capability. Django will not be used as a control module to launch a rocket to Mars. Django will not be used to create animations that power the next Pixar film. But since Django is Python, yes, Django *could* do these things but then Django looks less and less like, well, Django and more like a Python application.


What's a web framework you might ask?

Think of a web framework as a tool for a server (aka a remote computer) to send data back and forth to you. This data is often delivered via HTTP on links like https://www.nasa.gov or https://www.spacex.com/. The data comes in a form known as "html" which just makes it easy for your browser to read something back to you.

Imagine opening a text document on your local computer. Now imagine opening that same document with your toaster. Seems silly right?

The web has a bunch of standard practices called protocols that enable the passing of data in a logical, and hopefully secure, way. 

In the early days of the web, going to a website was much like asking your computer to open a text document -- it was easy and simple. Now, websites are expected to remember things (your username, email, password, cat videos, and all kinds of other data). This is is why tools started to emerge to help remember this data. Wordpress is one of these tools. At it's core, Wordpress just helps manage data that's remembered (aka stored in a database) so it is presented in a logical way as the site owner intends it to be (hopefully). Django is another such tool with *less* features out of the box although there are still *many, many* features.

As people wanted websites to remember more things (how many images do you have backed up in Google Photos, Apple photos, instagram, twitter, facebook ?), the computers that ran these computers needed to become more advanced; they needed to handle more requests for data from people all around the globe.


As the websites start to remember more, the data becomes more fractured and more specific. Why does Asana exist? Slack? LinkedIn? Each one of these sites are dedicated to remembering and displaying data in a very specific way to handle a very specific use case.

So Wordpress cannot run LinkedIn without major modifications. This is why Django exists. This is why web application frameworks exist.

If you're going to build a new table in your house, how would you do it? Run down to the hardware store and buy the materials you need right? Or would you go to the store and just buy a table that just needs to be assembled? 

Wordpress is the table that needs to be assembled.
Web frameworks are the hardware store with all the tools you need to design, cut, and assemble what you want and need.

Django is one such web framework written in Python. It's a collection of tools (aka pre-written code) that *can* work in harmony to deliver a cutting edge website that remembers data and displays as you decide. 

Node.js has web frameworks as well. Express.js might be the most commonly used one, Adonis.js seems to be the most fully featured (much like Django but in JavaScript).



### So is Django really just a Python Web Framework?

Yes and no. Technically speaking, Django is mostly Python. Django also has a key component for delivering HTML data via *Django Templates* and the *Django Template Engine*. If you've used Shopify's Liquid or Jinja in Flask, Django Templates are very similar. The templates themselves are not Python and they are not exactly HTML either -- it's a hybrid that is closer to PHP than anything else I can think of.

Django also has a huge community of supporters (developers, companies, educators, students, etc) that really make Django keep ticking. This community often supports one another on improving the tool we all love so well. This community is likely the reason Django is still around and still cutting edge. Instagram, NASA, & Pinterest have used Django for years.



### Can Node.js be a web framework?

Yes! Absolutely. You can write a bunch of JavaScript code to reproduce exactly what Django does technically. To get to a critical mass with an embedded community, that's another story.

Node.js already has a few popular web frameworks:
- Express.js -- similar to Flask & FastAPI in Python)
- Adonis.js -- the closest "batteries included" JavaScript web framework to Django
- Meteor.js
- Nest.js
> New web frameworks come out nearly daily so yeah, there's a lot for both Python and JavaScript.



### But I thought JavaScript was just for the browser?
JavaScript started in the browser, Node.js is what broke it free from the browser so you could use it as a runtime. Pretty neat huh?


### Can Django and Node work together?
Yes, absolutely! There are so many different ways to accomplish this. If you want them to run on the same web server for the same site, you can use NGINX to do this; otherwise you can have Django on one server and Node.js running on another server.


### Can Django and JavaScript work together?
Browser-based JavaScript and Django are good friends as well. Technologies like React.js, Vue.js, Angular, jQuery, and event vanilla JavaScript, can easily integrate into Django's built-in ecosystem.


### How do you integrate Django and JavaScript in the Browser?
There are a few ways that I'll mention:

- Via RESTful APIs
- Via WebSockets
- Via Graphql

**RESTful APIs** are an incredibly common way for web applications to communicate with any other software tool. JavaScript and Django are no different.

**WebSockets** In order for Django to leverage web sockets to their full potential, they have to use JavaScript. WebSockets offer near real-time communication with a backend. Unlike traditional RESTful APIs, WebSockets offer an open connection that allows for back and forth exchange of data.

**Graphql** is a lot like raw `SQL` queries in that you can make them incredibly flexible so that you don't have to think of your structure in advance like you do with RESTful apis. 

The above tools can be used in many/most web frameworks and are often powerful ways to grow your project.


Let me know if you have other questions in the comments below. This post will be updated as feedback comes in. 

Cheers!
