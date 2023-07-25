---
layout: post
title: "NDC Oslo 2013 Top Picks"
date: 2013-06-16 13:52:04 +0000
comments: true
tags: [NDC,Conference]
summary: "I just got back from NDC Oslo. It was a great week filled with some quality speakers and content. It’s very hard to pick what I enjoyed the most, because the overall quality was just so high. Nevertheless, I’m going to attempt it :-) Bear in mind it’s my subjective view and I’m happy if you disagree with me. I will update the post once the videos are up."
thumbnail: /images/posts/ndc/IMG_1079_thumb.jpg
---

<a href="/images/posts/ndc/IMG_1104_thumb.jpg"><img src="/images/posts/ndc/IMG_1104_thumb.jpg" class="post-image-right" alt="Clean Architecture" /></a>
I just got back from [NDC Oslo](https://twitter.com/search?q=%23ndcoslo&src=hash). It was a great week filled with some quality speakers and content. It’s very hard to pick what I enjoyed the most, because the overall quality was just so high. Nevertheless, I’m going to attempt it :-) Bear in mind it’s my subjective view and I’m happy if you disagree with me. I will update the post once the videos are up.
<!--more-->

Real World Polyglot Persistence
-------------------
**by [Jimmy Board](https://twitter.com/jbogard)**

A man behind the AutoMapper is not only a great speaker, but unlike so many other talks, he brought along a real-world example. I could really identify myself with his main view of – use the right tool for the job, rather than try to basterdise something that was not build for the purpose. That means, be prepared to use lots of different storage mechanisms in a large app and lots of different ways to put the systems together.

He showed off fake e-commerce app that would

- Store the main catalogue in relational DB (changes little, mostly read, caching)
- Store the cart info in Key-Value Store instead of Session (auto-expiry, transient data)
- Store order information in Document DB (typically a different system, when catalogue changes, order does not)
- Use Graph DB for suggestion engine

Each component worked together in a way that was also closer to the real-world scenario – HTTP post between components, Messaging Bus and AJAX on front-end to different systems. You can check out the slides and the code here.

Clean Architecture and Design
-------------------
**by [Robert C. Martin](https://twitter.com/unclebobmartin)**

<a href="/images/posts/ndc/IMG_1079_thumb.jpg"><img src="/images/posts/ndc/IMG_1079_thumb.jpg" class="post-image-right" alt="Clean Architecture" /></a>
I enjoyed Uncle Bob talking about clean architecture. The abstract was a bit vague so I feared the talk will be a bit too generic. I was wrong. I really enjoyed it, in particular, because he was describing an architectural anti-pattern that I’ve seen so many times in MVC.

He stressed the importance of Web being a presentation logic and that your presentation logic (or framework) should not dictate the core business model. I’ve seen so many times where the ViewModel logic has grown beyond manageable (calling services or data access layer) or view logic grown out of proportion. He stressed the importance of keeping ViewModels and Models separate.

A good talk, great speaker.

Under the Covers with ASP.NET SignalR
-------------------
**by [Davian Fowlwards](https://twitter.com/DavianFowlwards)**

The authors of SignalR [David Fowler](https://twitter.com/davidfowl) and [Damian Edwards](https://twitter.com/DamianEdwards) came in and stole the show. The most entertaining and enjoyable talk by a long mile. After a quick demo of what is SignalR they went on to and wrote a simpler version of SignalR  - on stage – live – re-using only one helper class and a message bus class.

This was both very ballsy and impressive but also gave a great inside to what SignalR does and how. Starting with the negotiation to the different transport types. It also wasn’t all about the code – whenever something needed to be explained, they would go in a bit of detail.

It was a great session – fun, full of content and great QA at the end. Awesome guys!

C# 5
-------------------
**by [Jon Skeet](https://twitter.com/jonskeet)**

I went to couple of talks by Jon at NDC. This session could have also been called async – await :-) I would recommend the session if you never used the async-await before. Jon gives a great explanation of the async-await pattern using an example of a cardboard box that contains something the result inside.

He also went on to cover the intricacies of unit-testing the awaitable functions – really worth checking it out.

Backbone
-------------------
**by [Kim Bekkelund](https://twitter.com/kimjoar) and [Hans Inderberg](https://twitter.com/hinderberg)**

I was glad to see Backbone on the list of topics – especially now that I work with it day-to-day. The guys  showed off some good tips and patterns to use when using backbone such as

- using subviews and pushing the logic from view to model
- avoid adding too much logic in initialize
- create base-classes for each component to share logic (baseviews, basemodels), 1 level of inheritance is usually enough
- don’t be shy to use vanilla JS

Overall, good tips and issues we’ve come across as well. They also suggested grouping Backbone components (view, model) together in feature folders. I guess that was the only suggestion I would have a problem with, given you would want to re-use some of the models.

 

There were plenty of other talks that I enjoyed, but I won’t bore you with details. In particular both sessions by Dominick Baier who gave good overview of Web API security and OAuth2. I also enjoyed Dan North talking about Effective Teams & Jon Skeet on Abusing C#. All in all, great conference.