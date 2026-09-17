# Adatfolyamok (Data Streams), Csővezetékek (Pipes) és Átirányítás (Redirection) a Linuxban

A Linuxban minden az adatfolyamok körül forog (különösen a szöveges adatfolyamok körül). A parancssori eszközök (amelyeket a cikk írója a "szűrők" helyett inkább "transzformátoroknak" nevez) ezeket az adatokat dolgozzák fel és alakítják át.

## 1. Az STDIO (Standard Input/Output) Alapjai

Az STDIO a Unix és Linux rendszerek alapja, amelyet Ken Thompson fejlesztett ki. Célja, hogy adatokat streameljen az egyik programból/fájlból/eszközből a másikba.

Három alapvető STDIO adatfolyam létezik, amelyek minden program indulásakor automatikusan megnyílnak fájlként (fájlleírókkal / file handles):

| Fájlleíró (ID) | Név | Leírás | Alapértelmezett cél / forrás |
| --- | --- | --- | --- |
| 0 | STDIN (Standard Input) | Szabványos bemenet | Billentyűzet |
| 1 | STDOUT (Standard Output) | Szabványos kimenet | Képernyő (Display) |
| 2 | STDERR (Standard Error) | Szabványos hibakimenet | Képernyő (Display) |

> **Megjegyzés:** Ha a STDOUT-ot átirányítod egy fájlba, a STDERR alapértelmezésben továbbra is a képernyőre íródik, így a hibaüzenetek nem vesznek el.

## 2. Egyszerű adatfolyamok létrehozása (Kísérlet)

Hozzuk létre a kísérleti környezetet a `/tmp` könyvtárban:

```bash
# cd /tmp ; mkdir test ; cd test
```

Hozzunk létre 10 darab fájlt, és töltsük meg őket a `dmesg` parancs kimenetével (egy `for` ciklus segítségével):

```bash
# for I in 0 1 2 3 4 5 6 7 8 9 ; do dmesg > file$I.txt ; done 
```

Ellenőrizzük, hogy a fájlok (`file0.txt`-től `file9.txt`-ig) létrejöttek-e és van-e tartalmuk:

```bash
# ll
```

## 3. A merevlemez felfedezése a `dd` paranccsal

A `dd` (disk dump) parancs kiválóan alkalmas eszközök, partíciók és fájlrendszerek felfedezésére (bár óvatosan kell vele bánni).

**A blokkeszközök (merevlemezek) listázása:**

```bash
[root@studentvm1 test]# lsblk -i
```

**A merevlemez Master Boot Record-jának (első 512 bájtos blokkjának) kiolvasása:**

```bash
[root@studentvm1 test]# dd if=/dev/sda bs=512 count=1
```

**Az első partíció első 100 blokkjának kiolvasása:**

```bash
[root@studentvm1 test]# dd if=/dev/sda1 bs=512 count=100
```

**A TELJES merevlemez tartalmának streamelése a képernyőre:** (Leállításhoz használd a `Ctrl-C` gombot!)

```bash
[root@studentvm1 test]# dd if=/dev/sda
```

**Veszélyes parancs - Klónozás (Ne futtasd!):** A `dd` képes a teljes adatfolyamot egy másik lemezre irányítani, klónozva azt.

```bash
[root@studentvm1 test]# dd if=/dev/sda of=/dev/sdx
```

## 4. Véletlenszerű adatok (Randomness) generálása

A véletlenszerű adatok nagyon hasznosak (pl. jelszógeneráláshoz, merevlemez biztonságos törléséhez). A `/dev/urandom` egy speciális fájl, ami végtelen adatfolyamként szolgáltat véletlenszerű karaktereket.

(A leállításhoz nyomj `Ctrl-C`-t!)

```bash
[student@studentvm1 ~]$ cat /dev/urandom
```

## 5. Csővezetékek (Pipes) építése

A csővezetékeket (Pipes) Douglas McIlroy találta fel. Lehetővé teszik, hogy egy parancs kimenete (STDOUT) egyenesen a következő parancs bemenetébe (STDIN) áramoljon, anélkül, hogy köztes fájlokat kellene létrehozni.

- `|` : A STDOUT átirányítása a következő parancs STDIN-jére.
- `|&` : A STDOUT és a STDERR együttes átirányítása a következő parancs STDIN-jére.

**Példa egy komplex pipeline-ra:**

Ez a parancssor kilistázza a bejelentkezett felhasználókat, levágja a fejlécet, kiveszi az első oszlopot (felhasználónév), sorba rendezi őket, és eltávolítja a duplikációkat, így minden belépett felhasználó csak egyszer jelenik meg:

```bash
[student@studentvm1 ~]$ w | tail -n +3 | awk '{print $1}' | sort | uniq
```

