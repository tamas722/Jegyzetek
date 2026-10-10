# Keretek és csomagok a számítógépes hálózatokban 


Az internet és a hálózatok egymás közötti kommunikációjának gerince.

Képzeld el az internetet és az általad küldött vagy fogadott adatokat úgy, mint egy csomó ingázót, akik különböző helyekre próbálnak eljutni. A keretek és a csomagok? Ők a járművek és a közlekedési szabályok, amelyek biztosítják, hogy minden hatékonyan és biztonságosan odaérjen, ahová kell.

## Keretek: A hálózati rétegek szállítóteherautói


A keretek az OSI-modell Adatkapcsolati rétegén (2. réteg) működnek, amely arról szól, hogy kapcsolatokat hozzon létre az ugyanazon a hálózaton lévő eszközök között. Gondolj egy keretre úgy, mint egy szállítóteherautóra, amely árukat (az adataidat) szállít. Ez a teherautó okos. Pontosan tudja, hová kell leadnia az árukat a helyi területen belül, köszönhetően a MAC (Media Access Control) címnek, amely a hálózati interfészek egyedi azonosítója.

A keret beburkolja az adataidat egy kezdő és záró jelzővel, beleértve a küldő és fogadó MAC-címeit, kicsit olyan, mint ahogy egy szállítólevél meghatározza, kitől származik és kinek szól. Emellett tartalmaz egy hibaelenőrző eszközt, az úgynevezett ellenőrző összeget (checksum), hogy biztosítsa az adatok sértetlen megérkezését. Ha a keret kátyúba fut a digitális autópályán, és az adatok összekeverednek, az ellenőrző összeg nem fog egyezni, és a keret megsemmisül, újraküldést kérve, hogy elkerülje a sérült áruk kézbesítését.

### Keretstruktúra


A keret olyan, mint egy csomag, amelyet egy futárszolgálaton keresztül küldenek, különböző részeket tartalmaz, hogy biztosítsa, hogy odaérjen, ahová kell, sértetlenül és a megfelelő címzetthez.

Íme egy tipikus keretstruktúra:

- **Preambulum:** Ez a bit figyelmet kelt az eszközökben. Olyan, mint egy figyelmeztetés, amely jelzi az eszközöknek, hogy figyeljenek, mert adat érkezik.
- **Keret kezdete (SOF):** Egy meghatározott sorozat, amely a keret kezdetét jelzi. Ez a vonal a homokban, amely azt mondja: „Az üzenet itt kezdődik!"
- **Cél MAC-cím:** Ez megmondja a hálózatnak, hová kell mennie a keretnek. Gondolj rá úgy, mint egy csomag szállítási címére.
- **Forrás MAC-cím:** A feladó címe, amely megmutatja, honnan származik a keret.
- **Típus:** Azonosítja a beágyazott adat protokollját, megmondva a fogadó eszköznek, hogyan értelmezze az adat hasznos terhét.
- **Adat és kitöltés:** A keret magja – a tényleges adataid, plusz némi extra kitöltés, ha az adat nem éri el a keret minimális hosszúsági követelményét.
- **Ciklikus redundancia-ellenőrzés (CRC):** Egy divatos ellenőrző összeg, amelyet az átvitt keret hibáinak észlelésére használnak. Olyan, mint egy ellenőrző kód, amely biztosítja, hogy a csomagot nem manipulálták szállítás közben.
- **Keret vége:** A keret végét jelzi. Ez egyértelmű jelzés arra, hogy a keret teljes.


## Csomagok: Az információs autópálya ingázói


Egy réteggel feljebb lépve a csomagok az OSI-modell Hálózati rétegében (3. réteg) élnek, elsősorban az adatok különböző hálózatok közötti mozgatásával foglalkozva. Ha a keretek szállítóteherautók, akkor a csomagok az ingázók, akik aktatáskányi információt hordoznak.

A csomagok olyanok, mint a célcímmel beburkolt adatok, de ezúttal az IP (Internet Protocol) címről van szó, amely segít nekik eligazodni a terjedelmes hálózati városokban és az internet hatalmas távolságaiban. A cél IP mellett a csomagok tartalmazzák a forrás IP-t is, hogy a fogadó fél tudja, honnan származik, és vissza tudjon küldeni egy választ vagy nyugtázást.

A csomagokkal az a helyzet, hogy olyanok, mint a tömegközlekedést használó ingázók; nem feltétlenül mind ugyanazt az útvonalat választják. A hálózati forgalomtól, a torlódástól és a működő útvonalválasztási protokolloktól függően a csomagok különböző útvonalakat vehetnek igénybe ugyanahhoz a célállomáshoz. Ez a rugalmasság segít az internetnek kezelni a terhelést, és biztosítani, hogy az adatok átjussanak, még akkor is, ha az egyik útvonal blokkolt vagy lassú. De ne aggódj, a célállomáson ezeket a csomagokat a helyes sorrendben állítják össze újra, így az üzenet világos és teljes.

**Videó (Csomagutazás – Hogyan mozognak a csomagok egy hálózaton)**

### Csomagstruktúra


Most térjünk át a csomagokra, amelyek olyanok, mint a postaládába dobott levelek, amelyeket arra terveztek, hogy hosszú távolságokat tegyenek meg különböző postai hálózatokon (vagy jelen esetben az interneten) keresztül. Íme, mi van egy csomagban:

