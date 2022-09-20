---
layout: post
title: Django rest framework - basics I wish to know before I started last project
published: false
comments: true
excerpt_separator: <!--more-->
tags: Python Django
---

The idea is to gather all information how to set up API endpoints based on model in Django and create some kind of cheat sheet.

<!--more-->

To make it work in Django, we need three things. Some kind of **Model** with **serializer** and the **View**. Then we plug it to url and voilà! Let's make it step by step.

## Model

Nothing fancy here, just an Article. Skip to the next point 😉.

```python
// models.py
from django.db import models
from django.contrib.auth import get_user_model

UserModel = get_user_model()

class Article(models.Model):
    author = models.ForeignKey(UserModel, on_delete=models.SET_NULL, null=True)
    title = models.CharField(max_length=50)
    text = models.CharField(max_length=100)
```

## Serializer

Serializer is a class that will convert our model instance to JSON and back. I don't want to focus on how it works but below you can find some functionalities of `rest_framework serializers`.

### Do not specify all fields

Use `__all__` when you want to serialize all fields

```python
// serializers.py
from rest_framework import serializers
from .models import Article

class ArticleSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = "__all__"

{
    "id": 1,
    "title": "sample title 1",
    "text": "sample article text",
    "author": 1
}
```

### Exclude

If you want to exclude single field (ex. password) you can use `exclude = [list of fields to exclude ]`

```python
class ArticleSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        exclude = ["author"]

{
    "id": 1,
    "title": "sample title 1",
    "text": "sample article text",
}
```

### Depth

Response is supplied with author PK. If we want to get more details about fields specified as foreign key, use `depth = 1`.
Then it will serialize one level more. If author model will have some foreign key field too and depth will be equal to 2, it will be serialized too.

```python
class ArticleSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = "__all__"
        depth = 1

{
  "id": 1,
  "title": "sample title 1",
  "text": "sample article text",
  "author": {
      "id": 1,
      "last_login": "2021-06-14T07:19:30.612463Z",
      "is_superuser": true,
      "username": "admin",
      "first_name": "",
      "last_name": "",
      "email": "admin@example.com",
      "is_staff": true,
      "is_active": true,
      "date_joined": "2021-06-14T07:13:58.546066Z",
      "groups": [],
      "user_permissions": []
  }
}
```

### Reverse Serialization

Imagine that you want to make serializer for User model and know what Articles he wrote. Since User Model hasn't any field like 'articles' and Article model has relation to User Model in 'author' field, we can specify fields in User Model serializer like:

```python
 fields = ("id", "all other fields", "article_set" )
```

For better understanding try it out in the Django shell.

```txt
>>> from django.contrib.auth import get_user_model
>>> UserModel = get_user_model()
>>> admin = UserModel.objects.all( )[0]
>>> admin
<User: admin>
>>> admin.email
'admin@example.com'
>>> admin.articles_set.all( )
Traceback (most recent call last):
  File "<console>", line 1, in <module>
AttributeError: 'User' object has no attribute 'articles_set'
>>> admin.article_set.all( )
<QuerySet [<Article: Article object (1)>, <Article: Article object (2)>, <Article: Article object (3)>]>
```

## Views

To generate views for the model I will use [generic views](https://www.django-rest-framework.org/api-guide/generic-views/).

To make it work we need to create a class which will inherit from `GenericViewSet` and classes that implements methods POST, GET, PUT, PATCH, DELETE.

```python
from rest_framework.mixins import RetrieveModelMixin, ListModelMixin, UpdateModelMixin, CreateModelMixin, DestroyModelMixin
from rest_framework.viewsets import GenericViewSet


class ArticleView(GenericViewSet, ..... ):
    serializer_class = ArticleSerializer
    queryset = Article.objects.all()

```

Don't forget to add `serializer_class` field and `queryset` field. Alternatively you can define `get_queryset( )` method or `get_serializer_class( )`.

### List View

Adds method GET to our View to list all resources (queryset). To override or extend behavior use: `.list(request, *args, **kwargs)` method.

```python
GET localhost:8000/api/articles
```

```python
// views.py
class ArticleView(GenericViewSet, ListModelMixin):
    serializer_class = ArticleSerializer
    queryset = Article.objects.all()
```

### Detail View (Retrieve view)

Adds method GET to our View which provides detail information about single entity. To override or extend behavior use: `.retrieve(request, *args, **kwargs)` method.

```python
GET localhost:8000/api/articles/{article pk}
```

```python
// views.py
class ArticleView(GenericViewSet, RetrieveModelMixin):
    serializer_class = ArticleSerializer
    queryset = Article.objects.all()
```

### Update View

Adds two methods to our View. PUT and PATCH. Use PUT request to update all fields and PATCH request as partial update. To override or extend behavior use: `.update(request, *args, **kwargs)" method or ".partial_update(request, *args, **kwargs)` for PATCH.

```python
PUT localhost:8000/api/articles/{article pk}
body: data to update as json representation of obj.

PATCH localhost:8000/api/articles/{article pk}
body: data to update as json with fields we want to update
```

```python
class ArticleView(GenericViewSet, UpdateModelMixin):
    serializer_class = ArticleSerializer
    queryset = Article.objects.all()
```

### Create View

Adds method POST to view. To override or extend behavior use: `.create(request, *args, **kwargs)` method.

```python
POST localhost:8000/api/articles
body: json representation of obj to create
```

```python
class ArticleView(GenericViewSet, CreateModelMixin):
    serializer_class = ArticleSerializer
    queryset = Article.objects.all()
```

### Destroy View

Adds method DELETE to view. Provides deletion of model instance. To override or extend behavior use: `.destroy(request, *args, **kwargs)` method.

```python
DELETE localhost:8000/api/articles/{article pk}
```

```python
class ArticleView(GenericViewSet, DestroyModelMixin):
    serializer_class = ArticleSerializer
    queryset = Article.objects.all()
```

---

I will update this post when I will find something noteworthy.
