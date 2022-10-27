---
title: AngularJS vs jQuery
slug: angularjs-vs-jquery

publish_timestamp: Sept. 22, 2016
url: https://www.codingforentrepreneurs.com/blog/angularjs-vs-jquery/

---


### AngularJS is a framework
### jQuery is a library

AngularJS is made to handle building rich web applications by providing all types of built-in features to make the development process faster while being better.

jQuery was made (a long time ago) to help add interaction to web pages. I would argue that jQuery was the first step into removing Flash from the web. jQuery provides a lot of great functions for manipulating the DOM but is far from a true framework. The developers of AngularJS as well as ReactJS recognized the fact that jQuery was really good at somethings but was really made as an add-on to other types of projects. In other words, it was a toolbox to add to your additional toolbox and allowed you to overcome writing your own javascript code (which can be complex).

AngularJS was created with the thought: what if HTML was invented today? What would it look like? A powerful feature in Angular is called "directives" and allows you to make your very own HTML tags "<your-whatever-tag></your-whatever-tag>" and have it do exactly what you want. Doing the same in jQuery would be a headache. AngularJS makes it almost as easy as writing a standard javascript function.

AngularJS is newer so it improves upon what was already created. This, I think, is probably the most important aspect of it in comparison to jQuery.

AngularJS is a framework and has these features:
- Two way data binding
   - This removes a lot of code that makes jQuery super crowded
- MVW pattern (MVC-ish)
   - This means it maps to Model-View which jQuery does not. Most Web Frameworks do some sort of MVC
- Template
   - There is probably a way to write templates in jQuery but Angular makes it easy. Like super easy.
- Custom-directive (reusable components, custom markup)-- see above
- REST-friendly
    - This is HUGE. Integrating with APIs is SO simple with Angular it's not even funny. It's why [Django + Angular](https://www.codingforentrepreneurs.com/projects/django-angularjs/) is so exciting.
- Deep Linking (set up a link for any dynamic page)
- Form Validation
- Server Communication
- Localization
- Dependency injection
- Full testing environment (both unit, e2e)

Read more [here](http://stackoverflow.com/questions/13151725/how-is-angularjs-different-from-jquery) -- lots of great discussion around it.
