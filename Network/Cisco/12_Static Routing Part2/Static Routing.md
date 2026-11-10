# CCNA Bővített Jegyzet – 11. Lecke: Statikus Útválasztás (Static Routing)

Az előző leckében lefektettük az útválasztás (routing) alapjait, és megnéztük az automatikusan létrejövő útvonalakat. Most továbblépünk, és megnézzük, hogyan taníthatjuk meg a routert arra, hogy olyan hálózatokba is el tudja juttatni az adatokat, amelyek nincsenek közvetlenül rákötve. Ez a **Statikus útválasztás (Static routing)**.

## 1. A probléma: Amit a routerek "nem látnak"

Amikor egy routeren beállítasz egy IP-címet, az automatikusan megtanulja a saját címét (**L - Local route**) és a közvetlenül rákötött hálózatot (**C - Connected route**).

**Példa:** Az R2 router ismeri a `192.168.12.0/24` és a `192.168.24.0/24` hálózatokat, mert a kábelek fizikailag be vannak dugva a portjaiba.

**De mi történik, ha egy távoli hálózattal akar kommunikálni?**

A router alapjáraton "vak" a hálózat többi részére. Ha az R2 kap egy csomagot, ami a PC4 felé (`192.168.4.10`) tart, belenéz a routing táblájába, nem talál egyezést, és könyörtelenül eldobja (drop) a csomagot. Hogy ezt elkerüljük, manuálisan kell megmondanunk neki, merre küldje az ilyen csomagokat.

## 2. A Kliensek (PC-k) és az Alapértelmezett Átjáró (Default Gateway)

Mielőtt a routereket konfigurálnánk, értenünk kell, hogyan gondolkodik a végpont (pl. a PC1 és a PC4, amik a példánkban Linux gépek `eth0` hálózati kártyával).

A PC-k tudnak közvetlenül kommunikálni a saját helyi hálózatukon (LAN) belüli gépekkel. De ha egy külső hálózatba akarnak adatot küldeni, a csomagot el kell küldeniük a helyi routerüknek. Ezt hívják **Default Gateway**-nek (Alapértelmezett átjáró).

A PC-ken (pl. PC1) be van állítva egy Gateway IP-cím (pl. `192.168.1.1`, ami az R1 router címe).
Ez gyakorlatilag egy számítógépes "default route", vagyis egy útvonal a `0.0.0.0/0` felé (a `/0` azt jelenti, hogy a címben semmi sem fix, ez az összes létező IP-címet lefedi, ez a legkevésbé specifikus útvonal).

### A MAC és az IP-címek változása utazás közben (Rövid betekintés)

Amikor a PC1 üzenetet küld a PC4-nek:

- **A Cél IP-cím:** Végig a PC4 IP-címe (`192.168.4.10`) marad. (Az IP-címek a feladótól a célig sosem változnak).
- **A Cél MAC-cím:** A PC1 először a saját routerének (R1) a MAC-címét írja a keretre (ehhez futtat egy ARP kérést). Amikor a csomag átér R1-hez, az leveszi a keretet, megnézi az IP-t, majd csinál egy **ÚJ** keretet, amire ráírja a következő router (R3) MAC-címét. (A MAC-címek ugrásról ugrásra (hop-by-hop) folyamatosan változnak!)

## 3. Statikus Útvonalak (Static Routes) Tervezése

A célunk, hogy PC1 és PC4 tudjon egymással kommunikálni. A példában a hálózat felső ágát fogjuk használni: a csomagok az R1 → R3 → R4 útvonalon fognak haladni.

### Kritikus szabály: A Kétirányú elérhetőség (Two-way reachability)

Nem elég, ha a csomag eljut PC1-től PC4-ig. Ha a visszaútra nincs útvonal, a "Ping" sikertelen lesz! Minden routernek az útvonalon tudnia kell, merre van a PC1 hálózata (`1.0/24`) és a PC4 hálózata (`4.0/24`).

**Fontos:** A routereknek nem kell tudniuk az összes köztes hálózatról! Az R1-nek fogalma sem kell legyen az R3 és R4 közötti belső kapcsolatról, neki csak annyit kell tudnia: "Ha PC4-nek küldesz valamit, add oda az R3-nak, a többit ő majd elintézi."

## 4. Cisco Parancsok: Statikus Útvonalak Konfigurálása

Egy statikus útvonal (Static Route) azt mondja meg a routernek: "Ha a Cél hálózatba akarsz küldeni adatot, továbbítsd a csomagot a Next-hop (Következő ugrás) IP-címére."

A parancs formátuma Globális konfigurációs módban:

```text
Router(config)# ip route <Cél-Hálózat> <Subnet-Maszk> <Next-hop-IP>
```

### R1 Konfigurálása

R1 már ismeri a saját `1.0` hálózatát, neki csak azt kell megtanítani, merre van a `4.0/24` (PC4). A következő router az útvonalon R3 (aminek az IP-je `192.168.13.3`).

