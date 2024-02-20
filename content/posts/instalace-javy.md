---
title: 'Instalace Javy'
date: 2024-02-19T19:51:52+01:00
tags: ['java', 'junior']
categories: ['Programování']
ShowReadingTime: true
cover:
  image: images/sdk-man.png
  caption: "Zdroj: https://sdkman.io/"
---

Na to, abyste spustili svůj první Java program, potřebujete mít nainstalovanou Javu. Je více způsobů, jak to udělat. 
Protože se do budoucna může stát, že budete chtít/potřebovat více verzí Javy a k tomu třeba ještě Maven (o tom potom), 
doporučuji rovnou nainstalovat [SKDMAN!](https://sdkman.io/). Usnadní vám život. 

SKDMAN! je v v podstatě takový správce JDK a SDK kitů. Používám ho, protože mi to přijde příjemnější, než abych pokaždé, 
když chci konkrétní verzi Javy, nebo třeba vyzkoušet Graal Cloud Native, musel lézt do vyhledávače a shánět se po nějakém 
stáhnutelném souboru. Nemluvě o následné instalaci a řešení cesty.

Navíc pak velmi snadno přepínáte mezi verzemi, aniž byste museli řešit, kam jste si co nainstalovali, co vlastně máte k dispozici a podobně.

### Instalace SDKMAN! (Mac / Linux) 

Instalace na Windows bude buďto součástí videa, které brzy udělám, nebo v samostatném článku. 

### Postup

Symbol `$` z následujících příkladů do terminálu nekopírujte. Pouze to, co následuje po něm.

Spusťte v terminálu příkaz:
```bash
$ curl -s "https://get.sdkman.io" | bash
```

Proklikejte se postupem a jako další příkaz spusťte:
```bash
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
```

Po zadání:
```bash
$ sdk version
```

Byste měli vidět podobný výstup:
```bash
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
