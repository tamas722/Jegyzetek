# CCNA Bővített Jegyzet – 38. Lecke: Layer 2 Felismerő Protokollok (CDP és LLDP)

Ebben a leckében a Hálózatfelderítő protokollokat (Layer 2 Discovery Protocols) vizsgáljuk meg. Ezek a protokollok lehetővé teszik a hálózati eszközök (routerek, switchek) számára, hogy "bemutatkozzanak" a közvetlen szomszédjaiknak, és megtudják, milyen eszköz van a kábel másik végén, még azelőtt, hogy IP-címeket kapnának! (A CCNA vizsgán a CDP és az LLDP működésének és parancsainak összehasonlítását kérik számon).

## 1. A Layer 2 Felismerő Protokollok Alapjai

Ezek a protokollok szigorúan a **2. rétegben (Datalink Layer)** működnek. Ahogy a Wireshark elemzésből is láttuk, az Ethernet keretükben **NINCS IP-csomag** (IP Header), kizárólag a MAC-címek alapján kommunikálnak.

### Alapvető Működés és Biztonsági Kockázat

A gépek bizonyos időközönként Multicast üzeneteket küldözgetnek a kábelen. Az üzenetet megkapó eszköz (pl. a szomszéd switch) elolvassa, berakja a saját szomszédtáblájába, **DE SOHA NEM TOVÁBBÍTJA** a következő gépnek! Ezért csakis a **Közvetlenül csatlakozó (Directly Connected)** gépek láthatják egymást.

Mivel ezek az üzenetek rengeteg érzékeny adatot tartalmaznak a hálózatról (Pl. az eszköz pontos modellje, az IOS verziószáma, az IP-címe), a hackerek imádják őket, így biztonsági okokból (Security Risk) sok hálózatban a rendszergazdák teljesen letiltják a használatukat.

## 2. CDP (Cisco Discovery Protocol)

A CDP a **Cisco saját (Proprietary)** protokollja. Minden Cisco eszközön (Router, Switch, IP Telefon, ASA Firewall) gyárilag (alapértelmezésben) **be van kapcsolva**! Két verziója van (V1 és V2), modern eszközökön mindig a V2 fut (A V2 olyan extrákat is tud, mint pl. figyelmeztetni, ha Native VLAN vagy Duplex hiba van a kábel két vége között).

### A CDP Értékei (Vizsgára Memorizálandó)

- **Multicast MAC Cím:** `0100.0ccc.cccc`
- **Message Timer (Üzenetküldés):** 60 másodpercenként.
- **Holdtime (Várakozási idő):** 180 másodperc. (Ha 180 mp-ig nem jön új üzenet a szomszédtól, a router törli a táblázatból).

### 2.1. CDP Konfiguráció és Verifikáció (CLI)

**A CDP be/kikapcsolása:**

- Globálisan a teljes gépen: `cdp run` (Be) / `no cdp run` (Ki).
- Csak egy adott interfészen: `cdp enable` (Be) / `no cdp enable` (Ki).

(A Timer-t a `cdp timer 60`, a Holdtime-ot a `cdp holdtime 180` parancsokkal lehet megváltoztatni globális módban).

**A legfontosabb SHOW parancsok:**

- `show cdp`: Megmutatja a beállított Timert és Holdtime-ot (valamint, hogy megy-e a V2).
- `show cdp traffic`: Statisztika arról, hány csomagot küldött és kapott.
- `show cdp interface`: Megmutatja portonként, hogy melyiken megy a CDP, milyen Timerekkel, és kiírja az Ethernet tokozás (Encapsulation) típusát is, ami az ARPA (Ethernet II).
- `show cdp neighbors`: A leggyakrabban használt parancs! Listázza a szomszédokat.
- `show cdp neighbors detail`: **Kritikus különbség!** Csak ezzel a bővített paranccsal láthatod a szomszéd IP-címét, a rajta futó IOS operációs rendszer verzióját, a VTP Domainjét, és a Native VLAN-t! (A Boson kvíz is ezt kérdezi: A sima "neighbors" parancsban az IP-cím és az IOS verzió még rejtve van).

### 2.2. A Szomszédtábla Olvasása (A `show cdp neighbors` oszlopai)

Amikor kiadod a parancsot, a táblázat fejlécei a következők:

- **Device ID:** A szomszéd Hostname-je (pl. SW1).
- **Local Intrfce (Helyi Interfész):** A **SAJÁT** routereden lévő port (pl. Gig 0/0), amibe a kábel be van dugva!
- **Holdtme:** A 180-as számláló, ahogy épp számol vissza.
- **Capability (Képességek):** Betűkódok! `R` = Router, `S` = Switch. (Ha a szomszéd egy Multilayer Switch, ami képes IP routingra, akkor az `R` és az `S` betű is ott lesz!). A `B` betű a "Source Route Bridge"-et jelzi.
- **Platform:** A szomszéd pontos hardvermodellje (pl. C2960 switch).
- **Port ID:** A **SZOMSZÉD** gép portja, amibe a kábel a túloldalon csatlakozik. (Vigyázz a Local Interface és a Port ID különbségére!).

