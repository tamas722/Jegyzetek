# 12. Fejezet: Software-Defined Network (SDN)

A Proxmox VE SDN rendszere virtuális zónák (**Zones**) és hálózatok (**VNets**) létrehozását teszi lehetővé. Ez drasztikusan leegyszerűsíti a haladó hálózati konfigurációkat és a multitenant (sok-bérlős) környezetek kialakítását. A konfiguráció a **Datacenter** szintjén történik, majd automatikusan, helyi Linux híd (**bridge**) formájában kigöngyölítődik (**roll-out**) minden csomópontra, amelyhez a VM/LXC gépek csatlakozhatnak.

## 12.2 – 12.4 Támogatási státusz, Telepítés és Konfiguráció

A Proxmox VE 8.1 óta az **SDN Core** csomagok és a hálózati felépítés alapértelmezetten telepítve vannak, de az alábbi kiegészítő lépések szükségesek lehetnek:

- **SDN Core (Alaprendszer):** Ha régebbi PVE-ről frissítettél, az összes gépen telepíteni kell a `libpve-network-perl` és az `ifupdown2` csomagokat. Emellett a `/etc/network/interfaces` fájl végén kötelezően szerepelnie kell ennek a sornak:

```bash
source /etc/network/interfaces.d/*
```

- **DHCP IPAM (Tech Preview):** A beépített IP Address Management DHCP kiosztásához minden node-ra fel kell tenni a `dnsmasq` csomagot, majd az alapértelmezett démont le kell tiltani (a Proxmox zónánként indítja majd a sajátjait):

```bash
apt install dnsmasq
systemctl disable --now dnsmasq
```

- **FRRouting (Tech Preview – BGP/OSPF):** Az EVPN és Layer 3 routing használatához telepíteni kell az `frr-pythontools` csomagot, majd engedélyezni:

```bash
apt install frr-pythontools
systemctl enable frr.service
```

**Működési elv:** A konfigurációk a `/etc/pve/sdn` mappában tárolódnak. A GUI-n végzett módosítások először **"függőben lévő" (pending)** állapotba kerülnek, és csak az **"Apply"** gomb megnyomásával válnak aktívvá (így atombiztosan, egyszerre érvényesül minden). Az aktuális állapotot a `.running-config` és a `.version` fájlok követik nyomon.

## 12.6 Zónák (Zones)

A zónák virtuálisan elszeparált hálózatok, amelyeket bizonyos csomópontokra (**Nodes**) és jogosultságokra lehet korlátozni. **Közös beállítások minden zónánál:** `Nodes` (mely gépeken legyen aktív), `IPAM` (IP kezelő, alapból `pve`), `DNS`, `ReverseDNS`, és `DNSZone` (pl. a VM hostnevének bejegyzéséhez a `<hostname>.<domain>` formátumban).

### Zóna típusok

**Simple (Egyszerű):**
Egy izolált, helyi híd (**Bridge**), ami nincs fizikai interfészhez kötve. A forgalom a node-on belül marad. NAT vagy routed beállításokhoz.

**VLAN:**
802.1Q taggeléssel különíti el a szegmenseket. Olyan meglévő helyi (fizikai) Linux vagy OVS hídra (**Bridge**) kell kötni, ami biztosítja a node-ok közötti kapcsolatot.

**QinQ (VLAN Stacking):**
Duplán címkézett VLAN, főleg szolgáltatóknak. Szükség van egy **Service VLAN** azonosítóra, és egy protokollra (802.1q vagy 802.1ad).
> **Vigyázat:** A dupla fejléc miatt a fizikai MTU-t legalább 4 bájttal csökkenteni kell (pl. 1500 $\rightarrow$ 1496).

**VXLAN:**
Layer 2 hálózat egy IP-hálózat (**underlay**) felett, UDP 4789-es porton kapszulázva. **Nincs titkosítás** (VPN kell alá)! Cél IP-címek listája (**Peers Address List**) vagy **SDN Fabric** szükséges hozzá. A kapszulázás miatt az MTU-t 50 bájttal kell csökkenteni.

