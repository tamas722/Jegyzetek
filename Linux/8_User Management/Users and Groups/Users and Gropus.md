# Csoportok (Groups) Létrehozása, Módosítása és Törlése Linuxon

A Linuxban a csoportok felhasználók gyűjteményei. A csoportok létrehozása és kezelése a legegyszerűbb módja annak, hogy egyszerre több felhasználó jogosultságait (fájlokhoz és erőforrásokhoz való hozzáférését) szabályozzuk.

A csoportok adatait (név, azonosító, tagok) az alapértelmezett `/etc/group` konfigurációs fájl tárolja.

## Elsődleges vs. Másodlagos csoportok

- Amikor a `useradd` paranccsal létrehozol egy új felhasználót, a Linux automatikusan létrehoz egy vele megegyező nevű csoportot is. Ez lesz a felhasználó **elsődleges (primary) csoportja**, amelynek ő az egyetlen tagja.
- Ezen felül egy felhasználó bármennyi **másodlagos (secondary/supplementary) csoportnak** is tagja lehet.

## 1. Új csoport létrehozása (`groupadd`)

Új csoport hozzáadásához használd a `groupadd` parancsot:

```bash
sudo groupadd demo
```

Amikor egy csoport létrejön, a rendszer automatikusan kioszt neki egy egyedi csoportazonosítót (GID). Ezt az `/etc/group` fájlban tudod ellenőrizni.

**Csoport létrehozása specifikus GID megadásával (`-g`):**

Ha egy konkrét azonosítót (pl. 1009) szeretnél adni a csoportnak:

```bash
sudo groupadd -g 1009 demo1
```

(Ha a megadott GID már foglalt, a rendszer hibaüzenetet ad, és a művelet megszakad. Ilyenkor próbálkozz egy másik számmal).

## 2. Csoportok módosítása (`groupmod`)

**A csoportazonosító (GID) megváltoztatása (`-g`):**

Ha egy már létező csoport GID-jét szeretnéd módosítani:

```bash
sudo groupmod -g 1011 demo1
```

**A csoport nevének megváltoztatása (`-n` / new-name):**

Ha a `demo1` nevű csoportot `test` névre szeretnéd átnevezni:

```bash
sudo groupmod -n test demo1
```

## 3. Felhasználók hozzáadása és eltávolítása a csoportból

**Felhasználó hozzáadása egy csoporthoz (`usermod`):**

Ha egy meglévő felhasználót (pl. `user1` és `user2`) hozzá akarsz adni a `demo` csoporthoz, a `usermod` parancsot kell használnod az `-a` (append) és `-G` (groups) kapcsolókkal:

```bash
sudo usermod -aG demo user1
sudo usermod -aG demo user2
```

A változtatások sikerességét az `id` paranccsal (vagy az `/etc/group` fájl megnézésével) ellenőrizheted:

```bash
id user1
# Kimenet pl.: uid=1005(user1) gid=1005(user1) groups=100(users),1009(demo)
```

**Felhasználó eltávolítása egy csoportból (`gpasswd`):**

Egy specifikus felhasználó adott csoportból való törléséhez a `gpasswd` parancs `--delete` opcióját érdemes használni:

```bash
sudo gpasswd --delete user1 demo
```

(Alternatív megoldásként manuálisan is szerkesztheted az `/etc/group` fájlt, és kitörölheted a felhasználó nevét a csoport sorából).

## 4. Csoport törlése (`groupdel`)

Ha egy csoportra már nincs szükség, a `groupdel` paranccsal teljesen eltávolíthatod a rendszerből:

```bash
sudo groupdel demo
```

