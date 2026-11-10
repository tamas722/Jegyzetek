# CCNA Bővített Jegyzet – 29. Lecke: OSPF 2. Rész (Költség, Szomszédság és Állapotok)

Ebben a leckében elmélyedünk az OSPF motorházteteje alatt! Megnézzük, hogyan számolja a gép az útvonalak "költségét" (Metrikáját), hogyan lehet ezt manuálisan felülbírálni, és legfőképpen: hogyan épül fel lépésről lépésre a híres OSPF Szomszédság (Adjacency) a 7 kritikus állapoton keresztül.

## 1. Az OSPF Költség (Cost / Metrika)

Az OSPF a kábelek sávszélessége alapján számolja ki, hogy melyik útvonal a legjobb. A képlet:

$$
\text{Referencia Sávszélesség} / \text{Interfész Sávszélesség} = \text{Költség}
$$

### A Probléma a Gyári Alapbeállítással

A Cisco IOS-ben az alapértelmezett **Referencia Sávszélesség 100 Mbps** (Megabit/s). Emiatt a matek így néz ki:

- Ethernet (10 Mbps) → $100 / 10 =$ **10** (Költség: 10)
- FastEthernet (100 Mbps) → $100 / 100 =$ **1** (Költség: 1)
- GigabitEthernet (1000 Mbps) → $100 / 1000 =$ **0.1**
- 10-Gig Ethernet (10000 Mbps) → $100 / 10000 =$ **0.01**

Az OSPF minden 1-nél kisebb eredményt **1-re kerekít!** Emiatt a gyári beállítással a FastEthernet, a Gigabit és a 10-Gigabit kábelek a router szemében hajszálpontosan ugyanolyan gyorsak (Költségük 1). Ez rossz útválasztáshoz vezet.

### A Referencia Sávszélesség Megváltoztatása

Hogy a router különbséget tudjon tenni a modern, gyors kábelek között, a Referencia Sávszélességet **MINDEN routeren egységesen** fel kell emelni (például 100 000 Mbps-re, ami a 100 Gbps-es kábelekre készíti fel a hálózatot). Ekkor a Gigabit költsége 100 lesz, a FastEthernet-é 1000.

```text
Router(config-router)# auto-cost reference-bandwidth 100000
```

(A számot mindig **Megabit/s**-ban kell megadni!)

### Más módok a Költség módosítására

Bár a `reference-bandwidth` globális módosítása a legjobb gyakorlat, az egyes interfészek költségét manuálisan is felülbírálhatod.

#### A Költség direkt átírása (Ajánlott)

```text
Router(config-if)# ip ospf cost 10000
```

(Ezzel erőszakkal megmondod a routernek, hogy ez a kábel "drága", anélkül, hogy a fizikai sávszélességet bántanád).

#### 2. Az Interfész Sávszélességének átírása (Nem ajánlott)

```text
Router(config-if)# bandwidth 100000
```

(A sávszélességet **Kilobit/s**-ban kell megadni. Ezt nem érdemes használni, mert nem csak az OSPF-et vered át vele, hanem a router QoS és más statisztikai szolgáltatásait is).

### Számolás a Routing Táblában

Egy célhoz vezető útvonal teljes költsége a **kifelé mutató (outgoing)** interfészek költségeinek összege. (A befelé jövő portok költsége nem számít!).

> **Extra infó:** A Loopback (Virtuális) interfészek fix költsége mindig **1**.

## 2. Az OSPF Szomszédság (Neighbor States) – A 7 Állapot

A legfontosabb troubleshooting feladat az OSPF-ben annak ellenőrzése, hogy a routerek felépítették-e a szomszédságot. Ehhez 7 lépésen (állapoton) kell hibátlanul átmenniük. A Hello üzeneteket a `224.0.0.5`-ös Multicast címre küldik. (Az Ethernet hálózatokon a **Hello Timer 10 másodperc**, a **Dead Timer 40 másodperc**).

### 1. Down (Lent) Állapot

