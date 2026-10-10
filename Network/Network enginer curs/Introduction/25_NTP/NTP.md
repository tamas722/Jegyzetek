# Network Time Protocol (NTP)


Az NTP időszinkronizálást biztosít csomagalapú hálózaton keresztül. Nagyon fontos megérteni, hogy az alkalmazástól függ, melyik protokollt kell kiválasztani. Ezen protokoll mellett létezik a Precision Time Protocol (PTP) is.

Ez a cikk az NTP protokoll működési elvét, példáit és kiemeléseit ismerteti. Az itt megadott leírás lehetővé teszi előnyeinek megértését, és segít kiválasztani az igényeidnek megfelelő megoldást.



## Áttekintés

Az **NTP (Network Time Protocol)** egy közkincs szoftvercsomag, amely az azonos nevű TCP/IP hálózati protokollt valósítja meg. Az 1980-as években Dave L. Mills indította útjára, azzal a céllal, hogy rendkívül pontos időszinkronizálást érjen el a számítógépek között hálózatokon keresztül. Ez a protokoll az algoritmusaival együtt több RFC-ben is részletezésre került, és folyamatos optimalizáláson esett át. Ma világszerte használják, és akár nanoszekundumos időpontosságot kínál. A maximálisan elért pontosság azonban az operációs rendszertől és a hálózati teljesítménytől is függ.

Jelenleg két verzió van használatban, amelyek keverhetők: a **v3**, a legújabb stabil verzió, amely számos operációs rendszerrel kompatibilis, és a **v4**, amely fejlesztéseket és jobb támogatást kínál egyes rendszerekhez. Emellett létezik egy egyszerűsített protokollverzió, az **SNTP (Simple Network Time Protocol)**. Az SNTP ugyanazt a TCP/IP csomagszerkezetet használja, de az egyszerűbb algoritmusok miatt jelentősen kisebb pontosságot kínál. A szoftver tartalmaz egy háttérszolgáltatást, amely a számítógép rendszeridejét külső referenciakforrásokkal szinkronizálja, például más hálózati eszközökkel vagy egy csatlakoztatott rádióórával.

Az NTP csomag eszközöket is tartalmaz az időszinkronizálás állapotának vezérléséhez és monitorozásához, valamint átfogó HTML dokumentációt.

## 2. Network Time Protocol globális szabvány (RFC 1305, 5905)

Az NTP egy régi internetes protokoll, amelyet még mindig széles körben használnak az idő terjesztésére. Egyszerű módot kínál az összes eszköz szinkronizálására egy normál hálózaton, sőt a nyilvános interneten keresztül is. A megbízható idő biztosításához egy helyi hálózatban a legjobb megoldás egy NTP szerver elhelyezése a hálózatban, amely egy GNSS antennához csatlakozik. Míg az időre órákhoz, beléptető rendszerekhez és más ilyen rendszerekhez van szükség, a pontosság elegendő. Előnye a robusztussága és az a képessége, hogy szabványos IT-berendezéseken fut.

A **User Datagram Protocol (UDP)**-t használja a kérések és válaszok továbbítására. Az NTP három különböző módot támogat.

### 2.1 Szinkronizálási elv – NTP Unicast

![Időbélyeg NTP Unicast módban]

Ahhoz, hogy az óráját egy távoli szerverrel szinkronizálja, a kliensnek ki kell számítania a **round-trip delay time**-t (oda-vissza késleltetési időt) és az **offset**-et (eltolást). A round-trip delay kiszámítása:

![Network Time Protocol (NTP)]

ahol

- **t0** a kliens időbélyege a kérés csomag elküldésekor,
- **t1** a szerver időbélyege a kérés csomag fogadásakor,
- **t2** a szerver időbélyege a válasz csomag elküldésekor,
- **t3** a kliens időbélyege a válasz csomag fogadásakor.

Ezért

- **t3 − t0** a kliens oldalán eltelt idő a kérés csomag kibocsátása és a válasz csomag fogadása között,
- **t2 − t1** az az idő, amelyet a szerver várt a válasz elküldése előtt.

A **θ offset** kiszámítása:

![Network Time Protocol (NTP)]

A szinkronizálás akkor helyes, ha a kliens és a szerver közötti bejövő és kimenő útvonalak névleges késleltetése szimmetrikus.

