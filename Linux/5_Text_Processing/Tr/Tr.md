# A Linux `tr` Parancs Használata és Példák

A `tr` (translate) parancs egy hatékony szövegfeldolgozó eszköz, amellyel karaktereket lehet lefordítani (cserélni), törölni, sűríteni (squeeze), illetve karakterhalmazok komplementerét (inverzét) venni.

## 1. A `tr` Parancs Szintaxisa és Kapcsolói

A parancs alapvető szintaxisa a következő:

```bash
tr [OPTION] SET1 [SET2]
```

- **SET1:** A lecserélendő vagy törlendő karakterek halmaza.
- **SET2:** Az a karakterhalmaz, amire a SET1 elemeit cserélni akarjuk.

**Kapcsolók (Options):**

- `-c` vagy `--complement`: A SET1 komplementerének (kiegészítőjének) használata (minden karakter, ami nincs benne a SET1-ben).
- `-d` vagy `--delete`: A SET1-ben megadott karakterek törlése a bemenetből (ilyenkor nem kell SET2).
- `-s` vagy `--squeeze-repeats`: Az ismétlődő karakterek sűrítése. (Ha a SET1-ben megadott karakter többször szerepel egymás után, azt egyetlen karakterré vonja össze).
- `-t` vagy `--truncate-set1`: A SET1 csonkolása a SET2 hosszára.

## 2. Gyakorlati Példák

### 2.1. Karakterek cseréje (Kisbetűből Nagybetű és fordítva)

A `tr` egyik leggyakoribb felhasználása a kis- és nagybetűk konvertálása.

**Kisbetűk cseréje nagybetűkre:**

```bash
echo "Hello, World!" | tr 'a-z' 'A-Z'
```

**Kimenet:**

```plaintext
HELLO, WORLD!
```

**Nagybetűk cseréje kisbetűkre:**

```bash
echo "Hello, World!" | tr 'A-Z' 'a-z'
```

**Kimenet:**

```plaintext
hello, world!
```

### 2.2. Karakterek Törlése (`-d`)

Tegyük fel, hogy el akarunk távolítani bizonyos nemkívánatos karaktereket a szövegből. Ebben a példában a vesszőt (`,`) és a felkiáltójelet (`!`) töröljük a bemenetből:

```bash
echo "Hello, World!" | tr -d ',!'
```

> **Megjegyzés:** Az eredeti angol leírás a `tr -d ',' '!'` formát hozta, de a szabványos és hibátlan szintaxis több karakter törlésére egy sztringben történik: `-d ',!'`.

**Kimenet:**

```plaintext
Hello World
```

### 2.3. Ismétlődő Karakterek Sűrítése (`-s`)

A sűrítés (squeezing) azt jelenti, hogy az egymás után többször ismétlődő karaktereket egyetlen előfordulásra cseréljük. Például, ha egy fájlban (vagy szövegben) több egymást követő szóköz van, azokat egyetlen szóközzé alakíthatjuk:

```bash
echo "Hello   World!" | tr -s ' '
```

**Kimenet:**

```plaintext
Hello World!
```

### 2.4. Karakterek Komplementerének vétele (Inverz kiválasztás, `-c`)

A komplementer jelentése, hogy a megadott karakterhalmazon kívül eső összes többi karakterrel dolgozunk. Például, ha csak a nagybetűket szeretnénk kinyerni (és minden mást - kisbetűket, szóközöket, írásjeleket - eltávolítani), a `-c` (komplementer) kapcsolót használhatjuk:

```bash
echo "Hello, World!" | tr -cd 'A-Z'
```

> **Megjegyzés:** Itt a `-c` és `-d` kapcsolókat kombináljuk: töröljön minden karaktert, ami NEM a nagy `A-Z` tartományba esik.

**Kimenet:**

```plaintext
HW
```

