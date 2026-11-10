# CCNA Bővített Jegyzet – 28. Lecke: OSPF 1. Rész (Alapok és Konfiguráció)

Ebben a leckében belevágunk a CCNA vizsga egyetlen hivatalosan nevesített és legfontosabb dinamikus útválasztási protokolljába, az **OSPF**-be (Open Shortest Path First). Míg a RIP és az EIGRP a "pletykákra" támaszkodik, az OSPF teljesen más logikát használ: feltérképezi a teljes hálózatot, és maga számolja ki a legjobb utakat.

## 1. Mi az az OSPF? (Alapfogalmak)

Az OSPF egy **Link State (Kapcsolatállapot)** belső átjáró protokoll (IGP).

- **Az algoritmus, amit használ:** **Dijkstra's Shortest Path First (SPF)** algoritmusa. (Ezt a nevet érdemes megjegyezni!)
- **Verziók:** Az OSPFv1 elavult. Az IPv4 hálózatokon (és a CCNA vizsgán is) a **OSPFv2**-t használjuk. (Az OSPFv3 az IPv6-hoz készült, bár képes IPv4-et is kezelni).
- **Metrika:** Az OSPF a **Költség (Cost)** alapján számol, ami a kábelek sávszélességéből adódik (a gyorsabb kábel "olcsóbb"). Az **AD (Adminisztratív Távolság) értéke 110**.

### Hogyan működik a Link State (Kapcsolatállapot) logika?

Szemben a Distance Vector (RIP, EIGRP) protokollokkal, a router itt nem csak azt osztja meg, hogy milyen messze van egy hálózat. Ehelyett **LSA (Link State Advertisement)** üzenetek formájában megosztja a szomszédokkal a saját "állapotát" (milyen portjai vannak, mik az IP-címeik, mennyi a költségük).

1. A routerek ezeket az LSA-kat elárasztják (**flood**) a hálózatban.
2. Minden router összegyűjti az összes LSA-t, és beteszi őket a saját **LSDB (Link State Database)** nevű adatbázisába. (A hálózaton belül minden routernek hajszálpontosan ugyanazzal az LSDB-vel kell rendelkeznie!).
3. Ebből az adatbázisból a router felépíti a teljes hálózat "térképét".
4. Végül a **Dijkstra** algoritmus segítségével minden router önállóan kiszámolja a legrövidebb (legkisebb költségű) útvonalat a térkép minden pontjához, és ezt beírja az IP Routing Table-be.

> (Az LSA-k alapértelmezetten 30 percenként "elöregszenek" és újra kiküldésre kerülnek frissítés gyanánt).

## 2. Az OSPF Területek (Areas) Tervezése

Egy hatalmas (pl. 500 routerből álló) hálózatban, ha egyetlen kábel elszakad a hálózat egyik végén, az LSA-t 500 gépnek kellene megkapnia, és mind az 500-nak újra le kéne futtatnia az erőforrás-igényes SPF algoritmust. Hogy ezt a katasztrófát elkerüljék, az OSPF a hálózatot kisebb **Területekre (Areas)** bontja.

A CCNA vizsgán csak a **Single-Area (Egyterületes)** OSPF-et kell tudni konfigurálni (ilyenkor mindig az Area 0-át használjuk), de az elméletet érteni kell a **Multi-Area (Többterületes)** hálózatokról is.

### A Területtípusok és a Routerek Szerepei

- **Area 0 (Backbone Area - Gerinchálózat):** Ez a hálózat magja. A legfontosabb OSPF szabály: **Minden más területnek KÖZVETLENÜL csatlakoznia kell az Area 0-hoz!** (Nem lehet Area 2-t az Area 1-hez kötni úgy, hogy nem érinti a 0-t!).
- **Internal Router (Belső router):** Olyan router, amelynek minden interfésze ugyanabban a területben (Area) van.
- **ABR (Area Border Router - Területhatár-router):** Olyan router, aminek az interfészei több területbe is belelógnak (pl. az egyik portja az Area 0-ban, a másik az Area 1-ben van). Ők kötik össze a területeket, és minden területhez külön LSDB-t (adatbázist) vezetnek. (Javasolt, hogy egy ABR maximum 2 területhez csatlakozzon).
- **Backbone Router (Gerincrouter):** Bármilyen router, aminek van legalább egy portja az Area 0-ban. (Tehát a "belső Area 0" routerek és az ABR-ek is ide tartoznak).
- **ASBR (Autonomous System Boundary Router - Rendszerhatár-router):** Egy OSPF hálózatot köt össze a "külvilággal" (pl. az Internettel, vagy egy másik protokollal futó hálózattal, és Default Route-ot reklámoz a többieknek).

