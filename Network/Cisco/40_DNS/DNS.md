# CCNA Bővített Jegyzet – 40. Lecke: A Domain Name System (DNS)

Ebben a leckében rátérünk a hálózatok (és a teljes Internet) "telefonkönyvére": a **DNS**-re. Mivel az emberek a nevekhez kötődnek (pl. `youtube.com`), a gépek és a routerek viszont kizárólag IP-címekkel dolgoznak (pl. `172.217.25.110`), szükségünk van egy fordítószolgálatra, ami a kettőt összeköti.

> (A CCNA vizsgatémák 4.3-as pontja csak a DHCP és a DNS szerepének ismeretét várja el, a mély belső működés nem követelmény, de a hibakereséshez érteni kell a folyamatot).

## 1. A DNS Alapműködése és Szereplői

A kliens gépek (PC-k, okostelefonok) általában automatikusan (a DHCP protokollon keresztül) kapják meg, hogy ki az a DNS szerver (pl. a Google híres `8.8.8.8`-as szervere), akihez fordulhatnak kérdés esetén. Bár egy PC-n manuálisan is beállítható a DNS szerver, egy vállalatnál sosem csinálunk ilyet, mert skálázhatatlan.

### Hogyan kommunikál?

- **Protokoll:** A DNS szabványos lekérdezésekhez (Query/Response) az **UDP 53-as portot** használja, mert gyors és kevés adatot (egy IP címet) kell átvinni.
- **A TCP Szabály:** Ha egy DNS válasz (vagy két szerver közötti adatcsere, pl. Zone Transfer) **meghaladja az 512 byte-ot**, a DNS protokoll automatikusan átvált **TCP 53-as portra**, hogy garantálja a megbízhatóságot! Ezt a határértéket (512 byte) és a protokoll-váltást mindenképp tudni kell a vizsgára!

### 1.1. DNS Rekordtípusok (A Legfontosabbak)

Amikor a PC megkérdezi a DNS szervert, megmondja, hogy pontosan milyen típusú bejegyzést (Rekordot) keres.

- **A Rekord (Address Record):** Ez egy **IPv4-es** IP-címet ad vissza a tartománynévhez.
- **AAAA Rekord (Quad-A Record):** Ez egy **IPv6-os** IP-címet ad vissza! Mivel az IPv6 128 bites, ami pont a négyszerese az IPv4 (32 bit) címének, ezért hívják "Quadruple A"-nek, azaz négyszeres A-nak.
- **CNAME (Canonical Name):** Egy **Álnév (Alias)** rekord. Egy nevet (pl. `www.google.com`) nem IP-címre, hanem egy másik domain névre fordít le, és onnan megy tovább a keresés (pl. az eredeti kiszolgáló szerver nevére).

## 2. A Hibakeresés Alapjai Kliens (Windows) Oldalon

Bár a CCNA a Cisco vizsgája, a CCNA vizsgakövetelmények explicit módon előírják az operációs rendszerek alapvető IP paramétereinek verifikálását, így pár Windows parancssori eszközt (Command Prompt) kívülről kell fújni.

### A DNS Kliens Gyorsítótára (DNS Cache)

Hogy a számítógéped ne árassza el az Internetet DNS kérésekkel, amikor másodpercenként 10-szer töltesz be egy weblapot a benne lévő képek és scriptek miatt, a PC minden lefordított nevet elment a saját, belső gyorsítótárába (Cache).

- `ipconfig /all`: Megmutatja az összes hálózati beállítást, a MAC-címtől kezdve a beállított DNS-szerverek címéig.
- `nslookup <domain.név>` (Name Server Lookup): Dedikált DNS parancs. Arra kényszeríti a PC-t, hogy bekérdezzen a beállított DNS szervernek, hogy kihez tartozik a megadott név.
- `ipconfig /displaydns`: Kilistázza a memóriában éppen eltárolt összes név-IP párost (A gyorsítótár tartalmát).
- `ipconfig /flushdns`: **Hibakeresésnél a legfontosabb!** Azonnal kitörli (üríti) a gyorsítótárat. Ha a hálózatban kicseréltek egy szervert egy másik IP-címűre, a PC továbbra is a régire próbál majd kapcsolódni a beragadt Cache miatt. A Flush után kénytelen lesz új DNS kérést indítani, és megtanulja az új IP-t.

### Az Eredeti Megoldás (A Hosts fájl)

Mielőtt kitalálták volna a DNS rendszert, a hálózatba kötött gépek a `C:\Windows\System32\drivers\etc\hosts` útvonalon lévő egyszerű, helyi szöveges fájlból (Hosts) nézték ki, hogy ki kicsoda. (A Linux/Mac rendszerekben ez az `/etc/hosts` alatt van). Ha egy rendszergazda trükközni akar, beleírhat IP-Név párosokat, amik megelőzik a DNS kérést! (A PC előbb mindig megnézi a Hosts fájlt, ha ott megtalálja a nevet, már el sem küldi a kérdést a DNS szerver felé!).

