# CCNA Bővített Jegyzet – 19. Lecke: DTP (Dynamic Trunking Protocol) és VTP (VLAN Trunking Protocol)

Ebben a leckében két olyan protokollal (a **DTP**-vel és a **VTP**-vel) ismerkedünk meg, amelyeket a Cisco fejlesztett ki a switchek konfigurálásának megkönnyítésére. Bár a legújabb CCNA vizsgakövetelményekből hivatalosan kikerültek, a vizsgán mégis előfordulhatnak, a valós életben pedig elengedhetetlen tudni róluk, mert a legtöbb Cisco eszközön a mai napig alapértelmezetten be vannak kapcsolva, és komoly biztonsági (vagy stabilitási) kockázatot jelenthetnek, ha nem érted a működésüket!

## 1. DTP (Dynamic Trunking Protocol) – A Dinamikus Trunk Képzés

A DTP egy kizárólag Cisco switcheken futó protokoll, aminek egyetlen célja van: megpróbálja automatikusan (emberi beavatkozás nélkül) kitalálni, hogy a két összedugott switchport **Trunk** (több VLAN-t szállító) vagy **Access** (egy VLAN-t szállító végponti) módban működjön-e.

Emlékezzünk az előző leckékből: egy portot manuálisan a `switchport mode access` vagy a `switchport mode trunk` paranccsal állíthattál be. A DTP azonban bevezeti a **dynamic** (dinamikus) módot, ami alapértelmezetten fut minden Cisco porton.

### A DTP Két Fő Üzemmódja

#### 1. Dynamic Desirable (Dinamikusan Vágyakozó)

Ebben a módban a port "agresszívan" szeretne Trunk lenni. Folyamatosan DTP üzeneteket küld a kábelen, amiben azt mondja: "Légy szíves, legyünk Trunk portok!"

> (Érdekesség: A régebbi Cisco switcheken ez volt a gyári alapbeállítás).

#### 2. Dynamic Auto (Dinamikusan Automatikus)

Ebben a módban a port "passzív". Nem kezdeményezi a Trunk létrejöttét, de ha a túloldal kéri tőle, akkor hajlandó Trunkké válni. Azt mondja: "Én elvagyok Access portként is, de ha te Trunk akarsz lenni, akkor én is az leszek."

> (A modern Cisco switcheken ez a gyári alapbeállítás).

### Hogyan dől el, mi lesz a portból? (A DTP Mátrix)

Ha összedugsz két portot, az eredmény a következő lesz:

- **Desirable + Desirable** → Trunk lesz.
- **Desirable + Auto** → Trunk lesz (mert az egyik akarja, a másik meg engedi).
- **Auto + Auto** → Access marad (mert egyik sem kezdeményezi aktívan).
- **Bármelyik dinamikus mód + Trunk (manuális)** → Trunk lesz.
- **Bármelyik dinamikus mód + Access (manuális)** → Access lesz (az Access mód "kőbe vési" a portot, hiába könyörög a túloldal a Trunkért).
- **Trunk (manuális) + Access (manuális)** → **Misconfig (Hiba!)** Semmi sem fog működni, a hálózat megszakad.

### A Biztonsági Kockázat (Miért kell kikapcsolni?)

Ha egy támadó ráköt egy laptopot a fali aljzatra (ami mögött a switchport `dynamic auto` vagy `desirable` módban maradt), egy szoftverrel el tudja hitetni a switchcsel, hogy a laptopja egy másik switch. Erre a switch Trunk módba vált, és rázúdítja az összes létező VLAN forgalmát a támadó laptopjára (ezt hívják **VLAN Hopping** támadásnak).

### Hogyan védekezzünk? (DTP kikapcsolása)

A DTP kikapcsolásának szabályai:

- Ha egy portot manuálisan **access** módba teszel, az automatikusan leállítja a DTP működését.
- **Vigyázat!** Ha egy portot manuálisan **trunk** módba teszel, a port továbbra is küldeni fogja a DTP kereteket, hogy megerősítse a túloldalt! Ha ezt is ki akarod kapcsolni, ki kell adnod egy extra parancsot:

```text
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport nonegotiate
```

A `switchport nonegotiate` parancs némítja el véglegesen a DTP protokollt az adott porton. (Routereken vagy PC-ken amúgy sem működik a DTP).

## 2. VTP (VLAN Trunking Protocol) – A Központosított VLAN Menedzsment

Képzelj el egy egyetemi campust 100 darab switchcsel, ahol létre akarsz hozni egy új VLAN-t (pl. VLAN 50 - Vendégek). Az eddig tanultak alapján mind a 100 switchre be kellene lépned, és kiadni a `vlan 50` parancsot. Rémálom!

