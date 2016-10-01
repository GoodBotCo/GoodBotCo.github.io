---
title: How Stateful Web Applications work
---

With Rails 5 out, a lot of programmers would love to explore the forever missing Action Cable and other stateful features with WebSockets. We thought it would be worth highlighting some points to consider when working on such features.

So what is the difference between HTTP and WebSockets at a very basic level? With HTTP, every request contains all information required, like the id of the `current_user` stored in a cookie. So from that perspective, it really doesn't matter which operating system or server is going to fulfill this request. However, with WebSockets, requests are not isolated, instead, it is more of a long-running conversation. Clients connect to a server, and until the connection is active, both the client and the server can exchange requests/responses over the WebSocket.

# Server Loads

In comparison to a stateless application, stateful applications can really take a toll on the server. Consider, you have a blogging platform (a rather famous one), and you render about 100 articles per second. This way your server takes a uniform load of about 100 connections per second. Enter WebSockets. Let's say, you want a way for your readers to view Live Comments, and say the average read time of your articles is 2 minutes (120 seconds). Your server would now need to handle 12,000 connections per second. The math is simple - (100 articles * 120s per article).

So it is important to consider these caveats before making your regular web application stateful. It is also important that your infrastructure is able to handle these requests concurrently. From your web proxy to your web server, you must be able to hold long running connections at the same time. And, since every single connection costs memory, you want a single web server to hold as many connections as possible.

To hold as many open connections as possible, it is important that your application handles machine resources (IO and CPU) as efficiently as possible. Most of the programming languages provide threads, which would not block IO (and provides CPU-based concurrency), but not all the programming languages can handle multi-core efficiently. So to hold 12,000 connections on a single machine, efficiently, it is important that the language you use is able to handle such load, and also not block IO. This should reduce a lot of infrastructure cost, and also improve user experience

# CPU Blocking

With stateful apps, multiple clients would all-time connected at the same time, sending events to the server, or receiving events from the server. It is important that your runtime is efficiently able to multiplex these always on connections, otherwise, latency can considerably increase, resulting in a pretty slow application.

To share an example of such a case. Say you have around 1000 running connections from multiple clients, each connection taking an average of 5ms of CPU time. So by the time your 1000th event gets access to your CPU, that client has already waited almost 5 seconds. Oh, crap! However, in a stateless application it is easier to solve this problem by load balancing, but with WebSockets, the machine you are connected to has to handle all the load.

# Comparing our primary weapons of choice

Two of our favorite web frameworks at the moment are Ruby on Rails and Phoenix. Let's compare how the underlying programming languages for both the frameworks would typically handle such workflows.

So Rails first (as it's still our favorite). We move all the CPU intensive tasks or tasks that do not require the request/response cycle to be blocked, to background jobs. So a typical flow of receiving an event, and broadcasting it to clients would go through the following steps:

- Client sends an event to the Channel
- Channel enqueues a job for the same
- The background jobs library (Sidekiq/Resque) will run the respective worker
- Worker passes the event to the pubsub adapter
- Pubsub adapter pushes the event to the broadcast server
- Broadcast server broadcasts the event to all clients

Phoenix, which is based on Elixir, let's see how it handles this workflow. Because Phoenix runs on the Erlang virtual machine, it has support for multicore systems.

- Client sends an event to the Channel
- Channel pushes the event to the pubsub adapter
- Pubsub adapter sends the broadcast to all the clients

The thing is Phoenix doesn't really require external PubSub adapters. Redis, PostgreSQL or even services like Pusher for that matter, because Phoenix channels run on the Erlang VM they can efficiently use all of your machine cores. So the broadcast that was sent to a particular machine, the same machine can broadcast the events to all clients, without having to interact with external services or other systems.

# TL;DR

With stateful applications, it is important to efficiently use multicore concurrency, which would help build a simpler application and a better experience for the user due to less latency.

If a programming language is not able to leverage that, developers would need to find workarounds for these limitations. Let's say when using Socket.IO for Node.js, developers would have to avoid long computations from taking CPU time so that they don't block Node.js' event loop.

This is why we feel that Phoenix would be a great choice for applications that require being stateful. The community support for the framework is as good as it was for Rails, back in the days when it was still young.

Hope this article helps you decide on which framework to choose, when you build your next stateful application. Happy Coding!
