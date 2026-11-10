# CCNA Bővített Jegyzet – 35. Lecke: IPv6 Befejezés (Fejléc, NDP, SLAAC és Statikus Útválasztás)

Ebben a videóban lezárjuk az IPv6 alapjait. Megnézzük az IPv6 fejlécét (Header), az ARP-t leváltó NDP-t (Neighbor Discovery Protocol), a SLAAC (Stateless Address Autoconfiguration) működését, és az IPv6 statikus útválasztás típusait. *(Minden elhangzott apróbetűs információ, trivia és parancs hiánytalanul szerepel a jegyzetben!)*

## 1. Helyesbítés: Az IPv6 Címek Ábrázolási Szabványa (RFC 5952)

Az IPv6 címek hexadecimális formátumát a hivatalos Internetes szabványok (RFC - Request for Comments) szabályozzák. Az IETF (Internet Engineering Task Force) által kiadott **RFC 5952** ("A Recommendation for IPv6 Address Text Representation") a következő szigorú szabályokat fektette le, amelyeket az előző videókban kissé lazábban kezeltünk:

- **Vezető nullák (Leading 0s) kötelező elhagyása:** A `0db8` kvartettet **KÖTELEZŐ** `db8`-ként írni.
- **Dupla kettőspont (`::`) használata:** **KÖTELEZŐ** a `::`-t használni a leghosszabb egybefüggő nullás kvartett-sorozat rövidítésére! Viszont ha csak egyetlen nullás kvartett van (pl. `:0000:`), azt tilos `::`-tal rövidíteni, szimplán `:0:`-nak kell írni.
- **Döntetlen a nullás sorozatoknál:** Ha a címben két egyformán hosszú nullás sorozat van (pl. kettő bal oldalt, és kettő jobb oldalt), a `::`-t kötelezően a **bal oldalira (az elsőre)** kell alkalmazni!
- **Kisbetűs Hexa Karakterek (A Legfontosabb!):** A hexadecimális karaktereket (`a, b, c, d, e, f`) szigorúan **kisbetűvel KÖTELEZŐ** írni! (Bár a Cisco IOS gyakran maga is megszegi ezt a szabályt, és nagybetűvel írja ki, a hivatalos szabvány a kisbetűket írja elő).

## 2. Az IPv6 Fejléc (Header) Áttekintése

Míg az IPv4 fejléce bonyolult volt és változó hosszúságú (20-60 byte), az IPv6 fejlécét drasztikusan leegyszerűsítették a gyorsabb hardveres feldolgozás (routing) érdekében. A mérete **Fixen 40 byte**.

**Az IPv6 Fejléc Mezői:**

- **Version (Verzió - 4 bit):** Mindig 6 (Binárisan: `0110`).
- **Traffic Class (Forgalomosztály - 8 bit):** A QoS (Quality of Service) számára. Hasonló az IPv4 "ToS" mezőjéhez. Priorizálja pl. a VoIP vagy a videós csomagokat.
- **Flow Label (Folyamcímke - 20 bit):** Egyedi forgalmi folyamok (pl. egy letöltés PC és Szerver között) azonosítására.
- **Payload Length (Tartalom hossza - 16 bit):** Csak a becsomagolt adat (pl. a Layer 4 szegmens) hosszát mutatja byte-ban. A fejléc saját 40 byte-ját **NEM** számolja bele (szemben az IPv4-gyel).
- **Next Header (Következő Fejléc - 8 bit):** Megmutatja, milyen protokoll (pl. TCP, UDP vagy ICMPv6) található a Payloadban. Ez az IPv4 "Protocol" mezőjének felel meg.
- **Hop Limit (Ugráskorlát - 8 bit):** Ugyanaz, mint az IPv4 TTL (Time to Live) mezője. Minden router csökkenti 1-gyel, és ha eléri a 0-t, a csomagot eldobják, hogy elkerüljék a végtelen hurkokat.
- **Source Address (Forrás Cím - 128 bit)**
- **Destination Address (Cél Cím - 128 bit)**

## 3. Neighbor Discovery Protocol (NDP) és a Solicited-Node Multicast

Mivel az IPv6-ban **NINCS Broadcast** (és így **NINCS ARP sem**), a MAC-címek kiderítésére az IPv6 az **NDP (Neighbor Discovery Protocol)** rendszert használja, amely az ICMPv6-ra és a hatékony Multicast címzésre épül.

