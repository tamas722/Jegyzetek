# Fájlkezelés Linuxban: Fájlok és könyvtárak létrehozása és törlése

## 1. Fájlok és könyvtárak létrehozása (Parancssori módszerek)

### Fájlok létrehozása

- **`touch` parancs:** Üres fájl létrehozása (vagy a meglévő fájl időbélyegének frissítése).

  ```bash
  touch example.txt
  ```

- **`echo` parancs:** Fájl létrehozása adott tartalommal (kimenet átirányítása).

  ```bash
  echo "Hello, World!" > hello.txt
  ```

- **`cat` parancs:** Fájl létrehozása interaktívan. A parancs kiadása után gépelheted a tartalmat, a mentéshez és kilépéshez a `Ctrl + D` billentyűkombinációt kell használni.

  ```bash
  cat > newfile.txt
  ```

### Könyvtárak létrehozása

- **`mkdir` parancs:** Új könyvtár (mappa) létrehozása.

  ```bash
  mkdir myfolder
  ```

- A `-p` kapcsolóval: Egymásba ágyazott (szülő-gyermek) könyvtárak létrehozása egyszerre.

  ```bash
  mkdir -p parentfolder/childfolder
  ```

> **Megjegyzés:** Grafikus felületen – pl. Ubuntu Nautilus – a jobb gombos menü "Új fájl" / "Új mappa" opcióival is lehetséges a létrehozás.

## 2. Fájlok és könyvtárak biztonságos törlése

### Fájlok törlése (`rm` parancs)

- Alapértelmezett törlés:

  ```bash
  rm filename.txt
  ```

- Több fájl törlése egyszerre (helyettesítő karakterek / wildcards használatával, pl. minden `.txt` fájl törlése):

  ```bash
  rm *.txt
  ```

### Könyvtárak törlése

- **`rmdir` parancs:** KIZÁRÓLAG üres könyvtárak törlésére szolgál. Nem tud törölni olyan mappát, amiben fájlok vannak.

  ```bash
  rmdir emptyfolder
  ```

- **`rm` parancs a `-r` (recursive) kapcsolóval:** Nem üres könyvtárak és azok teljes tartalmának törlése.

  ```bash
  rm -r nonemptyfolder
  ```

- Ugyanez kombinálva a megerősítést kérő opcióval:

  ```bash
  rm -ri nonemptyfolder
  ```

## 3. Hasznos kapcsolók (Flagek) és Opciók az `rm` parancshoz

A törlési parancsok funkcióinak bővítésére és a biztonság növelésére az alábbi kapcsolókat használhatjuk:

- **`-i` (interactive):** Megerősítést kér minden egyes törlés előtt. Különösen hasznos, ha egyszerre több fájlt törlünk.

  ```bash
  rm -i filename.txt
  ```

- **`-v` (verbose):** Bőbeszédű kimenet, pontosan megmutatja a folyamatot, hogy mi lett törölve.

  ```bash
  rm -v filename.txt
  ```

- **`-r` (recursive):** Rekurzívan törli a könyvtárakat és azok minden tartalmát.

- **`-f` (force):** Erőszakos törlés, rákérdezés (megerősítés) nélkül törli a fájlokat/könyvtárakat.

- **`--preserve-root`:** Megakadályozza a gyökérkönyvtár (root) véletlen törlését az `rm` parancs használatakor.
##Forrás
- [https://useful.codes/creating-and-deleting-files-directories-in-linux/]