- **Fejléc:** A csomag fejléce létfontosságú útvonalválasztási és működési információkat hordoz, beleértve:
  - **Verzió:** Melyik IP-verziót használják (IPv4 vagy IPv6).
  - **Fejléc hossza:** A fejléc mérete.
  - **Differenciált szolgáltatások:** Információk a prioritás és a szolgáltatásminőség kezeléséhez.
  - **Teljes hossz:** A teljes csomagméret, a fejlécet és az adatokat is beleértve.
  - **Azonosítás, jelzők, töredékeltolás:** A feldarabolt csomagok újraösszeállításához használják.
  - **Élettartam (TTL):** Korlátozza a csomag élettartamát, hogy megakadályozza a végtelen ciklusba kerülését.
  - **Protokoll:** Jelzi a beágyazott adat protokollját (TCP, UDP stb.).
  - **Fejléc ellenőrző összeg:** Ellenőrző összeg a fejléc hibaelenőrzéséhez.
  - **Forrás- és cél IP-címek:** Honnan származik a csomag, és hová tart.
- **Hasznos teher (Adat):** A csomag lényege, ez a ténylegesen továbbított adat.
- **Utótag (néha, a protokolltól függően):** Egy lábléc, amely hibaelenőrzést és egyéb, a fejlécben nem szereplő információkat tartalmazhat.

## Mi a különbség közöttük?

A fő különbségek a keretek és a csomagok között:

### 1. Működési réteg:

- A keretek az OSI-modell Adatkapcsolati rétegén (2. réteg) működnek. Ez a réteg arról szól, hogy megbízható kapcsolatot hozzon létre az ugyanazon a hálózaton lévő eszközök között, biztosítva, hogy az adatok hatékonyan átvihetők legyenek fizikai kapcsolatokon keresztül.
- A csomagok ezzel szemben a Hálózati rétegben (3. réteg) működnek. Itt történik az útvonalválasztás varázslata, amely lehetővé teszi, hogy az adatok több hálózaton keresztül jussanak el a forrástól a célállomásig.

### 2. Címzés:

- A keretek olyanok, mint a helyi szállítóteherautók; MAC-címeket használnak az adatok küldéséhez az ugyanazon a hálózaton vagy helyi hálózaton (LAN) belüli eszközökhöz. A MAC-címek olyanok, mint az internet utcacímei, minden eszközre egyedileg jellemzőek.
- A csomagok IP-címeket használnak a különböző hálózatok kiterjedtségében való navigáláshoz. Gondolj az IP-címekre úgy, mint az adatok GPS-koordinátáira, amelyek a küldőtől a címzettig vezetik azokat az interneten keresztül, függetlenül attól, milyen messze vannak egymástól.

### 3. Tartalom:

- A keretek beágyazzák a csomagokat, biztosítva a szükséges fejléceket és utótagokat az adatkapcsolati réteg kommunikációjához, beleértve a hibaelenőrzési információkat az adatintegritás biztosításához.
- A csomagok tartalmazzák a ténylegesen továbbított adatokat (az e-mailjeidet, webhely-információkat stb.) és a különböző hálózatokon keresztül a végső célállomás eléréséhez szükséges útvonalválasztási információkat.

### 4. Funkció és cél:

- A keretek a helyi kézbesítésről szólnak ugyanazon a hálózaton belül. Olyan dolgokkal foglalkoznak, mint a hibaelenőrzés és a fizikai adathordozóhoz való hozzáférés szabályozása.
- A csomagok a hosszútávfutók, amelyeket arra terveztek, hogy adatokat mozgassanak a hálózatok között. Arról szólnak, hogy megtalálják a legjobb utat ahhoz, hogy az adatok odaérjenek, ahová kell, navigálva az internet összetett hálójában.

Tehát a legegyszerűbben fogalmazva: a keretek a hálózaton belüli adatátvitelhez használt struktúra, amely MAC-címeket használ az eszközök azonosítására, míg a csomagok a hálózatok közötti adatátvitel módszere, amely IP-címekre támaszkodik az adatok interneten keresztüli irányításához. A keretek biztosítják, hogy az adataid jól viselkedjenek a helyi játszótéren, a csomagok pedig az adataid útlevele és kalauza a globális utazásban.

## Összefoglalva

Képzeld el, hogy egy szuper fontos e-mailt küldesz a New York-i laptopodról egy barátod tokiói okostelefonjára. Az e-mailje csomagokra bomlik. Ezek a csomagok minden egyes ugrásnál keretekbe vannak beágyazva az útjuk során, a routeredtől esetleg egy műholdig és különféle más hálózati eszközökön keresztül, minden alkalommal a helyi szabályokat (MAC-címeket) használva ahhoz, hogy egyik pontból a másikba jussanak a helyi hálózaton belül. Amint elérik a szélesebb internetet, az IP-címek vezetik őket a globális digitális tájon keresztül a végső célállomásukig, ahol újra összeállnak, hogy a barátod elolvashassa.

A keretek és a csomagok gondosan strukturáltak, hogy biztosítsák a megbízható és hatékony adatátvitelt. Szerkezetük minden része kulcsfontosságú szerepet játszik az internetet alkotó hálózatok összetett hálójában való navigálásban, biztosítva, hogy az adataid helyesen és biztonságosan érkezzenek meg, függetlenül a távolságtól.
