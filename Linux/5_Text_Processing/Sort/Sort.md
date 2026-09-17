# A Linux `sort` Parancs Használata és Példák

A `sort` parancs a GNU Coreutils csomag része. Arra szolgál, hogy egy megadott szövegfájl (vagy a szabványos bemenet) sorait növekvő vagy csökkenő sorrendbe rendezze. A rendezett kimenet megjelenhet a képernyőn (szabványos kimenet), vagy egy fájlba is menthető.

## 1. Alapszintaxis

```bash
sort [options] [file(s)]
```

(Ha nincs fájl megadva, a `sort` a szabványos bemenetről – pl. a billentyűzetről vagy egy pipe-ból – érkező adatokat rendezi).

## 2. Gyakran használt kapcsolók (Options / Flags)

- `-r` (reverse): Fordított sorrendbe rendezi a bemenetet (csökkenő).
- `-n` (numeric): Numerikus (számszaki) rendezés. Akkor hasznos, ha a szövegként reprezentált számokat nagyság szerint akarjuk sorba állítani.
- `-k` (key/field): Rendezés egy specifikus mező (oszlop) alapján.
- `-b` (blank): A vezető szóközök (üres karakterek) figyelmen kívül hagyása a sorok elején.
- `-t` (separator): A mezőelválasztó (field separator) karakter megadása.
- `-u` (unique): A duplikált (ismétlődő) sorok eltávolítása a kimenetből.
- `-o` (output): A kimeneti fájl megadása.

## 3. Gyakorlati Példák

### Alapértelmezett rendezés (Fájl tartalmának rendezése)

Alapértelmezésben a parancs növekvő sorrendbe rendez az első karakter alapján:

```bash
sort data.txt 
```

**Bemenet átadása a `sort` parancsnak alternatív módokon:**

```bash
sort < data.txt 
cat data.txt | sort 
```

### Rendezés fordított sorrendben (`-r`)

A sorok fordított (csökkenő) sorrendbe rendezése:

```bash
sort -r data.txt 
```

### Numerikus rendezés (`-n`)

Ha a fájl számokat tartalmaz, a `-n` kapcsolóval érhetjük el, hogy a számok valódi értékük szerint legyenek sorba állítva (ne pedig alfabetikusan):

```bash
sort -n data.txt 
```

### Rendezés mező (oszlop) alapján (`-k`)

A `-k` kapcsolóval megadható, hogy melyik oszlop alapján történjen a rendezés (a számozás 1-től indul). Ez táblázatos adatoknál nagyon hasznos. Példa a 2. oszlop alapján történő rendezésre:

```bash
sort -k 2 data.txt 
```

### Mezőelválasztó karakter megadása (`-t`)

Alapértelmezésben a `sort` szóközt vagy tabulátort vár elválasztóként. A `-t` paranccsal ezt felülbírálhatjuk. Például, ha egy CSV fájlban vessző (`,`) az elválasztó, és a 2. oszlop szerint rendeznénk:

```bash
sort -t "," -k 2 data.txt 
```

### Duplikált sorok eltávolítása (`-u`)

Nagy fájloknál hasznos, ha csak az egyedi sorokat szeretnénk megtartani (eltávolítja a másodpéldányokat):

```bash
sort -u data.txt 
```

### Kimenet mentése fájlba (`-o` vagy `>`)

A képernyőre való kiírás helyett a rendezett adatokat egy új fájlba is menthetjük a `-o` kapcsolóval:

```bash
sort data.txt -o sorted_data.txt 
```

Ugyanez elérhető a standard átirányító operátor (`>`) használatával is:

```bash
sort data.txt > sorted_data.txt 
```

## 4. Opciók Kombinálása (Haladó használat)

A kívánt eredmény eléréséhez a kapcsolók szabadon kombinálhatók.

- **Fordított + Numerikus + Mentés fájlba:**

  Rendezés számok szerint (`n`), fordított sorrendben (`r`), és a kimenet mentése a `reverse_sorted_data.txt` fájlba (`-o`):

  ```bash
  sort -nr data.txt -o reverse_sorted_data.txt 
  ```

- **Több oszlop + Egyedi elválasztó + Duplikációk szűrése:**

  Rendezés vessző (`,`) elválasztó alapján (`-t`), a 2. és 3. oszlopokat figyelembe véve (`-k 2,3`), valamint az ismétlődések eltávolítása (`-u`):

  ```bash
  sort -t "," -k 2,3 -u data.txt
  ```

  ##Forrás
  -[https://tecadmin.net/linux-sort-command/]
  -[https://labex.io/tutorials/linux-linux-sort-command-text-sorting-219196]
  -[https://linuxhandbook.com/sort-command/]
