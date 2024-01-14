---
layout: post
title: 2023, My year in review
---

Looking back into learnings, changes and... culture.

<!--excerpt-->

# Engineering
When you are joining an industry to develop as machine learning specialist, you envision yourself a problem solver of endless stream of projects. The thing is, that whereas it is important to sharpen data science skills, equally important it keep engineering side taken care of. Being backend & search engineer in the past helped me to embrace this truth and in 2023 I had my tooling set and predictions confirmed by experience.

## MLOps
**AWS Sagemaker** processing jobs and pipelines is the answer to the need to build high-quality services. Their advantages are vast: trackable executions and outcomes, resources allocation per intermediate task, reproducibility, caching, ability to retroactively review the runs. It also helps to separate a code of a Python package from a script that is running it in the cloud. It is not without its issues: jobs are sometimes hanging, convenient metrics tracking at the time of using was irritating, but overally it is an option worth exploring.

My journey in choosing backbone experimentation runner was involving DVC, Kedro, Prefect, Flyte and ordinary Python scripts. My learning was to keep it simple as long as possible, as every tool is adding additional complexity to the coding environment. Still, I definitely can imagine Prefect as a workflow manager in advanced pipelines.

## Tracking
Metrics tracking is subpar in a native Sagemaker, too limited in DVC Experiments and I didn't fully click with industry standard - mlflow; still it was a good tool that I would be comfortable using.

I found Weights & Biases well designed and approachable. What I like about W&B is that their integrations are well-thought and you are not forced to use all of them since the beginning, which was my impression with mlflow. You can pick and choose functionalities as your project grows and I ended up experimenting with not only tracking, but also their models storage. I really liked how easy it is to run it locally for experimentation and go online for finalized runs. There is also an Optuna callback for W&B, which makes it easier to inject hyperparameter tuning into your workflow.

Comet looked like an interesting competitor to W&B, but its offline functionality was limited and number of features at the same price - lower. Still, I count on them to provide competition in this field.

## Environment
As a Windows user I was happy to have WSL2 part of my workflow. It enabled me to use tools on my main OS and execute code on Ubuntu - very neat combination. Unfortunately when I upgraded to Windows 11 I found performance issues that I initially a attributed to Intellij IDEA. After digging, it turned out that biggest issue is related to the upgrade. It was a setback, as I was happy with the newest Windows, I was happy with my environment and workflow, but now needed to explore how to change it.

Issues with Windows and JetBrains being much too slow to fix their own issue, lead me to try wild options, like running Intellij IDEA from Ubuntu directly - it works pretty ok, but is a big buggy and is not supporting enough features like debugger or Jupyter Notebooks. In the end I am **sharing coding between Windows and WSL2/Ubuntu**, taking the convenience of tools working natively on my main OS and trying to have as much as possible run on Linux. Fingers crossed that Intellij IDEA/ PyCharm will be prioritized by JetBrains as it is getting harder to justify paying for an individual license.

## Coding assistance
Popularity of GPT is visible also in a coding area. I am not recommending code generation for every developer - you should try to write as much as possible yourself during early days of your career - but it can be a good help.

Last year I tried GitHub Copilot (before version X was introduced) and it was encouraging; only the integration in JetBrains products could be better. AWS CodeWhisperer was too sluggish and I was not happy with an experience. AI Assistance from JetBrains was impressive, well integrated and was adding chat in the UI, but the pricing on the top of current pricing is really bad.

I ended the year by trying **Codeium** and I am really impressed. It is similar to AI Assistance in IDEA, but - for now - free and capable. Coding suggestions, chat, features like explaining a file or generating documentation are great. For now it is an easy recommendation for private projects.

# Roles
Leading a team through ups and downs towards a business goals was an extraordinary experience. I was working with a great people who I tried to support in their growth. I believe I put my mark on technology and processes, especially on team management side where I introduced good practices like very open and trackable 1:1s (an inspiration from XXX).

Every journey has its end, however, and this year I decided to try myself in another role. I became a **Technical Program Manger**, with a task to oversee execution and delivery of the projects, while still keeping an eye on technology. It is the biggest change since I decided to switch from Java/backend into Python/ML few years ago. Now I am completing my overview of projects lifecycle, being always interested in more than just pure coding. Still, as you can see above I am enjoying it after a short sabbatical!

In both roles I helped to deliver features this year and my first project as TPM landed in a hands of customers. Looking forward to what 2024 will bring.

# Knowledge vault
Transitions are always creating new challenges, and this year my attention switched to non-engineering knowledge extensions as well.

## Google Project Manager specialization
A very broad and informative course on project management. I learnt a lot and hands-on tasks helped to put it in practice. I was doing it during transition period, learning on a topics I was immediately applying in a real life. I ended a course with a completion certificate. Whereas I can highly recommend it, I'd like to see a follow-up on topic of data-driven approach to executing projects, which was just briefly touched.

## GPT prompting - Deep Learning AI
GPT took the world by storm. To better understanding prompt engineering I took the short course from famous Andrew Ng. It is a solid intro to the matter, but practice and following trends is needed in this very young field.

## Finishing books
Project Phoenix was a quite interesting look into how a whole company can be transformed to be more agile. It also made DevOps culture a topic understandable for managers. I was happy to observe such transformation myself in an early years of my career, so the book was not so eye-opening for me. However, there were highlights to keep: for instance how different division on a company can see each other and compete without proper communication.

Introduction to Keras & Tensorflow is a solid book that organize and extend ideas beyond what can be found in documentation. I found parts of it useful and well explained, but the more you are experience the less you need such read.

NLP is a an easy recommendation for everyone interested in NLP field. It explains topics very well and I can only regret it was not extended more into state-of-the-art transformers and LLMs. As with Keras, it structures the knowledge that would be fragmented otherwise.

# Culture
As the last part, let me share few recommendation from a cultural field.

## Books
Overview can be found on [Goodreads](https://www.goodreads.com/user/year_in_books/2023/5724806), with Lord of the flies (EN) and Informacja Zwrotna (PL) being my instant classics due to their provocative themes.

## Music
It was a very Polish year for me, as Dawid Podsiadło, Taco Hemingway and Lor rocked my 2023. Listening to autobiography of Bono brought U2 back into my radar.

## Concerts
I was happy to see a few great bands: Arctic Monkeys (EN, Open'er festival in Gdynia), Rammstein (DE, for the second time, Chorzów), Dawid Podsiadło (PL, Sopot).

## Movies
Genres from musical, through drama up to animation cemented 2023 as a very [satisfying year](https://www.filmweb.pl/user/MickyThump) for me: Hamilton, Ghosts of Inisherin, Oppenheimer, Across Multiverse.

## TV Shows
Succession did with ending what Game of Thrones was not capable of with its ending, Fargo is beautifully written and Severance is a proof that you can always came up with a new, wild idea in an era of remixes and productions designed by accountants.

# 2024
More to come and share soon!