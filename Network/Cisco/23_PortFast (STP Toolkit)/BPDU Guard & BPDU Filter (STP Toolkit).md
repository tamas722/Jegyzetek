# CCNA Bővített Jegyzet – 22. Lecke: Az STP Eszköztár (Toolkit): BPDU Guard és BPDU Filter

Ebben a leckében folytatjuk a Spanning Tree Protokoll (STP) "eszköztárának" megismerését. Az előző részben bemutatott PortFast felgyorsította a végpontok csatlakozását, most pedig megnézzük azt a két funkciót – a **BPDU Guard**-ot és a **BPDU Filter**-t –, amelyek biztonságossá teszik és finomhangolják a PortFast működését.

## 1. A Probléma: A PortFast és az Illetéktelen Switchek (Bob esete)

Bár a PortFast azonnal továbbító (Forwarding) állapotba tesz egy portot, **nem kapcsolja ki rajta az STP-t!** Ez azt jelenti, hogy a switchport továbbra is 2 másodpercenként küldi kifelé az STP BPDU üzeneteket. Egy átlagos PC (mivel nem futtat STP-t) ezeket egyszerűen ignorálja.

### De mi történik, ha egy felhasználó hibázik?

Képzeljük el, hogy Bob az irodában megunja a lassú Wi-Fi-t, és behoz otthonról egy saját kis switchet, amit rádug a fali aljzatra (amelyen a PortFast aktív). Bob rádugja a gépeit a switchére, és örül, hogy van internete.

**A hálózati katasztrófa:** Bob switche elkezdi küldeni a saját BPDU üzeneteit a céges hálózat felé!

- Amint a PortFast-tal konfigurált port megkapja Bob BPDU-ját, azonnal elveszíti a PortFast státuszát, és visszavált egy normál, hagyományos STP porttá.
- Ha Bob otthoni switchének véletlenül alacsonyabb a Bridge ID-je (pl. régi switch kisebb MAC-címmel vagy alacsony prioritással), **Bob switche válhat a hálózat Gyökérhídjává (Root Bridge)!**
- Emiatt az egész céges hálózat forgalma elkezdhet Bob irodája felé áramlani, a hálózati utak átrendeződnek, és a teljesítmény drasztikusan leromlik.

Ezt a rémálmot akadályozza meg a **BPDU Guard**!

## 2. A Megoldás: BPDU Guard (BPDU Őr)

A BPDU Guard egy védelmi vonal. A feladata, hogy megvédje a végpontoknak (PC-k, nyomtatók) fenntartott portokat attól, hogy illetéktelen switcheket csatlakoztassanak rájuk.

### Hogyan működik?

Ha a BPDU Guard engedélyezve van egy porton, a port továbbra is küld BPDU-kat kifelé. Azonban, ha akár egyetlenegy BPDU-t is kap (vagyis egy másik switchet érzékel), a port azonnal letiltásra kerül, és **"Error-disabled" (err-disable)** állapotba lép!

A port onnantól kezdve halott: sem adatot, sem BPDU-t nem küld és nem fogad. Bobnak elmegy az internete, és kénytelen lesz hívni az IT-t.

### BPDU Guard Konfigurálása (Cisco CLI)

Kétféleképpen állíthatjuk be, akárcsak a PortFast-ot:

#### 1. Interfész szinten (Per-port)

Közvetlenül az adott porton kapcsoljuk be:

```text
Switch(config-if)# spanning-tree bpduguard enable
```

#### 2. Globálisan (Alapértelmezettként)

Ezzel a paranccsal a BPDU Guard automatikusan aktiválódik az összes olyan porton, ahol a PortFast engedélyezve van. Ez a legbiztonságosabb és legkényelmesebb módszer!

```text
Switch(config)# spanning-tree portfast bpduguard default
```

(Ha egy adott porton a globális beállítás ellenére mégis ki akarod kapcsolni a védelmet, az interfész alatt kiadhatod a `spanning-tree bpduguard disable` parancsot).

## 3. Az ErrDisable Állapot és a Helyreállítás

Ha a BPDU Guard lecsap, a port **err-disable** állapotba kerül. A switch konzolján hibaüzenetek jelennek meg, a port státusza `down/down` lesz, a fizikai LED pedig folyamatos narancssárga fénnyel világít (még akkor is, ha Bob közben ijedtében kihúzza a kábelt!).

