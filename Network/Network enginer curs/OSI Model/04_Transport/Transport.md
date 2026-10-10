# A 4. réteg megértése: Az OSI-modell szállítási rétege

**IT Index > A 4. réteg megértése: Az OSI-modell szállítási rétege**
**Frissítve: 2025. augusztus 4.**

A **szállítási réteg** az OSI-modell negyedik rétege, amely megbízható kommunikációt tesz lehetővé a különböző eszközökön lévő alkalmazások között. Olyan funkciókat kezel, mint az adatszegmentálás, az újraösszeállítás, a folyamatszabályozás a túlterhelés megelőzésére, valamint a torlódásszabályozás a hálózati telítettség elkerülésére.

Az alsóbb rétegekkel ellentétben, amelyek az adatok mozgatására összpontosítanak, a szállítási réteg zökkenőmentes kommunikációt biztosít olyan alkalmazások számára, mint a webböngészés, az e-mail és a streamelés.

E réteg megértése kulcsfontosságú a hálózatokat kezelő, hibaelhárítást végző és a teljesítményt optimalizáló IT-szakemberek számára. Hídként szolgál a hálózati útválasztás és az alkalmazásigények között, így a modern hálózatkezelés alapelemévé válik.

## Definíció és alapfogalmak

A **szállítási réteg** a negyedik rétegként működik a hétrétegű OSI fogalmi keretrendszerben. Az OSI-modell szabványosított megközelítést nyújt a hálózati kommunikációhoz, ahol minden réteg az alatta lévő réteg szolgáltatásaira épül.

A szállítási réteg elsődleges felelőssége az **végpontok közötti kommunikáció** az alkalmazásfolyamatok között, nem csupán az eszközök között. Ez a hosztok közötti kommunikáció biztosítja, hogy az egyik alkalmazásból küldött adatok eljussanak a célhoszt megfelelő alkalmazásához.

A **megbízható kézbesítés** e réteg alapkoncepcióját képezi, garantálva, hogy az adatok helyesen és hiánytalanul érkezzenek meg a célállomásukra. A réteg a **sorrendben történő kézbesítést** is biztosítja, fenntartva az adatszegmensek eredeti sorrendjét az újraösszeállítás során a fogadó oldalon.

A **szegmentálás** a nagy alkalmazásadatokat kisebb, kezelhető egységekre bontja, amelyek alkalmasak a hálózati átvitelre. A megfelelő **újraösszeállítási** folyamat rekonstruálja ezeket a szegmenseket az eredeti adatformátumba a célállomáson.

A **folyamatszabályozás** megakadályozza, hogy a gyors küldők elárasszák a lassabb fogadókat az adatátviteli sebesség kezelésével. A **torlódásszabályozás** a hálózati szintű aggályokat kezeli, megakadályozva, hogy a küldők túlterheljék magát a hálózati infrastruktúrát.

A **szolgáltatáspont-címzés** portszámokat használ az adott alkalmazások azonosítására egy hoszton. Ez lehetővé teszi a multiplexelést és demultiplexelést – lehetővé téve, hogy több alkalmazás osszon meg egyetlen hálózati kapcsolatot, miközben biztosítja, hogy az adatok eljussanak a helyes célalkalmazáshoz.

A szállítási réteg az alatta lévő **hálózati rétegre (3. réteg)** támaszkodik a csomagok hálózatok közötti útválasztásához. Szolgáltatásokat nyújt a felette lévő **munkamenet-rétegnek (5. réteg)**, kezelve a megbízható adatátvitel összetett részleteit.

A **Protocol Data Unit (PDU)** ebben a rétegben **szegmensnek** nevezik a Transmission Control Protocol (TCP) esetében, és **datagramnak** a User Datagram Protocol (UDP) esetében.

## Hogyan működik?

A szállítási réteg több kifinomult mechanizmust alkalmaz a megbízható, hatékony kommunikáció biztosítására az alkalmazások között a hálózatokon keresztül.

### Szolgáltatáspont-címzés (portszámok)

A **portszámok** 16 bites azonosítókként szolgálnak a szállítási réteg fejlécében, az adatokat adott alkalmazásfolyamatokhoz irányítva egy hoszton. Ez a címzési rendszer lehetővé teszi, hogy több alkalmazás működjön egyidejűleg egyetlen eszközön.

A **multiplexelés** lehetővé teszi a küldő hoszt számára, hogy több alkalmazás adatait egyetlen hálózati kapcsolatba kombinálja. Minden alkalmazás adatai tartalmazzák az egyedi portszámukat, különálló kommunikációs csatornákat hozva létre a megosztott hálózati infrastruktúrán.

