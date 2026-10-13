# 5. Lecke: Ethernet LAN Switching (1. Rész)

Ez a lecke részletesen bemutatja, hogyan utaznak az adatok egy LAN-on (helyi hálózaton) belül a számítógépek és a switchek között. A cél az adatok mozgásának, a fizikai címzésnek és a switchek döntéshozatali logikájának megértése.

## 1. Alapfogalmak és az OSI Modell Ismétlése

A hálózati kommunikációt rétegekre bontva vizsgáljuk:

**Layer 1 (Fizikai réteg):** Ez a réteg felel az adatok fizikai továbbításáért. A digitális biteket (0-k és 1-esek) elektromos jelekké (rézkábelen) vagy rádióhullámokká (Wi-Fi) alakítja. Ide tartoznak a feszültségszintek, a kábelek (pl. UTP), a csatlakozók (RJ45) és a maximális távolságok.

**Layer 2 (Adatkapcsolati réteg):** Ez a réteg felel a csomópontok közötti (például PC-től a switchig történő) adatátvitelért. A 2. réteg logikai címek (IP-cím) helyett fizikai címeket (MAC-cím) használ. A switchek ezen a rétegen működnek.

**LAN (Local Area Network):** Egy viszonylag kis fizikai területet (pl. egy irodát) lefedő hálózat.

A routerek feladata a különálló LAN-ok összekötése.

A switchek nem választják szét a hálózatot, hanem kibővítik azt. Egy router adott portjára kötött összes eszköz (akár több switchen keresztül is) egyetlen hatalmas, közös LAN-t alkot.

## 2. Az Adatbecsomagolás (Encapsulation) és a PDU-k

Ahogy az adat elindul a küldő géptől lefelé a hálózati rétegeken, minden réteg hozzáteszi a saját fejlécét (olyan ez, mint amikor egy levelet egyre nagyobb borítékokba teszünk). Ezeket az állapotokat **PDU**-nak (Protocol Data Unit) hívjuk:

- **Data (Adat):** A felső rétegek információja.
- **Segment:** Az adat + Layer 4 fejléc.
- **Packet (Csomag):** A szegmens + Layer 3 (IP) fejléc.
- **Frame (Keret):** A csomag + Layer 2 fejléc és utótag. Ebben a leckében az Ethernet keretekkel foglalkozunk.

## 3. Az Ethernet Keret (Frame) Pontos Felépítése

A Layer 2-ben a csomag elé egy fejléc (header), a végére pedig egy utótag (trailer) kerül. Ez összesen 26 bájt extra információ.

### Az Ethernet Fejléc (Header) 5 mezője

| Mező | Méret | Leírás |
|---|---|---|
| **Preamble (Preambulum)** | 7 bájt | Váltakozó 1-esek és 0-k sorozata (10101010). Ez egy "ébresztő" jel: lehetővé teszi a fogadó eszköz vevőórájának szinkronizálását (receiver clock synchronization), hogy felkészüljön a tényleges adatok fogadására. |
| **SFD (Start Frame Delimiter)** | 1 bájt | A preambulumhoz hasonló, de az utolsó bitje 1-es (10101011). Ez jelzi pontosan, hogy hol ér véget a "figyelemfelhívás", és hol kezdődik az érdemi adat. |
| **Destination (Cél) MAC-cím** | 6 bájt | Annak az eszköznek a fizikai címe, akinek a csomag szól. |
| **Source (Forrás) MAC-cím** | 6 bájt | A küldő eszköz fizikai címe. |
| **Type / Length (Típus / Hossz)** | 2 bájt | Ha az értéke 1500 vagy annál kisebb, a keretbe ágyazott csomag pontos méretét mutatja bájtokban. Ha 1536 vagy nagyobb, a beágyazott csomag típusát (Layer 3 protokoll) jelzi. Például a `0x0800` érték (a `0x` a hexadecimálist jelöli) az IPv4-et, a `0x86DD` pedig az IPv6-ot azonosítja. |

