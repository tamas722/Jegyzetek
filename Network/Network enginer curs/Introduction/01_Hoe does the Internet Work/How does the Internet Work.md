# Hogyan működik az Internet?

## 1. Mi az az Internet?

**Definíció:** Az internet a hálózatok globális hálózata, amely szabványosított protokollokon keresztül kommunikál.

**Elosztott rendszer:** Nincs központi irányítása vagy egyetlen meghibásodási pontja. Ha egy szerver vagy adatközpont leáll, a hálózat többi része továbbra is működőképes marad.

**Adattovábbítás:** Az információk bitek (fény- vagy elektromos impulzusok) formájában utaznak.

## 2. Fizikai infrastruktúra és hardver

Az adatok különböző közegeken keresztül áramlanak (rézkábelek, üvegszálas optikai kábelek, rádióhullámok/Wi-Fi). A hálózatot speciális hardverek működtetik:

- **Routerek (Útválasztók):** Az internet "közlekedési rendőrei". Célállomásuk alapján a megfelelő hálózatokba irányítják az adatcsomagokat.
- **Switchek (Kapcsolók):** Egy adott hálózaton (pl. helyi hálózat) belül kötik össze az eszközöket, és továbbítják a csomagokat a megfelelő géphez.
- **Webszerverek:** Nagy teljesítményű számítógépek, amelyek tartalmakat (weboldalakat, képeket) és adatbázisokat tárolnak, és kiszolgálják a felhasználói kéréseket.

## 3. A működés alapjai: Csomagok és Protokollok

A két legfontosabb koncepció, ami lehetővé teszi az internet működését:

### A) Csomagok (Packets) és Csomagkapcsolás (Packet switching)

- Az adatokat nem egyben, hanem apró darabokra (csomagokra) vágva továbbítják.
- Minden csomagnak van egy "fejléce" (header), amely tartalmazza a célállomás információit és az összeszerelési utasításokat.
- A csomagok egymástól függetlenül, akár különböző útvonalakon is haladhatnak (ami növeli a megbízhatóságot). A célállomáson a gép újra sorba rendezi őket.

### B) Protokollok

Szabványosított "nyelvek" vagy szabályrendszerek, amelyek biztosítják, hogy a különböző hardverek és szoftverek megértsék egymást.

- **IP (Internet Protocol):** Hálózatok közötti csomagtovábbítás.
- **TCP:** Biztosítja, hogy a csomagok sorrendben és hiánytalanul érkezzenek meg.
- **HTTP:** Weboldalak adatainak formázása és továbbítása.

## 4. Hogyan töltődik be egy weboldal? (Folyamat)

Amikor megnyitsz egy oldalt, az alábbi folyamat játszódik le milliszekundumok alatt:

1. **DNS lekérdezés (Domain Name System):** A böngésző megkeresi a beírt webcímhez (pl. cloudflare.com) tartozó IP-címet (olyan, mint egy internetes telefonkönyv).
2. **TCP kézfogás (Handshake):** Felépül a kapcsolat a géped és a szerver között.
3. **TLS kézfogás:** Létrejön egy titkosított (kriptográfiai) csatorna, hogy illetéktelenek ne olvashassák az adatokat (SSL/TLS alapú biztonság).
4. **HTTP kérés:** A böngésző kikéri a weboldal tartalmát.
5. **HTTP válasz:** A szerver adatcsomagokban visszaküldi a kódokat (HTML, CSS, JavaScript), amelyeket a böngésződ értelmez, és megjeleníti a képernyőn.

## 5. Kriptográfia és Kiberbiztonság

**Biztonság:** Az adatok titkosítása (nyilvános és privát kulcsok, SSL/TLS) elengedhetetlen a biztonságos kommunikációhoz.

**Kiberbiztonság:** Azok a védelmi intézkedések, technológiák és protokollok, amelyek megakadályozzák a kiberbűnözést, a hálózatok lehallgatását és az adatlopást.
