# Hogyan profitál egy vállalkozás a WLAN-ból?

Azáltal, hogy a munkát bárhol lehetővé teszi, a vezeték nélküli hálózatok nem csupán növelik a termelékenységet és kényelmet nyújtanak. Újradefiniálhatják a vállalati célokat és azok elérésének módját – irodákban, gyárakban, egészségügyi intézményekben és iskolákban.

## Hogyan működik a WLAN?

A sugárzó médiához hasonlóan a WLAN rádióhullámokon keresztül továbbítja az információt. Az információ csomagokban kerül elküldésre. A csomagok rétegeket tartalmaznak címkékkel és utasításokkal, amelyek együttműködnek a végpontokhoz rendelt egyedi MAC (Media Access Control) címekkel, lehetővé téve a kívánt helyre történő irányítást.

## Hogyan jön létre a WLAN?

A WLAN kétféleképpen konfigurálható:

- **Infrastruktúra:** Az otthoni vagy irodai Wi-Fi hálózat egy példa az infrastruktúra módban beállított WLAN-ra. A végpontok mind csatlakoztatva vannak, és egy bázisállomáson keresztül kommunikálnak egymással, amely internet-hozzáférést is biztosíthat.

  Egy alapvető infrastruktúra WLAN néhány részből állítható össze: egy vezeték nélküli router, amely bázisállomásként működik, és a végpontok. A végpontok lehetnek számítógépek, mobileszközök, nyomtatók vagy más eszközök. A legtöbb esetben a vezeték nélküli router az internetkapcsolat is egyben.

- **Ad hoc:** Ebben a beállításban a WLAN bázisállomás használata nélkül kapcsolja össze a végpontokat, például munkaállomásokat és mobileszközöket. Az ad hoc vezeték nélküli hálózathoz általában a Wi-Fi Direct technológiát használják. Az ad hoc WLAN könnyen beállítható, és alapvető peer-to-peer (P2P) kommunikációt biztosíthat.

  Egy ad hoc WLAN-hoz csak két vagy több beépített rádióadóval rendelkező végpontra van szükség, például számítógépekre vagy mobileszközökre. Miután a hálózati beállításokat ad hoc módra állították, az egyik felhasználó elindítja a hálózatot, amely láthatóvá válik a többiek számára.

## Biztonságos a WLAN?

A WLAN sebezhetőbb a behatolással szemben, mint egy fizikai hálózat. Egy vezetékes hálózatnál a rossz szereplőnek fizikai hozzáférést kell szereznie egy belső hálózathoz, vagy be kell törnie egy külső tűzfalat. Egy WLAN eléréséhez a rossz szereplőnek csak a hálózat hatótávolságán belül kell lennie.

A WLAN biztosításának legalapvetőbb módszere a MAC-címek használata a jogosulatlan állomások hozzáférésének megtagadására. Az eltökélt támadók azonban képesek lehetnek csatlakozni a hálózatokhoz úgy, hogy identitásukat engedélyezett címként álcázzák.

A WLAN leggyakoribb biztonsági módszere a titkosítás, beleértve a Wired Equivalent Privacy (WEP) és a Wi-Fi Protected Access (WPA) protokollokat, a WPA2-vel mint szabványos hitelesítési módszerrel.

## Hogyan működik a roaming egy WLAN-ban?

Bármilyen méretű hálózat esetén a hozzáférési pontok kiterjeszthetik a hozzáférési területet.

A Wi-Fi szabványokat úgy tervezték, hogy lehetővé tegyék egy nem helyhez kötött felhasználó kapcsolatának átugrását egyik hozzáférési pontról a másikra, bár egyes felhasználók és alkalmazások rövid kieséseket tapasztalhatnak. Még át nem fedő hozzáférési pontok esetén is a felhasználó kapcsolata egyszerűen szünetel, amíg létre nem jön a kapcsolat a következő hozzáférési ponttal.

A további hozzáférési pontok lehetnek vezetékesek vagy vezeték nélküliek. Amikor a hozzáférési pontok átfedik egymást, úgy konfigurálhatók, hogy a terhelés megosztásával és kezelésével optimalizálják a hálózatot.

## Mi az a mesh hálózat?

A mesh hálózat kiterjeszti a WLAN hatótávolságát és teljesítményét számos, egymással vezeték nélkül összekapcsolódó hozzáférési pont használatával. A mesh hálózat több átviteli útvonalat biztosít. Intelligens algoritmusokkal képes a jobb teljesítmény érdekében kezelni az útvonalválasztást.

## WLAN architektúra

### Állomások (Stations)

Az állomások olyan komponensek, amelyek vezeték nélkül csatlakoznak a hálózatokhoz. Ezek vagy hozzáférési pontok, vagy végpontok, és mindegyik egyedi hálózati címmel van azonosítva.

### Basic Service Set (BSS)

A BSS a hálózathoz csatlakozó állomások csoportja. Ad hoc hálózatokban az állomások csoportját Independent BSS-nek (IBSS) nevezik. Egy Extended Service Set (ESS) több BSS elosztórendszeren keresztüli összekapcsolásával jön létre.

### Elosztórendszer (Distribution system)

Az elosztórendszer összekapcsolja a hozzáférési pontokat egy ESS-ben. A kapcsolatok lehetnek vezetékesek vagy vezeték nélküliek. Egy vezeték nélküli elosztórendszer (WDS) használhat mesh-t vagy saját WDS protokollt. A fix vezeték nélküli egy speciális rádióátviteli forma egy földrajzilag távoli hozzáférési pont összekapcsolására.

### Hozzáférési pont (Access point)

A hozzáférési pont az a bázisállomás, amely hubként szolgál, amelyhez más állomások csatlakoznak. A hozzáférés az állomások hálózathoz való kapcsolódására utal, de internet-hozzáférést is jelenthet, mivel sok router internet modemként is működik. Egy ESS-ben a hozzáférési pontok Ethernet kábelekkel vagy vezeték nélkül kapcsolódhatnak össze.

*Lásd a hozzáférési pontokat*

### Híd (Bridge)

A hidat egy WLAN LAN-hoz vagy egy hozzáférési ponthoz való csatlakoztatására használják.

### Végpont (Endpoint)

A végpont bármely végfelhasználói állomás, például számítógép, mobileszköz, nyomtató vagy Internet of Things (IoT) eszköz.

## A WLAN előnyei

- **Kiterjesztett hatótávolság:** A WLAN-ok lehetővé teszik a számítástechnikát bárhol, még nagy adatterhelések és fejlett webalkalmazások esetén is.

- **Eszközrugalmasság:** A WLAN széles eszközkört támogat, például számítógépeket, telefonokat, táblagépeket, játékrendszereket és IoT-eszközöket.

- **Könnyebb telepítés és kezelés:** A WLAN kevesebb fizikai berendezést igényel, mint egy vezetékes hálózat – pénzt takarít meg, csökkenti a telepítési időt, és kisebb helyet foglal egy irodai környezetben.

- **Egyszerűbb skálázhatóság:** A WLAN könnyen skálázható. Felhasználók hozzáadása olyan egyszerű, mint bejelentkezési hitelesítő adatok hozzárendelése.

- **Virtuális hálózatkezelés:** A WLAN szinte minden kezelése virtuálisan elvégezhető. Egyetlen szoftverinterfész láthatóságot biztosíthat, kezelheti a felhasználókat, monitorozhatja a hálózat állapotát és adatokat gyűjthet.
