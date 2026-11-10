# CCNA Bővített Jegyzet – 36. Lecke: Hozzáférés-vezérlési Listák (ACLs) - Alapok és Standard ACL-ek

Ebben a leckében rátérünk a hálózatbiztonság alapjaira: a **Hozzáférés-vezérlési Listákra (Access Control Lists - ACL)**. Az ACL-ekkel utasíthatjuk a routert arra, hogy bizonyos forgalmat engedjen át (Permit), vagy dobjon el (Deny), még akkor is, ha amúgy tudná az utat a cél felé. Bár sok mindenre használhatók, mi a hálózatbiztonság szemszögéből (csomagszűrés - Packet Filtering) vizsgáljuk őket.

> (Vizsgainfó: A CCNA az IPv4 ACL-eket kéri számon, az IPv6 ACL-ek nem képezik a vizsga részét).

## 1. Az ACL Működési Logikája (A "Tűzfal" Alapjai)

Egy ACL nem más, mint egy sorszámozott lista (**Access Control Entries - ACE**-k), ami megmondja a routernek, hogy egy csomaggal mit tegyen.

### A Három Legfontosabb Szabály

- **Fentről Lefelé Történő Feldolgozás (Top-to-Bottom):** Amikor egy csomag megérkezik, a router mindig a legelső (legkisebb sorszámú) szabállyal kezdi az ellenőrzést. (Ezért hívják Szekvenciális listának - ahogy a Boson kvíz is rávilágított).
- **Azonnali Találat (First-Match) Szabály:** Amint a router talál egy szabályt, ami ráillik a csomagra, **VÉGREHAJTJA** a műveletet (Permit vagy Deny), és **AZONNAL KILÉP** az ACL-ből! A lista többi, lejjebb lévő szabályát már meg sem nézi, még akkor sem, ha azok pontosabbak lennének. Emiatt a legspecifikusabb (legszűkebb) szabályokat mindig a lista tetejére kell tenni!
- **Az "Implicit Deny" (A Rejtett Tiltás):** Minden egyes ACL legeslegvégén ott lapul egy láthatatlan, beépített szabály: **"Ha semmi sem illik a csomagra, DOBD EL (Deny All)!"**. Ha nem írsz be legalább egy "Permit" szabályt az ACL-be, a router minden forgalmat blokkolni fog!

### Az ACL Alkalmazása (Inbound vs. Outbound)

Egy ACL-t először létre kell hozni Globális konfigurációs módban, de addig nem csinál semmit, amíg rá nem teszed egy **Interfészre**!

Amikor ráteszed egy interfészre, meg kell mondanod az irányt (Direction) is:

- **Inbound (Befelé):** A router **Mielőtt** megnézné a Routing Table-t (hogy merre továbbítsa), azonnal ellenőrzi az ACL-t. Ha tiltott, eldobja (Kíméli a processzort az útválasztástól).
- **Outbound (Kifelé):** A router már kikeresi az utat, és **mielőtt** kipréselné a csomagot az adott porton, leellenőrzi az ACL-t.

> (Szabály: Egy interfészen, egy irányban **MAXIMUM EGY ACL** lehet aktív! Tehát max. 1 Inbound és 1 Outbound ACL lehet egy porton. Ha ráteszel egy újat, az felülírja a régit!).

## 2. ACL Típusok és Számozási Tartományok

A Cisco routereken rengeteg féle ACL létezik (IP, IPX, AppleTalk stb.). Nekünk az IP ACL-eket kell ismernünk, azon belül is két fő típust:

- **Standard ACL:** Csak és kizárólag a **Forrás IP-címet (Source IP)** képes vizsgálni! Nagyon buta, nem érdekli, hová megy a csomag, vagy milyen porton (pl. HTTP 80).
- **Extended ACL (Kiterjesztett):** Képes Forrás IP-t, Cél IP-t, Protokollt (TCP/UDP) és Port számokat is vizsgálni. (Ezt a következő leckében tárgyaljuk).

Mindkét típust létre lehet hozni **Számmal (Numbered)** vagy **Névvel (Named)**.

**A Vizsgán Kötelezően Memorizálandó Számtartományok (Standard ACL):**

- `1 - 99`
- `1300 - 1999` (Később adták hozzá, amikor az 1-99 már nem volt elég).

(Az Extended ACL-ek tartománya a `100-199` és `2000-2699` lesz).

### A Standard ACL Elhelyezésének Aranyszabálya

Mivel a Standard ACL nagyon buta (csak a feladót nézi), a lehető **legközelebb kell tenni a Célállomáshoz (Destination)!** Ha túl hamar, a feladóhoz közel blokkolod, akkor a feladó nem csak az adott célt, hanem semmi mást sem fog elérni azon a kábelen.

## 3. Standard ACL Konfiguráció (Cisco CLI)

