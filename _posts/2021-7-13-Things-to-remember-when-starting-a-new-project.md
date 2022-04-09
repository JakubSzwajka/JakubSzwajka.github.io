---
layout: post
title: Things to remember when starting a new project
published: false
comments: true
excerpt_separator: <!--more-->
---

Last project was definitely a game changer for me. From newbie waiting for specification to a quite aware developer? Maybe. Definitely a game changer.

I want to sum my thoughts up as a little list/checklist/guidebook for me or you. Don't know if those advices will suit you. Maybe they are project dependent. I was working as a single developer. Starting from backend only and ending as full stack. Consider it, reading this list.

<!--more-->

### First planning, then coding

Details cost me the most time. Right after I got some sort of 'specification', I went `django-admin startproject ...`. New functionalities were created immediately. Everyone was happy! Pure land of software happiness. But those days passed quickly. On one of the weekly statuses, we were discussing some new features. I found that everyone in a team sees this feature differently. Wait… what if those previous features I have already implemented were understood differently too? Shit… Huston, we have a problem.

You probably already see this problem, so I will move on to the next one. As I was a single developer here, the next one is my fault for sure. Besides misunderstanding the single features, I found myself in a place where those features were messed up on code level. As soon as I tried to fix part A, part B broke. As soon as B, C broke. Finally, I fix C. Meh… A broke.

Let's sum up this frustrating tornado. How I manage to fix everything. The short answer is I asked as many questions as I could. There were a few which I found stupid, but then I heard "_It's a very good question. We have to discuss it!_". Don't think if it is a good or bad question. Just ask, and you will know more about the system you are supposed to create. There is one more trick I found useful to create those questions. Drawing! To be honest, I don't know UML. I'm usually drawing for myself some kind of mind map.

Start by finding users in your system and go with them step by step. Does your user need to log in? With email or username? (question to ask not mentioned in the specification). I'm going with my user through core functionalities. What he can do and what cannot? Notification? What is the body of notification? The more questions you ask, the more details you know.

Finally, you will end up with quite a big mind map and a much better understanding of what you are supposed to create. This was the moment I saw some patterns in it. I extracted some models, some apps, and views. From this point, everything was easier. But what if there are some questions that can't be answered at this point, and you should start coding? The clock is ticking, and you can't wait for answers. Meh, the coding fun part begins. [_A good architecture allows you to defer framework decisions..._](https://twitter.com/unclebobmartin/status/118403913937453056). I will leave you here with that thought. ;)

### Automate — save your time. A lot!

Two important things. Tests and Deploy. Did you ever forget to change a global `BACKEND_ADDRESS` variable from `localhost` to some IP and notice it after 10 minutes, build completed? I did. There will be always something more important than setting environmental variables. But just set it and forget. So simple and yet difficult.

Tests were probably the only good decision made at the beginning. It is not the case now to tell you how awesome they are. It was my first project where I tried to go full TDD. And it worked. Just remember to write this dummy test that fails in the beginning. Still simple and yet difficult.

I made a habit. Every time I find a bug, I write a test. But only the name, basically. I'm trying to minimize interrupting my current work.

```python
def testShouldDoSomethingWhenUserDoAnotherThing(self):
    self.asserEquals(1,2)
```

I found it enough. Now forget about it and continue your previous work. You will come back to it later because it fails, and if you made a good name for it, you will figure out what was on your mind!

### The faster build, the happier developer.

Take care of your container. Make it as small as you can. During this project, I shortened the build time from 12 minutes to 6minutes. This is a lot!. `6 minutes * 2 builds a day * 30 days = 6hours`. 6 hours a month lost waiting for a build to finish just to figure out that `BACKEND_ADDRESS = localhost:8000`.

Is it only me?

### Design patterns?

I've tried to keep everything as decoupled as I could from the very beginning. But in the end, I saw `python cannot import (most likely due to a circular import)` a few times. Simple walk around and move on. Meh, that's not the way I like it.

I spent a week searching for ways to organize everything. And here comes design patterns. I can't tell you if I choose the right one, but the observer pattern did the job. I wrote about it [here](https://jakubszwajka.github.io/How-to-build-event-system-python/). Even if you will choose the bad one, it will probably make your code more clean and organized. It will be easier to change it later. Think about it.

### Commit and deploy as often as you can.

This is my weak point. But it is hard when you work alone. The Single commit has 10files changed, 5 removed and a few added? And the last 5 commits had the message 'bug fixes / new features'. Now go find there something. Good luck.

Despite this, deploying as often as I can do the job. You can show your changes more often, so it is easier to find out that you are on the wrong path. Automate tests, automate builds and deploys, and maybe automate getting feedback?

Maybe this is not revealing for you. For me, this knowledge is a game changer. What do you think about it? Do you have any advice to add to this list?
