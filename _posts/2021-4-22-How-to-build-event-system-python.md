---
layout: post
title: Build your own event system in python 
published: true
excerpt_separator: <!--more-->
---

Event system is not hard to build on your own. There are plenty of libraries ready to use, but for better understanding I want to implement it by myself. 

<!--more-->

The idea of this post comes from [this](video). Today I was searching the internet for tips how to manage my project which has a lot of different modules and I wanted to do it nice and clean. One of the ideas I found is [Observer](https://en.wikipedia.org/wiki/Observer_pattern) pattern from “Gang of four” book. This should be very simple implementation of it.   

First thing is that you have subscribers. They subscribe to different types of events. Every time the event is posted, you have to notify the subscriber about it. 

First create two methods for subscribing the event, and for posting.  

```python 
# src/event.py

from collections import defaultdict

subscribers = defaultdict(list)

def subscribe(event_type, fn):
    subscribers[event_type].append(fn)

def post_event(event_type, data): 
    if event_type in subscribers:
        for fn in subscribers[event_type]:
            fn(data)
```

In this example I will be creating the new user which will invoke event ``new_user_created``. Subscriber for this event will be module for sending welcome message by email. 

Let's create a user. It will be simple dictionary, but it can be an object too. 

```python 
# src/user.py

from .event import post_event

def register_new_user(name, password, email ):
    user = dict( name = name, 
                password = password, 
                email = email)

    post_event("new_user_created", user)

```
Now handle sending emails. We will need two things here. Some kind of email provider and some kind of handler for it. I want my email provider class to handle all 'business logic' only, so I created ``modules`` folder for such. 

```python 
# src/modules/email.py 

class Email:
    def sendEmail( email, subject, message ):
        print("==========================")
        print(f"From: {email}")
        print(f"Subject: {subject}")
        print(message)
        print("==========================")
```

And handler for email.
```python
# src/email_handler.py

from .modules.email import Email 
from .event import subscribe

def handle_user_registered_event(user):
    Email.sendEmail(user['email'], 'Welcome!', 'Some welcome message')

def setup_email_event_handlers():
    subscribe('new_user_created', handle_user_registered_event)
```

Now let's connect everything together. Make an app file. 

```python 
# src/app.py

from src.user import register_new_user
from src.email_handler import setup_email_event_handlers

setup_email_event_handlers()

register_new_user('Jakub', 'secret', 'name@domain.com')

```

First thing to do is to set email handler events up. That means, subscribe to event ``new_user_created``. 

If all the subscriptions are done, register new user. Output should be as follows. 

```txt
==========================
From: name@domain.com
Subject: Welcome!
Some welcome message
==========================
```

### Extend the system 

What if we want to extend this. Let's add some database. In modules, add ``database.py`` file and sample implementation of database as a list of users. 

```python 
# src/modules/database.py 

class DB:
    users = [ ]
    def register_new_user( user ):
        DB.users.append(user)
        print('=======================')
        print(DB.users)
        print('=======================')

```

And handler for it. 
```python 
# src/database_handler.py

from .modules.database import DB
from .event import subscribe

def handle_user_registered_event(user):
    DB.register_new_user(user)

def setup_database_event_handlers():
    subscribe('new_user_created', handle_user_registered_event)
```

Now the only thing to do is set database handler up in app file. 

```python 
# src/app.py

from src.user import register_new_user
from src.email_handler import setup_email_event_handlers
from src.database_handler import setup_database_event_handlers 

setup_email_event_handlers()
setup_database_event_handlers()

register_new_user('Jakub', 'secret', 'name@domain.com')
```

Now the result should be: 
```txt
==========================
From: name@domain.com
Subject: Welcome!
Some welcome message
==========================
==========================
[{'name': 'Jakub', 'password': 'secret', 'email': 'name@domain.com'}]
==========================
```

Nice thing about it, is that this way keeps your modules independent of each other and keeps their cohesion strong. Besides that, it allows you to manipulate order of listeners in event very quickly. No more 'ctrl + c | ctrl + v' whole code 😉.
