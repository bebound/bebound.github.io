+++
title = "How to disable auto strip in Charfield in Django"
date = 2021-12-19T21:20:00+08:00
lastmod = 2025-07-18T19:07:21+08:00
tags = ["Python", "Django"]
categories = ["Programming"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

In Django, when edit field in admin page or post data to forms, the leading and tailing whitespace in `CharField` and `TextField` are removed.

The reason is `strip=True` parameter in `forms.CharField`, which is added in Djagno 1.9. You can see the discussion in [django tiket #4960](https://code.djangoproject.com/ticket/4960) and here is [source code](https://github.com/django/django/blob/4ce59f602ed28320caf3035212cb4d1c5430da2b/django/forms/fields.py#L211). `models.CharField` and `models.TextField` use `formfield()` to create form to interact with user, then both of them eventually create a `forms.CharField`

It only affect the value return from forms, you can still update model manually and calling `save()` to save it with spaces.

Normally, this feature help us to keep text field clean. But sometimes you may want to get the original value, and here are three different solutions:

Suppose we have this `Test` model.

```python3
# models.py
class Test(models.Model):
    char = models.CharField(max_length=20)
    text = models.TextField()
```


### Change ModelAdmin {#change-modeladmin}

```python3
# admin.py
TestAdmin(admin.ModelAdmin):
    def formfield_for_dbfield(self, db_field, request, **kwargs):
        if db_field.name in ['char', 'text']:
            kwargs['strip'] = False
        return super().formfield_for_dbfield(db_field, request, **kwargs)
```

This method tackles the problem by overriding fields' default `fromfiled` method.


### Define Custom Form {#define-custom-form}

```python3
# forms.py
class CustomForm(forms.ModelForm):
    char = forms.CharField(strip=False)
    text = forms.CharField(strip=False, widget=forms.Textarea)

    class Meta:
        model = Test
        exclude = []

# admin.py
TestAdmin(admin.ModelAdmin):
    form = CustomForm
```

Now when edit data in admin panel, the whitespace is not removed anymore.


### Use Custom Field {#use-custom-field}

You can also use your custom field in `models.py`. For example:

```python3
# models.py
from django.db.models import TextField


class NonStrippingTextField(TextField):
    def formfield(self, **kwargs):
        kwargs['strip'] = False
        return super(NonStrippingTextField, self).formfield(**kwargs)

class Test(models.Model):
    text = NonStrippingTextField()
```

`==`


## REST Framework {#rest-framework}

If you use Django REST framework to edit data, you only need to change the serializer.

```python3
class TestSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Test
        fields = '__all__'
        extra_kwargs = {"char": {"trim_whitespace": False},
                        "text": {"trim_whitespace": False}}
```


## Ref {#ref}

1.  [StackOverflow - Django TextField and CharField is stripping spaces and blank lines](https://stackoverflow.com/questions/38995764/django-textfield-and-charfield-is-stripping-spaces-and-blank-lines)
2.  [Djanog - TextField constructor needs a strip=False option](https://code.djangoproject.com/ticket/30077)
3.  [StackOverflow - In Django REST control serializer does not automatically remove spaces](https://stackoverflow.com/questions/50019009/in-django-rest-control-serializer-does-not-automatically-remove-spaces)
4.  [Allow Whitespace to be a Valid CharField Value in Django Admin](https://www.aaronoellis.com/articles/allow-whitespace-to-be-a-valid-charfield-value-in-django-admin)
