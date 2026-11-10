# CCNA Bővített Jegyzet – 21. Lecke: A Spanning Tree Protokoll (STP) Állapotok és Finomhangolás

Ebben a leckében folytatjuk a Spanning Tree Protokoll (STP) mélyebb megismerését. Az előző leckében megtanultuk a döntési folyamatot (Root Bridge, Root Port, Designated Port választása). Most azt nézzük meg, mi történik időben, miközben egy port bekapcsol és elkezdi a hálózati forgalmat átengedni. Végül megnézünk néhány életmentő kiegészítő funkciót és az alapvető konfigurációs parancsokat.

## 1. Az STP Port Állapotai (Port States) és az Időzítők

A klasszikus STP (IEEE 802.1D) rendkívül óvatos. Ha egy portot (kábelt) csatlakoztatsz egy switchhez, az nem kezd el azonnal működni! Először át kell esnie egy tesztelési szakaszon, nehogy véletlenül hurkot (Layer 2 loop) okozzon.

### A Stabil Állapotok

- **Blocking (Blokkoló):** A Non-Designated portok stabil állapota. Ilyenkor a port **ZÁRVA** van az adatforgalom számára.
  - Sima adatot (unicast, broadcast) **NEM** küld és **NEM** fogad.
  - A MAC címeket **NEM** tanulja meg.
  - BPDU üzeneteket **NEM** küld, kizárólag fogad. (Erre azért van szüksége, hogy "hallja", él-e még a hálózat, és szükség esetén felébredjen).
- **Forwarding (Továbbító):** A Root Portok és a Designated Portok stabil állapota. Ilyenkor a port **NYITVA** van.
  - Teljesen normálisan működik: küld és fogad adatot, megtanulja a MAC címeket, és BPDU-kat is küld/fogad.

### Az Átmeneti (Tranzíciós) Állapotok

Ha egy port felébred, vagy a Blocking állapotból Forwarding állapotba kell kerülnie, két fázison esik át:

- **Listening (Hallgatózó):**
  - **Időtartam:** 15 másodperc (A Forward Delay időzítő alapján).
  - A port hallgatja a hálózatot (BPDU-kat küld és fogad), hogy ellenőrizze, nem fog-e hurkot okozni, ha bekapcsol.
  - **NEM** küld adatot, és még **NEM** tanulja meg a MAC-címeket!
- **Learning (Tanuló):**
  - **Időtartam:** Újabb 15 másodperc (Szintén a Forward Delay időzítő alapján).
  - A port még mindig **NEM** küld adatot, viszont elkezdi megtanulni a MAC-címeket a bejövő keretekből, hogy felépítse a MAC-tábláját (így amikor végre kinyit, nem kell rögtön elárasztania mindent ismeretlen unicastként).

> **Összefoglalva:** A klasszikus STP-nek 50 másodpercbe telhet, amíg egy Blocking portból Forwarding port lesz (20 sec Max Age várakozás + 15 sec Listening + 15 sec Learning).

### Az Időzítők (Timers)

- **Hello Timer (Alap: 2 másodperc):** Milyen gyakran küldi a Root Bridge a BPDU üzeneteket.
- **Forward Delay Timer (Alap: 15 másodperc):** Mennyi ideig tart a Listening és a Learning fázis (külön-külön!).
- **Max Age Timer (Alap: 20 másodperc):** Meddig "emlékszik" a switch egy BPDU üzenetre. Ha egy Blocking port 20 másodpercig nem kap BPDU-t a szomszédjától, feltételezi, hogy a szomszéd (vagy a kábel) meghalt, így újraindítja az STP számolást, és elindítja a 30 másodperces (Listen+Learn) átállást.

## 2. A BPDU Üzenet (Mi van a motorháztető alatt?)

A BPDU (Bridge Protocol Data Unit) a switchek kommunikációs nyelve. Két fontos tény:

- A **Cisco PVST+** verziójában (Per-VLAN Spanning Tree Plus) a BPDU-k egy különleges Multicast MAC címre mennek: `0100.0ccc.cccd`. (A normál iparági szabványé `0180.c200.0000`). Ezt érdemes megjegyezni!
- **Csak a Root Bridge** hoz létre és küld eredeti BPDU-kat (a kijelölt portjain)! A többi switch csak átveszi, kicsit frissíti az adatokat (pl. hozzáadja a saját Root Costját), és továbbítja a saját kijelölt portjain.

## 3. Az STP "Eszköztára" (Toolkit) – Extrák a Cisco-tól

Mivel a 30-50 másodperces várakozási idő (Listening → Learning) nagyon bosszantó tud lenni, a Cisco kitalált pár trükköt a gyorsításhoz és a biztonsághoz.

### 1. PortFast

Ha rádugod a PC-det a switchre, és az narancssárgán villog 30 másodpercig, miközben nem kapsz IP-címet, az az STP műve! De egy PC nem tud hurkot okozni, hiszen nincs rákötve másik switch!

A **PortFast** funkció kikapcsolja az STP ellenőrzést az adott porton: ha bedugod a kábelt, a port **AZONNAL** Forwarding állapotba ugrik (kihagyja a 30 másodperces Listening/Learninget).

- **Feltétel:** Csakis és kizárólag Végpontokhoz (Access port, PC, szerver, router) szabad használni! Trunk porthoz tilos!
- **Konfiguráció:**
  - Interfészen: `Switch(config-if)# spanning-tree portfast`
  - Globálisan (Minden Access portra bekapcsolja): `Switch(config)# spanning-tree portfast default`

### 2. BPDU Guard (BPDU Őr)

A PortFast veszélyes: mi van, ha egy idióta felhasználó behoz egy saját kis switchet otthonról, és rádugja a fali aljzatra, amin él a PortFast? Azonnal hurok keletkezik!

A **BPDU Guard** megvéd ettől: Ha bekapcsolod, és az adott port véletlenül egy STP BPDU üzenetet (azaz egy másik switchet) érzékel, azonnal **LEKAPCSOLJA (Error-disable)** a portot, így megmentve a hálózatot!

- **Konfiguráció:**
  - Interfészen: `Switch(config-if)# spanning-tree bpduguard enable`
  - Globálisan (Minden PortFast portra ráteszi): `Switch(config)# spanning-tree portfast bpduguard default`

> (Extra megjegyzés: Létezik a **Root Guard** (nem engedi, hogy egy új switch lenyúlja a Root Bridge szerepet) és a **Loop Guard** (megakadályozza, hogy egy Blocking port tévesen kinyíljon egyirányú kábelhiba esetén), de a CCNA vizsgán ezeket nem kell mélyrehatóan ismerni).

## 4. Cisco IOS Parancsok: STP Konfigurálás és Finomhangolás

### 1. A Mód megváltoztatása

A modern Cisco switcheken a Rapid-PVST fut. Ha a klasszikusat akarod használni (mint ebben a leckében):

```text
Switch(config)# spanning-tree mode pvst
```

### 2. A Root Bridge (Gyökérhíd) manuális megválasztása

Nem bízzuk a véletlenre (vagy a MAC címekre), hogy ki legyen a főnök. Kijelöljük az 1. számú (Primary) és a tartalék (Secondary) switchet.

```text
Switch(config)# spanning-tree vlan 10 root primary
Switch(config)# spanning-tree vlan 10 root secondary
```

(Ezek a parancsok valójában a `spanning-tree vlan 10 priority <szám>` parancsot hajtják végre a háttérben, a `primary` a 24576-ot, a `secondary` a 28672-t osztja ki).

### Miért kell a VLAN számát odaírni? (Load Balancing - Terheléselosztás)

Mivel a Cisco a PVST+ verziót használja, **MINDEN VLAN-nak SAJÁT, FÜGGETLEN STP fája van!** Ezt kihasználhatjuk arra, hogy a terhelést eloszlassuk.

Ha a VLAN 10-nek az SW1-et tesszük meg Root Bridge-nek, a VLAN 20-nak pedig az SW2-t, akkor a VLAN 10-es forgalom miatt a bal oldali kábel fog blokkolódni, míg a VLAN 20 miatt a jobb oldali. Így mindkét kábelt használjuk (load-balancing), és egyik sem áll 100%-ban üresen (Blocking) az egész hálózat számára!

