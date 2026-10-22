# 2. Fejezet: Hálózatok és Anomáliák (Networks and Anomalies)

A hálózati anomáliák (eltérések, támadások) felderítéséhez elengedhetetlen a hálózatok alapvető működésének, felépítésének és kommunikációs elveinek ismerete.

## 2.1 Hálózati alapok (Networking Basics)

A számítógép-hálózat egy komplex, kölcsönhatásban lévő rendszer. Két vagy több számítógép akkor alkot hálózatot, ha képesek egy megosztott közegen keresztül adatokat küldeni és fogadni. A hálózat teljes viselkedését az azt alkotó egyedi entitások együttes működése (ensemble behavior) határozza meg.

Egy hálózatban három fő kommunikáló entitást különböztetünk meg:

1. **Felhasználók (Users):** Emberek, akik különböző hálózati tevékenységeket végeznek (pl. böngészés, online vásárlás).
2. **Gazdagépek (Hosts):** Számítógépek vagy eszközök, amelyeket egyedi cím (pl. IP-cím) azonosít a hálózaton.
3. **Folyamatok (Processes):** Futtatható programok példányai, amelyek a kliens-szerver architektúrában kommunikálnak.
   - **Kliens folyamat:** Szolgáltatást kér (pl. egy webböngésző).
   - **Szerver folyamat:** Kiszolgálja a kérést (pl. egy webszerver, ami elküldi a weboldalt).

> 💡 **Gyakorlati ML / Biztonsági példa:** Egy fejlett gépi tanulási alapú behatolásjelző rendszer (IDS) mindhárom entitás szintjén gyűjthet jellemzőket (features). Vizsgálhatja a Felhasználó viselkedését (milyen időszakban lép be), a Host hálózati forgalmát (IP címek, forgalom volumene), és a Folyamatok rendszerhívásait (pl. egy webszerver folyamat hirtelen rendszergazdai parancsokat próbál futtatni – ez egy anomália).

### 2.1.1 A hálózat általános felépítése (Typical View of a Network)

A hálózatokban a gépeket és szervereket speciális hálózati eszközök (switch-ek, router-ek, hub-ok) kötik össze fizikai vagy vezeték nélküli kommunikációs közegeken (media) keresztül. A hálózat teljesítménye nagyban függ az alkalmazott közegtől és az összekötő eszközöktől.

> 📍 **[IDE JÖN A KÉP: Figure 2.1: A typical computer networking]**
>
> Ide illeszd be a vezetékes számítógép-hálózatot bemutató ábrát a switch-ekkel és routerekkel.

### 2.1.2 Kommunikációs közegek (Communication Media)

A kommunikációs közegek feladata a nyers bitfolyamok (raw bit streams) továbbítása egyik géptől a másikig. A különböző közegek teljesítménye négy fő paraméter alapján hasonlítható össze:

- Sávszélesség (Bandwidth)
- Késleltetés (Delay)
- Költség (Cost)
- Telepítés és karbantartás (Installation and maintenance)

A közegeket két nagy kategóriába soroljuk: **Vezetékes (Guided)** és **Vezeték nélküli (Unguided)**.

> 📍 **[IDE JÖN A KÉP: Figure 2.2: Classification of communication media]**
>
> Ide illeszd be a kommunikációs közegek osztályozását mutató fadiagramot.

#### Összehasonlító táblázat: Vezetékes vs. Vezeték nélküli közegek

| Jellemző | Vezetékes (Guided Media) | Vezeték nélküli (Unguided Media) |
| :--- | :--- | :--- |
| **Terjedés módja** | A jel energiája egy szilárd közegen belül marad (vezetve van). | A jel energiája elektromágneses hullámként terjed szabadon (nincs vezetve). |
| **Kommunikáció iránya** | Pont-pont (Point to point) kommunikáció. | Rádiós szórás (Broadcasting) minden irányba. |
| **Topológia** | Diszkrét hálózati topológiák értelmezhetőek. | Folytonos hálózati topológiák. |
| **Jelcsillapítás (Attenuation)** | A távolságtól exponenciálisan függ. | A távolság négyzetével arányos. |
| **Kapacitásbővítés** | Könnyű: egyszerűen több kábelt kell lefektetni. | Nincs rá fizikai lehetőség a spektrum korlátai miatt. |
| **Költség & Telepítés** | Drágább és időigényesebb kiépíteni. | Relatíve olcsóbb és gyorsabban telepíthető. |

#### 2.1.2.1 Vezetékes közegek (Guided Media)

- **Csavart érpár (Twisted Pair):** Két szigetelt rézvezeték helikális (DNS-szerű) formában összecsavarva. A csavarás kioltja a kábelek közötti interferenciát (sugárzást). Sávszélessége a kábel vastagságától és a távolságtól függ. Olcsó és megfelelő teljesítményű.
- **Koaxiális kábel (Coaxial Cable):** Egy merev rézmag, amit szigetelőanyag, majd egy fémháló (árnyékolás) és műanyag köpeny borít. Remek sávszélességet ($\sim 1$ GHz) és nagyon jó zajvédelmet (noise immunity) biztosít. Kábel TV-k és városi hálózatok (MAN) használják.
- **Száloptika (Fiber Optic Cables):** Üvegmagból (core) áll, amelyben fény terjed. Ezt egy alacsonyabb törésmutatójú üvegburkolat (cladding) veszi körbe, hogy a fény a magban maradjon, amit műanyag köpeny véd.
  - **Típusai:** Multi-mode (többmódusú, $\sim 50$ mikron mag) és Single-mode (egymódusú, $8-10$ mikron mag).
  - **Sávszélesség:** Akár $50$ Tbps felett is lehet, de a technológiai korlát ma tipikusan $10-100$ Gbps.

> 📍 **[IDE JÖN A KÉP: Figure 2.3: Fiber optic cable: end view of a sheath with 3 fibers]**
>
> Ide illeszd be a száloptikás kábel keresztmetszetét bemutató ábrát.

> 📍 **[IDE JÖN A TÁBLÁZAT: Table 2.2: Comparison of Guided Media]**
>
> Ide illeszd be a csavart érpár, koax és száloptika sávszélességét, adatátviteli sebességét és repeater távolságait összehasonlító táblázatot.

#### 2.1.2.2 Vezeték nélküli közegek (Unguided Media)

- **Rádióhullám (Radio Transmission, $30$ MHz – $1$ GHz):** Minden irányba terjed (omnidirectional), áthatol az épületeken. Az alacsony frekvencia könnyen átjut az akadályokon, de hamar veszít az erejéből. A magasabb frekvencia egyenesen terjed és visszapattan az akadályokról (akár az ionoszféráról is).
- **Mikrohullám (Microwave Transmission, $1$ GHz – $40$ GHz):** Keskeny sávban, szigorúan egyenes vonalban (irányítottan) terjed. Nem hatol át az épületeken. Földfelszíni pont-pont kapcsolatokra és műholdas kommunikációra használják. A Föld görbülete miatt időközönként átjátszóállomások (repeaterek) kellenek.
- **Fényhullám (Lightwave / Laser Transmission):** Egyirányú, épületek (tetők) közötti lézeres kapcsolat. Precíz rálátást (Line-of-Sight) és mindkét oldalon külön adó-vevőt (fotodetektort) igényel. Költséghatékony és nagyon nagy sávszélességű.
- **Infravörös és Milliméteres hullámok (Infrared):** Rövid hatótávolságú, egyirányú (pl. TV távirányító). Szilárd tárgyakon nem hatol át, cserébe nagyon olcsó és egyszerű eszköz.

