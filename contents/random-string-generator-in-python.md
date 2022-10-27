---
title: Random String Generator in Python
slug: random-string-generator-in-python

publish_timestamp: Feb. 3, 2017
url: https://www.codingforentrepreneurs.com/blog/random-string-generator-in-python/

---


Sometimes you need a random string for your projects. Here's an easy way to make one in Python.

```
import random
import string

def random_string_generator(size=10, chars=string.ascii_lowercase + string.digits):
    return ''.join(random.choice(chars) for _ in range(size))


print(random_string_generator())

print(random_string_generator(size=50))
```

You can also use this to [Generate a Unique Slug](https://www.codingforentrepreneurs.com/blog/a-unique-slug-generator-for-django/) for your Django models.
