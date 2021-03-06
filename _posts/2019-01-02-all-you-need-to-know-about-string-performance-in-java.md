---
layout: post
title: All you need to know about String memory performance in Java in 2019  
tags:  
- java
- performance
- string performance
- java performance
- memory
- performance guide
- java internals

---

Did you know that, according to Java implementators, [about 25% of memory](https://openjdk.java.net/jeps/192){:target="_blank"} consumed by large-scale applications are Strings? And what if I tell you that you can decrease this value with a single command?           

<!--excerpt-->

## How Strings are built in Java?

### Strings are immutable objects

The heart of String object is an array which stores its content. String is wrapping it and prevents from changing, offering multiple methods that returning new String as a result of computation.   

{% highlight java %}
/** Before Java 9 */
private final char value[];
/** Since Java 9 */
private final byte[] value;
{% endhighlight %}

Why are Strings immutable? Security and thread-safety are the good answers, but in terms of performance we will be mostly interested in objects caching, implemented via **String pool**. Technically, it is a memory space located now on a Java heap. When you create a new string variable with value already existing in the pool, JVM doesn't have to create another entry, and reference to existing object would be returned. It helps to reduce memory consumption of Java programs as Strings are very common objects in any application.  

Immutability and caching are also helpful when computing String's hash, as it can be saved after the first call to hashcode() method. It gives performance boost when using structures like HashMaps, where hashes are required to determine to which bucket object should be put. 

{% highlight java %}
public int hashCode() {
   int h = hash;
   if (h == 0 && value.length > 0) {
      hash = h = isLatin1() ? StringLatin1.hashCode(value)
                            : StringUTF16.hashCode(value);
   }
   return h;
}
{% endhighlight %}  

### Are Strings always returned from a pool?

No, they aren't, but the most convenient way of creating them - by string literal - works like that. Two strings literals from example below are not only equal in terms of value, but also share memory representation, pointing to one object in String pool. 

{% highlight java %}
String alice = "al" + "ice";
String bob = "alic" + "e";
System.out.println(alice.equals(bob));  // true
System.out.println(alice == bob);       // true
{% endhighlight %} 

There is another way of creating strings which involves calling its constructors. As a result, created objects will be separate entities with different memory representation on Java heap. If comes without saying that it is not advised way of dealing with strings in Java.

{% highlight java %}
String alice = new String("alice");
String bob = new String("alice");
System.out.println(alice.equals(bob));  // true
System.out.println(alice == bob);       // false
{% endhighlight %} 

You can manually place strings in their pool (or retrieve them if already exist) by calling String method intern().

{% highlight java %}
String alice = new String("alice");
String bob = new String("alice");
alice = alice.intern();
bob = bob.intern();
System.out.println(alice.equals(bob));  // true
System.out.println(alice == bob);       // true
{% endhighlight %} 

It is worth to mention that pool of literal values is not restricted to Strings. Other types in Java also supports this concept but in [a bit different way](https://stackoverflow.com/a/13098161){:target="_blank"}.

### So, should we put all strings into a pool?

Now a question may come to your mind: shouldn't you intern all the strings of a (hello) world if it such a efficient idea? As always - it depends. 

Everything has a cost and this is also a case of String pool. 

One thing is that calling intern method [has an execution cost](http://java-performance.info/string-intern-in-java-6-7-8/){:target="_blank"} and it would be unwise to do it unnecessarily. 

The second is that JVM has to keep hashed structure containing interned strings and search it. The longer the list, the slower it would be to found given objects due to hash collisions - multiple hash values can be put into one bucket of fixed-size hashtable that holds strings (for more on hashtable look [here, p. 196](https://amzn.to/2GaBOZl){:target="_blank"}, check collisions before job interview, i.e. [here](https://javarevisited.blogspot.com/2016/01/how-does-java-hashmap-or-linkedhahsmap-handles.html){:target="_blank"}). What's more, if you sorted the strings in your application by number of occurrences, you would see that the biggest advantage of string interning would be at the top of a list. At some point, the gain would be marginal or zero, but those entries still would be kept in a pool. It is a wise idea to put developers in charge and let them decide how big their pool should be - and exactly that can be done with option *-XX:StringTableSize=n*. The default size is 60013 since Java 7u40 and can be checked by using JVM option *-XX:+PrintStringTableStatistics*, for example:

{% highlight java %}
StringTable statistics:
Number of buckets       :     60013 =    480104 bytes, avg   8.000
Number of entries       :       921 =     22104 bytes, avg  24.000
Number of literals      :       921 =     62256 bytes, avg  67.596
Total footprint         :           =    564464 bytes
Average bucket size     :     0.015
Variance of bucket size :     0.015
Std. dev. of bucket size:     0.123
Maximum bucket size     :         2      
{% endhighlight %} 

The same program with previous default string table size has a very different values for buckets size: 

{% highlight java %}
StringTable statistics:
Number of buckets       :      1009 =      8072 bytes, avg   8.000
Number of entries       :       921 =     22104 bytes, avg  24.000
Number of literals      :       921 =     62256 bytes, avg  67.596
Total footprint         :           =     92432 bytes
Average bucket size     :     0.913
Variance of bucket size :     0.874
Std. dev. of bucket size:     0.935
Maximum bucket size     :         5
{% endhighlight %} 

Manual interning may not be a bad idea, though. It can be useful when you have many dictionaries of values that will be always heavily used in application. But I believe it is a way when your application is already pushed to the limits and narrow enough to supply those dictionaries. 

## String deduplication

### How it works?

There are situations when you know that duplicates are a real deal in your application and you are running out of memory because of them. Instead of manually interning everything you can use Java's [String deduplication](https://openjdk.java.net/jeps/192){:target="_blank"} feature of G1 garbage collector, which [became default](http://openjdk.java.net/jeps/248){:target="_blank"} with Java 9. Let implementators explain their motivations:

> Many large-scale Java applications are currently bottlenecked on memory. Measurements have shown that roughly 25% of the Java heap live data set in these types of applications is consumed by String objects. Further, roughly half of those String objects are duplicates. Having duplicate String objects on the heap is, essentially, just a waste of memory.

Technically, how is deduplication performed?

> When garbage collection is performed, live objects on the heap are visited. For each object we visit a check is applied to see if the object is a candidate for string deduplication. If the check indicates that this is a candidate then a reference to the object is inserted into a queue for later processing. A deduplication thread runs in the background and processes the queue.

It is worth to mention that not all strings will be candidates for deduplication. As [the JEP states](http://openjdk.java.net/jeps/192){:target="_blank"}, there is command-line option that can manage it (*StringDeduplicationAgeThreshold*) and its default value is 3 - candidates reaching this GC age will be taken into consideration. It may be important for those who want it to have immediate impact. On the other hand it can be a performance advantage - String interning has it cost and the same is with deduplication - not taking into account all objects can make process of deduplication less painful for garbage collector/CPU. But as in many situations in Java developers are in charge and can set it to other value.  

### How String Deduplication performs?

So how it performs? I've wrote a program that creates strings and adds them to list to prevent garbage collection. I also wanted strings to be not equal in terms of occurrences to make example less 'labolatory'.

{% highlight java %}
private static final List<String> KEEP_STRINGS = new ArrayList<>();
private static final Random random = new Random();
public static void main(String[] args) {
    for (int i = 1; i <= 100_000; i++) {
        KEEP_STRINGS.add("Test string " + random.nextInt(i));
    }
    System.out.println("Unique values=" + new HashSet<>(KEEP_STRINGS).size());
}
{% endhighlight %} 

With printing deduplication statistic enabled (*-XX:+UseG1GC -XX:+UseStringDeduplication -XX:+PrintStringDeduplicationStatistics*) I was able to check how string presence can drop. From 100k samples 50133 entries were unique and deduplication was able to cover 33476 strings from whole JVM process. Look nice but keep in mind that results from a real application would be very different.

Disadvantages? It is only directed to applications where repeating strings are a real problem, works only with G1 garbage collector which has one more constantly working process attached to itself (CPU usage) and statistics cannot be printed on newest Java version, OpenJDK 11.0.1, [due to error](https://bugs.openjdk.java.net/browse/JDK-8211821){:target="_blank"}. This is why I used JVM option *-XX:+UseG1GC* for Java 8, as before Java 9 G1 was not the default garbage collector. Keep also in mind that String deduplication was added in Java version 8u20.

## Compact Strings

### How it works?

Before Java 9 String's internal shape was rather simple: characters were stored in char array, two bytes for every of them. It is more tricky now, as JDK creators observed that in many situations one byte would be enough. Now, the additional *coder* in String is the identifier of the encoding used: Latin1 or UTF-16. 

Compact Strings are enabled by default. You can turn them off, but not entirely - inside the String there still will be byte array, but Java will not be using the shorter representation. The downside of this approach is that almost every method in String became less readable internally, with condition like in hashcode method at the top of the post.

### How Compact Strings performs?

I've decided to test this solution with program written for String deduplication. In the first test I've used enabled by default Compact Strings. In the second - I've disabled them with JVM option *-XX:-CompactStrings*. Garbage collector was called before every measurement. 

|                       | Compact Strings Enabled  |  Compact Strings Disabled | Compact Strings + Deduplication |
|-----------------------|:------------------------:|:-------------------------:| :------------------------------:|
| String objects        | 119 284                  | 119 208                   | 119 183
| String retained size  | 7 040 104                | 8 848 360                 | 5 175 704

It is easy to spot a difference: with similar number of string objects the retained size of compact strings version is much smaller. Fast recap: retained size is number of bytes that will be garbage collected together with objects for which calculations are performed. In this case - if all strings would be eligible for GC. What's interesting - combination of compact & deduplicated strings gives even better results in this example.  

One note - test was run on Java 10 as my profiler is still not able to work with newest Java out of the box. 

Does this solution has any disadvantages? Not many so far, and this is why it is enabled by default. If I had to stress something: reduced readability and maybe change in run-time performance, according to authors:

> Optimizing character storage for memory may well come with a trade-off in terms of run-time performance. We expect that this will be offset by reduced GC activity and that we will be able to maintain the throughput of typical server benchmarks.

## Other significant String performance factors

### Use StringBuilder for string concatenation   

As Strings are immutable its concatenation requires creating a new entry after every change, which is not fast enough when there is many such operations or String class methods are involved. Java has an answer for that problem in a form of [class StringBuilder](https://github.com/AdoptOpenJDK/openjdk-jdk11/blob/999dbd4192d0f819cb5224f26e9e7fa75ca6f289/src/java.base/share/classes/java/lang/AbstractStringBuilder.java){:target="_blank"}, not thread safe successor of StringBuffer.  

> Implements a modifiable string. At any point in time it contains some particular sequence of characters, but the length and content of the sequence can be changed through certain method calls.

It offers better performance during concatenation; exact comparison can be seen in [this post](http://www.corejavaguru.com/effective-java/items/51){:target="_blank"}. What's more, it helps avoid temporary Strings objects that would be a result of standard concatenation and offers additional methods to work on text.       

But should you always replace string concatenation with StringBuilder? In many situations it is the triumph of form over substance as you are loosing the readability of a code. What's more, compiler can do it for you under the hood when you are - for instance - explicitly adding few strings outside a loop. From [Java Language Specification](https://docs.oracle.com/javase/specs/jls/se11/html/jls-15.html#jls-15.18.1){:target="_blank"}:    

> 'To increase the performance of repeated string concatenation, a Java compiler may use the StringBuffer class or a similar technique to reduce the number of intermediate String objects that are created by evaluation of an expression.'     

The most important message from this section is that StringBuilder should be used always when many concatenations are performed, for example in a loop.

### Avoid creating too many strings 

It is very general rule and related to StringBuilder case, but I would like to illustrate it with small performance tip. When you are logging message, mostly on lower levels which can occur often, there is a temptation write something like that:

{% highlight java %}
log.debug("[Cached] User id=" + getUserId() + ", date=" + new Date());
{% endhighlight %} 

It will be fine in small to medium applications, but if you considered big ones and parts of code that are called all the time it may be an issue. Why? Because even is logging level would be higher (INFO, WARN etc.) the String would be created and saved in memory before evaluating that. There may be some computation cost, but I mostly think about memory here. The proper solution is:

{% highlight java %}
if (log.isDebugEnabled()) {
    log.debug("[Cached] User id=" + getUserId() + ", date=" + new Date());
}
{% endhighlight %}

It can be even prettier with lambdas:

{% highlight java %}
log.debug("Some long-running operation returned {}", () -> "[Cached] User id=" + getUserId() + ", date=" + new Date());
{% endhighlight %}

Now, there is no need to construct a String. It may not be the first thing to blame during performance research, but shows that you can easily write better code with minor changes.

Of course this post could be much longer and cover even more cases - but I wanted to extract those I see as most interesting and easy to adopt - even if the technology behind it is advanced. In the most cases, before you start troubling yourself with finesse code-level optimizations, you should use approaches that can provide immediate result - and buy you some time for better problem tackling. Don't get me wrong - the more you know, the better, always. But as [Donald Knuth said](https://en.wikiquote.org/wiki/Donald_Knuth){:target="_blank"}:

> We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil. Yet we should not pass up our opportunities in that critical 3%

Yes, I know that someone may call my logging example as a root of all evil ;-)

## Conclusion

Even if String performance does not look like a hot topic for performance engineer, it is better to know how they works and why. I hope this guide will be helpful for everyone, especially those who can't sleep because of the OutOfMemory exceptions.