# 8. Fejezet: Eszközök és Rendszerek (Tools and Systems)

## 8.1 Bevezetés (Introduction)

A digitális eszközök és rendszerek védelmében a hálózatbiztonság három legfőbb célja a **Bizalmasság (Confidentiality)**, **Sértetlenség (Integrity)** és **Rendelkezésre állás (Availability)** – röviden a **CIA-triász** – biztosítása.

A támadók ezeket a védelmi vonalakat próbálják áttörni. Napjainkban az internetről könnyen letölthető, kifinomult támadóeszközök (pl. HOIC, LOIC) segítségével szinte bárki képes hálózati fennakadásokat okozni. Ezzel párhuzamosan a hálózati adminisztrátoroknak is speciális eszközökre van szükségük a hálózat védelméhez, monitorozásához és elemzéséhez.

### 8.1.1 A Támadó Motivációja (Attacker's Motivation)

A hálózatokat támadó egyéneket a motivációjuk alapján különböző osztályokba sorolhatjuk:

- **Kíváncsiak:** Akik csak letöltenek egy eszközt a webről, és rosszindulat nélkül kísérleteznek vele, de ezzel is kárt okozhatnak.
- **Bosszantók (Nuisance):** Hozzáértő egyének, akiknek a célja csupán a hálózat zavarása, mások bosszantása.
- **Profitorientált hackerek:** Veszélyes támadók, akik szándékosan, anyagi haszonszerzés céljából (pl. fontos információk ellopása, "sniffing") törnek be.
- **Politikai/Terrorista indíttatásúak:** Céljuk a hálózat teljesítményének rontása, a biztonsági rendszer demonstratív feltörése politikai nézetek kifejezésére, vagy a maximális kár okozása.

### 8.1.2 A Támadás Indításának Lépései (Steps in Attack Launching)

Ahogy a 2. fejezetben már szerepelt, egy támadó általában négy alapvető lépést hajt végre:

1. **Információgyűjtés (Előkészítés):** A hálózat feltérképezése sebezhetőségek (vulnerabilities) után kutatva.
2. **Kihasználás és Támadás (Exploit & Launch):** A talált sebezhetőségeket kihasználva a támadó elindítja az akciót a célpont (victim) ellen.
3. **Kiskapuk hagyása a jövőbeli hozzáféréshez (Leave behind):** Sikeres betörés után a támadó extra szoftvereket (pl. hátsó ajtók / back doors, sniffer-ek) telepít a gépre, hogy később is visszatérhessen.
4. **Nyomok eltüntetése (Clean up):** A támadó megpróbálja törölni a támadás történetét (pl. registry kulcsok vagy log fájlok törlésével/módosításával).

> 📍 **[IDE JÖN A KÉP: Figure 8.1: Steps of attack launching]**

### 8.1.3 Támadások Indítása és Detektálása (Launching and Detecting Attacks)

A támadások indításához elengedhetetlen a hálózat gyengeségeinek ismerete, amit a szkennelési fázisban (információgyűjtés) szereznek meg. A támadó a szolgáltatások megszakítására törekszik (erőforrás- vagy sávszélesség-kimerítéssel), jellemzően legitim kérések elárasztásával (Flooding), mint például a TCP SYN Flood, ICMP Flood vagy UDP Flood.

A detektáláshoz a hálózati adminisztrátornak olyan vizualizációs vagy monitorozó rendszerre van szüksége, amely valós időben, hatalmas adatmennyiség feldolgozásával képes különbséget tenni a normál és az abnormális forgalom között a csomagfejlécek (packet header) vagy a hálózati folyamatok (network flow) alapján. Mivel lehetetlen minden létező támadásra felkészülni, az IDS fejlesztőknek gondosan kell megválasztaniuk az észlelési mechanizmusokat (statisztikai, lágy-számítási, valószínűségi, tudásalapú vagy hibrid).

#### 8.1.3.1 Támadás-indító Eszközök és Rendszerek

A publikus eszközök a TCP/IP modell bármely rétegében bevethetők, de legtöbbször a Hálózati (Network) és Szállítási (Transport) rétegeket célozzák.

- **Alkalmazási réteg (Application layer):** HTTP, FTP, SMTP és DNS alapú támadások. Jellemzően kártevők (Trójaiak, vírusok, férgek) generálják őket. Gyakori a HTTP GET DDoS, a Cross Site Scripting (XSS) és az SQL Injection. Eszközök: `slowhttptest`, `AppDDoS`.
- **Adatkapcsolati réteg (Data link layer):** MAC, ARP és VLAN alapú támadások generálására. Eszközök: `Dsniff`, `IRPAS`, `Ettercap`, `Libnet`, `Gobbler`.

#### 8.1.3.2 Támadás-detektáló Eszközök és Rendszerek

A védelmi rendszerek feladata az ismert és ismeretlen anomáliák felismerése minimális téves riasztással. A folyamat lépései: hálózati forgalom rögzítése (capture), előfeldolgozás, jellemzőkiválasztás, majd az adatok elemzése.

- **Snort:** Egy jól ismert, könnyűsúlyú csomagszimatoló (packet sniffer) és detektáló rendszer. Képes felismerni például a Queso TCP/IP ujjlenyomat-olvasó szkenneléseket, az Nmap szkenneléseket vagy különböző "probe" (puhatolózó) kísérleteket.

**Architektúra:** A detektáló rendszer lehet hoszt-alapú (HIDS) vagy hálózat-alapú (NIDS). Egy tipikus hálózati felépítés tartalmaz egy védett LAN-t, egy IDS konzolt és egy DMZ-t (Demilitarizált Zóna).

**DMZ fogalma:** Egy fizikai vagy logikai alhálózat, amely a szervezet kifelé (külső felhasználók felé) nyújtott szolgáltatásait tartalmazza, és elválasztja a belső hálózatot a megbízhatatlan internettől.

> 📍 **[IDE JÖN A KÉP: Figure 8.2: Egy tipikus hálózati struktúra (védett LAN, DMZ és IDS konzol) ábrázolása]**


## 8.2 Támadással Kapcsolatos Eszközök (Attack Related Tools)

A rosszindulatú szándékkal rendelkező személyek különböző eszközöket használnak a hálózatok megzavarására. A célpontjaik általában weboldalak, adatbázisok és szervezeti hálózatok, amelyeket a gyengeségeik (weaknesses) feltérképezésével támadnak meg. A támadók a kívánt támadás kategóriájának (class of attack) megfelelő, specifikus eszközöket vetnek be.

Ezzel párhuzamosan a hálózatbiztonsági kutatócsoportok és magánbiztonsági szakemberek rengeteg védelmi eszközt is közzétettek, amelyek eltérő célokkal, képességekkel és kezelőfelületekkel rendelkeznek.

### 8.2.1 Taxonómia (Taxonomy)

A meglévő támadással kapcsolatos eszközöket három fő kategóriába soroljuk:

1. **Információgyűjtés (Information gathering)**
2. **Támadás-indítás (Attack launching)**
3. **Hálózat-monitorozás (Network monitoring)**

> 📍 **[IDE JÖN A KÉP: Figure 8.3: Taxonomy of attack-related tools]**
>
> Az ábra ezt a három kategóriát és azok alkategóriáit mutatja be.

### 8.2.2 Információgyűjtő Eszközök (Information Gathering Tools)

Mielőtt egy támadó lecsapna, meg kell értenie a környezetet. Első lépésként információt gyűjt a hálózatról: a gépek számáról, típusáról, az operációs rendszerekről (OS). Ezután specifikus eszközökkel megkeresi a hálózat gyenge pontjait.

Két fő típusuk van: a **Szimatoló (Sniffing)** eszközök, valamint a **Hálózattérképező és Szkennelő (Network mapping and scanning)** eszközök.

#### 8.2.2.1 Szimatoló Eszközök (Sniffing Tools)

A sniffing eszközök célja a hálózaton áthaladó csomagok (packets) vagy keretek (frames) elfogása (capture), vizsgálata, elemzése és vizualizációja. Mivel megértik a hálózati protokollokat (TCP, UDP, ICMP stb.), a protokoll-paramétereket is képesek megjeleníteni és kinyerni.

**A legfontosabb (24 darab) sniffing eszköz áttekintése:**

