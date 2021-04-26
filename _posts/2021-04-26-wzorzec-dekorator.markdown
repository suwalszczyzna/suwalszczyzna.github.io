---
layout: post
title: "Dekorowanie w Pythonie: wzorzec dekorator i dekoratory"
date: 2021-04-26 18:00:00 +0100
categories: python
---
### Zawartość
- [Wzorzec dekorator](#wzorzec-dekorator)
- [Dekorowanie funkcji](#dekorowanie-funkcji)
- [Zastosowanie składni @dekorator](#zastosowanie-składni-dekorator)

## Wzorzec dekorator
Wzorzec dekorator pozwala obudować funkcjonalność obiektu nadając jej nowe cechy, bez ingerencji w dotychczasowy sposób działania. 

Aby zilustrować działanie wzorca dekoratora, przygotowałem prostą klasę `Client`, która używa api [jokeapi.dev](https://jokeapi.dev/) w celu pobrania losowego żartu. 
```python client.py
import json
from urllib import request


class Client:
    API_URL = 'https://v2.jokeapi.dev/joke/Any?type=single'

    def request(self):
        with request.urlopen(self.API_URL) as req:
            return json.loads(req.read().decode())

    def make_me_laugh(self):
        json_data = self.request()
        return json_data.get('joke')
```

Przykładowe użycie:
```python
def show_me_a_joke(client):
    print(client.make_me_laugh())


if __name__ == '__main__':
    api_client = Client()
    show_me_a_joke(api_client)
```

Wynik w konsoli:
```
I was struggling to figure out how lightning works, but then it struck me.
```



Najczęstrzym, a zarazem najlepiej pokazującym przykładem zastosowanie dekoratora jest na przykład implementacja logowania. Przed lub po wywołaniu metody chcemy zalogować tą czynność, w tym przypadku używając zwykłego printa. Żeby nadać temu przykładowi więcej sensu obliczam jak długo zajmuje naszemu programowi wykonanie akcji odpytania JokeAPI i odbioru jego odpowiedzi.

Implementacja polega na zbudowaniu nowej klasy klienta, która przyjmuje parametr w postaci obiektu pierwotnego klienta. Dodatkowo, nasza dekorująca klasa implementuje funkcję, która nazywa się tak samo jak funkcja, którą chcemy udekorować. W tym wypadku jest to `make_me_laugh()`. W środku nowej funkcji wywołujemy funkcję bazowej klasy tak, aby jej użycie było przezroczyste dla użytkownika, jednak teraz możemy dodać coś od siebie. 

```python
from client import Client
import time


class LogClient:
    def __init__(self, client):
        self.client = client

    def make_me_laugh(self):
        print("Getting data...")
        start = time.time()
        result = self.client.make_me_laugh()
        end = time.time()
        print(f"Data fetched in {end - start} sec.")
        return result
```

Wykorzystanie dekoratora może wyglądać następująco:
```python
def show_me_a_joke(client):
    print(client.make_me_laugh())


if __name__ == '__main__':
    api_client = Client()
    show_me_a_joke(LogClient(api_client))

```
Wynik:
```
Getting data...
Data fetched in 0.4514896869659424 sec.
Today, my son asked "Can I have a book mark?" and I burst into tears.
11 years old and he still doesn't know my name is Brian.
```

Przekazujemy do funkcji używającej obiektu typu `Client` i jej metody `make_me_laugh` inny obiekt, udekorowany naszą implementacją w/w metody. 
Czy nie prościej byłoby użyć dziedziczenia? 
Dobrą praktyką, w momencie kiedy mamy wybór między użyciem wzorca dekoratora, a dziedziczeniem, jest użycie dekoratora. Dlaczego? 
Użycie klasy dekorującej pozwala na dynamiczne podkładanie obiektu dekorowanego. Nic nie stoi na przeszkodzie aby udekorować sam dekorator. Jeśli poniższy przykład chcielibyśmy zrealizować za pomocą dziedziczenia to niezbędne byłoby stworzenie silnego wiązania między klasą `LogClient`, a nową klasą `FileSaverClient`, w postaci łańcucha dziedziczeń. Dodatkowo nie mielibyśmy możliwości, aby udekorować obiekt `Client` klasą `FileSaverClient` bez narzutu dekoratora `LogClient`.

Poniższe przykłady lepiej pokażą elastyczność wzorca dekoratora

Na początek klasa `FileSaverClient`. Pozwala ona na zrzut do tesktowego pliku wyniku zapytania do JokeAPI
```python
class FileSaverClient:
    def __init__(self, client):
        self.client = client

    def make_me_laugh(self):
        result = self.client.make_me_laugh()
        with open("data_dump.txt", "a") as file:
            file.writelines(["-------\n", result, '\n'])
        return result
```

Hipotetycznie, chcielibyśmy zawsze dekorować obiekt klasy `Client` poprzez klasę `LogClient`, jednak jeśli flaga `DEBUG_MODE` ma wartość `True`, to do tego chcemy dorzucić możliwość zrzutu odpowiedzi do pliku, dzięki klasie `FileServerClient`.

```python
if __name__ == '__main__':
    DEBUG_MODE = True
    api_client = Client()

    if DEBUG_MODE:
        api_client = FileSaverClient(api_client)

    show_me_a_joke(LogClient(api_client))
```

W powyższym przypadku w konsoli dostaniemy:
```
Getting data...
Data fetched in 0.44116830825805664 sec.
I have a joke about trickle down economics, but 99% of you will never get it.
```
a w pliku odłoży się:
```
I have a joke about trickle down economics, but 99% of you will never get it.
```

## Dekorowanie funkcji

Dekorowanie za pomocą rozbudowanych klas jest w wielu zastosowaniach przydatne, jednak co jeśli chcemy udekorować jakąś pojedyńczą, samotną funkcję? Co jeśli nie chcemy dekorować całego obiektu?
Możemy znacznie prościej napisać funkcję przyjmującą funkcję, która zwraca funkcję - jednak co nieco ustrojoną :)

Upraszczamy kod. Teraz JokeAPI jest pytane przez prostą funkcję:
```python
import json
from urllib import request


def make_me_laugh():
    with request.urlopen('https://v2.jokeapi.dev/joke/Any?type=single') as req:
        json_data = json.loads(req.read().decode())
    return json_data.get('joke')


print(make_me_laugh())
```

Powyższy kod robi to samo, co na samym początku tego wpisu: dostajemy w konsoli losowy żart.
Implementacja funkcji dekorującej, dodającej logowanie wygląda następująco:
```python
def logging(func):
    def wrapper(*args, **kwargs):
        print("Getting data...")
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"Data fetched in {end - start} sec.")
        return result
    return wrapper


def make_me_laugh():
    with request.urlopen('https://v2.jokeapi.dev/joke/Any?type=single') as req:
        json_data = json.loads(req.read().decode())
    return json_data.get('joke')


make_me_laugh = logging(make_me_laugh)

print(make_me_laugh())
```
Co tu się wydarzyło? 
Funkcja `logging` odpowiada klasie `LogClient`. Przyjmuje ona funkcję, która jest wywoływana w wenętrznej funkcji `wrapper`. Podobnie jak w przypadku klas, przed i po jej wywołaniu logujemy jakieś informacje. Na końcu funkcja `logging` zwraca funkcję `wrapper`, która jest przeprabiem funkcji przekazywanej jako `func`.

_Parametr `func`, jak i funkcja `wrapper` mogą być dowolne, jednak to jest przyjęta konwencja w świecie pythonowym. Bardzo często z nią się spotkasz przy poznawaniu innych dekoratorów._

Później tworzona jest zmienna, która nazywa się dokładnie jak dekorowana funkcja. Jednak pod tą nazwą kryje się teraz wywołanie funkcji `loggin` z przekazaniem jej funkcji `make_me_laugh`. Po tej operacji pod nazwą `make_me_laugh` czeka na użytkownika funkcja `wrapper`, czekająca na swoje odpalenie, które następuje w ostaniej linijce :) 

Na początku może być to trochę zagmatwane, więc warto sobie to przeanalizować i przećwiczyć.

## Zastosowanie składni @dekorator
Wykorzystanie dekoratów w Pythonie jest tak powszechnym zjawiskiem, że dodano do niego specjalny sugar syntax, który zastępuje wyżej opisane tworzenie zmiennej zasłaniającej dekorowaną funkcję. 

Przy definicji funkcji, używamy po prostu znaku `@` i nazwy funkcji z wrapperem. Poniższy przykład wyjaśni wszystko:
```python
def logging(func):
    def wrapper(*args, **kwargs):
        print("Getting data...")
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"Data fetched in {end - start} sec.")
        return result
    return wrapper


@logging
def make_me_laugh():
    with request.urlopen('https://v2.jokeapi.dev/joke/Any?type=single') as req:
        json_data = json.loads(req.read().decode())
    return json_data.get('joke')


print(make_me_laugh())
```

Efekt ten sam, a jak schuldnie. W taki sposób można teraz dekorować dowolną funkcję, np:
```python
@logging
def scream(content):
    time.sleep(1)
    return f"{content.upper()}!!!!!!"


print(scream("I'm so quite"))
```
Co w efekcie zwróci: 
```
Getting data...
Data fetched in 1.0010933876037598 sec.
I'M SO QUITE!!!!!!
```
