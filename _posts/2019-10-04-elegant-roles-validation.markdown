---
layout: post
title:  "Elegant roles validation"
date:   2019-10-04 22:00:00
tags: oop
comments: true
shareable: true
author: randommancer
preview: When writing applications, eventually you need to create a mechanism to restrict access to part of the system or even to some specific functionality. It's a pretty common issue and I’ve done that many times before, but I never considered doing it in an elegant way.

image: https://randommancer.com/images/caffeinated_concert_tickets.png
--- 

When writing applications, eventually you need to create a mechanism to restrict access to part of the system or even to some specific functionality. It's a pretty common issue and I’ve done that many times before, but I never considered doing it in an elegant way.

Recently, I came across this problem in one of my pet projects. I had to restrict the access to specific functionalities based on ownership, preventing common users from executing some tasks. Let me give you an example.

<figure class="articleimg">
    <img src="{{page.image}}" alt="Prank callers">
    <figcaption>
    Regular Show - Caffeinated Concert Tickets, by J.G. Quintel
    </figcaption>
</figure>

In my scenario, I can create virtual rooms and only the room’s owner is allowed to rename or close it, and all the common users are allowed to join the room anytime they want.

The interface below translates the business behavior. Given a specific room, you can rename, join or close it and, depending on your authorization level, you might succeed or not.

{% highlight java %}
interface Room {

   fun rename(name: String)
   
   fun close()

   fun join()

}
{% endhighlight %}

In the code below, I receive an authenticated user via DI along with the database connection and the room identifier (ID). Each method that requires owner privileges is checked before being executed and throws an exception if the role is not compatible with the requested feature.

{% highlight java %}
class PersistedRoom(
    private val connection: Connection,
    private val user: User,
    private val id: String
) : Room {

    override fun rename(name: String) {
        if(isOwner(user)){
            //Rename the room
        }
        throw RuntimeException("You are not the owner")
    }

    override fun join() {
        //Join the room
    }

    override fun close() {
        if(isOwner(user)){
            //Close the room
        }
        throw RuntimeException("You are not the owner")
    }

    private fun isOwner(user: User):Boolean {
        //Check if its the owner
    }

}
{% endhighlight %}

I don't like the owner checking before executing the methods, as it seems like a bad approach and a possible root of troubles regarding role validation. Let's say now we need to have a moderator because the owner is too busy taking care of multiple rooms and doesn't have much time to rename or close empty ones. We will need to add one more verification besides the owner’s one, like a `isModerator` method. This will decrease our class’s cohesion as, besides worrying about room-related features, it needs to be aware of the definition of the roles as well.

While trying to mitigate this problem, I changed the implementation of `PersistedRoom`. This class should only focus on the features that a common user can access and restrict everything else. In our case, we just need to implement the join method. By doing so, it will provide simpler code to reason about and a more cohesive class since we removed all the validation regarding roles.

{% highlight java %}
class PersistedRoom(
    private val connection: Connection,
    private val user: User,
    private val id: String
) : Room {

    override fun rename(name: String) {
        throw RuntimeException("You are not the owner")
    }

    override fun join() {
        //Join the room
    }

    override fun close() {
        throw RuntimeException("You are not the owner")
    }

}
{% endhighlight %}

Now we can use the same approach and focus only on owner features. That's why I created the new class implementation called `OwnedPersistedRoom`. Removing the role validation also increases the class cohesion, as you can see below:

{% highlight java %}
class OwnedPersistedRoom(
    private val origin: Room,
    private val connection: Connection,
    private val id: String
) : Room {

    override fun rename(name: String) {
        //Rename the room
    }

    override fun join() {
        throw RuntimeException("You are the owner of the room")
    }

    override fun close() {
        //Close the room
    }

}
{% endhighlight %}

One thing to notice is that `OwnedPersistedRoom` is a decorator, so we need to create an instance of `PersistedRoom`. I use this approach for two main reasons:
- First, it seems more semantically correct because every user can have a common room and this room could be owned by them.
- Second, because I believe in the future, features are very likely to have the same implementation being used by both classes and I don't want duplicated code.

The final step is to address the role validation code and make sure that we are creating the right implementation. In my opinion, the perfect place to insert that kind of code is the room search class since every room is originated there.

{% highlight java %}
class PersistedRooms(
    private val connection: Connection,
    private val user: User
) : Rooms {
    override fun room(id: String): Room {
        //Search for a specific room
        val databaseRow = //Search

        val room = PersistedRoom(
            connection,
            user,
            id
        )
        if (user.id() == databaseRow.getAs("owner")) {
            return OwnerPersistedRoom(
                room,
                connection,
                id
            )
        }
        return room
    }
{% endhighlight %}

In the class above, I implemented the roles validation and, with that information, I can create the right implementation for the caller. I like the separation that both implementations provide us with and it would be easy to create new types if necessary and the search class seems like a good fit for that specific role validation.

Design is a complex process and we can't nail everything. This is just one possibility I thought that’s worth to be shared. We are respecting OOP principles in all classes. They are more cohesive since every class is just concerned about its own business and it seems easy to be modified, and extended in the long run. Let me know your thoughts and, if you have any ideas or questions, please share them.