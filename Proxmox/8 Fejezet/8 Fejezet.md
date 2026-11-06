# 8. Fejezet: Hiperkonvergens Ceph Cluster (HCI) Telepítése és Üzemeltetése

A Ceph egy nyílt forráskódú, elosztott objektum-, blokk- és fájltároló rendszer. A Proxmox VE-be integrálva megszünteti a dedikált SAN/NAS iránti igényt, mivel ugyanaz a hardver futtatja a VM-eket és tárolja a redundáns adatokat.

## 8.2 Terminológia (A Ceph démonjai)

- **MON (Monitor):** A cluster térképének (`monmap`, `osdmap`) mesterpéldányát tárolja.
- **MGR (Manager):** A monitorok mellett fut, a cluster metrikáit, a PG-autoskálázást és a telemetriát kezeli.
- **OSD (Object Storage Daemon):** Maga a fizikai lemez és a rajta futó démon, ami az adatokat tárolja és replikálja.
- **MDS (Metadata Service):** Kizárólag a POSIX-kompatibilis CephFS (fájlrendszer) használatához szükséges.

## 8.3 Hardverajánlások egy egészséges Ceph-hez

A Ceph rendkívül hardverigényes. Minimálisan **3** (lehetőleg azonos) fizikai szerver kell hozzá.

- **CPU:** A MON és MGR közepes, az OSD és az MDS magas CPU-igényű. Ökölszabály: minden Ceph szolgáltatásnak dedikálj legalább **1 CPU magot/szálat**. Egy extrém gyors NVMe SSD-re épülő OSD akár **4-6 CPU szálat** is felemészthet!
- **Memória:** Egy OSD démon alapból **4 GiB**-ot kér, de a jó teljesítményhez **8 GiB RAM / OSD** az ajánlott. Mindig hagyj tartalékot (**headroom**) a RAM-ban, mert rebalance/recovery (helyreállítás) esetén a Ceph memóriafogyasztása megugrik.
- **Hálózat:** Kritikus pont! A Ceph hálózatot **kötelező fizikailag leválasztani** a Proxmox Corosync hálózatáról.
  - HDD-k esetén minimum dedikált **10 Gbps**.
  - NVMe SSD-k esetén **25, 40 vagy 100+ Gbps** szükséges, különben a hálózat lesz a szűk keresztmetszet.
  - Ajánlott felosztás nagy teljesítménynél: 1 hálózat a Ceph belső replikációnak (**Cluster net**), 1 a publikus VM forgalomnak (**Public net**), 1 a Corosync-nek (**1 Gbps**).
- **Lemezek:** **SOHA ne használj hardveres RAID vezérlőt** (**HBA kártya** kell). A HDD-k helyett a kis clusterekben SSD-k ajánlottak, mert egy lemezhiba után a helyreállítási idő (**recovery time**) drasztikusan lerövidül.

## 8.4 Telepítés és Kezdeti Konfiguráció

- **GUI:** A **Ceph** menüpont alatt egy varázsló letölti a csomagokat és beállítja a **Public Network** (publikus hálózat a VM-ek felé) és opcionálisan a **Cluster Network** (OSD replikációs hálózat) címét.
- **CLI megoldás:**

```bash
pveceph install
pveceph init --network 10.10.10.0/24
```

Ez legenerálja a `/etc/pve/ceph.conf` fájlt (amely azonnal replikálódik a **pmxcfs**-en keresztül minden node-ra), és létrehoz egy symlinket a `/etc/ceph/ceph.conf` útvonalon.

## 8.5 és 8.6 Monitorok (MON) és Managerek (MGR)

**MON:** Pontosan **3 darab** ajánlott a quorumhoz.

- Létrehozás: `pveceph mon create`
- Törlés: `pveceph mon destroy`

**MGR:** A Luminous verzió óta legalább **1 kötelező**, de HA céljából érdemes a 3 MON node mindegyikére tenni egyet (mindig csak 1 lesz az "aktív").

