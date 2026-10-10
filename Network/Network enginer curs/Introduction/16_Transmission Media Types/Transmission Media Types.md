# Átviteli médiumok és típusaik a számítógépes hálózatokban

**Készítette: Deepak Sharma**
**2026. június 6.**
Megosztás

Az átviteli médium az a fizikai vagy logikai útvonal, amely adatokat szállít egyik számítástechnikai eszközről a másikra. Ez a kapcsolódás gerince, amely meghatározza az adatátvitel sebességét, megbízhatóságát és biztonságát. Az átviteli médiumoknak két típusa van: **irányított vagy vezetékes média** (koaxiális kábel, optikai szál stb.) és **irányítatlan vagy vezeték nélküli média** (rádióhullámok, WiFi stb.).

Ebben a cikkben megismerkedhetsz a számítógépes hálózatokban használt átviteli médiumokkal, az átviteli médiumok különböző típusaival, példákkal, alkalmazásokkal és diagramokkal.

*CCNA képzést keresel Indiában?*
*Csatlakozz képzési programunkhoz, szerezz ingyenes demót és gyakorlati labokat.*

## Az átviteli médiumok jellemzői

Egy kommunikációs csatorna kiválasztásakor annak hatékonyságát öt kritikus teljesítménymutató alapján mérik, amelyek meghatározzák az adatáramlás hatékonyságát és megbízhatóságát.

### 1. Sávszélesség (átviteli kapacitás)

A sávszélesség azt a maximális adatátviteli sebességet képviseli, amelyet egy médium támogatni tud, általában bit per másodpercben (bps) mérve. Nagy teljesítményű hálózatokban az olyan médiumok, mint az optikai szál, jelentősen nagyobb sávszélességet kínálnak, mint a réz, lehetővé téve nagy felbontású videó és hatalmas adathalmazok egyidejű továbbítását torlódás nélkül.

### 2. Késleltetés és látencia

A sávszélesség azt a maximális adatátviteli sebességet képviseli, amelyet egy médium támogatni tud, általában bit per másodpercben (bps) mérve. Nagy teljesítményű hálózatokban az olyan médiumok, mint az optikai szál, jelentősen nagyobb sávszélességet kínálnak, mint a réz, lehetővé téve nagy felbontású videó és hatalmas adathalmazok egyidejű továbbítását torlódás nélkül.

### 3. Teljes birtoklási költség (TCO)

A költségtényező nem csak a fizikai médium kezdeti tőkekiadását (CAPEX) foglalja magában, hanem a működési kiadásokat (OPEX) is. Például, bár az optikai szálas kábel telepítése drágább, mint a csavart érpár, tartóssága és alacsony karbantartási igénye gyakran alacsonyabb hosszú távú TCO-t eredményez.

### 4. Csillapítás és átviteli távolság

A csillapítás a jel erősségének fokozatos elvesztése, ahogy halad. Minden médiumnak van egy meghatározott maximális hatékony távolsága, amely után ismétlőre vagy erősítőre van szükség. Az optikai szál itt kiemelkedik, kilométereken keresztül fenntartva a jel integritását, míg az Ethernet kábelek (UTP) általában 100 méterre korlátozódnak.

### 5. Jelbiztonság és EMI-immunitás

A biztonság azt méri, mennyire sebezhető egy médium a „lehallgatással" vagy az elektromágneses interferenciával (EMI) szemben. A vezeték nélküli jelek hajlamosak az elfogásra és a zajra, míg az optikai szál gyakorlatilag immunis az EMI-ra, és rendkívül nehéz észlelés nélkül lehallgatni, így ez a legbiztonságosabb választás érzékeny gerinchálózatokhoz.

## Az átviteli médiumok osztályozása

Az átviteli médiumokat fizikai jellemzőik és a jelek hálózaton keresztüli terjedési módja szerint kategorizálják. Széles körben két típusra oszthatók:

### 1. Irányított média (vezetékes / határolt)

Az irányított médiában a jeleket egy szilárd fizikai útvonal mentén irányítják. Ezek a csatornák nagy sebességet, biztonságos átvitelt és védelmet nyújtanak a külső interferenciával szemben.

