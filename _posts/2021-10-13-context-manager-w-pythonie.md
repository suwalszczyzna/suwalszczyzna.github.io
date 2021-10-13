---
layout: post
title: "Context manager w Pythonie"
date: 2021-10-13 18:00:00 +0100
categories: python
---
Jako programiści stronimy od wykonywania powtarzalnych czynności. Jeśli gdzieś widzimy możliwość zautomatyzowania czegoś - robimy to bez względu na koszty. Twórcy języków programowania czy bibliotek również o tym myślą i wprowadzają zmiany ułatwiające wszystkim życie. Bohaterem dzisiejszego wpisu jest [context manager](https://docs.python.org/3/library/contextlib.html)


# Otwórz, zamknij, otwórz zamknij...
Jako dobrze wychowani programiści (oraz tacy, którzy nie chcą spędzać sobót na debugowaniu problemów, których na pewno nie spodował nasz patch wgrany w piątek o 16:00) kiedy otwieramy plik powinniśmy go również zamknąć, a jeśli pobieramy coś z bazy danych to najpierw musimy nawiązać połączenie, pobrac dane, a jeśli dodawaliśmy jakieś dane to zacommitować je i zamknąć połączenie. Dużo roboty. Powtarzalnej roboty.

## Typowy przykład z file open
Poniżej znajduje się typowy przykład takiej powtarzalnej czynności. Czyli za pomocą funkcji `open()` otwieramy plik, `readlines()` zwraca nam iterator z kolejnymi liniami naszego pliku tekstowego. Na końcu, po wyświetleniu danych, należy zamknąć plik - tutaj metoda `close()`

```python
file = open('data.txt')
for line in file.readlines():
    print(line)
file.close()

```
Ten przykład jest dość prosty i w zasadzie można się pokusić o zostawienie tego jak jest - przecież wszystko jest czytelne. Tak, jednak zaraz się przekonamy, że przy bardziej skomplikowanych przykładach, kiedy tych infrastrukturalnych rzeczy jest więcej bardzo łatwo o pomyłkę.

Jednak na razie zmieńmy powyższy kod na tak, by używał składni specjalnej składni `with`

```python
with open('data.txt') as file:
    for line in file.readlines():
        print(line)

```

W zasadzie dwa powyższe bloki kodu robią to samo. Jednak w drugim nie musimy już precyzyjnie otwierać i zamykać pliku.

Czemu?

# With + własny context manager
Skąd ten `with` wie, że należy otworzyć i zamknąć plik? Wszystko dzięki managerom kontekstu (context managers). To co podajemy po `with` to właśnie ten manager, a on ma w sobie kilka ciekawych metod, które są odpalane przez `with` we właściwych momentach.

Zaimplementujmy własny manager kontekstu. Konkretnie: context manager dla bazy danych. Póki co tylko makietę.

## Koncepcja połączenia z bazą:

Kiedy chcemy skorzystać z bazy danych musimy się z nią połączyć. Później pobrać dane, a na końcu zamknąć połączenie. Jeśli coś dodajemy do bazy danych to obowiązkowo należy zacommitować zmiany (przed zamknięciem połączenia). Zamknąć połączenie należy również wtedy kiedy gdzieś po drodze wystąpi błąd (nie można zakładać, że Internet jest 100% dostępny, że nikt nie przetnie szpadlem kabla sieciowego podczas rutynowych prac, itp.)

Jeszcze zanim stworzymy sam kontekst, stwórzmy warstę abstrakcji, dzięki której będziemy mogli sterować naszą bazą danych.

```python
# database.py

import random
import logging

logging.basicConfig(level=logging.DEBUG)


class DatabaseException(Exception):
    """Raises when something goes wrong with db"""


class Database:
    def __init__(self):
        logging.info('Init db')

    def get_data(self):
        if random.randint(0, 1):
            raise DatabaseException("Problem with db")
        return 'Really important data'

    def connect(self):
        logging.info('Database connected')

    def commit(self):
        logging.info('Changes commited')

    def close_connection(self):
        logging.info('Connection closed')
```

Koncepcyjnie mamy zapewnioną całą funkcjonalność. Dodatkowo, aby nie używać mało eleganckiego `print`a, w tym wypadku użyłem loggera. Czemu? A czemu nie? 😎

Aby wywołać od czasu do czasu błąd użyłem biblioteki random. W losowych (tak, tak, w [pseudolosowych](https://jasonxiii.pl/modul-random-w-jezyku-python-wyjasnienie-metod) 🔭) przypadkach zostanie wywołany błąd zamiast zwrócić "dane". Jak w życiu.


Użycie bez `with`:

```python
db = Database()
db.connect()
try:
    data = db.get_data()
    logging.debug(data)
except:
    logging.error('Db error')
finally:
    db.close_connection()
```

Wynik:

```
INFO:root:Init db
INFO:root:Database connected
DEBUG:root:I've got the data!
INFO:root:Connection closed
```

Olaboga, ile kodu 🤯 Panie, a po co mnie w tym miejscu wiedzieć, że coś trzeba zamykać, otwierać, łapać wyjątki. Omg, daj mi po prostu dane, a jak nie to nie. Tu jest kiosk ruchu, ja tu logikę biznesową mam do ogarnięcia, a nie jakieś Pańskie infrastrukturalne dyrdymały. Które pewnie się zmienią i będę miał to na łbie.

No ok, zbudujmy w końcu ten context manager

## Class context manager

```python
# class_context.py
import logging
from contextlib import AbstractContextManager

from context_manager.custom_context.database import Database

logging.basicConfig(level=logging.DEBUG)


class DatabaseContext(AbstractContextManager):
    def __init__(self):
        self.db = Database()
    
    def __enter__(self):
        self.db.connect()
        return self.db
    
    def __exit__(self, exc_type, exc_value, exc_tb):
        if exc_type:
            logging.error('Error db!')
        else:
            self.db.commit()
        self.db.close_connection()
        return True
```

No i jeszcze więcej kodu. Cóż, piękno kolejnej warstwy abstrakcji musi boleć.

Jednak do rzeczy, co się tutaj wyprawia. 

Mamy coś takiego jak `AbstractContextManager`, klasa abstracyjna z której dziedziczy nasz context manager. Musimy zaimplementować dwie metody: `__enter__` oraz `__exit__`. 


> Tutaj uwaga. Jako, że działamy w świecie pythonowym, w którym podstawą jest [duck typing](https://pl.wikipedia.org/wiki/Duck_typing) to wcale nie musimy dziedziczyć po klasie `AbstractContextManager` lecz po prostu napisać powyższe metody. Jednak moim zdaniem warto to robić, aby przypilnować siebie oraz pokazać innym programistom w jasny sposób, że jest to legitny context manager 👴


* `__enter__` odpalane jest kiedy piszemy: `with DatabaseContext() as db`. W tym wypadku łączymy się z bazą danych i zwracamy nasz obiekt db, który będzie do użycia wewnątrz konstukcji with. 
* `__exit__` odpalane jest kiedy zakończy with kończy swoje działanie. Czyli to super miejsce na zamknięcie połączenia bazą danych czy wykonania jakichś czynności sprzątających. Tutaj też obsługiwane są wyjątki. W tym wypadku logujemy, że wystąpił błąd, ale dzięki `return True` nie rzucamy go dalej. Jeśli chcielibyśmy aby wyjątek poszedł wyżej to wystarczy nie zwracać nic (funkcja zwróci `None`)


A teraz użycie z `with` 🎉

```python
# main.py

import logging
from context_manager.custom_context.contexts.class_context import DatabaseContext
from context_manager.custom_context.contexts.func_context import func_db_context
from context_manager.custom_context.database import Database

logging.basicConfig(level=logging.DEBUG)


with DatabaseContext() as db:
    logging.debug(db.get_data())
```

Jak widać, samo użycie bazy danych to tak na prawdę dwie linijki kodu. Nie musimy zastanawiać się co się dzieje pod spodem, jak nawiązać połączenie i czy trzeba go zamykać itp.  


## Funcion context manager
Jako wprawnym pythonowcom coś może wam tu nie grać. Dużo tego kodu powstaje. Jakieś klasy abstrakcyjne, specjalne metody... nie da się prościej?

Pewnie, że się da 🐍

Twórcy pythona udostępnili nam [dekorator](https://daemon.codes/python/dekorowanie-w-pythonie/), który pozwala generator zamienić w context manager:

```python
# func_context.py

import logging

from contextlib import contextmanager
from context_manager.custom_context.database import Database, DatabaseException

logging.basicConfig(level=logging.DEBUG)


@contextmanager
def func_db_context(db: Database):
    db.connect()
    try:
        yield db
        db.commit()
    except DatabaseException:
        logging.error('Something goes wrong')
    finally:
        db.close_connection()
```

Sympatycznie, nieprawdaż? Dla prostych managerów kontekstu jak znalazł.