A **demultiplexelés** a fogadó oldalon történik, ahol az operációs rendszer megvizsgálja a bejövő portszámokat, és az adatokat a megfelelő alkalmazáshoz irányítja. Ez a folyamat biztosítja, hogy a webes forgalom elérje a böngésződet, míg az e-mail adatok az e-mail kliensedhez kerüljenek.

A gyakori portkiosztások közé tartozik a **80-as port** a HTTP webes forgalomhoz, a **443-as port** a HTTPS biztonságos webes forgalomhoz, és a **25-ös port** a Simple Mail Transfer Protocol (SMTP) e-mail továbbításhoz.

### Szegmentálás és újraösszeállítás

A nagy adatátvitelek kisebb egységekre osztást igényelnek a hálózati korlátokhoz való alkalmazkodás és az átviteli hatékonyság javítása érdekében. A szállítási réteg **szegmentálást** végez az alkalmazásadatok megfelelő méretű szegmensekre (TCP) vagy datagramokra (UDP) bontásával.

Minden szegmens tartalmaz **sorrendinformációt**, amely lehetővé teszi a megfelelő újraösszeállítást a célállomáson. A fogadó szállítási réteg ezt a sorrendadatot használja az eredeti alkalmazásadatok rekonstruálásához, még akkor is, ha a szegmensek a hálózati útvonalválasztási eltérések miatt sorrenden kívül érkeznek.

Ez a folyamat átláthatóan működik az alkalmazások számára, amelyek nagy fájlokat vagy adatfolyamokat küldhetnek anélkül, hogy aggódnának a mögöttes hálózati csomagméret-korlátok miatt.

### Kapcsolatkezelés (TCP esetében)

A TCP megbízható kapcsolatokat hoz létre egy **háromutas kézfogási** folyamaton keresztül. A kliens elküld egy SYN (synchronize) csomagot a kapcsolat kezdeményezéséhez. A szerver SYN-ACK (synchronize-acknowledge) csomaggal válaszol, megerősítve a vételt és a készenlétet. A kliens egy ACK (acknowledge) csomaggal fejezi be a kézfogást.

A kapcsolat megszüntetése egy **négyutas kézfogást** használ. Bármelyik fél elküld egy FIN (finish) csomagot a megszüntetés megkezdéséhez. A fogadó ACK-kal nyugtázza, majd elküldi a saját FIN csomagját. Az eredeti küldő egy végső ACK-kal nyugtázza, befejezve a megszüntetési folyamatot.

Ez a kapcsolatkezelés biztosítja, hogy mindkét fél egyetértsen a kommunikációs paraméterekben, és tisztán zárja le a kapcsolatokat, amikor befejezik.

### Megbízható adatátvitel (TCP esetében)

A TCP több mechanizmust valósít meg a megbízható adatkézbesítés garantálására. A **sorszámok** egyedi azonosítókat rendelnek minden bájthoz vagy szegmenshez, lehetővé téve a fogadó számára a hiányzó adatok észlelését és a megfelelő sorrend fenntartását.

A **nyugtázások (ACK-k)** visszajelzést nyújtanak a fogadótól a küldőnek, megerősítve a sikeres adatvételt. Ha a küldő nem kap ACK-t egy meghatározott időtúllépési perióduson belül, automatikusan újraküldi az adatokat.

A **hibaészlelés** ellenőrző összegeket használ az adatok integritásának ellenőrzésére minden szegmensen belül. A fogadó kiszámítja a bejövő adatok ellenőrző összegeit, és összehasonlítja azokat a továbbított ellenőrző összeg értékekkel a sérülés azonosításához.

A **duplikált adatok elvetése** megakadályozza ugyanazon adatok többszöri feldolgozását. A fogadó sorszámokat használ a hálózati újraküldésekből eredő duplikált szegmensek azonosítására és elvetésére.

### Folyamatszabályozás (TCP esetében)

A **csúszóablak-mechanizmus** megakadályozza, hogy a gyors küldők elárasszák a fogadó puffereit. A fogadó a TCP-fejlécekben lévő fogadói ablaksméret-mezőn keresztül hirdeti meg a rendelkezésre álló pufferméretét.

A küldő tiszteletben tartja ezt az ablaksméretet, csak annyi adatot továbbítva, amennyit a fogadó jelenleg kezelni tud. Ahogy a fogadó feldolgozza az adatokat és felszabadítja a pufferterületet, frissíti az ablaksméretet a későbbi nyugtázásokban.

