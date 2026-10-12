# Teljes CCNA Tanfolyam – 2. Lecke: Interfészek és Kábelek (Interfaces and Cables)

Üdvözöllek a teljes CCNA tanfolyamon. Az előző leckében a hálózatokban található különböző eszközökről volt szó. Ebben a leckében arra fókuszálunk, hogyan kötjük össze őket – egészen pontosan a kábelekkel. (A vezeték nélküli kapcsolatokkal a kurzus egy későbbi pontján foglalkozunk majd.)

## Interfészek és az RJ-45 Csatlakozó

Ahogy az előző leckében is tanultuk, a switch-ek egyik fő jellemzője, hogy rengeteg interfésszel (más néven port-tal) rendelkeznek – gyakran 24 darabbal –, amelyekhez end host-okat (például PC-ket és szervereket) lehet csatlakoztatni. Ezeknél a portoknál gyakran találkozhatsz olyan feliratokkal, mint például "10/100/1000Base-T ports (1 – 24) – Ports are Auto-MDIX". Mire a lecke végére érünk, pontosan érteni fogod, mit jelent ez.

<img width="2048" height="1152" alt="1" src="https://github.com/user-attachments/assets/349d9f8c-b380-4109-a538-4466ee41690e" />


Ha a számítógéped vezetékes hálózathoz csatlakozik, szinte biztosan RJ-45-ös portot használ. Az RJ rövidítés a Registered Jack kifejezést takarja. Az RJ-45 csatlakozó a réz alapú Ethernet kábelek végén található. Bár dizájnban és színben eltérhetnek, a csatlakozók szabványosak.

## Mi az az Ethernet?

Az Ethernet nem csupán egyetlen protokoll, hanem hálózati protokollok és szabványok gyűjteménye. Hogy miért van szükség szabványokra?

Ha két ember próbál beszélgetni, és az egyik csak angolul, a másik csak japánul tud, nem lesz köztük kommunikáció. Szükségük van egy közös nyelvre. A hálózati protokollok pontosan ezt a célt szolgálják az eszközök között. Ha a kábel gyártója és a switch gyártója nem állapodik meg a csatlakozó és a port méretében, formájában (fizikai szabványok), vagy az adatok küldésének módjában (logikai szabványok, mint az IP), az eszközök nem tudnának együttműködni.

## Bitek és Bájtok

A hálózati sebességet bit/másodperc (bps) mértékegységben mérjük.

- **Bit:** A számítógépek bináris kóddal működnek (0 és 1). A rézkábelen áthaladó elektromos jel változását a fogadó eszköz 0-ként vagy 1-ként (azaz egy bitként) értelmezi.
- **Byte (Bájt):** 8 bit tesz ki 1 bájtot.

Amikor adat utazik a kábelen, az bitenként, egymás után érkezik meg, nem pedig egyben, bájtonként. Ezért a hálózati sebességet mindig bitekben (kilobit, megabit, gigabit) adjuk meg. Fontos különbség: a merevlemezek tárhelyét bájtokban mérik. Egy Gigabájt (GB) tehát 8-szor nagyobb adatmennyiséget jelent, mint egy Gigabit (Gb).

(Méretezés: 1 kilobit = 1,000 bit, 1 megabit = 1 millió bit, 1 gigabit = 1 milliárd bit, 1 terabit = 1 trillió bit).

## Ethernet Szabványok (IEEE 802.3)

Az Ethernet kábelek szabványait az IEEE (Institute of Electrical and Electronics Engineers) 802.3-as szabványa határozza meg. Néhány fontosabb rézkábel szabvány:

| Sebesség | Közismert név | Informális szabványnév | Max. kábelhossz |
|----------|---------------|------------------------|-----------------|
| 10 Mbps | Ethernet | 10BASE-T | 100 méter |
| 100 Mbps | Fast Ethernet | 100BASE-T | 100 méter |
| 1 Gbps | Gigabit Ethernet | 1000BASE-T | 100 méter |
| 10 Gbps | 10 Gig Ethernet | 10GBASE-T | 100 méter |

A "Base" a baseband (alapsávi) jelátvitelt jelenti, a "T" pedig a Twisted Pair (csavart érpár) kábelezésre utal.

## UTP Kábelek (Unshielded Twisted Pair)

A réz Ethernet szabványokban használt kábelek az úgynevezett UTP (Unshielded Twisted Pair – Árnyékolatlan csavart érpár) kábelek.

- **Unshielded (árnyékolatlan):** A vezetékeknek nincs fém árnyékolásuk.
- **Twisted pair (csavart érpár):** A kábelben 4 pár (összesen 8) vezeték található, amelyek páronként össze vannak csavarva. A csavarás segít kivédeni az elektromágneses interferenciát (EMI).

