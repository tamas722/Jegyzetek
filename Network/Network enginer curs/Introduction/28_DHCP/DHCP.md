# Mi az a DHCP? És miért fontos?

## Összefoglaló

**Nézd és tanulj Suzanne-től: DHCP alapok 20 perc alatt**

- A DHCP szerver által küldött konfigurációs adatok és kulcsértékek
- DHCP bérleti idő kezelése
- DHCP használati forgatókönyvek
- DHCP opciók
- A DHCP szolgáltatás megvalósítása
- GYIK
  - Hogyan rendel a DHCP szerver IP-címeket?
  - Milyen előnyei vannak a DHCP használatának egy hálózatban?
  - Mi történik, ha a DHCP szerver meghibásodik?
  - Használható a DHCP vezetékes és vezeték nélküli hálózatokban is?

**Iratkozz fel hírlevelünkre**

## Kapcsolódó terminológia

- Mi az a DDI? (DNS-DHCP-IPAM)
- Mi az a DNS?
- Mi az az IPAM?
- Mi az a DHCP Lease?
- Mi az a DHCP Option?
- Mi az a DHCP Fingerprinting?

**Tudj meg többet**

*Miért a DHCP a leg alulbecsültebb hálózati szolgáltatás*

## Mi a DHCP – DHCP elv

A **Dynamic Host Configuration Protocol (DHCP)** egy szabványos hálózati protokoll, amely automatizálja az IP-címek és más konfigurációs adatok hozzárendelését a hálózaton lévő eszközökhöz. DHCP nélkül a hálózati adminisztrátoroknak minden eszközt manuálisan kellene konfigurálniuk – ez időigényes és hibalehetőségekkel teli feladat. A DHCP egyszerűsíti ezt azzal, hogy dinamikusan osztja el a hálózati konfigurációs információkat, biztosítva a hatékony és skálázható hálózatkezelést.

A DHCP lehetővé teszi az adminisztrátorok számára, hogy olyan hálózati szolgáltatásokat használjanak, mint a DNS, NTP, és bármely UDP vagy TCP alapú kommunikációs protokoll. A DHCP szerver dinamikusan rendel IP-címet és más hálózati konfigurációs paramétereket minden eszközhöz a hálózaton, hogy azok kommunikálhassanak más IP-hálózatokkal. A DHCP egy régebbi, BOOTP nevű protokoll továbbfejlesztése.

### Nézd és tanulj Suzanne-től: DHCP alapok 20 perc alatt

Ez a videó a DHCP demisztifikálva képzésünk kivonata, amely a DDI bevezető kurzusunkban érhető el, amely DHCP-ből, DNS-ből és IPAM-ból áll. A videó megtekintésével megtanulod a DHCP szerepét, a DHCP protokoll alapelveit és az üzenetáramlásokat egy kliens és egy szerver között.

## A DHCP szerver által küldött konfigurációs adatok és kulcsértékek

