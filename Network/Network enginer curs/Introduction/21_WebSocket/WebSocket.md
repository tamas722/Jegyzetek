# WebSocket vs HTTP: Melyik protokollt használd?

**William Imoh**

Ha hálózati mérnökként az első ösztönöd az, hogy WebSocketet használj, amikor egy funkció folyamatos frissítéseket igényel, anélkül hogy megértenéd, milyen kommunikációs mintára van szüksége az alkalmazásodnak, akkor lehet, hogy rosszat teszel.

A HTTP és a WebSocket egyaránt TCP-re épül, de különböző problémákat oldanak meg. A HTTP kérés-válasz modellje megbízhatóan működteti a web nagy részét, és továbbra is az alapértelmezett a hagyományos webfejlesztésben. Ugyanakkor a WebSocket egy állandó, kétirányú csatornát ad hozzá azokhoz az esetekhez, amikor a szervernek adatot kell küldenie anélkül, hogy megvárná a kérést.

## Tartalomjegyzék

- TL;DR: WebSocket vs. HTTP
- WebSocket vs HTTP: Összehasonlító táblázat
- Mi az a HTTP?
- Mi az a WebSocket?
- WebSocket vs HTTP: Fő különbségek
- Skálázhatósági kompromisszumok
- Melyiket tanuld? Döntési keretrendszer
- Összefoglalás

Ez az útmutató elmagyarázza, hogyan működik a HTTP és a WebSocket, mire alkalmas az egyes protokollok, és miért érdemes közelebbről megvizsgálni a Server-Sent Events technológiát.

## TL;DR: WebSocket vs. HTTP

### Válaszd a WebSocketet, ha...

- **Kétirányú kommunikációra van szükséged:** a kliens és a szerver egyaránt küld adatot anélkül, hogy megvárná a másik kérését.
- **Élő chatet építesz** (a Slack, a Discord és a WhatsApp Web mind WebSocketet használ az üzenetkézbesítéshez és a gépelési jelzésekhez), **többjátékos játékokat, amelyek 10 ms alatti késleltetést igényelnek**, **együttműködésen alapuló szerkesztőeszközöket** (a Google Docs és a Figma WebSocketet használ operational transformation vagy CRDT-technológiával az egyidejű szerkesztésekhez), vagy **élő kereskedési/aukciós platformokat**, ahol mindkét fél egyszerre küld és fogad eseményeket.
- **A felhasználási eseted bináris adatokat tartalmaz** (hang, videó képkockák, képek): a WebSocket natívan támogatja a bináris kereteket; a Server-Sent Events csak szöveges.
- **A lehető legalacsonyabb üzenetenkénti késleltetésre van szükséged:** a WebSocket keretek csak 2 bájtnyi overheadet hordoznak a kezdeti kapcsolat után, szemben a HTTP fejlécek több száz bájtjával minden kérésnél.

Ha ezek bármelyike illik ahhoz, amit építesz, a Network Engineer Roadmap megmutatja, hová illeszkedik a WebSocket a technikai fejlődésedben.

### Válaszd a HTTP-t, ha...

- **A funkciód kérés-válasz mintát követ:** a kliens kérdez, a szerver válaszol, és az interakció befejeződik (REST API-k, oldalbetöltések, űrlapbeküldések, fájlfeltöltések).
- **Szükséged van gyorsítótárazásra, CDN-terjesztésre vagy szabványos HTTP-infrastruktúrára:** a WebSocket kapcsolatok megkerülik a legtöbb HTTP gyorsítótárazási réteget.
- **A valós idejű igényed valójában egyirányú** (a szerver csak frissítéseket küld a kliensnek): használj Server-Sent Eventset HTTP felett a WebSocket helyett; az SSE egyszerűbb implementálni, szabványos HTTP-t használ, problémamentesen átjut a legtöbb tűzfalon és proxyn, és natívan támogatja az automatikus újracsatlakozást.
- **Ezeket építed:** élő értesítések, aktivitási hírfolyamok, telepítési állapotfrissítések (a GitHub SSE-t használ a build állapotához), hírfolyamok, részvénytickerek vagy LLM streamelési válaszok (az SSE az a protokoll, amely a ChatGPT-t és a legtöbb LLM streamelési API-t működteti).

