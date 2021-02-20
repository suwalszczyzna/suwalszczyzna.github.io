---
layout: post
title: "Pyenv - praca z wieloma wersjami pythona "
date: 2021-02-20 18:00:00 +0100
categories: python
---
Mówi się, że nie powinno się ruszać domyślnie zainstalowanej wersji systemowej pythona. Jeżeli już mamy zainstalowanego jakiegoś pythona to wcale nie znaczy, że twórcy systemu chcięli oddać w nasze ręce narzędzie w postaci Pythona (najczęściej w starszej wersji niż obecna). Python bardzo często jest domyślnie zainstalowany ponieważ pewne części systemu opierają się na nim. Dlatego nie powinniśmy aktualizować wersji samego węża jak i zainstalowanych bibliotek. 

## Python 2.7, 3.0, a może 3.9? Tak.
Jest wiele narzędzi pomagających w zarządzaniu wersjami pythona i wirtualnymi środowiskami, a jednym z nich jest [pyenv](<https://github.com/pyenv/pyenv-virtualenv>), który niedawno zagościł w moim workflow. Czas pokaże czy na stałe. 

Pyenv pozwala nam instalować, a później dowolnie wybierać która wersja Pythona ma być tą aktywną, dodatkowo uściślając scope wybranej wersji.

## Instalacja Pyenv + konfiguracja shella
W związku z tym, że konfiguracja shella po instalacji pyenva wygląda różnie dla różnych powłok, to wolę tego nie opisywać odsyłając do dokumentacji, w której jest to dość jasno opisane: https://github.com/pyenv/pyenv


## Wyświetlenie listy dostępnych wersji
Aby sprawdzić, jakie wersje pythona pyenv ma nam do zaoferowania używamy komendy (poinższa lista jest bardzo skrócona, tak na prawdę to poleceni wypluwa kilkadziesiąt różnych wersji pythona, również z różnymi implementacjami):
```bash
$ pyenv install --list
Available versions:
  2.7.17
  2.7.18
  3.9.1
  3.9.2
  activepython-3.6.0
  anaconda3-2020.11
  graalpython-20.3.0
  ironpython-2.7.7
  jython-2.7.1
  micropython-1.12
  miniconda3-4.7.12
  miniforge3-4.9.2
  pypy3.7-7.3.3
  pyston-0.6.1
  stackless-3.7.5

```

## Sprawdzenie zainstalowanych Pythonów w pyenv:
Gwiazdka wskazuje aktywną wersje w obecnym scope.
```bash
$ pyenv versions                
 * system
   3.9.2
```

## Python w zależności od scope
Dzięki pyenv możemy w łatwy sposób określić jaki python ma działać globalnie, jaki lokalnie (w obecnym katalogu i jego podkatalogach), a jaki tylko w aktualnej sesji shella.

```bash
$ pyenv global 3.9.0 # dostępny globalnie z każdego poziomu
$ pyenv local 3.7.2 # dostępny w aktualnym folderze i jego podfolderach
$ pyenv shell 2.7.18 # dostępny tylko w aktualnej sesji shella
```

Kiedy sprawdzimy sobie jaka jest obecna wersja pythona to wyświetli się wersja w zależności od w/w ustawień (poniżej przykład kiedy systemowa wersja to 3.8):
```bash
$ pyenv global 3.9.0
$ python -V
Python 3.9.2
```

## Wirtualne środowiska
Po wywołaniu komend `python` czy `pip` będzie używany python ustawiony przez pyenv to bez problemu możemy korzystać z wirtualnych środowisk i pipa tak samo jak na systemowej wersji:

Tworzenie środowiska
```bash
$ python -m venv venv
```

Aktywacja i deaktywacja środowiska
```bash
$ source /venv/bin/activate
$ deactivate
```

Instalacja paczek wewnątrz venva
```bash
$ pip install django
```