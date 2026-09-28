# IP Címek és Útválasztás (Routing) Konfigurálása Linuxon

A Linux hálózatkezeléséhez ma már két fő eszközkészlet áll rendelkezésre. Míg a hagyományos eszközök (mint az `ifconfig` vagy a `route`) még mindig használatban vannak, a modern disztribúciók áttértek az `ip` parancscsalád, valamint a NetworkManager (`nmcli`, `nmtui`) és a `systemd-networkd` használatára.

## 1. Az `ip` Parancs Használata (Modern CLI Standard)

Az `ip` parancs az `iproute2` csomag része, amellyel ideiglenes (újraindításig élő) hálózati beállításokat végezhetünk.

**IP-címek kezelése:**

- Aktuális konfiguráció listázása:

  ```bash
  ip addr show
  ```

- Statikus IP-cím hozzáadása egy interfészhez (pl. `eth0`):

  ```bash
  sudo ip addr add 192.168.1.10/24 dev eth0
  ```

- IP-cím törlése egy interfészről:

  ```bash
  sudo ip addr del 192.168.1.10/24 dev eth0
  ```

**Útválasztás (Routing) kezelése:**

A routing tábla (útválasztási táblázat) határozza meg, hogy a hálózati forgalom merre induljon el.

- A routing tábla megjelenítése:

  ```bash
  ip route show
  ```

- Új útvonal hozzáadása egy adott hálózathoz (egy specifikus átjárón keresztül):

  ```bash
  sudo ip route add 192.168.2.0/24 via 192.168.1.1
  ```

- Útvonal törlése:

  ```bash
  sudo ip route del 192.168.2.0/24
  ```

- Alapértelmezett átjáró (Default Gateway) beállítása:

  ```bash
  sudo ip route add default via 192.168.1.1
  ```

## 2. A NetworkManager használata (`nmcli`)

Sok modern Linux disztribúció (pl. Ubuntu, RHEL/CentOS) a NetworkManager-t használja a hálózatok kezelésére. Ennek parancssori eszköze az `nmcli`, amellyel állandó (perzisztens) beállításokat hozhatunk létre.

**IP és kapcsolatok kezelése:**

- Aktív és elérhető kapcsolatok listázása:

  ```bash
  nmcli connection show
  ```

- Statikus IP-cím beállítása egy adott kapcsolaton (pl. "Connection Name"):

  ```bash
  sudo nmcli con mod "Connection Name" ipv4.addresses "192.168.1.10/24" ipv4.method manual
  ```

- A kapcsolat aktiválása (újraindítása a beállítások érvényesítéséhez):

  ```bash
  sudo nmcli con up "Connection Name"
  ```

**Útválasztás ellenőrzése:**

- Egy adott kapcsolat routing szabályainak lekérdezése:

  ```bash
  nmcli connection show "Connection Name" | grep route
  ```

## 3. Dinamikus IP-kiosztás (DHCP) Kérése

Ha a hálózaton működik DHCP szerver, de a gép valamiért nem kapott IP-címet, manuálisan is kikényszeríthetjük az IP-cím lekérését a `dhclient` parancs segítségével (ez elindítja a DORA folyamatot).

```bash
# DHCP bérlet igénylése az eth0 interfészen
sudo dhclient eth0
```

## 4. Hibaelhárítási és Biztonsági Alapok

- **IPv6 Link-Local Címek:** Az IPv4 mellett az IPv6 is egyre elterjedtebb. Fontos tudni, hogy az IPv6 minden interfészen automatikusan létrehoz egy úgynevezett link-local címet (`fe80::/10`), amelyet a helyi hálózaton belüli kommunikációra használ, még DHCP szerver nélkül is.

- **Alapvető Hibaelhárítók:**
  - `ping`: Elérhetőség tesztelése.
  - `traceroute`: A csomagok útvonalának feltérképezése.
  - `netstat` (vagy `ss`): Hálózati kapcsolatok és routing táblák vizsgálata.

- **Hálózatvédelem (Tűzfalak):** A hálózat védelmére a komplex `iptables` / `nftables` mellett számos disztribúción a felhasználóbarátabb `ufw` (Uncomplicated Firewall) is használható a portok és a forgalom egyszerű engedélyezésére vagy tiltására.

