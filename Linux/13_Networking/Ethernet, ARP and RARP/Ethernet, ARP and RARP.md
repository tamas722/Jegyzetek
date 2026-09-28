# Hálózatkezelési alapok Linuxban

A hálózatkezelés döntő fontosságú szempont a Linux rendszerek kezelésében és használatában, akár egy egyszerű otthoni hálózatot építesz ki, szervereket kezelsz, vagy egy nagyobb vállalati hálózaton dolgozol. A Linux hálózatkezelés alapjainak megértése segít a hálózati kapcsolatok hatékony konfigurálásában, kezelésében és hibaelhárításában. Ez a cikk az alapvető hálózati koncepciókat, parancsokat és eszközöket tárgyalja Linux alatt.

## 1. Bevezetés a Linux hálózatkezelésbe

A Linux hálózatkezelés magában foglalja a hálózati interfészek konfigurálását és kezelését, az útválasztás (routing) megértését és beállítását, valamint a hálózati kapcsolatok hibaelhárításához és biztosításához szükséges különféle eszközök használatát. A Linux eszközök és segédprogramok széles skáláját kínálja ezen szempontok hatékony kezelésére.

## 2. Hálózati interfészek

A hálózati interfész az interakciós pont egy eszköz (például egy számítógép) és a hálózat között. Linuxban a hálózati interfészek lehetnek fizikaiak (pl. Ethernet kártyák) vagy virtuálisak (pl. loopback interfész).

**Hálózati interfészek megtekintése**

A Linux rendszeren elérhető hálózati interfészek megtekintéséhez használhatod az `ip` vagy az `ifconfig` parancsot.

```bash
ip link show
```

vagy

```bash
ifconfig -a
```

## 3. Hálózati interfészek konfigurálása

### Statikus IP konfiguráció

Egy statikus IP-cím beállításához szerkesztened kell a hálózati konfigurációs fájlokat. Debian-alapú rendszereken például az `/etc/network/interfaces` fájlt módosíthatod.

```bash
sudo nano /etc/network/interfaces
```

Add hozzá a következő konfigurációt a statikus IP-hez:

```plaintext
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

Mentsd el a fájlt, és indítsd újra a hálózati szolgáltatást:

```bash
sudo systemctl restart networking
```

### Dinamikus IP konfiguráció (DHCP)

A DHCP-t használó dinamikus IP-konfiguráció esetében a beállítás egyszerűbb. Az `/etc/network/interfaces` fájlban a következőknek kell szerepelnie:

```plaintext
auto eth0
iface eth0 inet dhcp
```

A módosítások elvégzése után ismét indítsd újra a hálózati szolgáltatást:

```bash
sudo systemctl restart networking
```

## 4. Hálózati konfigurációs fájlok

### `/etc/network/interfaces`

Ezt a fájlt elsősorban Debian-alapú rendszereken használják a hálózati interfészek konfigurálására.

### `/etc/netplan/`

Az újabb Ubuntu verziókon a Netplan használatos a hálózat beállítására. A konfigurációs fájlok az `/etc/netplan/` könyvtárban találhatók.

Példa Netplan konfigurációra:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: yes
```

Alkalmazd a változtatásokat a következővel:

```bash
sudo netplan apply
```

## 5. Alapvető hálózati parancsok

### `ifconfig`

Az `ifconfig` parancs a hálózati interfészek konfigurálására szolgál. Bár az `ip` parancs javára elavulttá nyilvánították, még mindig széles körben használják.

```bash
ifconfig eth0 up
ifconfig eth0 down
ifconfig eth0 192.168.1.100 netmask 255.255.255.0
```

### `ip`

Az `ip` parancs egy hatékony eszköz a hálózati interfészek, az útválasztás és az alagutak (tunnels) kezelésére.

```bash
ip addr show
ip addr add 192.168.1.100/24 dev eth0
ip link set eth0 up
ip link set eth0 down
```

### `ping`

A `ping` parancs ellenőrzi a kapcsolatot a helyi gép és egy távoli gazdagép között.

```bash
ping google.com
ping -c 4 google.com
```

### `traceroute`

A `traceroute` parancs megmutatja azt az útvonalat, amelyet a csomagok megtesznek, hogy elérjenek egy hálózati gazdagépet.

```bash
traceroute google.com
```

### `netstat`

A `netstat` parancs megjeleníti a hálózati kapcsolatokat, az útválasztási táblázatokat, az interfész statisztikákat, a maszkolt kapcsolatokat és a multicast tagságokat.

```bash
netstat -a
netstat -r
netstat -tuln
```

### `ss`

Az `ss` parancs a `netstat` modern helyettesítője.

```bash
ss -tuln
ss -a
```

## 6. DNS konfiguráció

A DNS (Domain Name System - Tartománynévrendszer) a domain neveket IP-címekké fordítja. A DNS feloldás konfigurációs fájlja az `/etc/resolv.conf`.

Példa:

```plaintext
nameserver 8.8.8.8
nameserver 8.8.4.4
```

