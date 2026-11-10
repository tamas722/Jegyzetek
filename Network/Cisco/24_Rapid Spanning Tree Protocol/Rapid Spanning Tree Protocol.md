# CCNA Bővített Jegyzet – 25. Lecke: RSTP (Rapid Spanning Tree Protocol)

Ebben a leckében rátérünk a modern hálózatok szabványára, a **Rapid Spanning Tree Protocol (RSTP)**-ra. Míg a klasszikus STP (802.1D) teszi a dolgát és megvéd a hurkoktól, addig a legnagyobb hátránya a sebessége (akár 50 másodpercet is igénybe vesz a hálózat helyreállítása). Az RSTP (802.1w) ezt az időt másodpercekre csökkenti, így a modern elvárásoknak megfelelően a hálózati kimaradások szinte észrevehetetlenek lesznek.

## 1. Az STP Verziók Összehasonlítása

Mielőtt belevágunk a részletekbe, tegyünk rendet a különböző Spanning Tree verziók és elnevezések között! Léteznek iparági szabványok (amelyeket minden gyártó, pl. HP, Juniper támogat) és vannak a Cisco saját (proprietary) verziói.

| Verzió Neve | Szabvány (IEEE) | Különálló fa minden VLAN-hoz? (Load Balancing) | Sebesség | Megjegyzés |
|---|---|---|---|---|
| **STP (Classic)** | 802.1D | Nem (1 közös fa mindenre) | Lassú (30-50 mp) | Az eredeti szabvány. |
| **PVST+** | Cisco saját | Igen | Lassú (30-50 mp) | A Cisco válasza a klasszikus STP-re. Támogatja a 802.1Q címkézést. |
| **RSTP** | 802.1w | Nem (1 közös fa mindenre) | Gyors (1-5 mp) | Az új, gyors iparági szabvány. |
| **Rapid-PVST+** | Cisco saját | Igen | Gyors (1-5 mp) | A modern Cisco switchek alapértelmezése! Ötvözi a gyorsaságot a VLAN-onkénti terheléselosztással. |
| **MSTP** | 802.1s | Igen (Csoportosítva) | Gyors (1-5 mp) | Iparági szabvány. Lehetővé teszi több VLAN egyetlen STP példányba (Instance) történő csoportosítását (pl. VLAN 1-100 = 1. fa, VLAN 101-200 = 2. fa). Kifejezetten nagy (enterprise) hálózatokhoz! |

> (Tipp: Amikor a Cisco CCNA anyagaiban RSTP-ről beszélünk, a gyakorlatban mindig a Cisco-féle Rapid-PVST+ verzióra gondolunk).

## 2. Mi teszi az RSTP-t "Gyorssá" (Rapid)?

A klasszikus STP időzítőkön alapul. Ha megszakad egy vonal, a switch vár 20 másodpercet (Max Age), majd vár 15 másodpercet (Listening), majd még 15-öt (Learning), és csak utána nyitja ki az új portot.

Az RSTP nem időzítőket használ a konvergenciához, hanem **aktív tárgyalást (Negotiation/Handshake)**. A switchek közvetlenül "megbeszélik" egymással (egy Sync/Proposal/Agreement folyamat során), hogy ki melyik portot nyissa ki, így a várakozási idő szinte azonnalira csökken.

### Az RSTP BPDU (Mi változott a motorháztető alatt?)

- **Verziószám:** A klasszikus STP BPDU verziószáma 0. Az RSTP BPDU verziószáma **2**.
- **Mindenki küldi:** A klasszikus STP-ben csak a Root Bridge generált BPDU-kat (a többiek csak továbbították). Az RSTP-ben **MINDEN switch saját maga is küld BPDU-kat** a kijelölt portjain!
- **Gyorsított öregedés (Aging):** A klasszikus STP 10 Hello időzítőt (20 másodperc) várt, hogy halottnak nyilvánítson egy szomszédot. Az RSTP mindössze **3 kihagyott BPDU-t (6 másodpercet)** vár! Ha a szomszéd 6 másodpercig néma, a port azonnal leválasztásra kerül, és a MAC táblát a switch azonnal törli (flush).
- **Flags Mezők:** A klasszikus STP csak 2 bitet használt a BPDU Flags mezőjéből (a Topology Change-hez). Az RSTP mind a 8 bitet aktívan használja a port szerepek, állapotok és a kézfogások kommunikálására.

> (Visszafelé kompatibilitás: Ha egy modern, RSTP-t futtató switchet rákötsz egy régi, 802.1D-t futtató switchre, az adott kábelszakaszon a modern switch automatikusan "lebutítja" magát a régi STP sebességére).

## 3. Az RSTP Új Port Állapotai (Port States)

Az RSTP radikálisan leegyszerűsítette a portok állapotát. Az 5 korábbi állapotból mindössze **3** maradt!

