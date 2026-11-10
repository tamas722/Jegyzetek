# CCNA Bővített Jegyzet – 14. Lecke: Subnetting (Alhálózatok képzése) 2. Rész

Ebben a leckében folytatjuk az alhálózatok képzését (Subnetting). Végigmegyünk az előző leckében feladott házi feladaton, megismerünk egy zseniális matematikai trükköt a gyorsabb számoláshoz, majd rátérünk a nagyobb, Class B hálózatok feldarabolására. A logika minden esetben hajszálpontosan ugyanaz!

## 1. Az előző lecke feladványa és a "Bűvös Szám" (Magic Number) Trükk

A feladat az volt: Oszd fel a `192.168.1.0/24` hálózatot 4 egyenlő alhálózatra, hogy mindegyikbe beférjen legalább 45 számítógép.

Megállapítottuk, hogy a `/26`-os prefix a tökéletes megoldás, mert 6 host bit marad ($2^6 - 2 = 62$ gép hálózatonként).

Tudtuk, hogy az első hálózat a `192.168.1.0/26`. De hogyan kapjuk meg a többit gyorsan?

### A hagyományos (bináris) módszer

- Felírjuk az 1. hálózatot: `192.168.1.0/26`. Kiszámoljuk a broadcast címét úgy, hogy minden host bitet 1-esre állítunk: ez lesz a `.63`.
- A 2. hálózat hálózati címe pontosan a broadcast cím után jön: `192.168.1.64/26`. A host biteket 1-re állítva a broadcast cím: `.127`.
- A 3. hálózat címe az előző után jön: `192.168.1.128/26`. A broadcast címe: `.191`.
- A 4. hálózat címe: `192.168.1.192/26`. A broadcast címe: `.255`.

### A gyors trükk (A "Bűvös Szám")

Ahelyett, hogy mindig átváltanánk binárisba, használhatunk egy sokkal gyorsabb módszert.

Nézzük meg a hálózat utolsó oktettjét binárisan a `/26` esetében:

- A hálózat rész a 128-as és a 64-es helyiértéken lévő két bitet vette kölcsön.
- A hálózat rész **LEGUTOLSÓ bitjének helyiértéke a 64**. Ez a bűvös számunk!

Ez a szám megmondja a hálózatok "lépésközét" (**block size**). Innentől kezdve csak 64-et kell hozzáadni az előzőhöz:

- Subnet 1: `.0`
- Subnet 2: $0 + 64 =$ `.64`
- Subnet 3: $64 + 64 =$ `.128`
- Subnet 4: $128 + 64 =$ `.192`

## 2. Gyakorlat: Hálózat felosztása X darab alhálózatra

**Feladat:** Kaptál egy `192.168.255.0/24` hálózatot. Oszd fel **5 darab egyenlő méretű alhálózatra!**

A gépek (hostok) száma most nem számít, csak az, hogy kijöjjön az 5 alhálózat.

A képlet az alhálózatok számára: $2^X$ (Ahol X a kölcsönvett bitek száma).

- Ha 1 bitet kérünk kölcsön: $2^1 = 2$ alhálózat. (Kevés).
- Ha 2 bitet kérünk kölcsön: $2^2 = 4$ alhálózat. (Kevés).
- Ha 3 bitet kérünk kölcsön: $2^3 = 8$ alhálózat. (**Ez a nyertes!**) Bár nekünk csak 5 kell, 8-nál kevesebbet nem tudunk csinálni, a maradék 3 hálózat megmarad tartaléknak.

Mivel 3 bitet kértünk kölcsön a `/24`-hez, a prefixünk `/27` lesz.
(A `/27` alhálózati maszkja decimálisan: `255.255.255.224`).

**Mik lesznek a hálózatok címei? Alkalmazzuk a trükköt!**

A `/27` esetén 3 bitet vettünk kölcsön (128, 64, 32). A legutolsó hálózati bit helyiértéke a **32**. Ez a bűvös számunk, a lépésköz!

- Subnet 1: `.0/27`
- Subnet 2: `.32/27`
- Subnet 3: `.64/27`
- Subnet 4: `.96/27`
- Subnet 5: `.128/27`

(És a három megmaradt tartalék hálózat: `.160`, `.192` és `.224`).

## 3. Gyakorlat: Melyik alhálózathoz tartozik egy adott gép?

### 1. Feladat

Van egy számítógéped a következő IP-címmel: `192.168.5.57/27`. Melyik alhálózatba (Network address) tartozik ez a gép?

**Megoldás lépésről lépésre:**

