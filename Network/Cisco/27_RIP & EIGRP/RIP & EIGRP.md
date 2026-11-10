# CCNA Bővített Jegyzet – 27. Lecke: Dinamikus Útválasztás (RIP és EIGRP)

Bár a jelenlegi CCNA vizsgakövetelményekben hivatalosan csak az OSPF szerepel, mint vizsgaanyag, a gyakorlatban és a teszteken továbbra is találkozhatsz a **RIP** (Routing Information Protocol) és az **EIGRP** (Enhanced Interior Gateway Routing Protocol) kérdéseivel. Ebben a leckében áttekintjük ezeknek a Távolságvektor (Distance Vector) protokolloknak a működését, konfigurációs alapjait és a kritikus különbségeket.

## 1. RIP (Routing Information Protocol) – Az Őskövület

A RIP egy rendkívül régi, iparági szabványú protokoll. Mivel a "Hop Count" (ugrások száma) alapján számolja a metrikát, és maximum 15 ugrást engedélyez (a 16. már elérhetetlen), ma már csak nagyon kis hálózatokban vagy laborkörnyezetben használják.

### RIPv1 vs. RIPv2

> **Soha ne használj 1-es verziót!**

- **RIPv1:** "Classful" (Osztályalapú). Nem ismeri a subnet maszkokat (CIDR, VLSM). Ha egy `10.1.1.0/24`-es hálózatot próbálsz beletenni, a RIPv1 levágja a végét, és a teljes `10.0.0.0/8`-as Class A hálózatot fogja hirdetni a szomszédoknak. Az üzeneteit Broadcast formájában (`255.255.255.255`) szórja.
- **RIPv2:** "Classless" (Osztálynélküli). Kezeli a modern alhálózatokat (subnets), és az üzeneteit a `224.0.0.9`-es Multicast címen küldi, ami sokkal hatékonyabb.

### RIP Alapkonfiguráció (A "Network" parancs logikája)

Belépés a konfigurációs módba és az alapbeállítások:

```text
Router(config)# router rip
Router(config-router)# version 2
Router(config-router)# no auto-summary
```

(A `no auto-summary` kikapcsolja a régi osztályalapú kerekítést).

#### Hogyan működik a "Network" parancs?

A legnagyobb tévhit, hogy a `network` paranccsal mondod meg a routernek, hogy mit reklámozzon! **NEM!**

Amikor kiadod a `network 10.0.0.0` parancsot (a RIP nem kér subnet maszkot, Classfulként kezeli a bemenetet), a router a következőt csinálja:

1. Végignézi a saját fizikai interfészeit.
2. Ha talál olyat, aminek az IP címe beletartozik a `10.0.0.0/8` tartományba (pl. `10.0.12.1` és `10.0.13.1`), azokon a portokon aktiválja a RIP protokollt.
3. Ezeken a portokon elkezd szomszédokat (adjacencies) keresni.
4. Végül a router fogja ezeknek a konkrét interfészeknek a valós, pontos alhálózati maszkját (pl. `10.0.12.0/30`), és ezt a pontos információt fogja reklámozni a szomszédoknak!

**Hasznos parancsok:**

- `passive-interface g2/0`: Ha a G2/0 porton nincsenek más routerek (csak PC-k), ezzel letilthatod, hogy a router feleslegesen pazarolja a sávszélességet a RIP üzenetek kiküldésével. (A hálózatot ettől függetlenül hirdetni fogja a többi portján lévő szomszédoknak).
- `default-information originate`: Ha a routereden van egy statikus útvonal az Internet felé (Default Route: `0.0.0.0 0.0.0.0`), ezzel a paranccsal a router "bepumpálja" ezt az információt a RIP-be, így a többi router is megtanulja az utat kifelé.

## 2. EIGRP (Enhanced Interior Gateway Routing Protocol) – A Cisco Büszkesége