**EVPN (BGP EVPN):**
Képes több datacenteren átívelő, irányított Layer 3 hálózatot (BGP-vel) felhúzni. A VNet-eknek lehet **"anycast"** IP/MAC címe (minden node-on ugyanaz az átjáró IP címe van). Az EVPN zóna megkövetel egy **Controller**-t (FRR), és opcionálisan beállíthatók:

- **VRF VXLAN ID** (útválasztási interconnect),
- **Exit Nodes** (kilépési pontok a valós hálózat felé),
- **Primary Exit Node** (kötelező SNAT vagy nem ECMP-képes upstream router esetén),
- **Exit Nodes Local Routing**,
- **Advertise Subnets** (teljes alhálózatok BGP bemondása "csendes" gépek miatt),
- **Disable ARP ND Suppression** (lebegő IP-k (**floating IPs**) használatához elengedhetetlen),
- **Route-target Import**.

## 12.7 VNet-ek és 12.8 Alhálózatok (Subnets)

**VNet:** Egy virtuális hálózat a zónán belül (létrehozás után egyező nevű lokális interfész jelenik meg a hostokon). **Beállítások:** `ID` (max 8 karakter), `Alias`, `Zone`, `Tag` (VLAN/VXLAN azonosító), `VLAN Aware` (a vendég VM maga is taggelhet), és `Isolate Ports` (a vendégek csak a híddal tudnak kommunikálni, egymással nem – ez csak a lokális gépen belül érvényes, a teljes hálózati izolációhoz a **VNET Firewall** szabályzata kell).

**Subnet:** IP tartomány (pl. `10.0.0.0/8`). **Beállításai:** `ID` (a CIDR cím), `Gateway` (az átjáró IP-je, EVPN és Simple esetén fizikailag is települ a VNet-re), `SNAT` (Source NAT engedélyezése kifelé), és `DNS Zone Prefix`.

## 12.9 Vezérlők (Controllers)

A **Control Plane** és a **Data Plane** szétválasztásához (jelenleg csak az EVPN használja).

**EVPN Controller:**
BGP (FRR) alapon működik. Kötelező az **ASN #** (BGP Autonóm Rendszer Szám, privát ajánlott: `64512`–`65534`). Megadható `Peers` vagy `SDN Fabric`. Extra opciók: `Peer Group Name` (alapból `VTEP`, de több controllernél kötelező újat megadni), `BGP Mode` (`iBGP`, `eBGP`, vagy `auto`), `ebgp-multihop`. A **VTEP IP** generálása automatikus: ha a `loopback` interfész létezik, onnan veszi, ha nem, a default route-ból származtatja.

**BGP Controller:**
Opcionális, eBGP vagy útvonal exportálásához. Opciók: `Node`, `ASN #`, `Peer`, `EBGP`, `Loopback Interface`, `ebgp-multihop`, és `bgp-multipath-as-path-relax` (ECMP eltérő ASN esetén).

**ISIS Controller:**
EVPN útvonalak ISIS tartományba exportálására. Opciók: `Node`, `Domain`, `Network Entity Title`, `Interfaces`, `Loopback`.

## 12.10 Hálózati Szövetek (Fabrics)

Automatikusan konfigurálják az **underlay** hálózati útválasztást a gépek (csomópontok) között, megteremtve a VXLAN vagy EVPN alapját. Telepíteni kell hozzá a `frr` és `frr-pythontools` csomagokat. Létrehozásukhoz és módosításukhoz `SDN.Allocate` (és a fizikai `interfaces` fájl írása miatt `Sys.Modify`) jogosultság szükséges. Minden Fabric létrehoz egy rejtett, passzív **"dummy"** (`loopback`) interfészt a node-on a **Router-ID**-val (IP-címmel), és legenerál egy **RouteMap**-et (a kimenő csomagok forrás címének átírására).

### 12.10.4 OpenFabric

Kifejezetten adatparki (**spine-leaf**) hálózatokhoz tervezett (IS-IS alapú) protokoll.

- **Fabric szint:** `Name`, `IPv4 Prefix`, `IPv6 Prefix` (hitelesítésre szolgál, hogy minden gép ebből a tartományból van-e). `Hello Interval` (alap: 3 mp) és `CSNP Interval` (alap: 10 mp).