A Cisco erre találta ki a VTP-t. A VTP lehetővé teszi, hogy egyetlen központi switchen (a Serveren) hozd létre, töröld vagy módosítsd a VLAN-okat, a többi switch pedig a Trunk portokon keresztül értesül erről, és automatikusan leszinkronizálja (lemásolja) a VLAN adatbázist.

> (Figyelem: A VTP **CSAK** a VLAN-ok létezését (L2 adatbázis) szinkronizálja! Azt továbbra is manuálisan kell beállítanod, hogy melyik PC (access port) melyik VLAN-ba kerüljön!)

### A VTP 3 Működési Módja

- **VTP Server (Szerver):** Ez a főnök. Lehet rajta VLAN-okat létrehozni, törölni és módosítani. Ezeket a változásokat szétküldi a többi switchnek. (Minden Cisco switch gyárilag Server módban van!) A szerver egy NVRAM nevű memóriába menti a VLAN-okat (így újraindulás után is megmaradnak).
- **VTP Client (Kliens):** Ő csak másol. Ezen a switchen a CLI-ből nem tudsz manuálisan VLAN-t létrehozni vagy törölni (a gép hibaüzenetet ad). Ő megkapja a listát a Servertől, beállítja magát aszerint, majd továbbítja az üzenetet a következő switchnek.
- **VTP Transparent (Átlátszó):** Ő egy különc. Nem érdekli a VTP rendszer, nem szinkronizálja a szerver adatait a sajátjára. Ő egy teljesen független, önálló VLAN adatbázissal dolgozik (itt is létrehozhatsz és törölhetsz VLAN-okat, amik csak ezen a switchen fognak létezni). Viszont továbbítja a szerver üzeneteit a Trunk portjain (tehát átengedi magán az infót, hogy a mögötte lévő Kliensek is frissülhessenek).

### A VTP Működési Feltételei és a Veszélye (A "Revision Number")

Hogy a switchek szinkronizáljanak, azonos **VTP Domain** (Tartomány) névvel kell rendelkezniük. Ha egy új (domain név nélküli) switch kap egy VTP üzenetet egy Servertől, automatikusan felveszi annak a domain nevét (pl. `vtp domain cisco`).

**A Configuration Revision Number (Konfigurációs Verziószám):**
Ez a VTP motorja. A verziószám alapból 0. Minden egyes alkalommal, amikor egy Serveren hozzáadsz, törölsz vagy átnevezel egy VLAN-t, a verziószám nő eggyel (1, 2, 3...).

> **A VTP legfontosabb szabálya:** A Switchek **MINDIG**, fenntartások nélkül a legmagasabb verziószámú adatbázist tartják a legújabbnak és a legjobbnak, és szó nélkül lemásolják azt (még a Serverek is lemásolják egy másik Server adatbázisát, ha annak magasabb a száma!).

### A VTP Végzete (Miért ne használd!)

Tegyük fel, hogy a cégednek jól működő hálózata van (VLAN 10, 20, 30, Verziószám: 5). Kiveszel a raktárból egy régi, tesztelésre használt leselejtezett switchet, és rádugod a hálózatra, hogy bővítsd a portokat. De elfelejtetted, hogy a régi switchen (ami szintén Server módban maradt) a Verziószám 50 volt, és csak a VLAN 99 létezett rajta.

Amint bedugod a kábelt, a régi switch szétküldi az 50-es verziószámot. Az összes működő céges switched látja az 50-est, és azt mondja: "Hű, ez újabb, mint az én 5-ösöm!"

Egy pillanat alatt az összes switched kitörli a VLAN 10-et, 20-at és 30-at, és beállítja a VLAN 99-et. A teljes céges hálózatod azonnal, menthetetlenül összeomlik, és minden PC elveszíti a kapcsolatot!

### Hogyan nullázd le a Verziószámot? (A VTP Reset trükkjei)

Mielőtt egy használt switchet rákötnél egy élő hálózatra, le kell nulláznod a VTP Revision Number-t. Erre két módszer van (mivel konkrét "reset" parancs nem létezik):

1. Változtasd meg a VTP Domaint valami kitalált névre (`vtp domain asdf`), majd válts vissza az eredetire. Ez nullázza a számlálót.
2. Tedd át a switchet Transparent módba (`vtp mode transparent`), majd azonnal válts vissza Server vagy Client módra. Ez is nullázza a számlálót. (Modern, biztonságos hálózatokban amúgy is minden switchet érdemes VTP Transparent módban tartani, hogy elkerüld a baleseteket).

