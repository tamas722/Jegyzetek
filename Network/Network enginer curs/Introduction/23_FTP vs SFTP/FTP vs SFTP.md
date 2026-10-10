# FTP vs SFTP: Mi a különbség? Melyiket használd?

**Szerző: Matteo Duò** | Frissítve: 2025. október 1.

![ftp vs sftp]

Összezavarodtál az FTP és az SFTP közötti különbségtől? Ha csatlakozni szeretnél a weboldalad szerveréhez fájlok átviteléhez vagy kezeléséhez, a legegyszerűbb megoldás egy FTP kliens használata. De amikor az FTP kliensedet konfigurálod, láthatod, hogy mind az FTP-t, mind az SFTP-t támogatja.

Szóval, mi a különbség az FTP és az SFTP között? És jobb-e az egyiket a másikkal szemben használni?

Nos, a rövid válasz az, hogy az SFTP sokkal biztonságosabb, mint az FTP, ezért itt a Kinstánál csak SFTP-t kínálunk, és azt javasoljuk minden felhasználónknak, hogy SFTP-t használjon.

A hosszabb válaszért olvass tovább, és végigvezetünk mindenen, amit tudnod kell erről a két protokollról, és hogy melyiket érdemes használnod.

## FTP vs SFTP magyarázat

Először beszéljünk arról, mi ugyanaz az FTP és az SFTP között.

Mindkét protokoll lehetővé teszi, hogy FTP klienst, például FileZillát használj a weboldalad szervereihez való csatlakozáshoz. A végfelhasználó (te) számára gyakorlatilag nulla különbség van az élményben az FTP és az SFTP között. Mindkét protokollal képes leszel:

- Csatlakozni a szerveredhez
- Böngészni a szervered összes fájlját (még a rejtetteket is)
- Fájlokat feltölteni a helyi számítógépedről a szerveredre
- Fájlokat letölteni a szerveredről a helyi számítógépedre
- És így tovább…

Vannak azonban kulcsfontosságú különbségek a motorháztető alatt, ezért fontos megérteni az FTP és az SFTP közötti különbséget.

Nézzük meg részletesebben.

## Mi az FTP?

Az FTP a **File Transfer Protocol** rövidítése. A kliens/szerver modellt használva az FTP támogatja a fájlok közvetlen átvitelét a választott FTP kliensed és a webszervered között.

Az FTP két külön csatornát használ az információátvitelhez: egy parancscsatornát és egy adatcsatornát. Alapértelmezés szerint mindkét csatorna titkosítatlan, ami azt jelenti, hogy rosszindulatú szereplők potenciálisan lehallgathatják az általad átvitt információkat.

*Ingyenes hosting 1 hónapra*
*Szerezd meg az első hónapot ingyen a kiválasztott csomagoknál!*
[Próbáld ki most]

## Mi az SFTP?

Az SFTP a **SSH File Transfer Protocol** rövidítése, bár gyakran **Secure File Transfer Protocol**-nak is nevezik.

Az SFTP ugyanazt az alapfunkciót kínálja, mint az FTP, de alagutazást (tunneling) használ, és SSH-n keresztül végzi a fájlátvitelt, ami eltér az FTP kliens-szerver és közvetlen átviteli megközelítésétől.

Szóval, mi az SSH?

Az SSH a **Secure Shell** rövidítése, egy kriptográfiai protokoll, amely biztonságos hozzáférést kínál egy géphez (esetünkben a szerveredhez) nem biztonságos hálózatokon keresztül.

Az SFTP csak egyetlen csatornát használ, és lehetővé teszi a kliensed hitelesítését vagy felhasználónév/jelszó, vagy SSH kriptográfiai kulcsok használatával.

## Akkor mi a különbség az FTP és az SFTP között?

A kulcsfontosságú különbség az FTP és az SFTP között az, hogy az SFTP biztonságos csatornát használ a fájlok átviteléhez, míg az FTP nem.

Az SFTP-vel a kapcsolatod mindig biztonságos, és az FTP kliensed és a webszervered között mozgó adatok titkosítottak. Ez azt jelenti, hogy rosszindulatú szereplők nem tudnak közbeékelődni és elfogni az adataidat – minden, amit átviszel, mindig titkosított.

Az FTP-vel felhasználónévvel és jelszóval kell hitelesítened, amikor először csatlakozol. Azonban a webszervered és az FTP kliensed között áthaladó adatok nincsenek titkosítva, ami azt jelenti, hogy egy rosszindulatú szereplő elméletileg lehallgathatja ezeket az információkat.

Ez különösen veszélyes lenne, ha érzékeny információkat tartalmazó fájlokat visznél át. Például egy WordPress oldal esetében átviheted a `wp-config.php` fájlt, amely tartalmazza az adatbázis hitelesítő adataidat, valamint más kritikus beállításokat.

Ha egy rosszindulatú szereplő megszerezné ezt a fájlt, mindene meglenne ahhoz, hogy átvegye a WordPress oldalad irányítását.

Ez a legfontosabb tanulság:

> Bár mindkét protokoll lehetővé teszi a fájlok átvitelét a kliensed és a szervered között, az SFTP sokkal biztonságosabb, mint az FTP.
