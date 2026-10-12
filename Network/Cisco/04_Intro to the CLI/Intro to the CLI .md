# 4. Lecke: A Cisco IOS CLI Alapjai

Üdvözöllek a Jeremy's IT Lab-ben! Ez egy teljes, 100%-ban ingyenes CCNA felkészítő kurzus, amely mindent tartalmaz a vizsga sikeres letételéhez. Ebben a negyedik leckében végre elkezdjük a Cisco eszközök tényleges konfigurálását.

A téma a Cisco IOS CLI. A Cisco IOS (Internetwork Operating System) a Cisco eszközök operációs rendszere (olyan, mint a Windows egy PC-n, vagy a macOS egy Mac-en). Fontos megjegyezni: a Cisco IOS-nek semmi köze az Apple iPhone-jain futó iOS-hez!

## Mi az a CLI?

A CLI a **Command-Line Interface** (Parancssoros felület) rövidítése. Ezt a felületet használjuk a Cisco eszközök (routerek, switchek, tűzfalak) konfigurálására.

Létezik grafikus felület is, a GUI (Graphical User Interface), amelyet például a Cisco tűzfalak (ASDM) konfigurálásához lehet használni. Ebben a kurzusban azonban nem foglalkozunk a GUI-val, mivel a legtöbb hálózati mérnök sokkal jobban szeret CLI-ből dolgozni.

## Hogyan csatlakozzunk egy Cisco eszközhöz?

Bár léteznek távoli (remote) csatlakozási módok is, amikor legelőször állítasz be egy eszközt, fizikailag kell csatlakoznod a console port-hoz (konzol porthoz). Ez általában úgy történik, hogy odaviszed a laptopodat az eszközhöz.

Egy tipikus Cisco (pl. Catalyst) switchen kétféle konzol port is lehet:

- Egy hagyományos RJ45 port (hasonlít a normál hálózati portokhoz).
- Egy USB mini-b csatlakozó.

<img width="382" height="523" alt="consol" src="https://github.com/user-attachments/assets/104f8a47-1c41-4f85-92d4-543ca709776e" />

Ha az RJ45 portot választod, egy speciális kábelre lesz szükséged, az úgynevezett **rollover cable**-re (gördülő kábel).

- Az egyik végén RJ45 csatlakozó van (ezt dugod a Cisco eszközbe).
- A másik végén egy DB9 (soros) csatlakozó található.

Mivel a mai modern laptopokon már szinte soha nincs DB9-es soros port, szükséged lesz egy **USB - Soros adapterre** is a csatlakoztatáshoz.

<img width="2048" height="1360" alt="licensed-image" src="https://github.com/user-attachments/assets/8156619b-50a9-414f-afb7-ea9e8f6c6d1d" />

> Miért "rollover"? A rollover kábelben is 8 pin (érintkező) van mindkét oldalon, de a bekötés pontosan meg van fordítva: az 1-es pin a 8-asba, a 2-es a 7-esbe, a 3-as a 6-osba megy, és így tovább egészen a 8-as pinig, ami az 1-esbe fut.

## Terminal Emulator (PuTTY) és a Soros Beállítások

Miután fizikailag összekötötted a gépeket, szükséged lesz egy szoftverre, egy **Terminal emulator**-ra (Terminál emulátor). Az egyik legnépszerűbb és leggyakoribb választás erre a **PuTTY** (letölthető a putty.org-ról).

A PuTTY-ban a **"Serial"** (Soros) kapcsolatot kell kiválasztani. A Cisco eszközök alapértelmezett soros beállításai, amelyeket a vizsgára is érdemes megjegyezni (bár ritkán kell módosítani őket, mert ez a standard):

| Beállítás | Érték |
|---|---|
| Speed (Baud rate - Sebesség) | 9600 bits per second (bps) |
| Data bits (Adatbitek) | 8 |
| Stop bits (Stopbitek) | 1 |
| Parity (Paritás a hibajavításhoz) | None (Nincs) |
| Flow control (Folyamatszabályozás) | None (Nincs) |

<img width="323" height="309" alt="images" src="https://github.com/user-attachments/assets/e808e791-4bf9-4a7e-93d5-8a16d1f0f046" />

## A Cisco IOS Parancssori Módjai (CLI Modes)

Amikor először belépsz (és nemet mondasz az initial configuration dialog-ra), egy fokozatosan bővülő jogosultsági rendszerrel találkozol.

Minden sor elején látható az eszköz neve (hostname). Alapértelmezés szerint ez a név a routereknél: `Router`.

### 1. User EXEC Mode (Felhasználói mód)

