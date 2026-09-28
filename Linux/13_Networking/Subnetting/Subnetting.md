# Az IP Alhálózatokra Osztás (Subnetting) Alapjai

A subnetting megértéséhez elsőként az IP-címek és a bináris számrendszer működését kell tisztázni.

## 1. Az IP-cím és a Bináris Forma

Az IP-cím négy, pontokkal elválasztott részből (oktettből) áll, amit "pontozott decimális formátumnak" hívunk (pl. `192.168.0.1`). A számítógépek ezt bináris (kettes számrendszerbeli) formában látják, ahol minden oktett 8 bitből áll (tehát az IP cím összesen 32 bites).

A bináris helyiértékek jobbról balra nőnek:

```plaintext
128 | 64 | 32 | 16 | 8 | 4 | 2 | 1
```

**Példa a `192.168.0.1` átváltására:**

- `192` = `11000000` (Mert 128 + 64 = 192, a többi helyiérték 0)
- `168` = `10101000` (Mert 128 + 32 + 8 = 168)
- `0` = `00000000`
- `1` = `00000001`

Így az IP-cím binárisan: `11000000.10101000.00000000.00000001`

## 2. Az Alhálózati Maszk (Subnet Mask) Szerepe

A subnet mask az az eszköz, ami megmondja a hálózati eszközöknek, hogy az IP-címből melyik rész azonosítja a Hálózatot (Network) és melyik rész azonosítja a Klienseket (Hostokat).

**Példa:** Ha a maszk `255.255.255.0` (ami binárisan csupa 1-es az első három oktettben, és csupa 0 az utolsóban), az azt jelenti, hogy az első 3 oktett a hálózat azonosítója, az utolsó oktett (a 0-k) pedig a hosztoké.

**Kötelező címek minden hálózatban:**

Minden alhálózatban le kell foglalni két címet, amelyeket nem kaphat meg számítógép:

- **Hálózati cím (Network Address):** A hálózat kezdő címe (pl. `192.168.0.0`).
- **Szórási cím (Broadcast Address):** A hálózat utolsó címe, amit minden gépnek szóló üzenetekre használnak (pl. `192.168.0.255`).

A kettő között lévő IP-címek (1-től 254-ig) adhatók ki a hosztoknak.

## 3. A Subnetting (Alhálózatokra osztás) Lépései

A subnetting során az IP-cím utolsó oktettjének bitjeit "kölcsönvesszük" a hálózat javára, hogy több, de kisebb hálózatot (subnetet) hozzunk létre.

**Cél:** Osszuk fel a `128.16.8.0`-ás címtartományt úgy, hogy hálózatonként legalább 20 hoszt elférjen!

**Tervezés (Bitek lefoglalása):**

Ahhoz, hogy 20 gépünk (plusz a hálózati és broadcast cím, tehát 22 cím) elférjen, ki kell számolnunk, hány bitre van szükségünk a hosztokhoz.

Helyiértékek: `128 | 64 | 32 | 16 | 8 | 4 | 2 | 1`

Ha az utolsó 5 bitet tartjuk meg a hosztoknak (16+8+4+2+1), az maximum 31 IP-címet ad. (A maradék 3 bitet a hálózat kapja).

**A Subnet Mask kiszámítása:**

Ha az utolsó oktett első 3 bitjét a hálózat kapja, a maszk ott `11100000` lesz.

A helyiértékek (128+64+32) összege 224.

Az új alhálózati maszk: `255.255.255.224`

**Az alhálózatok felosztása (Blokkméret: 32):**

Mivel a hosztok számára 5 bit maradt, minden alhálózat $2^5 = 32$ címet foglal magába.

Az alhálózatok kiosztása a `128.16.8.0`-ás IP-címen:

| Alhálózat (Subnet) | Hálózati Cím | Használható IP Tartomány | Broadcast Cím |
| --- | --- | --- | --- |
| 1. Hálózat | 128.16.8.0 | 128.16.8.1 - 128.16.8.30 | 128.16.8.31 |
| 2. Hálózat | 128.16.8.32 | 128.16.8.33 - 128.16.8.62 | 128.16.8.63 |
| 3. Hálózat | 128.16.8.64 | 128.16.8.65 - 128.16.8.94 | 128.16.8.95 |
| 4. Hálózat | 128.16.8.96 | 128.16.8.97 - 128.16.8.126 | 128.16.8.127 |
| ... | ... | ... | ... |
| 8. Hálózat | 128.16.8.224 | 128.16.8.225 - 128.16.8.254 | 128.16.8.255 |

## 4. Hogyan számol a gép? (Bitwise AND művelet)

A hálózati eszközök úgy találják meg a hálózati címet, hogy a kapott IP-címet és az Alhálózati maszkot binárisan "ÉS" (AND) művelettel összeadják. (A boolean algebrában, ha mindkettő 1-es, az eredmény 1, különben 0).

A 0-k elrejtik az értéket, az 1-esek átengedik azt.

**Példa az 1. hálózatra:**

```plaintext
  11000000.10101000.00000000.00000001   (IP Cím: 192.168.0.1)
  11111111.11111111.11111111.11100000 + (Subnet Mask: 255.255.255.224)
  -----------------------------------
  11000000.10101000.00000000.00000000 = Hálózati cím: 192.168.0.0
```

**Példa a 2. hálózatra:**

```plaintext
  11000000.10101000.00000000.00100000   (IP Cím: 192.168.0.32)
  11111111.11111111.11111111.11100000 + (Subnet Mask: 255.255.255.224)
  -----------------------------------
  11000000.10101000.00000000.00100000 = Hálózati cím: 192.168.0.32
```

