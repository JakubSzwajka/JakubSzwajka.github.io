---
layout: post
title: title to fill
published: false
comments: true
excerpt_separator: <!--more-->
---

This project was definitely a game changer for me. From newbie waiting for specification to quite aware developer? Maybe 🤷‍♀️. Definitely a game changer.

<!--more-->

I want to sum my thoughts up as a list/checklist/guidebook for me or you. Don't know if those advices will suit you. Maybe they are project dependent. I was working as a single developer. Starting from backend only and ending as full stack. Consider it reading this list.

### Details, details, details

It cost me the most time. You are obligated to deliver something, you obtain some 'specification'. Hurray! `django-admin startproject project` and? You will probably make something, then somebody will tell you some more details about the project, then you will go `python manage.py startapp anotherapp`, and finally you will end up having mess in your project.

My solution is to draw this project. Create some mind map starting from what is included in specification. Find the users. What they can do in this system? Why they are here? What are the main functions. Probably you will end up having lots of questions. I did. You have the answers? Go draw it again with new information. After a few iterations you have unanswered questions? Meh, 'coding fun part begins'. [_A good architecture allows you to defer framework decisions..._](https://twitter.com/unclebobmartin/status/118403913937453056).

### Automate — save your time

Two important things. Tests and Deploy. Did you ever forget to change global 'BACKEND_ADDRESS' variable from 'localhost' to some IP and notice it after 10 minutes build? 🙂 I did a lot. There will be always something more important than setting environmental variables. But just set it and forget. So simple and yet difficult.

Tests where probably the only good decision made at the beginning. It is not the case now to tell you how awesome they are. It was my first project where I tried go full TDD. And it worked. Just remember to write this dummy test that fails on the beginning. Still simple and yet difficult.

### FB or IG when docker compose up?

Take care about your container. Make it as small as you can. During this project I shortened build time from 12 minutes to 5minutes. `6 minutes * 2 builds a day * 30 = 6hours` 🙂. 6 hours a month lost waiting for a build to finish just to figure out that 'BACKEND_ADDRESS' = 'localhost:8000'.

Is it only me?

### Imagine your code.

I've tried to keep everything as decoupled as I could from the very beginning. But in the end, I saw `python cannot import (most likely due to a circular import)` a few times. Simple walk around and move on. Meh, that's not the way I like it.

I spent a week searching for ways to organize everything. And here comes design patterns. I can't tell you if I choose the right one, but the observer pattern did the job. I wrote about it [here](https://jakubszwajka.github.io/How-to-build-event-system-python/). Even if you will choose the bad one, it will probably make your code more clean and organized. It will be easier to change it later. Think about it 😉.

### Commit and deploy as often as you can.

This is my weak point. But it is hard when you work alone. Single commit has 10files changed, 5 removed and a few added? And last 5 commits had message 'bug fixes / new features'🙂. Now go find there something. Good luck.

Despite this, deploying as often as I can do the job. You can show your changes more often, so it is easier to find out that you are on the wrong path.
Automate tests, automate builds and deploys and maybe automate getting feedback?🤔

###

Maybe there is revealing for you. For me, this knowledge is a game changer. What do you think about it? Do you have any advice to add to this list?
