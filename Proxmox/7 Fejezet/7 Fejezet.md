# 7. Fejezet: Proxmox VE Tárhelyek (Koncepciók és Lokális Tárolók)

A Proxmox VE tárolási modellje rendkívül rugalmas: bármennyi tárhelyet (**Storage Pool**) definiálhatsz. Két fő tárolóosztály létezik:

- **Fájl-szintű (File level) tárolók:** Teljes értékű (POSIX) fájlrendszert adnak (pl. könyvtár, NFS, CIFS, CephFS, BTRFS). Előnyük, hogy bármilyen típusú tartalmat (ISO, backup, VM lemez) tárolhatnak.
- **Blokk-szintű (Block level) tárolók:** Nyers (raw) blokk-eszközöket biztosítanak (pl. ZFS, LVM, iSCSI, Ceph RBD). Ezeken általában csak VM lemezképek és konténer kötetek tárolhatók.

## 7.1 Thin Provisioning és a qcow2 snapshotok anomáliái

Több tároló és a QEMU **qcow2** formátuma is támogatja a **Thin Provisioning** technológiát. Lényege: ha létrehozol egy $32$ GB-os lemezt, de a vendég OS csak $3$ GB-ot ír tele, fizikailag csak $3$ GB foglalódik le a tárhelyen.

> **Figyelmeztetés (Over-provisioning):** Ha a fizikai tárhely betelik, az összes azt használó VM I/O hibát (**Input/Output error**) kap, ami fájlrendszer-korrupcióhoz és adatvesztéshez vezethet. Mindig monitorozni kell a szabad helyet!

> **Figyelmeztetés a qcow2 snapshotokról:** Fájl-alapú tárolókon a pillanatképek a qcow2 belső snapshot vagy láncolt funkcióját használják. Ennek elkészítése és törlése blokkolja a futó VM-et, és a teljesítménye (különösen hálózati tárolókon, pl. NFS) extrém rossz lehet. Nagy lemezeknél percekig, vagy akár órákig is tarthat. Ha ilyet tapasztalsz, a snapshot műveleteket kikapcsolt VM mellett végezd el.

## 7.2 és 7.3 Tárolók konfigurálása és a Kötetek (Volumes)

Az összes tárhely konfigurációja a `/etc/pve/storage.cfg` fájlban tárolódik, amely a **pmxcfs**-en keresztül minden node-ra azonnal replikálódik. Formátuma: `<type>: <STORAGE_ID>`.

> **Figyelmeztetés (Aliased storage):** Szigorúan kerülendő, hogy több különböző konfigurációs bejegyzés is pontosan ugyanarra a fizikai tárhelyre mutasson, mert ez ahhoz vezethet, hogy két különböző Volume ID ugyanazt a lemezképet hivatkozza meg, ami összezavarja a Proxmoxot.

**Közös (minden tárolónál elérhető) beállítási paraméterek:**

- **`nodes`:** Megadható, hogy a tároló mely node-okon legyen elérhető.
- **`content`:** A tárolt tartalom típusa:
  - `images` = VM lemezek,
  - `rootdir` = LXC adatok,
  - `vztmpl` = LXC sablonok,
  - `backup` = vzdump mentések,
  - `iso` = ISO képek,
  - `snippets` = hook szkriptek,
  - `import` = OVA/VM importálható lemezek.
- **`shared`:** Jelzi, hogy a tároló a hálózaton minden node számára azonos tartalommal érhető el (lokális tárolót ez nem tesz megosztottá, csak jelölő).
- **`disable`:** A tárhely teljes kikapcsolása.
- **`prune-backups`:** Mentések megtartási (retention) szabályai.
- **`format`:** Alapértelmezett lemezformátum (`raw`, `qcow2`, `vmdk`).
- **`preallocation`:** Fájl-alapú tárolókon a lemezfoglalás módja (`off`, `metadata`, `falloc`, `full`). Alapértelmezett a `metadata`. Hálózati tárolóknál az `off` segíthet elkerülni az időtúllépéseket (**timeout**).

