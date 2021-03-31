---
layout: post
title: Todoist Console Client - the beginig 
published: false
---


## Why? 

This time I want to show one of the current projects I'm working on. It's quite simple but very usefull for me. It's console client for [todoist](https://todoist.com/) task managment app. Whole idea comes from one reason. 

Too many open tabs in browser -> Let's close todoist -> don't look at tasks -> don't do tasks😒. 

First think that came to my mind was a little widget on desktop but let's be honest.. I would close it too. So where I spend most of my time working? VS code and console! The choice was simple. Almost as quick as checking ``git status`` I can type ``todoist`` and look at my tasks 🎉. 

## Take a look! 

![todoist_photo](https://github.com/JakubSzwajka/JakubSzwajka.github.io/blob/master/_posts/_images/todoist_1.png?raw=true)

## Let's clear something up 

The goal was not to implement all todoist API functions in console. The goal was to build the fastest access to basic functions like *show tasks*, *add task*, *complete task*. I'm not going to describe all work here now, instead I'll try to show you what you can learn from such simple 'app' 😉. 

## Connection

I have used todoist [Sync API](https://developer.todoist.com/sync/v8/) with python. It's quite simeple, all you need to have is your user token. Find it in your settings -> integrations -> token. Base on it create client object which will make all magic for you ✨.  

```python
    from todoist.api import TodoistAPI

    client = TodoistAPI('yourSuperSecretTokenGoesHereAsAString')
    client.sync()
```

And since then you have all your informations up to date.

## Config 

I have prepared some config file. So far you can set yor token and emojis there but I hope it will grow a bit. You can set your token by passing it to Json file or I have prepared a command for you 🎀. It will load config, change it and save it again. 

*  **to set the token** use ``` todoist --set-token your_token```

## Make it work! 

Below are some basic commands. Remeber that I'm still developing this project in my free time and you will probably see some difference. The most up-to-date version will be on [github](https://github.com/JakubSzwajka/todoist_console_client) so check out readme file. 

*  **to get tasks** that are/were for today and those overdue use ``` todoist ```
*  **to add task**  with default date value as current day, you don't have to use --date flag. In case you want to specify date use YYYY-MM-DD format. ```todoist --add "name of your task" --date yyy-mm-dd```
* **to complete task** ```todoist --complete id_of_task```