### 2.1.3 Hálózati szoftverek (Network Software)

A modern hálózatokban a szoftveres vezérlés legalább olyan fontos, mint a hardver. A szoftverek komplexitásának kezelésére rétegzett architektúrát (Layered Architecture) használnak (pl. OSI vagy TCP/IP modell).

**A rétegzés alapelvei:**
Minden réteg (layer-$n$) egy előre definiált szolgáltatást nyújt a felette lévő rétegnek (layer-$(n+1)$), elrejtve előle a megvalósítás technikai részleteit.

**Protokoll (Protocol):** Szabályok és konvenciók összessége, amely meghatározza, hogy két különböző gépen lévő, de azonos szintű réteg (pl. mindkét gépen a Hálózati réteg) hogyan kommunikál egymással és milyen formátumú üzeneteket cserél.

#### 2.1.3.2 Kapcsolatorientált és Kapcsolatmentes szolgáltatások

A hálózati rétegek kétféle alapvető szolgáltatást nyújthatnak:

- **Kapcsolatorientált (Connection-Oriented):**
  - **Folyamat:** 1. Kapcsolat felépítése $\rightarrow$ 2. Adatátvitel (előre letárgyalt paraméterek alapján) $\rightarrow$ 3. Kapcsolat bontása.
  - Megbízható (reliable), a csomagok sorrendje garantált.
- **Kapcsolatmentes (Connectionless / Datagram):**
  - Nincs előzetes kapcsolatfelépítés. A csomagok (datagramok) önállóan utaznak.
  - Megbízhatatlan (unreliable), nem garantált a megérkezés. Gyakori kliens-szerver kérés-válasz (request-reply) rendszereknél.

> 💡 **Gyakorlati ML / Biztonsági példa:** A gépi tanulási alapú anomália detektoroknak teljesen más feature-öket (jellemzőket) kell figyelniük a két típusnál.
> - **Kapcsolatorientált (TCP)** hálózatnál a modell olyan anomáliákat keres, ahol a kapcsolatfelépítés szabálytalan (pl. SYN Flood DDoS támadás esetén rengeteg kapcsolatot kezdeményeznek, de sosem fejezik be, ezt a TCP flag-ek arányának torzulása mutatja).
> - **Kapcsolatmentes (UDP)** hálózatnál (pl. videó stream vagy DNS) nincsenek flag-ek, így a modellnek a csomagok beérkezési idejét (inter-arrival time) vagy a csomagméretek eloszlását kell figyelnie egy "UDP flood" anomália felismeréséhez.

#### A szolgáltatások típusai

| Szolgáltatás típusa | Mód | Példa a valós életből |
| :--- | :--- | :--- |
| Megbízható üzenetfolyam (Reliable message stream) | Kapcsolatorientált | Weboldalak sorozatos letöltése |
| Megbízható bájtfolyam (Reliable byte stream) | Kapcsolatorientált | Távoli bejelentkezés (SSH / Telnet) |
| Megbízhatatlan kapcsolat (Unreliable connection) | Kapcsolatorientált | Digitalizált hanghívás (VoIP) |
| Megbízhatatlan datagram | Kapcsolatmentes | Elektronikus levélszemét (Spam email) |
| Nyugtázott datagram | Kapcsolatmentes | Tértivevényes levél |
| Kérés-Válasz (Request-reply) | Kapcsolatmentes | Adatbázis lekérdezés (DNS, SQL) |

#### 2.1.3.3 Szolgáltatás primitívek (Service Primitives)

A szoftveres folyamatok (programok) úgynevezett "primitíveken" (alapműveleteken) keresztül érik el a hálózati szolgáltatásokat. Egy megbízható kliens-szerver bájtfolyamhoz (pl. TCP) használt primitívek:

- **LISTEN:** A szerver folyamat blokkolódik (várakozik), amíg bejövő kapcsolatkérés nem érkezik.
- **CONNECT:** A kliens kapcsolatot kezdeményez, és felfüggesztődik, amíg választ nem kap.
- **RECEIVE:** A folyamat blokkolódik, várva a beérkező üzenetre.
- **SEND:** Üzenet küldése (ha célba ért, a szerver folyamat folytatódhat).
- **DISCONNECT:** A kapcsolat bontása (a másik fél nyugtázza).

#### 2.1.3.4 Különbség a Szolgáltatás és a Protokoll között

- **Szolgáltatás (Service):** Két egymás alatti/feletti réteg közötti interfész (Interface). Meghatározza, hogy az alsó réteg mit nyújt a felsőnek. (Képzeletbeli vertikális kommunikáció).
- **Protokoll (Protocol):** Azonos szintű (peer) entitások közötti szabályrendszer (pl. az én IP rétegem és a te IP réteged között). Meghatározza, hogy hogyan valósul meg a szolgáltatás. Egy protokoll bármikor lecserélhető egy másikra, amíg a nyújtott szolgáltatás (az interfész) változatlan marad.

> 📍 **[IDE JÖN A KÉP: Figure 2.4: A service related to the interface between two layers]**
>
> Ide illeszd be a két szomszédos réteg közötti interfészt és szolgáltatást bemutató ábrát.

### 2.1.4 Referenciamodellek (Reference Models)

A hálózati architektúrák megértéséhez a számítógépes kommunikációt szabványosított modellekkel írjuk le. Két legismertebb típus:

1. **OSI referenciamodell** (Open Systems Interconnection)
2. **TCP/IP referenciamodell**

> 📍 **[IDE JÖN A KÉP: Figure 2.4: The relationship between service and protocol]**
>
> Ez az ábra a korábbi blokkhoz tartozik, a protokollok és szolgáltatások vertikális/horizontális kapcsolatát mutatja.

#### 2.1.4.1 Az ISO OSI Referenciamodell

Az OSI modellt a Nemzetközi Szabványügyi Szervezet (ISO) hozta létre. Ez egy **7 rétegű keretrendszer**, amelynek célja a protokollszabványok fejlesztésének egységesítése. (A vizuális ábrázolásban a szomszédos rétegek között fizikai kommunikáció - solid line, míg a két gép azonos rétegei között logikai/virtuális kommunikáció - dotted line zajlik).

**A 7 réteg (felülről lefelé):**

1. **Alkalmazási réteg (Application layer):** Hozzáférést biztosít a felhasználóknak az OSI környezethez és az elosztott információs szolgáltatásokhoz (pl. böngésző, email kliens).
2. **Megjelenítési réteg (Presentation layer):** Biztosítja, hogy az alkalmazások függetlenek legyenek az adatok szintaktikai ábrázolásától (pl. karakterkódolás, titkosítás, tömörítés).
3. **Viszony/Munkamenet réteg (Session layer):** A kommunikáció vezérlőstruktúráját adja. Felelős az együttműködő alkalmazások közötti kapcsolatok (munkamenetek) felépítéséért, fenntartásáért és bontásáért.
4. **Szállítási réteg (Transport layer):** Megbízható és transzparens adatátvitelt biztosít a végpontok között. Felelős a végponttól-végpontig (end-to-end) történő hibajavításért és a forgalomszabályozásért (flow control).
5. **Hálózati réteg (Network layer):** Függetleníti a felsőbb rétegeket az adatátviteli és kapcsolási technológiáktól. Fő feladata az útválasztás (routing), valamint a kapcsolatok felépítése és bontása.
6. **Adatkapcsolati réteg (Data link layer):** Feladata a fizikai kapcsolaton keresztüli megbízható információátvitel. Blokkokat (kereteket - frames) továbbít, biztosítja a szinkronizációt, hiba- és forgalomszabályozást a két közvetlen szomszédos csomópont között.
7. **Fizikai réteg (Physical layer):** Strukturálatlan bitfolyam (stream of bits) továbbítása a fizikai közegen. Kezeli az ehhez szükséges mechanikai, elektromos és procedurális funkciókat (pl. kábelek, feszültségszintek).

