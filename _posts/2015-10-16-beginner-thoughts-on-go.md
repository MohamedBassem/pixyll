---
layout: post
title: Beginner Thoughts on Go
summary : "A few months ago, I couldn't answer the \"What's your main programming language?\" question. I knew many programming languages during my university studies, internships and side projects but I didn't dig deep into any of them. I decided I wanted to learn something new and dig deeper into it. That's when I first found out about Google's golang. I started learning go and worked on some side projects with it, and I actually like it. A few weeks ago, I started building a big sized backend service for @zoobeapp using go and it's in production now. In this blog post I'll write about some of my thoughts on Go after I've been using it for a few months now."
description : "A few months ago, I couldn't answer the \"What's your main programming language?\" question. I found out about Google's golang and started learning it. In this blog post I'll write about some of my thoughts on Go after I've been using it for a few months now."
date: '2015-10-16T21:30:00+2000'
author: Mohamed Bassem
image: /img/beginner-thoughts-on-go/gopher.png
tags:
  - go
  - golang
  - programming
  - languages
categories :
---

A few months ago, I decided to review my knowledge of programming languages and check whether I was up to date with the current technologies or not. I learned Java in my 1st year of university and it has been my main programming language for [university](https://github.com/MohamedBassem/RDBMS-T2) [projects](https://github.com/MohamedBassem/pipelined-MIPS-simulator) for almost two years. I liked Java back then. Java code seems very organized and readable, but coding in Java is very boring. Everything seems too old. Lots of code to do trivial things, even if it's automatically generated. But mainly, Java didn't seem like the suitable language to fulfill my passion at that time, Competitive programming.

Some of the main things you care about in competitive programming are the runtime and how long it takes to code your solution. Compared to C++, Java fails at both. I decided to switch to C++ for competitive programming. C++ macros can make you code the solution faster. C++ STL contains most of the things that you may need. From the runtime point of view, C++ is significantly faster. If there is a reason to solve a problem in Java, it would be Java's BigInteger class. My knowledge of C++ was limited to competitive programming until we had to work on [a](https://github.com/MohamedBassem/tetris) [project](https://github.com/medo/raafat-elhagan) for my graphics course at university using OpenGL and C++. That's when I found I know nothing about C++. The same happened again, a year later during my internship at @zoobeapp. I had some tasks to work on for a huge C++ project under the mentorship of @aguperezpala who is a very talented C++ developer.  But will I ever write a web app in C++? Never!

[![AgustinCodeReview](/img/beginner-thoughts-on-go/AgustinReview.png)](/img/beginner-thoughts-on-go/AgustinReview.png){:: data-lightbox="img1"}

<p class="image-caption">Agustin's code review on my two c++ pull requests.</p>

Besides C++ and Java, I started learning python. I worked on some [projects](https://github.com/MohamedBassem/dTests) in python as practice. Honestly, I didn't give python enough time and effort, especially since I was also learning Ruby at the same time and it was more interesting for me. I started to learn ruby during my internship at @Trustious. Ruby has a very good community and lots of libraries (gems). Coding in Ruby is fun and Rails is awesome. Then I started exploring Node.js. Honestly, I can't imagine writing a big project in javascript. Anyways, I didn't dig deep into any of those three languages.

At that point, I couldn't answer the "What's your main programming language?" question. Even in an internship interview at Google, I solved the algorithms problem in C++ and the design problem in Java. Which, honestly, was awful. I decided I wanted to learn something new and dig deeper into it. I spend lots of time on [Hacker News](https://news.ycombinator.com/), that's where I first found out about Google's golang. I started learning go and worked on some [side](https://github.com/MohamedBassem/getaredis) [projects](https://github.com/MohamedBassem/servgo) with it, and I actually like it. A few weeks ago, I started building a big sized backend service for @zoobeapp using go and it's in production now.

In this blog post I'll write about some of my thoughts on Go after I've been using it for a few months now.

###Go

[![AgustinCodeReview](/img/beginner-thoughts-on-go/gopher.png)](/img/beginner-thoughts-on-go/gopher.png){:: data-lightbox="img2"}

*Image from https://github.com/golang-samples/gopher-vector*

####Concurrency Model

Go has an awesome concurrency model. Starting a function in the background is as easy as prepending the keyword "go" to the function call which runs the function in a separate lightweight thread of execution known as goroutine. Communication between different goroutines is done using what's called a channel. So for example implementing the producer-consumer pattern in go would be something like :


{% highlight go %}
package main

import (
	"fmt"
	"time"
)

func consumer(id int, queue chan string) {
	for {
		str := <-queue
		fmt.Printf("Consumer %v : %v\n", id, str)
	}
}

func producer(queue chan string) {
	for i:=0; i < 20; i++{
		queue <- fmt.Sprintf("New Job %v", i)
		time.Sleep(time.Second)
	}
}

func main() {
	maxBuffer := 20
	numConsumers := 3
	jobQueue := make(chan string, maxBuffer)
	for i := 0; i < numConsumers; i++ {
		go consumer(i, jobQueue)
	}
	producer(jobQueue)
}
{% endhighlight %}

You can run this example here : [https://play.golang.org/p/2kg6QwySS6](https://play.golang.org/p/2kg6QwySS6)

####Panics and Errors

Go has a different method for handling errors. It doesn't have the concept of exceptions. Instead, since functions can return multiple params, errors are returned as a function return param. There has been a lot of discussion about this in the golang mailing list. As for myself, I can't really determine whether I like this new model or not.

{% highlight go %}
if err != nil {
    return nil, err
}
{% endhighlight %}

Since errors won't propagate by default, my code is filled with this "if err; return err" block. But at the same time I've always hated the try and catch blocks of Java in my code. I guess it's just a matter of taste.

Go also has what's called `panic` and `recover` but it's not encouraged to use them as `throw` and `catch` and they even behave differently.

####One-Liners

I'm a big fan of one-liners. I've always felt a sense of accomplishment when chaining methods in ruby together. Since functions in go can have multiple return params and return params are not indexable, and also since most of the times functions will have the extra error return param, function chaining is not easy anymore.

####Defer

One of the things I like the most about Go is the `defer` keyword. Defer pushes a function in a functions stack so that it is executed when the surrounding function returns. So instead of opening a file and having to close it before each return, you can just defer it after you've opened it directly. Another example would be releasing a lock after acquiring it.


{% highlight go %}

func dummy() {
    mutex.Lock()
    defer mutex.Unlock()
    // ...
}

{% endhighlight %}

####Reflecting Types

*Don't know if that's the suitable name*

In go, you can't do something like `"user".camelize.constantize.first`. You can't instantiate an instance of a struct while having it's name as a string. The way this can be done is by maintaining a large `map[string]interface{}` with a name to struct instance mapping. I still haven't found a good way to maintain such a map.

####Go-Vim

[Go-Vim](https://github.com/fatih/vim-go) has become one of my favourite vim plugins. With the help of go native tools, Go-Vim provides:

- automatic code formatting on save. Using `go fmt`.. Yes, Go has a standard code formating!
- automatically importing packages
- going to variable definition
- going to the documentation of a function

And many other things that you can find in the plugin's repo.

####Packages

I have a problem structuring my go code. Go files in the same directory must be in the same package. Packages can be built and even pulled on their own. The problem is, this kind of discourages me from separating my go files into directories (packages). I need something that binds some files that's stronger than having all the files in the same package yet weaker than separating them into completely independent package on their own. Some advice is needed here please :blush:

####Encapsulation

Go fields and methods have two access modes. Either public or private depending on the case of the first character. Go's private is Java's no access modifier. Meaning that any private field or method in a struct can be accessed by any function from the same package. Sometimes that's not what I want and I don't want to separate that struct into a separate package for the reasons I mentioned when talking about packages.

####Interfaces

Most of the posts I read mention Go interfaces as one the best language features. I understand the basic concept of implicitly implementing the interface but I still haven't been able to fully leverage its powers, so I'd really appreciate it if someone knows some good resources for it.

####Misc

- Go is a compiled language. It can be compiled into a single completely independent binary. You can also cross compile go code to different OS or architecture. This makes deploying go programs very easy.
- Go is a garbage-collected language so you don't need to worry about freeing unneeded objects. The garbage collector gets better with every new language version.
- Go comes natively with some nice tools, some example are: code formating, testing and benchmarking frameworks, pulling other libraries from github or other hosted repositories.
- Go standard library doesn't contain some of the basic data structures such as queues and stacks but you can easily `go get` them from many places.

### Resources

If you are willing to learn go, these are some of the resources that helped me.

- [Effective go](https://golang.org/doc/effective_go.html).
- [Learn X in Y minutes](http://learnxinyminutes.com/docs/go/).
- [Go Subreddit](https://www.reddit.com/r/golang/).
- [Go by Example](https://gobyexample.com/).
- [Go Forum](https://forum.golangbridge.org/) - It's still new.
- [Awesome Go](https://github.com/avelino/awesome-go).

I only have a few months of experience with go so I'm still a beginner. All of the previous points are just my thoughts that may or may not be wrong. If you have any comments/thoughts/resources feel free to share them in the comments.

*And as always, I want to thank @SaraAlaaKhodeir for her review. Thank you!*
