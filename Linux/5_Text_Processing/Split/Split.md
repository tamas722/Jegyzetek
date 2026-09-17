# A Linux `split` Parancs Használata és Példák (Nagy fájlok darabolása)

A `split` parancs egy rendkívül hasznos eszköz a Linuxban, amellyel hatalmas méretű fájlokat "vághatunk" kisebb, könnyebben kezelhető darabokra. Ez különösen hasznos lehet, ha e-mail mellékletként, vagy fájlátviteli protokollokon (FTP, SCP) keresztül akarunk adatot küldeni, ahol méretkorlátok vannak, illetve nagy adatbázisok feldolgozásánál és biztonsági mentéseknél.

## 1. Alapszintaxis és Alapműködés

**Szintaxis:**

```bash
split [opciók] [kívánt_méret] eredeti_fájl [kimeneti_prefix]
```

A `split` alapértelmezés szerint a feldarabolt fájlokat a munkakönyvtárba menti, és a nevük `xaa`, `xab`, `xac` stb. lesz (az "x" az alapértelmezett prefix).

**Darabolás méret alapján (`-b`):**

Egy fájl 100 MB-os darabokra vágása:

```bash
split -b 100M largefile
```

**Darabolás sorok száma alapján (`-l`):**

Egy szöveges fájl (pl. log vagy CSV) feldarabolása úgy, hogy minden kimeneti fájl pontosan 500 sort tartalmazzon:

```bash
split -l 500 bigfile.txt
```

## 2. A leggyakrabban használt kapcsolók (Flags)

| Kapcsoló | Leírás | Példa |
| --- | --- | --- |
| `-b` | Megadja minden kimeneti fájl méretét (byte, K, M, G). | `split -b 100M largefile` |
| `-l` | Megadja, hogy egy kimeneti fájl hány sort tartalmazzon. | `split -l 500 largefile` |
| `-d` | Betűk helyett számokat használ az utótaghoz (x00, x01...). | `split -d largefile` |
| `-a` | Beállítja az utótag (suffix) hosszát. | `split -a 4 largefile` |
| `--verbose` | Diagnosztikai üzenetet ír ki minden egyes kimeneti fájl megnyitása előtt. | `split --verbose largefile` |
| `--additional-suffix` | Egyedi kiterjesztést fűz a fájlnevek végéhez. | `split --additional-suffix=.txt largefile` |
| `--numeric-suffixes` | Ugyanaz, mint a `-d` kapcsoló. | `split --numeric-suffixes largefile` |
| `--filter` | Shell parancsot futtat minden egyes darabon (pl. tömörítés). | `split --filter='gzip > $FILE.gz' largefile` |

## 3. Haladó Használat és Példák

**Kimeneti fájlnevek testreszabása (`-d` és `-a`):**

Ha azt szeretnéd, hogy a fájlok számozottak legyenek, és a sorszám 4 karakter hosszú legyen (x0000, x0001...):

```bash
split -d -a 4 largefile
```

**Szűrők (Filter) használata tömörítéshez:**

A feldarabolt részeket rögtön, röptében tömöríthetjük a `gzip` segítségével a `--filter` opcióval. A kimeneti fájlok nevei `xaaa.gz`, `xaab.gz` stb. lesznek:

```bash
split --filter='gzip > $FILE.gz' largefile
```

**Egyedi fájlnév-prefix megadása:**

Az `x` helyett saját előtagot is megadhatsz:

```bash
split -b 50M largefile splitfile
# Eredmény: splitfileaa, splitfileab, stb.
```

## 4. Alternatívák a `split` parancsra

**A `dd` parancs:** Általában másolásra használják, de fájlok darabolására is alkalmas, ha finomabb kontrollra van szükség a blokkméretek felett. (Bár nem bontja fel automatikusan a teljes fájlt apró darabokra).

```bash
dd if=largefile of=smallfile bs=1M count=100
# Létrehoz egy 'smallfile' nevű fájlt, amely a 'largefile' első 100 MB-ját tartalmazza.
```

**Harmadik féltől származó grafikus (GUI) eszközök:** Ilyen például a GSplit, HJSplit vagy a KFK File Splitter. Ezek felhasználóbarátok, de telepítést igényelnek, és szkriptekben nem jól használhatók.

