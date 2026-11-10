# CCNA Bővített Jegyzet – 17. Lecke: VLAN-ok (Virtual LAN) 2. Rész – Trunk Portok és a Router on a Stick

Ebben a leckében folytatjuk a VLAN-ok (Virtuális Helyi Hálózatok) megismerését. Míg az előző leckében az egyetlen VLAN-hoz tartozó Access portokról volt szó, most megnézzük, hogyan lehet több VLAN forgalmát átvinni egyetlen kábelen, hogyan jelölik meg ezeket az adatokat, és hogyan lehet a routerrel hatékonyan összekötni a különböző VLAN-okat.

## 1. A Trunk Portok és a VLAN Tagging (Címkézés)

Egy átlagos hálózatban a VLAN-ok gyakran több switchen ívelnek át (pl. a Mérnökség gépei az 1. és a 2. switchen is jelen vannak). Ha Access portokat használnánk a switchek összekötésére, minden egyes VLAN-hoz egy külön fizikai kábelt kellene kihúzni a két switch közé. Ha van 50 VLAN-od, 50 kábelre lenne szükség. Ez rengeteg portot pazarolna el.

A megoldás a **Trunk Port**. A Trunk port egy olyan speciális switch interfész, amely képes **több VLAN forgalmát egyidejűleg szállítani egyetlen fizikai kábelen**.

### Hogyan tudja a switch, hogy az adott keret melyik VLAN-ba tartozik?

Amikor egy switch kiküld egy keretet a Trunk porton, ellátja azt egy "címkével" (Tag). Ezt a folyamatot hívják **VLAN Tagging**-nek (VLAN címkézés).

- Az **Access portokon** nincsenek címkék (**Untagged**), mert ott egyértelmű, melyik VLAN-hoz tartozik az adott port.
- A **Trunk portokon** utazó kereteket a switch megcímkézi (**Tagged**), így a fogadó switch pontosan tudja, melyik hálózatba kell továbbítania az adatot.

Két fő címkézési protokoll létezik:

- **ISL (Inter-Switch Link):** A Cisco régi, saját (proprietary) protokollja. Ma már szinte egyáltalán nem használják, a modern Cisco switchek sem támogatják.
- **802.1Q (dot1q):** Az IEEE által létrehozott iparági szabvány. Kizárólag ezt használjuk a modern hálózatokban, és a CCNA vizsgán is ezt kérik számon.

## 2. A 802.1Q Címke (Tag) Felépítése

A dot1q címke pontosan **4 bájt (32 bit)** hosszúságú. Nagyon okosan tervezték meg: a switch egyszerűen beszúrja ezt a 4 bájtot a normál Ethernet keret belsejébe, pontosan a Forrás MAC-cím (Source MAC) és a Típus/Hossz (Type/Length) mezők közé.

A címke két fő részből és további almezőkből áll:

- **TPID (Tag Protocol Identifier) – 16 bit:** Ez mindig egy fix hexadecimális értéket tartalmaz: `0x8100`. Ha a switch meglátja ezt az értéket, azonnal tudja, hogy ez egy 802.1Q címkézett keret.
- **TCI (Tag Control Information) – 16 bit:** Ez további 3 almezőre oszlik:
  - **PCP (Priority Code Point) – 3 bit:** A hálózati forgalom priorizálására (QoS / Class of Service) szolgál.
  - **DEI (Drop Eligible Indicator) – 1 bit:** Jelzi, hogy torlódás esetén a keret eldobható-e a fontosabb forgalom védelmében.
  - **VID (VLAN ID) – 12 bit:** Ez a legfontosabb mező! Ez a szám azonosítja a pontos VLAN-t (pl. VLAN 10). Mivel 12 bites, matematikailag $2^{12} = 4096$ VLAN azonosítót képes tárolni. A 0 és a 4095 fenntartott értékek, így a használható VLAN tartomány **1-től 4094-ig** terjed. (A 1-1005 a normál, az 1006-4094 a kiterjesztett tartomány).

## 3. A Native VLAN (Alapértelmezett, Címkézetlen VLAN)

A 802.1Q protokoll rendelkezik egy különleges funkcióval: minden Trunk porton kijelölhető egy úgynevezett **Native VLAN** (Natív VLAN).

- A Trunk port a Native VLAN-ba tartozó kereteket **címke nélkül (Untagged)** küldi át a kábelen!
- Ha egy switch egy Trunk porton címkézetlen keretet kap, automatikusan úgy veszi, hogy az a Native VLAN-hoz tartozik.
- Alapértelmezetten minden Cisco switchen a **VLAN 1** a Native VLAN. Ezt interfészenként lehet (és biztonsági okokból kifejezetten ajánlott is) módosítani egy nem használt, "üres" VLAN-ra (pl. VLAN 1001).