#### 2.1.1 DHCP vagy manuális IP-cím hozzárendelés

A **Dynamic Host Configuration Protocol (DHCP)** lehetővé teszi, hogy a hálózati konfigurációt egy szerver rendelje hozzá a kliensekhez.

**Hálózat DHCP szerverrel:**
Ha egy DHCP szerver elérhető egy Unicast hálózatban, a végberendezések, például órák egyszerűen csatlakoztathatók egy switch/routerhez, és a hálózati paraméterek és a szerver cím(ek) automatikusan hozzárendelődnek.

**Hálózat DHCP szerver nélkül:**
Ha nincs DHCP szerver, akkor olyan kezelőszoftverre van szükség, mint a MOBA-NMS (Network Management System). Ez a szoftver a végberendezések manuális konfigurálására használható.

### 2.2 Szinkronizálási elv – NTP Multicast

A multicast a broadcast egy típusa, de csak egy meghatározott végberendezés-csoport számára. A végberendezések egy dummy IP-címmel regisztrálják csoporttagságukat az **IGMP (Internet Group Management Protocol)** segítségével a switchen vagy routeren. A switch/router nyilvántartja, melyik portra kell melyik multicast csomagokat küldenie. Ez az időszinkronizálási típus csak minimális terhelést jelent a hálózatra, mivel a kommunikáció csak egy irányban történik, és csak az ugyanabban a multicast csoportban lévő résztvevőkhöz.

**Multicast definíciók:**

- A kliensek csatlakoznak egy csoporthoz, és „hallgatóznak" a csoportcímre
- A klienseknek nem szükséges IP-cím
- A szerver adatokat küld a csoportcímre
- Egyirányú kommunikáció
- A csoportcím egy IP-cím a 224.0.0.0 és 239.255.255.255 közötti tartományban

![NTP multicast konfiguráció sematikus ábrázolása, amely a hálózati időjelek elosztását mutatja több vevőhöz.]

### 2.3 Broadcast szinkronizálás

A broadcaston keresztüli szinkronizálás a gyakorlatban nem fontos, mivel a legtöbb hálózat nem engedélyezi a broadcast kommunikációt. A broadcast kommunikáció meglehetősen nagy terhelést jelenthet a hálózatokra, mivel a csomagok minden klienshez elküldésre kerülnek. Ez sok esetben nem kívánatos.

**Broadcast definíciók:**

- Az üzenetek az ugyanazon hálózaton belüli összes klienshez elküldésre kerülnek
- Fenntartott IP-cím: 255.255.255.255
- A kliensek ismeretlenek a szerver számára
- Egyirányú kommunikáció

### 2.4 Unicast és Multicast összehasonlítása

| Multicast | Unicast |
|---|---|
| A master óra periodikusan küldi az időt és opcionálisan az időzóna-információkat egy konfigurált csoportcímre. Minden slave óra, amely ugyanarra a csoportcímre van konfigurálva, fogadja az információkat és megjeleníti a megfelelő időt. | Minden slave óra periodikusan lekérdezi az időt és opcionálisan az időzóna-információkat a master órától. |
| Nincs szükség adminisztrációra / IP-címek hozzárendelésére. | Minden slave órának saját IP-címe van, amelyet vagy DHCP szerveren keresztül, vagy a hálózati adminisztrátor által kell hozzárendelni. |
| Nincs további konfiguráció szükséges. A slave órák egyedi MAC-címükkel azonosíthatók (MAC címkén látható). | A slave órák egyedi IP-címükkel vagy egyedi MAC-címükkel azonosíthatók (MAC címkén látható). ICMP ping lehetséges az IP-címre. |
| Gyakorlatilag nincs hálózati terhelés (1 csomag eléri az egész csoportot). Egyirányú kommunikáció. | Jelentősen nagyobb hálózati terhelés a kliensek általi „polling" miatt. Kétirányú kommunikáció. |
| A hálózati késleltetések befolyásolják a slave órák pontosságát (WLAN / LAN váltás: 100 ms lehetséges). | Futásidejű kompenzáció lehetséges. Így a hálózati késleltetéseknek már nincs hatása a pontosságra, amíg a késleltetések szimmetrikusak. |
| A multicast címzett csomagok átküldhetők hálózati határokon (router) az IGMP segítségével. Lehetséges, hogy egy master óra szükséges alhálózatonként, ha a router nem támogatja a Multicastot. | A unicast címzett csomagok átküldhetők hálózati határokon (router). |

