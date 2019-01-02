---
layout: post
title: How to add data to Solr document during indexing?
tags:
- solr
- indexing
- update request processor
- java
---
The process of indexing in Solr in an advanced topic covered by many publications. On the most basic level it can be described as putting data into previously prepared containers. But what if user wants to perform additional data processing depending on documents that already are in the index?

<!--excerpt-->
Throwing away most trivial scenerio (calling Solr before sending data to index and filling required fields), we can go instanly to a solution - [Update Request Processors](https://cwiki.apache.org/confluence/display/solr/Update+Request+Processors){:target="_blank"}.

<h2>What is update request processor?</h2>
Basically it is an abstract class to process document before it is indexed. Configured through a solrconfig file it is greatly extensible, as processors are usually grouped into a chain. Solr creates default one to handle basic use cases, but it can be easilly extended.

The applications of processors are wide as they allow to add, modify and delete entire fields. It can be used to add custom ones with entities extracted from content, assign document to one of predefined groups depending on terms it contains, detect language or add debug data.

<h2>The plan</h2>
Let's say we have a Solr plugin that updates indexed document depending on one of its fields. In a real application it would probably do some text-oriented operations, which is Solr's main area, but for the purpose of this post it will add a new field with title words count. As you can imagine avoiding populating model with debug data, such as length of a field, may be another advantage of using processors.

<h2>The code</h2>
Let's make input data plain and simple:
{% highlight java %}
@Getter @Setter*
public class SolrArticle {
    @Field(SolrField.ID)
    private int id;
    @Field(SolrField.TITLE)
    private String title;
    public SolrArticle(int id, String title) {
        this.id = id;
        this.title = title;
    }
}

public final class SolrField {
	public static final String ID = "id";
	public static final String TITLE = "title_t";
}
{% endhighlight %}
<em>*The annotations you see come from Lombok Project which [I previously described](http://itblues.pl/2016/06/28/what-is-project-lombok-and-why-you-should-use-it/){:target="_blank"}.</em>

Update request processors are being created through factories. It is advisable to use default ones and extend the chain with custom processor. When a chain is created it should be registered for updates:
{% highlight xml %}
<updateRequestProcessorChain name="process-articles">
    <processor class="pl.itblues.solrplugin.blg.SimpleArticleProcessorFactory"/>
    <processor class="solr.LogUpdateProcessorFactory" />
    <processor class="solr.DistributedUpdateProcessorFactory" />
    <processor class="solr.RunUpdateProcessorFactory" />
</updateRequestProcessorChain>

(...)

<initParams path="/update/**,/query,/select,/tvrh,/elevate,/spell,/browse">
<lst name="defaults">
  <str name="df">text</str>
  <str name="update.chain">process-articles</str>
</lst>
</initParams>

{% endhighlight %}
As you may noticed I used a custom path for my processor. It may be worth to mention that if you want to add your code to Solr you have to create a plugin, which would be loaded by class loader at [Solr startup](http://itblues.pl/2016/01/10/solr-startup-script-windows/){:target="_blank"}. There are multiple ways to do that, but the one I prefer is to create lib directory under SOLR_HOME.

The factory itself do nothing more but creates a processor:
{% highlight java %}
public class SimpleArticleProcessorFactory extends UpdateRequestProcessorFactory {
    @Override
    public UpdateRequestProcessor getInstance(SolrQueryRequest req, SolrQueryResponse rsp, UpdateRequestProcessor next) {
        return new SimpleArticleProcessor(next);
    }
}
{% endhighlight %}

Adding a new field is as simple as calling a few methods:
{% highlight java %}
public class SimpleArticleProcessor extends UpdateRequestProcessor {
    public SimpleArticleProcessor(UpdateRequestProcessor next) {
        super(next);
    }
    @Override
    public void processAdd(AddUpdateCommand cmd) throws IOException {
        String title = (String) cmd.getSolrInputDocument().get("title_t").getValue();        
        cmd.getSolrInputDocument().addField("title_length_i", title.split(" ").length);        
        super.processAdd(cmd);
    }
}
{% endhighlight %}

<h2>The results</h2>
Voila! Nothing more have to be done. If you index a new document it will go through a chain of processors and end with three fields. The response from Solr could looks like this:
{% highlight json %}
{
  "responseHeader":{
    "status":0,
    "QTime":0,
    "params":{
      "q":"*:*",
      "indent":"on",
      "fl":"id,title_t,title_length_i",
      "fq":"id:1",
      "wt":"json"}},
  "response":{"numFound":1,"start":0,"docs":[
      {
        "id":"1",
        "title_t":"My title",
        "title_length_i":2}]
  }}
{% endhighlight %}

Let's me explain now how is this connected to updating document based on data already existing in the index. Imagine that you have collection previously filled with information: dictionary values or important concepts. During document indexing you can use this data to perform operations on submitted fields, for example to replace dictionary IDs with its names or to retrieve concepts and index them in another field. You can call service that takes URL from text and replace it with name and link. The possibilities are endless, but that operations could not be trival to perform, of course.

In the [next post](http://itblues.pl/2017/03/30/extract-entities-with-solr-text-tagger/) I would show more advanced use case for using processor - extracting entities from document content. Stay tuned!
