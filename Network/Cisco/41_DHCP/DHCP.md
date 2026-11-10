# CCNA Bővített Jegyzet – 41. Lecke: Dynamic Host Configuration Protocol (DHCP)

Ebben a leckében az IP Services (IP Szolgáltatások) modul második pillérével, a **DHCP**-vel (Dynamic Host Configuration Protocol) foglalkozunk. Ha a DNS az Internet telefonkönyve volt, akkor a DHCP a hálózat "címkiosztó hivatala". Feladata, hogy a hálózatra csatlakozó új eszközök (PC-k, telefonok) azonnal, automatikusan megkapják a működésükhöz szükséges összes hálózati paramétert anélkül, hogy a rendszergazdának kézzel kellene azokat beállítgatnia.

> (A CCNA vizsgatémák 4.3-as és 4.6-os pontjai kifejezetten igénylik a DHCP szerepének ismeretét, valamint a kliens, a szerver és a relay agent konfigurálását).

## 1. A DHCP Alapvető Célja és Működése (A DORA Folyamat)

Amikor egy számítógép bekapcsol, vagy csatlakozik a Wi-Fi-hez, még semmilyen hálózati címmel nem rendelkezik. Hogy megkapja az IP-jét, egy 4 lépésből álló üzenetváltást (**DORA folyamat**) hajt végre a DHCP szerverrel.

### A Portok (Nagyon fontos vizsgaanyag!)

- **DHCP Szerver:** UDP port **67**
- **DHCP Kliens:** UDP port **68**

### A DORA 4 Lépése

- **D – Discover (Felfedezés):** A kliens gépnek még sincs IP-címe (Forrás IP: `0.0.0.0`), és a szervert sem ismeri. Ezért egy **Broadcast** üzenetet küld ki a hálózatra (`255.255.255.255` cél IP-vel és `FFFF.FFFF.FFFF` cél MAC-címmel), amiben megkérdezi: "Van itt DHCP szerver? Kellene egy IP-cím!".
- **O – Offer (Ajánlat):** A DHCP szerver (vagy a szerverek, ha több is van) meghallja a hívást, és válaszol egy ajánlattal. Ebben megnevez egy szabad IP-címet (pl. `192.168.1.100`), megadja a maszkot, a Gateway-t és a bérleti időt. (Ez a válasz lehet Unicast vagy Broadcast is, attól függően, mit kért a kliens a Bootp Flags mezőben).
- **R – Request (Igénylés):** A kliens választ az ajánlatok közül (általában az elsőre, amit kap). Küld egy **Broadcast** üzenetet a hálózatra, amiben jelzi: "Köszönöm, elfogadom az X szerver által ajánlott IP-címet!". (A Broadcast azért kell, hogy a többi DHCP szerver is lássa: "Jó, akkor én visszahúzom a saját ajánlatomat, más már lefoglalta").
- **A – Acknowledgement (Elismerés / Nyugta):** A kiválasztott DHCP szerver visszaküld egy végső nyugtát (Ack). Ezt megkapva a kliens végre rácsavarja (konfigurálja) az IP-címet a saját hálózati kártyájára, és elkezdi a normális adatforgalmat.

### További üzenettípusok

- **Release (Elengedés):** Ha a gép lekapcsolódik vagy ki akarja adni az IP-t, küld egy `ipconfig /release` parancsot a szervernek (Unicast üzenetben), hogy "Visszaadtam a címet, más is megkaphatja".
- **Renew (Megújítás):** Ha lejár a bérleti idő (Lease time) fele, a gép automatikusan megpróbálja megújítani a címét a szervernél.

## 2. A DHCP Relay Agent (A Közvetítő Ügynök)

A hálózati alapokból tudjuk, hogy a Broadcast üzenetek (mint a DHCP Discover) **nem hagyják el a helyi alhálózatot (Subnet)**. A routerek könyörtelenül eldobják őket.

