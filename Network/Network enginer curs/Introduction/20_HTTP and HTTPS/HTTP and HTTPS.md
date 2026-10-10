# Mi a különbség a HTTP és a HTTPS között?

A **Hypertext Transfer Protocol (HTTP)** egy protokoll vagy kommunikációs szabályrendszer a kliens-szerver kommunikációhoz. Amikor meglátogatsz egy weboldalt, a böngésződ egy HTTP kérést küld a webszervernek, amely egy HTTP válasszal válaszol. A webszerver és a böngésződ nyílt szövegként cserél adatokat. Röviden, a HTTP protokoll az alapvető technológia, amely a hálózati kommunikációt működteti. Ahogy a neve is sugallja, a **Hypertext Transfer Protocol Secure (HTTPS)** a HTTP egy biztonságosabb verziója vagy kiterjesztése. A HTTPS-ben a böngésző és a szerver biztonságos, titkosított kapcsolatot létesít az adatok továbbítása előtt.

## Hogyan működik a HTTP protokoll?

A HTTP egy alkalmazási rétegbeli protokoll az Open Systems Interconnection (OSI) hálózati kommunikációs modellben. Többféle kérés- és választípust határoz meg. Például, amikor meg szeretnél tekinteni bizonyos adatokat egy weboldalról, elküldöd a HTTP GET kérést. Ha információt szeretnél küldeni, például kitöltesz egy kapcsolatfelvételi űrlapot, elküldöd a HTTP PUT kérést.

Hasonlóképpen, a szerver különböző típusú HTTP válaszokat küld számkódok és adatok formájában. Íme néhány példa:

- **200** – OK
- **400** – Bad request (Hibás kérés)
- **404** – Resource not found (Az erőforrás nem található)

Ez a kérés-válasz kommunikáció általában láthatatlan a felhasználóid számára. Ez az a kommunikációs módszer, amelyet a böngésző és a webszerverek használnak, így a World Wide Web mindenki számára következetesen működik.

## Hogyan működik a HTTPS protokoll?

A HTTP titkosítatlan adatokat továbbít, ami azt jelenti, hogy a böngészőből küldött információkat harmadik felek elfoghatják és elolvashatják. Ez nem volt ideális folyamat, ezért kiterjesztették HTTPS-re, hogy egy másik biztonsági réteget adjanak a kommunikációhoz. A HTTPS kombinálja a HTTP kéréseket és válaszokat az SSL és TLS technológiával.

A HTTPS weboldalaknak SSL/TLS tanúsítványt kell beszerezniük egy független hitelesítésszolgáltatótól (CA). Ezek a weboldalak megosztják a tanúsítványt a böngészővel az adatcsere előtt, hogy bizalmat építsenek ki. Az SSL tanúsítvány kriptográfiai információkat is tartalmaz, így a szerver és a webböngészők titkosított vagy kódolt adatokat cserélhetnek. A folyamat így működik:

1. Meglátogatsz egy HTTPS weboldalt úgy, hogy beírod a `https://` URL formátumot a böngésződ címsorába.
2. A böngésző megpróbálja ellenőrizni az oldal hitelességét a szerver SSL tanúsítványának kérésével.
3. A szerver válaszként elküldi a nyilvános kulcsot tartalmazó SSL tanúsítványt.
4. A weboldal SSL tanúsítványa bizonyítja a szerver identitását. Miután a böngésző megbizonyosodott, a nyilvános kulcsot használja egy titkos munkamenet-kulcsot tartalmazó üzenet titkosításához és elküldéséhez.
5. A webszerver a privát kulcsát használja az üzenet visszafejtéséhez és a munkamenet-kulcs előhívásához. Ezután titkosítja a munkamenet-kulcsot, és nyugtázó üzenetet küld a böngészőnek.
6. Most a böngésző és a webszerver is ugyanazt a munkamenet-kulcsot használja az üzenetek biztonságos cseréjéhez.

*Tudj meg többet az SSL/TLS tanúsítványokról »*

## Mi a különbség a HTTP/2, a HTTP/3 és a HTTPS között?

Az 1996–97-ben kiadott eredeti HTTP verziót HTTP/1.1-nek nevezték. A HTTP/2 és a HTTP/3 magának a protokollnak a továbbfejlesztett verziói. Az adatátviteli rendszert módosították, hogy hatékonyabb legyen. Például a HTTP/2 bináris formátumban cserél adatokat szöveges formátum helyett. Lehetővé teszi a szerverek számára, hogy proaktívan válaszokat továbbítsanak a kliens gyorsítótáraknak ahelyett, hogy egy új HTTP kérésre várnának. A HTTP/3 viszonylag új, de megpróbálja a HTTP/2-t egy lépéssel tovább vinni. A HTTP/3 célja, hogy hatékonyabban támogassa a valós idejű streamelést és más modern adatátviteli követelményeket.

A HTTPS a HTTP adatbiztonsági aggályait helyezi előtérbe. A modern rendszerek a HTTP/2-t SSL/TLS-sel használják HTTPS-ként. Ahogy a HTTP/3 érik, a böngésző- és szervertechnológia végül ezt is integrálja majd a HTTPS alá.

## Miért válasszuk a HTTPS-t a HTTP helyett?

Az alábbiakban megvitatunk néhány előnyt, amelyet a HTTPS nyújt a HTTP-vel szemben.