Ez a dinamikus beállítás biztosítja a zökkenőmentes adatáramlást, függetlenül a küldő és a fogadó közötti feldolgozási sebességkülönbségektől.

### Torlódásszabályozás (TCP esetében)

A TCP **torlódási ablak kezelést** valósít meg a hálózati túlterhelés megelőzésére. A küldő fenntart egy torlódási ablakot, amely dinamikusan igazodik a hálózati feltételekhez, például a csomagvesztéshez vagy a megnövekedett késleltetéshez.

A gyakori torlódásszabályozási algoritmusok közé tartozik a **Slow Start**, amely fokozatosan növeli az átviteli sebességeket, és a **Fast Retransmit**, amely gyorsan reagál az észlelt csomagvesztésre. Ezek a mechanizmusok biztosítják a tisztességes sávszélesség-megosztást több hálózati felhasználó között, miközben maximalizálják az átviteli teljesítményt.

## Fő jellemzők és összetevők

A szállítási réteg számos alapvető funkciót nyújt, amelyek lehetővé teszik a modern hálózati kommunikációt:

- **Végpontok közötti kommunikáció** összekapcsolja az alkalmazásfolyamatokat a hálózatokon keresztül, elrejtve a komplex útválasztási részleteket az alkalmazások elől. Ez zökkenőmentes kommunikációt tesz lehetővé a mögöttes hálózati infrastruktúrától függetlenül.
- **Portcímzés** szabványosított módszert nyújt az alkalmazások azonosítására 16 bites portszámok használatával. A well-known portok (0-1023) rendszerszolgáltatások számára vannak fenntartva, míg a regisztrált portok (1024-49151) konkrét alkalmazásokat szolgálnak.
- **Megbízhatóság** TCP-n keresztül garantálja az adatkézbesítést, fenntartja a megfelelő sorrendet, és biztosítja a hibamentes átvitelt. Ezek a funkciók elengedhetetlenek az adatintegritást igénylő alkalmazásokhoz, mint a fájlátvitelek és az adatbázis-tranzakciók.
- **Folyamatszabályozási** mechanizmusok megakadályozzák a fogadói puffer túlcsordulását az átviteli sebességek kezelésével. Ez biztosítja a megbízható kommunikációt a különböző feldolgozási képességű eszközök között.
- **Torlódásszabályozás** megakadályozza a hálózati túlterhelést az átviteli sebességek dinamikus beállításával a hálózati feltételek alapján. Ez elősegíti a tisztességes erőforrás-megosztást és az optimális hálózati teljesítményt.
- **Multiplexelés és demultiplexelés** lehetővé teszi, hogy több alkalmazás hatékonyan osszon meg hálózati kapcsolatokat. Ez maximalizálja a hálózati erőforrások kihasználását, miközben fenntartja az alkalmazások elkülönítését.
- **Szegmentálás és újraösszeállítás** kezeli az adatok méretezését a hálózati átvitelhez, alkalmazkodva a különböző hálózati szegmensek eltérő Maximum Transmission Unit (MTU) követelményeihez.

## Gyakori protokollok a szállítási rétegben

Két elsődleges protokoll dominálja a szállítási réteg működését, mindegyik különböző alkalmazáskövetelményekre optimalizálva.

### TCP (Transmission Control Protocol)

A **TCP** kapcsolat-orientált, megbízható és sorrendben történő adatkézbesítést nyújt. Formális kapcsolatokat hoz létre háromutas kézfogásokon keresztül, és fenntartja ezeket a kapcsolatokat az adatátviteli munkamenetek során.

A fő TCP-funkciók közé tartoznak a sorszámok a sorrendezéshez, a nyugtázások a megbízhatósághoz, az automatikus újraküldések az elveszett adatokhoz, a folyamatszabályozás a fogadó védelméhez és a torlódásszabályozás a hálózat optimalizálásához.

A TCP olyan alkalmazásokhoz alkalmas, amelyek garantált adatkézbesítést és megfelelő sorrendezést igényelnek. A webböngészés HTTP-t és HTTPS-t használ TCP felett a teljes oldalbetöltés biztosításához. Az e-mail rendszerek SMTP-t, POP3-at és IMAP-ot használnak TCP felett a megbízható üzenetkézbesítéshez. A fájlátviteli alkalmazások FTP-t használnak TCP felett a teljes és pontos fájlátvitel garantálásához. A Secure Shell (SSH) kapcsolatok TCP-re támaszkodnak a megbízható távoli hozzáférési munkamenetekhez.