Az OSPF elindul az interfészen. Az R1 kiküldi az első Hello üzenetét, de még egyetlen szomszédról sem tud (üres a táblája). A csomagban a "Szomszéd Router ID" mező értéke `0.0.0.0`.

### 2. Init (Inicializálás) Állapot

Az R2 megkapja az R1 Hello üzenetét. Látja, hogy egy új router van a vonalon, de az ő saját neve (Router ID-je) még nem szerepel az R1 üzenetében. Ezt hívják Init állapotnak.

### 3. 2-way (Kétirányú) Állapot

Az R2 visszaküld egy Hello üzenetet, amibe már beírja: "Látlak téged, R1!". Amint mindkét router látja a saját nevét (Router ID-jét) a másik Hello üzenetében, létrejön a **2-way** állapot!

A 2-way állapot azt jelenti, hogy a kapcsolat stabil, és a felek felkészültek az adatbázisaik cseréjére.

### 4. Exstart (Csere-előkészítés) Állapot

Mielőtt kicserélnék az adatbázisukat (LSDB), el kell dönteniük, ki kezdje a beszédet. Ehhez a fázishoz **DBD (Database Description)** csomagokat küldenek egymásnak.

Amelyik routernek magasabb a **Router ID**-je, az lesz a **Master (Mester)**, az alacsonyabb a **Slave (Szolga)**. (A Master fogja diktálni a tempót a következő fázisban).

### 5. Exchange (Csere) Állapot

A routerek kicserélik a DBD csomagjaikat, amikben az LSDB (Link State Database) tartalomjegyzéke található (csak a fejezetek címei, nem a teljes adatok). A routerek összehasonlítják a szomszéd listáját a sajátjukkal, és felírják, mi hiányzik nekik.

### 6. Loading (Betöltés) Állapot

A tényleges adatok (LSA-k) letöltése! Ehhez háromféle csomagot használnak:

- **LSR (Link State Request - Kérés):** "Kérem az X, Y, Z adatait!"
- **LSU (Link State Update - Frissítés):** "Tessék, itt vannak az X, Y, Z részletes adatai (LSA-k)!"
- **LSAck (Link State Acknowledgment - Nyugta):** "Köszönöm, megkaptam és hibátlan."

### 7. Full (Teljes) Állapot

A szinkronizáció befejeződött! Mindkét router LSDB-je (Térképe) hajszálpontosan megegyezik. A szomszédság (Adjacency) felépült. Innentől csak 10 másodpercenként Hello üzenetekkel tartják fenn a kapcsolatot, hogy ne járjon le a 40 másodperces Dead Timer.

## 3. Extra OSPF Konfigurációs Tippek

### OSPF indítása direkt az Interfészen (A "Network" parancs kikerülése)

Ahelyett, hogy a `router ospf 1` menüből Wildcard maszkokkal szenvednél a `network` paranccsal, bemehetsz közvetlenül az interfész (pl. `g0/0`) beállításai alá, és egyetlen sorral rákapcsolhatod a portot az OSPF-re és a megfelelő Területre:

```text
Router(config-if)# ip ospf 1 area 0
```

(Az `1`-es az OSPF Process ID, a `0`-ás az Area).

### Passive-Interface megfordítása

Ha egy routernek van 20 portja, és ebből 19 a PC-k felé megy, nem akarod 19-szer beírni a `passive-interface` parancsot. Megfordíthatod a logikát: Letiltod globálisan az összeset, és csak azt az 1 portot kapcsolod vissza, ahol tényleg másik router van!

```text
Router(config-router)# passive-interface default
Router(config-router)# no passive-interface g0/0
```

## Kvíz a 29. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod az OSPF állapotokról és a Költségszámításról!

### Kérdés 1

Állítsd a megfelelő sorrendbe az OSPF szomszédság kiépítésének 7 állapotát!

(A horgonyként használandó sorrend):

**Down → Init → 2-way → Exstart → Exchange → Loading → Full.**