> 📍 **[IDE JÖN A KÉP: Figure 2.5: The OSI reference model]**
>
> A hét réteget bemutató klasszikus OSI modell ábra.

#### 2.1.4.2 A TCP/IP Referenciamodell

Bár az OSI modellt széles körben oktatják, a gyakorlatban a sokkal költséghatékonyabb és kompaktabb TCP/IP modell terjedt el az internet alapjaként. A könyv egy **5 rétegű** TCP/IP architektúrát tárgyal (néhol a fizikai és adatkapcsolati réteget összevonják "Host-to-Network" néven, de itt külön szerepelnek).

**Eszközök szintjei a hálózatban:**
- **A híd (Bridge) és Switch:** Az 1. és 2. réteget érti.
- **Az útválasztó (Router):** Az 1., 2. és 3. réteget érti és dolgozza fel.

**A TCP/IP rétegei (felülről lefelé):**

1. **Alkalmazási réteg (Application layer):** Támogatja a hálózati alkalmazásokat. Ide tartoznak az olyan protokollok, mint az HTTP (web), SMTP (email) és FTP (fájlátvitel).
2. **Szállítási réteg (Transport layer):** Az alkalmazási üzenetek mozgatása a kliens és a szerver között. Két kulcsprotokollja:
   - **TCP (Transmission Control Protocol):** Megbízható, kapcsolatorientált. Hosszú üzeneteket szegmensekre vág (segmentation), és torlódásvezérlést (congestion control) biztosít (pl. lassítja a küldést, ha a hálózat terhelt).
   - **UDP (User Datagram Protocol):** Kapcsolatmentes, megbízhatatlan (nincs garancia a megérkezésre). Egyszerűen csak "kiküldi" a csomagokat.
3. **Hálózati / Internet réteg (Network layer):** Datagramok (csomagok) útválasztása a hosztok között routereken/gateway-eken keresztül. Itt működik az IP (Internet Protocol) és az útválasztó protokollok (routing protocols).
4. **Adatkapcsolati réteg (Link layer):** A hálózati réteg erre támaszkodik, amikor egy csomagot egyik csomópontról (node) a közvetlenül következőre továbbít (hop-by-hop). Különböző protokollokat használhat a különböző szakaszokon (pl. egy csomag indulhat Etherneten, majd folytathatja PPP-n).
5. **Fizikai réteg (Physical layer):** A keretek (frames) egyedi bitjeinek mozgatása a fizikai közegen. Erősen függ az átviteli közegtől (pl. réz, koax, optika).

> 📍 **[IDE JÖN A KÉP: Figure 2.6: TCP/IP architecture]**
>
> A TCP/IP rétegeket és a hozzájuk tartozó hardvereket (bridge, router) bemutató ábra.

### 2.1.5 Protokollok (Protocols)

A protokoll az a szabályrendszer, amely meghatározza az egyenrangú (peer) entitások közötti üzenetek (csomagok) formátumát, jelentését és az átvitel során végrehajtandó műveleteket.

> 💡 **ML/IDS Szempont:** A gépi tanulási alapú anomália detektorok számára a protokollok fejlécei (headers) jelentik a legfontosabb bemeneti adatot (feature-öket). A "Packet Inspection" (csomagvizsgálat) során az ML algoritmusok ezeket a fejléc mezőket elemzik.

#### 2.1.5.1 TCP (Transmission Control Protocol)

**Jellemzők:** Szállítási rétegbeli, megbízható, kapcsolatorientált protokoll. Két rendszer között ideiglenes logikai kapcsolatot épít fel, amit portok (port values) azonosítanak.

**Fejléc mezők fontossága (Feature kinyeréshez):**
- **Source/Destination Port:** Meghatározza a futó alkalmazást (pl. 80=HTTP). Anomália lehet: forgalom egy szokatlan porton (pl. 6667 - IRC botnet).
- **Sequence & Acknowledgment number:** Sorrendiség és nyugtázás (flow/error control). Anomália: Érvénytelen szekvenciaszámok, amivel a TCP kapcsolatot akarják megszakítani (TCP Reset attack).
- **Window:** A fogadó fél szabad puffermérete.
- **Checksum:** 16-bites hibaellenőrző.

> 📍 **[IDE JÖN A KÉP: Figure 2.7: TCP header]**

#### 2.1.5.2 UDP (User Datagram Protocol)

**Jellemzők:** Szállítási rétegbeli, kapcsolatmentes, nem garantálja a megérkezést, a sorrendet vagy a duplikáció elkerülését.

**Alkalmazás:** Tranzakció-orientált (rövid) alkalmazásoknál, vagy ahol a sebesség fontosabb a hibátlanságnál (DNS lekérdezések, videó stream). Fejléce nagyon egyszerű (Portok, Hossz, Checksum).

> 📍 **[IDE JÖN A KÉP: Figure 2.8: UDP header]**

#### 2.1.5.3 IP (Internet Protocol - IPv4)

**Jellemzők:** Hálózati rétegbeli protokoll, feladata a hálózatokon átívelő útválasztás (routing). A szállítási rétegből kapott szegmenshez adja hozzá a saját fejlécét, így jön létre a datagram.

**Fejléc mezők:**
- **Source & Destination IP (32-bit):** A küldő és fogadó címe. Anomália: IP Spoofing (hamisított feladó cím).
- **Protocol:** Mutatja, mi van az IP csomagban (TCP, UDP, ICMP).
- **ID, Flags, Fragment Offset:** A csomagok darabolásához (fragmentation) és újraösszeszereléséhez. Anomália: Teardrop attack vagy szándékosan rosszul darabolt csomagok az IDS megkerülésére.

> 📍 **[IDE JÖN A KÉP: Figure 2.9: IPv4 header]**

#### 2.1.5.4 Alkalmazási szintű protokollok: SMTP, FTP, Telnet, SNMP

- **SMTP (Simple Mail Transfer Protocol):** Email üzenetek továbbítása a szerverek között (TCP felett). Funkciói: levelezőlisták, kézbesítési nyugták. Az üzenet létrehozását nem, csak a szállítást végzi.
- **FTP (File Transfer Protocol):** Szöveges és bináris fájlok átvitele. Két TCP kapcsolatot használ: egyet a vezérlésnek (parancsok, jelszavak), egyet pedig magának az adatnak (ezáltal overhead mentes).
- **Telnet (Terminal Emulation):** Távoli bejelentkezést tesz lehetővé (TCP felett). A hálózaton keresztül úgy viselkedik, mintha a felhasználó a gép előtt ülne. Két modulja van: User Telnet és Server Telnet. Szerintük ma már nem biztonságos (nincs titkosítás), de hálózati anomáliaként gyakran bukkannak fel Telnet brute-force próbálkozások IoT eszközök (pl. Mirai botnet) ellen.
- **SNMP (Simple Network Management Protocol):** Hálózati eszközök (switch-ek, routerek) menedzselése és monitorozása egy központi pontról (az eszköz interfészeinek lekérdezésével).

