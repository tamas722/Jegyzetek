# CCNA Bővített Jegyzet – 24. Lecke: Az STP Eszköztár (Toolkit): Loop Guard

Ebben a leckében folytatjuk és egyben lezárjuk a Spanning Tree Protocol (STP) "eszköztárának" (Toolkit) bemutatását. Miután megismertük a PortFast-ot, a BPDU Guard-ot, a BPDU Filter-t és a Root Guard-ot, egy utolsó, nagyon fontos védelmi funkciót veszünk szemügyre: ez a **Loop Guard** (Hurokvédő). A Loop Guard a hálózat fizikai rétegének (Layer 1) rejtett hibáiból – kifejezetten az egyirányú kábelhibákból – adódó katasztrófákat előzi meg.

## 1. A Probléma: Az Egyirányú Kábelhibák (Unidirectional Links)

Hogy megértsük a Loop Guard szükségességét, először meg kell ismernünk a problémát, amit megold. A hagyományos UTP (réz) kábelekkel ellentétben a száloptikás (fiber-optic) kábelek valójában két különálló üvegszálból állnak: az egyik szálon csak küld az eszköz (Tx - Transmit), a másikon csak fogad (Rx - Receive).

Normál esetben, ha elvágod a kábelt, a switch érzékeli, hogy megszakadt a kapcsolat, és azonnal leállítja a portot (Status: `down/down`). Nincs kapcsolat, nincs hurok.

### De mi történik, ha csak a kábel EGYIK fele törik meg?

(Vagy sérül a csatlakozó/SFP modul), de a switchek ezt valamiért nem érzékelik? Ilyenkor létrejön egy **Unidirectional Link** (Egyirányú kapcsolat).

- Az R2 switch tud küldeni adatot az R3-nak (mert az a szál jó).
- Az R3 hiába küldene adatot az R2-nek, az a szál eltört, így a jel sosem érkezik meg.
- A portok státusza mégis `up/up` marad, mert a switchek nem vették észre a fizikai hibát.

### Hogyan okoz ez Layer 2 Hurkot (STP Loop)?

Tegyük fel, hogy az R3 portja a normál működés szerint egy **Blokkoló (Blocking)** port. A blokkoló portok azért vannak lezárva, mert folyamatosan kapják a "felsőbbrendű" BPDU-kat a túloldalról (R2-ről).

1. Amikor a kábel egyirányúvá válik (és R2 jelei nem érnek át R3-hoz), az R3 nem kap többé BPDU-kat.
2. A **Max Age Timer** (20 másodperc) lejár.
3. Az R3 azt hiszi: "Már nem kapok BPDU-t, biztosan megszakadt a hurok, tehát kinyithatom a portomat!"
4. Az R3 portja Blokkolóból átvált **Designated (Továbbító)** porttá.
5. **Eredmény:** Mind az R2, mind az R3 portja Forwarding (Nyitott) állapotba kerül! Az R3 elkezdi bepumpálni a forgalmat az R2-be a működő szálon. Kialakult a **Layer 2 Hurok (Loop)**, és a hálózat percek alatt összeomlik egy Broadcast Storm miatt.

## 2. A Megoldás: Loop Guard (Hurokvédő)

A Loop Guard pontosan erre a szituációra jelent megoldást. A működési elve rendkívül egyszerű és logikus.

### Hogyan működik?

A Loop Guard-ot a **Root portokon** és a **Blokkoló (Non-designated) portokon** kell bekapcsolni (azokon a portokon, ahol a switch elvárja, hogy BPDU-kat kapjon egy másik switcstől).

- Amíg a port kapja a BPDU-kat, minden normálisan működik.
- Ha a port hirtelen, váratlanul nem kap több BPDU-t (és a Max Age timer lejár), a Loop Guard **MEGAKADÁLYOZZA**, hogy a port Designated (Nyitott/Továbbító) állapotba váltson!
- Ehelyett a port azonnal egy **Broken** (Törött/Hibás) állapotba kerül.

### A Letiltott Állapot: "Broken" és "Loop Inconsistent"

Amikor a Loop Guard lecsap:

- A konzolon megjelenik az üzenet: `%SPANTREE-2-LOOPGUARD_BLOCK: Loop guard blocking port GigabitEthernet0/1`
- A `show spanning-tree` kimenetben a port állapota (Sts) **BKN** (Broken) lesz.
- A szerepe mellett megjelenik a **LOOP_Inc** (Loop Inconsistent - Hurok inkonzisztens) jelzés.
- Ebben az állapotban a port teljesen le van fojtva: nem enged át semmilyen adatot, így a hurok kialakulása lehetetlen.

### A Helyreállás (Recovery)

A Root Guardhoz hasonlóan a Loop Guard is **automatikusan helyreáll** (Automatic Recovery)! Nincs szükség manuális `shutdown` / `no shutdown` parancsokra, sem az ErrDisable Recovery beállítására.

Amint a kábelt megjavítják (vagy az egyirányúság megszűnik), és a port újra elkezd BPDU-kat kapni, a Loop Guard azonnal leveszi a blokkolást, és a port visszatér a normál STP működéshez.

## 3. A Loop Guard Konfigurálása (Cisco CLI)

Kétféleképpen állíthatjuk be a switcheken:

### 1. Interfész Szinten (Per-Port)

Lépj be az adott interfész konfigurációs módjába, és add ki a parancsot:

```text
Switch(config-if)# spanning-tree guard loop
```

(Ha egy porton ki akarod kapcsolni, a `spanning-tree guard none` parancsot kell használnod).

### 2. Globálisan (Alapértelmezettként)

Ezzel a paranccsal a Loop Guard a switch összes portján aktiválódik alapértelmezettként (természetesen csak ott fog ténylegesen "őrködni", ahol az STP szabályai szerint értelme van, tehát a blokkoló és root portokon).

```text
Switch(config)# spanning-tree loopguard default
```

> (Tipp: Nagyobb hálózatokban a Cisco kifejezetten ajánlja a Loop Guard globális bekapcsolását).

## 4. Összegzés és egy Fontos Szabály: Loop Guard vs. Root Guard

A Loop Guard és a Root Guard a Spanning Tree két véglete, teljesen ellentétes dolgoktól védik a hálózatot. Emiatt a legfontosabb szabály: **A két funkciót SOHA nem lehet (és nem is szabad) ugyanazon a porton egyszerre bekapcsolni!** (Mutually Exclusive).

### Root Guard

- **Hol használjuk?** Ahol **NEM** akarunk "jobb" (Superior) BPDU-t látni. (Pl. ügyfelek, idegen hálózatok felé néző, normál esetben Designated portokon).
- **Mit csinál?** Letiltja a portot, ha kap egy BPDU-t, ami jobb, mint a miénk (**Root Inconsistent**).

### Loop Guard

- **Hol használjuk?** Ahol **ELVÁRJUK**, hogy folyamatosan BPDU-kat kapjunk. (Tehát a Root portokon és a Blokkoló portokon).
- **Mit csinál?** Letiltja a portot, ha hirtelen **NEM KAP** több BPDU-t (**Loop Inconsistent**).

> Ha ugyanarra a portra véletlenül mindkét parancsot megpróbálod kiadni (akár globális, akár port szinten keverve), a Cisco IOS szabálya szerint mindig a később kiadott, vagy a specifikusabb (interfész szintű) parancs fog érvényesülni, és az felülírja (kikapcsolja) a másikat! (Pl. hiába van globálisan bekapcsolva a Loop Guard, ha egy portra ráteszed a `guard root` parancsot, azon a porton a Loop Guard ki fog kapcsolni).