### UDP (User Datagram Protocol)

Az **UDP** kapcsolat nélküli, megbízhatatlan, best-effort kézbesítést kínál minimális protokoll-overheaddel. Gyorsabb átvitelt biztosít a kapcsolat létrehozásának, a nyugtázásoknak és az újraküldéseknek a kiküszöbölésével.

Az UDP jellemzői közé tartozik a minimális fejléc-overhead, a kapcsolatállapot fenntartásának hiánya és a kézbesítési garanciák hiánya. Az UDP-t használó alkalmazásoknak önállóan kell kezelniük a szükséges megbízhatósági mechanizmusokat.

Az UDP kiválóan alkalmas olyan alkalmazásokban, amelyek a sebességet a garantált kézbesítés elé helyezik. A Voice over IP (VoIP) és a videokonferencia az UDP-t részesíti előnyben a késleltetés minimalizálása érdekében, elfogadva az alkalmi csomagvesztést az átviteli késedelmekkel szemben. Az online játék UDP-t használ a valós idejű interakcióhoz, ahol a jelenlegi játékállapot fontosabb, mint a történelmi adatok. A Domain Name System (DNS) lekérdezések UDP-t használnak a gyors hosztnév-feloldáshoz. A Dynamic Host Configuration Protocol (DHCP) UDP-t alkalmaz a hatékony IP-cím-kiosztáshoz.

## Felhasználási esetek és alkalmazások

A szállítási réteg gyakorlatilag az összes internetes alkalmazást lehetővé teszi alapvető kommunikációs szolgáltatásain keresztül.

A **webszerverek és kliensek** a TCP-re támaszkodnak a teljes weboldal-betöltés biztosításához. Megbízható kézbesítés nélkül a hiányzó HTML-elemek, képek vagy szkriptek használhatatlanná tennék a weboldalakat. A TCP sorrendben történő kézbesítése biztosítja az összetett webes tartalom megfelelő megjelenítését.

A **valós idejű streamelési alkalmazások** kihasználják az UDP alacsony késleltetésű jellemzőit. Az élő videóközvetítések, az online játék és a hanghívások a jelenlegi adatokat a tökéletes kézbesítés elé helyezik. A rövid megszakítások kevésbé zavaróak, mint az újraküldési mechanizmusok által okozott késedelmek.

A **hálózati szolgáltatások** portszámokat használnak, hogy több szolgáltatást tegyenek lehetővé egyetlen szerveren. A webszerverek (80-as port), a biztonságos webszerverek (443-as port), az e-mail szerverek (25-ös port) és a DNS szerverek (53-as port) egyidejűleg működhetnek ugyanazon a hardveren.

A **vállalati alkalmazások** a TCP megbízhatóságára támaszkodnak a kritikus üzleti műveletekhez. Az adatbázis-tranzakciók, a pénzügyi rendszerek és a készletkezelés garantált adatintegritást igényelnek, amelyet csak megbízható protokollok tudnak nyújtani.

Az **Internet of Things (IoT)** eszközök gyakran UDP-t használnak periodikus érzékelőadat-továbbításhoz, ahol az alkalmi adatvesztés elfogadható, de az akkumulátor-kímélés kritikus.

## Előnyök és kompromisszumok

A szállítási réteg jelentős előnyöket nyújt, miközben bizonyos korlátokat vezet be, amelyeket az IT-szakembereknek figyelembe kell venniük.

### Előnyök

- **Alkalmazásabsztrakció** megszabadítja a fejlesztőket a hálózati bonyolultság részleteitől. Az alkalmazások az üzleti logikára összpontosíthatnak, míg a szállítási réteg kezeli a kommunikáció megbízhatóságát, sorrendezését és hibajavítását.
- **Megbízhatóság és folyamatszabályozás** TCP-n keresztül biztosítja az adatintegritást a kritikus alkalmazásokhoz. A pénzügyi tranzakciók, az orvosi feljegyzések és a jogi dokumentumok igénylik a TCP által nyújtott garantált kézbesítést.
- **Hatékonyság** UDP-n keresztül könnyűsúlyú kommunikációt kínál az időérzékeny alkalmazásokhoz. A minimális overhead lehetővé teszi az olyan valós idejű alkalmazásokat, amelyek nem tolerálják a megbízhatósági mechanizmusokhoz kapcsolódó késedelmeket.
- **Multiplexelési képességek** maximalizálják a hálózati erőforrások kihasználását azáltal, hogy több alkalmazás számára teszik lehetővé a kapcsolatok megosztását. Ez a hatékonyság csökkenti az infrastruktúra-követelményeket és javítja az általános hálózati teljesítményt.

