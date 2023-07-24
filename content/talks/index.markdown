---
layout: page
title: "Talks"
comments: true
---
Async Realtime Web Development with SignalR
-------------------

Achieving a truly real-time web application with the stateless HTTP protocol can be challenging.  Retrieval of new data with web-client has traditionally been achieved by continuously polling the web-server.

In order to accomplish a more sophisticated solution, clients can use long-polling with AJAX or web-sockets. Combined with JQuery and JSrender, we can easily create a true real-time experience, where data is pushed to all clients immediately. This session will demonstrate how to create an asynchronous web application based on the publish-subscribe pattern with .NET back-end. We will cover the basic architecture, background and the publish-subscribe model. The main focus will be on getting up and running with your first real-time asynchronous app, which will be based on MVC4/SignalR/JQuery/JSrender.

### DDD10 September 2012

[Slides](https://skydrive.live.com/redir?resid=84E23A97F665C5F2!229)  [Source Code](https://skydrive.live.com/redir?resid=84E23A97F665C5F2!230)

### DunDDD November 2012

[Slides](http://sdrv.ms/Qqack2) [Source Code](https://github.com/mirajavora/signalr-f1live)



*Run /boot to get your DB ready - you can check the default connection string in web.config. Please note the source code is intended as a demonstration of SignalR concepts and deliberately breaks several patterns and what is considered a good practice (logic in controllers, domain object passed down to clients, untidy JS and more)*