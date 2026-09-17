# Linux Navigáció és Fájlkezelés – Összefoglaló Jegyzet

## 1. Elméleti alapok

- **A fájlrendszer felépítése:** A Linux fájlrendszere hierarchikus. A legfelső szintet **gyökérkönyvtárnak** (root directory) hívják, amelyet egyetlen perjel (`/`) jelöl. Minden más fájl és mappa ezen belül található.
- **Saját könyvtár (Home directory):** Bejelentkezéskor ide kerülsz (pl. `/home/felhasznalonev`). Ez az a hely a rendszerben, ahol teljes jogosultsággal rendelkezel a fájlok és könyvtárak felett.
- **Útvonalak:**
  - **Abszolút útvonal:** Mindig a gyökérkönyvtárból (`/`) indul, így egyértelműen meghatározza a helyet (pl. `/usr/share`).
  - **Relatív útvonal:** Az aktuális tartózkodási helyhez képest adja meg az útvonalat. Itt nem kell perjelet használni az elején (pl. `locale`).
- **Plain text:** A Linux és a Unix-szerű operációs rendszerek nagymértékben hagyatkoznak az egyszerű, formázatlan szöveges fájlokra (pl. konfigurációs fájlok esetén).

## 2. Navigáció és Felfedezés (Helymeghatározás)

### pwd (Print Working Directory)

Kiírja a könyvtár pontos (abszolút) útvonalát, ahol éppen tartózkodsz.

- **Példa:** `pwd` ➔ Eredmény: `/home/sammy`

### ls

Kilistázza az aktuális (vagy megadott) könyvtár tartalmát.

- **Példa:** `ls` ➔ Fájlok és mappák egyszerű nézete.
- **Példa:** `ls -l` ➔ Részletes nézet ("long"). Megmutatja a fájltípust, jogosultságokat, linkek számát, tulajdonost, csoportot, méretet és a módosítás idejét.
- **Példa:** `ls -a` ➔ Megmutatja az összes fájlt, beleértve a rejtett fájlokat is (amelyek neve ponttal kezdődik, pl. `.bashrc`). Itt látható a `.` (ami az aktuális könyvtárat jelöli) és a `..` (ami a szülőkönyvtárat jelöli).

### cd (Change Directory)

Könyvtárváltás.

- **Példa:** `cd /usr/share` ➔ Ugrás egy megadott abszolút útvonalra.
- **Példa:** `cd locale` ➔ Ugrás az aktuális mappán belül található `locale` mappába (relatív útvonal).
- **Példa:** `cd en/LC_MESSAGES` ➔ Több szint ugrása relatív útvonallal.
- **Példa:** `cd ..` ➔ Visszalépés a szülőkönyvtárba (egy szinttel feljebb).
- **Példa:** `cd` vagy `cd ~` ➔ Visszatérés a saját (home) könyvtáradba.

## 3. Fájlok Megtekintése

### less

Lapozó (pager) program, amellyel a hosszú szöveges fájlokat olvashatod el anélkül, hogy a tartalom elárasztaná a terminált.

- **Példa:** `less /etc/services`
- **Navigáció:** Fel/le nyilak a görgetéshez; Space vagy Page Down (illetve `CTRL-f`) a lapozáshoz lefelé; Page Up (illetve `CTRL-b`) a lapozáshoz felfelé.
- **Keresés:** `/keresőszó` (pl. `/mail`). A következő találathoz nyomd meg az `n` billentyűt, az előzőhöz a nagy `N`-t.
- **Kilépés:** `q` billentyű.

### cat

A fájl teljes tartalmát egyszerre kiírja a képernyőre, majd egyből visszaadja a parancssort.

### head

Csak a fájl első 10 sorát jeleníti meg.

### tail

Csak a fájl utolsó 10 sorát jeleníti meg.

## 4. Fájlok és Könyvtárak Manipulációja

### touch

Létrehoz egy üres fájlt a megadott névvel. Ha a fájl már létezik, akkor frissíti a módosítási idejét.

- **Példa:** `touch file1`
- **Példa:** `touch /home/sammy/file2 /home/sammy/file3` (Több fájl egyidejű létrehozása abszolút útvonallal)

### mkdir (Make Directory)

Üres könyvtárak létrehozása.

- **Példa:** `mkdir test`
- **Példa:** `mkdir -p some/other/directories` ➔ A `-p` kapcsoló lehetővé teszi egész könyvtárstruktúrák (egymásba ágyazott mappák) létrehozását egyetlen lépésben.

