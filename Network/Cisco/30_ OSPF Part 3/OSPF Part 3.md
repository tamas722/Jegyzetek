# CCNA Bővített Jegyzet – 30. Lecke: OSPF 3. Rész (Hálózattípusok, Szomszédsági Feltételek és LSA-k)

Ebben a videóban lezárjuk a CCNA szintű OSPF tanulmányainkat. Ahogy a kérésedben is szerepelt, ez a jegyzet kivétel nélkül minden apró technikai részletet, Cisco parancsot és érdekességet tartalmaz az átiratból, hogy tökéletesen felkészült légy!

## 1. A Loopback Interfészek (Virtuális Interfészek) Szerepe

A korábbi leckékben már használtuk őket a Router ID beállításához, de mi is pontosan a Loopback interfész?

Egy Loopback interfész a router szoftverében létező **virtuális interfész**. Legnagyobb előnye, hogy **MINDIG up/up állapotban van** (kivéve, ha te magad adod ki rajta a `shutdown` parancsot).

### Miért jó ez?

Ha a hálózat többi része az R1-es routert az egyik fizikai interfészének IP-címén (pl. `10.0.13.1`) próbálja elérni, és az a kábel elszakad, a router elérhetetlenné válik azon az IP-n (hiába van másik, működő kábele is). Ha viszont a forgalmat a Loopback interfész (pl. `1.1.1.1`) felé irányítjuk, az a router bármelyik működő fizikai kábelén keresztül elérhető marad! Ezért kritikus, hogy a Router ID-t egy rendíthetetlen Loopback cím adja.

## 2. OSPF Hálózattípusok (Network Types)

Az OSPF aszerint változtatja a viselkedését, hogy milyen technológiájú kábellel vannak összekötve a routerek. Bár a modern hálózatokban szinte csak az Ethernetet használjuk, az OSPF vizsgán tudni kell a klasszikus felosztást:

- **Broadcast (Szórásos) hálózattípus:** Alapértelmezett az Ethernet és az FDDI interfészeken.
- **Point-to-Point (Pont-tól pont-ig):** Alapértelmezett a Soros (Serial) kapcsolatokon, amelyek PPP vagy HDLC tokozást (encapsulation) használnak.
- **Non-broadcast (Nem-szórásos):** Alapértelmezett a Frame Relay és az X.25 hálózatokon. (Érdekesség: Itt a routerek nem fedezik fel automatikusan a szomszédokat, azokat manuálisan kell konfigurálni a CLI-ben!). Van egy **Point-to-multipoint** altípusa is, de a CCNA-n ezt nem kérik számon.

### 2.1. A Broadcast Hálózattípus Részletei (Ethernet)

Mivel az Ethernet hálózatokon egyetlen switchhez akár 10 router is csatlakozhat, az OSPF-nek meg kell akadályoznia, hogy mindenki mindenkivel megossza az LSA adatbázisát (ez átláthatatlan káoszt és LSA-vihart okozna). Ezt a **DR/BDR rendszerrel** oldja meg.

- **DR (Designated Router - Kijelölt Router):** A hálózati szegmens "főnöke". Mindenki neki küldi a frissítéseket, és ő osztja szét.
- **BDR (Backup Designated Router - Tartalék):** Ha a DR meghal, azonnal átveszi a helyét.
- **DROther:** A többi mezei router.

### Hogyan kommunikálnak?

- A normál OSPF Hello üzeneteket mindenki a **`224.0.0.5`**-ös (All OSPF Routers) Multicast címre küldi.
- Amikor egy DROther frissítést akar küldeni a DR-nek és a BDR-nek, azt a dedikált **`224.0.0.6`**-os Multicast címre küldi (így a többi DROther nem is foglalkozik vele).

### 2.2. A DR / BDR Választás (Election) Szabályai

A választás minden egyes hálózati szegmensen (Subnet) külön-külön lezajlik!

1. **A Legmagasabb OSPF Interfész Prioritás nyer** (Alapértelmezett érték: **1**, a tartomány 0-tól 255-ig terjed). Ha egy port prioritását 0-ra állítod, az a router **SOHA** nem lehet DR vagy BDR!
   - Parancs interfész módban: `ip ospf priority 255`
2. **Döntetlen (alapértelmezés) esetén a Legmagasabb OSPF Router ID nyer.**

