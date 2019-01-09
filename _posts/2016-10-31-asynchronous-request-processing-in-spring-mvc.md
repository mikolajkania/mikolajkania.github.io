---
layout: post
title: Is asynchronous request processing worth your time?
tags:
- java
- asynchronous
- spring
- spring mvc
- jmeter
- performance
- jetty
---

Recently, going through the Spring MVC documentation, I found a feature I haven't previously used - asynchronous request processing. It is an addition of [Servlet 3 API](https://jcp.org/en/jsr/detail?id=315){:target="_blank"} and a part of Java EE since its sixth edition from 2009; [Spring started support it three years later](https://spring.io/blog/2012/05/07/spring-mvc-3-2-preview-introducing-servlet-3-async-support){:target="_blank"}. As it looks interesting (and as *async* is a popular word in developer's journey since at least early Web 2.0 days) I decided to go deeper into details of it. 

<!--excerpt-->

In the traditional - and still completely valid approach - an incoming request is handled by one thread from the beginning to the end of its lifecycle. It is a model which is dead-simple and effective due to performance of today's web applications servers, but have one big drawback: if there is a bunch of long-running requests, they can restict the availability of resources to other user threads. One can ask how it can be avoided, since server has to process everything and there is no magic in its code? 

Imagine a situation when thread has to wait for a external resource, like a database or REST api - it may do nothing but still is reserved! Or when developer knows that server would consume more smaller requests but is afraid of changing max threads number in they server due to possibility of oncoming heavy ones.


<h2>How it works?</h2>
In the Servlet 3.0 world request can be *left* by its initial container thread with AsyncContext implementation containing all necessary data required to resume execution. The leaving thread is exited but from the client perspective there is the same simple anticipation of a result - it will only be completed by another thread. 

Developer has to provide a *Callable* or *DeferredResult* as a return of MVC Controller and Spring will do the rest. Sounds simple? As always the devil is in the details, but this knowledge should be enough to bootstrap a exemplary project and test the difference between sync and async requests. For more detailed information about a flow, yet rather laconic, I recommend [Spring Docs.](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html#mvc-ann-async){:target="_blank"}


<h2>The test plan</h2>
As always when asynchrony appears, every developer wants to know what benefits and drawbacks they will face using it. Starting with a cake, I would use JMeter to test how server response times will change when asynchrony will be enabled and how to do that in Spring MVC. 

There will be a controller with majority of requests lasting 1 second (80%), leaving rest with 4 seconds. I will try to recreate existance of different requests life spans. 

A JMeter test plan would include warmup with 100 threads and 25 iterations, the proper test - 1000 threads started over half a minute in 10 iterations. The most important metric would be a throughput, requests per second. I will repeat tests multiple times and choose representative sample.

I will use Jetty 9 and Java 8 64 bit runtime. Jetty will be configured to have at most 100 threads. Server mode for compilation is the [only one supported in 64bit](http://www.oracle.com/technetwork/java/hotspotfaq-138619.html#compiler_types){:target="_blank"} Java so it would be obviously enabled.


<h2>The code</h2>
Although the necessary code is not difficult to write, I will provide a introductory Spring project to help bootstrap everything even faster. You can find it [here](https://github.com/mikolajkania/spring-mvc-startup){:target="_blank"}, it is a Spring MVC app without Spring Boot magic configuration. It will run on Jetty or Tomcat, I used the former.

First, you need to tell Spring to enable async support for given servlet. Unless you do that, the nasty exception appears at runtime. Add below line to AppInitializer:
{% highlight java %}
servlet.setAsyncSupported(true);
{% endhighlight %}

Secondly, a controller. 
{% highlight java %}
@GetMapping(value = "/sync")
public String sync() throws InterruptedException {
	sleep();
	return "ok";
}

@GetMapping(value = "/async")
public Callable<String> async() throws InterruptedException {
	return new Callable<String>() {
		@Override
		public String call() throws Exception {
			sleep();
			return "ok";
		}
	};
}

private void sleep() throws InterruptedException {
	int random = ThreadLocalRandom.current().nextInt(1, 11);
	if (random % 5 == 0) {
		Thread.sleep(4_000);
	} else {
		Thread.sleep(1_000);
	}
}
{% endhighlight %}
In Github repo I also included a *warm* method only to point out its importance in general testing - in such a simple use case it is only for fast detection of server problems.

Lastly, the code for Jetty threads from Jetty.xml:
{% highlight xml %}
<Arg name="threadpool"><New id="threadpool" class="org.eclipse.jetty.util.thread.QueuedThreadPool"/></Arg>   
<Get name="ThreadPool">
  <Set name="minThreads" type="int">100</Set>
  <Set name="maxThreads" type="int">100</Set>
  <Set name="idleTimeout" type="int">5000</Set>
  <Set name="detailedDump">false</Set>
</Get>
{% endhighlight %}


<h2>The results</h2>
Lets don't conceal it any longer - an asynchronous code is faster. When synchronous method has approximately 58 operations per second, asynchronous one - 173! Also the average times of requests are interesting, in former 13 second, in later - 8 times faster (1.6s). It is an obvious victory of asynchronous processing. Sample results from JMeter are presented below, I've choosen them from multiple tests.

![placeholder](https://raw.githubusercontent.com/mikolajkania/mikolajkania.github.io/master/_images/servlet3-sync.png "Synchronous results")
![placeholder](https://raw.githubusercontent.com/mikolajkania/mikolajkania.github.io/master/_images/servlet3-async.png "Asynchronous results")

To have a bigger picture I tested how both approaches would work with all requests taking one second to proceed. This time the difference is smaller, 95 to 250 operations per second. I would like to emphasize correctness of the results in case of sync processing - 95 requests per second is really close to 100 threads I set in Jetty configuration and may be treated as a *sanity check*.

Interestingly, when I changed sleep time to 10ms the roles swapped or were nearly equal (for example 331,5 vs 331,3 operations per second for synchronous method)! This result leads to another topic.

<h2>Drawbacks?</h2>
As always, there are some. Asynchronous processing will show its power only when requests are longer. Probably sometimes overhead resulting from preparing AsyncContext implementation may even reduce the performance or improvement will be negligible.

I'd also point out that the code becomes less straightforward and harder to debug, errors should have another handling, which also adds layer of complexity.

<h2>Conclusion</h2>
Is asynchronous request processing worth your time? Yes, but it depends. There are cases when it can be a good answer to performance problems: as I showed above it improved results three times in terms of throughput compared to synchronous equivalent. Keep in mind drawbacks and familiarize yourself with this solution and maybe it will save you someday.
