# CCNA Bővített Jegyzet – 34. Lecke: IPv6 Címtípusok és az EUI-64

Ebben a leckében folytatjuk az IPv6 felfedezését. Bár a múltkor láttuk, hogyan kell hexadecimális számokkal címet osztani, az IPv6 nem áll meg ennyinél: a routerek képesek saját maguknak is címet generálni egy speciális trükkel (EUI-64), és bemutatjuk a legfontosabb IPv6 címtípusokat (Global Unicast, Link-local, Unique Local, Multicast és Anycast), amelyeket a CCNA vizsgán meg kell tudnod különböztetni!

## 1. Az EUI-64 (A Router "Ön-Címző" Trükkje)

Mi történik, ha van egy IPv6 alhálózatod (pl. `2001:db8::/64`), és nem akarod manuálisan begépelni a host részt? Az IPv6 képes arra, hogy a fizikai interfész MAC-címét (ami 48 bites) átalakítsa egy 64 bites Interface ID-vé, így kiegészítve a hálózati prefixet egy teljes 128 bites címmé. Ezt a módszert hívják **Modified EUI-64**-nek (Extended Unique Identifier).

### A konverzió 3 lépése (Vizsgán KÖTELEZŐ!)

Tegyük fel, hogy az interfész MAC-címe: `1234:5678:90AB`

**1. Vágd ketté a MAC-címet!**

- Bal oldal: `1234:56`
- Jobb oldal: `78:90AB`

**2. Szúrj be a közepébe egy FFFE karaktersort!**

- Eredmény: `1234:56FF:FE78:90AB` (Ezzel 48 bitről 64 bitre növeltük a címet).

**3. Invertáld (fordítsd meg) a 7. bitet!**

- Hol van a 7. bit? A balról a 2. hexa karakter (`2`) harmadik bitje.
- A `2` binárisan `0010`. Ennek a 3. bitje egy `1`-es! (Ez a teljes cím 7. bitje).
- Ha az `1`-est átfordítod `0`-ra, az eredmény `0000` lesz, azaz `0`.
- A hexa `2`-ből így `0` lett.
- **Végeredmény:** `1034:56FF:FE78:90AB`

> (Ha az eredeti cím első karaktersora `00` volt, a 7. bit 0, amit 1-re fordítasz, így `02` lesz belőle. Lásd a fenti Boson kvízkérdést).

### Miért kell a 7. bitet piszkálni? (Érdekesség)

A MAC-címek 7. bitje a **U/L bit** (Universal/Local). Ha ez 0, akkor az egy gyári (Universal) MAC-cím. Ha 1, akkor a rendszergazda által kézzel beállított (Local) MAC-cím. Az EUI-64-nél a szabvány megfordította ezt a logikát: Ha az EUI-64 címben a 7. bit 1-es, az azt jelenti, hogy egy gyári MAC-címből készült. (Ez csak egy apró történelmi furcsaság).

### Cisco IOS Konfiguráció

```text
Router(config-if)# ipv6 address 2001:db8::/64 eui-64
```

A router veszi a megadott `/64`-es prefixet, kicsipegeti a saját fizikai MAC-címét, elvégzi rajta a fenti 3 lépést, és máris kész az IP-címe!

## 2. Az IPv6 Címtípusok (Kritikus Vizsgaanyag!)

Az IPv4-ben megszokott Public, Private és APIPA kategóriák az IPv6-ban új nevet (és prefixet) kaptak.

### 2.1. Global Unicast (GUA) - A Publikus Internet

Ezek a publikus, Interneten routolható címek (olyan, mint a publikus IPv4).

- **Tartomány:** `2000::/3` (Minden olyan cím, ami 2, vagy 3-al kezdődik).
- **Felépítése:**
  - **Global Routing Prefix** (Általában 48 bit): Ezt kapja a cég a szolgáltatótól (ISP).
  - **Subnet Identifier** (Általában 16 bit): A cég belső használatára, alhálózatok készítésére.
  - **Interface Identifier** (64 bit): A konkrét gép címe (lehet kézi vagy EUI-64).

### 2.2. Unique Local (ULA) - A Privát Hálózat

Ezek a címek **NEM routolhatóak az Interneten!** Kizárólag a cégen belüli privát kommunikációra valók (Olyan, mint az IPv4-es `192.168.x.x` vagy a `10.x.x.x`). A szolgáltatók eldobnak minden ilyen csomagot.

