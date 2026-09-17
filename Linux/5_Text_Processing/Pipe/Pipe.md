# Átfogó Útmutató a Linux Csővezetékekhez (Pipes)

A csővezetékek (pipes) a Linux parancssor egyik legfontosabb építőkövei. Lehetővé teszik, hogy összekapcsolj parancsokat: az egyik parancs kimenetét közvetlenül a másik bemeneteként használd. Ezzel megspórolható a köztes (ideiglenes) fájlok létrehozása a lemezen.

## 1. Mik azok a Pipe-ok és hogyan működnek?

A csővezetéket a függőleges vonal (`|`) karakter jelöli.

```bash
command1 | command2
```

Ebben az esetben a `command1` lefut, és a szabványos kimenete (stdout) közben közvetlenül a `command2` szabványos bemenetébe (stdin) áramlik.

**Működés a háttérben:** A POSIX szabvány szerint a pipe-ok akár 4 KiB adatot is pufferelnek, mielőtt blokkolnák az írást, de a legtöbb Linux rendszeren az adatok szinte azonnal áramlanak a parancsok között.

## 2. Szövegszűrés a `grep` paranccsal

A `grep` az egyik leggyakrabban használt eszköz a pipe-okkal. Adott mintákat (szavakat) keres a bemenetben, és csak azokat a sorokat írja ki, amelyek illeszkednek.

- **Egyszerű szűrés:** (Csak a "python" szót tartalmazó fájlokat listázza)

  ```bash
  ls -l /usr/bin | grep python
  ```

- **Kiterjesztett Regex (`-E`):** Több minta keresése egyszerre (fail, error, vagy warn):

  ```bash
  dmesg | grep -E '(fail|error|warn)'
  ```

- **Inverz találat (`-v`):** Azok a sorok, amelyek NEM tartalmazzák a "usb" szót:

  ```bash
  dmesg | grep -v usb
  ```

- **Egyéb opciók:**
  - `-i`: Kis- és nagybetűk figyelmen kívül hagyása (case insensitive).
  - `-A #` és `-B #`: A találat utáni (After) vagy előtti (Before) # számú sor megjelenítése kontextusként.
  - `-c`: Csak a találatok számát írja ki.

## 3. Szöveg rendezése (`sort`) és Duplikációk szűrése (`uniq`)

- **Rendezés méret szerint csökkenő sorrendben:**

  ```bash
  du -sh * | sort -h -r
  ```

  (`-h`: olvasható formátum pl. MB/GB, `-r`: fordított sorrend).

- **Rendezés egy adott oszlop (mező) alapján:**

  ```bash
  cat data.csv | sort -t, -k1
  ```

  (`-t,`: a vessző az elválasztó, `-k1`: az első oszlop alapján rendez).

- **Duplikált sorok eltávolítása:** (A `uniq` csak az egymás melletti duplikációkat távolítja el, ezért előtte mindig rendezni kell!)

  ```bash
  cat names.txt | sort | uniq
  ```

- **Előfordulások megszámolása:** (Minden sor elé odaírja, hányszor szerepelt).

  ```bash
  cat names.txt | sort | uniq -c
  ```

## 4. Átirányítás fájlokba (`>`, `>>` és `tee`)

A csővezeték végén az eredményt lemezre menthetjük.

- **Felülírás (`>`):**

  ```bash
  cat files.txt | grep -i python > python_files.txt
  ```

- **Hozzáfűzés (`>>`):**

  ```bash
  dmesg | grep hardware >> master_log.txt
  ```

- **Megjelenítés a képernyőn ÉS mentés fájlba egyszerre (`tee`):**

  ```bash
  uptime | tee system_uptime.txt
  ```

## 5. Hosszú kimenetek olvasása (`less`, `head`, `tail`)

- **`less`:** Interaktív lapozót nyit meg a kimenethez (navigálás nyilakkal/szóközzel, kilépés: `q`).

  ```bash
  dmesg | less
  ```

- **`tail` / `head`:** Csak az utolsó / első N sort írja ki.

  Kombinálva a `less`-szel, hogy csak az utolsó 30 sort böngésszük:

  ```bash
  dmesg | tail -30 | less
  ```

## 6. Számlálás a `wc` (word count) paranccsal

