Building APIs
=============
Django REST framework is a powerful and flexible toolkit for building Web APIs. To follow this tutorial it is expected to have little knowledge of Django, at least you should be able to create project and app and perform migrations.
In this section you will learn to build your own APIs and modify them.
In next section, you will see :ref:`how to secure APIs <secure_api>` using JWT Authentication.


Requirements
------------
* Python (3.7, 3.8, 3.9, 3.10)
* Django (2.2, 3.1, 3.2)

Installation
------------
Using pip you can install Django REST Framework:

.. code-block:: python

   pip install djangorestframework


Initial Setup
-------------
Add ``'rest_framework'`` to your ``INSTALLED_APPS`` in ``settings.py`` file.

.. code-block:: python

    INSTALLED_APPS = [
    ...
    'rest_framework',
    ]


In root ``urls.py`` file, add path that routes you to your app`s ``urls.py``. But of course, it is up to you, how you want to setup your routes.

.. code-block:: python

   from django.contrib import admin
   from django.urls import path, include

   urlpatterns = [
   ...
   path('admin/', admin.site.urls),
   path('api/', include('api.urls')),
   ...
   ]

In app`s ``urls.py``, we are using DefaultRouter for good practice. DefaultRouter is something used to handle all the requests like: GET, POST, PATCH, DELETE.

.. code-block:: python

   from django.urls import path, include
   from rest_framework.routers import DefaultRouter
   from api import views
   from api.views import UserViewSet

   # defaultRouter is something used to handle all the requests like: GET, POST, PATCH, DELETE
   router = DefaultRouter()

   # it can detect the request and calls methods like create(), list(), retrieve(), patch(), delete() automatically, from class UserViewSet, in views.py.
   # register 'users' as an endpoint of API.
   router.register(r'users', UserViewSet, basename='user')

   urlpatterns = [
    path('', include(router.urls)),
   ]

Example
-------
We're ready to create, read, update and delete our APIs now. You can access these APIs with: ``http://127.0.0.1:8000/api/users/``.
The following piece of code will go in your ``views.py``.

.. code-block:: python

   from rest_framework.response import Response
   from rest_framework import status, serializers, viewsets
   from django.contrib.auth.models import User


   # Serializers define the API representation in JSON.
   class UserSerializer(serializers.ModelSerializer):
       class Meta:
           model = User
           # we are serializing only 3 fields below
           fields = ["id", "username", "email"]


   class UserViewSet(viewsets.ViewSet):

       # this  method is used to create/insert new data in database.
       # accepts only POST request
       # http://127.0.0.1:8000/api/users/
       def create(self, request):
           # getting data
           username = "Chaitanya"
           email = "chaitanya@gmail.com"
           password = "admin123"

           # saving data in database table User
           user = User.objects.create_user(username=username, email=email, password=password)

           # serializer is used to convert the data from/to JSON
           serializer = UserSerializer(user, many=False)

           # returns the response to where the API was called from
           return Response(serializer.data)

       # this method is used to enlist number of records in table.
       # accepts only GET request with no arguments.
       # http://127.0.0.1:8000/api/users/
       def list(self, request):
           queryset = User.objects.all()
           serializer = UserSerializer(queryset, many=True)
           return Response(serializer.data, status=status.HTTP_302_FOUND)

       # this method is used to retrieve particular record from the table based of primary key.
       # accepts only GET request with an argument to find record in table.
       # http://127.0.0.1:8000/api/users/1/
       def retrieve(self, request, pk):
           queryset = User.objects.get(pk=pk)
           serializer = UserSerializer(queryset, many=False)
           return Response(serializer.data, status=status.HTTP_302_FOUND)

       # this method is used to update the given record
       # accepts only PATCH request with an argument to find record in table and update it.
       # http://127.0.0.1:8000/api/users/1/
       def patch(self, request, pk):
           user = User.objects.get(pk=pk)
           # user object contains existing data and variable 'data' contains new information to be updated
           serializer = UserSerializer(instance=user, data=request.data)
           serializer.is_valid(raise_exception=True)
           serializer.save()
           return Response(serializer.data, status=status.HTTP_202_ACCEPTED)

       # this method is used to delete the given record
       # accepts only DELETE request with an argument to find record in table and delete it.
       # http://127.0.0.1:8000/api/users/1/
       def delete(self, request, pk):
           user = User.objects.get(pk=pk)
           user.delete()
           return Response("record deleted", status=status.HTTP_204_NO_CONTENT)