### Biztonság

A HTTP üzenetek nyílt szövegűek, ami azt jelenti, hogy jogosulatlan felek könnyen hozzáférhetnek és elolvashatják azokat az interneten. Ezzel szemben a HTTPS minden adatot titkosított formában továbbít. Amikor a felhasználók érzékeny adatokat küldenek be, biztosak lehetnek abban, hogy harmadik felek nem tudják elfogni az adatokat a hálózaton. Jobb a HTTPS-t választani az esetlegesen érzékeny információk, például hitelkártya-adatok vagy az ügyfelek személyes adatainak védelme érdekében.

### Hitelesség

A keresőmotorok általában alacsonyabbra rangsorolják a HTTP weboldalak tartalmát, mint a HTTPS weboldalakat, mivel a HTTP kevésbé megbízható. Az ügyfelek is előnyben részesítik a HTTPS weboldalakat a HTTP-vel szemben. A böngésző láthatóvá teszi a HTTPS kapcsolatot a felhasználók számára egy lakat ikon elhelyezésével a böngésző címsorában a weboldal URL-je mellett. A felhasználók ezen további biztonsági és bizalmi tényezők miatt részesítik előnyben a HTTPS weboldalakat és alkalmazásokat.

### Teljesítmény és analitika

A HTTPS webalkalmazások gyorsabban töltődnek be, mint a HTTP alkalmazások. Hasonlóképpen, a HTTPS jobban követi a ajánló linkeket. Az ajánló forgalom a weboldalad forgalma harmadik féltől származó forrásokból, például hirdetésekből vagy közösségi média visszamutatóból. Engedélyezned kell a HTTPS-t, ha azt szeretnéd, hogy az analitikai szoftver pontosan azonosítsa a megbízható forgalmi forrásaidat.

## Drágább a HTTPS beállítása, mint a HTTP?

A HTTPS megköveteli, hogy SSL/TLS tanúsítványt szerezz be és tarts fenn a szervereden. A múltban a legtöbb hitelesítésszolgáltató éves díjat számított fel a tanúsítvány regisztrációjáért és karbantartásáért. Ez azonban már nem így van.

Számos forrásból szerezhetők be ingyenes SSL tanúsítványok. Például az Amazon Web Services (AWS) kínálja az AWS Certificate Manager (ACM) szolgáltatást. Az ACM biztosítja, kezeli és telepíti a nyilvános és privát SSL/TLS tanúsítványokat, amelyeket AWS szolgáltatásokkal és belsőleg csatlakoztatott erőforrásaiddal használhatsz. Az ACM megszünteti az időigényes manuális folyamatot, ahol SSL/TLS tanúsítványokat kellene vásárolnod, feltöltened és megújítanod.

## A különbségek összefoglalása: HTTP vs. HTTPS

| | HTTP | HTTPS |
|---|---|---|
| **Mit jelent?** | Hypertext Transfer Protocol | Hypertext Transfer Protocol Secure |
| **Alapul szolgáló protokollok** | A HTTP/1 és HTTP/2 TCP/IP-t használ. A HTTP/3 QUIC protokollt használ. | HTTP/2-t használ SSL/TLS-sel a HTTP kérések és válaszok további titkosításához |
| **Port** | Alapértelmezett 80-as port | Alapértelmezett 443-as port |
| **Mire használják?** | Régebbi szövegalapú weboldalak | Minden modern weboldal |
| **Biztonság** | Nincsenek további biztonsági funkciók | SSL tanúsítványokat használ nyilvános kulcsú titkosításhoz |
| **Előnyök** | Lehetővé tette az internetes kommunikációt | Javítja a weboldal hitelességét, bizalmát és keresőmotor-rangsorolását |

## Hogyan támogathatja az AWS a HTTPS követelményeidet?

Ezen a weboldalon áttekintheted azokat az AWS hálózati és tartalomkézbesítési szolgáltatásokat, amelyek alapértelmezés szerint támogatják a HTTPS-t és az SSL/TLS-t.

Az **Amazon Lightsail** lehetővé teszi, hogy gyorsan építs alkalmazásokat és weboldalakat alacsony költségű, előre konfigurált felhőerőforrásokkal. Emellett a Lightsail terheléselosztókat használhatsz biztonságos alkalmazások építéséhez és HTTPS forgalom fogadásához. A Lightsail megkönnyíti az SSL/TLS tanúsítványok kérését, biztosítását és karbantartását. A beépített tanúsítványkezelés a nevedben kéri és újítja meg a tanúsítványokat, és automatikusan hozzáadja a tanúsítványt a terheléselosztódhoz.

Az **Amazon CloudFront** három lehetőséget kínál a teljes weboldalad gyorsítására, és biztonságosan szállítja a tartalmat HTTPS-en keresztül az összes CloudFront edge helyről. Amellett, hogy biztonságosan szállít az edge-ről, a tartalomkézbesítő hálózatot (CDN) úgy is konfigurálhatod, hogy HTTPS kapcsolatokat használjon az origin lekérésekhez. Ez azt jelenti, hogy az adataid végpontok közötti titkosítással vannak védve az origin-től a felhasználóidig.

*Kezdje el a HTTPS-t az AWS-en egy ingyenes fiók létrehozásával még ma*
