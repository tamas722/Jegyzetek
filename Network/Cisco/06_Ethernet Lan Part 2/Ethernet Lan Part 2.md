# 6. Lecke: Ethernet LAN Switching (2. Rész) és az ARP

Ebben a leckében még mélyebbre ásunk abban, hogyan kommunikálnak a számítógépek egy helyi hálózaton (LAN) belül. Megnézzük a hálózati "csomagok" méretbeli szabályait, megismerünk egy létfontosságú hálózati "nyomozó" protokollt (az ARP-t), és megtanuljuk, hogyan ellenőrizhetjük a kapcsolatot a ping paranccsal.

## 1. Az Ethernet Keret (Frame) méretei és a "Kitöltés" (Padding)

Az előző leckében megismertük az Ethernet keret mezőit. Most nézzük meg ezek méretszabályait!

A **Preamble (Preambulum)** és az **SFD (Start Frame Delimiter)** technikailag minden Ethernet keret elején ott van, de a hálózati szakemberek nem számítják bele a klasszikus "Ethernet Fejléc" méretébe.

**A Fejléc (Header) és az Utótag (Trailer) mérete:** A fejléc (Cél MAC, Forrás MAC, Típus) összesen 14 bájt. Az utótag (FCS a hibajavításhoz) 4 bájt. Ez összesen 18 bájt.

**A minimális keretméret:** Egy hálózati keretnek kötelezően el kell érnie egy minimum méretet, ami 64 bájt (ebben nincs benne a Preamble és az SFD).

**A minimális hasznos teher (Payload):** Ha a teljes keretnek minimum 64 bájtnak kell lennie, és ebből 18 bájt a "boríték" (fejléc+utótag), akkor magának a levélnek (a hasznos adatnak) minimum 46 bájtnak kell lennie (64 - 18 = 46).

### Mi a megoldás, ha túl kicsi az adat? (A Padding)

Tegyük fel, hogy a géped csak egy apró, 34 bájtos üzenetet akar küldeni. Mivel ez nem éri el a 46 bájtos minimumot, a hálózati kártya automatikusan kiegészíti azt kitöltő bájtokkal (**Padding**). A kitöltéshez csupa nullákat használ (hexadecimális `00`). A mi példánkban hozzáad 12 bájtnyi nullát (34 + 12 = 46), így a keret már szabályos lesz, és a switch képes lesz feldolgozni.

## 2. IP-címek vs. MAC-címek: A kommunikációs szakadék

Hogy a számítógépek megértsék egymást, kétféle címzésre is szükség van:

- **IP-cím (Layer 3):** Olyan, mint egy postacím (pl. `192.168.1.1`). Ezt használják a programok, és ezt írod be te is, ha el akarsz érni egy gépet.
- **MAC-cím (Layer 2):** Olyan, mint a számítógép személyigazolvány-száma (pl. `0C2F.B011.9D00`). A switchek kizárólag ezt értik. (Ne feledd: az első fele az OUI, ami a gyártót, pl. a Ciscót jelöli, a második fele pedig magát az eszközt).

**A probléma:** Amikor a PC1 adatot akar küldeni a PC3-nak, te megadod a PC3 IP-címét (`192.168.1.3`). A gép be is csomagolja ezt az IP csomagba. De ahhoz, hogy a kábelen keresztül elküldje a switchnek, rá kell írnia a borítékra (az Ethernet keretre) a PC3 MAC-címét is. De a PC1 honnan tudná egy másik gép MAC-címét, ha csak az IP-címét ismeri?

Erre a problémára találta fel az informatika az ARP-t!

## 3. Az ARP (Address Resolution Protocol) működése

Az ARP egy hálózati nyomozó protokoll. Az a feladata, hogy megkeresse a hiányzó MAC-címet egy ismert IP-címhez. A folyamat két lépésből (üzenetből) áll:

### 1. Az ARP Request (Kérés) – A hálózati "kiáltás"

Mivel a PC1 nem tudja, kinek kell küldeni a kérdést (hiszen pont a MAC-címet nem ismeri), a kérdést mindenkinek felteszi a helyi hálózaton. Ezt hívják Broadcast (üzennetszóró) keretnek.

### ARP Request – A csomag címei

| Mező | Érték | Jelentés |
|---|---|---|
| **Küldő IP** | PC1 | Saját IP-je |
| **Cél IP** | PC3 | Keresett IP-je |
| **Küldő MAC** | PC1 | Saját MAC-címe |
| **Cél MAC** | `FFFF.FFFF.FFFF` | Speciális broadcast MAC-cím: "Mindenki a hálózaton!" |

### ARP Request – Mi történik a hálózaton?

