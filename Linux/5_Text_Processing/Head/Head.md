# A Linux `head` Parancs Használata (Fájlok elejének megjelenítése)

A `head` parancs egy hatékony eszköz, amellyel a szöveges fájlok legelejét tekinthetjük meg anélkül, hogy a teljes fájlt meg kellene nyitnunk. Olyan, mint egy nagyító a digitális nyomozáshoz: gyors betekintést enged a fájlok (pl. naplófájlok / logok) elejébe, ami hatalmas fájlok esetén is memóriakímélő és villámgyors megoldás.

## 1. Az alapok megértése

Alapértelmezés szerint a `head` parancs a megadott fájl első 10 sorát jeleníti meg.

**Parancs:**

```bash
head access.log
```

**Példa kimenet:**

```plaintext
192.168.1.100 - - [01/Jan/2024:00:00:01 +0000] "GET /index.html HTTP/1.1" 200 1234
192.168.1.101 - - [01/Jan/2024:00:00:02 +0000] "GET /style.css HTTP/1.1" 200 567
192.168.1.102 - - [01/Jan/2024:00:00:03 +0000] "GET /logo.png HTTP/1.1" 200 2345
192.168.1.103 - - [01/Jan/2024:00:00:04 +0000] "POST /login HTTP/1.1" 302 -
192.168.1.104 - - [01/Jan/2024:00:00:05 +0000] "GET /dashboard HTTP/1.1" 200 3456
192.168.1.105 - - [01/Jan/2024:00:00:06 +0000] "GET /api/user HTTP/1.1" 200 789
192.168.1.106 - - [01/Jan/2024:00:00:07 +0000] "GET /images/banner.jpg HTTP/1.1" 200 4567
192.168.1.107 - - [01/Jan/2024:00:00:08 +0000] "POST /comment HTTP/1.1" 201 -
192.168.1.108 - - [01/Jan/2024:00:00:09 +0000] "GET /search?q=linux HTTP/1.1" 200 2345
192.168.1.109 - - [01/Jan/2024:00:00:10 +0000] "GET /about HTTP/1.1" 200 1234
```

## 2. A sorok számának testreszabása (`-n`)

Az `-n` opció segítségével megadhatod, hogy pontosan hány sort szeretnél látni a fájl elejéről (az alapértelmezett 10 helyett).

**Parancs (csak az első 5 sor):**

```bash
head -n 5 access.log
```

**Kimenet:**

```plaintext
192.168.1.100 - - [01/Jan/2024:00:00:01 +0000] "GET /index.html HTTP/1.1" 200 1234
192.168.1.101 - - [01/Jan/2024:00:00:02 +0000] "GET /style.css HTTP/1.1" 200 567
192.168.1.102 - - [01/Jan/2024:00:00:03 +0000] "GET /logo.png HTTP/1.1" 200 2345
192.168.1.103 - - [01/Jan/2024:00:00:04 +0000] "POST /login HTTP/1.1" 302 -
192.168.1.104 - - [01/Jan/2024:00:00:05 +0000] "GET /dashboard HTTP/1.1" 200 3456
```

## 3. Több fájl vizsgálata egyszerre

A `head` parancsnak több fájlnevet is megadhatsz. Ilyenkor a kimenetet jól látható fejlécekkel (`==> fájlnév <==`) választja el egymástól, így könnyen összehasonlíthatod a fájlok kezdetét.

**Parancs:**

```bash
head access.log error.log
```

**Kimenet:**

```plaintext
==> access.log <==
192.168.1.120 - - [01/Jan/2024:00:00:53 +0000] "POST /about HTTP/1.1" 200 7616
192.168.1.147 - - [01/Jan/2024:00:00:45 +0000] "GET /dashboard HTTP/1.1" 200 7348
... (további sorok)

==> error.log <==
[01/Jan/2024:00:01:23 +0000] [error] [client 192.168.1.150] File does not exist: /var/www/html/missing.html
[01/Jan/2024:00:02:34 +0000] [error] [client 192.168.1.151] PHP Parse error:  syntax error, unexpected ';' in /var/www/html/index.php on line 30
... (további sorok)
```

