# Lecke: A TCP/IP Protokollcsalád és a Hálózati Modellek

Üdvözöllek a Jeremy's IT Lab CCNA tanfolyamán! Ez az anyag az Internet Protocol Suite-ról, más néven a TCP/IP-ről szól. Ez az a protokollcsalád, amely lehetővé teszi a kommunikációt az Interneten és más hálózatokon. Valószínűleg hallottál már néhány ilyen protokollról: IP, TCP, UDP, HTTP stb. Ha nem, az sem gond, a kurzus során részletesen kitérünk rájuk.

Ebből a protokollcsaládból építjük fel azt, amit általában TCP/IP modellnek hívnak: ez egy módszer a protokollok rétegekbe (layer) rendezésére a feladatuk alapján. Ez a modell egyfajta térképként szolgál ahhoz, hogy megértsük, "ki mit csinál" egy hálózatban. Ahogy később tanulsz az IP-címzésről, a router-ekről, switch-ekről, képes leszel minden új témát beilleszteni ebbe a keretrendszerbe.

## Protokollok és Szabványok (Protocols and Standards)

A protokoll egy olyan szabályrendszer, amely meghatározza, hogyan kell az adatokat továbbítani az eszközök között egy hálózaton. A protokollok lényegében a számítógépek "nyelvei". Ahogy egy csak angolul beszélő ember nem tud kommunikálni egy csak kínaiul beszélővel, a különböző protokollokat "beszélő" számítógépek sem tudnak adatot cserélni.

A hálózatépítés hőskorában gyakori volt, hogy a gyártók (például az IBM vagy az Apple) saját, zárt (proprietary) protokollokat fejlesztettek, amelyek csak a saját termékeikkel működtek. Ennek eredményeként egy IBM szerver és egy Apple kliens nem tudott szót érteni egymással.

Ezt a problémát úgy oldották meg, hogy a mai hálózatok gyártófüggetlen (vendor-neutral) szabványokat (standards) használnak. Egy szabvány egy elfogadott specifikáció arról, hogyan kell működnie egy technológiának. Ennek köszönhetően egy Windows-os PC küldhet e-mailt egy Androidos telefonnak, és egy Apple MacBook is meg tud nyitni egy Linux server-en hosztolt weboldalt.

## Egy kis történelem

A mai Internet elődjeinek fejlesztése az 1960-as években kezdődött az Egyesült Államokban. A Védelmi Minisztérium ARPA (Advanced Research Projects Agency) nevű részlege finanszírozta az ARPANET-et, amely 1969-ben indult el, hogy egyetemi és laboratóriumi számítógépeket kössön össze a keleti és nyugati parton.

Az ARPANET eleinte egy NCP (Network Control Program) nevű protokollt használt. A TCP/IP ekkor még nem létezett.

1974-ben Vint Cerf és Bob Kahn elkezdték fejleszteni a TCP-t (Transmission Control Program). Ezt később két különálló protokollra bontották, amelyeket ma is használunk:

- **TCP:** Transmission Control Protocol (A "Program" szót "Protocol"-ra cserélték).
- **IP:** Internet Protocol.

Ebből a kettőből született meg a TCP/IP protokollcsalád. Az ARPANET hivatalosan 1983. január 1-jén tért át a TCP/IP használatára. A TCP/IP azért tudott győzedelmeskedni a zárt rendszerek felett, mert nyílt szabványként bárki implementálhatta, és sokféle hálózaton képes volt futni.

## Kik alkotják a szabványokat?

A hálózati szabványokat független szervezetek készítik mérnökök bevonásával. Két fő szervezetet kell ismerned:

- **IEEE (Institute of Electrical and Electronics Engineers):** Ők fejlesztik a LAN (helyi hálózat) technológiákat, például az Ethernet-et (802.3-as szabványok) és a Wi-Fi-t (802.11-es szabványok). Ezek a fizikai specifikációkra (kábelek, rádiófrekvenciák) és az üzenetek formázására fókuszálnak.
- **IETF (Internet Engineering Task Force):** Ez egy nyílt közösség, amely az Interneten használt protokollokat (TCP, IP, UDP, HTTP, DNS) definiálja. Szabványaikat RFC (Request for Comments) dokumentumokban teszik közzé (pl. az IP-t az RFC 791 definiálja 1981-ből), amelyek bárki számára ingyenesen elérhetők.

## Rétegzett Modellek (Layered Models)