> **Tipp:** A bonyolult csővezetékeket lépésről lépésre, parancsonként érdemes felépíteni és tesztelni.

## 6. Átirányítás fájlokba (Redirection)

Az átirányítással az eredményeket a képernyő helyett fájlokba (vagy eszközökbe) küldhetjük.

**A `>` (nagyobb mint) operátor:** A STDOUT-ot egy fájlba irányítja (ha a fájl létezik, felülírja; ha nem, létrehozza).

Példa (Lemezhasználat mentése fájlba):

```bash
[student@studentvm1 ~]$ df -h > diskusage.txt
```

Ellenőrzés:

```bash
[student@studentvm1 test]# cat diskusage.txt 
```

**A `>>` (dupla nagyobb mint) operátor:** Hozzáfűzi az új adatokat a már létező fájl végéhez (Append).

```bash
[student@studentvm1 ~]$ df -h >> diskusage.txt
```

**A `<` (kisebb mint) operátor:** Egy fájl tartalmát irányítja egy program STDIN-jére. (Bár ritkábban kell használni, mert a legtöbb parancs közvetlenül is elfogadja a fájlnevet argumentumként).

## 7. Szűrés a `grep` paranccsal

A `grep` az egyik leggyakrabban használt eszköz (és valódi "szűrő", mert eldobja a nem kívánt sorokat).

Hozzuk létre a tesztkörnyezetet: Generáljunk 50.000 darab 80 karakteres, nyomtatható karakterekből álló jelszót a `pwgen` paranccsal, és mentsük el őket:

```bash
$ pwgen -sy 80 50000 > random.txt
```

Keressünk rá a fájlban egy konkrét karakterláncra (például a "see" szócskára). A parancs csak azokat a sorokat (jelszavakat) fogja kiírni az adatfolyamból, amelyek tartalmazzák ezt a mintát:

