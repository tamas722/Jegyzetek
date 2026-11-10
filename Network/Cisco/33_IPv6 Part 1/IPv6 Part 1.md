# CCNA Bővített Jegyzet – 33. Lecke: Az IPv6 Alapjai és Címzése

Ebben a leckében áttérünk a hálózatok jövőjére, az **IPv6**-ra (Internet Protocol version 6). Az IPv4-et nagyjából 30 éve tervezték, és mára végérvényesen kifogytunk az IP-címekből. (Az IPv5 nevű "Internet Stream Protocol" kísérleti projekt volt, amely sosem terjedt el széles körben, a fejlécben használt "5" miatt nevezték el a mostanit IPv6-nak). Az IPv6 nem csak több címet ad, de a felépítése, az ábrázolása (hexadecimális) és a rövidítési szabályai is teljesen újak.

## 1. Miért kell az IPv6? (Az IPv4 kimerülése)

Az IPv4 címek $2^{32}$, azaz körülbelül **4,3 milliárd** egyedi címet biztosítanak. A modern világban az IoT eszközök, okostelefonok, okostévék és a folyamatos hálózatbővülés miatt ez már nagyon régen elfogyott.

Bár a VLSM (változó hosszúságú alhálózati maszkok), a Privát IP-címek (pl. a `192.168.X.X` tartomány) és a NAT (Network Address Translation - Hálózati Címfordítás) ideig-óráig megmentették és kitolták a rendszert, ezek csak átmeneti megoldások.

Az IP-címek kiosztásáért felelős **IANA**, és az alá tartozó **RIR-ek** (Regional Internet Registries, mint pl. Európában a RIPE NCC, Amerikában az ARIN) lényegében teljesen kiürítették az IPv4 készleteiket.

### Az IPv6 Mérete

Az IPv6 cím **128 bites**. Ez nem négyszerese az IPv4-nek, hanem $2^{128}$, azaz körülbelül **340 Szextillió (Undecillion)** címet jelent. Ez olyan hatalmas szám, hogy gyakorlatilag sosem fog elfogyni.

## 2. A Hexadecimális (16-os Számrendszer) Áttekintése

Mivel egy 128 bites sorozatot binárisan (1 és 0) lehetetlen lenne emberként elolvasni, és még decimálisan is túl hosszú lenne, az IPv6 címeket **Hexadecimális** (16-os számrendszerű) formátumban írjuk. Ezt hálózatépítésben gyakran a `0x` előtaggal jelölik.

**A Hexa Számjegyek:** `0 1 2 3 4 5 6 7 8 9 A B C D E F` (Ahol A=10, B=11, C=12, D=13, E=14, F=15).

> **A legfontosabb szabály:** Minden egyes Hexadecimális számjegy pontosan **4 bináris bitet** (egy Nibble-t) képvisel! Így a maximális 4 bites bináris szám (`1111`) pontosan a legnagyobb hexa számjegyet (`F`, azaz 15) adja.

### Kézi Átváltás Binárisból Hexába

Ha megkapod az `1101 1011` bináris számot, oszd fel 4 bites csoportokra:

- `1101` → (8+4+1) → 13 → **D**
- `1011` → (8+2+1) → 11 → **B**

**Eredmény: DB**

## 3. Az IPv6 Címek Felépítése és Rövidítési Szabályai

Mivel 128 bitből áll, és egy hexa számjegy 4 bitet ér, az IPv6 cím pontosan **32 darab Hexadecimális karakterből** áll. Ezt 8 darab, kettősponttal (`:`) elválasztott **"Kvartettre"** (Quartet/Hextet) osztják, ahol minden kvartettben 4 hexa karakter található.

Példa egy teljes, íratlan címre: `2001:0db8:0000:0000:0000:0000:0000:0001`

Hogy ezt ne kelljen mindig kiírni, két kritikus rövidítési (abbreviation) szabályt kell a vizsgára álmádból is tudni!

### 1. Szabály: A Vezető Nullák (Leading Zeros) Elhagyása

Egy kvartetten belül a legelső, bal oldalon lévő nullákat (Vezető Nullák) kihúzhatod!

- `0db8` → `db8`
- `0001` → `1`
- `0000` → `0` (Legalább egy 0-t meg kell hagyni, ha az egész kvartett nulla volt).

> **(Kvíz Csapda!** A kvartett végén lévő "záró" nullákat **TILOS** kihúzni! A `1000`-ből **NEM** lehet `1`-et csinálni, mert az `0001`-et jelentene!).

### 2. Szabály: A Kettős Kettőspont (Double Colon `::`)

Ha van egymás mellett kettő vagy annál több teljes egészében nullákból álló kvartetted (pl. `0:0:0:0`), azt az egész blokkot lecserélheted egyetlen `::` (dupla kettőspont) jelre!

Példa: `2001:db8:0:0:0:0:0:1` → `2001:db8::1`