- **`Tcpdump`:** A legelső és legismertebb parancssoros csomagelemző információbiztonsági szakembereknek. Linux/Unix alapú, de a kimenetét (pcap) GUI-s szoftverek (pl. Wireshark) is tudják olvasni.
- **`Ethereal` (Mai nevén Wireshark):** GUI-alapú szoftver Windowsra és Unixra. Két fő könyvtárat használ: a GTK+-t a grafikához és a libpcap-et a csomagelfogáshoz és szűréshez. Több mint $400$ protokollt képes dekódolni.
- **`Gulp`:** Veszteségmentes (lossless) gigabites távoli csomagelfogó Linuxra. Két okból kiváló: (i) Sokkal kevesebb csomagot ejt el (drops), mint a versenytársak. (ii) Fejlett gyűrűs puffer (ring buffer) technológiája miatt sokkal gyorsabb a rögzítése.
- **`Net2pcap`:** Egy egyszerű Linux eszköz, amely a felületekről (interfaces) olvassa a csomagokat és `.pcap` fájllá alakítja őket könyvtárak (libraries) használata nélkül (kivéve a libc-t). Ellenséges (hostile) környezetekben használják.
- **`CDPsniffer`:** Egy egyszerű Perl-ben írt Cisco eszköz, ami a CDP (Cisco Discovery Protocol) csomagokat fogja el és dekódolja.
- **`Snoop`:** Hasonló a Tcpdump-hoz, de a kimeneti formátuma (Snoop) eltér a pcap-től (RFC 1761). Előnye, hogy terhelt hálózaton csökkenti a csomagvesztés esélyét.
- **`Snort`:** Egy könnyűsúlyú, platformfüggetlen visszaélés-felismerő (misuse detection) rendszer, ami egyben beépített sniffing és logoló támogatással is rendelkezik.
- **`Angst`:** Linuxra és OpenBSD-re. Egy aktív sniffer, ami adatokat fecskendez (inject) a hálózatba, hogy kapcsolt (switched) hálózatokat is tudjon hallgatni, például random MAC címekkel elárasztva a switcheket (MAC flooding).
- **`Ngrep`:** Tartalomszűrést (filtering) biztosít a csomagok payload-jára, miközben a tcpdump sniffing funkcionalitását is nyújtja (libpcap alapon).
- **`Ettercap`:** Egy aktív hacking eszköz multi-platformra. Képes switchelt hálózatokat hallgatni (interceptor), jelszavakat gyűjteni, kapcsolatokat megszakítani (kill connections), és aktív kapcsolatokba adatokat/parancsokat injektálni.
- **`Dsniff`:** Aktív sniffing eszközök gyűjteménye. Fő erőssége a Man-in-the-Middle (MitM) támadások végrehajtása SSHv1 és HTTPS munkamenetek ellen, forgalomátirányítással.
- **`Cain & Able`:** Windows-os többfunkciós sniffer. Főleg jelszó-visszafejtésre (password recovery) és Man-in-the-Middle támadásokra használják (pl. MSN, RADIUS, SSHv1).
- **`Aimsniff`:** Egy specifikus eszköz AOL Instant Messenger felhasználók IP-címének rögzítésére direkt kapcsolat létrejöttekor.
- **`Tcptrace`:** Egy erőteljes eszköz a `.pcap` fájlok utólagos elemzésére. Számos statisztikát generál a kapcsolatokról (küldött bájtok, eltelt idő, újra-küldések, válaszidők, RTT, ablakméretek).
- **`Tcptrack`:** Parancssoros eszköz, amely valós időben mutatja a hálózati felületen (interface) átmenő TCP kapcsolatokat (forrás IP, cél IP, portok, állapot, sávszélesség).
- **`Nstreams`:** TCP hálózati folyamatokat (streams) elemez és jelenít meg. Képes ipchains vagy ipfw tűzfalszabályokat generálni a megfigyelt folyamatok alapján.
- **`Argus`:** Robusztus, platformfüggetlen folyam-elemző (flow semantics). Az összes csomagparaméterről státuszjelentést ad (elérhetőség, időtartam, terhelés, csomagvesztés, jitter, késleltetés).
- **`Karpski`:** GUI-s eszköz limitált sniffing és szkennelő képességekkel, ami támadások indítására (attack launching) is használható a helyi hálózaton.
- **`IPgrab`:** Hálózati hibakereső (debugging) sniffer, amely több rétegen (adatkapcsolati, hálózati, szállítási) részletes fejléc-információkat mutat.
- **`Nast`:** Libnet-et és libpcap-et használ csomagok normál vagy promiscuous (minden csomagot befogadó) módban történő szimatolására. A payloadot ASCII vagy ASCII-hex formátumban menti.
- **`Aldebaran`:** Haladó TCP sniffing eszköz. Alapvető forrás/cél adatokat ad flag-ek nélkül. Jelszavak lopására (sniffing passwords) is alkalmas, és UDP-n keresztül képes a mentett adatokat egy másik hosztnak átküldeni, akár titkosítva is.
- **`ScoopLM`:** Windows-os sniffer az LM/NTLM hitelesítési (authentication) adatok elfogására a hálózaton, amit később jelszótörő (pl. BeatLM) programok használhatnak.
- **`Nfdump`:** Netflow forgalom vizualizálására és elemzésére szolgál (Netflow v5, v7, v9 támogatás). Az nfcapd daemont használja a logoláshoz, és képes "Top N" folyamok statisztikáinak megjelenítésére.
- **`Nfsen`:** Az Nfdump grafikus, Web-alapú frontendje. Lehetővé teszi az adatok böngészését (RRD adatbázissal), specifikus időablakok lekérdezését, és egyedi pluginok írását.

> **Szerzői megjegyzés:** A **Gulp** a legkiválóbb nyers csomagelfogó a minimális adatvesztés miatt, a **Wireshark (Ethereal)** a legjobb az utólagos vizuális csomagelemzésre, míg a Flow adatok (folyamatok) kezeléséhez az **Nfdump/Nfsen** páros a legfelhasználóbarátabb.

#### Táblázat 8.1: Szimatoló (Sniffing) Eszközök

| Eszköz neve | Használt Platform | Forrás (URL) |
| :--- | :--- | :--- |
| `Ethereal` | Linux, Unix, Windows | http://www.ethereal.com |
| `Tcpdump` | Linux, Unix | http://www.tcpdump.org |
| `Net2pcap` | Linux | http://www.secdev.org |
| `Cdpsniffer` | -- | -- |
| `Snoop` | Solaris | http://www.softpanorama.org |
| `Snort` | Linux, Windows | http://www.snort.org |
| `Angst` | Linux, BSD | http://www.angst.sourceforge.net |
| `Ngrep` | Linux, Windows | http://www.ngrep.sourceforge.net |
| `Ettercap` | Linux, Windows | http://www.ettercap.sourceforge.net |
| `Dsniff` | Unix | http://www.naughty.monkey.org |
| `Cain & Able` | Windows NT/XP | http://www.oxid.it |
| `Aimsniff` | Linux | http://www.sourceforge.net |
| `Tcptrace` | Linux, Windows | http://www.tcptrace.org/ |
| `Tcptrack` | Linux | http://www.rhythm.cx |
| `Nstream` | Linux, Windows | http://www.hsc.fr/cvs.nessus.org |
| `Argus` | Linux, Windows | http://www.qosient.com/argus/ |
| `Karpski` | Linux | http://www.softlist.net |
| `IPgrab` | Linux, BSD | http://www.ipgrab.sourceforge.net/ |
| `Nast` | Linux, BSD | http://www.nast.berlios.de |
| `ScoopLM` | Windows | http://www.securityfriday.com |
| `GULP` | Linux | http://staff.washington.edu/corey |
| `Libpcap` | Linux, Windows | http://www.tcpdump.org |
| `Nfsen` | Linux | http://www.nfsen.sourceforge.net |
| `Nfdump` | Linux | http://www.nfdump.sourceforge.net |

#### 8.2.2.2 Hálózat-térképező vagy Szkennelő Eszközök (Network Mapping or Scanning Tools)

A hálózat-szkennelő eszközök elsődleges célja az aktív hosztok (gépek) azonosítása egy hálózaton belül, két fő okból: (i) hogy később megtámadják őket, vagy (ii) hogy felmérjék a hálózat sebezhetőségeit (biztonsági audit). Ezek az eszközök átfogó állapotjelentést adnak a hálózati hosztokról, nyitott portokról és IP-címekről.

A portszkenneléseknek négy lehetséges típusa van:

1. **Egy-az-egyhez (one-to-one)**
2. **Egy-a-sokhoz (one-to-many)**
3. **Sok-az-egyhez (many-to-one)**
4. **Sok-a-sokhoz (many-to-many)**

> 📍 **[IDE JÖN A KÉP: Figure 8.4: Different types of port scans]**
>
> Az ábra ezt a négy szkennelési típust illusztrálja.

**A legfontosabb hálózat-szkennelő eszközök a következők:**

- **`Nmap`:** A legismertebb hálózat-felderítő (network exploration) és biztonsági auditáló eszköz. Rendkívül gyorsan képes hatalmas hálózatokat (vagy egyes hosztokat) átvizsgálni. Nyers (raw) IP csomagokat használ, hogy kinyerjen olyan hasznos információkat, mint: elérhető hosztok, az általuk futtatott szolgáltatások, a használt operációs rendszerek (OS) és az esetleges csomagszűrők vagy tűzfalak megléte. A biztonsági auditok mellett a hálózati adminisztrátorok napi rutinfeladatokra (hálózati leltár karbantartása, szolgáltatások frissítési ütemterveinek kezelése, uptime monitorozás) is használják.
- **`Amap`:** Egy olyan eszköz, amely az alkalmazási protokollokat detektálja anélkül, hogy a szabványos TCP/UDP portokra támaszkodna. Specifikus "kiváltó" (trigger) csomagokat küld az adott portra – amelyek általában egy alkalmazás-szintű kézfogást (handshake) imitálnak. Mivel a legtöbb hálózati démon (daemon) csak a megfelelő kézfogásra válaszol (pl. SSL), az Amap a kapott válaszokat egyezteti az adatbázisával. Támogatja a TCP és UDP protokollokat, a normál és SSL-titkosított ASCII, illetve bináris protokollokat is. Képes olvasni az Nmap gépi kimeneti fájlját, és a képernyőre, valamint logfájlba is tud menteni.
- **`Vmap`:** Ez a térképező (mapper) eszköz lehetővé teszi egy démon pontos verziójának azonosítását oly módon, hogy "ujjlenyomatot" (fingerprinting) készít arról, hogyan válaszol a démon a hamis (bogus) parancsokra.
- **`Unicornscan`:** Egy aszinkron szkenner és payload (hasznos adat) küldő eszköz. Kiemelkedően skálázható és rugalmas; rendkívül gyorsan gyűjti és korrelálja az információkat. A gyors válaszidő érdekében elosztott TCP/IP vermet (stack) használ, és felhasználóbarát felületet biztosít ahhoz, hogy ingereket (stimulus) küldjenek a TCP/IP-alapú eszközökbe a válaszok mérésére. Főbb funkciói: aszinkron protokoll-specifikus UDP szkennelés, aszinkron állapotmentes (stateless) TCP szkennelés széleskörű TCP flag-variációkkal, és aszinkron állapotmentes TCP banner-grabbing.
- **`Ttlscan`:** A libnet és libpcap könyvtárakat használja a hosztok azonosítására úgy, hogy TCP SYN csomagokat küld a hoszt minden portjára. Elfogja (sniffs) a hoszttól érkező választ, és a csomagok tűzfal mögötti hosztra való továbbításával felméri a futó szolgáltatásokat. Képes felismerni a tűzfal mögötti gép operációs rendszerét és annak verzióját specifikus TCP fejléc-paraméterek (pl. TTL, ablakméret, IPID) olvasásával.
- **`Ike-scan`:** Az IPSec VPN szerverek felderítésére, ujjlenyomat-olvasására és tesztelésére szolgál az IKE (Internet Key Exchange) protokoll alapján. GPL licenc alatt fut Linux, Unix, Mac OS és Windows környezetekben is.
- **`Paketto`:** Egy eszközkészlet, amely a TCP/IP hálózatok nem-hagyományos stratégiákon alapuló manipulálását segíti. Képessé teszik a meglévő infrastruktúrán belüli lehallgatást (tapping), és a protokollokat az eredeti céljukon túl is kiterjesztik. Példák a csomagban lévő eszközökre: (1) **Scanrand:** hálózati szolgáltatások és topológiák gyors felderítése; (2) **Minewt:** userspace NAT/MAT router; (3) **Linkcat:** ethernet link biztosítása stdio-hoz; (4) **Paratrace:** hálózati útvonalak követése új kapcsolatok nyitása nélkül; (5) **Phentropy:** adatforrások entrópiájának 3D fázistérben való megjelenítése az OpenQVIS használatával.

