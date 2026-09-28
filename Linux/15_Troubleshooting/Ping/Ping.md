# A `ping` parancs Linuxon

**Szerző:** Dejan Panovski
**Frissítve:** 2026. április 22.
**6 perc olvasás**

## Ezen az oldalon

- A `ping` parancs használata
- Csomagok számának megadása
- Forrásinterfész megadása
- Internetprotokoll megadása
- Intervallum és időtúllépés módosítása
- Csomagméret módosítása
- Csendes kimenet
- Gyors referencia
- Következtetés

A `ping` parancs az egyik leggyakrabban használt eszköz hálózati kapcsolódási problémák hibaelhárítására, tesztelésére és diagnosztizálására.

A `ping` úgy működik, hogy egy vagy több ICMP (Internet Control Message Protocol) Echo Request csomagot küld egy megadott cél-IP-re a hálózaton, és vár a válaszra. Amikor a cél megkapja a csomagot, egy ICMP echo reply-val válaszol.

A `ping` paranccsal megállapíthatod, hogy egy távoli cél-IP aktív-e vagy inaktív. Megtudhatod a céllal való kommunikáció oda-vissza késleltetését is, és ellenőrizheted, van-e csomagvesztés.

A `ping` az `iputils` (vagy `iputils-ping`) csomag része, amely szinte minden Linux disztribúción előre telepítve van. Elérhető Windows-on, macOS-en és FreeBSD-n is.

## A `ping` parancs használata

A `ping` parancs szintaxisa a következő:

```bash
ping [OPTIONS] DESTINATION
```

Hogy jobban szemléltessük, hogyan működik a `ping` parancs, pingeljük a `google.com`-ot:

```bash
ping google.com
```

A kimenet valahogy így fog kinézni:

```plaintext
PING google.com (172.217.22.206) 56(84) bytes of data.
64 bytes from muc11s01-in-f14.1e100.net (172.217.22.206): icmp_seq=1 ttl=53 time=40.2 ms
64 bytes from muc11s01-in-f14.1e100.net (172.217.22.206): icmp_seq=2 ttl=53 time=41.8 ms
64 bytes from muc11s01-in-f14.1e100.net (172.217.22.206): icmp_seq=3 ttl=53 time=47.4 ms
64 bytes from muc11s01-in-f14.1e100.net (172.217.22.206): icmp_seq=4 ttl=53 time=41.4 ms
^C
--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 40.163/42.700/47.408/2.790 ms
```

A `ping` parancs feloldja a domainnevet egy IP-címre, és elkezd ICMP-csomagokat küldeni a cél-IP-re. Ha a cél-IP elérhető, válaszol, és a `ping` parancs kiír egy sort, amely a következő mezőket tartalmazza:

- **Az adatbájtok száma.** Az alapértelmezett 56, ami 64 ICMP adatbájtra fordul le – `64 bytes`.
- **A cél IP-címe** – `from muc11s01-in-f14.1e100.net (172.217.22.206)`.
- **Az ICMP sorszám** minden csomaghoz – `icmp_seq=1`.
- **Az élettartam (Time to Live)** – `ttl=53` – Hogyan működik a TTL?.
- **A ping idő**, milliszekundumban mérve, ami a csomag oda-vissza útjának ideje a hoszt eléréséhez és a válasz visszatéréséhez a küldőhöz – `time=41.4 ms`.

Alapértelmezésben az új csomag küldése közötti intervallum egy másodperc.

A `ping` parancs addig folytatja az ICMP-csomagok küldését a cél-IP-címre, amíg megszakítási jelet nem kap. A parancs leállításához egyszerűen nyomd meg a `Ctrl+C` billentyűkombinációt.

Amikor a parancs leáll, statisztikákat jelenít meg, beleértve a csomagvesztés százalékát. A csomagvesztés azt jelenti, hogy az adat valahol a hálózatban elveszett, ami hálózati problémát jelez. Ha van csomagvesztés, a `traceroute` paranccsal azonosíthatod, hol történik a csomagvesztés. Csomagszintű rögzítéshez használd a `tcpdump`-ot.

Ha a `ping` nem kap választ, az azt jelenti, hogy a hálózati kommunikáció nem jött létre. Amikor ez megtörténik, az nem mindig jelenti azt, hogy a cél-IP nem aktív. Néhány hoszton lehet egy tűzfal, amely blokkolja az ICMP-forgalmat, vagy úgy van beállítva, hogy ne válaszoljon a ping-kérésekre.

Siker esetén a `ping` parancs 0-s kilépési kóddal tér vissza. Ellenkező esetben 1-es vagy 2-es kilépési kóddal tér vissza. Ez hasznos lehet, ha a `ping` segédprogramot shell szkriptben használod.

