# A Linux PATH környezeti változó

A `$PATH` környezeti változó a Linux rendszerek egyik legfontosabb eleme. Amikor beírsz egy parancsot a terminálba, a rendszer a `$PATH` változóra támaszkodva találja meg a futtatni kívánt program helyét.

## Mi az a PATH változó és hogyan működik?

Normál esetben egy egyedi program vagy szkript futtatásához a teljes útvonalat meg kellene adni (pl. `/path/to/script.sh`), vagy a mappájában állva a `./script.sh` parancsot kellene használni. Ugyanakkor számos parancsot (pl. `uptime`, `date`) a pontos elérési út megadása nélkül is futtathatunk.

Ennek oka a `$PATH` változó: ez tartalmazza azon könyvtárak listáját, ahol a Linux automatikusan keresi a beírt programokat.

> **Jelölési konvenciók:** A `$` jel a normál, nem privilegizált felhasználóként futtatott parancsokat jelöli, míg a `#` a root / sudo jogosultsággal futtatandókat.

## 1. A beállított könyvtárak megtekintése

- A `$PATH` változó jelenlegi tartalmának lekérdezése:

  ```bash
  $ echo $PATH
  ```

  (Ez kilistázza az összes könyvtárat, ami jelenleg a PATH-ban van, kettőspontokkal elválasztva).

- Egy adott parancs (program) helyének lekérdezése:

  A `which` paranccsal kideríthető, hogy egy adott parancs pontosan melyik könyvtárból fut.

  ```bash
  $ which date
  /bin/date
  ```

## 2. Könyvtár hozzáadása a PATH változóhoz

Ha hozzáadsz egy könyvtárat a PATH-hoz, a benne lévő programokat és szkripteket a rendszer bármelyik pontjáról futtatni tudod az útvonal megadása nélkül.

### A) Ideiglenes hozzáadás (csak az aktuális munkamenetre / session-re)

A következő parancs az aktuális munkamenet erejéig hozzáadja a `/bin/myscripts` könyvtárat a PATH-hoz (a gép újraindításakor vagy a terminál bezárásakor elveszik):

```bash
$ export PATH="/bin/myscripts:$PATH"
```

Ellenőrzés:

```bash
$ echo $PATH
/bin/myscripts [...]
```

### B) Végleges hozzáadás

A végleges módosításhoz az adott felhasználó `~/.bashrc` fájlját kell szerkeszteni.

1. Nyisd meg a `.bashrc` fájlt a kedvenc szövegszerkesztőddel (pl. `nano`):

   ```bash
   $ nano ~/.bashrc
   ```

2. A fájl legvégére szúrd be az új könyvtárat a következő formátumban:

   ```bash
   export PATH="/bin/myscripts:$PATH"
   ```

3. Mentsd el a fájlt és lépj ki a szerkesztőből.

4. Az új beállítások érvényesítéséhez az aktuális munkamenetben (kijelentkezés vagy újraindítás nélkül) futtasd a `source` parancsot:

   ```bash
   $ source ~/.bashrc
   ```

5. Ellenőrizd a sikeres módosítást:

   ```bash
   $ echo $PATH
   ```

## 3. Könyvtár eltávolítása a PATH-ból

A nem kívánt könyvtár eltávolításához a megfelelő konfigurációs fájlt kell szerkeszteni, és egyszerűen ki kell törölni a listából az adott mappát.

- **Felhasználói szintű módosítás** (csak egy adott user esetén): A `~/.bashrc` fájlban kell módosítani a bejegyzést.