## 3. LLDP (Link Layer Discovery Protocol)

Mivel a CDP csak Cisco gépeken működik, a szakma megalkotta a Nyílt Iparági Szabványt (**IEEE 802.1AB**), ez az LLDP. Ha a hálózatodban Juniper, HP vagy Palo Alto eszközök is vannak, az LLDP az egyetlen mód a szomszédok feltérképezésére.

Cisco eszközökön képes párhuzamosan futni a CDP-vel, de **gyárilag az LLDP ki van kapcsolva!** A rendszergazdának manuálisan kell felélesztenie.

### Az LLDP Értékei (Vizsgára Memorizálandó)

- **Multicast MAC Cím:** `0180.C200.000E`
- **Message Timer (Üzenetküldés):** 30 másodpercenként (Kétszer olyan gyors, mint a CDP).
- **Holdtime (Várakozási idő):** 120 másodperc.
- **Reinitialization Delay (Újra-inicializációs késleltetés):** 2 másodperc. (Ha az LLDP-t hirtelen lekapcsolod, majd visszakapcsolod (flapping), a gép vár 2 másodpercet a kiküldés előtt, védve a processzort).

### 3.1. LLDP Konfiguráció (Kritikus eltérés a CDP-től!)

Az LLDP bekapcsolása interfész szinten máshogy működik, mint a CDP! A CDP egyetlen paranccsal (`cdp enable`) adta és vette is a csomagokat. Az LLDP-nél ezt kettészedték:

- Globálisan a teljes gépen: `lldp run`
- Interfészen befelé (Vétel): `lldp receive`
- Interfészen kifelé (Küldés): `lldp transmit`

(Ahhoz, hogy az LLDP rendesen működjön a porton, mind a `receive`, mind a `transmit` parancsot ki kell adni! A timereket az `lldp timer 30`, `lldp holdtime 120`, `lldp reinit 2` parancsokkal lehet módosítani).

### 3.2. Az LLDP Szomszédtábla és Képesség-kódok

A parancsok szinte ugyanazok: `show lldp`, `show lldp traffic`, `show lldp neighbors`.

Viszont van egy nagyon fontos különbség a "Capability" (Képességek) betűkódoknál az LLDP esetében!
Míg a CDP a Switch-et az `S` betűvel jelölte, az **LLDP-ben NINCS 'S' BETŰ!** Helyette az Iparág a **'B' (Bridge)** betűkódot használja a Switchekre! (Ha a szomszéd Multilayer Switch, akkor a `B` és az `R` betű fog világítani).

További eltérés a `show lldp neighbors detail` kimenetben: A gép külön listázza a "System Capabilities" (Mire képes a hardver fizikailag, pl. B,R) és az "Enabled Capabilities" (Melyik funkció van aktuálisan bekapcsolva a konfigurációban, pl. csak R) mezőket. Ezen felül az LLDP **nem tudja megmutatni** a szomszéd VTP (VLAN Trunking Protocol) adatait, mert a VTP egy zárt Cisco protokoll, amit csak a CDP ért.

## Kvíz a 38. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod a CDP és az LLDP parancsaival és különbségeivel kapcsolatban!

### Kérdés 1

Az alábbi parancsok közül melyik **KETTŐ** mutatja meg neked az aktuálisan beállított CDP Időzítőket (Timers)? (Válassz KETTŐT!)

- A) `show cdp`
- B) `show cdp traffic`
- C) `show cdp interface`
- D) `show cdp neighbors`

**Helyes Válasz: A és C**

**Magyarázat:** A `show cdp` megmutatja a gép globális időzítőit, a `show cdp interface` pedig az adott porton érvényben lévő (pl. 60 sec, 180 sec) időzítőket. A `traffic` csak a csomagok számát (statisztikát) mutatja, a `neighbors` pedig a Holdtime-ot ugyan mutatja, ahogy visszaszámol, de az nem a fixen "beállított" konfigurációs érték, és a Message Timer (60s) nem is látszik benne.

### Kérdés 2

Melyik **KÉT** parancs képviseli a CDP gyári alapbeállításait (Default State) a Cisco routereken? (Válassz KETTŐT!)

- A) `no cdp run` (Globálisan)
- B) `cdp holdtime 120`
- C) `cdp enable` (Interfész módban)
- D) `cdp timer 60`

**Helyes Válasz: C és D**

**Magyarázat:** A Cisco eszközökön a CDP gyárilag be van kapcsolva az interfészeken (`cdp enable`), a kiküldési időzítő pedig alapból 60 másodperc (`cdp timer 60`). A Holdtime **180**, nem 120 (az az LLDP), a globális alapállapot pedig a `cdp run`, nem a `no cdp run`.