<img width="2048" height="1152" alt="2" src="https://github.com/user-attachments/assets/f662cf0e-c9f5-4cc1-aef7-6864c45236d4" />


Bár az RJ-45 csatlakozónak 8 pin-je (érintkezője) van, és a kábel is 8 vezetéket tartalmaz, a régebbi szabványok nem használták mindet:

- A 10BASE-T és a 100BASE-T csak 2 párt (4 vezetéket) használ.
- Az 1000BASE-T és a 10GBASE-T mind a 4 párt (8 vezetéket) használja.

## Hogyan működik a 10BASE-T / 100BASE-T? (Pin kiosztások)

Ezeknél a kapcsolatoknál csak az 1-es és 2-es, illetve a 3-as és 6-os pin-eket használják. A hálózati kommunikáció Full-Duplex, ami azt jelenti, hogy az eszközök egyszerre tudnak adatot küldeni és fogadni, mert külön vezetékpárokat használnak erre a célra (így elkerülik az ütközéseket).

- **PC / Router / Firewall (Tűzfal):** Ezek az eszközök az 1-es és 2-es pin-en küldik (Transmit - Tx) az adatot, és a 3-as és 6-os pin-en fogadják (Receive - Rx) azt.
- **Switch:** A switch épp fordítva működik: a 3-as és 6-os pin-en küldi az adatot, és az 1-es és 2-es pin-en fogadja.

## Straight-through és Crossover kábelek

Mivel a PC-k (küldenek 1,2-n) és a switch-ek (fogadnak 1,2-n) ellentétes pineket használnak, összeköthetők egy úgynevezett straight-through (egyeneskötésű) kábellel. Ennél a kábelnél az egyik vég 1-es pin-je a másik vég 1-es pin-jéhez csatlakozik, a 2-es a 2-eshez, és így tovább.

De mi történik, ha két azonos típusú eszközt (pl. Routert Routerrel, Switchet Switchcsel, vagy PC-t PC-vel) kötünk össze? Mivel mindkettő ugyanazon a pinen próbálna adatot küldeni, a kommunikáció meghiúsulna.

Erre a problémára a megoldás a crossover (keresztkötésű) kábel. Ennél a vezetékezés "keresztezi" egymást a két végpont között: az 1-es pin a 3-asba, a 2-es a 6-osba fut (és fordítva). Így az egyik eszköz adó (Tx) csatornája fizikailag a másik eszköz fogadó (Rx) csatornájához kötődik.

## Auto MDI-X

A modern hálózati eszközök fejlődésével a straight-through és crossover kábelek problémája jórészt megoldódott. Az újabb eszközök támogatják az Auto MDI-X funkciót, amely automatikusan érzékeli, hogy a szomszédos eszköz melyik pin-eken küld adatot, és ehhez igazítja a saját küldő/fogadó portjait. Ma már csak nagyon régi hardverek esetén kell aggódni a megfelelő kábeltípus miatt.

(Megjegyzés a gigabites kábelekhez: Az 1000BASE-T és 10GBASE-T szabványok mind a 8 vezetéket használják, ráadásul minden érpár kétirányú (bidirectional), azaz nem különülnek el dedikált küldő és fogadó párokra. Ez teszi lehetővé a sokkal nagyobb sebességet.)

## Száloptikás (Fiber-Optic) Kapcsolatok

A réz UTP kábelek olcsók és kiválóak egy LAN-on belül, de a maximum 100 méteres távolságkorlátjuk problémás nagyobb hálózatok esetén. Itt jön képbe a Fiber-optic (száloptika).

Néhány switch-en és router-en az RJ-45 portok mellett találhatók üres, fém keretes foglalatok is. Ezekbe egy úgynevezett SFP (Small Form-factor Pluggable) transceiver modult kell csatlakoztatni, amelybe utána már be lehet dugni a száloptikás kábelt.

<img width="2048" height="1368" alt="3" src="https://github.com/user-attachments/assets/16b5dd85-aabc-4f61-954c-3c4cf320f8bd" />


Ezek a kábelek nem rézvezetéken, elektromos jelekkel továbbítják az adatot, hanem üvegszálakon keresztül, fénnyel. Minden száloptikás kapcsolat két kábelt igényel: egyet az adatküldésre (transmit) és egyet az adatfogadásra (receive).

<img width="2048" height="1304" alt="4" src="https://github.com/user-attachments/assets/75da7300-8ae4-49b7-b0ae-5873860886bb" />


A kábel szerkezete belülről kifelé:

- **Üvegszál mag (Core):** Ezen halad a fény.
- **Cladding (Köpeny/Burkolat):** Visszaveri a fényt a magba.
- **Buffer:** Védőréteg, ami megóvja az üveget a töréstől.
- **Külső burkolat (Jacket).**