**Kötetek (Volumes) azonosítása és tulajdonjoga:**
A Proxmox egy speciális szintaxist használ: `<STORAGE_ID>:<volume_name>` (pl. `local:230/example-image.raw`).
Minden VM/CT lemezkép a hozzá tartozó gép tulajdona. Ha törlöd a VM/CT-t, a rendszer automatikusan megsemmisíti a hozzá tartozó köteteket is.

## 7.4 A Parancssori felület (CLI) – `pvesm` (Proxmox VE Storage Manager)

A **`pvesm`** a tárhelyek és kötetek legalsó szintű parancssoros eszköze. A dokumentáció az alábbi parancsokat listázza:

**Tárolók hozzáadása:**

```bash
pvesm add dir <STORAGE_ID> --path <PATH>
pvesm add lvm <STORAGE_ID> --vgname <VGNAME>
```

(További hálózatos példák: NFS és iSCSI hozzáadása is lehetséges hasonló szintaxissal)

**Tároló tiltása/engedélyezése:**

```bash
pvesm set <STORAGE_ID> --disable 1   # tiltás
pvesm set <STORAGE_ID> --disable 0   # engedélyezés
```

**Opciók módosítása:**

```bash
pvesm set <STORAGE_ID> --shared 1
pvesm set local --format qcow2
pvesm set <STORAGE_ID> --content iso
```

**Tároló eltávolítása (csak a konfigot törli, az adatokat nem!):**

```bash
pvesm remove <STORAGE_ID>
```

**Kötet lefoglalása:** $4$ GB-os lemez foglalása a $100$-as VM-nek. (Ha a név `''`, automatikusan generál egyet).

```bash
pvesm alloc local 100 '' 4G
```

**Kötet felszabadítása (VIGYÁZAT: ez végleg törli az adatokat!):**

```bash
pvesm free <VOLUME_ID>
```

**Állapot és listázás:**

```bash
pvesm status
pvesm list <STORAGE_ID> [--vmid <VMID>]
pvesm list <STORAGE_ID> --content iso
```

**Fizikai útvonal lekérése:**

```bash
pvesm path <VOLUME_ID>
```

(Visszaadja a fájlrendszerbeli útvonalat).

**Kötet exportálása:** (Főleg a belső `pvesm import` által használt `qcow2+size` formátumú export):

```bash
pvesm export local:103/vm-103-disk-0.qcow2 qcow2+size target --with-snapshots 1
```

## 7.5 Könyvtár (Directory) Backend (`dir`)

Helyi könyvtárakat vagy felcsatolt (pl. `/etc/fstab` alapján) fájlrendszereket használ. Bármit tárolhat.

- **Új opciók:** `path` (a könyvtár abszolút útvonala) és a `content-dirs` (az alapértelmezett almappa-kiosztás felülbírálása, pl. `content-dirs backup=custom/backup/dir`).
- **Elnevezési konvenciók:**
  - `vm-<VMID>-disk-[N].<FORMAT>` (normál VM lemez)
  - `base-<VMID>-disk-[N].<FORMAT>` (sablon / template lemez). A backend ezt írásvédetté (`0444`) teszi, és a fájlrendszeren az "immutable" taget (`chattr +i`) is beállítja.

> **Megjegyzés a gyorsítótárról:** Bizonyos fájlrendszerek nem támogatják az `O_DIRECT`-et, így a `cache=none` opció használata a VM-nél hibára futhat. Helyette a `cache=writeback` használandó.

## 7.9 Lokális ZFS Pool Backend (`zfspool`)

Lokális ZFS poolokat vagy dataset-eket (fájlrendszereket) kezel. A ZFS biztosítja az egyik legfejlettebb snapshot és klónozás támogatást. A VM-ek `raw` formátumban **ZVOL**-ként (blokkeszköz), a konténerek `subvol` formátumban datasetként tárolódnak.