- Létrehozás: `pveceph mgr create`
- Törlés: `pveceph mgr destroy`

## 8.7 Ceph OSD-k kezelése

Minden fizikai lemezhez egy OSD tartozik. A modern Ceph a **Bluestore** formátumot használja (a régi Filestore már elavult).

**Létrehozás:**

```bash
pveceph osd create /dev/sd[X]
```

**DB és WAL szétválasztása (Gyorsítás):** HDD-k esetén hatalmas gyorsulást hoz, ha a metaadatokat (**DB**) és a naplót (**WAL**) egy gyors NVMe/SSD meghajtóra teszed:

```bash
pveceph osd create /dev/sd[X] -db_dev /dev/sd[Y] -wal_dev /dev/sd[Z]
```

**Lemez nullázása (ZAP – VIGYÁZAT, ADATVESZTÉS!):** Ha a lemez korábban ZFS vagy OSD volt:

```bash
ceph-volume lvm zap /dev/sd[X] --destroy
```

**OSD törlése (Biztonságos sorrend):**

1. Ellenőrizd a helyet: `ceph osd df tree` (a többi OSD ne lépje át a **85%-os** `nearfull_ratio`-t).
2. Vedd ki a disztribúcióból: `ceph osd out <ID>`. Várj, amíg a rebalance befejeződik (0 PG marad rajta).
3. Biztonságos-e leállítani? `ceph osd ok-to-stop <ID>`. Ha igen: `pveceph stop --service osd.<ID>`.
4. Végleges törlés: `pveceph osd destroy <ID> [--cleanup]`.

## 8.8 Ceph Konfiguráció (CLI)

A beállítások nagy része már a MON-ok központi adatbázisában él, nem a `ceph.conf`-ban.

- Lekérdezés: `ceph config dump` vagy `ceph config get <entity>`
- Módosítás: `ceph config set <entity> <opció> <érték>` (pl. `ceph config set osd osd_memory_target 8G`)
- Törlés: `ceph config rm <entity> <opció>`

## 8.9 Ceph Poolok (Tárolókészletek)

A Pool-ok tartalmazzák a PG-ket (**Placement Groups**). Alapértelmezés: **128 PG**, `size=3` (3 kópia), `min_size=2`.

> **KRITIKUS FIGYELMEZTETÉS:** **SOHA ne állítsd a `min_size`-t 1-re!** Ha csak 1 kópia van meg, és az OSD meghal, azonnali és visszafordíthatatlan adatvesztést szenvedsz.

- **Létrehozás:** `pveceph pool create <név> --add_storages`
- **PG Autoscaler:** A Ceph automatikusan tudja skálázni a PG-k számát a betett adatmennyiség alapján. Módjai: `warn` (csak figyelmeztet), `on` (automatikusan átméretez), `off`. (Tipp: Az autoscaler 3-szoros eltérés felett kezd skálázni, ami óriási hálózati I/O terhelést generálhat, ezért érdemes a **Target Size** paramétert előre megadni).

**Erasure Coded (EC) Poolok (Hideg adatokhoz / Mentésekhez):**
Helytakarékosabb, mint a replikáció, de lassabb. Úgy működik, mint a RAID5/6 a hálózaton ($k$ adatdarab + $m$ paritás).

- **Létrehozás:** `pveceph pool create <név> --erasure-coding k=2,m=1`
- **FastEC (Technológiai előzetes optimalizáció):** A kis, véletlenszerű I/O-k (VM-ek) gyorsítására. Engedélyezése meglévő EC poolon: `ceph osd pool set <pool>-data allow_ec_optimizations 1`. (Ez **egyirányú út**, nem vonható vissza!)

## 8.10 CRUSH Térkép és Device Classes

A **CRUSH** algoritmus határozza meg, hová kerüljenek az adatok. Támogatja az eszköz-osztályokat (**device classes**), mint `nvme`, `ssd`, `hdd`.

