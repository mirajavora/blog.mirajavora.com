---
layout: post
title: "Open Sourcing Sendgrid Webhooks Library"
date: 2015-10-09 13:12:30 +0000
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
Since we're dealing with JSON - I pulled in a single dependency on the project. It is the Newtonsoft.JSON library.
ServiceStack.Text is my preferred option with a lower footprint and marginally higher performance, however, in this instance Newtonsoft.JSON is the one that is used more widely.

A sample that you may get from Sendgrid

[TODO insert sendgrid code sample]

The custom serialisation used the type attribute of each event to determine the underlying event type.
Each event was based of the same base X.

[TODO insert custom parse event]

I have also added couple of custom serialisers that deal with custom date conversion or csv to array. That way you have access to a proper DateTime date and List<String> of categories.

[TODO insert custom parsers]


Check out the code at [TODO LINK]


### Contributions
A surprising number of people started using the code and we've had few contributions already. Thanks to Andy McCready and [TODO]

Joining list of libraries on Sendgrid page
-------------------
And finally, I got in touch with Sendgrid if they wanted to add our community driven library for the webhooks on their list of libraries they suggest to devs.
They did! So now you can easily find that in the list!