A hálózatok rengeteg feladatot látnak el az adat célba juttatása során (fizikai jeltovábbítás, helyi kézbesítés, hálózatok közötti irányítás, alkalmazások kezelése). A modell segítségével a rokon feladatokat rétegekbe (layer) csoportosítjuk. Minden layer-nek megvan a maga szerepe: felhasználja az alatta lévő layer szolgáltatásait, és szolgáltatást nyújt a felette lévőnek. Ezek a protokollok együtt egy úgynevezett network stack-et (hálózati vermet) alkotnak.

### Analógia: Levélküldés a postán

Hogy könnyebb legyen elképzelni, gondoljunk a hagyományos postai levélküldésre, ha írsz egy levelet a barátodnak, Bobnak:

- **Tartalom (Content):** Maga a levél szövege, amit mondani akarsz.
- **Címzett (Recipient):** A "Címzett: Bob" felirat, ami megmondja, ki olvassa el a levelet a házon belül.
- **Cím (Address):** A ház pontos postacíme a borítékon.
- **Helyi kézbesítés (Local Delivery):** A postahivatalok közötti mozgatás postásautókkal.
- **Infrastruktúra (Infrastructure):** Az utak, amelyeken a járművek haladnak.

Ezek a rétegek egymásra épülnek, de különállóak. Ha megváltoztatod a levél tartalmát, az nem befolyásolja az utakat; ha repülővel viszik a levelet autó helyett, attól a ház címe még ugyanaz marad. Ugyanez a modularitás igaz a hálózatokra is!

## A TCP/IP 5-rétegű Modellje

Az analógiánkat átültetve a TCP/IP modell 5 rétegére, ha egy kliens PC (PC1) egy weboldalt kér le egy szerverről (SRV1):

- **Layer 5 - Application layer (Alkalmazási réteg):** Ez felel az adatok formázásáért és értelmezéséért az alkalmazások között. Ha a PC1-en futó Chrome egy weblapot kér, egy olyan protokollt használ, mint a HTTP vagy HTTPS (vagy fájlküldésnél FTP, TFTP). A hálózati eszközöket (pl. routereket) ez a szint nem érdekli, csak a küldő és a fogadó gép értelmezi az Application-layer adatait.
- **Layer 4 - Transport layer (Szállítási réteg):** Ez biztosítja az alkalmazások (folyamatok) közötti végpont-végpont (end-to-end) kommunikációt. A szerveren több szolgáltatás is futhat (pl. webszerver és fájlszerver). A szétválasztásukra a Transport layer port számokat használ. (Például a 80-as port a webszerveré, a 21-es a fájlszerveré). A legfontosabb protokollok itt a TCP és az UDP.
- **Layer 3 - Internet layer (Internet réteg):** Ez a réteg a hálózatok közötti végpont-végpont kézbesítésért felel, IP-címek használatával. Ha PC1 üzenetet küld SRV1-nek, megcímezi SRV1 IP-címére (pl. 10.1.1.1). A hálózatban a router-ek ezen a rétegen operálnak: megnézik a cél IP-címet, és az alapján továbbítják az üzenetet. Protokollok: IP (IPv4 és IPv6) és ICMP.
- **Layer 2 - Local Network / Data Link layer (Helyi hálózat / Adatkapcsolati réteg):** Ez felel a "hop-to-hop" (ugrásról ugrásra) történő kézbesítésért a helyi hálózaton (LAN) belül. Amíg az IP-cím a végső célt jelöli, a MAC-címek itt azt határozzák meg, hogy ki a következő állomás (pl. PC1-től az első Routerig). A switch-ek ezen a rétegen dolgoznak. Fő protokollok: Ethernet és Wi-Fi.
- **Layer 1 - Physical layer (Fizikai réteg):** Ez a réteg továbbítja a biteket (0-k és 1-esek) elektromos, optikai vagy rádiós jelekként a fizikai közegen (UTP kábel, optikai szál, Wi-Fi antenna, NIC - hálózati kártya). Itt nincsenek logikai címek, csak a fizikai kommunikáció.

## Adatbecsomagolás és Kicsomagolás (Encapsulation & Decapsulation)

<img width="730" height="419" alt="capsulation-tcpip-model" src="https://github.com/user-attachments/assets/288c66ef-de2a-407e-b5d2-3f4510aea758" />


Hogyan kerül bele ez a sok információ egyetlen üzenetbe? Ezt a folyamatot hívják encapsulation-nek (becsomagolás).

### Encapsulation (A küldő oldalon - lefelé a stacken)

Amikor az alkalmazás (Layer 5) előállítja az adatot (pl. HTTP kérés), az elindul lefelé a hálózati vermen (stack).

