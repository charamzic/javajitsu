---
title: 'Instalace Javy'
date: 2024-02-19T19:51:52+01:00
temata: ['java', 'junior']
kategorie: ['Programování']
serie: ['Java padawan']
ShowReadingTime: true
weight: 1
cover:
  image: sdk-man.png
  caption: "Zdroj: https://sdkman.io/"
---

Na to, abyste spustili svůj první Java program, potřebujete mít nainstalovanou Javu. A protože se do budoucna může stát, 
že budete chtít/potřebovat více verzí Javy a k tomu třeba ještě Maven (o tom potom), 
doporučuji rovnou nainstalovat [SKDMAN!](https://sdkman.io/). Usnadní vám život. 

SKDMAN! je v v podstatě takový správce JDK a SDK kitů. Můžete s ním snadno stahovat a instalovat různé verze a balíky a následně 
mezi nimi přepínat dle potřeby.

### Instalace SDKMAN!
Symbol `$` z následujících příkladů do terminálu nekopírujte. Pouze to, co následuje po něm.

Spusťte v terminálu příkaz:
```bash
$ curl -s "https://get.sdkman.io" | bash
```

Proklikejte se postupem a jako další příkaz spusťte:
```bash
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
```

Po zadání `sdk version` byste měli vidět podobný výstup:
```bash
$ sdk version


SDKMAN!
script: 5.18.2
native: 0.4.6
```

Originál popis instalace (i na Windows) je [tady](https://sdkman.io/install).

### Instalace Javy
Instalace Javy je teď jednoduchá. Příkaz bez dalších parametrů nainstaluje nejnovější verzi.

```bash
$ sdk install java
```

Po instalaci přijde otázka, zda chcete tuto verzi nastavit jako defaultní:
```bash
Do you want java 21.0.2-tem to be set as default? (Y/n):
```

Stačí potvrdit klávesou Enter a SKDMAN! vám oznámí:
```
Setting java 21.0.2-tem as default.
```

Tím je nainstalováno a nastaveno. Pro teď je to vše.

Pokud byste chtěli mít jó jistotu, že je Java doma, zadejte:
```bash
$ sdk current java
```

Měli byste dostat podobný výstup:
```bash
Using java version 21.0.2-tem
```

Mimochodem, tímhle jste nainstalovali komplet JDK (Java development kit), takže máte vše potřebné na běh i vývoj Java aplikací.

---

{{< youtube a4gywwELs1U  >}}