### Az Ethernet Utótag (Trailer) 1 mezője

| Mező | Méret | Leírás |
|---|---|---|
| **FCS (Frame Check Sequence)** | 4 bájt | Hibajavító és ellenőrző mező. A fogadó eszköz egy CRC (Cyclic Redundancy Check) nevű matematikai algoritmust futtat le a megérkezett adaton. A "redundancia" arra utal, hogy ez a 4 bájt új információt nem ad hozzá, csak megnöveli az üzenetet. A célja, hogy ellenőrizze, nem sérült-e meg az adat a kábelen. Ha hibát talál, a keretet eldobja. |

## 4. MAC-címek és a Hexadecimális Számolás

A MAC-cím a hálózati kártya 48 bites (6 bájtos) egyedi azonosítója (Burned-in Address), amelyet a gyártáskor kap.

- **OUI (Organizationally Unique Identifier):** A MAC-cím első fele (24 bit). Ez a gyártót (pl. Cisco, Dell, Apple) azonosítja.
- **Eszközazonosító:** A MAC-cím második fele (24 bit), amely garantálja, hogy az eszköz egyedi legyen.

### Hogyan működik a Hexadecimális (16-os) számrendszer?

A nagyon hosszú bináris kódok egyszerűsítésére a MAC-címeket 12 hexadecimális karakterrel írjuk (pl. `aaaa.aa00.0001`).

A decimális (10-es) rendszerben 10 számjegy van (0-9). Ha elfogy, nyitunk egy tízesek oszlopot (a 10 jelentése: 1 darab tízes, 0 darab egyes).

A hexadecimális rendszerben 16 számjegy van: 0-9-ig számok, majd A-tól F-ig betűk (A=10, B=11, C=12, D=13, E=14, F=15).

> **Példa laikusoknak:** Ha hexadecimálisan azt látod, hogy `10`, az nem tízet jelent! A második oszlop itt a tizenhatosok oszlopa. Tehát a `10` jelentése: 1 darab tizenhatos és 0 darab egyes (azaz decimálisan 16). A `1A` jelentése: 1 darab tizenhatos és 10 darab egyes (azaz decimálisan 26).

## 5. Hogyan gondolkodik a Switch? (Részletes működési logika)

A switchek okos elosztók: a céljuk, hogy az üzenetek csak oda jussanak el, ahova kell. Ezt egy belső "notesz", a **MAC Address Table** (MAC-cím tábla) segítségével érik el, amelyet folyamatosan és dinamikusan építenek fel. (A Cisco switchek 5 perc inaktivitás után automatikusan kitörlik az elévült bejegyzéseket).

Két alapvető lépésből áll a folyamat:

### 1. Lépés: Tanulás (A Forrás cím alapján)

Amikor a switch egy adatkeretet kap az egyik portján, mindig megnézi a **Source (Forrás) MAC-címet**.

Ezt a címet és a bejövő portot azonnal feljegyzi a MAC-cím táblájába: "Most már tudom, hogy az 'A' gép az 1-es porton keresztül érhető el."

### 2. Lépés: Döntéshozatal (A Cél cím alapján)

Ezután a switch megnézi a **Destination (Cél) MAC-címet**, hogy hova kell kézbesíteni a keretet.

**Szituáció A (Ismeretlen cél):** Ha a célállomás még nincs benne a noteszben (mert még sosem küldött adatot), ezt **Unknown unicast** keretnek hívják. Ilyenkor a switch "vakon" dolgozik: lemásolja a keretet, és elárasztja (**flooding**) az összes aktív portján (kivéve azon, ahonnan kapta). Az az egy gép, akinek szólt, feldolgozza, a többiek eldobják.

**Szituáció B (Ismert cél):** Ha a célállomás már szerepel a noteszben, a keretet **Known unicast**-nak nevezzük. Ebben az esetben a switch célzottan továbbítja (**forwarding**) a keretet, és kizárólag azon az egyetlen porton küldi ki, ahol a fogadó fél tartózkodik. Nem zavarja vele a hálózat többi részét.