## 3. A Stratum meghatározása az NTP időszinkronizálásban

![Az NTP konfigurációban a stratum szintek magyarázata]

Az NTP az óraforrások szintjeinek hierarchikus rendszerét használja. E hierarchia minden szintjét **stratumnak** nevezik, és egy nullával kezdődő rétegszámot rendelnek hozzá a tetején. A stratum szint határozza meg a referenciakórától való távolságát. Fontos megjegyezni, hogy a stratum nem a minőség vagy a megbízhatóság jelzése. A stratum ezen meghatározása eltér a távközlési rendszerekben használt óra-stratum fogalmától is.

A sárga nyilak közvetlen kapcsolatot jeleznek; a piros nyilak hálózati kapcsolatot jeleznek.

A GNSS órákat általában **stratum 0** órának tekintik. Ez azt jelenti, hogy egy GNSS időjel-vevő által szinkronizált NTP-szerver egy **stratum 1** órát képvisel.

➡️ A GNSS-ről bővebben ebben a cikkben olvashatsz.

## 4. Alkalmazások hálózati időszinkronizáláshoz

### 4.1 Vasutak, IT-hálózatok, kórházak

Fő időszinkronizáló szerver redundáns kialakításban (opcionális), GPS-szinkronizált. A különböző állomásokon (helyszíneken) helyi időszinkronizáló szervereket használnak. A LAN órák általában a helyi időszinkronizáló szerverek által vannak szinkronizálva. Meghibásodás esetén közvetlenül a fő időszinkronizáló szerver is szinkronizálhatja őket.

![Network Time Protocol (NTP)]

### 4.2 Ipar és egyetemek

A fő időszinkronizáló szerver, GPS-szinkronizált, az egész hálózatot szinkronizálja. A különböző állomásokon (helyszíneken) helyi időszinkronizáló szervereket használnak. A helyi önbeállító órák általában MOBALine-on keresztül, 2 vezetékes vonalon vannak szinkronizálva.

![Network Time Protocol (NTP)]

## 5. Hogyan tud a MOBATIME támogatni?

A MOBATIME NTP és PTP termékeket kínál, amelyek rendkívül pontos interfészeket kínálnak, mint a PRC, PTP IEEE1588 V2, NTP stb., a pontosság és a nyomon követhetőség előnyével. A Grandmasterek és időszinkronizáló szerverek kristály- vagy atomoszcillátorokkal vannak felszerelve, hogy időt, impulzust, fázist és frekvenciamegoldásokat biztosítsanak, támogatva a szinkronizálást a Global Navigation Satellite System segítségével közepes vagy nagy infrastruktúra-hálózatok számára.

Az NTP időszinkronizáló szerverek nagy pontossággal és megbízhatósággal szinkronizálják hálózataidat. Az IT-hálózat összes rendszere a pontos idővel rendelkezik a Network Time Protocolon keresztüli szinkronizálásnak köszönhetően. Az események így kronológiailag rendezhetők a pontos időbélyegeknek köszönhetően.

![Mobatime DTS 4135.timeserver időszinkronizáló szerver NTP elölnézet DCF IRIG szinkronizálás]
![Mobatime dts4135-4 időszinkronizáló szerver NTP részletnézet DCF IRIG szinkronizálás]
![Mobatime dts4135-5 időszinkronizáló szerver NTP részletnézet DCF IRIG szinkronizálás]

- Nagy pontosságú idő minden hálózati környezetben
- Network Time Protocol időszinkronizáló szerver különféle alkalmazásokhoz
- Kombinált időszinkronizáló szerver master óra összetett órarendszerekben való használathoz

## 6. GYIK az NTP-ről

### Hány kliens szinkronizálható egy időszinkronizáló szerverrel?

Multicast módban a kliensek száma gyakorlatilag korlátlan, mivel az NTP időcsomagok egyszerre kerülnek elküldésre az összes klienshez. A modern NTP szerverek több száz kérést képesek feldolgozni másodpercenként a pontosság elvesztése nélkül, ami a legtöbb esetben elegendő több száz kliens kiszolgálásához. Ha az időintervallumok a klienseknél eltérően vannak beállítva, akár ezer vagy több kliens is könnyedén csatlakoztatható.