Mi történik akkor, ha a kisgép az A hálózaton van, de a központi DHCP szerver a céges adatközpontban, a B hálózaton csücsül? A DHCP Discover sosem jut el hozzá.

### A Megoldás: A DHCP Relay Agent (IP Helper-Address)

Beállítjuk a helyi hálózatunk felé néző router interfészt, hogy játsza át a lokális Broadcast kéréseket a távoli szerver felé **Unicast** csomagként!

### Hogyan működik a Relay?

1. A PC kiküldi a Broadcast DHCP Discover üzenetet.
2. A router (a Relay Agent) megkapja a csomagot a belső portján. Látja, hogy ez egy DHCP kérés, de nem dobja el.
3. A router átalakítja a csomagot: a Forrás IP-címnek beírja a saját belső interfészének IP-jét, a Cél IP-címnek pedig berakja a távoli DHCP szerver IP-jét (és Unicastként elküldi az útvonalon keresztül a szervernek).
4. A szerver megkapja, látja a router IP-jéből, hogy melyik alhálózatról jött a kérés, kiválaszt egy IP-t abból a poolból, és visszaküldi a routernek az Offer/Ack csomagot. A router pedig átadja a PC-nek.

## 3. Cisco IOS Konfiguráció (Szerver, Relay és Kliens)

### 3.1. A Cisco Router, mint DHCP Szerver

Ha a kisméretű hálózatban a routerünk osztja ki az IP-címeket a PC-knek:

```text
Router(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
Router(config)# ip dhcp pool HOME_LAN
Router(config-dhcp)# network 192.168.1.0 255.255.255.0
Router(config-dhcp)# default-router 192.168.1.1
Router(config-dhcp)# dns-server 8.8.8.8
Router(config-dhcp)# domain-name jeremysitlab.com
Router(config-dhcp)# lease 0 5 30
```

- `ip dhcp excluded-address`: **Kritikus lépés!** Megtiltjuk a szervernek, hogy kiosztsa az elején lévő címeket (itt a `.1`-től `.10`-ig), mert ezeket fenntartjuk a statikus eszközöknek (nyomtatók, szerverek, routerek).
- `ip dhcp pool`: Létrehozunk egy nevű készletet (pool).
- `network`: Megadjuk a hálózatot és a maszkot (vagy `/24`-et is írhatsz).
- `default-router`: Megmondjuk a klienseknek, mi lesz a Gateway-ük.
- `lease`: Megadjuk a bérleti időt (Nap, Óra, Perc formátumban).

**Ellenőrzés:**

- `show ip dhcp binding`: Megmutatja az aktuálisan kiadott IP-címeket, a kliensek MAC-címét és a lejárati időt.

### 3.2. A Cisco Router, mint DHCP Relay Agent

Ha a router csak közvetítőként szolgál a távoli szerver felé:

```text
Router(config)# interface g0/1
Router(config-if)# ip helper-address 10.10.3.5
```

(Az `ip helper-address` parancsot azon a porton kell kiadni, ahová a kliens PC-k csatlakoznak, és a paraméterként a **távoli DHCP szerver** IP-jét kell megadni!).

### 3.3. A Cisco Router, mint DHCP Kliens (Ritka, de lehetséges)

Ha azt akarod, hogy a routered saját maga is egy külső szolgáltatótól kapjon IP-címet a WAN portján:

```text
Router(config)# interface g0/1
Router(config-if)# ip address dhcp
```

## Kvíz a 41. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod a DHCP folyamatból és a parancsokból!

### Kérdés 1

Melyik a helyes sorrendje a DHCP protokoll 4 üzenetének, amikor egy kliens IP-címet kér és kap a szervertől (A DORA folyamat)?

- A) Request, Offer, Discover, Acknowledgement
- B) Discover, Offer, Request, Acknowledgement
- C) Discover, Request, Offer, Acknowledgement
- D) Offer, Discover, Request, Acknowledgement

