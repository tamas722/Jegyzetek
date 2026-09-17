# Linux Fájljogosultságok és Tulajdonjog (File Ownership and Permissions)

A Linux egy többfelhasználós operációs rendszer. A biztonság érdekében a rendszer két tényező alapján dönti el, hogy ki férhet hozzá egy adott fájlhoz vagy könyvtárhoz: a fájl tulajdonjoga és a fájl jogosultságai.

> **Megjegyzés:** A cikkben említett vizuális magyarázó ábrák az `ls -l` kimenetéről és a jogosultsági blokkokról az eredeti oldalon tekinthetők meg: Linux Handbook - File Permissions

## 1. Fájl Tulajdonosok (File Ownership)

Minden fájlnak és könyvtárnak háromféle tulajdonosa van a Linuxban:

- **User (Felhasználó):** A fájl elsődleges tulajdonosa (általában az, aki létrehozta).

- **Group (Csoport):** Egy csoport több felhasználót is tartalmazhat (pl. fejlesztők, tesztelők). Ha a fájlt hozzárendeljük egy csoporthoz, a csoport minden tagja megkapja a csoportra vonatkozó jogosultságokat.

  > **Tipp:** A `groups` paranccsal megnézheted, milyen csoportokba tartozol. Az `id -gn <username>` paranccsal lekérhető a felhasználó elsődleges csoportja.

- **Other (Mindenki más):** Minden más felhasználó a rendszeren, aki nem a fájl tulajdonosa (User) és nem is tagja a tulajdonos csoportnak (Group).

## 2. Fájl Jogosultságok (File Permissions)

Mind a három tulajdonostípushoz (User, Group, Other) háromféle jogosultság állítható be:

**Fájlok esetén:**

- **Read (r) - Olvasás:** Megtekinthető vagy másolható a fájl tartalma.
- **Write (w) - Írás:** Módosítható a fájl tartalma.
- **Execute (x) - Végrehajtás:** Futtatható a fájl (ha az egy program/szkript).

**Könyvtárak (Mappák) esetén:**

- **Read (r) - Olvasás:** Kilistázható a könyvtár tartalma (fájlok másolása onnan).
- **Write (w) - Írás:** Fájlok hozhatók létre vagy törölhetők a könyvtárból (ehhez Execute jog is kell).
- **Execute (x) - Végrehajtás:** Be lehet lépni a könyvtárba (pl. `cd` paranccsal).

## 3. Jogosultságok megtekintése

A fájlok jogosultságait a `stat` vagy az `ls -l` paranccsal lehet lekérdezni.

**Példa az `ls -l` kimenetére:**

```bash
-rwxrw-r-- 1 abhi itsfoss 457 Aug 10 11:55 agatha.txt
```

**A kimenet felépítése:**

- **Fájltípus:** Az első karakter (`d` = könyvtár, `-` = normál fájl, `l` = szimbolikus link).
- **Jogosultságok (`rwxrw-r--`):** 9 karakterből álló blokk.
- **Hard linkek száma:** Hány hard link mutat a fájlra (alapértelmezés: 1).
- **User:** A fájl tulajdonosa (a példában `abhi`).
- **Group:** A fájl csoportja (a példában `itsfoss`).
- **Méret:** A fájl mérete bájtokban (457).
- **Módosítás ideje:** Dátum és idő (Aug 10 11:55).
- **Fájlnév:** A fájl vagy mappa neve (`agatha.txt`).

**A 9 karakteres jogosultsági blokk értelmezése (`rwxrw-r--`):**

A jogosultságok mindig User, Group, Other sorrendben, és azon belül mindig r, w, x sorrendben jelennek meg. A kötőjel (`-`) azt jelenti, hogy az adott jog nincs megadva.

- **`rwx` (User):** A tulajdonos (`abhi`) olvashatja, írhatja és futtathatja.
- **`rw-` (Group):** A csoport (`itsfoss`) olvashatja és írhatja, de nem futtathatja.
- **`r--` (Other):** Mindenki más csak olvashatja.

