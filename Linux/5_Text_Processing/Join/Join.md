# A Linux `join` Parancs Használata (Fájlok egyesítése)

A `join` parancs segítségével több fájl adatait tudjuk egyesíteni (összefűzni) egy közös mező alapján. Úgy működik, mint az adatbázis-kezelésből (SQL) ismert JOIN művelet.

Képzeljünk el egy HR forgatókönyvet, ahol az alkalmazottak adatai különböző fájlokban (adatbázisokban) vannak szétszórva.

## 1. Előkészületek és az Alapvető Egyesítés (Basic Join)

Tételezzük fel, hogy van két fájlunk:

**employees.txt** (ID, Név, Részleg):

```plaintext
1001 John Engineering
1002 Sarah Marketing
1003 Mike Sales
1004 Emily HR
1005 David Finance
```

**salaries.txt** (ID, Fizetés):

```plaintext
1001 75000
1002 65000
1003 70000
1004 60000
1005 80000
```

Az adatok egyesítéséhez futtasd az alábbi parancsot:

```bash
join employees.txt salaries.txt
```

**Kimenet:**

```plaintext
1001 John Engineering 75000
1002 Sarah Marketing 65000
1003 Mike Sales 70000
1004 Emily HR 60000
1005 David Finance 80000
```

**Mi történt?**

- A `join` parancs alapértelmezésben mindkét fájl első mezőjét (az alkalmazott ID-ját) vette alapul.
- Amikor egyező ID-t talált, egyesítette a két fájl adott sorait.
- A kimenetben az ID szerepel először, majd az első fájl többi mezője, végül a második fájl többi mezője (az ismétlődő ID nélkül).

## 2. A Kimenet Testreszabása (`-o`)

Néha meg akarjuk változtatni a mezők sorrendjét a kimenetben, vagy csak bizonyos mezőkre van szükségünk. Ezt az `-o` (output format) kapcsolóval tehetjük meg.

**Parancs:**

```bash
join -o 1.2,1.3,2.2,1.1 employees.txt salaries.txt
```

**Kimenet:**

```plaintext
John Engineering 75000 1001
Sarah Marketing 65000 1002
Mike Sales 70000 1003
Emily HR 60000 1004
David Finance 80000 1005
```

**Az `-o` utáni formátum magyarázata (Fájl.Mező):**

- `1.2`: 1. fájl, 2. mező (Név)
- `1.3`: 1. fájl, 3. mező (Részleg)
- `2.2`: 2. fájl, 2. mező (Fizetés)
- `1.1`: 1. fájl, 1. mező (ID)

## 3. Párosítatlan (Unmatched) Rekordok Kezelése (`-a`)

A valóságban előfordulhat, hogy egy rekord benne van az egyik fájlban, de nincs hozzá pár a másikban.

Szimuláljuk ezt egy új alkalmazott hozzáadásával, akinek még nincs fizetése:

```bash
echo "1006 Alex IT" >> employees.txt
```

Ha most lefuttatjuk a sima `join employees.txt salaries.txt` parancsot, Alex nem jelenik meg, mert nincs egyezés a fizetés fájlban.

Hogy a párosítatlan rekordok is bekerüljenek (hasonlóan a "Left Outer Join"-hoz), használd az `-a` opciót:

```bash
join -a 1 employees.txt salaries.txt
```

**Kimenet:**

```plaintext
1001 John Engineering 75000
1002 Sarah Marketing 65000
1003 Mike Sales 70000
1004 Emily HR 60000
1005 David Finance 80000
1006 Alex IT
```

(Az `-a 1` utasítja a `join`-t, hogy az 1. fájl minden sorát írja ki, még akkor is, ha nincs párja a 2. fájlban).

## 4. Egyesítés Eltérő Mezők Alapján (`-1` és `-2`)

Eddig az első mező alapján egyesítettünk. De mi van, ha a fájlok felépítése eltérő?

Hozzunk létre egy `departments.txt` fájlt, ami a részlegneveket és a rövidítéseiket tartalmazza:

```plaintext
Engineering ENG
Marketing MKT
Sales SLS
HR HRS
Finance FIN
IT ITS
```

Most egyesítsük ezt a fájlt az `employees.txt`-vel a Részleg neve alapján:

```bash
join -1 3 -2 1 employees.txt departments.txt
```

**Kimenet:**

```plaintext
Engineering 1001 John ENG
Marketing 1002 Sarah MKT
Sales 1003 Mike SLS
HR 1004 Emily HRS
Finance 1005 David FIN
IT 1006 Alex ITS
```

**A kapcsolók magyarázata:**

- `-1 3`: Az első fájl (`employees.txt`) 3. mezőjét használja kapcsolódási pontként.
- `-2 1`: A második fájl (`departments.txt`) 1. mezőjét használja kapcsolódási pontként.

## 5. További Hasznos `join` Kapcsolók (Gyors Referencia)

Ezek a kapcsolók további rugalmasságot biztosítanak adatok feldolgozásakor:

| Kapcsoló | Leírás |
| --- | --- |
| `-t CHAR` | Egyedi bemeneti és kimeneti mezőelválasztó (szóköz helyett, pl. CSV-hez: `-t ','`). |
| `-i` | Kis- és nagybetűk figyelmen kívül hagyása az összehasonlításnál (case-insensitive). |
| `-e STRING` | A hiányzó (üres) bemeneti mezők helyettesítése a megadott STRING-gel. |
| `-j FIELD` | Ugyanaz, mint a `-1 FIELD -2 FIELD` (ha mindkét fájlban ugyanaz az oszlopszám az egyezés alapja). |
| `-v FILE_NUMBER` | Hasonló az `-a`-hoz, de KIZÁRÓLAG a párosítatlan sorokat írja ki (a sikeresen egyesítetteket elrejti). |

##Forrás
-[https://labex.io/tutorials/linux-linux-join-command-file-joining-219193]

# A Linux `join` Parancs Hivatalos Kézikönyve (Man Page)

A `join` parancs két fájl sorait egyesíti egy közös kapcsolódási mező (join field) alapján. Ahol a megadott mezők megegyeznek, oda a parancs egyetlen, egyesített sort ír a szabványos kimenetre (stdout).

## 1. Alapszintaxis és Alapértelmezett Működés

**Szintaxis:**

```bash
join [OPTION]... FILE1 FILE2
```

Ha a `FILE1` vagy `FILE2` helyén `-` szerepel (de nem mindkettőnél), a parancs a szabványos bemenetről (stdin) olvas.

**Alapértelmezett működés:** A kapcsolódási mező az első mező. A mezőket alapértelmezésben üres karakterek (szóközök/tabulátorok) választják el. (Ha nincs megadva egyedi elválasztó, a sor eleji szóközöket a parancs figyelmen kívül hagyja).

## 2. A Legfontosabb Szabály: A Rendezettség (Sorting)

**Kritikus fontosságú:** A `FILE1` és `FILE2` fájloknak előzetesen **rendezettnek (sorted)** kell lenniük a kapcsolódási mező alapján! (Például a `sort` paranccsal). A `join` tiszteletben tartja az `LC_COLLATE` rendezési szabályokat. Ha a bemenet nem rendezett, és emiatt egyes sorokat nem lehet egyesíteni, a parancs figyelmeztető üzenetet (warning) fog dobni.

## 3. Parancssori Kapcsolók (Options)

| Kapcsoló | Leírás |
| --- | --- |
| `-a FILENUM` | A párosítatlan (unpairable) sorokat is kiírja a megadott fájlból (a `FILENUM` értéke 1 vagy 2 lehet, az első vagy második fájlra hivatkozva). Hasonló a "Left/Right Outer Join"-hoz. |
| `-v FILENUM` | Olyan, mint az `-a`, de elrejti a sikeresen egyesített sorokat. Kizárólag a párosítatlan sorokat jeleníti meg a megadott fájlból. |
| `-1 FIELD` | Az 1. fájl megadott számú mezőjét (`FIELD`) használja kapcsolódási pontként. |
| `-2 FIELD` | A 2. fájl megadott számú mezőjét (`FIELD`) használja kapcsolódási pontként. |
| `-j FIELD` | Közös kapcsolódási mező megadása. Teljesen egyenértékű a `-1 FIELD -2 FIELD` megadásával. |
| `-t CHAR` | Egyedi bemeneti és kimeneti mezőelválasztó karakter megadása (pl. `-t ','` CSV esetén). |
| `-e STRING` | A hiányzó (üres) bemeneti mezőket a megadott `STRING` karakterlánccal helyettesíti (általában a `-o` opcióval együtt használatos). |
| `-i`, `--ignore-case` | Kis- és nagybetűk különbségének figyelmen kívül hagyása a mezők összehasonlításakor. |
| `-o FORMAT` | A kimeneti sor felépítésének egyedi formázása a `FORMAT` alapján (lásd alább). |
| `--header` | A fájlok első sorát fejlécként kezeli. Kiírja őket anélkül, hogy megpróbálná párosítani vagy rendezettségüket ellenőrizni. |
| `--check-order` | Kényszeríti a bemeneti fájlok rendezettségének ellenőrzését, még akkor is, ha amúgy minden sor tökéletesen párosítható lenne. |
| `--nocheck-order` | Kikapcsolja a bemenet rendezettségének ellenőrzését. |
| `-z`, `--zero-terminated` | A sorokat NUL karakterrel zárja az újsor (newline) helyett. |

## 4. A Kimenet Formázása (`-o FORMAT`)

A `-o` kapcsoló után egy vesszővel vagy szóközzel elválasztott listát kell megadni, amely meghatározza, hogy mi és milyen sorrendben jelenjen meg a kimeneten.

**Formátum jelölések:**

- `FILENUM.FIELD`: Megadja a fájl sorszámát (1 vagy 2) és a mező sorszámát. (pl. `1.3` jelentése: az 1. fájl 3. mezője).
- `0`: Egy speciális jelölés, amely magát a közös kapcsolódási mezőt (join field) jelenti.
- `auto`: Ha a `FORMAT` értéke az `auto` kulcsszó, akkor a parancs a fájlok első sora alapján automatikusan határozza meg a kimeneti mezők számát.

**Alapértelmezett formátum (ha nincs `-o` megadva):**

A parancs először kiírja a közös kapcsolódási mezőt, majd a `FILE1` fennmaradó mezőit, végül a `FILE2` fennmaradó mezőit, az elválasztó karakterrel elválasztva. A mezők sorszámozása minden esetben 1-től indul.
-[https://linuxconfig.org/join]
-[https://www.man7.org/linux/man-pages/man1/join.1.html]
