# Felhasználókezelés (User Management) Linuxon

A felhasználók kezelése alapvető készség a Linux rendszereken, akár egy helyi szervert, akár felhős infrastruktúrát üzemeltetsz. Ez az útmutató bemutatja a Linux fiókok alapjait, típusait, azonosítóit, valamint a létrehozásukhoz és módosításukhoz szükséges parancsokat.

## 1. Mi az a Felhasználó (User) a Linuxban?

A felhasználó egy bejelentkezési fiók (login account), amely hozzáférést biztosít a számítógép erőforrásaihoz egy személy vagy egy rendszerfolyamat számára. Minden felhasználónak megvannak a saját jogosultságai (permissions), amelyek szabályozzák, hogy mit tehet és mit nem.

## 2. A Felhasználók Típusai

**Rendszer felhasználó (System User):**

- Automatikusan jön létre az operációs rendszer telepítésekor.
- Szolgáltatások (services) és háttérfolyamatok futtatására szolgál.
- Példák: `root`, `apache`, `daemon`.

**Normál felhasználó (Normal User):**

- Manuálisan hozza létre a rendszergazda.
- Hús-vér emberek használják bejelentkezésre és alkalmazások futtatására.
- Példák: `john`, `developer1`.

## 3. A UID (Egyedi Felhasználó Azonosító) megértése

Minden felhasználó kap egy egyedi számazonosítót, azaz UID-t (User Identifier).

| UID Tartomány | Leírás |
| --- | --- |
| 0 | Fenntartva a root (superuser / teljhatalmú rendszergazda) számára. |
| 1 – 999 | Fenntartva a rendszer- és szolgáltatás-felhasználók számára. |
| 1000 – 60000 | Normál (emberi) felhasználói fiókok tartománya. |

## 4. Hol tárolja a Linux a felhasználói adatokat?

- `/etc/passwd`: A felhasználói fiókok alapvető információit (pl. név, UID, Home könyvtár) tárolja.
- `/etc/shadow`: A felhasználók titkosított jelszavait és a jelszavak lejárati/biztonsági információit (aging info) tárolja.

## 5. Alapvető Felhasználókezelési Parancsok

**Új felhasználó létrehozása:**

```bash
useradd james
```

**Jelszó beállítása vagy megváltoztatása:**

(A rendszer biztonsági okokból kétszer fogja bekérni az új jelszót, gépelés közben a karakterek nem látszanak).

```bash
passwd james
```

**Felhasználói tulajdonságok lekérdezése:**

```bash
grep james /etc/passwd
```

**Jelszó tulajdonságainak (és lejáratának) lekérdezése:**

```bash
grep james /etc/shadow
```

**Váltás egy másik felhasználóra (Switch user):**

```bash
su james
```

**Meglévő felhasználónév megváltoztatása (`-l` / login name):**

```bash
# Szintaxis: usermod -l uj_nev regi_nev
usermod -l james daniel
```

**Felhasználó törlése:**

```bash
# Csak a felhasználói fiók törlése:
userdel james

# A felhasználó ÉS a hozzá tartozó saját könyvtár (home directory) teljes törlése:
userdel -r james
```

## 6. Csoportok (Groups) Kezelése

**Új csoport létrehozása:**

```bash
groupadd itdepartment
```

**Felhasználó hozzáadása egy csoporthoz:**

A `-aG` (append to Group) opció hozzáadja a felhasználót egy kiegészítő csoporthoz anélkül, hogy a többi csoportjából kiléptetné.

```bash
# Szintaxis: usermod -aG csoportnev felhasznalonev
usermod -aG itdepartment james
```

**Egy felhasználó csoportjainak listázása:**

```bash
groups james
```

## 💡 Tippek

Ha nem vagy biztos egy parancs működésében vagy az opciókban, használd a `man` (manual) vagy az `info` parancsot. Ezek beépített, részletes dokumentációt biztosítanak közvetlenül a terminálban.

Példa: `man useradd` vagy `man usermod`