- **Tartomány:** `FC00::/7` (De a szabvány előírja a 8. bit bekapcsolását, így a gyakorlatban mindig `FD`-vel kezdődnek!).
- **Felépítése:** `FD` + egy 40 bites véletlenszerűen generált **Global ID**. (Azért kell véletlenszerűnek lennie, hogy ha a te céged egyesül egy másik céggel, ne legyen ütközés (overlapping) a belső hálózatok között!). Ezt követi a 16 bites **Subnet ID** és a 64 bites **Interface ID**.

### 2.3. Link-Local - A Lokális Sziget

Ezek a címek kizárólag az adott fizikai kábelen (subneten) érvényesek. A router **soha, semmilyen körülmények között nem routolhatja** (továbbíthatja) a Link-Local csomagokat egy másik interfész felé! (Kicsit olyan, mint a Windows-os `169.254.x.x` APIPA).

- **Tartomány:** `FE80::/10` (A szabvány miatt csak `FE8`, `FE9`, `FEA`, `FEB` lenne lehetséges, de az utána lévő 54 bitnek nullának kell lennie, így a gyakorlatban **MINDIG `FE80`-al kezdődik**).
- A routerek az OSPFv3 hálózatépítésnél és a "Next-hop" (következő ugrás) megadásánál szinte kizárólag ezeket használják! A PC-k a Default Gateway-üknek is az `FE80...` címet látják.
- **Azonnal generálódik!** Ha egy interfészen kiadod az `ipv6 enable` parancsot, vagy bármilyen más IPv6 címet beállítasz, a router a háttérben automatikusan legenerál magának egy Link-Local címet is (az `FE80::` és a MAC-címből képzett EUI-64 segítségével). Emiatt a `show ipv6 interface brief` kimenetben mindig **2 cím** fog szerepelni egy portnál!

### 2.4. Multicast (Szórás)

Az IPv6-ban **NINCS Broadcast!** A Broadcast feladatait a Multicast vette át.

- **Tartomány:** `FF00::/8` (Minden cím, ami `FF`-el kezdődik).

**A 4 Legfontosabb Multicast Cím (Meg kell tanulni!):**

- `FF02::1` - **All Nodes** (Minden gép - Ez helyettesíti a Broadcast-ot a lokális subneten!).
- `FF02::2` - **All Routers** (Minden router a lokális subneten).
- `FF02::5` - **All OSPF Routers** (Ugyanaz, mint az IPv4-es `224.0.0.5`).
- `FF02::A` - **All EIGRP Routers** (Ugyanaz, mint az IPv4-es `224.0.0.10`).

**Multicast Scope (Hatókör):** A 4. hexa karakter megmondja, meddig mehet a Multicast üzenet!

- `FF01` - **Interface-Local:** Nem hagyja el a gépet (a szoftveren belül marad).
- `FF02` - **Link-Local:** Csak az adott subneten érvényes (A router nem engedi át). Ez a leggyakoribb!
- `FF05` - **Site-Local:** A telephelyen belül (A routerek átengedik, de a WAN-on nem mehet ki).
- `FF08` - **Organization-Local:** A cégen belül bárhová.
- `FF0E` - **Global:** Akár az Interneten keresztül is routolható (bár a szolgáltatók általában blokkolják).

### 2.5. Anycast (A Legújabb)

Az IPv6 újítása! Az Anycast cím ("Egy-a-sok-közül") pontosan úgy néz ki, mint egy normál Unicast (Global vagy Unique Local) cím, nincs külön prefixe!

**A trükk:** Ugyanazt az IP-címet beállítod 5 különböző routeren a világ 5 különböző pontján (és a végére odaírod az `anycast` kulcsszót a CLI-ben). Ha a kliens küld egy csomagot erre az IP-re, a routing protokoll a csomagot a hozzá fizikailag legközelebb eső routerhez fogja irányítani (a legjobb metrika alapján).

### 2.6. Egyéb Különleges Címek

- **Unspecified (Meghatározatlan) cím:** `::` (Kizárólag csupa nulla). Azt jelzi, hogy a gépnek még nincs címe. A Default Route hálózati maszkja is ez: `::/0` (Az IPv4-es `0.0.0.0` megfelelője).
- **Loopback cím:** `::1` (A gép saját belső hálózati kártyájának címe. Az IPv4-es `127.0.0.1` megfelelője. Itt nem egy `/8`-as blokkot pazaroltak el, csak egyetlen címet).

## Kvíz a 34. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod az EUI-64 és a Címtípusok rejtelmeiből!

### Kérdés 1

Az R1 G0/1 interfészének MAC-címe `0D2A.4FA3.00B1`. Milyen IPv6 címet generál a gép az `ipv6 address 2001:DB8:0:1::/64 eui-64` parancs kiadása után?