Mivel a maszkolához az EIGRP-nél és az OSPF-nél már megismert **Wildcard Maszkot** (Inverz maszkot) kell használni, a `255.255.255.0` helyett `0.0.0.255`-öt fogunk írni.

### 3.1. Számozott (Numbered) Standard ACL

Itt a Globális konfigurációs módból közvetlenül adjuk ki a parancsokat (Nem lépünk be külön menübe).

```text
Router(config)# access-list 1 deny 192.168.1.0 0.0.0.255
Router(config)# access-list 1 permit 10.0.0.0 0.255.255.255
Router(config)# access-list 1 permit any
```

**Rövidítések / Trükkök:**

- **Egyetlen gép (Host) megadása:** Nem kell beírni, hogy `192.168.1.1 0.0.0.0`. Ehelyett beírhatod úgy is, hogy `host 192.168.1.1`, vagy egyszerűen csak elhagyod a maszkot, és beírod, hogy `192.168.1.1`. Az IOS automatikusan `/32`-ként kezeli.
- **Mindenki más megengedése:** A `0.0.0.0 255.255.255.255` helyett a Cisco az `any` kulcsszót használja (ahogy a fenti példában a 3. sor mutatja). Ezzel véded ki a rejtett Implicit Deny-t.
- **Megjegyzés (Remark):** A konfiguráció olvashatósága érdekében írhatsz jegyzetet: `access-list 1 remark BLOCK_BOB` (Ez nem szabály, csak szöveg).

### 3.2. Nevesített (Named) Standard ACL

Ez az elegánsabb és modernebb mód. Itt belépünk egy külön menübe, és a szabályok automatikusan sorszámokat (**Sequence numbers**: 10, 20, 30...) kapnak. Ha akarsz, kézzel is adhatsz nekik sorszámot (pl. 5), hogy a régi szabályok közé ékelj egy újat!

```text
Router(config)# ip access-list standard BLOCK_BOB
Router(config-std-nacl)# 5 deny host 1.1.1.1
Router(config-std-nacl)# 10 permit any
```

> (Az IOS egy belső optimalizációs algoritmus miatt néha önállóan átrendezi a listát a memóriában: a specifikus `/32`-es Host szabályokat automatikusan a feljebb lévő `/24`-es szabályok elé teszi, hogy gyorsítsa a feldolgozást. Ez az egyetlen kivétel a szigorú Top-to-Bottom szabály alól, de a CCNA vizsgán ezt ritkán kérik számon).

### 3.3. Az ACL felhelyezése az Interfészre

Ha kész az ACL (legyen az számos vagy nevesített), rá kell kötni az interfészre!

```text
Router(config)# interface g0/2
Router(config-if)# ip access-group 1 out
```

> (Figyelem: A parancs az `ip access-group`, nem `access-list`! Az `out` jelenti az Outbound, az `in` az Inbound irányt).

**Ellenőrzés:**

- `show access-lists` (Megmutatja az összes ACL-t és a benne lévő sorszámozott szabályokat).
- `show ip access-lists` (Kizárólag az IP ACL-eket mutatja).

## Kvíz a 36. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod a Standard ACL szabályaiból és mechanizmusából!

### Kérdés 1

Az R2 router G0/1 interfészén kifelé (Outbound) szeretnéd alkalmazni azt az ACL-t, amely **KIZÁRÓLAG a PC1-et és a PC4-et** engedi át a `10.0.1.0/24`-es hálózat felé, mindenki mást kizár. Melyik ACL teljesíti a követelményt?

- A) `access-list 1 permit 192.168.1.1` majd `access-list 1 permit 192.168.2.2`
- B) `access-list 1 deny any` majd `access-list 1 permit 192.168.1.1` majd `access-list 1 permit 192.168.2.2`
- C) `access-list 1 permit any` majd `access-list 1 deny 192.168.1.1` majd `access-list 1 deny 192.168.2.2`
- D) `access-list 1 permit 192.168.1.1` majd `access-list 1 permit 192.168.2.2` majd `access-list 1 permit any`

**Helyes Válasz: A**

**Magyarázat:** Az A válasz pontosan a PC1-et és a PC4-et engedi át. De mi lesz a többiekkel? Itt lép életbe az **"Implicit Deny"** (A rejtett mindent-tiltó szabály a lista végén)! Mivel nem írtunk be semmi mást, a router a maradék forgalmat automatikusan eldobja, így a feladat teljesül! A B azért rossz, mert a legelső sorban letiltott mindenkit, így a router rögtön kilép az ACL-ből, és a PC-k sem mennek át. A D azért rossz, mert a `permit any` miatt a végén mindenkit beenged.

### Kérdés 2

