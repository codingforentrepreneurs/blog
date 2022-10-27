---
title: TemplateDoesNotExist error
slug: templatedoesnotexist-error

publish_timestamp: Jan. 19, 2017
url: https://www.codingforentrepreneurs.com/blog/templatedoesnotexist-error/

---

I see this error come up a lot so here's a few quick tips to ensure you have your template setup correctly

1.  Save the template file in the right directory (& check the spelling)
2. Ensure that your settings templates look something like:
     
     <pre>
     TEMPLATES = [
         {
             'BACKEND': 'django.template.backends.django.DjangoTemplates',
             'DIRS': [os.path.join(BASE_DIR, 'templates')],
             'APP_DIRS': True,
             'OPTIONS': {
                 'context_processors': [
                     'django.template.context_processors.debug',
                     'django.template.context_processors.request',
                     'django.contrib.auth.context_processors.auth',
                     'django.contrib.messages.context_processors.messages',
                     'social_django.context_processors.backends',
                     'social_django.context_processors.login_redirect',
                 ],
             },
         },
     ]
     </pre>

3. Ensure settings.py is saved


4. Ensure your app is within `INSTALLED_APPS`  in settings.py especially if your app directory is something like (most of the structure is left out):
    <pre>
    project/
         - settings.py
    yourapp/
         - templates/
               -yourapp/
                     -yourapp_detail.hmtl
         - modes.py
    </pre>
5. Restart your server

That's all I can think of now. Hope that helps!