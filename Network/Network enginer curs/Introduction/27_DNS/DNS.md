# Mi az a DNS?

A **Domain Name System (DNS)** az internet telefonkönyve. Ember által olvasható domainneveket fordít le géppel olvasható IP-címekre.


## Mi az a DNS?

A **Domain Name System (DNS)** az internet telefonkönyve. Az emberek domainneveken keresztül érik el az online információkat, mint például a nytimes.com vagy az espn.com. A webböngészők Internet Protocol (IP) címeken keresztül kommunikálnak. A DNS lefordítja a domainneveket IP-címekre, hogy a böngészők be tudják tölteni az internetes erőforrásokat.

Az internethez csatlakoztatott minden eszköz rendelkezik egy egyedi IP-címmel, amelyet más gépek az eszköz megtalálásához használnak. A DNS szerverek megszüntetik annak szükségességét, hogy az emberek megjegyezzék az olyan IP-címeket, mint a 192.168.1.1 (IPv4-ben), vagy a bonyolultabb újabb alfanumerikus IP-címeket, mint a 2400:cb00:2048:1::c629:d7a2 (IPv6-ban).

![DNS - nagyító vizsgálja az IP-címeket, megtalálja a www.example.com-ot]

## Hogyan működik a DNS?

A DNS-feloldás folyamata magában foglalja egy hostnév (például www.example.com) számítógép-barát IP-címmé (például 192.168.1.1) való átalakítását. Az interneten minden eszköz kap egy IP-címet, és ez a cím szükséges a megfelelő internetes eszköz megtalálásához – ahogy egy utcacím egy adott otthon megtalálásához szolgál. Amikor egy felhasználó be szeretne tölteni egy weboldalt, fordítást kell végrehajtani a felhasználó által a webböngészőbe beírt (example.com) és a example.com weboldal megtalálásához szükséges gépbarát cím között.

Annak érdekében, hogy megértsük a DNS-feloldás mögötti folyamatot, fontos megismerni a különböző hardverkomponenseket, amelyeken egy DNS-lekérdezésnek át kell haladnia. A webböngésző számára a DNS lookup „a színfalak mögött" történik, és a kezdeti kéréstől eltekintve nem igényel beavatkozást a felhasználó számítógépétől.

**Jelentés**
*2026-os biztonsági jelzések jelentés*
[Szerezd meg a jelentést]

### 4 DNS szerver vesz részt egy weboldal betöltésében:

