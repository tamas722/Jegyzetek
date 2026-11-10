# CCNA Bővített Jegyzet – A Routing (Útválasztás) Alapjai

Az előző leckékben a helyi hálózatokkal (LAN) és a switchek működésével foglalkoztunk. Ebben a leckében rátérünk a hálózatok egyik legfontosabb témájára: a **Routingra (Útválasztásra)**. Megnézzük, hogyan juttatják el a routerek az IP csomagokat a megfelelő célállomásokra, miben különbözik ez a switchek módszerétől, és hogyan épül fel a router "térképe", a routing tábla.

## 1. Mi az a Routing (Útválasztás)?

Amikor egy router kap egy hálózati csomagot, az a feladata, hogy továbbítsa azt a helyes célállomás felé. A **Routing** az a folyamat, amelynek során a routerek meghatározzák, milyen útvonalon kell haladnia az IP csomagnak a hálózaton keresztül.

- A **Switchek** a MAC-címeket figyelik, és egy MAC-cím táblát (MAC Address Table) építenek.
- A **Routerek** az IP-címeket figyelik, és egy **Routing táblát (Routing Table)** vezetnek az ismert célhálózatokról.

Amikor egy csomag beérkezik, a router belenéz a routing táblájába, hogy megtalálja a legjobb útvonalat (route). Alapvetően három módon kerülhet be egy útvonal ebbe a táblába:

- **Dinamikus routing (Dynamic routing):** A routerek intelligens protokollokat (pl. OSPF) használnak arra, hogy automatikusan "beszélgessenek" egymással és kicseréljék az útvonal-információkat. (Később tanulunk róla).
- **Statikus routing (Static routing):** A hálózati mérnök kézzel, manuálisan gépeli be a routerbe a pontos útvonalakat. (A következő lecke témája).
- **Automatikusan hozzáadott útvonalak:** Ezek a **Connected (Csatlakoztatott)** és a **Local (Helyi)** útvonalak. Ezek nem dinamikusak és nem is statikusak – egyszerűen maguktól létrejönnek, amint beállítasz egy IP-címet a routeren. Ebben a leckében ezekre fókuszálunk.

### Mi is egy "útvonal" (Route) valójában?

Gondolj rá úgy, mint egy egyszerű utasításra a router számára. Például:

- "Ha a csomag az X célállomásra megy, küldd el az Y 'next-hop'-nak (a következő routernek az útvonalon)."
- "Ha a célállomás közvetlenül hozzád van kötve, küldd ki közvetlenül az adott porton."
- "Ha a célállomás a te saját IP-címed, akkor tartsd meg a csomagot (mert neked szól), és ne küldd tovább."

## 2. IP-címek konfigurálása (Cisco CLI)

Mielőtt megnézzük a táblázatot, lássuk, hogyan állítunk be IP-címeket a routeren. A példában a WAN (Wide Area Network – Nagy kiterjedésű hálózat) kapcsolatokon dolgozunk, ahol az R1 router a főszereplő.

Lépjünk be a Globális konfigurációs módba, és állítsuk be a GigabitEthernet 0/0 interfészt:

```text
Router# configure terminal
Router(config)# interface g0/0
Router(config-if)# ip address 192.168.13.1 255.255.255.0
Router(config-if)# no shutdown
```

**Egy nagyon hasznos CLI trükk:** Ha most át akarsz menni a g0/1-es interfészre, nem kell beírnod az `exit` parancsot, hogy visszalépj a globális módba! Közvetlenül az egyik interfész konfigurációs módjából átléphetsz a másikba:

```text
Router(config-if)# interface g0/1
Router(config-if)# ip address 192.168.12.1 255.255.255.0
Router(config-if)# no shutdown
```

Sikeres beállítás után a `show ip interface brief` paranccsal ellenőrizheted, hogy a portok megkapták-e a címet, és az állapotuk `UP/UP` lett-e.

## 3. A Routing Tábla (`show ip route`) és a C/L kódok

A router térképét, azaz a routing táblát a Privileged EXEC módból a következő paranccsal tudod lekérdezni:

```text
Router# show ip route
```

A kimenet két fő részből áll:

- **Codes (Kódok magyarázata):** A lista tetején látod a rövidítéseket (pl. `O` = OSPF, `S` = Static). Ebből minket most kettő érdekel: a **C (Connected)** és az **L (Local)**.
- **Maguk az útvonalak:** A tényleges lista, ami alapján a router döntéseket hoz.

Amint beállítasz egy IP-címet egy interfészen, és bekapcsolod azt (`no shutdown`), a router automatikusan hozzáad két útvonalat a táblához:

### A "Connected" (C) Útvonal

Ez az útvonal magát a hálózatot jelöli, ahova az interfész csatlakozik.