> **A LEGSZIGORÚBB SZABÁLY:** A Dupla Kettőspontot (`::`) egy IP-címen belül **CSAK ÉS KIZÁRÓLAG EGYSZER** szabad használni!
>
> **Miért?** Mert ha a címben két helyen is lenne `::` (pl. `2001::abcd::1`), a router nem tudná kitalálni, hogy az összesen hiányzó mondjuk 5 darab nullás kvartettből hány darab való az első `::` helyére, és hány darab a második helyére. (Ilyenkor a hosszabb nullás sort szoktuk `::`-tal rövidíteni, a rövidebbnél meg csak meghagyjuk az `0:0`-t).

## 4. Az IPv6 Prefix (Hálózat Cím) és a Host Rész Megkeresése

Az IPv6 teljesen szakít az osztályalapú (Classful) IP ábrázolással és a pontozott decimális (`255.255.255.0`) alhálózati maszkokkal. Kizárólag **CIDR Prefix** formátumot használ (pl. `/64`).

Az Internetszolgáltatók (ISP-k) a vállalatoknak általában egy **/48**-as blokkot (Global Routing Prefix) osztanak ki.

Az IPv6 hivatalos, aranyszabálya szerint a kliensek/Hostok felé néző alhálózatok MASZKJA **MINDIG /64!** Ez azt jelenti, hogy a cím 128 bitjéből pontosan a fele (a bal oldali 64 bit, azaz az első 4 kvartett) a Hálózat azonosítója, a másik 64 bit (a jobb oldali 4 kvartett) a Gép azonosítója (Host ID). Így a cégek a /48-as blokkjuk és a /64-es határ között kapnak 16 bitnyi szabad területet (Subnet Identifier), amivel a saját belső alhálózataikat építhetik ki.

### A Prefix Keresése

Ha megkapod a `2001:db8:1234:5678:abcd:ef12:3456:7890 /64` címet, mi a Hálózat címe?

Mivel a /64 pont a felénél van, egyszerűen csak lenullázod a jobb oldali 64 bitet:
`2001:db8:1234:5678:0000:0000:0000:0000` ami röviden: `2001:db8:1234:5678::/64`. (A Host bitek lenullázása = a hálózat címe).

### Trükkösebb esetek (Ahol kell a bináris tudás!)

Ha a maszk `/93`! Mivel a 93 nem osztható 4-gyel (1 hexa karakter = 4 bit), a maszk belevág egy hexa karakter közepébe.

Keresd meg, hol a 93. bit! A 6. kvartett 80. bitnél ér véget (mert $5 \cdot 16 = 80$). A 6. kvartett: (84, 88, 92, 96). A `/93` tehát a 6. kvartett első és második hexa karaktere közé esik, pontosabban a második karakter 1. bitje még a hálózathoz tartozik, a többi 3 bitje már Host bit!

(Ha a karakter egy `B` (`1011`), és az utolsó 3 bitjét le kell nullázni (mert Host bitek), akkor `1000` lesz belőle, azaz `8`! A hálózat címe tehát ott már 8-cal fog folytatódni a B helyett!).

## 5. Cisco IOS Alapkonfiguráció

Az IPv6 parancsok kísértetiesen hasonlítanak az IPv4-re, csak az `ip` szót kell `ipv6`-ra cserélni.

> (Megjegyzés: A Cisco hivatalos oktatási anyagokban a példacímek mindig a `2001:db8::` dokumentációs tartományba esnek).

### A legfontosabb parancs: Az IPv6 Útválasztás Engedélyezése!

Alapértelmezésben a Cisco routerek IPv4-re vannak hangolva. Ha csak adsz egy IPv6 címet a portoknak, a router kommunikál velük lokálisan, de nem fogja őket routolni (továbbítani) a hálózatok között! Ezt globálisan, manuálisan kell bekapcsolni:

```text
Router(config)# ipv6 unicast-routing
```

### Címadás a portoknak

```text
Router(config)# interface g0/0
Router(config-if)# ipv6 address 2001:db8:0:0::/64
Router(config-if)# no shutdown
```

(Az IOS megérti a teljesen kifejtett, és a dupla kettősponttal rövidített alakot is).

### Ellenőrzés

- `show ipv6 interface brief`

> (Figyeld meg a kimenetet! Bár csak egy címet adtál meg a portnak, a listában **KÉT** IPv6 cím fog szerepelni! A router minden IPv6-os interfészhez automatikusan generál magának egy ún. **Link-Local** címet is, ami általában `FE80`-al kezdődik. Ezeket a következő leckékben tárgyaljuk).

## Kvíz a 33. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod az IPv6 címzési és rövidítési szabályokból!

### Kérdés 1

Az alábbiak közül melyik **HÁROM** cím számít érvényes IPv6 címnek? (Válassz HÁRMAT!)

- A) `2001:db8:100:0:0:0:0:1`
- B) `2001:db8:100::1`
- C) `2001:dbG:100::1`
- D) `2001:0db8:0100:0000:0000:0000:0000:0000:0001`
- E) `2001:0db8:0100:0000:0000:0000:0000:0001`
- F) `2001::db8::1`