| Klasszikus STP Állapot | RSTP Állapot | Mit csinál a port? |
|---|---|---|
| Disabled (Letiltott) | **Discarding** (Eldobó) | A port adminisztratív okokból (`shutdown`) van zárva. |
| Blocking (Blokkoló) | **Discarding** (Eldobó) | A port hurokmegelőzés miatt van lezárva. Csak BPDU-kat fogad. |
| Listening (Hallgatózó) | **Discarding** (Eldobó) | Ezt az állapotot az RSTP teljesen összevonta az előzőkkel. |
| Learning (Tanuló) | **Learning** (Tanuló) | Adatot még nem küld, de elkezdi építeni a MAC táblát. |
| Forwarding (Továbbító) | **Forwarding** (Továbbító) | Normál működés. Adatot és BPDU-t is küld/fogad. |

## 4. Az RSTP Új Port Szerepei (Port Roles)

Az RSTP a klasszikus STP (Root, Designated, Non-Designated) szerepeit kibővítette, hogy jobban leírja a "tartalék" (backup) útvonalakat.

- **Root Port (Gyökérport):** Ugyanaz maradt. A leggyorsabb út a Root Bridge felé. (Állapota: Forwarding).
- **Designated Port (Kijelölt port):** Ugyanaz maradt. Az adott kábelszakasz (Collision Domain) nyertes portja, amelyik közelebb van a Root Bridge-hez. (Állapota: Forwarding).
- **Alternate Port (Alternatív port):** A klasszikus "Blokkoló/Non-Designated" port új neve! Ez a port egy **MÁSIK** switchről kap "jobb" (Superior) BPDU-t, ezért lezár, hogy ne legyen hurok. (Állapota: Discarding).
  - **A varázslat:** Az Alternate port azonnali tartalékként funkcionál a Root port számára. Ha a Root port kábele elszakad, az Alternate port késlekedés nélkül, azonnal átveszi a helyét (Forwarding állapotba ugrik). Ezáltal a korábban a klasszikus STP-ben opcionális **UplinkFast** nevű funkció az RSTP-ben már gyárilag be van építve!
- **Backup Port (Tartalék port):** Egy ritka szerep. Ez a port a **SAJÁT** switchének egy **MÁSIK** portjáról kap "jobb" BPDU-t. Ez kizárólag akkor fordulhat elő, ha a switch két kábellel is rá van dugva egy régi, elavult Hub-ra. Ha a Designated port elszakad, a Backup port azonnal átveszi a helyét. (Állapota: Discarding).

> (Extra infó: A klasszikus STP **BackboneFast** nevű funkciója is beépítésre került az RSTP-be. Ez abban segít, hogy ha egy távoli switch meghibásodik és elkezd téves, saját maga által kreált (Inferior) BPDU-kat küldeni, a többi switch ezt azonnal felismeri, és nem várják meg a 20 másodperces Max Age lejártát, hanem rögtön reagálnak).

## 5. Az RSTP Kapcsolattípusai (Link Types)

Hogy az RSTP tárgyalási (handshake) folyamata működni tudjon, a switcheknek tudniuk kell, hogy mihez vannak csatlakoztatva. A kábel másik végén lévő eszköz határozza meg a Link Type-ot. Ezeket a switch automatikusan érzékeli (bár manuálisan is beállíthatók).

- **Point-to-Point (P2P - Pont-tól pont-ig):**
  - **Mikor használja?** Ha a port Full-Duplex módban működik (azaz egy másik switchhez vagy routerhez van kötve).
  - **Csak a Point-to-Point kapcsolatokon működik a villámgyors RSTP tárgyalás (Proposal/Agreement)!**
- **Shared (Megosztott):**
  - **Mikor használja?** Ha a port Half-Duplex módban működik (azaz egy elavult Hub-hoz van kötve).
  - A Shared kapcsolatokon a gyors tárgyalás nem működik, ezek a portok visszalassulnak a klasszikus STP (802.1D) sebességére. (Ma már ilyennel szinte nem is találkozol).
- **Edge (Peremhálózati):**
  - **Mikor használja?** Ha a portra végpontok (PC-k, szerverek) vannak kötve.
  - Az Edge port pontosan ugyanaz, mint a klasszikus STP-ben megismert **PortFast**! Ha egy portot Edge porttá akarsz tenni, pontosan ugyanazt a régi parancsot kell kiadnod: `spanning-tree portfast`. A port azonnal Forwarding állapotba ugrik, megkerülve a szinkronizációs fázisokat.

## Kvíz a 25. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod az RSTP (Rapid Spanning Tree) működésével kapcsolatban!

### Kérdés 1

Az IEEE 802.1D (Klasszikus STP) melyik két opcionális funkciója lett "gyárilag" beépítve a 802.1w (RSTP) szabványba, lehetővé téve a blokkolt portok azonnali Továbbító (Forwarding) állapotba állítását kábelszakadás esetén? (Válassz KETTŐT!)

