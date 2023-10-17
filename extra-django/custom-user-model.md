# Creating a custom user model in Django

Often we want to add custom fields to Django's user model. This might be something like adding an avatar, or potentially a relationship with another model.

Since Django uses its user model for some internal stuff (mainly authentication), we have replace it with our own model that _extends_ Django's _AbstractUser_ class.

You can create this model in any app in your project. In this example we'll use the `jwt_auth` app:

```py
from django.db import models
from django.contrib.auth.models import AbstractUser
# Create your models here.

class User(AbstractUser):

    # custom fields here...
    image = models.CharField(max_length=200)
```

With this done, we need to tell Django that we want to use this model for the project's user. In `project/settings.py` add the following:

```py
AUTH_USER_MODEL = 'jwt_auth.User'
```

It's important to note that this will modify some tables in Django's auth app, which will be disruptive if there is already some data in the database. It's very likely we'll need to drop our database, then make and run our migrations again.

When creating an API, the custom user model is the first thing we should create.

Finally, if we try visiting our register or login endpoints now, we will come across this error:

`AttributeError: 'NoneType' object has no attribute '_meta'`

This is caused by a circular import issue, and the solution to this is to replace this line:

```py
from django.contrib.auth.models import User
```

with:

```py
from django.contrib.auth import get_user_model
User = get_user_model()
```

wherever we are using our user model. And that's it!

## Further reading

- [Django Tips #6: Custom User Model - William Vincent](https://wsvincent.com/django-tips-custom-user-model/)
- [Specifying a custom user model - Django Docs](https://docs.djangoproject.com/en/2.2/topics/auth/customizing/#specifying-a-custom-user-model)
