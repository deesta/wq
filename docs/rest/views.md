---
order: 3
---

Views & Viewsets
========

[wq.db.rest.views]

[wq.db]'s [REST API] is built with a number of [views] and [viewsets] that extend Django REST Framework's default implementations.  At its core, a view is just a function that accepts a request and returns a response.  The views provided by wq.db are all [class-based views], which encapsulate code common to all views to avoid redundant implementations.  A [viewset] is just a class-based view that can generate multiple view types (e.g. both list and detail views).

The router provided in [app.py] can automatically generate reasonable default views for registered models, but sometimes it is necessary to customize these.  The default classes provided by wq.db are discussed below.

## Base Classes
### GenericAPIView

wq.db's `GenericAPIView` is a simple extension of DRF's [GenericAPIView] that consults the [app.py] router to determine the queryset, serializer class, and the number of items per page (overriding `get_queryset()`, `get_serializer_class()`, and `get_paginate_by()`, respectively).  wq.db's `GenericAPIView` also includes a simple heuristic to determine the template name based on the name of your view class (name - "View" + ".html", lower case).

### SimpleView
`SimpleView` is an extension to `GenericAPIView` that provides a default `get()` implementation, making it ready for quick use in rendering static HTML templates.

### SimpleViewSet
`SimpleViewSet` is like `SimpleView` but implemented as a viewset.  Instead of `get()`, the `list()` method is overridden.  This allows the class to be registered with a router instance (such as the one provided by [app.py]).  app.py uses SimpleViewSet to generate e.g. the [config.json] view.

## ModelViewSet Class

`ModelViewSet` extends `GenericAPIView` as well as Django REST Framework's [ModelViewSet].  It is the default class used for all models registered with [app.py]'s router.  If you need to customize the viewset, create an subclass of `ModelViewSet` and register it with the router:

```python
# myapp/views.py
from wq.db.rest.views import ModelViewSet

class MyViewSet(ModelViewSet):
   # custom code ...
```

```python
# myapp/rest.py
from wq.db.rest import app
from .models import MyModel
from .views import MyViewSet

app.router.register_model(MyModel, viewset=MyViewSet)
```
Note that it is not necessary to explicitly set the `model` or `queryset` attributes on the viewset class if you are only using it with wq.db's router.

`ModelViewSet` extends DRF's version with functionality to support the additional view modes defined by the [wq URL structure], which include:

  * "Edit" and "New" screens (`/[list_url]/[id]/edit` and `/[list_url]/[id]/new`)
  * REST-ful foreign key filters (`/[parent_list]/[id]/[child_list]`)

`ModelViewSet` also includes code to automatically determine whether a POST was submitted via an AJAX JSON request or via a traditional form request.  In the former case, a JSON response is returned, while in the latter case, the client is redirected via HTTP to the detail view for the new item.  This allows for progressive enhancement to support a wide array of browsers. 

[wq.db.rest.views]: https://github.com/wq/wq.db/blob/master/rest/views.py
[wq.db]: http://wq.io/wq.db
[REST API]: http://wq.io/docs/about-rest
[views]: http://www.django-rest-framework.org/api-guide/views/
[viewsets]: http://www.django-rest-framework.org/api-guide/viewsets/
[viewset]: http://www.django-rest-framework.org/api-guide/viewsets/
[app.py]: http://wq.io/docs/app.py
[class-based views]: https://docs.djangoproject.com/en/1.7/topics/class-based-views/
[GenericAPIView]: http://www.django-rest-framework.org/api-guide/generic-views/#genericapiview
[config.json]: http://wq.io/docs/config
[ModelViewSet]: http://www.django-rest-framework.org/api-guide/viewsets/#modelviewset
[wq URL structure]: http://wq.io/docs/url-structure
