# CCNA Bővített Jegyzet – 16. Lecke: VLAN-ok (Virtual LAN) 1. Rész

Ebben a leckében egy hatalmas és rendkívül fontos témakörbe, a **VLAN-ok (Virtuális Helyi Hálózatok)** világába lépünk be. A VLAN-ok megértése elengedhetetlen mind a CCNA vizsgához, mind a valódi hálózati mérnöki munkához. Lássuk, mik ezek és miért van rájuk ekkora szükségünk!

## 1. Alapfogalmak: Mi az a LAN és a Broadcast Domain?

Korábban úgy definiáltuk a LAN-t (Local Area Network), mint egy adott fizikai helyen (pl. iroda, otthon) lévő gépek csoportját. Most viszont nézzünk egy sokkal pontosabb, szakmai definíciót:

> **A LAN egyetlen Broadcast Domain (Üzennetszóró tartomány).**

### Mi az a Broadcast Domain?

Azoknak az eszközöknek a csoportja, amelyek megkapnak egy **Broadcast (üzennetszóró)** keretet, ha azt a csoport bármelyik tagja elküldi. (Emlékeztető: a broadcast keret Cél MAC-címe csupa F, azaz `FFFF.FFFF.FFFF`).

### Hogyan terjed a broadcast?

- Ha egy PC elküld egy broadcast keretet, a Switch megkapja, és **elárasztja (flood)** az összes portján (kivéve azon, ahonnan kapta). Így a hálózat minden gépe megkapja.
- Amikor a broadcast eléri a **Routert**, a router **NEM** továbbítja azt! A routerek megállítják a broadcastokat.

### Példa a tartományok megszámlálására

Képzelj el egy routert, aminek 3 portjára 3 különálló switch van kötve, amiken PC-k vannak, a 4. portjával pedig egy másik routerhez csatlakozik. Hány Broadcast Domain van itt? **Négy.**

Minden egyes switch (a rajta lévő PC-kkel és a router feléjük néző portjával) egy-egy külön tartomány. Illetve a két router közötti közvetlen kapcsolat (még ha csak 2 eszköz is van rajta) szintén egy különálló broadcast tartományt alkot.

## 2. A Probléma: Miért nem jó egy nagy, "lapos" LAN?

Képzeljünk el egy cég irodáját, ahol három osztály dolgozik: **Mérnökség (Engineering)**, **HR** és **Értékesítés (Sales)**. Alapértelmezetten minden gép egyetlen hatalmas switchre van rákötve, és mindenki a `192.168.1.0/24` IP-hálózatban van.

Ezzel két hatalmas probléma van:

- **Biztonság (Security):** Mivel mindenki egyetlen L2-es (Layer 2) hálózatban van, a gépek közvetlenül tudnak kommunikálni egymással. A forgalom nem megy át a routeren, így a routeren hiába is állítanál be biztonsági szabályokat (firewall, hozzáférés-vezérlés), azokat a belső gépek megkerülik. Bárki a Sales-ről közvetlenül hozzáférhet a HR gépeihez.
- **Teljesítmény (Performance):** Ha bárki küld egy broadcastot (vagy a switch eláraszt egy ismeretlen unicastot), azt az iroda összes gépe megkapja. Ez felesleges hálózati forgalmat (torlódást) generál, és jelentősen rontja a teljesítményt.

### A hibás próbálkozás (Csak az IP-címek szétválasztása)

Megpróbálhatjuk megoldani a problémát úgy, hogy 3 külön IP alhálózatra (Subnets) bontjuk a céget a 3. rétegben (Layer 3):

- Mérnökök: `192.168.1.0/26`
- HR: `192.168.1.64/26`
- Sales: `192.168.1.128/26`

**Sikerült megoldani? Részben!** A biztonsági probléma (Unicast) megoldódott, mert most már a különböző alhálózatokban lévő gépeknek (pl. `.1` és `.129`) az Alapértelmezett Átjárón (Routeren) keresztül kell küldeniük a normál adatot egymásnak, így a router tudja ellenőrizni a forgalmat.

**De a teljesítmény probléma megmaradt!** Amikor egy PC egy Broadcast keretet küld, a Switch csak a Layer 2-t látja (MAC-címeket). Neki fogalma sincs az IP alhálózatokról! Meglátja az `FFFF.FFFF.FFFF` célt, és gondolkodás nélkül rázúdítja a broadcastot mind a három osztály összes gépére.

