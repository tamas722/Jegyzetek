# CCNA Bővített Jegyzet – 18. Lecke: VLAN-ok 3. Rész – Haladó Konfigurációk és a Layer 3 Switch

Ebben a leckében lezárjuk a VLAN-ok alapvető konfigurációs témakörét. Megnézzük a Native VLAN működését és finomhangolását (ROAS esetén és a Wiresharkon keresztül), valamint megismerkedünk a modern vállalati hálózatok Jolly Jokerével: a **Layer 3 Switch**-csel, amivel sokkal hatékonyabban lehet az Inter-VLAN routingot megoldani, mint egy külső routerrel!

## 1. A Native VLAN és a Router (ROAS Finomhangolás)

Az előző leckében (ROAS - Router on a Stick) megtanultuk, hogy a 802.1Q címkézést a router alinterfészein (Subinterfaces) végezzük. A **Native VLAN** (Natív VLAN) különlegessége az, hogy a hozzá tartozó keretek **címke nélkül (Untagged)** utaznak a kábelen. Ez gyorsabb (kevesebb bájtot kell feldolgozni), de biztonsági kockázatot is jelent, ezért általában egy használaton kívüli VLAN-ra állítják be.

Ha mégis használni szeretnéd a Native VLAN-t a router és a switch közötti forgalomra (pl. VLAN 10), kétféleképpen konfigurálhatod azt a routeren:

### 1. Módszer: Az alinterfész kiegészítése a `native` kulcsszóval

Ez a legtisztább módszer. Létrehozol egy alinterfészt (pl. `g0/0.10`), és az `encapsulation` parancs végére egyszerűen odaírod a `native` szót. Ezzel a router tudni fogja, hogy ami címke nélkül jön be a fizikai kábelen, az ehhez az alinterfészhez tartozik, és amit erről az alinterfészről küld ki, arról hagyja le a címkét.

```text
Router(config)# interface g0/0.10
Router(config-subif)# encapsulation dot1q 10 native
Router(config-subif)# ip address 192.168.1.62 255.255.255.192
```

### 2. Módszer: A fizikai interfész használata az alinterfész helyett

Mivel a fizikai interfészhez amúgy sem tartozik címke, egyszerűen beállíthatod a Native VLAN (VLAN 10) IP-címét magára a fizikai portra (pl. `g0/0`), míg a többi VLAN-hoz (20, 30) továbbra is létrehozod a megfelelő alinterfészeket (`g0/0.20`, `g0/0.30`). Ilyenkor a `g0/0`-on a dot1q encapsulation parancsra egyáltalán nincs is szükség!

```text
Router(config)# interface g0/0
Router(config-if)# ip address 192.168.1.62 255.255.255.192
Router(config-if)# no shutdown
```

## 2. A 802.1Q Tag (Címke) a Wiresharkban

Amikor Wiresharkban megvizsgálunk egy pinget (ICMP), ami mondjuk a VLAN 20-ból tart a Router felé a Trunk kábelen:

- Az Ethernet fejléc Source MAC és a Type (ami normál esetben jelzi az IPv4-et) közé beékelődik egy új **4 bájtos blokk**.
- Az új Type mező (a TPID) `0x8100` hexadecimális értéket mutat. Ez a standard kódja a 802.1Q (dot1q) VLAN-nak.
- Látható a prioritási kód (PCP) és a dobásjelző (DEI) mező (általában 0-n).
- Látható a legfontosabb mező: a **VLAN ID (VID)**, ami esetünkben 20-at mutat.
- A címke után következik az igazi IPv4 Type mező (a kódja `0x0800`).

Ha megvizsgáljuk ugyanennek a keretnek a visszaútját a Native VLAN 10-en:
Tegyük fel, hogy a ping válasz a Native VLAN-on (VLAN 10) érkezik vissza. Amikor a router kiküldi a keretet, a fenti dot1q blokk egyszerűen **hiányzik** a Wireshark felvételből. Címke (tag) nélkül jön át a kábelen, és a switch automatikusan a Native VLAN 10-be sorolja be.

## 3. A Multilayer (Layer 3) Switch bemutatása