Az alapvető folyamat az, hogy a DHCP szerver konfigurációs adatokat oszt ki az adminisztrátor szabályzata alapján a kérést küldő kliensnek. A gyakran kért hálózati paraméterek (néha „DHCP opcióknak" nevezik) közé tartozik az alhálózati maszk, a router, a domainnév szerver, a hostnév és a domainnév.

Mivel a kérést küldő kliensnek nincs IP-címe, amikor csatlakozik a hálózathoz, broadcast üzenetet küld. A protokollt így az IP-kommunikáció nagyon korai szakaszában használják. Ha ilyen dinamikus protokollt nem használnak IP-cím megszerzésére, a kliensnek egy előre meghatározott IP-címet kell használnia, amelyet általában „statikus IP-címnek" neveznek, és amelyet manuálisan konfigurálnak a kliens hálózati interfészén konfigurációs fájlokban vagy egy adott paranccsal.

A DHCP szolgáltatás három kulcsértéket hoz:

- **Az üzemeltetési feladatok csökkennek:** a hálózati adminisztrátornak már nem kell manuálisan konfigurálnia minden klienst, mielőtt az használhatná a hálózatot
- **Az IP-címzési terv optimalizálódik:** a már nem használt címek felszabadulnak, és elérhetővé válnak az újonnan csatlakozó kliensek számára
- **A felhasználói mobilitás könnyen kezelhető:** az adminisztrátornak nem kell manuálisan újrakonfigurálnia egy klienst, amikor annak hálózati hozzáférési pontja megváltozik

## DHCP bérleti idő kezelése

A DHCP által hozzárendelt IP-cím információ csak korlátozott ideig érvényes, és ezt **DHCP bérletnek** (DHCP lease) nevezik. Az érvényességi időt **DHCP bérleti időnek** (DHCP lease time) hívják. Amikor a bérlet lejár, a kliens már nem használhatja az IP-címet, és le kell állítania minden kommunikációt az IP-hálózattal, hacsak nem kéri a bérlet „bérleti díjának" meghosszabbítását a DHCP bérleti megújítási cikluson keresztül. Annak elkerülésére, hogy a DHCP szerver ne legyen elérhető a bérleti idő végén, a kliensek általában a bérleti idő felénél kezdik megújítani a bérletüket. Ez a megújítási folyamat biztosítja az eszközök robusztus IP-cím-kiosztását. Bármely eszköz, amely új IPv4-címet kér a hálózatra érkezéskor, és nem kap választ, automatikus privát internetprotokoll-címzést (APIPA) használ egy cím kiválasztásához. Ezek a címek a 169.254.0.0/16 hálózati tartományban vannak.

*Mi az a DHCP-DHCP lease*

## DHCP használati forgatókönyvek

Négy kulcsfontosságú DHCP használati forgatókönyv létezik:

- **Kezdeti kliens csatlakozás:** a kliens IP-címet és más paraméterértékeket kér a DHCP szervertől a hálózati szolgáltatások eléréséhez
- **IP-használat meghosszabbítása:** a kliens kapcsolatba lép a DHCP szerverrel, hogy meghosszabbítsa jelenlegi IP-címének használatát
- **Kliens csatlakozás újraindítás után:** a kliens kapcsolatba lép a DHCP szerverrel, hogy megerősítse, hogy ugyanazt az IP-címet használhatja, amelyet az újraindítás előtt használt
- **Kliens leválasztás:** a kliens kéri a DHCP szervertől, hogy szabadítsa fel az IP-címét

## DHCP opciók

A DHCP opciók arra használhatók, hogy automatikusan információt nyújtsanak a klienseknek az általuk használható hálózati szolgáltatásokról. Ez egy nagyon hatékony módja annak, hogy az időszinkronizáló szerver, a levelezőszerver, a DNS szerver és a nyomtatósszerver IP-címét eljuttassák a kliensekhez. Ez arra is használható, hogy egy fájlnevet és egy fájlszervert biztosítsanak, amelyet a kliens egy adott boot folyamat indításához használ – főként IP-telefonokhoz és Wi-Fi hozzáférési pontokhoz használják, de használható kliensek és szerverek automatikus telepítéséhez is PXE-vel (Preboot eXecution Environment).

*Mi az a DHCP-DHCP opciók*

## A DHCP szolgáltatás megvalósítása

A DHCP szolgáltatás eredeti és legátfogóbb megvalósítását az **Internet Systems Consortium (ISC)** kínálja. Mind az IPv4, mind az IPv6 támogatásával az ISC DHCP egy teljes nyílt forráskódú megoldást kínál DHCP szerverek, relay agentek és kliensek megvalósításához. Más DHCP szerver termékek közé tartozik a Microsoft DHCP szerver.

A DHCP szolgáltatás DHCP failoverrel bővíthető a magas rendelkezésre állás és a forgalom terheléselosztása érdekében. Az ISC DHCP Failover együttműködő szerverpárra támaszkodik – egy elsődleges (master) szerverre és egy másodlagos (backup) szerverre. Ezután egy TCP-alapú kommunikációs csatornát, úgynevezett failover csatornát kell létrehozni a két szerver között.

## GYIK

### Hogyan rendel a DHCP szerver IP-címeket?

A DHCP szerver egy négy lépésből álló folyamattal rendel IP-címeket: **Discover, Offer, Request és Acknowledge** (általában DORA-ként rövidítve).

- **DHCP Discover:** A kliens broadcast üzenetet küld, hogy megtaláljon minden elérhető DHCP szervert.
- **DHCP Offer:** A szerver válaszol egy elérhető IP-címmel és más konfigurációs részletekkel.
- **DHCP Request:** A kliens elfogadja az ajánlatot, és formálisan kéri a felajánlott konfigurációt.
- **DHCP Acknowledge:** A szerver megerősíti a hozzárendelést, és a kliens elkezdi használni az IP-címet.

A konfigurációs adatok Type-Length-Value formátumban vannak csomagolva, ahol minden mező oktettekkel van ábrázolva. Ezek a mezők az RFC dokumentumokban meghatározott szabályokat követik, például az RFC 2131-et a DHCPv4-hez és az RFC 8415-öt a DHCPv6-hoz. Ez biztosítja, hogy a kicserélt információ szabványosított legyen, és minden megfelelő eszköz megértse.

### Milyen előnyei vannak a DHCP használatának egy hálózatban?

A DHCP használata számos előnyt kínál, különösen közepes és nagy léptékű hálózatokban:

- **Egyszerűsített hálózatkezelés:** Az eszközök automatikusan konfigurálódnak, csökkentve a manuális beállítást és az adminisztratív erőfeszítést.
- **Skálázhatóság:** A DHCP támogatja a nagy hálózatokat az IP-címkészletek dinamikus kezelésével.
- **Következetesség:** Biztosítja az egységes hálózati konfigurációt minden eszközön.
- **Rugalmasság:** Az olyan opciók, mint az alapértelmezett átjáró, a DNS szerver és a bérleti idők, könnyen kezelhetők és frissíthetők.
- **Hibacsökkentés:** Kiküszöböli a manuális beviteli hibákat, amelyek gyakran előfordulnak statikus konfiguráció során.

Emellett a DHCP megfelel a nyílt RFC szabványoknak, lehetővé téve az interoperabilitást több gyártó és platform között. Több alhálózatból is hozzárendelhet címeket, és szükség szerint dinamikusan újraoszthatja az IP-címeket.

### Mi történik, ha a DHCP szerver meghibásodik?

Ha egy DHCP szerver meghibásodik, és nincs redundancia, az új eszközök nem tudnak IP-címet szerezni, gyakorlatilag elveszítve a hálózati hozzáférést. Ez hatással lehet a műveletekre olyan környezetekben, ahol az IP-cím-kiosztás kulcsfontosságú, például VoIP rendszerekben vagy nagy vállalati hálózatokban.

Ennek megelőzésére sok szervezet failover vagy redundáns DHCP szervereket telepít, amelyek megosztják a bérleti információkat, és meghibásodás esetén zökkenőmentesen átvehetik a feladatot. A szabványosított protokollok használata biztosítja, hogy a failover mechanizmusok következetesen és megbízhatóan működjenek, még több gyártó környezetében is.

### Használható a DHCP vezetékes és vezeték nélküli hálózatokban is?

Igen, a DHCP-t mind vezetékes, mind vezeték nélküli hálózatokban használják. Függetlenül az adathordozótól, bármely hálózathoz csatlakozó eszköz általában DHCP-n keresztül kér IP-címet. Vezetékes hálózatokban ez a folyamat Etherneten keresztül történik. Vezeték nélküli környezetekben a kérés akkor történik, amikor az eszköz csatlakozik egy Wi-Fi hozzáférési ponthoz.

A DHCP mindkét adathordozóban következetes marad, mivel ugyanazokat a szabványosított folyamatokat és formátumokat követi, amelyeket az RFC specifikációk határoznak meg. A DHCP üzenet formátuma – beleértve a fejléceket, opciókat és oktettstruktúrát – ugyanaz marad, biztosítva az egységes viselkedést az eszközök és platformok között.
