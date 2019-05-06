---
layout: post
title:  "Parallelism in Ruby"
date:   2019-05-06 22:15:42 +0200
categories: ruby parallelism gil
---

If you ever tried to achieve true parallelism in Ruby, you sure faced one of these two problems:

1. I want to use threads, but I have to use a build different from MRI due to the global interpreter lock.
2. I want to user processes, but it is very hard to do reliable and efficient IPC (inter process communication).

This post is going to offer you a solution that:

1. Does not require you to change your Ruby build.
2. Provides you an easy and fast way to make your process communicate with each other.

#### __THE GLOBAL INTERPRETER LOCK__

The biggest drawback of using MRI with threads is the Global Interpreter Lock (GIL). The GIL is a mechanism that prevents two or more threads of the same process to execute simultaneously. This is a simple solution to achieve the so called `thread-safety`, i.e. avoiding two concurrent threads to modify the same variable at the same time.

This is useful because it removes the burden of mutual exclusion from the programmer's shoulders, but comes at the cost of losing true thread parallelism. _Soft parallelism_ can still be achieved if one thread of the process is IO-blocked (for example is reading from a file) and thus not using CPU cycles. The GIL sees that and, if another thread of the same process is requesting CPU time, it allows it to run, thus enabling some sort of parallel code execution.

But no matter how many cores you have in your machine, if you need simultaneous CPU time from two or more threads in your process, you simpy cannot do it when the GIL is there.

#### __THE INTER-PROCESS COMMUNICATION__

So what if you want to do true parallelism in Ruby? There's always the solution of using processes instead of threads. They are real, multi-core, parallel-enabled awesome sauce of programming excitement (ok maybe a little exaggerating here :D), but they have two big problems:

1. They don't share variables.
2. They must communicate with an extra-software tool.

When you spawn another process to execute a block of code in parallel, all the variables and memory space of your father process is going to be copied to the child and any possibility of sharing is lost. Actually, Unix systems use Copy-On-Write, which copies only resources that are modified by the child process, and uses the father's memory pointer if the child is only accessing the resource, but that doesn't change the outcome.

Thus, if you want to share data between sibling processes you must use an OS communication tool like: IO pipes, Sockets, Shared Objects, and so on. This is old and trusty tech, but that comes at the cost of doing always some extra low-level implementation, which is not something you want to do nowadays.

#### __PARALLAX: MY SOLUTION TO THE PROBLEM__

This is why I created the [Parallax](https://github.com/Pluvie/ruby-parallax) gem. This gem gives you a very simple yet efficient framework to do easy IPC between your processes. Even better, you don't have to deal with processes in any way, just focus on the best way to write a piece of code that has to be run in parallel.

This gem gives you the possibily of sending messages from child processes to the father one (which acts as a `collector`). Messages can contain complex Ruby object, which are de-serialized and serialized, preserving their structure.

Other than that, this gem gives you the possibility of calling actual methods in the father's collector object, enabling you a whole new level of parallelism capabilities.

To learn how to use the gem, refer to the link above.

Thanks for reading, see you soon!

---


Disclaimer: this post is going to be use as application blog post in [Toptal's Ruby developer page](https://www.toptal.com/ruby).
