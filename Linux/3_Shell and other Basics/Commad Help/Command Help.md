# A `help` parancs használata Linuxban

A `man` oldalak után a `help` segédprogram a legjobb és leggyorsabb módja annak, hogy a terminálból segítséget kérj, és megtanuld egy adott parancs opcióit és működését.

> **Eredeti cikk és képernyőmentések:** https://linuxhandbook.com/help-command/

## A parancs alapvető szintaxisa

```bash
help [options] [pattern]
```

- `[options]`: Kapcsolók, amelyek módosítják a `help` parancs alapértelmezett viselkedését.
- `[pattern]`: A téma vagy a parancs neve, amelyről információt szeretnél kapni.

## Parancsok és Példák

### 1. Segítség kérése magáról a `help` parancsról

Megmutatja a `help` parancs rövid leírását és a hozzá elérhető opciókat (kapcsolókat):

```bash
help help
```

### 2. Részletes információk lekérése (hasonlóan a man oldalakhoz)

Visszaadja a parancs leírását, a használható jelzőket (flageket) és azok magyarázatát.

**Szintaxis:** `help <command_name>`

Példa a `cd` parancs lekérdezésére:

```bash
help cd
```

### 3. Rövid leírás (Short description) lekérése

Ha nincs szükséged a teljes leírásra és az összes opcióra, a `-d` kapcsolóval csak egy egysoros, rövid összefoglalót kérhetsz a parancsról.

**Szintaxis:** `help -d <command_name>`

Példa:

```bash
help -d cd
```

### 4. Csak a parancs szintaxisának lekérése

Ha csak arra vagy kíváncsi, hogyan kell felépíteni / begépelni az adott parancsot, használd a `-s` kapcsolót.

**Szintaxis:** `help -s <command_name>`

Példa:

```bash
help -s cd
```

## Korlátok (Mikor használjunk inkább man oldalakat?)

A `help` parancs hátránya, hogy **nem tartalmaz információt a külső parancsokról** (olyan parancsokról, amelyek nem a shell beépített részei), mint például az `ls`, `apt`, `find`, stb.

Ilyen esetekben az egyetlen és legjobb lehetőséged a manuál oldalak használata, a `man` paranccsal (pl. `man ls`).

##Forrás
-[https://linuxhandbook.com/help-command/]

# Segítségkérés a Linux terminálban

Mielőtt az internet mindenki számára elérhető lett volna, a Linux (és az azt megelőző rendszerek) beépített súgókkal és kézikönyvekkel (man pages) rendelkeztek. Ezek ma is ugyanolyan fontosak, hiszen:

- Ezek az eredeti, leghitelesebb dokumentációk egy parancs működéséről.
- Linux vizsgákon (ahol nincs internet-hozzáférés) kizárólag ezek állnak rendelkezésre.

Két fő parancs létezik a segítségkérésre: a `help` és a `man`. Hogy melyiket kell használni, az attól függ, hogy a parancs a shell része-e, vagy egy külső program.

## Elmélet: Beépített (Built-in) vs. Külső (External) parancsok

- **Beépített shell parancsok (Shell built-ins):** Ezek a parancsok magának a shellnek (pl. bash) a részei. Példák: `echo`, `cd`, `alias`. Ezekhez nincsenek man oldalak. Ha beírod, hogy `man cd`, a rendszer hibaüzenetet ad (`No manual entry for cd`).

- **Külső parancsok:** Olyan szoftvercsomagok részei (pl. GNU coreutils), amelyek előre telepítve vannak a Linux disztribúciókon. Példák: `ls`, `mv`, `rm`, `cat`, `less`.

> **Tipp:** Ha látni szeretnéd az összes beépített shell parancs listáját, egyszerűen gépeld be a terminálba a `help` szót!

## 1. Parancsok dokumentációja a `man` paranccsal (Külső parancsokhoz)

A `man` (manual) parancs megnyitja egy külső parancs részletes kézikönyvét, amely tartalmazza a szintaxist, a használható opciókat és azok magyarázatát.

**Szintaxis:**

```bash
man command_name
```

A kézikönyvek általában a `less` programmal nyílnak meg, így a navigáláshoz az alábbi billentyűparancsokat használhatod:

| Billentyű | Művelet / Akció |
| --- | --- |
| Fel nyíl (Up arrow) | Egy sorral feljebb ugrás |
| Le nyíl (Down arrow) | Egy sorral lejjebb ugrás |
| Szóköz vagy PgDn | Egy teljes oldallal (képernyővel) lejjebb lapozás |
| b vagy PgUp | Egy teljes oldallal (képernyővel) feljebb lapozás |
| g | Ugrás a fájl (dokumentum) legelejére |
| G | Ugrás a fájl (dokumentum) legvégére |
| ng | Ugrás az n-edik sorra (pl. `5g` = ugrás az 5. sorra) |
| /minta (/pattern) | Keresés a szövegben. Az `n` billentyűvel ugorhatsz a következő találatra |
| q | Kilépés a kézikönyvből |

## 2. A `help` parancs használata (Beépített shell parancsokhoz)

Mivel a beépített shell parancsokhoz nincs man oldal, ezekhez a `help` parancsot kell használni. Ez – a `man`-nel ellentétben – nem nyit meg külön lapozható ablakot, hanem egyből a képernyőre (terminálba) írja ki a parancs opcióinak összefoglalóját.

**Szintaxis:**

```bash
help command_name
```

## 3. A `--help` opció (Gyors segítség minden parancshoz)

Ha a man oldalon túl sok az információ, és te csak a parancs opcióit (kapcsolóit) szeretnéd gyorsan átfutni, használhatod a `--help` opciót. Szinte minden Linux parancs támogatja ezt.

**Szintaxis:**

```bash
command_name --help
```

> **Megjegyzés:** Ez nem egy kőbe vésett szabály, néhány parancsnál – pl. az `ip` parancsnál – a súgó eléggé szűkszavú lehet.

## 4. További lehetőségek és alternatívák

- **`info` parancs:** A `man` parancshoz hasonlóan működik, és részletes dokumentációt biztosít:

  ```bash
  info command_name
  ```

- **`tldr` (Too Long; Didn't Read):** Ha a man oldalak túl bonyolultak vagy hosszúak, léteznek harmadik féltől származó, kezdőbarát eszközök, amelyek leegyszerűsítik a leírásokat és gyakorlati példákat mutatnak be (ezt a csomagot külön kell telepíteni).

- **`history` parancs:** A súgók mellett érdemes a `history` parancsot is használni, amellyel visszakeresheted a korábban begépelt parancsaidat.

##Forrás
-[https://itsfoss.com/linux-command-help/]
