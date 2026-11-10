# CCNA Bővített Jegyzet – A Dinamikus Útválasztás (Dynamic Routing) Alapjai

Ebben a leckében visszatérünk a 3. réteghez (Layer 3), és megismerkedünk a dinamikus útválasztással. Míg a statikus útválasztás (Static Routing) során minden útvonalat kézzel, manuálisan kell beállítani, a dinamikus protokollok lehetővé teszik a routerek számára, hogy automatikusan feltérképezzék a hálózatot, megtalálják a legjobb útvonalakat, és azonnal reagáljanak a hálózati hibákra (pl. kábelszakadásra).

## 1. Network Route (Hálózati útvonal) vs. Host Route (Gép útvonal)

Mielőtt belevágunk a protokollokba, tisztázzunk két alapfogalmat az útválasztási táblával (Routing Table) kapcsolatban:

- **Network Route (Hálózati útvonal):** Olyan útvonal, amely egy egész hálózatra vagy alhálózatra (Subnet) mutat. A maszkja mindig kisebb, mint `/32` (pl. `192.168.4.0/24` vagy `10.0.12.0/30`).
- **Host Route (Gép útvonal):** Olyan útvonal, amely egyetlen specifikus, konkrét IP-címre mutat. A maszkja mindig `/32` (pl. `10.0.12.1/32`). A routerek a saját interfészeik IP-címeit automatikusan Host route-ként (Local - L kód) adják hozzá a táblához.

## 2. A Dinamikus Útválasztási Protokollok Típusai

A dinamikus protokollok automatikusan "reklámozzák" (advertise) a szomszédoknak a saját hálózataikat, és megosztják az általuk ismert útvonalakat. Két fő kategóriába soroljuk őket a működési hatókörük alapján:

- **IGP (Interior Gateway Protocol - Belső átjáró protokoll):** Egyetlen autonóm rendszeren (Autonomous System - AS), azaz egyetlen szervezeten vagy vállalaton belül használják őket a forgalom irányítására.
- **EGP (Exterior Gateway Protocol - Külső átjáró protokoll):** Különálló autonóm rendszerek (pl. két különböző vállalat vagy internetszolgáltatók) közötti útválasztásra szolgálnak. A modern hálózatokban (és az Interneten) egyetlen EGP-t használnak: ez a **BGP (Border Gateway Protocol)**, amely Path Vector algoritmust használ.

### Az IGP Protokollok Algoritmusai

Az IGP protokollokat aszerint csoportosítjuk, ahogyan a hálózati adatokat kezelik és továbbítják:

#### 1. Distance Vector (Távolságvektor) Protokollok

- **Képviselőik:** RIP és EIGRP (a Cisco sajátja).
- **Működés ("Routing by rumor" - Útválasztás pletyka alapján):** A router nem látja át a teljes hálózatot. Csak annyit tud, amit a közvetlen szomszédja mond neki (milyen hálózatokat ismer, és milyen messze vannak). A router ezt az információt adja tovább.

#### 2. Link State (Kapcsolatállapot) Protokollok

- **Képviselőik:** OSPF és IS-IS.
- **Működés:** Minden router megosztja a saját interfészeinek (kapcsolatainak) állapotát, amit a többiek továbbítanak, amíg minden egyes router fel nem épít egy teljes, azonos "térképet" (Connectivity map) a teljes hálózatról. Ezután minden router önállóan, matematikai alapon kiszámolja a legjobb útvonalakat a térkép alapján. (Több erőforrást igényelnek, de gyorsabban reagálnak a változásokra).

## 3. A Metrika (Metric) és az ECMP

Ha egy router egy ugyanolyan dinamikus protokollon keresztül (pl. OSPF) két különböző útvonalat is tanul ugyanahhoz a célállomáshoz, hogyan dönti el, melyiket használja?

A **Metrika (Metric)** alapján! A metrika jelzi, hogy milyen "messze" van a cél. A legalacsonyabb metrikával rendelkező útvonal a nyertes, és az kerül be a Routing Table-be.

### Hogyan számolják a protokollok a Metrikát?