```bash
$ grep see random.txt
```
##Forrás
-[https://opensource.com/article/18/10/linux-data-streams]

# Linux Alapok – I/O, Szabványos Adatfolyamok és Átirányítás (Redirection)

A Linuxban (és a programozásban általánosan) a szabványos adatfolyamok (standard streams) bemeneti és kimeneti (I/O) kommunikációs csatornák egy program és a környezete között. Ezek általában a terminálhoz vannak csatlakoztatva, de átirányító operátorokkal és csővezetékekkel (pipes) megváltoztatható az irányuk.

## 1. Szabványos Adatfolyamok és Fájlleírók (File Descriptors)

A Linux a három szabványos adatfolyamot fájlleíróként (számokként) tartja nyilván.

| Fájlleíró (ID) | Név | Rövidítés | Leírás |
| --- | --- | --- | --- |
| 0 | Standard Input | stdin | Szabványos bemenet |
| 1 | Standard Output | stdout | Szabványos kimenet |
| 2 | Standard Error | stderr | Szabványos hibakimenet |

> **Megjegyzés:** Nem csak ez a három fájlleíró létezik, egyedi fájlleírók is létrehozhatók, de ezek a leggyakoribbak.

## 2. Átirányító Operátorok (Redirection Operators)

- `<` (Bemeneti operátor): Bemenetet biztosít egy parancsnak.
- `<>` (Bemeneti operátor - rw): Ugyanaz, mint a `<`, de olvasási/írási módba nyitja meg a fájlt (ha nem létezik, létrehozza). Nagyon ritkán használatos.
- `>` (Kimeneti operátor): A parancs kimeneti adatfolyamát irányítja fájlba. Ha a fájl létezik, felülírja, ha nem, létrehozza.
- `>>` (Kimeneti hozzáfűző operátor): Ugyanaz, mint a `>`, de felülírás helyett hozzáfűzi az adatokat a fájl végéhez.
- `&>` és `&>>` (Nem szabványos operátorok): Mind a szabványos kimenetet (stdout), mind a hibakimenetet (stderr) átirányítja egyszerre. Az `&>` felülír, az `&>>` hozzáfűz.
- `|` (Csővezeték / Pipe operátor): Az egyik parancs kimenetét átadja a másik parancs bemenetének. (Mint a vízvezeték, ami átviszi a vizet az egyik tartályból a másikba).

## 3. Szabványos Bemenet (STDIN - 0)

A szabványos bemenet a programba beáramló adat (legtöbbször a billentyűzetről, de lehet átirányított is).

**Bemenet a billentyűzetről:** A `cat` parancs alapértelmezésben a szabványos bemenetet várja és a képernyőre nyomtatja. (Kilépéshez nyomd meg a `CTRL+D` kombinációt!)

```bash
$ cat
Your lips move but I can't hear what you're saying
Your lips move but I can't hear what you're saying
$
```

**Bemenet fájlból (argumentumként):** A `cat` a billentyűzet helyett a megadott fájlt veszi bemenetnek.

```bash
$ cat ~/.bashrc
```

**Bemenet átirányítással (`<`):** Ugyanazt az eredményt éri el, de itt az átirányító operátort használjuk:

```bash
$ cat < ~/.bashrc
```

**Bemenet adatcsövön keresztül (`|`):** A `head` parancs kimenete lesz a `cat` bemenete.

```bash
$ head ~/.bashrc | cat
```

## 4. Szabványos Kimenet (STDOUT - 1)

A szabványos kimenet az az adatfolyam, amely alapértelmezésben a képernyőre (terminálra) kerül.

**Kimenet a képernyőre:** A `stat` parancs statisztikát ír ki a megadott fájlról.

```bash
$ stat ~/.bashrc
   File: /home/savona/.bashrc
   Size: 192           Blocks: 8          IO Block: 4096   regular file
 Device: fd02h/64770d    Inode: 13668265    Links: 1
... OUTPUT TRUNCATED ...
```

**Kimenet átirányítása fájlba (`>` vagy `>>`):** Célzott mentéshez (felülírással vagy hozzáfűzéssel).

```bash
$ stat ~/.bashrc > output.txt
$ stat ~/.bashrc >> output.txt
```

Ellenőrzés:

```bash
$ cat output.txt 
   File: /home/savona/.bashrc
...
```

**Kimenet adatcsövön keresztül (`|`):** A kimenet átadása a `grep` parancsnak.

```bash
$ stat ~/.bashrc | grep -i modify
 Modify: 2019-02-03 09:03:25.750855221 -0500
```

## 5. Szabványos Hibakimenet (STDERR - 2)

Ide kerülnek a folyamat által generált hibaüzenetek. Alapértelmezésben ez is a képernyőre íródik, de elkülönül a normál kimenettől.

**Hibaüzenet a képernyőn** (mert nem létezik a fájl):

```bash
$ ls zepplin
 ls: cannot access 'zepplin': No such file or directory
```

**Hibakimenet átirányítása fájlba (`2>` vagy `2>>`):** Fontos a `2`-es azonosító használata!

```bash
$ ls zepplin 2> errors.txt
$ ls zepplin 2>> errors.txt
```

Ellenőrzés:

```bash
$ cat errors.txt 
 ls: cannot access 'zepplin': No such file or directory
```

**Hibakimenet adatcsövön keresztül (Trükkös!):**

A hibakimenetet nem lehet közvetlenül a pipe-ba (`|`) küldeni, a normál kimenettel ellentétben (a hibaüzenet kikerüli a pipe-ot és a képernyőn landol):

```bash
$ ls zepplin floyd stones | grep zepplin
 ls: cannot access 'zepplin': No such file or directory
 ls: cannot access 'floyd': No such file or directory
 ls: cannot access 'stones': No such file or directory
```

**A MEGOLDÁS:** A hibakimenetet (`2`) előbb a normál kimenetbe (`1`) kell irányítani a `2>&1` kifejezéssel, és utána már átmegy a pipe-on!

```bash
$ ls zepplin floyd stones 2>&1 | grep zepplin
 ls: cannot access 'zepplin': No such file or directory
```

## 6. Fájlleírók Haladó Használata Átirányításnál

Különböző adatfolyamok finomhangolt átirányítása egy parancson belül.

**A STDOUT és STDERR átirányítása KÉT KÜLÖN fájlba:**

```bash
$ ls zepplin floyd stones 1> output.txt 2> errors.txt
```

(A sikeres listázások az `output.txt`-be, a hibák az `errors.txt`-be kerülnek).

**A STDOUT és STDERR átirányítása UGYANABBA a fájlba:**

```bash
$ ls zepplin floyd stones >alloutput.txt 2>&1
```

VAGY a nem szabványos (rövidített) operátorral:

```bash
$ ls zepplin floyd stones &> alloutput.txt
```

(A sikeres és hibás kimenet is az `alloutput.txt`-ben köt ki).

## 7. Végleges Átirányítás Szkriptekben (exec)

Ha Bash szkriptet írsz, és nem akarod minden egyes sor végére odaírni az átirányítást, az `exec` paranccsal globálisan és véglegesen átirányíthatod a szkript adatfolyamait.

```bash
#!/bin/bash

# Az összes stdout (normál kimenet) átirányítása az output.txt fájlba
exec 1> output.txt

# Az összes stderr (hibaüzenet) átirányítása az errors.txt fájlba
exec 2> errors.txt
```

##Forrás
-[https://www.putorius.net/linux-io-file-descriptors-and-redirection.html]
