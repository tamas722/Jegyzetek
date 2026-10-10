# Mi az a router?

A **router** egy olyan eszköz, amely két vagy több csomagkapcsolt hálózatot vagy alhálózatot köt össze. Két elsődleges funkciót lát el: kezeli a forgalmat e hálózatok között azáltal, hogy az adatcsomagokat a szánt IP-címükre továbbítja, és lehetővé teszi, hogy több eszköz ugyanazt az internetkapcsolatot használja.

Többféle router létezik, de a legtöbb router LAN-ok (helyi hálózatok) és WAN-ok (nagy kiterjedésű hálózatok) között továbbít adatokat. A LAN egy adott földrajzi területre korlátozódó, összekapcsolt eszközök csoportja. Egy LAN általában egyetlen routert igényel.

A WAN ezzel szemben egy nagy hálózat, amely egy hatalmas földrajzi területen terjed ki. A nagy szervezetek és vállalatok, amelyek az ország több helyszínén működnek, például külön LAN-okat igényelnek minden helyszínhez, amelyek aztán a többi LAN-hoz kapcsolódnak WAN-t alkotva. Mivel a WAN nagy területen oszlik el, gyakran több routert és switch-et* igényel.

*\*A hálózati switch az ugyanazon hálózatban lévő eszközcsoportok között továbbít adatcsomagokat, míg a router a különböző hálózatok között továbbít adatokat.*

**Ingyenes eszközök**
*Védd meg az internetedet ingyen az 1.1.1.1-gyel*
[Töltsd le az alkalmazást]

## Hogyan működik a router?

Gondolj a routerre úgy, mint egy légiirányítóra, az adatcsomagokra pedig úgy, mint a különböző repülőterekre (vagy hálózatokra) tartó repülőgépekre. Ahogy minden repülőgépnek egyedi célállomása van, és egyedi útvonalat követ, minden csomagot a lehető leghatékonyabban kell a célállomására irányítani. Ahogy a légiirányító biztosítja, hogy a repülőgépek eljussanak a célállomásaikra anélkül, hogy eltévednének vagy súlyos zavart szenvednének az útközben, a router segít az adatcsomagokat a cél IP-címükre irányítani.

A csomagok hatékony irányításához a router egy belső **routing táblázatot** használ – a különböző hálózati célállomásokhoz vezető útvonalak listáját. A router elolvassa a csomag fejlécét, hogy meghatározza, hová tart, majd megnézi a routing táblázatot, hogy kitalálja a leghatékonyabb útvonalat ahhoz a célállomáshoz. Ezután továbbítja a csomagot az útvonal következő hálózatához.

Ha többet szeretnél megtudni az IP-útvonalválasztásról és a folyamat során használt protokollokról, olvasd el: Mi az az útvonalválasztás?

**Iratkozz fel**
*Biztonság és sebesség bármely Cloudflare csomaggal*
[Kezdje ingyen]

## Mi a különbség a router és a modem között?

Bár egyes internetszolgáltatók (ISP-k) egyetlen eszközben kombinálhatják a routert és a modemet, ezek nem ugyanazok. Mindegyik más, de egyformán fontos szerepet játszik a hálózatok egymáshoz és az internethez való csatlakoztatásában.

A **router** hálózatokat hoz létre és kezeli az adatok áramlását ezeken a hálózatokon belül és közöttük, míg a **modem** ezeket a hálózatokat az internethez csatlakoztatja. A modemek úgy hoznak létre kapcsolatot az internettel, hogy az ISP-től származó jeleket olyan digitális jellé alakítják, amelyet bármely csatlakoztatott eszköz értelmezni tud. Egyetlen eszköz csatlakozhat egy modemhez az internethez való kapcsolódáshoz; alternatívaként egy router segíthet ezt a jelet több eszközhöz elosztani egy létrehozott hálózaton belül, lehetővé téve mindegyikük számára, hogy egyszerre csatlakozzon az internethez.

Gondolj rá így: Ha Bobnak van routere, de nincs modeme, létre tud hozni egy LAN-t, és adatokat tud küldeni a hálózat eszközei között. Azonban nem tudja csatlakoztatni ezt a hálózatot az internethez. Alice-nak ezzel szemben van modeme, de nincs routere. Egyetlen eszközt tud csatlakoztatni az internethez (például a munkalaptopját), de nem tudja elosztani ezt az internetkapcsolatot több eszközre (mondjuk a laptopjára és az okostelefonjára). Carolnak eközben van routere és modeme is. Mindkét eszközt használva létre tud hozni egy LAN-t az asztali számítógépével, táblagépével és okostelefonjával, és egyszerre csatlakoztathatja mindet az internethez.

## Milyen típusú routerek léteznek?

