# Tartománynév-feloldás (Domain név feloldás)

Általánosságban egy domainnév egy IP-címet képvisel, és a Domain Name System (DNS - Tartománynév-rendszer) kapcsolja hozzá.

Ez a cikk elmagyarázza, hogyan kell konfigurálni a domainnév-feloldást és hogyan történik a domainnevek feloldása.

## Name Service Switch

> Ez a cikk vagy szakasz bővítésre szorul. Ok: Említeni kell a `nss-mdns`, `nss-tls-gitAUR` és másokat. (Megbeszélés a Talk:Domain name resolution oldalon)

A Name Service Switch (NSS) szolgáltatás a GNU C Library (glibc) része, és a domainnevek feloldásához használt `getaddrinfo(3)` API mögött áll. Az NSS lehetővé teszi, hogy a rendszeradatbázisokat különálló szolgáltatások biztosítsák, amelyek keresési sorrendjét a rendszergazda állíthatja be az `nsswitch.conf(5)` fájlban. A domainnév-feloldásért felelős adatbázis a `hosts` adatbázis, amelyhez a glibc a következő szolgáltatásokat kínálja:

- `files`: az `/etc/hosts` fájlt olvassa, lásd `hosts(5)`
- `dns`: a glibc feloldó (resolver), amely az `/etc/resolv.conf` fájlt olvassa, lásd `resolv.conf(5)`

A systemd három NSS szolgáltatást nyújt a gépnév (hostname) feloldásához:

- `nss-resolve(8)` — egy gyorstárazó (caching) DNS stub resolver, a `systemd-resolved` cikkben leírva
- `nss-myhostname(8)` — helyi gépnév-feloldást biztosít az `/etc/hosts` szerkesztése nélkül
- `nss-mymachines(8)` — gépnév-feloldást biztosít a helyi `systemd-machined(8)` konténerek neveihez

### Domainnév feloldása az NSS használatával

Az NSS adatbázisok a `getent(1)` paranccsal kérdezhetők le. Egy domainnév feloldható az NSS-en keresztül a következőképpen:

```bash
$ getent ahosts domain_name
```

> **Megjegyzés:** Bár a legtöbb program az NSS használatával oldja fel a domainneveket, némelyik közvetlenül az `/etc/resolv.conf` és/vagy az `/etc/hosts` fájlt olvashatja. Lásd a Hálózati konfiguráció#A helyi gépnév a hálózaton keresztül oldódik fel részt.

## Glibc feloldó (resolver)

A glibc feloldó (resolver) minden feloldásnál beolvassa az `/etc/resolv.conf` fájlt, hogy meghatározza a használandó névszervereket és opciókat.

A `resolv.conf(5)` felsorolja a névszervereket néhány konfigurációs opcióval együtt.

Az elsőként felsorolt névszerverekkel próbálkozik először, legfeljebb három névszerver adható meg. A számjellel (`#`) kezdődő sorokat a rendszer figyelmen kívül hagyja.

> **Megjegyzés:** A glibc resolver nem gyorstárazza (cache) a lekérdezéseket. A lekérdezési idő javítása érdekében beállíthat egy gyorstárazó feloldót (caching resolver). A glibc resolver a DNSSEC érvényesítésére sem képes. Ehhez egy DNSSEC érvényesítő feloldóra van szükség. További információkért lásd a #DNS szerverek részt.

### Az `/etc/resolv.conf` felülírása

A hálózatkezelők (network managers) hajlamosak felülírni az `/etc/resolv.conf` fájlt, a részletekért lásd a megfelelő szakaszt:

- dhcpcd#/etc/resolv.conf
- Netctl#/etc/resolv.conf
- NetworkManager#/etc/resolv.conf
- ConnMan#/etc/resolv.conf

Hogy megakadályozzuk a programokat az `/etc/resolv.conf` felülírásában, írásvédetté is tehetjük a megváltoztathatatlan (immutable) fájlattribútum beállításával:

```bash
# chattr +i /etc/resolv.conf
```

> **Tipp:** Ha azt szeretné, hogy több folyamat is írhasson az `/etc/resolv.conf` fájlba, használhatja a `resolvconf`-ot.

### Alternatíva az `nmcli` használatával