## 3. A Router Szerepe a DNS-ben (Átenged, Kliens vagy Szerver)

A legfontosabb alapszabály: **A routernek alapesetben AZ ÉGVILÁGON SEMMIT nem kell tudnia a DNS-ről** ahhoz, hogy a mögötte lévő PC-k internetezni tudjanak! A router csak fogja a `8.8.8.8` (Google DNS) felé tartó UDP 53-as (DNS) csomagot, megnézi a Routing tábláját, és kidobja a megfelelő porton. Nincs szükség speciális konfigurációra.

Azonban a Cisco IOS operációs rendszert mi is megtaníthatjuk használni a DNS-t, akár úgy, mint egy **Klienst** (hogy tudjunk a konzolból nevekre pingelni), akár úgy, mint egy teljes értékű **Szervert** (Bár a gyakorlatban, enterprise szinten sosem bízzák a routerre a DNS szerver szerepét, arra mindig dedikált Windows / Linux szervereket használnak).

### 3.1. A Router, mint DNS Kliens (Host)

Ahhoz, hogy a router parancssorában beírhassuk, hogy `ping google.com`, a routernek is fel kell vennie a kapcsolatot egy DNS szerverrel.

- Engedélyezni kell a névfeloldást (Alapesetben be van kapcsolva!): `ip domain lookup` (A régi IOS verziókban ez `ip domain-lookup` volt kötőjellel).
- Meg kell mondani a gépnek, kit kérdezzen meg, ha nevet talál: `ip name-server 8.8.8.8` (Ez a DNS szerver IP-címe).
- (Opcionális) Saját tartománynév (Domain name) beállítása: `ip domain name jeremysitlab.com`. Ha ezt megadjuk, és azt írjuk a konzolba, hogy `ping Szerver1`, a router okosan kiegészíti a nevet a DNS kérésben így: `Szerver1.jeremysitlab.com`. (Erre a parancsra az SSH titkosított hozzáférés beállításánál is égető szükség lesz!).

### 3.2. A Router, mint DNS Szerver

Képesek vagyunk a router memóriájában is felépíteni egy "Hosts" fájl szerű táblázatot (hogy ki-milyen IP), majd a routert megbízni azzal, hogy válaszoljon a hálózatban lévő többi PC DNS kéréseire.

- Bekapcsoljuk a DNS szerver szolgáltatást a gépen: `ip dns server`
- Felvissük kézzel a Fix neveket: `ip host PC1 192.168.0.101`

(És ahogy az előbb láttuk, ha be van állítva egy `ip name-server 8.8.8.8` is, akkor a mi kis DNS routerünk, ha nem talál meg egy nevet a saját memóriájában, **Forwarderként** (Továbbítóként) átküldi a kérdést a Google-nek, megvárja a választ, és azt küldi vissza a mi PC-nknek!).

### Ellenőrzés a CLI-ben

A `show hosts` paranccsal kilistázható a router név-táblázata. Itt látszanak a manuálisan (`ip host`) felvitt állandó gépek (A "Perm" felirattal), és az internetről frissen megtanult és cache-elt ideiglenes címek (A "Temp" felirattal).

## Kvíz a 40. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod a DNS elméleti és parancssori működéséből!

### Kérdés 1

Melyik **KÉT** Windows Parancssori (Command Prompt) utasítás mutatja meg a te gépeden aktuálisan beállított DNS szerver IP-címét? (Válassz KETTŐT!)

- A) `ipconfig`
- B) `ipconfig /all`
- C) `ipconfig /displaydns`
- D) `nslookup`

**Helyes Válasz: B és D**

**Magyarázat:** A sima `ipconfig` csak a te saját IP-címedet, maszkot és a default gateway-t mutatja, a DNS szervert elrejti a gyorsaság miatt. Az `/all` kapcsoló az, ami feltárja az összes hálózati paramétert (beleértve a MAC-címet (Physical Address), DHCP szervert, DNS szervert). A D (`nslookup`) parancs lefuttatása legelső sorában azonnal kiírja a Default Server-t, akihez be van kötve a géped. A C csak az ideiglenes (cache) memóriát listázza.

### Kérdés 2

Melyik **KÉT** állítás IGAZ az Internetes Domain Name System-re (DNS) vonatkozóan? (Válassz KETTŐT!)

- A) A DNS kizárólag UDP-t használ a 512 byte-nál nagyobb, hosszú üzenetek küldésére (mert az kevesebb memóriát eszik).
- B) Az A rekord (A Record) a domain neveket egy IPv4-es IP címre fordítja le.
- C) A AAA (Triple-A) rekord a domain neveket IPv6-os IP címre fordítja le.
- D) Egy Cisco router egyidejűleg tud DNS Kliensként és DNS Szerverként is működni a hálózatban.