## Multimode vs. Single-mode Fiber

- **Multimode fiber (Többmódusú szál):** A mag (core) szélesebb. A fény (általában LED-es adóból) több szögben (módban) léphet be és verődhet vissza a kábel faláról. Olcsóbb megoldás, ami nagyobb távolságot bír a réznél, de rövidebbet, mint a single-mode.
- **Single-mode fiber (Egymódusú szál):** A mag sokkal vékonyabb. A fényt egy drága lézeres adó küldi, egyetlen szögben (módban), egyenesen a kábel közepén. Ez a legdrágább technológia, de ezzel hidalható át a legnagyobb távolság.

(Fontosabb optikai szabványok: 1000BASE-LX (1 Gbps, 550m multimode vagy 5km single-mode), 10GBASE-SR (10 Gbps, max 400m multimode), 10GBASE-LR (10 Gbps, max 10km single-mode), 10GBASE-ER (10 Gbps, max 30km single-mode).)

## UTP vs. Fiber-Optic Összegzés

| Szempont | UTP (Réz) | Fiber-Optic (Száloptika) |
|----------|-----------|--------------------------|
| Költség | Olcsóbb kábelek és RJ-45 portok. | Drágább kábelek, drága SFP portok. (Single-mode a legdrágább). |
| Távolság | Rövid, maximum 100 méter. | Hosszú távolságok, kilométerek is áthidalhatók. |
| Biztonság és EMI | Érzékeny az EMI-re, a kábel apró jelszivárgása miatt elméleti biztonsági kockázat. | Nem szivárog jel a kábelből (teljesen biztonságos), teljesen immunis az EMI-re. |

## Kvíz (Tudáspróba)

Gondold végig a kérdéseket, mielőtt elolvasod a megoldásokat!

### 1. Kérdés

Két régi router-t kötsz össze egy UTP kábellel, de az adatátvitel nem működik. Mi lehet a probléma?

- A) Straight-through kábellel vannak összekötve.
- B) Crossover kábellel vannak összekötve.
- C) Auto MDI-X módban működnek.

**Helyes válasz: A**

**Magyarázat:** Két azonos (ráadásul régi, Auto MDI-X nélküli) eszköz, mint két router összekötéséhez crossover kábelre van szükség, mivel mindkettő az 1-es és 2-es pineken küldi az adatot. Ha egyeneskötésű (straight-through) kábelt használsz, a kommunikáció meghiúsul.

### 2. Kérdés

A céged két különálló épületben lévő switch-et akar összekötni, amelyek 150 méterre vannak egymástól. A költségeket alacsonyan akarják tartani. Milyen kábelt használjanak?

- A) UTP
- B) Single-mode fiber
- C) Multimode fiber

**Helyes válasz: C**

**Magyarázat:** Az UTP kábel hossza maximum 100 méter lehet, így az kiesik. Bár a single-mode fiber gond nélkül elbírja a 150 métert, jóval drágább megoldás. A multimode fiber bírja a távolságot és költséghatékonyabb a single-mode-nál.

### 3. Kérdés

A céged két irodát akar összekötni, amelyek kb. 3 kilométerre vannak egymástól. Milyen kábelt használjanak (a lehető legköltséghatékonyabban)?

- A) UTP
- B) Single-mode fiber
- C) Multimode fiber

**Helyes válasz: B**

**Magyarázat:** Sem az UTP, sem a multimode fiber nem támogat ekkora távolságot. Egyedül a single-mode fiber alkalmas arra, hogy több kilométeres távolságokat is áthidaljon.

### 4. Kérdés

Egy switch portjain szerepel az Auto MDI-X felirat. Mi történik, ha egy teljesen azonos switch-csel kötöd össze egy straight-through kábel segítségével?

- A) Normálisan fognak működni.
- B) Csökkentett sebességgel fognak működni.
- C) Nem fognak tudni kommunikálni.

**Helyes válasz: A**

**Magyarázat:** Az Auto MDI-X technológia felismeri az ellenoldali eszköz pin-kiosztását, és automatikusan korrigálja a beállításokat. Így hiába azonos típusú eszközök és hiába straight-through a kábel, a kapcsolat hibátlanul fog működni.

### 5. Kérdés

A céged rengeteg end host-ot szeretne rákötni egy switch-re, ami ugyanazon az emeleten egy kapcsolószekrényben található. Milyen kábelt használjanak?

- A) UTP
- B) Single-mode fiber
- C) Multimode fiber

**Helyes válasz: A**

**Magyarázat:** A legtöbb end host (PC) RJ-45 hálózati kártyával rendelkezik, és a switch-eken is rengeteg ilyen port van dedikálva erre a célra. Helyi, irodai összeköttetésre az UTP kábel a sztenderd.