| Módszer | Rugalmasság | Egyszerű használat | Telepítést igényel? |
| --- | --- | --- | --- |
| Split parancs | Magas | Magas | Nem |
| `dd` parancs | Közepes | Közepes | Nem |
| GUI Eszközök | Alacsony | Magas | Igen |

## 5. Gyakori Problémák és Hibaelhárítás

**Nem egyenletes fájlméretek:**

Ha az eredeti fájl mérete nem pontos többszöröse a megadott darabméretnek, akkor az utolsó fájl kisebb lesz, mint a többi. Ez normális működés (by design).

**Speciális karakterek a fájlnévben:**

Ha a fájlnév szóközt vagy más speciális karaktert tartalmaz, tedd idézőjelek közé:

```bash
split -b 100M "large file"
```

**A darabolt fájlok újraegyesítése és az integritás ellenőrzése:**

Az átvitel után a darabokat a `cat` paranccsal lehet újra összerakni. Hogy biztosak lehessünk abban, hogy a fájl nem sérült meg, generáljunk MD5 ellenőrzőösszeget (`md5sum`) az eredetiről, majd hasonlítsuk össze az újraegyesített fájl ellenőrzőösszegével:

```bash
# Eredeti fájl ellenőrzőösszege:
md5sum largefile

# A darabok (xaa, xab, xac) újraegyesítése és ellenőrzése:
cat xaa xab xac | md5sum
```

(Ha a két kimenet megegyezik, az újraegyesített fájl tökéletesen azonos az eredetivel).

## 6. A `split` széleskörű alkalmazása (Biztonsági mentések és Adatfeldolgozás)

**Nagy adathalmazok feldolgozása:**

Egy hatalmas CSV fájlt feldarabolhatunk 10 000 soros részekre, hogy párhuzamosan vagy részletekben lehessen feldolgozni és elemezni:

```bash
split -l 10000 large_dataset.csv
```

**Biztonsági mentés (Backup) stratégiák:**

Ha egy nagy könyvtárat archiválsz `tar`-ral külső tárolóra (pl. FAT32 pendrive), ahol fájlméret-korlátozás van (pl. max 4GB), a `split` segítségével kapásból 2GB-os darabokra oszthatod az archívumot:

```bash
tar cf - /path/to/directory | split -b 2G - backup.tar.
# A végeredmény: backup.tar.aa, backup.tar.ab stb. (2GB-os darabokban).
```

(Extra tipp: Egy fájl inode-számát és alapvető metadata linkjét az `ls -i largefile` paranccsal ellenőrizheted).

