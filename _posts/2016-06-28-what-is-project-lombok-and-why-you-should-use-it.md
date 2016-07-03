---
layout: post
title: What is Project Lombok and why you should use it?
tags:
- project lombok
- lombok
- java
- library
- clean code
- java internals
---

<h2>Java & boilerplate code</h2>
Java is well known for its necessity to write quite a lot of code to perform simple tasks: all this getter/setter methods [handled nicely by a competitors](https://msdn.microsoft.com/en-us/library/aa287786(v=vs.71).aspx), common [Problem Factories](https://i.imgur.com/y41pi4n.jpg), Calendar & Date or logging jumbo. As more languages with plain syntax arise, staying put with actual aproach seems to be a bit out-of-date. There are even some propositons to [add JavaScript's-like val folding](http://blog.joda.org/2016/03/var-and-val-in-java.html) to change things, but with Oracle's lacking investment it is hard to believe that any changes appear in a finite time. On the other hand Java ecosystem is full of decent libraries that can fill this gap; one of this libraries is Project Lombok.


<h2>Project Lombok</h2>
[The Project Lombok's](https://projectlombok.org/) main goal is to reduce boilerplate code in Java. It provides bunch of annotations making classes much more cleaner and is able to generate required code during compilation. But before we immerse ourselves into the project's goodies, let's take a look into how annotations are processed in Java.

Lombok uses Pluggable Annotation Processing API which was introduced in Java 6 (2006 - ten years ago!). It allows programmers to write custom annotation processors that are plugged into compiling process. Processor receives a source code's AST ([abstract sytntax tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree)) which is a tree of nodes representing variables, operators, statements and others language components; imagine it as a Java files's DOM. Having it, processor can analyze, validate or generate new code. Through the years multiple libraries started to use this API, the best known are probably JPA and GWT.

It is worth to note that Lombok does more then API allows: it modifies [AST of a class](http://notatube.blogspot.com/2010/11/project-lombok-trick-explained.html). People behind project found an internal API which can be used for this purpose. It goes without saying that this aproach make them vulnerable to future JDK changes, but for now - and for many years - it works.

But let's go back to Lombok. Below I present [three of many available features](https://projectlombok.org/features/index.html) that I instantly put into my project, which of course is only a subset of available functionalities.


<h2>Automatic getter/ setter/ constructor</h2>
As I mentioned in the first paragraph, C# already has this feature. Using the annotations programmer marks fields which should be accessed via getter or setter and doesn't has to generate them with IDE. It may seem as a little gain, but think about the situation when you change parameter name or type - IDE's refactor or manual work is required for that; Lombok makes it instantly.

Isn't following code beautifully simple and readable? Note annotations can be used on a field or a class.
{% highlight java %}
@Getter
public class DiskArticle {
    private String title;
    private List<String> contents = new ArrayList<>();
    @Setter
    private Date creation;
    @Setter
    private long size;
}
{% endhighlight %}

What's more, finally it is possible to create real POJO/ DTO classes in Java:
{% highlight java %}
@Getter @RequiredArgsConstructor
public class Preview {
    private final String id;
    private final String title;
}
{% endhighlight %}


<h2>Logging simplification</h2>
Writing serious apps requires a lot of logging. The more information you log, the more repetetive declaring logger is, and usual you copy a piece of code from another class. With Lombok, it is dead simple:

{% highlight java %}
@Slf4j
public class ArticleSearcher {
	(...)
	private QueryResponse query(SolrQuery solrQuery) {
        QueryResponse response = null;
        try {
            response = solrClient.query(solrQuery);
        } catch (SolrServerException | IOException e) {
            log.error("Query processing error: " + e);
        }
        return response;
    }
}
{% endhighlight %}


<h2>Equals & hashcode generation</h2>
Equals & hashcode are usually another IDE-generated elements of a class. The more time goes by since a class creation, the less intuitive it is to recognise if some fields should be a part of those methods, or rather are intentionally omitted. With Lombok these problem is marginalised.

{% highlight java %}
@EqualsAndHashCode(exclude={"id"})
public class Article {
	(...)
}
{% endhighlight %}


<h2>Plugins</h2>
It is important to note that those features would be useless without a good code complation of modern IDE-s. Fortunately Intellij Idea - which I use and recommend - has the plugin that make finding Lombok-generated methods almost as easy as using standard ones. [Eclipse and Netbeans also are supported](https://projectlombok.org/download.html). Finding usages on a field can point you to a used getter, and when you use an object Idea prompts methods. Example is shown below:

![placeholder](https://raw.githubusercontent.com/mikolajkania/mikolajkania.github.io/master/_images/2016-06-28-lombok-code-completion.png "Logged http requests")


<h2>Final thouths</h2>
Lombok is an interesting library that makes Java code more readable and faster to write. There is a lot more features to play with and I encourage you to try it on your own, even with mentioned danger of changing internal API that project uses. Remember, Lombok doesn't greatly intervene with your code, so abandoning it wouldn't be a costly operation. There is a lot more to earn during fast development process, where classed are created and discarded, so you should give it a chance.