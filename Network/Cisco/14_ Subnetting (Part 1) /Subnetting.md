# CCNA Bővített Jegyzet – 13. Lecke: Subnetting (Alhálózatok képzése) 1. Rész

Ebben a leckében a hálózatépítés egyik legfontosabb, sokak számára ijesztőnek tűnő, de logikusan felépítve nagyon is egyszerű témáját, az alhálózatok képzését (Subnetting) kezdjük el tárgyalni. A Subnetting lehetővé teszi, hogy a meglévő IP-cím tartományainkkal okosan, pazarlás nélkül gazdálkodjunk.

## 1. A Probléma: Az IPv4 Címosztályok (Classful Addressing) és a Pazarlás

Hogy megértsük, miért van szükség az alhálózatokra, először meg kell vizsgálnunk, hogyan osztották ki régen az IP-címeket, és mi volt ezzel a baj.

A világon az IP-címeket egy **IANA (Internet Assigned Numbers Authority)** nevű szervezet osztja ki a vállalatoknak. Régen ezt szigorúan az "Osztályok" (A, B, C) alapján tették:

- **Class A (/8):** `0.0.0.0` - `127.255.255.255`. Hálózatonként ~16,7 millió kiosztható címet tartalmaz. (24 bit jut a hostokra).
- **Class B (/16):** `128.0.0.0` - `191.255.255.255`. Hálózatonként 65 536 kiosztható címet tartalmaz. (16 bit jut a hostokra).
- **Class C (/24):** `192.0.0.0` - `223.255.255.255`. Hálózatonként 256 kiosztható címet tartalmaz. (8 bit jut a hostokra).

(A Class D multicast, a Class E kísérleti célokra van fenntartva).

### Mi ezzel a baj? (A pazarlás)

Képzeld el, hogy a cégednek van egy **Point-to-Point** (Pont-tól pont-ig) hálózata, ami csupán két routert (R1 és R2) köt össze. Ide mindössze **2 darab IP-címre** van szükség.

Ha a régi rendszerben a legkisebb hálózatot, egy Class C hálózatot (pl. `203.0.113.0/24`) igényelsz erre a célra, a matek a következő:

- Kapsz 256 címet.
- Levonod a Hálózat címet (1) és a Broadcast címet (1).
- Kiosztasz egy címet R1-nek és egyet R2-nek (2).
- **Eredmény:** Elpazaroltál **252 darab IP-címet**, amit senki más nem használhat!

**Egy másik példa:** Egy cégnek 5000 IP-címre van szüksége. A Class C (254 gép) túl kicsi, így az IANA kénytelen adni nekik egy Class B hálózatot. Ezzel a cég kap 65 534 címet, amiből felhasznál 5000-et, és a maradék ~60 000 cím a kukában landol. Mivel az IPv4 címek (kb. 4 milliárd darab) már elfogytak, ez a rendszer fenntarthatatlan volt.

## 2. A Megoldás: CIDR (Classless Inter-Domain Routing)

1993-ban az IETF bevezette a **CIDR**-t (Osztály nélküli útválasztást), ami eltörölte a szigorú `/8`, `/16` és `/24` határokat.

A CIDR lehetővé teszi a hálózati mérnökök számára, hogy egy nagy hálózatot kisebb, egyedi méretű alhálózatokra (subnets) vágjanak szét, így pontosan akkora hálózatot hozhatnak létre, amekkorára szükség van.

A CIDR bevezette az úgynevezett peres (slash) jelölést (pl. `/26`, `/30`), amit ma "CIDR jelölésnek" (CIDR notation) hívunk. Ez felváltotta a hosszú, pontozott decimális maszkok (pl. `255.255.255.0`) állandó leírását, bár a rendszerek a háttérben továbbra is a maszkokat használják.

## 3. Subnetting a Gyakorlatban (Példa a Point-to-Point hálózatra)

Nézzük meg, hogyan tehetjük hatékonnyá azt az R1 és R2 közötti kapcsolatot, amihez csak 2 darab kiosztható cím (plusz a hálózat és a broadcast cím, tehát összesen 4 cím) kell!

Az alap hálózatunk a `203.0.113.0/24`. Kezdjük el mozgatni a "vonalat" jobbra, azaz vegyünk el biteket a Host (Gép) résztől, és adjuk hozzá a Hálózat (Network) részhez!

### Az alhálózatok méretének kiszámítása ($2^N - 2$ szabály)

- **/25 (Maszk: `255.255.255.128`):** Marad 7 host bit. $2^7 - 2 = 126$ kiosztható cím. (Még mindig 124 elvesztegetett cím).
- **/26 (Maszk: `255.255.255.192`):** Marad 6 host bit. $2^6 - 2 = 62$ kiosztható cím.
- **/27 (Maszk: `255.255.255.224`):** Marad 5 host bit. $2^5 - 2 = 30$ kiosztható cím.
- **/28 (Maszk: `255.255.255.240`):** Marad 4 host bit. $2^4 - 2 = 14$ kiosztható cím.
- **/29 (Maszk: `255.255.255.248`):** Marad 3 host bit. $2^3 - 2 = 6$ kiosztható cím. (Még mindig 4 megy a kukába).
- **/30 (Maszk: `255.255.255.252`):** Marad 2 host bit. $2^2 - 2 = 2$ kiosztható cím! Ez a tökéletes méret!