##Forrás
-[https://ioflood.com/blog/split-linux-command/]

# A Linux `split` Parancs: 9 Hasznos Példa

A `split` paranccsal nagy fájlokat bonthatunk több, kisebb fájlra. Számos módja van a paraméterek testreszabásának attól függően, hogy sorok száma, fájlméret, vagy darabszám alapján akarunk darabolni.

**Alapszintaxis:**

```bash
split [options] filename [prefix]
```

> **Tipp:** A darabolás után a `wc` (word count) vagy az `ls -l` (`ll`) paranccsal érdemes ellenőrizni a létrejött fájlokat.

## 1. Darabolás az alapértelmezett beállításokkal

Ha nem adunk meg opciókat, a `split` 1000 soronként hoz létre új fájlokat. Ha nem adunk meg prefixet (előtagot), alapértelmezésben az `x` betűt használja. A fájlok `xaa`, `xab`, `xac` stb. néven jönnek létre.

```bash
split someLogFile.log
```

Ha a fájl 17.170 soros, akkor 17 darab 1000 soros fájl jön létre, és egy utolsó fájl a maradék 170 sorral.

> **Tipp:** A `--verbose` kapcsolóval kiírathatod, hogy a program éppen melyik fájlt hozza létre: `split someLogFile.log --verbose`.

## 2. Darabolás megadott sorszám alapján (`-l`)

A `-l` (lines) opcióval felülbírálhatod az 1000 soros alapértelmezést. Például, ha 500 soronként akarod vágni a fájlt:

```bash
split someLogFile.log -l 500
```

**Ellenőrzés:**

```bash
chris@discodingo:~/Documents$ wc -l xbh xbi
500 xbh
170 xbi
```

## 3. Fájl darabolása "N" számú darabra (`-n`)

Ha nem a sorok száma, hanem a darabok száma a fontos, használd az `-n` kapcsolót. Ha pontosan 15 darabra akarod osztani a fájlt:

```bash
split someLogFile.log -n 15
```

## 4. Egyedi fájlnév-előtag (Prefix) használata

Az alapértelmezett `x` betű helyett saját előtagot is megadhatsz, egyszerűen úgy, hogy a forrásfájl után beírod a kívánt nevet.

```bash
split someLogFile.log someSeparatedLogFiles.log_
```

**Kimenet:** `someSeparatedLogFiles.log_aa`, `someSeparatedLogFiles.log_ab` stb.

## 5. Az utótag (Suffix) hosszának beállítása (`-a`)

Alapértelmezésben az utótag hossza 2 karakter (`aa`, `ab`). Az `-a` kapcsolóval megadhatod a kívánt hosszt. Például 5 karakter hosszú utótag:

```bash
split someLogFile.log someSeparatedLogFiles.log_ -a 5
```

**Kimenet:** `someSeparatedLogFiles.log_aaaaa`, `someSeparatedLogFiles.log_aaaab` stb.

## 6. Számozott (Numerikus) utótagok használata (`-d`)

A betűkódok helyett számokat is használhatsz, ami sokkal átláthatóbbá teszi a fájlok sorrendjét. Erre való a `-d` opció. (A fenti `-a 5`-tel kombinálva):

```bash
split someLogFile.log someSeparatedLogFiles.log_ -a 5 -d
```

**Kimenet:** `someSeparatedLogFiles.log_00000`, `someSeparatedLogFiles.log_00001` stb.

## 7. Hexadecimális utótagok használata (`-x`)

A `-x` kapcsoló hexadecimális (betűket és számokat váltakozva használó) utótagokat hoz létre.

Példa: Vágjuk a fájlt 50 darabra (`-n 50`), 6 karakter hosszú (`-a 6`), hexadecimális utótaggal (`-x`), és `_` előtaggal:

```bash
split someLogFile.log _ -x -n50 -a6
```

**Kimenet:** `_000000`, `_000001` ... `_00000a`, `_00000b` stb.

## 8. Darabolás konkrét fájlméret alapján (`-b`)

A `-b` (bytes) opcióval méret alapján darabolhatsz. Használhatod a `K`, `M`, `G` (Kibi/Mebi/Gibi = 1024 többszöröse), vagy a `KB`, `MB`, `GB` (Kilo/Mega/Giga = 1000 többszöröse) utótagokat.

Fájl darabolása pontosan 128 KB-os részekre, numerikus utótaggal:

```bash
split someLogFile.log someSeparatedLogFiles.log_ -d -b 128KB
```

## 9. Darabolás "Maximum" méret alapján, a sorok megtartásával (`-C`)

A `-b` hátránya, hogy a mérethatárnál kíméletlenül elvágja a fájlt, így sorok (szavak) törhetnek ketté.

A `-C` opció megadja a maximális méretet (pl. 1MB), de a `split` intelligensen a legutolsó teljes sornál vágja el a fájlt, így az adatszerkezet (sorstruktúra) sértetlen marad.

```bash
split someLogFile.log someNewLogFiles.log_ -d -C 1MB
```

## Bónusz Tipp: A darabolt fájlok újraegyesítése (Rejoining)

Ha a darabokat átküldted a célgépre, a `cat` (concatenate) paranccsal újra össze tudod fűzni őket az eredeti fájllá. Mivel a fájlok ábécé (vagy numerikus) sorrendben vannak, a `*` (asterisk/joker) karakterrel az összes darabot egyszerre befoghatod.

Tegyük fel, hogy a darabok `xaa`, `xab`, `xac` néven vannak:

```bash
cat x* > recoveredLogFile.log
```

Ha leellenőrzöd (`wc -l recoveredLogFile.log`), látni fogod, hogy a helyreállított fájl sorainak száma és tartalma tökéletesen megegyezik az eredetivel.

##Forrás
-[https://linuxhandbook.com/split-command/]