Ehhez szükség van egy különleges Multicast címre, a **Solicited-Node Multicast Address**-re. Ezt a router/host a saját normál Unicast (vagy Link-Local) IP-címéből generálja:

- **Fix előtag:** `FF02::1:FF`
- **Hozzáadva:** A gép Unicast IP-címének utolsó 6 hexadecimális karaktere.

(Példa: Ha a cím `2001:db8::78:9abc`, a Solicited-Node címe: `FF02::1:FF78:9ABC` lesz).

### 3.1. Az NDP MAC-cím keresése (Az ARP helyettesítése)

Két ICMPv6 üzenetet használ a rendszer az ARP Request és ARP Reply helyett:

- **Neighbor Solicitation (NS) - ICMPv6 Type 135:** "Szomszéd Kérés". Hasonló az ARP Request-hez.
  - Forrás IP: R1 IP-címe.
  - Cél IP: R2 Solicited-Node Multicast címe! (Már nem Broadcast!).
- **Neighbor Advertisement (NA) - ICMPv6 Type 136:** "Szomszéd Válasz". Hasonló az ARP Reply-hez.
  - Az R2 elküldi benne a saját MAC-címét, egyenesen (Unicastként) az R1 IP-címére.

Ezután a router az ARP tábla helyett egy **IPv6 Neighbor Table**-t (Szomszéd tábla) épít, amit a `show ipv6 neighbor` paranccsal nézhetsz meg. (A `Reach` állapot jelzi, hogy a gép elérhető).

### 3.2. DAD (Duplicate Address Detection - Címütközés Keresése)

Minden alkalommal, amikor egy interfész feléled (`no shut`), vagy kap egy IPv6 címet (akár manuálisan, akár SLAAC-kal), kötelezően elvégzi a **DAD** ellenőrzést, hogy megnézze, használja-e valaki más ugyanezt az IP-címet a hálózaton.

Küld egy NS (Neighbor Solicitation) üzenetet a saját IP-címére/Solicited-node címére. Ha nem kap NA választ, a cím egyedi. Ha kap, a konzolon egy `%DUPLICATE` hibaüzenet jelenik meg.

## 4. Router Felismerés és a SLAAC (Stateless Address Autoconfiguration)

Az NDP másik hatalmas funkciója, hogy a kliensek DHCP szerver (**Stateless = Állapotmentes**) nélkül is képesek IP-címet és alapértelmezett átjárót (Default Gateway) generálni maguknak!

- **Router Solicitation (RS) - ICMPv6 Type 133:** Amikor egy gép bekapcsol, küld egy kérést az `FF02::2` (Minden Router) Multicast címre, kérve, hogy az alhálózaton lévő routerek azonosítsák magukat.
- **Router Advertisement (RA) - ICMPv6 Type 134:** A router válaszol az `FF02::1` (Minden Gép / All Nodes) Multicast címre. Az üzenet tartalmazza az alhálózat Prefixét (pl. `2001:db8::/64`) és a router saját (Link-Local) IP-címét. (A routerek egyébként RS kérés nélkül is periodikusan küldözgetnek RA üzeneteket).

**SLAAC működésbe lép:** A PC (vagy a Cisco router interfésze) veszi a prefixet, hozzácsapja a MAC-címéből generált EUI-64 Host-azonosítóját (vagy egy random generáltat), és kész az érvényes IPv6 címe! A gép a router Link-Local címét pedig elmenti Default Gateway-nek.

**Cisco CLI Konfiguráció (SLAAC kérése a router egyik portjára):**

```text
Router(config-if)# ipv6 address autoconfig
```

## 5. IPv6 Statikus Útválasztás (Static Routing)

Az IPv6 statikus útválasztás 100%-ban ugyanazon a logikán alapul, mint az IPv4.

- **Alapparancs engedélyezése (Kritikus!):** `ipv6 unicast-routing` (E nélkül a router nem továbbít csomagokat).
- **A Routing Table megnézése:** `show ipv6 route` (A "C" a Connected, az "L" a Local /128-as host útvonalat jelzi). Érdekesség: A Link-Local címek **SOHA** nem kerülnek be az IPv6 routing táblába! Létezik egy `FF00::/8 via Null0` útvonal is, ami a Multicast szemetelés eldobására szolgál.