### Útvonaltípusok (Routes)

- **Intra-area Route:** Az útvonal (célpont) ugyanabban a területben van, mint maga a router.
- **Interarea Route:** Az útvonal egy másik területbe mutat (pl. Area 1-ből Area 2-be).

## 3. Cisco IOS Konfiguráció (OSPF Alapok)

Az OSPF aktiválása a RIP-hez és az EIGRP-hez nagyon hasonló, de van benne egy "Area" (Terület) csavar!

### 1. Az OSPF Process (Folyamat) elindítása

Az EIGRP-vel ellentétben (ahol az AS számnak egyeznie kellett a szomszédokon), az **OSPF Process ID csak helyileg (lokálisan) számít**. Lehet R1-en az 1, R2-n a 2, attól még vígan szomszédok lesznek!

```text
Router(config)# router ospf 1
```

### 2. A "Network" parancs (Wildcard maszkkal és Területtel!)

Az OSPF (akárcsak az EIGRP) **Wildcard maszkot** (Inverz maszkot) használ. Viszont a parancs végén **KÖTELEZŐ megadni a területet is!**

> **Emlékeztető:** A `network` parancs nem azt mondja meg, mit reklámozzon a router, hanem hogy **MELY fizikai portokon induljon el a protokoll!** A router a portjain található valódi hálózatot fogja hirdetni az LSA-kban.

```text
Router(config-router)# network 10.0.12.0 0.0.0.15 area 0
```

> (Tipp a CCNA-ra: Ha nem akarsz bajlódni a wildcard kiszámolásával, megadhatod konkrétan az interfész IP-címét egy `/32`-es, azaz `0.0.0.0` maszkkal is, pl: `network 10.0.12.1 0.0.0.0 area 0`. Ezzel pontszerűen aktiválod az OSPF-et az adott porton).

### 3. Hasznos Parancsok

- `passive-interface g2/0`: Mint a többi protokollnál, ez is megakadályozza, hogy a port OSPF Hello üzeneteket küldjön, de a hálózatát ettől még beleteszi a térképbe (ideális PC-k felé).
- `default-information originate`: Ezzel a paranccsal a router (amiből ettől a pillanattól kezdve hivatalosan is **ASBR** lesz!) szétküldi az összes szomszédjának a saját statikus Default Route-ját (pl. ami az Internet felé mutat).

## 4. Az OSPF Router ID (A Router Neve)

Minden OSPF routernek kell egy 32 bites (IP-cím formátumú) azonosító (pl. `1.1.1.1`). Ha nincs azonosító, az OSPF nem indul el. A kiválasztási sorrend teljesen megegyezik az EIGRP-vel:

1. **Manuális konfiguráció** (Ez a legtisztább, erősen ajánlott!): `router-id 1.1.1.1`
2. A legmagasabb IP-című **Loopback** interfész.
3. A legmagasabb IP-című aktív **Fizikai** interfész.

> **Csapda:** Ha az OSPF már elindult egy fizikai port IP-címével, és utólag adsz meg egy manuális `router-id`-t, az **nem lép életbe azonnal!** Vagy újra kell indítani a routert, vagy ki kell adni a (forgalmat pár másodpercre megszakító) `clear ip ospf process` parancsot a Privileged EXEC módban!

## Kvíz a 28. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod az OSPF területekről és a konfigurációkról!

### Kérdés 1

Az OSPF-re vonatkozó alábbi állítások közül melyik **KETTŐ hamis**? (Válassz KETTŐT!)

- A) Multi-area (Többterületes) OSPF hálózatokban minden nem-gerinc területnek (Area 1, 2, stb.) rendelkeznie kell egy ABR-rel, ami közvetlenül kapcsolódik az Area 0-hoz.
- B) A Single-area (Egyterületes) OSPF hálózatokban kötelező az Area 0-t használni.
- C) Két OSPF router különböző Process ID-vel (Folyamat Azonosítóval) is tud egymással szomszédságot (neighbor) kialakítani.
- D) A `network` parancsban kötelező megadni az Area (Terület) számát.
- E) Az ASBR összeköti a belső OSPF hálózatot az OSPF domainen kívüli (pl. Internet) hálózatokkal.
- F) Az OSPF Process ID-nek meg kell egyeznie az Area számával.

