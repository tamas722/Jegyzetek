# 3. Fejezet: Hoszt Rendszeradminisztráció

A Proxmox VE egy módosított Ubuntu kernellel futó Debian GNU/Linux rendszer. A legtöbb szabványos Debian eszköz és konfiguráció érvényes, de a Proxmox saját eszközöket is biztosít a virtualizációs feladatokhoz.

## 3.1 és 3.2 Csomagtárolók (Repositories) és Frissítések

A Proxmox az **APT** csomagkezelőt használja. A konfigurációk a `/etc/apt/sources.list` fájlban (régi egysoros formátum) vagy a `/etc/apt/sources.list.d/` könyvtárban (új **deb822** többsoros formátum) találhatók.

- **PVE Enterprise Repository:** A produkciós környezetbe szánt, legstabilabb tároló. Használatához érvényes, architektúra-specifikus (x86 vagy arm64) előfizetési kulcs szükséges.
- **PVE No-Subscription Repository:** Ingyenes, tesztelésre szánt tároló. A csomagok ide hamarabb bekerülnek, de nincsenek olyan alaposan letesztelve.
- **Ceph Repositories:** Külön tárolók a Ceph csomagoknak (itt is van Enterprise, No-Subscription és Test ág).

**Rendszerfrissítés:** A csomagok frissítéséhez a CLI-ben a következőket kell használni (a sima `apt upgrade` kerülendő, mert nem tudja kezelni a bonyolult függőség-változásokat, amik a Proxmoxnál előfordulnak):

```bash
apt update
apt full-upgrade
```

## További APT csomagtárolók és SecureApt

### Proxmox VE Test Repository

Fejlesztőknek szánt, legújabb funkciókat tartalmazó teszt tároló (**produkciós környezetbe tilos**).

### Debug Symbol Repository

Hibakereséshez (`gdb`, `crash`) szükséges csomagokat tartalmazza (`<package>-dbgsym`), amik hasznosak lehetnek rendszerösszeomlás elemzésekor.

### Debian Firmware Repository

A `non-free-firmware` komponens, ami a hardveres firmware-eket és a CPU mikrokódokat tartalmazza.

### SecureApt

Az APT a **GnuPG aláírásokat** használja a csomagok hitelességének ellenőrzésére. Ha a Proxmoxot Debianra telepíted, a kulcsot manuálisan kell letölteni és ellenőrizni a következő parancsokkal:

```bash
wget https://enterprise.proxmox.com/debian/proxmox-archive-keyring-trixie.gpg -O /usr/share/keyrings/proxmox-archive-keyring.gpg
sha256sum /usr/share/keyrings/proxmox-archive-keyring.gpg
md5sum /usr/share/keyrings/proxmox-archive-keyring.gpg
```

## Időszinkronizáció (Time Synchronization)

A Proxmox VE és különösen a Ceph rendkívül érzékeny a pontos időre. A Proxmox VE 7-től a **`chrony`** az alapértelmezett NTP démon (korábban a `systemd-timesyncd` volt). Egyedi NTP szerverek beállításához a `/etc/chrony/chrony.conf` fájlba kell felvenni a szervereket (`server ntp1.example.com iburst`), majd újraindítani a démont (`systemctl restart chronyd`). A működés ellenőrzése a logokból: `journalctl --since 1h -u chrony`.


## 3.3 Firmware és Microcode Frissítések

A hardveres stabilitás érdekében a BIOS/UEFI és a firmware-ek (hálózati kártyák, RAID vezérlők) frissítése kritikus.

**CPU Microcode:** A Linux kernel már a bootolás korai szakaszában be tud tölteni biztonsági mikrokód-frissítéseket a processzorba. Ehhez engedélyezni kell a Debian `non-free-firmware` tárolóját, majd telepíteni az `intel-microcode` vagy `amd64-microcode` csomagot.

## 3.4 Hálózatkezelés

A hálózat konfigurációja a `/etc/network/interfaces` fájlban történik. Az `ifupdown2` csomag (amely alapértelmezett) lehetővé teszi a hálózati változtatások újraindítás nélküli, azonnali alkalmazását az `ifreload -a` paranccsal.

**Elnevezési konvenciók:** A fizikai portok `en*` (systemd séma) vagy `eth*` (régi) neveket kapnak, a hidak `vmbr*`, a bond-ok `bond*` neveken futnak. A nevek felülbírálhatók a `pve-network-interface-pinning` eszközzel.

