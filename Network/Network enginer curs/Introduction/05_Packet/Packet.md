# Mi az a csomag?

A hálózatkezelésben a **csomag** egy nagyobb üzenet kis szegmense. A számítógépes hálózatokon, például az interneten keresztül küldött adatok csomagokra vannak felosztva. Ezeket a csomagokat aztán a fogadó számítógép vagy eszköz újra összeállítja.

Tegyük fel, hogy Alice levelet ír Bobnak, de Bob postaládája csak akkora, hogy elfogadja a kis kártya méretű borítékokat. Ahelyett, hogy a levelét normál papírra írná, majd megpróbálná átpréselni a postaládán, Alice sokkal rövidebb szakaszokra osztja a levelét, mindegyik néhány szó hosszú, és ezeket a szakaszokat kártyákra írja. Átadja a kártyacsomagot Bobnak, aki sorba rendezi őket, hogy elolvassa a teljes üzenetet.

Ez hasonló ahhoz, ahogy a csomagok működnek az interneten. Tegyük fel, hogy egy felhasználónak be kell töltenie egy képet. A képfájl nem egy darabban megy a webszerverről a felhasználó számítógépére. Ehelyett adatcsomagokra van bontva, elküldve az internet vezetékein, kábeleken és rádióhullámain keresztül, majd a felhasználó számítógépe újra összeállítja az eredeti fotót.

*A hálózat két vagy több összekapcsolt számítógép csoportja. Az internet hálózatok hálózata – több, világszerte egymással összekapcsolt hálózat.

## Miért használunk csomagokat?

Elméletileg lehetséges lenne fájlokat és adatokat küldeni az interneten anélkül, hogy kis információcsomagokra aprítanánk őket. Egy számítógép egy másik számítógépnek egy hosszú, megszakítatlan bitfolyam formájában küldhetné az adatokat (a bitek kis információegységek, amelyeket elektromos impulzusokként kommunikálnak, és amelyeket a számítógépek értelmezni tudnak).

Ez a megközelítés azonban gyorsan kivitelezhetetlenné válik, amikor kettőnél több számítógép vesz részt benne. Míg a hosszú bitfolyam áthalad a két számítógép közötti vezetékeken, egy harmadik számítógép nem tudná ugyanezeket a vezetékeket használni információ küldésére – várnia kellene a sorára.

Ezzel a megközelítéssel szemben az internet egy „csomagkapcsolt" hálózat. A csomagkapcsolás a hálózati berendezések azon képességére utal, hogy a csomagokat egymástól függetlenül dolgozzák fel. Azt is jelenti, hogy a csomagok különböző hálózati útvonalakat vehetnek igénybe ugyanahhoz a célállomáshoz, amíg mindegyik megérkezik a célállomásra. (Bizonyos protokollokban a csomagoknak a végső célállomásukra a helyes sorrendben kell megérkezniük, még akkor is, ha mindegyik csomag más útvonalat vett igénybe.)

A csomagkapcsolásnak köszönhetően több számítógép csomagjai gyakorlatilag bármilyen sorrendben utazhatnak ugyanazokon a vezetékeken. Ez lehetővé teszi, hogy egyszerre több kapcsolat jöjjön létre ugyanazon a hálózati berendezésen. Ennek eredményeként több milliárd eszköz tud adatot cserélni az interneten egyszerre, nem csak néhány.

## Mi az a csomagfejléc?

A csomagfejléc egyfajta „címke", amely információt nyújt a csomag tartalmáról, eredetéről és célállomásáról.

Amikor Alice elküldi a kártyasorozatát Bobnak, a kártyákon lévő szavak önmagukban nem adnak Bobnak elég kontextust a levél helyes elolvasásához. Alice-nek jeleznie kell a kártyák sorrendjét, hogy Bob ne olvassa el őket rossz sorrendben. Azt is jeleznie kell, hogy mindegyik tőle származik, arra az esetre, ha Bob másoktól is kapna üzeneteket, miközben ő a sajátját kézbesíti. Ezért Alice hozzáadja ezt az információt minden kártya tetejére, az üzenete tényleges szavai fölé. Az első kártyára azt írja: „Levél Alice-től, 1 a 20-ból", a másodikra: „Levél Alice-től, 2 a 20-ból", és így tovább.

