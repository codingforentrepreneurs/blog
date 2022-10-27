---
title: The Simple Power of Django Validators
slug: the-simple-power-of-django-validators

publish_timestamp: Oct. 7, 2017
url: https://www.codingforentrepreneurs.com/blog/the-simple-power-of-django-validators/

---


# The Simple Power of Django Validators

- Did *jon@targaryen.com* already register?
- Are the names "Jon, Tyrion, Eagon, Daenerys, Jamison, and Eddard" all Game of Thrones characters?
- Is the username *voldemort* in our blacklist? 

These are simple examples of validation. Validating data is concept that Django forms, models, and serializers (from [DRF](http://www.django-rest-framework.org/api-guide/validators/)) can do. Of course, data validation isn't limited to Django but this guide will show you how it's done just in Django. Ready to get started?

### Watch it [here](#watch)

### Built-in Validation
As per the [Django documentation](https://docs.djangoproject.com/en/1.11/ref/validators/#built-in-validators), there are some built-in validators:

```
from django.core.validators import validate_slug, validate_email
from django.db import models

SlugField = models.CharField(validators=[validate_slug])
EmailField = models.CharField(validators=[validate_email])
```

Of course, the example above is rendered useless because `SlugField` and `EmailField` are already fields in `from django.db import models`. But this shows us that Django uses it's own validation methodology it it's standard fields.

This means we can go further and build our own validation because that's probably more useful for us long term.


### Validation: The `clean` Method
Let's take a form, such as this *ContactForm*
```
# yourapp.forms.py
from django import forms

class ContactForm(forms.Form): 
    name        = forms.CharField()
    email       = forms.EmailField()
    content     = forms.CharField(widget=forms.Textarea())
```

A simple form that really validates only 1 thing that the `email` field is actually an email. Other than that, no validations are performed.

Let's do some validation with the `clean_<fieldname>` method. We replace `<fieldname>` with any given field from our form. So the above *ContactForm*'s `name` field would be `clean_name`. Let's ensure only a certain type of email comes through:

```
# yourapp.forms.py
from django import forms

class ContactForm(forms.Form): 
    name        = forms.CharField()
    email       = forms.EmailField()
    content     = forms.CharField(widget=forms.Textarea())

    def clean_email(self):
        email_passed = self.cleaned_data.get("email")
        if not "yourdomain.com" in email_passed:
            raise forms.ValidationError("Sorry, the email submitted is invalid. All emails have to be registered on this domain only.")
        return email_passed

```

This is a *field-level* clean method. It will produce a *field-level* error if a email that doesn't fit the validation criteria is passed.


Let's do the same on a *form-level* (aka as a *non_field_error*):

```
# yourapp.forms.py
from django import forms

class ContactForm(forms.Form): 
    name        = forms.CharField()
    email       = forms.EmailField()
    content     = forms.CharField(widget=forms.Textarea())

    def clean(self):
        # form level cleaning
        cleaned_data = super(ContactForm, self).clean()
        email_passed = cleaned_data.get("email")
        if not "yourdomain.com" in email_passed:
            raise forms.ValidationError("Sorry, the email submitted is invalid. All emails have to be registered on this domain only.",)
```

Pretty much the same. Not a big deal at all.


What if I wanted to reuse this same cleaning/validation method? What do I do?

### Validation: The Reusable Method

Writing custom validators is often the best way to have uniform validation throughout your project. So let's create our own validators using the above example.

```
# yourapp.validators.py

from django.core.exceptions import ValidationError
from django.utils.translation import ugettext_lazy as _

def validate_domainonly_email(value):
    """
    Let's validate the email passed is in the domain "yourdomain.com"
    """
    if not "yourdomain.com" in value:
        raise ValidationError(_"Sorry, the email submitted is invalid. All emails have to be registered on this domain only.", status='invalid')

```

Now we're talking. This new method `validate_domainonly_email` is perfect to be used anywhere on our project. It will accomplish *field-level* validation wherever we use it. Just like:

```
# yourapp.forms.py
from django import forms

from .validators import validate_domainonly_email

class ContactForm(forms.Form): # or forms.ModelForm
    name        = forms.CharField()
    email       = forms.EmailField(validators=[validate_domainonly_email])
    content     = forms.CharField(widget=forms.Textarea())

```

The nice thing is we can reuse this validator in *any* field including *Model fields*, *Form fields*, and even *DRF serializers*. 

```
#anotherapp.models.py 
from django.db import models

from yourapp.validators import validate_domainonly_email


class NewProjectSubmission(models.Model):
    email = models.EmailField(validators=[validate_domainonly_email])
```


Or in a [Django Rest Framework](https://kirr.co/svez0s) serializer:
```
#anotherapp.serializers.py 

from rest_framework import serializers

from yourapp.validators import validate_domainonly_email



class CommentSerializer(serializers.Serializer): # or serializers.ModelSerializer
    email = serializers.EmailField(validators=[validate_domainonly_email])
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
```


### Even further... a custom Field

To maximize reusability, you might consider even creating a custom Django Field, specifically for this validator. It's all going to depend on your project and what you need to validate.

```
from django.core.validators import validate_slug, validate_email
from django.db import models

from yourapp.validators import validate_domainonly_email


class DomainEmailField(models.CharField):
    description = "An email field specific to our domain."

    def __init__(self, *args, **kwargs):
        kwargs['max_length'] = 120
        kwargs['validators'] = [validate_email, validate_domainonly_email]
        super(DomainEmailField, self).__init__(*args, **kwargs)

```


What do you think of our post?

### Watch
<iframe width="560" height="315" src="https://www.youtube.com/embed/gEbbso4XG00?rel=0" frameborder="0" allowfullscreen></iframe>