## Kvíz a 19. Leckéhez

Ellenőrizd a tudásod a DTP és VTP rejtélyes világáról!

### Kérdés 1

Az SW1 és SW2 két vadonatúj (modern) switch. Egy leselejtezett, nagyon régi switchet (amiből minden konfigurációt töröltek) rádugsz az SW1-re egy hozzáférési (Access) porton keresztül. Azt veszed észre, hogy a két switch között mégis Trunk kapcsolat alakult ki! Mi okozhatta ezt?

- A) A régi switchek portjai alapértelmezetten `switchport mode trunk` beállítással rendelkeznek.
- B) A régi switchek portjai alapértelmezetten `dynamic desirable` módban vannak.
- C) Az Access portok csak a modern switchek sajátosságai.

**Helyes Válasz: B**

**Magyarázat:** A modern switcheken az alapértelmezett beállítás a `dynamic auto` (ami passzív). A régebbi switcheken viszont a `dynamic desirable` volt az alap (ami aktívan kéri a Trunköt). Az auto és a desirable találkozása a DTP mátrix alapján kötelezően Trunk kapcsolatot eredményez!

### Kérdés 2

Az SW1, SW2 és SW3 sorba van kötve (SW1 - SW2 - SW3). Azt szeretnéd, hogy az SW2 továbbítsa az SW1 VTP üzeneteit az SW3-nak, de NE szinkronizálja a saját VLAN adatbázisát az SW1-gyel (és ne is írja felül másokét). Melyik parancsot kell kiadnod az SW2-n?

- A) `vtp mode transparent`
- B) `vtp transparent mode`
- C) `vlan mode transparent`
- D) `vtp mode client`

**Helyes Válasz: A**

**Magyarázat:** Ez a VTP Transparent (Átlátszó) mód tankönyvi definíciója. Egy Transparent switch továbbítja a VTP csomagokat a Trunk portokon, de a saját NVRAM-jában egy független, önálló VLAN adatbázist vezet, ami semmit nem másol le, és őt sem tudják lemásolni mások. A Kliens (D) lemásolta volna az adatbázist.

### Kérdés 3

Melyik az a **KÉT** módszer, amellyel egy switch VTP Configuration Revision Number (Konfigurációs Verziószám) értékét vissza tudod állítani pontosan 0-ra? (Válassz KETTŐT!)

- A) A VTP Domain megváltoztatása egy nem használt, kitalált névre.
- B) A switch VTP Server módba állítása.
- C) A switch VTP Transparent módba állítása.
- D) A `vtp reset` parancs használata a Privileged EXEC módban.

**Helyes Válasz: A és C**

**Magyarázat:** Ahogy a leckében tárgyaltuk, a Cisco IOS nem rendelkezik beépített "reset" paranccsal erre (D kizárva). A Server mód sem nullázza a számot. A két bevált trükk a Domain név megváltoztatása (és visszaváltása), vagy a Transparent mód bekapcsolása (és visszaváltása).

### Kérdés 4

**Boson ExSim Drag-and-Drop Feladat – Milyen kapcsolat jön létre a DTP alapján? Párosítsd össze a beállításokat a végeredménnyel!**

(Képzeld el a bal oldalon és a jobb oldalon lévő beállításokat. Mi lesz az eredmény?)

- **Access + Access** → Access (Kikapcsolja a DTP-t, tisztán Access marad).
- **Access + Dynamic Auto** → Access (Az Auto passzív, az Access visszautasítja, így Access marad).
- **Access + Dynamic Desirable** → Access (A Desirable hiába kéri agresszívan a Trunköt, az Access fix beállítás, elutasítja, így Access marad).
- **Access + Trunk** → Misconfig (Hiba) (A valóságban ne csinálj ilyet, tönkrevágja a kapcsolatot).
- **Dynamic Auto + Dynamic Auto** → Access (Mindkettő passzív, egyik sem kezdeményez, Access marad).
- **Dynamic Auto + Dynamic Desirable** → Trunk (A Desirable kéri, az Auto megengedi, Trunk lesz).
- **Dynamic Auto + Trunk** → Trunk (A manuális Trunk is küld DTP-t, kéri, az Auto megengedi).
- **Dynamic Desirable + Dynamic Desirable** → Trunk (Mindkettő akarja, Trunk lesz).
- **Dynamic Desirable + Trunk** → Trunk (Mindkettő akarja, Trunk lesz).
- **Trunk + Trunk** → Trunk (Természetesen Trunk lesz, manuálisan konfigurálva).
