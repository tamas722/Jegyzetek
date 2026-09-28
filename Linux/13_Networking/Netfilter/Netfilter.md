# Csomagszűrés Linuxon – iptables, nftables és firewalld

**Közzétéve:** 2021. július 9. – Szerző: Adrian Wyssmann – 8 perc olvasás

## Mi az IP-cím és a csomagszűrés?

Az IP-szűrő (IP Filter) főként a TCP/IP referenciamodell 2. rétegében működik, de a 3. rétegben is képes dolgozni. Általában az IP-fejlécek alapján szűri a csomagokat (forrás- és célcím, TOS/DSCP/ECN, TTL, protokoll stb.). Az iptables-tutorial tartalmaz néhány példát arra, hogyan néznek ki ezek a fejlécek. Ha többet szeretnél megtudni, érdemes megnézni a vonatkozó RFC-ket is, amelyek részletesebben kifejtik ezt.

## Netfilter, iptables és nftables

A netfilter projekt egy nyílt forráskódú projekt a Linux csomagszűrésére:

> A netfilter projekt lehetővé teszi a csomagszűrést, a hálózati cím- [és port-] fordítását (NA[P]T), a csomagok naplózását, a felhasználói térben történő csomagsorba állítást és egyéb csomagmanipulációt.

Ahhoz, hogy a Linux kihasználhassa ezt, a kernelt bizonyos konfigurációkkal kell lefordítani. Egy részletes példa az iptables-hez megtalálható a dokumentációjukban. Azt mondanám, hogy sok disztribúció már eleve beépítve szállítja ezt a kerneljében.

A projekt két szoftverrészből áll:

- **iptables** egy IP-szűrő, amely lehetővé teszi szabálykészletek (rulesets) meghatározását. Az IP-táblán belüli minden szabály számos osztályozóból (iptables matches) és egy kapcsolódó műveletből (iptables target) áll.
  - Képes a csomagokat más, mélyebben a csomagban található fejlécek (TCP, UDP stb.) és sekélyebb (MAC forráscím) alapján is szűrni.
  - **Kapcsolatkövetés (Connection tracking):** Nyomon követi a csomagokat, és megállapítja, hogy ugyanahhoz a folyamhoz tartoznak-e (sorszámok, portszámok stb. alapján).
  - Képes cél- és forrás-hálózati címfordításra (általában DNAT és SNAT).
  - Képes a csomagok illesztésére.
  - Alapértelmezésben nem tudja összekapcsolni a különböző csomagok adatait, így soha nem lehet teljesen biztos abban, hogy minden esetben a teljes adatot látja.

- **nftables** az iptables utódja, teljesen új kódbázissal és számos fejlesztéssel. Más szintaxist és más parancssori eszközt használ, mint az iptables. A Linux kernel >= 3.13 verzióiban érhető el.
  - Kompatibilitási réteg, amely lehetővé teszi az iptables parancsok futtatását az új nftables kernel keretrendszer felett.
  - Gyorsabb csomagosztályozás a fejlettebb általános halmaz- és térkép-infrastruktúra révén.
  - Egyszerűsített kettős verem (dual stack) IPv4/IPv6 adminisztráció az új `inet` család révén, amely lehetővé teszi olyan alap láncok regisztrálását, amelyek egyszerre látják az IPv4 és IPv6 forgalmat.
  - Jobb dinamikus szabálykészlet-frissítési támogatás.
  - Netlink API biztosítása harmadik féltől származó alkalmazások számára, akárcsak a többi Linux Networking és Netfilter alrendszer.
  - A cím-szintaxison kívüli következetlenségek megszüntetése és szebb, kompaktabb szintaxis biztosítása.

Az OSI-modellt tekintve egy csomag egy vezetéken (vagy rádióhullámon) keresztül lép be a tűzfalba. Ezután feljut a kernel megfelelő eszközmeghajtójához. Ezután a csomag egy sor lépésen megy keresztül a kernelben, mielőtt vagy a megfelelő alkalmazáshoz (helyben) küldik, vagy továbbítják egy másik gazdagéphez – vagy bármi más történik vele. A netfilternek horgai (hooks) vannak – jól meghatározott pontok a csomag protokollveremen való áthaladásában –, ahol a szoftver „behorgonyoz”.

Lásd még: Táblák és láncok bejárása, vagy néhány más diagram, amely részletesebben bemutatja: linux-ip-diagrams, Netfilter-IPTables-Diagrams.