**ZFS specifikus beállítások:**

- **`pool`:** A használt ZFS pool/fájlrendszer neve.
- **`blocksize`:** ZFS blokkméret paraméter.
- **`sparse`:** ZFS thin-provisioning bekapcsolása (a lefoglalás mérete nem egyezik a kötet méretével).
- **`mountpoint`:** A csatolási pont (alapértelmezetten `/<pool>`).

**Elnevezési konvenciók:** `vm-<VMID>-<NAME>`, `base-<VMID>-<NAME>` és `subvol-<VMID>-<NAME>`.

> **Titkosítás (Encryption) kritikus megjegyzés:** A ZFS (0.8.0-tól) natívan támogatja a titkosítást (`encryption_root`). Ha a Proxmox VE-ben élő migrációt (Live Migration) vagy Storage Replikációt akarsz végrehajtani titkosított ZVOL-okon, a rendszer az adatokat titkosítási paraméterek nélkül küldi át, és a célgépen lévő ZFS határozza meg a titkosítás állapotát. Ahhoz, hogy ez működjön, az összes érintett node-on, minden résztvevő tárhelynek titkosítottnak kell lennie.

## 7.10 LVM Backend (`lvm`)

A Logical Volume Manager lokális vagy iSCSI/Fibre Channel LUN fölötti megosztott tárhelyként is funkcionálhat. A natív LVM snapshotok nagyon rossz teljesítményt nyújtanak (az egész Volume Group írási teljesítményét lezúzzák), ezért a Proxmox bevezette a **"snapshot-as-volume-chain"** technológiát.

**LVM specifikus beállítások:**

- **`vgname`:** A meglévő Volume Group neve.
- **`base`:** Alapkötet (Base volume), ami a tárhely elérése előtt automatikusan aktiválódik (főleg iSCSI SAN esetén hasznos).
- **`saferemove`:** (A GUI-n "Wipe Removed Volumes"). Törléskor a területet nullákkal írja tele, hogy a későbbi kötetek ne férhessenek hozzá a régi adatokhoz (biztonsági okokból). A `blkdiscard` paranccsal operál, ha nem támogatott, akkor `cstream`-el.
- **`saferemove-stepsize`** és **`saferemove_throughput`:** A törlés blokkmérete (max $32$ MiB) és sávszélessége (alapértelmezetten $10$ MiB/s).
- **`snapshot-as-volume-chain`:** (Technológiai előzetes / Technology Preview) Engedélyezi a qcow2-höz hasonló backing-chain snapshotokat a blokk-szintű LVM-en. Vastagon (thick-provisioned) allokálja az LVM köteteket, de a qcow2 rétegzését használja. Alapfeltétele a thin-provisioning és a discard támogatása a hardveres réteg alatt.

## 7.11 LVM-thin Backend (`lvmthin`)

A thin-provisioning logikára épülő LVM, amely kiváló, teljesítmény-veszteség nélküli snapshotokat és klónozást támogat. A kötetek blokkjai csak kiíráskor foglalódnak le, és automatikusan nullázódnak. **Nem osztható meg (Not Shared) több node között.**

**LVM-thin létrehozása parancssorból:**

```bash
lvcreate -L 100G -n data pve
lvconvert --type thin-pool pve/data
```

**Beállítások:** Csak a `vgname` és a `thinpool` (a thin-pool neve) kell hozzá.

## 7.16 BTRFS Backend (`btrfs`) (Technológiai előzetes)

Működésében a `dir` (Könyvtár) backendhez hasonlít, de a nyers (raw) lemezképeket BTRFS almappákban (**subvolume**) helyezi el. Ez lehetővé teszi az offline tárhely-migrációt úgy, hogy a snapshotok is megmaradnak.

> **O_DIRECT figyelmeztetés:** A BTRFS tiszteletben tartja az `O_DIRECT` flag-et. **Szigorúan tilos** a VM-eknél a `cache=none` opció használata, különben checksum hibák (adatkorrupció) lépnek fel!

