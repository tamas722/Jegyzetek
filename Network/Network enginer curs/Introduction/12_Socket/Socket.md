# Mik azok a socketek?

**K.M Ahnaf Zamil**

8 perc olvasás · 2022. július 16.

Ha programozó vagy, és nem egy szikla alatt élsz, valószínűleg elég gyakran hallottad már a „socket" kifejezést. Ebben a bejegyzésben megpróbálom egyszerű módon elmagyarázni, mik azok a socketek és hogyan működnek. Nem fogok példákat mutatni konkrét nyelvekhez, mert arra jobb források vannak. Csak keress rá a Google-ban, hogy „Sockets in (insert programming language)", és rengeteg tutorialt találsz.

## Mi ez a nagy felhajtás a socketek körül?

Nos, a socketek alapvetően egy mód arra, hogy más programokkal beszélj egy számítógépen, vagy más számítógépekkel egy hálózaton. Alapvetően ennyi, de még a felszínét sem karcoltuk meg. Egy kezdő programozó a socketet valamiféle absztrakciónak vagy mágiának tekintené, amivel szerverekhez vagy számítógépekhez lehet beszélni egy hálózaton (nem sértésképpen). De ennél sokkal többről van szó, SOKKAL többről; és erről szeretnék mesélni.

A socketek főként UNIX/Linux koncepció, de minden operációs rendszeren hasonló módon működnek, hogy fenntartsák a közös megvalósítási módot (tehát ha Windows-felhasználó vagy, a tudás így is hasznos lehet). Ez talán meglepő lehet számodra, de minden socket egyszerűen egy fájlleíró (file descriptor). A fájlleíró egy egész szám, amely egy nyitott fájlra utal/van hozzárendelve. A socketek a fájlleírókba való olvasással/írással kommunikálnak. És nem csak a socketek csinálják ezt; valójában minden UNIX program I/O-t végez fájlleíróba való olvasással vagy írással. Ez magában foglalja a hálózati kapcsolatokat is, amelyekre általában a socketeket használnánk.

Ebben a bejegyzésben csak az internet/hálózati socketekről fogok beszélni, nem más típusú socketekről (az örökké tartana, ha arról írnék).

## Hogyan használjuk a socketeket?

A socketeket az OS kernel kezeli és monitorozza, és rendszerrutinokkal hozhatsz létre socketeket. A socket() rendszerhívás visszaad egy fájlleírót, amelyre „socket leíróként" hivatkozhatsz. Ez a socket leíró egyszerűen egy egész szám, ahogy fentebb említettem. Most a send() és recv() rendszerhívásokkal küldhetsz és fogadhatsz adatokat abból a socketből (sokkal több funkció van, de csak ezt a 3-at adtam példaként).

Talán azon gondolkodsz: „Nincs erre egyszerűbb mód?" Nos, van. Ezen funkciók többsége elérhető C-ben a sys/socket.h fejlécfájlban (ez csak UNIX-szerű operációs rendszerekre vonatkozik. Windowsnak winsock2.h kell). Ismét, más programozási nyelveknek vannak könyvtárai, amelyek elrejtik ezt a komplex dolgot (pl. Pythonnak van socket modulja). Használd, amelyik nyelvet/könyvtárat akarod. DE, ha mélyebb megértést szeretnél arról, hogyan működnek a „nyers" socketek, és szeretnél vele kísérletezni, azt javaslom, hogy C/C++-t használj erre a célra. Később bármilyen más programozási nyelvet vagy könyvtárat használhatsz alkalmazások építéséhez a megszerzett tudással.

## A socketek típusai

Minden socketnek közös API-ja van a könnyebb használhatóság érdekében. A socketek API-ját a BSD vezette be hálózati és folyamatok közötti kommunikációhoz. Ezeket ma már a UNIX és más operációs rendszerek is implementálják és használják.

A socketek közötti fő különbség, hogy egyes socketek hálózatot használnak a kommunikációhoz, mások nem. Az előbbieket hálózati/internet socketeknek (más néven Berkeley socketeknek) nevezik, az utóbbiakat UNIX domain socketeknek. A Windows a Berkeley socket API-t a Winsock könyvtárában implementálta. Ebben a bejegyzésben csak az internet socketekről fogok beszélni, nem a tipikus UNIX socketekről.