A következő szakaszokban áttekintjük a leggyakrabban használt `ping` parancs opciókat.

> **Info:** Egyes rendszereken a `ping` emelt jogosultságokat igényel. Ha engedélyhibát látsz, futtasd `sudo`-val, vagy telepítsd a `ping`-et a megfelelő képességekkel.

## Csomagok számának megadása

Ahogy már említettük, alapértelmezésben a `ping` addig folytatja az ICMP-csomagok küldését, amíg megszakítási jelet nem kap. Az elküldendő Echo Request csomagok számának megadásához, amely után a `ping` kilép, használd a `-c` opciót, amelyet a csomagok száma követ:

```bash
ping -c 1 DESTINATION
```

Például, hogy a `linuxize.com`-ot csak egyszer pingeld, ezt használnád:

```bash
ping -c 1 linuxize.com
```

## Forrásinterfész megadása

A `ping` parancs alapértelmezett viselkedése az, hogy ICMP-csomagokat küld az alapértelmezett útvonalon. Ha több interfészed van a gépeden, megadhatod a forrásinterfészt a `-I` opcióval:

```bash
ping -I INTERFACE_NAME DESTINATION
```

A következő parancs a `linuxize.com`-ot pingeli az `em2` forrásinterfészt használva:

```bash
ping -I em2 linuxize.com
```

## Internetprotokoll megadása

Amikor a `ping` parancsot futtatod, az IPv4-et vagy IPv6-ot használ, a géped DNS-beállításaitól függően.

Az IPv4 használatának kényszerítéséhez add meg a `-4` opciót, vagy használd a `ping4` aliast:

```bash
ping -4 DESTINATION
```

IPv6-hoz add meg a `-6` opciót, vagy használd a `ping6`-ot:

```bash
ping -6 DESTINATION
```

## Intervallum és időtúllépés módosítása

Alapértelmezésben a `ping` másodpercenként egy csomagot küld. Három külön időértéket szabályozhatsz: a csomagok közötti intervallumot, a válaszonkénti időtúllépést és egy általános határidőt.

A `-i` opcióval módosíthatod, milyen gyakran küldi a csomagokat. A következő parancs két másodpercenként küld egy csomagot:

```bash
ping -i 2 google.com
```

A 0,2 alatti értékek root jogosultságokat igényelnek. Hosszabb intervallum beállítása hasznos könnyű háttérfigyeléshez; rövidebb beállítása lehetővé teszi, hogy gyorsan több mintát gyűjts.

A `-W` opcióval beállíthatod, mennyi ideig vár a `ping` egy válaszra, amikor nem jönnek vissza válaszok. A következő parancs két másodperces válaszablakot ad a `ping`-nek:

```bash
ping -W 2 google.com
```

Az alapértelmezett várakozási idő 10 másodperc. Csökkentése hasznos ismerten gyors linkeken, ahol a lassú válasz valódi problémát jelez.

A `-w` opcióval beállíthatsz egy általános határidőt másodpercben, amely után a `ping` leáll, függetlenül attól, hány csomagot küldött:

```bash
ping -w 10 google.com
```

Ez különösen hasznos shell szkriptekben, ahol a teljes futási időt szeretnéd korlátozni a csomagszámtól függetlenül.

## Csomagméret módosítása

Az ICMP hasznos teher méretének megváltoztatásához használd a `-s` opciót:

```bash
ping -s 120 DESTINATION
```

## Csendes kimenet

Csak összefoglaló megjelenítéséhez használd a `-q` opciót:

```bash
ping -q -c 5 DESTINATION
```

## Gyors referencia

| Opció | Leírás |
| --- | --- |
| `-c COUNT` | COUNT csomag küldése, majd kilépés |
| `-i INTERVAL` | Másodpercek a csomagok között (alapértelmezés: 1) |
| `-W TIMEOUT` | Másodpercek, amennyit válaszra vár, amikor nem érkeznek válaszok |
| `-w DEADLINE` | Leállás DEADLINE másodperc után összesen |
| `-s SIZE` | ICMP hasznos teher mérete bájtban (alapértelmezés: 56) |
| `-q` | Csendes kimenet (csak összefoglaló) |
| `-I INTERFACE` | Küldés az INTERFACE-en keresztül |
| `-4` | IPv4 kényszerítése |
| `-6` | IPv6 kényszerítése |

## Következtetés

A `ping` egy egyszerű, de hatékony eszköz az IP-szintű kapcsolódás tesztelésére, a késleltetés mérésére és a csomagvesztés felderítésére. Az összes elérhető opció mélyebb megismeréséhez futtasd a `man ping` parancsot a terminálban.

## Forrás 
-[https://itsfoss.com/ping-command/]
-[https://linuxize.com/post/linux-ping-command/]