## iptables

Az iptables a fent említett netfilter infrastruktúrát használja. Nézzük meg a kifejezéseket és kifejezéseket, hogy jobban megértsük, hogyan működik:

- **Drop/Deny** – A csomag törlődik, és nem küld vissza választ, sem a fogadó nem értesül.
- **Reject** – Ugyanaz, mint a drop, de a küldő értesül a eldobott csomagról. Nincs olyan funkció, amely a fogadó gazdagépet is értesítené a visszautasított csomagról.
- **State** – Egy csomag adott állapota egy teljes csomagfolyamhoz viszonyítva. Az állapotokat a kapcsolatkövető rendszer ismeri fel, amely nyomon követi az összes munkamenetet. Az állapotokról részleteket az állapotgépben találsz.
- **Chain (Lánc)** – Egy lánc olyan szabálykészletet tartalmaz, amelyet a láncon áthaladó csomagokra alkalmaznak. Minden láncnak meghatározott célja van (pl. melyik táblához kapcsolódik, ami meghatározza, mit tud tenni ez a lánc), valamint meghatározott alkalmazási területe (pl. csak továbbított csomagok, vagy csak az erre a gazdagépre irányuló csomagok). Az `rc.firewall` fájlt érdemes megnézni a láncok jobb megértéséhez.
  - Minden táblának vannak beépített láncai, pl. a filter táblának: PREROUTING, INPUT, OUTPUT, FORWARD és POSTROUTING.
    - `prerouting`: minden újonnan érkező IP-forgalom
    - `input`: a helyi gazdagépre címzett forgalom
    - `output`: a helyi gazdagépről származó forgalom
    - `forward`: azok a csomagok, amelyeket a gazdagép egyszerűen átad egyik hálózatból a másikba
    - `postrouting`: minden IP-forgalom, amely elhagyja a tűzfalat
- **Table (Tábla)** – 4 tábla van, mindegyik meghatározott céllal:
  - `raw`: Csak arra használják, hogy megjelöljenek csomagokat, amelyeket nem szabad a kapcsolatkövető rendszernek kezelnie.
  - `nat`: Csak NAT (hálózati címfordítás) céljára szabad használni különböző csomagokon, azaz a csomag forrás- vagy célmezőjének fordítására.
  - `mangle`: Csomagok manipulálására használják, pl. TOS (Type Of Service), TTL, MARK módosítására.
  - `filter`: Csomagok szűrésére használják.
- **Match (Illesztés)** – Ennek a szónak két különböző jelentése lehet:
  - Egyetlen illesztés, amely megmondja egy szabálynak, hogy ennek a fejlécnek ezt és ezt az információt kell tartalmaznia (pl. a forráscímnek egy adott hálózati tartománynak vagy gazdagépnek kell lennie).
  - Ha a csomag illeszkedik a teljes szabályra, a jump vagy target utasítások végrehajtódnak (pl. a csomag eldobásra kerül).
- **Target (Cél)** – Minden szabályhoz beállított cél egy szabálykészletben. Ha a szabály teljesen illeszkedett, a target specifikáció megmondja, mit kell tenni a csomaggal. Lásd: Iptables targets and jumps a részletekért.
- **Rule (Szabály)** – A szabály egy illesztés vagy több illesztés együttese egyetlen céllal.
- **Ruleset (Szabálykészlet)** – A szabálykészlet a teljes IP-szűrő implementációba helyezett szabályok teljes halmaza.
- **Jump (Ugrás)** – Az ugrás utasítás megmondja az iptables-nek, hogy a csomagot egy másik lánchoz küldje feldolgozásra. Lásd: Iptables targets and jumps a részletekért.
- **Connection tracking (Kapcsolatkövetés)** – Egy kapcsolatkövetést megvalósító tűzfal képes nyomon követni a kapcsolatokat/folyamokat.
- **Accept (Elfogadás)** – Egy csomag elfogadása és átengedése a tűzfalszabályokon.
- **Policy (Házirend)** – Kétféle házirendről beszélünk legtöbbször tűzfal implementálásakor:
  - **Lánc házirendek:** Megmondják a tűzfal implementációnak az alapértelmezett viselkedést egy csomaggal szemben, ha nem volt illeszkedő szabály.
  - **Biztonsági házirend:** Dokumentum, amelyet alaposan át kell gondolni és tanulmányozni kell, mielőtt elkezdenénk ténylegesen implementálni a tűzfalat.