##Forrás
-[https://www.linuxjournal.com/article/6287]

# IP Címzés és Alhálózatokra Osztás (Subnetting) Alapjai

A hálózatkezelésben az IP-címzés és az alhálózatokra osztás (subnetting) elengedhetetlen a hatékony adatátvitelhez és a biztonságos hálózati architektúrához.

## 1. IPv4 vs. IPv6 Címzés

Az IP-címek a hálózaton lévő eszközök egyedi azonosítói. Jelenleg két szabvány él egymás mellett:

| Jellemző | IPv4 | IPv6 |
| --- | --- | --- |
| Címtér mérete | 32-bites rendszer | 128-bites rendszer |
| Kapacitás | ~4,3 milliárd cím (Kimerülőben van) | ~340 undecillió cím (Gyakorlatilag végtelen) |
| Formátum | Decimális (Tízes számrendszer) | Hexadecimális (Tizenhatos számrendszer) |
| Példa | 192.168.1.1 | 2001:0db8:85a3:0000:0000:8a2e:0370:7334 |
| Szeparátor | Pont (.) | Kettőspont (:) |

## 2. A Subnetting (Alhálózatokra osztás) Célja

A subnetting egy nagy hálózat logikai felosztása kisebb, jól kezelhető alhálózatokra (subnetekre).

**Miért van rá szükség?**

- **Jobb teljesítmény:** Kisebb hálózatok = kisebb "szórási tartomány" (broadcast domain). Kevesebb a felesleges hálózati "zaj" és broadcast forgalom.

- **Fokozott biztonság:** Az alhálózatok elszigetelhetők egymástól (pl. külön subnet a vendég wifinek és a belső szervereknek), így egy behatolás hatóköre korlátozható.

- **Hatékony IP-gazdálkodás:** Megelőzi a rendelkezésre álló IP-címek pazarlását.

## 3. Subnetting Technikák és Fogalmak

A hálózatok felosztására különböző módszertanok léteznek:

- **FLSM (Fixed-Length Subnet Mask):** Fix hosszúságú alhálózati maszk. A hálózatot egyenlő méretű darabokra osztja. Egyszerű, de IP-címeket pazarolhat, ha az egyik részlegen 100, a másikon csak 2 gép van.

- **VLSM (Variable-Length Subnet Mask):** Változó hosszúságú maszk. Egyazon hálózaton belül különböző méretű alhálózatok hozhatók létre, így pontosan az igényekhez igazítható az IP-kiosztás.

- **CIDR (Classless Inter-Domain Routing):** A modern, "osztály nélküli" címzés. A régi (A, B, C osztályú) merev határok helyett egy perjelet és egy számot használ a hálózati bitek jelölésére (pl. `/24`). Hatékonyabbá teszi az útválasztási táblákat (routing tables).

**A Subnet Maszk működése:**

A 32 bites maszk 1-es bitjei jelentik a hálózatot (network), a 0-s bitjei a klienseket (host).

Például a `255.255.255.0` maszk (ami CIDR formátumban `/24`) azt jelenti, hogy az első 24 bit azonosítja a hálózatot, a maradék 8 bit (256 cím) pedig a gépeket.

## 4. Gyakorlati Példa: Hálózat Felosztása

Adott egy `192.168.1.0/24` hálózat (256 IP-címmel). Ezt szeretnénk felosztani 4 egyenlő méretű alhálózatra.

1. **Szükséges bitek kiszámítása:** 4 alhálózathoz 2 bitet kell "kölcsönkérnünk" a hoszt résztől ($2^2=4$).

2. **Új CIDR és Maszk:** Az eredeti `/24`-hez hozzáadunk 2 bitet, így `/26` lesz az új prefixum. (A maszk `255.255.255.0`-ról `255.255.255.192`-re változik).

3. **Az új alhálózatok** (darabonként 64 IP-címmel):
   - `192.168.1.0/26` (Hosztok: .1 - .62)
   - `192.168.1.64/26` (Hosztok: .65 - .126)
   - `192.168.1.128/26` (Hosztok: .129 - .190)
   - `192.168.1.192/26` (Hosztok: .193 - .254)

(Kiegészítés a point-to-point hálózatokról: Ha a `/24`-es hálózatot `/30`-as darabokra vágnánk, azzal 64 darab apró hálózatot kapnánk. Egy `/30`-as subnetben összesen 4 IP van, amiből csak 2 használható hosztok számára – ez kifejezetten routerek közötti közvetlen kapcsolatokra ideális, pl. `192.168.1.0/30`, `192.168.1.4/30` stb.)

## 5. Hasznos Eszközök Számoláshoz Linuxon

A bináris matek megkönnyítésére Linux alatt kiváló parancssori eszközök állnak rendelkezésre:

- **`ipcalc`:** Parancssori segédprogram, amely egy IP és egy maszk (vagy CIDR) megadásával azonnal kiszámolja a hálózati címet, a broadcast címet, a maszkot és a használható tartományt.

  **Használat:** `ipcalc 192.168.1.0/26`

- **Webes Subnet Kalkulátorok:** Bonyolultabb VLSM tervezéshez érdemes vizuális, online kalkulátorokat használni.

## Forrás
-[https://useful.codes/understanding-ip-addressing-and-subnetting-in-linux/]
-[https://www.howtouselinux.com/post/find-subnet-mask-on-linux]