A végleges változtatások elvégzéséhez konfiguráld a DNS-beállításokat a hálózati konfigurációs fájlokban, vagy használj olyan eszközöket, mint a `resolvconf`.

## 7. Útvonalak kezelése

Az útválasztás meghatározza, hogyan mozognak az adatcsomagok egyik hálózatból a másikba. Az útvonalakat az `ip route` paranccsal tekintheted meg és kezelheted.

**Az útválasztási tábla megtekintése**

```bash
ip route show
```

**Útvonal hozzáadása**

```bash
sudo ip route add 192.168.2.0/24 via 192.168.1.1 dev eth0
```

**Útvonal törlése**

```bash
sudo ip route del 192.168.2.0/24
```

## 8. Hálózati hibaelhárítás

**A `ping` használata**

A `ping` parancs a legalapvetőbb hálózati hibaelhárító eszköz, a kapcsolat tesztelésére használják.

**A `traceroute` használata**

A `traceroute` parancs segít azonosítani, hogy a csomagok hol vesznek el a cél felé vezető úton.

**Hálózati konfiguráció ellenőrzése**

Győződj meg arról, hogy a hálózati interfészek aktívak és megfelelően vannak konfigurálva az `ifconfig` vagy `ip` parancsok használatával.

**Szolgáltatások ellenőrzése**

Győződj meg arról, hogy a szükséges szolgáltatások (mint például az `sshd` az SSH-hoz) futnak és megfelelően vannak konfigurálva.

```bash
sudo systemctl status ssh
```

## 9. Fejlett hálózati eszközök

### `tcpdump`

A `tcpdump` egy erőteljes parancssori csomagelemző. Lehetővé teszi a hálózati csomagok rögzítését és megjelenítését.

```bash
sudo tcpdump -i eth0
sudo tcpdump -i eth0 port 80
```

### `wireshark`

A Wireshark egy grafikus felületű hálózati protokoll elemző. A hálózati forgalom részletes vizsgálatát biztosítja.

### `nmap`

Az `nmap` egy hálózatvizsgáló eszköz, amelyet számítógép-hálózatokon lévő gazdagépek és szolgáltatások felfedezésére használnak.

```bash
nmap -sP 192.168.1.0/24
nmap -sV 192.168.1.1
```

## 10. Legjobb gyakorlatok a hálózatbiztonság terén

- **Tartsd naprakészen a rendszeredet:** Rendszeresen alkalmazz frissítéseket és javításokat a sebezhetőségek kiküszöbölésére.
- **Használj erős jelszavakat:** Vezess be erős jelszóházirendeket, és használj olyan eszközöket, mint a `fail2ban` a brute-force (nyerserős) támadások elleni védelemhez.
- **Korlátozd a nyitott portokat:** Csak a szükséges portokat és szolgáltatásokat nyisd meg. Használj tűzfalakat, mint például az `ufw` vagy az `iptables` a forgalom szabályozására.
- **Titkosítsd a kommunikációt:** Használj titkosítási protokollokat, mint az SSH a biztonságos távoli hozzáféréshez.
- **Kísérd figyelemmel a hálózati forgalmat:** Használj olyan eszközöket, mint a `tcpdump`, a `wireshark`, és monitorozó rendszereket, mint a Nagios a hálózati tevékenység megfigyelésére.
- **Tiltsd le a nem használt szolgáltatásokat:** Kapcsold ki azokat a szolgáltatásokat, amelyek nincsenek használatban, hogy csökkentsd a támadási felületet.
- **Rendszeres auditok:** Végezz rendszeres biztonsági ellenőrzéseket a sebezhetőségek azonosítása és javítása érdekében.

## 11. Összegzés

A hálózatkezelés a Linux rendszerek használatának és kezelésének szerves része. A hálózati interfészek, útvonalak és jogosultságok konfigurálásának és kezelésének megértése elengedhetetlen minden Linux felhasználó vagy rendszergazda számára. Az ebben a cikkben tárgyalt parancsokkal és eszközökkel jól felszerelt leszel a Linux alapvető és haladó hálózati feladatainak kezelésére.

## Parancsok és fogalmak összefoglalása

```bash
# Hálózati interfészek megtekintése
ip link show
ifconfig -a

# Statikus IP-cím konfigurálása
sudo nano /etc/network/interfaces
sudo systemctl restart networking

# Dinamikus IP konfiguráció (DHCP)
sudo nano /etc/network/interfaces
sudo systemctl restart networking

# DNS konfiguráció megtekintése
cat /etc/resolv.conf

# Útvonal hozzáadása
sudo ip route add 192.168.2.0/24 via 192.168.1.1 dev eth0

# Útvonal törlése
sudo ip route del 192.168.2.0/24

# Alapvető hálózati parancsok
ping google.com
traceroute google.com
netstat -a
ss -tuln

# Fejlett hálózati eszközök
sudo tcpdump -i eth0
nmap -sP 192.168.1.0/24
```