- **Fájlok száma, amelyekben van találat:** (A `grep -l` csak a fájlneveket adja vissza).

  ```bash
  grep -l python *.txt | wc -l
  ```

- **Összes találatot tartalmazó sor megszámolása az összes fájlban:**

  ```bash
  grep python *.txt | wc -l
  ```

## 7. Oszlopok kinyerése (`cut` és `awk`)

- **A `cut` használata** (felhasználónevek kinyerése):

  ```bash
  cut -d: -f1 /etc/passwd
  ```

  (`-d:`: a kettőspont az elválasztó, `-f1`: első oszlop).

- **Az `awk` használata:**

  ```bash
  awk -F: '{print $1}' /etc/passwd
  ```

  (`-F:`: elválasztó, `$1`: az első oszlop kiírása).

- **Matematikai műveletek `awk`-kal:** (Összeadja a 2. oszlop értékeit és kiírja az összeget).

  ```bash
  df -h | awk '{total+=$2} END {print total}'
  ```

## 8. Komplex folyamatok összefűzése (Chaining)

Egyszerre akár 5-6 parancs is összekapcsolható egy hatalmas adatfeldolgozó gépezetté.

**Példa:** Kitömörít egy logfájlt -> kiszűri a "favicon" kéréseket -> kinyeri a 7. oszlopot -> sorba rendezi -> megszámolja a duplikációkat -> és csökkenő sorrendbe rendezi az előfordulások száma alapján:

```bash
zcat access.log.gz | grep -v "GET /favicon" | awk '{print $7}' | sort | uniq -c | sort -nr
```