- **Rendszerszintű módosítás** (minden felhasználóra érvényes): Az `/etc/environment` fájlt kell szerkeszteni. (Az innen eltávolított útvonalak az egész rendszerre kihatnak).
##Forrás
-[https://linuxconfig.org/linux-path-environment-variable]

# Hogyan keressük meg egy Linux parancs útvonalát (Path) profi módon

Amikor Linuxon (akár szerveren, akár WSL-ben) dolgozunk, gyakran szükségünk van arra, hogy megtudjuk egy-egy csomagkezelővel (pl. `apt`, `dnf`) telepített program pontos elérési útját.

## Mi az a Linux parancs?

Míg a Windows alapértelmezett felülete grafikus (GUI), a Linux szerverek általában szöveges interfésszel (CLI - parancssor, terminál, Bash, shell) működnek. A terminálba írt utasításokat hívjuk Linux parancsoknak (pl. `date`, `pwd`, `hello`, `ping`). A parancsokat mindig kisbetűvel kell beírni az Enter lenyomása előtt.

A Linux parancsok 5 fő típusa:

1. Belső vagy beépített shell parancs (Internal / built-in shell command)
2. Külső parancs / futtatható fájl (External command / executable / file)
3. Shell függvény (Shell function)
4. Alias (Helyettesítő név)
5. Kulcsszó (Keyword)

## 1. A parancs típusának meghatározása (`type`)

A `type` parancs megmutatja egy parancs útvonalát, és azt is, hogy az adott parancs beépített, alias, függvény vagy külső program-e.

**Szintaxis:**

```bash
type command
type -t command
type -a command
```

**Példák:**

```bash
type pwd
type date
type hello
type ping
```

A `-t` opció egyetlen szót ad vissza (`alias`, `keyword`, `function`, `builtin` vagy `file`):

```bash
type -t ping
type -t if
type -t vi
type -t nano
```

A `-a` opció kilistázza az összes olyan helyet, amely az adott nevű futtatható fájlt tartalmazza:

```bash
type -a ping
```

## 2. Mi az a `$PATH` változó?

A `PATH` változó olyan könyvtárak listáját tartalmazza, ahol a futtatható programok (pl. `ping`, `date`, `vi`, `docker`) tárolva vannak.

A jelenlegi PATH megtekintése:

```bash
echo "$PATH"
```

Emberi olvasásra alkalmasabb (soronkénti) formátumban:

```bash
echo "${PATH//:/$'\n'}"
```

A `/bin/` könyvtárban lévő összes futtatható fájl listázása:

```bash
ls /bin/
ls -l /bin/
```

## 3. Szimbolikus linkek (symlink) tényleges útvonalának feloldása

Ha egy parancs szimbolikus link, a kanonikus (valós) elérési út megtalálására a `realpath` vagy a `readlink` parancsok javasoltak:

```bash
ls -l /bin/ls
realpath /bin/ls
readlink -f /bin/ls
```

## 4. Parancsok útvonalának lekérdezése (`command -v` és `which`)

A `command` paranccsal kilistázható a parancs útvonala:

```bash
command -v date
command -v pwd
command -v ping
command -v docker
```

A `which` parancs a leggyakoribb eszköz a parancs útvonalának megkeresésére:

```bash
which gcc
which nano
```

Az összes lehetséges egyező útvonal kilistázása a `-a` kapcsolóval:

```bash
which -a ls
```

## 5. Binárisok, forráskódok és kézikönyvek (man oldalak) megkeresése (`whereis`)

A `whereis` parancs megkeresi a megadott programhoz tartozó bináris fájlt, a forráskódot és a manuál oldalt a lemezen.

**Szintaxis és alap példák:**

```bash
whereis command
whereis gcc
whereis docker
whereis lxc
whereis vim
```

Csak a bináris fájlok keresése (`-b`):

```bash
whereis -b ls
```

Csak a kézikönyvek / info oldalak keresése (`-m`):

```bash
whereis -m date
whereis -m gcc
```

## 6. Keresés fájlnév alapján (`locate` és `find`)

Fájlokat név alapján is megkereshetünk az egész rendszerben.

A `locate` parancs (gyors, adatbázis alapú keresés):

```bash
locate -b '\date'
```

A `find` parancs (élő, alapos keresés a fájlrendszerben):

```bash
find / -name "date" -ls
```

(Ha minden fájlhoz hozzá akarunk férni, használjunk `sudo`-t):

```bash
sudo find / -name "date" -ls
```

## 7. Segítség és leírások megjelenítése a parancsokhoz

Rövid leírás a `whatis` paranccsal (a manuál oldalak rövidítéseit keresi):

```bash
whatis ls
whatis clear
whatis date
whatis gcc
```

Részletes kézikönyv külső parancsokhoz (`man` vagy `info`):

```bash
man date
man ls
man gcc
man bash
info ls
info bash
man which
```

Segítség a Bash kulcsszavakhoz és beépített parancsokhoz (`help`):

```bash
help if
help exit
help logout
help type
help command
```

## 8. Összefoglaló példa (pl. Docker megkeresése)

Ha telepítetted a Dockert, az alábbi parancsokkal tudod megkeresni és lekérdezni az adatait:

```bash
whatis docker
whereis docker
type -a docker
which docker
find / -iname "docker"
locate -b '\docker'
man docker
```

## 9. Csomaginformációk lekérdezése (Debian / Ubuntu specifikus)

Ha tudni szeretnéd, hogy egy adott csomag (pl. `docker-ce`) pontosan milyen fájlokat telepített a szerverre, használd a `dpkg` parancsot:

```bash
dpkg -L docker-ce
dpkg -L docker-ce | more
```

Ha a csomag általános információira vagy kíváncsi (verzió, függőségek, méret, leírás), használd az `apt` parancsot:

```bash
apt show docker-ce
```
##Forrás
-[https://www.cyberciti.biz/howto/finding-a-path-of-a-linux-command-like-a-pro/]