**Helyes Válasz: B és F**

**Magyarázat:** Bár a Single-Area OSPF esetén szinte mindig az Area 0-t használjuk (ez a legjobb gyakorlat), a szoftver egyáltalán nem teszi ezt kötelezővé (használhatod akár az Area 5-öt is, amíg mindenki abban van). Az F szintén hamis: a Process ID (pl. `router ospf 1`) egy lokális folyamat sorszám a RAM-ban, semmi köze az Area sorszámához, és nem kell egyeznie a szomszédokéval.

### Kérdés 2

Melyik paranccsal tudod a Cisco CLI-ben manuálisan beállítani az OSPF Router ID-jét az `1.1.1.1`-re?

- A) `router-id 1.1.1.1` (OSPF konfigurációs módban)
- B) `ospf router-id 1.1.1.1` (Globális konfigurációs módban)
- C) `interface loopback 0` majd `ip address 1.1.1.1 255.255.255.255`
- D) `router ospf 1 router-id 1.1.1.1`

**Helyes Válasz: A**

**Magyarázat:** Az OSPF konfigurációs módba belépve (`router ospf 1`) egyszerűen a `router-id 1.1.1.1` parancsot kell kiadni. (Szemben az EIGRP-vel, ahol a parancs az `eigrp router-id` lenne). Bár a C válasz létrehoz egy Loopback interfészt, ami később válhat Router ID-vé, az nem a manuális beállítás módja.

### Kérdés 3

Az R1 router két fizikai interfészén szeretnéd aktiválni az OSPF-et egyetlen `network` paranccsal. A G0/1 IP-címe `10.0.12.1/28`, a G0/2 IP-címe pedig `10.0.13.1/26`. Melyik parancs aktiválja **MINDKÉT** interfészt az Area 0-ban?

- A) `network 10.0.12.0 0.0.0.255 area 0`
- B) `network 10.0.13.0 0.0.0.255 area 0`
- C) `network 10.0.12.0 0.0.1.255 area 0`
- D) `network 10.0.0.0 0.0.0.255 area 0`

**Helyes Válasz: C**

**Magyarázat:** Ahhoz, hogy egy parancs mind a `10.0.12.1`-et, mind a `10.0.13.1`-et lefedje, olyan Wildcard maszkra van szükségünk, amiben az első két oktett (a 10 és a 0) szigorúan egyezik, de a harmadik oktett megengedi mind a 12-t, mind a 13-at. A `0.0.1.255` wildcard maszk azt jelenti, hogy az első két oktett (`0.0`) fix, a harmadik oktettben 1 bit mozgásterünk van, a negyedikben (`255`) pedig bármi lehet. Ebbe pont beleesik mindkét interfész! (Az A csak a `12.1`-et, a B csak a `13.1`-et fogná meg).

### Bónusz Kérdés (Boson ExSim vizsgakérdés)

Te adminisztrálod a hálózatot. Kiadod a `default-information originate` parancsot a RouterA-n (amely futtatja az OSPF-et). Az alábbi állítások közül melyik **KETTŐ igaz**? (Válassz KETTŐT!)

- A) Az OSPF reklámozni (hirdetni) fogja a RouterA "Gateway of last resort"-ját (Alapértelmezett átjáróját).
- B) A RouterA-ból OSPF ABR (Area Border Router) lesz.
- C) Az OSPF összesíti (summarize) a RouterA közvetlenül csatlakoztatott útvonalait.
- D) A RouterA-ból OSPF ASBR (Autonomous System Boundary Router) lesz.
- E) Az OSPF redisztribuálni fogja a RouterA összes közvetlenül csatlakoztatott útvonalát.

**Helyes Válasz: A és D**

**Magyarázat:** Amikor kiadod a `default-information originate` parancsot az OSPF-ben, a router megnézi, hogy van-e neki statikus Default Route-ja (Gateway of last resort), és ha van, csinál belőle egy speciális LSA-t, majd reklámozza azt a többi routernek (így az A igaz). Mivel ezzel egy, az OSPF hálózaton kívüli célpont (az Internet) útvonalát fecskendezte be a rendszerbe, a RouterA funkciója megváltozik: onnantól kezdve **ASBR**-nek (Rendszerhatár-routernek) hívják, hiszen ő köti össze az OSPF-et a külvilággal (így a D is igaz). Az ABR-hez több OSPF Területbe kellene tartoznia.
