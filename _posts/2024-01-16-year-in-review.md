---
layout: post
title: 2023, My year in review
---

Looking back into learnings, changes and... culture.

<!--excerpt-->

# Engineering
When you are joining an industry to develop as a machine learning specialist, you envision yourself a problem solver in an endless stream of projects. The truth is, however, that whereas it is important to sharpen data science skills, equally important it keep engineering side taken care of. My experience as backend & search engineer helped me to embrace this truth quite early I am ready to share my basic **MLOps overview**.  

## Running experiments
Capability to have a predictable and transparent experiments execution is a crucial part of **[mature MLOps setup](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning)**.

**[Amazon Sagemaker pipelines](https://aws.amazon.com/sagemaker/pipelines/)** and processing jobs are an answer to the need to build high-quality services. Their advantages are broad: trackable executions, flexible resources allocation, reproducibility or caching are first coming to my mind. It is not without its issues, however: jobs are sometimes hanging, metrics tracking at the time of using was irritating and I would expect a better protection from unexpected costs. Overally it is an option worth considering, but of course, it is more about capabilities provided than actual vendor.

My journey in choosing a backbone experimentation runner was involving [DVC](https://dvc.org/doc/start/experiments), [Kedro](https://kedro.org/), [Prefect](https://www.prefect.io/) and **ordinary Python scripts**. I ended up with the last option, without adding additional level of abstraction - complexity should be added only when actual need is identified. Having said that, I can definitely see **Prefect** as a good workflow manager in the advanced pipelines.

## Tracking
Centralized metrics tracking is subpar in Sagemaker and too limited in [DVC](https://dvc.org/doc/start/data-management/metrics-parameters-plots). Although I found [mlflow](https://www.mlflow.org/) a bit too clunky, it is an industry standard for a good reason due to integrations and a solid documentation.

After further research, I gave **[Weights & Biases](https://wandb.ai/)** a chance, and it turned out to be well-designed and approachable. Its documentation is good and I found their integrations and model storages easy to set up; also ecosystem was inviting to start with only subset of features. I really liked how easy it is to run it locally for experimentation and go online for finalized runs. There is also an **[Optuna callback](https://github.com/nzw0301/optuna-wandb)** for W&B, which makes it easier to inject hyperparameter tuning into your workflow.

## IDE
As a Windows 10 user I was happy to have **[WSL (Windows Subsystem for Linux)](https://learn.microsoft.com/en-us/windows/wsl/about)** part of my workflow. It enabled me to run IDE on my main OS and execute code on Ubuntu. It is a very neat combination, especially if you want to avoid subtle differences between your code in development and production.

Unfortunately when I upgraded to Windows 11 I found [performance issues](https://twitter.com/MikolajKania/status/1695742269138026533) that were blocking me from using Intellij IDEA. The main reason was an upgrade, but JetBrains was slow to fix their part - it was resolved after few weeks in one of minor versions. It was so painful, that I was seriously considering giving VS Code a shot.

Before coming back to my original setup, I tested a few wild options like [running IDE directly on Ubuntu](https://towardsdev.com/the-complete-guide-to-using-wsl-in-jetbrains-ides-dd45d354f5e5) or connecting to it via [Gateway](https://www.jetbrains.com/remote-development/gateway/). I hope JetBrains will prioritize their support for WSL, because overally it a very capable tool I would happily continue to use. 

## Coding assistance
I am not recommending code generation for every developer - you should code as much as possible yourself, especially during early days of your career - but it can be a good support when used wisely.

Last year I tried [GitHub Copilot](https://github.com/features/copilot) (before version X was introduced) and it was encouraging; only the integration in JetBrains products could have been better. [Amazon CodeWhisperer](https://aws.amazon.com/codewhisperer/) was too sluggish for my taste and [JetBrains AI](https://www.jetbrains.com/ai/) impressed me until I saw their unappealing pricing on the top of existing subscription.

I ended the year trying **[Codeium](https://codeium.com/)**. It is very similar to JetBrains AI, but for now - free. Coding suggestions, chatbot, features like explaining a file or generating documentation are working great for my personal coding.

# Roles
Leading a team through ups and downs towards a business goals was an extraordinary experience. I was working with great people who I tried to support in their growth. I believe I have put my mark on both technology and processes. 

Every journey has its end, however, and this year I decided to try myself in another role. I became a **Technical Program Manager**, with a task to oversee execution and delivery of the projects, while still keeping an eye on ML technology. It is the biggest change since I decided to switch from backend development to machine learning few years ago and a welcome challenge.

In both roles I helped to deliver features this year and my first project as TPM landed in a [hands of customers](https://www.wolterskluwer.com/en/news/wolters-kluwer-integrates-genai-into-its-legal-research-products). I am looking forward to what 2024 will bring.

# Knowledge vault
Transitions are certainly bringing about new challenges, and this year my attention was occupied by both engineering and project management resources.

## Google Project Management
**[GPM](https://www.coursera.org/professional-certificates/google-project-management)** is a very broad and informative course on project management. I learnt a lot and was immediately able to use new skills in practice. I ended the course with a [certificate](https://coursera.org/share/468cd27fe0d809f226516cbd58c7ec6a) of completion, which was a satisfying milestone. Whereas I can highly recommend it, I'd like to see a follow-up on topic of data-driven approach to executing projects, which was only briefly touched.

## ChatGPT Prompt Engineering for Developers
For better understanding of prompts creation I took **[a short course](https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/)** from famous Andrew Ng. It is a solid intro to a prompting area, but practice and staying on the top of new findings is needed in this very young field.

## Finishing books
**[The Phoenix Project](https://www.oreilly.com/library/view/the-phoenix-project/9781457191350/)** was quite interesting look into a process of transforming company culture from waterfall to agile. It also made DevOps principles famous. I was fortunate to observe such transformation myself in the early years of my career, so the book was not so eye-opening for me. However, there were highlights to keep, for instance how different divisions in one company can see each other and compete without proper communication.

I like Mannings's hands-on books and **[Natural Language Processing in Action](https://www.manning.com/books/natural-language-processing-in-action)** is no different. It is an easy recommendation for everyone interested in NLP field. It explains topics very well and I can only regret it was not extended more into state-of-the-art transformers and LLMs.

**[Deep Learning with Python](https://www.manning.com/books/deep-learning-with-python-second-edition)** is a solid book that organizes deep learning fundaments and extends ideas beyond what can be found in Keras/Tensorflow documentation. I found parts of it useful and well explained.

# Culture
As the last part, let me share a few recommendations from a cultural field. We all need a break! 

## Books
Overview can be found on [Goodreads](https://www.goodreads.com/user/year_in_books/2023/5724806), with [Lord of the flies](https://www.goodreads.com/book/show/7624.Lord_of_the_Flies) (EN) and [Informacja Zwrotna](https://www.goodreads.com/book/show/56469273-informacja-zwrotna) (PL) being my instant classics due to their provocative themes. Audiobooks are helping me to catch-up.

## Movies
Movies of genres ranging from musical, through drama up to animation cemented 2023 as a very [satisfying year](https://www.filmweb.pl/user/MickyThump) for me: [Hamilton](https://www.imdb.com/title/tt8503618/), [Ghosts of Inisherin](https://www.imdb.com/title/tt11813216/), [Oppenheimer](https://www.imdb.com/title/tt15398776/), [Across Multiverse](https://www.imdb.com/title/tt9362722/) or [Pearl](https://www.imdb.com/title/tt18925334/).

## TV Shows
[Succession](https://www.imdb.com/title/tt7660850/) did with its ending what Game of Thrones was not capable of, [Fargo](https://www.imdb.com/title/tt2802850/) is beautifully written and [Severance](https://www.imdb.com/title/tt11280740/) proofs that you can always come up with a new, wild idea in an era of remixes and productions designed by accountants.

## Music
It was a very Polish year for me, as [Dawid Podsiadło](https://open.spotify.com/artist/6EB8VE9f7Ut6NOgviN6gDW), [Taco Hemingway](https://open.spotify.com/artist/7CJgLPEqiIRuneZSolpawQ) and [Lor](https://open.spotify.com/artist/0TwM0vzeyhAMTegVdIq8rx) dominated my 2023. Listening to [autobiography of Bono](https://www.audible.com/pd/Surrender-Audiobook/B09ZK5B962) brought [U2](https://open.spotify.com/artist/51Blml2LZPmy7TTiAg47vQ) back into my radar.

## Concerts
I was fortunate to see live few great bands: [Arctic Monkeys](https://open.spotify.com/artist/7Ln80lUS6He07XvHI8qqHH) (EN, Open'er festival in Gdynia), [Rammstein](https://open.spotify.com/artist/6wWVKhxIU2cEi0K81v7HvP) (DE, for the second time, Chorzów) and [Dawid Podsiadło]((https://open.spotify.com/artist/6EB8VE9f7Ut6NOgviN6gDW)) (PL, Sopot), among others.

# 2024
Thanks for reading to the end and more information coming soon