- A keret Cél MAC-címe egy speciális cím: `FFFF.FFFF.FFFF`. Ez a switchek nyelvén azt jelenti: "Ezt mindenkinek küldd el!"
- A csomag belsejében ez az üzenet áll: "Kihez tartozik a `192.168.1.3`-as IP-cím? Mondja meg nekem a `192.168.1.1`-nek!"
- A switch megkapja, megtanulja belőle PC1 címét (ezt láttuk az 5. leckében), majd a csupa F miatt elárasztja (flood) vele az egész hálózatot.
- A többi gép (pl. PC2, PC4) megkapja, de látják, hogy nem az ő IP-címüket keresik, ezért eldobják a kérést.

### 2. Az ARP Reply (Válasz) – A célzott felelet

A PC3 megkapja a kérést, és felismeri a saját IP-címét! Összeállítja a választ.

### ARP Reply – A csomag címei

| Mező | Érték | Jelentés |
|---|---|---|
| **Küldő IP** | PC3 | Saját IP-je |
| **Cél IP** | PC1 | A kérdező IP-je |
| **Küldő MAC** | PC3 | Végre elküldi a keresett címet! |
| **Cél MAC** | PC1 | A kérdező MAC-címe (már ismert az ARP kérésből) |

### ARP Reply – Mi történik a hálózaton?

- Mivel az ARP kérésben benne volt PC1 MAC-címe, PC3-nak már nem kell kiabálnia. Egy célzott **Unicast** keretet küld egyenesen PC1-nek.
- Az üzenet tartalma: "A `192.168.1.3`-as IP-cím az enyém, és a MAC-címem a következő..."
- A switch ezt már nem árasztja el, hiszen ismeri PC1 helyét, egyszerűen továbbítja (forward) a választ.

### Az ARP Tábla (ARP Table)

Amikor PC1 megkapja a választ, felírja magának ezt a párosítást (IP-cím = MAC-cím) a belső memóriájába, amit **ARP Táblának** hívunk. (Cisco eszközökön ezt a `show arp`, Windows/Mac/Linux gépeken a parancssorban az `arp -a` paranccsal lehet megnézni. A táblában a "dynamic" felirat jelzi, hogy az ARP segítségével tanulta meg a gép).

## 4. A PING, az ICMP és a hálózati elemzés (Wireshark)

### A Ping parancs

A **ping** egy apró segédprogram, amivel ellenőrizzük, hogy egy másik gép elérhető-e (reachability), és mennyi idő alatt ér oda-vissza egy csomag (round-trip time).

- Az **ICMP** (Internet Control Message Protocol) nevű protokollt használja.
- Két üzenete van: **Echo Request** (Ping kérés) és **Echo Reply** (Ping válasz).
- A Cisco IOS alapértelmezetten 5 darab, egyenként 100 bájtos pinget küld.

Példa a Cisco IOS-ben:

```text
Router# ping 192.168.1.3
```

### Miért vall kudarcot az első ping?

Ha megnézed a ping eredményét, gyakran ilyet látsz: `.!!!!` (1 sikertelen, 4 sikeres). Az első ping azért "hal meg" (timeout), mert a gépnek először le kell futtatnia a teljes ARP folyamatot a MAC-cím kiderítésére. Mire az ARP válasz megjön, az első ping ideje lejár. A maradék négy már gond nélkül átmegy, mert a cím már benne van az ARP táblában!

### Hálózati szimulátorok és a Wireshark

Jeremy megemlít két programot is: a **Packet Tracer** (ingyenes szimulátor, ami a CCNA-hez tökéletes) és a **GNS3** (profi eszköz, ami valódi Cisco IOS rendszereket futtat virtuálisan, de ehhez saját IOS licencre van szükség).

A GNS3-ba be van építve a **Wireshark** nevű program, amivel "le lehet hallgatni" a kábelen utazó forgalmat (packet capture). Ezzel tökéletesen látható a Padding (a csupa `00`-s bájtsorozat) és a Type mező, ami megmondja, mi van a keretben:

- Ha a Type `0800`: IPv4 csomag utazik benne.
- Ha a Type `86DD`: IPv6 csomag utazik benne.
- Ha a Type `0806`: ARP üzenet utazik benne.

## 5. Cisco Switch MAC-cím tábla parancsok (Kezelés és Törlés)

Lássuk a parancsokat, amelyekkel a switch legfontosabb memóriáját, a MAC-cím táblát kezelhetjük! (Minden parancsot a `#` promptnál, a Privileged EXEC módban használunk).

### 1. A tábla megtekintése

```text
Switch# show mac address-table
```

Megjelenített oszlopok: VLAN azonosító, MAC-cím, Típus (pl. DYNAMIC, ami 5 perc tétlenség után "elöregszik" és törlődik), és a Port (Interfész neve).