#### 2.1.5.6 ICMP (Internet Control Message Protocol)

**Jellemzők:** A hálózat diagnosztikai és hibajelző protokollja. A routerek/hosztok küldenek ICMP üzenetet, ha probléma van (pl. a csomag nem éri el a célt, vagy a router puffere megtelt).

**Felépítés:** A fejléc legalább 8 bájtos:
- **Type (8-bit):** Az üzenet kategóriája (pl. Echo Request / Ping).
- **Code (8-bit):** Az üzenet pontosabb paramétere.
- **Checksum (16-bit):** Hibaellenőrzés az egész ICMP üzenetre.
- **Data (32-bit):** Hosszabb paramétereknek.

> 💡 **ML/IDS Szempont:** Anomália lehet a túl sok ICMP Echo Request (Ping Flood DoS), vagy adatcsempészés (Data Exfiltration), amikor a támadó a "Data" mezőben rejtve viszi ki a lopott adatokat a hálózatból.

> 📍 **[IDE JÖN A KÉP: Figure 2.10: ICMP header]**


### 2.1.6 Hálózattípusok (Types of Networks)

A hálózatokat méretük, technológiájuk és a tervezési céljaik alapján különböző típusokba sorolhatjuk.

#### 2.1.6.1 Helyi hálózatok (Local Area Networks – LAN)

- **Jellemzők:** Magántulajdonban lévő, korlátozott földrajzi kiterjedésű hálózat (pl. egy épület, egyetem vagy iroda, maximum kb. $1$ km-es sugarú körben).
- **Kialakítás alapjai:** A lefedett terület mérete, a használt kommunikációs technológia (vezetékes vagy vezeték nélküli) és a hálózati topológia (csillag, gyűrű, sín) határozza meg.
- **Teljesítmény:** Hagyományosan $10-100$ Mbps sebesség mikroszekundumos késleltetéssel, de a modern LAN-ok (pl. gigabites Ethernet) akár a $10$ Gbps sávszélességet is elérik.

> 💡 **ML/IDS Szempont:** A LAN-ok belső hálózatok. Az itt betanított gépi tanulási modellek gyakran az ún. lateral movement (oldalirányú mozgás) felismerésére fókuszálnak. Ilyen anomália például az ARP Spoofing, a belső port szkennelés, vagy egy zsarolóvírus (Ransomware) belső terjedése a gépek között, ami hirtelen megnövekedett SMB (fájlmegosztó) forgalmat generál.

#### 2.1.6.2 Nagy kiterjedésű hálózatok (Wide Area Networks – WAN)

- **Jellemzők:** Országokat, kontinenseket átívelő hálózatok, amelyek magán vagy nyilvános kommunikációs alhálózatokat (subnets) használnak.
- **Működési elv:** A csomagokat "Tárold és továbbítsd" (Store and forward) vagy csomagkapcsolt (packet switched) módszerrel továbbítják.
- **Komponensek:** Két fő elemből áll: az átviteli közegből (kábelek, mikrohullám) és a kapcsolási eszközökből (intelligens útválasztók, azaz routerek).

> 📍 **[IDE JÖN A KÉP: Figure 2.11: Relation between hosts on LANs and the subnet]**
>
> Ez az ábra bemutatja, hogyan kapcsolódnak a LAN-on lévő végpontok a WAN alhálózat routereihez.

#### 2.1.6.3 Városi hálózatok (Metropolitan Area Network – MAN)

- **Jellemzők:** Több LAN összekapcsolásával jön létre, kiterjedése egy várost vagy kerületet fed le.
- **Példa:** A gyors vezeték nélküli internethozzáférést biztosító MAN hálózatok gyakran az IEEE 802.16 (WiMAX) szabványra épülnek.

#### 2.1.6.4 Vezeték nélküli hálózatok (Wireless Networks)

Irányítatlan közeget (unguided media, pl. rádióhullámokat) használnak. Három alapkategóriájuk:

1. **Rendszerösszekötő (System interconnection / PAN):** Személyes hálózat (Personal Area Network) rövid hatótávú rádiójelekkel (pl. Bluetooth egér, billentyűzet).
2. **Vezeték nélküli LAN (Wireless LAN / WLAN):** Épületen belüli kommunikáció rádiós modemek és antennák segítségével (pl. Wi-Fi). Kiváltja a kábelezést.
3. **Vezeték nélküli WAN (Wireless WAN):** Óriási kiterjedésű rádiós hálózat, tipikus példája a mobilszolgáltatók cellás hálózata (Cellular network).

> 💡 **ML/IDS Szempont:** A vezeték nélküli hálózatokban új típusú anomáliák jelennek meg, mint például a Deauthentication attack (amikor a támadó folyamatosan ledobálja a klienseket a hálózatról) vagy a Rogue Access Point (hamisított Wi-Fi állomás). Az ML modellek itt a 802.11-es menedzsment keretek (management frames) gyakoriságát és mintázatait figyelik.

#### 2.1.6.5 Internetworks és az Internet (2.1.6.6)

- **Internetwork:** Különböző, gyakran inkompatibilis hálózatok összekötése speciális átjárók (gateways) segítségével, amelyek hardveres és szoftveres protokollfordítást végeznek.
- **Az Internet:** A világ legnagyobb internetworkje. Kliensek (végpontok) milliárdjait köti össze a TCP/IP protokollcsalád segítségével.
- **Működése:** A csomagkapcsolás (packet switching) elvén működik, vagyis ugyanazt a fizikai utat több kommunikáló fél használja egyidőben. A hálózati hozzáférést az internetszolgáltatók (ISP - Internet Service Providers) biztosítják.

> 📍 **[IDE JÖN A KÉP: Figure 2.12: A generic end system interconnection of the Internet]**

### 2.1.7 A hálózatok léptékei (Scales of Networks)

A hálózatokat a processzorok közötti fizikai távolság alapján is osztályozhatjuk. Ez technológiai szempontból is kritikus:

| Processzorok közötti távolság | Elhelyezkedés | Hálózat típusa (Példa) |
| :--- | :--- | :--- |
| $0.1$ m | Ugyanaz az áramkör | Adatfolyam-gépek (Data flow machines) |
| $1$ m | Ugyanaz a rendszer | Személyi hálózat (PAN / Personal area network) |
| $10$ m – $1$ km | Egy szoba, épület, egyetem | Helyi hálózat (LAN) |
| $10$ km | Egy város | Városi hálózat (MAN) |
| $100$ km – $1000$ km | Egy ország vagy kontinens | Nagy kiterjedésű hálózat (WAN) |
| $10\,000$ km | Egy bolygó (a Föld) | Az Internet (Internetwork) |

### 2.1.8 Hálózati topológiák (Network Topologies)

A topológia a hálózati végpontok összekapcsolásának elrendezését jelenti. Két típusa van:

- **Fizikai topológia:** A kábelezés, az átviteli közeg geometriai alakja.
- **Logikai topológia:** Ahogyan az adat ténylegesen áramlik (ez routerekkel dinamikusan újra is konfigurálható, és eltérhet a fizikai felépítéstől).

#### 2.1.8.1 Sín (Bus)

Minden állomás egy közös, lineáris átviteli közegre (a sínre) csatlakozik egy interfészen (tap) keresztül.
Az adat mindkét irányba halad, amíg meg nem találja a célállomást. Mivel a közeg megosztott, központosított vagy elosztott ütközésfeloldó mechanizmusra van szükség (pl. az Ethernet CSMA/CD).