## 4. A `head` használata csővezetékekkel (Pipes)

A `head` parancs nagyon jól kombinálható más eszközökkel. Ha például csak az `access.log` HTTP kéréseit szeretnéd látni, használhatod a `cut` parancsot a sorok darabolására, a `head`-et pedig az eredmény limitálására.

**Parancs (az idézőjelek közötti 2. mező első 3 találata):**

```bash
cut -d '"' -f2 access.log | head -n 3
```

**Kimenet:**

```plaintext
POST /about HTTP/1.1
GET /dashboard HTTP/1.1
DELETE /comment HTTP/1.1
```

(Magyarázat: A `cut` idézőjelek (`"`) mentén bontja a sort, és a második részt (`-f2`) adja át a csővezetéken (`|`) a `head -n 3` parancsnak, amely csak az első 3 sort írja ki).

## 5. Keresés és a `head` kombinálása (`grep`)

Ha egy logfájlban keresel egy adott mintát, de rengeteg a találat, és te csak az első néhány előfordulásra vagy kíváncsi (például az `/admin` oldal első néhány hívására), kombinálhatod a `grep` parancsot a `head`-del.

**Parancs:**

```bash
grep "/admin" access.log | head -n 3
```

**Kimenet:**

```plaintext
192.168.1.148 - - [01/Jan/2024:00:00:33 +0000] "POST /admin/dashboard HTTP/1.1" 201 5546
192.168.1.115 - - [01/Jan/2024:00:00:22 +0000] "PUT /admin HTTP/1.1" 302 1113
192.168.1.163 - - [01/Jan/2024:00:00:56 +0000] "POST /admin/dashboard HTTP/1.1" 301 815
```

## 6. Nagy fájlok felfedezése

A `head` parancs hatalmas logfájlok esetén is azonnal fut, mivel csak a fájl elejét olvassa be, és nem tölti be az egészet a memóriába. Ez gyors betekintést nyújt a fájl szerkezetébe.

```bash
# Fájlméret ellenőrzése
ls -lh access.log

# Az első 15 sor gyors kiolvasása
head -n 15 access.log
```

## 7. További hasznos paraméterek (Extra)

- `-c` (bytes): A sorok száma helyett a fájl első néhány bájtját jeleníti meg (pl. `head -c 20 fájl` az első 20 bájtot írja ki).
- `-q` (quiet): Több fájl vizsgálatakor elrejti (nem írja ki) a fájlneveket tartalmazó fejléceket (`==> fájlnév <==`).
- `-v` (verbose): Mindig megjeleníti a fájlnevet tartalmazó fejlécet, még akkor is, ha csak egyetlen fájlt vizsgálsz.
##Forrás

