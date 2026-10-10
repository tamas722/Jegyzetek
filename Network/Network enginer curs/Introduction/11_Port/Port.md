# Mi az a port?

A **port** egy virtuális pont, ahol a hálózati kapcsolatok kezdődnek és végződnek. A portok szoftveralapúak, és a számítógép operációs rendszere kezeli őket. Minden port egy adott folyamathoz vagy szolgáltatáshoz kapcsolódik. A portok lehetővé teszik a számítógépek számára, hogy könnyen megkülönböztessék a különböző típusú forgalmakat: például az e-mailek más portra mennek, mint a weboldalak, még akkor is, ha mindkettő ugyanazon az internetkapcsolaton keresztül érkezik a számítógéphez.

## Mi az a portszám?

A portok szabványosítottak minden hálózatra csatlakoztatott eszközön, és minden port kap egy számot. A legtöbb port bizonyos protokollok számára van fenntartva – például minden Hypertext Transfer Protocol (HTTP) üzenet a 80-as portra megy. Míg az IP-címek lehetővé teszik, hogy az üzenetek adott eszközökre menjenek és onnan érkezzenek, a portszámok lehetővé teszik az adott eszközökön belüli konkrét szolgáltatások vagy alkalmazások célzását.

## Hogyan teszik hatékonyabbá a portok a hálózati kapcsolatokat?

Nagyon különböző típusú adatok áramlanak a számítógéphez és onnan ugyanazon a hálózati kapcsolaton keresztül. A portok használata segít a számítógépeknek megérteni, mit kell tenniük a fogadott adatokkal.

Tegyük fel, hogy Bob átküld egy MP3 hangfelvételt Alice-nak a File Transfer Protocol (FTP) használatával. Ha Alice számítógépe az MP3 fájl adatait Alice e-mail alkalmazásának adná át, az e-mail alkalmazás nem tudná értelmezni. De mivel Bob fájlátvitele az FTP-hez kijelölt portot (21-es port) használja, Alice számítógépe képes fogadni és tárolni a fájlt.

Eközben Alice számítógépe egyidejűleg HTTP weboldalakat tölthet be a 80-as port használatával, még akkor is, ha mind a weboldalfájlok, mind az MP3 hangfájl ugyanazon a WiFi kapcsolaton keresztül áramlik Alice számítógépéhez.

**Whitepaper**
*Hogyan szabaduljunk meg a hálózati hardvertől*
[Szerezze be a jelentést]
→

## A portok a hálózati réteg részei?

Az OSI-modell az internet működésének fogalmi modellje. 7 rétegre osztja a különböző internetes szolgáltatásokat és folyamatokat. Ezek a rétegek:

![OSI modell 7 réteg]

A portok egy szállítási réteg (4. réteg) koncepció. Csak egy szállítási protokoll, például a Transmission Control Protocol (TCP) vagy a User Datagram Protocol (UDP) tudja jelezni, hogy egy csomagnak melyik portra kell mennie. A TCP és UDP fejlécekben van egy szakasz a portszámok jelzésére. A hálózati réteg protokollok – például az Internet Protocol (IP) – nincsenek tisztában azzal, hogy melyik port van használatban egy adott hálózati kapcsolatban. Egy szabványos IP-fejlécben nincs hely annak jelzésére, hogy az adatcsomagnak melyik portra kell mennie. Az IP-fejlécek csak a cél IP-címet jelzik, nem pedig a portszámot azon az IP-címen.

Általában a port hálózati rétegben történő jelzésének képtelensége nincs hatással a hálózati folyamatokra, mivel a hálózati réteg protokollokat szinte mindig egy szállítási réteg protokollal együtt használják. Ez azonban hatással van a tesztelőszoftverek működésére, amelyek Internet Control Message Protocol (ICMP) csomagokkal „pingelik" az IP-címeket. Az ICMP egy hálózati réteg protokoll, amely képes pingelni a hálózatra csatlakoztatott eszközöket – de a konkrét portok pingelésének képessége nélkül a hálózati adminisztrátorok nem tudják tesztelni az adott eszközökön belüli konkrét szolgáltatásokat.

Néhány ping szoftver, például a My Traceroute, kínálja az UDP csomagok küldésének lehetőségét. Az UDP egy szállítási réteg protokoll, amely meg tudja adni egy adott portot, ellentétben az ICMP-vel, amely nem tud portot megadni. Az ICMP csomagokhoz UDP fejlécet adva a hálózati adminisztrátorok tesztelhetik a hálózatra csatlakoztatott eszközön belüli konkrét portokat.

**Iratkozzon fel**
*Biztonság és sebesség bármely Cloudflare csomaggal*
[Kezdje ingyen]
→

## Miért blokkolják néha a tűzfalak a konkrét portokat?