- **Típusai:** Lineáris (két végpont) vagy Elosztott (több leágazás).

> 📍 **[IDE JÖN A KÉP: Figure 2.13(a): Bus topology]**

#### 2.1.8.2 Gyűrű (Ring)

Az eszközök egy körkörös pályán, pont-pont (point-to-point) kapcsolatokkal vannak összekötve, és egyben jelismétlőként (repeater) is funkcionálnak.
Az átvitel egyirányú (unidirectional). A keretek (frames) addig keringenek, amíg a célállomás le nem másolja őket, majd visszatérnek a feladóhoz megsemmisítésre.

- **Példák:** IEEE 802.5 Token Ring, FDDI.

> 📍 **[IDE JÖN A KÉP: Figure 2.13(b): Ring topology]**

#### 2.1.8.3 Fa (Tree)

Az állomások egy központi kezdőpontból (headend) induló, elágazó rendszerhez csatlakoznak, hurkok (loop) nélkül.
A kereteket (frames) minden állomás felé elküldik, a keret fejléce tartalmazza a specifikus célcímet.

> 📍 **[IDE JÖN A KÉP: Figure 2.14(a): Tree topology]**

#### 2.1.8.4 Csillag (Star)

Minden állomás egy közös, központi csomóponthoz csatlakozik pont-pont kapcsolatokkal. Kétféle működési módja lehet a központi csomópont típusától függően:

- **Broadcast (Hub):** A beérkező keretet egyszerűen lemásolja és minden portjára kiküldi.
- **Keretkapcsolt (Switch):** A központi eszköz puffereli a keretet, majd csak azon a specifikus kimeneti porton küldi tovább, ahol a célállomás található.

> 💡 **ML/IDS Szempont:** A Csillag (Star) topológia Hub esetén biztonsági rémálom, mert bárki lehallgathatja a forgalmat (packet sniffing). Egy Switch használata ezt megakadályozza, mivel szeparálja a portokat. Támadóként ezt MAC Flooding támadással szokták feltörni (túlterhelik a Switch memóriáját, ami emiatt Hub-ként kezd el működni). Egy ML alapú IDS számára a másodpercenkénti $10\,000+$ különböző MAC címről érkező csomag egyértelmű, könnyen tanítható anomália (MAC Spoofing/Flooding).

### 2.1.9 Hardver komponensek (Hardware Components)

A hálózati kommunikáció megvalósításához szoftverek és hardverek együttműködésére van szükség. A hardverek biztosítják a fizikai közeghez való csatlakozást és a forgalom szabályozását.

#### 2.1.9.1 Hálózati kommunikációs eszközök

Öt alapvető hardvereszközt különböztetünk meg, amelyek különböző rétegekben működnek:

**1. Jelismétlők és Hub-ok (Repeaters and Hubs)**

- **Repeater (Jelismétlő - OSI 1. réteg):** Feladata a beérkező jelek fogadása, felerősítése (amplify) és kétirányú továbbítása. Kábelhosszak megnövelésére szolgál. Szoftveres szempontból transzparens.
- **Hub (OSI 1. réteg):** Több bejövő vonalat egyesít elektromosan. Egy beérkező keretet (frame) egyszerűen minden portjára kiküld.
- **Kritikus tulajdonság:** Egyetlen ütközési tartományt (collision domain) hoz létre. Ha két keret egyszerre érkezik, ütközés történik.

**Típusai:**
- **Aktív (Active):** Felerősíti a jelet (akár $2000$ láb távolságig), áramot igényel.
- **Passzív (Passive):** Csak elosztja a jelet (max $300$ láb), csillag topológiához.
- **Intelligens (Intelligent):** Hálózatmenedzsment funkciókkal és útválasztási képességekkel rendelkezik, rugalmas átviteli sebességeket is támogat.

> 💡 **ML/IDS Szempont:** A Hub-ok használata biztonsági kockázat, mivel minden gép látja a hálózat teljes forgalmát. Egy "Promiscuous" (kíváncsi) módba állított hálózati kártyával a támadó könnyedén elkaphatja a jelszavakat (Packet Sniffing).

**2. Hidak (Bridges)**

- **Jellemzők:** Az OSI 2. rétegében (Adatkapcsolati réteg) működnek. LAN-okat kötnek össze a MAC-címek alapján (az IP címeket nem nézik). Minimalizálják a feldolgozási overheadet (rezsit).
- **Típusai:**
  - **Transzparens Híd (Transparent bridge):** A hosztok számára láthatatlan. Figyeli a bejövő forgalom forrás MAC-címeit, ebből tanulja meg a hálózati topológiát (belső MAC táblát épít), és ez alapján továbbít.
  - **RBridge (Routing bridge):** A TRILL (TRansparent Interconnection of Lots of Links) protokollt használja. Képes optimális útválasztásra (multipathing), elkerüli a hálózati hurkokat, és az IS-IS link-state protokollt használja a topológia felderítésére.

**3. Kapcsolók (Switches)**

- **Jellemzők:** Szintén az OSI 2. rétegében működnek. Dedikált kapcsolatot építenek a portok között a cél MAC-cím alapján.
- **Működése:** A beérkező csomag feladó MAC-címét rögzíti a belső MAC táblájában. Ha a cél MAC-cím benne van a táblában, csak arra a portra továbbít. Ha nincs, minden portra kiküldi (Flooding).
- **Típusai feldolgozás alapján:**
  - **Store-and-forward:** A teljes keretet fogadja, puffereli, ellenőrzi, majd továbbítja.
  - **Cut-through:** Amint beolvasta a cél MAC-címet a keret elején, azonnal megkezdi a továbbítást a megfelelő portra (gyorsabb, de hibás kereteket is átenged).
- **Típusai rétegek alapján:**
  - **Layer-2 switch:** Klasszikus MAC alapú kapcsoló, olyan, mint egy multiport bridge.
  - **Layer-3 switch:** Átvesz útválasztási (IP szintű) feladatokat a routerektől a sebesség növelése érdekében (pl. azonos forrás-cél folyamok együttes kezelése).

> 📍 **[IDE JÖN A KÉP: Figure 2.15: Working of a switch]**

**4. Útválasztók (Routers)**

- **Jellemzők:** Az OSI 3. rétegében működnek. Hálózatokat kötnek össze IP-címek és útválasztó táblák (Routing table) alapján.
- **Útválasztó táblák:**
  - **Statikus:** Kézzel beállított, nem reagál dinamikusan a hálózati hibákra.
  - **Dinamikus:** Automatikusan alkalmazkodik a hibákhoz és torlódásokhoz.
- **Képességek:** Gyakran tartalmaznak tűzfal funkciókat (szűrés, blokkolás) és NAT (Network Address Translation) szolgáltatást.

> 📍 **[IDE JÖN A KÉP: Figure 2.16: Router in the TCP/IP model]**

**5. Átjárók (Gateway)**

- **Jellemzők:** Szerverként (proxyként) működnek más, közvetlenül el nem érhető szerverek nevében. Általában az Alkalmazási rétegben (OSI 7.) protokollfordítást végeznek, vagy biztonsági portálként (tűzfalként) szolgálnak. A kliens gyakran nem is tudja, hogy egy átjáróval kommunikál, nem pedig az eredeti szerverrel.

#### 2.1.9.2 Hálózati interfész kártyák (Network Interface Card – NIC)

