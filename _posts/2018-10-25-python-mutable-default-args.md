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
A third time and you'll get three elements, four and four, and so on. That is to say; there is a single list associated with the function foo and for each call you modify that once instance. There is extensive discussion about this on the net [here](https://softwareengineering.stackexchange.com/questions/157373/python-mutable-default-argument-why), [here](https://stackoverflow.com/questions/1132941/least-astonishment-and-the-mutable-default-argument), and [here](http://effbot.org/zone/default-values.htm).

The rational behind this seems to come down to performance and I guess that makes sense given how python has issues with performance. However, this is horribly counter-intuitive and prone to error. Please do not make use of this feature.
