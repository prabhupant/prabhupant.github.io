---
layout: post
title: "Optmization in Python - tuples vs lists"
---

In Python, tuples and lists are two similar data structures used to store sequential data. The well known difference between the two is that tuples are immutable whereas lists are not.

{% highlight python %}
>>> a = (1,2,3)
>>> a[1] = 4c
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>>
{% endhighlight %}

# Tuples vs Lists

Despite the fact that tuples are comparatively less popular than lists, they find its application in many crucial aspects of Python, like
* returning more than 1 item from a function
* dictioanry key and value pairs
* arguments and parameters

A normal Python program has thousands of tuples and lists

{% highlightt python %}