> Ez a cikk vagy szakasz a NetworkManager#/etc/resolv.conf részbe való beolvasztásra jelölt. Megjegyzések: A NetworkManagernek dedikált szakasza van erre a témára. (Megbeszélés a Talk:Domain name resolution oldalon)

Ha NetworkManagert használ, az `nmcli(1)` használható tartós (persistent) opciók beállítására az `/etc/resolv.conf` számára. Cserélje ki a "Wired"-et a kapcsolata nevére. Példa:

```bash
# nmcli con mod Wired +ipv4.dns-options 'rotate,single-request,timeout:1'
```

További opciókért tekintse meg az `nmcli(1)`, `nm-settings-nmcli(5)` és a `resolv.conf(5)` man oldalait.

### Keresési idő korlátozása

Ha nagyon hosszú gépnév-kereséssel szembesül (legyen az a pacmanben vagy böngészés közben), gyakran segít egy rövid időtúllépés (timeout) megadása, amely után egy alternatív névszervert használ a rendszer. Ehhez tegye a következőket az `/etc/resolv.conf` fájlba.

`/etc/resolv.conf`

```plaintext
options timeout:1
```

### Késleltetett gépnév-keresés IPv6 esetén

Ha 5 másodperces késedelmet tapasztal a gépnevek feloldásakor, annak oka lehet egy hibásan működő DNS-szerver/Tűzfal, amely csak egyetlen választ ad egy párhuzamos A és AAAA lekérdezésre.[1] Ezt a következő opció beállításával javíthatja az `/etc/resolv.conf` fájlban:

`/etc/resolv.conf`

```plaintext
options single-request
```

### Helyi domainnevek

Ahhoz, hogy a helyi gépek gépnevét a teljesen minősített domainnév (FQDN) nélkül tudja használni, adjon hozzá egy sort az `/etc/resolv.conf` fájlhoz a helyi domainnel, például:

`/etc/resolv.conf`

```plaintext
search example.org
```

Így a helyi gépekre, mint például a `mainmachine1.example.org`, egyszerűen `mainmachine1`-ként hivatkozhat az `ssh` parancs használatakor, de a `drill` parancsnak továbbra is szüksége van a teljesen minősített domainnevekre a keresések végrehajtásához.

## Keresési segédprogramok (Lookup utilities)

Specifikus DNS szerverek és DNS/DNSSEC rekordok lekérdezéséhez használhat dedikált DNS-kereső segédprogramokat vagy azokat, amelyeket a DNS szerverekkel szállítanak. Ezek az eszközök maguk valósítják meg a DNS-t, és nem használják az NSS-t.

