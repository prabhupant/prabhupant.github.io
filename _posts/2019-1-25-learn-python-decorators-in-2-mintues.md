---
layout: post
title: "Learn Python Decorators in 2 minutes"
---

Decorators in python allow the programmer to modify the behaviour of functions. They wrap another function in order to extend the behaviour of a wrapped function, without permanently modifying it.

Think of decorator as a function to which you pass another function to add or modify some of its functionality. A concrete example of decorator can be a function that divides two numbers

{% highlight python %}
def divide(num1, num2):
    print(num1 / num2)
divide(12, 2)    #output->6
divide(12, 0)    #ZeroDivisionError
{% endhighlight %}

In the above example, the divide function will work fine when called with values (12, 2) but with values (12, 0) it will give a division by zero error. To handle this case, we can use a decorator like this

{% highlight python %}
def check_divide(func):
    def inner(a, b):
        if b == 0:
            print("b can't be zero")
            return
        return func(a, b)
    return inner

@check_divide
def divide(num1, num2):
    return a/b
{% endhighlight %}

Now by using the check_divide decorator, we are making sure that the divide function doesn’t throw any error.
Working of a decorator

A decorator function takes a function as its argument. Inside this decorator function, you declare another function with the same arguments as that of the function passed to the decorator function. Whatever you want to append to your original function, you write inside this inner function and then return the passed function.

Finally, return the inner function to complete defining your decorator.

To take another example, suppose you create a function to enter a name

{% highlight python %}
def take_name(name):
    return name
{% endhighlight %}

Now you need to print this name in a sentence but you don’t want to edit the function. Therefore, you need to create a decorator

{% highlight python %}
def print_in_a_sentence(func):
    def inner(s):
        print("Hello, ", s)
        return func(s)
    return inner


@print_in_a_sentence
def take_name(name):
    return name
take_name('John Doe')
{% endhighlight %}

In this manner, you can use decorators in your python code.