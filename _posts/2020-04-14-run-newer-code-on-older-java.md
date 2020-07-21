---
layout: post
title: How to run newer code on older Java?
---

If you ever had a situation when you were forced to stick with specific Java version due to dependant library version or (ekhem) licensing restriction, you can now breathe a sigh of relief: due to fantastic JVM community it is possible to avoid this issue! I will show you how to achieve this in simple, ready to use, project.

<!--excerpt-->

## Jabel

The solution I mentioned above is called Jabel and the best part of it is insight it gives into JVM development. As it is stated in description:


> The JVM has evolved a lot for the past years. However, most language features that were added are simply a syntatic sugar. They do not require new bytecode, hence can be compiled to the Java 8.

Nice, right? Oracle changed whole release cycle, new features are being added continuously but the bytecode remains the same. Ok, but what bytecode is?

## Byteode

One of the reasons Java became so popular is its portability: your code should be easily run on every platform for which JVM exists. It is possible, because code is being translated into bytecode, which is similar representation to assembler for languages like C. It is set of basic instructions which are sophisticated enough to support wide range of actions. 

To follow the journey let's elaborate a bit on lifecycle of Java program. [Java compiler](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/javac.html){:target="_blank"} transforms code into mentioned bytecode, which also means that Java files are now represented as class file. When JVM starts, it interprets compiled instructions and program can be run. But is it the end?

## Beyond bytecode

Not really. Program can be run, but Java engineers are working on making it faster while keeping portability. One of approaches taken is to compile the program again during execution, but into machine code this time. Specific machine code (for instance for Linux or Windows) will always be faster than general one, but there is a cost of compiling again. This is why JIT (Just-In-Time compiler) has defined strategies when to do it and when don't. More frequent pieces of code are much more likely to being compiled again.

What JIT can do to make code faster? It can cache variables values, eliminate not used or duplicated code, optimize loops execution [among others](https://blogs.oracle.com/vaibhav/a-fast-overview-of-just-in-timejit-compiler){:target="_blank"}.

Side note - this is why you should always 'warm' your code before making test performance testing, especially when being interested in smaller execution times. Otherwise your code can give different results at the start of test suite, comparing to the end, for the same operations!

If you are into JVM internals you should look into my post about [Annotation Processing API in Java](http://mikolajkania.com/2016/06/28/what-is-project-lombok-and-why-you-should-use-it/){:target="_blank"}. Jabel is using similar path to provide its features.

## Using Jabel

Going back to our main problem - let's say that we have code written in Java 13 and we are obligated to run it on Java 10. Consider following method:

```java
public static void main(String[] args) throws InterruptedException, ClassNotFoundException {
    // var is available since Java 10
    var c = Repository.lookupClass("pl.itblues.JavaVersions");

    String msg = "   Compiled Java version= " + c.getMajor();
    System.out.println(msg);
    System.out.println("Runtime Java version= " + System.getProperty("java.version"));

    // text blocks can be used since Java 13
    System.out.println(
                    """
                    From Gdańsk
                    with love!
                    """);


    Thread.sleep(10_000);
```

Running this code on Java 10 must end with an error, as *text blocks* are available only since Java 13: 

```console
%JAVA10%\bin\java -jar java-versions-1.0-SNAPSHOT-jar-with-dependencies.jar
Error: LinkageError occurred while loading main class pl.itblues.JavaVersions
        java.lang.UnsupportedClassVersionError: pl/itblues/JavaVersions has been compiled by a more recent version of the Java Runtime (class file version 57.65535), this version of the Java Runtime only recognizes class file versions up to 54.0
```


But if you use it with Jabel the output will be much different:

```console
%JAVA10%\bin\java -jar java-versions-1.0-SNAPSHOT-jar-with-dependencies.jar
   Compiled Java version= 54
Runtime Java version= 10.0.2
From Gdańsk
with love!
```

Nice! But if you are following closely you can already ask yourself a question: why *compiled Java version* is 54, not 13? And from where this number came from in the first place? The reason for that is [convention](https://en.wikipedia.org/wiki/Java_class_file#General_layout){:target="_blank"} taken by Java creators, in which Java 10 has assigned number 54 and Java 13 - 57. The information is stored in class file that is created during compilation.

Going back to the example: we compiled the code using Java 13 (compiled version 57) and JVM sees it as compiled with Java 10 (compiled version 54). It shouldn't work because of text blocks but it is, so we fulfilled a promise. Of course running the code with Java 13 will also work:

```console
java -jar java-versions-1.0-SNAPSHOT-jar-with-dependencies.jar
   Compiled Java version= 54
Runtime Java version= 13.0.2
From Gdańsk
with love!
```

Cool, right? Jabel tricks compiler and removes checks that would prevent code to be run on older Java. Using it is as simple as adding annotation processor entry to your build process:      

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>        
        <release>10</release>
        <annotationProcessorPaths>
            <annotationProcessorPath>
                <groupId>com.github.bsideup.jabel</groupId>
                <artifactId>jabel-javac-plugin</artifactId>
                <version>0.2.0</version>
            </annotationProcessorPath>
        </annotationProcessorPaths>
        <annotationProcessors>
            <annotationProcessor>com.github.bsideup.jabel.JabelJavacProcessor</annotationProcessor>
        </annotationProcessors>
    </configuration>
</plugin>
```  

## Not a perfect solution

Despite being a very handy tool Jabel is not perfect time (or version) traveller for Java. You will reach its limits while using invocations or methods that are physically absent in older versions. The example of that might be String class method *strip*, which is present since release 11:   

```java
String msg = "   Compiled Java version= " + c.getMajor();
// strip method is available since Java 11, but can't be used even with Jabel
System.out.println(msg.strip());
```

Trying to compile this piece of code will end with error:


```console
Error:(16,31) java: cannot find symbol
```

As stated in official project issue tracker, making it cover such issues would be a very different aim for a project. Discussion around it can be found [here](https://github.com/bsideup/jabel/issues/3){:target="_blank"}.

## Summary

In this post I provided suggestion how to deal with situation when newer Java code has to be run on older JVM. I also dived deeper into what is bytecode and how process of creating a executable Java code looks like. If you want to try [Jabel](https://github.com/bsideup/jabel){:target="_blank"}  yourself head into its repo or use my [project](https://github.com/mikolajkania/run-newer-code-on-older-java-version){:target="_blank"}. 