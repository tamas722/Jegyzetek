# Mi a különbség az átviteli teljesítmény és a késleltetés között?

A késleltetés (latency) és az átviteli teljesítmény (throughput) két mérőszám, amelyek egy számítógépes hálózat teljesítményét mérik. A késleltetés a hálózati kommunikáció késleltetése. Azt mutatja meg, mennyi idő alatt jut el az adat a hálózaton keresztül. A hosszabb késleltetéssel vagy laggal rendelkező hálózatoknak magas a késleltetésük, míg a gyors válaszidővel rendelkezőknek alacsonyabb. Ezzel szemben az átviteli teljesítmény arra az átlagos adatmennyiségre utal, amely egy adott idő alatt ténylegesen áthaladhat a hálózaton. Jelzi a célállomásukra sikeresen megérkező adatcsomagok számát és az adatcsomag-veszteséget.

*Olvasson a késleltetésről »*

## Miért fontos az átviteli teljesítmény és a késleltetés?

A hálózati sebességet úgy határozhatja meg, hogy megnézi, milyen gyorsan tud egy hálózat adatcsomagokat továbbítani a célállomásaikra. Ez a sebesség olyan hálózati teljesítménytényezők eredménye, mint a késleltetés és az átviteli teljesítmény.

A késleltetés határozza meg azt a késést, amelyet a felhasználó tapasztal, amikor adatot küld vagy fogad a hálózatról. Az átviteli teljesítmény határozza meg azon felhasználók számát, akik egyszerre hozzáférhetnek a hálózathoz.

Az alacsony átviteli teljesítményű és magas késleltetésű hálózat nehezen tud nagy adatmennyiséget küldeni és feldolgozni, ami torlódást és gyenge alkalmazásteljesítményt eredményez. Ezzel szemben a magas átviteli teljesítményű és alacsony késleltetésű hálózat reagálóképes és hatékony. A felhasználók javított teljesítményt és növelt elégedettséget tapasztalnak.

A nagy teljesítményű hálózatok közvetlenül befolyásolják a bevételtermelést és a működési hatékonyságot. Emellett bizonyos felhasználási esetek – mint a valós idejű streamelés, az Internet of Things (IoT) adatelemzés és a nagy teljesítményű számítástechnika – bizonyos hálózati teljesítményküszöböket igényelnek az optimális működéshez.

## Fő különbségek: hálózati késleltetés vs. átviteli teljesítmény

Bár a késleltetés és az átviteli teljesítmény egyaránt hozzájárul egy megbízható és gyors hálózathoz, nem ugyanazok. Ezek a hálózati mérőszámok különböző statisztikákra összpontosítanak, és különböznek egymástól.

### Hogyan mérjük?

A hálózati késleltetést a ping idő mérésével mérheti. Ez egy olyan folyamat, amely során egy kis adatcsomagot küld, és visszaigazolást kap annak megérkezéséről.

A legtöbb operációs rendszer támogat egy ping parancsot, amely ezt az Ön eszközéről végzi. A round-trip-time (RTT) ezredmásodpercben jelenik meg, és képet ad arról, mennyi idő alatt továbbítja a hálózat az adatokat.

Az átviteli teljesítményt hálózati tesztelőeszközökkel vagy manuálisan mérheti. Ha manuálisan szeretné tesztelni az átviteli teljesítményt, küldene egy fájlt, és elosztaná a fájlméretet a megérkezéséhez szükséges idővel. Azonban a késleltetés és a sávszélesség befolyásolja az átviteli teljesítményt. Emiatt sokan hálózati tesztelőeszközöket használnak, mivel az eszközök az átviteli teljesítményt más tényezőkkel, például a sávszélességgel és a késleltetéssel együtt jelentik.

*Olvasson az RTT-ről a hálózatkezelésben »*

### Mértékegység

A késleltetést ezredmásodpercben méri. Ha alacsony az ezredmásodpercek száma, a hálózat csak kis késleltetést tapasztal. Minél magasabb az ezredmásodpercek száma, annál lassabb a hálózat teljesítménye.

Eredetileg a hálózati átviteli teljesítményt bit per másodpercben (bps) mérték. De ahogy az adatátviteli technológiák fejlődtek, mára sokkal magasabb értékeket lehet elérni. Emiatt az átviteli teljesítményt kilobyte per másodpercben (KBps), megabyte per másodpercben (MBps) és akár gigabyte per másodpercben (GBps) is mérheti. Egy byte nyolc bitnek felel meg.

