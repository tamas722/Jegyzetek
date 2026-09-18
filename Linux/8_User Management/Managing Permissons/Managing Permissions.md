# Linux Fájljogosultságok és Kezelésük (chmod, ls -l)

A Linuxban minden fájl és könyvtár rendelkezik jogosultságokkal (permission bits), amelyek meghatározzák, hogy ki és mit csinálhat az adott fájllal. Ezek az engedélyek biztosítják, hogy csak az arra jogosult felhasználók vagy folyamatok férhessenek hozzá a rendszer adataihoz.

## 1. A Jogosultsági Modell Alapjai

A Linux három különböző felhasználói osztályhoz (class) rendel engedélyeket minden fájl esetében:

- **Tulajdonos (Owner / u - user):** Az a felhasználó, aki a fájlt létrehozta (vagy akire átruházták).
- **Csoport (Group / g):** A fájlhoz rendelt csoport tagjai.
- **Mások (Others / o):** Mindenki más a rendszeren.

Mind a három osztályhoz háromféle alapvető jogosultság állítható be:

- **Olvasás (Read / r):** A fájl tartalmának megtekintése.
- **Írás (Write / w):** A fájl módosítása, törlése.
- **Végrehajtás (Execute / x):** A fájl futtatása programként vagy szkriptként.

(A fájl tulajdonosát a `chown`, a csoportját pedig a `chgrp` paranccsal lehet megváltoztatni).

## 2. A Jogosultságok Olvasása (`ls -l`)

A jogosultságok megtekintéséhez használd az `ls -l` parancsot:

```bash
ls -l file_name
# Kimenet: -rw-r--r-- 1 linuxize users 12288 Apr 28 10:10 file_name
```

A kimenet első 10 karaktere (`-rw-r--r--`) írja le a fájltípust és a jogosultságokat:

1. **karakter (Fájltípus):** `-` (normál fájl), `d` (könyvtár), `l` (szimbolikus link).
2. **2-4. karakter (Tulajdonos):** `rw-` (Olvasás és írás engedélyezett, végrehajtás nem).
3. **5-7. karakter (Csoport):** `r--` (Csak olvasás).
4. **8-10. karakter (Mások):** `r--` (Csak olvasás).

## 3. A Jogosultságok Hatása (Fájlok vs. Könyvtárak)

A jogosultságok mást jelentenek egy fájl és mást egy könyvtár (mappa) esetében.

| Karakter | Fájl esetén jelentése | Könyvtár (Mappa) esetén jelentése |
| --- | --- | --- |
| `r` (Read) | A fájl tartalma olvasható. | A mappa tartalma (a benne lévő fájlok nevei) kilistázható (`ls`). |
| `w` (Write) | A fájl tartalma módosítható. | A mappában új fájlok hozhatók létre, a meglévők törölhetők vagy átnevezhetők (de csak ha az `x` is be van állítva!). |
| `x` (Execute) | A fájl futtatható (pl. szkriptként). | A mappába be lehet lépni (`cd`), és el lehet érni a benne lévő fájlokat név alapján. |

(Speciális bitek: Az `s` a setuid/setgid bitet jelöli, amely futtatáskor a tulajdonos/csoport jogait adja meg a programnak. A `t` a sticky bit, mappákon pl. a `/tmp` könyvtáron akadályozza meg, hogy a felhasználók töröljék egymás fájljait).

## 4. A Jogosultságok Megváltoztatása (`chmod`)

A jogosultságokat a `chmod` paranccsal lehet módosítani. Ezt megtehetjük **Szimbolikus (betűs)** vagy **Numerikus (számos)** módszerrel.

### A) Szimbolikus Módszer (Betűkkel)

**Szintaxis:** `chmod [osztály][művelet][jogosultság] fájlnév`

- **Osztályok:** `u` (owner), `g` (group), `o` (others), `a` (all - mindenki).
- **Műveletek:** `+` (hozzáadás), `-` (elvétel), `=` (pontos beállítás).
- **Jogosultságok:** `r`, `w`, `x`