### Kompromisszumok és korlátok

- **TCP overhead** jelentősen befolyásolja a teljesítményt a további fejlécek, a nyugtázási forgalom és az újraküldések révén. Ezek a megbízhatósági mechanizmusok jelentős sávszélességet és feldolgozási erőforrásokat fogyaszthatnak.
- **Implementációs bonyolultság** kihívásosabbá teszi a TCP fejlesztését és hibaelhárítását az egyszerűbb protokollokhoz képest. A kapcsolatkezelés és a torlódásszabályozás állapotgépei gondos implementációt és monitorozást igényelnek.
- **Hálózati réteg függőség** azt jelenti, hogy a szállítási réteg nem működhet önállóan. Teljes mértékben a 3. rétegbeli útválasztási képességekre támaszkodik, és nem tudja közvetlenül irányítani a csomagok útvonalait a hálózatokon keresztül.
- **Head-of-line (HOL) blokkolás** a TCP-ben késleltetheti a következő adatszegmenseket, miközben a hiányzó szegmensek újraküldésére vár. Ez befolyásolhatja az alkalmazás teljesítményét, különösen nagy késleltetésű hálózatokban. A modern protokollok, mint a HTTP/2 és a QUIC, magasabb rétegekben történő multiplexeléssel kezelik ezt a korlátot.

## Kulcsfogalmak függelék

- **Szállítási réteg (4. réteg):** Az OSI-modell negyedik rétege, amely végpontok közötti kommunikációt biztosít az alkalmazásfolyamatok között.
- **OSI-modell:** Egy hétrétegű fogalmi keretrendszer, amely szabványosítja a hálózati kommunikációs protokollokat és szolgáltatásokat.
- **TCP (Transmission Control Protocol):** Kapcsolat-orientált, megbízható szállítási protokoll, amely garantálja a sorrendben történő adatkézbesítést.
- **UDP (User Datagram Protocol):** Kapcsolat nélküli, megbízhatatlan szállítási protokoll, amely a sebességre és a minimális overheadre optimalizált.
- **Portszám:** Egy 16 bites azonosító, amely egy adott alkalmazásfolyamatot határoz meg egy hálózati hoszton.
- **Szegmentálás:** A nagy adategységek kisebb szegmensekre vagy datagramokra osztásának folyamata a hálózati átvitelhez.
- **Újraösszeállítás:** Az eredeti adatok rekonstruálásának folyamata a fogadott szegmensekből vagy datagramokból a célállomáson.
- **Folyamatszabályozás:** Mechanizmusok, amelyek megakadályozzák, hogy a gyors küldők elárasszák a lassabb fogadókat az átviteli sebesség kezelésén keresztül.
- **Torlódásszabályozás:** Technikák, amelyek megakadályozzák a hálózati túlterhelést a küldő átviteli sebességének dinamikus beállításával a hálózati feltételek alapján.
- **Megbízható kézbesítés:** Protokollgaranciák, amelyek biztosítják, hogy az adatok hiánytalanul és helyesen érkezzenek meg a szánt célállomásukra.
- **Sorrendben történő kézbesítés:** Mechanizmusok, amelyek biztosítják, hogy az adatszegmensek az eredeti átviteli sorrendjükben álljanak össze újra.
- **Háromutas kézfogás:** A TCP kapcsolat-létrehozási folyamata SYN, SYN-ACK és ACK csomagok használatával.
- **Sorszámok:** Egyedi azonosítók, amelyeket adatbájtokhoz vagy szegmensekhez rendelnek a megfelelő sorrendezés és a duplikátumészlelés lehetővé tételéhez.
- **Nyugtázások (ACK-k):** Megerősítő üzenetek, amelyeket a fogadók küldenek a sikeres adatvétel jelzésére.
- **Head-of-Line (HOL) blokkolás:** Teljesítményromlás, amely akkor következik be, amikor egy hiányzó csomag késlelteti a következő csomagok kézbesítését.
- **Hálózati réteg (3. réteg):** Az OSI-réteg, amely a csomagok különböző hálózatok közötti útválasztásáért felelős.
- **Alkalmazási réteg (7. réteg):** Az OSI-réteg, amely közvetlenül nyújt hálózati szolgáltatásokat a felhasználói alkalmazásoknak.
- **PDU (Protocol Data Unit):** Az adategység formátuma minden OSI-rétegben – szegmensek a TCP-hez, datagramok az UDP-hez a szállítási rétegben.