> **A legfontosabb szabály:** Mielőtt újraélesztenéd a portot, meg kell szüntetni a kiváltó okot (Bob switchét le kell kötni)! Ha ezt nem teszed meg, a port a bekapcsolás utáni első másodpercben újra le fog tiltani.

### Hogyan lehet újraéleszteni a portot? Két módszer létezik:

#### 1. Manuális Helyreállítás

A hálózati mérnök belép a port konfigurációjába, lekapcsolja, majd visszakapcsolja azt.

```text
Switch(config-if)# shutdown
Switch(config-if)# no shutdown
```

#### 2. Automatikus Helyreállítás (ErrDisable Recovery)

Beállíthatjuk a switchet, hogy egy bizonyos idő után automatikusan próbálja meg újraéleszteni a letiltott portokat.

- Az állapot ellenőrzése: `show errdisable recovery` (Alapból minden funkció le van tiltva, az időzítő pedig 300 másodperc / 5 perc).
- Az időzítő módosítása (opcionális): `errdisable recovery interval <másodperc>`
- Az automatikus helyreállítás bekapcsolása a BPDU Guard esetére:

```text
Switch(config)# errdisable recovery cause bpduguard
```

Ezután, ha egy port letilt, a switch 5 perc múlva automatikusan megpróbálja visszakapcsolni (ha Bob még mindig ott van a switch-csel, a port újra le fog tiltani).

## 4. BPDU Filter (BPDU Szűrő)

Míg a BPDU Guard a bejövő BPDU-kra reagál, a BPDU Filter a kimenő BPDU-kat is kezeli.

**A Probléma:** Egy PC-nek felesleges 2 másodpercenként BPDU üzeneteket küldözgetni. Ez minimálisan ugyan, de pazarolja a sávszélességet és a switch processzorát, ráadásul biztonsági kockázatot is jelent (kiszivárogtatja a hálózat STP topológiájának adatait egy esetleges támadó felé).

**A Megoldás:** A BPDU Filter megakadályozza, hogy a port BPDU-kat küldjön kifelé.

### A BPDU Filter KÉT ARCA (Kritikus vizsgatéma!)

A BPDU Filter viselkedése drágán függ attól, hogyan konfigurálod! Ez egy hatalmas csapda lehet.

#### 1. Veszélyes módszer: Interfész szinten (Per-port)

```text
Switch(config-if)# spanning-tree bpdufilter enable
```

**Mit csinál?** Nem küld BPDU-t kifelé. De ami rosszabb: **teljesen ignorálja a bejövő BPDU-kat is!**

> **A Veszély:** Ez gyakorlatilag **KIKAPCSOLJA az STP-t** az adott porton! Ha ide bedugnak egy switchet, a port figyelmen kívül hagyja, és azonnali, végleges Layer 2 hurkot okozhat, ami megbénítja a hálózatot. Ezt a módszert szinte soha nem javasolt használni!

#### 2. Biztonságos módszer: Globálisan (Alapértelmezettként)

```text
Switch(config)# spanning-tree portfast bpdufilter default
```

**Mit csinál?** Aktiválódik az összes PortFast porton. A port nem küld BPDU-t kifelé.

> **A Varázslat:** Ha a port mégis kap egy BPDU-t (mert Bob rádugta a switchét), a BPDU Filter és a PortFast azonnal kikapcsol magától, és a port visszavált egy normál, hagyományos STP porttá! Nem ignorálja a veszélyt, hanem lereagálja. Ez a "két világ legjobbja".

## 5. Összegzés: Mi történik, ha a Guard és a Filter is be van kapcsolva?

Gyakori kérdés, hogy mi történik, ha egy porton mind a BPDU Guard, mind a BPDU Filter aktív, és bejön egy BPDU (egy másik switchről). A válasz ismét a konfiguráció módjától függ:

- **Ha a Filter GLOBÁLISAN van beállítva (Jó eset):** A port megkapja a BPDU-t. A globális Filter kikapcsol, és a port normál STP porttá akarna válni, DE a BPDU Guard ekkor azonnal közbelép, és letiltja a portot (err-disable). Ez a tökéletes védelem.
- **Ha a Filter INTERFÉSZ SZINTEN van beállítva (Rossz eset):** A port megkapja a BPDU-t, de mivel az interfész szintű Filter teljesen ignorálja (eldobja) azt, a BPDU sosem jut el a Guard-ig feldolgozásra! A BPDU Guard **NEM** fog aktiválódni, a port nyitva marad, és kialakulhat a hurok.