### Kérdés 3

Kiadod a `show lldp entry SW1` parancsot az R1-es routeren. A szomszédos SW1 egy Multilayer Switch. Milyen betűkódokat vársz a "System capabilities" (Rendszer képességek) mezőben az LLDP kimenetében?

- A) System Capabilities: S, R
- B) System Capabilities: B, R
- C) System Capabilities: S, R, I
- D) System Capabilities: B

**Helyes Válasz: B**

**Magyarázat:** Az LLDP iparági szabvány **nem ismeri az 'S' (Switch) betűkódot!** Ők a switchekre a **'B' (Bridge)** betűt használják. Mivel a gép Multilayer Switch (Képes IP Routingra is), így a Bridge (`B`) és a Router (`R`) képesség is világítani fog a kimenetben.

### Kérdés 4

Az alábbi állítások közül melyik **KETTŐ igaz** a Link Layer Discovery Protocol (LLDP) működésére? (Válassz KETTŐT!)

- A) Az LLDP egy Cisco-proprietary (Cisco saját) protokoll.
- B) Az LLDP-t az interfészen befelé (Receive) és kifelé (Transmit) külön-külön is engedélyezni kell.
- C) Az LLDP alapértelmezett üzenetküldési (Message) időzítője 60 másodperc.
- D) Az LLDP képes lekérdezni a szomszédos Cisco eszközök OSPF beállításait.
- E) Az LLDP képes lekérdezni a szomszédos Cisco eszközök VTP (VLAN Trunking Protocol) beállításait.
- F) Az LLDP képes lekérdezni a szomszédos eszközök operációs rendszerének (OS) verzióját.

**Helyes Válasz: B és F**

**Magyarázat:** Szemben a CDP `cdp enable` alapparancsával, az LLDP-t külön kell aktiválni a `lldp transmit` és `lldp receive` parancsokkal a porton (B). A Detail paranccsal meg tudja mondani az OS verziót (F). Az LLDP egy nyílt szabvány (Az A hamis), az időzítője 30 másodperc (A C hamis), és sosem tud Cisco specifikus VTP dolgokat lekérdezni, csak a CDP tud ilyet (E hamis). OSPF beállításokat pedig egyik sem tud lekérdezni (D hamis).

### Kérdés 5

Nézd meg a következő `show cdp neighbors` kimenetet: Az R2-es gépen vagy. Melyik **INTERFÉSZRE** dugták be a kábelt az R2-es routeren, ami a szomszédos SW2 switch felé megy? (A Képen látható oszlopok: Device ID: SW2 | Local Intrfce: Gig 0/1 | Holdtme: 140 | Capability: S I | Platform: C2960 | Port ID: Gig 0/3)

- A) G0/0
- B) G0/1
- C) G0/2
- D) G0/3

**Helyes Válasz: B**

**Magyarázat:** A kérdés a **SAJÁT** géped (R2) fizikai kábelcsatlakozására kérdez rá. Ezt az eredménytábla **Local Intrfce (Helyi Interfész)** oszlopa mutatja meg, ami a Gig 0/1. A sor legvégén lévő "Port ID" (Gig 0/3) azt a portot jelzi a **túloldalon**, ahová a szomszéd SW2-es switchben belement a kábel! Erre a különbségtételre nagyon figyelni kell a vizsgán.

### Bónusz Kérdés (Boson ExSim vizsgakérdés - A "Detail" trükk)

Kiadod a `show cdp neighbors` parancsot a routereden. Az alábbi 6 információ közül melyik **NÉGYET** fogsz ténylegesen látni a parancs kimenetében (anélkül, hogy más parancsot használnál)? (Válassz NÉGYET!)

- A) A szomszédos eszköz IP címe.
- B) A szomszédos eszközön lévő (a mi gépünkhöz csatlakozó) interfész azonosítója.
- C) A szomszédos eszköz "Device ID"-je (Hostname-je).
- D) A szomszédos eszközön futó szoftver (IOS) verziója.
- E) A helyi routereden lévő interfész azonosítója (Amelyikhez a szomszéd csatlakozik).
- F) A szomszédos eszköz képességei (Capability) és termékszáma (Platform).

**Helyes Válasz: B, C, E és F**

**Magyarázat:** A `show cdp neighbors` (az alap parancs) egy gyors, tömör táblázatot ad. Ahogy a leckében Jeremy többször is kiemelte, a sima parancs **SOHA nem mutatja meg az IP-címet (A) és a Szoftver (IOS) verziót (D)!** Ahhoz, hogy azokat az információkat is kikényszerítsd a rendszerből, hozzá kell adnod a szót a parancs végéhez: `show cdp neighbors detail`. Így maradt meg a négy helyes, táblázatban is látható oszlop: a túloldali port (B), a szomszéd neve (C), a mi portunk (E), és a modellszám/képességek (F).
