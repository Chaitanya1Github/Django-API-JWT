.. _secure_api:

Securing APIs
==============

Authentication
---------------
Authentication is a mechanism of associating an incoming request with a set of credentials like username and password or token that are responsible for identifying a legit user. REST framework provides several authentication schemes out of the box, and also allows you to implement custom schemes.

JWT Authentication
-------------------
JSON Web Token is a fairly new scheme of token-based authentication. Unlike the built-in TokenAuthentication scheme, JWT Authentication doesn't need to use a database to validate a token. A package for JWT authentication is ``djangorestframework-simplejwt``. It is a third party package.

Requirements
------------
* Python (3.7, 3.8, 3.9, 3.10)
* Django (2.2, 3.1, 3.2)
* Django REST Framework (3.10, 3.11, 3.12)

Installation
------------
Using pip you can install Simple JWT:

.. code-block:: python

   pip install djangorestframework-simplejwt


Initial Setup
-------------
Your django project must be configured to use the library. In ``settings.py``, add ``rest_framework_simplejwt.authentication.JWTAuthentication`` to the list of authentication classes:

.. code-block:: python

   REST_FRAMEWORK = {
   'DEFAULT_AUTHENTICATION_CLASSES': (
       'rest_framework_simplejwt.authentication.JWTAuthentication',
   )
   }

In root ``urls.py`` file, add path that routes you to your app`s ``urls.py``. But then again it is up to you, how you wish to route to your app.

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
   # register 'user' as the endpoint of API.
   router.register(r'users', UserViewSet, basename='user')

   urlpatterns = [
    path('generate-token/', views.ApiGenerateToken.as_view(), name='api_generate_token'),
    path('', include(router.urls)),
   ]

Example
-------
We're ready to generate access-token. To generate access-token, make a POST request on: ``http://127.0.0.1:8000/api/generate-token/``.
The following piece of code will go in your ``views.py``.

.. code-block:: python

   from django.contrib.auth import authenticate
   from django.contrib.auth.models import User
   from rest_framework import viewsets, status
   from rest_framework.response import Response
   from rest_framework.views import APIView
   from rest_framework_simplejwt.tokens import RefreshToken


   class ApiGenerateToken(APIView):
       def post(self, request):

           # getting username and password
           username = "chaitanya"
           password = "admin"

           # checking if user exists in database for authentication
           user = authenticate(request, username=username, password=password)
           tokens = {}
           if user is not None:

               # if user exists in database then generate token for that user.
               refresh = RefreshToken.for_user(user)
               tokens = {
                   'refresh': str(refresh),
                   'access': str(refresh.access_token),
               }
               tokens = tokens

           # return the response to where the API was called from
           return Response(tokens)


The above example will return you a dictionary of **access** and **refresh** tokens. Just like so. You need to have only value of **access** to access APIs.

.. code-block:: python

   {
       "access":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNjQ2MjE5OTcxLCJpYXQiOjE2NDYyMTk2NzEsImp0aSI6IjIwNjdmNDQzY2ViYTQ4YWM4OWEwY2NmZmEwNDE5YWRjIiwidXNlcl9pZCI6MX0.dceTY6kZRceMGUgHSJk3ewa7zQDP-ZNYii4074h1WDc",
       "refresh":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0b2tlbl90eXBlIjoicmVmcmVzaCIsImV4cCI6MTY0NjMwNjA3MSwiaWF0IjoxNjQ2MjE5NjcxLCJqdGkiOiIxYzJmYWZjMjIxMTA0YTNjOGM0YjY1ZmRlNzdiMDQ0ZSIsInVzZXJfaWQiOjF9.bbdn1sgCOTEl2aCpSPSrU78K7Z_2t0eoVg6fRU0WkNo"
   }

The following code will again go in your ``views.py`` file to read the list of data.

.. code-block:: python

   from django.contrib.auth.models import User
   from rest_framework import serializers, viewsets, status
   from rest_framework.response import Response
   from rest_framework.permissions import IsAuthenticated


   class UserSerializer(serializers.ModelSerializer):
       class Meta:
           model = User
           fields = ["id", "username", "email"]


   class UserViewSet(viewsets.ViewSet):

       # 'IsAuthenticated' class checks if the user is authenticated with the given token.
       permission_classes = [IsAuthenticated]

       def list(self, request):
           queryset = User.objects.all()
           serializer = UserSerializer(queryset, many=True)
           return Response(serializer.data, status=status.HTTP_302_FOUND)

Now we will use the access token to read the list of users from database table. If you are using Client User Interface for making requests then this is how you would want to add **access** token in your request:

.. code-block:: python

   headers = {"Authorization": "Bearer " + access}
   response = requests.get("http://127.0.0.1:8000/api/users/", headers=headers)
   response = response.json()

The variable 'response' contains list of data in JSON format.


Overall program with APIs and Authentication.

.. code-block:: python

   from django.contrib.auth import authenticate
   from django.contrib.auth.models import User
   from rest_framework import serializers, viewsets, status
   from rest_framework.response import Response
   from rest_framework.views import APIView
   from rest_framework_simplejwt.tokens import RefreshToken
   from rest_framework.permissions import IsAuthenticated


   class ApiGenerateToken(APIView):
       def post(self, request):

           # getting username and password
           username = "chaitanya"
           password = "admin"

           # checking if user exists in database for authentication
           user = authenticate(request, username=username, password=password)
           tokens = {}
           if user is not None:

               # if user exists in database then generate token for that user.
               refresh = RefreshToken.for_user(user)
               tokens = {
                   'refresh': str(refresh),
                   'access': str(refresh.access_token),
               }
               tokens = tokens

           # return the response to where the API was called from
           return Response(tokens)


   class UserSerializer(serializers.ModelSerializer):
       class Meta:
           model = User
           fields = ["id", "username", "email"]


   class UserViewSet(viewsets.ViewSet):

       # 'IsAuthenticated' class checks if the user is authenticated with the given token.
       permission_classes = [IsAuthenticated]

       def list(self, request):
           queryset = User.objects.all()
           serializer = UserSerializer(queryset, many=True)
           return Response(serializer.data, status=status.HTTP_302_FOUND)

       def retrieve(self, request, pk):
           queryset = User.objects.get(id=pk)
           serializer = UserSerializer(queryset, many=False)
           return Response(serializer.data, status=status.HTTP_302_FOUND)

       def patch(self, request, pk=None):
           user = User.objects.get(pk=pk)
           # user object contains existing data and variable 'data' contains new information to be replaced
           serializer = UserSerializer(instance=user, data=request.data)
           serializer.is_valid(raise_exception=True)
           serializer.save()
           return Response(serializer.data, status=status.HTTP_202_ACCEPTED)

       def delete(self, request, pk=None):
           user = User.objects.get(pk=pk)
           user.delete()
           return Response(status=status.HTTP_204_NO_CONTENT)