(A root felhasználó szupererővel bír, és általában mindent olvashat/írhat/futtathat, még ha ez nincs is feltüntetve a jogosultságok között).

## 4. Jogosultságok megváltoztatása (`chmod` parancs)

A jogosultságokat a `chmod` (change mode) paranccsal lehet módosítani. Kétféleképpen használható: abszolút (numerikus) és szimbolikus módban.

### A) Abszolút (Numerikus) Mód

Minden jognak van egy számértéke (oktális rendszer):

- `r` (read) = 4
- `w` (write) = 2
- `x` (execute) = 1
- `-` (nincs jog) = 0

Ezek összeadásával kapjuk meg a jogosultsági szintet (0-tól 7-ig):

- `0` = `---`
- `1` = `--x`
- `2` = `-w-`
- `3` = `-wx` (2+1)
- `4` = `r--`
- `5` = `r-x` (4+1)
- `6` = `rw-` (4+2)
- `7` = `rwx` (4+2+1)

**Példa:** Adjunk olvasási és írási jogot (6) mind a 3 kategóriának (User, Group, Other):

```bash
chmod 666 agatha.txt
```

Eredmény: `-rw-rw-rw-`

### B) Szimbolikus Mód

Ebben a módban betűkkel és operátorokkal dolgozunk, így elég csak azt módosítani, amit tényleg akarunk.

- **Kinek?** `u` (user), `g` (group), `o` (other), `a` (all - mindenki).
- **Mit csináljon?** `+` (jog hozzáadása), `-` (jog elvétele), `=` (pontosan erre állítja be, felülírva a régit).

**Példák:**

Csoport (group) számára futtatási (execute) jog hozzáadása:

```bash
chmod g+x agatha.txt
```

Több módosítás egyszerre: Mindenki mástól (o) vegyük el az olvasási és írási jogot, de adjunk futtatásit, a tulajdonosnak (u) pedig adjunk futtatásit:

```bash
chmod o-rw+x,u+x agatha.txt
```

Mindenkitől (a) vegyük el a futtatási jogot:

```bash
chmod a-x agatha.txt
```

## 5. Tulajdonjog megváltoztatása (`chown` és `chgrp` parancsok)

A fájl tulajdonosának és csoportjának megváltoztatásához a `chown` (change owner), illetve a `chgrp` (change group) parancsokat használjuk. Mivel a tulajdonjog cseréje kényes művelet, általában `sudo` jog (superuser) szükséges hozzá.

- Csak a felhasználó (user) cseréje:

  ```bash
  chown <new_user_name> <filename>
  ```

- Felhasználó ÉS csoport cseréje egyszerre (kettősponttal elválasztva):

  ```bash
  chown <new_user_name>:<new_user_group> <filename>
  ```

- Példa: Változtassuk a tulajdonost és a csoportot is root-ra:

  ```bash
  sudo chown root:root agatha.txt
  ```

- Csak a csoport (group) cseréje (kétféleképpen is lehet):

  ```bash
  chown :<new_user_group> <filename>
  # VAGY
  chgrp <new_user_group> <filename>
  ```

## Bónusz Tipp: A jogosultságok prioritása (Precedence)

Képzeljük el a következő szituációt:

```bash
----r--rw- 1 abhi itsfoss 457 Aug 10 11:55 agatha.txt
```

A tulajdonosnak (`abhi`) semmilyen joga nincs (`---`), a csoportnak olvasási van (`r--`), mindenki másnak pedig olvasási és írási (`rw-`).

**Kérdés:** El tudja olvasni a tulajdonos (`abhi`) a fájlt?

**Válasz:** NEM.

A Linux szigorú sorrendben ellenőrzi a jogosultságokat: User -> Group -> Other.

Amikor `abhi` megpróbálja megnyitni a fájlt, a rendszer látja, hogy ő a tulajdonos (User). Megnézi a User jogokat: `---`. Mivel a tulajdonosi szinten nincs joga, a rendszer azonnal elutasítja a hozzáférést, és nem vizsgálja tovább sem a Group, sem az Other jogokat (hiába tagja a csoportnak, és hiába van az Other-nek teljesebb joga).