Amikor a WebSocket által bevezetett komplexitást nem indokolja a funkció, az SSE az erősebb választás.

A legtöbb ilyen felhasználási eset nem igényel állandó kapcsolatot. A HTTP jól kezeli őket. Az API Design Roadmap lefedi, hogyan metszi ez a protokolltervezési döntéseidet.

## WebSocket vs HTTP: Összehasonlító táblázat

| | HTTP | WebSocket |
|---|---|---|
| **Kapcsolat modell** | Kérés-válasz (állapotmentes) | Állandó, teljes duplex |
| **Irány** | A kliens kezdeményezi a kéréseket a szerver felé | Kétirányú |
| **Protokoll** | HTTP/1.1, HTTP/2, HTTP/3 | HTTP kapcsolatként indul, és WebSocketre frissül (ws:// vagy wss://) |
| **Késleltetés** | Magasabb ismételt kéréseknél a kérés- és fejléc-overhead miatt | Legalacsonyabb a folyamatos kétirányú kommunikációhoz |
| **Gyorsítótárazás** | Teljes HTTP gyorsítótárazási támogatás | Nem gyorsítótárazható |
| **Tűzfal/proxy kompatibilitás** | Univerzális | Általában támogatott, bár egyes örökölt proxyk blokkolhatják a frissítéseket |
| **Automatikus újracsatlakozás** | Nem alkalmazható | Manuálisan kell implementálni |
| **Bináris támogatás** | Igen | Igen |
| **Legjobb erre** | REST API-k, oldalbetöltések, fájlátvitelek | Chat, játékok, együttműködésen alapuló szerkesztés, élő kétirányú kommunikáció |

A fenti táblázat papíron jól mutatja a WebSocketet. A gyakorlatban a választás azt jelenti, hogy feladod a HTTP gyorsítótárazást és a natív újracsatlakozást azért, hogy ezeket a komplikációkat magad kezeld. Ahhoz, hogy megértsd, miért kell ezeket a kompromisszumokat meghoznod, tudnod kell, hogyan épült a web a kommunikációra eredetileg.

Mielőtt a WebSocket szóba került, ott volt a HTTP.

## Mi az a HTTP?

A **HTTP (HyperText Transfer Protocol)** az adatcsere alapja a weben, amely 1991 óta kérés-válasz modellben működik. Minden alkalommal, amikor egy böngésző betölt egy oldalt, beküld egy űrlapot, letölt egy fájlt vagy meghív egy REST API-t, HTTP-t használ. A kliens elküld egy kérést, a szerver visszaad egy választ, és még akkor is, amikor a modern beállítások újrahasznosítják a kapcsolatokat az idő megtakarítása érdekében, minden egyes kérés izolált beszélgetésként kezdődik és végződik.

Ez a modell nagy része annak, amiért a HTTP a webalkalmazások alapértelmezettje maradt. Kiszámítható, széles körben támogatott, és szinte minden szabványos webes forgalmat panasz nélkül kezel.

![HTTP kapcsolatok]

### Hogyan működik a HTTP?

A HTTP úgy működik, mint egy beszélgetés, ahol a kliens mindig először beszél. Elküld egy kérést egy TCP kapcsolaton, a szerver feldolgozza és visszaküld egy választ, és a kapcsolat általában bezárul, kivéve ha nyitva tartásra van konfigurálva. Minden kérés hordoz egy metódust (GET, POST, PUT, DELETE) és fejléceket, amelyek megmondják a szervernek, mit szeretne a kliens és hogyan kell kezelni.

A HTTP állapotmentes. Minden kérés izolált az előzőtől. A szervernek alapértelmezés szerint nincs emlékezete a korábbi kérésekről, így minden interakció frissen kezdődik.

A HTTP/2 azzal javított ezen, hogy lehetővé tette több kérés- és válaszfolyam számára, hogy egyetlen kapcsolatot osszon meg, miközben a fejléceket tömörítette az ismétlődő overhead csökkentése érdekében. A HTTP/3 tovább ment, a TCP-t QUIC-re cserélve, hogy csökkentse a kapcsolat létesítéséhez szükséges időt.

### A HTTP előnyei

A HTTP évtizedek óta a webes kommunikáció alapértelmezettje, és ennek oka van. Kialakítása sok gyakori problémát kezel anélkül, hogy sokat kérne a fejlesztőtől.

- **Univerzális kompatibilitás:** Mindenhol működik: böngészők, mobilalkalmazások, API-k, CDN-ek, proxyk, tűzfalak és terheléselosztók mind használják. Nem kell azon gondolkodnod, hogy egy kérés átjut-e a hálózaton, vagy hogy az infrastruktúra kezelni tudja-e.
- **Beépített gyorsítótárazás:** A HTTP-t gyorsítótárazással tervezték. A válaszok gyorsítótárazhatók a böngésző, a CDN és a proxy szintjén. A Cache-Control fejlécek meghatározzák, meddig tárolódnak ezek a válaszok, és újra felhasználhatók-e. Ez csökkenti a szerver terhelését, csökkenti az infrastruktúra költségeit és felgyorsítja a válaszidőket.
- **Egyszerűség:** A kérés-válasz modell könnyen érthető. Egy REST API építése, egy harmadik féltől származó szolgáltatás integrálása vagy egy végpont hibakeresése mind ugyanazt a kiszámítható mintát követi. Már ismered, mielőtt elkezdenéd.
- **Könnyen skálázható:** A HTTP állapotmentes, így a szervereknek nem kell nyomon követniük minden klienssel való aktív kapcsolatot a kérések között. Bármely elérhető szerver fogadhatja a bejövő kérést. Ez egyszerű problémává teszi a forgalom elosztását több szerver között, ahogy a forgalom növekszik.
- **Érett biztonsági modell:** A webes biztonság a HTTP köré épül. Ez azt jelenti, hogy a HTTPS a HTTP-t TLS titkosítással kombinálja, és a termelési alkalmazások standardjává vált. Minden böngésző megköveteli, és minden hosting szolgáltató támogatja.

A kérés-válasz modell jól megoldja a legtöbb kommunikációs problémát. Ott kezd repedezni, amikor egy alkalmazásnak azonnal reagálnia kell az új adatokra, amint azok megérkeznek.

### A HTTP korlátai valós idejű funkciókhoz

A HTTP-t soha nem folyamatos, eseményvezérelt kommunikációra tervezték. A szerver nem tud adatot küldeni a kliensnek anélkül, hogy a kliens először kérne. Ez a kérés-válasz modell kemény korlátja.

Vannak módok e megkerülésére. A HTTP long polling nyitva tart egy kérést, amíg a szervernek nincs adata visszaküldeni. A HTTP streaming hasonlóan működik. Ezek a megközelítések csökkentik a szakadékot, de egyik sem ad valódi kétirányú kommunikációt, ahol a kliens és a szerver egymásnak küldenek adatot anélkül, hogy várnának.

Azokhoz a funkciókhoz, amelyek mindkét oldalt egyszerre aktívan igénylik, a HTTP-nek a saját tervezése körül kell dolgoznia. A WebSocketet azért építették, hogy megszüntesse ezt a korlátot.

Végül is, hogy ezek a korlátok meddig terjednek, attól függ, mi van a kliensed és a szervered között. Ha meg szeretnéd érteni, hogyan viselkedik a HTTP különböző hálózati infrastruktúrákon, a Network Engineer Roadmap mindent lefed.

## Mi az a WebSocket?

A **WebSocket** egy kommunikációs protokoll, amely állandó, teljes duplex csatornát biztosít egyetlen TCP kapcsolat felett. 2011-ben szabványosították RFC 6455 néven. Ellentétben a HTTP-vel, ahol a kliens mindig kezdeményez és a szerver mindig válaszol, a WebSocket lehetővé teszi, hogy bármelyik fél bármikor üzeneteket küldjön a kapcsolat létrejötte után.

### Hogyan működik a WebSocket?

#### Hogyan működik a WebSocket kézfogás?

A WebSocket kapcsolat nem valami újként indul. Egy HTTP kéréssel kezdődik. A kliens elküld egy HTTP/1.1 kérést, amely tartalmaz egy `Upgrade: websocket` fejlécet, jelezve, hogy protokollt szeretne váltani.

Ha a szerver támogatja a WebSocketet, válaszol egy `HTTP 101 Switching Protocols` üzenettel. Ekkor a kapcsolat megszűnik HTTP lenni, és WebSocket kapcsolattá válik.

Onnantól a kapcsolat állandó és kétirányú. Mind a kliens, mind a szerver bármikor küldhet adatkereteket anélkül, hogy a másik fél kérésére várna. A kapcsolat nyitva marad, amíg az egyik fél be nem zárja.

Íme egy egyszerűsített példa, amely a WebSocket kapcsolatot kezdeményező HTTP Upgrade kérést mutatja:

```http
GET /chat HTTP/1.1
Host: websocket.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

# WebSocket kézfogás és kapcsolat

Ha a szerver támogatja a WebSocketet, így válaszol:

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

Miután a szerver visszaadja a `101 Switching Protocols` választ, a kapcsolat HTTP-ről WebSocketre vált. Onnantól bármelyik fél küldhet üzeneteket anélkül, hogy a másikra várna.

A következő példa bemutatja, hogyan lehet WebSocket kapcsolatot létesíteni a böngésző WebSocket API-jával:

```javascript
const socket = new WebSocket("wss://example.com/chat");

socket.addEventListener("open", () => {
  console.log("Connection established");
  socket.send("Hello from the client");
});

socket.addEventListener("message", (event) => {
  console.log("Server says:", event.data);
});

socket.addEventListener("close", () => {
  console.log("Connection closed");
});

socket.addEventListener("error", (error) => {
  console.error("WebSocket error:", error);
});
```

Amikor az `open` esemény végrehajtódik, az azt jelenti, hogy a kézfogás befejeződött, és a kapcsolat aktív. Minden utána küldött üzenet ugyanazon a kapcsolaton keresztül halad, amíg az egyik fél be nem zárja.

A böngészők nagyon kevés információt fednek fel az `error` eseményen keresztül. Azt mondják, valami elromlott, de azt nem, hogy miért. A tényleges ok megtalálásához ellenőrizd a `close` eseményt, a böngésző fejlesztői eszközeit vagy a szervernaplókat.

## A WebSocket előnyei

A WebSocketet azért építették, hogy megoldja azokat a problémákat, amelyeket a HTTP nem tud. Minden, ami következik, ebből az egyetlen tervezési döntésből ered:

- **Teljes duplex kommunikáció**: A kommunikáció már nincs a kérés-válasz ciklushoz kötve. Bármelyik fél azonnal továbbíthatja az adatokat, amint van küldenivalója.

- **Alacsonyabb overhead**: A kezdeti kézfogás után a WebSocket eldobja azt a nehéz fejlécstruktúrát, amelyet a HTTP minden kérésnél hordoz. Ami marad, az egy minimális keret, akár 2 bájt rövid üzeneteknél, ezért marad alacsony az üzenetenkénti overhead még nagy üzenetgyakoriság mellett is.

- **Alacsony késleltetés**: Mivel a kommunikáció egy már létrehozott kapcsolaton keresztül történik, az üzenetek új HTTP kérések létrehozása nélkül cserélhetők. A kereskedési platformok, többjátékos játékok és együttműködési eszközök számára ez a különbség nem elméleti. Megmutatkozik a termékben.

- **Valós idejű push**: A szerver nem vár arra, hogy a kliens frissítéseket kérjen. A szerver azonnal küldi az adatokat, amint elérhetők. Ez kiküszöböli a pollingot és a vele járó késleltetést.

- **Nincs kapcsolat-örvény**: A HTTP minden cserénél TCP kapcsolatokat nyit és zár. A WebSocket egyet tart életben a munkamenet időtartamára. Ez eltávolítja az új kapcsolat létesítésének CPU- és sávszélesség-költségét minden alkalommal, amikor adatot kell mozgatni.

- **Kontrollált kapcsolat életciklus**: A HTTP-vel a kapcsolat életciklusa a válasz elküldésének pillanatában véget ér. Nincs jelzés, csak egy lezárt csere. A WebSocket másképp kezeli ezt. Amikor bármelyik fél készen áll a bontásra, küld egy Close Frame-et. Mindkét fél nyugtázza, a még úton lévő adatok feldolgozása befejeződik, és a kapcsolat bezárul anélkül, hogy bármit elveszítene a munkamenet közben.

## A WebSocket korlátai

A WebSocket valódi problémákat old meg, amelyeket a HTTP nem tud felvenni a versenybe. Újakat is bevezet; amit cserébe kér, érdemes tudni, mielőtt elkötelezed magad mellette.

- **Nincs beépített újracsatlakozás**: Amikor egy WebSocket kapcsolat megszakad, semmi sem kapja el helyetted. A kliensnek magának kell észlelnie a hibát és újracsatlakoznia. A Server-Sent Events (SSE) megoldja ezt helyetted, a WebSocket viszont nem.

- **Nem gyorsítótárazható**: A WebSocket kapcsolatokat nem lehet CDN-ek vagy proxyk gyorsítótárazni. Minden üzenet végponttól végpontig utazik, minden alkalommal. Az olyan tartalomhoz, amely nem változik gyakran, ez olyan overhead, amelyet a HTTP nem hordozna.

- **Tervezésileg állapotfüggő**: A HTTP nem törődik azzal, melyik szerver kezeli a kérést. A WebSocket igen. A szervernek fenn kell tartania a kapcsolat állapotát minden aktív klienshez, ami azt jelenti, hogy az A szerverhez csatlakozott kliensnek A szerveren kell maradnia. Ez vagy sticky sessionöket igényel a terheléselosztón, vagy egy megosztott üzenetbrókert, mint a Redis Pub/Sub, hogy az üzeneteket a szerverek között eloszthassa.

- **Nagyobb implementációs komplexitás**: A WebSocket rád hagyja a termelési problémák megoldását. Egy kapcsolat némán elhallgathat anélkül, hogy bármelyik fél tudná, ezért ping-pong kereteket kell implementálnod az elavult kapcsolatok észleléséhez és bezárásához, mielőtt erőforrásokat pazarolnának. Amikor egy kapcsolat megszakad, nem áll helyre magától, ezért neked kell megírni az újracsatlakozási logikát és az exponenciális visszalépést. A WebSocketnek nincs beépített üzenetkövetése sem, így a kliens leválasztása közben küldött üzenetek elvesznek, hacsak nem építész egy sorba állító réteget, amely elkapja és újrajátssza őket. Semmi sem jön ingyen.

A tűzfal- és proxy-kompatibilitás ma már megoldott probléma. A legtöbb modern infrastruktúra problémamentesen támogatja a WebSocket kapcsolatokat. A különböző szervezetek örökölt vállalati környezetei a kivétel. Az ilyen környezetekben lévő proxyk a kérés-válasz forgalomra vannak optimalizálva, és lecsupaszítják az Upgrade fejlécet, vagy megszüntetik az állandó kapcsolatokat, amikor valamit látnak, ami nem illik a mintába.

A `wss://` használata javítja a kompatibilitást, mivel a WebSocket forgalom TLS felett fut, és a legtöbb hálózaton szabványos HTTPS-ként jut át. Ha vállalati ügyfeleknek építesz, teszteld a tényleges hálózati infrastruktúrájuk mögött a telepítés előtt.

> **AI Tutor**
> Vezess végig a WebSocket kézfogás működésén lépésről lépésre.

## WebSocket vs HTTP: Fő különbségek

A HTTP és a WebSocket egyaránt TCP felett fut. Utána a hasonlóságok megszűnnek. Ami elválasztja őket, az minden, ami a kapcsolat létrejötte után történik. Íme, hol mutatkozik meg:

### Kapcsolat modell: állandó vs kérés-válasz

A HTTP diszkrét (minden interakciót külön beszélgetésként kezel). A kliens kérdez, a szerver válaszol, és a kommunikációs csere véget ér. A szervernek nincs módja arra, hogy saját maga küldjön adatot, hacsak a kliens nem kér.

A WebSocket egy hosszú életű kapcsolatot hoz létre, amelyet mindkét fél használhat a munkamenet során. Onnantól az adatcsere inkább eseményvezérelté válik, mint kérésvezérelté.

Ez a megkülönböztetés egyes funkciók esetében jobban számít, mint másoknál. A beérkező postafiók 30 másodpercenkénti ellenőrzése jól működik HTTP felett. Abban a pillanatban, amikor gépelési jelzéseket, olvasási visszaigazolásokat és élő üzeneteket adsz ugyanahhoz a képernyőhöz, a kérés-válasz modell késleltetést kezd mutatni.

### Overhead és teljesítmény

A HTTP minden kérésnél metaadat-fejléceket hordoz, ami több száz bájt. Alacsony gyakoriságú interakciók esetén ez nem probléma. De a nagy gyakoriságú cseréknél, mint a kereskedési platformok vagy a többjátékos játékok, ez az overhead gyorsan összeadódik.

A WebSocket eltávolítja ezen overhead nagy részét a kézfogás után. Ami marad, az egy minimális keretstruktúra, ezért marad alacsony az üzenetenkénti költség még nagy üzenetmennyiség mellett is.

A fogás az, hogy a WebSocket minden aktív klienshez nyitva tart egy kapcsolatot. A szerver memóriája allokálva marad, még akkor is, ha nem küldenek üzeneteket. Olyan munkaterheléseknél, ahol több ezer egyidejű, de tétlen kapcsolat van, ez összeadódik. A HTTP a válasz elküldésének pillanatában felszabadítja a kapcsolat erőforrásait, ami tisztábban skálázódik ezekhez a mintákhoz.

Légy tudatában annak, hogy a teljesítmény a munkaterhelésedtől függ. A WebSocket jó a nagy gyakoriságú kétirányú cserékhez. A HTTP/2 és az SSE hasonló átviteli teljesítményt érhet el az állandó kapcsolatok kezelésének memóriaköltsége nélkül.

### Skálázhatósági kompromisszumok

A kérések függetlenek egymástól, így bármely szerver kezelhet bármely kérést koordináció nélkül. Amikor az interakció véget ér, a kapcsolat felszabadul, és nem marad meg kérésállapot.

A WebSocket ezt megváltoztatja, mert a kapcsolata állapotfüggő. Az A szerverhez csatlakozott kliensnek A szerveren kell maradnia. A B szerveren érkező üzenet soha nem éri el azt a klienst. Ezért van szükségük a WebSocket rendszereknek sticky sessionökre a terheléselosztón vagy egy megosztott üzenetbrókerre, mint a Redis Pub/Sub, hogy az üzeneteket a szerverek között irányítsák.

Egyetlen szerveres beállításban az állapotfüggő kapcsolatok kezelése egyszerű. Több millió egyidejű kapcsolatnál szándékos architektúrát igényel: munkamenet-állapot kezelés, elosztott üzenetszórás és kapcsolat életciklus monitorozás.

### Biztonság

Mindkét protokoll támogatja a szállítási rétegbeli titkosítást. Használj HTTPS-t a HTTP-hez, WSS-t a WebSockethez, és soha ne szállíts termelési alkalmazást titkosítatlan `ws://` kapcsolaton. Ezt javítsd ki mindenekelőtt.

A WebSocket HTTP-ként indul, de nem marad ott. A frissítés után sok HTTP biztonsági ellenőrzés már nem érvényes a folyamatos kommunikációra. A Content Security Policy szabályok és a CORS preflight ellenőrzések nem ugyanúgy működnek, mint a normál HTTP kéréseknél. A hitelesítést az alkalmazási rétegben kell kezelni, gyakran egy token segítségével a kapcsolat URL-jében vagy az első üzenetben, amelyet a kapcsolat megnyílása után küldenek.

Ez nem jelenti azt, hogy a WebSocket kevésbé biztonságos. Csak másképp kezeli a biztonságot, és ezt a különbséget figyelembe kell venni a tervezési szakaszban.

> **AI Tutor**
> Magyarázd el a WebSocket és a HTTP polling közötti különbséget egy konkrét példával.

## Melyiket tanuld? Döntési keretrendszer

A megfelelő protokoll a funkciód kommunikációs mintájától, a csapatod operatív komfortjától és az infrastruktúrádtól függ. Íme, hogyan gondolkodj a döntésről.

| Funkció vagy felhasználási eset | Ajánlott protokoll |
|---|---|
| REST API vagy szabványos weboldal | HTTP |
| Fájlfeltöltés vagy -letöltés | HTTP |
| Irányítópult rendszeres adatfrissítésekkel | HTTP |
| Élő chat, gépelési jelzések, jelenléti állapot | WebSocket |
| Többjátékos játék valós idejű játékosműveletekkel | WebSocket |
| Együttműködésen alapuló dokumentumszerkesztés | WebSocket |
| Élő kereskedési vagy aukciós platform | WebSocket |

### Mikor a WebSocket a helyes választás

A legvilágosabb jel az, amikor mind a kliensnek, mind a szervernek adatot kell küldenie ugyanazon munkamenet során anélkül, hogy a másik fél kérésére várnának. Ez az a fő felhasználási eset, amelyre a WebSocketet építették.

Másodlagos jelek gyakran megjelennek mellette: a késleltetés kemény követelmény, a bináris adatoknak, például hang- vagy videó képkockáknak mindkét irányban mozogniuk kell, vagy maga a munkamenet olyan állapotot hordoz, amely számít az üzenetek között, ahogy a többjátékos játékokban és az együttműködésen alapuló szerkesztőeszközökben.

Amikor ezek a követelmények együtt jelennek meg, a WebSocket a megfelelő protokoll, amelyet használni szeretnél.

### Mikor elég a HTTP

Ha a kliens kezdeményezi az összes cserét és a szerver válaszol, a szabványos HTTP az, amit fontolóra kell venned. Nincs szükséged állandó kapcsolatra, és nem kell kezelned a kapcsolat állapotát vagy további infrastruktúrát építened hozzá.

### Karrier és tanulási kontextus

A HTTP alapvető. Minden webfejlesztőnek meg kell értenie a kérés-válasz kommunikációt, az állapotkódokat, a fejléceket, a metódusokat és a gyorsítótárazást. Ezek a fogalmak szinte minden általad épített alkalmazáson keresztülmennek, és részletesen tárgyalja őket az API Design Roadmap.

A WebSocket akkor válik fontossá, amikor a backend és