## nftables

Az nftables ugyanazt a Netfilter infrastruktúrát használja, mint a régi iptables, de új csomagosztályozási rendszerrel. Íme a legjelentősebb változások az iptables-hez képest:

- Az nftables új, kompakt szintaxist használ, amelyet a tcpdump ihletett.
- A táblák és láncok teljesen konfigurálhatók, nincsenek előre meghatározott táblák vagy láncok.
- Egyetlen nftables szabály több műveletet is végrehajthat.
- Új általános halmaz-infrastruktúra, amely fejlett konfigurációkat tesz lehetővé, mint például térképek, verdikt-térképek és intervallumok a teljesítményorientált csomagosztályozás eléréséhez.
- Jobb támogatás a dinamikus szabálykészlet-frissítésekhez.
- Új protokollok támogatása kernelfrissítés nélkül.
- Konkatenációk támogatása.
- Az nftables lehetővé teszi a szűrést különböző szinteken, amelyeket családokba (families) absztrahálnak – az iptables-ben minden szinthez külön eszköz van:
  - `ip`: Az ebbe a családba tartozó táblák az IPv4 forgalmat/csomagokat látják.
  - `ip6`: Az ebbe a családba tartozó táblák az IPv6 forgalmat/csomagokat látják.
  - `inet`: Az ebbe a családba tartozó táblák mind az IPv4, mind az IPv6 forgalmat/csomagokat látják, egyszerűsítve a kettős verem támogatását.
  - `arp`: Az ebbe a családba tartozó táblák az [ARP](https://en.wikipedia.org/wiki/Address_Resolution_Protocol) szintű (azaz L2) forgalmat látják, mielőtt a kernel bármilyen L3 kezelést végezne.
  - `bridge`: Az ebbe a családba tartozó táblák a hidakon (azaz kapcsolókon) áthaladó forgalmat/csomagokat látják.
  - `netdev`: A netdev család abban különbözik a többitől, hogy egyetlen hálózati interfészhez csatolt alap láncok létrehozására használják, azaz az adott interfészen minden hálózati forgalomra.

## Firewalld

Számos felhasználói felület érhető el az iptables vagy nftables kezelésére. Az egyik legkiemelkedőbb, amely előre telepítve és alapértelmezett tűzfal-kezelő eszközként érkezik, a firewalld. Ez az alapértelmezett a következőknél:

- RHEL 7 és újabb
- CentOS 7 és újabb
- Fedora 18 és újabb
- SUSE 15 és újabb
- OpenSUSE 15 és újabb

A firewalld dinamikusan kezelt tűzfalat biztosít, támogatva a hálózati/tűzfal zónákat, amelyek meghatározzák a bizalmi szintet:

- Teljes D-Bus API
- IPv4, IPv6, bridge és ipset támogatás
- IPv4 és IPv6 NAT támogatás
- Tűzfal zónák
- Előre meghatározott zónák, szolgáltatások és icmptípusok listája
- Egyszerű szolgáltatás-, port-, protokoll-, forrásport-, maszkolás-, porttovábbítás-, icmp-szűrés-, gazdag szabály-, interfész- és forráscím-kezelés a zónákban
- Egyszerű szolgáltatásdefiníció portokkal, protokollokkal, forrásportokkal, modulokkal (netfilter helpers) és célcím-kezeléssel
- Rich Language a rugalmasabb és összetettebb szabályokhoz a zónákban
- Időzített tűzfalszabályok a zónákban
- Az elutasított csomagok egyszerű naplózása
- Közvetlen interfész
- Lockdown: A tűzfalat módosítható alkalmazások fehérlistázása
- Linux kernel modulok automatikus betöltése

A D-Bus interfésznek köszönhetően a változtatások azonnal végrehajthatók a futásidejű környezetben, a szolgáltatás vagy démon újraindítása nélkül. Van egy magrétege, amely a konfigurációt és a háttérrendszereket kezeli, mint az iptables, ip6tables, ebtables, ipset és a modulbetöltő. A mag tetején a D-Bus réteg található, amely elsődleges módja a tűzfal konfiguráció módosításának és létrehozásának a biztosított eszközökkel.

## Forrás
-[https://wyssmann.com/blog/2021/07/packet-filtering-in-linux-iptables-nftables-and-firewalld/]
-[https://netfilter.org/]
