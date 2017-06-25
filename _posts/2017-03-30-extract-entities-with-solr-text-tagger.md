---
layout: post
title: Extract entities from document with Solr Text Tagger
tags:
- solr
- entity extraction
- concept extraction
- solr text tagger
- fst
- finite state transducer
- indexing
- update request processor
- java
- curl
---
Algorithms for extracting entities from text are ones of the most crucial aspects of text analysis. They lead to better understanding of the content, enable additional operations like filtering or grouping and - most importantly - allow to process data automatically. In the [previous post](http://itblues.pl/2017/02/19/how-to-add-data-to-solr-during-indexing/) I announced combination of text indexing & such extraction and in order to keep my promise I created a [fork](https://github.com/mikolajkania/SolrTextTagger) of [Solr Text Tagger](https://github.com/OpenSextant/SolrTextTagger).

<!--excerpt-->

<h2>The goal</h2>
To demonstrate entity extraction I will track down cities names in the indexed documents and then add their idies into documents. Then operations like faceting or filtering could go in a standard Solr way. 

I will also show that cities might be found in a query, with their offsets. This can be used to change initial text to enable proximity search or query another set of fields.

<h2>How it works?</h2>
On a high level of abstraction it would work as follows: Solr Text Tagger scans text in search of previosly declared entities. Those entites are Solr documents with id & names, in my case *city_names_tags*. 
{% highlight xml %}
<dynamicField name="*_tag" type="tag" indexed="true" stored="true"/>
<dynamicField name="*_tags" type="tag" indexed="true" stored="true" multiValued="true"/>

<fieldType name="tag" class="solr.TextField" positionIncrementGap="100" postingsFormat="Memory" omitTermFreqAndPositions="true" omitNorms="true">
  <analyzer type="index">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.EnglishPossessiveFilterFactory" />
    <filter class="solr.ASCIIFoldingFilterFactory"/>
    <filter class="solr.LowerCaseFilterFactory" />
    <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
    <filter class="org.opensextant.solrtexttagger.ConcatenateFilterFactory" />
  </analyzer>
  <analyzer type="query">
	<tokenizer class="solr.StandardTokenizerFactory"/>
	<filter class="solr.EnglishPossessiveFilterFactory" />
	<filter class="solr.ASCIIFoldingFilterFactory"/>
	<filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
	<filter class="solr.LowerCaseFilterFactory" />
  </analyzer>
</fieldType>
{% endhighlight %}

Let's discuss what is important here: during indexing tokens in *tag* field are **concatenated into final one**, as we need complete match in text. Another thing is that terms of this field are kept in memory in efective structure called Finite State Transducer (FST).

**What is FST?** It is a kind of finite state machine (FSM), that is a graph with nodes (states) & labels (edges). Finite state machine has an input which determines transitions between nodes. FST is an extension of state machine with an output node. In Lucene it is used to map term into integer and is implemented by Michael McCandless, whose [example of a structure](http://blog.mikemccandless.com/2010/12/using-finite-state-transducers-in.html) is quite clear:
![placeholder](https://raw.githubusercontent.com/mikolajkania/mikolajkania.github.io/master/_images/2017-03-30-fst.png "fst")
{% highlight text %}FST maps the sorted words mop, moth, pop, star, stop and top 
to their ordinal number (0, 1, 2, ...). As you traverse the 
arcs, you sum up the outputs, so stop hits 3 on the s and 1 
on the o, so its output ordinal is 4.{% endhighlight %}
**These mapping reduces memory usage, but increases CPU cost** of lookups. As the memory is main problem with Solr/ Lucene, it's a cost we can afford. 

The last question to answer in this section is **how the extraction algorithm works** with this FST representation. Let's say we are quering a phrase *new york times* - there is a hit for *new* as there are multiple city names starting that way. Then the second token in taken into consideration, and still there is a hit in directory (*new york*), but *york* alone is also marked as a takeoff for a new entity. The next steps would be checking *new york times* and *york times*, which in both cases mean end of lookup. *Times* is also a dead end, so we finish with entities *york* & *new york* unless overlapping is enabled. The path for such result is as follow:
{% highlight text %}new
new york
york
new york times
york times
times{% endhighlight %}

<h2>Indexing content</h2>
Before we will start tagging indexing part should be set up. For the needs of that I wrote my own update Solr client: *ProcessorAwareConcurrentUpdateSolrClient*. I wanted to be able to pass *update.chain* parameter which decides which update processor will be used. For cities I need a default one, for articles (my source documents) it has to use my entity extraction processor.
{% highlight java %}
public UpdateResponse add(String collection, Collection<SolrInputDocument> docs, int commitWithinMs, String processor) throws SolrServerException, IOException {
	UpdateRequest req = new UpdateRequest();
	req.add(docs);
	req.setParam("update.chain", processor);
	req.setCommitWithin(commitWithinMs);
	return req.process(this, collection);
}
{% endhighlight %}
{% highlight xml %}
<updateRequestProcessorChain name="process-articles">
 <processor class="pl.itblues.solrplugin.ArticleProcessorFactory"/>
 <processor class="solr.LogUpdateProcessorFactory" />
 <processor class="solr.DistributedUpdateProcessorFactory" />
 <processor class="solr.RunUpdateProcessorFactory" />
</updateRequestProcessorChain>
<updateRequestProcessorChain name="process-default">
 <processor class="solr.LogUpdateProcessorFactory" />
 <processor class="solr.DistributedUpdateProcessorFactory" />
 <processor class="solr.RunUpdateProcessorFactory" />
</updateRequestProcessorChain>
{% endhighlight %}
For more information about that process I would again point you to my [previous post](http://itblues.pl/2017/02/19/how-to-add-data-to-solr-during-indexing/).

Processor I wrote retrieves one of the document fields and pass it to handler responsible for text tagging. Returned cities ids are added as a new field to the document. 
{% highlight java %}
@Override
public void processAdd(AddUpdateCommand cmd) throws IOException {
  SolrQueryRequest req = cmd.getReq();
  SolrIndexSearcher searcher = req.getSearcher();

  NamedList<String> tagParams = new NamedList<>();
  tagParams.add(TaggerRequestHandler.TEXT_TO_TAG, 
    cmd.getLuceneDocument().
	getField("title_t").
	stringValue());

  SolrQueryResponse response = new SolrQueryResponse();
  SolrRequestHandler tagHandler = 
    searcher.getCore().getRequestHandler("/tag");
  tagHandler.handleRequest(new LocalSolrQueryRequest
    (searcher.getCore(), tagParams), response);
	

  List<NamedList> tags = (List<NamedList>) 
    response.getValues().get("tags");
  if (tags != null && tags.size() > 0) {
    List<Integer> tagIds = new ArrayList<>();
	for (NamedList tag : tags) {
	  List<Integer> ids = (List<Integer>)tag.get("ids");
	  tagIds.addAll(ids);
	}
	cmd.getSolrInputDocument().addField("tags_im", tagIds);
  }
  super.processAdd(cmd);
}
{% endhighlight %}

It is worth to mention, that cities list I used is part of Solr Text Tagger so you can also use it for tests.

<h2>Tagging</h2>
Despite the fact that Solr Text Tagger is a working piece of code I decided to fork it and change behaviour a bit - I've changed a way it processes text that has to be tagged:
{% highlight xml %}
<requestHandler name="/tag" class="org.opensextant.solrtexttagger.TaggerRequestHandler">
  <lst name="defaults">
    <str name="field">city_names_tags</str>    
    <str name="overlaps">LONGEST_DOMINANT_RIGHT</str>  
  </lst>
</requestHandler>
{% endhighlight %}
*Field* says from which field tags should be retieved, *overlaps* - how overlapping entities should be proceed (for example *York* & *New York*). In this case only the longest one will be returned. More detailed information about changes I made to tagger could be found [on Github](https://github.com/mikolajkania/SolrTextTagger/blob/b01f4bb7d385d1cad7a691a138c2aee407925219/src/main/java/org/opensextant/solrtexttagger/TaggerRequestHandler.java).

<h2>Testing</h2>
I added article with title *Mario Balotelli Brendan Rodgers will help me realise my potential at Liverpool FC - Liverpool Echo* to Solr. As expected, indexed tag was the one with id 2644210 - Liverpool.

{% highlight json %}{"id":"101",
"title_t":"Mario Balotelli Brendan Rodgers will help me realise my potential at Liverpool FC - Liverpool Echo",
"tags_im":[2644210]}

{"id":"2644210",
"city_name_tag":"Liverpool",
"city_names_tags":["Gorad Liverpul",
"LPL", "Liverpool" (...)]}
{% endhighlight %}

I also used *curl* to test how request handler would work with provided text - query to analysis could be served that way. What's more, I've also used different options for overlapping: first query chooses longest city name, second one should return all of them:
{% highlight bash %}
curl -X POST "http://localhost:8983/solr/articles/tag?overlaps=LONGEST_DOMINANT_RIGHT&tagsLimit=5000&fl=city_name_tag&wt=json&indent=on" -H "Content-Type:text/plain" -d "the new york times"
curl -X POST "http://localhost:8983/solr/articles/tag?overlaps=ALL&tagsLimit=5000&fl=city_name_tag&wt=json&indent=on" -H "Content-Type:text/plain" -d "the new york times"
{% endhighlight %}
It also worked as expected. I'd suggest paying attention to offsets returned by those queries: in both cases starts & ends of extracted entities are valid.

![placeholder](https://raw.githubusercontent.com/mikolajkania/mikolajkania.github.io/master/_images/2017-03-30-results.PNG "curl results")

I run more queries and didn't encounter any problems with Solr Text Tagger - the only difficulties are result of data. I preprocessed them a bit - filtered out cities with population less then 100k to avoid entries like *Merkel* - but more should be done. For example if in title appears word *liga* (*league* in Polish) it would be recognized as Latvian *Riga*. Cities names should be adjusted to language of a document and only a subset of them should be used - but this & other improvements are out of scope of this post.

<h2>Summary</h2>
In this post I proved that - with minor modifications - Solr Text Tagger can be used to extract entities from texts of documents being indexed. They can be a subject of further processing, like addition to a document field or modification of user query. Another approach could be using their offsets to add payloads to texts, and then boost documents - but it may be a topic for another post.