**Konfiguráció:** Ha egy már felcsatolt könyvtárat adsz hozzá, kötelező az `is_mountpoint` opció megadása a `storage.cfg`-ben (pl. `is_mountpoint /mnt/data2`).

**Snapshotok (Pillanatképek):** Amikor snapshot készül egy lemezről vagy almappáról, a BTRFS egy írásvédett (**read-only**) almappát hoz létre ugyanazzal az útvonallal, de egy `@` jellel és a snapshot nevével kiegészítve.

## 7 Fejezet második fele 

# 7. Fejezet: Proxmox VE Tárhelyek (Hálózati és Elosztott Tárolók)

A hálózati tárolók legnagyobb előnye, hogy lehetővé teszik az élő migrációt (**Live Migration**) nulla leállással, hiszen a lemezképet nem kell mozgatni a node-ok között, csupán a RAM tartalmát.

## 7.6 NFS Backend (`nfs`)

Fájl-szintű tároló, amely a `dir` (könyvtár) backendre épül, így ugyanazt a mappaszerkezetet és fájlelnevezési logikát használja. Előnye, hogy a Proxmox VE közvetlenül kezeli az NFS csatolást, nem kell a `/etc/fstab` fájlt kézzel szerkeszteni.

**Beállítások:**

- **`server`:** IP vagy DNS név (de az IP ajánlott a DNS késések elkerülésére).
- **`export`:** A megosztás útvonala.
- **`path`:** Lokális csatolási pont (alapértelmezetten `/mnt/pve/<STORAGE_ID>/`).
- **`content-dirs`:** Egyedi mappastruktúra.
- **`options`:** NFS csatolási opciók.

> **Figyelmeztetés a timeoutok miatt (`soft` opció):** Ha az NFS kérés időtúllépésbe fut, alapértelmezetten a végtelenségig újrapróbálkozik, ami a kliens (VM) lefagyásához vezethet. Csak olvasható (read-only) tartalomnál vagy instabil hálózatnál érdemes a `soft` opciót (pl. `options vers=3,soft`) beállítani, ami 3 próbálkozás után megszakítja a kérést.

**Funkciók:** Snapshotok és klónozás csak `qcow2` formátummal lehetséges.

**CLI lekérdezés:**

```bash
pvesm scan nfs <server>
```

## 7.7 CIFS Backend (`cifs`)

A Microsoft SMB/CIFS protokollját használó fájl-szintű tároló (szintén nem kell `/etc/fstab` módosítás).

**Beállítások:**

- **`server`:** A szerver címe.
- **`share`:** A megosztás neve.
- **`username`:** Alapból `guest`.
- **`password`:** Titkosítva a `/etc/pve/priv/storage/<STORAGE-ID>.pw` fájlban tárolódik, amit csak a `root` olvashat.
- **`domain`:** Munkacsoport.
- **`smbversion`:** SMB protokoll verzió; alapból a legmagasabb SMB2+ verziót egyezteti, az SMB1 biztonsági okokból **nem támogatott**.
- **`path`:** Csatolási pont.
- **`subdir`:** Egy konkrét almappa felcsatolása a megosztáson belül.

**Funkciók:** Csak `qcow2` lemezekkel támogatja a pillanatképeket.

**CLI hozzáadás:**

```bash
pvesm add cifs <storagename> --server <server> --share <share> [--username <username>] [--password]
```

## 7.8 Proxmox Backup Server Backend (`pbs`)

Közvetlen integráció a Proxmox Backup Serverrel. A VM-eknél blokk-szintű, a konténereknél fájl-szintű deduplikált mentést végez.

**Beállítások:**

- **`server`:** A PBS szerver címe.
- **`port`:** Alapértelmezett: `8007`.
- **`username`:** Kötelező megadni a realmet is, pl. `archiver@pbs`.
- **`password`:** Csak `root` által olvasható fájlba kerül.
- **`datastore`:** Az adattár ID-ja a PBS-en.
- **`fingerprint`:** A PBS API TLS tanúsítványának ujjlenyomata – önaláírt cert esetén **kötelező**.