> **Kritikus szabály:** A Native VLAN beállításának hajszálpontosan egyeznie kell a kábel két végén lévő switchen! Ha az egyik switchen a VLAN 10, a másikon a VLAN 30 a Native, akkor az egyik hálózat forgalma átfolyik (kiszivárog) a másikba, vagy a switch egyszerűen eldobja a hibás forgalmat.

## 4. Cisco IOS Parancsok: Trunk Portok Konfigurálása

A Trunk portokat az adott interfész (pl. `g0/0`) konfigurációs módjából állítjuk be.

### 1. Trunk mód bekapcsolása és az Encapsulation (Címkézés)

Egyes (régebbi vagy komolyabb) Cisco switcheken, amelyek még támogatják a régi ISL protokollt, az interfész "Auto" módra van állítva. Ilyenkor a `switchport mode trunk` parancsot a gép visszautasítja (`Command rejected`), mert előbb meg kell mondanod neki, melyik címkézési protokollt akarod használni. Ezt a `dot1q` paranccsal teheted meg. (Olyan switcheken, amik csak dot1q-t tudnak, erre nincs szükség).

```text
Switch(config)# interface g0/0
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport mode trunk
```

### 2. Az Engedélyezett VLAN-ok (Allowed VLANs) módosítása

Alapértelmezetten egy Trunk port az összes létező VLAN (1-4094) forgalmát átengedi. Biztonsági és teljesítménybeli okokból ezt mindig érdemes leszűkíteni csak azokra a VLAN-okra, amelyekre tényleg szükség van az adott kábelen.

```text
Switch(config-if)# switchport trunk allowed vlan 10,30
```

(Ez a parancs felülírja a listát, és innentől **CSAK** a 10-es és a 30-as VLAN mehet át).

További opciók a listák finomhangolására:

- `switchport trunk allowed vlan add 20` (Hozzáadja a 20-ast a meglévő listához).
- `switchport trunk allowed vlan remove 20` (Eltávolítja a 20-ast a listáról).
- `switchport trunk allowed vlan all` (Visszaállítja a gyári állapotot: minden engedélyezett).
- `switchport trunk allowed vlan none` (Mindent letilt, semmi nem megy át).
- `switchport trunk allowed vlan except 1-5, 10` (Mindent átenged, kivéve az 1-5-ig tartókat és a 10-est).

### 3. A Native VLAN megváltoztatása

Biztonsági okokból állítsuk át az alapértelmezett 1-es VLAN-t egy használaton kívülire (pl. 1001).

```text
Switch(config-if)# switchport trunk native vlan 1001
```

### Ellenőrzés

A beállításokat a következő paranccsal tudod leellenőrizni Privileged EXEC módban:

```text
Switch# show interfaces trunk
```

Ez egy listát ad, amiben láthatod, mely portok működnek Trunk módban, mi a Native VLAN-juk, és mely VLAN-okat engedik át (Allowed VLANs).

## 5. Router on a Stick (ROAS) – Inter-VLAN Routing 1 kábellel

Emlékezzünk: a switchek **soha** nem továbbítanak adatot két különböző VLAN között. Ahhoz, hogy a VLAN 10-ben lévő PC kommunikáljon a VLAN 30-ban lévővel, a forgalmat el kell küldeni a Routernek (Layer 3), ami átirányítja azt.

Ha 50 VLAN-od van, nem köthetsz 50 kábelt a router és a switch közé. Ezt oldja meg a **Router on a Stick (ROAS)** (Router egy pálcán).

### A koncepció

A switch és a router között egyetlen fizikai kábelt használunk. A switch portját Trunk módra állítjuk. A router fizikai portját (pl. `g0/0`) szoftveresen **Alinterfészekre (Subinterfaces)** bontjuk. Ezek olyan logikai portok (pl. `g0/0.10`, `g0/0.20`), amelyek külön-külön úgy viselkednek, mintha önálló fizikai kábelek lennének. Minden alinterfész megkap egy saját IP-címet, ami az adott VLAN Alapértelmezett Átjárója (Default Gateway) lesz, és beállítjuk rajta a 802.1Q címkézést.

### ROAS Konfiguráció a Routeren

A fizikai portot csak be kell kapcsolni (IP-címet nem kap):

```text
Router(config)# interface g0/0
Router(config-if)# no shutdown
```

Létrehozzuk az alinterfészt a VLAN 10 számára (a pont utáni szám jelöli az alinterfészt, a konvenció szerint ezt érdemes a VLAN azonosítójával megegyezőre állítani a káosz elkerülése végett):

```text
Router(config)# interface g0/0.10
```

Megadjuk, hogy ez az alinterfész a 802.1Q (dot1q) protokollt használja, és a 10-es VLAN címkéjét figyelje:

```text
Router(config-subif)# encapsulation dot1q 10
```

Végül kiadjuk a VLAN 10-hez tartozó Default Gateway IP-címet (pl. az alhálózat utolsó kiosztható címe):

```text
Router(config-subif)# ip address 192.168.1.62 255.255.255.192
```

