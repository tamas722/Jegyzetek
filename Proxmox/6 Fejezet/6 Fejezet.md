# 6. Fejezet: Proxmox Cluster File System (pmxcfs)

A Proxmox Cluster File System (**pmxcfs**) egy adatbázis-vezérelt (SQLite) fájlrendszer, amely a konfigurációs fájlok tárolására szolgál. A **Corosync** cluster motor segítségével ez valós időben, transzparens módon replikálódik a cluster összes csomópontja (node-ja) között. Bár az adatok perzisztensen a fizikai lemezen tárolódnak, a gyorsaság érdekében egy másolat a RAM-ban is él. Ennek maximális mérete **$128$ MiB**, de ez a korlát bőven elegendő több ezer virtuális gép (VM) konfigurációjának tárolásához.

**A pmxcfs architektúra legfőbb előnyei:**

- **Valós idejű szinkronizáció:** Zökkenőmentesen replikálja a konfigurációkat minden node-ra.
- **Konzisztencia-ellenőrzés:** Szigorúan garantálja, hogy ne jöhessen létre két azonos VMID a clusterben.
- **Biztonság Quorum vesztéskor:** Ha a cluster elveszíti a többséget (quorum), a fájlrendszer azonnal írásvédett (**read-only**) módba kapcsol.
- **Corosync frissítés:** A Corosync konfigurációját automatikusan frissíti a node-ok között.
- **Elosztott zárolás:** Integrált, elosztott zárolási mechanizmust (**distributed locking**) tartalmaz.

## 6.1 és 6.2 POSIX Kompatibilitás és Jogosultságok

A fájlrendszer **FUSE** (Filesystem in Userspace) alapú, viselkedése POSIX-szerű, de a Proxmox tudatosan kihagyott néhány funkciót, amire nincs szükség, vagy veszélyeztetné a konzisztenciát:

- Létrehozhatsz normál fájlokat és mappákat, de **szimbolikus linkeket nem**.
- **Nem nevezhetsz át nem üres könyvtárakat** (ezzel könnyebb garantálni az egyedi VMID-ket).
- **Nem tudsz fájljogosultságokat módosítani** (a jogosultságok fixen, útvonalak alapján vannak bedrótozva).
- A fájl-létrehozások (`O_EXCL` és `O_TRUNC` opciókkal) **nem atomiak** (FUSE és NFS örökségből fakadó korlátozások).

A fájlrendszerben minden fájl tulajdonosa a `root` felhasználó, csoportja pedig a `www-data`. Csak a `root` kapott írási jogot, de a `www-data` a legtöbb fájlt olvashatja. Szigorú kivételt képeznek az alábbi útvonalak, amelyekhez kizárólag a `root` férhet hozzá:

- `/etc/pve/priv/`
- `/etc/pve/nodes/${NAME}/priv/`

## 6.3 és 6.4 Technológia és a Fájlrendszer Felépítése

A háttérben a **Corosync Cluster Engine** felel a kommunikációért, a fájlokat **SQLite** adatbázis tárolja, az operációs rendszer felé pedig **FUSE**-zal van felcsatolva a `/etc/pve` könyvtárba.

### A `/etc/pve` alatt található legfontosabb fájlok és könyvtárak

- **`authkey.pub`:** A jegyrendszer (ticket system) publikus kulcsa.
- **`ceph.conf`:** Ceph konfigurációs fájl (a `/etc/ceph/ceph.conf` egy symlink, ami ide mutat).
- **`corosync.conf`:** Corosync cluster konfiguráció (PVE 4.x előtt `cluster.conf` volt).
- **`datacenter.cfg`:** Cluster-szintű adatközpont-beállítások (pl. proxy, billentyűzetkiosztás).
- **`domains.cfg`:** PVE hitelesítési tartományok.
- **`firewall/cluster.fw`, `firewall/<NAME>.fw`, `firewall/<VMID>.fw`:** Tűzfalszabályok cluster, node, illetve VM/CT szinten.
- **`ha/crm_commands`, `ha/manager_status`, `ha/resources.cfg`, `ha/rules.cfg`:** A magas rendelkezésre állás (HA) erőforrásai, parancsai, állapotai és szabályai.
- **`nodes/<NAME>/config`:** Node-specifikus beállítások.
- **`nodes/<NAME>/qemu-server/<VMID>.conf`** és **`nodes/<NAME>/lxc/<VMID>.conf`:** A VM-ek és konténerek konfigurációs fájljai.
- **`nodes/<NAME>/openvz/`:** PVE 4.0 előtti konténer konfigok (elavult, törlésre kerül).
- **`nodes/<NAME>/pve-ssl.pem`, `pve-ssl.key`:** A webszerver publikus tanúsítványa (a cluster CA által aláírva) és annak privát kulcsa.
- **`nodes/<NAME>/pveproxy-ssl.pem`, `pveproxy-ssl.key`:** Egyéni SSL tanúsítvány (opcionális).
- **`priv/authkey.key`:** Jegyrendszer privát kulcsa.
- **`priv/authorized_keys`, `priv/known_hosts`:** SSH kulcsok a tagok közötti azonosításhoz és hitelesítéshez.
- **`priv/ceph*`:** Ceph hitelesítési kulcsok.
- **`priv/lock/*`:** Különböző szolgáltatások által használt lock (zároló) fájlok a biztonságos műveletekhez.
- **`priv/pve-root-ca.key`, `pve-root-ca.pem`:** A cluster saját CA (Certificate Authority) kulcsa és tanúsítványa.
- **`priv/shadow.cfg`:** PVE Realm felhasználók jelszó-hash fájlja.
- **`priv/storage/<STORAGE-ID>.pw`:** Egy adott tároló (storage) jelszava egyszerű szöveges (plain text) formátumban.
- **`priv/tfa.cfg`:** Kétlépcsős azonosítás (TFA) Base64-kódolt beállításai.
- **`priv/token.cfg`:** API tokenek titkai.
- **`pve-www.key`:** A CSRF tokenek generálásához használt privát kulcs.
- **`sdn/*`:** Software Defined Networking (SDN) megosztott beállítások.
- **`status.cfg`:** Külső metrika szerverek beállításai.
- **`storage.cfg`:** PVE tárhely-konfiguráció (Storage).
- **`user.cfg`:** Hozzáférés-vezérlési konfiguráció (felhasználók/csoportok).
- **`virtual-guest/cpu-models.conf`:** Egyedi CPU modellek tárolása.
- **`vzdump.cron`:** Cluster-szintű biztonsági mentések ütemezése.