1. Váltsuk át az utolsó oktettet (57) binárisra! → `0011 1001` (32+16+8+1).
2. A `/27` miatt tudjuk, hogy az első 3 bit a Hálózat (Network) része, a maradék 5 bit a Host része.
3. A hálózat címének megkapásához **MINDEN host bitet 0-ra kell állítani**.
4. Tehát a `0011 1001`-ből a host biteket kinullázva ez marad: `0010 0000`.
5. Váltsuk vissza decimálisra: A 32-es helyiértéken áll az egyetlen 1-es, így az érték 32.

**Eredmény:** A gép a `192.168.5.32/27` hálózatba tartozik!

### 2. Feladat

Melyik alhálózathoz tartozik a `192.168.29.219/29` IP-című gép?

- A 219 binárisan: `1101 1011` (128+64+16+8+2+1).
- A `/29` azt jelenti, hogy 5 bitet vettünk kölcsön, tehát csak az utolsó 3 bit a Host rész.
- Nullázzuk ki az utolsó 3 bitet: `1101 1000`.
- Váltsuk vissza decimálisra: 128 + 64 + 16 + 8 = **216**.

**Eredmény:** A gép a `192.168.29.216/29` hálózatba tartozik!

## 4. Class B Hálózatok Subnettingje

A Class B hálózatoknál (pl. `/16`) pontosan ugyanaz a matematika, mint a Class C-nél. Az egyetlen különbség, hogy sokkal több host bit áll rendelkezésünkre, így akár a harmadik és a negyedik oktettbe is belenyúlhatunk! (A `show ip route` parancsnál amúgy sincs jelentősége a CIDR jelölések miatt).

### 1. Feladat

Adott a `172.16.0.0/16` hálózat. Szükségünk van **80 darab alhálózatra**. Milyen prefixet használjunk?

- Képlet: $2^X \ge 80$.
- Ha 6 bitet kérünk: $2^6 = 64$ (Kevés).
- Ha 7 bitet kérünk: $2^7 = 128$ (Ez jó!).

Mivel a `/16`-hoz hozzáadtunk 7 bitet, a helyes prefix a **/23**. (Maszk: `255.255.254.0`).

A hálózatok címei (2-es lépésközzel a harmadik oktettben): `172.16.0.0/23`, `172.16.2.0/23`, `172.16.4.0/23`, stb.

### 2. Feladat

Adott a `172.22.0.0/16` hálózat. Célunk **500 darab alhálózat** létrehozása.

- Képlet: $2^X \ge 500$.
- Ha 9 bitet kérünk kölcsön: $2^9 = 512$ alhálózat.

A prefixünk: `/16 + 9 = /25`. Bizony, a Class B hálózatnál bőven belekölcsönözhetünk az utolsó oktettbe is!

### 3. Feladat

Adott a `172.18.0.0/16`. Célunk **250 alhálózat**, és mindegyikbe kell **250 gép (host)**.

- Alhálózatokhoz: $2^X \ge 250$ → 8 bitet kell kölcsönkérni ($2^8 = 256$ hálózat).
- Az új prefix: `/16 + 8 = /24`.
- Ellenőrizzük a hostokat! Mivel 32 bit a teljes cím, és a prefix 24, marad $32 - 24 = 8$ host bitünk. $2^8 - 2 = 254$ kiosztható címünk van hálózatonként. A 254 kényelmesen kiszolgálja a kért 250 gépet. A feladat hibátlanul megoldva!

## Kvíz a 13. Leckéhez (Részletes Levezetésekkel!)

A videó végén Jeremy 5 házi feladatot adott fel. Bár ő a következő leckére ígérte a megoldásokat, a kérésednek megfelelően elkészítettem a kvízt A, B, C, D válaszlehetőségekkel, és rögtön mellékeltem a pontos matematikai levezetéseket! Teszteld a tudásod!

### Kérdés 1

Adott a `172.30.0.0/16` hálózat. Szükségünk van 100 darab alhálózatra, és mindegyikbe be kell férnie legalább 500 gépnek (hostnak). Milyen prefixet (prefix length) kell használnod?

- A) /22
- B) /23
- C) /24
- D) /25

**Helyes Válasz: B (/23)**

**Magyarázat levezetése:**

- **Alhálózatok száma:** 100 hálózat kell. Ehhez 7 bitet kell kölcsönkérni ($2^7 = 128$). Tehát $16 + 7 = 23$. (Tehát a `/23` a jelöltünk).
- **Hostok száma:** Ellenőrizzük le a gépeket! Ha `/23`-as a prefix, akkor marad 9 host bitünk ($32 - 23 = 9$). A képlet szerint: $2^9 - 2 = 512 - 2 = 510$ kiosztható IP címünk van. Mivel 510 nagyobb, mint az elvárt 500, a `/23`-as prefix tökéletesen megoldja mindkét feltételt!