##Forrás
-[https://labex.io/tutorials/linux-linux-tr-command-with-practical-examples-422963]

# A Linux `tr` Parancs Használata és Példák (Kiegészítés)

A `tr` (translate) parancs karakterek egyik halmazának egy másikra történő egyszerű, de rendkívül hasznos lefordítására (cseréjére) szolgál. Egyszerű és gyors fordításokra tökéletes, de összetettebb szövegmanipulációra (pl. teljes szavak cseréjére) az `awk` vagy `sed` parancsok alkalmasabbak.

## 1. Alapszintaxis és Kapcsolók (Options)

**Szintaxis:**

```bash
tr [options] charset1 [charset2]
```

**Hasznos kapcsolók:**

- `-d`: Törli a megadott (első halmazban lévő) karaktereket.
- `-c`: Az első halmaz komplementerét (kiegészítőjét) veszi (azokra a karakterekre fog hatni, amelyek NINCSENEK az első halmazban).
- `-s`: Eltávolítja az első halmaz karaktereinek többszörös, egymás melletti ismétlődéseit (sűrítés / squeeze).
- `-t`: Csonkolja (truncate) az első halmazt a második halmaz hosszára.

## 2. Gyakorlati Példák

A példákhoz az alábbi `sample.txt` fájlt használjuk:

```plaintext
This is my life
and my file
```

### 1. Kisbetű – Nagybetű konverzió (és fordítva)

**Szöveg átalakítása csupa nagybetűssé:**

```bash
cat sample.txt | tr 'a-z' 'A-Z'
```

Vagy karakterosztályok (character classes) használatával:

```bash
cat sample.txt | tr [:lower:] [:upper:]
```

**Kimenet:**

```plaintext
THIS IS MY LIFE
AND MY FILE
```

A csupa kisbetűs alakhoz csak fel kell cserélni a halmazokat (`tr 'A-Z' 'a-z'`).

### 2. Egyik karakterhalmaz cseréje egy másikra

A cserélés karakterenként (egyesével) történik. Az első halmaz első karaktere kicserélődik a második halmaz első karakterére, és így tovább.

```bash
cat sample.txt | tr 'ilm' 'tyz'
```

(Az `i` lecserélődik `t`-re, az `l` `y`-ra, az `m` pedig `z`-re).

**Kimenet:**

```plaintext
Thts ts zy ytfe
and zy ftye
```

**Szabály eltérő hosszúságú halmazokra:** Ha a második halmaz rövidebb, mint az első, akkor a második halmaz utolsó karaktere ismétlődik.

Például: `tr 'ilm' 'ty'` esetén az `l` és az `m` is `y`-ra lesz cserélve.

(Gyakorlati haszon: `()` cseréje `{}`-re, vagy alulvonás `_` cseréje kötőjelre `-`, esetleg tabulátorok szóközre cserélése).

### 3. Specifikus karakter(ek) törlése (`-d`)

Töröljük ki az `i` és `s` karaktereket:

```bash
cat sample.txt | tr -d 'is'
```

**Kimenet:**

```plaintext
Th my lfe
and my fle
```

> **Figyelem:** A parancs minden egyes `i` és `s` betűt törölt. Ha kifejezetten az "is" szót akartad volna törölni, a `tr` nem jó megoldás, arra a `sed` parancsot kell használni.

### 4. Ismétlődő karakterek (pl. többszörös szóközök) eltávolítása (`-s`)

A `-s` (squeeze) opció a többszörösen ismétlődő karaktereket egyetlen előfordulásra csökkenti.

**Több szóköz cseréje egyetlen szóközre:**

```bash
echo "Text has too many spaces" | tr -s " "
```

**Kimenet:**

```plaintext
Text has too many spaces
```

**Az ismétlődések azonnal le is cserélhetők egy másik karakterre (pl. pontosvesszőre):**

```bash
echo "Text has too many spaces" | tr -s " " ";"
```

**Kimenet:**

```plaintext
Text;has;too;many;spaces
```

### 5. Minden NEM számjegy (non-digit) karakter törlése (`-c` és `-d` kombinálása)

Tegyük fel, hogy egy szöveg betűket és számokat is tartalmaz, és mi csak a számokat akarjuk megtartani. Itt a komplementer (`-c`) és a törlés (`-d`) opciókat kombináljuk:

```bash
echo "Phone number is 123456789" | tr -cd [:digit:]
```

**Kimenet:**

```plaintext
123456789
```

### 6. Keresési minta csonkolása (`-t`)

Ha az első karakterhalmaz hosszabb, mint a második, akkor (ahogy a 2. pontban láttuk) a második halmaz utolsó karaktere kitölti a maradékot.

Ha bekapcsoljuk a `-t` (truncate) opciót, a rendszer levágja (csonkolja) az első halmazt a második hosszára, és a felesleget érintetlenül hagyja.

**Csonkolás NÉLKÜL:**

```bash
cat sample.txt | tr 'isef' '12'
```

(Az `i`->1, és a maradék `s`, `e`, `f` mind 2-es lesz).

**Kimenet:**

```plaintext
Th12 12 my l122
and my 21l2
```

**Csonkolással (`-t`):**

```bash
cat sample.txt | tr -t 'isef' '12'
```

(A halmaz csonkolódik az első két karakterre (`is`), így csak az `i` lesz 1, az `s` pedig 2. Az `e` és az `f` érintetlen marad).

**Kimenet:**

```plaintext
Th12 12 my l1fe
and my f1le
```

##Forrás:
-[https://linuxhandbook.com/tr-command/]