Ezek az eszközök csatlakoztatják a számítógépeket (állomásokat) a hálózathoz (LAN).

- **Ethernet kártya (IEEE 802.3):** A legelterjedtebb szabvány (pl. 100baseT, csavart érpáron, RJ-45 csatlakozóval). Minden kártyának gyárilag egyedi MAC címe van, amely azonosítja a hálózaton.
- **LocalTalk:** Az Apple régi, mára elavult rendszere (Daisy chain topológia, csavart érpár, $230.4$ Kbps sebesség).
- **Csatlakozók (Connectors):** Fizikai, elektromechanikus kapcsolatot biztosítanak (pl. RJ-45 / 8P8C, BNC koaxhoz, USB).
- **Vezérjeles gyűrű kártyák (Token Ring Cards):** A gyűrű hálózatokhoz. A "vezérjel" (token) körbejár a hálózatban. Az az állomás küldhet adatot, akinél a token van. Megbízható magas sávszélességű multimédia átvitelre ($4$ vagy $16$ Mbps).

#### 2.1.9.3 Adó-vevők (Transceivers) és Média konverterek (2.1.9.4)

- **Transceiver:** Rácsatlakozik a kábelre, feladata a hordozójel (carrier) és az ütközések (collision) érzékelése. Ha ütközést észlel, érvénytelen jelet küld a kábelre. Ő rakja össze az adatokat megfelelő keretformátumba és számolja ki a checksum-ot.
- **Média konverter (Media Converter):** Két különböző fizikai közeg összekapcsolására szolgál az OSI 1. rétegében (pl. réz alapú csavart érpárból csinál száloptikás kapcsolatot), gyakran vállalati környezetben (Enterprise networking).

### 2.1.10 Hálózati teljesítmény (Network Performance)

Több ezer gépes hálózatokban a komplex interakciók gyakran okoznak rejtélyes teljesítménycsökkenést. Ezt öt fő szempont alapján vizsgálja a könyv:

> 📍 **[IDE JÖN A KÉP: Figure 2.17: Various aspects of network performance]**
>
> Ide jön a hálózati teljesítmény 5 aspektusát mutató ábra.

**Teljesítménykorlátok (Network Performance Constraints - NPC):**
- Ideiglenes túlterhelés (congestion) a routereken csomagvesztést okoz.
- **Erőforrás-egyensúlyhiány:** Egy gigabites hálózatra kötött lassú gép nem győzi feldolgozni a csomagokat, azok elvesznek, majd az újra küldések (retransmission) miatt sávszélességet pazarolnak.
- **Broadcast storm (Szórási vihar):** Egy hibás csomagra milliónyi gép küld vissza hibaüzenetet, ami megbénítja a hálózatot (gyakori DoS anomália).

**Paraméterek hangolása (NPPT):** Iteratív folyamat: mérések elvégzése, a történések megértése, a paraméterek módosítása, majd újra mérés, amíg a teljesítmény nem javul.

**Teljesítmény-orientált rendszertervezés (POSD):**
- Szabályok jó hálózatok tervezéséhez: A CPU sebesség fontosabb a hálózati sebességnél; csökkentsd a csomagszámot (kisebb overhead); kerüld az adatmásolást (copy); a nagy sávszélesség előnyösebb az alacsony késleltetésnél; előzd meg a torlódást, ne pedig utólag próbálj felépülni belőle.

**Protokollok gigabites hálózatokhoz (PGN):** Mivel a kommunikációs sebesség sokkal gyorsabban nőtt, mint a CPU sebesség, a hagyományos protokolltervezési elvek megbuknak. A sebességre kell optimalizálni, nem csak a sávszélesség kihasználására.

**A TPDU (Transport Protocol Data Unit) gyorsabb feldolgozása (FPTPD):**
- Cél a feldolgozási overhead csökkentése (bájtonként és csomagonként). Különválasztják a normál folyamatokat a speciális esetektől. Ha a kapcsolat létrejött (Established state), a csomagok (TPDU-k) feldolgozása egyenes vonalú (straightforward), ami drasztikusan felgyorsítja az adatátvitelt.

- ## 2.2 Anomáliák a hálózatban (Anomalies in a Network)

Az anomáliák olyan adatpéldányok (pontok, objektumok, események, mintázatok), amelyek nem felelnek meg a normális viselkedés elfogadott fogalmának. A hálózatokban az anomália olyan körülményeket jelent, amelyek miatt a hálózati műveletek eltérnek a megszokottól.

Az anomáliák két fő kategóriába sorolhatók:

1. **Teljesítményfüggő anomáliák (Performance-related anomalies):** Rendszerint a hálózati sebezhetőségekből, túlterhelésből vagy rossz konfigurációból fakadnak.
2. **Biztonsági anomáliák (Security-related anomalies):** Rosszindulatú tevékenységekből (támadások, behatolások) származnak.

### 2.2.1 Hálózati sebezhetőségek (Network Vulnerabilities)

A sebezhetőségek a hálózat tervezésében, implementációjában és menedzsmentjében lévő belső gyengeségek, amelyek fogékonnyá teszik a rendszert a fenyegetésekre. Az ismert sebezhetőségek általában három alapvető forrásra vezethetők vissza:

- **Gyenge tervezés (Poor design):** Hardveres vagy szoftveres architekturális hibák. Példa: A korai UNIX rendszerek "sendmail" hibája, amely root (rendszergazdai) hozzáférést biztosított a hackereknek.
- **Gyenge megvalósítás/konfiguráció (Poor implementation):** Tapasztalathiányból vagy hanyagságból fakadó beállítási hibák. Példa: Kritikus futtatható fájlokon hagyott "mindenki által írható" jogosultság.
- **Menedzsment hiánya (Lack of management):** Ellenőrzési mechanizmusok és dokumentáció hiánya. Egyetlen embernek sem szabadna totális kontrollal rendelkeznie egy biztonságos rendszer felett (checks and balances hiánya).

A könyv a sebezhetőségeket 6 specifikus altípusra bontja:

#### 2.2.1.1 Konfigurációs sebezhetőségek (Network Configuration Vulnerabilities)

- **Gyenge hálózati architektúra:** Gyorsan növekvő cégeknél a biztonság gyakran elmarad a hálózat növekedésétől, "hátsó ajtókat" (backdoor) hagyva.
- **Adatáramlás-szabályozás hiánya (Lack of data flow controls):** Nem megfelelő hozzáférés-vezérlési listák (Access Control List - ACL) használata.
- **Biztonsági eszközök rossz konfigurációja:** Gyári (default) jelszavak meghagyása a routereken, nyitott portok, feleslegesen futó szolgáltatások, vagy hibás tűzfalszabályok.
- **Eszközkonfigurációk biztonsági mentésének hiánya:** Egy támadás utáni helyreállítás (Disaster Recovery) lehetetlen megfelelő backup nélkül.
- **Védetlen jelszóátvitel:** Titkosítatlan csatornán küldött jelszavak, amelyeket a támadó lehallgathat (eavesdropping / sniffing).
- **Jelszavak túl hosszú ideig tartó használata:** Lehetővé teszi az offline szótáralapú (dictionary) támadásokat.

#### 2.2.1.2 Hardveres sebezhetőségek (Network Hardware Vulnerabilities)

**Fizikai védelmi és kontrollmechanizmusok hiánya:**