> **Összegzés:** A csoportok használata alapvető fontosságú a Linux jogosultsági rendszerében (permission structure). Logikusan osztályozzák a felhasználókat, ezáltal egyszerűvé és hatékonnyá teszik a fájlokhoz való hozzáférések kezelését.
> ##Forrás
> -[https://www.redhat.com/en/blog/linux-groups]

# Csoportok Kezelése Linuxon (groupadd, groupmod, groupdel)

A csoportok elengedhetetlenek a Linux hozzáférés-vezérlésében (DAC - Discretionary Access Control). Segítségükkel egyszerűen szabályozható, hogy kik férhetnek hozzá bizonyos fájlokhoz és könyvtárakhoz.

## 1. Elsődleges (Primary) vs. Kiegészítő (Supplementary) Csoportok

A legtöbb modern Linux disztribúció a UPG (User Private Group) házirendet használja.

- **Elsődleges csoport:** Amikor létrehozol egy új felhasználót, a rendszer automatikusan létrehoz egy vele megegyező nevű csoportot is. Ez lesz a felhasználó elsődleges csoportja. A felhasználó által létrehozott minden új fájl ehhez a csoporthoz fog tartozni.

  > **Tipp:** Ez a házirend teszi lehetővé a `002`-es umask használatát, amely alapból `775`-ös jogot ad a mappáknak és `664`-est a fájloknak, így a csoport tagjai is olvashatják/írhatják azokat.

A csoportok adatait az `/etc/group` fájl tárolja. Minden sor 4 mezőből áll (kettősponttal elválasztva):

```plaintext
csoportnév : jelszó (általában üres/x) : GID (Csoport Azonosító) : tagok_vesszővel_elválasztott_listája
```

> **Megjegyzés:** Ha egy csoport egy felhasználó elsődleges csoportja, az a felhasználó nem jelenik meg a 4. mezőben.

## 2. Csoport Létrehozása (`groupadd`)

**Egyszerű csoport létrehozása:**

```bash
sudo groupadd linuxconfig
```

A rendszer automatikusan kiosztja a legalacsonyabb elérhető GID-t (Group ID). A kiosztható GID-k tartományát az `/etc/login.defs` fájl `GID_MIN` (általában 1000) és `GID_MAX` változói határozzák meg.

**Csoport létrehozása egyedi GID megadásával (`-g`):**

```bash
sudo groupadd -g 1002 tux
```

**Rendszercsoport (System Group) létrehozása (`-r`):**

A rendszercsoportokat általában szolgáltatások (services) használják. Ezek azonosítója (SYS_GID) egy alacsonyabb tartományból kerül kiosztásra (általában 201 és 999 között).

```bash
sudo groupadd -r tux
```

**Tagok hozzáadása azonnal a létrehozáskor (`-U`):**

Létrehozhatod a csoportot, és rögtön hozzá is adhatsz létező felhasználókat (vesszővel elválasztva, szóközök nélkül).

```bash
sudo groupadd -U foo,bar tux
```

## 3. Csoport Módosítása (`groupmod`)

**A csoport GID-jének megváltoztatása (`-g`):**

```bash
sudo groupmod -g 1006 tux
```

> **Fontos:** Ha a csoport egy felhasználó elsődleges csoportja, a rendszer automatikusan frissíti a felhasználó bejegyzését az `/etc/passwd` fájlban az új GID-re. Azonban a régi GID-vel létrehozott fájlok tulajdonjogát manuálisan kell frissíteni a fájlrendszeren!

**A csoport nevének megváltoztatása (`-n`):**

Ha a `tux` csoportot `bar` névre szeretnéd átnevezni:

```bash
sudo groupmod -n bar tux
```

**A csoport tagjainak újrarendezése / cseréje (`-U`):**

A `-U` opcióval itt is megadhatsz felhasználókat.

> ⚠️ **VIGYÁZAT:** Ez az opció teljesen felülírja a jelenlegi taglistát! Bárki, aki eddig benne volt a csoportban, de nincs felsorolva az új parancsban, el lesz távolítva a csoportból.

## 4. Csoport Törlése (`groupdel`)

**Egyszerű törlés:**

Ha a csoportnak vannak tagjai (mint kiegészítő csoport), azok automatikusan elveszítik a tagságukat, amikor a csoport törlődik.

```bash
sudo groupdel tux
```

**Elsődleges csoport törlésének problémája:**

A rendszer biztonsági okokból alapértelmezésben megtagadja egy olyan csoport törlését, amely egy felhasználó elsődleges csoportja:

```bash
sudo groupdel tim
# Hibaüzenet: groupdel: cannot remove the primary group of user 'tim'
```

(Ideális esetben az elsődleges csoport a felhasználóval együtt törlődik a `userdel` parancs használatakor).

**Erőszakos törlés (`-f`):**

Ha mindenképpen törölni akarod az elsődleges csoportot, használhatod az `-f` (force) kapcsolót:

```bash
sudo groupdel -f tim
```

**Következmény:** A felhasználó (`tim`) ezután egy olyan GID-hez lesz társítva, ami már nem létezik. Ezt a problémát úgy oldhatod meg, ha egy új elsődleges csoportot rendelsz hozzá a `usermod -g uj_csoport tim` paranccsal.

> ##Forrás
> -[https://linuxconfig.org/how-to-manage-groups-on-linux]
