---
layout: post
title: Are you on the road to be a Spring MVC engineer? 
tags:
- java
- spring
- spring mvc
- servlet
- configuration
- test yourself
---
It's not bad to use a framework without complete knowledge of its internals, but there are fundamental topics that every developer should have covered. For this reason the title is a bit provocative, as I won't write about any deep themes: I'm asking three basic questions which are - sadly - not obvious for everyone working with Spring MVC. Are you really know what is going on under the hood when your web application starts? If so, it will be your shortest visit on this website.

1. What is WEB-INF directory and where it come from?
2. What is web.xml and is it necessary to have it in the application?
3. How to properly configure services factories in Spring MVC?

<!--excerpt-->

<h2>@1</h2>
I believe the first question is fairly easy. To find out what WEB-INF is, it's best to read description from the source, [Oracle's Servlet Specification](https://java.net/downloads/servlet-spec/Final/servlet-3_1-final.pdf):
{% highlight text %}Most of the WEB-INF node is not part of the public document tree of the application. Except for static resources and JSPs packaged in the META-INF/resources of a JAR file that resides in the WEB-INF/lib directory, no other files contained in the WEB-INF directory may be served directly to a client by the container. However, the contents of the WEB-INF directory are visible to servlet code using the getResource and getResourceAsStream method calls on the ServletContext.{% endhighlight %}

In other words: WEB-INF is used to standardize structure of Java enterprise applications & to prevent URL access to some of their resources.

So, *should every enterprise Java web application have WEB-INF directory*? YES, it is mandatory; check output war of your application and see it yourself. *But I haven't configured it, is it magic?* No, it isn't, you probably used [Maven War Plugin](https://maven.apache.org/plugins/maven-war-plugin/examples/adding-filtering-webresources.html), whose responsibility is to create proper directory structure. 

As it is the end of the story, let's move to the next question, way more interesting one.


<h2>@2</h2>
As [Google puts it](https://cloud.google.com/appengine/docs/java/config/webxml):

{% highlight text %}Java web applications use a deployment descriptor file to determine how URLs map to servlets, which URLs require authentication, and other information. This file is named web.xml, and resides in the app's WAR under the WEB-INF/ directory. web.xml is part of the servlet standard for web applications.{% endhighlight %}

Let's rehearse a common knowledge about JEE fundamental concept, a servlet. [Java Servlet](http://docs.oracle.com/javaee/6/tutorial/doc/bnafd.html) is a class which responds to - mostly http - requests and fulfill Servlet Specification. Servlets works within a servlet container (i.e. Tomcat or Jetty) which handles their requests and responses, network communication etc. Spring MVC creators assumed that there is a [Dispatcher Servlet](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-servlet) that takes incoming requests and passes them to one of Spring controllers that developers actually write; which one should be used is determined by URL mapping.

So, *is web.xml necessary and are you sentenced to use xml to configure your web apps*? 

The answer is *no*, starting with Servlet 3.0 specification web.xml is not required and Spring also went this way. In MVC you only need a class that implements a [WebApplicationInitializer](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/WebApplicationInitializer.html) interface and which takes on the role of mentioned file. There are some devs who mourn about good-old-days when everything was harder to debug and configured in one xml file, but in my opinin the code-based configuration is much easier to handle, modern approach. 

Simple example of web.xml equivalent from Spring docs:
{% highlight java %}public class MyWebAppInitializer 
	implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext container) {
      XmlWebApplicationContext appContext = 
		new XmlWebApplicationContext();
      appContext.setConfigLocation(
		"/WEB-INF/spring/dispatcher-config.xml");

      ServletRegistration.Dynamic dispatcher =
        container.addServlet("dispatcher", 
			new DispatcherServlet(appContext));
      dispatcher.setLoadOnStartup(1);
      dispatcher.addMapping("/");
    }
}
{% endhighlight %}

'Springification' of questions increases - the first one was a pure Java, the last will be completely about Spring.

<h2>@3</h2> Building modern web application requires having a template to get things started. Most developers don't dive into every line of code as long as it works, so they may be surprised that there are [Spring guideline](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-container-config) on how to provide beans and configuration for given app or servlet. Have you ever considered that? If not, I'll present the basics below.

For a simple application snippet from question two is enough - there is only one servlet and one beans source, it's not important if it is XML or code-based. Things get a bit more tricky when you have multiple servlets, as from [Spring docs](http://docs.spring.io/spring/docs/3.1.x/javadoc-api/org/springframework/web/WebApplicationInitializer.html):
{% highlight java %}
public class MyWebAppInitializer 
	implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext container) {
      // Create the 'root' Spring application context
      AnnotationConfigWebApplicationContext rootContext =
        new AnnotationConfigWebApplicationContext();
      rootContext.register(AppConfig.class);

      // Manage the lifecycle of the root application context
      container.addListener(
		new ContextLoaderListener(rootContext));

      // Create the dispatcher servlet's application context
      AnnotationConfigWebApplicationContext dispatcherContext=
		new AnnotationConfigWebApplicationContext();
      dispatcherContext.register(DispatcherConfig.class);

      // Register and map the dispatcher servlet
      ServletRegistration.Dynamic dispatcher =
        container.addServlet("dispatcher", 
			new DispatcherServlet(
				dispatcherContext));
      dispatcher.setLoadOnStartup(1);
      dispatcher.addMapping("/");
    }
 }
{% endhighlight %}

So, *what is rootContext here, how should it be seperated from servlet context? Are contexts scoped or is there inheritence between them?* The answer lies in this paragraph from Spring's [The DispatcherServlet section](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-servlet):

{% highlight text %}In the Web MVC framework, each DispatcherServlet has its own WebApplicationContext, which inherits all the beans already defined in the root WebApplicationContext. The root WebApplicationContext should contain all the infrastructure beans that should be shared between your other contexts and Servlet instances. These inherited beans can be overridden in the servlet-specific scope, and you can define new scope-specific beans local to a given Servlet instance.{% endhighlight %}

This explanation is plain, simple and logical. Separating beans on this level helps to reduce redundancy and draw a visible line between servlets, which is crucial when problem with Spring context occurs. I guess everyone came across spaghetti Spring configuration at some point and for many devs it is the first time when they think about a need for order. 

![placeholder](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/images/mvc-context-hierarchy.png "Contexts scopes in Spring MVC")

In [my Spring startup project](https://github.com/mikolajkania/spring-mvc-startup/blob/master/src/main/java/pl/itblues/AppInitializer.java) on Github I followed this principles and I can say that finally I have a feeling that everything is configured just fine. Keep this in mind starting your next web application.

<h2>Final thoughts</h2>
There is nothing special or magical about above questions, but from my experience - both at work and from the Internet - there are many doubts about them. Especially when devs join a rather mature projects they don't want wasting time on fundamental principles, but rather create something visible and fresh. In my opinion, however, this knowledge is something that distinguish a software engineer from a coder, so I encourage everyone to explore this 'boring' topics ('stay hungry, stay foolish', someone would say).