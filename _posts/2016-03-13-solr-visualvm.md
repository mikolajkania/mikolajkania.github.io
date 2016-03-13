---
layout: post
title: Solr with VisualVM via JMX
---

Recently I had a need to measure Solr memory usage and I decided to use free Oracle tool - VisualVM. As usual an [official documentation](https://cwiki.apache.org/confluence/display/solr/Taking+Solr+to+Production#TakingSolrtoProduction-EnableRemoteJMXAccess) provides some help, but to make things simpler I extended my [Solr startup script for Windows](http://itblues.pl/2016/01/10/solr5-startup-script/) to put all important information in one place.

The core change is to add a few JVM parameters on Solr startup:
	
{% highlight shell %} -Dcom.sun.management.jmxremote \ -Dcom.sun.management.jmxremote.local.only=false \ -Dcom.sun.management.jmxremote.ssl=false \ -Dcom.sun.management.jmxremote.authenticate=false \ -Dcom.sun.management.jmxremote.port=18983 \ -Dcom.sun.management.jmxremote.rmi.port=18983{% endhighlight %}

Documentation provides simpler way to do that, but I prefer to declare things explicitly. Having this parameters passed to starting Solr you can connect to server by opening VisualVM and choosing:

> Local > Add JMX Connection > localhost:[port number, in our case 18983]

Below you can see an example of VisualVM heap memory output for Solr 5 indexing 1k documents.

![placeholder](https://raw.githubusercontent.com/mikolajkania/mikolajkania.github.io/master/_images/2016-03-13-memory.png "Memory usage during indexing documents")

Happy using but be careful, JMX connection should not be allowed in production environment! 