### A Statikus Útvonalak 3 Fő Típusa

A parancs formátuma: `ipv6 route <célhálózat/maszk> [exit-interface] [next-hop-ip] [AD]`

**1. Directly Attached (Közvetlenül csatolt) Static Route:** Csak a kimenő (exit) interfészt adjuk meg.

- Példa: `ipv6 route 2001:db8:0:3::/64 s2/0`
- **Kritikus Cisco IPv6 Csapda:** Ez a módszer IPv6-ban **NEM működik Ethernet (pl. g0/0) interfészeken!** Beírhatod a CLI-be, bekerül a configba, de a router csendben nem fog rajta elküldeni semmilyen csomagot. **Csak Point-to-Point (Soros/Serial) interfészen használható!**

**2. Recursive (Rekurzív) Static Route:** Csak a Next-hop IP címet (a következő ugrás routerét) adjuk meg.

- Példa: `ipv6 route 2001:db8:0:3::/64 2001:db8:0:12::2`
- Azért hívják rekurzívnak, mert a routernek kétszer kell megnéznie a Routing táblát: Először megnézi, hogy a Cél hálózathoz melyik IP-re kell ugrani (pl. `12::2`), majd újra le kell futtatnia a keresést, hogy kiderítse, az a `12::2`-es IP vajon melyik fizikai interfészén (pl. `g0/0`) keresztül érhető el.

**3. Fully Specified (Teljesen specifikált) Static Route:** Az Exit interfészt **ÉS** a Next-hop IP-t is megadjuk!

- Példa: `ipv6 route 2001:db8:0:3::/64 g0/0 2001:db8:0:12::2`
- **A Link-Local Szabály:** Ha a Next-hop IP-cím gyanánt a szomszéd router Link-Local (`FE80`) címét akarod használni (amit gyakran szoktak!), akkor **KÖTELEZŐ** ezt a Fully Specified módszert használni (megadni mellé az exit-interfészt is), mert a Link-Local címek nem szerepelnek a routing táblában, így a rekurzív keresés elbukna!

(Természetesen itt is léteznek Host Route-ok `/128`-al, Default Route-ok `::/0`-val, és Floating Static Route-ok, amiknek az utolsó paramétereként megemeljük az Adminisztratív Távolságot [AD]).

## Kvíz a 35. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod a CCNA egyik legösszetettebb anyagából!

### Kérdés 1

Az R2 üzenetet küld az R1-nek, hogy informálja az R1-et az R2 G0/0 interfészének MAC-címéről (Miután R1 megkérdezte azt). Milyen típusú ICMPv6 üzenetet küld az R2?

- A) RA (Router Advertisement)
- B) NA (Neighbor Advertisement)
- C) RS (Router Solicitation)
- D) NS (Neighbor Solicitation)

**Helyes Válasz: B**

**Magyarázat:** Az ARP Request helyettesítője az NS (Neighbor Solicitation) üzenet, amiben az R1 megkérdezi a MAC címet. Az ARP Reply (a válasz, amiben a MAC címet elárulják) megfelelője a **Neighbor Advertisement (NA)**. A Router Advertisement/Solicitation a Default Gateway keresésére szolgál.

### Kérdés 2

Konfigurálsz egy új IPv6 címet az R1 G0/0 interfészén. Milyen üzenetet fog kiküldeni a router, hogy elvégezze a DAD (Duplicate Address Detection - Címütközés-kereső) folyamatot?

- A) RA
- B) NA
- C) RS
- D) NS

**Helyes Válasz: D**

**Magyarázat:** A cím beállításakor a routernek meg kell kérdeznie a hálózatot, hogy "Hé, használja valaki ezt a MAC (vagy IP) címet?". A "kérdezés" az NDP-ben az **NS (Neighbor Solicitation)** üzenettel történik, amit a router a saját maga Solicited-Node Multicast címére küld. Ha csend van (Nincs NA válasz), a cím használható.

### Kérdés 3

Az R1 egy RA (Router Advertisement) üzenetet küld a helyi szegmensen (local link) lévő eszközöknek, hogy informálja őket a jelenlétéről és a hálózat Prefixéről (SLAAC). Milyen cél IPv6 címet használ ehhez a R1?

