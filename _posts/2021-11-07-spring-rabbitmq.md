---
layout: post
title: "Spring i RabbitMQ - prosta implementacja"
date: 2021-11-07 18:00:00 +0100
categories: java
---
Dziś zajmiemy się obsługą kolejek RabbitMQ przy użyciu Javy i frameworka Spring

# Instalacja RabbitMQ
Najprostrzą opcją jest postawienie testowego rabbita przy użyciu dockera:
```
docker run -d --hostname my-rabbit --name some-rabbit -p 15672:15672 -p 5672:5672 rabbitmq:3-management
```

Na porcie `5672` jest serwer RabbitMQ
Na porcie `156721` działa webowy interfejs, RabbitMQ Management

Domyślny użytkownik: `guest/guest`

# Wymiana Stringów
Na początek zajmiemy się wymanianą prostych stringów. Utworzymy publishera oraz consumera. Publisher, przy pomocy api restowego, będzie publikował wiadomości na wskazanej kolejce, a consumer będzie je pobierał. 

W consumerze stworzymy dwie opcje pobierania wiadomości: 
- prosta funkcja odpalana z RestAPI, która przy każdym wywołaniu pobierze kolejną wiadomość. 
- drugą opcją będzie użycie `RabbitListenera`, który będzie cały czas nasłuchiwał na wiadomości, kiedy jakaś się pojawi to zostanie wyprintowana na ekran


Przy tworzeniu projektów publishera i consumera użyłem Spring Initializr i modułów:
- Spring Web 
- Spring for Rabbit
## Publisher
```java

// file Publisher.java
import ...


// (1)
@RestController
public class Publisher {

// (2) start
    private final RabbitTemplate rabbitTemplate;

    @Autowired
    public Publisher(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }
// (2) end

// (3)
    @GetMapping("/sendMessage")
    public String sendMessage(@RequestParam String message){
        rabbitTemplate.convertAndSend("events","events" ,message);
        return "sent";
    }
}

```

1. Tworzę klasę o nazwie "Publisher" i oznaczam ją jako Springowy component, tutaj wskazuje, że użyję go do RestApi (stąd anotacja `@RestConttroller`, ale zwykły `@Component` też zadziała)
2. Wstrzykuję zależlność przy użyciu konstruktora (zalecana metoda). Technicznie, zadziała również brak konstruktora. Wtedy anotację `@Autowired` trzeba dodać przy polu `rabbitTemplate`
3. Tworzę metodę `sendMessage` z `GetMapping` - tutaj ustalam jaki restowy endpoint odpali tą metodę. 
   - Spodziewam się `@RequestParam` o nazwie message, który będzie stringiem.
   - Używam wstrzykniętej zależności `rabbitTemplate` odpalając metodę `convertAndSend` (mogę użyć również samego `send`, ale wtedy interfejs rabbitTemplate oczekuje typu `Message`), jako pierwszy argument podaję `exchange`, następnie `routingKey`, a w końcu podaję treść stringa z `RequestParam`.
   - Na końcu zwracam stringa "sent" dla potwierdzenia wysłania wiadomości.

Kiedy wyślę zapytanie typu GET do mojego Springowego serwera (w tym wypadku działającego na porcie 8080) to kolejka `events` dostanie pierwszą wiadomość. 

```bash
GET localhost:8080/sendMessage?message=tutaj_tresc_wiadomosci
```

## Consumer
Teraz przyszła kolej na stworzenie aplikacji, która będzie miała za zadanie odbierać wiadomości z kolejki `events`

```java

// file Consumer.java

import ...

// (1)
@RestController
public class Consumer{
// (2) start
    private final RabbitTemplate rabbitTemplate;

    @Autowired
    public Consumer(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }
// (2) end
// (3)
    @GetMapping("/receiveMessage")
    public String getMessage(){
        Object message = rabbitTemplate.receiveAndConvert("events");
        return message.toString();
    }
}
```

1. Podobnie jak w przypadku Publishera tworzę klasę z anotacją `@RestController`
2. Oraz identycznie wstrzykuję zależność
3. Tworzę funkcję `getMessage` zmapowaną pod endpoint `/receiveMessage`
   - Używam zależności `rabbitTemplate` tym razem do pobrania wiadomości. Ze względu na to, że operujemy stringami używam metody `receiveAndConvert` z parametrem "events" (nazwa kolejki), która za mnie pobierze i przekonwertuje body wiadomości na obiekt. W takim przypadku Java nie wie co tam dostanie, jednak ze względu na nasz słowny kontrakt z Publisherem wiemy, że to będzie String. Dlatego pozwalam sobie na użycie metody `toString` na moim obiekcie.

Przy każdym odpaleniu metody `getMessage()` będę dostawał kolejną wiadomość z kolejki, a pobrane wiadomości zniknął z kolejki. 

Kiedy kolejka będzie pusta, a my nadal będziemy wywoływać metodę `getMessage` to dostaniemy NullPointerException bo do zmiennej `message` zostanie przekazany `null`, a my będziemy próbować na tym obiekcie wykonać metodę `toString`, która nie będzie przecież dostępna. 

Na razie to tak zostawmy i nie obsługujmy tego błędu, za chwilę zajmiemy się innym sposobem na pobieranie wiadomości.

W tym wypadku w naszym Springu, w `application.properties` wskazałem inny port servera. Chcę aby publisher i consumer pracowali jednocześnie, ale jak wiadomo nie mogę odpalić dwóch aplikacji na tym samym porcie.

```java

// file application.properties
server.port=8081
```

Po wysłaniu zapytania typu GET na adres localhost:8081/receiveMessage otrzymam jedną wiadomość z kolejki

```bash
GET localhost:8081/receiveMessage
```
### RabbitListener
Jak wcześniej wspominałem, podejdziemy do pobierania wiadomości z kolejek też w inny sposób. Skorzystamy z `@RabbitListenera`. Jest to banalnie proste, jak wszystko w Springu 😎

Dodaję do klasy `Consumer` metodę `rabbitListener` z odpowiednią adnotacją:

```java

@RabbitListener(queues = "events")
public void rabbitListener(String s){
    System.out.println(s);
}

```
- W parametrze anotacji podaję nazwę kolejki
- A w tym wypadku po otrzymaniu każdej wiadomości zostanie ona wyprintowana na ekran


Teraz po odplaniu Springowej aplikacji z Cosumerem zobaczymy w konsoli każdą kolejną wiadomość wrzucaną przez Publishera. Bez odpalania żadnego Restowego Api.