### 3. Port Cost (Költség) és Priority (Prioritás) módosítása

Ezekkel a parancsokkal kényszeríthetjük rá az STP-t, hogy egy bizonyos kábelt válasszon Root Portnak (felülírva a gyári sebesség-költségeket):

```text
Switch(config-if)# spanning-tree vlan 10 cost 100
Switch(config-if)# spanning-tree vlan 10 port-priority 64
```

## Kvíz a 21. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod az időzítőkről és a biztonsági funkciókról!

### Kérdés 1

Rádugsz egy PC-t a switchre, de kb. fél percig nem kap hálózatot. Melyik **KÉT** megoldás segíthet a hálózati hozzáférés felgyorsításában? (Válassz KETTŐT!)

- A) Kapcsold be a PortFast-ot az adott switchporton.
- B) Csökkentsd az STP Hello időzítőjét.
- C) Csökkentsd az STP Forward Delay időzítőjét.
- D) Csökkentsd az STP Max Age időzítőjét.

**Helyes Válasz: A és C**

**Magyarázat:** Az A a legjobb (gyártói) megoldás: a PortFast azonnal átugorja az ellenőrzést, és a port Forwarding lesz. A C technikailag lehetséges megoldás: a Forward Delay (alapból 15 sec) határozza meg a Listening és a Learning fázis hosszát. Ha ezt lecsökkentjük, a port gyorsabban vált Forwarding állapotba (bár a gyári értékek megváltoztatása nem javasolt).

### Kérdés 2

Egy hálózati forgalomelemzés (Packet Capture) során megnézed egy switchport BPDU-ját, és azt látod, hogy az STP Port ID hexadecimálisan `0x8002`. Mi ennek a portnak a decimális STP Port Prioritása?

- A) 80
- B) 32
- C) 128
- D) 224

**Helyes Válasz: C (128)**

**Magyarázat:** A Port ID két részből áll: a prioritásból és a port sorszámából. A `0x80` hexadecimálisan (128) a port prioritása, a `02` pedig a port száma. (A 128 a gyári alapértelmezett prioritás minden Cisco switch porton).

### Kérdés 3

Szeretnéd biztosítani, hogy garantáltan ne alakulhasson ki Layer 2 hurok (loop), ha egy felhasználó véletlenül egy másik switchet dugna a végpontjának (PC-jének) fenntartott fali csatlakozóba. Melyik STP kiegészítő funkció szolgál erre?

- A) PortFast
- B) Loop Guard
- C) Root Guard
- D) BPDU Guard

**Helyes Válasz: D (BPDU Guard)**

**Magyarázat:** A BPDU Guard "őrzi" a portot: amint meglát egy másik switchet (mert az BPDU üzenetet küld), azonnal lekapcsolja (Error-disable) az interfészt, ezzel megvédve a hálózatot a huroktól.

### Bónusz Kérdés (Boson ExSim vizsgakérdés)

Szeretnéd lecsökkenteni az időt, amíg a Switch A portjai elkezdik a forgalom továbbítását. Jelenleg a Switch A egyetlen portján sincs bekapcsolva a PortFast. Kiadod a globális konfigurációs módban a `spanning-tree portfast default` parancsot. Mely portok fognak innentől a Switch A-n PortFast módban működni? Válassza ki a legjobb választ!

- A) Egyik sem, mert a PortFast nem engedélyezhető globálisan.
- B) Az összes Trunk port.
- C) Minden egyes port.
- D) Az összes Access (Hozzáférési) port.

**Helyes Válasz: D**

**Magyarázat:** Ahogy a leckében tárgyaltuk, a `default` globális parancs egy biztonsági hálót alkalmaz: kizárólag azokat a portokat teszi PortFast módba, amik `switchport mode access` (végponti) konfigurációban vannak. Így a Trunk (switch-to-switch) portokat nem sodorja veszélybe.