- A) PortFast
- B) Loop Guard
- C) BPDU Guard
- D) UplinkFast
- E) BackboneFast

**Helyes Válasz: D és E**

**Magyarázat:** Az UplinkFast felel azért, hogy az RSTP "Alternate" portja azonnal átvehesse a megszakadt Root port helyét (várakozás nélkül). A BackboneFast pedig abban segít, hogy a hálózat távolabbi pontjain történő szakadások (amikor egy switch hirtelen magát hiszi Root Bridge-nek és Inferior BPDU-kat küld) azonnali reagálást váltsanak ki a Max Age kivárása helyett. A PortFast (Edge) továbbra is manuális konfigurációt igényel az Access portokon!

### Kérdés 2

Melyik paranccsal tudsz egy RSTP portot Edge porttá (Peremhálózati porttá) konfigurálni a Cisco CLI-ben?

- A) `spanning-tree link-type edge`
- B) `spanning-tree mode edge`
- C) `spanning-tree link-type portfast`
- D) `spanning-tree portfast`

**Helyes Válasz: D**

**Magyarázat:** A Cisco IOS nem vezetett be új parancsot az Edge portokhoz. Az RSTP Edge funkciója 100%-ban megegyezik a klasszikus PortFast funkcióval, így továbbra is a jól ismert `spanning-tree portfast` parancsot kell kiadni hozzá (akár interfész, akár globális szinten). A `link-type` paranccsal csak a Point-to-Point vagy a Shared típust lehet kikényszeríteni.

### Kérdés 3

A klasszikus STP (802.1D) öt port állapotát (Disabled, Blocking, Listening, Learning, Forwarding) az RSTP háromra csökkentette. Melyik új RSTP állapot foglalja magában a korábbi Disabled, Blocking és Listening állapotokat?

- A) Disabled (Letiltott)
- B) Discarding (Eldobó)
- C) Inactive (Inaktív)
- D) Blocking (Blokkoló)

**Helyes Válasz: B**

**Magyarázat:** Az RSTP megalkotói rájöttek, hogy felesleges különbséget tenni a Disabled, a Blocking és a Listening állapotok között, hiszen a végeredmény (a switchport adat-továbbítási szempontból) ugyanaz: a port nem küld és nem fogad normál adatot, és nem tanul MAC-címeket. Ezt a hármat vonták össze a Discarding (Eldobó) állapotba. (A Learning és a Forwarding továbbra is megmaradtak önálló állapotokként).

### Kérdés 4

Mi a különbség az RSTP "Alternate" és "Backup" portszerepei között?

- A) Az Alternate port a Designated port tartaléka, a Backup port a Root port tartaléka.
- B) Az Alternate port egy MÁSIK switchről kap "jobb" (Superior) BPDU-t, míg a Backup port a SAJÁT switchének egy másik interfészéről kap "jobb" BPDU-t (általában egy Hub miatt).
- C) Az Alternate port csak a Point-to-Point kapcsolatokon, a Backup port csak az Edge kapcsolatokon működik.

**Helyes Válasz: B**

**Magyarázat:** Az Alternate (Alternatív) port veszi át a klasszikus Blocking (Non-Designated) portok túlnyomó többségének szerepét. Mivel egy másik gép mondja neki, hogy "én jobb utat kínálok", a port blokkol (discarding lesz). Ez egyben a Root port azonnali tartaléka is. A Backup (Tartalék) port egy rendkívül speciális eset: csak akkor jön létre, ha egy switch két kábellel (két porton) csatlakozik egy megosztott közeget alkotó Hub-ba, és így a switch gyakorlatilag önmagától kapja vissza a saját (jobb prioritású portjáról küldött) BPDU-ját.

### Bónusz Kérdés (Boson ExSim vizsgakérdés)

Az alábbi Spanning Tree (STP) opcionális funkciók közül melyik csökkenti a konvergencia időt azáltal, hogy a peremhálózati portokat (Edge ports) azonnal Továbbító (Forwarding) állapotba helyezi?

- A) Root Guard
- B) BPDU Guard
- C) PortFast
- D) BPDU Filter
- E) Loop Guard

**Helyes Válasz: C**

**Magyarázat:** A PortFast az egyetlen olyan funkció a felsoroltak közül, aminek a közvetlen célja a sebesség növelése (a Listening és Learning fázisok kikerülésével). A többi felsorolt funkció (Guard-ok és a Filter) mind biztonsági célokat szolgál, és a hálózat integritását védik a hurkokkal és a nem kívánt BPDU üzenetekkel szemben, anélkül hogy magát a konvergencia időt közvetlenül rövidítenék. Az RSTP-ben a PortFast egyenértékű az Edge Port fogalmával.