Lehetne venni 3 különálló fizikai switchet a 3 osztálynak, de ez drága és rugalmatlan megoldás lenne. Itt jönnek képbe a VLAN-ok!

## 3. A Megoldás: VLAN (Virtual Local Area Network)

A VLAN lehetővé teszi, hogy egyetlen fizikai switchet szoftveresen több, logikailag teljesen elkülönített, független switchre osszunk fel a 2. rétegben (Layer 2).

- A Mérnökség portjait betesszük a **VLAN 10**-be.
- A HR portjait a **VLAN 20**-ba.
- A Sales portjait a **VLAN 30**-ba.

### Hogyan működik?

A switch minden VLAN-t egy teljesen különálló LAN-ként (külön Broadcast Domainként) kezel. **Egy switch SOHA nem továbbít adatot két különböző VLAN között!**

Ha a Mérnökség egyik gépe küld egy broadcastot, a switch azt kizárólag a VLAN 10-be tartozó portokon fogja elárasztani. A HR és a Sales gépei meg sem érzik a forgalmat!

Ha a VLAN 10 egyik gépe akar beszélni a VLAN 20 egy gépével, azt a switch nem engedi át közvetlenül. A forgalmat ki kell küldeni a Routerhez, ami majd visszairányítja a megfelelő másik VLAN-ba. (Ezt hívják **Inter-VLAN routing**-nak, amiről később részletesen tanulunk).

### VLAN-ok összefoglalása

- A switcheken portonként (interfészenként) konfiguráljuk őket.
- Logikailag szétválasztják a végpontokat a 2. rétegben (L2).
- Csökkentik a felesleges broadcast forgalmat (jobb teljesítmény) és elszeparálják a hálózatokat (nagyobb biztonság).

## 4. Cisco IOS Parancsok: VLAN-ok Konfigurálása

Lássuk, hogyan hozunk létre és rendelünk hozzá VLAN-okat a portokhoz a Cisco parancssorban (CLI)!

### Alapértelmezett VLAN-ok ellenőrzése

A Privileged EXEC módból indítsd el a következő parancsot:

```text
Switch# show vlan brief
```

Eredményként egy táblázatot kapsz, ahol láthatod az eszközön lévő VLAN-okat. Alapból ezek léteznek, és nem törölhetők:

- **VLAN 1 (Név: default):** Alapértelmezetten a switch minden egyes portja ehhez a VLAN-hoz van rendelve!
- **VLAN 1002 - 1005:** Régi, elavult technológiákhoz (FDDI, Token Ring) fenntartott VLAN-ok. (A CCNA-n nem kell velük mélyebben foglalkozni).

### Portok hozzárendelése VLAN-okhoz (Access portok)

Tegyük fel, hogy a Gigabit 1/0-tól a 1/3-ig terjedő portokat be akarjuk tenni a VLAN 10-be.

Lépjünk be a Globális konfigurációs módba, majd az interfész tartományba:

```text
Switch# configure terminal
Switch(config)# interface range g1/0 - 3
```

Beállítjuk a portot **Access port** módba. (Az Access port olyan port, amihez végpontok – PC-k – csatlakoznak, és csak egyetlen VLAN-hoz tartozhat. Létezik még a **Trunk port**, ami több VLAN-t szállít, de arról a következő leckében lesz szó).

```text
Switch(config-if-range)# switchport mode access
```

Hozzárendeljük az interfészt a VLAN 10-hez:

```text
Switch(config-if-range)# switchport access vlan 10
```

> **Figyelem:** Ha a VLAN 10 még nem is létezett a switchen, a rendszer egy üzenetet küld, hogy létrehozta neked automatikusan! (`%Access VLAN does not exist. Creating vlan 10.`).

### VLAN nevek beállítása (Opcionális, de hasznos)

Bár a rendszer létrehozta a VLAN 10, 20 és 30-at, a neveik alapértelmezetten csúnyák (pl. `VLAN0010`). Adjunk nekik érthető neveket! Globális konfigurációs módból be kell lépnünk az adott VLAN konfigurációjába (ez egyben létre is hozza a VLAN-t, ha még nem létezett):

```text
Switch(config)# vlan 10
Switch(config-vlan)# name ENGINEERING

Switch(config-vlan)# vlan 20
Switch(config-vlan)# name HR

Switch(config-vlan)# vlan 30
Switch(config-vlan)# name SALES
```