> **A "Non-Preemptive" (Nem kisajátító) szabály:**
> **Kritikus vizsgatéma!** A DR/BDR választás **nem pre-emptív**. Ez azt jelenti, hogy ha a választás már lezajlott (az R5 lett a DR, az R4 a BDR), és te utólag belépsz az R2-re és feltolod a prioritását 255-re, az R2 **NEM** fogja azonnal átvenni a DR szerepet! Egészen addig DROther marad, amíg az OSPF folyamatot újra nem indítják, vagy a jelenlegi DR/BDR meg nem hal.
> Ha a DR (R5) kiesik, az eddigi BDR (R4) azonnal DR-ré lép elő (hiába van kisebb prioritása, mint az R2-nek!), és a megüresedett BDR posztra választják meg az R2-t. Ezt az OSPF folyamat törlésével lehet siettetni: `clear ip ospf process`.

### 2.3. Szomszéd (Neighbor) vs. Szomszédság (Adjacency)

A Boson vizsgakérdés is erre épül:

- A **DROther** routerek egymás között csak **2-Way (Kétirányú)** állapotig jutnak el. Ők OSPF **szomszédok (Neighbors)**, de **NEM** építenek fel Teljes Szomszédságot (Adjacency), azaz nem cserélnek LSA-kat egymással!
- A **FULL (Teljes)** szomszédságot a DROther routerek **KIZÁRÓLAG** a DR és a BDR routerekkel építik ki! (A DR/BDR pedig mindenkivel FULL állapotban van).

## 3. A Point-to-Point Hálózattípus (Soros Kapcsolatok Gyorstalpalója)

A Soros (Serial) kapcsolatok elavultak, de az OSPF pont-tól-pontig működésének megértéséhez tudni kell róluk párat:

- Egy Soros kábelnek két vége van: a **DCE (Data Communications Equipment** - Ő diktálja a sebességet) és a **DTE (Data Terminal Equipment)**.
- A DCE oldalon a fizikai porton be kell állítani az órajelet (sebességet) bit/sec-ben: `clock rate 64000`. (Szemben az Ethernettel, ahol a `speed` parancsot használjuk).
- Ezt a `show controllers s2/0` paranccsal tudod ellenőrizni (kiírja, hogy DCE vagy DTE).
- A Cisco routereken a soros kábelek tokozása alapból **HDLC** (pontosabban a Cisco sajátja, a cHDLC). Ezt a porton az `encapsulation ppp` paranccsal lehet átállítani PPP-re (fontos: a kábel két végén egyeznie kell!). MAC-címeket itt nem használnak.

### OSPF viselkedés Point-to-Point módban

Mivel a kábelen csak két router van, értelmetlen DR/BDR-t választani! Az OSPF itt **NEM választ DR-t és BDR-t**, a két router azonnal FULL szomszédságot épít ki egymással (A `show ip ospf neighbor` kimenetben a státusznál egy kötőjel `-` látszik a DR helyett).

> **Tipp:** Ha két routert egy sima Ethernet kábellel kötsz össze közvetlenül, manuálisan is átállíthatod a portokat Point-to-Point módba, hogy megspórold a felesleges DR választást: `ip ospf network point-to-point`. (Figyelem: A Hello/Dead timerek itt is 10/40 másodperc maradnak, akárcsak a Broadcastnál. A Non-broadcastnál lennének 30/120-ak).

## 4. OSPF Szomszédsági Feltételek (Miért nem áll fel a Neighbor?)

Az OSPF troubleshooting legfontosabb része. Ahhoz, hogy két router OSPF szomszéd lehessen (2-way vagy FULL), az alábbiaknak kell teljesülniük:

- **Area (Terület) egyezés:** Mindkét portnak ugyanabban a területben (pl. Area 0) kell lennie.
- **Subnet (Alhálózat) egyezés:** Az IP-címeknek és a maszkoknak egyezniük kell a két porton.
- **A Processzis futása:** Az OSPF nem lehet leállítva (`router ospf 1` menüben kiadható a `shutdown` parancs, amivel leállítod az OSPF-et anélkül, hogy törölnéd a configot).
- **Egyedi Router ID-k:** Ha a hálózatban két gépnek azonos a Router ID-je, a szomszédság nem áll fel. Hibaüzenet: `%OSPF-4-DUP_RTRID_PRFX: OSPF detected duplicate router-id...`
- **Hello és Dead Timerek egyezése:** Ha a porton manuálisan átírod az egyiket (`ip ospf hello-interval 5` vagy `ip ospf dead-interval 20`), a szomszédság megszakad!
- **Hitelesítés (Authentication) egyezése:** Jelszavas védelem esetén mindkét oldalon egyeznie kell a kulcsnak.
  - `ip ospf authentication-key jeremy` (Beállítja a jelszót).
  - `ip ospf authentication` (Élesíti a védelmet a porton).
