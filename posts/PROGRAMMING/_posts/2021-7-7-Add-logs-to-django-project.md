---
layout: post
title: Django logging - forget about 'print' when debugging
published: true
comments: true
excerpt_separator: <!--more-->
tags: Python Django
---

Let the first one cast the stone who didn't use the print statement to debug and forgot to remove it after! What if I tell you that you can build your 'prints' system while developing? Setting logging in Django is quite simple. I've summed up Django [docs](https://docs.djangoproject.com/en/3.2/topics/logging/) about logging in form of a quick snippet. There are a few logging ideas that help me a lot. Hope you find it useful.

<!--more-->

### Quick setup

Go to your `settings.py` and prepare the `LOGGING` dict. This is the basic configuration that you will probably see in your console.

```python
LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,
    "handlers": {
        "console": {"class": "logging.StreamHandler"},
    },
    "loggers": {
        "django": {
            "handlers": ["console"],
            "level": "INFO",
        },
    }
}

```

To add some logs in your console.

```python
import logging
logger = logging.getLogger('django')

logger.info('here goes your message')
```

### Loggers

This is our log entry point. It consists of a logger handler, with some configuration, etc. Get instance of it with 'logging.getLogger(name)' method.
To add a log you need to choose one of the following log levels.

- DEBUG
- INFO
- WARNING
- ERROR
- CRITICAL

All those levels are represented as a method of your logger instance. For example 'logger.critical('message')' will add log with level CRITICAL.

### Handlers

So handlers are the next step in the logging process. When we define a logger in settings.py, we attach a list of handlers to it. Handlers are basically some kind of mechanizes to handle our logs. In the above exercise, I defined a console handler to write logs into the console. You can define multiple handlers for loggers. With the below configuration, Django logs will be saved to file too.

```python
LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,
    "handlers": {
        "console": {"class": "logging.StreamHandler"},
        "file": {
            "level": "INFO",
            "class": "logging.FileHandler",
            "filename": "logs/platform.log",
        },
    },
    "loggers": {
        "django": {
            "handlers": ["console", "file"],
            "level": "INFO",
        },
    }
}

```

### Formatters

This part is used only to format logging messages. Since those logs base on a python built-in logging module, you can use its [attributes](https://docs.python.org/3/library/logging.html#logrecord-attributes).
With this configuration, my logs will be printed to console with the short form and to file with some more details.

```python
LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "verbose": {
            "format": "{levelname} {asctime} {module} || {message}",
            "style": "{",
        },
        "simple": {
            "format": "{levelname} {message}",
            "style": "{",
        },
    },
    "handlers": {
        "console": {"class": "logging.StreamHandler","formatter": "verbose",},
        "file": {
            "formatter": "verbose",
            "class": "logging.FileHandler",
            "filename": "logs/platform.log",
        },
    },
    "loggers": {
        "django": {
            "handlers": ["console", "file"],
            "level": "INFO",
        },
    }
}

```

### Filters

Filters are attached to handlers. For example, you can filter logs with level DEBUG to be printed to console. Defining your own filter is also possible.

```python
LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,
    "filters" : {
        "debug_only" : {
            "()" : "django.utils.log.RequireDebugTrue",
        },
    },
    "handlers": {
        "console": {"class": "logging.StreamHandler"},
        "file": {
            "filters": ["debug_only"],
            "class": "logging.FileHandler",
            "filename": "logs/platform.log",
        },
    },
    "loggers": {
        "django": {
            "handlers": ["console", "file"],
            "level": "INFO",
        },
    }
}

```

### Django built-in loggers and handlers

There are a few [Django loggers and handlers](https://docs.djangoproject.com/en/3.2/topics/logging/#django-s-logging-extensions) that you can use. Consider them instead of building your own.

### Some handler Ideas

- I made a separate logger for events. Every time event is posted, it helps me to track the order of events in the system. In the end, you can generate some kind of event flow specification with it.
- If your app is using some API you can add a logger in class where requests are handled. Simply track every request in a separate file.
- Prepare handler for critical errors. Maybe send them via email to Admin?
- On my last project, I was sending a lot of emails. You can make a separate file with the addressee and title message.