### 2. Minden dinamikus MAC-cím törlése a switchen

```text
Switch# clear mac address-table dynamic
```

### 3. Egyetlen, konkrét MAC-cím törlése a táblából

```text
Switch# clear mac address-table dynamic address 0c2f.b011.9d00
```

### 4. Egy adott interfészen (porton) lévő összes cím törlése

```text
Switch# clear mac address-table dynamic interface gi0/0
```

## Gyakorló Kvíz és Magyarázatok

**Kérdés 1: Egy 36 bájtos pinget küldesz egy másik számítógépnek, és hálózati forgalomelemzést (packet capture) végzel. Észreveszed, hogy az Ethernet hasznos teher (payload) legvégén egy hosszú, 00000000 bájtokból álló sorozat található. Mivel magyarázod ezeket a nullákat?**

- A) A pingek valójában nullák sorozatai.
- B) Ezek kitöltő bájtok (padding).
- C) Ezek az Ethernet FCS (Frame Check Sequence) részei.
- D) Ezek az IP-cím bináris megfelelői.

> **Helyes válasz:** B
>
> **Magyarázat:** Az Ethernet keretbe ágyazott hasznos teher (payload) kötelező minimális mérete 46 bájt. Mivel a te ping üzeneted csak 36 bájt volt, a hálózati kártya automatikusan hozzáadott 10 bájtnyi nullát (kitöltést), hogy a keret elérje az előírt legkisebb méretet.

**Kérdés 2: Az alábbi üzenetek közül melyiket küldik el a helyi hálózat összes gépének (broadcast)?**

- A) ARP Request (ARP Kérés)
- B) ARP Reply (ARP Válasz)
- C) ICMP Echo Request (Ping Kérés)
- D) ICMP Echo Reply (Ping Válasz)

> **Helyes válasz:** A
>
> **Magyarázat:** Amikor egy eszköz el akarja küldeni a keretet, de csak az IP-címet ismeri, szüksége van a MAC-címre. Mivel nem tudja, kihez tartozik a cím, az ARP Request üzenetet mindenkinek ki kell kiabálnia a hálózatba (broadcast). Az ARP válasz és a Ping üzenetek ezzel szemben már mind egyetlen konkrét címzettnek szóló, célzott (unicast) üzenetek.

**Kérdés 3: Milyen mezők (oszlopok) jelennek meg egy Cisco switchen a `show mac address-table` parancs kimenetében?**

- A) MAC address és ports
- B) VLAN, MAC address és ports
- C) VLAN, MAC address, type és ports
- D) Internet address, physical address és type

> **Helyes válasz:** C
>
> **Magyarázat:** A Cisco switchek MAC-cím táblája mindig négy oszlopot mutat: a hálózat azonosítóját (VLAN), a megtanult fizikai címet (MAC address), a tanulás módját (Type - ami legtöbbször "dynamic") és azt az interfészt/portot, amögött a gép található (Ports). A D opcióban szereplő oszlopok a Windows/PC-k ARP táblájára (`arp -a`) jellemzőek!

**Kérdés 4: Milyen típusú kereteket áraszt el (flood) a switch az összes interfészén, kivéve azon, amelyiken a keretet megkapta?**

- A) Broadcast és Unknown unicast
- B) Broadcast és Known unicast
- C) Known unicast és Unknown unicast
- D) Broadcast, Unknown unicast és Known unicast

> **Helyes válasz:** A
>
> **Magyarázat:** A Broadcast keretek (mint az ARP kérés) definíció szerint mindenkinek szólnak, így a switchnek továbbítania kell mindenfelé. Az Ismeretlen unicast (Unknown unicast) keretek csak egy gépnek szólnának, de mivel a switch még nem tudja, hol van a célállomás (nincs bejegyzés a MAC táblájában), kénytelen ezt is elárasztani, hogy biztosan célba érjen. Az ismert unicast kereteket a switch sosem árasztja el.

**Kérdés 5: Melyik parancsot használják a Cisco switcheken arra, hogy a MAC-cím táblából töröljék az összes dinamikusan megtanult MAC-címet egy konkrét interfészre vonatkozóan?**

- A) `clear mac address-table interface <interface-id>`
- B) `clear mac-address-table dynamic interface <interface-id>`
- C) `clear mac-address table dynamic interface <interface-id>`
- D) `clear mac address-table dynamic interface <interface-id>`

> **Helyes válasz:** D
>
> **Magyarázat:** A Cisco IOS-ben a helyes szintaxis: először a `clear mac address-table` jön (egy kötőjellel az address és a table között), majd meg kell adni, hogy a `dynamic` (dinamikus) címeket akarjuk törölni, végül az `interface` kulcsszó után megadni a konkrét portot.
