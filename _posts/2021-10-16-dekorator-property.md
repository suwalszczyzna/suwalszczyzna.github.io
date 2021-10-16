---
layout: post
title: "Dekorator @property"
date: 2021-10-16 18:00:00 +0100
categories: python
---
Na tym blogu już pisałem o tym czym są [dekoratory](https://daemon.codes/python/dekorowanie-w-pythonie/) i pokazałem przykładowe zastosowanie. Mówiliśmy sobie też o dekoratorze [Context manager](https://daemon.codes/python/context-manager-w-pythonie/). Przyszedł czas na kolejny bardzo praktyczny dekorator wbudowany w język: **@property**


# Gettery i settery

Jeśli mieliście styczność z innymi obiektowymi jęzkami, np. Java, to mogliście spotkać się z tym, że aby wyciągnąć jakąś wartość z pola obiektu to należało dopisać odpowiednie funkcje, np. `getUrl` lub `setUrl` (tzw. gettery i settery), dla pola prywatnego pola `url`. Nie dodanie np settera oznaczało, że dane pole jest read-only - nie dawaliśmy możliwości do jakiejkolwiek zmiany.

# Zabijanie małych kotków

Jako, że `Python` to bardzo elastyczny język to pozwala nie używać w ogóle tego typu konstrukcji i tworzyć pola i zmieniać ich wartości zupełnie bez jakichkolwiek deklaracji:
```python
class Resource:
    pass


resource = Resource()
resource.url = "http://google.com"
print(resource.url)

>> Wynik:
http://google.com
```

Trochę bolesne rozwiązanie i każdy szunający się programista, który to czyta powinien poczuć, że gdzieś tam umarł mały kotek 💀. Autor takiego rozwiązania na pewno nie lubi kotków, a już na pewno nie lubi innych ludzi oraz siebie 😅.

Czemu? Pisząc kod w taki sposób na pewno wiele razy natkniemy sie na problemy `AttributeError`, a ktoś patrzący na klasę `Resource` zupełnie nie ma możliwości aby zorientować się po co ona w ogóle została napisana i co oferuje.

Mniej bolesna implementacja powyższego scenariusza może wyglądać tak:
```python
class Resource:
    def __init__(self):
        self.url = ''
```
Czyli mamy zadeklarowane pole `url`, które po utworzeniu obiektu przyjmie pustego stringa. Jednak co wtedy, gdy chcięlibyśmy trochę rozwinąć ten pomysł i wprowadzić walidację wprowadzanych danych?

# Gettery i settery w Pythonowy sposób: @property

Pythonowym sposobem na dodanie getterów i setterów, o których pisałem na początku wpisu jest dekorator `@property`. 

Przykład:

```python
class Resource:
    def __init__(self):
        self._url = ''

    @property
    def url(self) -> str:
        return self._url
```
> Przyjęło się, że aby oznaczyć metodę, pole lub zmienną jako prywatną to dodajemy do nazwy prefix `_`, czyli `_url`. Co prawda nie ma to znaczenia dla interpretera, jednak IDE będzie już krzyczeć przy próbie dostania się bezpośrednio do tak oznaczonego elementu.

Use case tak napisanej klasy nie różni się wcale od tego co mieliśmy w poprzednim akapicie. Przy pracy z instancją klasy do url odwołujemy się tak samo, po kropce. 

Jednak w naszym wypadku nie będzie możliwości przypisania elementu. Nie dodaliśmy do konstruktora możliwości przekazania wartości dla pola `url`, ani też nie napisaliśmy `settera`.

> Samo `@property` oznacza **getter**

```python
from property.Models.Resource import Resource

resource = Resource()
resource.url = "http://google.com/"
print(resource.url)
```

```
>> Wynik:
Traceback (most recent call last):
  File "E:\Code\Python\playground\property\main.py", line 4, in <module>
    resource.url = "http://google.com/"
AttributeError: can't set attribute
```

Aby wprowadzić do naszej klasy setter należy użyć dekoratora: `@nazwa_pola_oznaczonego_property.setter`:

```python
@url.setter
def url(self, new_url: str):
    if not valid_url(new_url):
        raise NotValidUrlException('Your url is not valid')

    self._url = new_url
```

Dodając powyższy kod do klasy uda nam się bez problemu przypisać nową wartość pola `url`.

## Walidacja w setterze
Wprowadźmy wspominaną wcześniej funkcjonalność walidowania przypisywanej wartości. Użyjemy tutaj regexów i osobnej funkcji `valid_url`, która zwróci `True/False` dla danego url.

```python
import re

def valid_url(url: str) -> bool:
    regex = r"^(?:http(s)?:\/\/)?[\w.-]+(?:\.[\w\.-]+)+[\w\-\._~:/?#[\]@!\$&'\(\)\*\+,;=.]+$"
    return bool(re.match(regex, url, flags=re.I))
```

Kiedy będziemy próbowali w naszej klasie przypisać nieprawidłową wartość rzucimy customowym wyjątkiem:

```python
class NotValidUrlException(Exception):
    pass
```

Implementacja settera z powyższymi elementami będzie wyglądać następująco:

```python
@url.setter
def url(self, new_url: str):
    if not valid_url(new_url):
        raise NotValidUrlException('Your url is not valid')

    self._url = new_url
```

Użycie:

```python
resource = Resource()
resource.url = "a invalid_url"
print(resource.url)
```

```
>> Wynik:
Traceback (most recent call last):
  File "E:\Code\Python\playground\property\main.py", line 4, in <module>
    resource.url = "a invalid_url"
  File "E:\Code\Python\playground\property\Models\Resource.py", line 24, in url
    raise NotValidUrlException('Your url is not valid')
property.Models.Resource.NotValidUrlException: Your url is not valid
```

Tym samym zaimplementowaliśmy klasę, która używa dekoratora property do manipulacji polami oraz zaimplementowaliśmy walidację danych.

Napisałem również testy do naszej klasy. Całość kodu poniżej.


```python
# Models/Resoruce.py
import re


def valid_url(url: str) -> bool:
    regex = r"^(?:http(s)?:\/\/)?[\w.-]+(?:\.[\w\.-]+)+[\w\-\._~:/?#[\]@!\$&'\(\)\*\+,;=.]+$"
    return bool(re.match(regex, url, flags=re.I))


class NotValidUrlException(Exception):
    pass


class Resource:
    def __init__(self):
        self._url = ''

    @property
    def url(self) -> str:
        return self._url

    @url.setter
    def url(self, new_url: str):
        if not valid_url(new_url):
            raise NotValidUrlException('Your url is not valid')

        self._url = new_url
```

```python
# tests.py
import pytest
from property.Models.Resource import Resource, NotValidUrlException


VALID_URLS = ['https://github.com/', 'https://github.com/test?val1=key1&val2=key2', 'daemon.codes', 'daemon.codes/test.php']

INVALID_URLS = ['', 'github', 'http://invalid.com/perl.cgi?key= | http://web-site.com/cgi-bin/perl.cgi?key1=val1&key2']


@pytest.mark.parametrize("valid_url", VALID_URLS)
def test_given_valid_url(valid_url):
    resource = Resource()
    resource.url = valid_url
    assert resource.url == valid_url


@pytest.mark.parametrize("invalid_url", INVALID_URLS)
def test_when_tries_to_set_invalid_url(invalid_url):
    with pytest.raises(NotValidUrlException):
        resource = Resource()
        resource.url = invalid_url

```