**Helyes Válasz: B és D**

**Magyarázat:** Az A rekord a leggyakoribb IPv4-es címzés. A Cisco router tényleg tud egyszerre kérdezni a külső szerverektől, és válaszolni a belső hálózat kéréseire (Amiatt ez a két funkció tökéletesen megfér egymás mellett). Az A azért hamis, mert az 512 byte fölötti nagy csomagok kötelezően áttérnek a megbízható TCP-re! A C azért hamis, mert az IPv6-os címekhez 4 darab A betű (Quadruple-A, azaz **AAAA** rekord) tartozik! (A Triple-A az egy hálózati biztonsági keretrendszer neve - Authentication, Authorization, Accounting).

### Kérdés 3

A mögötted lévő PC1 be van állítva úgy, hogy a `8.8.8.8`-as (Google) külső szervert használja DNS-nek. Te adminisztrálod az R1-es (Köztes) routert, ami az internetre köt. Milyen konfigurációs parancsot **KELL** kiadnod az R1-en, hogy a PC1 DNS lekérdezései eljussanak a Google-höz?

- A) `ip dns server`
- B) `ip name-server 8.8.8.8`
- C) `ip domain lookup`
- D) Nincs szükség semmilyen DNS-sel kapcsolatos parancsra az R1-en.

**Helyes Válasz: D**

**Magyarázat:** Ez egy becsapós kérdés! A DNS csomag az a router szemében csak egy teljesen hétköznapi UDP csomag. A router dolga a csomagok továbbítása (Routolás). Amíg az R1 tudja (a routing táblájából vagy egy default route-ból), hogy merre van az Internet és a `8.8.8.8`-as IP-cím, minden további nélkül át fogja engedni (forwarding) a PC kérését.

### Kérdés 4

Melyik Cisco IOS paranccsal tudod kilistázni a router memóriájában lévő, manuálisan konfigurált és a DNS-ből megtanult (ideiglenesen elmentett) Hosztneveket és a hozzájuk tartozó IP-címeket?

- A) `show hosts`
- B) `show ip dns`
- C) `ipconfig /displaydns`
- D) `show dns-cache`

**Helyes Válasz: A**

**Magyarázat:** A Cisco IOS erre a rendkívül egyszerű `show hosts` parancsot használja. A C parancs a Windows OS-é, a többi meg nem létező, kitalált IOS parancs.

### Bónusz Kérdés (Boson ExSim vizsgakérdés - A TCP/UDP Működési Logikája)

A HostA számítógépen valaki megnyit egy webböngészőt, és beírja a WWW_server címét. A HostA soha életében nem kereste még ezt a szervert (nincs a cache-ben), és nem használ belső "Hosts" fájlt sem.

Az alábbi három eszköz (DNS Szerver, Default Gateway, WWW_Szerver) közül melyikkel (Vagy melyekkel) fog a HostA **TCP-KAPCSOLATOT (TCP Connection)** kiépíteni ebben a forgatókönyvben? Válaszd a legjobb választ!

- A) Csak a DNS Szerverrel és a WWW Szerverrel.
- B) A DNS Szerverrel, a Default Gateway-jel és a WWW Szerverrel is.
- C) Csak a Default Gateway-jel és a WWW Szerverrel.
- D) KIZÁRÓLAG a WWW Szerverrel.

**Helyes Válasz: D**

**Magyarázat:** Elemezzük a forgalmat az OSI modell és a protokollok szintjén!

1. A HostA-nak meg kell kérdeznie a DNS Szervert a weboldal IP-címéért (Mivel a gyorsítótár üres). A DNS lekérdezés szabványosan **UDP** (Connectionless) csomagon megy ki. Nincs kapcsolatépítés (3-Way Handshake)!
2. A csomag áthalad a Default Gateway-en (A Helyi Routeren). A router működése tisztán Layer 3 (IP címzés) szintű (Irányítja a csomagot). Nem épít fel semmilyen Layer 4-es (TCP) kapcsolatot a gépekkel (Hacsak nem a konzoljára lépsz be SSH-val, de az most nem áll fenn).
3. A HostA végre megkapja a DNS-től az IP-címet, és elindítja a HTTP (Webes) kérést a WWW Szerver felé. A HTTP webes forgalom kötelezően a **TCP** (80-as port) protokollt használja! Mielőtt a weblap letöltődne, a HostA-nak végre kell hajtania egy TCP 3-Way Handshake-t a WWW Szerverrel.

Tehát az egyetlen eszköz, amivel a TCP kapcsolat felépül, a **WWW_Szerver (D)**. Ez a kérdés ragyogóan fésüli össze a DNS (UDP) és a HTTP (TCP) elméleti tudást a Layer 3-as továbbítással!
