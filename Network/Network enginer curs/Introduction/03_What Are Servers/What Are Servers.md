# Mi az a számítógépes szerver?

A „számítógépes szerver" rövidítéseként a **szerver** egy olyan hardver- vagy szoftverrendszer, amely adatokat, erőforrásokat, szolgáltatásokat, alkalmazásokat és egyéb kimeneteket szolgáltat a kliens eszközök számára belső hálózatokon vagy az interneten keresztül. A szervezetek és magánszemélyek szervereiket többféleképpen üzemeltethetik:

- Házon belül/helyszíni telepítéssel
- Nagy, megosztott adatközpontokban, amelyek szerverekkel teli helyiségeket tartalmaznak
- A felhőben

*(Olvasson az adatközpont-optimalizálásról és az adatközpont-biztonságról.)*

## Hogyan működnek a szerverek?

Egyszerűen fogalmazva, a szerverek digitális kéréseket (bemeneteket) dolgoznak fel, és válaszokat (kimeneteket) generálnak. Amikor a szerver fogad egy kérést, feldolgozza azt, és ha alkalmazható, visszaadja a kért kimenetet.

A kérések eredhetnek:

- Kliens eszközöktől („kliensek"), beleértve a laptopokat, okostelefonokat, táblagépeket, az Internet of Things (IoT) eszközöket, valamint az AI-vezérelt alkalmazásokat és utasításokat.
- Belsőleg a termelési feladatütemezőktől, automatizált feldolgozási feladatoktól vagy a szerveren közvetlenül futó egyéb folyamatoktól.

A számítógépes szerverek kliens eszközként is működhetnek, kéréseket küldve más szervereknek. Másodpercenként számos tranzakciós kérést képesek feldolgozni, és kliensként viselkedhetnek, amikor adatot kérnek egy másik szolgáltatástól. (Például egy webszerver, amely adatot kér egy DBMS szervertől.)

## A szerver összetevői

A szervert üzemeltető hardver egy összetett rendszer, amely számos kritikus komponensből áll az adattároláshoz, feldolgozáshoz, kommunikációhoz, biztonsági mentésekhez és egyéb funkciókhoz, többek között:

- **Központi feldolgozóegységek (CPU-k):** Egy vagy több feldolgozóegység (chip), amelyek tranzakciókat és szoftverutasításokat hajtanak végre, számításokat végeznek, kezelik az alaplap komponensei közötti adatáramlást és még sok mást.
- **Grafikus feldolgozóegységek (GPU-k):** A GPU-k gyorsan feldolgozzák az összetett logikai, vizuális, matematikai és grafikus megjelenítést igénylő kéréseket. Játékokhoz, videóhoz, tervezéshez és legfőképpen az AI és a nagy nyelvi modellek (LLM-ek) számítási feladatainak feldolgozásához használják őket.
- **Memória:** Az illékony RAM (Random Access Memory) a szerver feldolgozása során ideiglenes tárolásra szolgál.
- **Tárolás:** A tárolás, más néven „másodlagos" vagy állandó tárolás, olyan nem illékony adathordozókat foglal magában, mint az SSD-k, HDD-k és NVMe meghajtók, ahol az adatok kikapcsolás után is megmaradnak. Ide tartoznak az operációs rendszereket, adatokat és alkalmazásokat tároló eszközök, például merevlemezek, felhőtárhely, lemeztömbök, hordozható tárolók és egyéb adathordozók.
- **Alaplap:** Egy áramköri lap, amely elosztja az elektromosságot, és összekapcsolja, valamint lehetővé teszi a kommunikációt az összes többi szerverkomponenssel.
- **Hálózati kapcsolat:** Minden szerver rendelkezik egy vagy több hálózati interfészkártyával (NIC) és belső hálózati port(ok)kal. A NIC-k lehetővé teszik a helyi és internethez csatlakoztatott számítógépek közötti kommunikációt. A modern szerverek gyakran SFP+ vagy QSFP portokat használnak optikai szálas csatlakozásokhoz, nem csak szabványos RJ45 (Ethernet) portokat.
- **Tápellátás:** A szervereknek egy vagy több tápegységre van szükségük, beleértve a szükséges tartalék áramellátást is.

## A szerverek típusai

Számos típusú számítógépes szerver létezik. Bár minden szerver különálló célt szolgál, a nagyobb szerverek több szervertípust is futtathatnak, olyan tényezőktől függően, mint:

- A szerver kapacitása
- A használt operációs rendszer (például Windows, Linux, Mainframe vagy Midrange)
- A szerver architektúrája (fizikai, virtuális vagy felhőalapú)

### Fájlszerverek

A fájlszerverek strukturálatlan felhasználói fájlokat tárolnak, kezelnek és biztosítanak megosztott hozzáférést hozzájuk, beleértve a dokumentumokat, táblázatokat, PDF-eket, fényképeket, videókat és végrehajtható fájlokat.

A fájlszerverek futhatnak felhőszervereken (Google, AWS és mások), Windowson, Linuxon és néhány örökölt szerveren, például midrange, mainframe, Unix és AIX platformokon.

### DBMS szerverek

A Database Management System (DBMS) szerverek adatbázis-szolgáltatásokat nyújtanak SQL és más strukturált relációs adatbázisokhoz. A DBMS szerverek támogatják:

- Összetett lekérdezési tranzakciókat
- Adatintegritást
- Adatkezelési feldolgozást

Példák közé tartozik a Microsoft SQL Server, a MySQL, az Oracle Database, az IBM DB2, a PostgreSQL, a NoSQL DBMS platformok és más felhőalapú adatbázis-szolgáltatások.

### Nyomtatósszerverek

A nyomtatósszerverek nyomtatási kéréseket fogadnak a hálózati kliensektől, és minden kérést a megfelelő nyomtatóhoz irányítanak. Rendszerezik és sorba állítják a nyomtatási feladatokat a szűk keresztmetszetek elkerülése érdekében.

### Alkalmazásszerverek

Az alkalmazásszerverek üzleti feldolgozásban használt digitális megoldásokat üzemeltetnek – tekinthetjük őket szoftverszervereknek. A kliensek egy felhasználói felületet, például böngészőt, terminált vagy kliensalkalmazást futtatnak a szerveralkalmazások eléréséhez és futtatásához.

Az alkalmazásszerverek lehetővé teszik a kliensek számára a központosított alkalmazások elérését anélkül, hogy nagy mennyiségű adat letöltésének vagy a szoftver helyi futtatásának terhét viselnék. Több millió alkalmazásszerver fut az interneten, beleértve:

- Levelezőszerverek
- Webszerverek (ezek gyakran az alkalmazásszerverek előtt helyezkednek el a kezdeti forgalom kezelésére)
- Vállalati/üzleti alkalmazásszerverek
- Multimédia szerverek
- A szervezeteken belül használt egyéb szoftvermegoldások

Az alkalmazásszerverek számos operációs rendszeren futnak, beleértve a Windows, Linux, mainframe, midrange, Unix, konténerek és egyéb platformokat. Az alkalmazásszervereket gyakran klaszterezik, hogy nagy mennyiségű, egy központi helyre küldött felhasználói kérést dolgozzanak fel.

### Internetszerverek

Olyan szerverek, amelyek kéréseket és válaszokat irányítanak a kliensek és szerverek között helyi hálózatokon és az interneten keresztül. Az internetszerverek közé tartoznak:

- **DHCP szerverek (Dynamic Host Configuration Protocol):** Automatikusan IP-címeket és hálózati konfigurációs beállításokat rendelnek a kliensekhez és szerverekhez.
- **DNS szerverek (Domain Name System):** A felhasználóbarát domainneveket (webes URL-eket) internetes IP-címekké oldják fel, hogy lehetővé tegyék az internetes forgalom hatékony irányítását.
- **Proxy szerverek:** Átjáró szerverek, amelyek a kliensforgalmat védik azáltal, hogy elrejtik a kérelmező valódi IP-címét, és továbbítják a forgalmat a célcímére. A proxy szerverek számos módon fokozzák a biztonságot, beleértve a felhasználói IP-címek (helyek) elrejtését a rosszindulatú szereplők elől, a terheléselosztás támogatását, valamint a forgalom gyorsítótárazását és tömörítését.
- **Sima és biztonságos FTP (File Transfer Protocol) szerverek:** Az FTP szerverek megkönnyítik a fájlátvitelt a számítógépek között.

### Virtuális gépek (VM)

Az IT-csapatok hipervizor szoftvert vagy virtuális gépmegfigyelőt (VMM) használnak virtuális gépek létrehozásához egyetlen eszközön belül. Egy hipervizor több száz virtuális gépet képes kezelni egyetlen hardveren, virtualizálva a fizikai szerverek teljes rackjeit.

### AI szerverek

Az AI szerverek GPU-kat vagy a Google Tensor Processing Unit (TPU) egységeit használják LLM-feldolgozás végrehajtásához mesterséges intelligencia feladatokhoz. Az AI szerverek gyakran speciális nagy sebességű összeköttetéseket igényelnek (mint az NVIDIA NVLink vagy az InfiniBand), hogy lehetővé tegyék több GPU gyors kommunikációját. (Ez egy kulcsfontosságú technikai különbség a GPU-val rendelkező szabványos szerver és a célra épített AI szerver között.)

Az AI szerverek összetett modellezést, nyelvalapú feladatokat képesek végrehajtani, beleértve az adatelemzést, a mintafelismerést, a döntéshozatali folyamatokat, a kimenetgenerálást (szöveg, képek és előrejelzések) és még sok mást.

*(Kapcsolódó olvasmány: AI Infrastructure.)*

### Biztonsági szerverek

Speciális biztonsági feldolgozást végeznek, hogy megvédjék az adatokat és a hálózatokat a jogosulatlan hozzáféréstől, a kibertámadásoktól, az adatszivárgásoktól és egyebektől. A biztonsági szerverek védelmet nyújtanak olyan funkciókhoz, mint a tűzfalak, a behatolásészlelés/megelőzés, a hitelesítés és egyéb szolgáltatások.

## Hogyan válasszuk ki a szervezetünknek megfelelő szervert?

Vegye figyelembe ezeket az elemeket a legjobb döntés meghozatalához, amikor szervereket választ szervezete számára:

- **Szerverkövetelmények.** Válassza ki az alkalmazásaihoz igazított megfelelő hardvert és operációs rendszereket. Tervezzen a kapacitásra és a skálázhatóságra a teljesítmény fenntartása és a jövőbeli növekedés támogatása érdekében.
- **Fizikai típus.** A fizikai szerverek számos formátumban kaphatók, beleértve a torony-, rack- és blade szervereket. A felhőszerverek és a konténerek internetalapú szolgáltatások, amelyeket a szervezetek bérelnek vagy előfizetnek.
- **Elhelyezés és karbantartás.** Futtatja a munkaterheléseket bare metalon, virtuális gépeken vagy konténerizált orchestrációs platformon, mint a Kubernetes? Milyen karbantartásra lesz szükség?
- **Költségvetés.** Mennyit költhet szerverekre? A szerverköltségek tervezésekor vegye figyelembe az üzemeltetési fenntartási költségeket is.
- **Karbantartás:** Kövesse a szerverek karbantartására vonatkozó bevált gyakorlatokat, beleértve a rendszeres javítást, a szervermegfigyelést és a biztonsági mentéseket.