- Minden réteg hozzáad egy saját fejlécet (header), ami az adott réteghez szükséges információkat (pl. port szám, IP-cím) tartalmazza.
- A Transport layer (Layer 4) hozzáadja az L4 headert.
- Az Internet layer (Layer 3) hozzáadja az L3 headert.
- A Local Network layer (Layer 2) hozzáad egy L2 headert és egy trailer-t (utótagot, amit a hibák ellenőrzésére használnak).
- Végül a Physical layer (Layer 1) a teljes csomagot bitekként kisugározza a kábelre.

### Decapsulation (A fogadó oldalon - felfelé a stacken)

Amikor a fogadó gép (vagy router) megkapja a jeleket (Layer 1), felfelé küldi a stacken. A Layer 2 megnézi az L2 headert és trailert, majd eltávolítja azokat (ez a decapsulation). Ezután továbbadja a Layer 3-nak, amely feldolgozza és leveszi az L3 headert, és így tovább, amíg a tiszta adat meg nem érkezik a fogadó alkalmazásához (Layer 5).

## PDU (Protocol Data Unit) és a Payload

Az üzenetnek az adatbecsomagolás minden szakaszában saját neve van. Ezeket összefoglaló néven PDU-nak (Protocol Data Unit) nevezzük.

- **Layer 4 PDU:** Ha TCP-t használunk, a neve Segment (szegmens). Ha UDP-t használunk, a neve Datagram. (Tehát TCP -> Segment, UDP -> Datagram).
- **Layer 3 PDU:** A Layer 3 headerrel kiegészített üzenet neve a Packet (Csomag). Ez a leggyakrabban használt kifejezés, bár szigorúan véve csak erre a szintre vonatkozik.
- **Layer 2 PDU:** Az L2 headerrel és trailerrel kiegészített üzenet neve a Frame (Keret). A vezetéken valójában mindig csak frame-ek utaznak.
- **Payload:** A PDU belsejében lévő tartalom (a fejlécek nélkül).
  - A Layer 4 payload-ja maga az Application adat.
  - A Layer 3 (Packet) payload-ja az egész Layer 4-es Segment/Datagram.
  - A Layer 2 (Frame) payload-ja pedig az egész Layer 3-as Packet.

## Rétegek Közötti Interakciók

- **Adjacent-layer interaction (Szomszédos rétegek interakciója):** A stacken belül egy adott réteg kiszolgálja a felette lévőt, és igénybe veszi az alatta lévő szolgáltatását. (Pl. a Layer 2 az IP csomagok továbbításával kiszolgálja a Layer 3-at).
- **Same-layer interaction (Azonos rétegek interakciója):** Bár az adatok fizikailag le- és felfelé haladnak, logikailag egy eszköz adott rétege a másik eszköz ugyanazon rétegével "beszélget" (pl. az egyik gép Transport rétege a másik gép Transport rétegének küld portszámot).

A modularitás nagy előnye a Separation of layers (a rétegek szétválasztása). Lecserélhetjük a vezetékes Ethernetet Wi-Fi-re (L1 és L2) anélkül, hogy a felsőbb rétegekben futó IP-nek (L3) vagy a TCP-nek (L4) bármit is tudnia kéne erről.

## A TCP/IP és az OSI Modell Összehasonlítása

Bár ma szinte minden hálózat a TCP/IP-t használja, létezik egy másik történelmi modell is. Az 1970-es és 80-as években az ISO (International Organization for Standardization) megalkotta a 7-rétegű OSI (Open Systems Interconnection) modellt és protokollcsaládot.

Céljuk egy univerzális, nyílt szabvány létrehozása volt, de a bürokratikus, felülről lefelé (top-down) irányított fejlesztés miatt túl későn és túl bonyolultan készültek el, így az alulról felfelé építkező TCP/IP nyert a való világban.

<img width="2003" height="913" alt="osi-tcpip-diagram" src="https://github.com/user-attachments/assets/dcbc5d4c-4e1c-4747-9219-7e7faf0e0a14" />


Amiért mégis fontos az OSI: Az oktatásban és a szakmai nyelvben az OSI modell 7 rétege és az elnevezései maradtak fenn!

- Bár a TCP/IP modellünk 5 rétegű, az alkalmazási réteget sokszor Layer 7-nek hívják (az OSI modell 7. rétege után).
- A TCP/IP Layer 3-ját az OSI nyelvén Network layer-nek (Hálózati réteg) hívják.
- A TCP/IP Layer 2-jét az OSI nyelvén Data Link layer-nek (Adatkapcsolati réteg) hívják.

Ezek a fogalmak folyamatosan vissza fognak térni a kurzus során (Layer 1, Layer 2, Layer 3 stb.), ezért fontos alaposan megérteni, melyik réteg miért felel.
