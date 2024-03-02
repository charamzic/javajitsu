---
title: 'Asynchronni Http Request'
date: 2024-03-02T16:04:11+01:00
temata: ['java', 'junior', 'http', 'vim']
kategorie: ['Programování']
serie: ['Randori']
ShowToc: false
TocOpen: false
ShowReadingTime: true
---

Minule jsem psal o [Synchronním http requestu](../http-request-bez-fw) a dnes to zkusím 
trochu pimpnout a poslat víc požadavků najednou. Použitá Java knihovna je stejná, přístup je trochu jiný. 

Střídám výrazy request a požadavek. Omlouvám se za to, píšu jak mi 
to přijde pod prsty. Berte to tak, že mluvím o tom samém. 

Při synchronním requestu se program zastaví, dokud nedostane odpověď. Je dobré nastavit requestu 
nějaký timeout, abyste nečekali do Vánoc. Proto se tomu také říká blokující (blocking) request.

U asynchronního (non-blocking) požadavku je to tak, že jej program odešle a ihned vrátí objekt `CompletableFuture`, 
aniž by čekal na odpověď. K tomu se pak dají přidat callbacky a nastavit, jak se bude program chovat, až 
dorazí odpověď. Nebo nedorazí...

### Asynchronní http request v Javě

Jak si jistě všimnete, neošetřuji všechny možné scénáře. V praxi by to chtělo jistě více péče, ale na tohle 
domácí zkoušení to prozatím stačí takhle punkově.

#### Importy

Nad rámec toho, co jsem importoval minule:
```java
    import java.net.URI;
    import java.net.http.HttpClient;
    import java.net.http.HttpRequest;
    import java.net.http.HttpResponse;
```

Musím přidat ještě:
```java
    import java.time.Duration;                          // protože tentokrát nastavuji timeout
    import java.util.ArrayList;                         // sbírám odpovědi do listu
    import java.util.concurrent.CompletableFuture;      // objekt, který se vrací s async volání
    import java.util.concurrent.ExecutionException;     // pro případ, že něco neklapne
```

#### Argumenty programu

Chtěl jsem mít možnost rozhodnout se při spouštění programu, kolik chci odeslat požadavků. 
To lze samozřejmě snadno udělat zadáním argumentu. Jednoduše ve stylu `java <název souboru> <arg>`. 
Proto jsem jsem se hned na začátku programu chtěl ujistit, že byl uživatelem nějaký 
argument zadán. Pokud ne, tak se program ukončí a poučí uživatele, jak má postupovat.

```java
    if (args.length < 1) {
        System.out.println("Please, add desired amount of requests.");
        System.out.println("Usage: java <file name> <number>");
        return;
    }
```

#### Vytvoření http klienta

Tahle část je stejná, jako v případě synchronního requestu. Přibyl akorát zmíněný timeout.
```java
    HttpClient client = HttpClient.newHttpClient();

    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://www.boredapi.com/api/activity"))
            .timeout(Duration.ofMinutes(1))
            .GET()
            .build();
``` 

#### Poslání daného počtu GET požadavků

Jak jsem psal výše, asynchronní request vrací `CompletableFuture` objekt. 
Cyklus `for` se stará o počet iterací, který budu zadávat při spouštění programu.
```java
    ArrayList<String> results = new ArrayList<>();      // sem uložím stringy z response body
    int numberOfRequests = Integer.parseInt(args[0]);   // tohle by bylo lepší ošetřit, hrozí vyjímka 

    // generika a pole způsobí, že kompilátor bude hlásit warning: [unchecked] unchecked conversion
    CompletableFuture<Void>[] responseFutures = new CompletableFuture[numberOfRequests];

    for (int i = 0; i < numberOfRequests; i++) {
        responseFutures[i] = client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
                    .thenApply(HttpResponse::body)
                    .thenAccept(responseBody -> {
                        results.add(responseBody);
                    })
                    .exceptionally(e -> {
                        e.printStackTrace();
                        return null;    // pokud funkce výše vyhodí vyjímku, zachytí se zde a CompletableFuture objekt nabyde hodnotu null
                    });

    System.out.println("The " + (i + 1) + ". request has been sent.");

    }
```

Pak už jen počkám na všechny odpovědi, aby program neskončil dříve, než přijdou a následně 
vytisknu stringy, které jsem si uložil.

```java
    CompletableFuture.allOf(responseFutures).join();

    System.out.println("\nHere are the results:");

    for (String s : results) {
        System.out.println(s);
    }
```
#### Výsledek

Povedlo se. Takhle vypadá výstup, když zapomenu zadat počet iterací jako argument:
```bash
    > java AsyncApiCall 
    Please, add desired amount of requests.
    Usage: java <file name> <number>
```

A toto je výsledek s voláním tří požadavků:
```java
    > java AsyncApiCall 3
    The 1. request has been sent.
    The 2. request has been sent.
    The 3. request has been sent.

    Here are the results:
    {"activity":"Learn how to fold a paper crane","type":"education","participants":1,"pric
    e":0.1,"link":"","key":"3136036","accessibility":0.05}
    {"activity":"Go see a Broadway production","type":"recreational","participants":4,"pric
    e":0.8,"link":"","key":"4448913","accessibility":0.3}
    {"activity":"Watch a movie you'd never usually watch","type":"relaxation","participants
    ":1,"price":0.15,"link":"","key":"9212950","accessibility":0.15}
```

Nesnažil jsem se tentokrát vycucnout jen tu aktivitu, takže je tu celé tělo odpovědi. 
Pokud vás zajímá, jak pomocí regexu získat pouze aktivitu a nic víc, koukněte na 
[příspěvek o synchronním requestu](../http-request-bez-fw).

Tak zas příště!