A `/30`-as alhálózat (Subnet) kiosztása pontosan 4 címet emészt fel:

- `.0` (Hálózat cím)
- `.1` (R1 IP-címe)
- `.2` (R2 IP-címe)
- `.3` (Broadcast cím)

**A Varázslat:** Ezzel a módszerrel a teljes `203.0.113.0/24` hálózatból csak a `.0` - `.3` közötti tartományt használtuk el. A maradék címeket (`.4`-től egészen `.255`-ig) most már felhasználhatjuk a hálózat más részein! Nem pazaroltunk el 252 címet.

## 4. A Különleges Kivételek: A /31 és a /32

### A /31 (Maszk: `255.255.255.254`)

Ha a matematikát nézzük, itt csak 1 host bit marad. $2^1 - 2 = 0$ kiosztható cím. Emiatt régen tilos volt használni.

**DE:** A modern routereken (pl. Ciscónál) egy kizárólag routereket összekötő Point-to-Point kapcsolaton megengedték a szabály áthágását. Mivel ezen a két végponton kívül senki más nincs a kábelen, nincs szükség Hálózat és Broadcast címre.

Így a `.0` és a `.1` címet egyenesen ki lehet osztani az R1-nek és az R2-nek! Ez a leghatékonyabb megoldás, mert egyetlen címet sem pazarol el, a Cisco routerek csak egy apró figyelmeztetést dobnak rá.

### A /32 (Maszk: `255.255.255.255`)

Itt 0 host bit marad, a formula szerint $-1$ kiosztható címet kapunk.

Ezt soha nem használjuk egy valódi hálózat/interfész konfigurálására (hiszen nincs benne mozgástér). Ezt kizárólag akkor alkalmazzuk, ha a routing táblában egy konkrét, egyetlen számítógépre/hostra akarunk rámutatni (ahogy ezt a statikus útvonalaknál a "Local" route esetében láttuk).

## 5. Szituációs Feladat és Kvíz (A következő lecke felvezetése)

### A Probléma

Kaptál egy `192.168.1.0/24` méretű (Class C, 256 címet tartalmazó) hálózatot. Ezt el kell osztanod **4 darab egyenlő méretű alhálózatra**. Minden alhálózatban van **45 darab számítógép**, és minden hálózathoz csatlakozik az R1-es router egyik portja (ami szintén IP-t igényel).

### 1. Lépés: A követelmények felmérése

- Hálózatonként kell: 45 (PC) + 1 (Router) = **46 kiosztható cím**.
- Ehhez hozzáadjuk a hálózat és a broadcast címet (+2), így alhálózatonként **48 darab IP-cím** "helyre" van szükségünk.
- Mivel 4 hálózatunk van: $48 \times 4 = 192$ cím. A rendelkezésünkre álló 256 cím bőségesen elég lesz.

### 2. Lépés: A megfelelő Prefix / Alhálózat (Subnet) kiválasztása

Keresünk egy olyan prefixet, ami legalább 46 kiosztható gépet elbír:

- **/28** ($2^4 - 2 = 14$ gép) → Túl kicsi.
- **/27** ($2^5 - 2 = 30$ gép) → Túl kicsi.
- **/26** ($2^6 - 2 = 62$ gép) → **Ez a nyertes!** A `/26`-os maszk (`255.255.255.192`) hálózatonként 64 címet foglal le (62 gép + net + broadcast), ami tökéletesen befogadja a kért 46 gépet. A megmaradt extra címek tartalékok lesznek a jövőbeli bővítéshez.

### Kvíz Kérdés

Ha tudjuk, hogy `/26`-os alhálózatokat (subnets) használunk, és az első alhálózat (Subnet 1) a `192.168.1.0/26`. Mik lesznek a maradék alhálózatok (Subnet 2, 3 és 4) hálózati címei?

**(Megoldás és Magyarázat, amelyet a jegyzet kiegészítéseként a logika alapján levezetünk):**

Mivel minden `/26`-os hálózat pontosan 64 címet eszik meg:

- **Subnet 1:** Kezdődik a `192.168.1.0/26`-tól, a broadcast címe a `.63` lesz.
- **Subnet 2:** Mivel az előző `.63`-nál végződött, a következő hálózat a `192.168.1.64/26`-tól indul, és a `.127`-nél ér véget.
- **Subnet 3:** Mivel az előző `.127`-nél végződött, a harmadik a `192.168.1.128/26`-tól indul, és a `.191`-nél ér véget.
- **Subnet 4:** Mivel az előző `.191`-nél végződött, az utolsó a `192.168.1.192/26`-tól indul, és a `.255`-tel ér véget (ami az egész `/24`-es blokkunk vége is egyben).
