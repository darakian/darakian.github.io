---
layout: post
title:  "Mutable default arguments"
date:   2018-10-25 20:00:00 -0700
---
# A feature that's a bug that's a feature
Today I came across the most frightening aspect of python that I've ever seen. Mutable default arguments. If you haven't come across these yet they look like This
```
def foo(list=[]):
    list.append(1)
    return list
```
When you call foo twice you get the following
```
>>> foo()
[1]
>>> foo()
[1, 1]
>>>
```
A third time and you'll get three elements, four and four, and so on. That is to say; there is a single list associated with the function foo and for each call you modify that once instance. There is extensive discussion about this on the net [here](https://softwareengineering.stackexchange.com/questions/157373/python-mutable-default-argument-why), [here](https://stackoverflow.com/questions/1132941/least-astonishment-and-the-mutable-default-argument), and [here](http://effbot.org/zone/default-values.htm) for example.

This becomes more confusing and dangerous when you consider a function like
```
def func(var=[], input=None):
    var.append(input)
    if 'foo' in var:
            print 'hax'
```
On the surface this looks fine, so lets call this functions.
```
>>> func(input='Some')
>>>
>>> func(input='foo')
hax
```
As expected we don't get any output for the incorrect input and we do get output when we input the magic string. However, now that we've input the magic string
```
>>> func(input='Some')
hax
```
The list `var` has persisted across calls to the function and all subsequent calls to `func` will now travel down the `if 'foo' in var` code path. This gets even worse if we override the list variable
```
>>> func(var=[], input='Some')
>>> func(input='Some')
>>>
>>> func(input='Some')
hax
```
The default variable is still in the python vm memory space and it maintains its state across the lifetime of the program. Replace the `if foo in var` with something like `if action in allowed` and you can imagine how badly this can goes wrong. It does take poorly written code to be truly exploitable with this syntax, but the world is full of poorly written code.

```
Thanks for reading
```
