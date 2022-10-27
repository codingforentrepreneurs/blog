---
title: AJAXify Django Forms
slug: ajaxify-django-forms

publish_timestamp: July 6, 2017
url: https://www.codingforentrepreneurs.com/blog/ajaxify-django-forms/

---


AJAX, short for asynchronous JavaScript and XML, is a technology that enables asynchronous web applications. Basically, it makes your web app run without the need to reload the page every time an action happens.

Think about the Facebook Like button, you click the button and it shows that you've liked the item. The page doesn't reload but the data is saved. That's the key. 

### Ajax = Save or Get Data without page reload.

So how do you do it? 

AJAX is a client-side technology because it requires the client-side to communicate with the server-side. So basically, JavaScript needs to talk to Django. 

So we need to setup our Django backend to work with the frontend. 

#### Guide prerequisites:
- Django Installed [windows](https://www.codingforentrepreneurs.com/blog/install-python-django-on-windows/) | [mac or linux](https://www.codingforentrepreneurs.com/blog/install-django-on-mac-or-linux/)
- [Try Django (latest)](https://www.codingforentrepreneurs.com/projects/try-django-111/) complete

#### Reference code on [github](https://github.com/codingforentrepreneurs/AJAXify-Django-Forms)
#### Watch the [video](#video)
--------

### **Setup Django Backend**

#### 1. Django form
```
# yourapp.forms.py
from django import forms

class JoinForm(forms.Form): # or forms.ModelForm
    email = forms.EmailField()
    name = forms.CharField(max_length=120)

```

#### 2. Standard Form View
```
# yourapp.views.py

from django.views.generic import FormView

from .forms import JoinForm


class JoinFormView(FormView):
    form_class = JoinForm
    template_name  = 'forms/ajax.html'
    success_url = '/form-success/'
```

#### 3. Update URL routing
```
# yourproject.urls.py
from django.conf.urls import url
from django.contrib import admin

from yourapp.views import JoinFormView
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^join/', JoinFormView.as_view()),
]
```

#### 4. Ajax-fied Form View

```
# from step 2

from django.http import JsonResponse

class JoinFormView(FormView):
    form_class = JoinForm
    template_name  = 'forms/ajax.html'
    success_url = '/form-success/'

    def form_invalid(self, form):
        response = super(JoinFormView, self).form_invalid(form)
        if self.request.is_ajax():
            return JsonResponse(form.errors, status=400)
        else:
            return response

    def form_valid(self, form):
        response = super(JoinFormView, self).form_valid(form)
        if self.request.is_ajax():
            print(form.cleaned_data)
            data = {
                'message': "Successfully submitted form data."
            }
            return JsonResponse(data)
        else:
            return response

```

#### 5. Create AJAX Mixin 

```
# yourapp.mixins.py

class AjaxFormMixin(object):
    def form_invalid(self, form):
        response = super(AjaxFormMixin, self).form_invalid(form)
        if self.request.is_ajax():
            return JsonResponse(form.errors, status=400)
        else:
            return response

    def form_valid(self, form):
        response = super(AjaxFormMixin, self).form_valid(form)
        if self.request.is_ajax():
            print(form.cleaned_data)
            data = {
                'message': "Successfully submitted form data."
            }
            return JsonResponse(data)
        else:
            return response
    
```
#### 6. Update Ajax-fied Form View

```
# yourapp.views.py 

from .mixins import AjaxFormMixin

class JoinFormView(AjaxFormMixin, FormView):
    form_class = JoinForm
    template_name  = 'forms/ajax.html'
    success_url = '/form-success/'
```

--------

### **Setup Frontend**


#### 1. Base.html on Django
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Base Template</title>
  </head>
  <body>
    <div>
    {% block content %}{% endblock content %}
    </div>

    <!-- jQuery (required for this Ajax tutorial) -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
  </body>
</html>

```



#### 2. Django Template
```
{% extends "base.html" %}
{% comment "template_location" %}
    'forms/ajax.html'
{% endcomment %}
{% block content %}
    <form class='my-ajax-form' method='POST' action='.' data-url='{{ request.build_absolute_uri|safe }}'>
        {% csrf_token %}
        {{form.as_p|safe}}
        <button type='submit'>Submit</button>
    </form>
{% endblock content %}
```


### 3. The Required jQuery (JavaScript code):
```
<script>
$(document).ready(function(){
    var $myForm = $('.my-ajax-form')
    $myForm.submit(function(event){
        event.preventDefault()
        var $formData = $(this).serialize()
        var $thisURL = $myForm.attr('data-url') || window.location.href // or set your own url
        $.ajax({
            method: "POST",
            url: $thisURL,
            data: $formData,
            success: handleFormSuccess,
            error: handleFormError,
        })
    })

    function handleFormSuccess(data, textStatus, jqXHR){
        console.log(data)
        console.log(textStatus)
        console.log(jqXHR)
        $myForm.reset(); // reset form data
    }

    function handleFormError(jqXHR, textStatus, errorThrown){
        console.log(jqXHR)
        console.log(textStatus)
        console.log(errorThrown)
    }
})
</script>

```

#### 4. Enable Django CSRF-ready AJAX Calls
This is directly from the Django [docs](https://docs.djangoproject.com/en/dev/ref/csrf/#ajax). Put this code before any jQuery that uses AJAX as a catch-all for cross site request forgery protection (CSRF).

```
// using jQuery
function getCookie(name) {
    var cookieValue = null;
    if (document.cookie && document.cookie !== '') {
        var cookies = document.cookie.split(';');
        for (var i = 0; i < cookies.length; i++) {
            var cookie = jQuery.trim(cookies[i]);
            // Does this cookie string begin with the name we want?
            if (cookie.substring(0, name.length + 1) === (name + '=')) {
                cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                break;
            }
        }
    }
    return cookieValue;
}

var csrftoken = getCookie('csrftoken');

function csrfSafeMethod(method) {
    // these HTTP methods do not require CSRF protection
    return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
}
$.ajaxSetup({
    beforeSend: function(xhr, settings) {
        if (!csrfSafeMethod(settings.type) && !this.crossDomain) {
            xhr.setRequestHeader("X-CSRFToken", csrftoken);
        }
    }
});
```


### Video
<iframe width="560" height="315" src="https://www.youtube.com/embed/zojnkKGRXp0" frameborder="0" allowfullscreen></iframe>