A hagyományos (Layer 2) switchek nem értenek az IP-címekhez. Ha a VLAN 10-ben lévő géped akar beszélni a VLAN 20-szal, azt csak egy külső router (ROAS) bevonásával tudod megtenni. Ez hatalmas hálózatokban a "Ping-pong hatás" (egy be, egy ki a routeren) miatt durva torlódáshoz (bottleneck) vezethet.

A megoldás a modern vállalati hálózatokban a **Layer 3 Switch (Multilayer Switch)**.

- Kinézetre egy normál switch, de a "belsejében" egy villámgyors mini-router lapul.
- Megérti az IP-címeket, tud routing táblát építeni (és statikus/dinamikus routingot futtatni).
- Képes úgy Inter-VLAN routingot csinálni, hogy az adat el sem hagyja a fizikai dobozt!

### Kétféle módon tud Routingolni (L3-as módban működni)

#### 1. Routed Port (Fizikai Layer 3 port)

Bármelyik fizikai switchportot (pl. `g0/1`) "átalakíthatod" egy valódi router porttá. Onnantól kezdve nem tartozik semmilyen VLAN-hoz (a trunk vagy access parancsok megszűnnek működni rajta), és adhatsz neki saját IP-címet, amivel csatlakozhatsz pl. a külső Internet-routerhez.

```text
Switch(config)# interface g0/1
Switch(config-if)# no switchport
Switch(config-if)# ip address 192.168.1.193 255.255.255.252
```

#### 2. SVI (Switch Virtual Interface) – Logikai Layer 3 port

Ez a ROAS (Router on a Stick) modern helyettesítője! Ahelyett, hogy a switchet rákötnéd egy routerre, minden egyes VLAN-hoz létrehozol a switchen belül egy virtuális interfészt (SVI), ami IP-címet kap. Ezek a belső "szoftveres" portok lesznek a gépek Alapértelmezett Átjárói (Default Gateway), így a forgalmat maga a switch irányítja át a VLAN-ok között.

## 4. Cisco IOS Parancsok: L3 Switch és az SVI Konfigurálása

A legfontosabb, hogy be kell kapcsolni a routing képességet a switchen! E nélkül meg tudod csinálni az SVI-kat, de a gép nem fog adatot átirányítani közöttük!

```text
Switch(config)# ip routing
```

SVI létrehozása (pl. a VLAN 10-nek):
A parancs megegyezik magának a VLAN-nak az elnevezésével, de elé kell tenni az `interface` szót.

```text
Switch(config)# interface vlan 10
Switch(config-if)# ip address 192.168.1.62 255.255.255.192
Switch(config-if)# no shutdown
```

> (Ne feledd: Az SVI is alapból le van tiltva, kell a `no shutdown`! A normál VLAN-t a `vlan 10` paranccsal hozod létre, a virtuális interfészt az `interface vlan 10` paranccsal!)

### Hogyan lehet egy SVI állapota UP / UP (Működő)?

Ahhoz, hogy az `interface vlan 10` szoftveresen bekapcsoljon, négy kritikus feltételnek **EGYÜTTESEN** kell teljesülnie:

1. Az SVI-t engedélyezni kell (`no shutdown`).
2. Magának a VLAN-nak (L2 adatbázisban) fizikailag léteznie kell a switchen! (Nem hozza létre automatikusan, ha csak az SVI-t hozod létre!).
3. Magának a VLAN-nak aktívnak kell lennie (A `vlan config` menüből le lehet tiltani a `shutdown` paranccsal).
4. Lennie kell legalább egy olyan fizikai Access portnak, ami a VLAN 10-ben van és be van dugva rajta egy gép (UP/UP), **VAGY** lennie kell egy Trunk portnak, ami UP/UP állapotú és engedélyezi (átengedi) a VLAN 10-et! (Ha a hálózat teljesen üres, a virtuális interfész sem éled fel).

Ha mindent jól csináltál, a `show ip route` paranccsal egy teljes értékű routing táblát fogsz látni magán a switchen!

## Kvíz a 18. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod a L3 switchekről és a Native VLAN működéséről!

### Kérdés 1

Az alábbiak közül melyik **KÉT** lehetőség érvényes megoldás arra, hogy beállítsd a Native VLAN-t egy Cisco routeren ROAS (Router on a Stick) konfiguráció esetén? (Válassz KETTŐT!)