- **RIP:** Hop count (Ugrások száma). Csak azt számolja, hány routeren kell átmenni. (Nagyon buta rendszer, mert a 10 Mbps-es és a 10 Gbps-es kábelt is 1 ugrásnak veszi).
- **EIGRP:** Sávszélesség (Bandwidth) és Késleltetés (Delay) bonyolult képlete.
- **OSPF:** Költség (Cost). A kábelek sávszélessége alapján számol (a gyorsabb kábel "olcsóbb", így alacsonyabb a metrikája).
- **IS-IS:** Költség (Cost). Bár alapból minden kábelnek fix 10-es költséget ad, így konfiguráció nélkül ugyanúgy viselkedik, mint a RIP.

### ECMP (Equal Cost MultiPath - Egyenlő Költségű Többszörös Útvonal)

Ha a router két útvonalat tanul (ugyanazzal a protokollal, pontosan ugyanahhoz a célhoz), és a metrikájuk hajszálpontosan megegyezik, akkor mindkettőt beírja a Routing Table-be, és megosztja a forgalmat a két útvonal között (Load Balancing / Terheléselosztás). Ez a statikus útvonalakkal is működik, ha két azonos célú útvonalat írsz be manuálisan.

## 4. Az Adminisztratív Távolság (Administrative Distance - AD)

Mi történik akkor, ha a router egy útvonalat az OSPF-től (Metrika: 30), ugyanazt az útvonalat pedig az EIGRP-től (Metrika: 33280) is megtanulja? Mivel a két metrikát nem lehet összehasonlítani (almát a körtével), a router az **Adminisztratív Távolságot (AD)** használja a döntéshez.

Az AD mutatja meg, hogy a router mennyire "bízik meg" a protokollban. A legalacsonyabb AD érték nyer! (A CCNA vizsgára a következő táblázatot **KÖTELEZŐ** kívülről tudni):

| Útvonal Forrása / Protokoll | Adminisztratív Távolság (AD) |
|---|---|
| Connected (Közvetlenül csatlakoztatott) | 0 (A legmegbízhatóbb) |
| Static (Statikus útvonal) | 1 |
| eBGP (Külső BGP) | 20 |
| EIGRP | 90 |
| IGRP (Régi Cisco protokoll) | 100 |
| OSPF | 110 |
| IS-IS | 115 |
| RIP | 120 |
| EIGRP (External - Külső) | 170 |
| iBGP (Belső BGP) | 200 |
| Használhatatlan / Megbízhatatlan | 255 (A router SOHA nem írja be a táblába) |

**Példa:** Az EIGRP (AD: 90) mindig le fogja győzni az OSPF-et (AD: 110), az OSPF pedig a RIP-et (AD: 120).

## 5. Lebegő Statikus Útvonal (Floating Static Route) és a Longest Prefix Match

### Floating Static Route

A statikus útvonalak alapértelmezett AD-je 1. Ennek a megváltoztatásával (pl. `ip route 192.168.4.0 255.255.255.0 10.0.12.2 100`) az AD-t 100-ra növelhetjük.

Ez az útvonal inaktív marad, és nem kerül be a Routing Table-be mindaddig, amíg egy jobb AD-jű dinamikus protokoll (pl. EIGRP, ami 90) biztosítja az útvonalat. Ha a dinamikus protokoll megszakad, ez az útvonal azonnal "beúszik" (float) a táblába, mint egy tökéletes biztonsági háló (backup route).

### A Legfontosabb Útválasztási Szabály (Longest Prefix Match)

Ha egy csomag megérkezik a routerhez (pl. Cél IP: `10.20.0.14`), a router **először nem az AD-t vagy a Metrikát nézi!** Azt az útvonalat fogja választani a táblából, amelyik a leghosszabb / legspecifikusabb egyezést (Longest Prefix Match) adja.

Egy `/28`-as útvonal mindig legyőz egy `/26`-os, `/24`-es vagy `/22`-es útvonalat, függetlenül attól, hogy az egyik Statikus (AD: 1), a másik pedig OSPF (AD: 110). (Lásd a Boson kvízkérdést lentebb).

## Kvíz a 24. Leckéhez

### Kérdés 1

Az R1 négy különböző protokolltól (RIP, EIGRP, OSPF, IS-IS) tanul meg egy útvonalat pontosan a `192.168.1.0/24` hálózat felé. Melyik útvonal (vagy útvonalak) kerülnek be a Routing Table-be?

- A) Csak a RIP.
- B) Csak az EIGRP.
- C) Csak az OSPF.
- D) Csak az IS-IS.
- E) A RIP és az EIGRP, mert mindkettő distance vector.