- **a) Csavart érpár kábel:** A LAN-ok leggyakoribb médiuma; árnyékolt (STP) és árnyékolatlan (UTP) kategóriákba sorolható.
- **b) Koaxiális kábel:** Központi vezetővel és fonott árnyékolással rendelkezik; elsősorban kábeltévéhez és nagy sebességű internethez használják.
- **c) Optikai szálas kábel:** Fényimpulzusokkal továbbít adatokat üveg vagy műanyag magokon keresztül. Ez a nagy sávszélességű, nagy távolságú kommunikáció aranystandardja.

### 2. Irányítatlan média (vezeték nélküli / határolatlan)

Az irányítatlan média elektromágneses hullámokat továbbít a levegőn, vákuumon vagy vízen keresztül fizikai vezető nélkül. Ez rugalmasságot biztosít a mobilitásban, de érzékenyebb az interferenciára.

- **a) Rádióhullámok:** Mindenirányú hullámok, amelyeket AM/FM rádióhoz és vezeték nélküli telefonokhoz használnak; könnyen átjutnak a falakon.
- **b) Mikrohullámok:** Egyirányú, nagy frekvenciájú hullámok, amelyek látótávolságú (LoS) terjedést igényelnek; műholdas és földi kommunikációhoz használják.
- **c) Infravörös:** Rövid hatótávolságú, nagy frekvenciájú hullámok, amelyeket eszközök közötti kommunikációhoz (például távirányítókhoz) használnak, mivel nem tudnak áthatolni szilárd akadályokon.

![Az átviteli médiumok típusainak folyamatábrája]

Most nézzük meg az egyes átviteli médiumtípusokat, és értsük meg azok használatát, előnyeit és hátrányait.

## Irányított átviteli média

Az irányított média, más néven vezetékes vagy határolt média, olyan fizikai átviteli útvonalakra utal, amelyekben az adatjeleket egy meghatározott útvonal mentén továbbítják, általában egy fizikai médiumon, például kábelen keresztül.

A jeleket ezeken a fizikai határokon belül irányítják és tartják, így az irányított média alkalmas korlátozott földrajzi területeken való kommunikációra, ahol a sebesség, a megbízhatóság és a biztonság elengedhetetlen.

Az irányított átviteli média három típusa:

### 1. Csavart érpár kábel

A csavart érpár kábelek egyfajta irányított átviteli médium, amelyet hang- és adatkommunikációhoz használnak. Két szigetelt rézvezetékből állnak, amelyeket összecsavarnak az elektromágneses interferencia és az áthallás csökkentése érdekében. Ez a technológia elektromos jeleket továbbít, és széles körben használják telefonvonalakban és Ethernet hálózatokban egyszerűsége és költséghatékonysága miatt.

![Csavart érpár kábel képe]

A csavart érpár kábeleknek két fő típusa van:

1. **Árnyékolatlan csavart érpár (UTP):** Ez a kábeltípus nem rendelkezik árnyékoló réteggel, így érzékenyebb a külső interferenciára. Az UTP-t általában telefon- és Ethernet hálózatokban használják.
2. **Árnyékolt csavart érpár (STP):** Az STP kábelek árnyékoló réteggel rendelkeznek (általában fólia vagy fonott réz), amely segít megvédeni a kábeleket a külső interferenciától. Ezeket olyan környezetekben használják, ahol nagyobb adatátviteli sebességre és biztonságosabb átvitelre van szükség.

**A csavart érpár kábel előnyei:**

- Megfizethető és könnyen telepíthető.
- A csavarás segít kioltani az elektromágneses zajt.
- Támogatja a nagy sebességű adatátvitelt Ethernet hálózatokhoz.

**A csavart érpár kábel hátrányai:**

- A jelek hosszabb kábelhosszon gyengülnek.
- Nem olyan gyors, mint az optikai szálas kábelek.
- Kevésbé ellenálló a zajjal szemben, mint a koaxiális és az optikai szál.

### 2. Koaxiális kábel

A koaxiális kábelek olyan irányított átviteli médiumok, amelyeket főként TV-hálózatokban és szélessávú internetben használnak. Elektromos jeleket továbbítanak, és réteges szerkezettel rendelkeznek: központi rézvezető, szigetelő réteg, fém árnyékolás és külső borítás. Ez a kialakítás erős védelmet nyújt az interferencia ellen, így ideális nagy frekvenciájú adatátvitelhez.