Többféle internet socket létezik, de általában csak a 4 típus valamelyikét használjuk. Szóval, mi ez a 4 típus?

### Stream socketek

Más néven SOCK_STREAM, ezek kapcsolat-orientált socketek, ahol a csomagok kézbesítése garantált. A kapcsolat-orientált egyszerűen azt jelenti, hogy a kliens és a szerver valamilyen kézfogáson keresztül logikai kapcsolatot létesít, és nyugtázzák egymást a csomagok küldésének megkezdéséhez.

Ha 2 elemet küldenél a socketen keresztül: „H, i", ugyanabban a sorrendben érkeznek meg – „H, i". A stream socketek a TCP protokollt használják az adatátvitelhez. Az adatrekordok azonban nem tartanak fenn semmilyen határt, ami azt jelenti, hogy nem tudod, hol végződik/fejeződik be az üzenet. Ennek kezelésére a küldők egy lezáró null karaktert adnak az üzeneteik végéhez, jelezve, hogy az üzenet ott végződik.

A webböngészők a HTTP protokollt használják, amely a TCP feletti absztrakció. Ez azt jelenti, hogy a webböngészők is stream socketeket használnak az oldalak letöltéséhez. Ha többet szeretnél tudni a TCP-ről, javaslom, hogy olvasd el ezt az RFC-t.

### Datagram socketek

Más néven SOCK_DRAM, ezek kapcsolat nélküli socketek, ahol az üzenetek egyáltalán nem is érkezhetnek meg (jaj). A datagram socketek az UDP protokollt használják az adatátvitelhez. Ezzel a protokollal a kliens nem létesít logikai kapcsolatot a szerverrel; helyette csak dobálja a csomagokat a szervernek, és imádkozik a kézbesítésért.

Az üzenetek sorrendje sem marad fenn, tehát ha „H, i"-t küldesz, lehet, hogy „i, H"-t kapsz. Ez rosszul hangozhat, de vannak előnyei. Mivel nem kell kapcsolatot létesítenie a szerverrel, sem nyugtáznia minden egyes csomagot, gyorsabb, mint a stream socketek. Ezt általában videóstreameléshez vagy konferenciákhoz használják, ahol néhány képkocka kiesése elfogadható, de gyors lejátszást kapsz. Az üzenethatárok is megmaradnak, mivel egyedi üzeneteket küldesz a szervernek anélkül, hogy kapcsolatot tartanál nyitva.

Több információért az UDP protokollról olvasd el ezt az RFC-t.

![Datagram/UDP dióhéjban]

### Nyers socketek

Más néven SOCK_RAW, ezekkel a socketekkel szórakoztató játszani (kivéve, ha nem tudod, mit csinálsz, természetesen). Lényegében a nyers socketek lehetővé teszik IP-csomagok küldését és fogadását bármilyen konkrét protokoll nélkül, ellentétben a stream vagy datagram socketekkel, bár használhatsz protokollt, ha szeretnél (magadnak kellene implementálnod).

Más socket típusok nem adnak hozzáférést a hasznos teherrel együtt sugárzott protokollfejlécekhez, de a nyers socketek tartalmazzák a fejléc hasznos terheket. A nyers socketek csodálatosak saját szállítási réteg protokollok építéséhez, amelyeket a kernel natívan nem támogat. Olyan útválasztási protokollokhoz is használják őket, mint az OSPF, ICMP stb.

Megpróbálhatod elolvasni ezt az RFC-t több információért az Internet Protocolról (mivel a nyers socketek ezen futnak).

### Sorba rendezett csomag socketek

Egyesek nem tekintik ezt „legitim" socketnek, és meglehetősen népszerűtlen. Más néven SOCK_SEQPACKET. Ezek kapcsolat-orientált socketek, hasonlóan a stream socketekhez (TCP). Az egyetlen különbség, hogy a seq csomagok fenntartanak egy üzenethatárt, ami sokszor nagyon hasznos lehet. Egy rekord/üzenet küldhető egy vagy több „send" művelettel, és olvasható egy vagy több „read" művelettel, de egyetlen művelet soha nem küld több mint egy rekord/üzenet részeit. Az emberek szeretik „programozóbarát" socket típusnak nevezni.