- **Példa:** Ha az R1-es router G0/2 portjának IP-címe `192.168.1.1/24`, akkor a C útvonal a `192.168.1.0/24`-es hálózatra fog mutatni.
- **Jelentése a router számára:** "Bármilyen csomag, ami a `192.168.1.0` és `192.168.1.255` közötti tartományba tart (pl. egy PC-nek), azt küldd ki egyenesen a G0/2-es porton, mert azok a gépek ott vannak közvetlenül rám kötve."

### A "Local" (L) Útvonal

Ez az útvonal a router saját, pontos IP-címét jelöli.

- **Példa:** Erre a `192.168.1.1/32` jelölést használja. A `/32`-es maszk (ami decimálisan `255.255.255.255`) azt jelenti, hogy mind a 32 bit fix! Vagyis ez nem egy tartományt jelöl, hanem kizárólag ezt az egyetlen, konkrét IP-címet.
- **Jelentése a router számára:** "Ha egy csomag pontosan a `192.168.1.1`-es célcímre érkezik, az személyesen nekem szól! Ne küldd tovább sehova, hanem dolgozd fel te magad (pl. vedd át a Pinget)."

> **Apró megjegyzés:** A `show ip route` kimenetben láthatsz ilyen sorokat is: `"192.168.1.0/24 is variably subnetted..."`. Ezek a sorok valójában **NEM** útvonalak, csak információs fejlécek a kategóriákhoz. Egyelőre egyszerűen hagyd őket figyelmen kívül, csak a **C** és **L** betűvel kezdődő sorokra koncentrálj!

## 4. Route Selection (Az Útvonalválasztás Szabálya)

Amikor egy csomag megérkezik, a router megnézi a csomag **Cél IP-címét**, és keres egy egyező (matching) útvonalat a táblában. Egy útvonal akkor egyezik, ha a cél IP-cím beletartozik az útvonalban megadott hálózati tartományba.

**De mi történik akkor, ha több útvonal is egyezik?**

Tegyük fel, a csomag célja: `192.168.1.1`.
A routing táblában ott van a Connected útvonal (`192.168.1.0/24`), ami lefedi az 1-től 254-ig terjedő címeket (így a cél is benne van). És ott van a Local útvonal (`192.168.1.1/32`), ami pontosan erre az egy címre mutat.

### Az Aranyszabály (A Legspecifikusabb Egyezés / Longest Prefix Match)

Ha több egyező útvonal van, a router **MINDIG a legspecifikusabb (legpontosabb) egyezést választja ki.**
Ezt a prefix hossz (a `/` utáni szám) határozza meg. A magasabb szám (a leghosszabb prefix) nyer!

- A `/24` egy 256 címet tartalmazó halmaz.
- A `/32` egyetlen egy címet tartalmaz.

Mivel a `/32` specifikusabb, mint a `/24`, a router a `/32`-es (Local) útvonalat fogja választani, és a csomagot megtartja magának!

### Mi történik, ha egyetlen útvonal sem egyezik? (Router vs. Switch különbség)

- A **Switch**, ha nem ismeri a célt (Unknown unicast), elárasztja (flood) a keretet minden porton.
- A **Router SOHA nem áraszt el csomagokat!** Ha a router kap egy csomagot, aminek a célcíméhez semmilyen egyezést nem talál a routing táblájában, akkor csendben **eldobja (drop)** a csomagot.

## 5. Gyakorlati Példák az Útvonalválasztásra

Nézzük meg, mit tesz az R1 router az alábbi 5 beérkező csomaggal!

### 1. A csomag célja: `192.168.1.1`

- **Egyezik:** A `1.0/24` (Connected) és a `1.1/32` (Local) útvonallal.
- **Döntés:** A `/32` specifikusabb. Ez az R1 saját címe. A router **megtartja** a csomagot.

### 2. A csomag célja: `192.168.13.3`

- **Egyezik:** Csak a `192.168.13.0/24` (Connected) útvonallal.
- **Döntés:** A router **továbbítja** (kiküldi) a csomagot a G0/0-s interfészen, mert ez a hálózat oda csatlakozik.

### 3. A csomag célja: `192.168.1.244`

- **Egyezik:** Csak a `192.168.1.0/24` (Connected) útvonallal (a `.244`-es gép benne van ebben a tartományban).
- **Döntés:** A router **továbbítja** a csomagot a G0/2-es interfészen.

### 4. A csomag célja: `192.168.12.1`

- **Egyezik:** A `12.0/24` (Connected) és a `12.1/32` (Local) útvonallal.
- **Döntés:** A `/32` specifikusabb. Ez is az R1 saját (másik portjának) címe. A router **megtartja** a csomagot.