- A) `2001:DB8:0:1:0D2A:4FFF:FEA3:00B1`
- B) `2001:DB8:0:1:0C2A:4FFF:FEA3:00B1`
- C) `2001:DB8:0:1:0F2A:4FA3:00B1:FFFE`
- D) `2001:DB8:0:1:0F2A:4FFF:FEA3:00B1`

**Helyes Válasz: D**

**Magyarázat:** MAC: `0D2A:4FA3:00B1`.
1. Középre beszúrjuk: `0D2A:4FFF:FEA3:00B1`.
2. Invertáljuk a 7. bitet: A `0D` (ahol D=13, azaz `1101`), a `0` binárisan `0000`. A második hexa karakter a `D` (`1101`). Ennek a harmadik bitje (ami a teljes cím 7. bitje) egy `0`. Ezt megfordítjuk `1`-re, így lesz a `1101`-ből `1111`, ami az `F`!
Eredmény: `0F2A:4FFF:FEA3:00B1`.

### Kérdés 2

Melyik része a "Global ID" (Globális Azonosító) a következő Unique Local címnek: `FDAB:CDEF:1234:5678:9ABC:DEF1:2345:6789`?

- A) `FD`
- B) `AB:CDEF:1234:56`
- C) `78`
- D) `9ABC:DEF1:2345:6789`

**Helyes Válasz: B**

**Magyarázat:** A Unique Local (Privát) címek `FD`-vel kezdődnek. Ezt a fix 8 bitet követi egy véletlenszerűen generált 40 bites (10 hexa karakterből álló) Global ID. Jelen esetben az `AB` (2 karakter) + `CDEF` (4 karakter) + `1234` (4 karakter) + `56` (az `5678` első 2 karaktere). A maradék `78` a Subnet Identifier.

### Kérdés 3

Az R3 egy IPv6 Multicast csomagot küld a LOKÁLIS ALHÁLÓZAT (Subnet) ÖSSZES ROUTERÉNEK. Mi lesz a csomag cél IP-címe? Válaszd a legjobb választ!

- A) `FF01::1`
- B) `FF01::2`
- C) `FF02::1`
- D) `FF02::2`

**Helyes Válasz: D**

**Magyarázat:** Mivel a lokális subneten marad, a Link-local scope-ot kell használni, azaz az `FF02`-őt (Ezzel az A és a B kiesik, mert azok Interface-local scope-ok, nem hagyják el a gépet). Az `1`-es az "All Nodes" (Minden host), a `2`-es az "All Routers" címe. Mivel a feladat routereknek akar küldeni, a D a helyes.

### Kérdés 4

Milyen IPv6 címtípus jön létre **AUTOMATIKUSAN** a porton, amint kiadod az `ipv6 enable` parancsot?

- A) Unique Local
- B) Node-Local
- C) Link-Local
- D) EUI-64

**Helyes Válasz: C**

**Magyarázat:** Amint az IPv6 életre kel egy porton, a router azonnal generál egy `FE80::` kezdetű Link-Local címet. (A D azért csapda, mert bár a Link-Local generálásához a router az EUI-64-es matematikai módszert használja, maga a CÍMTÍPUS hivatalos neve a Link-Local, és a vizsgákon a legpontosabb választ várják).

### Bónusz Kérdés (Boson ExSim vizsgakérdés - A Nem Routolható Címek)

Az alábbi IPv6 prefixek (címtartományok) közül melyik **KETTŐ** olyan, amit a routerek **NEM routolhatnak** (nem továbbíthatnak egy másik alhálózat felé)? (Válassz KETTŐT!)

- A) `2000::/3` (Global Unicast)
- B) `FE80::/10` (Link-Local Unicast)
- C) `FC00::/7` (Unique Local)
- D) `FD00::/8` (Unique Local)
- E) `FF05::/16` (Site-Local Multicast)
- F) `FF02::/16` (Link-Local Multicast)

**Helyes Válasz: B és F**

**Magyarázat:** Routolni azt jelenti, hogy a router továbbengedi a csomagot egy másik alhálózat felé. A "Link-Local" kifejezés (legyen az akár Unicast a B esetben, vagy Multicast az F esetben) pontosan azt jelenti, hogy a csomag szigorúan csak a fizikai kábelen lévő szomszédokig mehet, a router eldobja, ha próbál továbbmenni! A Unique Local (C, D) bár az Interneten nem routolható, de a céges, belső hálózatban (Pl. VLAN-ok között) tökéletesen routolható, tehát nem jó válasz. Az E (Site-Local Multicast) a telephelyen belül szintén routolható a belső routereken keresztül.
