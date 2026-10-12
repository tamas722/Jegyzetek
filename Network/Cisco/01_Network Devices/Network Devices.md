# Tanfolyam – Bevezetés

Üdvözöllek a teljes CCNA tanfolyamon. Mivel ez az első anyag, engedd meg, hogy bemutassam a kurzust. A cél az, hogy ez egy hiánytalan, 100%-ban ingyenes CCNA felkészítő legyen, amely mindent tartalmaz, amire a vizsgán szükséged lesz.

## A kurzus tartalma

- Előadások, amelyek lefedik a Cisco hivatalos vizsgatémáit.
- Gyakorló kvízek minden lecke anyagából.
- Tanulókártyák (flashcardok), amelyek segítenek az ismétlésben.
- Gyakorló laborok a Cisco hálózatszimulációs szoftverével, a Packet Tracerrel.
- A kurzus végén több gyakorló vizsga a felkészüléshez.

Bár a tanfolyam a hivatalos vizsgakövetelményekre épül, extra információkat is megosztok, hogy általánosságban is jobban megértsd a hálózatokat, ami a vizsgatémák elsajátítását is megkönnyíti.

## Kinek szól ez a tanfolyam?

- Mindenkinek, aki le akarja tenni a CCNA 200-301-es vizsgát (amely 2020. február 24-én debütált).
- Mindenkinek, aki tanulni szeretne a számítógép-hálózatokról, még akkor is, ha nem célja a vizsga megszerzése.

**Előfeltétel:** Csak alapvető számítógépes ismeretekre van szükség. Nem kell előzetes hálózati vagy programozói tudás, az alapokat a kurzus elején tisztázzuk.

## Mi az a hálózat (Network)?

A Wikipédia meghatározása szerint: "A számítógép-hálózat egy olyan digitális telekommunikációs hálózat, amely lehetővé teszi a csomópontok (node-ok) számára az erőforrások megosztását."

Ahhoz, hogy ezt megértsük, tisztáznunk kell, mik is azok a node-ok (csomópontok). A hálózatokban különböző típusú csomópontok (eszközök) találhatók:

- **Router:** Valószínűleg hallottad már a szót, a funkcióját később részletezzük.
- **Switch:** Más funkciót lát el, mint a router, de sok mindenben hasonlít rá.
- **Firewall (Tűzfal):** Valószínűleg a te gépeden is fut egy szoftveres firewall. A nagy hálózatok azonban dedikált hardveres hálózati firewall eszközöket használnak a védelemre.
- **Server:** Ha ezt az anyagot olvasod/nézed, az adatokat egy internetes server biztosítja számodra.
- **Client:** Azt az eszközt (okostelefon, laptop, asztali PC), amit épp használsz, client-nek hívjuk.

> **Megjegyzés:** A klienseket és a szervereket összefoglaló néven end host-oknak (végpontoknak) is nevezik.

## Hálózat építése az alapoktól

Képzeljünk el két számítógépet (PC1 és PC2). Önmagukban nem alkotnak hálózatot, de amint összekötjük őket egy kábellel, létrejön egy egyszerű hálózat. Ezzel a kapcsolattal a két node már képes erőforrásokat megosztani, lényegében "beszélgetni" egymással.

### Client (Kliens)

Client lehet egy Windows-t futtató asztali PC vagy laptop, egy macOS-t futtató iMac, vagy egy iOS-t futtató iPhone.

**Definíció:** A client egy olyan eszköz, amely egy server által elérhetővé tett szolgáltatáshoz fér hozzá.

### Server (Szerver)

Ha a server szót hallod, talán nagy, erős IBM vagy DELL gépekre gondolsz, amik adatközpontokban sorakoznak. Valóban léteznek ilyenek, de valójában bármilyen client (pl. egy PC) is funkcionálhat server-ként.

**Definíció:** A server egy olyan eszköz, amely funkciókat vagy szolgáltatásokat nyújt a client-ek számára.

