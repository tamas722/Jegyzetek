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

## Hálózat és MTU tesztelés

**Mesh hálózat:** 3-5 node-os kisebb clusterek esetén, ha nincs 10+ Gbps switch, a fizikai hálózati kártyák közvetlen összekötése (**Mesh network**) is hivatalosan támogatott opció a Ceph számára.

**Ping teszt Jumbo Frame-mel:** Hálózatcsere esetén az MTU (pl. 9000) helyes működését IP+ICMP overhead kalkulálásával kell tesztelni. 9000-es MTU esetén IPv4-nél **8972 byte** (`ping -M do -s 8972 <IP>`), IPv6-nál **8952 byte** csomagmérettel.

**IPv6 beállítások:** Ha IPv4-ről IPv6-ra váltasz (vagy fordítva), a `ceph.conf`-ban kötelezően módosítani kell az `ms_bind_ipv4` és `ms_bind_ipv6` változókat `true` vagy `false` értékre.

## 2. OSD Létrehozás (DB/WAL) és Filestore

**DB/WAL méretezés fallback logikája:** Ha NVMe/SSD gyorsítótárat adsz egy HDD-hez, de nem specifikálod a `-db_size` vagy `-wal_size` méreteket, a Ceph a következő sorrendben keresi a méreteket:

1. `ceph.conf` OSD szekció,
2. `ceph.conf` Global szekció,
3. Alapértelmezett beállításként a lemez méretének **10%-a** a DB-nek, **1%-a** a WAL-nak.

**Filestore:** Bár a Nautilus óta a Bluestore az alapértelmezett, a régi Filestore alapú OSD is létrehozható maradt, de már nem a `pveceph` paranccsal, hanem közvetlenül a háttér-eszközzel:

```bash
ceph-volume lvm create --filestore --data /dev/sd[X] --journal /dev/sd[Y]
```

## 3. Titkosított OSD Lockbox Kulcsok

> **Veszély:** Minden titkosított OSD használ egy dedikált `client.osd-lockbox.<uuid>` felhasználót a lemez titkosítási kulcsának lekéréséhez. Ezt a kulcsot **soha nem szabad** a normál `ceph auth` paranccsal rotálni, mert a lemez LVM tag-jét és a Ceph adatbázist egyszerre kell frissíteni. Erre kizárólag a PVE beépített migrációs eszköze alkalmas.

## 4. Ceph Konfigurációs Hierarchia

**Felülbírálás:** Ha a `ceph config set` paranccsal beállítasz egy értéket a MON adatbázisában, de az az opció szerepel a lokális `ceph.conf` fájlban is, a **lokális fájl felülbírálja a központit** (a MON DB-nek alacsonyabb a prioritása).

## 5. Pool és Erasure Coding extra részletek

**Pool opciók:** A GUI/CLI létrehozásnál olyan extrák is megadhatók, mint:

- **Target Ratio** (adatok elvárt aránya a többi poolhoz képest),
- **Target Size** (elvárt méret gigabájtban),
- **Min. # of PGs** (az autoscaler nem vonhatja össze a PG-ket ezen érték alá).

**EC Pool matek:** Az Erasure Coded poolok `min_size` értéke így alakul:

- ha $m=1$, akkor a `min_size` megegyezik $k$-val,
- ha $m>1$, akkor $min\_size = k+1$.

(A dokumentáció hivatalosan a $k+2$-t ajánlja biztonsági okokból).

**EC Pool csatolása PVE Storage-ként:** Mivel az EC pool mellé létrejön egy kötelező metaadat pool is, a Proxmox tárhelyhez történő hozzáadásakor kötelező a `--data-pool` kapcsoló használata:

```bash
pvesm add rbd <name> --pool <meta_pool> --data-pool <data_pool>
```

## 6. PG Autoscaler és CRUSH összefüggések

**Autoscaler limit:** A PG autoscaler alapértelmezetten csak akkor avatkozik be, ha a tényleges PG szám **3-szorosan** eltér az optimálistól. Mivel ez a beavatkozás masszív adatmozgatással jár, érdemes előre beállítani a **Target Size**-t.

**CRUSH inkompatibilitás:** Az autoscaler csak akkor tudja méretezni a poolokat, ha az összes pool ugyanolyan típusú CRUSH szabályt használ (pl. vagy mindegyiknél van megadva **Device Class**, vagy egyiknél sem). Kevert szabályok esetén az autoscaler átugorja a hibás poolokat.

## 7. CephFS mélyebb opciók

**Multiple Active MDS:** A Luminous verziótól kezdve lehetséges több aktív Metaadat-szervert futtatni egyszerre. Ez csak extrém mennyiségű, párhuzamosan író/olvasó kliens esetén szükséges, mert az MDS ritkán szűk keresztmetszet.

**CephFS struktúra:** A `pveceph fs create` parancs a háttérben két poolt hoz létre:

- `cephfs_data` poolt (alapból 128 PG-vel),
- `cephfs_metadata` poolt (aminek a PG száma mindig a data pool negyede, jelen esetben 32).

**Törlés kapcsoló:** A fájlrendszer törlésekor a `--remove-storages` kapcsolóval érhető el, hogy a `storage.cfg`-ből is eltűnjön a bejegyzés.

## 8. Karbantartás és Hibaelhárítás extrák

**Egészségügyi riasztás tartós némítása:** A `ceph health mute <név>` parancsnál a `--sticky` (GUI-n: **Keep Mute**) kapcsoló használatával elérhető, hogy a riasztás némítva maradjon akkor is, ha a probléma súlyosbodik (pl. egy újabb OSD esik ki ugyanazzal a hibával).

**Alapvető logok:** Mélyebb diagnosztikához a `/var/log/ceph/` könyvtár és a rendszernapló szükséges:

```bash
journalctl --since "2 days ago"
```

## 9. Kulcs rotáció (CVE-2025-30156) teljes menete

Az előző jegyzet csak a cluster-keys forgatását említette, de a felhasználói és tárolókulcsok migrálása egy sokkal komplexebb, több lépcsős folyamat:

1. **Kliens kompatibilitás ellenőrzése:** A CephFS és az LXC RBD mountok a Linux kernel klienst használják. Az `aes256k` titkosításhoz kötelezően legalább **7.0-s Linux kernel** kell a szerveren!
2. **Kulcsok ideiglenes felvétele (Staging):** A tárolókulcsok frissítését a `--rotate-all-storage-keys --rotate-admin-key` kapcsolókkal kell elindítani. Ez "stage"-eli az új kulcsot, de a régit is érvényben hagyja, amíg a kliensek frissülnek.
3. **Kliensek újraolvastatása:** A futó VM-eket élőben migrálni kell, vagy teljesen leállítani és elindítani. A sima újraindítás (`guest reboot`) **nem elég** a kulcs újraolvasásához! A konténereket le kell állítani, majd újraindítani.
4. **Megerősítés vagy Visszagörgetés:** Ha minden kliens átállt, a `--confirm-all-clients-refreshed` kapcsolóval lehet véglegesíteni a folyamatot. Ha baj van, a `--abort-staged-key` paranccsal vonható vissza az új kulcs.
5. **Emergency Override:** Ha kizárod magad a rendszerből, a `ceph.conf`-ban a `mon_auth_emergency_allowed_ciphers` ideiglenes beállításával nyerhetsz vissza hozzáférést a régi kulcsokkal.