### Kérdés 2

Melyik alhálózathoz (Network address) tartozik a következő IP-címmel rendelkező gép: `172.21.111.201/20`?

- A) `172.21.96.0/20`
- B) `172.21.111.0/20`
- C) `172.21.100.0/20`
- D) `172.21.108.0/20`

**Helyes Válasz: A (`172.21.96.0/20`)**

**Magyarázat levezetése:**

Mivel a prefix `/20`, a turpisság a 3. oktettben (111) történik (hiszen 16 és 24 bit közé esik).

- Váltsuk át a 111-et binárisba: 64+32+8+4+2+1 → `0110 1111`.
- A `/20` azt jelenti, hogy a 3. oktett első 4 bitje a Hálózat része, az utolsó 4 bitje a Host része.
- Hálózat címhez nullázzuk a host biteket: `0110 0000`.
- Váltsuk vissza decimálisra: 64 + 32 = **96**.

A helyes alhálózat tehát a `172.21.96.0/20`.

### Kérdés 3

Mi a Broadcast címe (üzennetszóró címe) annak az alhálózatnak, amiben a `192.168.91.78/26` IP-című gép található?

- A) `192.168.91.127`
- B) `192.168.91.63`
- C) `192.168.91.191`
- D) `192.168.91.255`

**Helyes Válasz: A (`192.168.91.127`)**

**Magyarázat levezetése:**

- Váltsuk a 78-at binárisba: 64+8+4+2 → `0100 1110`.
- A `/26` miatt az utolsó oktett első 2 bitje hálózat, az utolsó 6 bitje host.
- Hálózat címhez lenullázzuk a host biteket: `0100 0000` (azaz decimálisan 64. A hálózat a `91.64`-es).
- De a **Broadcast cím** a kérdés! Ahhoz minden host bitet 1-re kell állítanunk: `0111 1111`.
- Váltsuk vissza decimálisra: 64 + 32 + 16 + 8 + 4 + 2 + 1 = **127**.

### Kérdés 4

Feldarabolod a `172.16.0.0/16` hálózatot 4 darab egyenlő méretű alhálózatra. Mi lesz a MÁSODIK (2.) alhálózat hálózati címe és broadcast címe?

- A) Hálózat: `172.16.64.0`, Broadcast: `172.16.127.255`
- B) Hálózat: `172.16.32.0`, Broadcast: `172.16.63.255`
- C) Hálózat: `172.16.128.0`, Broadcast: `172.16.191.255`
- D) Hálózat: `172.16.64.0`, Broadcast: `172.16.128.255`

**Helyes Válasz: A**

**Magyarázat levezetése:**

- 4 hálózathoz 2 bitet kell kölcsönkérni ($2^2 = 4$). A prefixünk `/18` lesz.
- A `/18` azt jelenti, hogy a 3. oktettből vettünk el 2 bitet (128 és 64). A bűvös számunk (a lépésköz) tehát a **64** lesz a 3. oktettben.
- Írjuk fel a hálózatokat:
  - hálózat: `172.16.0.0/18`
  - hálózat: `172.16.64.0/18`
  - hálózat: `172.16.128.0/18`
- A 2. hálózat címe megvan: `172.16.64.0`. A broadcast cím mindig pontosan eggyel azelőtt van, hogy a következő (3.) hálózat elkezdődne. Ha a 3. hálózat a `128.0`-nál indul, akkor a 2. hálózat az eggyel ez alatti számnál ér véget: `172.16.127.255`.

### Kérdés 5

Fel akarod darabolni a `172.30.0.0/16` hálózatot úgy, hogy minden alhálózatba beférjen 1000 darab számítógép. Maximum hány darab alhálózatot fogsz tudni létrehozni ebből a keretből?

- A) 32
- B) 64
- C) 128
- D) 256

**Helyes Válasz: B (64 alhálózat)**

**Magyarázat levezetése:**

- Hány host bit kell 1000 géphez? $2^N - 2 \ge 1000$. A $2^9 = 512$ (nem elég). A $2^{10} = 1024$. Tehát pontosan **10 host bitre** van szükségünk.
- Egy IPv4 cím összesen 32 bites. Ha 10 bitet fenntartunk a hostoknak, akkor a hálózati prefixünk $32 - 10 = 22$ (azaz `/22` lesz).
- Hány bitet kértünk kölcsön az eredeti `/16`-hoz képest? $22 - 16 = 6$ bitet kértünk kölcsön.
- Mivel 6 bitet mozgattunk át, a létrehozható alhálózatok száma: $2^6 = 64$.
