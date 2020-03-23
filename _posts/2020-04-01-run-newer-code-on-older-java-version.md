---
layout: post
title: How to run newer code on older Java version?
---

If you ever had a situation when you were forced to stick with specific Java version due to dependant library version or (ekhem) licensing restriction, you can now breathe a sigh of relief: due to fantastic JVM community it is possible to avoid this issue! I will show you how to achieve this in simple, ready to use, project.

<!--excerpt-->

## Jabel

The project I mentioned above is called Jabel and the best part of it is insight it gives into JVM development. As it is stated in description:

{% highlight text %}
The JVM has evolved a lot for the past years. However, most language features that were added are simply a syntatic sugar. They do not require new bytecode, hence can be compiled to the Java 8.
{% endhighlight %}

Nice, right? Oracle changed whole release cycle, new features are being added continuously but the bytecode remains the same. Ok, but what it is?

## Byteode

One of the reasons why Java became so popular is its portability: your code should be easily run on every platform for which JVM exists. It is possible, because code is being translated into bytecode, which is similar representation to assembler for languages like C. It is set of basic instructions which are sophisticated enough to support wide range of actions. 

To follow the journey let's elaborate a bit on lifecycle of Java program. [Java compiler](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/javac.html){:target="_blank"} transforms code into above mentioned bytecode, which also means that Java files are now represented as class file. When JVM starts, it interprets compiled instructions and program can be run. But is it the end?

## Beyond bytecode

Not really. Program can be run, but Java engineers are working on making it faster but keeping portability. One of approaches taken is to compile the program again during execution, but into machine code this time. Specific machine code (for Linux or Windows for instance) will always be faster than general one but there is a cost of compiling again. This is why JIT (Just-In-Time compiler) has defined strategies when to do it and when don't. More frequent pieces of code are much more likely to being compiled again.

What JIT can do to make code faster? It can cache variables values, eliminate not used or duplicated code, optimize loops execution [among others](https://blogs.oracle.com/vaibhav/a-fast-overview-of-just-in-timejit-compiler){:target="_blank"}.

Side note - this is why you should always 'warm' your code before making test performance testing, especially when being interested in smaller execution times. Otherwise your code can give different results at the start of test suite, comparing to the end, for the same operations!

If you are into JVM internals you should look into my post about [Annotation Processing API in Java](http://itblues.pl/2016/06/28/what-is-project-lombok-and-why-you-should-use-it/){:target="_blank"}
