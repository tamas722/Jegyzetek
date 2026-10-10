# Mi az a storage area network (SAN)?


## Mi az a SAN?

A **storage area network (SAN)** egy dedikált hálózat, amelyet egy adott környezetre szabtak – szervereket, tárolórendszereket, hálózati switcheket, szoftvereket és szolgáltatásokat kombinálva.

### Miért fontosak a storage area networkök?

A számítógépes memória és a helyi tárolási erőforrások nem feltétlenül nyújtanak elegendő tárhelyet, tárolásvédelmet, többfelhasználós hozzáférést vagy sebességet és teljesítményt vállalati alkalmazásokhoz. Ezért a legtöbb szervezet alkalmaz valamilyen SAN-t a **network-attached storage (NAS)** mellett a jobb hatékonyság és jobb adatkezelés érdekében.

Hagyományosan csak korlátozott számú tárolóeszköz csatlakozhatott egy szerverhez, ami korlátozta a hálózat tárolási kapacitását. A SAN azonban hálózati rugalmasságot vezet be, amely lehetővé teszi, hogy egy szerver, vagy sok heterogén szerver több adatközponton keresztül közös tárolási segédprogramot osszon meg. A SAN kiküszöböli a hagyományos dedikált kapcsolatot a hálózati fájlszerver és a tárolás között – és azt a koncepciót, hogy a szerver ténylegesen birtokolja és kezeli a tárolóeszközöket –, megszüntetve a sávszélesség-szűk keresztmetszeteket. A SAN kiküszöböli az egyetlen meghibásodási pontokat a tárolás megbízhatóságának és rendelkezésre állásának fokozása érdekében.

A SAN a **katasztrófa-helyreállítás (DR)** szempontjából is optimális, mivel egy hálózat sok tárolóeszközt tartalmazhat, beleértve a lemezt, a mágnesszalagot és az optikai tárolást. A tárolási segédprogram távol is lehet az általa használt szerverektől.

**A legfrissebb technológiai hírek, szakértői betekintéssel**
*Maradj naprakész a legfontosabb – és legérdekesebb – iparági trendekről az AI, az automatizálás, az adatok és azon túl a Think hírlevéllel. Lásd az IBM adatvédelmi nyilatkozatát.*

- Keresztnév*
- Vezetéknév*
- Üzleti e-mail*
* Minden mező kötelező

*Az előfizetésed angol nyelven érkezik. Minden hírlevélben találsz egy leiratkozási linket. További információért lásd az IBM adatvédelmi nyilatkozatát.*

[Feliratkozás]

## Melyek a SAN előnyei?

A SAN felszabadítja a tárolóeszközt, hogy az ne egy adott szerverbuszon legyen. A tárolást közvetlenül a hálózathoz csatolja, externalizálva és funkcionálisan elosztva a tárolást a szervezet egészében. A SAN központosítja a tárolóeszközöket és a szerverek klaszterezését, potenciálisan könnyebb és olcsóbb központosított adminisztrációt érve el, és csökkentve a teljes birtoklási költséget.

Jellemzően **blokk-szintű tárolórendszereket** használva a SAN-ok lehetővé teszik az adatmozgató alkalmazások jobb teljesítményét azáltal, hogy az adatokat közvetlenül a forrásból a célba továbbítják, minimális szerverbeavatkozással. A szervezetek azonban bármilyen, az infrastruktúrájukhoz megfelelő **network file system (NFS)**-t használhatnak. A SAN-ok lehetővé teszik több hoszt számára, hogy több, ugyanahhoz a hálózathoz csatlakoztatott tárolóeszközhöz férjenek hozzá új hálózati architektúrákban. A SAN a következő előnyöket kínálhatja:

- **Javított alkalmazás-rendelkezésre állás:** A tárolás az alkalmazásoktól függetlenül létezik, és több útvonalon keresztül érhető el a fokozott megbízhatóság, rendelkezésre állás és szolgáltathatóság érdekében.
- **Jobb alkalmazásteljesítmény:** A SAN-ok tehermentesítik és áthelyezik a tárolási feldolgozást a szerverekről külön hálózatokra.
- **Központosított és konszolidált:** A SAN-ok egyszerűbb kezelést, skálázhatóságot, rugalmasságot és magas rendelkezésre állást tesznek lehetővé.
- **Távoli helyszín adatátvitele és tárolása:** A SAN-ok védik az adatokat a katasztrófáktól és rosszindulatú támadásoktól távoli másolattal.
- **Egyszerű központosított kezelés:** A SAN-ok leegyszerűsítik a kezelést a tárolási média egységes képeinek létrehozásával.

**Think Keynote**

*Gyorsítsd fel az AI ROI-t hibrid felhővel | Skyla Loomis, GM, IBM Z Software, IBM*

