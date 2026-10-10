# Mi az az Internet Control Message Protocol (ICMP)?

Az **Internet Control Message Protocol (ICMP)** a hálózati problémák diagnosztizálására szolgál az interneten.

## Mi az Internet Control Message Protocol (ICMP)?

Az **Internet Control Message Protocol (ICMP)** egy hálózati rétegbeli protokoll, amelyet a hálózati eszközök a hálózati kommunikációs problémák diagnosztizálására használnak. Az ICMP-t elsősorban annak meghatározására használják, hogy az adatok időben eljutnak-e a szánt célállomásukra. Az ICMP protokollt általában hálózati eszközökön, például routereken használják. Az ICMP kulcsfontosságú a hibajelentéshez és a teszteléshez, de elosztott szolgáltatásmegtagadási (DDoS) támadásokban is használható.

**Iratkozzon fel**
*Biztonság és sebesség bármely Cloudflare csomaggal*
[Kezdje ingyen]
→

## Mire használják az ICMP-t?

Az ICMP elsődleges célja a hibajelentés. Amikor két eszköz csatlakozik az interneten keresztül, az ICMP hibákat generál, amelyeket megoszt a küldő eszközzel, ha bármely adat nem érte el a szánt célállomását. Például, ha egy adatcsomag túl nagy egy router számára, a router eldobja a csomagot, és ICMP üzenetet küld vissza az adat eredeti forrásának.

Az ICMP protokoll másodlagos felhasználása a hálózati diagnosztika; a gyakran használt terminál segédprogramok, a traceroute és a ping is ICMP-vel működnek. A traceroute segédprogramot két internetes eszköz közötti útvonalválasztási útvonal megjelenítésére használják. Az útvonalválasztási útvonal az összekapcsolt routerek tényleges fizikai útvonala, amelyen egy kérésnek át kell haladnia, mielőtt eléri a célállomását. Az egyik router és a másik közötti utazást „ugrásnak" nevezik, és a traceroute jelenti az egyes ugrásokhoz szükséges időt is az útvonal mentén. Ez hasznos lehet a hálózati késleltetés forrásainak meghatározásához.

A ping segédprogram a traceroute egyszerűsített változata. A ping teszteli a kapcsolat sebességét két eszköz között, és pontosan jelenti, mennyi idő alatt ér el egy adatcsomag a célállomására, és tér vissza a küldő eszközére. Bár a ping nem nyújt adatokat az útvonalválasztásról vagy az ugrásokról, mégis nagyon hasznos mérőszám a két eszköz közötti késleltetés felméréséhez. Az ICMP echo-request és echo-reply üzeneteket általában ping végrehajtására használják.

Sajnos a hálózati támadások kihasználhatják ezt a folyamatot, olyan zavarási módokat teremtve, mint az ICMP flood támadás és a ping of death támadás.

**Whitepaper**
*Stratégia kidolgozása a hálózat korszerűsítéséhez*
[Olvassa el a whitepaper-t]
→

## Hogyan működik az ICMP?

Az Internet Protocol (IP)-vel ellentétben az ICMP nem kapcsolódik szállítási rétegbeli protokollhoz, mint a TCP vagy az UDP. Ez az ICMP-t kapcsolat nélküli protokollá teszi: az egyik eszköznek nem kell kapcsolatot nyitnia egy másik eszközzel, mielőtt ICMP üzenetet küldene. A normál IP forgalmat TCP-vel küldik, ami azt jelenti, hogy bármely két adatcserélő eszköz először TCP kézfogást hajt végre, hogy biztosítsa, mindkét eszköz készen áll az adatok fogadására. Az ICMP nem nyit kapcsolatot ilyen módon. Az ICMP protokoll sem teszi lehetővé egy adott port célzását egy eszközön.

## Mi az ICMP csomag?

Az ICMP csomag egy olyan csomag, amely az ICMP protokollt használja. Az ICMP csomagok egy normál IP fejléc után egy ICMP fejlécet tartalmaznak. Amikor egy routernek vagy szervernek hibaüzenetet kell küldenie, az ICMP csomag törzse vagy adatszakasza mindig tartalmazza a hibát okozó csomag IP fejlécének másolatát.

## Hogyan használják az ICMP-t DDoS támadásokban?

### ICMP flood támadás

A ping flood vagy ICMP flood akkor történik, amikor a támadó megpróbálja elárasztani a célzott eszközt ICMP echo-request csomagokkal. A célnak fel kell dolgoznia és válaszolnia kell minden csomagra, felemésztve annak számítástechnikai erőforrásait, amíg a legitim felhasználók nem tudnak szolgáltatást kapni.

![ICMP flood támadás: több echo request és echo reply]

### Ping of death támadás

A ping of death támadás akkor történik, amikor a támadó a maximálisan megengedett csomagméretnél nagyobb pinget küld a célzott gépnek, ami a gép lefagyását vagy összeomlását okozza. A csomag feldarabolódik a célállomás felé vezető úton, de amikor a cél újra összeállítja a csomagot annak eredeti, maximális méretet meghaladó méretére, a csomag mérete puffer túlcsordulást okoz.

A ping of death támadás nagyrészt történelmi jelentőségű ezen a ponton. A régebbi hálózati berendezések azonban még mindig sebezhetők lehetnek vele szemben.

### Smurf támadás

Egy Smurf támadásban a támadó egy hamisított forrás IP-címmel rendelkező ICMP csomagot küld. A hálózati berendezés válaszol a csomagra, a válaszokat a hamisított IP-re küldve, és elárasztja az áldozatot nem kívánt ICMP csomagokkal. A „ping of death"-hez hasonlóan ma a Smurf támadás csak örökölt berendezésekkel lehetséges.

Az ICMP nem az egyetlen hálózati rétegbeli protokoll, amelyet a 3. rétegbeli DDoS támadásokban használnak. A támadók a múltban GRE csomagokat is használtak például.

Általában a hálózati rétegbeli DDoS támadások a hálózati berendezéseket és infrastruktúrát célozzák, szemben az alkalmazási rétegbeli DDoS támadásokkal, amelyek webes tulajdonságokat céloznak. A Cloudflare Magic Transit az egyik módja a hálózati rétegbeli DDoS támadások elleni védekezésnek.