Amikor először belépsz, alapértelmezés szerint ebben a módban találod magad.

- **Jele:** `>` (kacsacsőr)
- Ezt a módot nagyon korlátozott jogok jellemzik. Csak néhány dolgot lehet megnézni, de semmilyen beállítást nem lehet megváltoztatni.

```text
Router>
```

### 2. Privileged EXEC Mode (Kiváltságos mód)

Ahhoz, hogy több jogod legyen (pl. az eszköz újraindítása, konfigurációk megtekintése, óra átállítása, fájlok mentése), ebbe a módba kell lépned. Ezt az `enable` paranccsal teheted meg. (A beállításokat még itt sem tudod érdemben megváltoztatni).

- **Jele:** `#` (hash / hashtag / font jel)
- **Parancs a belépéshez:**

```text
Router> enable
Router#
```

### 3. Global Configuration Mode (Globális konfigurációs mód)

Ha ténylegesen módosítani szeretnéd a router vagy a switch beállításait, ide kell belépned a Privileged EXEC módból. Ezt a `configure terminal` paranccsal teheted meg.

- **Jele:** `(config)#` (a név után beszúrva)
- **Parancs a belépéshez:**

```text
Router# configure terminal
Router(config)#
```

## CLI Trükkök és Gyorsítók (Shortcuts)

A Cisco CLI nagyon felhasználóbarát, ha ismered a trükköket:

- **Tabulátor (Tab) gomb:** Ha elkezdesz gépelni egy parancsot (pl. `en`), és megnyomod a Tab-ot, a rendszer automatikusan kiegészíti a szót (`enable`), feltéve, hogy nincs más parancs, ami ugyanígy kezdődik.
- **Rövidítések (Shortcuts):** Nem is kell kiírnod a teljes szót! Elég csak annyit beírnod, amennyiből a rendszer már egyértelműen azonosítani tudja a parancsot.
  - Az `enable` legrövidebb formája: `en`
  - A `configure terminal` legrövidebb formája: `conf t`
  - Az `exit` legrövidebb formája: `ex`
  - (Jeremy innentől kezdve gyakran a `conf t` rövidítést fogja használni a laborokban).
- **Kérdőjel (?):** Ez a legjobb barátod!
  - Ha nem tudod, milyen parancsok érhetők el, csak üss egy `?`-et.
  - Ha elkezdesz egy szót (pl. `e?`), kilistázza az "e"-vel kezdődő parancsokat (pl. `enable`, `exit`).
  - Ha egy teljes parancs után teszel szóközt és `?`-et (pl. `enable password ?`), megmutatja, milyen paramétert vár a rendszer a folytatásban.

## Jelszavak Beállítása és Titkosítása

Nem akarjuk, hogy bárki, aki hozzáfér az eszközhöz, át tudjon lépni Privileged EXEC módba, és átnézze vagy módosítsa a beállításokat. Ezért jelszót kell beállítani hozzá. A jelszavak **kis- és nagybetű érzékenyek** (case-sensitive)! (Ha elírod a jelszót 3-szor, a rendszer "bad secrets" hibaüzenettel kidob).

### 1. enable password (Sima jelszó)

Ez a parancs beállít egy jelszót (pl. `CCNA`), amit az `enable` beírásakor fog kérni a rendszer. (Gépelés közben biztonsági okokból nem látszanak a karakterek).

```text
Router(config)# enable password CCNA
```

**Probléma:** Ha megnézzük a konfigurációs fájlt, ez a jelszó tiszta szövegként (plain text) olvasható. Ez óriási biztonsági kockázat.

### 2. service password-encryption (Gyenge titkosítás)

Ez a parancs titkosítja a konfigurációs fájlban szereplő összes jelenlegi és jövőbeli jelszót egy jelszó-karaktersorozattá (pl. `08026F6028`).

```text
Router(config)# service password-encryption
```

Ha ezt használod, a jelszó előtt egy **7-es szám** jelenik meg (Type 7). Ez a Cisco saját, de sajnos nagyon gyenge titkosítási algoritmusa. A weben található "Cisco Type 7 cracker" oldalak pillanatok alatt visszafejtik.

### 3. enable secret (A legbiztonságosabb módszer)

Ez a parancs sokkal erősebb jelszót állít be (pl. `Cisco`).

```text
Router(config)# enable secret Cisco
```

A konfigurációs fájlban a jelszó előtt egy **5-ös szám** jelenik meg (Type 5). Ez MD5 titkosítást jelent, ami nagyságrendekkel biztonságosabb a 7-esnél.

**Fontos szabályok az enable secret-tel kapcsolatban:**