### Kérdés 2

Melyik állítás **IGAZ** az OSPF alapértelmezett Költség (Cost) számítására vonatkozóan?

- A) Minden interfésznek azonos a költsége, függetlenül a sávszélességtől.
- B) A FastEthernet és az Ethernet portoknak azonos a költsége.
- C) A FastEthernet, a GigabitEthernet és a 10-GigabitEthernet portoknak azonos a költsége.
- D) Minden porttípusnak (Ethernet, Fast, Gigabit, 10-Gig) egyedi, eltérő költsége van.

**Helyes Válasz: C**

**Magyarázat:** Mivel a Cisco IOS alapértelmezett OSPF Referencia Sávszélessége 100 Mbps (tehát $100/X$), a 100 Mbps (FastEthernet) eredménye pontosan 1. Mivel a Gigabit és a 10-Gigabit esetében az eredmény 1-nél kisebb lenne (0.1 és 0.01), a rendszer ezeket erőszakosan 1-re kerekíti fel. Emiatt a C válasz a helyes: alapértelmezés szerint a router szemében a Fast, a Giga és a 10-Giga portok pontosan ugyanolyan gyorsak (Költség = 1). (Ezt a `auto-cost reference-bandwidth` paranccsal kell javítani).

### Kérdés 3

Melyik OSPF szomszédsági állapotban (State) dől el, hogy a két router közül melyik lesz a Master (Mester) és melyik a Slave (Szolga)?

- A) Exstart
- B) 2-way
- C) Exchange
- D) Loading

**Helyes Válasz: A**

**Magyarázat:** A Master és a Slave szerepköröket (amelyeket kizárólag a legmagasabb Router ID dönt el) az **Exstart** (Csere-előkészítés) fázisban beszélik meg. Ez a szereposztás arra kell, hogy a következő, Exchange (Csere) fázisban a Master kezdhesse meg a tartalomjegyzékek (DBD csomagok) kiküldését, rendszerezetten irányítva a forgalmat.

### Kérdés 4

Melyik paranccsal tudod a FastEthernet portod OSPF költségét (Metrikáját) pontosan 100-ra beállítani anélkül, hogy a port fizikai tulajdonságait piszkálnád?

- A) `auto-cost reference-bandwidth 100`
- B) `auto-cost reference-bandwidth 1000`
- C) `auto-cost reference-bandwidth 10000`
- D) `auto-cost reference-bandwidth 100000`

**Helyes Válasz: C**

**Magyarázat:** Képlet: Referencia Sávszélesség / Valós Sávszélesség = Költség. A FastEthernet valós sávszélessége 100 Mbps. Tehát $X / 100 = 100$. Az X (A referencia) értéke matematikai alapon **10000**. A parancs vége mindig Megabit/s-ben várja az értéket, így a 10000 a helyes (C).

### Kérdés 5

Mik az OSPF alapértelmezett Hello és Dead (Halott) időzítőinek (Timers) értékei egy normál Ethernet kapcsolaton? (A megadott értékek másodpercben értendők).

- A) Hello: 2, Dead: 20
- B) Hello: 10, Dead: 40
- C) Hello: 30, Dead: 120
- D) Hello: 60, Dead: 180

**Helyes Válasz: B**

**Magyarázat:** Ethernet és Point-to-Point hálózatokon a Hello üzeneteket 10 másodpercenként küldik. A Dead Timer (amikor a router halottnak nyilvánítja a szomszédot a Hello hiánya miatt) mindig a Hello **négyszerese**, azaz **40 másodperc**. A 30/120 (C válasz) bizonyos drága WAN technológiáknál használatos, amit a CCNA-n ritkábban kérnek.

### Bónusz Kérdés (Boson ExSim vizsgakérdés - Költségszámítás Topológián)

Képzeld el a hálózatot: RouterA csatlakozik RouterB-hez és RouterD-hez (mindkettőhöz Gigabit kábellel). RouterB csatlakozik RouterE-hez (Gigabit kábellel). RouterD csatlakozik RouterE-hez (FastEthernet kábellel). A célpont a RouterC, ami RouterE-hez csatlakozik (Gigabit kábellel).