## Forrás
-[https://dev.to/iaadidev/a-beginners-guide-to-linux-networking-fundamentals-dev-ops-prerequisite-7-434o]

# Hálózatkezelés Alapjai Linuxon (Linux Networking Basics)

A Linux robusztus hálózati maggal (networking core) rendelkezik, amelynek köszönhetően a világ szervereinek jelentős része ezen az operációs rendszeren fut. Nyílt forráskódú lévén rendkívül rugalmasan konfigurálható szerverként, kliensként vagy akár útválasztóként (router) is.

## 1. Alapfogalmak

### Hálózati Interfész (Network Interface)

A kapcsolódási pont a számítógép és a hálózat között. Két fő típusa van:

- **Hardveres (NIC - Network Interface Card):** Fizikai komponens, például az Ethernet hálózati kártya vagy a Wi-Fi adapter.

- **Szoftveres (Virtuális):** Fizikai hardver nélküli, szoftveresen létrehozott interfész. A legismertebb a loopback (`lo`), amely a gépen belüli (belső) kommunikációra és tesztelésre szolgál külső hálózat nélkül.

### DNS (Domain Name System)

Az internet "telefonkönyve". A számítógépek IP-címekkel kommunikálnak (pl. `172.217.160.133`), de a felhasználók számára könnyebben megjegyezhető domain neveket (pl. `google.com`) használunk. A DNS a háttérben elvégzi a nevek és az IP-címek közötti fordítást (névfeloldást).

### Hálózati Szolgáltatás (Network Service)

Olyan szoftver, amely a hálózaton keresztül valamilyen funkciót biztosít (pl. webszerverek, e-mail szolgáltatások, fájlátvitel). Többnyire az alkalmazási rétegben (OSI 7. réteg) működnek, kliens-szerver vagy peer-to-peer (P2P) modell alapján.

### Tűzfal (Firewall)

Biztonsági kapuőr a belső (megbízható) hálózat és a külső (nem megbízható, pl. internet) hálózat között. Előre meghatározott szabályok alapján elemzi és engedélyezi vagy blokkolja a bejövő és kimenő adatcsomagokat.

## 2. Hálózati Konfiguráció és Diagnosztikai Parancsok

> **Megjegyzés:** A modern rendszereken az `ifconfig` és `netstat` helyett gyakran az `ip` és `ss` parancsokat használják, de a régi eszközök ismerete még mindig alapkövetelmény a hibaelhárításban.

### Interfészek kezelése (`ifconfig`)

Az `ifconfig` parancs listázza az elérhető hálózati interfészeket, és segítségével (ideiglenesen) IP-címet is oszthatunk.

```bash
# Interfészek listázása
ifconfig

# IP-cím és alhálózati maszk beállítása az 'eth0' interfészen
sudo ifconfig eth0 192.168.1.2 netmask 255.255.255.0
```

### Kapcsolat tesztelése (`ping`)

A hálózati kapcsolat ellenőrzésének legelső lépése. Csomagokat küld a célnak, és a válasz alapján méri a késleltetést (latency) és az esetleges csomagvesztést.

```bash
ping google.com
```

### Nyitott Portok és Szolgáltatások (`netstat`)

A hálózati szolgáltatások specifikus portokon kommunikálnak. A `netstat` segít azonosítani a nyitott portokat, a figyelő (listening) szolgáltatásokat és az aktív kapcsolatokat.

```bash
# TCP és UDP figyelő portok listázása numerikus formátumban
netstat -tuln
```

### Névfeloldás (DNS) tesztelése (`nslookup`)

A DNS szerverek lekérdezésére szolgál. Segítségével ellenőrizhető egy adott domain névhez tartozó IP-cím, vagy kideríthető, ha hibás a DNS konfiguráció.

```bash
nslookup example.com
```

### Tűzfalszabályok kezelése (`iptables`)

Az `iptables` a Linux hagyományos tűzfal-konfigurációs eszköze.

```bash
# Példa: A bejövő SSH forgalom (TCP 22-es port) engedélyezése
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

### Hálózati forgalom elemzése (`tcpdump`)

Erőteljes eszköz a hálózati csomagok valós idejű rögzítésére (packet sniffing) és elemzésére. Nélkülözhetetlen a mélyreható hálózati hibaelhárításhoz.

```bash
# A forgalom rögzítése dedikáltan az 'eth0' interfészen
sudo tcpdump -i eth0
```

### Hálózati útvonal felderítése (`traceroute`)

Megmutatja azt a teljes útvonalat (minden köztes routert / hop-ot), amelyet az adatcsomagok bejárnak a rendszeredtől a célig. Segít megtalálni, hol akad el a forgalom.

```bash
traceroute google.com
```

### Konfigurációs fájlok (Debian/Ubuntu alapokon)

A Linux a hálózati beállításokat specifikus fájlokban tárolja. A hagyományos rendszereken az interfészek statikus beállításait (IP, átjáró, DNS) az `/etc/network/interfaces` fájlban lehet megadni.

```bash
# A hálózati konfigurációs fájl szerkesztése
sudo nano /etc/network/interfaces
```

## Forrás
-[https://centlinux.com/linux-networking/]