**Helyes Válasz: A, B és E**

**Magyarázat:** Az A a teljes kifejtett, rövidítetlen cím (amiben a kvartettek első nulláit kihagyták). A B ugyanaz a cím, tökéletesen lerövidítve a `::` segítségével. Az E a cím klasszikus, 32 hexa karakterből (8 db 4-es kvartettből) álló formája.
A C azért hibás, mert a `G` betű nem hexa karakter. A D azért hibás, mert 9 darab kvartettből áll (túl hosszú). Az F azért hibás, mert a `::` jel kétszer is szerepel benne, ami tilos (ambiguus, a gép nem tudná, hol hány nulla hiányzik).

### Kérdés 2

Melyik parancs kötelező a Globális Konfigurációs módban ahhoz, hogy a router elkezdjen IPv6 csomagokat továbbítani (Routolni) a különböző alhálózatok között?

- A) `ipv6 routing` (Interfész módból)
- B) `ipv6 routing` (Globális módból)
- C) `ipv6 unicast-routing` (Globális módból)
- D) `ipv6 unicast-routing` (Interfész módból)

**Helyes Válasz: C**

**Magyarázat:** A Cisco IOS vizsgaklasszikusa. Az IPv6 routing alapesetben ki van kapcsolva a routereken memóriaspórolás céljából. Hogy a router valóban routerként (és ne csak Hostként) viselkedjen az IPv6-on, a globális `ipv6 unicast-routing` paranccsal kell felébreszteni a motort.

### Kérdés 3

Adott az alábbi IPv6 cím: `2001:0db8:0000:00ab:0000:0000:0000:0001`. Melyik az egyetlen szabályosan lerövidített (abbreviated) formája a felsoroltak közül?

- A) `2001:db8:0:ab::1`
- B) `2001:db8::ab::1`
- C) `2001:db8:0:ab:0:0:0:1`
- D) `2001:db8::ab:0:0:0:1`

**Helyes Válasz: A**

**Magyarázat:** Vezessük le! A vezető nullákat (leading zeros) mindenhol elhagyhatjuk, így lesz: `2001:db8:0:ab:0:0:0:1`. Mivel a `::` (dupla kettőspont) jellel a leghosszabb (három kvartettes) egybefüggő nullás szakaszt tudjuk elegánsan eltüntetni (a vége előtt), és azt csak egyszer használhatjuk, a helyes eredmény: `2001:db8:0:ab::1` (ami az A válasz). A B hibás (kétszer van `::`). A C és a D esetben nem vonták össze `::`-tal a hátsó hosszú nullás sort (ami ugyan nem érvénytelen cím, de az A sokkal rövidebb, és a kérdés a "legjobban" rövidített formára vonatkozik, ráadásul a D-ben a rövidebb nullás részt rövidítették a hosszú helyett, ami bad practice).

### Bónusz Kérdés (Boson ExSim vizsgakérdés - IPv6 Static Route)

A hálózatodban a RouterA és a RouterB között Point-to-Point kapcsolat van (a közös hálózatuk: `2001:db8:1::/64`). A RouterA interfészének címe a `::1`, a RouterB címe a `::2`.
A RouterB egy másik portján keresztül kapcsolódik a RouterC-hez, ami a `2001:db8:2::/64` hálózatban van.

Milyen parancsot kell kiadnod a RouterA-n ahhoz, hogy statikusan, IPv6 alapon megtalálja a RouterC hálózatát? Válaszd a legjobb választ!

- A) `ipv6 route 2001:db8:1::/64 2001:db8:1::2`
- B) `ipv6 route 2001:db8:2::/64 2001:db8:1::2`
- C) `ipv6 route 2001:db8:1::/64 2001:db8:2::2`
- D) `ipv6 route 2001:db8:2::/64 2001:db8:2::2`

**Helyes Válasz: B**

**Magyarázat:** Az IPv6 statikus útválasztás (Static Routing) szintaxisa 100%-ban megegyezik az IPv4-el, csak pontozott maszkok helyett CIDR `/64` maszkot használ, és az `ip route` helyett `ipv6 route`-al kezdődik.

Képlet: `ipv6 route <Cél Hálózat> <Maszk> <Next-Hop (Következő IP-címe ugrás)>`.

- A cél a RouterC hálózata, azaz a `2001:db8:2::/64`.
- A Next-hop (A szomszéd, aki felé ki kell dobni a csomagot az R1-ből) a RouterB hozzánk néző interfésze, ami a `2001:db8:1::2`.

Összerakva: `ipv6 route 2001:db8:2::/64 2001:db8:1::2`, ami a B válasz! (Az A és a C rossz célt ad meg, a D pedig egy olyan IP-t ad meg next-hop-nak, amit a RouterA fizikailag még nem is lát a szomszédjában).
