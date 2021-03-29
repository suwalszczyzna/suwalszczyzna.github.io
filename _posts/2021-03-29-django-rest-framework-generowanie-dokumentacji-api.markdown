---
layout: post
title: "Django Rest Framework - generowanie dokumentacji api"
date: 2021-03-29 18:00:00 +0100
categories: python
---
Z pomocą Django Rest Framework i kilku pythonowych bibliotek bardzo łatwo wygenerować dokumentację do naszego API. 

## OpenApi - endpoint 'schema'
Jednym ze sposobów dokumentacji API jest udostępnienie endpointu, który w odpowiedzi zwróci cały schemat API.

### Instalacja bibliotek
```python
pip install pyyaml
pip install uritemplate
```

### Zmiana w pliku url.py
W głównym pliku url.py, zawierającym ściezki do poszczególnych appek djangowych, należy dodać `path` prowadzący do `schema_view`

```python
#importy
from rest_framework.schemas import get_schema_view

url += [
    path('schema', get_schema_view(
            title='Payer API',
            description='Documentation for Payer REST API',
            version='1.0.0'
        ), name='openapi-schema')
]
```

Po zawołaniu endpointu `/schema` zobaczymy dokumentację api w formie [OpenApi](<https://swagger.io/specification/>):
```json
info:
  description: Documentation for Payer REST API
  title: Payer API
  version: ''
openapi: 3.0.0
paths:
  /api/transaction/:
    get:
      operationId: transaction_list
      tags:
      - transaction

servers:
- url: http://127.0.0.1:8000/schema
```

## CoreApi

Innym typem dokumentacji może być specjalna strona internetowa, nawet z możliwośćią przetestowania API, co zapewnia nam biblioteka CoreAPI

### Instalacja bibliotek

```python
 pip install coreapi
```

### Zmiany w pliku settings.py
Dodajemy linijkę:

```python
REST_FRAMEWORK = {'DEFAULT_SCHEMA_CLASS': 'rest_framework.schemas.coreapi.AutoSchema'}
```

### Zmiany w url.py
Dodajemy `path`, np `docs/`

```python
# Importy
from rest_framework.documentation import include_docs_urls

url += [path('docs/', include_docs_urls(title='Payer REST API'))]
```

Pod ścieżką `docs/` kryje się tego typu strona, będąca pełnoprawną dokumentacją:

![DRF - CoreAPI docs]({{ BASE_PATH }}/assets/post_images/drf-coreapi-docs.png "DRF - CoreAPI docs")

Z taką dokumentacją można nawet wejść w interakcję!

![DRF - CoreAPI docs]({{ BASE_PATH }}/assets/post_images/drf-coreapi-docs-2.png "DRF - CoreAPI docs")