- A) A fizikai interfészen kiadod a `switchport trunk native vlan 10` parancsot.
- B) A subinterfészen (pl. `g0/0.10`) kiadod az `encapsulation dot1q 10 native` parancsot.
- C) Az IP-címet közvetlenül a fizikai interfészen (pl. `g0/0`) konfigurálod be, mindenféle encapsulation parancs nélkül.
- D) A subinterfészen kiadod a `switchport mode access` parancsot.

**Helyes Válasz: B és C**

**Magyarázat:** Az A és D parancsok Switcheken működnek (Layer 2), nem pedig routereken. A B a dedikált, tiszta megoldás, ahol az alinterfészen megjelölöd, hogy ő a natív (így nem kerül rá címke). A C a "trükkös" megoldás: mivel a fizikai kábelt (`g0/0`) alapból nem érdekli a VLAN és a címke, ha ezen adod ki az IP-címet, az automatikusan "untagged", azaz Native VLAN-ként fog működni a szemben lévő switch számára.

### Kérdés 2

Létrehozol egy SVI-t a VLAN 225-höz az SW1-en, beállítod az IP-címet, és kiadod a `no shutdown` parancsot. Ennek ellenére az interfész down/down állapotban marad. Melyik **KÉT** dolog okozhatta ezt az alábbiak közül? (Válassz KETTŐT!)

- A) A VLAN 225 fizikailag nem létezik a switchen.
- B) Nem adtad ki a `switchport mode trunk` parancsot az SVI-n.
- C) Nem adtad ki a `switchport access vlan 225` parancsot az SVI-n.
- D) Nincs egyetlen aktív (up/up) interfész sem a switchen, ami a VLAN 225-ben lenne (vagy átengedné azt trunként).

**Helyes Válasz: A és D**

**Magyarázat:** Az SVI (Switch Virtual Interface) egy szoftveres port, nem pedig egy fizikai dugalj. Ezért a B és C válaszok (amik fizikai portokat szabályoznak) érvénytelenek rá. Ahhoz, hogy a szoftveres SVI feléledjen (UP/UP), magának a VLAN adatbázisnak léteznie kell (A), és legalább egyetlen fizikai portnak a való világban élnie kell, amin át tud folyni a VLAN 225 forgalma (D).

### Kérdés 3

Melyik paranccsal lehet egy hagyományos Switch interfészt egy Layer 3 Routed Porttá (Router Porttá) alakítani?

- A) `no switchport`
- B) `ip address <ip> <maszk>`
- C) `ip routing`
- D) `switchport mode route`

**Helyes Válasz: A**

**Magyarázat:** A Layer 3 (Multilayer) switcheken a fizikai portok alapértelmezetten L2 (Switchport) módban vannak. A `no switchport` paranccsal mondod meg az adott interfésznek, hogy dobja el a kapcsoló (switch) viselkedést, és váljon egy "tiszta" IP porttá, amire ezután (a B válasznak megfelelően) be lehet állítani egy fix IP-címet. Az `ip routing` (C) az egész dobozon globálisan kapcsolja be az útválasztást.

### Bónusz Kérdés (Boson ExSim vizsgakérdés)

Kiadtad az alábbi parancsokat egy Catalyst switchen a FastEthernet 0/7 porton:

```text
interface fastEthernet 0/7
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 44
```

Az alábbi állítások közül melyik **IGAZ** a VLAN forgalomra, amikor az áthalad ezen a porton? Válassza ki a legjobb választ!

- A) A VLAN 1 forgalma untagged (címkézetlen) lesz.
- B) A VLAN 44 forgalma untagged (címkézetlen) lesz.
- C) Minden VLAN forgalma tagged (címkézett) lesz.
- D) Minden VLAN forgalma untagged (címkézetlen) lesz.

**Helyes Válasz: B**

**Magyarázat:** A konfiguráció egyértelműen kimondja: `switchport trunk native vlan 44`. A 802.1Q szabvány szerint a beállított Native VLAN az a hálózat, amelyiknek a forgalmát a switch címke (tag) nélkül tolja ki a trunk porton. Mivel felülírtuk a gyári alapértelmezést (A), most a VLAN 44 lesz az "untagged" forgalom. A többi átmenő VLAN (1-43, 45-4094) mind kap egy 4 bájtos dot1q címkét.