**Példák:**

```bash
# Végrehajtási (futtatási) jog adása a csoportnak:
chmod g+x filename

# Írási jog elvétele mindenkitől:
chmod a-w filename

# Olvasás és Írás beállítása a tulajdonosnak, Olvasás a csoportnak, minden jog elvétele a többiektől:
chmod u=rw,g=r,o= filename

# Végrehajtási jog elvétele a "többiektől" rekurzívan (minden almappában is):
chmod -R o-x dirname
```

### B) Numerikus (Oktális) Módszer

A leggyorsabb módszer, ahol 3 számjegy határozza meg a három osztály (tulajdonos, csoport, mások) jogait. A számok a jogosultságok értékeinek összeadásából jönnek ki:

- Olvasás (`r`) = 4
- Írás (`w`) = 2
- Végrehajtás (`x`) = 1
- (Nincs jog = 0)

**Példa számítás (Tulajdonos: rwx, Csoport: r-x, Mások: r--):**

- Tulajdonos: 4 + 2 + 1 = 7
- Csoport: 4 + 0 + 1 = 5
- Mások: 4 + 0 + 0 = 4
- A végeredmény: **754**.

**Példák:**

```bash
# 644 (Tulajdonos: rw, Csoport: r, Mások: r): Alapértelmezett beállítás fájlokra.
chmod 644 file

# 755 (Tulajdonos: rwx, Csoport: rx, Mások: rx): Alapértelmezett beállítás mappákra és futtatható fájlokra.
chmod 755 file

# 700 (Tulajdonos: rwx, mindenki másnak semmi): Maximálisan privát mappa/fájl.
chmod 700 file
```

> **Tipp:** A fájl aktuális jogosultságait numerikus formátumban a `stat -c "%a" file_name` paranccsal tudod lekérdezni.

## 5. Gyors Referencia Táblázat (Leggyakoribb Numerikus Értékek)

| Mód (Szám) | Szimbolikus | Ki mit tehet? |
| --- | --- | --- |
| 777 | rwxrwxrwx | Bárki (Mindenki) olvashat, írhat és futtathat. (Rendkívül veszélyes, kerülni kell!) |
| 755 | rwxr-xr-x | Tulajdonos: Teljes hozzáférés. Csoport és Mások: Csak olvasás és futtatás (mappába belépés). |
| 750 | rwxr-x--- | Tulajdonos: Teljes hozzáférés. Csoport: Olvasás és futtatás. Mások: Semmi hozzáférés. |
| 700 | rwx------ | Tulajdonos: Teljes hozzáférés. Csoport és Mások: Semmi hozzáférés. |
| 664 | rw-rw-r-- | Tulajdonos és Csoport: Olvasás és írás. Mások: Csak olvasás. |
| 644 | rw-r--r-- | Tulajdonos: Olvasás és írás. Csoport és Mások: Csak olvasás. (Általános fájl jog). |
| 600 | rw------- | Tulajdonos: Olvasás és írás. Csoport és Mások: Semmi hozzáférés. |

## 6. Alapértelmezett Jogosultságok (A `umask`)

Amikor létrehozol egy új fájlt vagy mappát, nem kell kézzel beállítanod a jogokat. A rendszer egy alapértelmezett maszkot használ, ami "kivonja" a jogokat egy maximum értékből (Fájloknál 666, mappáknál 777).

A jelenlegi maszk lekérdezése:

```bash
umask
# Példa kimenet: 0022
```

##Forrás
-[https://linuxize.com/post/understanding-linux-file-permissions/]
-[https://linuxhandbook.com/linux-file-permissions/]
-[https://labex.io/tutorials/linux-permissions-of-files-270252]
Egy `0022`-es maszk azt jelenti, hogy a "csoport" és a "mások" írási joga (2) le van tiltva. Így az új fájlok `644`-es (666-022), az új mappák pedig `755`-ös (777-022) jogosultsággal jönnek létre.
