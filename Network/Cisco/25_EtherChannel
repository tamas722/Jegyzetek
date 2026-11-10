# CCNA Bővített Jegyzet – 26. Lecke: EtherChannel

Ebben a leckében a hálózati sávszélesség növelésének és a redundancia biztosításának egyik legfontosabb eszközét, az **EtherChannel**-t ismerjük meg. Megnézzük, miért van rá szükség, hogyan kezeli a forgalmat, és hogyan kell konfigurálni Layer 2 és Layer 3 környezetben.

## 1. A Probléma: A Spanning Tree Protokoll (STP) "Keresztbe tesz"

Képzeljük el, hogy van egy Access switchünk (ASW1), amire 40 számítógép van rákötve. Ez a switch egy Distribution switchhez (DSW1) csatlakozik, amely továbbítja a forgalmat az internet felé. Ha a 40 gép egyszerre forgalmaz, a két switch közötti egyetlen kábel (Uplink) szűk keresztmetszetté (bottleneck) válhat, torlódás alakul ki.

**A logikus próbálkozás:** Húzzunk ki még egy kábelt a két switch közé, hogy megduplázzuk a sávszélességet! Sőt, húzzunk ki mindjárt 4 kábelt!

**A valóság:** A Spanning Tree Protokoll (STP) feladata a Layer 2 hurkok (loops) megakadályozása. Ha 4 kábellel kötöd össze a két switchet, az STP azonnal beavatkozik: az egyik kábelt nyitva hagyja (Forwarding), a másik hármat pedig leblokkolja (Blocking). A forgalom továbbra is csak egyetlen kábelen fog folyni, a másik három csak biztonsági tartalék lesz. Hiába van 4 kábeled, a sávszélesség nem nőtt!

## 2. A Megoldás: EtherChannel (Port Channel)

Az EtherChannel lehetővé teszi, hogy akár **8 darab fizikai kábelt** is összefogjunk egyetlen, nagy logikai "csővé" (virtuális interfésszé).

> (Más elnevezései: Port Channel, LAG - Link Aggregation Group).

### Hogyan látja ezt az STP?

Mivel a switchek az összefogott 4 kábelt szoftveresen egyetlen interfészként (pl. Port-channel 1) kezelik, az STP nem lát benne hurkot! Így az STP nyitva hagyja ezt az egyetlen logikai portot, és a forgalom mind a 4 fizikai kábelen egyidejűleg elkezd áramlani. A sávszélességünk megnégyszereződött!

### Az EtherChannel Terheléselosztása (Load Balancing)

Hogyan dönti el a switch, hogy egy adott adatcsomagot a 4 fizikai kábel közül melyiken küldje ki? Ezt egy matematikai algoritmus (Load-Balancing) alapján számolja ki.

> **A legfontosabb szabály:** A terheléselosztás folyamatok (Flows) alapján történik! Egy adott forrás és cél közötti teljes beszélgetés csomagjai mindig ugyanazon a fizikai kábelen fognak utazni. Ezt azért csinálják, nehogy a csomagok összekeveredett sorrendben érkezzenek meg a célállomásra. Ha egy másik gép kezd el kommunikálni, az kaphat egy másik fizikai kábelt a kötegen belül.

**A Load Balancing számítási alapjai (Inputs):**
A switchek modelljétől függően az algoritmus többféle bemeneti adat alapján is tud számolni:

- Forrás MAC-cím (Source MAC)
- Cél MAC-cím (Destination MAC)
- Forrás ÉS Cél MAC-cím (Ez a legkiegyensúlyozottabb a Layer 2-ben)
- Forrás IP / Cél IP / Forrás ÉS Cél IP
- Layer 4 Port számok (TCP/UDP portok)

**Konfigurálása és ellenőrzése:**

- Megnézni: `show etherchannel load-balance`
- Megváltoztatni (Globális konfigurációs módban): `port-channel load-balance src-dst-mac` (Ez a parancs állítja be az elosztás logikáját).

## 3. EtherChannel Protokollok (PAgP vs. LACP)

Akárcsak a trunkölésnél (DTP), a switchek képesek "megtárgyalni" (negotiate), hogy akarnak-e EtherChannelt építeni. Két dinamikus protokoll és egy statikus módszer létezik. (Az LACP enged 16 kábelt összefogni, de ebből csak max. 8 lesz aktív, a többi tartalék/standby).