- **Alapértelmezett híd (Bridged):** Úgy működik, mint egy fizikai switch. A `vmbr0` híd rácsatlakozik az `eno1` fizikai kártyára, és a VM-ek virtuális kártyái ide csatlakoznak.
- **Útválasztott (Routed) mód:** Tárhelyszolgáltatóknál hasznos, ahol egy fizikai kártyához csak egy MAC-címet engedélyeznek. A szerver `ip_forward` használatával irányítja át a forgalmat a belső `vmbr` híd felé.
- **Masquerading (NAT):** Ha csak egyetlen publikus IP-címed van, a VM-ek egy privát IP-tartományt kapnak, amit a gazdagép NAT-ol (`iptables -t nat -A POSTROUTING -j MASQUERADE`).
- **Linux Bond (Link Aggregation):** Több fizikai kártya összefogása sávszélesség-növelés vagy hibatűrés céljából. Ha a switch támogatja, a **802.3ad (LACP)** mód javasolt, egyébként az `active-backup`. A Proxmox cluster (**Corosync**) forgalmához LACP esetén kötelező a `bond-lacp-rate fast` beállítás.
- **VLAN (802.1Q):** Kétféleképpen működik. Vagy a hídnál bekapcsolod a `bridge-vlan-aware yes` opciót (transzparens VLAN), vagy "hagyományos" módon dedikált hidat csinálsz (pl. `vmbr0v5` a VLAN 5-nek).

## Külső metrika szerverek (External Metric Server)

A Proxmox képes a statisztikáit (CPU, RAM, hálózat, tárhely) folyamatosan külső szervereknek továbbítani.

**Támogatott rendszerek:**

- **Graphite** (alapértelmezett port: 2003, UDP/TCP)
- **InfluxDB** (alapértelmezett port: 8089, UDP vagy v2 esetén HTTP/HTTPS API tokenes hitelesítéssel)
- **Open Telemetry**

A konfiguráció a `/etc/pve/status.cfg` fájlban tárolódik.

## Lemezek állapotának monitorozása (Disk Health Monitoring)

A **`smartmontools`** csomag alapértelmezetten telepítve van. A `smartd` démon 30 percenként ellenőrzi a lemezeket (`/dev/sdX`, `/dev/nvmeX`), és hiba esetén e-mailt küld a `root` felhasználónak.

- Lekérdezés manuálisan: `smartctl -a /dev/sdX`
- SMART engedélyezése a lemezen: `smartctl -s on /dev/sdX`

## BTRFS fájlrendszer

A Proxmox támogatja a BTRFS-t (mint **technológiai előzetes / Technology Preview**). Funkcióiban hasonlít a ZFS-re (Copy-on-write, snapshotok, öngyógyítás checksumokkal, szoftveres RAID0/1/10 támogatás). A **RAID 5/6 szint kísérleti és veszélyes**, használata kerülendő.

A BTRFS tiszteletben tartja az `O_DIRECT` flaget, ezért a VM-eknél **nem szabad a `cache=none` beállítást használni**, különben checksum hibák léphetnek fel.

**Fájlrendszer létrehozása** (pl. szimpla lemezre vagy RAID1-be):

```bash
mkfs.btrfs -m single -d single -L My-Storage /dev/sdb
mkfs.btrfs -m raid1 -d raid1 -L My-Storage /dev/sdb1 /dev/sdc1
```

**Alkötetek (Subvolumes) listázása, létrehozása és törlése** (a törléshez nem kell üresnek lennie a mappának):

```bash
btrfs subvolume list /
btrfs subvolume create /some/path
btrfs subvolume delete /some/path
```

**Snapshot (pillanatkép) készítése:**

```bash
btrfs subvolume snapshot -r /some/path /a/new/path
```

**Valós helyfoglalás ellenőrzése** (a hagyományos `df` parancs fals értékeket mutathat BTRFS-nél):

```bash
btrfs fi usage /my-storage
```

## 3.8 LVM (Logical Volume Manager)

A lokális lemezek blokk-szintű kezelése.

- A normál **LVM** esetén a virtuális gépek formátuma `raw` vagy `qcow2`. Az LVM snapshotok (pillanatképek) teljesítmény-visszaesést okoznak, ezért jött be a **"Snapshot as a volume chain"** funkció, ami a `qcow2` rétegzését használja a probléma kiküszöbölésére.
- **LVM-thin:** Kifejezetten a "thin-provisioning" (csak a ténylegesen kiírt adat foglal helyet) és a hatékony snapshotok/klónozások miatt ajánlott lokális tároló.

