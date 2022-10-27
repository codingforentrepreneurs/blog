---
title: *args and **kwargs
slug: args-and-kwargs

publish_timestamp: Dec. 22, 2016
url: https://www.codingforentrepreneurs.com/blog/args-and-kwargs/

---

So when you write any given function, that function can have un-named arguments (args) or named (aka keyword) arguments (kwargs). You can also use *args and **kwargs as a "catch all" solution for arguments that you may not have included in defining the original function but might be passed to this function without your knowledge. This is very good practice when overwriting methods (like in Classes).


```
def some_func(arg1, arg2, kwarg1="Something", kwarg2=False):
       return arg1
```

When you override default methods you can't know for sure if args or keyword args are being passed. So you use *args and **kwargs to capture all args and keyword that might be passed.

Try this:
```
def another_func(*args, **kwargs):
     print("args are:")
     print(args)
     print("kwargs are")
     print(kwargs)
​
​another_func("hello", True, None, name='ChrisW', location='Unknown') 
​
​another_func2("hello", True, None, name='Justin', location='California')
```

Hopefully that illustrates the concept for you. Did you try this out on your own?