# CCNA Bővített Jegyzet – 20. Lecke: A Spanning Tree Protokoll (STP) Alapjai

Ebben a leckében a hálózatépítés egyik legkritikusabb és legösszetettebb témakörét, a **Spanning Tree Protokollt (STP)** kezdjük el tárgyalni. Megnézzük, miért létfontosságú a redundancia, mi történik, ha rosszul csináljuk (Broadcast Viharok), és hogyan menti meg a hálózatot a klasszikus STP (IEEE 802.1D) a Layer 2 hurkok (loops) kiküszöbölésével.

## 1. Hálózati Redundancia és a Layer 2 Hurkok (Loops) Veszélye

A modern vállalati hálózatoknak napi 24 órában, az év 365 napján működniük kell. Ha egy kábel vagy egy switch meghibásodik, a hálózatnak azonnal, megszakítás nélkül egy alternatív útvonalat kell használnia. Ezt a biztosítást hívjuk **redundanciának**. A redundáns hálózatokban a switcheket több útvonalon, körbe (loop) kötjük össze.

### De mi ezzel a probléma a Layer 2-n (MAC szinten)?

Míg a Layer 3-as IP csomagoknak van egy **TTL (Time to Live)** mezőjük, amely biztosítja, hogy a csomagok ne keringjenek a végtelenségig, az Ethernet kereteknek (Layer 2) **NINCS** TTL mezőjük.

### A Broadcast Vihar (Broadcast Storm)

Képzelj el három switchet (SW1, SW2, SW3) körbekötve, STP nélkül.

1. Egy PC küld egy ARP Request (Broadcast) keretet az SW1-nek.
2. Az SW1 (mivel broadcastról van szó) elárasztja a keretet, átküldi az SW2-nek és az SW3-nak.
3. Az SW2 és az SW3 szintén elárasztja a kapott kereteket, így azok átkerülnek egymáshoz és vissza az SW1-hez.
4. Mivel a keretek sosem semmisülnek meg, a switchek a végtelenségig másolják és küldözgetik őket egymás között, az óramutató járásával megegyezően és azzal ellentétesen is.
5. A hálózat másodpercek alatt megtelik ezekkel a "szellemkeretekkel" (torlódás), az igazi adatforgalom megbénul, a switchek MAC táblája összeomlik (**MAC Address Flapping**), és a teljes hálózat leáll.

Ezt a katasztrófát akadályozza meg a Spanning Tree Protokoll!

## 2. Mi az az STP (Spanning Tree Protocol)?

A klasszikus STP (**IEEE 802.1D**) egy Layer 2-es biztonsági protokoll, amely minden gyártó switchén alapértelmezetten be van kapcsolva.

A célja: **Megtartani a hálózat fizikai redundanciáját, de logikailag megszakítani a hurkokat.**

### Hogyan csinálja?

A switchek egymás között speciális üzeneteket, úgynevezett **BPDU (Bridge Protocol Data Unit)** kereteket küldözgetnek (alapból 2 másodpercenként). A BPDU-k alapján a switchek feltérképezik a hálózatot, és megállapodnak abban, hogy az alternatív, hurkot okozó portokat **Blokkoló (Blocking)** állapotba helyezik. A blokkolt portokon nem megy át adat, így a kör megszakad. Ha egy működő (Továbbító / Forwarding) kapcsolat megszakad, a blokkolt port azonnal feléled és átveszi a helyét.

> (Terminológia: A hálózatépítés őskorában a switchek helyett hidakat - Bridges - használtak. Bár ma már csak switcheket használunk, az STP protokoll nyelvezete megőrizte a "Bridge" szót, így amikor Bridge-ről vagy Root Bridge-ről beszélünk, valójában egy Switchre gondolunk!)

## 3. Az STP Döntési Folyamata (A Fa Felépítése)

Az STP egy szigorú, 3 lépéses folyamat alapján dönti el, mely portok működhetnek és melyeket kell blokkolni. A legfontosabb, hogy ezek a lépések egymásra épülnek, és mindig csak a megadott tie-breakereket (döntetlen esetén használatos szabályokat) szabad használni!

### 1. LÉPÉS: A Gyökérhíd (Root Bridge) Megválasztása

A hálózatnak kell egy központ, ez lesz a Gyökérhíd. Azt a switchet választják meg, amelyiknek a **LEGALACSONYABB Bridge ID-je (Híd Azonosítója)** van.

**A Bridge ID (Híd Azonosító) felépítése:**

- **Bridge Priority (Híd Prioritás) – 16 bit:** Ennek alapértéke minden switchen **32768**. Ahhoz, hogy minden VLAN-nak saját független STP fája lehessen (ezt a Cisco PVST - Per-VLAN Spanning Tree-nek hívja), ehhez a 32768-hoz mindig hozzáadják a VLAN sorszámát (Extended System ID). Így a VLAN 1-ben az alap prioritás $32768 + 1 =$ 32769. (Ha manuálisan akarod módosítani a prioritást, azt csakis 4096-os lépésközökkel teheted meg: 0, 4096, 8192, stb.).
- **MAC Cím – 48 bit:** Ha a prioritások megegyeznek (ami az alapbeállítás miatt szinte mindig így van), a switchek a MAC-címüket hasonlítják össze. Amelyiknek kisebb a MAC-címe, az nyeri a választást!

