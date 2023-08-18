---
layout: post
title: "Open Sourcing Sendgrid Webhooks Library"
date: 2016-05-21 13:12:30 +0000
comments: true
thumbnail: /images/posts/webhooks/github.png
summary: "It's been a while since I wrote an open-source contribution. A while back, a mini-project I was involved required parsing of Sendgrid Webhooks in C#. As it turned out, there wasn't much around and Sendgrid didn't have an official library.  Although at the time I pretty much stopped writing any C#, it was a good opportunity for an open source project."
tags: [sendgrid, metrics, opensource, github, csharp]
archive: 2016
aliases:
    - /sendgrid-webhooks-csharp-library/
---

It's been a while since I wrote an open-source contribution. A while back, a mini-project as I was involved required parsing of Sendgrid Webhooks in C#.
As it turned out, there wasn't much around and Sendgrid didn't have an official library.  Although at the time I pretty much stopped writing any C#, it was a good opportunity for an open source project.

Since then, it was included on Sengrid's [library list](https://sendgrid.com/docs/Integrate/libraries.html#-C-sharp-webhooks) and so it means we're famous right ..? Also, people wrote about it on [stack overflow](http://stackoverflow.com/questions/31218563/can-my-asp-net-code-get-confirmation-from-sendgrid-that-an-email-has-been-sent/31248116) so we're def famous!
<!--more-->


Sendgrid Webhooks
-------------------
Sendgrid is a 3rd party service that provides email delivery on your behalf, providing you with an API.
On top of that it can give you a functionality, where every user action can result in a callback to your endpoint.
This effectively means that every time an email is processed, send, open, clicked, spam reported, unsubscribed or bounced - it will call your servers with additional details. I blogged about creating [reporting from the sendgrid webhooks earlier on](/sendgrid-webhooks-reporting/).

If you're serious about your channel this is super important, because you get info such as *clicked url, IP address or the user-agent of the client*.
Plus, you can pass custom arguments along with the API calls as email meta-data that make it to the callbacks as well.

Parse the Webhook Callbacks
-------------------
I wanted to provide a library that would take the JSON in the callback and turned that into a polymorphic list of Sendgrid Webhook Events.
Sendgrid tends to aggregate the callbacks into batches so you usually receive more than 1 event in a single callback.

{{< highlight json "linenos=inline" >}}
    [
      {
        "email": "john.doe@sendgrid.com",
        "timestamp": 1337197600,
        "smtp-id": "<4FB4041F.6080505@sendgrid.com>",
        "event": "processed"
      },
      {
        "email": "john.doe@sendgrid.com",
        "timestamp": 1337966815,
        "category": "newuser",
        "event": "click",
        "url": "https://sendgrid.com"
      }
    ]
{{< / highlight >}}


### Using the library

The library allows you to easily deseralise the callbacks and create a list of typed events of a common base. All custom fields, date and list conversions are handled for you.

{{< highlight csharp "linenos=inline" >}}
    var parser = new WebhookParser();
    var events = parser.ParseEvents(json);

    var webhookEvent = events[0];

    // Shared base properties
    webhookEvent.EventType; // Enum - type of the event as enum
    webhookEvent.Categories; // IList<string> - list of categories assigned ot the event
    webhookEvent.TimeStamp; // DateTime - datetime of the event converted from Unix time
    webhookEvent.UniqueParameters; // IDictionary<string, string> - map of key-value unique parameters

    // Event-specific properties
    var clickEvent = webhookEvent as ClickEvent; // Cast to the parent based on EventType
    clickEvent.Url; // string - URL on what the user has clicked
{{< / highlight >}}

Serialisation
-------------------
Since we're dealing with JSON - I pulled in a single dependency on the project. It is the Newtonsoft.JSON library.
`ServiceStack.Text` is my preferred option with a lower footprint and marginally higher performance, however, in this instance Newtonsoft.JSON is the one that is used more widely.

The custom serialisation used the type attribute of each event to determine the underlying event type.
Each event was based of the same base `WebhookEventBase`.

{{< highlight csharp "linenos=inline" >}}
  namespace Sendgrid.Webhooks.Events
  {
      public abstract class WebhookEventBase
      {
          public WebhookEventBase()
          {
              UniqueParameters = new Dictionary<string, string>();
          }

          [JsonProperty("event"), JsonConverter(typeof(StringEnumConverter))]
          public WebhookEventType EventType { get; set; }

          [JsonProperty("email")]
          public string Email { get; set; }

          [JsonProperty("category"), JsonConverter(typeof(WebhookCategoryConverter))]
          public IList<string> Category { get; set; }

          [JsonProperty("timestamp"), JsonConverter(typeof(EpochToDateTimeConverter))]
          public DateTime Timestamp { get; set; }

          public IDictionary<string, string> UniqueParameters { get; set; }
      }
  }
{{< / highlight >}}

I have also added couple of custom serialisers that deal with custom date conversion or csv to array. That way you have access to a proper DateTime date and List<String> of categories. You can also delcare the `WebhookParser` with your own version of the `JsonConverter`, allowing for virtually any changes possible.

The custom converters are the `WebhookCategoryConverter`

{{< highlight csharp "linenos=inline" >}}
    namespace Sendgrid.Webhooks.Converters
    {
        public abstract class GenericListCreationJsonConverter<T> : JsonConverter
        {

            public override bool CanConvert(Type objectType)
            {
                return true;
            }

            public override bool CanRead
            {
                get { return true; }
            }

            public override bool CanWrite
            {
                get { return false; }
            }

            public override object ReadJson(JsonReader reader, Type objectType, object existingValue,
                JsonSerializer serializer)
            {
                if (reader.TokenType == JsonToken.StartArray)
                {
                    return serializer.Deserialize<List<T>>(reader);
                }
                else
                {
                    T t = serializer.Deserialize<T>(reader);
                    return new List<T>(new[] {t});
                }
            }

            public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
            {
                throw new NotImplementedException();
            }
        }
    }
{{< / highlight >}}


and `EpochToDateTimeConverter`

{{< highlight csharp "linenos=inline" >}}
    namespace Sendgrid.Webhooks.Converters
    {
        public class EpochToDateTimeConverter : JsonConverter
        {
            private static readonly DateTime EpochDate = new DateTime(1970, 1, 1, 0, 0, 0, DateTimeKind.Utc);

            public override bool CanConvert(Type objectType)
            {
                return objectType == typeof(DateTime);
            }

            public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
            {
                if (value == null)
                    return;

                var date = (DateTime) value;
                var diff = date - EpochDate;

                var secondsSinceEpoch = (int) diff.TotalSeconds;
                serializer.Serialize(writer, secondsSinceEpoch);
            }

            public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
            {
                var timestamp = (long) reader.Value;

                return EpochDate.AddSeconds(timestamp);
            }
        }
    }
{{< / highlight >}}

Code
-------------------

Check out the code at [https://github.com/mirajavora/sendgrid-webhooks](https://github.com/mirajavora/sendgrid-webhooks) or pull down from nuget using

{{< highlight bash "linenos=inline" >}}
    Install-Package Sendgrid.Webhooks
{{< / highlight >}}

Contributions
-------------------
A surprising number of people started using the code and we've had few contributions already, which is great! Thanks to [Andy McCready](https://github.com/andymccready), [vanillajonathan](https://github.com/vanillajonathan), [brianp101](https://github.com/brianp101) and [Petteroe](https://github.com/Petteroe).