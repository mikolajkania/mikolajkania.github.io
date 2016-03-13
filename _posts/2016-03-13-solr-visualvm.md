---
layout: post
title: Solr with VisualVM via JMX
---

Recently I had a need to measure Solr memory usage and I decided to user free Oracle tool - VisualVM. As usual an [official documentation](https://cwiki.apache.org/confluence/display/solr/Taking+Solr+to+Production#TakingSolrtoProduction-EnableRemoteJMXAccess) provides some help but to make things simpler I extended may [Solr startup script for Windows](http://itblues.pl/2016/01/10/solr5-startup-script/) to put all important information in one place. Happy using but be careful, JMX connection should not be allowed in production environment!

The core change is to add a few JVM parametes on Solr startup:
	
{% highlight text %} -Dcom.sun.management.jmxremote \ -Dcom.sun.management.jmxremote.local.only=false \ -Dcom.sun.management.jmxremote.ssl=false \ -Dcom.sun.management.jmxremote.authenticate=false \ -Dcom.sun.management.jmxremote.port=18983 \ -Dcom.sun.management.jmxremote.rmi.port=18983{% endhighlight %}

Solr documentation provides simpler way to do that, but I prefer to declare things explicitly.
