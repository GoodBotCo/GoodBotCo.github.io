---
title: An Introduction to the Actor Model
---

Our CPUs are not getting any faster, but what we are getting instead are multicore CPUs. In one of our [previous blogposts](http://blog.goodbot.co/how-stateful-web-applications-work/), we mentioned that it is very important for a modern programming language to take advantage of the hardware available to it, and one way to do that is run code concurrently. For a few decades now programmers have worked with `threads`, and even though they have been great, programmers know that they are not the way forward. This is where the Actor Model comes in, it is an alternative to using `threads`. Let's discuss how it actually works.

The Actor model is a conception model. It is designed to deal with concurrent computation, in which "actors" are the universal primitives. In simple English, the model asserts that *everything is an actor*. This philosophy is similar to the *everything is an object* assertion used by Object Oriented Languages. One of the most famous languages that use this model is Erlang, and now it's child prodigy Elixir.

### What are Actors?

As described above, and Actor is the most primitive unit of computation. It's the unit that receives messages, and based on that does some kind of computation.

This idea is actually not very different from that of Objected Oriented Programming (OOP). In OOP, an Object receives a message, or you can say a method call, based on which it performs certain actions. The main difference to note here is that Actors are completely isolated from each other and that they do not share memory. Also, actors maintain a private state, that another actor can never change. To communicate with each other actors use messages, which are delivered to a unique address assigned to each actor.

### Mailboxes

The messages are sent asynchronously to actors. The place where these messages are stored is called a Mailbox. While an actor processes a message, all the other messages are stored in its mailbox.

One of the important thing to understand is that, although actors run concurrently, but the messages passed to actors are processed sequentially. For example, if you send 3 messages to an actor, it will execute those messages one at a time. So to concurrently execute these messages, you would have to create 3 actors, and then send one message to each one of them.

### Actors in Action

When you message an actor, it can do a few things:

- Send a finite number messages to other actors
- Create a finite number of new actors
- Designate the behavior to be used for the next message it receives

The first two points are pretty straightforward, the last one is a bit interesting to understand. Like we discussed above, the actors always maintain a private state. So what designating the behavior to be used for the next message means is how its state will look to the next message the actor receives, or in other words how the actor will mutate its state.

To put this in an example. Let's assume you have an actor that calculates the sum of all numbers. Its initial state would be the number 0, let's say you pass it a message with the number 7. The actor would now designate that the for the next message it receives, it's state would be 7.

### Crash Handling

One of the philosophy that Erlang works on is *let it crash*. This works on the basic idea that programmers do not need to write defensive code, wherein they spend time thinking about all the possible ways the code can break, and then handle that. So here is where Erlang wins with the *letting it crashe* idea. What Erlang does is it suggests that this critical code be rather supervised by someone whose sole purpose is to know that what should be done when something crashes, all this is made possible by the actor model.

In Erlang, every code runs inside a `process`, which Erlang calls its actors. And as we learned above, this `process` is completely isolated, and its state would not change the state of any other `process`. There is another `process` called the `supervisor`. The `supervisor` is notified when a supervised `process` crashes.

This setup makes it possible to create a self-healing system. Say when an actor reaches an exceptional state, and crashes - a supervisor is notified and it can perform an action based on that message. One of the most common things to do is to reset that actor its initial state.

### Easy Distribution

One of the most interesting aspects of the Actor Model is distribution. It actually does not matter if the actor that's sending a message, and the one that is receiving the message reside on the same machine.

As the actor is nothing but a unit of code with a mailbox and an internal state, and it responds to messages - how does it matter if it is on the same machine, or on a remote one? As long as the two actors are able to communicate. This setup makes allows programmers to leverage distributed computers, and create a really fault tolerant system wherein even one system fails there is another system to recover from.

--

This philosophy is the basis of Erlang and Elixir. Alos, Ruby's [Celluloid ](https://github.com/celluloid/celluloid) framework is based on this model.

I plan to follow this post up with some code examples of Actors in action with Elixir.