- **DNS recursor** – A recursor úgy gondolható el, mint egy könyvtáros, akit megkérnek, hogy keressen meg egy adott könyvet valahol a könyvtárban. A DNS recursor egy szerver, amelyet arra terveztek, hogy lekérdezéseket fogadjon kliensgépektől olyan alkalmazásokon keresztül, mint a webböngészők. Általában a recursor felelős további kérések küldéséért a kliens DNS-lekérdezésének kielégítése érdekében.
- **Root nameserver** – A root szerver az első lépés az ember által olvasható hostnevek IP-címekké való fordításában (feloldásában). Úgy gondolható el, mint egy könyvtári index, amely különböző könyvespolcokra mutat – általában más, pontosabb helyekre mutató referenciaként szolgál.
- **TLD nameserver** – A top level domain szerver (TLD) úgy gondolható el, mint egy adott könyvespolc a könyvtárban. Ez a nameserver a következő lépés egy adott IP-cím keresésében, és egy hostnév utolsó részét tárolja (az example.com esetében a TLD szerver a „com").
- **Authoritative nameserver** – Ez a végső nameserver úgy gondolható el, mint egy szótár a könyvespolcon, amelyben egy adott név lefordítható a definíciójára. Az authoritative nameserver az utolsó állomás a nameserver lekérdezésben. Ha az authoritative nameserver hozzáfér a kért rekordhoz, visszaadja a kért hostnév IP-címét a DNS Recursornak (a könyvtárosnak), amely az eredeti kérést küldte.

**Gyors és biztonságos DNS**
*Ingyenes DNS bármely Cloudflare csomaggal*
[Kezdje ingyen]

## Mi a különbség az authoritative DNS szerver és a recursive DNS resolver között?

Mindkét fogalom a DNS-infrastruktúra szerves részét képező szerverekre (szervercsoportokra) utal, de mindegyik más szerepet tölt be, és a DNS-lekérdezés folyamatában más helyen található. A különbségre úgy gondolhatunk, hogy a recursive resolver a DNS-lekérdezés elején van, az authoritative nameserver pedig a végén.

### Recursive DNS resolver

A recursive resolver az a számítógép, amely válaszol a kliens rekurzív kérésére, és időt szán a DNS-rekord felkutatására. Ezt úgy teszi, hogy sorozatos kéréseket küld, amíg el nem éri a kért rekord authoritative DNS nameserverét (vagy időtúllépés vagy hiba esetén hibát ad vissza, ha nem talál rekordot). Szerencsére a recursive DNS resolvereknek nem mindig kell több kérést küldeniük a kliens válaszadásához szükséges rekordok felkutatásához; a gyorsítótárazás egy adatmegőrzési folyamat, amely segít lerövidíteni a szükséges kéréseket azáltal, hogy a kért erőforrásrekordot korábban szolgálja ki a DNS lookup során.

![DNS Record Request Sequence - DNS Recursive Resolver kapja a kérést a klienstől]

### Authoritative DNS server

Egyszerűen fogalmazva, egy authoritative DNS szerver egy olyan szerver, amely ténylegesen tárolja és felelős a DNS erőforrásrekordokért. Ez a szerver a DNS lookup lánc alján, amely a lekérdezett erőforrásrekorddal válaszol, végső soron lehetővé téve a kérést küldő webböngésző számára, hogy elérje a weboldal vagy más webes erőforrások eléréséhez szükséges IP-címet. Egy authoritative nameserver saját adataiból ki tudja elégíteni a lekérdezéseket anélkül, hogy más forrást kellene lekérdeznie, mivel ez bizonyos DNS-rekordok végső igazságforrása.

![DNS Record Request Sequence - DNS lekérdezés eléri a cloudflare.com authoritative nameserverét]

Érdemes megemlíteni, hogy azokban az esetekben, amikor a lekérdezés egy aldomainre vonatkozik, például foo.example.com vagy blog.cloudflare.com, egy további nameserver kerül a sorozatba az authoritative nameserver után, amely az aldomain CNAME rekordjának tárolásáért felelős.

![DNS Record Request Sequence - DNS lekérdezés a blog.cloudflare.com aldomain CNAME rekordjához]

Van egy kulcsfontosságú különbség a sok DNS-szolgáltatás és a Cloudflare által nyújtott szolgáltatás között. A különböző DNS recursive resolverek, mint a Google DNS, az OpenDNS és az olyan szolgáltatók, mint a Comcast, mind adatközponti telepítéseket tartanak fenn DNS recursive resolverekből. Ezek a resolverek gyors és egyszerű lekérdezéseket tesznek lehetővé optimalizált, DNS-re optimalizált számítógépes rendszerek klaszterein keresztül, de alapvetően különböznek a Cloudflare által hosztolt nameserverektől.

A Cloudflare infrastruktúra-szintű nameservereket tart fenn, amelyek az internet működésének szerves részét képezik. Egy kulcsfontosságú példa az f-root szerverhálózat, amelynek hosztolásáért a Cloudflare részben felelős. Az F-root az egyik root szintű DNS nameserver infrastruktúra-komponens, amely felelős az internet napi milliárdnyi kéréséért. Az Anycast hálózatunk egyedülálló helyzetbe hoz minket, hogy nagy mennyiségű DNS-forgalmat kezeljünk szolgáltatáskiesés nélkül.

## Mik a DNS lookup lépései?

A legtöbb esetben a DNS arról szól, hogy egy domainnevet lefordítanak a megfelelő IP-címre. Hogy megtanuljuk, hogyan működik ez a folyamat, érdemes követni a DNS lookup útját, ahogy a webböngészőtől a DNS lookup folyamatán keresztül visszatér. Nézzük meg a lépéseket.

> **Megjegyzés:** A DNS lookup információ gyakran gyorsítótárazva van vagy helyben a lekérdező számítógépben, vagy távolról a DNS-infrastruktúrában. Általában 8 lépés van egy DNS lookup során. Amikor a DNS-információ gyorsítótárazva van, a lépések kimaradnak a DNS lookup folyamatából, ami gyorsabbá teszi azt. Az alábbi példa mind a 8 lépést felvázolja, amikor nincs semmi gyorsítótárazva.

### A 8 lépés egy DNS lookup során:

1. A felhasználó beírja az „example.com"-ot egy webböngészőbe, és a lekérdezés az internetre kerül, és egy DNS recursive resolver fogadja.
2. A resolver ezután lekérdez egy DNS root nameservert (.).
3. A root szerver ezután válaszol a resolvernek egy Top Level Domain (TLD) DNS szerver (például .com vagy .net) címével, amely a domainjei információit tárolja. Amikor az example.com-ot keressük, a kérésünk a .com TLD felé irányul.
4. A resolver ezután kérést küld a .com TLD-nek.
5. A TLD szerver ezután válaszol a domain nameserverének, az example.com-nak az IP-címével.
6. Végül a recursive resolver lekérdezést küld a domain nameserverének.
7. Az example.com IP-címe ezután visszakerül a resolverhez a nameservertől.
8. A DNS resolver ezután válaszol a webböngészőnek az eredetileg kért domain IP-címével.

Miután a DNS lookup 8 lépése visszaadta az example.com IP-címét, a böngésző képes a weboldal kérésére:

9. A böngésző HTTP kérést küld az IP-címre.
10. Az azon az IP-n lévő szerver visszaadja a weboldalt, amely megjelenik a böngészőben (10. lépés).

![A DNS lookup és weboldal-lekérdezés 10 lépésének diagramja]

## Mi az a DNS resolver?

A DNS resolver az első állomás a DNS lookup során, és felelős a kezdeti kérést küldő kliens kezeléséért. A resolver indítja el a lekérdezések sorozatát, amely végső soron egy URL szükséges IP-címmé való fordításához vezet.

> **Megjegyzés:** Egy tipikus, nem gyorsítótárazott DNS lookup rekurzív és iteratív lekérdezéseket egyaránt tartalmaz.

Fontos különbséget tenni a rekurzív DNS lekérdezés és a rekurzív DNS resolver között. A lekérdezés a DNS resolverhez intézett kérésre utal, amely megköveteli a lekérdezés feloldását. A DNS recursive resolver az a számítógép, amely elfogad egy rekurzív lekérdezést, és feldolgozza a választ a szükséges kérések elküldésével.

![DNS rekurzív lekérdezés a DNS klienstől a DNS recursive resolverhez]

## Milyen típusú DNS lekérdezések vannak?

Egy tipikus DNS lookup során három típusú lekérdezés fordul elő. Ezen lekérdezések kombinációjának használatával a DNS-feloldás optimalizált folyamata csökkentheti a megtett távolságot. Ideális esetben a gyorsítótárazott rekordadatok elérhetők, lehetővé téve a DNS nameserver számára egy nem rekurzív lekérdezés visszaadását.

### 3 típusú DNS lekérdezés:

- **Rekurzív lekérdezés** – Egy rekurzív lekérdezésben a DNS kliens megköveteli, hogy egy DNS szerver (általában egy DNS recursive resolver) vagy a kért erőforrásrekorddal, vagy hibaüzenettel válaszoljon a kliensnek, ha a resolver nem találja a rekordot.
- **Iteratív lekérdezés** – Ebben az esetben a DNS kliens megengedi, hogy egy DNS szerver a lehető legjobb választ adja vissza. Ha a lekérdezett DNS szerver nem talál egyezést a lekérdezés nevére, egy referralt ad vissza egy DNS szerverhez, amely a domain namespace alacsonyabb szintjéért authoritative. A DNS kliens ezután lekérdezést küld a referral címére. Ez a folyamat további DNS szerverekkel folytatódik a lekérdezési láncban, amíg hiba vagy időtúllépés nem történik.
- **Nem rekurzív lekérdezés** – Ez általában akkor fordul elő, amikor egy DNS resolver kliens olyan rekordot kérdez le egy DNS szervertől, amelyhez hozzáfér, akár azért, mert authoritative a rekordhoz, akár azért, mert a rekord a gyorsítótárában található. Általában egy DNS szerver gyorsítótárazza a DNS rekordokat, hogy megakadályozza a további sávszélesség-fogyasztást és az upstream szerverek terhelését.

## Mi az a DNS gyorsítótárazás? Hol történik a DNS gyorsítótárazás?

A gyorsítótárazás célja az adatok ideiglenes tárolása egy olyan helyen, amely javítja az adatkérések teljesítményét és megbízhatóságát. A DNS gyorsítótárazás magában foglalja az adatok tárolását a kérést küldő klienshez közelebb, hogy a DNS lekérdezés korábban feloldható legyen, és a DNS lookup láncban további lekérdezések elkerülhetők legyenek, ezáltal javítva a betöltési időket és csökkentve a sávszélesség/CPU fogyasztást. A DNS adatok különböző helyeken gyorsítótárazhatók, amelyek mindegyike egy time-to-live (TTL) által meghatározott ideig tárolja a DNS rekordokat.

### Böngésző DNS gyorsítótárazás

A modern webböngészők alapértelmezés szerint úgy vannak tervezve, hogy egy meghatározott ideig gyorsítótárazzák a DNS rekordokat. A cél itt nyilvánvaló; minél közelebb történik a DNS gyorsítótárazás a webböngészőhöz, annál kevesebb feldolgozási lépést kell végrehajtani a gyorsítótár ellenőrzéséhez és a helyes kérések küldéséhez egy IP-címre. Amikor egy DNS rekordra vonatkozó kérés érkezik, a böngésző gyorsítótára az első hely, ahol a kért rekordot ellenőrzik.

A Chrome-ban a DNS gyorsítótár állapotát a `chrome://net-internals/#dns` oldalon tekintheted meg.

### Operációs rendszer (OS) szintű DNS gyorsítótárazás

Az operációs rendszer szintű DNS resolver a második és egyben utolsó helyi állomás, mielőtt egy DNS lekérdezés elhagyja a gépedet. Az operációs rendszeredben e lekérdezés kezelésére tervezett folyamatot általában „stub resolver"-nek vagy DNS kliensnek nevezik. Amikor egy stub resolver kérést kap egy alkalmazástól, először ellenőrzi a saját gyorsítótárát, hogy megvan-e a rekord. Ha nincs, akkor egy DNS lekérdezést küld (rekurzív jelzővel beállítva), a helyi hálózaton kívülre, az internetszolgáltatóban (ISP) lévő DNS recursive resolverhez.

Amikor az ISP-n belüli recursive resolver DNS lekérdezést kap, mint minden korábbi lépésnél, ez is ellenőrzi, hogy a kért host-IP-cím fordítás már tárolva van-e a helyi megőrzési rétegében.

A recursive resolvernek további funkciói is vannak a gyorsítótárában lévő rekordtípusoktól függően:

- Ha a resolvernek nincsenek A rekordjai, de vannak NS rekordjai az authoritative nameserverekhez, akkor közvetlenül azokat a nameservereket kérdezi le, megkerülve a DNS lekérdezés több lépését. Ez a rövidítés megakadályozza a root és .com nameserverektől való lekérdezéseket (az example.com keresésünkben), és segít a DNS lekérdezés feloldásának gyorsabb megtörténésében.
- Ha a resolvernek nincsenek NS rekordjai, akkor lekérdezést küld a TLD szervereknek (esetünkben a .com), kihagyva a root szervert.
- Abban a valószínűtlen esetben, ha a resolvernek nincsenek a TLD szerverekre mutató rekordjai, akkor a root szervereket kérdezi le. Ez az esemény általában egy DNS gyorsítótár kiürítése után következik be.

*Tudj meg többet arról, mi különbözteti meg a Cloudflare DNS-t más DNS-szolgáltatóktól.*
