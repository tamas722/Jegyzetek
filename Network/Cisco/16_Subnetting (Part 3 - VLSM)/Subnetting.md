# CCNA Bővített Jegyzet – 15. Lecke: Subnetting (Alhálózatok képzése) 3. Rész és a VLSM

Ebben a videóban lezárjuk a Subnetting (alhálózat-képzés) témakörét. Először ellenőrizzük az előző lecke végén feladott házi feladatokat (részletes levezetésekkel), majd megnézzük, hogyan kell számolni a hatalmas Class A hálózatokkal. Végül megismerkedünk a hálózatépítés egyik leghasznosabb és legokosabb koncepciójával: a **VLSM**-mel (Variable-Length Subnet Masks – Változó Hosszúságú Alhálózati Maszkok).

## 1. Az Előző Lecke Házi Feladatainak Megoldása

Nézzük meg lépésről lépésre, hogyan kellett megoldani a 14. lecke végén kapott feladatokat!

### 1. Feladat

Adott a `172.30.0.0/16` hálózat. 100 darab alhálózatra van szükség, és mindegyikben legalább 500 gépnek (hostnak) kell lennie. Milyen prefixet (prefix length) kell használnod?

- **Lépés 1 (Alhálózatok):** Hány bitet kell kölcsönkérni 100 hálózathoz? Számolhatsz az ujjaidon: 2, 4, 8, 16, 32, 64, 128. Ehhez **7 bitet** kellett kölcsönkérni ($2^7 = 128$).
- **Lépés 2 (Hostok):** Ha a `/16`-hoz hozzáadunk 7 bitet, a prefixünk `/23` lesz. Egy IPv4 cím 32 bites, így marad 9 host bitünk ($32 - 23 = 9$). Elbír ez 500 gépet? $2^9 - 2 = 512 - 2 = 510$ gép. Igen, tökéletes!

**Megoldás:** A helyes prefix a **/23** (Dotted decimal formátumban: `255.255.254.0`).

### 2. Feladat

Melyik alhálózathoz (Network address) tartozik a `172.21.111.201/20` IP-című gép?

- A `/20` azt jelenti, hogy a 3. oktettben (111) van a választóvonal (az első 4 bitje a Hálózat, az utolsó 4 bitje a Host része).
- Váltsuk át a 111-et binárisba: `0110 1111` (64+32+8+4+2+1).
- A hálózat címéhez minden host bitet 0-ra állítunk: `0110 0000`.
- Visszaváltva decimálisra: 64 + 32 = 96.

**Megoldás:** A gép a `172.21.96.0/20` alhálózatba tartozik.

### 3. Feladat

Mi a Broadcast címe (üzennetszóró címe) a hálózatnak, amiben a `192.168.91.78/26` gép van?

- A `/26` miatt az utolsó oktettben (78) a választóvonal az első 2 bit után van (az utolsó 6 bit a Host része).
- A 78 binárisan: `0100 1110` (64+8+4+2).
- A Broadcast címhez minden host bitet 1-esre kell állítani: `0111 1111`.
- Visszaváltva decimálisra: 64 + 32 + 16 + 8 + 4 + 2 + 1 = 127.

**Megoldás:** A broadcast cím a `192.168.91.127`.

### 4. Feladat

Feldarabolod a `172.16.0.0/16` hálózatot 4 egyenlő alhálózatra. Mi a MÁSODIK (2.) alhálózat Hálózati és Broadcast címe?

- 4 hálózathoz 2 bitet kell kölcsönkérni ($2^2 = 4$). A prefix `/18` lesz.
- A `/18` a 3. oktettből vett el 2 bitet (a 128-ast és a 64-est). A "Bűvös Szám" (lépésköz) tehát a **64**.
- A hálózatok: 1. (`.0.0`), 2. (`.64.0`), 3. (`.128.0`), 4. (`.192.0`).