Minden routeren kiadták az `auto-cost reference-bandwidth 1000` parancsot!

Mennyi lesz az összköltsége (Total Cost) a legrövidebb útnak a RouterA-tól a RouterC-ig?

- A) 11
- B) 3
- C) 12
- D) 2
- E) 20

**Helyes Válasz: B (3)**

**Magyarázat:** A trükk a `reference-bandwidth 1000` parancs. Így a Gigabit (1000) kábelek költsége: $1000/1000 = 1$. A FastEthernet (100) kábel költsége: $1000/100 = 10$.

A router megpróbálja kikerülni a drága FastEthernet kábelt, ha tudja. A legrövidebb út (RouterA → RouterB → RouterE → RouterC) tisztán Gigabit kábeleken halad. Minden Gigabit kábel költsége (csak a kifelé menőket számítva) 1. Összesen 3 ugrás van Gigabit kábeleken, így $1+1+1 = 3$. (Ha a RouterD felé menne a FastEthernet miatt, az út költsége $1+10+1 = 12$ lenne, amit az OSPF azonnal elvet).

# 1. Az IP Protokoll Szám (IP Protocol 89)

Amikor az OSPF üzeneteket küld, **nem használ TCP-t vagy UDP-t** (mint más alkalmazások). Az OSPF üzenetek egyenesen az IPv4 csomagba vannak ágyazva. Az IP fejléc "Protocol" mezőjében a **89-es szám** jelzi, hogy a csomag belsejében egy OSPF üzenet utazik! (A CCNA vizsgán néha szeretnek rákérdezni ilyen port/protokoll számokra).

# 2. Az 5 OSPF Üzenettípus Sorszámozása

Bár a jegyzetben felsoroltam a csomagtípusokat (Hello, DBD, LSR, LSU, LSAck), Jeremy mutatott egy táblázatot, ahol ezek szigorúan **1-től 5-ig** vannak számozva, ahogy a hálózati forgalomban (packet capture) is megjelennek:

- **Type 1:** Hello
- **Type 2:** DBD (Database Description)
- **Type 3:** LSR (Link State Request)
- **Type 4:** LSU (Link State Update)
- **Type 5:** LSAck (Link State Acknowledgment)

# 3. A Loopback Költség Matematikája a Valóságban

A jegyzetben szerepelt, hogy a Loopback interfészek költsége mindig **1**. Jeremy ezt egy konkrét példán is levezette a `show ip route` kimenetben: Ha az R1-es router el akarja érni az R2-es router `2.2.2.2`-es Loopback címét, és az odavezető kábel (pl. Gigabit, aminek a költségét felemeltük 100-ra) költsége 100, akkor a Routing Table-ben a teljes útvonal költsége **100 + 1 = 101** lesz.

# 4. A show ip ospf neighbor és a visszaszámlálás

Amikor kiadod a `show ip ospf neighbor` parancsot, látsz egy **"Dead Time"** oszlopot. Jeremy felhívta a figyelmet arra, hogy ez **nem egy fix 40 másodperces érték!** Valójában másodpercenként számol visszafelé (39, 38, 37...), és mivel 10 másodpercenként jön egy új Hello üzenet a szomszédtól, a számláló sosem megy 30 alá, mert a Hello beérkezésekor azonnal visszaugrik 40-re. Ha egyszer mégis 30 alá megy, az már intő jel, hogy a szomszéd késik az üzenettel.

Ezen kívül megemlítette a `show ip ospf interface brief` parancsot is, ami egy szép, tömör táblázatot ad az összes OSPF-es portunkról és a költségeikről.

Ezekkel az "utolsó csepp" mikrorészletekkel együtt immár kivétel nélkül minden egyes szava és képernyője le van fedve a 29. leckének (Day 27) is! Mehetünk tovább a következőre!