Az EIGRP a Cisco saját (proprietary) protokollja, a régi IGRP modernizált változata. Egy "Fejlett" Távolságvektor protokoll, amely sokkal gyorsabban reagál a hibákra, mint a RIP, és a metrikája a Sávszélességen (Bandwidth) és a Késleltetésen (Delay) alapul.

> **Kritikus apróság:** A metrika számításakor a router a teljes útvonal leglassabb (legszűkebb) sávszélességét veszi alapul, de a késleltetéseket összeadja.

Az EIGRP üzeneteit a `224.0.0.10` Multicast címen küldi.

### Az EIGRP Egyedi Szuperképessége

Ez az egyetlen IGP protokoll, amely képes az **Unequal Cost Load-Balancing**-re (Egyenlőtlen Költségű Terheléselosztásra). Míg a RIP és az OSPF csak akkor osztja meg a forgalmat két útvonal között, ha azok metrikája hajszálpontosan megegyezik, az EIGRP képes beállítani, hogy pl. a forgalom 70%-a menjen a gyorsabb Gigabit kábelen, 30%-a pedig a lassabb FastEtherneten!

### EIGRP Konfiguráció és a Wildcard Maszk (Inverz maszk)

**AS Szám:** A konfiguráció elindításakor meg kell adni egy AS (Autonomous System) számot. Ennek a számnak egyeznie kell a többi routeren, különben nem állnak szóba egymással!

```text
Router(config)# router eigrp 1
Router(config-router)# no auto-summary
```

#### A Wildcard Maszk

A `network` parancsnál az EIGRP (és az OSPF is!) nem normál alhálózati maszkot használ, hanem az inverzét, a **Wildcard Maszkot**.

**Hogyan számoljuk ki?** Vond ki a normál maszk minden oktettjét 255-ből!

Példa egy `/28`-as hálózatra:

- Normál maszk: `255.255.255.240`
- Kivonás: (255-255) . (255-255) . (255-255) . (255-240)
- **Wildcard maszk:** `0.0.0.15`

**Mit jelent a 0 és az 1 a wildcard maszkban?**

- **0:** A bitnek pontosan meg kell egyeznie!
- **1:** A bit bármi lehet.

Példa: `network 172.16.1.0 0.0.0.15` → A router keres egy interfészt, ahol az első 28 bit pontosan megegyezik a `172.16.1.0`-val.

### Az EIGRP Router ID (A Router "Neve")

Az EIGRP (és az OSPF) minden routernek ad egy 32 bites azonosítót (olyan formátumban, mint egy IP-cím, pl. `1.1.1.1`).

A kiválasztás sorrendje (priority):

1. **Manuális konfiguráció** (Ez a legerősebb): `eigrp router-id 1.1.1.1`
2. A legmagasabb IP-című **Loopback** (Virtuális) interfész.
3. A legmagasabb IP-című **Fizikai** interfész.

Az EIGRP útvonalak a Routing Table-ben a **D** betűvel jelennek meg (a Dual algoritmus miatt), és az AD értékük 90 (ha a hálózaton belülről jönnek).

## Kvíz a 27. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod a dinamikus protokollokról!

### Kérdés 1

Az R1 router négy különböző útválasztási protokollt futtat (RIP, EIGRP, OSPF, IS-IS), és mind a négytől kap egy útvonalat, ami hajszálpontosan a `192.168.1.0/24` hálózatra mutat. Melyik útvonal (vagy útvonalak) fog(nak) bekerülni az IP Routing Table-be?

- A) Csak az OSPF útvonal.
- B) Csak az EIGRP útvonal.
- C) A RIP és az EIGRP útvonalak, mert mindkettő Distance Vector.
- D) Mind a négy útvonal bekerül.

**Helyes Válasz: B**

**Magyarázat:** Ha a célpont IP tartománya (és a maszk is) hajszálpontosan megegyezik több protokoll esetében, a router az Adminisztratív Távolságot (AD) használja a döntéshozatalra. A legalacsonyabb AD nyer. RIP = 120, OSPF = 110, IS-IS = 115, EIGRP = 90. Az EIGRP a legmegbízhatóbb, így csak ő kerül be a táblába.

