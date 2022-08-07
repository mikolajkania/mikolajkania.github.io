---
layout: post
title: Can DVC be used for Kaggle?
---

The more I am working with data science, the more need for organized process I see. It can be enforced be conventions, documentation, code reviews, but as always it is automation that pays off the most. 'Developer *must* be lazy', as one of my teachers used to say. This is why, when I took part in another [Kaggle competition](https://github.com/mikolajkania/kaggle-03-house-prices), I tried to build it around more mature ideas than just experimenting in Jupyter Notebook.

<!--excerpt-->

# Introductions

Kaggle is a data science competition, in which any developer can try their luck against defined problem. It is common for participants to create cluttered Jupyter Notebooks, that is files containing both code and its output. They are a great for parts of data science lifecycle (like data or model analysis), but are also leading to spaghetti code that is hard to reproduce. Engineering side of data science, MLOps, is an answer to that particular issue. My tool of choice, to organize code better this time, was DVC.

[DVC](https://dvc.org/) is a tool that became famous by providing ability to easily track data used for training machine learning models. Instead of storing them directly in Git, it offers ability to keep only references to them, and physically store in another location, like AWS S3. It is being actively developed and now offers much more features. I have already tried it for my another personal project and also in commercial work.

# What does DVC give?

## Standardized structure

Despite a good documentation it takes time to adapt to DVC requirements. However, once you get to know them, you can more easily find yourself in every new project that is using it. You will start by looking into [dvc.yaml](https://github.com/mikolajkania/kaggle-03-house-prices/blob/main/dvc.yaml) to get insight what stages are defined. Quick scan of [params.yaml](https://github.com/mikolajkania/kaggle-03-house-prices/blob/main/params.yaml) will provide state of algorithms and data processing applied and in [metrics file](https://github.com/mikolajkania/kaggle-03-house-prices/blob/main/models/metrics-train.json) you fill find current results. Such familiarity can be particularly useful when you are changing projects relatively often or you want to avoid writing new framework for every Kaggle competition. As everything is stored in a code, there is lesser need to write documentation.

![placeholder](https://raw.githubusercontent.com/mikolajkania/mikolajkania.github.io/master/_images/2022-08-07-dvc.png "dvc.yaml")

But did it help with Kaggle? Yes - I build code around known framework.

## Ease of conducting experiments

Having DVC configured, running new experiment is as easy as putting [*dvc repro*](https://dvc.org/doc/command-reference/repro) in your favourite terminal. It fires new process with stages defined in already mentioned dvc.yaml. What is important, those stages will be cached and DVC might decide to run only subset of them that changed. Stages could be loading, preprocessing, training, evaluation, submission creation...

... but in my case I had only two: training and submission (where final model was also trained). As number of samples was small, splitting it and caching was not required. But *dvc repro* was used few times a day and I love I can run it from terminal.

## Results tracking

This is where DVC can shine, when engineer helps it to. Every experiment run ends with new artifacts generated and metrics saved to files on disk. As you can imagine, those files are stored in Git, so for every run (again - fired easily with *dvc repro*) you are getting combination of code, parameters and results. One could argue that the same can be achieved with notebook - yes, to some extent. But notebooks are really bad to compare changes between runs and here Git comes to the rescue. You can imagine how powerful it is when you are working with other data scientists - discussing experiments and reviewing their scope is much more transparent.

For Kaggle I built process around above concept. For every major experiment type (like introduction of [models voting](https://github.com/mikolajkania/kaggle-03-house-prices/pull/13)) I created a branch. Within a branch I was experimenting a lot with given idea and persisting notable outcomes (both successes and failures). In the end configuration that was providing best results was submitted to Kaggle and code merged to the master. Checking improvements was very easily due to comparison with history in Git.

Here you can see it in action. Firstly I updated [params.yaml](https://github.com/mikolajkania/kaggle-03-house-prices/pull/13/commits/48ce984c153aa9ee9c70f50fbbb077b3cf0f4ae5#diff-f2f91cb656b58c7f581dcbdf3227f06412b648c6e0a5c10ea26e131c1eaa07e8) and run *dvc repro*:

![placeholder](https://raw.githubusercontent.com/mikolajkania/mikolajkania.github.io/master/_images/2022-08-07-params.png "dvc params.yaml")

Secondly, model was trained, [metrics stored in a file](https://github.com/mikolajkania/kaggle-03-house-prices/pull/13/commits/48ce984c153aa9ee9c70f50fbbb077b3cf0f4ae5#diff-0ea8d36aa8b430bd09f958eed7f3f695dc963bc366b3fb4362a3c96b30b60f8e) and after my review - [committed to repo](https://github.com/mikolajkania/kaggle-03-house-prices/pull/13/commits). The last step was to merge code into main branch. 

Because of that it was also easier to gather all metrics, as they are sitting in Git history. You can see my summary [here](https://github.com/mikolajkania/kaggle-03-house-prices/blob/main/README.md#results). 

## Reproducibility

Git helps to track experiments and with project structure that DVC enforces you always have configuration that is connected to metrics. It goes without saying that with such approach it is easy to re-run experiment and get the same results. 

DVC has also a feature to store resources in an external repository, i.e. [S3 bucket](https://dvc.org/blog/aws-remotes-in-dvc). It can be applied to both data and training. When experimenting with particular preprocessing it can be saved and restored, which is much better than intermediate files with naming like *preproc-data-3.pk* in a project directory. In similar way, given model can be saved and evaluated later.

I was coming back to particular configurations many times, as not every experiment is a success. When initially promising one was not yielding expected results, I was just resetting branch config to previous state, without searching in broken notebook history. On the other hand, again due to small training set size, I didn't push intermediate resources to S3.

# What didn't work as expected?

## DVC experiments

In my another personal project I was successfully using [DVC experiments feature](https://dvc.org/doc/start/experiments). It helps to improve tuning by scheduling experiments runs with different parameters and automatic comparison of results provided by DVC. It is different to *dvc repro*, which always runs with state taken from a code; here you are adding all conducted experiments to a *leaderboard* with parameters used and metrics obtained. You can add them to queue and go for a walk, as DVC will handle running them for you. Additionally, DVC is creating Git [branch per experiment](https://dvc.org/doc/start/experiments#comparing-and-persisting-experiments), so you can explore results after run ended. If run is particularly interesting, its configuration can be promoted as a new default to your master branch with a single command.

When starting working with this feature, I was prepared for some drawbacks. Firstly, seeing results in a terminal is not as intuitive as it would be with dedicated UI. I hope Iterative will provide self-hosting project mitigating this issue in the future. Secondly, I already knew that this feature is not working well on Windows, so quite early I migrated to [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install) aka WSL. It is a polished project and I recommend it to every engineer using Windows as a main operating system.

What I didn't expected, though, were issues while [scheduling experiments](https://dvc.org/doc/user-guide/experiment-management/running-experiments#the-experiments-queue).

![placeholder](https://raw.githubusercontent.com/mikolajkania/mikolajkania.github.io/master/_images/2022-08-07-dvc-queue.png "dvc queue")

Instant run of single experiment, which is similar to *dvc repro* but with experiment added to a *leaderboard*, was working ok. Unfortunately, it was not a use case for me, as I had high expectation towards scheduling multiple runs with different parameters and comparison of its outputs. It was especially important for me at the end of experimentation, when many parameters and algorithms were already available in my framework. Running *dvc repro* for every combination was tedious, but idea of 'schedule and go for a walk' was appealing. Not this time and it was a major blow.

## Notebooks advantage

I already mentioned that Jupyter Notebook is great for prototyping/ analysis and I used it for [an EDA of training set](https://github.com/mikolajkania/kaggle-03-house-prices/blob/main/notebooks/eda_train.ipynb). 

But if I spent more time on this competition, I'd explore how to introduce notebook into a training process. Idea I had in mind was notebook with model evaluation, orchestrated by [papermill](https://github.com/nteract/papermill), which I already touched briefly in a [previous run on Kaggle](https://github.com/mikolajkania/kaggle-02-disaster-tweets/blob/main/notebooks/ppm_lstm.py). Such notebook could describe trained model in a deeper way, giving an option to play with candidate before creating a Kaggle submission. DVC and Git are too limited on visualization side, but one could argue that notebook serves a different purpose. Still, you can't (easily) fully get rid of it if you want to achieve best results.

# Summary

In this blog post I described how to use DVC as building block of Kaggle competition. Despite a few drawbacks it helps to organize code in a more elegant way, easily run and track experiments, making it a good introduction into MLOps topic. What's more, beyond Kaggle, its impact can be even strengthened in environment where few data scientists are working together, due to clear structure and easy to track configuration. I highly recommend giving it a chance.










