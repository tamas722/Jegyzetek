# Mi az a Secure Shell (SSH) protokoll?

A **Secure Shell (SSH)** protokoll egy módszer arra, hogy biztonságosan küldjünk parancsokat egy számítógépre nem biztonságos hálózaton keresztül. Az SSH kriptográfiát használ az eszközök közötti kapcsolatok hitelesítéséhez és titkosításához. Az SSH alagutazást (tunneling) vagy porttovábbítást (port forwarding) is lehetővé tesz, ami azt jelenti, hogy az adatcsomagok olyan hálózatokat is át tudnak lépni, amelyeket egyébként nem tudnának. Az SSH-t gyakran használják szerverek távoli vezérlésére, infrastruktúra kezelésére és fájlok átvitelére.

Utazás közben egy üzlet tulajdonosa utasításokat adhat alkalmazottainak távolról, hogy az üzlet zökkenőmentesen működjön, amíg távol van. Hasonlóképpen, az SSH lehetővé teszi az adminisztrátorok számára, hogy távolról kezeljék a szervereket és eszközöket. A régebbi távoli kezelési protokollok, mint a Telnet, olyan formában továbbították az adminisztrátorok parancsait, amelyet bárki láthatott. (Képzeld el, ha az alkalmazottak hangszóróra tennék az üzlet tulajdonosát egy hívás során; az üzlet összes vásárlója kihallgathatná a privát utasításokat.) A Telnet-tel ellentétben az SSH biztonságos – innen a Secure Shell név.

**Jelentés**
*2026-os biztonsági jelzések jelentés*
[Szerezd meg a jelentést]

## Mit csinál az SSH?

- **Távoli titkosított kapcsolatok:** Az SSH kapcsolatot hoz létre a felhasználó eszköze és egy távoli gép, gyakran egy szerver között. Titkosítást használ a kapcsolaton áthaladó adatok kódolásához. Egy elfogó fél csak valami statikus zajt találna – véletlenszerű adatokat, amelyek semmit sem jelentenek, hacsak nem dekódolják őket. (Az SSH olyan titkosítási módszereket használ, amelyek a visszafejtést tiltóan nehézzé teszik a kívülállók számára.)

- **Az alagutazás képessége:** A hálózatkezelésben az alagutazás egy módszer a csomagok mozgatására egy hálózaton keresztül olyan protokoll vagy útvonal használatával, amelyet egyébként nem tudnának használni. Az alagutazás úgy működik, hogy az adatcsomagokat* további információkkal – úgynevezett fejlécekkel – burkolja be, hogy megváltoztassa a célállomásukat. Az SSH alagutak egy porttovábbítás nevű technikát használnak a csomagok egyik gépről a másikra küldéséhez. Az SSH porttovábbítást alább részletesebben ismertetjük.

*\*Minden adat, amely átlép egy hálózatot, kisebb darabokra bomlik; ezeket a darabokat „csomagoknak" nevezik.*

**Iratkozz fel**
*Biztonság és sebesség bármely Cloudflare csomaggal*
[Kezdje ingyen]

## Hogyan működik az SSH?

### TCP/IP

Az SSH a TCP/IP protokollcsomag tetején fut – amelyre az internet nagy része támaszkodik. A TCP/IP szállítja és kézbesíti az adatcsomagokat. A TCP használata az egyik módja annak, ahogy az SSH különbözik más alagútprotokolloktól, amelyek közül néhány a gyorsabb, de kevésbé megbízható UDP-t használja helyette.

### Nyilvános kulcsú kriptográfia

Az SSH azért „biztonságos", mert titkosítást és hitelesítést épít be egy nyilvános kulcsú kriptográfia nevű folyamaton keresztül. A nyilvános kulcsú kriptográfia egy módja az adatok titkosításának vagy aláírásának két különböző kulccsal. Az egyik kulcs, a nyilvános kulcs, bárki számára elérhető. A másik kulcs, a privát kulcs, a tulajdonosa által titokban tartott. Mivel a két kulcs megfelel egymásnak, a kulcs tulajdonosának identitásának megállapításához a nyilvános kulcshoz tartozó privát kulcs birtoklása szükséges.

Ezek az „aszimmetrikus" kulcsok – azért nevezik így őket, mert különböző értékeik vannak – lehetővé teszik a kapcsolat két oldala számára, hogy azonos, megosztott szimmetrikus kulcsokat tárgyaljanak meg a csatornán történő további titkosításhoz. Amint ez a tárgyalás befejeződött, a két oldal a szimmetrikus kulcsokat használja a kicserélt adatok titkosításához.

Egy SSH kapcsolatban mindkét oldalnak van egy nyilvános/privát kulcspárja, és mindkét oldal hitelesíti a másikat ezekkel a kulcsokkal. Ez megkülönbözteti az SSH-t a HTTPS-től, amely a legtöbb implementációban csak a webszerver identitását ellenőrzi egy kliens-szerver kapcsolatban. (További különbségek közé tartozik, hogy a HTTPS általában nem teszi lehetővé a kliens számára a szerver parancssorának elérését, és hogy a tűzfalak néha blokkolják az SSH-t, de szinte soha nem blokkolják a HTTPS-t.)

