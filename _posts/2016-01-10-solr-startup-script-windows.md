---
layout: post
title: Solr startup scripts on Windows
tags:
- solr
- windows
- script
---

As we all know from [Solr](http://lucene.apache.org/solr/){:target="_blank"} website:

> Solr is the popular, blazing-fast, open source enterprise search platform built on Apache Lucene.

It's all true but on the other hand Solr is not known from its ease-of-use. In the fifth version community decided to address one of the main issues making Solr more demanding for newcomers than competition, that is simplicity of startup. 

<!--excerpt-->

Despite making Solr [a standalone server](https://cwiki.apache.org/confluence/display/solr/Major+Changes+from+Solr+4+to+Solr+5){:target="_blank"} and preparing [in-depth documentation](https://cwiki.apache.org/confluence/display/solr/Solr+Control+Script+Reference) some users, especially Windows-based beginners, may struggle to run it. For them, and for my own convenience of course, I've created really simple and straightforward scripts for [creating sample index and starting a server](https://github.com/mikolajkania/solr-windows-scripts){:target="_blank"}. 

The material extending Solr documentation contains - among others - debugging configuration, which make analysis of server issues much easier. 

Hope it will save a few lives!
