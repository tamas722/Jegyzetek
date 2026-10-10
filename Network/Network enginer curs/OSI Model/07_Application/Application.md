# Mi az internet 7. rétege?

A **7. réteg** az internet 7 rétegű OSI-modelljének legfelső rétegére utal. „Alkalmazási rétegnek" is nevezik. Ez az adatfeldolgozás legfelső rétege, amely közvetlenül a felhasználók által használt szoftveralkalmazások felszíne alatt vagy a színfalak mögött történik. Például a weboldalak betöltéséhez használt HTTP-kérések és -válaszok 7. rétegbeli események.

Az ezen a szinten történő DDoS-támadásokat **7. rétegbeli támadásoknak** vagy **alkalmazási rétegbeli támadásoknak** nevezik. A DDoS-támadások az OSI-modell 3. vagy 4. rétegében is előfordulhatnak.

## Mi az OSI-modell?

Az **OSI (open systems interconnection) modell** egy hálózati rendszer funkcióit 7 rétegre osztja, mindegyik réteg elvonatkoztatva az alatta lévőtől. A modellen belül minden réteg csak a felette és alatta lévő rétegekkel kommunikál.

Érdemes szem előtt tartani, hogy az OSI-modell pusztán elméleti, és arra tervezték, hogy segítsen leírni, mi történik a hálózati kommunikációban, nem pedig a tényleges technológia leírására. Az, hogy az OSI-modell csak fogalmi keretrendszer, nem jelenti azt, hogy ne lenne hasznos; a modellre hivatkozva a mérnökök, fejlesztők és IT-szakemberek pontosan meghatározhatják, mit tesz egy termék vagy protokoll, és hová tartozik a hálózati kommunikáció folyamatában.

## Az OSI-modell

A modell alján a **fizikai réteg (1. réteg)** található, vagyis az elektromosság impulzusai, amelyek az információ bitekeit továbbítják a kábeleken, routereken, switcheken és WiFi hálózatokon keresztül, amelyek az internet infrastruktúráját alkotják. A tetején, a 7. rétegben vannak azok a protokollok és szolgáltatások, amelyeket az alkalmazások a működésükhöz használnak. Közöttük különféle funkciók és protokollok találhatók, amelyeken az adatok áthaladnak a hálózati kommunikáció során.

Az egyes rétegek részletesebb lebontásáért lásd: „Mi az OSI-modell?"

## Mit csinál a 7. réteg?

Bár a 7. réteg alkalmazási rétegként ismert, ez nem maguknak az alkalmazásoknak a felhasználói felülete. Inkább a 7. réteg biztosítja azokat a funkciókat és szolgáltatásokat, amelyeket a felhasználóknak szánt szoftveralkalmazások az adatok megjelenítéséhez használnak. Ha egy alkalmazás olyan, mint egy ház, akkor a 7. réteg az alap, nem maga a ház.

Az API-hívások és -válaszok ehhez a réteghez tartoznak, és a használt fő protokollok közé tartozik a **HTTP** és az **SMTP** (Simple Mail Transfer Protocol, amelyet e-mail alkalmazások használnak).

## Hogyan kommunikál a 7. réteg a többi OSI-réteggel?

A 7. rétegből származó adatok lefelé haladnak a stackben, bár a 7. réteg csak a 6. réteggel kommunikál. Ahogy az adatok lefelé haladnak a stackben, csomagokra bomlanak, és bizonyos rétegek fejléceket és lábléceket adnak minden csomaghoz – például a 3. rétegben egy IP-fejlécet adnak hozzá minden csomaghoz, amely tartalmazza a cél- és forrás IP-címeket. A stack alján az adatok bitekké alakulnak, és átvitelre kerülnek a fizikai interneten.

Amikor elérik a célállomásukat, az adatok visszafelé haladnak a stackben, az 1. rétegtől kezdve. Minden rétegben a fejléc- és láblécadatokat értelmezik és eltávolítják, és az adatokat a következő réteg számára használható formába helyezik. Amint az adatok elérik a 7. réteget a másik oldalon, elérhetővé válnak az alkalmazások számára. (Mindezen lépések ellenére a teljes folyamat csak ezredmásodperceket vesz igénybe.)

Az OSI-modell működésének megértéséhez kulcsfontosságú az a tény, hogy minden réteg csak az interakció másik végén lévő ugyanazzal a réteggel kommunikál. A 7. réteg adatait csak a kommunikáció fogadó végén lévő 7. réteg értelmezi; a fogadó végén lévő többi réteg csupán továbbítja az adatokat a 7. réteghez. Hasonlóképpen, az IP-fejléc adatait, amelyet az egyik oldalon a 3. rétegben csatolnak az adatcsomagokhoz, csak a másik oldalon a 3. réteg olvassa és értelmezi.

## Hogyan működnek a 7. rétegbeli DDoS-támadások?

A **7. rétegbeli vagy alkalmazási rétegbeli DDoS-támadások** megpróbálják elárasztani a hálózati vagy szervererőforrásokat forgalommal (jellemzően HTTP-forgalommal). Példa erre több ezer kérés küldése egy bizonyos weboldalra másodpercenként, amíg a szerver túlterheltté válik, és nem tud válaszolni az összes kérésre. Egy másik példa egy API ismételt hívása, amíg a szolgáltatás összeomlik.

Ha többet szeretnél megtudni, lásd: „Alkalmazási rétegbeli DDoS-támadások."