| Protokoll / Módszer | Típus | Leírás | Üzemmódok (Módszerek) |
|---|---|---|---|
| **PAgP** (Port Aggregation Protocol) | Cisco Proprietary (Saját) | Csak Cisco switchek között működik. Ma már ritkán használják. | `desirable` (aktívan akarja)<br>`auto` (passzívan várja) |
| **LACP** (Link Aggregation Control Protocol) | IEEE Szabvány (802.3ad) | Bármely gyártó switche (pl. Cisco és HP között is) támogatja. Ez az iparági szabvány és az ajánlott módszer! | `active` (aktívan akarja)<br>`passive` (passzívan várja) |
| **Static** (Statikus) | Nincs protokoll | Nincs tárgyalás, azonnal és erőből összeköti a portokat. Veszélyes lehet (pl. kábelhiba esetén). | `on` (csak `on`-nal tud párba állni) |

**Kombinációk (Kik állnak össze?):**

- **PAgP:** `desirable` + `desirable` → OK. `desirable` + `auto` → OK. `auto` + `auto` → **NINCS** kapcsolat.
- **LACP:** `active` + `active` → OK. `active` + `passive` → OK. `passive` + `passive` → **NINCS** kapcsolat.

## 4. Cisco IOS Konfiguráció: Layer 2 EtherChannel

> **A legfontosabb lépés:** A fizikai portok beállításainak hajszálpontosan meg kell egyezniük! (Azonos sebesség, duplex mód, Switchport mód - Access/Trunk -, valamint egyező engedélyezett és natív VLAN-ok). Ha egy port eltér a többitől, a switch felfüggeszti (Suspended) azt a csatornából! Emiatt mindig az `interface range` paranccsal konfiguráljuk a tagokat egyszerre.

### Lépés 1: A fizikai portok bekötése a virtuális csoportba

```text
Switch(config)# interface range g0/0 - 3
Switch(config-if-range)# channel-group 1 mode active
```

(Az `active` kulcsszó mondja meg a switchnek, hogy az LACP protokollt használja. A `1` lesz a virtuális port azonosítója. Ahogy leütöd az entert, a gép automatikusan létrehozza a Port-channel 1 nevű virtuális interfészt).

### Lépés 2: A virtuális port (Port-Channel) konfigurálása

Innentől kezdve **SOSEM** a fizikai portokon, hanem mindig a Port-Channel-en állítjuk be a trunkölést vagy az access módot! (Amit itt beállítasz, azt a switch a háttérben automatikusan rátolja a 4 fizikai kábere).

```text
Switch(config)# interface port-channel 1
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport mode trunk
```

### Ellenőrzés: A Legfontosabb Parancs

```text
show etherchannel summary
```

Ezt a parancsot a CCNA vizsgán álmádból felkeltve is tudni kell értelmezni!

A kimenetben keresd a **Po1 (SU)** jelölést:

- **S:** Layer 2 EtherChannel (Switchport). (Ha L3-as lenne, R betű lenne, mint Routed).
- **U:** In Use (Használatban van, működik a csatorna).

A fizikai portok mellett (pl. Gi0/0) keresd a **(P)** betűt:

- **P:** Port is bundled in port-channel. (A port sikeresen a csatornába lett kötegelve és működik).
- (Ha **s** betűt (suspended) vagy **D** betűt (down) látsz, akkor a port kiesett a kötegből).

## 5. Layer 3 EtherChannel (Routerként összekötött Switchek)

A modern hálózattervezés (Network Design) egyik alapelve, hogy a nagy Core és Distribution switchek között egyáltalán nem használunk Layer 2 kapcsolatot (így az STP-nek nincs mit blokkolnia). Ehelyett a switcheket Layer 3 (Routed) portként, egyfajta belső routerként kötjük össze.

Egy EtherChannelt is megtehetünk Layer 3-as (IP címmel rendelkező) porttá!

**Konfiguráció:**
A fizikai portokon először ki kell adni a `no switchport` parancsot, mielőtt a csoportba tennénk őket!

```text
Switch(config)# interface range g0/0 - 3
Switch(config-if-range)# no switchport
Switch(config-if-range)# channel-group 1 mode active
```

A gép létrehozza a Port-channel 1-et, ami szintén automatikusan Layer 3 módba kerül. Ezután belépünk a virtuális portba, és adunk neki egy IP-címet (mintha egy router interfész lenne).

```text
Switch(config)# interface port-channel 1
Switch(config-if)# ip address 10.0.0.1 255.255.255.252
```

A `show etherchannel summary` parancsban most az "SU" helyett az **"RU"** (Routed, In Use) rövidítést fogod látni.

## Kvíz a 26. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod az EtherChannel szabályaival kapcsolatban!

### Kérdés 1

Az alábbi channel-group mode kombinációk közül melyik **HÁROM** fog eredményesen egy működő EtherChannelt felépíteni két switch között? (Válassz HÁRMAT!)

