---
layout: post
title: How Python defaultdict made my day  
published: true
---

### Everything starts from the problem 

And I have already solve this problem! I had to map some git repo folder structure based on github API response, to python dict with structure like: 

```json
{
    "folder_1_lvl_1" : {
        "folder_1_lvl_2" : {
            "file_1" : "some_file_name_1",
            "file_2" : "some_file_name_2",
            "file_3" : "some_file_name_3"
        },
        "folder_2_lvl_2" : {
            "file_1" : "some_file_name_1",
            "file_2" : "some_file_name_2",
            "file_3" : "some_file_name_3"
        }
    },
    "folder_2_lvl_1" : {
        "folder_1_lvl_2" : {
            "file_1" : "some_file_name_1",
            "file_2" : "some_file_name_2",
            "file_3" : "some_file_name_3"
        },
        "folder_2_lvl_2" : {
            "file_1" : "some_file_name_1",
            "file_2" : "some_file_name_2",
            "file_3" : "some_file_name_3"
        }
    }
}
```
Seems quite simple but what was my api response code (Json below)? It's only some part of structure above. I hope you will get the point. The main difference is type on dict in "tree" list. You can see that three is representing some folder and blob is some file. First thing is to filter this list, next is to know folder structure for every file (blob here), we have a "path" 🤷‍♀️. Let's do it simple. `` path_list = file['path'].split('/') `` .  

```json
{
    "sha": "some_sha",
    "url": "some_url",
    "tree": [
        {
            "path": "folder_1_lvl_1",
            "mode": "040000",
            "type": "tree",
            "sha": "another sha",
            "url": "another url"
        },
        {
            "path": "folder_1_lvl_1/folder_1_lvl_2",
            "mode": "040000",
            "type": "tree",
            "sha": "sha",
            "url": "url"
        },
        {
            "path": "folder_1_lvl_1/folder_1_lvl_2/file_1",
            "mode": "040000",
            "type": "blob",
            "sha": "sha",
            "url": "url"
        },
    ]
}

```

### Little extra background 

One thing extra before we get to the point. I assumed that folder structure will corespond to structure like: 

```txt
    semester_1 
        | - course_1
                | - lab_1
                      | - excercise_1    <- it is file already
                      | - excercise_2    
                      | - excercise_3    
                | - lab_2
                | - lab_3
        | - course_2
        | - course_3

```
So imagine the list of dictionaries with fields like *semester*, *course*, *lab*, *excercise*. Let's make it one dict for backend response 😎


### Old way that worked

Yes, it worked and I had no complaints about it until I got to know another approach!   

```python 
def group_courses(files):
    result = {}
    
    for file in files:
        if file["course"] not in result.keys():
            result[file["course"]] = {}

        if file["lab"] not in result[file["course"]].keys():
            result[file["course"]][file["lab"]] = {}

        if file["excercise"] not in result[file["course"]][file["lab"]].keys():
            result[file["course"]][file["lab"]][file["excercise"]] = {
                "file" : file["excercise"],
                "semester": file["semester"],
                "lab" : file["lab"],
                "course" : file["course"],
            }
    return result

```

### DEFAULTDICT from Collections

Imagine you don't have to check wheter there is such key as *course* or *lab*. It would be nice. So check this out🚀! Below example won't raise exception. It will automatically create simple dictionary on level below *defaultdict*. 

```python 
from collections import defaultdict 

file_dict = defaultdict(dict)

# won't raise KeyError
file_dict['some_folder_lvl_1']['some_folder_lvl_2']  = {
    "file": "file_name"
}

```

But what if we want go deeper. Imagine ``file_dict['some_folder_lvl_1']['some_folder_lvl_2']['some_folder_lvl_3]``, it raises ``KeyError`` becasue *defaultdict* is only on top level of out dictionary. Defaultdict with no argument acts like dict. We could do `` my_dict = defaultdict(defaultdict(defaultdict( 👾 ))) `` but lol, nope. 

Here comes **recursive defaultdict** 😎. Sounds cool enough for me!  

Let's just tell python to create another level of defaultdict, everytime *KeyError* in basic dict would happen. 

```python 
from collections import defaultdict 

def nested_dict():
    return defaultdict(nested_dict)

# and above function wolud look like this now!
def group_courses(files):
    result = nested_dict()
    
    for file in files:
        result[file["course"]][file["lab"]][file["excercise"]] = {
            "file" : file["excercise"],
            "semester": file["semester"],
            "lab" : file["lab"],
            "course" : file["course"],
        }
    return result
```

And you know what? Result is the same but how much less places to make mistake! How much less if statements that I understood only at the time of writing! I like it a lot! 

If you want some fancy details about default check out [docs](https://docs.python.org/3/library/collections.html#collections.defaultdict). Basicly the idea here is to handle ``KeyError`` and supply dict with some default variable. And here it is even recursive!

****

Main credits for [Duncaster](https://patternite.com/users/d5a991ecf2/duncster) from Patternite as I found this idea in his [article](https://patternite.com/patterns/4ec8658c96/automatically-create-nested-dictionaries-python) first.  