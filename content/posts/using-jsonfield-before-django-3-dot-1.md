+++
title = "Using JSONField before Django 3.1"
author = ["KK"]
date = 2021-09-11T21:12:00+08:00
lastmod = 2021-09-11T21:15:12+08:00
tags = ["Python", "Django"]
draft = false
noauthor = true
nocomment = true
nodate = true
nopaging = true
noread = true
+++

In Django 3.1, Django support save python data into database as JSON encoded data and it is also possible to make query based on field value in JSONField. The detailed usage can be found [here](https://docs.djangoproject.com/en/3.2/topics/db/queries/#querying-jsonfield). If you are using older version and want to try this feature. Though there are many packages ported this function, I recommend [django-jsonfield-backport](https://github.com/laymonage/django-jsonfield-backport).


## django-jsonfield-backport {#django-jsonfield-backport}

This package save data as JSON in database and also support JSON query. If your database meet the requirements (MySQL > 5.7, PG > 9.5, MariaDB > 10.2 or SQLite > 3.9 with [JSON1](https://docs.djangoproject.com/en/3.1/ref/databases/#sqlite-json1) extension), you can use JSONField like Django's native implementation.

```python3
from django.db import models
from django_jsonfield_backport.models import JSONField

class ContactInfo(models.Model):
    data = JSONField()

ContactInfo.objects.create(data={
    'name': 'John',
    'cities': ['London', 'Cambridge'],
    'pets': {'dogs': ['Rufus', 'Meg']},
})
ContactInfo.objects.filter(
    data__name='John',
    data__pets__has_key='dogs',
    data__cities__contains='London',
).delete()
```


## jsonfield {#jsonfield}

[jsonfield](https://github.com/rpkilby/jsonfield) is another popular package to use JSONField. It will save data as `Text` in database, but you can manipulate field value as python data. In addition, it does not provide JSON querying capability as `django-jsonfield-backport`.


### Django REST framework {#django-rest-framework}

As data is stored as JSON string in database, the output is string rather than object when Django DRF serialize `jsonfield.JSONField`. If you prefer to get and update the data like object, you need to manually specify it as \`serializer.JSONField\` like this:

```python3
from rest_framework import serializers
from .models import Product

class ProductSerializer(serializers.ModelSerializer):
    images = serializers.JSONField()
    class Meta:
        model = Product
        fields = '__all__'
```

(You do not need to do this when using `django-jsonfield-backport`, everything just works.)


## Ref: {#ref}

1.  [GitHub - django-jsonfield-backport](https://github.com/laymonage/django-jsonfield-backport)
2.  [Use JSONField with Django Rest Framework](https://librenepal.com/article/use-jsonfield-with-django-rest-framework/)
3.  [JSONField in serializers – Django REST Framework](https://www.geeksforgeeks.org/jsonfield-in-serializers-django-rest-framework/)
4.  [Django REST framework - jsonfield](https://www.django-rest-framework.org/api-guide/fields/#jsonfield)