### Befolyásoló tényezők: késleltetés vs. átviteli teljesítmény

Különböző tényezők befolyásolhatják a késleltetés és az átviteli teljesítmény mérőszámait.

#### Késleltetés

A késleltetésnek számos tényezője járul hozzá ahhoz, hogy magas vagy alacsony legyen.

**Elhelyezkedés**

Az egyik legfontosabb tényező az adat származási helye és a szánt célállomás. Ha a szerverek egy másik földrajzi régióban vannak, mint az eszköze, az adatnak tovább kell utaznia, ami növeli a késleltetést. Ezt a tényezőt terjedésnek (propagation) nevezik.

**Hálózati torlódás**

A hálózati torlódás akkor következik be, amikor nagy mennyiségű adatot továbbítanak egy hálózaton. A megnövekedett forgalom a hálózaton azt okozza, hogy a csomagok hosszabb útvonalakat vesznek igénybe a célállomásuk eléréséhez.

**Protokoll hatékonyság**

Egyes hálózatok további protokollokat igényelnek a biztonság érdekében. Az extra kézfogási lépések késleltetést okoznak.

**Hálózati infrastruktúra**

A hálózati eszközök túlterheltté válhatnak, ami elveszett csomagokat eredményez. Ahogy a csomagok késlekednek vagy elvesznek, az eszközök újraküldik őket. Ez további késleltetést ad hozzá.

#### Átviteli teljesítmény

Az átviteli teljesítmény sebességét közvetlenül befolyásolják más tényezők.

**Sávszélesség**

Ha a hálózati kapacitás elérte az átviteli médium maximális sávszélességét, az átviteli teljesítmény soha nem tudja túllépni ezt a határt.

**Feldolgozási teljesítmény**

Bizonyos hálózati eszközök speciális hardver- vagy szoftveroptimalizálással rendelkeznek, amelyek javítják a feldolgozási teljesítményüket. Néhány példa erre a dedikált alkalmazásspecifikus integrált áramkörök vagy a szoftveralapú csomagfeldolgozó motorok.

Ezek az optimalizálások lehetővé teszik az eszköz számára, hogy nagyobb mennyiségű forgalmat és összetettebb csomagfeldolgozási feladatokat kezeljen, ami magasabb átviteli teljesítményhez vezet.

**Csomagvesztés**

A csomagvesztés több okból is előfordulhat, beleértve a hálózati torlódást, a hibás hardvert vagy a hibásan konfigurált hálózati eszközöket. Amikor a csomagok elvesznek, újra kell küldeni őket. Ez késedelmeket eredményez, és csökkenti a hálózat általános átviteli teljesítményét.

**Hálózati topológia**

A hálózati topológia a hálózati eszközök számára, a hálózati kapcsolatok sávszélességére és a hálózati útvonalon lévő eszközök közötti távolságra utal.

Egy jól megtervezett hálózati topológia több útvonalat biztosít az adatátvitelhez, csökkenti a forgalmi szűk keresztmetszeteket és növeli az átviteli teljesítményt. A több eszközzel vagy hosszabb távolságokkal rendelkező hálózatok összetett hálózati topológiákat igényelnek a magas átviteli teljesítmény eléréséhez.

## A sávszélesség, a késleltetés és az átviteli teljesítmény kapcsolata

A késleltetés és az átviteli teljesítmény együtt dolgoznak a magas hálózati kapcsolódás és teljesítmény biztosítása érdekében. Mivel mindkettő hatással van az adatcsomagok továbbítására, hatással vannak egymásra is.

Ha egy hálózati kapcsolatnak magas a késleltetése, alacsonyabb átviteli teljesítménye lehet, mivel az adatok tovább tartanak a továbbításhoz és megérkezéshez. Az alacsony átviteli teljesítmény azt is a látszatot keltheti, hogy a hálózatnak magas a késleltetése, mivel a nagy mennyiségű adat megérkezése tovább tart.

Mivel szorosan kapcsolódnak egymáshoz, mind a késleltetést, mind az átviteli teljesítményt figyelnie kell a magas hálózati teljesítmény eléréséhez.

### Sávszélesség és hálózati átviteli teljesítmény

A sávszélesség azt a teljes adatmennyiséget jelöli, amelyet egy hálózaton keresztül továbbíthat. A teljes sávszélessége arra az elméleti maximális adatmennyiségre utal, amelyet egy hálózaton keresztül továbbíthat. Megabyte per másodpercben (MBps) méri. A sávszélességre úgy gondolhat, mint a hálózat elméleti maximális átviteli teljesítményére.