Új thin-pool készítése:

```bash
lvcreate -L 100G -n data pve
lvconvert --type thin-pool pve/data
```

## 3.9 ZFS on Linux

Szoftveres RAID-et, kötetkezelést, tömörítést, bitrot elleni védelmet és snapshotokat biztosító fejlett fájlrendszer. **Hardveres RAID vezérlőn tilos használni.**

- **Memória limitálás (ARC):** A ZFS alapértelmezetten a RAM max $10\%$-át, vagy a PVE 8.1 előtti rendszereken akár $50\%$-át is felhasználhatja. A `/etc/modprobe.d/zfs.conf` fájlban a `zfs_arc_max` értékkel lehet bájtokban korlátozni a fogyasztását.
- **ZIL (Log) és L2ARC (Cache):** A teljesítmény gyors, **PLP**-vel (Power-Loss-Protection) ellátott Enterprise SSD-k hozzáadásával növelhető. ZIL hozzáadása szinkron írások gyorsítására: `zpool add <pool> log <eszköz>`. Cache a gyakori olvasásokhoz: `zpool add <pool> cache <eszköz>`.
- **Special Device:** Gyors SSD kötet (pl. RAID-1 mirror) hozzáadása a metaadatok, deduplikációs táblák és a pici fájlok tárolására, ami radikálisan gyorsítja a forgólemezes (HDD) ZFS poolokat.

Új pool létrehozása parancssorból (RAID-1 mirror példa):

```bash
zpool create -f -o ashift=12 <pool_neve> mirror /dev/sda /dev/sdb
```

## 3.11 Proxmox Node Menedzsment

A `pvenode` parancssori eszközzel lehet kezelni a gép-specifikus feladatokat.

- **Wake-on-LAN:** Alvó node-ok felébresztése a clusterből (`pvenode wakeonlan <node>`).
- **Tömeges műveletek:** Sok VM egyszerre történő indítása (`pvenode startall`), leállítása (`pvenode stopall`), vagy migrációja másik szerverre (`pvenode migrateall`).

## 3.12 Tanúsítványok (Certificates) és ACME

A webes GUI és az API HTTPS kapcsolaton fut. Alapértelmezetten a cluster saját maga által aláírt (**self-signed**) tanúsítványát használja.

**Let's Encrypt (ACME):** Ingyenes, hivatalosan megbízható tanúsítvány igényelhető a beépített ACME klienssel. Két validációs mód létezik:

- **HTTP-01:** A szerver 80-as portjának elérhetőnek kell lennie az internet felől.
- **DNS-01:** Nincs szükség bejövő portnyitásra. Az API-n keresztül (pl. Cloudflare, PowerDNS) beír egy TXT rekordot a domain zónájába, ezzel bizonyítva a tulajdonjogot.

## 3.13 Host Bootloader (Rendszertöltő)

A Proxmox VE telepítési módtól függően kétféle bootloadert használ.

- **GRUB:** Ext4, XFS rendszereken, BIOS/Legacy módban, illetve ZFS rendszereken, ha a Secure Boot engedélyezve van.
- **systemd-boot:** UEFI-s ZFS rendszereken (Secure Boot nélkül).
- **`proxmox-boot-tool`:** Egy eszköz, ami szinkronban tartja az EFI System Partíciókat (ESP) az összes lemezen (ZFS szoftveres RAID esetén), hogy bármelyik lemez meghibásodása esetén is bootolható maradjon a gép.
- **Secure Boot (Biztonságos rendszerindítás):** A Proxmox VE 8.1-től hivatalosan is támogatott. Ehhez a rendszer automatikusan a Microsoft által aláírt `shim-signed` és a Proxmox által aláírt GRUB csomagokat használja.

## 3.14 KSM (Kernel Samepage Merging)

Memória-deduplikációs funkció a Linux kernelben. Keresi az azonos tartalmú memória-lapokat (pl. ha 5 db Windows VM fut, ugyanazokat a rendszerfájlokat olvassák), és összevonja őket, jelentős RAM-ot spórolva. Biztonsági okokból (side-channel támadások ellen) szolgáltatói környezetben érdemes lehet kikapcsolni:

```bash
systemctl disable --now ksmtuned
```