### Szimbolikus linkek

A cluster fájlrendszer tartalmaz beépített szimbolikus linkeket, hogy minden node a saját fájljait könnyebben érje el a `/etc/pve` mappán belül:

- `local` $\rightarrow$ `nodes/<LOKÁLIS_HOSZTNÉV>`
- `lxc` $\rightarrow$ `nodes/<LOKÁLIS_HOSZTNÉV>/lxc/`
- `openvz` $\rightarrow$ `nodes/<LOKÁLIS_HOSZTNÉV>/openvz/`
- `qemu-server` $\rightarrow$ `nodes/<LOKÁLIS_HOSZTNÉV>/qemu-server/`

### Rejtett státuszfájlok és a Debug (Hibakeresés) mód

A `/etc/pve` tartalmaz rejtett, speciális JSON fájlokat is az állapotok olvasásához:

- **`.version`** (fájlverziók a módosítások detektálásához)
- **`.members`** (cluster tagok információi)
- **`.vmlist`** (összes VM listája)
- **`.clusterlog`** (az utolsó 50 cluster naplóbejegyzés)
- **`.rrd`** (a legfrissebb RRD statisztikai adatok)

A részletes (verbose) syslog naplózás, azaz a debug mód bekapcsolása:

```bash
echo "1" >/etc/pve/.debug
```

A debug mód kikapcsolása:

```bash
echo "0" >/etc/pve/.debug
```

## 6.5 Helyreállítás (Recovery)

Ha egy szerver hardvere menthetetlenül tönkremegy, a beállítások megmenthetők, ha kimented a `/var/lib/pve-cluster/config.db` fájlt. Egy új gépen a `pve-cluster` szolgáltatás leállítása után ezt a fájlt be kell másolni (szigorúan **0600**-as jogosultsággal), hozzá kell igazítani a `/etc/hostname` és `/etc/hosts` fájlokat az elveszett gép nevéhez, majd újraindítani a rendszert.

### Cluster konfiguráció törlése

Az ajánlott módszer egy gép kivétele után a teljes újratelepítés, mert így biztosan megsemmisül minden titkos kulcs. A másik, "újratelepítés nélküli lokális módba tétel" is alkalmazható (lásd 5. fejezet).

### Halott node-ról VM-ek manuális mentése (Kritikus művelet!)

A Proxmox VE koncepciója szerint a konfigurációs fájlt tároló fizikai node számít a VM "tulajdonosának". Ez teszi lehetővé a drága, cluster-szintű zárolások helyett a sokkal gyorsabb, lokális zárolások (**local locks**) használatát a gépek módosításakor.

Ha egy node meghal (áramszünet, kernel panic), a rajta lévő VM-eket nem lehet normál úton migrálni, mert az offline géptől lehetetlen elkérni a "zárolást" (lockot).

- **HA-menedzselt (High Availability) VM-ek:** Nincs teendő. A watchdog funkciók automatikusan újraindítják őket más gépen.
- **Kézi megmentés (csak Shared Storage esetén):** Ha a halott gépen egy nem-HA menedzselt VM volt, és annak a lemezei egy megosztott tárhelyen (pl. NAS, Ceph) vannak, kézzel kell átköltöztetni. A cluster egy túlélő tagján `root`-ként át kell mozgatni a konfigot (pl. a 100-as VM-et a `node1`-ről a `node2`-re):

```bash
mv /etc/pve/nodes/node1/qemu-server/100.conf /etc/pve/nodes/node2/qemu-server/
```

> **Két óriási figyelmeztetés ehhez a művelethez:**
>
> 1. **Fencing (Elszigetelés) kötelező:** Mielőtt ezt a manuális `mv` parancsot kiadod, abszolút biztosnak kell lenned abban, hogy a halott gép fizikai értelemben is ki van kapcsolva (áramtalanítva van). Ha a gép mégis él, és hálózati hiba miatt szakadt le, a fájl áthelyezésével felrúgod a Proxmox zárolási elveit, ami **katasztrofális adatkorrupcióhoz** vezethet.
> 2. **Lokális lemezek:** A fenti trükk kizárólag a megosztott tárhelyes gépeknél működik. Ha a VM lemeze a halott node lokális LVM vagy ZFS tárolóján volt, akkor esélytelen átmozgatni: meg kell várnod, amíg a hardvert megjavítják és visszatér a clusterbe, vagy biztonsági mentésből (**backup**) kell visszaállítanod egy másik gépre.
