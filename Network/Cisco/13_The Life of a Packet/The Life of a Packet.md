# CCNA Bővített Jegyzet – 12. Lecke: Egy Csomag Élete (The Life of a Packet)

Ebben a leckében nem tanulunk új parancsokat. Ehelyett összekötjük az eddig tanultakat, és megnézzük a teljes képet. Végigkísérünk egy adatcsomagot, amint az elindul az egyik géptől (PC1), áthalad a routereken, és megérkezik a célállomásra (PC4). A cél, hogy tökéletesen megértsük a Layer 2 (MAC-címek) és a Layer 3 (IP-címek) együttműködését a hálózatban.

## 1. A Kiinduló Állapot és a Cél

- **Küldő (Forrás):** PC1 (IP: `192.168.1.1` - MAC: `1111`)
- **Fogadó (Cél):** PC4 (IP: `192.168.4.1` - MAC: `4444`)
- **A hálózat:** PC1 → Switch1 → Router1 → Router2 → Router4 → PC4

Amikor a PC1 adatot akar küldeni a PC4-nek, összeállít egy IP-csomagot.

- **Forrás IP:** `192.168.1.1` (PC1)
- **Cél IP:** `192.168.4.1` (PC4)

A PC1 látja, hogy a cél IP-cím (`192.168.4.x`) nem az ő saját hálózatában (`192.168.1.x`) van. Ebből tudja, hogy a csomagot az **Alapértelmezett Átjárójának (Default Gateway)**, vagyis az R1 routernek kell elküldenie. Ahhoz viszont, hogy a kábelen keresztül rá tudja tenni az adatot, szüksége van az R1 MAC-címére.

## 2. Első Lépés: Az ARP Folyamat a Kliens és a Router között

Mivel a PC1 nem ismeri az R1 MAC-címét, lefuttat egy **ARP (Address Resolution Protocol)** lekérdezést.

- **ARP Request (Kérés):** A PC1 küld egy Broadcast (mindenkinek szóló) keretet.
  - Cél MAC: `FFFF.FFFF.FFFF` (Broadcast)
  - Üzenet: "Hé `192.168.1.254` (R1), mi a MAC-címed?"
- A Switch1 (SW1) megtanulja PC1 MAC-címét, majd elárasztja (flood) a kérést az összes portján.
- **ARP Reply (Válasz):** Az R1 megkapja a kérést, és visszaküld egy célzott Unicast választ a PC1-nek.
  - Üzenet: "Itt a `192.168.1.254` (R1), a MAC-címem: `AAAA`."
- A SW1 közben megtanulja az R1 MAC-címét is.

Most már a PC1 tudja az R1 MAC-címét (`AAAA`), így becsomagolja az IP-csomagot egy Ethernet keretbe, és elküldi az R1-nek.

**Az első keret fejléce a PC1 és R1 között:**

| | Forrás | Cél |
|---|---|---|
| **IP** | PC1 | PC4 |
| **MAC** | PC1 (`1111`) | R1 (`AAAA`) |

## 3. Második Lépés: Utazás a Routerek között (R1 → R2)

Az R1 megkapja a keretet, leveszi róla az Ethernet fejlécet (**de-encapsulation**), majd belenéz a tiszta IP csomagba. Meglátja, hogy a cél a PC4 (`192.168.4.1`).

Belenéz a saját routing táblájába, és látja, hogy ahhoz a hálózathoz a következő ugrás (**next-hop**) az R2 router lesz.

Hogy átküldje az R2-nek, az R1-nek be kell csomagolnia a csomagot egy **ÚJ** Ethernet keretbe. De ehhez tudnia kell az R2 MAC-címét!

- Az R1 küld egy ARP Request-et az R2 felé (Cél MAC: csupa F).
- Az R2 küld egy ARP Reply-t az R1-nek, amiben elárulja a MAC-címét (ami az R2 G0/0 portján `CCCC`).
- Az R1 most elkészíti az új keretet, és elküldi az R2-nek.

**A második keret fejléce az R1 és R2 között:**

| | Forrás | Cél |
|---|---|---|
| **IP** | PC1 | PC4 (Figyelem: Az IP-címek nem változtak!) |
| **MAC** | R1 (`BBBB`) | R2 (`CCCC`) (A MAC-címek viszont lecserélődtek!) |

## 4. Harmadik Lépés: Utazás a Routerek között (R2 → R4)

Az R2 megkapja a keretet, leveszi a fejlécet, megnézi az IP-címet, és megnézi a routing tábláját. Látja, hogy a PC4-hez az R4 router a következő ugrás.

Ugyanaz a folyamat ismétlődik:

- ARP Request és Reply az R2 és R4 között, hogy R2 megtudja az R4 MAC-címét (`EEEE`).
- Az R2 elkészíti az új Ethernet keretet, és átküldi az R4-nek.

**A harmadik keret fejléce az R2 és R4 között:**

| | Forrás | Cél |
|---|---|---|
| **IP** | PC1 | PC4 |
| **MAC** | R2 (`DDDD`) | R4 (`EEEE`) |

## 5. Negyedik Lépés: Megérkezés a Célhálózatba (R4 → PC4)

Az R4 megkapja a keretet, leveszi a fejlécet, és megnézi az IP-címet (`192.168.4.1`).

A routing táblájából látja, hogy ez a hálózat közvetlenül hozzá van csatlakoztatva (**Connected route** a G0/2 porton).

Vagyis most már nincs több ugrás (nincs next-hop router), egyenesen a PC4-nek kell küldeni a keretet! Ehhez az R4-nek szüksége van a PC4 MAC-címére.