Ahhoz, hogy egy LAN-t az internethez csatlakoztasson, a routernek először kommunikálnia kell egy modemmel. Erre két elsődleges mód van:

- **Vezeték nélküli router:** A vezeték nélküli router Ethernet kábellel csatlakozik a modemhez. Az adatokat úgy osztja el, hogy a csomagokat bináris kódból rádiójelekké alakítja, majd antennákkal vezeték nélkül sugározza azokat. A vezeték nélküli routerek nem hoznak létre LAN-okat; ehelyett WLAN-okat (vezeték nélküli helyi hálózatokat) hoznak létre, amelyek vezeték nélküli kommunikációval kapcsolnak össze több eszközt.
- **Vezetékes router:** A vezeték nélküli routerhez hasonlóan a vezetékes router is Ethernet kábellel csatlakozik a modemhez. Ezután külön kábelekkel csatlakozik a hálózaton belüli egy vagy több eszközhöz, LAN-t hoz létre, és összekapcsolja a hálózaton belüli eszközöket az internettel.

A kis LAN-okhoz használt vezeték nélküli és vezetékes routerek mellett számos speciális típusú router létezik, amelyek konkrét funkciókat szolgálnak:

- **Core router:** Ellentétben az otthoni vagy kisvállalkozási LAN-on belül használt routerekkel, a core routert nagyvállalatok és olyan vállalkozások használják, amelyek nagy mennyiségű adatcsomagot továbbítanak a hálózatukon belül. A core routerek a hálózat „magjában" működnek, és nem kommunikálnak külső hálózatokkal.
- **Edge router:** Míg a core router kizárólag egy nagy léptékű hálózaton belüli adatforgalmat kezel, az edge router mind a core routerekkel, mind a külső hálózatokkal kommunikál. Az edge routerek a hálózat „peremén" helyezkednek el, és a BGP (Border Gateway Protocol) segítségével küldenek és fogadnak adatokat más LAN-októl és WAN-októl.
- **Virtuális router:** A virtuális router egy szoftveralkalmazás, amely ugyanazt a funkciót látja el, mint egy szabványos hardveres router. Használhatja a Virtual Router Redundancy Protocol (VRRP) protokollt elsődleges és tartalék virtuális routerek létrehozásához, ha az egyik meghibásodik.

## Mi az az SSID?

Az **SSID** a „service set identifier" rövidítése, és ez a technikai kifejezés annak a hálózatnak a nevére, amelyet a WLAN routerek sugároznak. Az SSID-k lehetővé teszik a felhasználók számára, hogy megtalálják és csatlakozzanak a router által sugárzott vezeték nélküli hálózathoz (a megfelelően biztosított routernek jelszó megadását is kérnie kell). A WiFi hálózatokhoz használt fogyasztói routerek általában a gyári alapértelmezett SSID-t az oldalukra vagy az aljukra nyomtatva tartalmazzák.

## Milyen biztonsági kihívások kapcsolódnak a routerekhez?

- **Sebezhetőségek kihasználása:** Minden hardveralapú routerhez automatikusan telepített szoftver, úgynevezett **firmware** tartozik, amely segít a routernek a funkciói ellátásában. Mint minden más szoftver, a router firmware gyakran tartalmaz sebezhetőségeket, amelyeket a kibertámadók kihasználhatnak (egy példa), és a routergyártók rendszeresen adnak ki frissítéseket e sebezhetőségek javítására. Ezért a router firmware-t rendszeresen frissíteni kell. A nem javított routereket a támadók kompromittálhatják, lehetővé téve számukra a forgalom monitorozását vagy a router botnet részeként való használatát.

- **DDoS támadások:** A kis- és nagy szervezetek gyakran célpontjai a hálózati infrastruktúrájukra irányuló elosztott szolgáltatásmegtagadási (DDoS) támadásoknak. A nem mérsékelt hálózati rétegbeli DDoS támadások túlterhelhetik a routereket, vagy összeomlaszthatják azokat, ami hálózati leállást eredményez. A Cloudflare Magic Transit az egyik megoldás a routerek és hálózatok ilyen típusú DDoS támadásoktól való védelmére.

- **Adminisztratív hitelesítő adatok:** Minden routerhez tartozik egy adminisztratív hitelesítő adatkészlet az adminisztratív funkciók ellátásához. Ezek a hitelesítő adatok alapértelmezett értékekre vannak beállítva, például „admin" a felhasználónév és „admin" a jelszó. A felhasználónevet és a jelszót a lehető leghamarabb biztonságosabbra kell visszaállítani: a támadók tisztában vannak ezen hitelesítő adatok gyakori alapértelmezett értékeivel, és felhasználhatják azokat a router távoli irányítására, ha azokat nem állítják vissza.
