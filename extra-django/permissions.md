# Permissions

Django REST Framework comes with some powerful built in permissions. They are very simple to use and can be added any view by passing them as a list or tuple to a `permission_classes` property.

Here's an example:

```py
from rest_framework.permissions import IsAuthenticatedOrReadOnly
from rest_framework.response import Response
from rest_framework.views import APIView

class BookList(ListCreateAPIView):

  permission_classes = (IsAuthenticatedOrReadOnly,)

  def get(self, _request):
      books = Book.objects.all() # get all the books
      serializer = BookSerializer(books, many=True)
      return Response(serializer.data)
```

`IsAuthenticatedOrReadOnly` allows an unauthenticated user read only access eg: INDEX and SHOW, but only allows unsafe access, eg: CREATE, UPDATE and DELETE to authenticated users.

There are several permissions that come with DRF:

- `AllowAny`: This is the same as having no permissions set, but is included to show the intention of no permissions.
- `IsAuthenticated`: The user must be logged in for ALL routes (including INDEX and SHOW for example)
- `IsAdminUser`: The user must be logged in _and_ be a super user
- `IsAuthenticatedOrReadOnly`: The user must be logged in for unsafe routes only (CREATE, UPDATE and DELETE)


## Object-level permissions

We often want to allow a user the ability to UPDATE and DELETE a record _only if they are the user that created the record._ To do this we can write an _object-level permission._

```py
class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Object-level permission to only allow owners of an object to edit it.
    Assumes the model instance has an `owner` attribute.
    """

    def has_object_permission(self, request, view, obj):
        # Read permissions are allowed to any request,
        # so we'll always allow GET, HEAD or OPTIONS requests.
        if request.method in permissions.SAFE_METHODS:
            return True

        # Instance must have an attribute named `owner`.
        return obj.user == request.user
```

This permission will look for a `user` field on the model, and check that it matches the logged in user. `request.user` is the logged in user, the same as `req.currentUser` from Express.

We can now use this permission by adding it to our permission classes, and by calling `self.check_object_permissions` in each method we want to check for:

```py
from rest_framework.response import Response
from rest_framework.views import APIView

from .permissions import IsOwnerOrReadOnly

class BookDetailView(APIView):

    permission_classes = (IsOwnerOrReadOnly,)

    def get(self, _request, pk):
        book = Book.objects.get(pk=pk) # get a specific book
        self.check_object_permissions(_request, book) # Note: we only have to call this because we've written this GET method. If this was a Generic View, simply adding the permission class would have been sufficient, unless we had overriden GET, PUT, or DELETE ourselves.
        serializer = BookSerializer(book)
        return Response(serializer.data)
```


In order that we can store the user on the model, we need to add a `ForeignKey` field to the model:

```py
from django.contrib.auth.models import User

class Book(models.Model):
    title = models.CharField(max_length=50, unique=True)
    author = models.CharField(max_length=50, unique=True)
    image = models.CharField(max_length=200)
    user = models.ForeignKey(User, related_name='books', on_delete=models.CASCADE)

    def __str__(self):
        return f'{self.title} - {self.author}'
```

And we also need to set the user to the model in the view.

```py
from rest_framework.permissions import IsAuthenticatedOrReadOnly
from rest_framework.response import Response
from rest_framework.views import APIView

class BookList(ListCreateAPIView):

    permission_classes = (IsAuthenticatedOrReadOnly,)

    def post(self, request):
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(user=request.user) # add the user to the model on save
            return Response(serializer.data, status=201)

        return Response(serializer.errors, status=422)
```

## Displaying the user

With the user being automatically added to the book, we can now display the user by adding it to the `BookSerializer`:

```py
class BookSerializer(serializers.ModelSerializer):

    class Meta:
        model = Book
        fields = ('id', 'title', 'author', 'image', 'user',)
```

## Further reading

- [Authentication & Permissions - Django REST Framework Tutorial](https://www.django-rest-framework.org/tutorial/4-authentication-and-permissions/)
- [Permissions - Django REST Framework Docs](https://www.django-rest-framework.org/api-guide/permissions/)