- **Hierarchia megtekintése:** `ceph osd crush tree --show-shadow`
- **Szabály létrehozása (pl. csak NVMe-re):** `ceph osd crush rule create-replicated <szabály_neve> default host nvme`
- **Pool hozzárendelése a szabályhoz:** `ceph osd pool set <pool> crush_rule <szabály_neve>`

## 8.13 CephFS (Fájlrendszer)

A Ceph POSIX fájlrendszere mentésekhez, ISO-khoz. Szüksége van egy **MDS (Metadata Server)** démonra.

- **MDS Létrehozás:** `pveceph mds create`. A gyors failover érdekében a tartalék MDS tartós melegen tartható a `mds standby replay = true` beállítással a `ceph.conf`-ban.
- **CephFS Létrehozás:** `pveceph fs create --pg_num 128 --add-storage`
- **CephFS Törlése (Veszélyes!):**
  1. Kliensek lecsatlakoztatása (`umount /mnt/pve/<név>`).
  2. MDS démonok leállítása és törlése: `pveceph stop --service mds.<NÉV>`, majd `pveceph mds destroy <NÉV>`.
  3. Fájlrendszer megsemmisítése: `pveceph fs destroy <NÉV> --remove-storages --remove-pools`.

## 8.14 Karbantartás (Maintenance) és Hibaelhárítás

- **Trim/Discard:** Erősen ajánlott a VM-ekben az `fstrim` rendszeres futtatása, hogy a Ceph visszakapja a felszabadult blokkokat.
- **Scrubbing:** A Ceph naponta (metadata scrub) és hetente (deep scrub – checksum ellenőrzéssel) ellenőrzi az adatintegritást.
- **Egészségügyi riasztás némítása (Mute):** Ha egy hibaüzenet zavar, de tudsz róla: `ceph health mute <riasztás_neve> 2w` (2 hétre némítja).
- **Titkosítási kulcsok migrálása (CVE-2025-30156 javítás):** A Ceph régi `aes` titkosítása sebezhető, az új `aes256k`-ra kell migrálni. Ehhez a Proxmox egy beépített szkriptet ad:

```bash
/usr/share/pve-manager/migrations/pve-cephx-rotate-service-keys --rotate-cluster-keys
```

(A `--apply` kapcsolóval hajtja végre a tényleges rotációt).

**A Proxmox + Ceph HCI teljes leállítása (Shutdown folyamat):**

1. Minden VM és konténer leállítása.
2. Kapcsold be a `noout` flag-et, hogy a Ceph ne kezdjen el eszeveszetten rebalancolni, amikor a node-ok eltűnnek: `ceph osd set noout`.
3. Állítsd le azokat a gépeket, amiken **NINCS Monitor (MON)**.
4. Végül állítsd le a Monitoros gépeket.

Visszakapcsolásnál fordítva: először a MON gépek jöjjenek fel. Miután minden zöld (`HEALTH_OK`), kapcsold ki a flag-et: `ceph osd unset noout`.

**Hálózati IP-k módosítása (Network Changes):**
Rendkívül kényes művelet. A `ceph.conf`-ban át kell írni a `public_network` vagy `cluster_network` paramétert, majd a szolgáltatásokat node-onként **EGYESÉVEL** újra kell indítani (`systemctl restart ceph-osd.target`, stb.), megvárva, amíg a cluster újra `HEALTH_OK` lesz. **Tilos egyszerre több gépen újraindítani az OSD-ket**, mert a lecsökkenő replikaszám azonnal megakasztja az I/O-t!

**Monitorozás CLI-ből:**

- Valós idejű állapot: `watch ceph --status`
- Ceph összeomlások (**crashek**) olvasása: `ceph crash ls`, majd archiválásuk (hogy eltűnjön a hibaüzenet a GUI-ról): `ceph crash archive-all`.
- Ha egy OSD offline-nak tűnik, de a lemez amúgy egészséges, újra lehet aktiválni: `ceph-volume lvm activate --all`.
