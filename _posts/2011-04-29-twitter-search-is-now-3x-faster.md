---
title: '[转]Twitter Search is Now 3x Faster'
author: jolestar
layout: post
permalink:  /twitter-search-is-now-3x-faster/
sina_t:
  - 'true'
tags:
  - 全部
  - java
  - netty
  - twitter
---
# 

原始地址[Twitter Search is Now 3x Faster](http://engineering.twitter.com/2011/04/twitter-search-is-now-3x-faster_1656.html)) 由于众所周知的原因，原始地址不能正常访问，所以转帖到这儿。

<!--more-->

### Twitter Search is Now 3x Faster

In the spring of 2010, the search team at Twitter started to rewrite our search engine in order to serve our ever-growing traffic, improve the end-user latency and availability of our service, and enable rapid development of new search features. As part of the effort, we launched a new [real-time search engine][1], changing our back-end from MySQL to a real-time version of [Lucene][2]. Last week, we launched a replacement for our Ruby-on-Rails front-end: a Java server we call Blender. We are pleased to announce that this change has produced a 3x drop in search latencies and will enable us to rapidly iterate on search features in the coming months.

## PERFORMANCE GAINS

Twitter search is one of the most heavily-trafficked search engines in the world, serving over one billion queries per day. The week before we deployed Blender, the [#tsunami][3] in Japan contributed to a significant increase in query load and a related spike in search latencies. Following the launch of Blender, our 95th percentile latencies were reduced by 3x from 800ms to 250ms and CPU load on our front-end servers was cut in half. We now have the capacity to serve 10x the number of requests per machine. This means we can support the same number of requests with fewer servers, reducing our front-end service costs.

[![][5]][5]

[][5]*95th Percentile Search API Latencies Before and After Blender Launch*

## TWITTER’S IMPROVED SEARCH ARCHITECTURE

In order to understand the performance gains, you must first understand the inefficiencies of our former Ruby-on-Rails front-end servers. The front ends ran a fixed number of single-threaded rails worker processes, each of which did the following:

 

*   parsed queries
*   queried index servers synchronously
*   aggregated and rendered results

 

We have long known that the model of synchronous request processing uses our CPUs inefficiently. Over time, we had also accrued significant technical debt in our Ruby code base, making it hard to add features and improve the reliability of our search engine. Blender addresses these issues by:

 

1.  Creating a fully asynchronous aggregation service. No thread waits on network I/O to complete.
2.  Aggregating results from back-end services, for example, the real-time, top tweet, and geo indices.
3.  Elegantly dealing with dependencies between services. Workflows automatically handle transitive dependencies between back-end services.

 

The following diagram shows the architecture of Twitter’s search engine. Queries from the website, API, or internal clients at Twitter are issued to Blender via a hardware load balancer. Blender parses the query and then issues it to back-end services, using workflows to handle dependencies between the services. Finally, results from the services are merged and rendered in the appropriate language for the client.

[![][7]][7]

[][7]*Twitter Search Architecture with Blender*

## BLENDER OVERVIEW

Blender is a Thrift and HTTP service built on [Netty][8], a highly-scalable NIO client server library written in Java that enables the development of a variety of protocol servers and clients quickly and easily. We chose Netty over some of its other competitors, like Mina and Jetty, because it has a cleaner API, better documentation and, more importantly, because several other projects at Twitter are using this framework. To make Netty work with Thrift, we wrote a simple Thrift codec that decodes the incoming Thrift request from Netty’s channel buffer, when it is read from the socket and encodes the outgoing Thrift response, when it is written to the socket.

Netty defines a key abstraction, called a Channel, to encapsulate a connection to a network socket that provides an interface to do a set of I/O operations like read, write, connect, and bind. All channel I/O operations are asynchronous in nature. This means any I/O call returns immediately with a ChannelFuture instance that notifies whether the requested I/O operations succeed, fail, or are canceled.

When a Netty server accepts a new connection, it creates a new channel pipeline to process it. A channel pipeline is nothing but a sequence of channel handlers that implements the business logic needed to process the request. In the next section, we show how Blender maps these pipelines to query processing workflows.

## WORKFLOW FRAMEWORK

In Blender, a workflow is a set of back-end services with dependencies between them, which must be processed to serve an incoming request. Blender automatically resolves dependencies between services, for example, if service A depends on service B, A is queried first and its results are passed to B. It is convenient to represent workflows as [directed acyclic graphs][9] (see below).

 

[![][11]][11]

[][11]*Sample Blender Workflow with 6 Back-end Services*

In the sample workflow above, we have 6 services {s1, s2, s3, s4, s5, s6} with dependencies between them. The directed edge from s3 to s1 means that s3 must be called before calling s1 because s1 needs the results from s3. Given such a workflow, the Blender framework performs a [topological sort][12] on the DAG to determine the total ordering of services, which is the order in which they must be called. The execution order of the above workflow would be {(s3, s4), (s1, s5, s6), (s2)}. This means s3 and s4 can be called in parallel in the first batch, and once their responses are returned, s1, s5, and s6 can be called in parallel in the next batch, before finally calling s2.

Once Blender determines the execution order of a workflow, it is mapped to a Netty pipeline. This pipeline is a sequence of handlers that the request needs to pass through for processing.

## MULTIPLEXING INCOMING REQUESTS

Because workflows are mapped to Netty pipelines in Blender, we needed to route incoming client requests to the appropriate pipeline. For this, we built a proxy layer that multiplexes and routes client requests to pipelines as follows:

 

*   When a remote Thrift client opens a persistent connection to Blender, the proxy layer creates a map of local clients, one for each of the local workflow servers. Note that all local workflow servers are running inside Blender’s JVM process and are instantiated when the Blender process starts.
*   When the request arrives at the socket, the proxy layer reads it, figures out which workflow is requested, and routes it to the appropriate workflow server.
*   Similarly, when the response arrives from the local workflow server, the proxy reads it and writes the response back to the remote client.

 

We made use of Netty’s event-driven model to accomplish all the above tasks asynchronously so that no thread waits on I/O.

## DISPATCHING BACK-END REQUESTS

Once the query arrives at a workflow pipeline, it passes through the sequence of service handlers as defined by the workflow. Each service handler constructs the appropriate back-end request for that query and issues it to the remote server. For example, the real-time service handler constructs a realtime search request and issues it to one or more realtime index servers asynchronously. We are using the [twitter commons][13] library (recently open-sourced!) to provide connection-pool management, load-balancing, and dead host detection.

The I/O thread that is processing the query is freed when all the back-end requests have been dispatched. A timer thread checks every few milliseconds to see if any of the back-end responses have returned from remote servers and sets a flag indicating if the request succeeded, timed out, or failed. We maintain one object over the lifetime of the search query to manage this type of data.

Successful responses are aggregated and passed to the next batch of service handlers in the workflow pipeline. When all responses from the first batch have arrived, the second batch of asynchronous requests are made. This process is repeated until we have completed the workflow or the workflow’s timeout has elapsed.

As you can see, throughout the execution of a workflow, no thread busy-waits on I/O. This allows us to efficiently use the CPU on our Blender machines and handle a large number of concurrent requests. We also save on latency as we can execute most requests to back-end services in parallel.

## BLENDER DEPLOYMENT AND FUTURE WORK

To ensure a high quality of service while introducing Blender into our system, we are using the old Ruby on Rails front-end servers as proxies for routing thrift requests to our Blender cluster. Using the old front-end servers as proxies allows us to provide a consistent user experience while making significant changes to the underlying technology. In the next phase of our deploy, we will eliminate Ruby on Rails entirely from the search stack, connecting users directly to Blender and potentially reducing latencies even further.

—@twittersearch

## ACKNOWLEDGEMENTS

The following Twitter engineers worked on Blender: Abhi Khune, Aneesh Sharma, Brian Larson, Frost Li, Gilad Mishne, Krishna Gade, Michael Busch, Mike Hayes, Patrick Lok, Raghavendra Prabhu, Sam Luckenbill, Tian Wang, Yi Zhuang, Zhenghua Li.

 

 [1]: http://engineering.twitter.com/2010/10/twitters-new-search-architecture.html
 [2]: http://lucene.apache.org/java/docs/index.html
 [3]: http://twitter.com/#!/search/#tsunami
 []: http://jolestar.com/images/netty/Blender_Tsunami.jpg
 [5]: http://jolestar.com/images/netty/Blender_Tsunami.jpg
 []: http://jolestar.com/images/netty/Blender_workflow.jpg
 [7]: http://jolestar.com/images/netty/Blender_workflow.jpg
 [8]: http://www.jboss.org/netty
 [9]: http://en.wikipedia.org/wiki/Directed_acyclic_graph
 []: http://jolestar.com/images/netty/Blender_S1.jpg
 [11]: http://jolestar.com/images/netty/Blender_S1.jpg
 [12]: http://en.wikipedia.org/wiki/Topological_sorting
 [13]: https://github.com/twitter/commons