- Fizikai védelem hiánya (bárki bemehet a szerverszobába).
- **Nem biztonságos fizikai portok:** Szabadon hagyott USB vagy PS/2 portok (ahova egy támadó bedughat egy keyloggert vagy pendrive-ot).
- **Környezetszabályozás hiánya (Environment control):** Hűtés leállása, ami a processzorok túlmelegedéséhez és a szerverek leállásához (fizikai DoS) vezet.
- Fizikai hozzáférés-vezérlés (beléptetés) és fizikai adatmentés hiánya.

#### 2.2.1.3 Hálózati peremvédelmi sebezhetőségek (Network Perimeter Vulnerabilities)

- **Jól definiált perem (Security perimeter) hiánya:** Nem világos, hol végződik a belső hálózat és hol kezdődik a publikus zóna.
- **Helytelen tűzfal konfiguráció:** Lehetővé teszi a rosszindulatú szoftverek (malware) hálózatok közötti terjedését.
- **Vezérlő és nem-vezérlő forgalom keverése:** (Különösen ipari / SCADA rendszereknél kritikus, ahol az irodai netet és a gyártósori hálózatot fizikailag/logikailag szeparálni kell).

#### 2.2.1.4 Monitorozási és naplózási sebezhetőségek

A naplók (logok) hiánya lehetetlenné teszi az incidensek okainak utólagos felderítését (forensics). A rendszeres monitorozás hiányában a támadók hetekig észrevétlenül tevékenykedhetnek.

#### 2.2.1.5 Kommunikációs sebezhetőségek

- Monitorozási útvonalak hiánya.
- **Jól dokumentált, tiszta szöveges (clear-text) protokollok használata:** Olyan régi protokollok, mint az FTP, Telnet vagy NFS nem használnak titkosítást, így a támadók hálózati analizátorokkal (pl. Wireshark) könnyen kiolvashatják az adatokat.
- **Hitelesítés (Authentication) hiánya:** Eszközök vagy felhasználók azonosításának hiánya, ami megkönnyíti az adatok hamisítását (spoofing) vagy a "Visszajátszásos" (Replay) támadásokat.
- **Integritás-ellenőrzés (Integrity checking) hiánya:** A támadó észrevétlenül módosíthatja a kommunikációt (Man-in-the-Middle).

#### 2.2.1.6 Vezeték nélküli (Wireless) sebezhetőségek

- **Nem megfelelő hitelesítés:** Gyenge védelem miatt a legitim kliens egy támadó által üzemeltetett hamis hozzáférési ponthoz (Rogue Access Point / Evil Twin) csatlakozhat.
- **Nem megfelelő adatvédelem:** Titkosítatlan adatok sugárzása a levegőben.

### 2.2.2 Biztonsági anomáliák (Security-Related Network Anomalies)

Ez a gépi tanulás szempontjából a legfontosabb kategória. A hálózatokban a biztonsági anomáliákat három fő ML-kategóriába soroljuk (a könyv egy bankkártyás csalás példáján keresztül mutatja be, de kiegészítem hálózati példákkal is):

**1. Pontszerű anomália (Point anomaly):**
- **Definíció:** Egyetlen, önmagában álló adatpéldány, amely drasztikusan eltér a teljes adathalmaztól (izolált objektum: O1).
- **Könyvbeli példa:** Egy hirtelen, irreálisan magas összegű bankkártyás vásárlás.
- **Hálózati ML példa:** Egyetlen, extrém méretű ICMP (Ping) csomag, ami a normál 64 bájt helyett 65000 bájtos (ez a Ping of Death támadás).

**2. Kontextuális anomália (Contextual anomaly):**
- **Definíció:** Egy adott kontextuson vagy tartományon belül számít csak kiugrónak, míg globálisan talán normális lenne (objektum: O2 a C2 csoport kontextusában).
- **Hálózati ML példa:** Egy adatbázis szerver 5 GB adatot küld kifelé. Ez önmagában nem baj (nincs Point anomaly). De ha ezt hajnali 3-kor teszi, miközben a napi kontextus (munkaidő) szerint ekkor 0 forgalomnak kellene lennie, az már kontextuális anomália (esetleges adatcsempészés / Data Exfiltration).

**3. Kollektív anomália (Collective anomaly):**
- **Definíció:** Az egyes adatpontok külön-külön vizsgálva nem feltétlenül anomáliák, de egy adott csoportjuk együtt már eltér a normálistól (objektumcsoport: C3).
- **Hálózati ML példa:** DDoS (Distributed Denial of Service) támadás vagy Port Scan. Egyetlen apró TCP SYN csomag teljesen normális. Viszont 10 000 darab TCP SYN csomag egymás után az elmúlt 1 másodpercben már egyértelmű kollektív anomália.

> 📍 **[IDE JÖN A KÉP: Figure 2.18: Point, collective and contextual anomalies]**
>
> Ez az ábra vizuálisan reprezentálja az O1 (Point), O2 (Contextual) és C3 (Collective) anomáliákat egy kétdimenziós koordináta-rendszerben.

### 2.2.3 Ki támadja a hálózatokat? (Who Attacks Networks)

A hálózati támadás (network attack) egy rosszindulatú kísérlet a rendszer sebezhetőségeinek kihasználására. Az elkövetőt támadónak (attacker) vagy behatolónak (intruder) nevezzük.

Anderson (1980-as klasszikus modellje alapján) a behatolókat két fő típusra osztja:

1. **Külső behatolók (External intruders):**
   - Jogosulatlan felhasználók, akik kívülről próbálnak betörni a rendszerbe (pl. egy interneten keresztüli hacker). Nincs semmilyen fiókjuk vagy hozzáférésük a hálózaton belül.
2. **Belső behatolók (Internal intruders):**
   - Olyan felhasználók, akiknek van valamilyen szintű legitim (de korlátozott) hozzáférésük a rendszerhez, de nincs rendszergazdai (root/super user) jogosultságuk, és ezzel próbálnak visszaélni (Insider threat).

**A Belső behatolók két további alkategóriája:**

- **Álcázott behatoló (Masquerade intruder):** Egy másik, legitim felhasználó fiókjába lép be (pl. ellopta a kollégája jelszavát), hogy hozzáférjen az általa nem látható szenzitív adatokhoz.
- **Rejtett/Titkos behatoló (Clandestine intruder):** A legveszélyesebb típus. Olyan magas (vagy megszerzett) jogosultsággal rendelkezik, amellyel képes kikapcsolni az auditálást és a naplózást (audit control). Mivel eltünteti a nyomait, az IDS rendszerek számára rendkívül nehéz detektálni.

### 2.2.4 A támadás előfutárai (Precursors to an Attack)

A támadásokat ritkán hajtják végre egyetlen, hirtelen lépésben. A behatoló egy előre megtervezett eseménysorozatot (precursors) hajt végre a cél elérése érdekében. Ezeknek a lépéseknek a sorrendje és időtartama függ a támadó képzettségétől, a kihasználandó sebezhetőségtől és a támadó kiindulási helyzetétől.

Egy sikeres hálózati támadás hagyományosan négy alapvető fázisból áll:

1. **Előkészítés (Prepare):**
   - A támadó információt gyűjt a hálózatról és a konfigurációkról (pl. port szkennerekkel, mint az nmap).
   - **Cél:** Aktív IP címek, futó operációs rendszerek, nyitott portok és a rajtuk figyelő szoftverek (és azok verziószámainak) azonosítása.
