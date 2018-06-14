---
layout: post
title: Genetic Ranker - genetic algorithms for search 
tags: 
- todo

---

Working as a search engineer myself I decided to [develop a framework](https://github.com/mikolajkania/genetic-ranker) for finding optimal query weights for search engines like ElasticSearch or Solr. It is based on a machine learning branch called genetic programming, inspired by the process of natural selection. In this post I'll describe it and briefly discuss how the good process of building the quality of search should look like. Let's start!

<!--excerpt-->

## Problem to solve

Imagine you are a search engineer or analyst whose responsibility is the *quality of search*. Consider an index of millions of documents, every with tens of searchable fields. When user types query your ranking system evaluates final score of a document taking into account all of those fields with different weights assigned. It's your job to figure out values, somehow.

## Why is it a rocket science?

Hopefully, with a help of the experts, you will be able to prepare a list of expected documents for given queries. In advanced systems such a list may easily consist of hundreds of entries. Now, imagine intuitively changing weight of one of those fields, running queries manually and guessing if the results are better and what to do next... nightmare. *You need a process* that in a simplest version can look as follows. 

The first step would be *running all queries* with default settings and checking how the results deviates from expectations. You need an *automatic system* to perform that and metrics to get initial overview; it may be a number of valid to all queries or percentage of errors. After *analysis*, when you come up with *patterns of errors*, you can propose a solution. Then, it should be registered, applied and tested again. It should be done until results cannot be noticeably improved.

## Sounds easy? It isn't

*Every iteration takes time* of at least one person who must run queries, analyze the results and come up with a solution. It is not guaranteed that every proposed idea will be helpful in the long run - a promising direction can lead to a dead end. Spotting the impending danger is extramely hard, as sometimes it requires taking a few steps back in thinking and discarding the newest work. 

Another thing is necessity to automatically perform and view a summary of the results. It is crucial to quickly check promising ideas before applying them on greater scale. Building a human-friendly interface may be time-consuming and not necessarily effective as people with background in tech and analysis are looking into different features. But even in the ideal world testing & understanding of every change will take *a lot of time and resources*.   

## What is a solution?

Not surprisingly, you need an automatic system that do the most of the work for you. Having some background in search engineering and machine learning I've come up with idea of **[genetic programming framework](https://github.com/mikolajkania/project-x)** doing exactly that. 

## Genetic algorithms

Genetic algorithms are inspired by a process of natural selection. Let me borrow a more in-depth explanation from [a quality guide](https://www.tutorialspoint.com/genetic_algorithms/genetic_algorithms_quick_guide.htm):
> In genetic algorithms, we have a pool or a population of possible solutions to the given problem. These solutions then undergo recombination and mutation (like in natural genetics), producing new children, and the process is repeated over various generations. Each individual (or candidate solution) is assigned a fitness value and the fitter individuals are given a higher chance to mate and yield more 'fitter' individuals.

## Genetic algorithms for search

Why genetic programming is suitable for this kind of a problem? Putting aside the proven results of various optimisation experiments, the nature of our task also matches its applications.    

Firstly, it is easy to *define a problem as set of numeric weights* that can be altered during processing. Mutation, crossover & reproduction parts of the algorithm can easily be done on numbers. Think about an individual of a population as a series of digits, where every entry is a weight of a another field (i.e. [5, 1, 5, 7, 9, 4, 3, 0]). 

Secondly, genetic algorithms are able to preserve solutions that are promising, even without actual understanding of a problem. Better species (individuals) will be kept and be an ancestors for even better ones.

Thirdly, due to mutation & crossover parts, algorithm *adds a bit of randomness to the process*. Every algorithm is prone to find a local minimum, that is a good but not the best solution. But with random changes applied to the individuals genetic algorithm may catch on the trail.  

Putting everything together it is clear, that genetic algorithm is a good bet for these kind of problems.

## Results

## Performance

## Summary

In this post I described how the process of building the most basic search solution should look like. I have also presented why there is a need of automation in these area and proposed [my own project](https://github.com/mikolajkania/project-x) discovering optimal search weights. I hope you will find it interesting & helpful!