**A koaxiális kábelek előnyei:**

- Nagy sávszélesség és alacsony jelveszteség.
- Jó védelmet nyújt a zaj és az interferencia ellen.
- Könnyen bővíthető a hálózat további kábelek hozzáadásával.

**A koaxiális kábelek hátrányai:**

- Drága nagy távolságú kommunikációhoz.
- Vaskos és nehezebb telepíteni szűk helyeken.
- Egyetlen meghibásodási pont az egész hálózatot érintheti.

### 3. Optikai szálas kábel

Az optikai szálas kábelek fejlett irányított médiumok, amelyek fényimpulzusokkal továbbítanak adatokat. Vékony üveg- vagy műanyagszálakból készülnek, és minimális veszteséggel szállítják a fényjeleket, így ideálisak nagy távolságú és nagy sebességű kommunikációhoz. Szerkezetük nagy sávszélességet, alacsony interferenciát és kiváló teljesítményt biztosít a modern hálózatokban.

![Optikai szálas kábel képe]

**Az optikai szálas kábelek előnyei:**

- Rendkívül nagy sávszélesség, hatalmas mennyiségű adat továbbítására képes.
- Immunis az elektromágneses interferenciára, így alkalmas nagy elektromos zajszintű környezetekben.
- Könnyű és képes nagy távolságú kommunikációra jelentős jelveszteség nélkül.

**Az optikai szálas kábelek hátrányai:**

- Drága telepíteni és karbantartani.
- Törékeny és hajlamos a sérülésre, ha nem kezelik óvatosan.
- Nehéz összeilleszteni és javítani, ha megsérül.

*Tudj meg többet Cisco Enterprise kurzusainkkal. Vedd fel a kapcsolatot a tanulmányi tanácsadókkal, hogy megtaláld a számodra megfelelő kurzust!*

## Irányítatlan átviteli média

Az irányítatlan média, más néven vezeték nélküli vagy határolatlan média, olyan átviteli útvonalakra utal, ahol az elektromágneses jeleket a levegőn keresztül továbbítják fizikai kábelek szükségessége nélkül.

Ez a médiatípus nagy távolságú vezeték nélküli kommunikációra és olyan területeken használatos, ahol a kábelek telepítése nem praktikus vagy lehetetlen.

Az irányítatlan átviteli média három típusa:

### 1. Rádióhullámok

A rádióhullámok egyfajta irányítatlan átviteli médium, amelyek elektromágneses jeleket szállítanak nagy távolságokra. Áthatolnak a falakon és akadályokon, így ideálisak olyan műsorszóró technológiákhoz, mint az AM/FM rádió, a televízió és a mobilkommunikáció. Ez a vezeték nélküli technológia széles területű lefedettséget támogat, és széles körben használják a mindennapi kommunikációs rendszerekben.

**A rádióhullámok előnyei:**

- Könnyen előállíthatók és nagy távolságokat tesznek meg.
- Ideálisak széles területek feletti műsorszórásra.
- Áthatolnak az épületeken és más akadályokon.

**A rádióhullámok hátrányai:**

- Hajlamosak más jelekből és környezeti tényezőkből származó interferenciára.
- Kevésbé biztonságosak, mert a jeleket jogosulatlan felhasználók elfoghatják.

### 2. Mikrohullámok

A mikrohullámú kommunikáció egy vezeték nélküli átviteli módszer, amely nagy frekvenciájú rádióhullámokat használ pont-pont adatátvitelhez. Elektromágneses jeleket továbbít, és tiszta látótávolságot igényel az antennák között. Széles körben használják mobilhálózatokban, műholdas kapcsolatokban és épületek közötti kommunikációban, támogatja a nagy sebességű, nagy távolságú adatátvitelt, de érzékeny a fizikai akadályokra.

**A mikrohullámú kommunikáció előnyei:**

- Támogatja a nagy sebességű adatátvitelt.
- Alkalmas nagy távolságú kommunikációra.
- Nagy mennyiségű adatforgalmat képes kezelni.

**A mikrohullámú kommunikáció hátrányai:**

