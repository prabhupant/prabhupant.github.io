---
layout: post
title: "Processes and Threads"
---

Threads and processes can seem to be the same thing with almost zero or no differences. But there is a lot that separates them.

## A Process

A process at its core is an abstraction of a running program. It is an instance of an executing program, including the current value of the the program counter, registers and variables. All the runnable software on the computer is organized into a number of processes, ranging from input-output tasks (called IO bound processes) to computational based tasks (called CPU bound processes)

In their infancy, computers used to perform a single process at a time but nowadays they can handle plethora of them with ease. They use different algorithms and technologies to do this like multiprogramming etc.

When you open a terminal session and type a command, the shell creates a new process that does the job and return the output to the screen. This way, the current session for the user is maintained and the user doesn't leave the touch with the ongoing process (shell).

Processes in Linux systems are created by the OS using `fork()` which makes a system call and returns the process ID, or PID. The process that creates a process is called `parent` and the created process is called `child`.

Here is the basic implementation in C.

{% highlight c %}
#include  <stdio.h>
#include  <string.h>
#include  <sys/types.h>

#define   MAX_COUNT  200
#define   BUF_SIZE   100

void  main(void)
{
     pid_t  pid;
     int    i;
     char   buf[BUF_SIZE];

     fork();
     pid = getpid();
     for (i = 1; i <= MAX_COUNT; i++) {
          sprintf(buf, "This line is from pid %d, value = %d\n", pid, i);
          write(1, buf, strlen(buf));
     }
}
{% endhighlight %}


## A Thread

A thread is the smallest unit of processing that can perform a task. It is the single line of execution of sequential command that a process wants to execute. It can be thought of a labour that a process creates to perform some task.

## Thread vs Process

A process and thread might look like the same from the top level of abstraction but underneath they are quite different. A thread is created by a process and the thread executes within the process.

A process is used to group resources together. These may include open files, child processes, signal handlers, etc. By putting them together in the form of a process they can be managed more easily if they point to a single task

On the other hand, a thread must execute in some process. It can't use resources that fall outside the scopre of the process in which it is running.


## Creating a process in Python

The `os` module in Python provides a way of using operating system dependent functions. It has two functions that can be used to create a process - `fork()` and `getpid()`.

`fork()` function takes no argument and returns 0 if everything goes well and a child process is created. Checking this condition, we create a function called `child()` that returns the PID of the new child process created.

{% highlight python %}
import os

def child():
   print('New child created - ',  os.getpid())
   os._exit(0)

def parent():
   while True:
      newpid = os.fork()
      if newpid == 0:
         child()

parent()

{% endhighlight %}

## Creating a thread in Python

Python's `threading` library provides easy thread creation and management. Here we are going to make a simple thread to print numbers from 1 to 10. There are two ways to create a thread in Python -

* ### With function call

{% highlight python %}
from threading import Thread
import time

def loop1_10():
    for i in range(1, 11):
        time.sleep(1)
        print(i)

Thread(target=loop1_10).start()

{% endhighlight %}

* ### With object

{% highlight python %}
from threading import Thread
import time

class MyThread(Thread):
    def run(self):
        for i in range(1, 11):
        time.sleep(1)
        print(i)

def main():
    mythread = MyThread(name = "Thread-{}".format(x + 1))
    mythread.start()

if __name__ == '__main__':
    main()
{% endhighlight %}

## GIL and when to use threading

In CPython, the global interpreter lock, or GIL, is a mutex that protects access to Python objects, preventing multiple threads from executing Python bytecodes at once. In other words, what the GIL prevents is, is making use of more than one CPU core or separate CPUs to run threads in parallel. Hence,GIL makes sure only one thread is executing Python code at a time; control still switches between threads.

Therefore in Python, if you want to speed up a computational process by breaking it down in threads, then it won't be effective as the GIL will only allow one thread to execute the Python bytecode at a time, making the other threads useless.

So the only possible scenario when you can use threading in Python is for IO bound processes. For example, if you are making a bulk SMS sending program and to send a SMS the Python code makes an API call to the SMS sending service like Twilio, then you can make threads for each individual API call, thereby reducing the dependency of the runtime of your code on the speed of the internet or the time it takes to receive the response from the server.