```text
R1(config)# ip route 192.168.4.0 255.255.255.0 192.168.13.3
```

**Mit látunk a routing táblában (`show ip route`)?**

Egy **S** betűvel jelölt útvonal jelenik meg, ami a "Static" (Statikus) szót jelöli.
Megjelenik mellette egy ilyen szám is: `[1/0]`. Az `1` az **Adminisztratív távolságot (Administrative distance)**, a `0` a **Metrikát** jelenti. Ezekkel a vizsgára még sokat fogunk foglalkozni, egyelőre elég csak felismerni őket.

### R3 Konfigurálása

Az R3 van középen. Neki mind a két PC hálózata távoli, így két útvonalat kell beállítani rajta:

Az `1.0` hálózat felé R1 a következő ugrás (`13.1`):

```text
R3(config)# ip route 192.168.1.0 255.255.255.0 192.168.13.1
```

A `4.0` hálózat felé R4 a következő ugrás (`34.4`):

```text
R3(config)# ip route 192.168.4.0 255.255.255.0 192.168.34.4
```

### R4 Konfigurálása

R4 ismeri a `4.0`-t (PC4), neki csak a PC1 hálózatát kell megmutatni a visszaútra. A következő ugrás visszafelé R3 (`34.3`).

```text
R4(config)# ip route 192.168.1.0 255.255.255.0 192.168.34.3
```

**Eredmény:** Ha most kiadunk egy pinget PC1-ről PC4-re, a válasz sikeres lesz, mert a teljes útvonal és a visszaút is fel van építve!

## 5. Haladó opció: A Kilépő Interfész (Exit Interface) megadása

A Next-hop IP-cím helyett (vagy mellett) megadhatod azt is, hogy a router melyik saját fizikai portján küldje ki a csomagot. (Ezt a videóban az R2-n szemléltettük).

### A) Csak a Kilépő Interfész megadása

```text
R2(config)# ip route 192.168.1.0 255.255.255.0 g0/0
```

**Érdekesség:** Ha ezt használod, a routing táblában az útvonal úgy fog megjelenni, hogy `"is directly connected"` (közvetlenül csatlakoztatott), mintha a gép rá lenne dugva a portra! Ez egy **Proxy ARP** nevű trükkös hálózati funkcióra támaszkodik (amivel a CCNA-n nem kell mélyebben foglalkoznod).

### B) A Kilépő Interfész ÉS a Next-hop együttes megadása

Ez a legprecízebb megoldás:

```text
R2(config)# ip route 192.168.4.0 255.255.255.0 g0/1 192.168.24.4
```

(Ezek közül egyik sem feltétlenül "jobb", mint a másik, bármelyiket használhatod a vizsgán és a valóságban is).

## 6. Az Alapértelmezett Útvonal (Default Route) a Routeren

Ahogy a PC-knek van alapértelmezett átjárója, a routereket is be lehet állítani úgy, hogy: "Ha egy csomag érkezik, de fogalmad sincs, hova kell küldeni (nincs specifikus egyezés a tábládban), akkor küldd el erre az alapértelmezett útvonalra!"

Ezt leggyakrabban arra használják, hogy az összes belső hálózatba nem tartozó forgalmat kiküldjék az Internet felé.

### A parancs (Csupa nullák)

A cél hálózat és a maszk is `0.0.0.0`. Ezzel lefedi a világ összes létező IP-címét.