- Soha nem jelenik meg tiszta szövegként, mindig titkosítva van (akkor is, ha a `service password-encryption` ki van kapcsolva).
- Ha mind az `enable password`, mind az `enable secret` be van állítva, a rendszer **csak az `enable secret`-et fogja figyelembe venni**, az `enable password` figyelmen kívül lesz hagyva. (A gyakorlatban mindig a secret-et használd!).

## A Konfigurációs Fájlok és a Mentés

A Cisco eszközökön egyszerre két konfigurációs fájl is jelen van:

### running-config (Futó konfiguráció)

Ez az aktuális, aktív konfiguráció. Ahogy beírsz egy parancsot a CLI-be, az azonnal módosítja ezt a fájlt. (Ez a memóriában, RAM-ban fut).

Megtekintése (Privileged EXEC módból):

```text
Router# show running-config
```

### startup-config (Indító konfiguráció)

Ezt a fájlt tölti be az eszköz, amikor újraindul (reload) vagy bekapcsolják (Ez az NVRAM-ban, nem felejtő memóriában tárolódik).

Megtekintése:

```text
Router# show startup-config
```

> Ha bent vagy a Global Config módban, és ki akarsz adni egy `show` parancsot, nem kell kilépned. Csak írd elé, hogy `do`! Például: `Router(config)# do show running-config` vagy `Router(config)# do sh run`.

### Hogyan mentsük a konfigurációt?

Ahhoz, hogy az elvégzett beállítások újraindítás után is megmaradjanak, a `running-config`-ot el kell menteni a `startup-config`-ba. Erre három, egyenértékű parancs is létezik (mindhármat Privileged EXEC módból kell kiadni):

```text
Router# write
Router# write memory
Router# copy running-config startup-config
```

## Parancsok törlése / visszavonása (no)

Ha egy korábban beállított parancsot vissza akarsz vonni (vagy törölni akarsz valamit), egyszerűen írd elé, hogy `no`.

Például:

```text
Router(config)# no service password-encryption
```

> **Megjegyzés:** Ha kikapcsolod a jelszótitkosítást, a már titkosított jelszavak nem fejtődnek vissza, azok úgy maradnak. Csak az újonnan beírt jelszavak lesznek ismét tiszta szövegesek.

## Kvíz (Tudáspróba)

**1. Milyen kábelt használunk ahhoz, hogy csatlakozzunk egy Cisco eszköz RJ45-ös konzol portjához?**

> **Helyes válasz:** Rollover kábel. (A crossover kábel két azonos hálózati eszköz összekötésére való, az USB kábel pedig az USB konzol porthoz kell, nem az RJ45-öshöz).

**2. Beírod az `enable` parancsot a Privileged EXEC módba lépéshez, de a megadott jelszót nem fogadja el a gép. Mi lehet a probléma?**

> **Helyes válasz:** Be van kapcsolva a Caps Lock. (A jelszavak kis- és nagybetű érzékenyek! A `service password-encryption` állapota nem számít, mert az csak a mentett szöveg megjelenését befolyásolja, magát a jelszót nem).

**3. Mi a legbiztonságosabb módszer a Privileged EXEC mód jelszavas védelmére?**

> **Helyes válasz:** Az `enable secret` parancs. (Ez sokkal erősebb, MD5 / Type 5 szintű titkosítást használ, mint az `enable password` még akkor is, ha ez utóbbi mellé be van kapcsolva a `service password-encryption`).

**4. Mi történik az `enable` parancs kiadásakor, ha az `enable password` és az `enable secret` is be van állítva a routeren?**

> **Helyes válasz:** Csak az `enable secret` jelszavát kell megadni. (A titkos (secret) jelszó mindig felülbírálja és érvényteleníti a sima jelszót, soha nem kéri be a rendszer mindkettőt).

**5. A `conf t` paranccsal lépsz be a globális konfigurációs módba. Mi ennek a parancsnak a teljes hossza?**

> **Helyes válasz:** `configure terminal`.

<img width="817" height="802" alt="en" src="https://github.com/user-attachments/assets/8212701c-bb5d-4f9d-8564-7b3128074ac0" />

<img width="827" height="840" alt="priv" src="https://github.com/user-attachments/assets/fb942c08-a71f-48ab-9551-874c159a78ea" />

<img width="805" height="850" alt="config" src="https://github.com/user-attachments/assets/e1464e5f-20e3-432d-9fd5-2aaa2863708b" />

<img width="788" height="842" alt="interfaces pmg" src="https://github.com/user-attachments/assets/47e9bff3-468a-4e34-adbb-05d443a70cb5" />