*Gyorsítsd fel az AI ROI-t hibrid felhővel*
*Tudd meg, hogyan segít a teljes verem hibrid felhő megközelítés a szervezeteknek az AI megbízható futtatásában, a szabályozási és biztonsági követelmények teljesítésében és a fenntartható ROI nagy léptékű szállításában.*

[Fedezd fel a Power Virtual Servert]

## Hogyan működik a SAN?

Néha a szerverek mögötti hálózatnak nevezik, a SAN tartalmaz egy kommunikációs infrastruktúrát, amely fizikai kapcsolatokat biztosít, és lehetővé teszi bármely eszköz számára, hogy áthidalja a hálózatot összekapcsolt elemeken, például switcheken és direktorokon keresztül. A SAN a **tárolóbusz-koncepció** kiterjesztése is. Ez a koncepció lehetővé teszi a tárolóeszközök és szerverek összekapcsolását hasonló elemek, például **local area network (LAN)**-ok és **wide-area network (WAN)**-ok használatával. A SAN tartalmaz egy kezelési réteget is, amely rendszerezi a kapcsolatokat, a tárolóelemeket és a számítógépes rendszereket, ami viszont robusztus adatátvitelt támogat.

A mai SAN-ok új módszereket hoznak létre a tárolás szerverekhez csatolására, lehetővé téve a magas rendelkezésre állást és a teljesítményjavulást. Összekapcsolják a megosztott tárolótömböket és szalagkönyvtárakat több szerverrel, amelyeket klaszterezett szerverek használnak feladatátvételhez. Megkerülhetik a hagyományos hálózati forgalmi szűk keresztmetszeteket, elősegítve a közvetlen, nagy sebességű adatátvitelt a szerverek és tárolóeszközök között három módon:

- **Szerverről tárolásra:** Ennek a hagyományos interakciós modellnek az előnye, hogy ugyanaz a tárolóeszköz sorosan vagy egyidejűleg több szerver által is elérhető.
- **Szerverről szerverre:** A szerverek SAN-t használhatnak a nagy sebességű, alacsony késleltetésű és nagy volumenű kommunikáció elősegítésére.
- **Tárolásról tárolásra:** Az adatok szerverbeavatkozás nélküli mozgatásának képessége felszabadítja a szerver processzorciklusait más tevékenységekre, például alkalmazásfeldolgozásra. Példák közé tartozik egy lemezmeghajtó eszköz, amely szerverbeavatkozás nélkül menti adatait egy szalageszközre, vagy egy távoli eszköz tükrözése a SAN-on keresztül.

## Mi az a SAN switch?

A **SAN switch** a legtöbb storage area network szívében van. Egyetlen célja a tárolási adatforgalom mozgatása a szerverek és a megosztott tárolókészletek között. Egy switch több gazdaszervert kapcsol össze, amelyek tárolószerverekből és eszközökből állnak, hogy SAN-t hozzanak létre. Egyes switchek önálló eszközként használhatók egy egyszerű SAN fabric építéséhez.

Mások más switchekkel összekapcsolhatók egy nagyobb SAN fabric építéséhez. A **SAN fabricok** több SAN switch aktív, intelligens és nem megosztott összekapcsolásai. Növelik a lehetséges kapcsolatok számát egy SAN-ban. A **Fibre Channel host bus adapterek (HBA)** összekapcsolják a switcheket a fájlszerverekkel.

### SAN összetevők

A SAN alapvető összetevői a szerverek, a tárolás és a hálózati infrastruktúra.

- **Szerverek:** A szerverinfrastruktúra minden SAN megoldás mögöttes oka, és ez az infrastruktúra szerverplatformok keverékét tartalmazza. Az olyan kezdeményezésekkel, mint a szerverkonszolidáció és az internetes kereskedelem, a SAN-ok iránti igény növekszik, ami nagyobbá teszi a hálózati tárolás fontosságát.
- **Tárolás:** Egy tárolórendszer állhat lemezrendszerekből és szalagrendszerekből. A lemezrendszer tartalmazhat HDD-ket, SSD-ket vagy flash meghajtókat. A szalagrendszer állhat szalagmeghajtókból, szalagautomatákból és szalagkönyvtárakból.
- **Hálózati infrastruktúra:** A SAN kapcsolódás hardver- és szoftverkomponensekből áll, amelyek összekapcsolják a tárolóeszközöket és szervereket, beleértve a Fibre Channelt. A hardver tartalmazhat hubokat, switcheket, átjárókat, direktorokat és routereket. A szoftver tartalmazza a SAN kezelőszoftvert.

## A SAN tárolási kapcsolatok típusai

A **storage area network protokoll** egy olyan kapcsolattípus, amely meghatározza, hogyan kommunikálnak az eszközök és switchek egymással egy SAN fabricon belül. Egy SAN használhat egy protokollt vagy többet. Bizonyos eszközök többprotokollos routerek és eszközök.

