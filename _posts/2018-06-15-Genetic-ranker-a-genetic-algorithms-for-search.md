---
layout: post
title: Genetic Ranker - genetic algorithms for search 
tags:  
- genetic algorithms
- search quality
- search
- python
- github
- open source
---

Working as a search engineer myself I decided to [develop a framework](https://github.com/mikolajkania/genetic-ranker){:target="_blank"} for finding optimal query weights for search engines like Elasticsearch or Solr. It is based on a machine learning branch called genetic programming, inspired by the process of natural selection. In this post I'll describe it and briefly discuss how the good process of building the quality of search should look like. Let's start!

<!--excerpt-->

## Problem to solve

Imagine you are a search engineer or analyst whose responsibility is the quality of search. Consider an index of millions of documents, every with tens of searchable fields. When user types query your ranking system evaluates final score of a document taking into account all of those fields with different weights assigned. It's your job to figure out values, somehow.

## Why is it a rocket science?

Hopefully, with a help of the experts, you will be able to prepare a list of expected documents for given queries. In advanced systems such a list may easily consist of hundreds of entries. Now, imagine intuitively changing weight of one of those fields, running queries manually and guessing if the results are better and what to do next... nightmare. You need a process that in a simplest version can look as follows. 

The first step would be running all queries with default settings and checking how the results deviates from expectations. You need an automatic system to perform that and metrics to get initial overview; it may be a number of valid to all queries or percentage of errors. After analysis, when you come up with patterns of errors, you can propose a solution. Then, it should be registered, applied and tested again. It should be done until results cannot be noticeably improved.

## Sounds easy? It isn't

Every iteration takes time of at least one person who must run queries, analyze the results and come up with a solution. It is not guaranteed that every proposed idea will be helpful in the long run - a promising direction can lead to a dead end. Spotting the impending danger is extramely hard, as sometimes it requires taking a few steps back in thinking and discarding the newest work. 

Another thing is necessity to automatically perform and view a summary of the results. It is crucial to quickly check promising ideas before applying them on greater scale. Building a human-friendly interface may be time-consuming and not necessarily effective as people with background in tech and analysis are looking into different features. But even in the ideal world testing & understanding of every change will take a lot of time and resources.   

## What is a solution?

Not surprisingly, you need an automatic system that do the most of the work for you. Having some background in search engineering and machine learning I've came up with idea of [genetic programming framework](https://github.com/mikolajkania/genetic-ranker){:target="_blank"} doing exactly that. 

Genetic algorithms are inspired by a process of natural selection. Let me borrow a more in-depth explanation from [a quality guide](https://www.tutorialspoint.com/genetic_algorithms/genetic_algorithms_quick_guide.htm){:target="_blank"}:
> In genetic algorithms, we have a pool or a population of possible solutions to the given problem. These solutions then undergo recombination and mutation (like in natural genetics), producing new children, and the process is repeated over various generations. Each individual (or candidate solution) is assigned a fitness value and the fitter individuals are given a higher chance to mate and yield more 'fitter' individuals.

## Why use genetic algorithms?

Putting aside the results of various optimisation experiments, genetic programming looks like a good bet for these kind of problem.    

Firstly, it is easy to define a candidate solution as a set of numeric weights that can be altered during processing. Mutation, crossover & reproduction parts of the algorithm can easily be done on numbers. Think about an individual of a population as a series of digits, where every entry is a weight of another field (i.e. [5, 1, 5, 7, 9, 4, 3, 0]). 

Secondly, genetic algorithms are able to preserve solutions that are promising, even without actual understanding of a problem. Better species (individuals) will be kept as an ancestors for even better ones.

Thirdly, due to mutation & crossover parts, algorithm adds a bit of randomness to the process. Every algorithm is prone to find a local minimum, that is a good but not the best solution. But with random changes applied to the individuals genetic algorithm may catch on the trail.  

## Performance

It may come as a surprise how many combinations of fields and weights are available. According to [a variation with repetition](http://www.emathematics.net/combinavrepeticion.php){:target="_blank"} having 7 fields with 5 possible weights (i.e. even numbers between 0 and 9) we end up with 78125 combinations. We are almost sure that among those combinations is a perfect one, but a cost of computation is a big one. And it is only [an easy example](https://github.com/mikolajkania/genetic-ranker#test-it-yourself){:target="_blank"} I've prepared to illustrate how Genetic Ranker works, not a real, complicated use case. For a document with 20 fields there is 95 trillions of combinations!

How Genetic Ranker looks in a comparison? It can repeatedly deliver the best solution in less than 500 evaluations, which was then reduced by half by introducing cache: during processing there is probability that certain combinations would reappear due to crossover or mutation. And Genetic Ranker didn't work on even numbers but on the whole range from 0 to 10.

## Summary

[Genetic Ranker](https://github.com/mikolajkania/genetic-ranker){:target="_blank"} is a framework using the power of machine learning to help users deliver better search experience. Depending on the index size they can choose more generations to improve quality or end expensive computations sooner and treat results as a starting point for further analysis. Either way it will help. Give it a try!