```text
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

**Hogyan néz ki a routing táblában?**

Egy **S\*** kóddal jelenik meg. A csillag (`*`) jelentése: **Candidate default** (Alapértelmezett útvonal jelölt).

A routing tábla legtetején megjelenik egy üzenet: `"Gateway of last resort is 203.0.113.2 to network 0.0.0.0"` (A végső menedék átjárója be van állítva). Ha korábban nem volt ilyen, ott az állt, hogy `"Gateway of last resort is not set"`.

## Kvíz a 11. Leckéhez

Ellenőrizd a tudásod az alábbi kérdésekkel és a hozzájuk tartozó részletes magyarázatokkal!

### Kérdés 1

Az alábbi parancsok közül melyik állít be egy alapértelmezett útvonalat (default route) a Cisco routeren?

- A) `ip route 0.0.0.0 0.0.0.0 10.1.1.255`
- B) `ip route 0.0.0.0/0 10.1.1.255`
- C) `ip route 0.0.0.0 255.255.255.255 10.1.1.255`
- D) `ip route 0.0.0.0 /0 10.1.1.255`

**Helyes Válasz: A**

**Magyarázat:** A Cisco routereken a statikus útvonalak beállításakor a hálózati maszkot ki kell írni pontozott decimális formátumban (nem használható a peres `/` jelölés, emiatt a B és D hibás). Az alapértelmezett útvonalhoz mind a hálózatot, mind a maszkot csupa nullára (`0.0.0.0 0.0.0.0`) kell állítani. A C válaszban lévő csupa 255-ös maszk a `/32`-nek felelne meg, ami csak egyetlen gépet jelölne ki, nem a teljes internetet.

### Kérdés 2

Nézd meg az R1 router tábláját! Melyik interfészt fogja használni, ha egy `8.8.8.8`-as célállomásra szánt csomagot kell továbbítania? (A táblázatban szerepel több hálózat, és egy S útvonal a `0.0.0.0/0` felé a `203.0.113.2`-es next-hop-pal, ami a GigabitEthernet0/2 interfészen lévő hálózathoz csatlakozik).

- A) GigabitEthernet0/0
- B) GigabitEthernet0/1
- C) GigabitEthernet0/2
- D) Eldobja a csomagot

**Helyes Válasz: C**

**Magyarázat:** Mivel a routing táblában nincs semmilyen specifikus útvonal a `8.8.8.8`-ra, a router a végső menedéket (Default Route) fogja választani (`0.0.0.0/0`). Ez a `203.0.113.2`-es IP-címre küldi a csomagot. Ha megnézzük a táblázatot, a `203.0.113.0` hálózat a GigabitEthernet0/2-es porthoz van közvetlenül csatlakoztatva, így fizikailag itt fog kilépni a csomag.

### Kérdés 3

Adott egy 4 routerből álló zárt körhálózat (R1, R2, R3, R4). Ha a PC1 és PC4 közötti kommunikációhoz az R1 → R2 → R4 útvonalat (alsó ág) választjuk, mik lennének a megfelelő statikus útvonalak?

- A) Az R1-nek csak R2 felé kell út, R2-nek R4 felé, R4-nek nem kell semmi.
- B) Minden routernek az útvonalon ismernie kell mind a PC1 hálózatát, mind a PC4 hálózatát (kétirányú elérhetőség).
- C) Az R2-nek elegendő csak a PC4 hálózatát ismernie.

**Helyes Válasz: B**

**Magyarázat:** A routing legfontosabb szabálya a kétirányú elérhetőség (two-way reachability). A PC1-ről elinduló csomaghoz R1-nek kell egy útvonal a `4.0/24` felé, R2-nek szintén a `4.0/24` felé. De a Ping válaszhoz visszafelé R4-nek kell egy útvonal az `1.0/24` felé, és R2-nek is visszafelé az `1.0/24` felé! Ha valamelyik hiányzik, a csomag elveszik.

### Kérdés 4

A `show ip route` parancs kimenetében a következő sort látod: `S 172.20.0.0/16 is directly connected, GigabitEthernet0/1`. Milyen paranccsal konfigurálták ezt a statikus útvonalat?

- A) `ip address 172.20.0.0 255.255.0.0`
- B) `ip route 172.20.0.0 255.255.0.0 10.1.1.1`
- C) `ip route 172.20.0.0 255.255.0.0 g0/1`
- D) `ip route 172.20.0.1 255.255.0.0 g0/1`

**Helyes Válasz: C**

**Magyarázat:** Az `S` betű elárulja, hogy ez egy statikus útvonal (nem pedig `C`, ami az automatikus lenne az IP-cím beállításakor, így az A kizárva). A trükk az, hogy a feliratban a hálózat "directly connected" (közvetlenül csatlakoztatottként) szerepel. Ez a jelenség pontosan akkor történik meg, ha a statikus útvonal konfigurálásakor nem Next-hop IP-címet, hanem csak egy Kilépő Interfészt (Exit interface) (jelen esetben a `g0/1`-et) adunk meg.

### Kérdés 5

Tegyük fel, hogy a leckében szereplő hálózat közepén álló R3 router csak a két közvetlenül rákötött hálózatot ismeri. Hány darab statikus útvonalat kellene manuálisan bekonfigurálnod rajta ahhoz, hogy a diagramon szereplő ÖSSZES távoli hálózatot ismerje? (A diagramon van: PC1 hálózata, PC4 hálózata, egy belső kapcsolat R1 és R2 között, és egy R2 és R4 között).

- A) Kettőt
- B) Hármat
- C) Négyet
- D) Ötöt

**Helyes Válasz: C (Négyet)**

**Magyarázat:** Az R3 a G0/0 portján ismeri a `13.0/24`-es hálózatot, a G0/1-en pedig a `34.0/24`-est. Ezeken kívül van még négy hálózat a képen, amit a router "nem lát": PC1 hálózata (`1.0/24`), PC4 hálózata (`4.0/24`), R1-R2 közötti hálózat (`12.0/24`), és az R2-R4 közötti hálózat (`24.0/24`). Ha azt akarod, hogy az R3 mindenhova tudjon adatot küldeni a topológiában, erre a 4 "távoli" (remote) hálózatra kell statikus útvonalat írnod.
