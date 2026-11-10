# CCNA Bővített Jegyzet – 9. Lecke: Switch Interfészek és Konfigurációk

Ez a lecke a hálózati switchek fizikai (Layer 1) és adatkapcsolati (Layer 2) interfészeinek beállításaira fókuszál. Megnézzük a portok sebességét, a kommunikációs módokat (duplex), a felmerülő hibákat, és azt, hogyan konfiguráljuk mindezt a Cisco parancssorban (CLI).

## 1. Router vs. Switch Interfészek (Fizikai és Szoftveres különbségek)

**Fizikai különbségek:**

- A **Routereken** viszonylag kevés interfész található (pl. néhány optikai SFP port vagy RJ45), mert az ő feladatuk a különböző hálózatok összekötése.
- A **Switcheken** rengeteg port van (pl. 48 darab RJ45 port), mivel az ődleges feladatuk, hogy minél több végpontot (end hostot, azaz PC-t, nyomtatót, szervert) csatlakoztassanak a helyi hálózathoz (LAN).

**Alapértelmezett szoftveres állapotok:**

- **Router:** Biztonsági okokból minden interfész alapértelmezetten ki van kapcsolva a `shutdown` paranccsal. Állapotuk: `Administratively down / down`.
- **Switch:** A switch portok nincsenek kikapcsolva. Úgy vannak tervezve, hogy amint bedugsz egy kábelt, azonnal működjenek (plug-and-play).
  - Ha csatlakoztatva van egy eszközhöz: Állapotuk `UP / UP`.
  - Ha nincs bedugva kábel: Állapotuk `DOWN / DOWN` (Ez nem azt jelenti, hogy le vannak tiltva, csak nincs fizikai kapcsolat).

## 2. Kommunikációs Módok (Duplex) és az Ütközések

A **Duplex** azt határozza meg, hogy egy eszköz képes-e egyidőben adatot küldeni és fogadni.

- **Half Duplex (Fél-duplex):** Az eszköz nem képes egyszerre küldeni és fogadni. Ha épp adatot fogad, várnia kell, mielőtt küldeni tudna. Olyan, mint egy walkie-talkie.
- **Full Duplex (Teljes-duplex):** Az eszköz képes egyidőben, várakozás nélkül küldeni és fogadni. Olyan, mint egy telefonbeszélgetés. Ez a modern hálózatok standardja.

### A Hubok, a Switchek és a Collision Domain (Ütközési tartomány)

- **Hub (A múlt technológiája):** A Hub egy buta "jelismétlő" (Layer 1 eszköz). Bármit kap az egyik portján, azt lemásolja és kiküldi az összes többin. Mivel a Hubnál a kábelek logikailag egy közös csatornát alkotnak, az összes rá kötött gép egy közös ütközési tartományban (**Collision Domain**) van. Ha két gép egyszerre küld adatot, a jelek a Hubban összeütköznek (**Collision**), és megsemmisülnek. Ezért a Hubra kötött gépek kötelezően **Half Duplex** módban kell, hogy működjenek.
- **Switch (A modern megoldás):** A Switch (Layer 2 eszköz) megérti a MAC-címeket, és különálló virtuális csatornákat hoz létre. A switchnél minden egyes port egy külön ütközési tartomány. Nem fog ütközni az adat, így a switchek **Full Duplex** módban, maximális sebességgel tudnak működni.

### A CSMA/CD mechanizmus (Hogyan védekeztek régen az ütközések ellen?)

A Half Duplex hálózatokon (Huboknál) használták a **CSMA/CD** (Carrier Sense Multiple Access with Collision Detection) nevű eljárást:

1. **Hallgatózás (Listen):** A gép "belehallgat" a kábelbe. Ha csend van (senki sem küld adatot), elkezd sugározni.
2. **Ütközés érzékelése (Collision Detection):** Ha mégis egyszerre indítottak adást, és ütközés történik, a gépek ezt észlelik.
3. **Zavaró jel (Jamming signal):** Szétküldenek egy zavaró jelet, hogy mindenki értesüljön az ütközésről.
4. **Véletlenszerű várakozás:** Minden gép vár egy véletlenszerű ideig, mielőtt újra próbálkozna.

## 3. Automatikus Egyeztetés (Autonegotiation) és a Hibalehetőségek

A modern switcheken a portok sebessége (Speed) és kommunikációs módja (Duplex) alapértelmezetten **Auto** értékre van állítva.

Ha két gépet összekötsz, azok "megbeszélik" egymással (**Autonegotiation**), és kiválasztják a mindkét fél által támogatott legnagyobb sebességet és a legjobb duplex módot (általában Full Duplex). Például az `a-full` felirat a switchen azt jelenti: automatikusan (a) egyeztetett full duplex.