##### Táblázat 8.2: Térképező/Szkennelő (Mapping/Scanning) Eszközök

| Eszköz (Tool) | Használt Platform | Forrás (Source) |
| :--- | :--- | :--- |
| `Nmap` | Linux, Windows | http://www.insecure.org |
| `Amap` | Linux, Unix | http://www.freeworld.thc.org |
| `Vmap` | Unix | http://www.tools.l0t3k.net |
| `Unicornscan` | Linux, Unix | http://www.unicornscan.org |
| `Ttlscan` | Linux | http://www.freebsd.org |
| `Ike-scap` | Linux, Unix | http://www.stearns.org |
| `Paketto` | Linux | http://www.packages.com |

**Értékelés:** Mint látható, ezek az eszközök szinte kivétel nélkül Linux-alapúak. A térképező és szkennelő eszközök közül az **Nmap** bizonyult a leghatékonyabbnak és legrobusztusabbnak, mivel minden módban (egy-az-egyhez, egy-a-sokhoz, sok-az-egyhez, sok-a-sokhoz) képes feltérképezni a hálózatot. Az Nmap egyik változata, az **rNmap**, koordinált módban (sok-az-egyhez vagy sok-a-sokhoz) teszi lehetővé távoli gépek szkennelését, különböző flag beállításokkal. (Mint a 4. fejezetben említettük, a TUIDS Portscan adathalmaz is nagyrészt ezzel az eszközzel lett generálva).

Ezek az eszközök a támadó számára pofonegyszerűvé teszik az áldozat azonosítását (victim identification), mivel pontosan felfedik a célgép tűzfal- és szűrőbeállításait, az elérhető szolgáltatásokat és a futó operációs rendszert.

### 8.2.3 Támadás-indító Eszközök (Attack Launching Tools)

A világhálón rengeteg olyan hálózatbiztonsági eszköz érhető el, amellyel támadásokat lehet indítani. Bárki szabadon letöltheti és felhasználhatja ezeket olyan rosszindulatú tevékenységekre, mint a Trójaiak terjesztése, hálózati térképezés, puhatolózó (probe) támadások, puffer-túlcsordulás (buffer overflow), DoS/DDoS támadások és az alkalmazás réteg elleni támadások.

Léteznek specifikus eszközök az egyes hálózati rétegek és protokollok (HTTP, SMTP, FTP, SNMP) ellen, míg más eszközök kifejezetten azért jöttek létre, hogy másodpercek alatt megbénítsanak egy teljes hálózatot vagy weboldalt a DoS/DDoS technikák segítségével. Bizonyos eszközök vezetékes, mások pedig vezeték nélküli (wireless) hálózatokon képesek értékes információk elfogására és kihasználására (exploit).

> 📍 **[IDE JÖN A KÉP: Figure 8.5: DDoS attack graph]**
>
> Az ábra egy elosztott szolgáltatásmegtagadással járó támadás folyamatát / struktúráját mutatja be.

#### 8.2.3.1 Trójaiak (Trojans)

A Trójaiak olyan rosszindulatú végrehajtható (executable) programok, amelyeket kifejezetten egy számítógép vagy hálózat biztonsági rendszerének áttörésére fejlesztettek ki. Egy Trójai ártalmatlan (benign) programként bújik meg a rendszerben, majd amikor a felhasználó megpróbálja megnyitni a fájlt, a kód lefut, és végrehajtja a kártékony műveletet. Az áldozatok általában tudtukon kívül töltik le őket, többek között: (i) az Internetről (fertőzött weboldalak), (ii) FTP archívumokból, (iii) peer-to-peer (P2P) fájlcserélő hálózatokon IRC-n keresztül, vagy (iv) azonnali üzenetküldőkön (IM) át.

A Trójaiakat hagyományosan hét (7) különböző típusba sorolják:

1. **Távoli hozzáférésű Trójaiak (Remote Access Trojans - RAT):** Ezek a kártevők "hátsó ajtókat" (back doors) használnak arra, hogy adminisztrátori (rendszergazdai) jogokkal irányítsák a célgépet. A RAT-ok észrevétlenül töltődnek le, ha a felhasználó mondjuk egy játékot vagy egy e-mail mellékletet kér le. Miután a támadó kompromittálta a gépet, ezt a gépet használja ugródeszkának további gépek megfertőzésére, hogy így építsen fel egy óriási botnetet egy DoS vagy DDoS támadás indításához. *Példa: Danger trójai.*
2. **Adatküldő Trójaiak (Sending Trojans):** Ezen trójaiak célja, hogy érzékeny információkat (jelszavakat, hitelkártya-adatokat, log fájlokat, e-mail címeket, IM kapcsolatlistákat) rögzítsenek, és továbbítsanak a támadónak. Ennek eléréséhez általában egy keylogger-t (billentyűzetleütés-rögzítőt) telepítenek, ami minden gépelést elkap és elküld.
3. **Destruktív Trójaiak (Destructive Trojans):** Rendkívül pusztítóak; általában úgy programozzák őket, hogy automatikusan töröljenek alapvető végrehajtható fájlokat, konfigurációs fájlokat és DLL-eket (dinamikus csatolású könyvtárakat), hogy teljesen tönkretegyék a számítógépet. Ez a Trójai működhet (a) egy háttérszerver (back-end) közvetlen utasításai alapján, vagy (b) előre beprogramozott "időzített bombaként", ami egy adott napon és órában csap le. *Példák: Bugbear vírus és Goner féreg.*
4. **Proxy Trójaiak (Proxy Trojans):** Ez a típus az áldozat számítógépét "proxy szerverként" (köztes csomópontként) igyekszik felhasználni. A Trójai átveszi a gép feletti uralmat, és ezen keresztül hajt végre illegális tevékenységeket, például csalárd hitelkártya-tranzakciókat vagy támadásokat más hálózatok ellen, elrejtve a valódi támadó identitását.
5. **FTP Trójaiak (FTP Trojans):** Ez a típus megpróbálja kinyitni a 21-es portot az áldozat gépén, hogy közvetlen File Transfer Protocol (FTP) kapcsolatot építsen ki a támadóval a fájlok transzferéhez.
6. **Biztonsági szoftvert letiltó Trójaiak (Security Software Disable Trojans):** Az a feladatuk, hogy elpusztítsák vagy megbénítsák a gépen futó védelmi mechanizmusokat (antivírus programokat, tűzfalakat). Gyakran nem önmagukban, hanem egy másik (pl. adatküldő vagy RAT) Trójai kísérőjeként (payload) érkeznek, hogy kikövezzék neki az utat.
7. **DoS Trójaiak (DoS Trojans):** Arra tervezték őket, hogy azonnal, hatalmas mennyiségű haszontalan forgalommal (useless traffic) árasszák el a hálózatot, amíg az teljesen megbénul és nem képes további szolgáltatást nyújtani. Jól ismert klasszikus példák: **ping-of-death** és **Teardrop**.

8. #### 8.2.3.2 Szolgáltatásmegtagadással Járó (DoS) Támadások

A **Szolgáltatásmegtagadás (Denial of Service - DoS)** egy rendkívül gyakori és súlyos támadási forma. Lényege, hogy a támadó szándékosan megpróbálja megakadályozni, vagy blokkolni a legitim felhasználókat abban, hogy hozzáférjenek a kívánt erőforrásokhoz vagy szolgáltatásokhoz. Ez történhet centralizált és elosztott módon is. Klasszikus példák: SYN Flooding, smurf, fraggle, jolt, land és ping-of-death.

