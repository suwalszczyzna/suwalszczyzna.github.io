---
layout: post
title:  "Mixiny - pythonowy sposób na interfejsy"
date:   2021-02-17 18:00:00 +0100
categories: python clean_code
---
### Zawartość
- [Jak uzyskuje się kompozycję obiektów w Pythonie?](#jak-uzyskuje-się-kompozycję-obiektów-w-pythonie)
- [Mixiny, całe na biało...](#mixiny-całe-na-biało)
- [Mixiny i nadpisywanie metod](#mixiny-i-nadpisywanie-metod)


Po przejściu do Pythona z innych języków programowania, które również są zorientowane na programowanie obiektowe, zdziwić może fakt, że ów Python nie posiada takiej konstrukcji jak interfejsy. Można zadać sobie pytanie: JAK TO?!

![](<https://media.giphy.com/media/fUqfaPVjiAQcfticZH/source.gif>)

## Jak uzyskuje się [kompozycję ](<https://blog.helion.pl/dziedziczenie-vs-kompozycja/>)obiektów w Pythonie?

Zachłystując się programowaniem obiektowym - konkretnie dziedziczeniem, można bardzo łatwo przesadzić, tworząc niepotrzebne kaskady dziedziczeń.

![](<https://daemon.codes/wp-content/uploads/2021/01/kaskada_dziedziczen.png>)

W takich łańcuchach, kiedy zajdzie potrzeba dokonania zmiany zachowania, np w klasie Ssak, wywołuje to lawinę zmian w dół takiej hierarchii. Analizując powyższy przykład można też dojść do wniosku, że zakładamy sobie kajdanki uznając, że wszystkie ssaki są czteronożne.

W takich przypadkach, na scenę wychodzą Mixiny (domieszki).

## Mixiny, całe na biało...

![](<https://media.giphy.com/media/6m7ZA00RzVD89WnkTVqE/source.gif>)

... yy... albo po prostu na czarno-biało 😎

Dzięki temu, że w języku Python istnieje możliwość wielokrotnego dziedziczenia, tzn. dziedziczenia wielu klas na raz, to nie ma problemu, aby zastąpić łańcuch dziedziczeń klasami nadającymi określone cechy naszym klasom. W świecie pythonowym nazywa się je Mixinami

```python
class CDPlayerMixin():
    def play_music_from_cd():
        print("I'm playing music from Compact-Disk! Wow!")
        pass

class AnalogRadioMixin():
    def play_music_from_local_radio():
        print("Analog radio waves never died!")
        pass

class CassetteMixin():
    def play_music_from_cassette():
        print("Playing music from cassette... Is your tape head clean enough?")
        pass

class InternetRadioMixin():
    def play_music_from_internet():
        print("Playing music from internet. Digital, freedom, 5G speed of music")
        pass
```

Powyżej mamy zbiór zachowań - pewnego rodzaju interfejsów, z których możemy swobodnie dziedziczyć. Dla przykładu, możemy również dodać abstrakcyjną klasę bazową, z której będą dziedziczyć wszystkie klasy:

```python
class Player(ABC):
    def __init__(self, speakers_number):
        self.speakers = speakers_number
        self.current_volume = 0

    def volume_up(self):
        self.current_volume += 1 * self.speakers
```

Zakładamy, że chcemy, aby wszystkie przyszłe klasy miały pewną pulę zachowań i właściwości wspólnych. Przy pomocy mixinów nadamy im odpowiedni charakter.

```python
class OldschoolPlayer(Player, CassetteMixin, AnalogRadioMixin):
    def __init__(self, speakers_number):
        super().__init__(speakers_number)

class Player2000(Player, CDPlayerMixin, InternetRadioMixin):
    def __init__(self, speakers_number):
        super().__init__(speakers_number)

oldschool_player = OldschoolPlayer(2)
oldschool_player.play_music_from_cassette()
oldschool_player.play_music_from_local_radio()
oldschool_player.volume_up()

player_2000 = Player2000(8)
player_2000.play_music_from_cd()
player_2000.play_music_from_internet()
player_2000.volume_up()
```

Jak wyżej widać, stworzyłem dwa odtwarzacze, nadając im indywidualne cechy. Jednak ich wspólnym elementem jest klasa Player, którą wspólnie dziedziczą. Dzięki temu mają wspólne metody i pola.

## Mixiny i nadpisywanie metod

Opisywany koncept możemy zastosować w inny sposób. Wyżej pokazałem, jak można nadać klasom indywidualne funkcjonalności. Nie ma jednak problemu, aby mixiny potraktować trochę bardziej jak typowe interfejsy. Gdzie każdy z nich będzie miał ten sam sposób wywoływania zachowania, jednak jego implementacja będzie różna.

```python
class CDPlayerMixin():
    def play_music(self):
        print("I'm playing music from Compact-Disk! Wow!")

class AnalogRadioMixin():
    def play_music(self):
        print("Analog radio waves never died!")

class Player(ABC):
    def __init__(self, speakers_number):
        self.speakers = speakers_number
        self.current_volume = 0

    def volume_up(self):
        self.current_volume += 1 * self.speakers

class OldschoolPlayer(Player, AnalogRadioMixin):
    def __init__(self, speakers_number):
        super().__init__(speakers_number)

class Player2000(Player, CDPlayerMixin):
    def __init__(self, speakers_number):
        super().__init__(speakers_number)

oldschool_player = OldschoolPlayer(2)
oldschool_player.play_music()
oldschool_player.volume_up()

player_2000 = Player2000(8)
player_2000.play_music()
player_2000.volume_up()
```

Mixiny są sprytnym konceptem, który warto stosować, istnieje ryzyko wplątania się w zagmatwane, wielopoziomowe dziedziczenie.