- Az R4 küld egy ARP Request-et a helyi hálózaton (a Switch4-en keresztül).
- A PC4 megkapja, és küld egy ARP Reply-t, amiben közli a MAC-címét (`4444`).
- Most az R4 elkészíti a legutolsó Ethernet keretet, és elküldi a PC4-nek.

**A negyedik (végső) keret fejléce az R4 és a PC4 között:**

| | Forrás | Cél |
|---|---|---|
| **IP** | PC1 | PC4 |
| **MAC** | R4 (`FFFE`) | PC4 (`4444`) |

A PC4 megkapja a keretet, leveszi az Ethernet fejlécet, és látja, hogy a benne lévő IP csomag pontosan neki szól. A csomag megérkezett!

## A Legfontosabb Szabály: A Címek Változása

Ebből a példából két kritikus dolgot kell megtanulni az utazó csomagokról:

1. **Az IP-címek SOHA nem változnak az út során!** A forrás végig a PC1, a cél végig a PC4 maradt.
2. **A MAC-címek UGRÁSRÓL UGRÁSRA (hop-by-hop) folyamatosan változnak!** Minden egyes router lefejti a régi Ethernet borítékot, és egy újat tesz rá a saját MAC-címével mint forrás, és a következő router (vagy célgép) MAC-címével mint cél.

> (Ez alól a szabály alól az útvonalban lévő **Switchek** jelentenek kivételt: a Switchek nem bontják fel és nem cserélik le az Ethernet fejlécet, ők csak egyszerűen továbbítják a kereteket).

## Kvíz a 12. Leckéhez

A videó alapján az alábbi kvíz kérdései ugyanarra a fenti hálózati ábrára és szituációra vonatkoznak. Most **A PC4 KÜLD EGY VÁLASZT A PC1-NEK** (visszafelé R4 → R2 → R1 útvonalon).

> (Mivel korábban már volt oda-vissza kommunikáció, a gépek már emlékeznek a MAC-címekre az ARP táblájukban, így most nem lesz szükség újabb ARP lekérdezésekre, csak az adat fog utazni).

### Kérdés 1

A PC4 elküldi a csomagot a PC1-nek. Mi lesz a keret Cél MAC-címe (Destination MAC) akkor, amikor a PC4 kilövi azt a saját hálózati kártyáján?

- A) A PC1 MAC-címe
- B) Az R4 G0/2 portjának MAC-címe (`FFFE`)
- C) Az R4 G0/1 portjának MAC-címe (`EEEE`)

**Helyes Válasz: B**

**Magyarázat:** Mivel a PC1 egy távoli hálózatban van, a PC4-nek a csomagot a saját Alapértelmezett Átjárójának (Default Gateway), azaz az R4-es routernek kell küldenie. Ezért a cél MAC-cím az R4-nek arra a portjára fog mutatni, amelyikre a PC4 rá van kötve (G0/2, aminek a címe `FFFE`).

### Kérdés 2

A PC4 elküldi a csomagot a PC1-nek. Mi lesz a Forrás MAC-cím (Source MAC) akkor, amikor a keret megérkezik az R1 G0/0 portjára?

- A) A PC4 MAC-címe
- B) Az R4 MAC-címe
- C) Az R2 G0/0 portjának MAC-címe (`CCCC`)

**Helyes Válasz: C**

**Magyarázat:** A csomag a visszafelé úton az R4-ről az R2-re, onnan pedig az R1-re érkezik. Amikor az R2 átadja a csomagot az R1-nek, egy új Ethernet keretet készít, amelyben a Forrás MAC-cím a saját kilépő portjának a MAC-címe lesz (R2 G0/0 portja, aminek a címe `CCCC`).

### Kérdés 3

A PC4 elküldi a csomagot a PC1-nek. Mi lesz a Forrás MAC-cím (Source MAC) akkor, amikor a keret kilép a Switch1 (SW1) G0/1 portján a PC1 felé?

- A) Az R1 G0/2 portjának MAC-címe (`AAAA`)
- B) A Switch1 MAC-címe
- C) A PC4 MAC-címe

**Helyes Válasz: A**

**Magyarázat:** Amikor az R1 elküldi a keretet a PC1-nek a Switchen keresztül, a Forrás MAC-cím az R1 G0/2 portjának címe (`AAAA`) lesz. Amikor a keret áthalad a Switchen (SW1), a Switch **nem módosítja és nem cseréli le a MAC-címeket!** Ő csak továbbítja a keretet. Így a forrás továbbra is az R1 MAC-címe marad.

### Kérdés 4

A PC4 elküldi a csomagot a PC1-nek. Mi lesz a Cél IP-cím (Destination IP) akkor, amikor a keret kilép az R4 G0/1 portján az R2 felé?

- A) Az R2 IP-címe
- B) Az R1 IP-címe
- C) A PC1 IP-címe (`192.168.1.1`)

**Helyes Válasz: C**

**Magyarázat:** Az IP-címek az egész utazás során (a feladótól egészen a végső célig) soha nem változnak meg. Mivel a csomag végső célállomása a PC1, a cél IP-cím végig a PC1 címe (`192.168.1.1`) marad. Csak a MAC-címek változnak menet közben.

### Kérdés 5

A PC4 elküldi a csomagot a PC1-nek. Mi lesz a Forrás IP-cím (Source IP) akkor, amikor a keret megérkezik az R1 G0/0 portjára?

- A) Az R2 IP-címe
- B) Az R4 IP-címe
- C) A PC4 IP-címe (`192.168.4.1`)

**Helyes Válasz: C**

**Magyarázat:** A 4. kérdés logikájához hasonlóan a Forrás IP-cím is végig változatlan marad a csomag teljes élettartama alatt. Mivel a csomagot a PC4 készítette és indította, a forrás IP-cím végig a PC4 címe (`192.168.4.1`) marad.
