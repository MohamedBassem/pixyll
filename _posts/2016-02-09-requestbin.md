---
layout: post
title: RequestBin
summary : ""
description : ""
date: '2016-02-09T13:49:00+0000'
author: Mohamed Bassem
image: 
published: false
tags:
  - golang
  - requestbin
  - testing
categories :
---

At Zoobe, we have an important component in our backend that is called the "Push Notification Server" or the PNS. The responsibility of the PNS is collecting events through its API, aggregating, generating, localizing, scheduling and finally delivering push notifications to our users. The PNS is written in Go generates thousands of push notifications everyday. That's why we have it covered with many unit and functional tests.

The PNS uses Parse's API for delivering the PNs to the users. One day I thought it may be a good idea to the deliveryers logs more verbose so I added:

{% highlight go %}
func deliver(){
  // .........
  request, err := getParseRequest(..)
  body, _ := ioutil.ReadAll(request.Body)
  log.LogParseBody(body)
  if request != nill {
    _, err := &(http.Client{}).Do(request)
  }
  // .........
}
{% endhighlight %}

You may have noticed the problem by now but back then I didn't. Also during development I usually have the condition as `if false && request != nil` to avoid spamming our parse app with useless notifications. The addition was ready and tests were passing so we deployed it. Immediately after deploying it we noticed that all the requests to parse were failing.

The problem with the line I added was that it read the Body Reader before the request can do, so all the requests were sent to parse with no body. We quickly deployed a fix and everything went back to normal but there were some lessons learned. We didn't have any thing testing the outgoing requests to a remote server in our test suite. That's why I developed [RequestBin](https://github.com/MohamedBassem/requestBin).

*The name RequestBin is originally used in [http://requestb.in/](http://requestb.in/) which does the same thing but using a web interface.*

### Mocking Remote Servers

RequestBin is an open source golang package that is used to mock remote servers. It starts a mock server locally, runs a user defined function passing the mock server's URL to it and collects all the requests that were sent to the server.

Having this package we added many more tests that emulates events and then expects requests to be sent to the mock server ( as if it's parse ). For example, a sanity check for the like push notification would be written like:


{% highlight go %}
func TestLikeSanityCheck(t *testing.T) {
  reqs := requestBin.NewRequestBin(requestBin.MockServerConfig{
    MaximumRequestCount: 1,
    GlobalTimeout: 1,
  }).CaptureRequests(func(url string) {
    tmp := config.PARSE_ENDPOINT
    config.PARSE_ENDPOINT = url
    emulateLikeEvent()
    config.PARSE_ENDPOINT = tmp
  })

  assert.Equal(t, 0, len(reqs))
  assert.Equal(t, "{...}", reqs[0].Body)
  assert.Equal(t, "...", reqs[0].Headers.Get("PARSE_APP_KEY"))
  assert.Equal(t, "POST", reqs[0].Method)
}
{% endhighlight %}

The server in code above expects one request and then it will return. If a request was not received within one second timeout the server will exit also and the `reqs` slice will be empty. It may be the case also that the function you call keeps generating a certain amount of requests and then it will stop, you can capture the requests sent by this function by what's called a RequestTimeout. If the RequestTimeout is set, the server has to receive a new request every `RequestTimeout` seconds or it will exit.

### Contribution

As I mentioned the package is open source and hosted on Github: [https://github.com/MohamedBassem/requestBin](https://github.com/MohamedBassem/requestBin). Your contributions with code, feature requests, issues or ideas are welcome. The code is simple, small and documented so it should be easy to contribute.

*Let me know what you think of this package in the comments below. Thanks for reading!*
