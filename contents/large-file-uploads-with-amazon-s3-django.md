---
title: Large File Uploads with Amazon S3 + Django
slug: large-file-uploads-with-amazon-s3-django

publish_timestamp: Aug. 14, 2017
url: https://www.codingforentrepreneurs.com/blog/large-file-uploads-with-amazon-s3-django/

---


Handling large file uploads can be a challenge. In this one, we aim to make it a little easier to implement in your projects. Overall, this guide, https://kirr.co/e1133t, will be your final destination for getting this done right. 

### Watch on [cfe](https://www.codingforentrepreneurs.com/projects/large-file-uploads/), [youtube](https://kirr.co/mcoqlf), or [here](#watch).

==========

## Amazon AWS Setup for S3, Policy/Permissions, User & Group
This section is similar to the [Setting up S3 S3 Static & Media Files for Django guide](https://www.codingforentrepreneurs.com/blog/s3-static-media-files-for-django/) but different as it will not handle static files but rather will be provisioned for handling large file uploads. You can use this guides together as the settings will not overrite each other.


### 1. Create AWS Developer Account [here](http://console.aws.amazon.com) 


### 2. Create AWS S3 Bucket

1. Login to [console.aws.amazon.com](http://console.aws.amazon.com) (create user account if needed.)

2. Navigate to **Services > S3**.

3. Click **+ Create Bucket**

4. Add a `Bucket name` and `region` and all defaults otherwise. 
    - If you go with your default region, take note of the `region=<your-region>` in the current url. For example, mine is `US West (Oregon)` and my region in my url is `us-west-2`
    - Example:
     
        Bucket Name: `cfe-awesome-bucket`
        
        Region: `US West (Oregon)`
        
        url_region = `us-west-2`

    - Make note of the above settings, it's going to be useful later

5. Now that your bucket is created 
    - Select your bucket (like `cfe-awesome-bucket`) 

    - Then select **Permissions**

    - Then click **CORS configuration** and add the following:

    ```
    <CORSConfiguration>
        <CORSRule>
            <AllowedOrigin>*</AllowedOrigin>
            <AllowedMethod>PUT</AllowedMethod>
            <AllowedMethod>POST</AllowedMethod>
            <AllowedMethod>GET</AllowedMethod>
            <MaxAgeSeconds>3000</MaxAgeSeconds>
            <AllowedHeader>*</AllowedHeader>
        </CORSRule>
    </CORSConfiguration>
    ```
6. Bucket is now created and ready for API user uploads.


### 3. Create AWS Managed Policy


1. Login to [console.aws.amazon.com](http://console.aws.amazon.com) (create user account if needed.)

2. Navigate to **Services > IAM > Policies**.

3. Click **Create policy**

4. Select **Create your own policy**

5. Use the following settings:

    - **Policy Name**: 
    ```
    cfe-awesome-bucket-s3-policy
    ```

    - **Description**: 
    ```
    My policy for the cfe-awesome-bucket I created in step 2.
    ```
    - **Policy Document** :

        - **Format**:
        Replace `<your-new-bucket>` with your S3 bucket name from above.
        ```
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": [
                        "s3:ListAllMyBuckets"
                    ],
                    "Resource": "arn:aws:s3:::*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "s3:ListBucket",
                        "s3:GetBucketLocation",
                        "s3:ListBucketMultipartUploads",
                        "s3:ListBucketVersions"
                    ],
                    "Resource": "arn:aws:s3:::<your-new-bucket>"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "s3:*Object*",
                        "s3:ListMultipartUploadParts",
                        "s3:AbortMultipartUpload"
                    ],
                    "Resource": "arn:aws:s3:::<your-new-bucket>/*"
                }
            ]
        }

        ```

        - **Example** using the `cfe-awesome-bucket` s3 bucket:
        ```
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": [
                        "s3:ListAllMyBuckets"
                    ],
                    "Resource": "arn:aws:s3:::*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "s3:ListBucket",
                        "s3:GetBucketLocation",
                        "s3:ListBucketMultipartUploads",
                        "s3:ListBucketVersions"
                    ],
                    "Resource": "arn:aws:s3:::cfe-awesome-bucket"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "s3:*Object*",
                        "s3:ListMultipartUploadParts",
                        "s3:AbortMultipartUpload"
                    ],
                    "Resource": "arn:aws:s3:::cfe-awesome-bucket/*"
                }
            ]
        }

        ```

### 4. Create IAM Group

1. Login to [console.aws.amazon.com](http://console.aws.amazon.com) (create user account if needed.)

2. Navigate to **Services > IAM > Groups**.

3. Click **Create new group**

4. Add a **Group Name** such as `CFE_AwesomeGroup`

5. Add **Policy** from previous step. Mine is `cfe-awesome-bucket-s3-policy`

6. Hit **next** and **Create Group**


### 5. Create User for new Group

1. Login to [console.aws.amazon.com](http://console.aws.amazon.com) (create user account if needed.)

2. Navigate to **Services > IAM > Users**.

3. Click **Create user**

4. Add a **User name** such as `CFE_user_justin`

5. Set **Access type** to at least **Programmatic access**

6. Click **Next: Permissions**

7. Under *Add user to group* (which should be already selected), select your recently created group.

8. Click **Next: Review** 

9. Ensure details are correct, click **Create User**

10. Click **Download .csv**

11. Open downloaded csv file (likely called `credentials.csv`) to find:
    ```
    User name: CFE_user_justin

    Password: (blank)

    Access key ID: AKIAJQEJZQD6WMEQIK7A

    Secret access key: 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

    Console login link: (doesn't matter for this us here)

    ```


### 6. Compile AWS Settings for later use in Django:

**Format**:
```
AWS_UPLOAD_BUCKET = <your-upload-bucket-name>
AWS_UPLOAD_USERNAME = <your-new-username>
AWS_UPLOAD_GROUP = <your-group-name>
AWS_UPLOAD_REGION = <your-aws-region>
AWS_UPLOAD_ACCESS_KEY_ID = <your-aws-key-id>
AWS_UPLOAD_SECRET_KEY = <your-aws-secret-key>
```

**Example**:
```python
AWS_UPLOAD_BUCKET = "cfe-awesome-bucket"
AWS_UPLOAD_USERNAME = "CFE_user_justin"
AWS_UPLOAD_GROUP = "CFE_AwesomeGroup"
AWS_UPLOAD_REGION = "us-west-2"
AWS_UPLOAD_ACCESS_KEY_ID = "AKIAJQEJZQD6WMEQIK7A"
AWS_UPLOAD_SECRET_KEY = "ntRLzMLxie6YZI2jQ8ouXltZM9vdcAF1IVjD+tK+"
```
 



## Django Configuration
Assuming your using the [Blank Django Project in this guide](https://www.codingforentrepreneurs.com/blog/create-a-blank-django-project/) we're going to setup your Django project to be ready for File Uploads direct to Amazon S3. The size doesn't really matter but we'll be able to handle large files as well.


**1. AWS-related installs. Not required, but recommended:**

```bash
python3 -m pip install boto boto3 cryptography urllib3 requests
```

**2. Install [django rest framework](http://www.django-rest-framework.org/) and [python-decouple](https://github.com/HBNetwork/python-decouple):**

```bash
pyhton3 -m pip install djangorestframework python-decouple
```

Update settings modules (`local.py`, `base.py`, `production.py`):
```python
INSTALLED_APPS = [
    ...
    'rest_framework'
    ...
]
```

**3. Create `Files app`:**

```bash
python manage.py startapp files
```

**4. Update `files/models.py`:**

```python
class FileItem(models.Model):
    user                            = models.ForeignKey(settings.AUTH_USER_MODEL, default=1)
    name                            = models.CharField(max_length=120, null=True, blank=True)
    path                            = models.TextField(blank=True, null=True)
    size                            = models.BigIntegerField(default=0)
    file_type                       = models.CharField(max_length=120, null=True, blank=True)
    timestamp                       = models.DateTimeField(auto_now_add=True)
    updated                         = models.DateTimeField(auto_now=True)
    uploaded                        = models.BooleanField(default=False)
    active                          = models.BooleanField(default=True)

    @property
    def title(self):
        return str(self.name)
```

**5. Add `Files app` to `INSTALLED_APPS`:**

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'files',
    ...
]
```

**6. Add AWS values to `.env`:
```bash
AWS_UPLOAD_BUCKET="cfe-awesome-bucket"
AWS_UPLOAD_USERNAME="CFE_user_justin"
AWS_UPLOAD_GROUP="CFE_AwesomeGroup"
AWS_UPLOAD_REGION="us-west-2"
AWS_UPLOAD_ACCESS_KEY_ID="AKIAJQEJZQD6WMEQIK7A"
AWS_UPLOAD_SECRET_KEY="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"
```
**7. Create `config_aws.py` from AWS Setup Above (setup 3):**

Using [python-decouple](https://github.com/HBNetwork/python-decouple) to load in the files from `.env` with:

```python
from decouple import config

AWS_UPLOAD_BUCKET = config("AWS_UPLOAD_BUCKET", default=None)
AWS_UPLOAD_USERNAME = config("AWS_UPLOAD_USERNAME", default=None)
AWS_UPLOAD_GROUP = config("AWS_UPLOAD_GROUP", default=None)
AWS_UPLOAD_REGION = config("AWS_UPLOAD_REGION", default=None)
AWS_UPLOAD_ACCESS_KEY_ID = config("AWS_UPLOAD_ACCESS_KEY_ID", default=None)
AWS_UPLOAD_SECRET_KEY = config("AWS_UPLOAD_SECRET_KEY", default=None)
```

**8. Create a Policy view and update `files/views.py`:**

```python
import base64
import hashlib
import hmac
import os
import time
from rest_framework import permissions, status, authentication
from rest_framework.response import Response
from rest_framework.views import APIView
from .config_aws import (
    AWS_UPLOAD_BUCKET,
    AWS_UPLOAD_REGION,
    AWS_UPLOAD_ACCESS_KEY_ID,
    AWS_UPLOAD_SECRET_KEY
)
from .models import FileItem

class FilePolicyAPI(APIView):
    """
    This view is to get the AWS Upload Policy for our s3 bucket.
    What we do here is first create a FileItem object instance in our
    Django backend. This is to include the FileItem instance in the path
    we will use within our bucket as you'll see below.
    """
    permission_classes = [permissions.IsAuthenticated]
    authentication_classes = [authentication.SessionAuthentication]

    def post(self, request, *args, **kwargs):
        """
        The initial post request includes the filename
        and auth credientails. In our case, we'll use
        Session Authentication but any auth should work.
        """
        filename_req = request.data.get('filename')
        if not filename_req:
                return Response({"message": "A filename is required"}, status=status.HTTP_400_BAD_REQUEST)
        policy_expires = int(time.time()+5000)
        user = request.user
        username_str = str(request.user.username)
        """
        Below we create the Django object. We'll use this
        in our upload path to AWS. 

        Example:
        To-be-uploaded file's name: Some Random File.mp4
        Eventual Path on S3: <bucket>/username/2312/2312.mp4
        """
        file_obj = FileItem.objects.create(user=user, name=filename_req)
        file_obj_id = file_obj.id
        upload_start_path = "{username}/{file_obj_id}/".format(
                    username = username_str,
                    file_obj_id=file_obj_id
            )       
        _, file_extension = os.path.splitext(filename_req)
        filename_final = "{file_obj_id}{file_extension}".format(
                    file_obj_id= file_obj_id,
                    file_extension=file_extension

                )
        """
        Eventual file_upload_path includes the renamed file to the 
        Django-stored FileItem instance ID. Renaming the file is 
        done to prevent issues with user generated formatted names.
        """
        final_upload_path = "{upload_start_path}{filename_final}".format(
                                 upload_start_path=upload_start_path,
                                 filename_final=filename_final,
                            )
        if filename_req and file_extension:
            """
            Save the eventual path to the Django-stored FileItem instance
            """
            file_obj.path = final_upload_path
            file_obj.save()

        policy_document_context = {
            "expire": policy_expires,
            "bucket_name": AWS_UPLOAD_BUCKET,
            "key_name": "",
            "acl_name": "private",
            "content_name": "",
            "content_length": 524288000,
            "upload_start_path": upload_start_path,

            }
        policy_document = """
        {"expiration": "2019-01-01T00:00:00Z",
          "conditions": [ 
            {"bucket": "%(bucket_name)s"}, 
            ["starts-with", "$key", "%(upload_start_path)s"],
            {"acl": "%(acl_name)s"},
            
            ["starts-with", "$Content-Type", "%(content_name)s"],
            ["starts-with", "$filename", ""],
            ["content-length-range", 0, %(content_length)d]
          ]
        }
        """ % policy_document_context
        aws_secret = str.encode(AWS_UPLOAD_SECRET_KEY)
        policy_document_str_encoded = str.encode(policy_document.replace(" ", ""))
        url = 'https://{bucket}.s3-{region}.amazonaws.com/'.format(
                        bucket=AWS_UPLOAD_BUCKET,  
                        region=AWS_UPLOAD_REGION
                        )
        policy = base64.b64encode(policy_document_str_encoded)
        signature = base64.b64encode(hmac.new(aws_secret, policy, hashlib.sha1).digest())
        data = {
            "policy": policy,
            "signature": signature,
            "key": AWS_UPLOAD_ACCESS_KEY_ID,
            "file_bucket_path": upload_start_path,
            "file_id": file_obj_id,
            "filename": filename_final,
            "url": url,
            "username": username_str,
        }
        return Response(data, status=status.HTTP_200_OK)
```



**9. Update `urls.py`:**

```python
from django.views.generic.base import TemplateView
from files.views import FilePolicyAPI
urlpatterns = [
    ...
    url(r'^upload/$', TemplateView.as_view(template_name='upload.html'), name='upload-home'),
    url(r'^api/files/policy/$', FilePolicyAPI.as_view(), name='upload-policy'),
    ...
]
```


**10. Create `templates/base.html`:**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <title>CFE Upload</title>
  </head>
  <body>
    <div class='container main-container'>
      {% block content %}{% endblock content %}
    </div>
    <script src="https://code.jquery.com/jquery-3.2.1.min.js"></script> <!-- Ensure it's not jquery-3.2.1.slim.min.js -->
  </body>
 </html>
```


**11. Create upload template `templates/upload.html` with the contents:**

```html
{% extends "base.html" %}
{% block content %}
<div class='row'>
    <div class='col-6 offset-3'>
        <div class='item-loading-queue'>

        </div>
        <form class='cfeproj-upload-form'>
            <input class='cfeproj-upload-file form-control' type='file' accept='audio/*,video/*,image/*' multiple='multiple' />
        </form>
    </div>
</div>
{% endblock content %}
```

**12. Ensure Django has static files setup like we do [in this guide](https://www.codingforentrepreneurs.com/blog/s3-static-media-files-for-django/) or any other static file setup you choose.**


## jQuery Configuration
Below is the jQuery code we're going to use.

```html
<script>
$(document).ready(function(){

    // setup session cookie data. This is Django-related
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
    // end session cookie data setup. 



// declare an empty array for potential uploaded files
var fileItemList = []

// auto-upload on file input change.
$(document).on('change','.cfeproj-upload-file', function(event){
    var selectedFiles = $(this).prop('files');
    formItem = $(this).parent()
    $.each(selectedFiles, function(index, item){
        var myFile = verifyFileIsImageMovieAudio(item)
        if (myFile){
            uploadFile(myFile)
        } else {
            // alert("Some files are invalid uploads.")
        }
    })
    $(this).val('');

})



function verifyFileIsImageMovieAudio(file){
    // verifies the file extension is one we support.
    var extension = file.name.split('.').pop().toLowerCase(); //file.substr( (file.lastIndexOf('.') +1) );
    switch(extension) {
        case 'jpg':
        case 'png':
        case 'gif':
        case 'jpeg':
            return file  
        case 'mov':
        case 'mp4':
        case 'mpeg4':
        case 'avi':
            return file
        case 'mp3':
            return file
        default:
            notAllowedFiles.push(file)
            return null
    }
};

function constructFormPolicyData(policyData, fileItem) {
   var contentType = fileItem.type != '' ? fileItem.type : 'application/octet-stream'
    var url = policyData.url
    var filename = policyData.filename
    var repsonseUser = policyData.user
    // var keyPath = 'www/' + repsonseUser + '/' + filename
    var keyPath = policyData.file_bucket_path
    var fd = new FormData()
    fd.append('key', keyPath + filename);
    fd.append('acl','private');
    fd.append('Content-Type', contentType);
    fd.append("AWSAccessKeyId", policyData.key)
    fd.append('Policy', policyData.policy);
    fd.append('filename', filename);
    fd.append('Signature', policyData.signature);
    fd.append('file', fileItem);
    return fd
}

function fileUploadComplete(fileItem, policyData){
    data = {
        uploaded: true,
        fileSize: fileItem.size,
        file: policyData.file_id,

    }
    $.ajax({
        method:"POST",
        data: data,
        url: "/api/files/complete/",
        success: function(data){
            displayItems(fileItemList)
        },
        error: function(jqXHR, textStatus, errorThrown){ 
            alert("An error occured, please refresh the page.")
        }
    })
}

function displayItems(fileItemList){
    var itemList = $('.item-loading-queue')
    itemList.html("")
    $.each(fileItemList, function(index, obj){
        var item = obj.file
        var id_ = obj.id
        var order_ = obj.order
        var html_ = "<div class=\"progress\">" + 
          "<div class=\"progress-bar\" role=\"progressbar\" style='width:" + item.progress + "%' aria-valuenow='" + item.progress + "' aria-valuemin=\"0\" aria-valuemax=\"100\"></div></div>"
        itemList.append("<div>" + order_ + ") " + item.name + "<a href='#' class='srvup-item-upload float-right' data-id='" + id_ + ")'>X</a> <br/>" + html_ + "</div><hr/>")

    })
}


function uploadFile(fileItem){
        var policyData;
        var newLoadingItem;
        // get AWS upload policy for each file uploaded through the POST method
        // Remember we're creating an instance in the backend so using POST is
        // needed.
        $.ajax({
            method:"POST",
            data: {
                filename: fileItem.name
            },
            url: "/api/files/policy/",
            success: function(data){
                    policyData = data
            },
            error: function(data){
                alert("An error occured, please try again later")
            }
        }).done(function(){
            // construct the needed data using the policy for AWS
            var fd = constructFormPolicyData(policyData, fileItem)
            
            // use XML http Request to Send to AWS. 
            var xhr = new XMLHttpRequest()

            // construct callback for when uploading starts
            xhr.upload.onloadstart = function(event){
                var inLoadingIndex = $.inArray(fileItem, fileItemList)
                if (inLoadingIndex == -1){
                    // Item is not loading, add to inProgress queue
                    newLoadingItem = {
                        file: fileItem,
                        id: policyData.file_id,
                        order: fileItemList.length + 1
                    }
                    fileItemList.push(newLoadingItem)
                  }
                fileItem.xhr = xhr
            }

            // Monitor upload progress and attach to fileItem.
            xhr.upload.addEventListener("progress", function(event){
                if (event.lengthComputable) {
                 var progress = Math.round(event.loaded / event.total * 100);
                    fileItem.progress = progress
                    displayItems(fileItemList)
                }
            })

            xhr.upload.addEventListener("load", function(event){
                console.log("Complete")
                // handle FileItem Upload being complete.
                // fileUploadComplete(fileItem, policyData)

            })

            xhr.open('POST', policyData.url , true);
            xhr.send(fd);
        })
}});
</script>
```

## Django File Upload Complete API Endpoint

1. Update `files/views.py`
```python
class FileUploadCompleteHandler(APIView):
    permission_classes = [permissions.IsAuthenticated]
    authentication_classes = [authentication.SessionAuthentication]
    
    def post(self, request, *args, **kwargs):
        file_id = request.POST.get('file')
        size = request.POST.get('fileSize')
        data = {}
        type_ = request.POST.get('fileType')
        if file_id:
            obj = FileItem.objects.get(id=int(file_id))
            obj.size = int(size)
            obj.uploaded = True
            obj.type = type_
            obj.save()
            data['id'] = obj.id
            data['saved'] = True
        return Response(data, status=status.HTTP_200_OK)
```
2. Update `urls.py`
```python
from django.views.generic.base import TemplateView
from files.views import FilePolicyAPI, FileUploadCompleteHandler
urlpatterns = [
    ...
    url(r'^upload/$', TemplateView.as_view(template_name='upload.html'), name='upload-home'),
    url(r'^api/files/complete/$', FileUploadCompleteHandler.as_view(), name='upload-complete'),
    url(r'^api/files/policy/$', FilePolicyAPI.as_view(), name='upload-policy'),
    ...
]
```
3. Update jquery:
```javascript
xhr.upload.addEventListener("load", function(event){
                console.log("Complete")
                // handle FileItem Upload being complete.
                fileUploadComplete(fileItem, policyData)

 })
```


### Watch a how-to guide on [youtube](https://www.youtube.com/watch?v=CbB7UoPyErY)