### Példák a Client-Server kapcsolatra

- **Fájlmegosztás két PC között:** Ha PC1 elkér egy 'image.jpg' fájlt PC2-től, és PC2 elküldi azt, akkor PC1 a szolgáltatást kérő client, PC2 pedig a szolgáltatást nyújtó server.
- **Videó streamelés:** Amikor videót nézel, a számítógéped (a client) kérést küld a YouTube server-ének. Az adatok az Interneten keresztül érkeznek meg hozzád (az Internetet a hálózati ábrákon gyakran egy felhővel jelölik a komplexitása miatt). A YouTube folyamatos adatfolyamot (streamet) küld, amíg meg nem nézed a videót.
- **Fájlküldés telefonról:** Ha a barátod elküld egy videót iPhone-on AirDrop segítségével, akkor a te telefonod a client (amely kéri a fájlt), a barátod telefonja pedig a server (amely biztosítja azt).

> **Fontos:** Ugyanaz az eszköz bizonyos helyzetekben lehet client, máskor pedig server.

## Hálózati Eszközök Részletesen

Ha egy nagyvállalati (Enterprise) hálózatot építünk fel – mondjuk egy New York-i és egy tokiói irodával –, az end host-okat (PC-ket, szervereket) nem közvetlenül egymáshoz kötjük. Ehelyett a kapcsolatokat egy switch nevű eszközben fogjuk össze.

### Switches (Switchek)

Egy tipikus nagyvállalati switch-en (például a Cisco Catalyst 9200 vagy 3650 modelleken) rengeteg port (interfész) található, általában 24 vagy több, ahová a PC-ket és szervereket csatlakoztathatjuk.

**Funkció:** A switch-ek feladata a forgalom továbbítása egy LAN-on (Local Area Network – Helyi hálózat) belül. Például az egy irodában, egy emeleten vagy az otthoni hálózatodon lévő gépek egy azonos LAN-t alkotnak, és a switch segítségével kommunikálnak.

**Korlát:** A switch-ek önmagukban nem biztosítanak kapcsolatot a különböző LAN-ok között, sem pedig az Internet felé. Erre egy másik eszközre van szükség.

### Routers (Routerek)

Ahhoz, hogy a New York-i iroda LAN-ja kommunikálni tudjon a tokiói iroda LAN-jával az Interneten keresztül, a switch-eket router-ekhez kell csatlakoztatni (például Cisco ISR 1000, 4000 vagy 900 széria).

**Működés:** Ha egy New York-i PC adatot kér egy tokiói szervertől, a kérés a helyi switch-en át a New York-i router-hez kerül, ami átirányítja azt az Interneten keresztül a tokiói router-hez, majd a tokiói switch-en át a szerverhez. A válasz ugyanezen az útvonalon tér vissza.

**Jellemzők:** A switch-ekkel ellentétben a router-eknek viszonylag kevés hálózati interfészük (portjuk) van. Amíg a switch a LAN-on belül irányít, a router a hálózatok között (így az Interneten át) továbbítja az adatokat.

### Firewalls (Tűzfalak)

Mivel az Interneten támadók is leselkedhetnek a céges adatokra, a hálózatot védeni kell. Bár a router-ek is ellátnak alapvető biztonsági feladatokat, a valódi védelmet a firewall adja.

**Funkció:** A hálózati firewall egy olyan dedikált hardvereszköz, amely a konfigurált biztonsági szabályok (rules) alapján ellenőrzi és szabályozza a hálózatba be- és kilépő forgalmat. Megengedheti a szabályos kommunikációt az irodák között, de blokkolja a támadók behatolási kísérleteit.

**Elhelyezés:** A firewall elhelyezhető a router-en "kívül" és "belül" is, sőt, akár mindkét helyen egyszerre.

**Típusok:**

