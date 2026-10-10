# Simple Network Time Protocol (SNTP)

## Tartalomjegyzék

- Mi az SNTP?
- Hogyan működik az SNTP szinkronizálás?
- Az SNTP és az NTP közötti különbségek

A hálózatok és a rendszerek közötti folyamatok elterjedésével szükségessé vált egy egységes megoldás megtalálása a rendszeridő szinkronizálására. A hivatalos szabvány a **Network Time Protocol (NTP)** volt, amely különféle algoritmusokat használ az eltérések korrigálására, és hivatalos világidőt használ referenciakaraként. Az 1990-es évek elején kiadott egyszerűsített változat, a **Simple Network Time Protocol (SNTP)** létrehozása elsősorban az akkoriban elérhető számítógépek korlátozott számítási teljesítményének köszönhető. Az SNTP-n keresztüli szinkronizálás során a feldolgozási teljesítmény sokkal kevésbé kerül felhasználásra, mint az alapvető NTP protokoll esetében, ami felszabadítja a processzort más feladatokra – a legtöbb modern eszköz azonban olyan feldolgozási teljesítménnyel rendelkezik, amely képes kezelni az NTP komplexitását.

## Mi az SNTP?

A **Simple Network Time Protocol (SNTP)** a TCP/IP protokollcsalád időszinkronizálási protokollja. A **kapcsolat nélküli User Datagram Protocol (UDP)**-n alapul, és minden támogató eszközön használható az IP-hálózatokban (IPv4 és IPv6) a rendszeridő szinkronizálására. Az alap egy kliens-szerver struktúra, amelyben az egyes kliensek az időinformációt egy kiválasztott időszinkronizáló szerverről szerzik be ugyanabban a hálózatban. Abban az esetben, ha ez nem elérhető, további időszinkronizáló szerverek állnak rendelkezésre tartalékként. Az SNTP első verziója 1992-ből származik, és az RFC 1361 szabályozza. Ez a protokollt a Network Time Protocol egyszerűsített változataként írja le, amelyet szintén a rendszeridő beállítására használnak a hálózatokban.

Több átdolgozás után a Delaware-i Egyetem munkacsoportja által kifejlesztett protokoll jelenleg a negyedik verziójában (SNTPv4) érhető el. Ezt a verziót az egyetem 2006-ban tette közzé az RFC 4330-ban.

> **Definíció: SNTP**
> A Simple Network Time Protocol (SNTP) az internetprotokoll-család egyik protokollja, amelyet a rendszeridő hálózatokban történő szinkronizálására használnak. A jelenlegi SNTPv4 verzió támogatja mind az IPv4, mind az IPv6 hálózatokat, és részletesebben az RFC 4330 írja le. Az SNTP kliensek egy előre kiválasztott időszinkronizáló szervert használnak a beállítások elvégzéséhez.

## Hogyan működik az SNTP szinkronizálás?

Az NTP-hez hasonlóan az SNTP-t arra használják, hogy könnyedén szinkronizálják a hálózat összes eszközének rendszeridejét. Erre a célra egy kliens-szerver modellt használnak, és a hivatalos világidőt („Coordinated Universal Time") referenciaként. Az úgynevezett **időszinkronizáló szerverek** az idő „elosztóiként" működnek az SNTP kliensek, azaz az érintett eszközök számára, amelyek rendszeridejét a Simple Network Time Protocol segítségével kell szinkronizálni. Az ehhez szükséges IP-csomagok az **UDP 123-as portján** keresztül működnek, és mivel az SNTP nagyon hasonló az NTP-hez, a szinkronizálási folyamathoz gyakran NTP szervereket használnak.

Az IP-csomagok továbbítása történhet egyszerű közvetlen kapcsolatokon (unicast) keresztül egy kliens és egy szerver között, vagy többpontos kapcsolatokon (broadcast és multicast) keresztül egy szerver és több kliens között.

## Az SNTP és az NTP közötti különbségek

Az SNTP és az NTP a következő két pont szerint különböztethető meg:

- A szinkronizálási folyamathoz használt szerverek száma
- Az időeltérések kompenzálására és a lehető legpontosabb eredmények biztosítására használt algoritmusok száma

### Az SNTP szinkronizálás gyakran egyetlen időszinkronizáló szerveren alapul

A bonyolultabb NTP protokollal ellentétben a Simple Network Time Protocol az egyszerűséget helyezi előtérbe. Az SNTP specifikációja azt javasolja, hogy az időinformációt csak egyetlen szerverről szerezzék be, és kerüljék a további kliens-szerver függőségeket. Továbbá az SNTP szerveroldali használata csak akkor szándékolt, ha más szinkronizálási forrás nem elérhető. Az NTP-n keresztüli szinkronizálás ezzel szemben különböző szerverek összetett konstrukciójára támaszkodik, amelyek az információt egy hierarchikus rétegrendszerben adják tovább. A láncban elfoglalt pozíciótól függően a folyamatban részt vevő rendszerek egyaránt lehetnek kliensek és szerverek.

### A Simple Network Time Protocol kevesebb algoritmust tartalmaz

Az SNTP nem használ néhány olyan algoritmust, amelyek az NTP specifikációjában szabványként szerepelnek. Ezek az algoritmusok arra szolgálnak, hogy kompenzálják a helyi rendszeridő eltéréseit, hogy mindig a lehető legjobb viszonyt tudják biztosítani a világidőhöz. Ennek eredményeként az SNTP alacsonyabb pontosságot kínál, mint az NTP, ami alkalmassá teszi az egyszerűsített protokollt olyan alkalmazásokhoz és folyamatokhoz, amelyek hihetetlenül pontos időszinkronizálást igényelnek. A kevesebb algoritmus és a kliens-szerver kommunikáció kombinációjának köszönhetően az SNTP szinkronizálás jelentősen kevesebb erőforrást igényel, ami különösen hasznos egyszerű eszközök vagy alacsony számítási teljesítményű rendszerek számára.