### Mi történik, ha a folyamat elbukik? (A Cisco Autonegotiation Szabálya)

Tegyük fel, hogy a Switch1 próbál egyeztetni, de a Switch2-n az adminisztrátor fixen beállított mindent (100 Mbps, Full Duplex), és kikapcsolta az egyeztetést. Ilyenkor a Switch1 nem kap választ a "kérdéseire".

Ekkor a Switch1 az alábbi vészforgatókönyvet követi:

- **Sebesség (Speed):** Megpróbálja passzívan "megérezni" (sense), milyen sebességen jön a jel. Ha érzi, hogy 100 Mbps, ő is beállítja magát 100 Mbps-re. (Ha nem tudja megállapítani, a leglassabb értékre, 10 Mbps-re vált).
- **Duplex (A legkritikusabb pont):** A duplexet nem lehet "megérezni". Ezért be van égetve egy szabály:
  - Ha az érzékelt sebesség **10 vagy 100 Mbps**, a switch biztonsági okokból **Half Duplex**-re vált.
  - Ha az érzékelt sebesség **1000 Mbps (Gigabit) vagy annál több**, a switch **Full Duplex**-re vált.

**A probléma:** A mi példánkban a Switch1 100 Mbps-t érzékelt, ezért Half Duplexre állt. De a Switch2 fixen Full Duplexen maradt! Ezt hívják **Duplex Mismatch**-nek (Duplex eltérés). A Full Duplex oldal folyamatosan küldi majd az adatot (mert azt hiszi, szabad), míg a Half Duplex oldal ezt nem bírja feldolgozni → rengeteg ütközés lesz, és a hálózat borzalmasan belassul. Ezért javasolt mindenhol bekapcsolva hagyni az Autonegotiation-t!

## 4. Cisco IOS Parancsok (Interfészek Konfigurálása)

A konfigurációs módból (`conf t`) állíthatjuk be a portokat.

### 1. Sebesség és Duplex manuális beállítása (pl. az F0/1-es porton):

```text
Switch(config)# interface f0/1
Switch(config-if)# speed 100
Switch(config-if)# duplex full
Switch(config-if)# description ### Connected to R1 ###
```

### 2. Több port egyidejű konfigurálása (Interface Range)

Ha például az összes nem használt portot le akarod kapcsolni biztonsági okokból, ahelyett, hogy egyesével lépnél be beléjük, használhatod a `range` parancsot!

```text
Switch(config)# interface range f0/5 - 12
Switch(config-if-range)# shutdown
```

> **Tipp:** Ha nem egymás melletti portokat akarsz módosítani, vesszővel elválaszthatod őket: `interface range f0/5 - 6 , f0/9 - 12`.

## 5. Hasznos "Show" Parancsok Switcheken

Ezekkel ellenőrizheted a hálózat fizikai állapotát (Privileged EXEC módból):

- **`show ip interface brief`:** Megmutatja az IP-címet (ha van), a fizikai állapotot (Status - L1) és az adatkapcsolati állapotot (Protocol - L2).
- **`show interfaces status`:** Egy nagyon kompakt, kifejezetten switchekre kitalált táblázat. Mutatja a portot, a beállított leírást (Name), az állapotot (`connected` vagy `notconnect`), a VLAN-t, a Duplexet (pl. `a-full`) és a Sebességet (pl. `a-100`).
- **`show interfaces description`:** Egy gyors lista, ami csak a portokat, az állapotukat és az általad beírt Description (Leírás) mezőket mutatja.
- **`show interfaces <port>`:** (Pl. `show interfaces f0/1`). Egy brutálisan részletes technikai jelentés az adott portról. Megmutatja a gyári MAC-címet (BIA), a fizikai paramétereket, és a legalján a Hiba számlálókat (Error counters).

## 6. Interfész Hibák (Errors) és Számlálók

A `show interfaces` parancs kimenetének alján találhatók a számlálók. Bár az adatkapcsolat "UP/UP" állapotú is lehet, mégis előfordulhatnak adatsérülések. A legfontosabb hiba típusok, amiket ismerned kell:

- **Runts (Zsugorított keretek):** Olyan keretek, amelyek mérete kisebb, mint a kötelező Ethernet minimum (64 bájt). Általában ütközések vagy rossz hálózati kártyák okozzák.
- **Giants (Óriás keretek):** Olyan keretek, amelyek mérete meghaladja a maximálisan engedélyezett Ethernet méretet (1518 bájt).
- **CRC (Cyclic Redundancy Check) hibák:** Azok a keretek, amelyek megbuktak a hibajavító teszten (az FCS mező alapján). A keret megérkezett, de a tartalma útközben megsérült (pl. hibás kábel vagy interferencia miatt).
- **Frame (Keret hiba):** Olyan keretek, amelyek formátuma szabálytalan vagy olvashatatlan.
- **Input / Output errors:** A fenti hibák és egyéb anomáliák összesített statisztikája (beérkező, illetve kimenő irányba).

## Kvíz a 9. Leckéhez

### Kérdés 1

Egy Duplex eltérés (Duplex mismatch) van a Switch1 (F0/1) és a Switch2 (F0/1) összekötött portjai között, és az Autonegotiation (Automatikus egyeztetés) ki van kapcsolva. Mi lesz ennek az eredménye?

- A) Javuló hálózati teljesítmény.
- B) Ütközések (Collisions) fognak előfordulni.
- C) A Switch1 megérzi a Switch2 duplex beállítását, és ahhoz igazítja a sajátját.

**Helyes Válasz: B**

**Magyarázat:** A Half Duplex oldalon lévő eszköz nem képes egyszerre küldeni és fogadni. A Full Duplex oldalon lévő eszköz viszont erről nem tud (mert kikapcsolták az egyeztetést), és folyamatosan tolja az adatot. Ha a Full Duplex oldal akkor küld, amikor a Half Duplex oldal is próbálna, azonnal ütközés jön létre, ami drasztikusan rontja a hálózat sebességét. (Az A és C válaszok hamisak).

### Kérdés 2

Milyen eljárást használnak a Half Duplex interfészeken az ütközések észlelésére és elkerülésére?

- A) CSMA/CD
- B) CSMA/CA
- C) Autonegotiation (Automatikus egyeztetés)
- D) Duplex Auto

**Helyes Válasz: A**

**Magyarázat:** A CSMA/CD (Carrier Sense Multiple Access with Collision Detection) a klasszikus Ethernet protokollja, amellyel a Hub-okra kötött Half Duplex gépek "belehallgattak" a vonalba, és ütközés esetén visszakoztak. A CSMA/CA a Wi-Fi (vezeték nélküli) hálózatok hasonló technológiája.

### Kérdés 3

Melyik parancs mutatja meg a portokon észlelt hibák számlálóit (error counters)?

- A) `show interfaces`
- B) `show ip interface brief`
- C) `show interfaces status`
- D) `show interfaces errors`

**Helyes Válasz: A**

**Magyarázat:** A részletes `show interfaces` (vagy egy adott portra szűrve: `show interfaces f0/1`) parancs kimenetének alján találhatók a Runts, Giants, és CRC hiba statisztikák. A többi parancs (B és C) csak egy gyors összefoglalót ad, a D parancs pedig nem létezik.

### Kérdés 4

Az alábbiak közül melyek egy hálózati interfészen előforduló valós hiba (Error) típusok?

- A) runts, giants, broadcasts
- B) shorts, longs, oversizes
- C) packets, bytes, inputs, outputs
- D) runts, giants, CRC

**Helyes Válasz: D**

**Magyarázat:** A Runts (64 bájtnál kisebb), a Giants (1518 bájtnál nagyobb) és a CRC (sérült adat) mind hivatalos hiba számlálók a Cisco eszközökön. Az A válasz azért rossz, mert a broadcast üzenetek a hálózat normális működésének részei, nem hibák.

### Kérdés 5

A Switch1 (SW1) automatikus egyeztetést (Autonegotiation) próbál végezni a Switch2-vel (SW2). A Switch2-n azonban az egyeztetés ki van kapcsolva, és fixen 100 Mbps sebességre, valamint Full Duplexre van beállítva. Milyen sebesség és duplex beállítást fog alkalmazni a Switch1?

- A) Speed 100 Mbps, Duplex Full
- B) Speed 100 Mbps, Duplex Half
- C) Speed 10 Mbps, Duplex Full
- D) Speed 10 Mbps, Duplex Half

**Helyes Válasz: B**

**Magyarázat:** Ez a Cisco vészforgatókönyve! Az SW1 képes "megérezni" az SW2 sebességét a jelekből, ezért sikeresen beállítja magát 100 Mbps-re. Azonban a Duplexet nem lehet megérezni. A szabály az, hogy ha a sebesség 10 vagy 100 Mbps, a switch biztonsági okokból kötelezően visszavált Half Duplex módra (csak Gigabit, azaz 1000 Mbps esetén maradna Full Duplex). Ezért az eredmény 100 Mbps / Half Duplex lesz (ami azonnal egy duplex eltérés hibához vezet).