### 5. A csomag célja: `192.168.4.10`

- **Egyezik:** Egyikkel sem. (Nincs ilyen hálózat a táblában).
- **Döntés:** A router **eldobja (drop)** a csomagot!

## Kvíz a 11. Leckéhez

Ellenőrizd a tudásod a lecke alapján! Itt vannak a kérdések, a válaszlehetőségek és a magyarázatok.

### Kérdés 1

Egy router interfészére beállított konkrét IP-cím milyen típusú útvonalként jelenik meg a routing táblában?

- A) Connected (Csatlakoztatott)
- B) Static (Statikus)
- C) Local (Helyi)
- D) Dynamic (Dinamikus)

**Helyes Válasz: C**

**Magyarázat:** Amikor konfigurálsz egy IP-címet (pl. `192.168.1.1/24`), a hálózat (`1.0/24`) Connected (C) útvonalként jelenik meg, de maga a pontos IP-cím (`1.1/32`) egy Local (L) útvonalként jön létre, jelezve, hogy az a router sajátja.

### Kérdés 2

Nézd meg az R1 router tábláját! Mit fog tenni a router, ha egy olyan csomagot kap, amelynek a cél IP-címe `192.168.3.25`? (Adott egy L - `192.168.3.25/32` útvonal a táblában a G0/2-es interfészen).

- A) Eldobja a csomagot.
- B) Megtartja/fogadja a csomagot saját magának.
- C) Továbbítja a csomagot a G0/2-es interfészen egy másik eszköznek.
- D) Elárasztja (flood) a csomagot minden interfészén.

**Helyes Válasz: B**

**Magyarázat:** A routing táblában van egy Local (L) útvonal, ami pontosan a `192.168.3.25/32`-re mutat. A Local útvonalak a router saját interfészeinek IP-címeit jelölik, így a csomag személyesen a routernek szól. A router felbontja és magának dolgozza fel.

### Kérdés 3

Az alábbi állítások közül melyek igazak a routerek és a switchek viselkedésére? (Válassz ki KETTŐT!)

- A) A switchek eldobják a kereteket, ha a célállomás ismeretlen számukra.
- B) A switchek elárasztják (flood) a kereteket, ha a célállomás ismeretlen számukra.
- C) A routerek eldobják a csomagokat, ha a célállomás ismeretlen számukra.
- D) A routerek elárasztják (flood) a csomagokat, ha a célállomás ismeretlen számukra.

**Helyes Válasz: B és C**

**Magyarázat:** Ez egy alapvető, kritikus különbség a Layer 2 és a Layer 3 között! A Switch sosem dob el ismeretlen (Unknown unicast) keretet, hanem biztonságból "kiabál" (floodol), hogy mindenképp célba érjen. A Router soha nem floodol ismeretlen IP csomagot; ha nem ismeri az útvonalat, azonnal eldobja azt (drop).

### Kérdés 4

Melyik az a két típusú útvonal, ami automatikusan hozzáadódik a routing táblához, amint beállítasz egy IP-címet egy interfészen, és bekapcsolod azt?

- A) C (Connected) és L (Local)
- B) S (Static) és D (Dynamic)
- C) C (Connected) és O (OSPF)
- D) L (Local) és S (Static)

**Helyes Válasz: A**

**Magyarázat:** A `no shutdown` és az IP-cím beállítása után a hálózat (pl. `/24`) megkapja a C (Connected) kódot, a konkrét IP-cím (mindig `/32`-es maszkkal) pedig megkapja az L (Local) kódot. Se a statikus (manuális), se a dinamikus (OSPF) útvonalak nem jönnek létre maguktól.

### Kérdés 5

Tegyük fel, hogy a routing táblában van egy `10.0.1.0/24` (Connected) és egy `10.0.1.23/32` (Local) útvonal. Ha a router egy `10.0.1.23`-as célcímű csomagot kap, hány útvonal egyezik a céllal, és melyik lesz a legspecifikusabb egyezés?

- A) Csak egy útvonal egyezik (a `10.0.1.0/24`).
- B) Két útvonal egyezik, és a legspecifikusabb a `10.0.1.0/24`.
- C) Két útvonal egyezik, és a legspecifikusabb a `10.0.1.23/32`.
- D) Csak egy útvonal egyezik (a `10.0.1.23/32`).

**Helyes Válasz: C**

**Magyarázat:** A csomag célja (a `.23`) beletartozik a `1.0/24`-es tartományba is, és milliméterre megegyezik a `.23/32`-es konkrét címmel is. Tehát **KÉT** útvonal egyezik. Mivel az útvonalválasztás aranyszabálya a "Longest Prefix Match" (a legnagyobb `/` szám nyer), a `/32` a nyertes, mert az sokkal specifikusabb.
