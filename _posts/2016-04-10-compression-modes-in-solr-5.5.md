---
layout: post
title: Compression modes in Solr 5.5
tags:
- solr
- compression
- index
- codec
---

The ideal situation is when whole index can be located in memory, due to disk operations are much slower then those in RAM. What's more, often companies have to fit the requirenments of the tender or reduce server costs, which put pressure on developers to come up with a solution that will make the index smaller.

Having this all in mind it's not surprising that field compression is an important feature due to its ability to significantly decrease an index size.

Since Solr 4.1 stored fields - responsible for majority of an index size - are by default compressed. In Solr 5.5 release limited ability to adjust compression level occurs, [with two modes to choose from](https://cwiki.apache.org/confluence/display/solr/Codec+Factory): BEST_SPEED (BS) & BEST_COMPRESSION (BC). 

I decided to check them using my 1k documents collection. Data were straightforward: articles with stored title & content, with additional fields type & creationDate.

Best speed mode is a default option and if you want to enable the other one all you have to do is to change solrconfig.xml with:
{% highlight xml %}
<codecFactory class="solr.SchemaCodecFactory">
  <str name="compressionMode">BEST_COMPRESSION</str>
</codecFactory>
{% endhighlight %}

What was the outcome? My findings leaves no doubt that saving is evident. BS provided 80,4 MB index and BC - 60,7 MB. It's 25%! The whole difference comes from the stored files (the ones with .fdt extension), which were, respectively, 48,1 MB & 28,4 MB. You can imagine that with more documents the gain will be proportionally higher.

What's about the speed? As the ad from official website says, Solr is truly blazingly fast, so my index is a way too small to push it to its limits. I made a few queries though, and an average response from Solr were 20ms vs 5ms in favor of the BS; but it rather shows direction, not a real difference.

Putting it all together - if a search response time is not your main concern or you are short on a disk space - choosing different compression mode may be your way to go.

Post scriptum: noSQL Solr 6 with SQL support [has been released](http://lucene.apache.org/solr/6_0_0/changes/Changes.html)!