> **Tipp:** A hosszú parancsokat a `\` karakterrel érdemes több sorba tördelni az olvashatóság érdekében.

## 9. Névleges Csővezetékek (Named Pipes - `mkfifo`)

A hagyományos (névtelen) pipe-ok egyirányúak és ideiglenesek. A `mkfifo` paranccsal létrehozhatunk egy fájlként viselkedő kétirányú csővezetéket folyamatok (scriptek) közötti kommunikációra.

**Létrehozás:**

```bash
mkfifo mypipe
```

**Olvasás az egyik folyamatban:**

```bash
cat < mypipe
```

**Írás egy másik folyamatból:**

```bash
echo "Hello" > mypipe
```

## 10. Legjobb gyakorlatok és hibaelkerülés

- Használj szóközöket a `|` karakter körül az olvashatóságért.
- Minden pipe szegmens (stage) külön alsó-héjban (subshell) fut párhuzamosan – a változók/állapotok nem osztódnak meg közöttük!
- Ne használj pipe-ot olyan parancsokhoz, amik interaktív terminál bemenetet várnak (pl. `sudo` vagy `su`).
- Ha egy parancs a pipe közepén hibával leáll, a csővezeték megszakad, a későbbi parancsok nem dolgozzák fel az adatot.
- Használj idézőjeleket a speciális karaktereknél (`*`, `$`), hogy a shell ne bontsa ki őket idő előtt.
- Hatalmas adatmennyiség vagy komplex adatelemzés esetén a pipe-ok helyett néha jobb ideiglenes (temp) fájlokat használni a köztes lépésekhez. Ez megbízhatóbb, és hiba esetén nem kell elölről kezdeni a teljes (pl. több órás `grep`) műveletet.

- ##Forrás
- -[https://thelinuxcode.com/linux-pipe-command-examples/]

# Csővezetékek (Pipes) és Átirányítás (Redirection) a Linuxban

A Linuxban minden futó parancs automatikusan három kommunikációs adatcsatornával (stream) rendelkezik:

- **STDIN (0)** – Szabványos bemenet (a programba táplált adatok).
- **STDOUT (1)** – Szabványos kimenet (a program által kiírt adatok, alapértelmezésben a terminál).
- **STDERR (2)** – Szabványos hibakimenet (a hibaüzenetek számára, szintén alapértelmezésben a terminál).

A piping (csövezés) és redirection (átirányítás) segítségével ezeket az adatfolyamokat köthetjük össze programok és fájlok között.

## 1. Kimenet átirányítása fájlba (`>` és `>>`)

Ha a kimenetet a képernyő helyett fájlba szeretnénk menteni, a nagyobb mint (`>`) operátort használjuk.

**Fájl létrehozása és mentés:** Ha a fájl nem létezik, a terminál automatikusan létrehozza.

```bash
ls > myoutput
cat myoutput
```

> **Megjegyzés:** A fájl már azelőtt létrejön, hogy a parancs lefutna, így a kimeneti fájl szerepelni fog a saját listájában.

**Meglévő fájl felülírása vs. Hozzáfűzés (`>>`):**

A `>` operátor teljesen törli a meglévő fájl tartalmát, majd beírja az újat.

A `>>` operátor hozzáfűzi (append) az új adatokat a fájl végéhez, megőrizve a korábbi tartalmat.

```bash
# Példa: a sorok számának hozzáfűzése a myoutput fájl végéhez
wc -l barry.txt >> myoutput
```

## 2. Bemenet átirányítása fájlból (`<`)

A kisebb mint (`<`) operátorral egy fájl tartalmát tápláljuk be a programba a STDIN streamen keresztül.

**A különbség az argumentum és az átirányítás között:**

Ha a `wc -l myoutput` parancsot argumentumként megadott fájllal futtatjuk, a kimenet tartalmazza a fájl nevét is. Ha a `<` átirányítással adjuk át (`wc -l < myoutput`), az adat névtelenül érkezik meg, így a fájlnév nem jelenik meg a kimeneten.

```bash
wc -l < myoutput
# Eredmény: csak a szám mutatkozik, a név nélkül
```

## 3. Hibakimenet átirányítása (`2>`) és egyesítés

A hibaüzenetek (STDERR) a 2-es stream azonosítóval rendelkeznek. Ha meg akarjuk őket menteni, a `>` elé kell írnunk a 2-es számot (`2>`).

**Csak a hibák mentése fájlba:**

```bash
ls -l video.mpg blah.foo 2> errors.txt
```

**Normál kimenet és hibák mentése EGYETLEN fájlba:**

A hibákat átirányíthatjuk a normál kimenetbe úgy, hogy az `&` jelet tesszük a stream száma elé:

```bash
ls -l video.mpg blah.foo > myoutput 2>&1
```

## 4. Csővezetékek (Piping - `|`)

A pipe (`|`) operátor az egyik program kimenetét (STDOUT) közvetlenül átadja a másik program bemenetének (STDIN), köztes ideiglenes fájlok használata nélkül.

**Példa (Csak az első 3 fájl listázása):**

```bash
ls | head -3
```

**Több parancs láncolása:**

```bash
ls | head -3 | tail -1
```

(Ez a parancs kilistázza a fájlokat -> veszi az első hármat -> majd abból a háromból kivágja az utolsót).

> 💡 **Aranyszabály a Pipe-ok építéséhez:** Soha ne próbáld meg az egész összetett parancsot egyszerre leírni! Építsd fel lépésről lépésre (incrementally): futtasd az első parancsot, ellenőrizd az eredményt, add hozzá a másodikat, ellenőrizd újra, és így tovább. Ez rengeteg fejfájástól megkímél.

## 5. Gyakorlati Példák (Összetett Workflows)

**Rendezés úgy, hogy a mappák (directories) elöl legyenek:**

```bash
ls -l /etc | tail -n +2 | sort
```

**Írási joggal (group write permission) rendelkező fájlok keresése a saját mappában:**

```bash
ls -l ~ | grep '^.....w'
```

**Adott mappában lévő fájlok tulajdonosainak megszámolása (fájlok száma felhasználónként):**

```bash
ls -l /projects/ghosttrail | tail -n +2 | sed 's/\s\s*/ /g' | cut -d ' ' -f 3 | sort | uniq -c
```

## 6. Gyors Referencia (Összefoglaló)

| Operátor | Művelet |
| --- | --- |
| `>` | Kimenet mentése fájlba (felülírja a meglévőt). |
| `>>` | Kimenet hozzáfűzése egy meglévő fájl végéhez. |
| `<` | Bemenet olvasása fájlból (névtelen adatfolyamként). |
| `2>` | Hibaüzenetek (STDERR) átirányítása. |
| `|` | Az egyik program kimenetének átadása a másik bemeneteként. |

##Forrás
-[https://ryanstutorials.net/linuxtutorial/piping.php#piping]
-[https://linuxsimply.com/what-is-piping-in-linux/]