Alice létrehozott egy miniatűr fejlécet a kártyáihoz, hogy Bob ne veszítse el vagy keverje össze őket. Hasonlóképpen, minden hálózati csomag tartalmaz egy fejlécet, hogy a fogadó eszköz tudja, honnan származnak a csomagok, mi a céljuk, és hogyan kell feldolgozni őket.

A csomagok két részből állnak: a fejlécből és a hasznos teherből (payload). A fejléc információkat tartalmaz a csomagról, például annak eredetéről és cél IP-címéről (az IP-cím olyan, mint egy számítógép postai címe). A hasznos teher a tényleges adat. Visszatérve a fotó példájához, a képet alkotó több ezer csomag mindegyikének van hasznos terhe, és a hasznos teher a kép egy kis darabját hordozza.

## Honnan származnak a csomagfejlécek?

A gyakorlatban a csomagoknak valójában több fejlécük van, és mindegyik fejlécet a hálózati folyamat más-más része használja. A csomagfejléceket bizonyos típusú hálózati protokollok csatolják.

A protokoll az adatok formázásának szabványosított módja, hogy bármely számítógép értelmezni tudja az adatokat. Sok különböző protokoll működteti az internetet. Ezen protokollok némelyike fejléceket ad a csomagokhoz az adott protokollhoz kapcsolódó információkkal. Az interneten áthaladó legtöbb csomag legalább egy Transmission Control Protocol (TCP) fejlécet és egy Internet Protocol (IP) fejlécet tartalmaz.

## Mik azok a csomagutótagok és lábjegyzetek?

A csomagfejlécek minden csomag elején találhatók. A routerek, switchek, számítógépek és bármi más, ami egy csomagot feldolgoz vagy fogad, először a fejlécet látja. Egy csomagnak lehetnek utótagjai és lábjegyzetei is a végén. A fejlécekhez hasonlóan ezek is további információkat tartalmaznak a csomagról.

Csak bizonyos hálózati protokollok csatolnak utótagokat vagy lábjegyzeteket a csomagokhoz; a legtöbb csak fejléceket csatol. Az ESP (az IPsec csomag része) egy példa olyan hálózati réteg protokollra, amely utótagokat csatol a csomagokhoz.

## Mi az az IP-csomag?

Az IP (Internet Protocol) egy hálózati réteg protokoll, amely az útvonalválasztással foglalkozik. Arra használják, hogy biztosítsák a csomagok helyes célállomásra való megérkezését.

A csomagokat néha az általuk használt protokoll alapján határozzák meg. Az IP fejléccel rendelkező csomagot „IP-csomagnak" nevezhetjük. Az IP fejléc fontos információkat tartalmaz arról, hogy honnan származik a csomag (forrás IP-címe), hová tart (cél IP-címe), mekkora a csomag, és meddig kell a hálózati routereknek továbbítaniuk a csomagot, mielőtt eldobják. Azt is jelezheti, hogy a csomag feldarabolható-e, és tartalmazhat információt a feldarabolt csomagok újraösszeállításáról.

## Csomagok vs. datagramok

A „datagram" egy csomagkapcsolt hálózaton keresztül küldött adatszegmens. A datagram elegendő információt tartalmaz ahhoz, hogy a forrásától a célállomásáig irányítsák. E meghatározás szerint az IP-csomag a datagram egyik példája. Lényegében a datagram a „csomag" alternatív kifejezése.

## Mi a hálózati forgalom? Mi a rosszindulatú hálózati forgalom?

A hálózati forgalom egy olyan kifejezés, amely a hálózaton áthaladó csomagokra utal, ugyanúgy, ahogy az autós forgalom az utakon közlekedő autókra és teherautókra utal.

Azonban nem minden csomag jó vagy hasznos, és nem minden hálózati forgalom biztonságos. A támadók rosszindulatú hálózati forgalmat generálhatnak – olyan adatcsomagokat, amelyeket egy hálózat veszélyeztetésére vagy túlterhelésére terveztek. Ez megnyilvánulhat elosztott szolgáltatásmegtagadási (DDoS) támadás, sebezhetőség-kihasználás vagy számos egyéb kibertámadási forma formájában.

A Cloudflare számos terméket kínál, amelyek védelmet nyújtanak a rosszindulatú hálózati forgalom ellen. A Cloudflare Magic Transit például a vállalati hálózatokat védi a DDoS támadásoktól a hálózati rétegben azáltal, hogy kiterjeszti a Cloudflare globális felhőhálózatának erejét a helyszíni, hibrid és felhőalapú infrastruktúrára.