**Megoldás:** A második alhálózat hálózati címe `172.16.64.0/18`. A broadcast címe (eggyel a 3. hálózat előtt): `172.16.127.255`.

### 5. Feladat

A `172.30.0.0/16` hálózatot úgy darabolod fel, hogy mindenhol 1000 host legyen. Hány alhálózatot tudsz létrehozni?

- 1000 hosthoz 10 host bitre van szükség ($2^{10} - 2 = 1022$).
- Ha a 32 bitből 10 a hostoké, akkor a prefix $32 - 10 =$ `/22`.
- Eredetileg `/16`-unk volt. $22 - 16 = 6$ bitet kértünk kölcsön.
- $2^6 = 64$.

**Megoldás:** 64 darab alhálózatot tudunk létrehozni.

## 2. Class A Hálózatok Subnettingje

Bár a Class A hálózatok (pl. a `/8`) óriásiak (24 host bit áll rendelkezésre), az alhálózatokra bontásuk hajszálpontosan ugyanazzal a logikával működik, mint a Class C vagy Class B esetében. Csak a számok nagyobbak!

### 1. Gyakorlat

Kaptál egy `10.0.0.0/8` hálózatot. Létre kell hoznod **2000 darab alhálózatot**. Milyen prefixet használsz, és hány gép (host) fér be hálózatonként?

- Hány bit kell 2000 hálózathoz? Számoljunk: 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048. (Ehhez **11 bitet** kellett kölcsönkérni, $2^{11}$).
- **Prefix:** `/8 + 11 = /19`.
- **Hostok száma:** Marad 13 host bitünk ($32 - 19 = 13$). $2^{13} - 2 = 8192 - 2 =$ **8190 host** hálózatonként.

### 2. Gyakorlat

Adott a PC1 a `10.217.182.223/11` IP-címmel. Keresd meg a hálózat 5 alapértékét!

- A `/11` miatt a választóvonal a 2. oktettben (217) van (az első 3 bit hálózat, a maradék 5 bit host).
- Váltsuk át a 217-et binárisba: `1101 1001` (128+64+16+8+1).
- **Hálózat cím:** Csak az első 3 bitet tartjuk meg: `1100 0000` (128+64 = 192). Így a hálózat: `10.192.0.0`.
- **Broadcast cím:** A host biteket 1-re állítjuk: `1101 1111` (128+64+16+8+4+2+1 = 223). A 3. és 4. oktett is csupa 1-es lesz (255). Így a broadcast: `10.223.255.255`.
- **Első kiosztható cím:** `10.192.0.1` (Hálózat + 1)
- **Utolsó kiosztható cím:** `10.223.255.254` (Broadcast - 1)
- **Hostok száma:** Mivel 21 host bitünk maradt ($32 - 11$): $2^{21} - 2 =$ **2 097 150 gép!**

## 3. VLSM (Variable-Length Subnet Masks) – A Haladó Subnetting

Eddig az úgynevezett **FLSM** (Fixed-Length Subnet Masks – Fix hosszúságú maszkok) módszerét használtuk. Ennek az a lényege, hogy egy nagy hálózatot 4, 8 vagy 16 tökéletesen egyenlő darabra vágunk.

Mi ezzel a baj? Hatalmas pazarláshoz vezethet! Ha az egyik irodádban 100 gép van, a másikban csak 2, az FLSM-mel mindkettőnek adnod kell egy 128 fős hálózatot.

A megoldás a **VLSM (Változó Hosszúságú Alhálózati Maszkok)**. Ezzel a technikával "testre szabhatod" az alhálózatok méretét, akár egy szabó a ruhát, hogy pontosan annyi IP-címet foglaljanak le, amennyire szükség van, különböző prefixeket használva egyetlen főhálózaton belül!

> **A VLSM Aranyszabálya:**
> Mindig a **LEGNAGYOBB** hálózati igénnyel rendelkező alhálózattal kezdd, majd haladj csökkenő sorrendben a legkisebb felé!

