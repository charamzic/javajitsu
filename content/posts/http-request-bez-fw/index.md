---
title: 'Http Request Bez FW'
date: 2024-02-27T14:22:35+01:00
temata: ['java', 'junior', 'http', 'vim']
kategorie: ['Programování']
serie: ['Randori']
ShowToc: false
TocOpen: false
ShowReadingTime: true
---

V rámci učení se základů jsem se díval do Core knihoven Javy a napadlo mě zkusit si napsat minimalistický HTTP request. Obecně mě teď 
baví zkoumat základy, protože se chci víc zorientovat v tom, co jsme ve jménu co nejrychlejšího fungovaní v praxi, na rekvalifikaci záměrně přeskočili.

Během rekvalifikace jsme totiž velmi brzy vlítli do Springbootu, aniž bychom moc tušili, o co jde. Nějaké vysvětlení samozřejmě proběhlo, ale 
i když jsem si vyslechl povídání o rozdílu mezi Springem a Springbootem, moc mi to tehdy nezaklaplo. Natož abych tušil, co se děje pod pokličkou.

Proto zkouším psát a buildit v terminálu co možná nejvíc. A vymýšlet, jak udělat maximum s minimem.

### Jak vypadá HTTP request v Javě v úplném základu?

Vystačil jsem si balíkem [java.net](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/net/package-summary.html) a vlastně to jediné, 
co jsem na konci trochu řešil, bylo parsování JSON formátu a vytahování si jenom konkrétní části body z odpovědi. Na to totiž nic v Core knihovnách není. 
Nebo jsem to neobjevil. Ale co tam je? Regex matcher a s tím už to šlo.

Začal jsem importem toho, co budu potřebovat. Jak jsem říkal, píšu ve Vimu a nepoužívám žádné pluginy, ani LSP. Nic se mi automaticky nedoplňuje. 
Je to dobré cvičení. A v tomhle případě ne moc složité. Na program, který pošle HTTP požadavek a přijme odpověď mi stačí čtyři importy ze zmíněného 
`java.net` balíku.

```java
import java.net.URI;                    // uniform resource identifier objekt bude potřeba k identifikování zdroje na webu
import java.net.http.HttpClient;        // client na poslání požadavku a přijetí odpovědi 
import java.net.http.HttpRequest;       // a samotné objekty pro požadavek a odpověď
import java.net.http.HttpResponse;
``` 

### Hlavní část programu pak vypadala takto

Definoval jsem klienta a požadavek a vybral si jednu z mnoha open APIs, které jsou na webu k dispozici. Tento EP vrací na každé zavolání náhodnou aktivitu, 
kterou se můžete zabavit, když se zrovna nudíte: `www.boredapi.com/api/activity`.  
Poté jsem deklaroval objekt odpovědi a následně už se v `try` bloku jal vysílat. 🙂 

```java
HttpClient httpClient = HttpClient.newHttpClient();
        
HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://www.boredapi.com/api/activity"))
        .GET()
        .build();

HttpResponse<String> response = null;

try {
    response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());

    System.out.println("Response Code: " + response.statusCode());
    System.out.println("Response Body: " + response.body());
} catch (Exception e) {
    e.printStackTrace();
}
```

### HttpClient

HttpClient nabízí možnost poslat synchronní, nebo asynchronní požadavek, viz. [dokumentace](https://docs.oracle.com/en/java/javase/21/docs/api/java.net.http/java/net/http/HttpClient.html). Já zvolil první možnost, což znamená, že se program zastaví, dokud mu nepřijde odpověď. 
Případně se ukončí na nějakém timeoutu, který lze také definovat.

Mně odpověď přišla a protože jsem ji rovnou společně se status kódem vytiskl, můžeme se společně pokochat: 
```
Response Code: 200
Response Body: {"activity":"Donate to your local food bank","type":"charity","participants":1,"price":0.5,"link":"","key":"4150284","accessibility":0.8}
```
Šlechetná to aktivita, že? Jak vidíte, kromě doporučení samotné aktivity přišlo v odpovědi ještě pár dalších věcí. Typ aktivity, počet účastníků, případná cena atd..


### There is no JSON parser, Neo...
Mně zajímala jen ta aktivita samotná a trochu jsem se zarazil, když mi došlo, že vlastně nemám po ruce jednoduchý způsob, jak tu hodnotu z přijatého textového 
řetězce vytáhnout. Ano, dalo by se vmýšlet se substringy a podobně, ale to jsem hned zavrhl jako kostrbaté a navíc, co když se příště vrátí odpověď 
v trochu jiném tvaru? Prostě se mi to nelíbilo. Nenapadlo mě rozumné řešení.

Takže jsem koukl, co je ještě v Core libraries k dispozici a vyštrachal: 
```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;
```

### Regex

Jestli je pro mě něco vyšší dívčí, tak je to práce s regulárními výrazy. Ale základy zmáknu.

Nastavil jsem vzor pro regex takto: `"activity"\s*:\s*"(.*?)"`.

1. Najdi přesnou shodu pro `"activity"`, včetně uvozovek.
2. Potom následuje nula nebo více mezer `\s*`.
3. Za tím je dvojtečka `:`.
4. A opakují se možné mezery `\s*`.
5. Po kterých následuje počáteční uvozovka hodnoty o kterou mi jde `"`.
6. Načež přijde libovolný počet znaků, ale co možná nejméně `(.*?)`, ohraničeno skupinou.
7. A uzavře to druhá uvozovka `"`. 

Dohromady a patřičně "odeskejpováno" to v kódu vypadalo následovně:

```java
Pattern treasurePattern = Pattern.compile("\"activity\"\\s*:\\s*\"(.*?)\"");
``` 

Potom už jen stačilo předat Matcheru:

```java
Matcher matcher = treasurePattern.matcher(response.body());

while (matcher.find()) {
    String value = matcher.group(1);
    System.out.println("Activity  = " + value);
}
```

A výsledek nyní vypadá takto:
```
Activity  = Donate to your local food bank
```

## Tip na závěr

Kompiloval jsem program v terminálu. Buďto klasicky pomocí compileru, který je součástí JDK `javac <class name>.java`. Nebo přes compiler ve Vimu `:make %`.

Spouštěl jsem běžně příkazem `java <class name>`. 

Pokud chcete, aby byl program ukecanější a viděli jste, co se za běhu děje, můžete zkusit příkaz `java -verbose <class name>`. 

Volně navazující randori příspěvek je [Asynchronní Http Request](../asynchronni-http-request).