**A DDoS (Elosztott Szolgáltatásmegtagadás):**
A DDoS egy koordinált támadás a célpont (vagy célpontok csoportja), illetve a hálózati erőforrások rendelkezésre állása ellen. Ezt közvetve, az interneten lévő, nagyszámú kompromittált gépről indítják.

- **Architektúra:** A támadók általában $m:1$ (sok kompromittált gép támad egy célpontot) vagy $m:n$ megközelítést alkalmaznak, ami rendkívül megnehezíti a detektálást és a védekezést.
- **Botnetek és Zombik:** A támadást általában ágens-kezelők (agent handlers) vagy IRC (Internet Relay Chat) hálózatok segítségével koordinálják. A támadó gépek (zombik) leggyakrabban szélessávú kapcsolattal rendelkező személyi számítógépek, amelyeket vírusokkal vagy Trójai programokkal (botokkal) fertőztek meg. Az irányítás távolról, botnet parancsokon és vezérlőcsatornákon (C&C) keresztül történik.

**A legismertebb DoS/DDoS támadás-generáló eszközök (21 darab):**

- **`Jolt`:** Ez a DoS eszköz nagyszámú, töredezett (fragmented) ICMP csomagot küld a Windows 95 vagy NT célgépekre úgy, hogy a gép ne tudja azokat újra összerakni. Eredményképp a gép teljesen lefagy (nem reagál az egérre és a billentyűzetre sem). A kár nem maradandó, egy egyszerű újraindítás megoldja.
- **`Burbonic`:** Windows 2000 gépeket céloz. Hatalmas mennyiségű TCP csomagot küld véletlenszerű beállításokkal azzal a céllal, hogy a gép terhelését (load) annyira megnövelje, amíg az össze nem omlik (crash).
- **`Targa`:** Nem egyetlen eszköz, hanem 16 különböző DoS támadóprogram gyűjteménye. A támadások egyenként vagy csoportosan is indíthatók, azonnali kárt okozva a hálózatban.
- **`Blast20`:** Egy TCP szolgáltatás-stresszelő eszköz, amivel gyorsan felderíthetők a hálózati szerverek gyengeségei. (Például HTTP GET kérésekkel árasztja el a szervert).
- **`Crazy Pinger`:** ICMP csomagok (pingek) óriási tömegét küldi egy célpontra vagy egy távoli hálózatra.
- **`UDPFlood`:** Egy specifikus IP-címet és portot képes azonnal elárasztani UDP csomagokkal. A felhasználó beállíthatja az elárasztás sebességét, a maximális időtartamot és a csomagok számát. Szerverek teljesítménytesztelésére is használják.
- **`FSMax`:** Szerver stressz-tesztelő program. Célja a puffertúlcsordulási (buffer overflow) hibák keresése. Egy bemeneti szövegfájl alapján tesztek sorozatát futtatja le a szerveren.
- **`Nemsey`:** Jelenléte egy gépen azt jelenti, hogy a számítógép fertőzött (malware van rajta). Támadásokat tud indítani a támadó által megadott csomagszámmal, mérettel, protokollal és porttal.
- **`Panther`:** UDP-alapú DoS eszköz. Adott IP-címet és portot képes pillanatok alatt elárasztani.
- **`Slowloris`:** Hatalmas számú kapcsolatot nyit a célzott webszerver felé úgy, hogy részleges (partial) kéréseket küld, és ezeket a kapcsolatokat a végtelenségig nyitva tartja. Emiatt a szerver kimeríti a maximális egyidejű kapcsolat-készletét (concurrent connection pool), és kénytelen megtagadni az új, legitim felhasználók csatlakozását.
- **`BlackEnergy`:** Webalapú DDoS eszköz. Egy HTTP-alapú botnet, amely IRC-alapú parancs-és-vezérlő (C&C) módszert használ.
- **`HOIC` (High Orbit Ion Cannon):** HTTP DDoS eszköz, amely nagysebességű, többszálú (multithreaded) HTTP elárasztásra fókuszál. Akár $256$ weboldalt is képes egyidejűleg támadni. Beépített szkriptrendszere ún. "gyorsítókat" (boosters) használ, amelyekkel megkerüli a DDoS elleni védelmi mechanizmusokat.
- **`TRINOO`:** Hatékony DDoS eszköz, amely egy "mester" (master) gépet és több "közvetítő" (broadcast) gépet használ. A támadó TCP-n ad parancsot a mesternek, aki UDP-n utasítja a közvetítőket, hogy árasszák el a célt UDP csomagokkal, véletlenszerű portokon. Használatához a támadónak már előzetesen hozzá kell férnie (kompromittálnia kell) a gépekhez a szoftver telepítéséhez.
- **`Shaft`:** A TRINOO egy változata, amely statisztikákat szolgáltat a TCP, UDP és ICMP elárasztásokról. Ezzel a támadó látja a célpont állapotát (életben van-e vagy összeomlott), és eldöntheti, hogy bevonjon-e új zombikat.
- **`Knight`:** IRC-alapú eszköz Windows gépek ellen. SYN flood, UDP flood és Urgent pointer flood támadásokra képes.
- **`Kaiten`:** IRC-alapú eszköz, amely véletlenszerűen generált (spoofed) forrás IP-címekkel képes UDP, TCP, SYN és PUSH+SYN támadásokat indítani.
- **`RefRef`:** Meglévő SQL injekciós sebezhetőségeket és a MySql SELECT jogosultságait használja fel. Rosszul formázott (malformed) SQL lekérdezéseket küld, amik arra kényszerítik az SQL szervert, hogy saját magát terhelje túl és kimerítse az erőforrásait (Perl compiler-rel működik).
- **`LOIC` (Low Orbit Ion Cannon):** Anonim támadóeszköz IRC-n keresztül. TCP, UDP és HTTP módban tud támadni, többszálú (multithreaded) végrehajtással. Van bináris és webalapú verziója is.
- **`Hgod`:** Windows XP-alapú eszköz. Képes meghamisítani a forrás IP-t, és megadni a protokollt/portot. Alapértelmezetten TCP SYN flood-ra használják.
- **`TFN` (Tribe Flood Network):** A Trinoo-hoz hasonló, kliensből és démonokból álló DDoS eszköz. ICMP, UDP, SYN és Smurf támadásokra képes. A továbbfejlesztett TFN2K titkosítást, rejtőzködő (stealth) támadásokat és shell parancsok küldését is támogatja.
- **`Stacheldrath`:** (Németül: szögesdrót). A TFN és a Trinoo hibridizációja. Kiegészítő funkciói: komponensek közötti titkosított kommunikáció és a démonok automatikus frissítése.

> **Összegzés a DoS/DDoS eszközökről:**
> Míg a klasszikus DoS támadásokhoz a **Targa** a legelterjedtebb, a pusztító elosztott (DDoS) támadásokhoz a **LOIC**, a **HOIC** és a **Stacheldrath** a legnépszerűbbek. A LOIC és a HOIC grafikus felülete (GUI) lehetővé teszi a forgalom intenzitásának finomhangolását. A Stacheldrath kifejezetten azért veszélyes, mert a hibrid képességei miatt többféle forgatókönyvet (pulzáló támadás, konstans sebességű, alcsoportos, egyre növekvő sebességű támadás) tud végrehajtani.

#### 8.2.3.3 Csomathamisító Eszközök (Packet Forging Attack Tools)

Ezek az eszközök a hálózati csomagok információinak manipulálására vagy hamisítására (forging) szolgálnak. Segítségükkel a támadó olyan forgalmat generálhat, amelynek IP-címe vagy fejléce meghamisított.

**A legfontosabb csomathamisító eszközök (13 darab):**

- **`Packeth`:** Linux-alapú, grafikus felületű eszköz. Bármilyen csomagot (vagy szekvenciát) képes elküldeni nyers foglalatok (raw sockets) használatával az etherneten. Rengeteg opciója van; direkt beállíthatók benne helytelen paraméterek is (pl. hibás ellenőrzőösszeg / checksum, vagy rossz fejléchossz) a hálózat teszteléséhez.
- **`Packit`:** Hálózati auditáló eszköz, amely testreszabja, injektálja, figyeli és manipulálja az IP forgalmat. Kiváló NIDS-ek tesztelésére, tűzfalak teljesítményének értékelésére és hálózati forgalom szimulálására.
- **`Packet excalibur`:** Képes csomagokat szimatolni (sniff), egyedi csomagokat építeni és fogadni, valamint hamisítani. Grafikus felülete van, amin szöveges szkripteket lehet építeni, és új protokollokat lehet definiálni.
- **`Nemesis`:** Unix és Windows alapú, parancssoros csomaggyártó (crafting) és injektáló eszköz. Bármilyen NIDS, tűzfal vagy IP-verem tesztelésére alkalmas. Szkriptelhető, és az IP/Ethernet injektálási módjaiban gyakorlatilag bármilyen egyedi csomag legyártható vele.
- **`Tcpinject`:** Széles skálájú TCP/IP csomagok küldését teszi lehetővé. Minden megadható benne: forrás/cél IP és port, csomagméret, payload, TCP vezérlő flag-ek és TCP ablakméret.
- **`Libnet`:** Egy magas szintű API (fejlesztői eszközkészlet), ami hordozható és egyszerűsített interfészt ad a programozóknak, hogy a saját alkalmazásaikból tudjanak csomagokat konstruálni és injektálni.
- **`SendIP`:** Parancssoros hamisító eszköz. Tetszőleges IP csomagokat tud küldeni, és minden fejlécmező tartalmát engedi manipulálni. Bármilyen egyedi adat hozzáilleszthető a küldött csomaghoz.
- **`IPsorcery`:** GTK+ grafikus felülettel rendelkező TCP/IP csomag-generátor, ami TCP, UDP és ICMP csomagok küldésére alkalmas.
- **`Pacgen`:** Linux-alapú Ethernet/IP/TCP/UDP csomag-generátor. A teljes protokollverem (és a payload) konfigurálható. Extra funkciója a darabszám-meghatározás (packet count) és a csomagok közötti programozható időköz.
- **`Arp-sk` (ARP Swiss Knife):** Teljesen tetszőleges, manipulált ARP kérések és csomagok készítésére szolgál, hálózatbiztonság és konnektivitás teszteléséhez.
- **`Arpspoof`:** ARP gyorsítótár-mérgezésként (ARP Cache Poisoning) ismert. Meghamisítja egy távoli gép ARP tábláját a helyi hálózaton (LAN). Kihasználja, hogy a helyi hálózaton a kommunikáció MAC címmel történik, míg a külső hálózat felé IP-címmel.
- **`Libpal`:** Felhasználóbarát csomag-összeállító könyvtár. Hamisított Ethernet, IP, ICMP, TCP és UDP csomagokat tud építeni "struct"-ok (adatstruktúrák) használatával.
- **`Aicmpsend`:** ICMP csomagküldő eszköz. Támogatja az ICMP elárasztást (flooding) és hamisítást (spoofing). Az összes létező ICMP flag és kód implementálható vele.

