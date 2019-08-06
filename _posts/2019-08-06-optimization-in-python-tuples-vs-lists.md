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

## Tuples vs Lists

Despite the fact that tuples are comparatively less popular than lists, they find its application in many crucial aspects of Python, like
* returning more than 1 item from a function
* dictioanry key and value pairs
* arguments and parameters

A normal Python program has thousands of tuples and lists

{% highlight python %}
>>> import gc
>>>
>>> def count_type(obj_type):
...     count = 0
...     for obj in gc.get_objects():
...             if type(obj) == obj_type:
...                     count += 1
...     return count
...
>>> count_type(list)
512
>>> count_type(tuple)
1467
>>>
{% endhighlight %}

Here I have used the Python module called `gc` which stands for Garbage Collector. It underlines the underlying memory management function of Python, the automatic garbage collector. The `get_objects()` method returns a list of all objects tracked by the collector, excluding the list returned.

### Empty lists vs empty tuples

There is a big difference when considering empty lists and tuples. An empty tuple acts as a singleton but it isn't true for lists. While creating an empty tuple, Python points to the already preallocated one in such a way that both of them has the same address, thereby saving memory.

{% highlight python %}
>>> a = ()
>>> b = ()
>>> a is b
True
>>> id(a)
140235655475272
>>> id(b)
140235655475272
>>>
{% endhighlight %}

For lists, this doesn't hold

{% highlight python %}
>>> a = []
>>> b = []
>>> a is b
False
>>> id(a)
140235617336584
>>> id(b)
140235617345672
>>>
{% endhighlight %}


### Allocation

To reduce memory fragmentation and allocation time complexity, Python reuses old tuples. Instead of permanently deleting a tuple, Python moves it to a `free list` if the tuple has less than 20 items.

{% highlight python %}
>>> a = (1,2,3)
>>> id(a)
140235617388224
>>> del a
>>> b = (4,5,6)
>>> id(b)
140235617388224
>>>
{% endhighlight %}

Lists also use the same optimization method as tuples.

{% highlight python %}
>>> a = [1,2,3]
>>> id(a)
140235617336584
>>> del(a)
>>> b = [4,5,6]
>>> id(b)
140235617336584
>>>
{% endhighlight %}

### List resizing

To avoid the cost of resizing, Python does not resize a list everytime we add or remove an item. Instead, it over-allocates memory to a list in the form of `empty slots` which are hidden from the user. The documentation describes it as follows

*This over-allocates proportional to the list size, making room for additional growth. The over-allocation is mild but is enough to give linear-time amortized behavior over a long sequence of appends() in the presence of a poorly-performing system realloc().*

*The growth pattern is: 0, 4, 8, 16, 25, 35, 46, 58, 72, 88, ...*

For example, if you want to append an item to a list of length 8, Python will resize it to16 slots and add the 9th item. The rest of the slots will be hidden and reserved for new items.

### Tuples do not over-allocate

Unlike lists, as we saw above, tuples do not use over-allocation. They are of fixed size and can store data more compactly.

{% highlight python %}
>>> import sys
>>>
>>> a = tuple(range(10))
>>> b = list(range(10))
>>>
>>> sys.getsizeof(a)
128
>>> sys.getsizeof(b)
200
>>>
{% endhighlight %}

### Lists have faster append

Due to lists over-allocation, it reduces the cost of append operation as Python doesn't need to allocate it memory at that time. Hence the append operation of a list is faster than tuple's. I wrote a script to check it.

{% highlight python %}
import time

a = (1,2,3)
b = [1,2,3]

start = time.time()
a = a + (4,)
tuple_time  = time.time() - start

print('Time for tuple - ', tuple_time)

start = time.time()
b.append(4)
list_time = time.time() - start

print('Time for list - ', list_time)

print('List') if list_time < tuple_time else print('Tuple')
{% endhighlight %}

On executing the script, the results were as I expected

{% highlight bash %}
$ python3 speed.py
Time for tuple -  1.1920928955078125e-06
Time for list -  7.152557373046875e-07
List
{% endhighlight %}


### Tuples do not need to be copied

Since tuples are immutable, they do not have to be copied. Rather, Python just refers the memory address of the old tuple to the new one.

{% highlight python %}
>>> a = (1,2,3)
>>> b = tuple(a)
>>> b is a
True
>>>
{% endhighlight %}

This is not true for lists.

{% highlight python %}
>>> a = [1,2,3]
>>> b = list(a)
>>> a is b
False
>>>
{% endhighlight %}

### Tuples can be constant folded

Constant folding is the process of recognizing and evaluating constant expressions at compile time rather than computing them at runtime. Lists, on the other hand are evaluated at the runtime and need to be build up from scratch. This significantly reduces the time complexity of operations using tuples.