2. **Kihasználás (Exploit):**
   - A feltérképezett sebezhetőségek (vulnerabilities) alapján a támadó elindítja magát a támadást (vagy akár egyszerre többet is). Ekkor töri fel a rendszert.
3. **Hátrahagyás (Leave behind):**
   - A sikeres betörés után a támadó biztosítja, hogy később is könnyedén vissza tudjon térni.
   - **Eszközök:** Hátsó ajtók (Backdoors) telepítése, hálózati lehallgatók (Snifferek) elhelyezése a forgalom további megfigyelésére.
4. **Takarítás (Cleanup):**
   - A támadó eltünteti a behatolás nyomait, hogy az adminisztrátorok (vagy az egyszerűbb IDS rendszerek) ne vegyék észre.
   - **Lépések:** Logfájlok (naplók) törlése, a betörés során összeomlott szolgáltatások (daemons) újraindítása, és a rendszerfájlok módosítása úgy, hogy a kártékony szoftverek rejtve maradjanak a standard ellenőrző parancsok elől (pl. Rootkit telepítése).

> 💡 **ML/IDS Szempont:** A gépi tanulási modellek feladata sokszor az, hogy már az 1. fázisban (Prepare) riasztást adjanak. Ha a modell felismeri a port szkennelésre utaló finom anomáliákat (pl. túl sok "Connection Rejected" ICMP válasz vagy TCP RST csomag rövid időn belül), a támadás még a 2. fázis előtt megállítható.

### 2.2.5 Hálózati támadások taxonómiája (Network Attacks Taxonomy)

A taxonómia egy konzisztens osztályozási rendszer, amely segít megérteni a támadások és alkategóriáik közötti kapcsolatot.

> 📍 **[IDE JÖN A TÁBLÁZAT: Table 2.6: Taxonomy of Attacks]**
>
> Ez a táblázat foglalja össze a támadásokat a kivitelezésük módja szerint 7 fő kategóriába.

**A támadások 7 fő kategóriája (Kivitelezés alapján):**

1. **Fertőzés (Infection):** Vírusok, Férgek (Worms), Trójaiak.
2. **Kimerítés / Robbantás (Exploding):** Puffer-túlcsordulás (Buffer Overflow).
3. **Feltérképezés (Probe):** Lehallgatás (Sniffing), Port térképezés, Biztonsági szkennelés.
4. **Csalás / Hamisítás (Cheat):** IP Spoofing, MAC Spoofing, DNS Spoofing, Munkamenet-eltérítés (Session Hijacking), XSS (Cross Site Scripting).
5. **Bejárás / Törés (Traverse):** Brute Force, Szótár alapú támadások (Dictionary), Doorknob támadások.
6. **Párhuzamosság (Concurrency):** Flooding (elárasztás), DDoS (Kiosztott szolgáltatásmegtagadás).
7. **Egyebek (Others):** Rendszerhibák triviális kihasználása (általában képzetlen "script kiddie"-k által).

#### Passzív vs. Aktív támadások

- **Passzív támadások:** Közvetett hatásuk van. Céljuk a forgalom megfigyelése, rögzítése és információszerzés egy későbbi aktív támadáshoz. Mivel nincs "nyílt" kártékony tevékenység (csak hallgatóznak), nagyon nehéz őket detektálni. (Példa: Packet sniffing).
- **Aktív támadások:** Sokkal veszélyesebbek, közvetlen és pusztító hatással vannak a hálózatra.

#### A DARPA osztályozási rendszer (Az Aktív támadások 4 fő kategóriája)

A hálózati anomália detekció szakirodalmában (és az ML adathalmazokban, mint a KDD Cup 99 vagy az NSL-KDD) szinte mindig ezt a négy kategóriát használják az aktív támadások besorolására:

**1. Szolgáltatásmegtagadás (Denial of Service – DoS)**
- **Cél:** Blokkolni a hozzáférést a hálózati erőforrásokhoz (pl. email, weboldal), vagy teljesen leállítani a hálózati kapcsolatot.
- **Működés:** Vagy újraindítja/lefagyasztja a célgépet, vagy annyira kimeríti az erőforrásait (sávszélesség, RAM, CPU), hogy a legitim felhasználók kéréseit már nem tudja kiszolgálni.
- **Gyakori típusok:** Ping of Death (PoD), TCP SYN Flood, Smurf, Neptune, Teardrop, Land attack.
- **ML Detekció:** Magas felismerési arány (általában $95\%+$), mivel a DoS támadások drasztikusan megváltoztatják a forgalom statisztikai jellemzőit (pl. csomagszám/másodperc).

**2. Felhasználóból Rendszergazda (User to Root – U2R)**
- **Cél:** A támadó normál (korlátozott) felhasználói jogosultsággal kezdi, és a rendszer sebezhetőségeit kihasználva rendszergazdai (root / superuser) jogosultságot szerez.
- **Működés:** A normál fiókot jelszólopással (sniffing) vagy szótár támadással szerzi meg, majd belülről támad (pl. hibásan megírt, rootként futó programok puffereinek túlcsordításával).
- **Gyakori típusok:** Buffer overflow, rootkit, loadmodule, Perl exploiotok.
- **ML Detekció:** A legnehezebben felismerhető osztály. A hálózati forgalom teljesen "normálisnak" tűnik (hiszen a támadó egy létező, legitim fiókba van belépve). Itt a hálózati ML modellek gyakran elvéreznek, és inkább a gazdagép alapú (Host-based IDS) rendszerhívás-elemzés működik jól.

**3. Távoliról Helyi (Remote to Local – R2L)**
- **Cél:** A támadónak nincs felhasználói fiókja a célgépen. Csomagokat küld a hálózaton keresztül egy távoli gépre, hogy helyi (felhasználói) hozzáférést szerezzen.
- **Működés:** Nyilvános szolgáltatásokat (HTTP, FTP) vagy védett, de internetre kötött szolgáltatásokat (POP, IMAP) támad (pl. jelszó találgatással vagy ellenőrizetlen bemeneti adatokkal).
- **Gyakori típusok:** Jelszó kitalálása (Password guessing), warezclient, ftpwrite, multihop, phf, spy.

**4. Feltérképezés (Probe)**
- **Cél:** Hasznos információk gyűjtése (érvényes IP címek, futó szolgáltatások, op. rendszerek) a hálózat különféle szkennelésével.
- **Működés:** Ezek általában egy jövőbeli, nagyobb támadás előfutárai. A támadó megkeresi a hálózat gyenge pontjait.
- **Gyakori típusok:** IPsweep, portsweep, nmap.
- **ML Detekció:** Jól detektálható a "kapcsolat nélküli" csomagok arányából (pl. sok TCP portra küldött próbálkozás, amire a szerver hibaüzenetet ad).

### 2.2.6 Összefoglalás és Kitekintés (Discussion)

A 2. fejezet lefektette azokat az alapokat, amelyek a gépi tanulási modellek megértéséhez és fejlesztéséhez szükségesek.

- **Áttekintettük a hálózatok működését** (média típusok, OSI és TCP/IP referenciamodellek, protokollok, topológiák és hardverek). Ebből tudjuk, hogy az ML modell számára honnan és milyen formában nyerhetjük ki az adatokat (feature extraction).
- **Definiáltuk a hálózati anomáliákat**, különválasztva a teljesítményfüggő és a biztonsági anomáliákat (támadásokat).
- **Megismertük a betörők típusait**, a támadások fázisait és az Aktív támadások (DoS, U2R, R2L, Probe) kategóriáit.