##### Táblázat 8.3: Csomathamisító (Packet Forging) Támadóeszközök

| Eszköz (Tool) | Használt Platform | Forrás (Source) |
| :--- | :--- | :--- |
| `Packeth` | Linux, Mac, Windows | http://www.sourceforge.net |
| `Packit` | Linux, Mac | http://www.packetfactory.openwall.ne |
| `Packet Excalibur` | Linux, Windows | http://www.freecode.com |
| `Nemesis` | Unix-szerű, Windows | http://www.sourceforge.net |
| `Tcpinject` | Linux | http://www.packetstormsecurity.org |
| `Libnet` | Linux | http://www.packetfactory.net/libnet |
| `SendIP` | Linux, Windows | http://www.softpedia.com |
| `IPsocery` | Linux | http://www.tools.l0t3k.net |
| `Pacgen` | Linux | http://www.sourceforge.net |
| `Arp-sk` | Linux | http://www.arp-sk.org |
| `ARP-SPOOF` | Linux, Windows | http://www.sourceforge.net |
| `Libpal` | Linux | http://www.sourceforge.net |
| `Aicmpsend` | Linux, Unix | http://www.packetstormsecurity.nl |

> **Összegzés a csomathamisító eszközökről:**
> Ezek közül az eszközök közül a **Packeth**, a **Packet excalibur** és a **Nemesis** a leghatékonyabbak. Sikerük titka a felhasználóbarát felület, az elképesztő mennyiségű beállítási lehetőség, a vizualizációs támogatás és a teljesen egyedi, "személyre szabott" csomagok injektálásának képessége. Bár képesek IP, ICMP és UDP csomagok kezelésére is, a gyakorlatban leginkább TCP/IP alapú forgalmi anomáliák generálására és tesztelésére használják őket.
>
> #### 8.2.3.4 Alkalmazási Réteg elleni Támadóeszközök (Application Layer Attack Tools)

Az alkalmazási réteg (Application Layer) elleni támadások során a támadó látszólag legitim, szabványos alkalmazási rétegbeli kéréseket (pl. HTTP kéréseket) használ – egy teljesen legitim módon csatlakozott hálózati gépről –, hogy túlterheljen (overwhelm) egy webszervert.

Ezek a támadások megnyilvánulhatnak munkamenet-elárasztás (session flooding), kérés-elárasztás (request flooding) vagy aszimmetrikus támadások formájában.

> **Nehézség:** Az alkalmazási rétegbeli DDoS támadások sokkal kifinomultabbak (subtle) és rejtőzködőbbek, mint a hálózati rétegbeli (pl. ICMP vagy SYN flood) támadások, és a detektálásuk rendkívül bonyolult, mivel a forgalom protokollja és a felépített kapcsolat is teljesen szabályosnak tűnik az IDS számára.

Négy fő típusukat különböztetjük meg:

1. **HTTP-alapú támadások:** Ide tartoznak a leginkább elterjedt eszközök és kártevők, mint például a Code Red féreg és mutációi, a Nimda féreg, a Cross Site Scripting (XSS) támadások, a rosszindulatú URL-ek (Malicious URLs) és az AppDDoS.
2. **SMTP-alapú támadások (E-mail):** Példák az e-mail elárasztás (SMTP Mail Flooding), SMTP férgek és mutációik, kiterjesztett Relay támadások, valamint a tűzfal-átlépő (Firewall Traversal) támadások.
3. **FTP-alapú támadások (Fájlátvitel):** Ezek az eszközök lehetővé teszik az FTP bounce (visszapattanó) támadásokat, FTP port-injekciós támadásokat, passzív FTP támadásokat és TCP szegmentációs támadásokat.
4. **SNMP-alapú támadások (Hálózatmenedzsment):** Ilyen például az SNMP elárasztás (SNMP flooding), az alapértelmezett "community" sztringek elleni támadások, és az SNMP put támadások.

> **Kutatási kihívás:** A négy típus közül a HTTP-alapú támadások tettek szert a legnagyobb jelentőségre. Bár születtek benchmark adathalmazok a detektáló mechanizmusok tesztelésére, a speciális XSS (Cross Site Scripting) támadástípusok robbanásszerű növekedése miatt a monitorozásukhoz és észlelésükhöz szükséges megfelelő eszköz(ök) még mindig hiányoznak. Erre a területre a biztonsági kutatóknak kiemelt figyelmet kell fordítaniuk.

#### 8.2.3.5 Ujjlenyomat-készítő Támadóeszközök (Fingerprinting Attack Tools)

A "fingerprinting" (ujjlenyomat-készítés) célja egy adott hálózati protokoll-implementáció specifikus jellemzőinek azonosítása, a bemeneti és kimeneti viselkedés elemzésével. Ezzel kideríthető a protokoll verziója, a szoftver gyártója (vendor) és a konfigurációs paraméterek, valamint az is, hogy milyen operációs rendszer fut a távoli célgépen. Szinte minden kulcsfontosságú internetes protokoll (ICMP, TCP, Telnet, HTTP) sebezhető a fingerprintinggel szemben. **Kétélű fegyver:** az adminisztrátorok leltározásra, a támadók sebezhetőség-keresésre, az IDS-ek pedig az abnormális féreg-viselkedések észlelésére használják.

**A legfontosabb ujjlenyomat-készítő eszközök (8 darab):**

- **`Nmap`:** A létező egyik legjobb eszköz Unixra és Windowsra is. Feltérképezi a portokat, az operációs rendszert (OS), a tűzfal módszereit és a futó szolgáltatásokat. Emellett az Nmap Scripting Engine (NSE) segítségével a felhasználók saját szkripteket írhatnak sebezhetőségek és hátsó ajtók (back doors) keresésére.
- **`P0f`:** Egy passzív OS ujjlenyomat-készítő eszköz. Ellentétben az Nmap aktív szkennelésével (amikor a program egyedi csomagokkal provokálja a célt), a P0f némán "szimatolja" a hálózatot, és csak az elcsípett forgalom alapján osztályozza a hosztokat. Ez sokkal nehezebb, de cserébe teljesen észrevehetetlen.
- **`Xprobe`:** Egy OS ujjlenyomat-készítő eszköz, amely az Nmap-hez hasonlóan egyszerű, de a felismeréshez az ICMP (Ping) protokollt használja ki.
- **`CronOS`:** Nmap-CronOS-ba ágyazott OS felismerő eszköz. Három fő opciója van: a `-s` a SYN RCVD állapot időtúllépését (timeout) találgatja, az `-i` az utolsó ACK állapot időtúllépését határozza meg, az `-f` opció pedig a FIN WAIT 1 állapot idejét használja ujjlenyomatként.
- **`Queso`:** Linux és Solaris rendszereken fut. Képes távolról meghatározni az OS verzióját és a gyártó (manufacturer) információit a hálózati csomagok elemzésével.
- **`AmapV4.8`:** Nem a szabványos portokra hagyatkozik (nem az alapértelmezett portokon hallgatózik), hanem hamis (bogus) kommunikációs adatokat küld, és az arra adott válaszok alapján ismeri fel az alkalmazásokat és szolgáltatásokat, beleértve a nem-ASCII alapú enterprise (vállalati) szoftvereket is.
- **`Disco`:** Főleg az egyedi IP-címek felderítésére (discovery) szolgál a hálózatban, emellett TCP SYN csomagokból is készít ujjlenyomatot.
- **`Sprint`:** OS azonosító eszköz, amely képes a gépek bekapcsolási idejének (up-times) kiszámítására is, és haladó "banner grepping" (szolgáltatásfejlécek kinyerése) funkciókkal is bír. Ha a `-n` kapcsolóval futtatják, a Netcraft működését szimulálja.

##### Táblázat 8.4: Ujjlenyomat-készítő (Fingerprinting) Eszközök

| Eszköz (Tool) | Használt Platform | Forrás (Source) |
| :--- | :--- | :--- |
| `Nmap` | Linux, Windows | http://www.nmap.org |
| `P0f` | Linux, Windows | http://www.lcamtuf.coredump.cx |
| `Xprobe` | Linux | http://www.sourceforge.net |
| `CronOS` | -- | -- |
| `Queso` | Linux, Solaris | http://www.tools.l0t3k.net |
| `AmapV4.8` | Linux, Unix | http://www.linux.softpedia.com |
| `Disco` | Linux, BSD | http://www.tools.l0t3k.net |
| `Sprint` | Unix | http://www.safemode.org |