**Helyes Válasz: B**

**Magyarázat:** Ha több különböző protokolltól érkezik útvonal hajszálpontosan ugyanahhoz a célhoz, a router az Adminisztratív Távolság (AD) alapján dönt. A legalacsonyabb nyer. RIP = 120, OSPF = 110, IS-IS = 115, EIGRP = 90. Mivel az EIGRP-nek van a legalacsonyabb (legmegbízhatóbb) AD értéke, kizárólag ez az egy útvonal kerül a táblába.

### Kérdés 2

Melyik útválasztási protokoll típust hívják gyakran "routing by rumor" (útválasztás pletyka alapján) módszernek?

- A) Link state (Kapcsolatállapot)
- B) Path vector
- C) Distance vector (Távolságvektor)
- D) Interior gateway

**Helyes Válasz: C**

**Magyarázat:** A Distance Vector protokollok (pl. RIP, EIGRP) nem látják át a teljes hálózatot. Csak azt "hiszik el" és adják tovább, amit a közvetlen szomszédjuk mesélt nekik az útvonalakról és azok távolságáról, innen ered a "pletyka" elnevezés.

### Kérdés 3

Az R1 két útvonalat tanul meg a `172.16.0.0/16` hálózathoz a RIP protokollon keresztül (az egyiket a `10.0.0.1` felé, a másikat a `10.1.0.1` felé). Mindkét útvonal Metrikája (Hop count) pontosan 5. Melyik útvonal(ak) kerülnek be a táblába?

- A) Mindkét útvonal.
- B) Csak a `10.0.0.1` felé.
- C) Csak a `10.1.0.1` felé.
- D) Egyik sem, mert a RIP AD értéke túl magas.

**Helyes Válasz: A**

**Magyarázat:** Mivel a router mindkét útvonalat ugyanattól a protokolltól (RIP) kapta, pontosan ugyanahhoz a hálózathoz, és a metrikájuk egyenlő (5), mindkét útvonalat beírja a Routing Table-be. Ezt hívják ECMP-nek (Equal Cost MultiPath), a router pedig megosztja a forgalmat a két útvonal között (load balancing). A D azért helytelen, mert nincs másik, jobb AD-jű protokoll a képben, ami legyőzné a RIP-et.

### Bónusz Kérdés (Boson ExSim vizsgakérdés - A Legnagyobb CCNA Csapda!)

A RouterA `show ip route` parancsának kimenetében négy útvonalat látsz:

```text
S 10.20.0.0/22 [1/0] via ...
R 10.20.0.0/24 [120/1] via ...
D 10.20.0.0/26 [90/2560] via ...
O 10.20.0.0/28 [110/10] via ...
```

A RouterA egy adatcsomagot kap, aminek a Cél IP-címe `10.20.0.14`. A fenti útvonalak közül melyiket fogja használni a csomag továbbítására?

- A) A RIP útvonalat, mert a legmagasabb az AD-je.
- B) Az OSPF útvonalat, mert ez rendelkezik a leghosszabb prefix (maszk) egyezéssel.
- C) A statikus útvonalat (S), mert a statikus útvonalak az AD=1 miatt mindig legyőzik a dinamikusokat.
- D) Az EIGRP útvonalat (D), mert a dinamikus protokollok közül ennek van a legalacsonyabb AD-je (90).

**Helyes Válasz: B**

**Magyarázat:** A `10.20.0.14`-es cél IP-cím mind a négy felsorolt tartományba beletartozik, tehát a router bármelyiket választhatná. Ilyenkor a kezdő diákok egyből az AD-t (Adminisztratív Távolságot) nézik, és rávágják, hogy a Statikus (AD: 1) vagy az EIGRP (AD: 90) nyer. **EZ HIBÁS!**

A routerek legelső és legszigorúbb szabálya a **Longest Prefix Match** (A leghosszabb prefix egyezés). A router mindig a legpontosabb/legspecifikusabb célzást választja. Mivel a `/28` a leghosszabb maszk, az OSPF útvonal lesz a győztes, hiába van neki rosszabb (110-es) AD-je, mint a statikusnak vagy az EIGRP-nek! (Az AD-t csak és kizárólag akkor használják, ha a prefixek is, azaz a per utáni számok is hajszálpontosan megegyeznek!).