- ## Forrás
- -[https://www.linuxjournal.com/content/linux-networking-simplified-guide-ip-addresses-and-routing]

- # Útválasztás (Routing) Beállítása Linuxon az `ip` Paranccsal

A modern Linux rendszereken az `ip` parancs (az `iproute2` csomag része) a hálózati beállítások (IP-címek, interfészek, útválasztási táblák) kezelésének elsődleges eszköze, felváltva a régi `route` és `ifconfig` parancsokat.

> ⚠️ **Figyelmeztetés:** A kernel útválasztási táblájának módosítása a hálózati kapcsolat elvesztéséhez vezethet, különösen, ha távolról (SSH-n keresztül) vagy bejelentkezve!

## 1. A Rendszer Állapotának Lekérdezése

Mielőtt útvonalat módosítanánk, fontos ismerni a jelenlegi hálózati interfészeket és a routing táblát.

**Hálózati interfészek listázása:**

```bash
ip link show
# Vagy NetworkManager esetén: nmcli device status
```

**Az útválasztási tábla (Routing Table) megtekintése:**

```bash
ip route show
# vagy: ip route list
```

(Egy minta sor magyarázata: `192.168.1.0/24 dev ra0 proto kernel scope link src 192.168.1.106` - Ez azt jelenti, hogy a `192.168.1.0`-s hálózat felé a forgalom közvetlenül az `ra0` interfészen megy ki).

## 2. Útvonalak Hozzáadása és Törlése (Ideiglenes beállítások)

Az parancssorban kiadott `ip` parancsok csak a következő újraindításig (reboot) érvényesek!

**Alapszintaxis:**

```bash
ip route add <hálózat>/<maszk> via <átjáró> dev <interfész>
```

**Közvetlenül csatlakoztatott hálózat hozzáadása:**

Ha minden csomagot egyenesen egy interfészen (pl. `eth0`) akarunk kiküldeni egy helyi hálózat felé:

```bash
sudo ip route add 192.168.1.0/24 dev eth0
```

**Statikus útvonal hozzáadása egy specifikus átjárón (gateway) keresztül:**

Ha egy másik hálózatot csak egy bizonyos routeren keresztül érünk el:

```bash
sudo ip route add 192.168.2.0/24 via 192.168.2.254 dev eth0
```

**Alapértelmezett átjáró (Default Gateway) beállítása:**

Azon csomagok, amelyeknek a célja nincs benne a routing táblában, ide fognak kerülni.

```bash
sudo ip route add default via 192.168.1.254
```

**Útvonalak törlése:**

Az útvonal törléséhez az `add` szót egyszerűen `delete`-re cseréljük:

```bash
sudo ip route delete 192.168.1.0/24 dev eth0
sudo ip route delete default
```

## 3. Útválasztás Tesztelése és Ellenőrzése

Miután beállítottad az útvonalakat, a hálózati kapcsolatot tesztelni kell.

```bash
# Gateway vagy célgép elérhetőségének tesztelése
ping 192.168.1.254
ping google.com
```

**Hogyan érek el egy adott IP-t?**

Az `ip route get` parancs pontosan megmutatja, hogy a kernel melyik interfészt és átjárót választja egy konkrét cél IP eléréséhez:

```bash
ip route get 1.1.1.1
ip route get 10.83.200.242
```

## 4. Állandó (Perzisztens) Útválasztási Beállítások

Ahhoz, hogy az útválasztási szabályok túléljék az újraindítást, a beállításokat a disztribúciónak megfelelő konfigurációs fájlokba kell menteni.

### A) Ubuntu / Debian (Régebbi rendszerek) - `/etc/network/interfaces`

Szerkeszd a fájlt (pl. `nano /etc/network/interfaces`):

```plaintext
auto eth0
iface eth0 inet static
    address 192.168.2.24
    netmask 255.255.255.0
    gateway 192.168.2.254
```

### B) Ubuntu (Modern rendszerek: 18.04, 20.04, 22.04 LTS) - Netplan

A modern Ubuntuk YAML alapú Netplan konfigurációt használnak (általában `/etc/netplan/01-netcfg.yaml` vagy hasonló néven).

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.2.24/24
      gateway4: 192.168.2.254
      nameservers:
        addresses: [192.168.2.236, 192.168.2.237]
```

A beállítások érvényesítéséhez futtasd:

```bash
sudo netplan apply
```

### C) CentOS / RHEL / Rocky / Alma Linux

Hozd létre vagy szerkeszd az `/etc/sysconfig/network-scripts/route-eth0` fájlt (ahol az `eth0` az interfész neve), és add hozzá a statikus útvonalat:

```plaintext
10.105.28.0/24 via 10.105.28.1 dev eth0
```

Mentés után indítsd újra a hálózati szolgáltatást:

```bash
sudo systemctl restart network
```
## Forrás
-[https://www.cyberciti.biz/faq/howto-linux-configuring-default-route-with-ipcommand/]