### Titkosítás (Encryption)

Kliens-oldali **AES-256 GCM** titkosítás. Két kulcsopció van:

- **`encryption-key`:** Ha az értéke `autogen`, a Proxmox generál egy új kulcsot. Ez a `/etc/pve/priv/storage/<STORAGE-ID>.enc` fájlba kerül.
- **`master-pubkey`:** Egy RSA publikus mesterkulcs, amely a biztonsági mentésbe ágyazva egy titkosított másolatot tárol az AES kulcsból a vészhelyzeti helyreállításhoz.

> **Kritikus figyelmeztetés a titkosításhoz:** Ha elveszted a kulcsot, a mentések **olvashatatlanná válnak!** Ne tárold a kulcsot kizárólag azon a gépen, amit mentesz. A dokumentáció javasolja a jelszókezelőbe mentést, az USB pendrive-ot, és a papíralapú mentést (QR kód formájában):

```bash
proxmox-backup-client key paperkey /etc/pve/priv/storage/<STORAGE-ID>.enc --output-format text > qrkey.txt
```

**CLI hozzáadás:**

```bash
pvesm add pbs <id> --server <server> --datastore <datastore> --username <username> --fingerprint 00:B4:... --password
```

## 7.12 Open-iSCSI Initiator (`iscsi`)

Blokk-szintű tároló, széles körben támogatott (pl. OpenMediaVault, SAN-ok). Használatához telepíteni kell az `open-iscsi` Debian csomagot (alapból nincs fent az erőforrások spórolása miatt). Low-level menedzsmentre az `iscsiadm` használható.

**Beállítások:**

- **`portal`:** IP/DNS név és opcionálisan port.
- **`target`:** Az iSCSI target azonosítója.

> **Gyakorlati tipp (LVM on iSCSI):** Mivel az iSCSI protokoll nem tud köteteket kiosztani (csak egy nagy LUN-t ad), a legjobb gyakorlat, ha beállítod a `content none` paramétert az iSCSI tárolón, és létrehozol rajta egy LVM-et. Így a VM-ek nem közvetlenül a nyers iSCSI LUN-ra írnak, hanem az iSCSI felett lévő LVM-re.

**Funkciók:** Nincs snapshot, és nincs klónozás.

## 7.13 User Mode iSCSI Backend (`iscsidirect`)

Szintén iSCSI kliens, de kernel driver nélkül, egy user-level (felhasználói szintű) könyvtárat használ (teljesítményoptimalizálás). A `libiscsi-bin` csomag szükséges hozzá.

- **Hátrány:** Erre az iSCSI LUN-ra nem lehet LVM-et építeni. A tárhelyelosztást a szerver oldalán kell megoldani.
- **Korlátozás:** Csak VM-ekkel (KVM) működik, LXC konténerek nem tudják használni.

## 7.14 Ceph RADOS Block Devices (`rbd`)

A Ceph blokk-alapú eszköze, ami vékony allokációt (**thin provisioning**), elosztott redundanciát (OSD-ken átívelő striping), öngyógyítást és méretezhető teljesítményt nyújt.

**Beállítások (Külső Ceph cluster esetén):**

- **`monhost`:** Monitor daemon IP-k, külső cluster esetén kötelező.
- **`pool`:** A Ceph pool neve.
- **`username`:** A `client.` előtag nélkül, pl. `admin` helyett.
- **`krbd`:** A kernel modul kényszerítése RBD eléréshez.

### Hitelesítés külső Ceph esetén

Létre kell hozni egy dedikált keyringet (**SOHA ne a `client.admin`-t használd**), majd átmásolni a Proxmoxra:

```bash
scp <external-ceph-server>:/etc/ceph/ceph.client.pve-rbd.keyring /root/rbd.keyring
pvesm add rbd <name> --monhost "10.1.1.20" --pool ceph-external --content images --username pve-rbd --keyring /root/rbd.keyring
```

A parancs után a `/root/rbd.keyring` **törlendő**, mert a Proxmox a fájlt áthelyezi.

**Opcionális Ceph kliens konfig:** Ha speciális beállítások kellenek a kliensnek, létrehozható egy konfigurációs fájl pontosan a storage nevével megegyezően: `/etc/pve/priv/ceph/<STORAGE_ID>.conf`.

## 7.15 Ceph Filesystem (`cephfs`)

A Ceph POSIX-kompatibilis fájlrendszere. Mivel fájl-szintű, alkalmas ISO-k és mentések tárolására Ceph környezetben. A használatához le kell cserélni a gyári Debian Ceph klienst a Proxmox saját Ceph repójának felvételével (`apt update` és `apt dist-upgrade` szükséges).

> **Figyelmeztetés:** **Tilos** más, külső Ceph tárolót beállítani a csomagkezelőben, mert a kevert csomagverziók tönkreteszik a node-ot!

**Beállítások:**

- **`fs-name`:** A CephFS neve.
- **`monhost`:** Monitor daemon IP-k.
- **`path`:** Csatolási pont.
- **`username`:** A Ceph felhasználó.
- **`subdir`:** Almappa.
- **`fuse`:** Kernel kliens helyett FUSE használata.

### Hitelesítés (Bare Key)

A CephFS nem keyring formátumot vár, hanem a nyers, csupasz titkos kulcsot (**bare key**). Ezt kell kiexportálni a külső Ceph-ből:

```bash
ssh root@<external-ceph-server> 'ceph auth get-key client.pve-cephfs' > /root/cephfs.secret
pvesm add cephfs <name> --monhost "10.1.1.20" --fs-name cephfs --content backup --username pve-cephfs --keyring /root/cephfs.secret
```

**Funkciók:** A snapshotok támogatottak, de a dokumentáció figyelmeztet, hogy még nem garantált a stabilitásuk (technológiai előzetes jellegű ezen a szinten).

## 7.17 ZFS over iSCSI Backend (`zfs`)

Ez a backend egy olyan távoli (remote) szervert ér el, amely ZFS-t és iSCSI célpontot (Target) is futtat. A Proxmox VE SSH-n keresztül bejelentkezik a távoli szerverre, létrehoz ott egy ZVOL-t, majd azt iSCSI LUN-ként exportálja és felcsatolja a VM-hez. (**Nem használható arra, hogy sima SAN-on hozz létre ZFS pool-t!**)

**Támogatott iSCSI Target megoldások:**

- **LIO** (Linux)
- **IET** (Linux)
- **ISTGT** (FreeBSD)
- **Comstar** (Solaris/Illumos)

### SSH hitelesítés beállítása (Jelszó nélküli kulcs!)

```bash
mkdir /etc/pve/priv/zfs
ssh-keygen -f /etc/pve/priv/zfs/192.0.2.1_id_rsa
ssh-copy-id -i /etc/pve/priv/zfs/192.0.2.1_id_rsa.pub root@192.0.2.1
```

**Beállítások:**

- **`pool`:** A távoli ZFS pool neve.
- **`portal`:** Az iSCSI portal címe.
- **`target`:** Az iSCSI target azonosítója.
- **`iscsiprovider`:** LIO, IET, stb.
- **`comstar_tg`, `comstar_hg`, `lio_tpg`:** Target-specifikus beállítások.
- **`nowritecache`:** Írási gyorsítótár tiltása a targeten.
- **`blocksize`:** Blokkméret.
- **`sparse`:** ZFS thin provisioning bekapcsolása.

**Funkciók:** Bár iSCSI alapú, mivel a háttérben ZVOL-okat hoz létre külön minden VM-nek, **teljesen támogatja a ZFS snapshot (pillanatkép) funkciókat megosztott tárhelyként is**.
