---
layout: post
title: Calling SAP RFC modules from Python - what you need to remember
published: true
comments: true
excerpt_separator: <!--more-->
tags: Python SAP
---

One of the most popular ways to obtain data from SAP system is OData. To set it up, you need at least basic knowledge about ABAP and SAP. There is another way too using mainly Python.

<!--more-->

Here you need to know what is `function module`. For this example I'll make it very simple. Treat it like a simple function with input and output. That's it 😉.

### What we need

- [SAP NetWeawer RFC SDK](https://support.sap.com/en/product/connectors/nwrfcsdk.html)
- [PyRFC](https://github.com/SAP/PyRFC)
- Some SAP system 🤷‍♀️

Follow [this](http://sap.github.io/PyRFC/install.html) installation steps. I found them very useful. I warn you, it is not easy with Windows 😉.

### SAP Client

First thing is to make connection with system. Remember to check if your IP is authorized to connect with it.

```python
from pyrfc import Connection

con = Connection(
    user=config["user"],
    passwd=config["password"],
    mshost=config["mshost"],
    msserv=config["msserv"],
    sysid=config["sysid"],
    group=config["group"],
    saprouter=config["saprouter"],
    client=config["client"],
)
```

I like to pass all configuration as a dictionary and keep it in env variables or another file.

### GET DATA

The whole idea is to trigger the FM (function module) as I mention. In SAP standard lib there is FM called `RFC_READ_TABLE`. We can use it to make a select statement to database.

Use method `call` of `Connection` object. The first parameter is the FM name. In this case it is `RFC_READ_TABLE`. Next parameters are the same as in interface in FM. You can check it in `se37` T-code in SAP.

```python
response = con.call(
    fm_name,
    QUERY_TABLE=table_name,
    DELIMITER=delimiter,
    FIELDS=fields,
    OPTIONS=options,
    ROWCOUNT=rowcount,
    ROWSKIPS=rowskips,
)
```

Display the obtained data and play with string splitting a bit 😅.

Nice to remember:

- All rows from table are single strings. Use delimiter that fits in your code and split them
- To know the exact type of fields you can use FM `DDIF_FIELDINFO_GET`
- `RFC_READ_TABLE` has a problem with wide tables. Using `DDIF_FIELDINFO_GET`, you can check how many fields such table has, which are the keys and download them in chunks.
- It always good idea to limit fields in select statement. Not only in `RFC_READ_TABLE`. Use parameter FIELDS to pass a list with fields names which you want to obtain.

You can use PyRFC to trigger your custom FM too. For example, you can make some python script which will invoke some workflows in SAP etc.

Do you use PyRFC? If you know more good practices in calling FM from Python, write them down in a comment 😉
