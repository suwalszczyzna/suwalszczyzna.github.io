---
layout: post
title: "Django Rest Framework - Uprawnienia"
date: 2021-02-19 18:00:00 +0100
categories: python django
---
### Zawartość
- [Implementacja uprawnień](#implementacja-uprawnień)
    - [Dostęp do endpointa tylko dla AdminUser](#dostęp-do-endpointa-tylko-dla-adminuser)
    - [Dostęp do entpointa zarządzany z poziomu Django](#dostęp-do-entpointa-zarządzany-z-poziomu-django)
    - [Customowe uprawnienie: edycja tylko dla autora obiektu Post](#customowe-uprawnienie-edycja-tylko-dla-autora-obiektu-post)
- [Testy jednostkowe](#testy-jednostkowe)


Wraz z całym dobrodziejstwiem Django otrzymujemy możliwość zarządzania uprawnieniami użytkowników co do zarządzania modelem danych (dodaj, usuń, zmień itp), łączenia uprawnień w grupy czy dodania własnych uprawnień. Bez problemu można te uprawnienia wykorzystać w Django Rest Framework.

Poniżej przykłady dla prostego api bloga.

## Zmiany w settings.py

Na początek należy ustawić ogólny poziom dostępu do naszego REST API, dla userów przed autentykacją.

[Mamy 4 opcje:](<https://www.django-rest-framework.org/api-guide/permissions/#api-reference>)

- AllowAny
- IsAuthenticated
- IsAdminUser
- IsAuthenticatedOrReadOnly

Dodajemy poniższą zmienną w settings.py

```python
REST_FRAMEWORK = {
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticatedOrReadOnly",
    ]
}
```

# Implementacja uprawnień

Reszty zmian należy dokonać w pliku views.py, w naszej appce odpowiadającej za rest api

Import klas z uprawnieniami:

```python
from rest_framework.permissions import (
    IsAdminUser,
    DjangoModelPermissions,
    BasePermission,
    SAFE_METHODS,
)
```

### Dostęp do endpointa tylko dla AdminUser

Poniżej klasa odpowiadająca za endpoint zwracający wszystkie obiekty modelu Post. Definicja uprawnień to tylko jedna linijka: `permission_classes = [IsAdminUser]`

```python
class PostList(generics.ListCreateAPIView):
    permission_classes = [IsAdminUser]
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

### Dostęp do entpointa zarządzany z poziomu Django

Jeśli chcemy aby uprawnienia były zgodne z tymi ustawionymi w panelu admina naszego projektu Django to trzeba wskazać klasę `DjangoModelPermissions`.

```python
class PostList(generics.ListCreateAPIView):
    permission_classes = [DjangoModelPermissions]
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

### Customowe uprawnienie: edycja tylko dla autora obiektu Post

Należy zdefiniować nową klasę, która będzie dziedziczyła po `BasePermission` z `rest_framework.permissions`.

Poniższa klasa nadaje możliwość edycji obiektu o ile jest on jego autorem.

```python
class PostUserWritePermission(BasePermission):
    message = "Editing post is restricted to the author only."

    def has_object_permission(self, request, view, obj):
        if request.method in SAFE_METHODS:
            return True
        return obj.author == request.user
```

Jednak jakie to będą opcje edycji, ustala się w implementacji powyższego customowego uprawnienia. Trzeba zauważyć, że klasa `PostDetail` odpowiadająca za endpoint wyświetlający pojedyńczy obiekt dziedziczy dodatkowo po klasie z customowym uprawnieniem. Dodatkowo, trzeba ją wskazać w `permission_classes` w samym `PostDetail`

```python
class PostDetail(generics.RetrieveUpdateDestroyAPIView, PostUserWritePermission):
    permission_classes = [PostUserWritePermission]
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

# Testy jednostkowe
W tym przypadku użyję wbudowanej biblioteki unitTest.

Tworzymy najpierw klasę testową i przygotowujemy dane

```python
from django.urls import reverse
from rest_framework import status
from rest_framework.test import APIClient, APITestCase
from blog.models import Post, Category
from django.contrib.auth.models import User

class PostTests(APITestCase):
    @classmethod
    def setUpTestData(self):
        self.client = APIClient() # utworzenie klienta
        self.test_category = Category.objects.create(name="django") # utworzenie kategorii
        
        # Utworzenie dwóch testowych userów
        self.test_user1 = User.objects.create_user(
            username="test_user1", password="123456"
        )
        self.test_user2 = User.objects.create_user(
            username="test_user2", password="123456"
        )

        # Stworzenie blog-posta, autorem jest test_user1
        Post.objects.create(
            title="Post title",
            author_id=1,  # test_user1
            excerpt="Some text",
            content="Some content",
            category_id=1,
            status="published",
        )
```

Następnie przypadek testowy. Chcemy na początku przetestować, czy użytkownik będący autorem wpisu, może wysłać zapytanie PUT, czyli edytować pola obiektu

```python
    def test_post_update_by_author(self):

        # logowanie usera test_user1
        self.client.login(username=self.test_user1.username, password="123456")

        # pobranie url dla Post o id 1
        url = reverse(("blog_api:detailcreate"), kwargs={"pk": 1})

        # wysłanie zapytania PUT
        response = self.client.put(
            url,
            {
                "title": "It's edited post",
                "author": 1,
                "excerpt": "New value of excerpt",
                "content": "Amazing content",
                "status": "published",
            },
            format="json",
        )
        
        # Spodziewamy się otrzymać CODE 200 - w tym wypadku oznacza, że edycja powiodła się
        self.assertEqual(response.status_code, status.HTTP_200_OK)
```
Wszystko jest ok, po odpaleniu testów powyższy przechodzi.
```cmd
Ran 1 test in 0.396s

OK
```

Teraz przetestujemy czy edycję posta jeśli zalogujemy się jako inny user niż autor

```python

    def test_post_update_not_by_author(self):

        # Tym razem logujemy się jako inny user niż autor
        self.client.login(username=self.test_user2.username, password="123456")
        url = reverse(("blog_api:detailcreate"), kwargs={"pk": 1})
        response = self.client.put(
            url,
            {
                "title": "It's edited post",
                "author": 1,
                "excerpt": "New value of excerpt",
                "content": "Amazing content",
                "status": "published",
            },
            format="json",
        )
        
        # Spodziewamy się CODE 403 Forbidden
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
```

Po odpaleniu testów:
```cmd
Ran 2 tests in 0.531s

OK
```

Jest tak jak się spodziewaliśmy.

Więcej o autentykacji i uprawnieniach można przeczytać w [dokumentacji](<https://www.django-rest-framework.org/>) Django REST Framework. Polecam również zajrzeć do oficjalnego tutorialu na ten temat: [Authentication & Permissions](<https://www.django-rest-framework.org/api-guide/permissions/#api-reference>)