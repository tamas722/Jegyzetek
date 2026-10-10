# Mi az a hálózati protokoll?

A hálózatkezelésben a **protokoll** az adatok formázására és feldolgozására vonatkozó szabályok összessége. A hálózati protokollok olyanok, mint a számítógépek közös nyelve. A hálózaton belüli számítógépek nagyon különböző szoftvereket és hardvereket használhatnak; a protokollok használata azonban lehetővé teszi számukra, hogy ennek ellenére kommunikáljanak egymással.

A szabványosított protokollok olyanok, mint egy közös nyelv, amelyet a számítógépek használhatnak, hasonlóan ahhoz, ahogy a világ különböző részeiről származó két ember nem feltétlenül érti egymás anyanyelvét, de kommunikálhatnak egy közös harmadik nyelven. Ha az egyik számítógép az Internet Protocol (IP) protokollt használja, és a második számítógép is, akkor képesek lesznek kommunikálni – ahogy az Egyesült Nemzetek Szervezete is a 6 hivatalos nyelvére támaszkodik, hogy a világ minden tájáról származó képviselők kommunikálhassanak egymással. De ha az egyik számítógép IP-t használ, a másik pedig nem ismeri ezt a protokollt, akkor nem tudnak kommunikálni.

Az interneten különböző protokollok léteznek a különböző típusú folyamatokhoz. A protokollokat gyakran aszerint tárgyalják, hogy melyik OSI-modell réteghez tartoznak.

**eBook**
*Erősítse meg a biztonságot egy egységes platformmal*
[Szerezze be az eBook-ot]

**Whitepaper**
*Stratégia kidolgozása a hálózat korszerűsítéséhez*
[Olvassa el a whitepaper-t]

## Mik az OSI-modell rétegei?

Az Open Systems Interconnection (OSI) modell egy absztrakt ábrázolása annak, hogyan működik az internet. 7 réteget tartalmaz, amelyek mindegyike a hálózati funkciók egy-egy különböző kategóriáját képviseli.

![Az OSI-modell]

A protokollok teszik lehetővé ezeket a hálózati funkciókat. Például az Internet Protocol (IP) felelős az adatok útvonalválasztásáért azáltal, hogy jelzi, honnan származnak az adatcsomagok*, és mi a célállomásuk. Az IP teszi lehetővé a hálózatok közötti kommunikációt. Ezért az IP-t hálózati réteg (3. réteg) protokollnak tekintik.

Egy másik példaként a Transmission Control Protocol (TCP) biztosítja, hogy az adatcsomagok hálózatokon keresztüli szállítása zökkenőmentesen menjen. Ezért a TCP-t szállítási réteg (4. réteg) protokollnak tekintik.

*\*A csomag egy kis adatszegmens; a hálózaton keresztül küldött összes adat csomagokra van felosztva.*

**Iratkozzon fel**
*Biztonság és sebesség bármely Cloudflare csomaggal*
[Kezdje ingyen]

## Mely protokollok futnak a hálózati rétegen?

Ahogy fentebb leírtuk, az IP egy hálózati réteg protokoll, amely az útvonalválasztásért felelős. De nem ez az egyetlen hálózati réteg protokoll.

- **IPsec:** Az Internet Protocol Security (IPsec) titkosított, hitelesített IP-kapcsolatokat hoz létre virtuális magánhálózaton (VPN) keresztül. Technikailag az IPsec nem egy protokoll, hanem protokollok gyűjteménye, amely magában foglalja az Encapsulating Security Protocol (ESP), Authentication Header (AH) és Security Associations (SA) protokollokat.

- **ICMP:** Az Internet Control Message Protocol (ICMP) hibákat jelent és állapotfrissítéseket nyújt. Például, ha egy router nem tud kézbesíteni egy csomagot, ICMP üzenetet küld vissza a csomag forrásának.

- **IGMP:** Az Internet Group Management Protocol (IGMP) egy-a-többhöz hálózati kapcsolatokat hoz létre. Az IGMP segít a multicasting beállításában, ami azt jelenti, hogy több számítógép fogadhat egy IP-címre irányított adatcsomagokat.

## Milyen egyéb protokollokat használnak az interneten?

Néhány a legfontosabb ismerendő protokoll közül:

- **TCP:** Ahogy fentebb leírtuk, a TCP egy szállítási réteg protokoll, amely biztosítja a megbízható adatszolgáltatást. A TCP-t az IP-vel együtt kell használni, és a két protokollra gyakran együtt TCP/IP-ként hivatkoznak.

- **HTTP:** A Hypertext Transfer Protocol (HTTP) a World Wide Web alapja, az interneté, amellyel a legtöbb felhasználó kapcsolatba kerül. Az eszközök közötti adatátvitelre használják. A HTTP az alkalmazási réteghez (7. réteg) tartozik, mert olyan formátumba helyezi az adatokat, amelyet az alkalmazások (pl. egy böngésző) közvetlenül használhatnak, további értelmezés nélkül. Az OSI-modell alsóbb rétegeit a számítógép operációs rendszere kezeli, nem az alkalmazások.

- **HTTPS:** A HTTP-vel az a probléma, hogy nem titkosított – bármely támadó, aki elfog egy HTTP üzenetet, elolvashatja azt. A HTTPS (HTTP Secure) ezt azzal korrigálja, hogy titkosítja a HTTP üzeneteket.

- **TLS/SSL:** A Transport Layer Security (TLS) az a protokoll, amelyet a HTTPS a titkosításhoz használ. A TLS-t korábban Secure Sockets Layer (SSL) néven ismerték.

- **UDP:** A User Datagram Protocol (UDP) a TCP gyorsabb, de kevésbé megbízható alternatívája a szállítási rétegben. Gyakran használják olyan szolgáltatásokban, mint a videóstreamelés és a játékok, ahol a gyors adatszolgáltatás a legfontosabb.

## Milyen protokollokat használnak a routerek?

A hálózati routerek bizonyos protokollokat használnak a többi routerhez vezető leghatékonyabb hálózati útvonalak felfedezésére. Ezeket a protokollokat nem felhasználói adatok továbbítására használják. Fontos hálózati útvonalválasztási protokollok:

- **BGP:** A Border Gateway Protocol (BGP) egy alkalmazási réteg protokoll, amelyet a hálózatok arra használnak, hogy bejelentsék, mely IP-címeket irányítják. Ez az információ lehetővé teszi a routerek számára, hogy eldöntsék, mely hálózatokon haladjanak át az adatcsomagok a célállomásaik felé vezető úton.

- **EIGRP:** Az Enhanced Interior Gateway Routing Protocol (EIGRP) azonosítja a routerek közötti távolságokat. Az EIGRP automatikusan frissíti az egyes routerek legjobb útvonalainak nyilvántartását (útválasztási táblázatnak nevezik), és ezeket a frissítéseket továbbítja a hálózaton belüli többi routerhez.

- **OSPF:** Az Open Shortest Path First (OSPF) protokoll különféle tényezők, köztük a távolság és a sávszélesség alapján számítja ki a leghatékonyabb hálózati útvonalakat.

- **RIP:** A Routing Information Protocol (RIP) egy régebbi útválasztási protokoll, amely azonosítja a routerek közötti távolságokat. A RIP egy alkalmazási réteg protokoll.