- **Next-generation firewall (Újgenerációs tűzfal):** A modern firewall-ok (mint a Cisco klasszikus ASA 5500-X szériájának modernizált változatai, vagy a Firepower 2100) már fejlett, modern szűrési funkciókkal rendelkeznek, például IPS-sel (Intrusion Prevention System – Behatolásmegelőző rendszer).
- **Host-based firewall (Gazdagép-alapú tűzfal):** Szemben a hálózati hardvereszközökkel, ezek olyan szoftverek, amelyek közvetlenül a client gépeken (pl. a te PC-den) futnak. Egy biztonságos hálózatban a hardveres firewall mellett minden PC-n futnia kell egy szoftveres firewall-nak is, mint extra védelmi vonal.

## Kvíz (Tudáspróba)

Gondold végig a kérdéseket, mielőtt elolvasod a megoldásokat!

### 1. Kérdés

A céged hálózati hardvert szeretne vásárolni, amelyhez a részlegeden lévő 30 PC-t csatlakoztathatják. Milyen típusú hálózati eszköz a megfelelő?

- A) Router
- B) Firewall
- C) Switch
- D) Server

**Helyes válasz:** C (Switch)

**Magyarázat:** A router hálózatok közötti forgalomirányításra való, és nincs is rajta ennyi port. A firewall szűrésre való, szintén kevés porttal. A server maga is egy end host, nem összekötő eszköz. A switch viszont pont arra való, hogy egy LAN-on belül sok end host-ot kössön össze, és rendelkezik is ehhez elegendő interfésszel.

### 2. Kérdés

Egy videófájlt kaptál a barátod Apple iPhone-járól AirDrop segítségével. Milyen szerepet töltött be az ő iPhone-ja ebben a folyamatban?

- A) Server
- B) Client
- C) LAN (Local Area Network)

**Helyes válasz:** A (Server)

**Magyarázat:** A te telefonod a client, mert az kérte a szolgáltatást. Egy eszköz önmagában nem LAN, csak a része lehet annak. Mivel a barátod telefonja nyújtotta a fájlt (a szolgáltatást), ezért server-ként funkcionált.

### 3. Kérdés

Milyen szerepet tölt be a számítógéped vagy az okostelefonod, miközben ezt a tananyagot nézed/olvasod?

- A) Server
- B) End host
- C) Client

**Helyes válasz:** C (Client)

**Magyarázat:** A te eszközöd fogadja a szolgáltatást, nem nyújtja, így nem server. Bár az eszközöd valóban egy end host, ez a kifejezés nem írja le a jelenlegi funkcióját (hiszen a szerverek is end hostok). Mivel a YouTube (vagy az adott oldal) szervereitől kapod az adatot, a géped client-ként működik.

### 4. Kérdés

A céged hálózati hardvert akar venni, hogy a különálló hálózatait összekösse egymással. Milyen eszköz a megfelelő?

- A) Firewall
- B) Host
- C) LAN
- D) Router

**Helyes válasz:** D (Router)

**Magyarázat:** Bár a firewall is köthet össze hálózatokat, a fő célja a forgalom szűrése és ellenőrzése. A "host" bármilyen csomópont lehet, a LAN pedig maga a hálózat, nem egy eszköz. A router az az eszköz, amelyet kifejezetten a hálózatok összekötésére és a köztük lévő forgalom irányítására terveztek.

### 5. Kérdés

A céged le akarja cserélni a több éve használt régi hálózati tűzfalát egy olyanra, amely fejlettebb funkciókat biztosít. Milyen tűzfalat vegyen?

- A) Host-based firewall
- B) Next-level firewall
- C) Next-generation firewall
- D) Top-layer firewall

**Helyes válasz:** C (Next-generation firewall)

**Magyarázat:** A host-based firewall az az end host-okon futó szoftver, nem pedig hálózati eszköz. A "Next-level" és "Top-layer" kitalált fogalmak. A hagyományos tűzfal funkcióit fejlett szűrési lehetőségekkel kombináló eszközt next-generation firewall-nak nevezzük.
