# Mi az az IP-cím?

**Írta: Coursera Staff • Frissítve: 2025. okt. 20.**

Megosztás

Fedezd fel az IP-cím történetét és fontosságát, hogyan működik, a nyilvános és privát IP-címek közötti különbséget, és hogy miért van szükségünk rájuk az interneten való kommunikációhoz.

![Kiemelt kép] Egy kockás inget viselő személy egy íróasztalnál ül, és egy laptopot valamint egy külön kijelzőt használva kódot néz át.

Az **IP-cím** (internet protocol address) egy egyedi szám minden olyan eszköz számára, amely csatlakozik az internethez. Az IP-címek lehetővé teszik, hogy az eszközök a világ bármely pontjáról csatlakozzanak az internethez. Az információ az interneten egyedi darabokban, úgynevezett **csomagokban** kerül továbbításra, amelyek tartalmazzák a kért adatot és egy metaadatokat tartalmazó fejlécet. A routereknek és hálózati rendszereknek azonosítaniuk kell mind a csomag kérőjét, mind a csomag küldőjét, hogy a kért adat a megfelelő helyről kerüljön elküldésre és a megfelelő kérőhöz érkezzen meg. Az IP-cím azonosítja a kérőt és a küldőt, amelyek mindegyike egyedi IP-címmel rendelkezik.

## Mi alkotja az IP-címet?

Az **IPv4-cím** egy 32 bites hosszúságú azonosítót használ, míg az **IPv6-cím** akár 128 bitet is használhat az azonosítóban. A **bit** egy bináris számjegy, amely vagy egy, vagy nulla. Az IP-címek tartalmaznak egy **hálózati előtagot** és egy **hostszámot**. Az IP-címek a **pontozott decimális rendszer** alapján vannak címkézve, amely négy oktettet tartalmaz, mindegyik nyolc bittel.

Íme egy példa egy IP-címre bináris formában, pontozott decimálissá alakítva:

- **Bináris:** 11100000.10000000.11110000.00001000
- **Pontozott decimális:** 224.128.240.8

1993-ig a hálózati előtagban lévő bitek száma osztályokba sorolta az IP-címeket. A három fő osztály a következő volt:

- **A osztály**, amely több mint 65 536 hostazonosítót tartalmaz
- **B osztály**, amely 256 és 65 534 közötti hostazonosítót tartalmaz
- **C osztály**, amely kevesebb mint 254 hostazonosítót tartalmaz

1993-ban az Internet Engineering Task Force létrehozta a **CIDR** (Classless Inter-Domain Routing) jelölést, az osztályalapú jelölés hiányosságai és az IP-címek végleges kimerülésének veszélye miatt. A CIDR lehetővé teszi az IP-címzés olyan módszerét, amely az osztályalapú IP-címeket alhálózatok (egy hálózat, amely több alhálózatra, azaz subnetre van felosztva) sorozatára bontja alhálózati maszkokon keresztül. Ez hatékonyabbá teszi az adott számú hosthoz szükséges IP-címek felhasználását.

## IP-cím jelölés

Az osztályalapú IP-cím jelölés egy hálózati előtagon és egy hostszámon alapul. Egy kulcsot használnak annak azonosítására, hogy egy adott IP-cím melyik osztályba tartozott. Az osztályalapú IP-k hálózati előtagjai és hostszámai a következők:

- **A osztály:** Hálózati előtag 8 bit, hostszám 24 bit
- **B osztály:** Hálózati előtag 16 bit, hostszám 16 bit
- **C osztály:** Hálózati előtag 24 bit, hostszám 8 bit

Mivel a CIDR jelölés figyelmen kívül hagyja az osztályok használatát, megmutatja egy cím hálózati maszkját, és egy perjel és egy szám segítségével jelzi a bitek teljes számát. Például a **/16** azt jelenti, hogy 16 bit van az adott hálózati maszkban. A CIDR jelölés szintén mellőzi az IP-címek bináris jelöléssel való kiírásának szükségességét, hozzáférhetőbbé és gyorsabban olvashatóvá téve azokat. Az internetszolgáltatók (ISP-k) IP-címblokkokat vásárolnak, hogy eladják azokat ügyfeleiknek.

## Nyilvános vs. privát IP-címek

A router összeköti a helyi rendszert (privát IP-cím, beleértve a routert) a rendszeren kívüli világgal (nyilvános IP-címek). A privát címek csak a helyi hálózatban irányítják a csomagokat. A privát címen kívüli bármely forgalom egy „nyilvános" IP-címhez csatlakozik. A router megmondja minden csomagnak, hogy hová menjen a helyi hálózat és a nyilvános internet között nyilvános és privát IP-címeken keresztül. Ez az útválasztás konkrét használata csak IPv4-címekkel használatos, hogy maximalizálja a használatban lévő véges számú IP-címet. Az IPv6 bevezetése megszüntetné az IP-címek „nyilvános" és „privát" útválasztásának szükségességét, mivel olyan sok IPv6-cím létezhet.

Az a tény, hogy a helyi hálózatod (otthon vagy vállalat) egy „privát" IP-cím, nem jelenti azt, hogy csak te ismered az IP-címedet. Amikor csak az internetet használod, a nyilvános IP-címed megjelenik az adott weboldal számára. A támadók megsérthetik ezeket a szervernaplókat az IP-címekkel, és potenciálisan elosztott szolgáltatásmegtagadási (DDoS) támadást hajthatnak végre a hálózatod ellen. A hackerek által végrehajtott DDoS-támadás arra összpontosít, hogy túlterhelje a hálózati berendezéseket kérésekkel, hogy a normál internethasználók szolgáltatáskiesést tapasztaljanak. Fontos, hogy soha ne add meg az otthoni vagy vállalati IP-címedet senkinek online, hogy mérsékeld a DDoS-támadásokat és a személyazonosságod elleni támadásokat.