### mv (Move)

Fájlok és könyvtárak áthelyezése vagy átnevezése.

> **Vigyázat:** ha a célfájl már létezik, figyelmeztetés nélkül felülírja!

- **Áthelyezés:** `mv file1 test` (a `file1`-et áthelyezi a `test` mappába)
- **Visszahelyezés:** `mv test/file1 .` (a pont `.` használatával áthelyezi a fájlt a jelenlegi mappába)
- **Átnevezés:** `mv test testing` (a "test" mappát átnevezi "testing"-re)

### cp (Copy)

Fájlok és könyvtárak másolása.

> **Vigyázat:** a meglévő célfájlokat felülírja!

- **Fájl másolása:** `cp file3 file4`
- **Könyvtár másolása:** `cp -r some again` ➔ A `-r` (recursive) kapcsoló szükséges ahhoz, hogy a mappát és annak teljes tartalmát lemásolja. Ha a célmappa már létezik, a másolat abba kerül bele.

### rm (Remove)

Fájlok végleges törlése.

> **Vigyázat:** a Linux parancssorban nincs "Visszavonás/Undo" gomb vagy Kuka!

- **Példa:** `rm file4`

### rmdir

Kizárólag üres könyvtárak törlésére szolgál.

- **Példa:** `rmdir testing/example`

### rm -r

Nem üres könyvtárak és azok tartalmának rekurzív törlése.

- **Példa:** `rm -r again`

## 5. Fájlok Szerkesztése

### nano

Egy népszerű, kezdőbarát parancssori szövegszerkesztő, amely elfoglalja az egész terminált a szerkesztés idejére.

- **Fájl megnyitása/létrehozása:** `nano file1`
- **Kezelés:** A képernyő alján lévő parancsikonok segítenek. A `^` karakter a Ctrl gombot jelenti.
- **Fontosabb parancsok:**
  - **Súgó:** `Ctrl + G`
  - **Mentés (WriteOut):** `Ctrl + O` (Mentéskor az Enter megnyomásával kell jóváhagyni a fájl nevét)
  - **Kilépés:** `Ctrl + X` (Ha nem mentettél, a program rákérdez: `Y` = Igen, `N` = Nem, `Ctrl + C` = Mégsem).

### Egyéb szerkesztők

A szöveg megemlíti a `vim` vagy `vi` szerkesztőket is, amelyek nagyon erősek, de meredekebb a tanulási görbéjük. Kezdőként nyugodtan használható helyettük a `nano`.

