# A Linux `paste` Parancs Használata (Sorok egyesítése fájlokból)

A `paste` egy kevésbé ismert Linux/Unix parancssori segédprogram, amely a fájlok sorait vízszintesen fésüli össze. A bemeneti fájlok megegyező sorszámú sorait teszi egymás mellé, alapértelmezés szerint egy TAB (tabulátor) karakterrel elválasztva.

## 1. Alapszintaxis

```bash
paste [OPTION]... [FILE]...
```

(Ha nincs fájl megadva, vagy a fájl argumentum `-`, akkor a `paste` a szabványos bemenetről (stdin) olvas).

## 2. Alapvető Működés és Példák

Tételezzük fel, hogy van két fájlunk:

**file1:**

```plaintext
Iron Man
Thor
Captain America
Hulk
Spider Man
```

**file2:**

```plaintext
Black Widow
Captain Marvel
Dark Phoenix
Nebula
```

Ha opciók nélkül futtatjuk, a `paste` beolvassa a fájlokat, és a megfelelő sorokat TAB-bal elválasztva írja ki:

```bash
paste file1 file2
```

**Kimenet:**

```plaintext
Iron Man        Black Widow
Thor    Captain Marvel
Captain America Dark Phoenix
Hulk    Nebula
Spider Man
```

(Különböző hosszúságú fájlok esetén: A `paste` addig fut, amíg az összes fájl a végére nem ér. A rövidebb fájl hiányzó sorai helyén egy üres mező szerepel, de az elválasztó karakter megmarad – ahogy a fenti példában a "Spider Man" után).

**Kimenet átirányítása fájlba:**

Ha a képernyő helyett fájlba akarod menteni az eredményt, használd a `>` (felülírás) vagy a `>>` (hozzáfűzés) operátorokat:

```bash
paste file1 file2 > file3
```

## 3. Hasznos Opciók és Kapcsolók (Flags)

### Egyedi elválasztó megadása (`-d`, `--delimiters`)

A TAB helyett más elválasztó karaktert is megadhatsz.

Példa alulvonás (`_`) használatára:

```bash
paste -d '_' file1 file2
```

**Kimenet:**

```plaintext
Iron Man_Black Widow
Thor_Captain Marvel
Captain America_Dark Phoenix
Hulk_Nebula
Spider Man_
```

**Több elválasztó karakter használata:**

Ha egy listát adsz meg, a `paste` azokat felváltva (körkörösen) fogja használni. Amikor a lista végére ér, újra az első karakterrel folytatja.

```bash
paste -d '%|' file1 file2 file1
```

(Itt az 1. és 2. fájl között `%`, a 2. és 3. fájl között `|` lesz az elválasztó).

**Kimenet:**

```plaintext
Iron Man%Black Widow|Iron Man
Thor%Captain Marvel|Thor
Captain America%Dark Phoenix|Captain America
Hulk%Nebula|Hulk
Spider Man%|Spider Man
```

### Sorosítás / Fájlok egy sorba fűzése (`-s`, `--serial`)

Az `-s` opció megváltoztatja az alapműködést: nem oszloponként halad, hanem egy adott fájl összes sorát egyetlen vízszintes sorba vonja össze, majd a következő fájllal egy új sorban teszi ugyanezt.

```bash
paste -s file1 file2
```

**Kimenet:**

```plaintext
Iron Man        Thor    Captain America Hulk    Spider Man
Black Widow     Captain Marvel  Dark Phoenix    Nebula
```

### Null-karakteres lezárás (`-z`, `--zero-terminated`)

Újsor karakter (newline) helyett NUL (null) karakterrel választja el a sorokat. Ez különösen akkor hasznos, ha a `paste`-t a `find -print0` és `xargs -0` parancsokkal együtt használod speciális karaktereket (pl. szóközöket) tartalmazó fájlnevek kezelésére.

## 4. Haladó Trükkök és Gyakori Kérdések (FAQ)

**Mi a különbség a `cat` és a `paste` között?**

A `cat` függőlegesen fűzi össze a fájlokat (az egyik fájl végét a másik elejéhez ragasztja). A `paste` vízszintesen (egymás mellé, oszlopszerűen) rendezi a megfelelő sorokat.

**Hogyan egyesíthetek minden második sort egyetlen fájlon belül?**

Használd a `-` karaktert kétszer argumentumként, ami azt jelenti, hogy a stdin-ről olvas kétszer:

```bash
paste - - < file
```

(Minden egymást követő sorpár egyetlen kimeneti sorba kerül, TAB-bal elválasztva).

**Készíthetek CSV fájlt a `paste` segítségével?**

Igen, a `-d ','` használatával vesszővel elválasztott kimenetet kapsz:

```bash
paste -d ',' file1 file2 > output.csv
```

