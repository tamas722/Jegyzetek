# A Linux Rendszergazdai (Superuser) Hozzáférés

A Linuxon minden a "root"-tól indul: a merevlemeznek van egy root (gyökér) partíciója (`/`), és hagyományosan az alapértelmezett kezdeti felhasználó is a root (vagy más néven a "superuser"). Régebben a root fiókot használták bejelentkezésre és a normál felhasználók létrehozására.

Napjainkban azonban a root felhasználó közvetlen használata már felesleges (és biztonsági kockázat). Ehelyett a `sudo` parancsot használjuk a "superuser" feladatok elvégzésére.

## 1. A root fiók zárolása telepítéskor

Egy behatoló biztosan tudja, hogy minden Unix/Linux rendszeren létezik egy `root` nevű felhasználó. Ha ezt a fiókot zároljuk, azzal jelentősen megnehezítjük az illetéktelen behatolók dolgát.

- **Telepítés alatti zárolás:** Telepítéskor (pl. az Anaconda telepítőben) lehetőség van a root fiók zárolására. Ez megakadályozza, hogy bárki közvetlenül root-ként jelentkezzen be.

- **A wheel csoport:** Ha a root fiókot zárolod, a telepítéskor létrehozott normál felhasználónak adminisztrátori jogokat kell adni. Ezzel a felhasználó bekerül a speciális `wheel` csoportba, amelynek tagjai alapértelmezés szerint bármilyen parancsot futtathatnak a `sudo` segítségével (lényegében root jogosultsággal), ami sokkal biztonságosabb, mint a `su` paranccsal közvetlenül root-ra váltani.

## 2. A root fiók utólagos letiltása (Disable root)

Ha telepítéskor nem zároltad a root fiókot, adminisztrátori jogokkal utólag is megteheted. Ehhez a root felhasználó alapértelmezett parancsértelmezőjét (shell) `/sbin/nologin`-ra kell állítani az `/etc/passwd` fájlban.

**A letiltó parancs:**

```bash
$ sudo sed -i 's_root:/bin/bash_root:/sbin/nologin_' /etc/passwd
```

**A módosítás tesztelése** (próbáljunk meg root-ra váltani a `su` paranccsal):

```bash
$ sudo su -
[sudo] password for tux: 
This account is currently not available.
```

## 3. Az adminisztráció delegálása (Szelektív jogok)

A `su` parancs "mindent vagy semmit" elven működik: ha tudod a root jelszavát, teljhatalmad van; ha nem, semmit sem tehetsz.

A `sudo` és a `/etc/sudoers` fájl segítségével azonban finomhangolhatjuk a jogokat, azaz delegálhatunk feladatokat (pl. megengedhetjük egy felhasználónak, hogy új csoportokat hozzon létre, de semmi mást).

A `/etc/sudoers` fájlt kizárólag a `visudo` paranccsal szabad szerkeszteni. Ez alapértelmezetten a `vi` szerkesztőt használja, de az `EDITOR` változóval felülbírálható:

```bash
$ sudo EDITOR=nano visudo
```

**Példa jogok delegálására:**

Keresd meg a fájlban a parancsjogsultságokat definiáló részt, és add hozzá a kívánt felhasználót és parancsot. Engedélyezzük a `shadowman` felhasználónak a `groupadd` parancs futtatását:

```plaintext
## Allow root to run any commands anywhere 
# root    ALL=(ALL)     ALL
shadowman ALL=/sbin/groupadd
```

**A beállítás tesztelése:**

Jelentkezzünk be `shadowman` fiókkal, futtassuk az engedélyezett, majd egy nem engedélyezett parancsot:

```bash
$ su - shadowman
Password: 
$ sudo groupadd qa
[sudo] password for shadowman: 
$ sudo ls
Sorry, user bogan is not allowed to execute '/bin/ls' as root on darkstar.sysadmin.local.
```

(A rendszer engedi a `groupadd` lefutását, de az `ls`-t már megtagadja root-ként).

## 4. Sudo Aliasok használata

A sudoers fájlban aliasokat (helyettesítő neveket) is létrehozhatunk, amelyekkel gépeket, parancsokat és felhasználókat csoportosíthatunk egybe. (Például létrehozhatunk egy `admin` csoportot felhasználókezelő parancsokkal, és egy `szoftver` csoportot csomagkezelő parancsokkal).

**Példa alias létrehozására:**

Engedélyezzük a `softadmins` Unix csoport minden tagjának, hogy futtassák a `SOFTWARE` nevű parancs aliasban megadott összes parancsot (`dnf` és `rpm`):

```plaintext
## Installation and management of software
Cmnd_Alias SOFTWARE = /usr/bin/dnf, /usr/bin/rpm
softadmins ALL=SOFTWARE
```

## Összegzés

Soha ne add oda a kulcsokat indokolatlanul! Tiltsd le a root fiókot a széleskörű és ellenőrizetlen jogosultság-kiterjesztés elkerülése érdekében. Használd a `sudo` parancsot és egy jól karbantartott sudoers fájlt annak precíz kezelésére, hogy ki és mit menedzselhet a számítógépeken.