- `drill(1)` — Egy eszköz, amelyet a DNS-ből származó információk lekérésére terveztek. Csak a titkosítatlan DNS-t támogatja. (https://nlnetlabs.nl/projects/ldns/ || ldns)

  Például, ha egy adott névszervert szeretne lekérdezni a `drill` segítségével egy domain TXT rekordjaiért: `$ drill domain @névszerver TXT`

  Hacsak nincs megadva DNS szerver, a `drill` az `/etc/resolv.conf` fájlban meghatározott névszervereket fogja használni.

- `adig(1)` — Lekérdezéseket küld a DNS szervereknek a névről, és kinyomtatja a kapott információkat. (https://c-ares.org/ || c-ares)
- `dnsi` — Egy parancssori eszköz a DNS különböző aspektusainak vizsgálatára. (https://github.com/NLnetLabs/dnsi || dnsiAUR)
- `dnslookup` — Egy egyszerű parancssori segédprogram DNS keresésekhez. Támogatja az összes ismert DNS protokollt. (https://github.com/ameshkov/dnslookup || dnslookupAUR)
- `dog(1)` — Egy parancssori DNS kliens, mint a `dig`. (https://github.com/ogham/dog || dog)
- `doggo` — Parancssori DNS kliens embereknek. (https://github.com/mr-karan/doggo || doggo)
- `q` — Egy apró parancssori DNS kliens. (https://github.com/natesales/q || q-dnsAUR)

Néhány DNS-szerver csomag DNS-kereső segédprogramokkal érkezik, amelyek a DNS-szerver futtatása nélkül is használhatók:

- A `knot` biztosítja a `khost(1)` és a `kdig(1)` programokat.
- Az `unbound` biztosítja az `unbound-host(1)` programot.
- A `bind` biztosítja a `dig(1)`, `host(1)` és `nslookup(1)` programokat.
- A `powerdns` biztosítja az `sdig(1)` programot.

> **Tipp:** A `systemd-resolved` rendelkezik a `resolvectl(1)` paranccsal, amely biztosít egy `query` alparancsot a DNS kereséshez. Ezt csak a `systemd-resolved`-del lehet használni.

## Feloldó (Resolver) teljesítmény

A Glibc resolver nem gyorstárazza (cache) a lekérdezéseket. A helyi gyorstárazás megvalósításához használja a `systemd-resolved`-t, vagy állítson be egy helyi gyorstárazó DNS-szervert, és használja azt névszerverként úgy, hogy a `127.0.0.1` és a `::1` címeket állítja be névszerverként az `/etc/resolv.conf` vagy az `/etc/resolvconf.conf` fájlban, ha az `openresolv`-ot használja.

> **Tipp:** A `drill`, `dig` és `kdig` kereső segédprogramok jelentik a lekérdezési időt.

Egy router általában a saját gyorstárazó feloldóját állítja be a hálózat DNS szervereként, ezzel biztosítva a DNS gyorsítótárat az egész hálózat számára.

Ha túl sokáig tart átváltani a következő DNS szerverre, megpróbálhatja csökkenteni az időtúllépést (timeout).

## Adatvédelem és biztonság

> Ez a cikk vagy szakasz bővítésre szorul. Ok: Dokumentálni kell a DNS szűrést (DNS sinkholing). Példaként hivatkozni az Unbound#Domain blacklisting és a dnsmasq#Domain blocklisting cikkekre. (Megbeszélés a Talk:Domain name resolution oldalon)

A DNS protokoll (Do53) titkosítatlan, és nem biztosítja a bizalmasságot, integritást vagy hitelesítést, így ha nem megbízható hálózatot vagy rosszindulatú internetszolgáltatót (ISP) használ, a DNS lekérdezéseit lehallgathatják, a válaszokat pedig manipulálhatják. Továbbá a DNS szerverek végezhetnek DNS eltérítést (hijacking).

Bíznia kell abban, hogy a DNS szervere bizalmasan kezeli a lekérdezéseit. A DNS szervereket internetszolgáltatók és harmadik felek biztosítják. Alternatív megoldásként futtathat saját rekurzív névszervert (más néven rekurzív feloldó vagy DNS recursor), ami azonban több erőfeszítést igényel. Ha DHCP klienst használ nem megbízható hálózatokon, győződjön meg róla, hogy statikus névszervereket állít be, hogy elkerülje a tetszőleges DNS szerverek használatát és kiszolgáltatottságát, vagy alternatívaként használjon VPN-t egy biztonságos hálózathoz való csatlakozáshoz, és használja annak DNS szervereit. A távoli DNS szerverrel folytatott kommunikáció biztosításához használhat titkosított protokollt, feltéve, hogy mind a felfelé irányuló (upstream) szerver, mind a helyi feloldó támogatja a protokollt. A gyakori titkosított DNS protokollok a következők:

- DNS over TLS (DoT)—RFC 7858,
- DNS over HTTPS (DoH)—RFC 8484,
- DNS over QUIC (DoQ)—RFC 9250,
- DNSCrypt.

Annak ellenőrzésére, hogy a válaszok ténylegesen mérvadó (authoritative) névszerverektől származnak-e, érvényesítheti a DNSSEC-t, feltéve, hogy mind a felfelé irányuló szerver(ek), mind a helyi feloldó támogatja azt.

### TLS Server Name Indication (Kiszolgálónév-jelzés)

Bár használható titkosított DNS-feloldó, egy TLS-kapcsolat továbbra is kiszivárogtatja a domainneveket a Server Name Indication (SNI) mezőben a domain tanúsítványának kérésekor. Ez a szivárgás ellenőrizhető a Wireshark `tls.handshake.extensions_server_name_len > 0` szűrőjével, vagy a következő `tshark` paranccsal:

```bash
# tshark -p -Tfields -e tls.handshake.extensions_server_name -Y 'tls.handshake.extensions_server_name_len>0'
```

A javasolt megoldás az Encrypted Client Hello (ECH), egy TLS 1.3 protokoll kiterjesztés használata.

### Alkalmazásszintű DNS

Legyen tisztában azzal, hogy egyes kliensszoftverek, például a nagyobb webböngészők[2][3], kezdik implementálni a DNS over HTTPS-t. Bár a lekérdezések titkosítása gyakran bónusznak tekinthető, ez azt is jelenti, hogy a szoftver kikerüli a rendszer-feloldó (system resolver) konfigurációját a lekérdezések során.[4]

A Firefox konfigurációs opciókat biztosít a DNS over HTTPS engedélyezéséhez vagy letiltásához, és a DNS-szerver kiválasztásához. A Mozilla létrehozott egy Trusted Recursive Resolver (TRR - Megbízható Rekurzív Feloldó) programot átláthatósági információkkal az alapértelmezett szolgáltatóikról. Figyelemre méltó, hogy a Firefox támogatja és automatikusan engedélyezi az Encrypted Client Hello-t (ECH) a TRR szolgáltatók számára, lásd Firefox/Privacy#Encrypted Client Hello.

A Chromium megvizsgálja a felhasználó rendszer-feloldóját, és engedélyezi a DNS over HTTPS-t, ha a rendszer-feloldó címei ismertek arról, hogy DNS over HTTPS-t is nyújtanak. További információkért és a DNS over HTTPS letiltásának módjáért lásd ezt a blogbejegyzést.

A Mozilla javasolta az alkalmazásszintű DNS univerzális letiltását, ha a rendszer-feloldó nem tudja feloldani az `use-application-dns.net` domaint. Jelenleg ez csak a Firefoxban van megvalósítva.

### Oblivious DNS over HTTPS

Az Oblivious DNS over HTTPS (ODoH)—RFC 9230—egy olyan rendszer, amely számos DNS-adatvédelmi aggályt kezel. További információért lásd a Cloudflare cikkét. Hozzáadta a DNS over HTTPS-t az akadémiai Oblivious DNS dizájnhoz. A különbségek tárgyalásához lásd a Improving the privacy of DNS and DoH with oblivion című cikket.

## Rekurzív feloldó

> Ez a cikk vagy szakasz bővítésre szorul. Ok: El kell magyarázni a QNAME minimalizálást. (Megbeszélés a Talk:Domain name resolution oldalon)

A rekurzív feloldók és a gyökérszerverek (root servers) közötti kommunikáció nem titkosított, és a gyökérszerver-üzemeltetők ellenzik annak bevezetését. A mérvadó (authoritative) szerverekkel folytatott titkosított kommunikációra létezik a kísérleti RFC 9539, amely lehetővé teszi a DNS over TLS és a DNS over QUIC opportunista használatát.

## Harmadik féltől származó DNS szolgáltatások

> **Megjegyzés:** Mielőtt harmadik féltől származó DNS-szolgáltatást használna, ellenőrizze az adatvédelmi irányelveit arra vonatkozóan, hogy hogyan kezelik a felhasználói adatokat. A felhasználói adatok értéket képviselnek, és eladhatók más feleknek.

Erősen ajánlott titkosított protokoll használata harmadik féltől származó DNS-szolgáltatásokhoz való csatlakozáskor.

Különféle harmadik féltől származó DNS-szolgáltatások léteznek. A Wikipédián található egy lista a "nevezetes" nyilvános DNS-szolgáltatókról, míg a curl projekt wikijén a nyilvánosan elérhető DNS over HTTPS szerverek egy átfogóbb listája található (amelyek közül sok támogatja a DNS over TLS-t is). A systemd csomag tartalék (fallback) DNS-t konfigurál a systemd-resolved számára, amikor nincsenek DNS-szerverek konfigurálva (kézzel vagy DHCP/RA révén).

Használhatja a `dnsperftest`-et a legnépszerűbb DNS-feloldók teljesítményének tesztelésére a saját tartózkodási helyéről. A dnsperf.com globális teljesítményteszteket biztosít a szolgáltatók között.

### Harmadik féltől származó DNS-kliens szoftverek

Néhány DNS-szolgáltatás dedikált szoftvert is biztosít:

- `alfis` — Alternative Free Identity System. (https://github.com/Revertron/Alfis || alfisAUR)
- `nextdns` — Egy DNS-over-HTTPS parancssori kliens a NextDNS-hez (https://github.com/nextdns/nextdns || nextdnsAUR)
- `opennic-up` — Automatizálja a DNS szerverek megújítását a leggyorsabban reagáló OpenNIC szerverekkel (https://github.com/kewlfft/opennic-up || opennic-upAUR)

## DNS szerverek

A DNS-szerverek lehetnek mérvadók (authoritative) és rekurzívak. Ha egyik sem, akkor stub resolvereknek nevezzük őket, és egyszerűen továbbítanak minden lekérdezést egy másik rekurzív névszervernek. A stub resolvereket jellemzően a DNS gyorstárazás (caching) bevezetésére használják a helyi gépen vagy hálózaton. Vegye figyelembe, hogy ugyanez egy teljes értékű névszerverrel is elérhető. Ez a szakasz összehasonlítja az elérhető DNS-szervereket; részletesebb összehasonlításért lásd a Wikipedia:Comparison of DNS server software cikket.

| Név | Csomag | Képességek (resolvconf) | Támogatott protokollok: Mérvadó | Rekurzív | Gyorstár (Cache) | Érvényesíti a DNSSEC-t | DNS | DNSCrypt | DNS over TLS | DNS over HTTPS | DNS over QUIC |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BIND | bind | Igen | Igen | Igen | Igen | Igen | Igen | Nem | Igen (Szerver) | Nem | Nem |
| CoreDNS | corednsAUR | Igen | Nem | Igen | Nem | Nem | Igen | Nem | Igen (Szerver) | Nem | Nem |
| DNS-over-HTTPS | dns-over-https | Nem | Nem | Nem | Nem | Nem | Igen | Nem | Nem | Igen | Nem |
| Deadwood (MaraDNS recursor) | maradnsAUR | Nem | Igen | Igen | Nem | Nem | Igen | Nem | Nem | Nem | Nem |
| dnscrypt-proxy | dnscrypt-proxy | Nem | Nem | Igen | Nem | Nem | Szerver | Feloldó | Nem | Igen | Nem |
| dnsmasq | dnsmasq | Részleges¹ | Nem | Igen | Igen² | Igen | Igen | Nem | Nem | Nem | Nem |
| dnsproxy | dnsproxy | Nem | Nem | Igen | Nem | Nem | Igen | Igen | Igen | Igen | Igen |
| Knot Resolver | knot-resolver | Nem | Igen | Igen | Igen | Nem | Igen | Nem | Igen (Szerver) | Nem | Nem |
| pdnsd | pdnsd | Részleges¹ | Igen | Tartós³ | Nem | Igen | Igen | Nem | Nem | Nem | Nem |
| PowerDNS Recursor | powerdns-recursor | Nem | Igen | Igen | Igen² | Igen | Igen | Nem | Részleges | Nem | Nem |
| Rescached | rescachedAUR | Nem | Nem | Igen | Nem | Igen | Igen | Nem | Igen | Igen | Nem |
| RouteDNS | routedns-gitAUR | Nem | Nem | Igen³ | Nem | Nem | Igen | Nem | Igen | Igen | Igen |
| SmartDNS | smartdns | Nem | Nem | Igen | Nem | Nem | Igen | Nem | Igen | Igen | Feloldó |
| Stubby | stubby | Nem | Nem | Nem | Igen² | Nem | Szerver | Nem | Feloldó | Nem | Nem |
| systemd-resolved | systemd | Nem | Nem | Igen | Kísérleti² | Igen | Feloldó és korlátozott szerver | Nem | Feloldó | Nem | Nem |
| Unbound | unbound | Részleges | Igen | Igen³ | Igen | Igen | Igen | Szerver | Igen | Szerver | Szerver |

1 A Wikipédiából: korlátozott mérvadó (authoritative) támogatás, inkább belső hálózati használatra szánták, mint nyilvános internetes használatra.

2 A DNSSEC érvényesítés alapértelmezés szerint le van tiltva, és a konfigurációs fájlban kell engedélyezni.

3 Támogatja a tartós (persistent) gyorstárat a Redis háttérprogram használatával.

> **Tipp:** Azok a DNS szerverek, amelyek nem támogatják a DNS over TLS-t, használhatják az `stunnel`-t a TLS titkosítás hozzáadására. Resolver funkcionalitáshoz ez megköveteli a TCP kényszerítésének képességét a továbbítás során.

### Csak mérvadó (authoritative) szerverek

| Név | Csomag | DNSSEC aláírás | Földrajzi terheléselosztás |
| --- | --- | --- | --- |
| gdnsd | gdnsd | Nem | Igen |
| Knot DNS | knot | Igen | Igen |
| MaraDNS | maradnsAUR | Nem | Nem |
| NSD | nsd | Igen | Nem |
| PowerDNS | powerdns | Igen | Igen |

## Feltételes továbbítás

Lehetséges specifikus DNS-feloldókat használni adott domainnevek lekérdezésekor. Ez különösen hasznos VPN-hez való csatlakozáskor, így a VPN hálózatra irányuló lekérdezéseket a VPN DNS-e oldja fel, míg az internetre irányuló lekérdezéseket továbbra is az Ön szabványos DNS-feloldója fogja feloldani. Helyi hálózatokon is használható.

A megvalósításához egy helyi feloldót kell használnia, mivel a glibc nem támogatja.

Dinamikus környezetben (laptopok és bizonyos mértékig asztali gépek) a csatlakozott hálózat(ok) alapján kell konfigurálnia a feloldóját. Ennek legjobb módja az `openresolv` használata, mivel az több feliratkozót (subscribers) is támogat. Egyes hálózatkezelők támogatják ezt, akár az `openresolv`-on keresztül, akár a resolver közvetlen konfigurálásával. A NetworkManager `openresolv` nélkül is támogatja a feltételes továbbítást.

# Az `nslookup` Parancs Linuxon (DNS Lekérdezések és Hibaelhárítás)

Az `nslookup` egy népszerű parancssori eszköz, amellyel lekérdezhetők a DNS (Domain Name System) rekordok, domain nevek vagy IP-címek. Különösen hasznos hálózati és DNS-problémák hibaelhárításához.

> **Megjegyzés:** Bár a `dig` parancs szintén nagyon népszerű és hatékony alternatíva, az `nslookup` sokszor sokkal egyszerűbben és áttekinthetőbben használható a gyors lekérdezésekhez.

## 1. Telepítés a Különböző Linux Disztribúciókon

A legtöbb rendszeren alapból elérhető, de ha hiányzik, az alábbi csomagkezelőkkel telepíthető:

- **Ubuntu, Debian, Linux Mint (Debian-alapúak):**

  ```bash
  sudo apt install dnsutils
  ```

- **CentOS, Fedora, RHEL (Red Hat-alapúak):**

  ```bash
  sudo dnf install bind-utils
  ```

- **Arch Linux, Manjaro (Arch-alapúak):**

  ```bash
  sudo pacman -S dnsutils
  ```

## 2. Gyakorlati Parancssori Példák

**Alapvető DNS lekérdezés (A-rekord):**

Ha megadunk egy domain nevet, az `nslookup` visszaadja a hozzá tartozó IP-címet.

```bash
nslookup redhat.com
```

**Fordított DNS lekérdezés (Reverse DNS):**

Ha egy IP-címet adunk meg argumentumként, a parancs megmutatja, hogy az milyen domain névhez tartozik:

```bash
nslookup 209.132.183.105
```

**E-mail szerverek (MX rekordok) lekérdezése:**

A `-query=mx` (vagy `-type=mx`) opcióval megnézhetjük, hogy hová érkeznek a domainre küldött e-mailek:

```bash
nslookup -query=mx redhat.com
```

**Névkiszolgálók (NS rekordok) listázása:**

A `-type=ns` kapcsoló megmutatja, hogy mely DNS szerverek felelnek az adott domain kiszolgálásáért:

```bash
nslookup -type=ns redhat.com
```

**További hasznos lekérdezések:**

- **TXT rekordok** (pl. SPF, DMARC, domain-ellenőrzés):

  ```bash
  nslookup -type=txt redhat.com
  ```

- **CNAME rekordok** (aliasok):

  ```bash
  nslookup -type=cname www.redhat.com
  ```

- **AAAA rekordok** (IPv6 címek):

  ```bash
  nslookup -type=aaaa redhat.com
  ```

- **Adott DNS szerver használata** (pl. Google DNS):

  ```bash
  nslookup redhat.com 8.8.8.8
  ```

- **Interaktív mód** (több lekérdezés egymás után):

  ```bash
  nslookup
  > server 8.8.8.8
  > set type=mx
  > redhat.com
  > exit
  ```

## 3. Az `nslookup` Kimenetének Értelmezése

Egy tipikus kimenet két részből áll:

- **Server:** A választ adó DNS szerver neve és IP-címe.
- **Non-authoritative answer:** A válasz egy gyorstárazó (caching) szervertől érkezett, nem a domain hivatalos (authoritative) névszerverétől.
- **Name / Address:** A lekérdezett név és a hozzá tartozó IP-cím(ek).

Ha a válasz `NXDOMAIN` (Non-Existent Domain) hibát jelez, az azt jelenti, hogy a keresett domain nem létezik.

## 4. Hibaelhárítási Tippek

- **Nincs válasz / timeout:** Ellenőrizd az `/etc/resolv.conf` fájlt, hogy a DNS szerverek elérhetők-e.
- **Rossz válasz:** Próbáld meg közvetlenül egy nyilvános DNS szervert lekérdezni (pl. `8.8.8.8` vagy `1.1.1.1`).
- **DNS gyorsítótár problémák:** Ha a válasz elavult, próbáld meg kiüríteni a helyi DNS gyorsítótárat (pl. `systemd-resolved` esetén `sudo resolvectl flush-caches`).
- **DNSSEC problémák:** Az `nslookup` nem támogatja a DNSSEC érvényesítést; erre használj `dig +dnssec` parancsot.

## Forrás
-[https://linuxconfig.org/nslookup-linux-command]

# A `dig` Parancs Használata és Példák

A `dig` parancsot Linuxon leggyakrabban egy távoli szerver DNS-információinak lekérdezésére használják. Ismerd meg a `dig` parancs használatát és a kimenetének értelmezését.

## A `dig` parancs Linuxon

A `dig` parancsot rendszer- és hálózati adminisztrátorok körében széles körben használják Linuxon. A név a „Domain Information Groper” rövidítése, és arra szolgál, hogy lekérdezze egy adott szerver DNS-ét, valamint hogy válaszokat kapjon a lekérdezett domain-kiszolgálóktól.

Nézzük meg, hogyan működik a parancs, és hogyan kell értelmezni a kimenetét:

## A `dig` parancs futtatása

A legelső sor a program verzióját írja ki (9.11.3), és jelzi, hogy honnan hová indul a lekérdezés. Ebben az esetben az én Ubuntu gépemről a linuxhandbook.com szerverre.

Ezután megjeleníti a (domain) szervertől kapott választ. Kiírja azt a címet, amelyre a `linuxhandbook.com` név egy A-rekord típusban mutat. Ez lehet, hogy a szerver IP-címe, de az is lehet, hogy nem, mert ha valami DNS-tűzfalat vagy „homlokzat” szervert használ biztonsági vagy szűrési célból, akkor azt látnánk először – de a linuxhandbook.com szerver esetében nem ez a helyzet. Sok esetben a `dig` elég ahhoz, hogy megtaláljuk egy weboldal IP-címét.

Végül statisztikákat ad a lekérdezésről, ami hasznos lehet, ha a lekérdezés sebességét szeretnénk felmérni.

## Rendben, de mi a `dig` parancs használata vagy értéke?

Valójában attól függ, milyen típusú információt keresel. Ne feledd, hogy először egy kicsit értened kell a DNS-hez, például hogy milyen típusú DNS-rekordok léteznek, és mire használják őket.

Egy gyakori példa annak kiderítése, hogy egy adott domain hol hosztolja az e-mailjeit. Ebben az esetben:

Megpróbáljuk „dig”-elni a `microsoft.com` domain MX-rekordját, mert szeretnénk tudni, hol van hosztolva. Láthatjuk, hogy a válasz ez:

`microsoft-com.mail.protection.outlook.com.`

Ez a Microsoft e-mail-védelmi szolgáltatása, amelyet arra használnak, hogy megvédjék a `microsoft.com` domainre bejövő és onnan kimenő e-maileket, és így megelőzzék a vírusokat, trójaiakat, spamet stb.

## Mi van, ha többet szeretnék megtudni egy IP-címről?

Ez a `dig` parancs egy másik felhasználási módja. Ha így adod meg:

```bash
dig -x IP_CÍM
```

Akkor többet tudhatsz meg egy adott IP-címről. Ebben az esetben a linuxhandbook.com jelentett IP-címét használtuk a `-x` kapcsolóval, és a válasz az volt, hogy az az IP a `cloudwayapps.com`-hoz tartozik, amely annak a szolgáltatásnak a része, amely a linuxhandbook.com weboldal jelenlegi hosting cége.

## Több domain lekérdezése egyszerre

Arra is használhatod, hogy egyszerre több domaint „dig”-elj, egyszerűen csak felsorolva azokat a domaineket, amelyekről több információt szeretnél:

```bash
dig domain1.com domain2.com domain3.com
```

## Összegzés

Összefoglalva, a `dig` parancs lehetővé teszi, hogy lényegében lefúrj egy adott domainről és/vagy IP-címről szóló információkat, és többet megtudj a DNS-beállításairól. Az opciók kombinációja a legfontosabb rész, és alapvetően mindig használhatod a parancs man oldalait, hogy többet tudj meg a különböző operátorokról és arról, mit nyújthatnak. Leegyszerűsítettem a leggyakoribb használatokat, de rengeteg mindent lehet „kiásni” ebben a parancsban!

## Forrás
-[https://linuxhandbook.com/dig-command/]

# DNS-feloldás beállítása a `resolv.conf` fájllal – Példák

**Közzétéve:** 2016. december 27. – Szerző: admin

Az `/etc/resolv.conf` konfigurációs fájl olyan információkat tartalmaz, amelyek lehetővé teszik a számítógép számára, hogy az alfanumerikus domainneveket numerikus IP-címekké alakítsa.

A domainnevek IP-címekké alakításának folyamatát **feloldásnak (resolving)** nevezzük.

DHCP használatakor a `dhclient` általában újraírja a `resolv.conf` fájlt a DHCP szervertől kapott információkkal.

## Hogyan szerkesszem az `/etc/resolv.conf` fájlt?

Használj szövegszerkesztőt, például `vi`-t vagy `gedit`-et Linux asztali környezetben:

```bash
# vi /etc/resolv.conf
```

Három fő konfigurációs direktíva van az `/etc/resolv.conf` fájlban:

- `nameserver` – DNS szerver IP-címe
- `domain` – A helyi gép domainneve
- `search` – Mely domaineket kell keresni

## A `nameserver` direktíva

A `nameserver` direktíva egy névszerver IP-címére mutat.

```plaintext
nameserver <IP-cím>
```

Ez lehet a saját névszervered, valamely nyilvános névszerver, vagy az internetszolgáltatód névszervere.

> **Megjegyzés:** Legfeljebb 3 névszerver adható meg.

**Példa:**

```plaintext
nameserver 192.168.0.100
nameserver 8.8.8.8
nameserver 8.8.4.4
```

## A `domain` direktíva

A helyi domainnév.

```plaintext
domain <helyi domainnév>
```

**Példa:**

```plaintext
domain domain.com
nameserver 192.168.0.100
nameserver 8.8.4.4
```

### Hogyan működik a `domain` direktíva a `resolv.conf` fájlban?

A `domain` direktívát rövid gépnevek (hostname-ek) – például `test` – feloldására használhatod.

Tehát ha a következő van az `/etc/resolv.conf` fájlban:

```plaintext
domain example.com
```

Akkor a számítógéped megpróbálja feloldani a `test.example.com` nevet.

## A `search` direktíva

A gépnév-keresés keresési listája. A keresési lista általában a helyi domainnévből származik, de beállítható domainek listájaként is.

```plaintext
search <keresési lista>
```

**Példa:**

```plaintext
search example.com company.net
nameserver 192.168.0.100
nameserver 8.8.8.8
```

### Hogyan működik a `search` direktíva a `resolv.conf` fájlban?

A `search` direktívát rövid gépnevek – például `test` – feloldására kell használni.

Tehát ha a következő van az `/etc/resolv.conf` fájlban:

```plaintext
search example.com company.net
```

Akkor a számítógéped megpróbálja feloldani a `test.example.com` nevet, majd ezután a `test.company.net` nevet. Az első sikeres lekérdezés eredményét adja vissza.

## Forrás
-[https://www.shellhacks.com/setup-dns-resolution-resolvconf-example/]