A többprotokollos routerek és eszközök javított skálázhatóságot, biztonságot és kezelhetőséget nyújtanak. Lehetővé teszik a különböző SAN fabricokban lévő eszközök számára, hogy kommunikáljanak anélkül, hogy a fabricokat egyetlen nagy meta-SAN fabricba olvasztanák. A gyártótól függően a többprotokollos routerek és eszközök számos protokollt támogatnak, például a **server message block (SMB)**-t, és saját funkciókat kínálnak, például zónázást. Íme a SAN kapcsolattípusok listája:

- **Internet Small Computer System Interface (iSCSI):** Az iSCSI egy IP-alapú szabványos protokoll adattároló eszközök hálózaton keresztüli összekapcsolására és adatok továbbítására azáltal, hogy SCSI parancsokat visz át IP-hálózatokon. Az iSCSI protokoll használata IP-alapú SAN-okhoz lehetővé teszi az ügyfelek számára, hogy ugyanazokat a hálózati technológiákat használják a tároláshoz, a tároláskezeléshez és az adathálózatokhoz. Mivel az iSCSI **Transmission Control Protocol/Internet Protocol (TCP/IP)**-t használ, az iSCSI szinte bármilyen fizikai hálózaton futtatható.

- **Fibre Channel Protocol (FCP):** Az FCP a Fibre Channel (FC) hálózatokon használt soros SCSI parancsprotokoll. Nagyobb átviteli teljesítményt nyújt, mint egy **local area network (LAN)**. Ez egy gigabit sebességű hálózati technológia, amelyet elsősorban tárolóhálózatkezeléshez használnak, és ez a nyílt rendszerek szabványos protokollja. Először a szuperszámítógép-területen használták, az FCP a vállalati tárolásban a SAN-ok szabványos kapcsolattípusává vált.

- **Fibre Channel over Ethernet (FCoE):** Az FCoE egy protokoll az FC csomagok Etherneten keresztüli irányítására. Javíthatja a SAN infrastruktúra rugalmasságát és egyszerűsítését. A LAN-ok és SAN-ok dedikált kapcsolási megoldásait egyetlen eszközzel helyettesíti, amely mindkét típusú adatot továbbíthatja: IP-csomagokat és tárolási adatokat. Ezeket a telepítéseket **konvergált hálózatoknak** nevezik.

- **Fibre Channel over IP (FCIP):** Az FCIP más néven Fibre Channel tunneling vagy storage tunneling. Ez a módszer lehetővé teszi a Fibre Channel információ továbbítását az IP-hálózaton keresztül alagutazva. A legtöbb szervezet rendelkezik meglévő IP-infrastruktúrával, így vonzónak találják, hogy viszonylag alacsony költséggel kapcsolhatják össze a földrajzilag szétszórt SAN-okat.

- **Non-Volatile Memory Express over Fibre Channel (FC-NVMe):** Az FC-NVMe egy hosztvezérlő interfész és tárolási protokoll. Felgyorsítja az adatok átvitelét a vállalati és kliensrendszerek, valamint a **solid-state drive (SSD)**-k között egy számítógép nagy sebességű **Peripheral Component Interconnect Express (PCIe)** buszán keresztül.

## SAN versus NAS tárolás

A **direct-attached storage (DAS)**-tól eltérően a hálózatalapú tárolás lehetővé teszi, hogy több számítógép hozzáférjen hozzá egy hálózaton keresztül, ami jobbá teszi az adatmegosztáshoz és az együttműködéshez. A távoli tárolási képessége is jobban alkalmassá teszi a biztonsági mentésekhez és az adatvédelemhez. Két tipikus hálózatalapú tárolási beállítás a **network-attached storage (NAS)** és a **storage area network (SAN)**.

A NAS gyakran egyetlen eszköz, amely redundáns tárolókonténerekből vagy egy **redundant array of independent disks (RAID)**-ból áll. A SAN tárolás lehet több eszköz hálózata, beleértve az SSD- és flash-tárolást, hibrid tárolást, hibrid felhőtárolást, biztonsági mentési szoftvereket és berendezéseket, valamint felhőtárolást. Fontos a megfelelő kiválasztása, amely illik a felhasználási eseteidhez. Íme, hogyan különböznek a NAS és a SAN:

| SAN | NAS |
|---|---|
| Több eszköz hálózata | Egyetlen tárolóeszköz vagy RAID |
| Blokk-tároló rendszer | Fájltároló rendszer |
| Fibre Channel hálózat | TCP/IP Ethernet hálózat |
| Több felhasználóra optimalizálva | Korlátozott felhasználók |
| Gyorsabb teljesítmény | Korlátozott sebesség |
| Erősen bővíthető | Korlátozott bővítési lehetőségek |
| Magasabb költség és összetett beállítás | Alacsonyabb költség és egyszerű beállítás |
