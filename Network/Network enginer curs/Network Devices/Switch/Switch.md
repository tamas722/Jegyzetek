# Mi az a hálózati switch?

A **hálózati switch** egy hálózaton belüli eszközöket kapcsol össze (gyakran egy helyi hálózatot, azaz LAN-t*), és adatcsomagokat továbbít ezekhez az eszközökhöz és azoktól. A routerrel ellentétben a switch csak ahhoz az egyetlen eszközhöz küld adatot, amelynek szánják (amely lehet egy másik switch, egy router vagy egy felhasználó számítógépe), nem pedig több eszközből álló hálózatokhoz.

![A hálózati forgalom az internetről a routerhez, majd a hálózati switchhez, majd a számítógépekhez megy]

*\*A helyi hálózat (LAN) egy fizikailag közeli, összekapcsolt eszközök csoportja. Az otthoni WiFi hálózatok a LAN egy gyakori példája.*

**Whitepaper**
*Hogyan szabaduljunk meg a hálózati hardvertől*
[Szerezd meg a jelentést]

**Whitepaper**
*Stratégia kidolgozása a hálózat korszerűsítéséhez*
[Olvasd el a whitepaper-t]

## Mi a különbség a switch és a router között?

A **routerek** útvonalakat választanak az adatcsomagok számára, hogy azok hálózatokat keresztezve eljussanak a célállomásaikra. A routerek ezt úgy teszik, hogy különböző hálózatokhoz kapcsolódnak, és adatokat továbbítanak hálózatról hálózatra – beleértve a LAN-okat, a nagy kiterjedésű hálózatokat (WAN), vagy az autonóm rendszereket, amelyek az internetet alkotó nagy hálózatok.

A gyakorlatban ez azt jelenti, hogy a routerek szükségesek az internetkapcsolathoz, míg a switcheket csak az eszközök összekapcsolására használják. Az otthonoknak és kis irodáknak routerre van szükségük az interneteléréshez, de a legtöbbjüknek nincs szükségük hálózati switchre, hacsak nem igényelnek nagy mennyiségű Ethernet* portot. A nagy irodák, hálózatok és adatközpontok azonban több tucat vagy több száz számítógéppel általában igényelnek switcheket.

*\*Az Ethernet egy 2. rétegbeli protokoll az eszközök közötti adatküldéshez. A WiFi-vel ellentétben az Ethernet fizikai kapcsolatot igényel Ethernet kábelen keresztül.*

## Mi az a 2. rétegbeli switch? Mi az a 3. rétegbeli switch?

A hálózati switchek az OSI 2. rétegén (adatkapcsolati réteg) vagy 3. rétegén (hálózati réteg) működhetnek. A 2. rétegbeli switchek a cél MAC-cím alapján továbbítják az adatokat (lásd alább a meghatározást), míg a 3. rétegbeli switchek a cél IP-cím alapján továbbítják az adatokat. Egyes switchek mindkettőt tudják.

A legtöbb switch azonban 2. rétegbeli switch. A 2. rétegbeli switchek leggyakrabban Ethernet kábelekkel csatlakoznak a hálózatukban lévő eszközökhöz. Az Ethernet kábelek fizikai kábelek, amelyek Ethernet portokon keresztül csatlakoznak az eszközökhöz.

## Mi az a nem menedzselt switch? Mi az a menedzselt switch?

A **nem menedzselt switch** egyszerűen több Ethernet portot hoz létre egy LAN-on, hogy több helyi eszköz férhessen hozzá az internethez. A nem menedzselt switchek az eszközök MAC-címei alapján továbbítják az adatokat oda-vissza.

A **menedzselt switch** ugyanezt a funkciót látja el sokkal nagyobb hálózatokhoz, és a hálózati adminisztrátoroknak sokkal nagyobb kontrollt kínál a forgalom priorizálása felett. Lehetővé teszik az adminisztrátorok számára virtuális LAN-ok (VLAN) beállítását is, hogy a helyi hálózatot tovább osszák kisebb darabokra.

**Iratkozz fel**
*Biztonság és sebesség bármely Cloudflare csomaggal*
[Kezdje ingyen]

## Mi a különbség a MAC-cím és az IP-cím között?

A hálózati switchek MAC-címekre hivatkoznak annak érdekében, hogy az internetes forgalmat a megfelelő eszközökhöz küldjék, nem IP-címekre.