**A Gyökérhíd Szabálya:** A megválasztott Root Bridge-en (Gyökérhídon) **MINDEN port Designated (Kijelölt)** porttá válik, azaz Forwarding (Továbbító) állapotban lesz. Ő sosem blokkol portot! Innentől kezdve a hálózatban kizárólag a Root Bridge generálja a BPDU üzeneteket, a többiek csak továbbítják azokat.

### 2. LÉPÉS: A Gyökérportok (Root Ports) Kijelölése

Miután megvan a "főnök", a hálózat összes többi (nem Root) switchének ki kell választania **EGYETLEN EGY** portot, ami a leggyorsabban/legközelebb vezet a Root Bridge-hez. Ez lesz a **Root Port (Gyökérport)**. Ez a port mindig Forwarding állapotban lesz.

> (Egy switchen csak és kizárólag egyetlen Root port lehet!)

**A Root Port kiválasztásának szigorú sorrendje (tie-breakerek):**

1. **A legalacsonyabb Root Cost (Gyökérköltség):** A switch kiszámolja, "mennyibe kerül" eljutni a Gyökérhídig. (A számolásnál mindig csak a kifelé mutató (outgoing) portok költségét adják össze).
   - 10 Mbps (Ethernet) = 100
   - 100 Mbps (FastEthernet) = 19
   - 1 Gbps (GigabitEthernet) = 4
   - 10 Gbps (10GigabitEthernet) = 2
2. **(Döntetlen Root Cost esetén): A legalacsonyabb Szomszédos Bridge ID:** Ha két útvonal ugyanolyan "drága", a switch megnézi, melyik szomszédos switchen keresztül érte el az utat. Amelyik szomszédos switchnek alacsonyabb a Bridge ID-je, az arra néző port lesz a Root Port.
3. **(Döntetlen Szomszédos Bridge ID esetén – pl. két kábel fut ugyanahhoz a szomszédhoz): A legalacsonyabb Szomszédos Port ID:** Ha a kábel a szomszéd switch G0/0 és G0/1 portjaiba csatlakozik, a mi switchünk azt a saját portját választja Root Portnak, amelyik a szomszéd G0/0 (az alacsonyabb azonosítójú) portjába van bedugva!

### 3. LÉPÉS: A Kijelölt és Nem-Kijelölt portok (Designated / Non-Designated) megválasztása

Most már minden switch tudja, merre van a Root Bridge (ezek a Root Portok, amik mindig nyitva vannak). Az összes többi fennmaradó kábel (ami pl. két sima switch között húzódik) egy-egy Ütközési tartományt (Collision Domain) alkot.

> **Az Aranyszabály:** Minden egyes kábelszakaszon (Collision Domain) **PONTOSAN EGY** darab Designated (Kijelölt / Továbbító) portnak kell lennie. A kábel másik végén lévő port Non-Designated (Nem Kijelölt) porttá válik, és **BLOKKOLÓ (Blocking)** állapotba kerül! (Ez maga a hurok megszakítása).

**Hogyan döntik el két egymással szemben lévő port között, hogy melyik nyer (Designated) és melyik veszít (Non-Designated / Blokkolt)?**

1. **A legalacsonyabb Root Cost:** Amelyik switch összességében "olcsóbban" (gyorsabban) éri el a Root Bridget, annak a portja lesz a Designated (a nyertes). A másik oldal portja blokkolva lesz.
2. **(Döntetlen Root Cost esetén): A legalacsonyabb Bridge ID:** Amelyik switchnek alacsonyabb a MAC-címe / Prioritása, azé lesz a Designated port, a másik blokkolva lesz.

> (Megjegyzés: A PC-khez, szerverekhez vagy routerekhez csatlakozó portok mindig Designated / Forwarding portok maradnak, hiszen onnan sosem jön vissza BPDU, így ott nem alakulhat ki hurok).

## Kvíz a 20. Leckéhez (STP Gyakorló Feladatok)

Mivel a Spanning Tree kizárólag logikai levezetésekkel érthető meg, az alábbi kérdések végigkísérik a teljes döntési folyamatot. Olvasd el a levezetést figyelmesen!

### Kérdés 1

Egy 4 switchből álló hálózatban az alábbi paraméterek adottak. Melyik switch lesz a Root Bridge (Gyökérhíd)?

- SW1: Priority: 12289, MAC: `014A.380F.1111`
- SW2: Priority: 32769, MAC: `014A.380F.2222`
- SW3: Priority: 12289, MAC: `014A.3802.3333`
- SW4: Priority: 28673, MAC: `014A.380F.4444`

- A) SW1
- B) SW2
- C) SW3
- D) SW4

**Helyes Válasz: C (SW3)**

