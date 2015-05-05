---
layout: post
title: Codeforces Total Account Takeover
summary : ""
description : ""
date: '2015-05-06T17:49:00+0000'
author: Mohamed Bassem
image: /img/my-journey-with-trustious/IMG_2034_Modified.JPG
tags:
  - Codeforces
  - Security
  - CSRF
categories :
---

I was playing in Codeforces and inspecting the networking of the website when I decided to check if Codeforces is vulnerable to CSRF ([Cross-Site Request Forgery](http://en.wikipedia.org/wiki/Cross-site_request_forgery)) attacks or not. I found that all requests contain CSRF tokens but I decided to test it anyways. I copied the request to my terminal and removed the CSRF token and it's working! I tried with different requests and apparently the CSRF tokens - although they exist - are never validated. Codeforces is vulnerable to CSRF attacks.

I decided to write a proof of concept before reporting it to Mike (Codeforces Admin). I wrote a script that can be injected in any page as a hidden Iframe which will change the email of the user without noticing. It will also send a forget password email to the attacker's email to notify him that a new account is hacked (Codes are attached at the end of the post). It worked! Whenever the victim opens this page his account will be hijacked without him even noticing. A Codeforces blog post containing a link to this page will yield many accounts. I decided to deploy it live to make sure it's working before reporting it.

### Origin Header Validation
To test it live, I span a digital ocean droplet and installed nginx for serving the webpage. I deployed the script but it's not working. Opening the request manually showed the following message:

[![CF CSRF Protection](/img/codeforces-csrf/csrf-protection.png)](/img/codeforces-csrf/csrf-protection.png){:: data-lightbox="img3"}

I diffed the request from the live website with my local request and the differences were the Referer and the Origin headers in the request.

[![Request Headers diff](/img/codeforces-csrf/headers-diff.png)](/img/codeforces-csrf/headers-diff.png){:: data-lightbox="img3"}

<p align="center" class="image-caption">Localhost on the left, The live server on the right.</p>


Origin headers are added by the browser when it's making a cross origin request to tell the website from where this request is originated. Apparently validating the Origin header was Codeforces' way for protecting itself from CSRF attacks and it makes sense. I googled for a while for a way to bypass the Origin header and was about to lose hope when I found this [post](http://garage4hackers.com/showthread.php?t=6023) and that was exactly what I was searching for.

The post suggests that base-64 encoding the Iframe would make the Iframe parent-less and Origin header would be null. I tried it and it's working! Now I do have a working website for Codeforces accounts hijacking.

### Unique Email Validation
Changing the password of the user would have been a better target for the request forgery. Codeforces requires the old password in order to change your password which is something we don't have. Changing email is the second best target because having control on the email you can reset the password and have full control on the account. The problem with email hijacking is that email fields are unique. If this vulnerability was used for mass accounts hijacking only the first account would be hacked and all other requests will get "This email is used by another account error".

The first thing that comes into mind is changing the static html to a dynamic website which changes the email with each request. The problem with that is that it would need a large amount of emails and it would be a pain for the hacker to check all those emails.

I remembered the gmail dots thing which you can read about it [here](https://support.google.com/mail/answer/10313?hl=en). Gmail ignores the dots in the email address. For instance "medoox240@gmail.com", "medoox.240@gmail.com" and even "medoox...240@gmail.com" are all the same for gmail. They will all go to the same inbox. For Codeforces they are different emails which is exactly what we want. The dynamic website will permute the place of the dot in the email and then can start adding more dots to the email giving us a huge amount of emails that leads to the same inbox. Now we are done.

I recorded a video as a proof of concept and uploaded the scripts and reported the vulnerability to Mike. Few days later the bug was fixed.

### Timeline
22/4/2015 - Reported the bug.

22/4/2015 - Mike confirmed the bug.

28/4/2015 - The bug was fixed.


### Proof of concept and codes.

<iframe width="560" height="315" src="https://www.youtube.com/embed/Znw9bpa-sWk?rel=0" frameborder="0" allowfullscreen></iframe>

Codes : [https://gist.github.com/MohamedBassem/cbbf60c1393bafe6052b](https://gist.github.com/MohamedBassem/cbbf60c1393bafe6052b)

<br />
**In a parallel universe,** While you were reading this, You could have lost your CF account :smiling_imp:

I want to thank Mike for his fast response and fix. I want also to thank @SymbianSyMoh for his [post](https://www.facebook.com/SymbianSyMoh/posts/1111635182184901?pnref=story) [In Arabic] that motivated me to search for vulnerabilities.

Your comments are welcomed!