### VLSM Szituációs Gyakorlat

Kaptál egy `192.168.1.0/24` méretű hálózatot (256 cím). Ebből kell ellátnod a vállalat következő részeit:

- **Tokyo LAN A:** 110 host
- **Tokyo LAN B:** 8 host
- **Toronto LAN A:** 29 host
- **Toronto LAN B:** 45 host
- **Point-to-Point (P2P)** kapcsolat a két város routere között: 2 host

#### 1. Lépés: Rakjuk sorrendbe a legnagyobb igénytől a legkisebbig!

1. Tokyo LAN A (110) → 2. Toronto B (45) → 3. Toronto A (29) → 4. Tokyo B (8) → 5. P2P (2).

#### 2. Lépés: Osszuk ki a hálózatokat sorban!

(Minden hálózat ott kezdődik, ahol az előző broadcast címe véget ért!)

1. **Tokyo LAN A (110 host):**
   - 110 géphez 7 host bit kell ($2^7-2 = 126$ cím). Prefix: `/25`.
   - Hálózat: `192.168.1.0/25`
   - Broadcast: `.127`
2. **Toronto LAN B (45 host):**
   - Kezdőcím: `.128`. 45 géphez 6 host bit kell ($2^6-2 = 62$ cím). Prefix: `/26`.
   - Hálózat: `192.168.1.128/26`
   - Broadcast: `.191` (128 + 63)
3. **Toronto LAN A (29 host):**
   - Kezdőcím: `.192`. 29 géphez 5 host bit kell ($2^5-2 = 30$ cím). Prefix: `/27`.
   - Hálózat: `192.168.1.192/27`
   - Broadcast: `.223` (192 + 31)
4. **Tokyo LAN B (8 host):**
   - Kezdőcím: `.224`. 8 géphez 4 host bit kell ($2^4-2 = 14$ cím). (Vigyázz! A 3 bit nem elég, mert $2^3-2=6$!). Prefix: `/28`.
   - Hálózat: `192.168.1.224/28`
   - Broadcast: `.239` (224 + 15)
5. **P2P kapcsolat (2 host):**
   - Kezdőcím: `.240`. 2 géphez 2 host bit kell (a CCNA-n hagyományosan a `/30`-at preferálják, ami $2^2-2=2$ címet ad, bár a valóságban a `/31` is jó lenne). Prefix: `/30`.
   - Hálózat: `192.168.1.240/30`
   - Broadcast: `.243` (240 + 3)

**Az eredmény:** Gyönyörűen, hézagmentesen "bepakoltuk" mind az öt hálózatot az egyetlen `/24`-es blokkunkba (egészen `.243`-ig), úgy, hogy senki nem maradt IP-cím nélkül, és a végén még maradt is egy kis szabad helyünk (a `.244` és `.255` között) jövőbeli bővítésre! (Fix hálózatokkal ez lehetetlen lett volna).

## 4. További Gyakorlási Lehetőségek (Házi Feladat)

A Subnetting tipikusan olyan készség, amit csak és kizárólag gyakorlással lehet elsajátítani. Ahhoz, hogy a vizsgán gondolkodás nélkül menjen a bináris matek, rutint kell szerezni!

Jeremy ebben a videóban nem ad külön kvízt, helyette házi feladatként azt kéri, hogy a következő egy hétben minden nap oldj meg legalább egy feladatot az alábbi weboldalak mindegyikén:

- **subnettingquestions.com**
- **subnetting.org**
- **subnettingpractice.com** (Jeremy személyes kedvence, mert komplex és kihívást jelentő kérdéseket ad).

A következő leckében egy Packet Tracer gyakorlati labor vár ránk, ahol élesben (Cisco CLI) fogjuk beállítani a most kiszámolt alhálózatokat és IP-címeket a routereken!