## Forrás
- [Basic Linux Navigation and File Management – DigitalOcean](https://www.digitalocean.com/community/tutorials/basic-linux-navigation-and-file-management)

 # Terminál Alapok: Navigáció `cd`, `ls` és `pwd` parancsokkal - Összefoglaló Jegyzet

## 1. Elméleti alapok
* **Mi az a terminál?** A terminál (más néven parancssor vagy konzol) egy szöveges alapú felhasználói felület az operációs rendszerhez. Szöveg begépelésével adhatsz ki parancsokat, és a rendszer is szöveges formátumban adja vissza az eredményt.
* **Miért érdemes parancssort használni?** Bár a grafikus felhasználói felület (GUI) egyszerű és egyértelmű, nem nyújtja azt a sebességet és sokoldalúságot, mint a terminál. A terminálparancsokkal sokkal hatékonyabban végezhetők el a feladatok, és akár egyetlen sor kóddal is automatizálhatók az ismétlődő folyamatok.

---

## 2. A legfontosabb navigációs parancsok

### `pwd` (Print Working Directory)
* **Elmélet:** Kiírja a jelenlegi (munkakönyvtár) teljes útvonalát. Különösen hasznos, ha mélyen navigálsz a könyvtárszerkezetben, és tudni szeretnéd, pontosan hol tartózkodsz.
* **Használat:** 
  `pwd`
* **Példa kimenet:**
  `/home/username/Documents`

### `ls` (List)
* **Elmélet:** Kilistázza az aktuális (vagy a megadott) könyvtárban található fájlokat és mappákat. Alapértelmezés szerint nem jeleníti meg a rejtett fájlokat.
* **Használat:** 
  `ls [opciók] [könyvtár]`
* **Opciók és példák:**
  * `ls` ➔ Kilistázza az összes nem rejtett fájlt és mappát az aktuális könyvtárban.
  * `ls -l` ➔ Részletes nézet (long format). Megmutatja az egyes elemek részletes információit: jogosultságok, linkek száma, tulajdonos, csoport, méret és az utolsó módosítás ideje.
  * `ls -a` ➔ Az összes elemet kilistázza, **beleértve a rejtett fájlokat is** (amelyek neve ponttal kezdődik, pl. `.bashrc`).
* **Példa kimenet:**
  `Document.txt  Downloads  Images  Videos`

### `cd` (Change Directory)
* **Elmélet:** Könyvtárváltásra szolgál, ezzel navigálhatsz a fájlrendszer mappái között. Ha nem adsz meg utána paramétert, alapértelmezésben a saját (home) könyvtáradba visz.
* **Használat:** 
  `cd [könyvtár]`
* **Példák:**
  * `cd /path/to/directory/` ➔ Ugrás egy konkrét, megadott útvonalra.
  * `cd Downloads` ➔ Ugrás az aktuális mappán belüli *Downloads* könyvtárba.
  * `cd ..` ➔ Mozgás egy könyvtárszinttel feljebb (a szülőkönyvtárba).
  * `cd` ➔ Visszatérés a felhasználó saját (home) könyvtárába.

---

## 3. Hasznos tipp a hatékonysághoz
* **Aliasok (Rövidítések) használata:** Ha gyakran használsz hosszú vagy ismétlődő parancsokat, az `alias` parancs segítségével saját rövidítéseket hozhatsz létre. Ez jelentősen felgyorsítja a munkát és növeli a produktivitást a terminálban. A rendszeres gyakorlás pedig segít az "izommemória" felépítésében.

## Forrás
- [Basic Navigation Commands: cd, ls, and pwd – Linux Bash](https://linuxbash.sh/post/basic-navigation-commands-cd-ls-and-pwd)

# Linux Állományrendszer és Navigációs Alapok - Összefoglaló Jegyzet

## 1. Állományrendszer és Fájlrendszer Hierarchia (FHS)

**“/” Gyökérkönyvtár (Root):** Minden mappa és fájl ezen a szinten alatt helyezkedik el a hierarchiában.

### Fontosabb könyvtárak:
* **`/bin`**: A legfontosabb futtatható programok (boot folyamathoz, alapvető rendszeradminisztrációs feladatokhoz); pl. `ls`, `sh`, `rm`, `mv`, `mount` stb.
* **`/boot`**: Tömörített kernel image-ek (vmlinuz*) és a kernellel kapcsolatos segédfájlok.
* **`/dev`**: Hardverekhez tartozó fájlbejegyzések (eszközfájlok).
* **`/etc`**: Konfigurációs fájlok helye.
  * `/etc/init.d/`: A rendszer szolgáltatásait kezelő scriptek.
  * `/etc/fstab/`: Fájlrendszer csatolási beállításai.
* **`/home`**: E könyvtár alatt van minden normál felhasználónak egy saját, ún. home (saját) könyvtára (pl. `/home/jozsi`). Erre a saját könyvtárra a `~` karakterrel is lehet hivatkozni bash shell esetén.
* **`/lib`**: A `/bin` alatti programok által használt megosztott könyvtárak (shared library-k, kb. mint a DLL-ek) gyűjtőhelye.
* **`/mnt`**: Ideiglenes csatolási pontok; az itt lévő alkönyvtárakba szokás becsatolni ("mountolni") a különböző fájlrendszereket.
* **`/proc`**: Virtuális fájlrendszer; egyes fájlokból a rendszer aktuális állapota olvasható ki (pl. `/proc/cpuinfo`: cpu információ lekérdezése), másokba írva a kernel beállításait változtathatjuk meg (pl. `/proc/sys/net/ipv4/ip_forward`: IP forwarding engedélyezése vagy tiltása). Minden futó processznek van itt egy könyvtára a PID-je (azonosítója) alapján.
* **`/root`**: A rendszergazda (root felhasználó) home könyvtára. Technikai okokból célszerű külön helyre tenni (lehetővé téve a külön partíción tárolást).
* **`/sbin`**: Rendszergazda által használt binárisok; pl. `mke2fs`, `cfdisk`, `init`.
* **`/tmp`**: Ideiglenes (temp) fájlok helye; mindenki számára írható, tartalma időnként (beállítástól függően) törlődik.
* **`/usr`**: Unix System Resources. A programok saját könyvtárai, fájljai vannak itt. Alatta majdnem a teljes FHS le van duplikálva. Itt található pl. a `/usr/local` (a gépfüggő) és a `/usr/share` (a gépre nézve nem egyedi) fájloknak.
* **`/var`**: Gyakran változó tartalmú adatfájlok vannak itt; pl. naplófájlok (logok), mail spool, print spool.

---

## 2. Navigációs Alapfogalmak és Útvonalak

* **Saját könyvtár (Home Directory):** Amikor megnyitsz egy terminált, alapértelmezés szerint ide kerülsz. Bárhonnan visszatérhetsz ide a `cd` parancs kiadásával.
* **Abszolút útvonal (Absolute path):** Mindig a gyökérkönyvtárból (`/`) indul (pl. `/home/student`). Egyértelműen meghatározza a célt, függetlenül attól, hogy te épp hol tartózkodsz a fájlrendszerben.
* **Relatív útvonal (Relative path):** Az aktuális munkakönyvtárhoz képest adja meg az útvonalat, és **nem** kezdődik perjellel (`/`). Például, ha a `/usr` mappában vagy, a `cd bin` parancs kiadása a `/usr/bin` mappába fog vinni.
* **Szülőkönyvtár (`..`):** Az a könyvtár, amely egy szinttel feljebb található, és tartalmazza a jelenlegi munkakönyvtáradat. (Dupla pont).
* **Aktuális könyvtár (`.`):** Magát a jelenlegi könyvtárat jelöli. (Szimpla pont).

---

## 3. Alapvető Parancsok és Példák a Navigációhoz

# Linux fájlrendszer-navigáció alapjai – lépésről lépésre

Az alábbi utasítások az abszolút minimum, amelyet egy kezdő GNU/Linux felhasználónak el kell sajátítania ahhoz, hogy a GNU/Linux parancssorban a legegyszerűbb feladatokat is el tudja végezni. Miután megtanultad az alábbi alapokat, készen állsz arra, hogy haladóbb parancssori témák felé mozdulj.

Amikor egy shell terminálban dolgozol, mindig egy adott könyvtárban tevékenykedsz. Annak meghatározásához, hogy melyik könyvtárban vagy, használd a `pwd` parancsot:

    student@linuxconfig:$ pwd
    /usr/local/bin
    student@linuxconfig:$ cd
    student@linuxconfig:$ pwd
    /home/student
    student@linuxconfig:$

A home könyvtárad az a könyvtár, amelyben a terminál első megnyitásakor vagy. Ahhoz, hogy bárhonnan a home könyvtáradba menj, csak írd be a `cd` parancsot:

    student@linuxconfig:$ pwd
    /usr/local/bin
    student@linuxconfig:$ cd
    student@linuxconfig:$ pwd
    /home/student
    student@linuxconfig:$

Az abszolút útvonalnév olyan, amely a `/` karakterrel kezdődik, ami a fájlrendszer fa gyökerét jelöli. Ezért a home könyvtáradba jutás másik módja:

    student@linuxconfig:/etc$ cd /home/student
    student@linuxconfig:$ pwd
    /home/student
    student@linuxconfig:$

A relatív és abszolút útvonalakkal kapcsolatos további információkért látogasd meg a bash scripting tutorialunkat. Ha új vagy a bash scriptingben, az útvonalak megértése alapvető fogalom.

A relatív útvonal olyan, amely az aktuális könyvtárhoz kapcsolódó könyvtár nevével kezdődik. Például, ha a `/usr` könyvtárban vagy, akkor csak a `cd bin` beírása (anélkül, hogy a „bin” elé `/`-t tennél) a következő hatással van:

    student@linuxconfig:$ pwd
    /usr
    student@linuxconfig:$ cd bin
    student@linuxconfig:$ pwd
    /usr/bin
    student@linuxconfig:$

és a `/usr/bin` könyvtárba mész, nem pedig a `/usr/local/bin` vagy a `/bin` könyvtárba.

Ahhoz, hogy az aktuális munkakönyvtárat tartalmazó könyvtárba (más néven szülőkönyvtárba) menj, írd be:

    student@linuxconfig:$ pwd
    /usr/bin
    student@linuxconfig:$ cd ..
    student@linuxconfig:$ pwd
    /usr
    student@linuxconfig:$

Az aktuális munkakönyvtár relatív útvonalnevét `.`-nak (pont) nevezzük. Ezért ha beírod:

    student@linuxconfig:$ pwd
    /usr/bin
    student@linuxconfig:$ cd .
    student@linuxconfig:$ pwd
    /usr/bin
    student@linuxconfig:$

az nem változtatja meg az aktuális munkakönyvtárat.

## Forrás
- [Filesystem Basics – LinuxConfig](https://linuxconfig.org/filesystem-basics)