Ha most újra kiadod a `show vlan brief` parancsot, látni fogod az egyedi neveket, és azt, hogy mely portok lettek áthelyezve az alapértelmezett VLAN 1-ből az új, elszeparált VLAN-okba!

## Kvíz a 16. Leckéhez

Ellenőrizd a tudásod a VLAN-okról az alábbi 5 kérdés segítségével!

### Kérdés 1

Hány Broadcast Domain (Üzennetszóró tartomány) van egy olyan hálózaton, ahol két router van összekötve, és mindkét router egy-egy különálló switchhez csatlakozik, amelyeken 3-3 PC található? (Feltételezve, hogy nincsenek VLAN-ok beállítva).

- A) 1
- B) 2
- C) 3
- D) 6

**Helyes Válasz: C (3 darab)**

**Magyarázat:** Mivel nincsenek VLAN-ok, az 1. switch és a rajta lévő PC-k (a router feléjük néző portjával együtt) alkotnak 1 tartományt. A 2. switch és a PC-k alkotnak még 1 tartományt. A két router közötti közvetlen kapcsolat pedig a 3. tartományt alkotja (mivel a routerek megállítják a broadcastokat a portjaikon).

### Kérdés 2

Ha ugyanazt a fenti hálózatot nézzük, de az első switchen a gépeket betesszük 3 különálló VLAN-ba, a második switchen pedig 2 különálló VLAN-ba, összesen hány Broadcast Domain lesz a hálózaton?

- A) 3
- B) 5
- C) 6
- D) 8

**Helyes Válasz: C (6 darab)**

**Magyarázat:** A VLAN-ok logikailag felszeletelik a switcheket különálló broadcast tartományokra. Az 1. switchen lévő 3 VLAN = 3 tartomány. A 2. switchen lévő 2 VLAN = 2 tartomány. Plusz a két router közötti kapcsolat = 1 tartomány. Összesen 3 + 2 + 1 = 6 Broadcast Domain.

### Kérdés 3

Mi történik a Cisco switchen, ha kiadod a `switchport access vlan 50` parancsot egy interfészen, de a VLAN 50 még nem lett létrehozva?

- A) A parancs hibát dob (fail) és elutasításra kerül.
- B) A switch automatikusan létrehozza a VLAN-t számodra.
- C) Az interfész letiltásra kerül (disabled), amíg manuálisan létre nem hozod a VLAN-t.
- D) Minden 1 és 1005 közötti VLAN alapértelmezetten létezik, így ez a helyzet nem fordulhat elő.

**Helyes Válasz: B**

**Magyarázat:** A Cisco IOS felhasználóbarát funkciója, hogy ha egy még nem létező VLAN-hoz rendelsz egy portot, a szoftver azonnal létrehozza neked a VLAN-t egy üzenet kíséretében (`%Access VLAN does not exist. Creating vlan 50.`).

### Kérdés 4

Ha egy PC a VLAN 20-ban küld egy broadcast üzenetet, hány eszköz fogja azt megkapni a hálózaton? (A VLAN 20-ban összesen 2 PC és 1 Router port található a Switchen).

- A) Csak a Switch kapja meg.
- B) Az összes PC a switchen (függetlenül a VLAN-tól).
- C) 3 eszköz (A switch, a másik PC a VLAN 20-ban és a Router portja).
- D) 2 eszköz (Csak a másik PC és a Router).

**Helyes Válasz: C (3 eszköz)**

**Magyarázat:** A switch először megkapja a keretet, majd – mivel broadcastról van szó – elárasztja azt. De csak és kizárólag a VLAN 20-ba tartozó portokon teheti ezt meg. Így megkapja a hálózatban lévő másik PC és a Router is (amely aztán persze nem továbbítja a saját hálózatán kívülre).

### Kérdés 5

Létrehoztad a VLAN 10, 20 és 30-at egy vadonatúj Cisco switchen. Hány darab VLAN fog megjelenni összesen a `show vlan brief` parancs kimenetében?

- A) 3
- B) 5
- C) 8
- D) 10

**Helyes Válasz: C (8 darab)**

**Magyarázat:** Minden Cisco switchen gyárilag ott van 5 darab törölhetetlen VLAN: a VLAN 1 (az alapértelmezett), valamint a VLAN 1002, 1003, 1004 és 1005 (a legacy technológiáknak). Ezt az 5 alapértelmezettet adod össze az általad létrehozott 3 új VLAN-nal (10, 20, 30), így jön ki a 8.
