# CCNA Bővített Jegyzet – 23. Lecke: Az STP Eszköztár (Toolkit): Root Guard

Ebben a leckében folytatjuk a Spanning Tree Protocol (STP) "eszköztárának" (Toolkit) felfedezését. Az előző részekben megismertük a PortFast-ot (gyorsítás végpontoknál), valamint a BPDU Guardot és a BPDU Filtert (védelem és szűrés a végpontoknál). Most egy olyan funkciót vizsgálunk meg, amely a hálózat magját (core), pontosabban a Gyökérhíd (Root Bridge) pozícióját védi az illetéktelen átvételtől: ez a **Root Guard**.

## 1. A Probléma: A Root Bridge Pozíciójának Veszélye

A Spanning Tree hálózatban a Gyökérhíd (Root Bridge) a hálózat központja. Az STP algoritmus az összes többi port állapotát (Root, Designated, Blocking) a Gyökérhídhoz viszonyítva határozza meg, hogy kiküszöbölje a hurkokat.

### Miért fontos, hogy ki a Gyökérhíd?

- **Optimális Forgalomirányítás (Traffic Flow):** A hálózati forgalom mindig a Root Bridge felé áramlik. Ha a Root Bridge egy lassú, régi switch a hálózat szélén, az összes adatnak oda kell utaznia (növekszik a késleltetés/latency) és ott kell áthaladnia (torlódás/congestion), mielőtt kijutna az internetre (pl. egy routeren keresztül).
- **Stabilitás:** A Root Bridge-nek a hálózat legmegbízhatóbb, legnagyobb teljesítményű (core/elosztó) switchének kell lennie.

A saját hálózatodon belül te irányítasz: beállítod a legjobb switched prioritását a legalacsonyabb értékre (pl. 0 vagy 4096), így az biztosan Gyökérhíd lesz.

### De mi van akkor, ha a hálózatodat összekötöd egy másik hálózattal, amit NEM te irányítasz?

**Példa:** Te egy internetszolgáltató (Service Provider) vagy, aki Metro Etherneten keresztül összeköt két ügyfelet. A te hálózatodban a Root Bridge az SW1 (Prioritás: 1). Az egyik ügyfél (a Customer) a saját hálózatában szintén beállít egy switchet (SW6) Root Bridge-nek (Prioritás: 1).

Amikor a két hálózat összekapcsolódik, a switchek elkezdik cserélgetni a BPDU-kat. Mivel a prioritások megegyeznek (mindkettő 1), a MAC-cím fog dönteni. Ha az ügyfél SW6 switchének történetesen alacsonyabb a MAC-címe, mint a te SW1 switchednek, a te hálózatod (a szolgáltatói hálózat) is az SW6-ot fogja kikiáltani az egész, egyesített hálózat Gyökérhídjává!

**A következmény:** A szolgáltatói hálózat STP topológiája teljesen átalakul, a fővonalak leblokkolhatnak, és a szolgáltatói forgalom az ügyfél lassú hálózatán keresztül kezdhet el áramlani. Ez katasztrófa.

## 2. A Megoldás: Root Guard

A Root Guard funkció pontosan ezt a problémát akadályozza meg. Lehetővé teszi, hogy egy adott porton "kényszerítsük" az STP-t: "Ezen a porton soha, semmilyen körülmények között nem fogadunk el olyan BPDU-t, ami jobb (Superior), mint a miénk!"

### Hogyan működik a Root Guard?

A Root Guard-ot azokra a portokra konfiguráljuk, amelyek olyan switchekhez csatlakoznak, amelyeket nem mi felügyelünk (pl. a szolgáltató ráteszi az ügyfelek felé néző portokra).

- A port továbbra is küld és fogad BPDU-kat.
- Amíg normál (Inferior, azaz rosszabb vagy egyenlő) BPDU-kat kap, a port normálisan továbbít.
- Ha azonban egy "Superior" (Jobb, tehát alacsonyabb Bridge ID-vel rendelkező) BPDU érkezik a portra (mert a túloldal Root Bridge akar lenni), a Root Guard azonnal letiltja a portot!

### A Letiltott Állapot: "Broken" és "Root Inconsistent"

Amikor a Root Guard lecsap egy porton:

- A port fizikai állapota nem down, hanem az STP állapota **Broken** (Törött/Hibás) lesz.
- A port konkrétan a **Root Inconsistent** (Gyökér Inkonzisztens) állapotba kerül.
- Ebben az állapotban a port le van blokkolva: nem továbbít és nem fogad semmilyen normál hálózati adatot (a kommunikáció teljesen megszakad az ügyfél és a szolgáltató között).

### A Helyreállás (Recovery)

A BPDU Guard-dal ellentétben (ami err-disable állapotba teszi a portot, és manuális vagy időzített beavatkozást igényel), a **Root Guard automatikusan helyreáll!**

Ahhoz, hogy a port újraéledjen, meg kell szüntetni a kiváltó okot. Vagyis a túloldali (ügyfél) switch tulajdonosának meg kell változtatnia a beállításait (pl. feljebb venni a prioritását 4096-ra), hogy a switche már ne küldjön "Superior" BPDU-t.

Amint a régi (superior) BPDU-k kiöregszenek a mi switchünkből (ez alapértelmezetten a Max Age időzítő, azaz 20 másodperc), és csak az új (inferior) BPDU-kat kapjuk, a port automatikusan visszavált Forwarding állapotba, és a forgalom helyreáll.

## 3. A Root Guard Konfigurálása (Cisco CLI)

A Root Guard konfigurálása rendkívül egyszerű, de van egy fontos különbség a korábban tanultakhoz képest:

A PortFast, a BPDU Guard és a BPDU Filter rendelkezett egy "globális" (default) paranccsal, amivel az összes Access portra egyszerre rá lehetett tenni őket. **A Root Guardnak NINCS globális parancsa!** Csak és kizárólag interfész szinten (per-port) lehet beállítani, hiszen nagyon pontosan meg kell válogatni, mely trunk vagy hozzáférési portokon akarjuk érvényesíteni ezt a korlátozást.

A parancs (Interfész konfigurációs módban):

```text
Switch(config)# interface g0/2
Switch(config-if)# spanning-tree guard root
```

Ha letilt a port, a konzolon egy ilyen üzenet jelenik meg:
`%SPANTREE-2-ROOTGUARDBLOCK: Port 0/2 tried to become non-designated in VLAN 1. Moved to root-inconsistent state`

Az ellenőrzés a `show spanning-tree` paranccsal történik:
A port státuszánál (Sts) a **BKN** (Broken) rövidítés látszik, a szerepénél (Role) pedig a **Desg** (Designated), de mellette a **ROOT_Inc** felirat mutatja, hogy inkonzisztens állapotban van.

## 4. Összegzés: Mikor (és mikor NEM) használjuk a Root Guardot?

- **Használd:** Ha te vagy a hálózat (pl. szolgáltató) üzemeltetője, és a saját hálózatod határán (edge) lévő portokon, amik külső/idegen switchekhez csatlakoznak, biztosítani akarod, hogy a saját Root Bridge-ed maradjon a főnök.
- **NE használd:** Az ügyfél (Customer) oldalán! Ha az ügyfél bekapcsolja a Root Guardot a szolgáltató felé néző portján, akkor a portja azonnal le fog tiltani, amint a szolgáltató "jobb" BPDU-ja megérkezik (hiszen a szolgáltatónak kell, hogy legyen a Root Bridge-e). Ezzel az ügyfél saját magát vágná el a hálózattól.
