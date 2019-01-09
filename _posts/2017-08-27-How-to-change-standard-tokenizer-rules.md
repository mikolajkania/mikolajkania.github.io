---
layout: post
title: How to change Solr standard tokenizer rules? 
tags: 
- solr
- lucene
- java
- jflex
- tokenization
- payloads

---
Although Solr comes with standard tokenizer implementation, which is well prepared to tokenize most of the texts, there are cases when it is helpless. Imagine a document with many numbers, of which many are followed by percentage sign. In a certain contexts it is expected to distinguish queries that refer to those percentages & plain numbers. How to achieve that? We need a custom tokenizer.

<!--excerpt-->

Another example may be connected with payloads, that is additional data that can be stored with a token. In a basic Solr convention they are served as _token`|`number_, which will be split into two tokens, loosing connection between them. Both cases would be addressed in these post. 

There may be multiple ways to define rules how text should be tokenized, but the one used in Lucene/Solr is based on Unicode Text Segmentation algorithm. As stated in [StandardTokenizer docs](http://lucene.apache.org/core/6_6_0/core/org/apache/lucene/analysis/standard/StandardTokenizer.html){:target="_blank"}:

{% highlight xml %}This class implements the Word Break rules from the Unicode Text Segmentation algorithm, as specified in Unicode Standard Annex #29.{% endhighlight %}

Those rules are defined in a [file](https://github.com/apache/lucene-solr/blob/master/lucene/core/src/java/org/apache/lucene/analysis/standard/StandardTokenizerImpl.jflex){:target="_blank"} together with class structure (name, package, imports) and then [JFlex library](http://jflex.de/){:target="_blank"} uses this notation to generate Java class that can be used in Solr.  


Rules alone are not something you want to analyse if you don't have to, so let's look at the most interesting part of a [documentation](http://unicode.org/reports/tr29/#Table_Word_Break_Property_Values){:target="_blank"}. It's easy to spot that values from these table can be mapped to variables mentioned in [StandardTokenizerImpl.jflex](https://github.com/apache/lucene-solr/blob/master/lucene/core/src/java/org/apache/lucene/analysis/standard/StandardTokenizerImpl.jflex){:target="_blank"}. For example _Hebrew_Letter_ can be found as a variable _HebrewLetter_ and as a part of a rule: 

{% highlight text %}HebrewOrALetterEx = [\p{WB:HebrewLetter}\p{WB:ALetter}] [\p{WB:Format}\p{WB:Extend}]* {% endhighlight %}

Knowing that we can start implementing changes that lead to having percentages & payloads kept in token stream next to corresponding base tokens. As we don't want to seriously change long-lasting rules, for payloads we've added a new one, which can be translated as: _do not split token when after a word there is a vertical bar followed by a number_. It looks like this:

{% highlight text %}
{HebrewOrALetterEx}+ \| {NumericEx}+
    { return WORD_TYPE; }  
{% endhighlight %}

The second snippet is for percentages and requires a small change in existing rule: 

{% highlight text %}
{ExtendNumLetEx}* {NumericEx} ( ( {ExtendNumLetEx}* | {MidNumericEx} ) {NumericEx} )* {ExtendNumLetEx}* \%*
   { return NUMERIC_TYPE; }
{% endhighlight %}
                        
Now the new tokenizer has to be built and tested. Exact configuration for that can be found on my [Github](https://github.com/mikolajkania/payloads-solr){:target="_blank"}, but in short only _mvn compile_ is required to see how generated _PayloadStandardTokenizerImpl_ looks like. 

For the testing part I've added a schema field that during indexing is using newly created tokenizer, and during querying - the standard one. Of course proper factory has be defined and new classes added to Solr as a library.

{% highlight xml %} 
<fieldType name="text_payloads" class="solr.TextField" positionIncrementGap="100">
    <analyzer type="index">
        <tokenizer class="pl.itblues.solrplugin.analysis.PercentStandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>       
        <filter class="solr.DelimitedPayloadTokenFilterFactory" encoder="float"/>
    </analyzer>
    <analyzer type="query">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
    </analyzer>
</fieldType>                        
{% endhighlight %}

For testing purposes I've prepared a query: _The population of Gdańsk`|`100 accounts for 0.012% of the population of Poland_. Gdańsk has added payload and there is percentage next to a number. By the way: why would we like to add payload to Gdańsk? For example to strengthen it during search, because others would have value of 1. I'll write about that soon, but for now let's go back to the results of experiment:

![placeholder](https://raw.githubusercontent.com/mikolajkania/mikolajkania.github.io/master/_images/2017-08-27_Payloads_query.JPG "fst")

As you can see the indexing part (the one we've updated) keeps payloads & percentages, while during querying (standard tokenizer used) percentage and vertical bar are removed and values are kept separately. There is nothing wrong about both behaviours - everything depends on a context of search.