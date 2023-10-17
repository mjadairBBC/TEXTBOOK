# Views

Django considers itself a _Model-Template-View_ framework, however coming from a _Model-View-Controller_ architecture you can think of a Django _View_ as a _Controller_. It gets the required data from the model and passes it to the correct template or _serializer_ then sends the result to the client.

There are many ways of creating a _view_ in Django, we'll look at a few different ways here:

## Functional view

The simplest view is a function. A typical INDEX route might look like this:

```py
def index(request):
    books = Book.objects.all()
    return render(request, 'index.html', {'books': books})
```

or with a serializer:

```py
def index(_request):
    books = Book.objects.all()
    serializer = BookSerializer(books, many=True)
    return Response(serializer.data)
```

This can now be added to the app's URLs like so:

```py
urlpatterns = [
    path('/books', index)
]
```

This is very similar to the way we made our controllers with Express.


## Classical view

An alternative option is to use a class rather than a function. This can simplify our code somewhat because we can have a class handle different methods to the same URL, for example INDEX and CREATE, or SHOW, UPDATE and DELETE.

Typically each URL would map to a class. The INDEX and CREATE route would be mapped to a `List` class and SHOW, UPDATE and DELETE to a `Detail` class. The class needs to extend Django's `View` class for HTML, or Django REST Framework's `APIView` for APIs. We'll use the `APIView` in this example:

```py
from rest_framework.response import Response
from rest_framework.views import APIView

class BookList(View):

    def get(_request):
        books = Book.objects.all()
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)

    def create(request):
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=HTTP_201_CREATED)

        return Response(serializer.errors, status=HTTP_422_UNPROCESSABLE_ENTITY)
```

This can now be added to the app's URLs like so:

```py
urlpatterns = [
    path('/books', BookList.as_view())
]
```


## Generics

Since the standard REST functionality INDEX, CREATE, SHOW, UPDATE and DELETE is very predictable, as developers we tend to find ourselves writing the same code for each resource we have (Books, Authors, Genres etc), which is laborious and has the potential for typos.

To help mitigate this Django and DRF come with built in _generic views_ which will take care of these mundane tasks for us.

Making a fully RESTful resource is now trivial:

```py
from rest_framework.views import APIView
from rest_framework.generics import ListCreateAPIView, RetrieveUpdateDestroyAPIView

class BookList(ListCreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer


class BookDetail(RetrieveUpdateDestroyAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```

With those 6 lines of code we have created all 5 RESTful routes! We can now hook them up to the app's URLs like so:

```py
urlpatterns = [
    path('/books', BookList.as_view()),
    path('/books/<int:pk>', BookDetail.as_view())
]
```

> **Note**: When using _generic views_ we **must** use `pk` in our URLs

It's worth noting that DRF has a number of _generic views_, so here's a quick overview of them:

#### List views:

- `CreateAPIView`: Just the CREATE route
- `ListAPIView`: Just the INDEX route
- `ListCreateAPIView`: INDEX and CREATE routes


#### Detail views:

- `RetrieveAPIView`: Just the SHOW route
- `DestroyAPIView`: Just the DELETE route
- `UpdateAPIView`: Just the UPDATE route
- `RetrieveUpdateAPIView`: SHOW and UPDATE routes
- `RetrieveDestroyAPIView`: SHOW and DELETE routes
- `RetrieveUpdateDestroyAPIView`: SHOW, UPDATE and DELETE routes


## Further reading:

- [Writing Views - Django Docs](https://docs.djangoproject.com/en/2.2/topics/http/views/)
- [Class Based Views - Django REST Framework Tutorial](https://www.django-rest-framework.org/tutorial/3-class-based-views/)
- [Generic Views - Django REST Framework Docs](https://www.django-rest-framework.org/api-guide/generic-views)