- Az antennák pontos beállítását igényli a tiszta látótávolságú átvitelhez.
- Befolyásolják az időjárási viszonyok, például az eső és a hó, amelyek ronthatják a jelminőséget.

### 3. Infravörös kommunikáció

Az infravörös (IR) kommunikáció egy rövid hatótávolságú vezeték nélküli átviteli módszer, amely infravörös fényjeleket használ. Gyakran megtalálható távirányítókban és személyes eszközökben, közvetlen látótávolságot igényel, és nem tud áthatolni a falakon. Korlátozott hatótávolsága fokozza a biztonságot a külső interferencia és a jogosulatlan hozzáférés csökkentésével.

**Az infravörös kommunikáció előnyei:**

- Immunis más rádiójelekből származó interferenciára.
- Biztonságos kapcsolatot nyújt rövid hatótávolságú kommunikációhoz.
- Gyakran használják személyes elektronikus eszközökben és otthoni automatizálási rendszerekben.

**Az infravörös kommunikáció hátrányai:**

- Korlátozott hatótávolság, és nem tud áthatolni falakon vagy akadályokon.
- Látótávolságot igényel az adó és a vevő között.

### Műholdas kommunikáció (külön említés)

A műholdas kommunikáció a mikrohullámú átvitel egy speciális formája, amely műholdakat használ nagy frekvenciájú jelek továbbítására földi állomások között. Ellentétben a tipikus mikrohullámú kapcsolatokkal, amelyek rövid távolságokon közvetlen látótávolságot igényelnek, a műholdas rendszerek globális lefedettséget tesznek lehetővé, beleértve a távoli területeket is. Azonban magasabb költségekkel és hosszabb késleltetéssel járnak a világűrbe és vissza történő hatalmas jelterjedési távolság miatt.

**A műholdas kommunikáció előnyei:**

- Globális lefedettséget nyújt, így ideális nagy távolságú kommunikációhoz.
- Használható távoli vagy megközelíthetetlen területeken, ahol a hagyományos kommunikációs infrastruktúra nem elérhető.

**A műholdas kommunikáció hátrányai:**

- A telepítés és karbantartás magas költsége.
- Jel késleltetés a Föld és a műholdak közötti nagy távolságok miatt.
- Az olyan időjárási viszonyok, mint a heves eső, ronthatják a jelminőséget.

## Mi az átviteli torzulás?

Az átviteli torzulások minden olyan tényezőre utalnak, amelyek negatívan befolyásolják a jel minőségét vagy integritását, ahogy az egy kommunikációs médiumon halad.

![Az átviteli torzulás folyamatábrája]

A három fő átviteli torzulástípus:

| Torzulástípus | Meghatározás | Mérséklés |
|---|---|---|
| Csillapítás | A jel erősségének fokozatos elvesztése, ahogy egy médiumon halad. | Használj erősítőket, jó minőségű médiumot, és minimalizáld a távolságot. |
| Torzítás | A jel formájának vagy alakjának változása az átvitel során. | Használj kiegyenlítési technikákat a fáziskülönbségek korrigálására. |
| Zaj | Nem kívánt jelek, amelyek zavarják a továbbított jelet. | Használj árnyékolást, szűrést és megfelelő földelési technikákat. |

## Hogyan válasszunk jó átviteli médiumot?

Egy kommunikációs rendszer tervezésekor számos tényezőt kell figyelembe venni a megfelelő átviteli médium kiválasztásakor. Ezek a tényezők befolyásolják a hálózat általános teljesítményét, költségét és megbízhatóságát.

| Tényező | Magyarázat |
|---|---|
| Sávszélesség | A nagyobb sávszélesség gyorsabb adatátvitelt tesz lehetővé. |
| Csillapítás | Az alacsonyabb csillapítású médiumok jobban alkalmasak nagy távolságú kommunikációra. |
| Interferencia | Egyes médiumok ellenállóbbak az elektromágneses interferenciával szemben, mint mások. |
| Költség | A különböző médiumok eltérő telepítési, karbantartási és működési költségekkel rendelkeznek. |
| Telepítés | A telepítés egyszerűsége fontos a gyakorlati szempontok és a hálózat telepítésének kezdeti költsége szempontjából. |