### Kérdés 2

Melyik paranccsal tudsz hirdetni egy alapértelmezett (`0.0.0.0/0`) statikus útvonalat az Internet felé az összes többi, RIP-et futtató szomszédos routerednek?

- A) A Privileged EXEC módban kiadod az `ip route default` parancsot.
- B) A `router rip` konfigurációs módban kiadod a `network 0.0.0.0` parancsot.
- C) A `router rip` konfigurációs módban kiadod a `default-information originate` parancsot.
- D) Globális konfigurációs módban kiadod a `passive-interface default` parancsot.

**Helyes Válasz: C**

**Magyarázat:** A `default-information originate` a hivatalos parancs a RIP (és az OSPF) protokollban arra, hogy a router "befecskendezze" a saját Default Route-ját a dinamikus protokollba, és továbbadja a többieknek. A `network` parancs interfészeket aktivál, nem pedig statikus utakat.

### Kérdés 3

Mi a helyes sorrend (a legmagasabb prioritástól a legalacsonyabbig) annak eldöntésekor, hogy mi legyen egy router EIGRP Router ID-je (Azonosítója)?

- A) Manuális konfiguráció → Legmagasabb Loopback interfész IP → Legmagasabb Fizikai interfész IP
- B) Legmagasabb Fizikai interfész IP → Manuális konfiguráció → Legmagasabb Loopback interfész IP
- C) Manuális konfiguráció → Legalacsonyabb Fizikai interfész IP → Legalacsonyabb Loopback interfész IP
- D) Legmagasabb Loopback interfész IP → Legmagasabb Fizikai interfész IP → Manuális konfiguráció

**Helyes Válasz: A**

**Magyarázat:** A rendszer mindig a manuális emberi beavatkozást tartja a legerősebbnek. Ha ilyen nincs, akkor a Loopback (virtuális) interfészeket preferálja, mert azok sosem "esnek el" kábelszakadás miatt. Végül, ha az sincs, a fizikai portok közül választja a legmagasabb IP-címűt.

### Bónusz Kérdés (Boson ExSim vizsgakérdés - A Legnagyobb CCNA Csapda Emlékeztető!)

(Ezt a kérdést az előző leckében is tárgyaltuk, de rendkívül fontos a fixálása).

A routered kap egy csomagot a `10.20.0.14` cél IP-címre.
A Routing tábládban a következő útvonalak szerepelnek:

- Statikus útvonal a `10.20.0.0/22` felé (AD: 1)
- RIP útvonal a `10.20.0.0/24` felé (AD: 120)
- EIGRP útvonal a `10.20.0.0/26` felé (AD: 90)
- OSPF útvonal a `10.20.0.0/28` felé (AD: 110)

A fenti útvonalak közül melyiket fogja használni a router a csomag továbbítására?

- A) A RIP útvonalat.
- B) A statikus útvonalat, mert annak van a legalacsonyabb AD-je (1).
- C) Az EIGRP útvonalat, mert a dinamikus protokollok közül neki a legalacsonyabb az AD-je.
- D) Az OSPF útvonalat, mert neki van a leghosszabb prefix egyezése (Longest Prefix Match).

**Helyes Válasz: D**

**Magyarázat:** Ha az összes felsorolt útvonal különböző hálózatokra / különböző maszkokra mutat (ahogy jelen esetben a `/22`, `/24`, `/26` és `/28`), az Adminisztratív Távolság (AD) és a Metrika **TELJESEN LÉNYEGTELEN**! A csomagtovábbítás legelső vastörvénye, hogy a router mindig a **Longest Prefix Match**-et (a legpontosabb találatot, jelen esetben a `/28`-at) választja. Az AD-t csak akkor használja, ha a maszkok is (pl. mindkettő `/24`) tökéletesen megegyeznek!