### Hitelesítés

Míg a nyilvános kulcsú kriptográfia hitelesíti az SSH-ban a csatlakoztatott eszközöket, egy megfelelően biztosított számítógép továbbra is megköveteli a személy hitelesítését, aki az SSH-t használja. Ez gyakran felhasználónév és jelszó megadásának formájában történik.

A hitelesítés befejezése után a személy parancsokat hajthat végre a távoli gépen, mintha a saját helyi gépén tenné.

### SSH porttovábbítás, avagy „alagutazás"

A porttovábbítás olyan, mint egy üzenet továbbítása két ember között. Bob küldhet egy üzenetet Alice-nak, aki továbbadja Dave-nek. Hasonlóképpen, a porttovábbítás egy gépen egy IP-címre és portra irányított adatcsomagokat küld egy másik gépen lévő IP-címre és portra.

Például képzeld el, hogy egy adminisztrátor változtatást szeretne végrehajtani egy szerveren egy általa kezelt privát hálózaton belül, és ezt távoli helyről szeretné megtenni. Biztonsági okokból azonban az a szerver csak a privát hálózaton belüli más számítógépektől fogad adatcsomagokat. Az adminisztrátor ehelyett csatlakozhat a hálózaton belüli második szerverhez – amely nyitott az internetes forgalom fogadására –, majd SSH porttovábbítást használhat az első szerverhez való csatlakozáshoz. Az első szerver szempontjából az adminisztrátor adatcsomagjai a privát hálózaton belülről érkeznek.

### Az SSH használata

A Linux és Mac operációs rendszerek beépített SSH-val rendelkeznek. A Windows gépeknek szükségük lehet egy SSH kliensalkalmazás telepítésére. Mac és Linux számítógépeken a felhasználók megnyithatják a Terminál alkalmazást, és közvetlenül beírhatják az SSH parancsokat.

## Mire használják az SSH-t?

Technikailag az SSH bármilyen tetszőleges adatot továbbíthat egy hálózaton, és az SSH alagutazás számtalan célra beállítható. A leggyakoribb SSH felhasználási esetek azonban:

- Szerverek, infrastruktúra és alkalmazotti számítógépek távoli kezelése
- Fájlok biztonságos átvitele (az SSH biztonságosabb, mint a titkosítatlan protokollok, mint az FTP)
- Felhőszolgáltatások elérése anélkül, hogy a helyi gép portjait kitennénk az internetre
- Távoli csatlakozás privát hálózaton lévő szolgáltatásokhoz
- Tűzfal-korlátozások megkerülése

## Melyik port az SSH?

A **22-es port** az SSH alapértelmezett portja. Néha a tűzfalak blokkolhatják a tűzfal mögötti szerverek bizonyos portjaihoz való hozzáférést, de a 22-es portot nyitva hagyják. Az SSH ezért hasznos a tűzfal túloldalán lévő szerverek eléréséhez: a 22-es portra irányított csomagok nincsenek blokkolva, és aztán bármely más portra továbbíthatók.

## Vannak az SSH-hoz kapcsolódó biztonsági kockázatok?

Mivel az SSH hozzáférés általában emelt szintű jogosultságokkal jár, például alkalmazások telepítésének képességével egy szerveren, vagy adatok törlésével, módosításával vagy kinyerésével, az SSH hozzáférés káros lehet egy támadó – vagy akár egy jó szándékú bennfentes – kezében. Az SSH-t számos dokumentált támadásban használták privát adatok kiszivárogtatására, hátsó ajtók nyitására egy biztonságos hálózatba, és root hozzáférés megszerzésére szervereken.

Az SSH olyan tűzfalakon is átjuthat, amelyek nyitva hagyják a 22-es portot (ahogy sok teszi), lehetővé téve a támadók számára, hogy behatoljanak a biztonságos hálózatokba.

A támadók SSH kulcsokat is ellophatnak privát számítógépek és szerverek eléréséhez. Valójában az SSH kulcskezelés jelentős biztonsági probléma a nagy szervezetek számára, mivel sok szerverük több ezer vagy akár több millió kulcsot használhat, és ezeknek a kulcsoknak a manuális nyomon követése és frissítése szinte lehetetlen. Az SSH kulcsok nem járnak le, hacsak kifejezetten vissza nem vonják őket, így miután egy támadó megszerez egy kulcsot, hónapokig vagy évekig tartós hozzáférése lehet.

## Hogyan különbözik az SSH más alagútprotokolloktól?

Az SSH és más alagútprotokollok közötti egyik fő különbség az az OSI réteg, amelyen működnek. A GRE, az IP-in-IP és az IPsec mind hálózati rétegbeli protokollok. Mint ilyenek, nem ismerik a portokat (egy szállítási rétegbeli koncepció), helyette IP-címek között működnek. (Az SSH pontos OSI rétege nincs szigorúan meghatározva, de a legtöbb forrás a 7. réteg/alkalmazási réteg protokollként írja le, mint a HTTP, FTP és SMTP.)

