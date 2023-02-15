---
layout: post
title: Adding new django models field without errors
date: 2023-02-15 15:02:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: python-django.png # Add image post (optional)
tags: [django, web-dev, python, backend] # add tag
---


Have you ever tried to add a new model field in an existing model that already has data?

Yes?

You will always get this problem:

```powershell
(myproject) PS C:\Users\my_project> python manage.py makemigrations
It is impossible to add a non-nullable field 'item4' to table without specifying a default. This is because the database needs something to populate existing rows.
Please select a fix:
 1) Provide a one-off default now (will be set on all existing rows with a null value for this column)
 2) Quit and manually define a default value in models.py.
Select an option:
```


The next question is, how do you solve it?

## How to Solve you are trying to add a non-nullable field to without a default

Lets say you/I have this model

```python
class test(models.Model):
    item1 = models.CharField(max_length=100)
    item2 = models.CharField(max_length=100)
    item3 = models.CharField(max_length=100)
    
    def __str__(self):
        return self.item1
```

You now want to add a new field to this model ```item4```.

```python
class test(models.Model):
    item1 = models.Charfield(max_length=100)
    item2 = models.CharField(max_length=100)
    item3 = models.CharField(max_length=100)
    item4 = models.CharField(max_length=100)
    
    def __str__(self):
        return self.item1
```

When now you try to ```makemigrations```, you will get the issue.

output:
```
You are trying to add a non-nullable field 'item4' to table without a default; we can't do that (the database needs something to populate existing rows).
```

How can you solve this?

1. **Adding ```null=True``` to the field**
Code:
```python
# The new field
item4 = models.CharField(max_length=100, null=True)
```

This fixes the error. You can now ```makemigrations``` and ```migrate``` successfully.

2. **Adding ```default=''``` to the field**
This is almost the same as the previous one, only that here we use default keyword to set the default value.
code:
```python
item4 = models.CharField(max_length=100, default='test')
```

3. **Deleting and recreating the model**
Here, you need to delete your model class in models.py then migrate your app models. After migrations, rewrite back the model and migrate the model.
```
Delete Model -> Makemigrations Models -> Migrate Models -> Rewrite Back the updated model -> Makemigrations Models -> Migrate Models
```

> **NOTE**: *Your model data will be lost*