### Mi történik több switch esetén? (A két switch példa)

Ha PC1 küldeni akar PC3-nak, de közöttük két switch (SW1 és SW2) van összekötve:

1. PC1 elküldi a keretet SW1-nek. SW1 megtanulja PC1-et, de mivel nem ismeri PC3-at, elárasztja a hálózatot. Az egyik másolat átmegy SW2-höz.
2. SW2 megkapja a keretet SW1 felől. SW2 megnézi a forrást, és ő is megtanulja PC1 MAC-címét, és hozzárendeli ahhoz a portjához, ami SW1 felé néz. (Ez nagyon fontos: a switch azt az irányt, azt az interfészt jegyzi meg, amerre a gép található, függetlenül attól, hogy a gép fizikailag be van-e dugva abba az adott portba, vagy egy másik switchen keresztül jön).
3. SW2 sem ismeri még PC3-at, ezért ő is elárasztást csinál. PC3 végre megkapja.
4. Amikor PC3 válaszol PC1-nek, a visszafelé úton a switchek megtanulják PC3-at is. Mivel PC1 már benne van a tábláikban, a választ már nem árasztják el, hanem pontosan, célzottan (forwarding) küldik vissza.

## 6. Gyakorló Kvíz és Magyarázatok

**1. Az Ethernet keret melyik mezője biztosítja a vevőóra szinkronizálását (receiver clock synchronization)?**

- A) Preamble (Preambulum)
- B) SFD
- C) Type
- D) FCS

> **Helyes válasz:** A
>
> **Magyarázat:** A preambulum a legelső, 7 bájtos jelsorozat (1-esek és 0-k), amely felkészíti a gépet az adat fogadására. Az SFD csak ennek végét jelzi, a Type a belső protokoll típusát mutatja, az FCS pedig a hibák detektálására való.

**2. Milyen hosszú egy hálózati eszköz fizikai címe (MAC-cím)?**

- A) 32 bájt
- B) 32 bit
- C) 48 bájt
- D) 48 bit

> **Helyes válasz:** D
>
> **Magyarázat:** A MAC-cím pontosan 48 bitből áll (amely egyenlő 6 bájttal). Érdekességként: a későbbi leckékben szereplő IP-cím 32 bit hosszú lesz.

**3. Mi az OUI-ja ennek a MAC-címnek: E8BA.7011.2874?**

- A) E8Ba
- B) E8BA.70
- C) 7011
- D) E8BA.7011

> **Helyes válasz:** B
>
> **Magyarázat:** Az OUI (amely a gyártót azonosítja) minden esetben a MAC-cím pontosan első felét, azaz az első 24 bitet (6 hexadecimális karaktert) jelenti.

**4. Az Ethernet keret melyik mezőjét használja a switch a MAC-cím táblájának feltöltésére (a tanulásra)?**

- A) Preamble
- B) Length
- C) Source (Forrás) MAC Address
- D) Destination (Cél) MAC Address

> **Helyes válasz:** C
>
> **Magyarázat:** A switch abból tudja megjegyezni, hogy ki hol található, hogy mindig megnézi a beérkező csomag Forrás MAC-címét, majd párosítja azzal a fizikai porttal, ahol a kábel be van dugva. A cél címet kizárólag a továbbítási irány meghatározására használja.

**5. Milyen típusú keretet áraszt el (flood) a switch az összes interfészén, kivéve azt, amelyiken megkapta?**

- A) Unknown unicast (Ismeretlen unicast)
- B) Known unicast (Ismert unicast)
- C) Allcast

> **Helyes válasz:** A
>
> **Magyarázat:** Ha a switch táblázata még üres, vagy nem ismeri az adott célállomást, az keret egy "ismeretlen unicast". Ilyenkor nem tudja a pontos útirányt, ezért biztonsági okokból mindenfelé kiküldi, nehogy elvesszen az adat. Az "allcast" kifejezés nem létezik a hálózatokban.
