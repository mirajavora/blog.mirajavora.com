---
layout: post
title: "Open Sourcing Sendgrid Webhooks Library"
date: 2015-09-31 10:12:30 +0000
comments: true
image: /images/posts/sendgrid/sendgrid-stats-small.png
summary: ""
categories: [sendgrid, metrics, opensource, csharp]
---

It's been a while since I wrote an open-source contribution. A recent mini-project as I was involved required parsing of Sendgrid Webhooks in C#.

As it turned out, there wasn't much around and Sendgrid didn't have an official library.  Although I don't sent to write a lot of C# these days, it was a good opportunity.
<!--more-->


Sendgrid Webhooks
-------------------
Sendgrid is a 3rd party service that provides email delivery on your behalf, providing you with an API.
On top of that it can give you a functionality, where every user action can result in a callback to your endpoint.
This effectively means that every time an email is processed, send, open, clicked, spam reported, unsubscribed or bounced - it will call your servers with additional details.

If you're serious about your channel, this is important, because you get info such as clicked url, IP address or the user-agent of the client.
Plus, you can pass custom arguments along with the API calls as email meta-data that make it to the callbacks as well.

Parse the Webhook Callbacks
-------------------
I wanted to provide a service that would take the JSON in the callback and turned that into a polymorphic list of Sendgrid Webhook Events.
Sendgrid tends to aggregate the callbacks into batches so you usually receive more than 1 event in a single callback.

[TODO code snippet]

### Custom Serialisation







### Data flows


### Contributions


Joining list of libraries on Sendgrid page
-------------------
We went ahead an