-[https://labex.io/tutorials/linux-linux-head-command-file-beginning-display-214302]

# A Linux `head` Parancs Használata és Példák (Kiegészítés)

Míg a `cat` parancs a teljes fájl tartalmát kiírja a terminálra, a `head` egy kiváló alternatíva arra, ha csak egy szövegfájl elejét – egy megadott számú sort – szeretnénk megtekinteni.

## 1. Alapszintaxis és Alapértelmezett Működés

**Szintaxis:**

```bash
head [option] [filename]
```

Ha nem adsz meg semmilyen opciót, a `head` alapértelmezés szerint az első 10 sort írja ki a fájlból. (Természetesen, ha a fájl 10 sornál rövidebb, akkor a teljes tartalmat megjeleníti).

```bash
head agatha.txt 
```

**Kimenet:**

```plaintext
The Mysterious Affair at Styles
The Secret Adversary
The Murder on the Links
The Man in the Brown Suit
The Secret of Chimneys
The Murder of Roger Ackroyd
The Big Four
The Mystery of the Blue Train
The Seven Dials Mystery
The Murder at the Vicarage
```

## 2. Gyakorlati Példák és Opciók

### 1. Csak az első N darab sor kiíratása (`-n`)

Ha egy specifikus számú sort akarsz látni a fájl elejéről, használd az `-n` opciót. Például az első 3 sor megjelenítése:

```bash
head -n 3 agatha.txt 
```

**Kimenet:**

```plaintext
The Mysterious Affair at Styles
The Secret Adversary
The Murder on the Links
```

### 2. Az összes sor kiíratása, KIVÉVE az utolsó N sort (Negatív érték)

Kizárhatsz egy adott számú sort a fájl végéről úgy, hogy az `-n` opciónak negatív számot adsz meg. A parancs kinyomtatja a fájl összes sorát, kivéve a legutolsó tizenötöt:

```bash
head -n -15 agatha.txt 
```

**Kimenet:**

```plaintext
The Mysterious Affair at Styles
The Secret Adversary
The Murder on the Links
The Man in the Brown Suit
The Secret of Chimneys
```

### 3. Több fájl egyidejű olvasása

A `head` parancsnak több fájlt is átadhatsz egyszerre:

```bash
head -n 2 agatha.txt sherlock.txt 
```

**Kimenet:** (Látható, hogy a kimeneteket a fájlneveket tartalmazó fejlécek választják el):

```plaintext
==> agatha.txt <==
The Mysterious Affair at Styles
The Secret Adversary

==> sherlock.txt <==
A Scandal in Bohemia
The Red-Headed League
```

### 4. Fejlécek (fájlnevek) kezelése (`-q` és `-v`)

- **`-q` (quiet mode - csendes mód):** Eltünteti a fájlneveket tartalmazó fejléceket, ha több fájlt vizsgálsz.

  ```bash
  head -q -n 2 agatha.txt sherlock.txt 
  ```

  **Kimenet** (fejlécek nélkül fűzi egymás alá a sorokat):

  ```plaintext
  The Mysterious Affair at Styles
  The Secret Adversary
  A Scandal in Bohemia
  The Red-Headed League
  ```

- **`-v` (verbose mode - bőbeszédű mód):** Kényszeríti a fejléc kiírását még akkor is, ha csak egyetlen fájlt vizsgálsz.

  ```bash
  head -v -n 2 agatha.txt 
  ```

  **Kimenet:**

  ```plaintext
  ==> agatha.txt <==
  The Mysterious Affair at Styles
  The Secret Adversary
  ```

### 5. Specifikus számú bájt / karakter kiíratása (`-c`)

Ha nem sorokat, hanem bájtokat akarsz kiírni, használd a `-c` opciót. (Mivel egy standard ASCII karakter pontosan egy bájt, ez felfogható karakterszámlálásként is).

**Az első 3 karakter (bájt) kiírása:**

```bash
head -c3 agatha.txt 
```

**Kimenet:**

```plaintext
The
```

(Ide is adhatsz meg negatív értéket: a `head -c -50 agatha.txt` kiírja az egész fájlt, kivéve az utolsó 50 bájtot).

## Bónusz Tipp: A `head` és `tail` kombinálása (Sorok kivágása a fájl közepéből)

Mi van akkor, ha a fájl közepéről (pl. a 10. és a 15. sorok közötti részt) szeretnéd kinyerni? A megoldás a `head` és a `tail` parancsok összekapcsolása egy pipe (`|`) segítségével.

**Példa: A 10. sortól a 15. sorig tartó rész kiírása:**

```bash
head -n 15 agatha.txt | tail -n +10
```

(Magyarázat: A `head` kivágja a fájl legelső 15 sorát, majd a `tail -n +10` ebből a 15 sorból kiírja az összeset a 10. sortól lefelé).

**Példa: Kizárólag a 15. sor (egy konkrét sor) kiírása:**

```bash
head -n 15 agatha.txt | tail -n 1
```

(Magyarázat: A `head` veszi az első 15 sort, a `tail` pedig ebből a csomagból csak a legutolsót – ami értelemszerűen az eredeti fájl 15. sora lesz).

##Forrás
-[https://linuxhandbook.com/head-command/]