- **IP MTU egyezés:** Ha az egyik porton átírod az MTU-t (`ip mtu 1400`), a szomszédok látszólag megtalálják egymást (Init, 2-Way), de a nagy LSA adatbázis cseréjénél elakadnak! A kapcsolat végtelenül beragad az **EXSTART** állapotba!
- **Hálózattípus (Network Type) egyezés:** A legszemetebb hiba! Ha az egyik port Broadcast, a másik Point-to-Point módban van, a szomszédság felépül (FULL lesz), de a Routing Table (útválasztási tábla) **NEM** fog frissülni az útvonalakkal, mert a két gép LSA struktúrája inkompatibilis lesz!

## 5. Az OSPF LSA Típusok (1, 2 és 5)

Az OSPF vizsgán 11 LSA típusból csak a legfontosabbakat kell ismerni. Az LSDB-t a `show ip ospf database` paranccsal tudod lekérdezni.

- **Type 1 (Router LSA):** Minden egyes OSPF router generálja. Saját magát azonosítja benne (Router ID), és felsorolja a rajta aktív OSPF interfészeket és azok költségeit.
- **Type 2 (Network LSA):** **KIZÁRÓLAG A DR** (Designated Router) generálja a Multi-access (Ethernet Broadcast) hálózatokon. Felsorolja az összes routert, ami az adott szegmenshez csatlakozik. (Ha egy routernek van olyan portja, ahol egyedül van – pl. Point-to-Point – ott nem generálódik Type 2 LSA).
- **Type 5 (AS-External LSA):** **KIZÁRÓLAG AZ ASBR** (Autonomous System Boundary Router) generálja. Olyan útvonalakat tartalmaz, amelyek az OSPF hálózaton kívülről származnak (Például egy Default Route, amit a `default-information originate` paranccsal fecskendeztek be).

> (Kvíz Érdekesség: A Type 3 a Summary LSA, ami az ABR-eken jön létre a területek közötti kommunikációhoz).

## Kvíz a 30. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod az OSPF mélyebb rejtelmeiről!

### Kérdés 1

Melyik állítás mutat be olyan tulajdonságot az OSPF Point-to-Point hálózattípusban, amely Eltér az OSPF Broadcast hálózattípustól?

- A) DR és BDR választás történik.
- B) DR és BDR választás NEM történik.
- C) A szomszédokat dinamikusan fedezi fel.
- D) A szomszédokat NEM dinamikusan fedezi fel.

**Helyes Válasz: B**

**Magyarázat:** A Point-to-Point (Mivel eleve csak két gép lehet a vonalon) kihagyja a felesleges DR/BDR választást. A Broadcast hálózatok ezzel szemben választanak. A szomszédokat mindkét típus dinamikusan fedezi fel (C), tehát ez nem egy eltérő tulajdonság (ellentétben a Non-Broadcast módokkal, ahol manuálisan kéne megadni őket).

### Kérdés 2

Egy OSPF Broadcast hálózatban 5 router van összekötve (egy közös switchen). Az R1-es router a DR az adott interfészén. Hány darab TELJES (FULL) OSPF szomszédsága (Adjacency) van az R1-nek ezen a portján?

- A) 1 (a BDR-rel)
- B) 2 (a DR-rel és a BDR-rel)
- C) 4 (az összes szomszéddal)
- D) 5 (az összes szegmenshez csatlakozó routerrel)

**Helyes Válasz: C**

**Magyarázat:** Mivel az R1 maga a DR (Főnök), neki az összes többi (4 darab) routerrel kötelezően FULL szomszédságot kell kiépítenie, hogy szinkronizálhassa velük az LSDB-t. (Ha az R1 egy DROther lenne, akkor csak 2 FULL szomszédsága lenne a DR és BDR irányába, a többiekkel csak 2-Way maradna). A D helytelen, mert saját magával nem épít szomszédságot.

### Kérdés 3

Az alábbiak közül melyik **KETTŐ** kötelező feltétele annak, hogy két router OSPF szomszéddá válhasson? (Válassz KETTŐT!)