- A) `FF01::1`
- B) `FF01::2`
- C) `FF02::1`
- D) `FF02::2`

**Helyes Válasz: C**

**Magyarázat:** Az RA üzenet célja, hogy a hálózat minden gépe (PC-k, okostelefonok stb.) megtanulja az átjárót és az IP-címet. Ezért az R1 az "All Nodes" (Minden Gép) Multicast címet, azaz az **`FF02::1`**-et fogja használni. A D válasz az "All Routers" lenne, de a PC-k nem routerek, így nem vennék a lapot.

### Kérdés 4

Kiadod a következő IPv6 statikus útvonal parancsot: `ipv6 route 2001:db8:0:3::/64 g0/0 2001:db8:0:12::2`. Mely **KÉT** statikus útvonal típus jellemző erre a parancsra? (Válassz KETTŐT!)

- A) Fully specified
- B) Network
- C) Host
- D) Directly attached
- E) Recursive
- F) Default

**Helyes Válasz: A és B**

**Magyarázat:** A parancs tartalmazza a kilépő interfészt (`g0/0`) **ÉS** a következő ugrás IP-címét (`12::2`) is, tehát ez egy **Fully specified** útvonal (A). Mivel a cél (`2001:db8:0:3::/64`) egy egész alhálózat (Subnet) a `/64`-es prefixszel, ez egy **Network (Hálózati)** útvonal (B). (A Host `/128` lenne, a Default `::/0`).

### Kérdés 5

Az alábbi parancsok közül melyik hoz létre egy **Rekurzív, HOST** útvonalat?

- A) `ipv6 route 2001:db8::1/128 s2/0`
- B) `ipv6 route 2001:db8::1/128 s2/0 2001:db8:0:12::2`
- C) `ipv6 route 2001:db8::1/128 2001:db8:0:12::2`
- D) `ipv6 route 2001:db8::/64 2001:db8:0:12::2`

**Helyes Válasz: C**

**Magyarázat:** A "Rekurzív" útvonal azt jelenti, hogy **CSAK** a Next-hop IP címet adtuk meg (Exit interfészt nem). Így kiesik az A és a B válasz. A "Host" útvonal azt jelenti, hogy pontosan egyetlen gépet (IP-t) célzunk meg, aminek IPv6-ban kötelezően `/128` a maszkja. A D válasz `/64`-es, ami Network (Hálózati) útvonal. A C az egyetlen, ami Host is (`/128`), és rekurzív is (Csak next-hop van megadva).

### Bónusz Kérdés (Boson ExSim vizsgakérdés - Hibakeresés Statikus Routingnál)

Kiadod az `ipv6 route 2001:db8:2::/64 2001:db8:1::2` parancsot a RouterA-n, hogy a forgalom eljuthasson a RouterC-ig.
Amikor megpróbálod megpingelni a RouterC interfészét a RouterA-ról a `ping ipv6 2001:db8:2::2` paranccsal, a ping elbukik!

Az alábbiak közül melyik a legvalószínűbb probléma? Válaszd a legjobb választ!

- A) A RouterB-nek nincs útvonala a `2001:db8:1::/64` hálózathoz.
- B) A RouterA-nak nincs alapértelmezett átjárója (Default gateway).
- C) A RouterC-nek nincs útvonala a `2001:db8:1::/64` hálózathoz.
- D) A RouterB-nek nincs útvonala a `2001:db8:2::/64` hálózathoz.

**Helyes Válasz: C**

**Magyarázat:** A Pingeléshez (ICMP Echo) oda-vissza kommunikációra van szükség. A RouterA elküldi a ping csomagot a RouterC felé (mert megírtuk neki a statikus utat). A RouterC meg is kapja a csomagot. Ahhoz viszont, hogy válaszolni tudjon, a RouterC-nek tudnia kellene, hol van a feladó, azaz a RouterA hálózata (`2001:db8:1::/64`). Mivel a hibaüzenet szerint a ping elbukott, valószínűleg a RouterC-n felejtették el beállítani a **VISSZAFELÉ mutató (return route)** statikus útvonalat az A router hálózata felé (így a C a helyes válasz). A RouterB-nek mindkét említett hálózat (az 1-es és a 2-es is) közvetlenül csatlakoztatott (Connected) hálózata, így az A és D opció eleve kizárható, neki nem kell statikus út a saját kábeleihez.
