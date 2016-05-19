---
layout: post
title: Handling User Defined Signals in Go
summary : "Signals represent a limited form of inter-process communication that is used to interrupt the execution of a certain program to execute a predefined handler. In this post, we are interested in the user-defined signals \"USR1\" and \"USR2\". Those signals are there for you to define their handlers. I'll be sharing some ideas for what you can do with these signals."
description : "Signals represent a limited form of inter-process communication that is used to interrupt the execution of a certain program to execute a predefined handler. In this post, we are interested in the user-defined signals \"USR1\" and \"USR2\". Those signals are there for you to define their handlers. I'll be sharing some ideas for what you can do with these signals."
date: '2016-05-15T19:50:00+0200'
author: Mohamed Bassem
image: /img/handling-user-defined-signals-in-go/terminal.png
tags:
  - golang
  - signal
  - unix
  - usr1
  - usr2
categories :
---

Signals represent a limited form of inter-process communication that is used to interrupt the execution of a certain program to execute a predefined handler. For example, whenever you press CTRL+C in the terminal to stop a running program, you are sending a signal that's called SIGINT. Usually, the default handler for SIGINT is to kill the process, but some processes can trap (catch) this signal and ignore it or do some cleaning work before exiting. Another famous signal is the brutal SIGKILL signal. Whenever a process receives this signal it will terminate immediately without even having the option to ignore it. There are many other signals that you can find [on wikipedia](https://en.wikipedia.org/wiki/Unix_signal#POSIX_signals).

You can send a signal to a process using the command `kill <signal> <PID>` where PID is the process id. For example, to terminate a certain process you can send a SIGKILL signal using `kill -KILL <PID>`. You can use `pkill` instead to kill the process by its name.

In this post, we are interested in the user-defined signals `USR1` and `USR2`. Those signals are there for you to define their handlers. I'll be sharing some ideas for what you can do with these signals. The examples in this post are written in Go but they can also be applied with any other programming language.


### Trapping the Signal

In Go, you can trap unix signals using the function `signal.Notify(chan os.Signal, syscall.Signal)`. This function takes a channel and the signal you want to trap. Whenever the program receives the specified signal, it will send an `os.Signal` to the provided channel. So one way to handle this is :

<script src="https://gist.github.com/MohamedBassem/0f294d6df244e4def877e01d98af47af.js?ts=4&file=handle.go"></script>

Now that we know how to trap a signal and define our own handlers. Let's discuss some use cases.

### Toggling Verbose Logging

Let's say that you have a web app that has some logs. Usually, it's not a good idea to make your logs verbose as the log file will explode quickly. That's why there are many logging levels (INFO, DEBUG, WARN,..). You can start you app with one of those levels, but if, for example, you want to debug some problem, you'll have to restart your app with another logging level.

For simplicity, assume that you have only two logging levels (Silent and Verbose). The following piece of code will toggle the logging levels whenever the process receives a `USR1` signal.

<script src="https://gist.github.com/MohamedBassem/0f294d6df244e4def877e01d98af47af.js?ts=4&file=logging.go"></script>

In action :

<script type="text/javascript" src="https://asciinema.org/a/45626.js" id="asciicast-45626" async></script>

### Reloading Configuration File

One other use case for signals is to reload your configuration file without restarting your app. Let's say you have an app that reads its configuration from a YAML file and just prints a certain value from this config file. With a small modification, we can just swap the old config with the new one on receiving the signal.

<script src="https://gist.github.com/MohamedBassem/0f294d6df244e4def877e01d98af47af.js?ts=4&file=config.go"></script>

In action :

<script type="text/javascript" src="https://asciinema.org/a/45625.js" id="asciicast-45625" async></script>

### Finally

The previous uses  were just two examples that came to mind, but you can do whatever you want with the signals. If you have any other ideas, comments or questions, please share them with me in the comments.

*And as always, I want to thank @SaraAlaaKhodeir for reviewing the post. Thank you!*

 **EDIT :** Adding the mutex guarding the io.Writer. Thanks to [/u/jeffrallen](https://www.reddit.com/user/jeffrallen).
