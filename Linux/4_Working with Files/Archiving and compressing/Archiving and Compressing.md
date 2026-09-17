# Fájlok Csomagolása és Tömörítése (Packaging and Compression)

A csomagolás (packaging/archiving) és a tömörítés (compression) két különböző probléma megoldására szolgál:

- **Archívum (Csomag):** Egy teljes könyvtárfát (mappastruktúrát) fűz össze egyetlen adatfolyammá, megőrizve a neveket és a struktúrát.

- **Tömörítés:** Ugyanazt az adatot kisebb (kompaktabb) méretben reprezentálja.

**Kiterjesztések jelentése:**

- `.tar`: Egy archívum (tömörítés nélkül).
- `.gz`: Gzip-pel tömörített adat (önmagában csak egy fájl).
- `.tar.gz`: Egy gzip-pel tömörített tar archívum (könyvtárfa + tömörítés).

A biztonságos munkafolyamat a fájlok felülírásának elkerülése érdekében mindig ez: **ellenőrzés (inspect) -> kicsomagolás egy új mappába (extract) -> ellenőrzés (verify)**.

## 1. Könyvtárfa létrehozása (Példa környezet)

Hozzuk létre a forráskönyvtárakat és a bennük lévő minta fájlokat a teszteléshez:

```bash
cd /home/labex/project/archive-lab
mkdir -p source/{config,notes,data}

printf 'mode=training\nport=8080\n' > source/config/app.conf
printf 'Archive the complete project tree.\n' > source/notes/README.txt
printf 'alpha,beta,gamma\n1,2,3\n' > source/data/sample.csv
```

Ellenőrizzük a fájlokat relatív útvonalakkal:

```bash
find source -type f -printf '%P\n' | sort
```

## 2. Tar Archívum létrehozása és ellenőrzése (Tömörítés nélkül)

A `tar` parancs főbb kapcsolói:

- `c` (create): létrehozás
- `t` (list): tartalom listázása
- `x` (extract): kicsomagolás
- `f` (file): archívum fájlneve
- `-C`: Munkakönyvtár megváltoztatása (hogy elkerüljük a hosszú, abszolút útvonalak elmentését).

**Létrehozás:** (Az eredeti fájlok megmaradnak, a tar csak olvassa őket)

```bash
cd /home/labex/project/archive-lab
tar -cf project.tar -C /home/labex/project/archive-lab source
```

**Fájlok és méret ellenőrzése:**

```bash
ls -lh project.tar
ls -l source/config/app.conf
```

**A csomag tartalmának listázása (Kicsomagolás előtt):**

Minden tagnak (member) relatív útvonallal (pl. `source/`) kell kezdődnie a `/` helyett. Ez teszi biztonságossá és hordozhatóvá a visszaállítást.

```bash
tar -tf project.tar > tar-members.txt
cat tar-members.txt
```

## 3. Biztonságos Kicsomagolás és Eredmény összehasonlítása

Soha ne csomagolj ki közvetlenül a forrásfájlokra, inkább használj egy üres célkönyvtárat (a `-C` kapcsolóval)!

```bash
cd /home/labex/project/archive-lab
mkdir tar-restore
tar -xf project.tar -C tar-restore
```

**A visszaállított útvonalak ellenőrzése:**

```bash
find tar-restore -type f -printf '%P\n' | sort
```

**Összehasonlítás a `diff -r` paranccsal** (ha nincs eltérés, a parancs nem ír ki semmit):

```bash
diff -r source tar-restore/source
echo "tar restore matches source" > tar-restore-ok.txt
cat tar-restore-ok.txt
```

## 4. Egyetlen fájl tömörítése és kibontása Gzip-pel

A `gzip` abban különbözik az archívumoktól, hogy önmagában csak egyetlen fájlt tud tömöríteni (nem csomagol könyvtárakat).

Készítsünk egy másolatot (hogy az eredeti megmaradjon), majd tömörítsük:

```bash
cd /home/labex/project/archive-lab
cp source/data/sample.csv sample-copy.csv
gzip sample-copy.csv
```

(Alapértelmezésben a `gzip` lecseréli a bemeneti fájlt a `.gz` végződésű tömörített verzióra).

**Az eredmény ellenőrzése:**

```bash
ls -lh sample-copy.csv.gz
file sample-copy.csv.gz
```

**Kibontás és összehasonlítás:**

```bash
gunzip sample-copy.csv.gz
cmp source/data/sample.csv sample-copy.csv
echo "gzip round trip matches" > gzip-round-trip-ok.txt
cat gzip-round-trip-ok.txt
```

## 5. Tömörített Tar Archívum (.tar.gz) létrehozása és visszaállítása

Itt ötvözzük a csomagolást és a tömörítést. A `z` kapcsoló mondja meg a tar-nak, hogy fűzze át az adatokat a gzip-en.

**Létrehozás és listázás (`-czf` és `-tzf`):**

```bash
cd /home/labex/project/archive-lab
tar -czf project.tar.gz -C /home/labex/project/archive-lab source
tar -tzf project.tar.gz > targz-members.txt
cat targz-members.txt
```

**Biztonságos kicsomagolás külön mappába és összehasonlítás (`-xzf`):**

```bash
mkdir targz-restore
tar -xzf project.tar.gz -C targz-restore
diff -r source targz-restore/source
echo "tar.gz restore matches source" > targz-restore-ok.txt
cat targz-restore-ok.txt
```

## 6. ZIP Archívum kezelése (Keresztplatformos csere)

A ZIP egy nagyon gyakori formátum Linux, macOS és Windows rendszereken is.

A `zip -r` (recursive) a teljes könyvtárfát hozzáadja a csomaghoz.

**Létrehozás:**

```bash
cd /home/labex/project/archive-lab
zip -r project.zip source
```

**A csomag tartalmának listázása (kicsomagolás nélkül):**

```bash
unzip -l project.zip
```

**Kicsomagolás külön mappába (a `-d` kapcsolóval) és összehasonlítás:**

A `-q` (quiet) csendes módban végzi a kicsomagolást.

```bash
unzip -q project.zip -d zip-restore
diff -r source zip-restore/source
echo "zip restore matches source" > zip-restore-ok.txt
cat zip-restore-ok.txt
```
