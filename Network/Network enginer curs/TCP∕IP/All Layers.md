4. Alkalmazási réteg (Application Layer)
Ez a réteg biztosítja a felületet a hálózaton kommunikáló szoftverek és a hálózati protokollok között. Magába foglalja az OSI modell felső három rétegét (Alkalmazási, Megjelenítési, Viszony/Session).

Feladata: A felhasználói adatok előállítása, formázása, kódolása, tömörítése és titkosítása (pl. TLS/SSL), valamint a kommunikációs munkamenetek kezelése.

PDU (Protokoll adategység): Adat (Data) vagy Üzenet (Message).

Kulcsfontosságú protokollok:

Web és adatátvitel: HTTP/HTTPS (port 80/443), FTP (port 20/21), TFTP (port 69).

Levelezés: SMTP (port 25), POP3 (port 110), IMAP (port 143).

Hálózatkezelés és feloldás: DNS (port 53), DHCP (port 67/68), SNMP (port 161).

Távoli elérés: SSH (port 22), Telnet (port 23).

Működési elv: Amikor a böngésződben megnyitsz egy oldalt, ez a réteg generálja a HTTP GET kérést, és ezt az adatot adja tovább lefelé a szállítási rétegnek (payloadként).

3. Szállítási réteg (Transport Layer)
Ez a réteg felel a végpontok (hostok) közötti logikai kommunikációért és az adatok megbízható vagy gyors továbbításáért.

Feladata: Az alkalmazási rétegtől kapott nagy méretű adatok feldarabolása, a kapcsolatok felépítése, hibajavítás, és a különböző alkalmazások adatfolyamainak szétválasztása (multiplexálás).

PDU: Szegmens (TCP esetén) vagy Datagram (UDP esetén).

Címzés: Portszámok (16 bites azonosítók, 0-65535 között), amelyek azonosítják a forrás és cél szolgáltatást.

Kulcsfontosságú protokollok:

TCP (Transmission Control Protocol): Kapcsolatorientált protokoll.

3-utas kézfogás (3-way handshake): A kommunikáció megkezdése előtt felépíti a kapcsolatot (SYN -> SYN-ACK -> ACK).

Megbízhatóság és sorszámozás: Minden szegmens kap egy sorszámot (Sequence Number). A fogadó fél nyugtázza (ACK) a megkapott adatokat. Ha egy csomag elvész, a küldő újra elküldi.

Forgalomszabályozás (Flow control): Ablakozás (Windowing) segítségével szabályozza, mennyi adatot küldhet a forrás anélkül, hogy túlterhelné a fogadót.

UDP (User Datagram Protocol): Kapcsolatmentes protokoll. Nincs kézfogás, nincs nyugtázás, nincs sorszámozás. Gyors, de nem megbízható (best-effort). Ideális valós idejű alkalmazásokhoz (VoIP, videó streaming, online játékok), ahol az adatvesztés kevésbé kritikus, mint a késleltetés.

2. Internet réteg (Internet Layer)
Ez a réteg felelős az adatok logikai megcímzéséért és a hálózatok (például a te otthoni hálózatod és egy felhős adatközpont) közötti optimális útvonal megtalálásáért (routing).

Feladata: A szállítási rétegtől kapott szegmensek IP-fejléccel való ellátása, az optimális útválasztás (routing) a hálózatok között, valamint szükség esetén a csomagok darabolása (fragmentation), ha azok mérete meghaladja a hálózati réteg maximális átviteli egységét (MTU).

PDU: Csomag (Packet).

Címzés: Logikai IP-cím (IPv4 esetén 32 bit, IPv6 esetén 128 bit).

Kulcsfontosságú protokollok:

IPv4 / IPv6: Az adatok tényleges logikai továbbítását végző protokollok. (A fejlécek tartalmazzák a Forrás és Cél IP címet, a TTL - Time to Live mezőt, amely megakadályozza a végtelen hurkokat, stb.)

ICMP (Internet Control Message Protocol): Hibaüzenetek és diagnosztika (erre épül a ping és a traceroute).

IPsec: Az IP csomagok titkosítását és hitelesítését végző protokollkészlet (VPN-ek alapja).

Működési elv: A routerek (L3 eszközök) ezen a rétegen működnek. Kibontják az adatcsomagot az L3 fejlécig, megnézik a Cél IP-címet, és az útválasztó táblájuk (Routing Table) alapján eldöntik, melyik interfészen küldjék tovább az adatot.

1. Hálózati hozzáférési réteg (Network Access Layer)
A fizikai hálózathoz való csatlakozásért és a bitek tényleges továbbításáért felel. (Mivel ezt az előző válasz részletesen tárgyalta, itt a teljesség kedvéért rövidebben, rendszerbe ágyazva szerepel).

Feladata: Az IP-csomagok keretekbe (Frame) foglalása, fizikai címzés biztosítása egy adott hálózati szegmensen belül, hibaészlelés (FCS - Frame Check Sequence), és az adatok bitekké (0 és 1) alakítása a fizikai médián (réz, optika, rádió).

PDU: Keret (Frame) az adatkapcsolati szinten, és Bit a fizikai szinten.

Címzés: MAC-cím (48 bites, a hálózati kártyába égetett fizikai azonosító).

Protokollok: Ethernet (802.3), Wi-Fi (802.11), ARP (az IP címeket fordítja MAC címekké).

Hardverek: Switch-ek (L2), Access Pointok, Hálózati kártyák (NIC), kábelek.

Az adatbeágyazás (Encapsulation) folyamata röviden:
Amikor adatot küldesz, a folyamat felülről lefelé halad:

Az Alkalmazási réteg átadja az Adatot.

A Szállítási réteg rátesz egy TCP/UDP fejlécet (Portszámokkal) -> létrejön a Szegmens.

Az Internet réteg rátesz egy IP fejlécet (IP-címekkel) -> létrejön a Csomag.

A Hálózati hozzáférési réteg rátesz egy Ethernet fejlécet (MAC-címekkel) és egy lezárót -> létrejön a Keret.

A hálózati kártya a keretet bitekké (elektromos jelekké) alakítja, és kiküldi a kábelre.

A fogadó oldalon pontosan ez a folyamat játszódik le visszafelé (Decapsulation), rétegről rétegre hámozva le a fejléceket, amíg az adat el nem jut a célalkalmazáshoz.
