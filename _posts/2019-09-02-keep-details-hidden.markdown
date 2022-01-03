---
layout: post
title:  "Keep all details hidden"
date:   2019-09-02 21:00:00
tags: architect, oop
comments: true
shareable: true
author: randommancer
preview: While integrating with different backend apps, there is a good chance we face some communication problems, especially if the company lacks the presence of an architect and leaves space for the teams to do whatever they want. With no proper documentation and standards, working with other teams could become a tricky task.

image: https://maviteixeira.com/images/prank_callers.png
--- 

While integrating with different backend apps, there is a good chance we face some communication problems, especially if the company lacks the presence of an [architect](https://maviteixeira.com/2019/08/02/software-architect.html) and leaves space for the teams to do whatever they please. With no proper documentation and standards, working with other teams could become a tricky task.

One of the most common issues in project communication is sending data in non-standard formats, which forces us to adapt our code to process it.

<figure class="articleimg">
    <img src="{{page.image}}" alt="Prank callers">
    <figcaption>
    Regular Show - Prank callers, by J.G. Quintel
    </figcaption>
</figure>

To illustrate the problem, let's say we need to receive messages from 3 different apps. All of them are sending a string that works as a client ID. Our job is to find that specific client and process something depending on the sender. 
The problem is the client ID is being passed differently by the apps. 

To make everything clear, you can check the formats below:

- FizzApplication - `12345612341234123456`
- BuzzApplication - `CLIENT:12345612341234123456`
- DogApplication  - `123456-1234-1234123-456`

Usually, when we need to transform the entries on some specific format, we create a `Parser`, `Converter` or maybe put all this logic in a service like in this code below:

{% highlight java %}
    public class ClientService {
        private final ClientConverter clientConverter;
        Person fetch (int id);
        public Client findClient(ClientDto clientDto) {
            String clientIdentity = clientConverter.convertClientIdentity(clientDto.getIdentity());
            //Search in the database and return the Client
    }
{% endhighlight %}

`ClientService` is receiving a `DTO` with the client ID within it, then the service is converting the ID to `String`. After that, `ClientService` is sending `DTO` to the repository that will search for it and return an implementation of the `Client` interface.

Now let's take a look in the converter:

{% highlight java %}
    public class ClientConverter {
        private static final String SEPARATOR = "-";
        private static final String CLIENT = "CLIENT:";

        public String convertClientIdentity(String clientIdentity) {
            if (clientIdentity.contains(CLIENT)) {
                return clientIdentity.replace(CLIENT, "");
            }
            if (clientIdentity.contains(SEPARATOR)) {
                return clientIdentity.replace("-", "");
            }
            return clientIdentity;
        }
    }
{% endhighlight %}

In our case it’s pretty simple, as our formats are easy to reason about it. But don't be fooled: this class can get messy quickly, especially if in the future we add some validations or need to do a HTTP request for some kind of decryption.

Anyway, here are some topics that illustrate the major problems that I see using this approach:
 - The class `ClientConverter` doesn't mean anything to the business and we lose the semantic value that would make our understanding easier.
 - This class will eventually grow, making every new non-standard client format more complex and hard to test.
 - This class has no abstraction (needs an interface).
 - This class has too many responsibilities, as it needs to check and convert 3 types of formats at once.

One thing I always encourage to do is thinking about the meaning of your abstractions and relating them to your business, this will help you design meaningful interfaces. If you have already read about DDD, you would know what I'm talking about.

To make everything clear, let's start with an example. First, we need to find the meaning; in our case, it’s obviously a client ID. That being said, it's clear how we should create the first abstraction:

{% highlight java %}
    public interface ClientIdentity {
        String value();
    }
{% endhighlight %}

This interface identifies the contract that represents a client ID to the app, so we can change the services to meet this contract’s requirements

{% highlight java %}
    public interface Clients {
        Client client(ClientIdentity identity);
    }
{% endhighlight %}

This interface will accept any implementation from `ClientIdentity` and return a `Client`. Because of this interface, we can create implementations that will handle specific formats and that's exactly what we need. For example, let's assume that the first format should be the standard format `12345612341234123456`, so we could come up with a simple class that just return a `String`:

{% highlight java %}
    public class FizzClientClientIdentity implements ClientIdentity {
        private final String identity;

        public FizzClientIdentity(String identity) {
            this.identity = identity;
        }

        public String value() {
            return identity;
        }
    }
{% endhighlight %}

This class handles the default format by doing nothing and returning it.

Now, for the second pattern, we need to remove some characters before returning `CLIENT:12345612341234123456`. Here is the implementation for it:

{% highlight java %}
    public class BuzzClientIdentity implements ClientIdentity {
        private static final String CLIENT = "CLIENT:";
        private final String identity;

        public BuzzIdentity(String identity) {
            this.identity = identity;
        }

        public String value() {
            return identity.replace(CLIENT, "");
        }
    }
{% endhighlight %}

The same happens for the third and last pattern, `123456-1234-1234123-456`:

{% highlight java %}
    public class DogClientIdentity implements ClientIdentity {
        private static final String SEPARATOR = "-";
        private final String identity;

        public DogClientIdentity(String identity) {
            this.identity = identity;
        }

        public String value() {
            return identity.replace(SEPARATOR, "");
        }

    }
{% endhighlight %}

Now, we have 3 classes that represent different patterns using the same contract `ClientIdentity`. `Clients` doesn't know what implementation is being passed and it doesn’t matter either, as it will just follow the contract and require the value to search for the client.
This adds flexibility and we can handle any number of patterns we want to, while keeping the classes short and cohesive. They also are easy to test and understand it as they mean something to the business.

To summarize, we need to pay more attention to the business and how we translate that into the code, thinking more carefully about our abstractions and using all the power that OOP is offering, avoiding procedural code and classes that will grow indefinitely like our converter above.
After years of using the converter path, I can't say it's easy to think this way, but it’s surely getting easier over time and if we want to deliver better software we need to keep learning and paying attention to the details.