Ezt a folyamatot megismételjük minden egyes VLAN esetében (`g0/0.20`, `g0/0.30` stb.).

A router innentől kezdve pontosan tudja: ha kap egy keretet a fizikai kábelen, amiben a 10-es címke van, azt az adatot logikailag a `g0/0.10` alinterfészre kell továbbítania. Ha az adat a VLAN 30-as hálózatba megy tovább, a router ráteszi a 30-as címkét, és úgy küldi vissza a switchnek a közös fizikai kábelen.

## Kvíz a 17. Leckéhez

Ellenőrizd a tudásod a VLAN Trunking és a ROAS szabályaival kapcsolatban!

### Kérdés 1

Úgy szeretnéd beállítani az SW1 GigabitEthernet0/1 interfészét (ami egy Trunk port), hogy a VLAN 10 forgalmát címke nélkül (untagged) küldje át. Melyik parancs a megfelelő?

- A) `encapsulation dot1q 10`
- B) `switchport trunk allowed vlan 10`
- C) `switchport trunk allowed vlan add 10`
- D) `switchport trunk native vlan 10`

**Helyes Válasz: D**

**Magyarázat:** A 802.1Q szabványban a Native VLAN (Natív VLAN) felelős a címkézetlen forgalomért. Bármelyik VLAN-t beállítod Native-nak, annak a kereteiről a switch lehagyja a 4 bájtos címkét a kábelen. Az A válasz a router alinterfészének beállítására való, a B és C válaszok pedig a Trunk porton áthaladó VLAN-ok listáját szabályozzák.

### Kérdés 2

Miután módosítottad a Trunk interfészen engedélyezett VLAN-ok (Allowed VLANs) listáját, szeretnéd azt visszaállítani az eredeti, alapértelmezett állapotába. Melyik paranccsal teheted ezt meg?

- A) `switchport trunk allowed vlan default`
- B) `switchport trunk allowed vlan all`
- C) `switchport trunk allowed vlan none`
- D) `switchport trunk allowed vlan 1, 1001-1005`

**Helyes Válasz: B**

**Magyarázat:** Egy Cisco switchen a Trunk portok gyári alapértelmezett viselkedése az, hogy az összes létező (1-4094) VLAN forgalmát átengedik. Ennek megfelelően az `all` (összes) kulcsszó állítja vissza ezt az állapotot.

### Kérdés 3

Próbálod Trunk módba állítani a Cisco switched egyik interfészét a `switchport mode trunk` paranccsal, de a gép visszautasítja (`Command rejected`). Melyik parancs oldhatja meg ezt a problémát?

- A) `switch port mode trunk`
- B) `switchport trunk encapsulation 802.1q`
- C) `switchport trunk encapsulation dot1q`
- D) `switchport trunk encapsulation auto`

**Helyes Válasz: C**

**Magyarázat:** Ez a hiba olyan switcheken fordul elő, amelyek támogatják mind az iparági szabvány 802.1Q-t, mind a Cisco régi ISL protokollját, és a portjuk "Auto" címkézési módban van. Mielőtt trunkké alakítanád a portot, meg kell mondanod neki, hogy a két nyelv közül melyiket beszélje. A helyes szintaxis erre a `dot1q`.

### Kérdés 4

A 802.1Q címke (tag) melyik almezője azonosítja magát a VLAN-t (a VLAN számát)?

- A) TPID
- B) VID
- C) TCI
- D) VLN

**Helyes Válasz: B**

**Magyarázat:** A VID (VLAN ID) a TCI (Tag Control Information) 16 bites mezőjén belül található 12 bites mező. Ez hordozza a tényleges azonosító számot (1-től 4094-ig). A TPID (Tag Protocol Identifier) csak a fix `0x8100` hexadecimális értéket hordozza, ami jelzi, hogy ez egy dot1q keret. A VLN egy kitalált fogalom.

### Kérdés 5

Kiadtad a `switchport trunk allowed vlan add 10` parancsot egy interfészen. Ennek ellenére a `show interfaces trunk` parancs futtatásakor a VLAN 10 nem jelenik meg a "Vlans allowed and active in management domain" (Engedélyezett és aktív VLAN-ok) szekcióban. Mi lehet ennek az oka?

- A) A VLAN 10 még nem létezik magán a switchen.
- B) A parancs szintaktikailag érvénytelen.
- C) A helyes parancs a `switchport trunk allowed vlan 10` lett volna.
- D) A VLAN 10 le van foglalva, ezért nem használható.

**Helyes Válasz: A**

**Magyarázat:** Ahhoz, hogy egy VLAN átmenjen egy Trunk porton, nem elég engedélyezni a listán; a VLAN-nak fizikailag is léteznie kell a switch belső adatbázisában. Ha még nem hoztad létre globálisan a `vlan 10` paranccsal, akkor a switch logikusan nem tekinti "aktívnak", így nem jeleníti meg a kimenet azon részén (hiába engedélyezted a trunk porton).