Melyik interfészre és milyen irányban kell felhelyezni a következő Standard ACL-t ahhoz, hogy a lehető legjobb hálózati tervezési elveket (Best Practice) kövessük? (Cél: Letiltani az R1 hálózatát a `10.0.2.0/24` felé, de minden mást megengedni).

- A) Az R1 G0/1 interfészén, befelé (Inbound)
- B) Az R1 G0/2 interfészén, kifelé (Outbound)
- C) Az R2 G0/0 interfészén, befelé (Inbound)
- D) Az R2 G0/2 interfészén, kifelé (Outbound)

**Helyes Válasz: D**

**Magyarázat:** Az Aranyszabály: A Standard ACL-t (mivel csak a feladót nézi, és nem tudja, hová tart a csomag) a lehető **legközelebb kell tenni a Célállomáshoz (Destination)!** Jelen esetben a cél a `10.0.2.0/24` hálózat. Ez az R2-es router mögött van, a G0/2 portján. Tehát az ACL-t az R2 G0/2 portján, **KIFELÉ (Outbound)** kell felrakni, mielőtt leérne a szerverhez. Ha hamarabb raknánk fel (pl. az R1-en), akkor az R1-es gépek az égvilágon semmi mást (pl. az Internetet vagy a `10.0.1.0/24`-et) sem tudnának elérni!

### Kérdés 3

Kiadod a következő **KÉT** parancsot az R2 router egyazon interfészén:

```text
ip access-group 5 out
ip access-group 10 out
```

(Az 5-ös ACL letiltja a `172.16.x.x` hálózatot, a 10-es ACL letiltja a `10.0.x.x` hálózatot). Mi lesz a végeredmény?

- A) Minden forgalom blokkolva lesz.
- B) A `10.0.x.x` hálózat lesz blokkolva.
- C) A `172.16.x.x` hálózat lesz blokkolva.
- D) A `192.168.x.x` hálózat lesz blokkolva.

**Helyes Válasz: B**

**Magyarázat:** A Cisco IOS szabálya, hogy egy interfészen, egy adott irányban (in vagy out) **MAXIMUM EGY ACL LEHET AKTÍV**. Ha kiadod a második parancsot (group 10), az gondolkodás nélkül felülírja (lecseréli) az elsőt (group 5). Így csak a 10-es ACL szabályai (A `10.0.x.x` blokkolása) lesznek érvényben.

### Kérdés 4

Mi történik, ha egy adatcsomag IP-címe **NEM** egyezik meg (nem matchel) az ACL egyetlen beírt szabályával (ACE-vel) sem?

- A) A csomag továbbításra kerül a Default Gateway felé.
- B) A csomagot a router leellenőrzi a következő elérhető ACL-en.
- C) A csomagot a router eldobja (Drop).
- D) A legspecifikusabb találat szabálya lép érvénybe.

**Helyes Válasz: C**

**Magyarázat:** Ez a klasszikus **"Implicit Deny"** (A rejtett mindent-tiltó szabály). Minden Cisco ACL legvégén, láthatatlanul ott van egy `deny any` szabály. Ha a csomag túléli a listát, a végén könyörtelenül eldobják.

### Bónusz Kérdés (Boson ExSim vizsgakérdés - Az ACL Feldolgozása)

Az alábbi állítások közül melyik **IGAZ** a Hozzáférés-vezérlési Listákkal (ACLs) kapcsolatban? Válaszd a legjobb választ!

- A) Az ACL-eket a legkevésbé specifikus bejegyzéstől a leginkább specifikus bejegyzés felé haladva dolgozza fel a router.
- B) Az ACL-eket a listában lévő első bejegyzéstől kezdve az utolsó bejegyzés felé haladva (Fentről lefelé) dolgozza fel a router.
- C) Az ACL-eket a listában lévő utolsó bejegyzéstől kezdve az első bejegyzés felé haladva dolgozza fel a router.
- D) Az ACL-eket a leginkább specifikus bejegyzéstől a legkevésbé specifikus bejegyzés felé haladva dolgozza fel a router.

**Helyes Válasz: B**

**Magyarázat:** Az ACL egy szekvenciális (sorrendi) lista. A router mindig szigorúan a lista legtetején (First entry) kezdi, és halad lefelé. Bár emberként logikus lenne (és a hálózattervezés szabálya is azt mondja), hogy a legspecifikusabb (`/32`) szabályokat rakd a lista tetejére (különben egy bővebb szabály előbb elkapná a csomagot), magát a routert hardveresen egyáltalán nem érdekli a "specifikusság" vagy a subnet maszk mélysége a feldolgozás sorrendjénél! Ő gépiesen fentről lefelé halad, amíg meg nem találja az **ELSŐ** olyan szabályt, ami igaz a csomagra. Amint megvan, azonnal kilép a listából. Tehát a **B** a helyes, mechanikai válasz.