#### 8.2.3.6 Felhasználói Támadóeszközök (User Attack Tools)

Ebbe a kategóriába azok a betörések tartoznak, amikor a támadó (i) normál, jogosult felhasználóként próbál root / szuperfelhasználói jogokat szerezni, vagy (ii) fiók (account) nélkül próbál meg helyi hozzáférést szerezni a géphez valamilyen hálózati sebezhetőségen keresztül. Ezeket borzasztó nehéz detektálni, mivel a viselkedésük a normál rendszerhasználatra hasonlít. Két fő kategóriájuk van:

**(i) U2R (User to Root) Támadás:**

> 📍 **[IDE JÖN A KÉP: Figure 8.6: U2R attack]**

Ebben a fázisban a támadó először jogosult felhasználóként szerez hozzáférést a géphez (pl. jelszó-szimatolás, szótártámadás, vagy Social Engineering útján). Ezután megkeresi a helyi operációs rendszer hibáit és sebezhetőségeit, hogy átváltson a mezei felhasználóból (User) root/szuperfelhasználó (Root) szintre. Amint megvan a root jog, a támadóé a teljes kontroll: hátsó ajtókat építhet ki, és rendszerfájlokat manipulálhat.

**Két jól ismert U2R eszköz:**

- **(a) `Yaga`:** A registry (rendszerleíró adatbázis) feltörésével hoz létre egy új adminisztrátori fiókot. Átírja a registry fájlokat úgy, hogy a gép bizonyos rendszerszolgáltatásai összeomoljanak (crash), majd kicsikarja az új admin fiók létrejöttét.
- **(b) `Sqlattack`:** A támadó TCP kapcsolatot épít ki egy SQL adatbázis szerverrel egy Unix gépen. Amikor az adatbázis-héjat (database shell) egy speciális "escape" szekvencia küldésével kilépteti, egy Perlmagic szkript futtatásával elindítja a gép root shell-jét.

**(ii) R2L (Remote to Local) Támadás:**

> 📍 **[IDE JÖN A KÉP: Figure 8.7: R2L attack]**

Ebben a támadásban a távoli támadó – akinek nincs helyi fiókja – csomagokat küld az áldozat gépének, és a gép sebezhetőségeinek kihasználásával próbál helyi (lokális) hozzáférést kicsikarni. Erre online és offline szótártámadásokat (dictionary attacks) alkalmaznak, amivel megpróbálják kitalálni a meglévő felhasználóneveket és a gyenge jelszavakat.

**Két jól ismert R2L eszköz:**

- **(a) `Netcat` (Trójaiként):** Egy Trójai program titokban feltelepíti és futtatja a Netcat-et az áldozat gépén az 53-as porton (ami általában a DNS portja, így átmegy a tűzfalon). Ezután a Netcat hátsó ajtóként (backdoor) funkcionál, amire a támadó jelszó és felhasználónév nélkül, parancssorral be tud lépni.
- **(b) `ntfsdos`:** A támadó konzol-hozzáférést szerez egy WinNT géphez az ntfsdos futtatásával. A program felcsatolja (mount) a gép merevlemezeit, így a támadó szabadon átmásolhatja a titkos fájlokat (pl. jelszó-hash-eket) egy külső adathordozóra.

> **Szakmai megjegyzés:** Ezek közül a SQL támadások (`Sqlattack`) a legveszélyesebbek. Mivel az SQL injekciós és manipulációs (crafted) variációk száma végtelen, a hagyományos statisztikai IDS módszerek gyakran elbuknak velük szemben.

#### 8.2.3.7 Egyéb Támadóeszközök (Other Attack Tools)

Ezek a hálózati diagnosztikai és tesztelő eszközök direkt vagy indirekt módon szintén a támadás-indítás (és az információgyűjtés) alapkövei:

- **(i) `Ping`:** Eredetileg a hálózati konnektivitás és elérhetőség (reachability) tesztelésére készült eszköz. ICMP Echo Request csomagokat küld, amire a célgép Echo Reply-jal válaszol, megmutatva a válaszidőt. Az ICMP protokoll alacsony prioritású, így lassabb a normál hálózati forgalomnál.
- **(ii) `Hping2`:** Egyedi, testreszabott (custom) TCP/IP csomagok küldésére szolgál. Kezeli a csomagtöredezettséget (fragmentation), képes tetszőleges méretű csomagokat küldeni, sőt, fájlátvitelre is alkalmas. Használják tűzfalszabályok tesztelésére, portszkennelésre, és a maximális átviteli egység útvonalának felderítésére (Path MTU discovery).
- **(iii) `Hping3`:** Ugyanazt tudja, mint a Hping2, de fejlettebb szekvenciaszám-kezeléssel (sequence numbers) rendelkezik. Minden elküldött csomagnál egy véletlenszerű alap forrásportról indul, és lépésenként növeli azt (vagy beállítható statikusra is).
- **(iv) `Traceroute`:** Megmutatja az útvonalat és felsorolja az összes köztes routert (ugrást/hopot) a forrás és a célpont között. Ezzel a támadó (vagy az admin) tökéletesen feltérképezheti, hogy az ISP hogyan csatlakozik a hálózathoz. Szinte minden operációs rendszerben beépített.
- **(v) `Tctrace`:** Bár hasonlít a traceroute-hoz, ICMP helyett TCP SYN csomagokat használ. Hatalmas előnye, hogy ha a támadó tudja, hogy a tűzfal átenged egy bizonyos TCP szolgáltatást (pl. 80-as port), akkor azokon a tűzfalakon is át tud látni vele.
- **(vi) `Tcptraceroute`:** UDP vagy ICMP ECHO kéréseket küld, és minden ugrásnál (hop) megnöveli a TTL (Time to Live) mezőt, amíg el nem éri a célpontot. Mivel a tűzfalak egy része szűri ezeket a csomagokat, gyakran elakad a cél elérése előtt.
- **(vii) `Traceproto`:** Egy C nyelvben írt traceroute-helyettesítő (replacement), amiben a felhasználó szabadon választhat a használni kívánt protokollok között (TCP, UDP, ICMP), így kiváló tűzfalak és csomagszűrők tesztelésére, kikerülésére.
- **(viii) `Fping`:** Fejlettebb és erősebb, mint a normál Ping. Bármennyi gépet, vagy egy listát (szövegfájlt) is képes végigszkennelni. Nem vár az időtúllépésre (timeout) egy gépen, hanem "Round-Robin" körkörös módszerrel kiküldi a pingeket az egész listának, és utólag veszi le a listáról a válaszolókat. Kiválóan automatizálható és szkriptelhető.
- **(ix) `Arping`:** Linux platformon fut. ARP Request (Címfeloldási protokoll) üzenetet küld egy LAN-on lévő célgépnek. Leginkább arra használják, hogy leellenőrizzék, foglalt-e már (in use) egy adott IP-cím a helyi hálózaton.

##### Táblázat 8.5: Kiegészítő Támadóeszközök (Additional Attack Tools)

| Eszköz (Tool) | Használt Platform | Forrás (Source) |
| :--- | :--- | :--- |
| `Ping` | Linux, Windows | http://www.download.cnet.com |
| `Hping2` | Linux, Mac | http://www.hping.org |
| `Hping3` | Linux, Mac | http://www.hping.org |
| `Traceroute` | Linux, Solaris, Windows | -- |
| `Tctrace` | Linux, Solaris | http://www.tcptrace.org/ |
| `Tcptraceroute` | Linux, Solaris | http://www.michael.toren.net/ |
| `Traceproto` | Linux | http://www.traceproto.sourceforge.net |
| `Fping` | Linux, Windows | http://www.softpedia.com |
| `Arping` | Linux | http://www.linux.softpedia.com |

### 8.2.4 Hálózat-monitorozó Eszközök (Network Monitoring Tools)

A hálózati forgalom monitorozása elengedhetetlen a hálózati adminisztrátorok számára az anomáliák megfigyeléséhez, elemzéséhez és végső soron azok azonosításához. Ahhoz, hogy az elemzések eredményeit (a logokat és statisztikákat) a szakemberek meg tudják érteni és értelmezni, elengedhetetlen a nyers forgalom vizualizációja. A vizualizáció segít azonnal észrevenni egy abnormális mintázatot (pl. hirtelen kiugró forgalmat), és lehetővé teszi a gyors beavatkozást, mielőtt a fertőzés továbbterjedne a hálózatban.

#### 8.2.4.1 Vizualizációs Eszközök (Visualization Tools)

A következő eszközök nemcsak a vizualizációban, hanem magában a hálózati elemzésben is segítenek:

- **`Tnv`:** Egy idő-alapú (time-based) forgalomvizualizáló eszköz. Bemutatja a csomagszintű részleteket és a kapcsolatokat a helyi és távoli hosztok között. Képes megnyitni libpcap fájlokat, élő (live) forgalmat rögzíteni, és a rögzített adatokat egy MYSQL adatbázisba menteni, ami kiemelkedően rugalmassá teszi a későbbi elemzést.
- **`Network Traffic Monitor`:** Támogatja a részletes forgalmi forgatókönyvek bemutatását és szkennelését egy alkalmazási folyamat kezdetétől fogva, és lehetővé teszi a forgalmi részletek mélyreható elemzését.
- **`Rumint`:** Ez az eszköz Windows környezetben teszi lehetővé az élőben rögzített forgalom, valamint a mentett PCAP fájlok vizualizációját.
- **`EtherApe`:** Lehetővé teszi az élő csomagadatok szimatolását (sniff) és a rögzített adatok monitorozását Unix környezetben (egy nagyon látványos, node-alapú, gráfos felülettel).
- **`NetGrok`:** Valós idejű hálózati vizualizációs eszköz. Grafikus elrendezést (layout) és fatérképet (tree map) kínál a hálózati adatok vizuális megszervezéséhez. Támogatja a valós idejű csomag- és nyomvonal (trace) rögzítést, valamint a szűrést.
- **`NetViewer`:** Nemcsak a rögzített élő forgalom összesített (aggregate) megfigyelését támogatja, hanem segít az anomáliák azonosításában is. Bemutatja a hasznos forgalmi jellemzőket a védelmi mechanizmusok finomhangolásához.
- **`VizNet`:** Segít vizualizálni a hálózat teljesítményét a sávszélesség-kihasználtság (bandwidth utilization) alapján.