Egy másik különbség az SSH TCP használata. A TCP, ahogy fentebb leírtuk, egy szállítási rétegbeli protokoll, és az interneten használt egyik fő protokoll. Egy másik széles körben használt szállítási rétegbeli protokoll az UDP, a User Datagram Protocol. Az UDP egy „best-effort" szállítási protokoll – csomagokat küld anélkül, hogy biztosítaná azok kézbesítését –, ami gyorsabbá teszi, de néha csomagvesztést eredményez. Bár a TCP lassabb, mint az UDP, garantálja az összes csomag sorrendben történő kézbesítését, ezért megbízhatóbb.

Az IPsec kizárólag UDP-t használ TCP helyett, hogy az IPsec csomagok átjuthassanak a tűzfalakon. Ezért az IPsec alagutak általában gyorsabbak, mint az SSH alagutak, de elveszíthetnek csomagokat az átvitel során. A GRE és az IP-in-IP TCP-vel vagy UDP-vel is használható.

Végül az SSH egyszerre csak egy alkalmazást titkosít, nem az összes forgalmat, amely egy eszközhöz megy és onnan érkezik. Ez megkülönbözteti az SSH-t az IPsec-től, amely az összes hálózati forgalmat titkosítja, függetlenül attól, hogy melyik alkalmazástól származik. Ezért az SSH-t nem VPN-ek beállítására használják.

## Hogyan teszi a Cloudflare biztonságosabbá az SSH-t?

Bár az SSH biztonsági kockázatokkal jár, sok szervezet támaszkodik rá a termelékenység fenntartásához. E kockázatok mérséklésére a Cloudflare SSH-t kínál Access for Infrastructure szolgáltatással a SASE platformjának részeként. Ez a szolgáltatás Zero Trust koncepciókat alkalmaz az SSH-ra, lehetővé téve a szervezetek számára a részletes kontrollt afölött, hogyan csatlakozhatnak a felhasználók az SSH szervereikhez. Korlátozza, mit tehet egy felhasználó, csökkentve a jogosulatlan SSH használat hatását egy támadó vagy bennfentes részéről.

Az SSH with Access for Infrastructure megszünteti az SSH kulcsok kezelésének szükségességét is (ami hatalmas kihívás, ahogy fentebb leírtuk). Ehelyett a szervezetek rövid életű SSH tanúsítványokat használhatnak, amelyeket egy Cloudflare által kezelt hitelesítésszolgáltató (CA) bocsát ki.

A Cloudflare azt is lehetővé teszi a szervezetek számára, hogy monitorozzák és naplózzák az SSH hozzáférést és használatot, ami elengedhetetlen mind az incidens-helyreállításhoz, mind a szabályozási megfeleléshez.

*Tudj meg többet az SSH with Access for Infrastructure szolgáltatásról.*

## GYIK

### Mi az SSH?

Az SSH (Secure Shell) egy hálózati protokoll, amely titkosított kapcsolatokat hoz létre számítógépek között biztonságos távoli hozzáféréshez. A 22-es TCP porton működik, és hitelesítést, titkosítást és integritást biztosít a nem biztonságos hálózatokon továbbított adatok védelméhez.

### Hogyan hitelesítik az SSH kulcsok a felhasználókat?

Az SSH kulcsok aszimmetrikus kriptográfiával hitelesítik a felhasználókat egy nyilvános-privát kulcspár segítségével a hagyományos jelszavak helyett. A privát kulcs a kliens rendszerén marad, míg a nyilvános kulcs a szerveren található, lehetővé téve a biztonságos hitelesítést anélkül, hogy a tényleges hitelesítő adatokat továbbítanák a hálózaton.

### Milyen előnyei vannak az SSH alagutazásnak?

Az SSH alagutazás egy biztonságos kapcsolatot hoz létre, amelyen keresztül más adatok is átjuthatnak, gyakorlatilag titkosítva a titkosítatlan forgalmat. Ez lehetővé teszi a biztonságos porttovábbítást és védi az érzékeny információkat az elfogástól az átvitel során.

### Hogyan viszonyul az SSH a Telnet-hez?

A Telnet nyílt szövegben továbbítja az adatokat. Az SSH titkosítja az összes kommunikációt, hogy megakadályozza a lehallgatást és az útközbeni támadásokat. Az SSH gyakorlatilag felváltotta a Telnet-et a távoli szerverhozzáférés standardjaként, kiválóbb biztonsági funkciói miatt.

### Milyen SSH kulcstípusokat használnak gyakran?

A gyakori SSH kulcstípusok közé tartozik az RSA, az ECDSA és az EdDSA, amelyek különböző szintű biztonságot és teljesítményt kínálnak. Az Ed25519 – az EdDSA aláírási séma egy változata, amely elliptikus görbét használ – egyre népszerűbb, mert erős biztonságot nyújt rövidebb kulcshosszúsággal és gyorsabb műveletekkel.