## Hogyan működik a motorháztető alatt?

Ha szeretnéd tudni, hogyan csomagolódnak be az adatok és küldődnek el más számítógépekhez, javaslom, hogy először nézd meg az OSI-modellt (írhatnék egy egész bejegyzést erről, de jobbak is léteznek). Mivel ez a bejegyzés a socketekről szól, nem fogok arról beszélni, hogyan küldik az adatokat a hálózatra több rétegen, protokollon, szerializálón, kódolón keresztül. Inkább arról szeretnék áttekintést adni, hogyan hozza létre és kezeli a (Linux) kernel a socketeket.

Amikor a socket() rendszerhívást használod (közvetlenül vagy valamilyen könyvtáron keresztül), az egy csomó belső metódust indít el. Ezek főként olyan metódusok, amelyek néhány előzetes validálást futtatnak, és lefoglalnak egy mutatót egy socket struct-hoz, amely a socketről tartalmaz információkat. A socket struct tartalmaz egy proto_ops interfészt, amely a socket által használt protokollról tartalmaz információkat. Egy sock_create nevű belső metódus kitölti a socket szükséges információit ebben a lépésben. Miután ezek a lépések készen vannak, a sock_alloc() függvény lefoglal egy tényleges socketet számunkra (egy inode-ot is lefoglal, de az unalmas). Ezután a socketek IP-családokhoz, portokhoz és egyebekhez vannak kötve (nem akarok túl mélyre menni ebben).

Most talán azon gondolkodsz, hogyan működnek ezek a socketek? Hogyan végzi a kernel ezeket a hálózati dolgokat? Valójában egészen egyszerű.

Az adatok csomagok formájában kerülnek átadásra, tehát a NIC (hálózati interfész) nem úgy kapja meg az adatokat, ahogy vannak. Inkább több csomag formájában kapja meg őket, metaadatokkal. Amikor a NIC fogad egy adatcsomagot, a kernel megszakítódik (vagy egyszerűen lekérdezi a NIC-et új adatokért). Miután a kernel fogad egy csomagot a NIC-től, dekódolja a csomagot, és megnézi a metaadatokat (a csomagfejlécekben). Ellenőrzi a forrás IP/portot, cél IP/portot stb. Ezt az információt a kernel arra használja, hogy megkeressen egy aktív socketet a memóriában, amely ugyanazt az IP/port kombinációt használja (mivel nem lehet egynél több hálózati socket ugyanazon az IP/port kombináción hallgatózni, mindegyik egyedi).

Minden socketnek vannak pufferei mind az olvasáshoz, mind az íráshoz, amelyeket „fogadási puffernek" és „írási puffernek" neveznek. Miután a kernel kitalálja, melyik sockethez tartozik a fogadott adat, beírja azt a socket olvasási pufferébe (és ott tartja). Amikor egy felhasználó/program futtatja a read() rendszerhívást, hogy adatot olvasson abból a socketből, a kernel átmásolja az adatokat a socket pufferéből a read() rendszerhívás meghívásakor megadott pufferbe (ez a felhasználó által megadott puffer lehet valamilyen lefoglalt hely vagy változó a kódodban). A „beolvasott" adatok most eltávolítódnak a socket fogadási pufferéből, hogy új adatok jöhessenek be.

A socketbe való írás hasonlóan működik. Amikor a write() rendszerhívást hívod, átmásolja az adatokat az általad megadott pufferből, és a socket írási pufferébe helyezi. A kernel most elveszi az adatokat az írási pufferből, szerializálja/kódolja (nézd meg az OSI-modellt több információért a különböző hálózati rétegekről), majd átadja a NIC-nek, amely ténylegesen elküldi az adatokat a hálózaton keresztül.

Az olvasás és írás késleltetve lehet attól, amikor a felhasználó meghívja a rendszerhívásokat, számos ok miatt, pl. a NIC elfoglalt lehet, a protokoll ablak tele van, a kernel mással van elfoglalva stb.

Írhatnék egy egész bejegyzést erről az egyetlen témáról, és mélyebbre mennék a hallgatózásban, kapcsolatok elfogadásában, backlagekben, szemantikában stb. De nem fogom ebben a bejegyzésben. Szólj, ha szeretnél egy bejegyzést ezekről a témákról :)