##### Táblázat 8.6: Támadóeszközök (Attacking Tools)

| Eszköz (Tool) | Használt Platform | Forrás (Source) |
| :--- | :--- | :--- |
| `Jolt` | Windows 95/NT | -- |
| `Burbonic` | Linux, Windows | http://www.packetstormsecurity.org/ |
| `Targa` | Linux | http://www.packetstormsecurity.org/ |
| `Blas20` | Linux, Windows | -- |
| `Crazy Pinger` | Linux, Windows | http://www.softwaretopic.informer.com |
| `UDPFllod` | Windows | http://www.foundstone.com |
| `FSMax` | Windows | -- |
| `Nemsey` | Windows | http://packetstormsecurity.org/ |
| `Panther` | Windows | http://www.bestspywarescanner.net |
| `Land & LaTierra` | Windows | -- |
| `Slowloris` | Windows | http://www.ha.ckers.org/slowloris/ |
| `Blackenergy` | Linux, Windows | http://www.airdemon.net |
| `HOIC` | Windows, Linux | https://www.rapidshare.com |
| `Shaft` | Linux, Windows | -- |
| `Knight` | Windows | http://www.cert.org |
| `Kaiten` | Windows | http://www.mcafee.com |
| `RefRef` | Windows | -- |
| `Hgod` | Windows | -- |
| `LOIC` | Linux, Windows | http://www.sourceforge.net |
| `Trinoo` | Linux, Windows | http://www.nanog.org |
| `TFN` | Linux, Windows | http://www.codeforge.com |
| `TFN2K` | Linux, Windows | http://www.goitworld.com |
| `Stachaldraht` | Linux, Windows | http://www.packetstormsecurity.org |
| `Mstream` | Linux, Windows | -- |
| `Trinity` | Linux, Windows | -- |

##### Táblázat 8.7: Vizualizációs Eszközök (Visualization Tools)

| Eszköz (Tool) | Használt Platform | Forrás (Source) |
| :--- | :--- | :--- |
| `Tnv` | Linux, Windows | http://www.tnv.sourceforge.net |
| `Network Traffic Monitor 1.02` | Windows | http://www.monitor-networktraffic.winsite.com |
| `Rumint` | Windows | http://www.rumint.org |
| `EtherApe` | Linux, Unix | -- |
| `Netgrok` | Windows | http://www.softpedia.com |
| `Netviewer` | Windows | -- |
| `VizNet` | Windows | http://www.viznet.ac.uk |

##### Táblázat 8.8: Különféle Támadás-generáló Eszközök (Various Attack Generation Tools)

| Kategória | Eszköz (Tool) | Forrás (Source) |
| :--- | :--- | :--- |
| **Trójaiak** | `NukeNabbler` | -- |
| | `AIMSpy` | http://www.securitystronghold.com |
| | `NetSpy` | http://www.netspy-trojan-horse.downloads |
| **Információgyűjtés** | `ASS` | http://www.manpages.ubuntu.com |
| | `NMap` | http://www.nmap.org |
| | `p0f` | http://www.lcamtuf.coredump.cx/p0f.shtml |
| | `MingSweeper` | http://www.hoobie.net/mingsweeper |
| | `THC Amap` | http://www.freeworld.thc.org/thc-amap |
| | `Angry IP Scanner` | http://www.angryziber.com/w/Download |
| **DoS támadóeszközök** | `Targa` | http://www.security-science.com/ |
| | `Burbonic` | http://www.softpedia.com |
| | `Blast20` | -- |
| **Hamisító (Spoofing) támadóeszközök** | `Engage Packet Builder` | http://www.engage-packetbuilder.software.informer.com/ |
| | `Hping` | http://www.hping.org |
| | `Nemesis` | http://www.nemesis.sourceforge.net |
| | `Packet Excalibur` | http://www.linux.softpedia.com |
| | `Scapy` | http://www.softpedia.com |
| **TCP session eltérítő (Hijacking) eszközök** | `Firesheep` | http://www.codebutler.github.com/firesheep/ |
| | `Hunt` | http://www.packetstormsecurity.org/sniffers/hunt |
| | `Juggernaut` | http://www.tools.l0t3k.net/Hijacking/1.2.tar.gz |
| | `TTY Watcher` | http://www.security-science.com |
| | `IP Watcher` | http://www.download.cnet.com |
| | `Hjksuitv0.1.99` | http://www.tools.l0t3k.net/Hijacking/hjksuite-0.1.99.tar.gz |
| **Puhatolózó (Probe) támadóeszközök** | `Solarwind` | http://www.solarwinds.com |
| | `Network Probe` | http://www.softpedia.com |
| | `NMap` | http://nmap.org |
| **Hamisító (Spoofing) támadóeszközök vezeték nélküli (wireless) hálózaton** | `Kismet` | http://www.linux.die.net |
| | `libpcap` | http://www.sourceforge.net/projects/libpcap |
| | `libnet` | http://www.libnet.sourceforge.net |
| | `libdnet` | http://www.libdnet.sourceforge.net/ |
| | `libradiate` | http://www.packetfactory.net/projects/libradiate |
| **Alkalmazási réteg elleni támadóeszközök** | `HOIC` | https://www.rapidshare.com |
| | `LOIC` | http://www.softpedia.com |
| | `RefRef` | http://www.softpedia.com |

## 8.3 Támadás-detektáló Rendszerek (Attack Detection Systems)

Az előzőekben bemutatott támadások kivédésére létrejöttek azok a komplett IDS rendszerek, amelyek statisztikai, adatbányászati vagy lágy-számítási megközelítéseket alkalmaznak az ismert és ismeretlen behatolások azonosítására.

### 8.3.1 A legismertebb publikus / nyílt forráskódú Rendszerek

- **`Bro` (Ma már: Zeek):** Linux-alapú nyílt forráskódú rendszer, amely passzívan monitorozza a hálózatot valós időben. A csomagokból mély szintű alkalmazási szemantikát nyer ki. Nemcsak a szignatúrákat ismeri fel, hanem eseményorientált (event-oriented) és szokatlan támadásokat is, csomagvesztés nélkül, gyorsabban, mint a versenytársai. Biztosít viselkedési monitorozást, többrétegű elemzést és logolást. Erősen szkriptelhető.
- **`Snort`:** Könnyűsúlyú, szignatúra-alapú IDS, ami jellemzőszabályok (feature rules) és tartalom-minta illesztés (content pattern matching) alapján dolgozik. Kiváló puffer-túlcsordulások, rejtőzködő portszkennelések és CGI támadások ellen. Ha egyezést talál a szabállyal, háromféleképpen tud reagálni: `pass` (ejti a csomagot), `log` (elmenti a csomagot), `alert` (riasztást generál).
- **`Nessus`:** Nyílt forráskódú aktív sebezhetőség-kereső (vulnerability scanner). Windows verziója hatalmas adatbázisnyi szkripttel (plug-in) rendelkezik a hálózati gyengeségek felismerésére, egy barátságos HTTP szerveres webes interfésszel.
- **`SHADOW` (Secondary Heuristic Analysis for Defensive Online Warfare):** Költséghatékony, nyílt forráskódú NIDS, ami a tcpdump és Perl szkriptek kombinációja. Két részből áll: Szenzor állomás (tcpdump a rögzítéshez) és Elemző állomás (a rögzített fájlok elemzéséhez). Hátránya: Nincs benne valós idejű riasztás.
- **`STAT`:** Valós idejű visszaélés-felismerő (misuse) rendszer. Állapotátmenet-elemzést (state transition analysis) végez. Egy keretrendszert biztosít moduláris, "stateful" (állapottartó) behatolásjelzők fejlesztéséhez.
- **`NID`:** Nyílt forráskódú IDS, amit főleg kormányzati szervezeteknek terveztek. Támogatja az IP forgalom figyelését, beleértve az FDDI hálózatokat is. Három módban auditál: (a) ismert minták szignatúrával, (b) általános biztonsági paraméterek figyelése, (c) ismeretlen anomáliák.
- **`ISS RealSecure`:** Kereskedelmi IDS, eredetileg Windows NT-re kifejlesztve. Valós időben figyeli a forgalmat, egyezteti a szignatúrákkal, riaszt és képes ellenintézkedések automatikus végrehajtására. Azt feltételezi, hogy a legtöbb támadás belülről érkezik, ezért a tűzfalon átengedett forgalom mellett a belső aktivitásokat is monitorozza.

### 8.3.2 A kutatói közösség által fejlesztett IDS-ek

