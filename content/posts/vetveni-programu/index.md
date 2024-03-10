---
title: 'Větvení Programu'
date: 2024-03-09T21:27:16+01:00
temata: ['java', 'junior', 'http', 'vim']
kategorie: ['Programování']
serie: ['Randori']
ShowToc: false
TocOpen: false
ShowReadingTime: true
---

Jako vždy, tady je k tomu video. Jsem tam rozespalý a mumlám, ale stejně je krásný!

{{< youtube 95d53F9-QCw >}}

---

V tomto videu jsem navázal na [předposlední příspěvek](../http-request-bez-fw) o synchronním 
http requestu. Rozšířil jsem ho o vstup uživatele, který si může vybrat, jaký API call 
program učiní. Je to jednoduchá změna, ale hned je s tím trochu víc zábavy.

Jako druhý endpoint posloužila simulovaná databáze na localhost, kterou jsem v rámci tohoto 
videa také napsal. Takže si tu vyzkoušíme, jak si udělat takový pidi server. Příště to posunu 
zase o kousek dál, jak zmiňuji na konci videa.

## Java server

Dnes jsem tedy začal aplikací, která měla sloužit jako server a vracet na požádání nějaká data.

### Metoda main

Pokud jde o použité knihovny, držím se stále základu.
```java
    import com.sun.net.httpserver.HttpServer;
    import com.sun.net.httpserver.HttpExchange;
    import com.sun.net.httpserver.HttpHandler;

    import java.io.IOException;
    import java.io.OutputStream;
    import java.net.InetSocketAddress;
    import java.util.Date;
    import java.util.ArrayList;
    import java.util.List;
    import java.util.StringJoiner;
```
Hlavní metoda aplikace se stará o vytvoření serveru, který bude poslouchat na portu 8080 a 
jediný endpoint k dispozici bude `/data`.

```java
    public static void main(String[] args) throws IOException {
        HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0); 

        server.createContext("/data", new DataHandler());
        server.setExecutor(null);
        server.start();

        System.out.println("Server started on port 8080...");
    }
```

### HttpHandler

Třída, ve které jsem implementoval `HttpHandler` interface, se stará o zpracování příchozího 
`GET` requestu. Je zde k tomu účelu `@Override` metody `handle()`.

Metodě se předává `HttpExchange` objekt. Potom kouknu, jestli jde o očekávaný `GET`, připravím data 
a headery a pokusím se vrátit odpověď. S ošetřováním každého možného usecase jsem si moc práce 
nedal. Prostě pokud to nebude request, který je zde očekávaný, vrátí se status code `405 Nepovolená metoda`. 
To jednoduše znamená, že server požadavku rozumí, ale nemá na něj zde odpověď.
```java
    @Override
    public void handle(HttpExchange exchange) throws IOException {
        if ("GET".equals(exchange.getRequestMethod())) {

            String responseData = getUsersAsJson();
            
            exchange.sendResponseHeaders(200, responseData.getBytes().length);

            try (OutputStream os = exchange.getResponseBody()) {
                os.write(responseData.getBytes());
            }
        } else {
            exchange.sendResponseHeaders(405, -1);
        }

        exchange.close();
    }
```

## Rozšíření o uživatelský vstup

Když byl server připravený a vyzkoušený, dal jsem se do úpravy programu, který bude posílat 
requesty dle zadaného vstupu od uživatele. Celá třída pak vypadala následovně. 

```java
    import java.net.URI;
    import java.net.http.HttpClient;
    import java.net.http.HttpRequest;
    import java.net.http.HttpResponse;
    import java.util.Scanner;

    public class ContinuosApiCall {

        public static void main(String[] args) {

            Scanner scanner = new Scanner(System.in); // vstup od uživatele beru přes Scanner

            while (true) { // skoro by se tu hodilo do while, že?
                // úvodní nabídka
                System.out.println("\nChoose an option:\n1.Get a random activity\n2.Get users\nType 'exit' to end the program\n");

                String userInput = scanner.nextLine();

                // při zadání příkazu exit se program ukončí
                if (userInput.equalsIgnoreCase("exit")) {
                    break;
                }

                // tady by bylo ošetření vyjímek také na místě
                int userChoice = Integer.parseInt(userInput);

                // podle volby od uživatele se nastaví adresa, kam se bude request posílat
                String apiUrl = (userChoice == 1) ? "https://www.boredapi.com/api/activity" : "http://localhost:8080/data";

                // zbytek zůstal od minule kromě té proměnné s hodnotou adresy prakticky beze změny
                HttpClient httpClient = HttpClient.newHttpClient();
                
                HttpRequest request = HttpRequest.newBuilder()
                        .uri(URI.create(apiUrl))
                        .GET()
                        .build();

                HttpResponse<String> response = null;

                try {
                    response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());

                    System.out.println("Response Code: " + response.statusCode());
                    System.out.println("Response Body:\n" + response.body());
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
                // úklid a rozloučení
                System.out.println("Program ended. Have a great day!");
                scanner.close();
        }
    }
```

Jak je mým zvykem, na konci ještě v rychlosti ukážu, jak vypadá můj script na 
zvětšování velikosti fontu v terminálu pro účely nahrávání. A proč mi nefungoval :)

No a plán na příště je takový, že pustím tyhle dva mini programy v dockeru, abych si procvičil 
docker compose a vůbec práci s tímhle nástrojem.

Jako vždy se video neobešlo bez menších debuggů a zmatků ve Vimu. 

#### Vim moves

A víte co? Hodím na konec článku vždy příkazy, které jsem během natáčení neznal a pak si je dohledal. Např. během psaní 
tohoto programu tam kňourám, že nevím, jak skočit na matching pair závorek. Takže jsem si to našel a teď už to snad nezapomenu. 
Zároveň jsem se i podíval, jak skočit kurzorem zpět na místo, kde byl předtím.

```
    %   -> skočí na matching pair
    `'  -> skočí na předešlou pozici kurzoru (to jsem si přemapoval pouze na ``, protože na české klávesnici je tahle kombinace na pěst
```