- A) `on` – `on`
- B) `passive` – `passive`
- C) `desirable` – `auto`
- D) `auto` – `auto`
- E) `active` – `desirable`
- F) `on` – `desirable`
- G) `active` – `active`

**Helyes Válasz: A, C és G**

**Magyarázat:** Az A (`on`-`on`) a statikus, működő kapcsolat. A C (`desirable`-`auto`) a Cisco saját PAgP protokollja, ahol az egyik akarja, a másik engedi, így létrejön. A G (`active`-`active`) az iparági LACP protokoll, ahol mindketten akarják. Az E (`active`-`desirable`) és az F (`on`-`desirable`) két teljesen eltérő technológiát kever, így a parancs elutasításra kerül. A B és a D esetében mindkét oldal passzív, így egyikük sem kezdeményezi a hálózatépítést, és nem jön létre a kapcsolat.

### Kérdés 2

A `show etherchannel summary` kimenetében azt látod, hogy a fizikai interfészek (pl. Gi0/1) mellett egy **(P)** jelölés szerepel. Mit jelent ez a P betű?

- A) A portok LACP passive módban vannak.
- B) A portok be vannak kötegelve (Bundled) a port-channel-be.
- C) A portok fel vannak függesztve (Paused), amíg a túloldalt be nem állítják.
- D) Ez egy Layer 2-es EtherChannel.

**Helyes Válasz: B**

**Magyarázat:** A kimenet tetején lévő jelmagyarázat (Legend) egyértelműen mutatja: a P betű a *bundled in port-channel* rövidítése, vagyis a port sikeresen a köteg része és forgalmat továbbít. (A Layer 2-es csatornát magánál a logikai portnál lévő nagy S betű jelöli).

### Kérdés 3

Az alábbi paraméterek közül melyik **KETTŐNEK** kell kötelezően megegyeznie az összes fizikai interfészen ahhoz, hogy sikeresen betehesd őket egy közös EtherChannelbe? (Válassz KETTŐT!)

- A) Az Interface ID-nek (az interfész sorszáma).
- B) Az IP-címnek.
- C) Az interfész sebességének (Speed).
- D) A Switchport módnak (Access vagy Trunk).

**Helyes Válasz: C és D**

**Magyarázat:** Az EtherChannel csak akkor hajlandó összefogni a kábeleket, ha a fizikai beállítások teljesen egyeznek. Az interfész sebességnek és duplexitásnak, valamint az operációs módnak (Trunk / Access, beleértve az engedélyezett VLAN-okat) azonosnak kell lenniük (C, D). Az A hibás, hiszen fizikailag két külön portot fogsz össze (pl. g0/1 és g0/2). A B hibás, mert Layer 3 EtherChannel esetén az IP-címet **SOHA** nem a fizikai portokra, hanem a logikai Port-channel interfészre adjuk ki.

### Bónusz Kérdés (Boson ExSim vizsgakérdés)

Kiadod az alábbi parancsokat a Switch A-n és a Switch B-n:

**Switch A:**

```text
interface port-channel 1
interface range fastEthernet 0/5 - 6
channel-protocol lacp
channel-group 1 mode on
```

**Switch B:**

```text
interface port-channel 1
interface range fastEthernet 0/5 - 6
channel-protocol pagp
channel-group 1 mode on
```

Az alábbi állítások közül melyik **IGAZ** az eredményül kapott EtherChannel kapcsolatra vonatkozóan?

- A) A kapcsolat LACP használatával jön létre, mert azt konfigurálták először.
- B) Nem jön létre kapcsolat (No link is formed).
- C) A kapcsolat PAgP használatával jön létre, mert azt konfigurálták utoljára.
- D) A kapcsolat statikusan jön létre, aggregációs protokoll nélkül.

**Helyes Válasz: B (Nem jön létre kapcsolat)**

**Magyarázat:** Ez egy csapda! A probléma a Cisco IOS szintaxisában van. Amikor a fizikai portokon manuálisan kikényszeríted, hogy a `channel-protocol lacp` (vagy `pagp`) nyelvet beszéljen, a rendszer várni fogja az ehhez tartozó módokat. Ha ezek után kiadod a `channel-group 1 mode on` parancsot (ami a statikus, protokoll **NÉLKÜLI** módszer), a Cisco IOS visszautasítja (`Command Rejected`) a parancsot! Mivel mindkét switchen elutasításra kerül az utolsó parancs, a fizikai portok egyáltalán nem lépnek be a port-channel csoportba, így a kapcsolat nem fog felépülni. Ha statikus csatornát akarsz (`mode on`), előtte tilos dedikált `channel-protocol`-t konfigurálni.