**Helyes Válasz: B**

**Magyarázat:** Kiváló emlékeztető a **D-O-R-A** mozaikszó: Discover (Felfedezés) → Offer (Ajánlat) → Request (Igénylés) → Acknowledgement (Nyugta).

### Kérdés 2

Melyik Windows Parancssori (Command Prompt) parancs kényszeríti ki a PC-nél, hogy eldobja a jelenlegi IP-címét, és újat kérjen a hálózati DHCP szervertől?

- A) `ipconfig /release`
- B) `ipconfig /flushdns`
- C) `ipconfig /renew`
- D) `ipconfig /all`

**Helyes Válasz: C**

**Magyarázat:** Az `ipconfig /renew` parancs elküld egy új DHCP Discover folyamatot a hálózatra. (A `/release` eldobja, a `/flushdns` a DNS gyorsítótárat üríti ki, az `/all` pedig listázza az adatokat).

### Kérdés 3

Amikor egy Cisco routeren konfigurálod a DHCP szervert, melyik paranccsal tudod megakadályozni, hogy a szerver kiosztsa a hálózati alap-gateway és a nyomtató IP-címét a dinamikus klienseknek?

- A) `ip dhcp reserved-address`
- B) `ip dhcp excluded-address`
- C) `ip dhcp static-address`
- D) `ip dhcp block-address`

**Helyes Válasz: B**

**Magyarázat:** Az `ip dhcp excluded-address` paranccsal tudunk címtartományokat (vagy egyedi címeket) kizárni a dinamikus kiosztás alól, védve a statikus eszközeinket az IP-ütközéstől (IP Address Conflict).

### Kérdés 4

Az alábbi DHCP üzenetek közül melyeket küldi **KIZÁRÓLAG a Kliens** a Szerver felé? (Válassz KETTŐT!)

- A) DHCP Discover
- B) DHCP Offer
- C) DHCP Request
- D) DHCP Acknowledgement (Ack)
- E) DHCP Release

**Helyes Válasz: A és E**

**Magyarázat:** A Discover-t a kliens küldi (mert ő akar címet), és a Release-t is a kliens küldi (mert ő mondja le a címet). Az Offer és az Ack kizárólag a szerver válaszüzenetei. A Requestet bár a kliens küldi, de a hálózatban lévő összes szerver megkapja (Broadcast), hogy lássák, kit választott.

### Bónusz Kérdés (Boson ExSim vizsgakérdés - A Relay Agent Telepítése)

A hálózatodban a DHCP szolgáltatást a NetworkB alhálózatban lévő szerver biztosítja. A routereken önmagukban nincsenek futó DHCP szerver szolgáltatások.

Milyen parancsot **KELL** kiadnod a NetworkA alhálózat felé néző routeren (RouterA) ahhoz, hogy a NetworkA-n lévő kliensek sikeresen IP-címet kapjanak a távoli szervertől? Válaszd a legjobb választ!

- A) `ip dhcp-relay 10.10.3.5` (A szerver címe a paraméter)
- B) `ip helper-address 10.10.1.1` (A lokális router címe a paraméter)
- C) `ip helper-address 10.10.3.5` (A szerver címe a paraméter)
- D) `ip dhcp server 10.10.3.5`

**Helyes Válasz: C**

**Magyarázat:** A Cisco IOS-ben a Relay Agent funkciót a klasszikus `ip helper-address` paranccsal hívjuk életre.

Ezt a parancsot azon a routeren (RouterA) kell kiadni, amelyik közvetlenül látja a DHCP klienseket (Broadcast tartomány).

A parancs után a **távoli DHCP szerver IP-címét** (`10.10.3.5`) kell megadni, nem pedig a saját magadét! Így a router tudni fogja, hogy a helyi broadcast kéréseket erre az IP-re kell átfordítania Unicast csomaggá.
