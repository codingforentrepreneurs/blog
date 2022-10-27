---
title: How to Create a Custom Django User Model
slug: how-to-create-a-custom-django-user-model

publish_timestamp: March 22, 2021
url: https://www.codingforentrepreneurs.com/blog/how-to-create-a-custom-django-user-model/

---

> This guide was updated for Django 4.0 but check supported versions below.

Django's built-in User model and authentication support is incredible. It's been a staple of Django for a long time. For me, it's one of the primary reasons I use Django over Flask, FastAPI, AIOHttp, and many other web frameworks.

Every once and a while, you need to change the functionality of the default user model. Let's say you no longer want a `username` field at all. Or you want the `email` field to be your default "username" field.  For this, we need to customize our default Django User model.

> Keep in mind that customizing Django defaults adds a lot of complexity to an already complex system. Stick with the defaults whenever possible. In other words, don't make a custom user model if you don't have to; [Foreign Keys](/topics/foreign-keys) are great for exactly that.


## Recommended Resources
- **New to Django?** Check out any [Try Django](/topics/try-django) series.
- Do you know **Django Migrations** or changing **any Django model** *well*? If not, check out [Django Models Unleashed](/projects/django-models-unleashed-2021/)
- Watch a video tutorial [on youtube](https://www.youtube.com/watch?v=HshbjK1vDtY) or [here](https://www.codingforentrepreneurs.com/courses/ecommerce/custom-django-user-model/). This videos are a bit older but show almost the exact steps as below.


## Assumptions
- You already have a Django project
- You are using Django 2.2+ (this works for 1.11+)
- You are working in a virtual environment ([huh? how?](/topics/try-django))


### 1. Create a new app
This will hold your custom user model. I'll call it `accounts`

```bash
python manage.py startapp accounts
```
I would avoid calling this app `users` as Django has a built-in app called users and could cause some confusion.

Add `accounts` to `INSTALLED_APPS` in settings.py:

```python
INSTALLED_APPS = [
    ...
    'accounts', 
]
```

### 2. Create the Custom User Model in `models.py`
You can use the [Django example](https://docs.djangoproject.com/en/dev/topics/auth/customizing/#a-full-example), or follow with ours below. We will simplify the process here. 

`accounts/models.py`
```python
from django.db import models
from django.contrib.auth.models import (
    BaseUserManager, AbstractBaseUser
)


class User(AbstractBaseUser):
    email = models.EmailField(
        verbose_name='email address',
        max_length=255,
        unique=True,
    )
    is_active = models.BooleanField(default=True)
    staff = models.BooleanField(default=False) # a admin user; non super-user
    admin = models.BooleanField(default=False) # a superuser

    # notice the absence of a "Password field", that is built in.

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = [] # Email & Password are required by default.

    def get_full_name(self):
        # The user is identified by their email address
        return self.email

    def get_short_name(self):
        # The user is identified by their email address
        return self.email

    def __str__(self):
        return self.email

    def has_perm(self, perm, obj=None):
        "Does the user have a specific permission?"
        # Simplest possible answer: Yes, always
        return True

    def has_module_perms(self, app_label):
        "Does the user have permissions to view the app `app_label`?"
        # Simplest possible answer: Yes, always
        return True

    @property
    def is_staff(self):
        "Is the user a member of staff?"
        return self.staff

    @property
    def is_admin(self):
        "Is the user a admin member?"
        return self.admin
```

So what's the `USERNAME_FIELD` exactly? Well that's how Django is going to recognize this user. It replaces the built-in `username` field for whatever you designate. In this case, we decided to opt for an `EmailField` but you can also consider using a phone number.

> What should you choose? Most of my projects, I still opt for the default Django User Model (aka not customizing it) with the `username` field. Some projects require an email / password for authentication. Some require a phone number / password for authentication. Some use a one-time password. Above shows you that you can be incredibly flexible with Django.



### 3. Create the User model manager

Django has built-in methods for the User Manager. We have to customize them in order to make our custom user model work correctly. 

`accounts/models.py`
```python
class UserManager(BaseUserManager):
    def create_user(self, email, password=None):
        """
        Creates and saves a User with the given email and password.
        """
        if not email:
            raise ValueError('Users must have an email address')

        user = self.model(
            email=self.normalize_email(email),
        )

        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_staffuser(self, email, password):
        """
        Creates and saves a staff user with the given email and password.
        """
        user = self.create_user(
            email,
            password=password,
        )
        user.staff = True
        user.save(using=self._db)
        return user

    def create_superuser(self, email, password):
        """
        Creates and saves a superuser with the given email and password.
        """
        user = self.create_user(
            email,
            password=password,
        )
        user.staff = True
        user.admin = True
        user.save(using=self._db)
        return user

# hook in the New Manager to our Model
class User(AbstractBaseUser): # from step 2
    ...
    objects = UserManager()

```



### 4. Update `settings` module (aka `settings.py`):

**1. Run Migrations**
```console
python manage.py makemigrations
python manage.py migrate
```


**2. Update `settings.py`**
```python
AUTH_USER_MODEL = 'accounts.User'
```

**3. Run Migrations again**
```console
python manage.py makemigrations
python manage.py migrate
```

**4. Create a superuser**
```console
python manage.py createsuperuser
```

**5. Some practical thoughts**

Setting the `AUTH_USER_MODEL` means that we can use many third party packages that leverage the Django user model; packages like Django Rest Framework, Django AllAuth, Python Social Auth, come to mind. 

Now, every time you need the user model, use:
```python
from django.contrib.auth import get_user_model
User = get_user_model()
```
This provides other developers and our future selves the kind of piece of mind that only well thought out code can provide; `get_user_model` is one of those pieces the core Django developers decided on long ago.

**6 But what about User foreign keys?**
At some point you'll be faced with this situation:
```python
class SomeModel(models.Model):
    user = models.ForeignKey(...)
```
So what to use? `get_user_model` `accounts.models.User` or what? Actually, you'll use `settings.AUTH_USER_MODEL` *every time* regardless of user model customiation. So it will look like this:

```python
from django.conf import settings

User = settings.AUTH_USER_MODEL

class SomeModel(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
```


### 5. Create the Forms for Register, Change, and Admin-Level Create
Since we updated the `AUTH_USER_MODEL`, we can actually use the [built-in User Creation Form](https://docs.djangoproject.com/en/dev/topics/auth/default/#django.contrib.auth.forms.UserCreationForm) since it's a model form based on our user model but let's learn how to implement our own anyways.


`accounts/forms.py`
```python
from django import forms
from django.contrib.auth import get_user_model
from django.contrib.auth.forms import ReadOnlyPasswordHashField

User = get_user_model()

class RegisterForm(forms.ModelForm):
    """
    The default 

    """

    password = forms.CharField(widget=forms.PasswordInput)
    password_2 = forms.CharField(label='Confirm Password', widget=forms.PasswordInput)

    class Meta:
        model = User
        fields = ['email']

    def clean_email(self):
        '''
        Verify email is available.
        '''
        email = self.cleaned_data.get('email')
        qs = User.objects.filter(email=email)
        if qs.exists():
            raise forms.ValidationError("email is taken")
        return email

    def clean(self):
        '''
        Verify both passwords match.
        '''
        cleaned_data = super().clean()
        password = cleaned_data.get("password")
        password_2 = cleaned_data.get("password_2")
        if password is not None and password != password_2:
            self.add_error("password_2", "Your passwords must match")
        return cleaned_data


class UserAdminCreationForm(forms.ModelForm):
    """
    A form for creating new users. Includes all the required
    fields, plus a repeated password.
    """
    password = forms.CharField(widget=forms.PasswordInput)
    password_2 = forms.CharField(label='Confirm Password', widget=forms.PasswordInput)

    class Meta:
        model = User
        fields = ['email']

    def clean(self):
        '''
        Verify both passwords match.
        '''
        cleaned_data = super().clean()
        password = cleaned_data.get("password")
        password_2 = cleaned_data.get("password_2")
        if password is not None and password != password_2:
            self.add_error("password_2", "Your passwords must match")
        return cleaned_data

    def save(self, commit=True):
        # Save the provided password in hashed format
        user = super().save(commit=False)
        user.set_password(self.cleaned_data["password"])
        if commit:
            user.save()
        return user


class UserAdminChangeForm(forms.ModelForm):
    """A form for updating users. Includes all the fields on
    the user, but replaces the password field with admin's
    password hash display field.
    """
    password = ReadOnlyPasswordHashField()

    class Meta:
        model = User
        fields = ['email', 'password', 'is_active', 'admin']

    def clean_password(self):
        # Regardless of what the user provides, return the initial value.
        # This is done here, rather than on the field, because the
        # field does not have access to the initial value
        return self.initial["password"]

```




### 6. Update the Django Admin
Let's put our forms from above to good use

`accounts/admin.py`
```python
from django.contrib import admin
from django.contrib.auth import get_user_model
from django.contrib.auth.models import Group
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin

from .forms import UserAdminCreationForm, UserAdminChangeForm

User = get_user_model()

# Remove Group Model from admin. We're not using it.
admin.site.unregister(Group)

class UserAdmin(BaseUserAdmin):
    # The forms to add and change user instances
    form = UserAdminChangeForm
    add_form = UserAdminCreationForm

    # The fields to be used in displaying the User model.
    # These override the definitions on the base UserAdmin
    # that reference specific fields on auth.User.
    list_display = ['email', 'admin']
    list_filter = ['admin']
    fieldsets = (
        (None, {'fields': ('email', 'password')}),
        ('Personal info', {'fields': ()}),
        ('Permissions', {'fields': ('admin',)}),
    )
    # add_fieldsets is not a standard ModelAdmin attribute. UserAdmin
    # overrides get_fieldsets to use this attribute when creating a user.
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': ('email', 'password1', 'password2')}
        ),
    )
    search_fields = ['email']
    ordering = ['email']
    filter_horizontal = ()


admin.site.register(User, UserAdmin)



```


### 7. Challenge. With the above, try to add a new field to your model. 
Learn exactly how in the video [on youtube](https://www.youtube.com/watch?v=HshbjK1vDtY) or [in this section](https://www.codingforentrepreneurs.com/courses/ecommerce/custom-django-user-model/). This videos are a bit older but show almost the exact steps as below.


<iframe width="560" height="315" src="https://www.youtube.com/embed/HshbjK1vDtY?rel=0" frameborder="0" allowfullscreen></iframe>