##Forrás
-[https://www.redhat.com/en/blog/linux-superuser-access]

# Linux Felhasználói Fiókok és Szuperuser (Superuser) Hozzáférés

## 1. Linux Felhasználói Fiókok Típusai

Egy önálló Linux rendszeren öt fő felhasználói fióktípus létezik:

- **Root fiók:** Az alapértelmezett, legmagasabb jogosultságú fiók, amely minden Linux disztribúcióban elérhető. Minden parancsot futtathat, minden fájlt szerkeszthet. A legtöbb disztribúció alapértelmezés szerint zárolja, vagy azt javasolja, hogy ne használjuk rutin feladatokhoz.

- **Superuser fiókok:** Azok a disztribúciók használják, amelyek alapértelmezésben nem zárolják a root fiókot. Rendelkeznek egy alapértelmezett jogosultsági csoporttal, a `wheel` csoporttal. Bármely regular (normál) felhasználóból superuser készíthető, ha beletesszük a `wheel` csoportba. Ez a státusz állandó (amíg ki nem veszik a csoportból).

- **Sudo user fiókok:** Azok a disztribúciók használják, amelyek alapértelmezésben zárolják a root fiókot. A `sudo` vagy `su` parancsok használatával válnak ideiglenes adminisztrátorrá. A `sudo` egyetlen parancs futhat le emelt joggal, míg a `su` egy teljes sessiont nyit az `exit` parancsig.

- **Regular (Normál) felhasználói fiókok:** Mérsékelt jogosultságúak, mindennapi feladatokra használatosak, minden disztribúcióban léteznek.

- **Service (Szolgáltatás) fiókok:** A legkevesebb jogosultsággal rendelkeznek. A különféle háttérfolyamatok (szolgáltatások) futtatására szolgálnak, ezekkel nem lehet standard felhasználóként bejelentkezni. A szoftverek telepítése/eltávolítása automatikusan hozza létre vagy törli őket.

### Superuser vs. Sudo felhasználók összehasonlítása

- **Jogosultságok:** Megegyeznek (mindketten emelt jogú normál felhasználók).
- **Elérhetőség:** A superuser olyan disztribúciókon van, ahol a root nincs lezárva; a sudo user ott, ahol le van zárva.
- **Időtartam:** A superuser állandó (míg a `wheel` csoport tagja), a sudo user ideiglenes (parancsonkénti vagy `su` session idejére szól).

## 2. Superuser Hozzáférés Ellenőrzése és Használata

A superuser korlátlan hozzáféréssel bír: módosíthat bármilyen fájlt, futtathat bármilyen parancsot és megváltoztathat bármilyen rendszerkonfigurációt.

**Jelenlegi felhasználó ellenőrzése:**

```bash
whoami
id
```

**Váltás Superuser fiókra:**

A `su -` paranccsal közvetlenül átválthatunk superuser fiókra (a kötőjel jelzi a teljes környezet átvételét, jelszó megadása szükséges):

```bash
su -
```

## 3. Root Fiók Zárolása és Letiltása

A közvetlen root bejelentkezés megakadályozása jelentősen növeli a biztonságot.

### A Root fiók zárolásának ellenőrzése

- Az `/etc/passwd` fájl ellenőrzése:

  ```bash
  sudo cat /etc/passwd | grep root
  ```

  (Ha a kimenet `/sbin/nologin`-ra vagy `/usr/sbin/nologin`-ra végződik, a root fiók le van zárva).

- Az `/etc/shadow` fájl ellenőrzése:

  ```bash
  sudo cat /etc/shadow | grep root
  ```

  (Ha a második mezőben egy felkiáltójel (`!`) szerepel, a root zárolva van).

### Root fiók letiltása vagy feloldása a `passwd` paranccsal

- Zárolás (`-l` opció):

  ```bash
  sudo passwd -l root
  ```

- Feloldás (`-u` opció):

  ```bash
  sudo passwd -u root
  ```

### Root fiók letiltása a shell módosításával

A bejelentkezési shell átállítása `/sbin/nologin`-ra a `sed` paranccsal:

```bash
sudo sed -i 's_root:/bin/bash_root:/sbin/nologin_' /etc/passwd
```

## 4. Adminisztráció Delegálása (Szelektív Jogosultságok)

Ha nem akarunk teljes superuser hozzáférést adni egy felhasználónak, hanem csak egy-egy specifikus parancsot engedélyeznénk neki, a `visudo` paranccsal szerkeszthetjük a `/etc/sudoers` fájlt:

```bash
sudo visudo
```

**Formátum a sudoers fájlban:**

```plaintext
username ALL=(ALL) /path/to/command
```

Például, hogy a `bob` felhasználó futtathassa a `cat` parancsot a rendszeren:

```plaintext
bob ALL=(ALL) /usr/bin/cat
```

**Access Control List (ACL) használata:**

Speciális fájlok vagy könyvtárak elérésének finomhangolására a `setfacl` parancs használható.

## 5. Sudo Aliasok Használata

A sudo aliasok segítségével parancsokat, felhasználókat vagy futtatókat csoportosíthatunk a sudoers fájlban (`/etc/sudoers` vagy `/etc/sudoers.d/`).

**Alias típusok:**

- `Cmnd_Alias`: Parancsok csoportja.
- `User_Alias`: Felhasználók csoportosítása egyetlen név alatt.
- `Runas_Alias`: Alternatív nevek definiálása felhasználók számára.

**Példa Sudo Alias létrehozására és alkalmazására:**

1. Nyisd meg a sudoers fájlt:

   ```bash
   sudo visudo
   ```

2. Definiáld az aliasokat (pl. fájlszerkesztő parancsokhoz és adminokhoz):

   ```plaintext
   Cmnd_Alias FILE_EDIT = /usr/bin/vim /path/to/file, /usr/bin/nano /path/to/file
   User_Alias ADMINS = user1, user2, user3
   ```

3. Jogosultság megadása az aliasra (pl. a `Editors` csoportnak jelszó nélkül, `NOPASSWD:ALL` módon):

   ```plaintext
   Editors ALIAS_NAME=FILE_EDIT NOPASSWD:ALL
   ```

##Forrás
-[https://www.computernetworkingnotes.com/linux-tutorials/difference-between-the-root-user-and-super-sudo-user.html]
-[https://www.scaler.com/topics/super-user-in-linux/]
