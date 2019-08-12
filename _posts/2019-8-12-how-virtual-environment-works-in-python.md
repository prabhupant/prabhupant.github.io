---
layout: post
title: "How virtual environment works in Python"
---

A virtual environment is a tool which is used to keep dependencies used by different projects separate by creating isolated Python virtual environment for them. This is one of the most important tool used while making a Python project.

## Why use virtual environment?

Suppose you are building a web app that uses Flask 1.0.2. So you decide to system wide general installation of Flask by using `pip3 install flask=1.0.2`. After installing Flask, you start working on the project and all of a sudden you need to make changes in some other project, or start a different project in parallel. But that project is based on Flask 1.1.0. Hence, your installed version of Flask won't be compatible for the second project.

In this scenario, virtual environments play a crucial role. We create separate virtual environments for separate projects and install the required dependencies as per each project, hence separting the environment of projects.

## How virtual environment works

When Python starts its interpreter, it searches for a site-specific directory where all the packages are located. The search commences from the parent directory of the Python executable file and continues going up towards the parent directories until it reaches the root directory. To determine if it's a site-specific directory, Python looks for the os.py module, which is a mandatory requirement by Python in order to work.

## Creating a virtual environment

First install the `virtualenv` package

{% highlight bash %}
$ sudo apt-get install virtualenv
{% endhighlight %}

How go to the project directory and type

{% highlight bash %}

$ python3 -m venv venv
$ ls
venv
$

{% endhighlight %}

This will create a virtual environment called `venv` and a directory of the same name. The name is specified by the third argument of the command `python3 -m venv venv`. So changing it to `python3 -m venv virtual` will create a virtual environment with the name `virtual`.

To activate the virtual environment, type

{% highlight bash %}
$ source venv/bin/activate
(venv) $
{% endhighlight %}

`venv` will get prepended to the bash command prompt which indicates that you are inside the `venv` virtual environment. To deactivate it, simply type

{% highlight bash %}
(venv) $ deactivate
$
{% endhighlight %}

## Under the hood of virtual environment

Let's see what's under the `venv` virtual environment.

{% highlight bash %}

$ tree -L 3
.
└── venv
    ├── bin
    │   ├── activate
    │   ├── activate.csh
    │   ├── activate.fish
    │   ├── easy_install
    │   ├── easy_install-3.5
    │   ├── pip
    │   ├── pip3
    │   ├── pip3.5
    │   ├── python -> python3
    │   └── python3 -> /usr/bin/python3
    ├── include
    ├── lib
    │   └── python3.5
    ├── lib64 -> lib
    ├── pyvenv.cfg
    └── share
        └── python-wheels

8 directories, 11 files
$

{% enghighlight %}

From above, we can conclude that the environmet was created by copying the Python binary to the local directory (/venv/bin/python).

Now let's look at the environment variables i.e `$PATH`. Before activating the virtual environment, the `$PATH` variable points to the original location of the executable binaries but after activating the virtual environment, it changes the `$PATH` variable in such a way that Python will point to our local version. So basically it prepends our local path to the `$PATH` variable.

{% highlight bash %}
prabhu@prabhu-pc:~/project$ which python
/usr/bin/python

prabhu@prabhu-pc:~/project$ source venv/bin/activate
(venv) prabhu@prabhu-pc:~/project$ which python
/home/prabhu/project/venv/bin/python
{% endhighlight %}

But copying the executable to the new local location is not a good idea. So in Python3, the `pyvenv.cfg` file is used to created instead of Python binary and its modules and you can specify their location in this config file.

{% highlight bash %}
(venv) prabhu@prabhu-pc:~/project/venv$ cat pyvenv.cfg
home = /usr/bin
include-system-site-packages = false
version = 3.5.2
(venv) prabhu@prabhu-pc:~/project/venv$
{% endhighlight %}