- **`ADAM`:** Adatbányászati rendszer, amely az asszociációs szabálybányászatot és az osztályozást (classification) ötvözi tcpdump adatok elemzéséhez.
- **`MINDS` (Minnesota IDS):** Adatbányászatra (Netflow v5) épül. Kiszűri az érdektelen forgalmat, majd a maradékon kiugró érték detekciós (outlier detection) algoritmust hajt végre, és anomália pontszámot ad a kapcsolatoknak.
- **`DNIDS`:** Egy kombinált K-Legközelebbi Szomszéd (CSI-kNN) algoritmust használ. Óriási előnye a megbízhatóság (Dependable): még támadás alatt is képes fenntartani a szolgáltatást. A forgalmat két mérőszámmal értékeli: a "furcsasággal" (strangeness) és az elszigeteltséggel (isolation).
- **`HIDE`:** Hierarchikus anomália rendszer, statisztikai modellezéssel és neurális hálózatokkal. Behatolásjelző Ágenseket (IDAs) használ a különböző szinteken.
- **`NSOM`:** Önszervező Térképek (Self-Organizing Maps) alapján méri fel a szokásos/elfogadható viselkedést, és a szabálytalant potenciális behatolásként (intrusive) jelöli meg, valós idejű Ethernet hálózatokon.
- **`FSAS`:** Folyam-alapú statisztikai aggregáció. 2 rétegű architektúra (jellemzőgenerátor és folyam-detektor). 22 specifikus jellemzőt ad vissza a DoS támadások hatékony észleléséhez.
- **`N@G` (Network at Guard):** Hibrid IDS (hálózati és hoszt szenzorokkal). Valós idejű védelmet nyújt a kliens számítógépeknek a Windows hoszt fájlok vagy a Messenger szolgáltatás kéretlen módosításai ellen. Dinamikusan tud hozzáférési listákat (ACL) küldeni Cisco routereknek a támadások blokkolására.
- **`FIRE` (Fuzzy Intrusion Recognition Engine):** Fuzzy logikát és hálózati forgalmi metrikákat használ egyszerű adatbányászattal a behatolások valószínűségének megállapítására.
- **`NFIDS`:** Neuro-fuzzy hierarchikus IDS. 3 szintből (Tier) áll, autonóm ágensekkel. Az I. szint monitoroz, a II. szint összegzi a LAN státuszát, a III. szint korrelálja az adatokat és riasztást küld.
- **`D-WARD`:** Adaptív, forrás-oldali (source-end) DDoS védelmi rendszer. Teljesen autonóm, minimális "járulékos kárt" (collateral damage) okoz a normál forgalomban, miközben sebészi pontossággal lassítja be a gépből kifelé induló DDoS támadást.
- **`LADS`:** Nagyskálás (Tier-1 ISP), automatizált DDoS detektáló rendszer Netflow adatokon.
- **`ANTID`:** Kifejezetten a hamisított csomagokkal (spoofed packets) operáló DDoS támadások szűrésére készült. Úgy védekezik, hogy minden IP csomagba beágyaz egy egyedi útvonal-ujjlenyomatot (path fingerprint), ami jelzi a bejárt útvonalat. Ha a szerver hirtelen ugrást észlel olyan csomagokból, amiknek az ujjlenyomata nem egyezik a várt útvonallal, tudja, hogy a csomag IP-címe hamisított.
- **`DCD`:** Változás-aggregációs fákat (Change Aggregation Trees - CAT) használ a folyam-szintű DDoS elárasztások detektálására, különböző internetszolgáltatók (ISP) routerein átívelve, egy közös Biztonságos Infrastruktúra Protokollal (SIP).

## 8.4 Összegzés és Megbeszélés (Discussion)

A hálózatbiztonsági kutatásokhoz elérhető eszközök és rendszerek részletes áttekintése alapján a könyv szerzői az alábbi kritikus megfigyeléseket (observations) tették a jelenlegi technológiai palettáról:

- **Hiányzik az integrált, "Minden-az-egyben" eszköz:** Nincs olyan szoftver, amely egyszerre támogatná a folyam-alapú (flow) ÉS a csomagszintű (packet) adatok rögzítését, előfeldolgozását, elemzését és vizualizációját. A meglévő eszközök (mint a Wireshark, Nfsen, Nfdump) vagy az egyiket, vagy a másikat tudják, de a kettőt együtt nem.
- **Protokoll-centrikus jellemzőkinyerés:** A meglévő feature-extrakciós eszközök (mint a tcptrace) túlságosan a TCP protokollra fókuszálnak, és csak alapvető jellemzőket tudnak kinyerni a nyers csomagokból. Bonyolultabb, támadás-specifikus jellemzők kinyeréséhez mindig egyedi C/C++ programokra vagy Perl szkriptekre van szükség.
- **Egyrétegű DDoS eszközök:** A jelenlegi DDoS támadóeszközök nem képesek egyszerre több hálózati rétegen (multi-layer) támadni. Csak egyetlen rétegre (pl. csak alkalmazási vagy csak hálózati rétegre) korlátozódnak.
- **Korlátozott testreszabhatóság a támadásoknál:** A legtöbb meglévő DDoS eszköz csak néhány beépített támadási forgatókönyvet (scenario) ismer, és nem lehet őket könnyen átprogramozni új, egyedi támadási sémák generálására.
- **Paraméter-érzékenység:** A legtöbb létező NIDS túlságosan függ a felhasználó által megadott bemeneti paraméterektől, és a rendszer teljesítménye extrém módon érzékeny ezek apró változásaira.
- **Magas téves riasztási arány (False alarms):** Szinte az összes anomália-alapú NIDS vagy csak "közel" valós időben (near real-time), vagy teljesen offline működik, és a legnagyobb rákfenéjük továbbra is a rengeteg téves riasztás.
- **Korreláció hiánya:** Továbbra is hiányzik egy olyan hatékony eszköz, amely képes lenne korrelációt vonni (összefüggéseket találni) a folyam-forgalom (flow traffic) és a csomag-forgalom (packet traffic) között.

### Kutatási Kihívások (Research Challenges) a Jövőre Nézve

A fenti megfigyelésekből kiindulva a következő megoldandó feladatok és kihívások állnak a hálózatvédelmi (network defenders) kutatók előtt:

1. **Integrált eszköz fejlesztése:** Egy olyan univerzális szoftver megalkotása, amely egyaránt képes a folyam- és csomagforgalom rögzítésére, szűrésére, jellemzőkiválasztására, elemzésére és vizualizációjára.
2. **Minden protokollra kiterjedő kinyerés:** Egy olyan villámgyors rögzítő és előfeldolgozó eszköz fejlesztése, amely a szállítási réteg (Transport layer) összes protokolljából képes minden típusú jellemzőt kinyerni.
3. **Többrétegű, GUI-alapú DDoS generátor (Kutatóknak):** Egy olyan felhasználóbarát, grafikus felületű DDoS forgalom-generátor építése, amely képes az összes lehetséges támadási forgatókönyv szimulálására egyszerre több hálózati rétegen (pl. alkalmazási és szállítási rétegeken egyaránt) a védelmi rendszerek teszteléséhez.
4. **Paraméter-független Anomália NIDS:** Egy olyan behatolásjelző rendszer kifejlesztése, amely minimális felhasználói paraméterezést igényel, képes a nulla-napi (ismeretlen) támadások valós idejű kezelésére, és mindezt minimális téves riasztás (false alarm) mellett teszi.
5. **Transzparens DDoS védelem a Célpontnál (Victim-end):** Egy olyan valós idejű detekciós rendszer megalkotása, amely képes megfogni az alacsony és magas intenzitású DDoS támadásokat közvetlenül az áldozat oldalán anélkül, hogy az a legitim felhasználók kapcsolatát vagy a normál szolgáltatás minőségét negatívan befolyásolná.

### Táblázat 8.9: Támadás-detektáló Rendszerek: Egy Összehasonlító Tanulmány

**Jelmagyarázat a táblázathoz:**

- **H/N/Hy (Típus):**
  - **H** = Hoszt (Host)
  - **N** = Hálózat (Network)
  - **Hy** = Hibrid (Hybrid)
- **M/A/B (Detekciós mód):**
  - **M** = Visszaélés (Misuse)
  - **A** = Anomália (Anomaly)
  - **B** = Mindkettő (Both)
- **R/N (Időbeliség):**
  - **R** = Valós idejű (Real time)
  - **N** = Nem valós idejű (Non real time / Offline)
- **C/D (Feldolgozás módja):**
  - **C** = Központosított (Centralised)
  - **D** = Elosztott (Distributed)

| Rendszer (System) | H/N/Hy | M/A/B | R/N | C/D | Forrás (Source) |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **Bro** | N | M | R | C | www.bro-ids.org |
| **Snort** | N | M | R | C | www.snort.org |
| **Nessus** | N | M | R | C | www.nessus.org/plugins |
| **SHADOW** | N | M | R | C | www.nswc.navy.mil |
| **STAT** | N | M | R | C | sourceforge.net/project/stat |
| **NID** | N | M | R | C | ciac.llnl.gov/ctsc/nid/ |
| **ISS RealSecure** | N | M | R | C | www.iss.net |
| **FIRE** | N | A | N | C | [98] |
| **ADAM** | N | A | R | C | [83] |
| **HIDE** | N | A | R | C | [425] |
| **NSOM** | N | A | R | C | [215] |
| **MINDS** | N | A | R | C | [110] |
| **NFIDS** | N | A | N | C | [258] |
| **N@G** | Hy | B | R | C | [358] |
| **D-WARD** | N | B | R | D | [253] |
| **ANTID** | N | A | R | C | [217] |
| **FSAS** | N | A | R | C | [346] |
| **LADS** | N | A | R | C | [329] |
| **DNIDS** | N | A | R | C | [208] |
| **DCD** | N | A | R | C | [69] |
