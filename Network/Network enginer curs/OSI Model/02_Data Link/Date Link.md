# Az OSI-modell 2. és 3. rétegének megértése

**Network+**
**Network**
**2024. december 18.**
**Steve Linthicum**

Az Open Systems Interconnection (OSI) modell olyan rétegek sorozata, amelyeket a számítógépes rendszerek a kommunikációhoz használnak. Hét réteg létezik, és a hálózati réteg a 3. réteg. A hálózati réteg felelős a csomagtovábbításért, beleértve az útválasztást a köztes routereken keresztül. Amikor azonban a hálózati adatkézbesítés megértéséről van szó, át kell jutnunk a 2. rétegen, mielőtt továbblépnénk a 3. rétegre.

Az **adatkapcsolati réteg**, vagyis a 2. réteg, az a protokollréteg, amely a hálózati szegmensen lévő csomópontok között továbbítja az adatokat a fizikai rétegen keresztül, vagy ami közismert nevén egy hoszt fizikai címe. A 2. réteg két részre oszlik: a **MAC** és az **adatkapcsolati** alrétegekre, amelyek a címzést és az adatkeretek elrendezését részletezik, míg a 3. réteg tartalmazza a hoszt **logikai címét**. Nézzük meg közelebbről, hogyan működnek együtt ezek a rétegek a zökkenőmentes adatáramlás biztosítása érdekében.

## Mi az OSI-modell hét rétege?

Mielőtt mélyen belemerülnénk a 2. és 3. rétegbe, kezdjük egy kis háttérinformációval az Open Systems Interconnection (OSI) modellről. Az OSI-modellt a Nemzetközi Szabványügyi Szervezet (ISO) fejlesztette ki. Leírja azokat a rétegeket, amelyeket a számítógépes rendszerek a hálózaton keresztüli kommunikációhoz használnak, és ez volt az első szabványos hálózati kommunikációs modell, amelyet az összes jelentős számítógépes és távközlési vállalat elfogadott az 1980-as évek elején.

Az OSI-modellnek hét rétege van:

1. Fizikai
2. Adatkapcsolati
3. Hálózati
4. Szállítási
5. Munkamenet
6. Megjelenítési
7. Alkalmazási

## Mi a 2. réteg az OSI-modellben?

Ahogy fentebb említettük, a 2. réteg két részre oszlik: a **media access control (MAC)** alrétegre és az **adatkapcsolati** alrétegre. Ezek az alrétegek egyaránt hozzájárulnak a számítógép fizikai címéhez.

### Media Address Control (MAC) alréteg

Hasznos egy ismerős és könnyen érthető példát bemutatni a címzés jellemzőinek megértéséhez.

Tekintsük ezt egy lakópark példáján. Minden lakóparkban lévő házhoz egy sorozatszám van tartósan rögzítve a ház vázára. A számítógépeknek hasonló címük van, amelyet **media access control (MAC) címnek** neveznek. Ez a cím a számítógép hálózati interfészkártyájára van „égetve", és egyedi jellegű. A MAC-cím példája az alábbi 1. ábrán látható.

![1. ábra: Media Access Control (MAC) cím]

A MAC (vagy fizikai) címzés hatékony módszert nyújt az adatok mozgatására a számítástechnikai eszközök egy kis közösségében, ahol nincs szükség a forgalom útválasztására több közösség között. Ezeknek a kis hálózatoknak a forgalomelosztását a 2. rétegbeli switchek kezelik, amelyek megtanulják a switch portjaihoz csatlakoztatott számítógépek MAC-címeit. A switchek ezután a küldő számítógéptől közvetlenül a fogadó eszközhöz küldik a csomagokat. Ez a forgalomirányítási módszer segít csökkenteni a lehetséges hálózati torlódást.

A MAC-cím 12 hexadecimális számból áll. Az első hat szám hasznos a hálózati interfészkártya gyártójának azonosításához. Az utolsó hat számot a kártya gyártója rendeli hozzá, egyedi címzési sémát biztosítva. Ez elkerüli annak lehetőségét, hogy két számítógép ugyanazon a hálózaton ugyanazzal a fizikai címmel rendelkezzen.

### Adatkapcsolati alréteg