**Magyarázat:** A Root Bridge mindig a legalacsonyabb Bridge ID-vel rendelkező switch. Első lépésben a Prioritást nézzük: SW1 és SW3 rendelkezik a legalacsonyabbal (12289). Második lépésben (döntetlen esetén) a MAC-címet vizsgáljuk. Bár mindkettő `014A.38`-cal kezdődik, az SW3 esetében a következő karakter egy `0`, míg az SW1 esetében egy `0F` (ami hexadecimálisan magasabb). Így az SW3-é a legkisebb érték!

### Kérdés 2

A hálózati rajzodon a Root Bridge az SW1. Az SW2 a G0/1 interfészén csatlakozik az SW1-hez, a G0/0 interfészén pedig egy harmadik switchhez. Milyen állapotban (szerepben) van a Root Bridge (SW1) összes portja?

- A) Mindegyik Root Port (Gyökérport)
- B) Mindegyik Designated Port (Kijelölt port)
- C) Az egyik Root, a másik Non-Designated
- D) Mindegyik Non-Designated (Blokkolt) port

**Helyes Válasz: B (Designated Port)**

**Magyarázat:** Ez az STP legbiztosabb szabálya: A Root Bridge-en (Gyökérhídon) **MINDEN** port Designated (Továbbító) port. A Root Bridge a fa gyökere, ő nem blokkol semmit és neki nincs szüksége Root portra (hiszen már a célnál van).

### Kérdés 3

Adott egy hálózat, ahol az SW4 a Root Bridge. Az SW1 három Gigabit (költség = 4) kábellel csatlakozik a hálózat különböző pontjaihoz: a G0/1 egyenesen az SW4-hez megy. A G0/2 egy másik úton, de szintén az SW4-hez megy. Melyik portot fogja az SW1 Root Portnak (Gyökérportnak) választani?

- A) A G0/1-et, mert mindkettőnek azonos a Root Costja és a szomszédos Bridge ID-je, de a G0/1 alacsonyabb azonosítójú port a saját gépén.
- B) A G0/2-t, mert az a szomszédos SW4 magasabb azonosítójú portjához csatlakozik.
- C) Azt a portját, amelyik a szomszédos SW4 (a Root Bridge) legalacsonyabb azonosítójú portjához (Port ID) van bedugva.

**Helyes Válasz: C**

**Magyarázat:** Ha két egyforma kábel fut ugyanahhoz a switchhez (döntetlen a Root Cost és a szomszédos Bridge ID is), a tie-breaker **MINDIG A SZOMSZÉD (Neighbor) Port ID-je**! Nem az számít, hogy az SW1 melyik portját használja, hanem az, hogy a kábelt az SW4 melyik portjába (pl. G0/0 vs G0/1) dugták bele. A kisebb nyer.

### Kérdés 4

Az SW2 és az SW3 között van egy Gigabit kábeles összeköttetés. Egyik switch sem Root Bridge, és a saját Root portjukat már mindketten kiválasztották egy másik irányba. A köztük lévő kábelszakaszon le kell fojtani a forgalmat a hurok megakadályozására. Melyik port lesz leblokkolva (Non-Designated)?

- A) Mindkettő blokkolva lesz.
- B) Azé a switché, amelyiknek alacsonyabb a MAC-címe.
- C) Azé a switché, amelyik magasabb (rosszabb) Root Cost-tal (Gyökérköltséggel) rendelkezik a Root Bridge irányába. Ha ez egyenlő, akkor azé, amelyiknek magasabb a Bridge ID-je.
- D) Azé a switché, amelyiknek kisebb a Root Costja.

**Helyes Válasz: C**

**Magyarázat:** A fennmaradó kábelszakaszokon (ahol már nincs Root Port), a Designated (Továbbító) és a Non-Designated (Blokkoló) port sorsát az dönti el, hogy melyik switchnek "jobb" (kisebb) a kapcsolata a Root Bridge-dzsel (Root Cost). Amelyiknek rosszabb, az veszít, és az ő portja lesz leblokkolva. Döntetlen költség esetén az a switch veszít és blokkol, amelyiknek "rosszabb" (magasabb) a Bridge ID-je.

### Bónusz Kérdés (Boson ExSim előzetes a következő leckéhez)

Bár a PortFast-ról (ami lerövidíti az STP idejét) csak a következő videóban tanulunk, érdemes megjegyezni a szintaxis logikáját: Ha globális konfigurációs módban kiadod a `spanning-tree portfast default` parancsot, mely portokon aktiválódik automatikusan a PortFast funkció a switchen?

- A) Semelyiken, a PortFast-ot nem lehet globálisan bekapcsolni.
- B) Az összes porton.
- C) Csak az Access (Hozzáférési) portokon.
- D) Csak a Trunk portokon.

> (A megoldást Jeremy a következő videó elején árulja el, de a logika: Mivel a PortFast-ot kizárólag a végpontok – PC-k, szerverek – esetében biztonságos használni, a helyes válasz a **C** lesz: a parancs csak a `switchport mode access`-re állított portokon fog aktiválódni, nehogy véletlenül egy másik switch felé (trunk) hurkot okozzunk!)
