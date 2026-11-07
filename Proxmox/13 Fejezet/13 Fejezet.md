# 13. Fejezet: Proxmox VE Tűzfal (Firewall)

A Proxmox VE beépített tűzfala **elosztott (distributed)** szolgáltatásként működik a clusterben. A szabályok (**rules**) nem egyetlen központi gépen értékelődnek ki, hanem minden egyes fizikai csomóponton (**Node**) és virtuális gépen/konténeren lokálisan hajtódnak végre (a háttérben az **iptables / netfilter**, vagy az újabb **nftables** segítségével). Ez hatalmas sávszélességet, teljes izolációt és kiemelkedő biztonságot nyújt a virtuális gépek (VM-ek) és konténerek (LXC) számára.

## 13.1 Irányok és Zónák (Directions & Zones)

A hálózati forgalom szűrésekor tisztában kell lenni a Proxmox VE által használt logikai irányokkal és zónákkal.

### 13.1.1 Irányok (Directions)

Az irány mindig annak a logikai eszköznek a szemszögéből értendő, amire a tűzfalszabályt alkalmazzák (legyen az maga a **Datacenter**, a **Node**, vagy egy **VM** virtuális hálózati kártyája).

- **`in` (Bejövő):** A hálózatról az eszköz (Host, VM, CT) felé irányuló forgalom.
- **`out` (Kimenő):** Az eszközből a hálózat felé irányuló forgalom.

### 13.1.2 Zónák (Zones)

A logikai elszeparálást segítik elő, különösen az **SDN** (Software-Defined Network) integráció esetén (bár a dokumentáció ezen a ponton röviden, a 12. fejezet SDN zónáira utal vissza, amelyekkel a tűzfal automatikusan együttműködik).

## 13.2 Konfigurációs Fájlok (Configuration Files)

A tűzfal beállításai több hierarchikus szinten tárolódnak a `/etc/pve` (**Proxmox Cluster File System – pmxcfs**) könyvtárban. Magát a tűzfalat globálisan is engedélyezni kell, valamint az egyes VM-ek hálózati interfészeinél külön is (a `firewall=1` opcióval a hálózati kártya beállításaiban).

### 13.2.1 Cluster Szintű Beállítások (Cluster Wide Setup)

A legfelső szint. Az itt megadott beállítások, biztonsági csoportok (**Security Groups**), IP aliasok és IP-készletek (**IP Sets**) minden node-on és VM-en elérhetők lesznek, és a beállított alapértelmezett (**Default**) szabályok a teljes clusterre vonatkoznak.

**Fájl helye:** `/etc/pve/firewall/cluster.fw`

### 13.2.2 Host Specifikus Konfiguráció (Host Specific Configuration)

Az adott fizikai gépre (Node-ra) vonatkozó szabályok, amelyek a host fizikai hálózati kártyáira, vagy a rajta futó lokális szolgáltatások (pl. SSH, GUI port 8006, Corosync) védelmére szolgálnak.

**Fájl helye:** `/etc/pve/nodes/<NODENAME>/host.fw`

### 13.2.3 VM/Konténer Konfiguráció (VM/Container Configuration)

Kizárólag az adott virtuális gép (VMID) vagy konténer hálózati interfészein áthaladó forgalmat szabályozza. Bár a fájlok a host szintű könyvtárban találhatók, logikailag a VM-hez kötődnek, és a VM migrációjakor a tűzfal beállításai is vele együtt mozognak.

- **Fájl helye (VM):** `/etc/pve/nodes/<NODENAME>/qemu-server/<VMID>.fw` (illetve a pmxcfs szimbolikus linkjein át: `/etc/pve/qemu-server/<VMID>.fw`)
- **Fájl helye (LXC):** `/etc/pve/nodes/<NODENAME>/lxc/<VMID>.fw` (illetve `/etc/pve/lxc/<VMID>.fw`)

### 13.2.4 VNet Konfiguráció (VNet Configuration)

Az SDN (Software-Defined Network) virtuális hálózatai (**VNets**) számára automatikusan generált, illetve ott definiált specifikus tűzfalszabályok, amelyek az adott VNet izolációját és forgalomszabályozását végzik.

## 13.3 Tűzfalszabályok (Firewall Rules)

A tűzfalszabályok felépítése követi a hagyományos hálózati konvenciókat (**Action, Source, Destination, Protocol, Port**).