A 2. réteg másik funkciója az adatátvitelhez a hálózatra helyezett adatkeretek szerkezetével kapcsolatos protokollok létrehozása. Az egységesség fontos jellemző, és ez magában foglalja mind az adatküldő, mind az adatfogadó címének elhelyezését. Az alábbi 2. ábra egy Ethernet II keret példáját mutatja.

![2. ábra: Ethernet II keret]

Minden keret tartalmazza a keret forrásának MAC-címét és a szánt címzett MAC-címét. Amikor egy eszköz csomagot küld a broadcast MAC-címre (FF:FF:FF:FF:FF:FF), az a helyi hálózat összes állomásához kézbesítésre kerül.

A MAC-címzés jól működött a hálózatok korai napjaiban, amikor csak néhány számítástechnikai eszköz volt egyetlen lapos hálózaton. Emiatt nem volt szükség a hálózat kiterjesztésére a korlátozott fizikai határain túl. A számítástechnikai eszköz nevének a MAC-címéhez kapcsolásához kapcsolódó folyamat, akár táblázat, akár broadcast protokollok, mint a Microsoft NetBEUI-ja révén, lehetővé tette az eszközök hatékony kommunikációját.

Ahogy azonban a hálózatok növekedtek, a hálózati forgalom is növekedett, és a forgalmi torlódás problémássá vált. Ennek következtében szükségessé vált egy második címzési réteg, amelyet két vagy több hálózat összekapcsolásának megkönnyítésére terveztek, hogy kezelje mind a forgalmi torlódást, mind a késleltetési problémákat. Így született meg az adatkapcsolati alréteg.

## A 3. OSI-réteg megértése

Általában az OSI-modell hálózati rétegeként emlegetik, a 3. réteg biztosítja a struktúrát az adatok hatékony átviteléhez egyik hálózatból a másikba. Térjünk vissza a lakópark példájához.

Amikor egy egységet elhelyeznek egy lakóparkban, az általában egy telekre (gyakran „helyként" emlegetik) kerül. A telek címe használható postai célokra. Mint tudod, ha egy csomagot szeretnél postázni az egyik lakóparkból a másikba, a sorozatszámok nem működnek. De ha megfelelően címzed a csomagot, a postai szolgálat által megkövetelt formátumot használva, némi biztosítékunk van a kézbesítésre.

Ami a számítógépes hálózatkezelést illeti, a „postai szolgálatunk" **routerek** néven ismert számítástechnikai eszközökből áll. A 3. rétegben való működéshez egy külső boríték létrehozása szükséges a 2. rétegbeli keret fölé, amely tartalmazza a csomag küldőjének 3. rétegbeli címét és a címzett 3. rétegbeli címét.

A 3. rétegbeli címek oly módon vannak felosztva, hogy azonosítják a konkrét hálózati címet és a konkrét hosztot vagy hosztcsoportot. Amikor megállapítják, hogy a címzett hálózati címe különbözik a küldő hálózati címétől, a csomagokat a hálózat routeréhez irányítják kézbesítéskezelés céljából.

A mai modern hálózatkezelésben a TCP/IP protokollcsomagot használjuk a hálózatokat keresztező kommunikációhoz. A postai szolgálat eszközt nyújt annak megértéséhez, hogyan működik a logikai címzés az IP protokollal.

A fenti logikai cím használatával a postai szolgálat a következők alapján tudná kézbesíteni a csomagot:

![3. ábra]

Az IP (4-es verzió) címzési séma használatával a 172.16.1.12/24 IP-cím esetében a cím így bomlik fel:

![4. ábra]

A logikai hálózatkezeléshez kapcsolódó folyamatok és eljárások, konkrétan a subnetting és a supernetting, meghaladják e cikk hatókörét.

Fontos megérteni, hogy az IP-cím egy része mind a hálózat, mind a hoszt azonosítására van allokálva. Az ugyanazon a hálózaton lévő eszközök hatékonyan tudnak kommunikálni a 2. rétegben. A különböző hálózatokon lévő eszközök úgy kommunikálnak, hogy kimeneti adataikat a routerekhez irányítják, amelyek a kézbesítés felelősségét kapják.