## 5. Gyors Referencia (Összefoglaló Táblázat)

| Parancs | Leírás |
| --- | --- |
| `paste file1 file2` | Fájlok egyesítése egymás mellé (TAB-bal elválasztva). |
| `paste -d ',' file1 file2` | Egyedi elválasztó használata (pl. vessző CSV-hez). |
| `paste -d '%|' file1 file2 file3` | Több elválasztó karakter körkörös használata. |
| `paste -s file1` | Egy fájl összes sorának egyesítése egyetlen vízszintes sorba. |
| `paste -s file1 file2` | Fájlok szerializálása: minden fájl tartalma egy-egy különálló, saját vízszintes sorba kerül. |
| `paste -z file1 file2` | NUL karakter használata sorválasztóként az újsor (newline) helyett. |
| `paste - - < file` | Egyetlen fájl minden két egymást követő sorának egymás mellé (egy sorba) fűzése. |
| `paste file1 file2 > out.txt` | Az egyesített kimenet fájlba irányítása (mentése). |

##Forrás
-[https://linuxize.com/post/paste-command-in-linux/]

# A Linux `paste` Parancs Haladó Használata és Példák

Míg a `cut` parancs oszlopokat vág ki az adatokból, a `paste` parancs pont az ellenkezőjét teszi: több bemeneti fájlt von össze sorról sorra (oszlopokká fűzve őket), hogy egy új, elválasztójelekkel tagolt szövegfájlt hozzon létre.

## 1. Oszlopok összefűzése (Pasting columns)

A leggyakoribb használati mód, amikor N darab bemeneti fájlt párhuzamosan, sorról sorra fűzünk össze. Az alapértelmezett elválasztó karakter (delimiter) a TAB (`\t`).

**Példa alap bemenetek generálására és összefűzésére:**

```bash
sh$ printf "%s\n" {a..e} | tee letters
sh$ printf "%s\n" {1..5} | tee digits

sh$ paste letters digits
a    1
b    2
c    3
d    4
e    5
```

**Több CSV fájl összefűzése:**

```bash
sh$ paste *.csv | head -3
```

Bár a konzolon az oszlopok nem mindig tűnnek tökéletesen igazítottnak, a `sed` paranccsal láthatóvá tehetjük a rejtett TAB karaktereket (`\t`):

```bash
sh$ paste *.csv | head -3 | sed -n l
ACCOUNTLIB\tACCOUNTNUM\tCREDIT\tDEBIT$
TIDE SCHEDULE\t623477\t\t00000001615,00$
```

(Ha két `\t` van egymás mellett, az egy üres mezőt jelent).

## 2. Az elválasztó karakter megváltoztatása (`-d`)

A `-d` opcióval a TAB helyett más elválasztót (pl. pontosvesszőt) is megadhatunk. (Érdemes idézőjelbe tenni, hogy a shell ne parancselválasztóként értelmezze).

```bash
sh$ paste -d ';' *.csv | head -3
ACCOUNTLIB;ACCOUNTNUM;CREDIT;DEBIT
TIDE SCHEDULE;623477;;00000001615,00
```

## 3. Adatok transzponálása Soros módban (`-s`)

Alapértelmezésben a `paste` "párhuzamosan" olvas. Ha az `-s` (serial) kapcsolót használjuk, "soros" módba vált: először az első fájl összes sorát beolvassa és egyetlen vízszintes kimeneti sorba fűzi, majd a második fájllal csinálja ugyanezt a második sorban. (Lényegében felcseréli a sorokat és az oszlopokat - transzponál).

**Párhuzamos (alap) mód:**

```bash
sh$ paste *.sample
ACCOUNTLIB    ACCOUNTNUM
TIDE SCHEDULE    623477
```

**Soros mód (`-s`):**

```bash
sh$ paste -s *.sample
ACCOUNTLIB    TIDE SCHEDULE    VAT BS/ENC    PAYABLES    ACCOMMODATION GUIDE
ACCOUNTNUM    623477    445452    4356    623372
```

## 4. Munka a szabványos bemenettel (stdin / `-`)

A `paste` képes a szabványos bemenetről is olvasni, amit a `-` (kötőjel) fájlnév jelöl.

### 4.1. Egy fájl összes sorának egyesítése

A soros mód (`-s`) segítségével egy fájl összes sorát egyetlen hosszú sorrá fűzhetjük össze:

```bash
sh$ head -5 ACCOUNTLIB.csv | paste -s -d':'
ACCOUNTLIB:TIDE SCHEDULE:VAT BS/ENC:PAYABLES:ACCOMMODATION GUIDE
```

(Különbség a `tr '\n' ':'` paranccsal szemben: A `tr` a legutolsó újsor karaktert is elválasztóra cseréli, míg a `paste` érintetlenül hagyja a sor végi lezáró újsor karaktert).

### 4.2. Többoszlopos formázás egyetlen bemenetből

Ha a `-` jelet többször adjuk meg, a `paste` ugyanabból a bemeneti adatfolyamból olvas egymás után. Ez hasznos adatok "tördelésére":

```bash
sh$ seq 9 | paste - - -
1    2    3
4    5    6
7    8    9
```

(A fenti parancs 3 oszlopba rendezi az egymás után következő számokat).

> **Fontos buktató:** A több `-` használata soros (`-s`) módban nem fog működni, mert az első `-` kimeríti a teljes bemeneti adatfolyamot, a többi számára pedig nem marad adat.

## 5. Különböző hosszúságú fájlok kezelése (Hiányzó adatok)

Ha a `paste` eltérő hosszúságú fájlokat fűz össze, a hiányzó adatokat egyszerűen üres mezőkkel (blanks) helyettesíti az adott fájl végén.

- **A probléma:** A `paste` csak a sorok fizikai pozícióját látja, a tartalmukat nem. Ha egy fájlból mondjuk a 3. sor hiányzik, a `paste` nem fogja "összepárosítani" az adatokat, hanem mindent elcsúsztat, és a fájl legvégére teszi az üres mezőt.

- **A megoldás:** Ha hiányos adatokkal (pl. adatbázisszerű relációkkal) dolgozol, a `paste` helyett a `join` parancsot kell használni, amely a tartalom (kulcs) alapján párosít.

## 6. Elválasztók körkörös használata (Cycling over delimiters)

A `-d` opciónak több karaktert is megadhatunk. Ilyenkor a `paste` ezeket felváltva (körkörösen) használja a mezők között.

```bash
sh$ paste -d':+-' ACCOUNT*.csv CREDIT.csv DEBIT.csv | head -5
ACCOUNTLIB:ACCOUNT NUM+CREDIT-DEBIT
TIDE SCHEDULE:623477+-00000001615,00
```

**Trükk extra elválasztókhoz (`/dev/null`):**

Mivel elválasztó csak két létező mező közé kerülhet, ha egyedi formázást akarunk, beiktathatjuk a `/dev/null`-t "üres" oszlopként. Zárójelezés példája:

```bash
sh$ paste -d'()' ACCOUNT*.csv /dev/null | head -5
ACCOUNTLIB(ACCOUNTNUM)
TIDE SCHEDULE(623477)
```

Ha a `\` (visszaperjel) karaktert akarod elválasztóként használni, duplázni kell (`\\`):

```bash
sh$ paste -d'/\\' - ACCOUNT*.csv CREDIT.csv DEBIT.csv - < /dev/null
```

## 7. Több-bájtos (UTF-8) karakterek elválasztóként

A `paste` parancs egyik nagy hátránya, hogy a régi Unix korszakból származik, így csak 1 bájtos karaktereket fogad el elválasztóként.

Ha egy UTF-8 speciális karaktert (pl. `◇`, ami bájtokban `e2 97 87`) adsz meg a `-d` után, a `paste` nem egyetlen karakterként kezeli, hanem körkörösen fogja használni a 3 különálló bájtot! Emiatt a kimenetben ismeretlen karakterek (pl. ``) jelennek meg.

A megoldás ilyen esetekben is a `/dev/null` trükk vagy a `sed` / `awk` használata.

## 8. Bónusz: A `\0` csapdája és a `-z` opció

- **Fájlok összefűzése elválasztó NÉLKÜL:** Történelmi okokból a `paste -d ""` hibát dobhat bizonyos rendszereken. A hordozható és hivatalos megoldás az elválasztó teljes elhagyására a `\0` megadása:

  ```bash
  paste -d "\0"
  ```

  > **Figyelem:** Itt a `\0` jelentése "nincs elválasztó", és NEM a NUL bájtot jelenti.

- **A GNU `-z` opciója (NUL karakter használata):**

  A GNU `paste` verziója rendelkezik egy `-z` kapcsolóval. Ez arra utasítja a parancsot, hogy az újsor (newline / `\n`) helyett a NUL (`\0` vagy `0x00` bájt) karaktert tekintse sorválasztónak a bemeneten és a kimeneten is. Ez fájlnevek biztonságos feldolgozásánál hasznos.

**Példa a `-z` használatára:**

Készítsünk NUL-ra végződő (zero-terminated) fájlokat:

```bash
sh$ tr '\n' '\0' < ACCOUNTLIB.csv > ACCOUNTLIB.zero
sh$ tr '\n' '\0' < ACCOUNTNUM.csv > ACCOUNTNUM.zero
```

Fűzzük össze őket a `-z` opcióval:

```bash
sh$ paste -z *.zero | tr '\0' '\n' | head -3
```
#Forrás
-[https://linuxhandbook.com/paste-command/]
