---
layout: post
title:  "Wzorzec Observer - przykład w pythonie"
date:   2021-01-01 18:00:00 +0100
categories: python wzorce
---
```python
from abc import ABC, abstractmethod
import urllib.request
import json

class Subscriber(ABC):

    @abstractmethod
    def update(self, json_data) -> None:
        pass

class Joker:
    def __init__(self):
        self.subscribers = []

    def add_subscriber(self, joke_subscriber: Subscriber) -> None:
        self.subscribers.append(joke_subscriber)

    def notify(self, data: dict) -> None:
        for subscriber in self.subscribers:
            subscriber.update(data)

    def run(self):
        with urllib.request.urlopen('https://v2.jokeapi.dev/joke/Any?type=single') as req:
            json_data = json.loads(req.read().decode())
            self.notify(json_data)
```

```python
from Joker import Subscriber

class JokeSubscriber(Subscriber):
    def update(self, json_data) -> None:
        print(
            f'=========='
            f'\nCategory: {json_data.get("category")}'
            f'\nJoke: {json_data.get("joke")}'
        )

class JokeSubscriber2(Subscriber):
    def update(self, json_data) -> None:
        print(
            f'=========='
            f'\nid: {json_data.get("id")}'
            f'\nJoke: {json_data.get("joke")}'
        )
```

```python
from Joker import Joker
from JokeSubscriber import JokeSubscriber, JokeSubscriber2

joke_subscriber_1 = JokeSubscriber()
joke_subscriber_2 = JokeSubscriber2()

joker = Joker()
joker.add_subscriber(joke_subscriber_1)
joker.add_subscriber(joke_subscriber_2)

joker.run()
```

Obrazek z nagłówka - [https://www.freepik.com/jcomp](<https://www.freepik.com/jcomp>)