> **Vigyázat, IPv6:** IPv6 esetén kötelező minden gépen globálisan engedélyezni az IP-továbbítást, különben a hálózat nem működik (kivéve full-mesh esetén). Ehhez a `/etc/network/interfaces` fájlban egy interfészhez be kell írni:

```bash
post-up sysctl -w net.ipv6.conf.all.forwarding=1
```

(ez hatással van a SLAAC-re és az útválasztó hirdetésekre is). A **Network Entity Title (NET)** generálásakor az IPv6-nál az utolsó 3 szegmens adja az egyedi címet.

- **Node és Interface szint:** Meg kell adni az egyedi IP-t (ami a loopback címe lesz), ki kell választani a fizikai `Interfaces`-eket, és beállítható a `Hello Multiplier` (alap: 10).

### 12.10.5 OSPF

Tradicionális IP-útválasztó protokoll (**Link-State**).

**Beállítások:** `IPv4 Prefix`, `Area` (OSPF terület, 32 bites azonosító, pl. a `0` a gerinc/backbone). `Redistribute` (más protokollok, pl. `bgp`, `kernel`, `static` beemelése az OSPF-be Route Map segítségével). A fizikai interfészek, ha nem kapnak IP-t, automatikusan pont-pont (**point-to-point**) kapcsolatként viselkednek.

### 12.10.6 WireGuard

Titkosított VPN hálózat kiépítésére (Önmagában nem csinál dinamikus útválasztást, de ráfűzhető OSPF vagy BGP). Telepíteni kell a `wireguard-tools` csomagot.

- **Fabric szint:** `Name`, `Persistent Keepalive` (NAT/tűzfal áttöréséhez hasznos).
- **Belső (PVE) és Külső node-ok:** Mindegyik kap egy **Endpoint**-ot (IP vagy hosztnév, IPv6 esetén szögletes zárójelben pl. `[2001:db8::1]:51820`), egy **Listen Port**-ot és egy **Allowed IPs** listát (CIDR formátumban). A privát és publikus kulcsokat a PVE automatikusan generálja a `/etc/pve/priv/wg-keys.cfg` fájlba.
- A **Skip Route Generation** pipával megakadályozható, hogy a PVE automatikusan beleírja a kernel routing táblájába a társak Allowed IPs tartományait.

## 12.11 Prefix Listák és Route Map-ek

Ezek szabályozzák (szűrik és módosítják), hogy milyen hálózati útvonalakat (**routes**) cserélnek ki a gépek a BGP, OSPF vagy OpenFabric társakkal. A Proxmox a `/etc/pve/sdn/` mappában tárolja őket, majd az FRR-nek (`/etc/frr/frr.conf`) fordítja le.

**Prefix Listák:** Útvonalak engedélyezésére (`permit`) vagy tiltására (`deny`) CIDR alapján. Szabályok: `Sequence Nr.` (alapból 5-ös ugrásokkal sorszámoz, pl. 10, 15, 20), `Action`, `Prefix` (pl. `0.0.0.0/0`), `Prefix >=` (ge), `Prefix <=` (le). (Pl.: `10.0.0.0/8 ge 16 le 24` lefedi a /16 és /24 közötti alhálózatokat). Rezervált, tiltott nevek: `only_default`, `loopbacks_ips`.

**Route Map (Útvonaltérkép):** Módosíthatja az útvonal paramétereit (metrika, local-preference). Beállítások: `Order` (0-65535, a kiértékelés sorrendje), `Action` (`permit`/`deny`), `Match` (egyeztetés pl. prefix-lista, route-type, vni alapján), `Set` (módosítás, pl. `local-preference=50`), `Exit Action` (kiértékelés folytatása: megszakítás, `on-match-next`, `on-match-goto`, `continue`), és `Call` (egy másik Route Map hívása).

**BGP Példa:** Két Route Map van, az egyik a bejövő (**Incoming**), a másik a kimenő (**Outgoing**) BGP forgalmat szűri.

### 12.11.5 BGP Szövet (Fabric)

A BGP eBGP **"unnumbered"** (IP-cím nélküli fizikai linkek IPv6 link-lokális címmel) technológiát használ. Minden gépnek külön ASN-je van.

**Beállítások:** `IPv4 Prefix`, `IPv6 Prefix`, `BFD` (kapcsolati hiba ultra-gyors észlelése), `Route Filter`, `Incoming/Outgoing Route Map`, `Redistribute`.