- Minden szabály egy irányra (**IN** vagy **OUT**) vonatkozik.
- Az **Action** lehet **ACCEPT** (engedélyez), **DROP** (csendben eldob), vagy **REJECT** (eldob, de küld egy hibaüzenetet a feladónak).
- A szabályok kiértékelése fentről lefelé halad (**Top-Down**). Az első illeszkedő szabály (**match**) érvényesül.
- A szabályok engedélyezhetők (**Enable**) vagy letilthatók (**Disable**) anélkül, hogy törölni kellene őket.
- Használhatók a hálózati **makrók** (pl. Ping, SSH, HTTP), amik automatikusan beállítják a protokollokat és a portokat.

## 13.4 Biztonsági Csoportok (Security Groups)

A biztonsági csoportok (**Security Groups**) lehetővé teszik, hogy egy gyakran használt szabálykészletet (pl. egy webszerver szükséges bejövő és kimenő portjait) egy logikai csoportba fogj.

- A csoportokat a **Cluster** szintjén kell létrehozni (`/etc/pve/firewall/cluster.fw`).
- Ezután ezek a csoportok hozzárendelhetők bármelyik VM vagy Node tűzfalszabályzathoz (ahelyett, hogy gépenként újra be kellene gépelni az összes szabályt).
- Ha a biztonsági csoport szabályait módosítod a Cluster szinten, a változás azonnal életbe lép az összes olyan VM-en, ami használja azt a csoportot.

## 13.5 és 13.6 IP Aliasok és IP-készletek (IP Sets)

### 13.5 IP Aliasok

Az IP aliasok segítségével bonyolult IP-címek helyett beszédes, könnyen olvasható neveket (pl. `internal_network = 10.0.0.0/8`) használhatsz a tűzfalszabályokban.

**Standard IP Alias (`local_network`):** A Proxmox VE alapértelmezetten definiál egy `local_network` nevű aliast, amely a rendszer indulásakor automatikusan felveszi a host menedzsment IP-címének hálózati tartományát.

### 13.6 IP-készletek (IP Sets)

Az IP-készletek (**IP Sets**) logikai konténerek, amelyek több IP-címet, hálózatot (CIDR) vagy akár IP-tartományt fognak össze. Ez rendkívül hatékony, mert egyetlen tűzfalszabályban (pl. a Source vagy Destination mezőben) hivatkozhatsz egy több ezer IP-t tartalmazó készletre, anélkül, hogy a kernel terhelése megnőne (a háttérben **ipset**-eket használ).

**Standard (Beépített) IP-készletek:**

- **`management`:** Ez a halmaz azon IP-címek és hálózatok listáját tartalmazza, amelyekről engedélyezett a Proxmox VE host menedzsment felületének (GUI – TCP 8006, SSH – TCP 22, API) elérése. Ha biztonságosra akarod zárni a rendszered, töltsd fel ezt a készletet az adminisztrátori alhálózatok IP-címeivel.
- **`blacklist`:** Ebbe a készletbe felvett IP-címek és hálózatok forgalmát a tűzfal a teljes clusteren mindenhol azonnal eldobja (**Drop**).
- **`ipfilter-net` (pl. `ipfilter-net0`):** Ez egy speciális, VM/CT szinten használt IP-készlet. Ha létrehozol egy `ipfilter-net0` nevű készletet a VM-nél, és engedélyezed az **"IP Filter"** opciót a VM hálózati kártyáján (`net0`), a tűzfal megakadályozza, hogy a virtuális gép egy olyan IP-címet használjon forrás-IP-ként (**IP spoofing**), ami nincs benne ebben a listában. Ez kritikus védelmi funkció egy sok-bérlős (**multitenant**) környezetben!

## 13.7 és 13.8 Szolgáltatások (`pve-firewall`) és Alapértelmezett Szabályok

### Szolgáltatások és Parancsok

A Proxmox tűzfalát a **`pve-firewall`** démon (service) kezeli, amely a **pmxcfs**-en keresztül kapja a konfigurációkat, és másodperceken belül lefordítja azokat iptables / nftables szabályokra. A parancssorból is ezzel az eszközzel lehet beavatkozni:

```bash
pve-firewall start
pve-firewall stop
pve-firewall status
pve-firewall compile
```

### 13.8 Alapértelmezett Tűzfalszabályok (Default Rules)

Ha a tűzfal globálisan be van kapcsolva, az alábbi szigorú alapértelmezések érvényesülnek:

- **Datacenter bejövő (incoming):** **DROP** (Minden bejövő forgalom tiltott, kivéve, ami a `management` IPSet-ből jön, vagy amire van explicit ACCEPT szabály, pl. a Corosync vagy Ping forgalom).
- **Datacenter kimenő (outgoing):** **ACCEPT** (A host gépek kifelé alapértelmezetten bármit küldhetnek).
- **VM/CT bejövő (incoming):** **DROP** (A virtuális gépek alapértelmezetten nem fogadnak semmilyen hálózati forgalmat, hacsak nem nyitsz nekik portokat).
- **VM/CT kimenő (outgoing):** **ACCEPT** (A virtuális gépek alapból kommunikálhatnak a külvilággal, kivéve, ha ezt egy DROP szabállyal felülbírálod).

## 13.9 Naplózás (Logging)

A tűzfal képes a szabályokra illeszkedő forgalmat naplózni (**syslog**).
A logolás szintje beállítható (pl. `nolog`, `emerg`, `alert`, `crit`, `err`, `warning`, `notice`, `info`, `debug`).

**13.9.1:** A felhasználó által definiált (**User defined**) tűzfalszabályokhoz egyenként is bekapcsolható a naplózás. Ha a szabály akciója mellé (pl. ACCEPT vagy DROP) beállítod a log-szintet (pl. `info`), az érintett csomagok adatai bekerülnek a tűzfal naplófájljába, ami a GUI-n (a **Firewall Log** panelen) is nyomon követhető.

## 13.10 Tippek és Trükkök (Tips and Tricks)

**13.10.1 FTP engedélyezése:** A passzív FTP protokoll dinamikus portokat használ, ami problémás a tűzfalaknak. A dokumentáció leírja, hogy az FTP (port 21) mellé az FTP-szerver konfigurációjában (pl. ProFTPd, vsftpd) rögzíteni kell a passzív porttartományt (pl. TCP 50000–50100), majd ezt a porttartományt is külön ACCEPT szabállyal be kell engedni a Proxmox tűzfalán.

**13.10.2 Suricata IPS integráció:** A Proxmox tűzfala (az **NFLOG** target segítségével) képes a csomagokat átadni külső behatolás-megelőző rendszereknek (**Intrusion Prevention System**), mint amilyen a **Suricata**. Ezzel mély csomagelemzés (**DPI**) és fejlett támadás-kivédés valósítható meg.

## 13.11 és 13.12 IPv6 és Használt Portok

**13.11 IPv6 Megjegyzések:** A Proxmox tűzfal (a háttérben az ip6tables vagy nftables révén) zökkenőmentesen támogatja az IPv6-ot. A szabályoknál automatikusan felismeri, hogy a megadott CIDR vagy IP IPv4 vagy IPv6 formátumú-e, és aszerint alkalmazza a szűrést. **Fontos:** ha egy szabályhoz nem adsz meg IP-t, vagy "makrót" használsz, az mindkét protokollra érvényes lesz.

**13.12 A Proxmox VE által használt portok:** A rendszer működéséhez elengedhetetlen (beépített) portok, amiket a tűzfal automatikusan kezel (de egyedi szigorításoknál érdemes tudni róluk):

- **TCP 8006:** Webes GUI és REST API (menedzsment forgalom).
- **TCP 22:** SSH (cluster node-ok közötti kommunikáció, migráció, távoli elérés).
- **UDP 5405–5412:** Corosync cluster kommunikáció (Multicast vagy Unicast / Kronosnet).
- **TCP 3128:** Belső SPICE proxy.
- **TCP 5900–5999:** VNC / WebSocket portok a noVNC (HTML5) konzolokhoz.
- **TCP 85:** (PVE 8.1 óta) pveproxy belső kommunikáció.

## 13.13 Nftables (Az Új Generáció)

A Linux kernel csomagszűrő mechanizmusa éveken át az **iptables** volt. A Proxmox VE (Debian alapokon) fokozatosan átáll a modernebb, gyorsabb és hatékonyabb **nftables**-re.

**13.13.1 Telepítés és Használat:** Bár az iptables (ill. az `iptables-nft` wrapper) még támogatott, az **nftables** a javasolt jövőbiztos megoldás, mely alapértelmezetten rendelkezésre áll a modern Debian rendszereken.

**13.13.2 Usage (Működés):** Az nftables saját, optimalizált struktúrát (táblákat, láncokat, szabályokat és set-eket) használ. A Proxmox tűzfal démonja (**`pve-firewall`**) képes transzparensen legenerálni a szabályokat az nftables formátumába, így a GUI-n vagy a konfigfájlokban megszokott beállítások a háttérben automatikusan az új motorra fordulnak le.

**13.13.3 Hasznos Parancsok (Helpful Commands):** Az üzemeltetés és hibakeresés során az `nft` parancssori eszközt kell használni az iptables helyett:

```bash
nft list ruleset
nft list table inet filter
```
