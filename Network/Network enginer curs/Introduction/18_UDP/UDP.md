# Mi az az UDP?

A **User Datagram Protocol (UDP)** egy szállítási rétegbeli kommunikációs protokoll, amely nagyon gyakori a hang- és videóforgalom számára.

## Mi a User Datagram Protocol (UDP/IP)?

A **User Datagram Protocol**, vagy **UDP**, egy kommunikációs protokoll, amelyet az interneten keresztül használnak különösen időérzékeny átvitelekhez, például videólejátszáshoz vagy DNS keresésekhez. Felgyorsítja a kommunikációt azáltal, hogy nem létesít formális kapcsolatot az adatok átvitele előtt. Ez lehetővé teszi az adatok nagyon gyors átvitelét, de azt is okozhatja, hogy a csomagok elvesznek az átvitel során – és lehetőséget teremt a kihasználásra DDoS támadások formájában.

## Hogyan működik az UDP?

Mint minden hálózati protokoll, az UDP is egy szabványosított módszer az adatok továbbítására két számítógép között egy hálózatban. Más protokollokhoz képest az UDP egyszerű módon valósítja meg ezt a folyamatot: csomagokat (adatátviteli egységeket) küld közvetlenül egy cél számítógépnek, anélkül hogy először kapcsolatot létesítene, jelezné az említett csomagok sorrendjét, vagy ellenőrizné, hogy azok a szándéknak megfelelően érkeztek-e meg. (Az UDP csomagokat „datagramoknak" nevezik.)

### TCP vs. UDP

Az UDP gyorsabb, de kevésbé megbízható, mint a TCP, egy másik gyakori szállítási protokoll. Egy TCP kommunikációban a két számítógép egy „kézfogás" nevű automatizált folyamaton keresztül kapcsolatot létesít. Csak miután ez a kézfogás befejeződött, akkor továbbít az egyik számítógép ténylegesen adatcsomagokat a másiknak.

Az UDP kommunikáció nem megy keresztül ezen a folyamaton. Ehelyett az egyik számítógép egyszerűen elkezdhet adatokat küldeni a másiknak:

![TCP vs UDP kommunikáció]

Ezenkívül a TCP kommunikáció jelzi, hogy az adatcsomagokat milyen sorrendben kell fogadni, és megerősíti, hogy a csomagok a szándéknak megfelelően érkeznek meg. Ha egy csomag nem érkezik meg – például a közbenső hálózatok torlódása miatt –, a TCP megköveteli annak újraküldését. Az UDP kommunikáció ezeket a funkciókat nem tartalmazza.

Ezek a különbségek előnyöket teremtenek. Mivel az UDP nem igényel „kézfogást" vagy ellenőrzi, hogy az adatok megfelelően érkeznek-e meg, sokkal gyorsabban képes adatokat továbbítani, mint a TCP.

Ez a sebesség azonban kompromisszumokkal jár. Ha egy UDP datagram elveszik az átvitel során, nem küldik újra. Ennek eredményeként az UDP-t használó alkalmazásoknak tolerálniuk kell a hibákat, a veszteséget és a duplikációt.

(Technikailag az ilyen csomagvesztés kevésbé az UDP hibája, mint az internet felépítésének következménye. A legtöbb hálózati router tervezésük szerint nem végez csomagsorrendezést és érkezési visszaigazolást, mert ez elfogadhatatlan mennyiségű extra memóriát igényelne. A TCP egy módja annak, hogy betöltse ezt a hiányt, amikor egy alkalmazás igényli.)

## Mire használják az UDP-t?

Az UDP-t általában időérzékeny kommunikációban használják, ahol az alkalmi csomagvesztés jobb, mint a várakozás. A hang- és videóforgalmat gyakran ezzel a protokollal küldik, mert mindkettő időérzékeny, és arra tervezték, hogy bizonyos szintű veszteséget kezeljenek. Például a VoIP (voice over IP), amelyet sok internetalapú telefonszolgáltatás használ, általában UDP-n keresztül működik. Ennek az az oka, hogy egy statikus zajos telefonbeszélgetés előnyösebb, mint egy kristálytiszta, de erősen késleltetett.

Ez teszi az UDP-t az online játékok ideális protokolljává is. Hasonlóképpen, mivel a DNS szervereknek gyorsnak és hatékonynak kell lenniük, azok is UDP-n keresztül működnek.

## Hogyan használják az UDP-t DDoS támadásokban?

Az UDP „kockázatai", mint a csomagvesztés, a legtöbb felhasználási esetben nem jelentenek komoly problémát. Az UDP-t azonban rosszindulatú célokra is ki lehet használni. Mivel az UDP nem igényel kézfogást, a támadók „eláraszthatják" a célzott szervert UDP forgalommal anélkül, hogy először megszereznék annak a szervernek az engedélyét a kommunikáció megkezdéséhez.

Egy tipikus UDP flood támadás nagy számú UDP datagramot küld a cél számítógép véletlenszerű portjaira. Ez arra kényszeríti a célt, hogy ugyanolyan nagy számú ICMP csomaggal válaszoljon, amelyek jelzik, hogy ezek a portok elérhetetlenek voltak. A minden egyes hamis datagramra való válaszadáshoz szükséges számítástechnikai erőforrások kimeríthetik a célt, ami a legitim forgalom szolgáltatásmegtagadását eredményezi. (Ha többet szeretnél megtudni, olvasd el az UDP flood támadásokról szóló cikkünket.)

A szervezetek különféle módszerekkel védekezhetnek az UDP flood támadások ellen. Az egyik az ICMP csomagok válaszsebességének korlátozása, bár ez a megközelítés a legitim csomagokat is kiszűrheti. Egy másik módszer az UDP forgalom fogadása és megválaszolása egy sok elosztott adatközpontból álló közvetítő hálózaton keresztül, megakadályozva, hogy egyetlen eredeti szerver túlterhelődjön a hamis kérésekkel. Tudj meg többet arról, hogyan használja a Cloudflare ezt a stratégiát, hogy segítsen a szervezeteknek mérsékelni a DDoS támadásokat.