A tűzfal egy hálózati biztonsági rendszer, amely biztonsági szabályok alapján blokkolja vagy engedélyezi a hálózati forgalmat. A tűzfalak általában egy megbízható hálózat és egy nem megbízható hálózat között helyezkednek el; a nem megbízható hálózat gyakran az internet. Például az irodai hálózatok gyakran tűzfalat használnak, hogy megvédjék hálózatukat az online fenyegetésektől.

Egyes támadók rosszindulatú forgalmat próbálnak küldeni véletlenszerű portokra abban a reményben, hogy azok a portok „nyitva" maradtak, ami azt jelenti, hogy képesek forgalmat fogadni. Ez a művelet némileg olyan, mint egy autótolvaj, aki végigsétál az utcán, és megpróbálja kinyitni a parkoló járművek ajtaját, remélve, hogy az egyik nyitva van. Emiatt a tűzfalakat úgy kell konfigurálni, hogy blokkolják a legtöbb rendelkezésre álló portra irányuló hálózati forgalmat. A rendelkezésre álló portok túlnyomó többségének nincs legitim oka forgalom fogadására.

A megfelelően konfigurált tűzfalak alapértelmezés szerint blokkolják az összes portra irányuló forgalmat, kivéve néhány előre meghatározott, köztudottan gyakran használt portot. Például egy vállalati tűzfal csak a 25-ös (e-mail), 80-as (webes forgalom), 443-as (webes forgalom) és néhány más portot hagyhat nyitva, lehetővé téve a belső alkalmazottak számára ezen alapvető szolgáltatások használatát, majd blokkolja a többi 65 000+ portot.

Konkrétabb példaként a támadók néha megpróbálják kihasználni az RDP protokoll sebezhetőségeit azáltal, hogy támadó forgalmat küldenek a 3389-es portra. E támadások megállításához a tűzfal alapértelmezés szerint blokkolhatja a 3389-es portot. Mivel ezt a portot csak távoli asztali kapcsolatokhoz használják, egy ilyen szabály kis hatással van a napi üzleti műveletekre, kivéve, ha az alkalmazottaknak távolról kell dolgozniuk.

## Milyen különböző portszámok léteznek?

65 535 lehetséges portszám létezik, bár nem mindegyik van gyakori használatban. Néhány a leggyakrabban használt portok közül, a hozzájuk kapcsolódó hálózati protokollal együtt:

- **20-as és 21-es port:** File Transfer Protocol (FTP). Az FTP fájlok átvitelére szolgál egy kliens és egy szerver között.

- **22-es port:** Secure Shell (SSH). Az SSH számos alagútprotokoll egyike, amelyek biztonságos hálózati kapcsolatokat hoznak létre.

- **25-ös port:** Történelmileg Simple Mail Transfer Protocol (SMTP). Az SMTP-t e-mailhez használják.

- **53-as port:** Domain Name System (DNS). A DNS a modern internet elengedhetetlen folyamata; az ember által olvasható domainneveket géppel olvasható IP-címekhez párosítja, lehetővé téve a felhasználók számára, hogy weboldalakat és alkalmazásokat töltsenek be anélkül, hogy hosszú IP-címlistát kellene megjegyezniük.

- **80-as port:** Hypertext Transfer Protocol (HTTP). A HTTP az a protokoll, amely lehetővé teszi a World Wide Webet.

- **123-as port:** Network Time Protocol (NTP). Az NTP lehetővé teszi a számítógépes órák szinkronizálását egymással, ami elengedhetetlen folyamat a titkosításhoz.

- **179-es port:** Border Gateway Protocol (BGP). A BGP elengedhetetlen a hatékony útvonalak kialakításához az internetet alkotó nagy hálózatok között (ezeket a nagy hálózatokat autonóm rendszereknek nevezik). Az autonóm rendszerek a BGP-t használják annak bejelentésére, hogy mely IP-címeket irányítják.

- **443-as port:** HTTP Secure (HTTPS). A HTTPS a HTTP biztonságos és titkosított változata. Minden HTTPS webes forgalom a 443-as portra megy. Az HTTPS-t titkosításra használó hálózati szolgáltatások, például a DNS over HTTPS, szintén ezen a porton kapcsolódnak.

- **500-as port:** Internet Security Association and Key Management Protocol (ISAKMP), amely a biztonságos IPsec kapcsolatok beállítási folyamatának része.

- **587-es port:** Modern, biztonságos SMTP, amely titkosítást használ.

- **3389-es port:** Remote Desktop Protocol (RDP). Az RDP lehetővé teszi a felhasználók számára, hogy távolról csatlakozzanak asztali számítógépeikhez egy másik eszközről.

Az Internet Assigned Numbers Authority (IANA) tartja nyilván a portszámok és a hozzájuk rendelt protokollok teljes listáját.