##Forrás
-[https://dev.to/austinozor/user-management-in-linux-a-beginners-guide-to-creating-modifying-and-deleting-users-fhf]

# Felhasználói Fiókok Kezelése Linuxon (useradd, usermod, userdel)

A felhasználókezelés minden Linux rendszergazda alapvető feladata. A fiókok kezelésére a beépített `useradd`, `usermod` és `userdel` parancssori eszközöket használjuk.

## Alapértelmezett Beállítások (`/etc/login.defs`)

Amikor létrehozunk egy új felhasználót, a rendszer az `/etc/login.defs` fájlban megadott alapértelmezett szabályokat alkalmazza. Ezeket a parancssori kapcsolókkal (opciókkal) felül lehet írni. Néhány fontosabb beállítás a fájlból:

- `CREATE_HOME`: Létrejöjjön-e automatikusan a felhasználó saját (home) mappája (általában `yes`).
- `UID_MIN` / `UID_MAX`: A normál felhasználóknak automatikusan kiosztható UID-k (azonosítók) tartománya (általában 1000 - 60000).
- `SYS_UID_MIN` / `SYS_UID_MAX`: A rendszerfelhasználóknak kiosztható UID-k tartománya.
- `PASS_MAX_DAYS`: A jelszó érvényességének maximális ideje (napokban).

## 1. Új Felhasználó Létrehozása (`useradd`)

Az alapértelmezett beállítások szerinti felhasználó létrehozása (automatikusan létrejön a home könyvtár is, ha a fent említett beállítás engedélyezi):

```bash
sudo useradd newuser
```

> **Fontos:** A felhasználó létrehozása után mindig be kell állítani a jelszavát a `passwd` paranccsal: `sudo passwd newuser`.

**Home könyvtár szabályozása:**

Ha explicit módon kérni akarjuk a home mappa létrehozását (`-m`), vagy épp meg akarjuk tiltani azt (`-M`):

```bash
sudo useradd -m newuser   # Létrehozza a home mappát
sudo useradd -M newuser   # NEM hozza létre a home mappát
```

**Alapértelmezett Shell megadása (`-s`):**

Ha nem a rendszer alapértelmezett shelljét (pl. bash) akarjuk használni:

```bash
sudo useradd -s /bin/bash newuser
```

**Egyedi UID megadása (`-u`):**

```bash
sudo useradd -u 1005 newuser
```

**Rendszerfelhasználó (System User) létrehozása (`-r`):**

A rendszerfelhasználókat démonok és szolgáltatások használják. Nem kapnak home könyvtárat, és az azonosítójuk (UID) az 1000 alatti tartományba esik.

```bash
sudo useradd -r newuser
```

**Hozzáadás kiegészítő csoportokhoz létrehozáskor (`-G`):**

Például a `wheel` (vagy Ubuntu/Debian alatt `sudo`) csoporthoz adás, hogy a felhasználó futtathasson rendszergazdai parancsokat:

```bash
sudo useradd -G wheel newuser
```

## 2. Meglévő Felhasználó Módosítása (`usermod`)

A `usermod` parancs szintaxisa szinte megegyezik a `useradd` paranccsal, de már létező fiókokon hajt végre változtatásokat.

**Hozzáadás meglévő csoporthoz (`-aG`):**

Ha egy felhasználót hozzá akarsz adni egy újabb csoporthoz, mindig használd az `-a` (append) és a `-G` kapcsolókat együtt! (Ha csak a `-G`-t használod, a rendszer felülírja a felhasználó teljes csoportlistáját, és kilépteti az eddigi csoportjaiból!)

```bash
sudo usermod -aG newgroup newuser
```

**Felhasználói fiók (jelszó) zárolása és feloldása (`-L` és `-U`):**

Ha ideiglenesen meg akarod tiltani a bejelentkezést, zárolhatod a fiókot (Lock). Ez egy `!` jelet tesz a jelszó hash elé az `/etc/shadow` fájlban.

```bash
sudo usermod -L newuser   # Fiók zárolása
sudo usermod -U newuser   # Fiók feloldása (Unlock)
```

**Felhasználónév (Login Name) megváltoztatása (`-l`):**

```bash
# Szintaxis: usermod -l uj_nev regi_nev
sudo usermod -l linuxconfig newuser
```

> **Megjegyzés:** Ez csak a bejelentkezési nevet változtatja meg, a home könyvtár nevét nem!

**Home könyvtár módosítása és a fájlok áthelyezése (`-d` és `-m`):**

Ha megváltoztatod a felhasználó home könyvtárát (`-d`), és a régi mappából minden adatot át is akarsz mozgatni az újba (`-m` / move-home):

```bash
sudo usermod -d /home/newuser_new -m newuser
```

**UID és Elsődleges GID módosítása (`-u` és `-g`):**

Ha megváltoztatod az UID-t vagy a primer csoportazonosítót (GID), a rendszer automatikusan frissíti a felhasználó home könyvtárában lévő fájlok tulajdonjogait is.

```bash
sudo usermod -u 1000 newuser
sudo usermod -g 1006 newuser
```

## 3. Felhasználó Törlése (`userdel`)

A `userdel` paranccsal törölhetjük a fiókot.

**Csak a fiók törlése:**

Ez a parancs törli a felhasználót, de a lemezen hagyja a saját (home) mappáját és a benne lévő fájlokat.

```bash
sudo userdel newuser
```

**Fiók ÉS a saját (home) könyvtár törlése (`-r`):**

A leggyakoribb eset: a felhasználó törlése az összes saját adatával és a levelezési (spool) fájljaival együtt.

```bash
sudo userdel -r newuser
```

**Erőszakos törlés (`-f`):**

Ha a felhasználó éppen be van jelentkezve, vagy folyamatokat futtat, a rendszer visszautasítja a törlést. A `-f` (force) kapcsolóval kikényszeríthető, de ez veszélyes, mert inkonzisztens állapotban hagyhatja a rendszert.

```bash
sudo userdel -f -r newuser
```

##Forrás
-[https://linuxconfig.org/how-to-create-modify-and-delete-users-account-on-linux]
