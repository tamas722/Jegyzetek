# CCNA Bővített Jegyzet – Az STP Eszköztár (Toolkit): PortFast

Ez a lecke a Spanning Tree Protokoll (STP) "eszköztárának" egyik leghasznosabb és leggyakrabban használt funkciójára, a **PortFast**-ra fókuszál. Bár az STP megvédi a hálózatot a hurkoktól, a hagyományos működése komoly frusztrációt okozhat az egyszerű végfelhasználók számára. A PortFast ezt a problémát oldja meg.

## 1. A Probléma: A Bosszantó 30 Másodperces Várakozás

Képzelj el egy hétköznapi szituációt: egy irodai dolgozó bedugja a laptopját a fali aljzatba (ami egy Cisco switchhez csatlakozik).

- A hálózati kártya bekapcsol, a link fizikailag létrejön (a switchport státusza a `show ip interface brief` kimenetben: `up/up`).
- A felhasználó azonban megnyitja a böngészőt, és azt látja, hogy "Nincs internet".
- Ha megnézzük a switchen a LED-et, az nem zölden, hanem **narancssárgán villog**.

### Miért történik ez?

Bár a kábel be van dugva, a klasszikus STP (802.1D) elkezdi a portot a tranzíciós fázisokon végigvezetni:

- **Listening (Hallgatózó) állapot:** 15 másodperc. A port csak BPDU-kat figyel.
- **Learning (Tanuló) állapot:** 15 másodperc. A port elkezdi tanulni a MAC-címeket, de adatot még mindig nem enged át.
- Végül, 30 másodperc elteltével a port átlép **Forwarding (Továbbító)** állapotba. A LED zöldre vált, a felhasználó gépe kap egy IP-címet a DHCP-től, és az internet elkezd működni.

A probléma az, hogy ez a 30 másodperces várakozás egy PC esetében **teljesen felesleges**. Az STP azért várakoztatja a portot, hogy megbizonyosodjon róla, nem alakul ki Layer 2 hurok (loop). De egy PC, nyomtató vagy szerver soha nem fog hurkot okozni, hiszen nincs rákötve másik switchre!

## 2. A Megoldás: A PortFast

A **PortFast** egy Cisco által kifejlesztett kiegészítő funkció (amelyet ma már a Rapid STP is beépítve használ **Edge Port** néven).

### Mit csinál a PortFast?

Ha egy porton be van kapcsolva a PortFast, a port csatlakoztatáskor **AZONNAL**, a 30 másodperces várakozás nélkül, egyenesen a Forwarding állapotba ugrik.
Bypassolja (kikerüli) a Listening és a Learning fázisokat. Amint a felhasználó bedugja a kábelt, a LED azonnal zöldre vált, és a hálózat azonnal használható.

> **Kritikus Biztonsági Szabály:**
> A PortFast-ot **CSAK ÉS KIZÁRÓLAG Végpontokhoz** (Access portok: PC-k, szerverek) szabad használni! **SOHA** ne kapcsold be olyan porton, ami egy másik switchhez csatlakozik! Ha két switchet kötsz össze egy PortFast-os porton, az STP-nek nem lesz ideje reagálni, és azonnal egy ideiglenes (vagy végleges) hurok keletkezik a hálózatban, ami megbéníthatja azt.

## 3. A PortFast Konfigurálása (Cisco CLI)

A PortFast-ot kétféleképpen lehet bekapcsolni a switchen: egyenként a portokon, vagy globálisan az egész eszközön.

### 1. Interfész Szintű Konfiguráció (Per-Port)

Lépj be az adott port konfigurációs módjába, és add ki a parancsot:

```text
Switch(config-if)# spanning-tree portfast
```

> **Figyelem:** Amint kiadod a parancsot, a Cisco IOS egy hosszú figyelmeztető üzenetet (Warning) fog küldeni a képernyőre, amiben felhívja a figyelmedet a hurkok veszélyére, ha véletlenül switchet dugnál a portra. Ez a normális működés része, nem hibát jelent.
> Egy másik figyelmeztetés is megjelenik: a PortFast alapesetben csak akkor lép életbe, ha a port Access módban van (nem trunk).

### 2. Globális Konfiguráció (Default)

Ha van egy 48 portos switched, ahol 46 porton PC-k vannak, nem akarod 46-szor begépelni a parancsot. Ehelyett bekapcsolhatod az egészet egyszerre, Globális konfigurációs módból:

```text
Switch(config)# spanning-tree portfast default
```

**Mit csinál pontosan ez a parancs?**
Engedélyezi a PortFast-ot a switch **ÖSSZES Access (hozzáférési) portján**. A Trunk portokra (amikkel tipikusan a switcheket kötjük össze) a parancs hatástalan marad! Emiatt ez a legbiztonságosabb és legkényelmesebb módszer.

Ha egy konkrét Access porton valamiért mégis ki akarod kapcsolni (pl. mert oda mégis egy switchet akarsz dugni access linken keresztül), az adott interfészen kiadhatod a tiltó parancsot:

```text
Switch(config-if)# spanning-tree portfast disable
```

## 4. A PortFast és a Trunk Portok (Kivételek)

Bár a PortFast alapesetben csak Access portokon működik, vannak olyan ritka esetek, amikor egy Trunk porton is szükségünk van az azonnali (várakozás nélküli) bekapcsolásra:

- **Router on a Stick (ROAS):** A router egy Trunk porttal csatlakozik a switchhez, de nem okoz hurkot.
- **Virtualizációs Szerverek (VMware, Hyper-V):** A szerverek gyakran Trunk kábellel csatlakoznak, hogy több VLAN forgalmát vigyék át a virtuális gépekhez, de ők sem okoznak hurkot.

Ilyenkor a port konfigurációs módjában felül kell bírálni a gyári védelmet az alábbi paranccsal:

```text
Switch(config-if)# spanning-tree portfast trunk
```

(Ezzel a port Trunk marad, de a PortFast mégis aktiválódik rajta).

## 5. Az "Edge" Kulcsszó (PortFast Edge)

A modern Cisco IOS verziókban, ha megvizsgálod a futó konfigurációt (`show running-config interface g0/1`), észreveheted, hogy a rendszer automatikusan kiegészítette az általad beírt parancsot egy `edge` kulcsszóval. (Például a `spanning-tree portfast` parancsból `spanning-tree portfast edge` lett).

Ennek az oka: A PortFast-nak létezik egy másik, a CCNA anyagán kívül eső változata is (a **PortFast Network**, ami a Bridge Assurance nevű funkcióhoz kell). Hogy egyértelmű legyen, a Cisco bevezette az "edge" (peremhálózati) jelölést. A CCNA vizsgán a PortFast és a PortFast Edge fogalmak **teljesen egyenértékűek**. A konfiguráció során használhatod a régi, rövidebb parancsot is, a gép automatikusan átalakítja az "edge" verzióra.

Az ellenőrzéshez használt parancs:

```text
Switch# show spanning-tree interface g0/1 detail
```

Ez megmutatja a port állapotát, és ha a funkció aktív, a kimenetben szerepelni fog a következő sor: `"The port is in the portfast edge mode"`.