- A) A Hello és Dead időzítőknek (Timers) egyezniük kell.
- B) Az OSPF Process ID-knek egyezniük kell.
- C) Az OSPF Router ID-knek egyezniük kell.
- D) Az interfészeknek azonos Területen (Area) kell lenniük.
- E) Az interfészeknek különböző Területen kell lenniük.
- F) Az interfészeknek különböző Alhálózatban (Subnet) kell lenniük.

**Helyes Válasz: A és D**

**Magyarázat:** Az időzítőknek és a Terület számának szigorúan egyeznie kell. A Process ID (B) egy lokális dolog, nem kell egyeznie. A Router ID-knek (C) pont hogy egyedinek (Különbözőnek) kell lenniük. A subnetnek (F) szintén egyeznie kell, nem lehet különböző.

### Kérdés 4

Az alábbi OSPF LSA típusok közül melyiket generálja **KIZÁRÓLAG** egy Multi-access hálózat (pl. Broadcast Ethernet) Kijelölt Routere (DR-je)?

- A) Type 1
- B) Type 2
- C) Type 3
- D) Type 5

**Helyes Válasz: B**

**Magyarázat:** A Type 2 (Network LSA) a DR kiváltsága. Ezzel sorolja fel az adott Ethernet szegmenshez csatlakozó összes routert. A Type 1-et mindenki generálja, a Type 5-öt pedig csak az ASBR.

### Kérdés 5

Az R1 egy OSPF Broadcast hálózathoz csatlakozik a G0/0 portján. Jelenleg az R4 a DR, az R3 a BDR. Minden router OSPF prioritása a gyári 1-es. Kiadod az `ip ospf priority 100` parancsot az R1 G0/0 portján, hogy ő legyen a DR. A parancs kiadása **UTÁN** melyik **KÉT** állítás lesz igaz? (Válassz KETTŐT!)

- A) Az R1 azonnal DR lesz.
- B) Az R1 azonnal BDR lesz.
- C) Az R1 továbbra is DROther marad, mert a prioritása még mindig nem elég magas.
- D) Ha kiadod a `clear ip ospf process` parancsot az R4-en (a jelenlegi DR-en), az R1 BDR lesz.
- E) Ha kiadod a `clear ip ospf process` parancsot az R4-en, az R1 azonnal DR lesz.
- F) A DR és a BDR szerepek a parancs kiadása után változatlanok maradnak.

**Helyes Válasz: D és F**

**Magyarázat:** A DR/BDR választás **Non-preemptive**. A prioritás átírása (bármilyen magasra) a parancs pillanatában semmit nem változtat a fennálló hálózaton (F). Amikor viszont az R4 (a jelenlegi DR) OSPF folyamatát manuálisan kilövöd, a jelenlegi BDR (R3) azonnal, automatikusan előlép DR-ré (hiába maradt a prioritása 1). Ezután új választást tartanak az üres BDR posztra. Mivel most már az R1-nek van a legmagasabb prioritása (100), megnyeri a BDR posztot (D).

### Bónusz Kérdés (Boson ExSim vizsgakérdés - A Legnagyobb OSPF Átverés!)

Kiadod a `show ip ospf interface fastEthernet 0/1` parancsot a Router1-en. A kimenetben többek között ezt látod:

```text
State DROTHER, Priority 50
Timer intervals configured, Hello 10, Dead 40
Neighbor Count is 5, Adjacent neighbor count is 2
```

Az alábbi állítások közül melyik a **HELYES**? Válaszd ki a legjobbat!

- A) A Router1 a DR a szegmensen.
- B) A Router1 egy point-to-multipoint hálózathoz csatlakozik.
- C) A Router1 hibás időzítő (timer) beállításokkal rendelkezik.
- D) A Router1 2 darab teljes (FULL) OSPF szomszédságot épített fel.
- E) A BDR prioritása garantáltan magasabb, mint 50.

**Helyes Válasz: D**

**Magyarázat:** Ez a kérdés a "Neighbor" és az "Adjacent Neighbor" közti különbségre fókuszál. A Router1 egy DROther (Ezt a kimenet is írja, így az A hamis). Összesen 5 OSPF szomszédja van (Neighbor Count = 5). Viszont mivel ő DROther, a többi DROther routerrel csak a 2-Way állapotig jut el (akik a sima szomszédok). Csakis a DR-rel és a BDR-rel (összesen 2 router) épít ki FULL szomszédságot (Adjacency). Ezért az "Adjacent neighbor count" pontosan 2. (Az E azért hamis, mert a BDR prioritása lehet, hogy szintén 50, és csak a magasabb Router ID miatt nyerte meg a választást a Router1-el szemben).