Az internethez csatlakozó minden eszköznek van IP-címe. Az IP-cím alfanumerikus karakterek sorozata, például 192.0.2.255 vagy 2001:0db8:85a3:0000:0000:8a2e:0370:7334. Az IP-címek úgy működnek, mint egy postai cím, lehetővé téve, hogy az adott címre irányított internetes kommunikáció elérje azt az eszközt. Az IP-címek gyakran változnak: mivel korlátozott számú IPv4-cím létezik, a felhasználói eszközökhöz általában újakat rendelnek, amikor új kapcsolatot létesítenek egy hálózattal.

Az IP-címeket a 3. rétegben használják, ami azt jelenti, hogy a számítógépek és eszközök az egész interneten IP-címeket használnak az adatok küldéséhez és fogadásához, függetlenül attól, hogy melyik hálózathoz csatlakoznak. Minden IP-csomag tartalmazza a forrás- és cél IP-címét a fejlécében, ahogy egy postai küldeménynek is van célcíme és feladó címe.

Ezzel szemben a **MAC-cím** minden hardverdarab állandó azonosítója, némileg olyan, mint egy sorozatszám. Az IP-címekkel ellentétben a MAC-címek nem változnak. A MAC-címeket a 2. rétegben használják, nem a 3. rétegben – ami azt jelenti, hogy nem szerepelnek az IP-csomag fejlécekben. Más szóval, a MAC-címek nem részei az internetes forgalomnak. Csak egy adott hálózaton belül használják őket.

## Hogyan ismerik a hálózati switchek a hálózatukban lévő eszközök MAC-címeit?

A 2. rétegbeli hálózati switchek a memóriában fenntartanak egy táblázatot, amely a MAC-címeket a switch Ethernet portjaihoz párosítja. Ezt a táblázatot **Content Addressable Memory (CAM)** táblázatnak nevezik.

Tegyük fel, hogy A számítógép egy Ethernet kábelhez csatlakozik, amely a switch 1-es portjába dugódik, B számítógép a 2-es portba, C számítógép pedig a 3-as portba. Amikor adat érkezik A számítógéphez, a switch megnézi a CAM táblázatát, látja, hová van A számítógép csatlakoztatva, és tudja, hogy az A számítógéphez tartó forgalmat az 1-es porton kell továbbítani, nem a 2-es vagy 3-as porton.

A switch CAM táblázata valahogy így nézne ki:

| MAC-cím | Port |
|---|---|
| A számítógép MAC-címe | 1 |
| B számítógép MAC-címe | 2 |
| C számítógép MAC-címe | 3 |

A switch CAM táblázata a memóriában tárolódik. Ha a switch ki van kapcsolva, a táblázat eltűnik, és a switchek újra meg kell tanulniuk a táblázatot, amikor újraindul.

Most tegyük fel, hogy a switchet éppen bekapcsolták, és még nem hozta létre a CAM táblázatát. Nem tudja, mely portokhoz vannak csatlakoztatva A, B és C számítógépek. A MAC-címeiket sem ismeri.

| MAC-cím | Port |
|---|---|
| ? | ? |
| ? | ? |
| ? | ? |

Tegyük fel, hogy A számítógép üzenetet küld B számítógépnek. A switch a következő lépéseket teszi, hogy eljuttassa az üzenetet B számítógéphez, és elkezdje kitölteni a CAM táblázatát:

- Rögzíti A számítógép MAC-címét és azt a portot, amelyen az üzenete bejött.
- Továbbítja A számítógép üzenetét a hálózat összes többi számítógépének (kivéve A számítógépet); ezt „floodingnak" nevezik.
- Amikor B számítógép válaszol, rögzíti B számítógép MAC-címét és portját is.

| MAC-cím | Port |
|---|---|
| A számítógép MAC-címe | 1 |
| B számítógép MAC-címe | 2 |
| ? | ? |

Most a switch CAM táblázata tudja, hol van A és B számítógép. A MAC-címeiket is ismeri.

## Hogyan védi a Cloudflare a hálózati switcheket?

A **Cloudflare Magic Transit** védi a hálózati infrastruktúra-eszközöket, például a switcheket és routereket az olyan DDoS támadási forgalomtól, amely offline állapotba helyezheti vagy kompromittálhatja azokat. A Magic Transit védi a helyszíni, felhőalapú és hibrid hálózatokat. Tudj meg többet a Magic Transitról vagy a 3. rétegbeli támadásokról.
