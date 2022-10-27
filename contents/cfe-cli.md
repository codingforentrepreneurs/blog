---
title: CFE CLI
slug: cfe-cli

publish_timestamp: Aug. 21, 2017
url: https://www.codingforentrepreneurs.com/blog/cfe-cli/

---


<div class='alert alert-warning'>
The <b>cfe-cli</b> has been deprecated. Please use <a href='https://www.codingforentrepreneurs.com/blog/pipenv-virtual-environments-for-python/'>pipenv</a> instead. 
</div>
The Coding for Entrepreneurs CLI is here. We're experimenting with the idea of shortcutting to generating our projects. The inspiration came from the popularity of [this guide](https://www.codingforentrepreneurs.com/blog/create-a-blank-django-project/).

We currently only guarantee support for linux/mac. It's recommended to be working in a virtualenv for this.

To get started:

```
# /path/to/your/virtual/env/
(env) $ pip install cfe
```

To create a new, blank project:

```
(env) $ cfe --new 
```

Your project has been created under "backend/":

```
/path/to/your/virtual/env/backend/src/
```

To start from scratch while replacing the project generated above:

```
(env) $ cfe --replace
```