**Node opciók:** Minden gépnek kell saját `ASN` és egy `IPv4` (vagy `IPv6`) cím.

**EVPN és BGP kombinálása:** Ha BGP-t használsz EVPN alaphálózatként (**underlay**), és az EVPN Controller `"auto"` módú (iBGP), az EVPN (**overlay**) ASN-jének különböznie kell a fizikai gépek BGP ASN-jétől (pl. gépek: 65001, 65002; EVPN: 65000). Ha `"external"` (eBGP) módú a VTEP, akkor használható ugyanaz.

## 12.12 – 12.14 IPAM, DNS és automatikus DHCP

- **PVE IPAM (Beépített):** Alapértelmezett. A GUI-n (**Datacenter $\rightarrow$ SDN $\rightarrow$ IPAM**) manuálisan is hozzáadhatók/szerkeszthetők VM-ek IP-címei.
  > **Tipp:** Ha egy DHCP-t használó VM IP-jét megváltoztatod az IPAM-ban, utána a gépben újra kell indítani a hálózati stacket a frissítéshez.
- **Külső IPAM Pluginok:** **NetBox** (meg kell adni az URL-t, API Tokent és az API SSL tanúsítvány SHA-256 Fingerprintjét) vagy **phpIPAM** (URL, Token és Section ID, ami alapból az 1-es).
- **DNS Plugin (PowerDNS):** `api=yes`, `webserver=yes` a PowerDNS-ben. PVE-ben URL, Key és TTL beállítása.
- **DHCP (`dnsmasq`):** Csak **"Simple Zone"**-okon támogatott jelenleg. Bekapcsolás a Zone "Advanced" fülén, majd a Subnet beállításánál egy **DHCP Range** és egy **Gateway** (átjáró) megadása kötelező. A démon konfigurációs fájljait (`/etc/dnsmasq.d/<zone>/00-default.conf`, `ethers`) a Proxmox generálja.
  > **Tilos ezeket a fájlokat szerkeszteni!** Ha módosítani akarsz, csinálj egy saját pl. `90-custom.conf` fájlt a mappában.

## 12.15 Tűzfal Integráció

Az SDN automatikusan dinamikus IP-készleteket (**IPSet**) generál a tűzfal számára, így nem kell kézzel karbantartani a szabályokat.

- **VNet IPSet-ek:** `vnet-all` (a teljes alhálózat), `vnet-gateway` (csak az átjáró IP-je), `vnet-no-gateway` (a hálózat az átjáró NÉLKÜL), `vnet-dhcp` (a DHCP tartomány IP-jei).
- **IPAM IPSet:** A PVE IPAM által kiosztott címek a `guest-ipam-<vmid>` készletbe kerülnek.

> **Veszély:** Ha egy géptől megvonsz minden IP-címet az IPAM-ban, miközben egy tűzfalszabály hivatkozik a `guest-ipam-<vmid>` IPSetre, a **tűzfal összeomlik**, mert nem tud betölteni egy nem létező/üres IPSet-et.

## 12.16 Teljes Példakonfigurációk (Összefoglaló)

- **Simple Zone Source NAT-tal:** Ha a Subnet beállításnál bekapcsolod az `SNAT`-ot és megadsz egy `Gateway`-t (pl. `172.16.0.1`), a Proxmox gép NAT átjáróként (**router**) fog viselkedni a VNet-re kötött VM-ek számára a külvilág felé.
- **QinQ példa:** 1-es ügyfél Zónája `Service VLAN: 20`, 2-es ügyfél Zónája `Service VLAN: 30`. Mindkét ügyfél létrehoz egy VNet-et (`Belső VLAN Tag: 100`). Bár a VNet Tag ugyanaz, a dupla tokozás miatt a két ügyfél 100-as VLAN-ja fizikailag soha nem kommunikálhat egymással.
- **VXLAN / EVPN példa:** Két VM két külön hoston (`node1`, `node3`). Mindkettő ugyanazt a `/24`-es IP tartományt használja, és a VXLAN Zone/EVPN Fabric transzparensen (L2 alagúton) átviszi a forgalmat a gépek fizikai IP-hálózatán keresztül.