A sávszélesség az, hogy mennyi adatot tud továbbítani, míg az átviteli teljesítmény a valós hálózati korlátok alapján bármely adott pillanatban ténylegesen továbbított adat mennyisége. A magas sávszélesség nem garantál sebességet vagy jó hálózati teljesítményt, de a magasabb sávszélesség magasabb átviteli teljesítményhez vezet.

## Hogyan javíthatjuk a késleltetést és az átviteli teljesítményt?

A késleltetés javításához lerövidítheti a forrás és a célállomás közötti terjedést. Az átviteli teljesítményt a teljes hálózati sávszélesség növelésével javíthatja.

Az alábbiakban néhány javaslatot adunk a késleltetés és az átviteli teljesítmény együttes javítására.

### Gyorsítótárazás

A gyorsítótárazás a hálózatkezelésben azt a folyamatot jelenti, amely során a gyakran hozzáfért adatokat földrajzilag közelebb tárolják a felhasználóhoz. Például tárolhat adatokat proxy szervereken vagy tartalomkézbesítő hálózatokon (CDN).

A hálózata sokkal gyorsabban tud adatokat szolgáltatni a gyorsítótárazott helyről, mintha az eredeti forrásból kellene lekérnie. És a felhasználó sokkal gyorsabban kapja meg az adatokat, javítva a késleltetést. Emellett, mivel az adatok a gyorsítótárból származnak, csökkenti az eredeti forrás terhelését. Ez lehetővé teszi számára, hogy egyszerre több kérést kezeljen, javítva az átviteli teljesítményt.

### Szállítási protokollok

Az adott alkalmazásokhoz használt szállítási protokoll optimalizálásával javíthatja a hálózati teljesítményt.

Például a TCP és az UDP két gyakori hálózati protokoll. A TCP kapcsolatot létesít, és ellenőrzi, hogy hiba nélkül kapja-e meg az adatokat. A csomagvesztés csökkentésének célja miatt a TCP magasabb késleltetéssel és magasabb átviteli teljesítménnyel rendelkezik. Az UDP nem ellenőrzi a csomagvesztést vagy a hibákat, helyette több duplikált csomagot továbbít. Így minimális késleltetést, de magasabb átviteli teljesítményt nyújt.

Attól függően, hogy melyik alkalmazást használja, a TCP vagy az UDP lehet a jobb választás. Például a TCP hasznos az adatátvitelhez, míg az UDP a videóstreameléshez és a játékokhoz.

### Szolgáltatásminőség

Használhat szolgáltatásminőség (QoS) stratégiát a hálózati teljesítmény kezelésére és optimalizálására. A QoS lehetővé teszi a hálózati forgalom meghatározott kategóriákba sorolását. Minden kategóriához hozzárendelhet egy prioritási szintet.

A QoS-konfigurációk priorizálják a késleltetésre érzékeny alkalmazásokat. Egyes alkalmazások és felhasználók alacsonyabb késleltetést tapasztalnak, mint mások. A QoS-konfigurációk a típus szerinti adatpriorizálást is lehetővé teszik, csökkentve a csomagvesztést és növelve az átviteli teljesítményt bizonyos felhasználók számára.

## A különbségek összefoglalása: átviteli teljesítmény vs. késleltetés

| | Átviteli teljesítmény | Késleltetés |
|---|---|---|
| **Mit mér?** | Az átviteli teljesítmény az adott időszakban a hálózaton áthaladó adat mennyiségét méri. Az átviteli teljesítmény befolyásolja, hogy mennyi adatot tud továbbítani egy adott időszakban. | A késleltetés az adatküldés időbeli késését méri. A magasabb késleltetés hálózati késést okoz. |
| **Hogyan mérjük?** | Manuálisan számítsa ki az átviteli teljesítményt egy fájl küldésével vagy hálózati tesztelőeszközökkel. | Számítsa ki a késleltetést ping idők használatával. |
| **Mértékegység** | Megabyte per másodperc (MBps). | Ezredmásodperc (ms). |
| **Befolyásoló tényezők** | Sávszélesség, hálózati feldolgozási teljesítmény, csomagvesztés és hálózati topológia. | Földrajzi távolságok, hálózati torlódás, szállítási protokoll és hálózati infrastruktúra. |
