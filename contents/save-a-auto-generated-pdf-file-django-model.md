---
title: Save an Auto Generated PDF File to Django model
slug: save-a-auto-generated-pdf-file-django-model

publish_timestamp: April 11, 2018
url: https://www.codingforentrepreneurs.com/blog/save-a-auto-generated-pdf-file-django-model/

---


To save a auto-generated PDF file to your model, like what we do with [render_to_pdf](https://www.codingforentrepreneurs.com/blog/html-template-to-pdf-in-django/), you can use the below code snippet. Be sure to use `Python 3.5+` for this one.

```
from io import BytesIO
from django.core.files import File
from .utils import render_to_pdf

# models.py
class YourModel(models.Model):
       slug 			        = models.CharField(max_length=120)
       pdf 					= models.FileField(upload_to='pdfs/', null=True, blank=True)
  

# to generate and save your pdf to your model

def generate_obj_pdf(instance_id):
     obj = YouModel.objects.get(id=instance_id)
     context = {'instance': obj}
     pdf = render_to_pdf('your/pdf/template.html', context)
     filename = "YourPDF_Order{}.pdf" %(obj.slug)
     obj.pdf.save(filename, File(BytesIO(pdf.content)))
```
Simple enough right? Did you try it out?
