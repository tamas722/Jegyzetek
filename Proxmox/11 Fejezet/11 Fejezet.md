# 11. Fejezet: Proxmox Container Toolkit (LXC)

A konténerek (CT) nem emulálnak külön operációs rendszert (OS), hanem a host (Proxmox VE) kernelét használják elszeparált névterekben (**namespaces**) és erőforrás-csoportokban (**cgroups**). Ez a megközelítés szinte nulla futási költséggel (**overhead**) jár, így a teljesítményük megegyezik a fizikai gépével.

**Korlátozások:** Kizárólag **Linux disztribúciók** futtathatók bennük (Windows vagy FreeBSD nem). Biztonsági okokból bizonyos kernel rendszerhívások (**syscalls**) tiltva vannak.

**OCI (Open Container Initiative) integráció:** Technológiai előzetesként (**tech preview**) a Proxmox VE már képes szabványos alkalmazáskonténer-képeket (**Docker/OCI**) is letölteni, amelyeket a háttérben automatikusan az LXC stackre konvertál.

## 11.2 – 11.3 Támogatott rendszerek és Sablonkezelés (`pveam`)

A Proxmox VE számos alaprendszerhez biztosít hivatalos sablonokat (Alpine, Arch, CentOS/Alma/Rocky, Debian, Devuan, Fedora, Gentoo, OpenSUSE, Ubuntu), valamint **TurnKey Linux** "appliance" sablonokat. Ezek a `pve-daily-update` időzítőn keresztül naponta frissülnek.

A **`pveam`** (Proxmox VE Appliance Manager) parancsai:

**Lista frissítése:**

```bash
pveam update
```

**Elérhető sablonok listázása:**

```bash
pveam available --section system
```

**Sablon letöltése (pl. a `local` tárolóra):**

```bash
pveam download local debian-10.0-standard_10.0-1_amd64.tar.gz
```

**Letöltött sablonok listázása:**

```bash
pveam list local
```

**Sablon törlése:**

```bash
pveam remove local:vztmpl/debian-10.0-standard....
```

## 11.4 Konténer Beállítások (Mélyfúrás)

### Általános és Biztonsági Beállítások

- **Unprivileged (Kiváltságok nélküli – ALAPÉRTELMEZETT):** A konténeren belüli `root` felhasználó (UID 0) a hoston egy nem privilegizált felhasználóhoz van rendelve. Ha egy támadó kitör a konténerből, a hoston egy jogok nélküli felhasználó (**nobody**) szerepében találja magát. (**Feltétel:** a konténerben a systemd verziónak $\ge 220$-nak kell lennie).
- **Privileged (Kiváltságos):** **Csak 100%-ig megbízható környezetben használandó!** A konténer `root` felhasználója a hoston is `root` jogosultságokkal bír. Csak akkor használd, ha olyan speciális hardverhez (pl. fizikai USB vagy blokkeszköz) kell hozzáférni, amit az unprivileged mód megakadályozna.

### CPU és Memória

- **CPU korlátozások (cgroups):** A `cores` a konténer által látott magok számát adja meg (a hoszton kiosztott magok ellenőrzése: `pct cpusets`). A `cpulimit` tört szám is lehet (pl. `0.5` = fél magnyi maximális processzoridő). A `cpuunits` (alapból `100` a cgroup v2-ben) a súlyozás: a magasabb értékkel rendelkező konténer prioritást élvez processzor-szűkület esetén.
- **Memória és Swap:** Külön vezérelhető a RAM (`memory`) és a lapozófájl (`swap`). A `swap` értéke a RAM-hoz adódik hozzá (a kernel `memory.memsw.limit_in_bytes` értéke = `memory` + `swap`).

### Csatolási Pontok (Mount Points: `rootfs` és `mp0`-`mp255`)

Egy konténerhez akár **256** extra tárhely csatolható.

- **Tároló-alapú (Storage Backed):** A Proxmox által kezelt `raw` lemezképek, ZFS subvolume-ok vagy könyvtárak.
  - **Lefoglalás parancssorból:**

```bash
pct set 100 -mp0 thin1:10,mp=/path/in/container
```

(Létrehoz egy 10GB-os lemezt a `thin1` tárolón, és becsatolja a konténerbe).

- **Bind Mounts:** A host fizikai fájlrendszerének közvetlen becsatolása a konténerbe (pl. `/mnt/bindmounts/shared,mp=/shared`).

> **Figyelem:** **SOHA ne csatolj be system mappákat** (pl. `/`, `/var`), mert az súlyos biztonsági kockázat!
> **Kivétel:** Ezekről a mappákról a `vzdump` **NEM** készít biztonsági mentést.

- **Device Mounts:** Fizikai blokkeszközök közvetlen átadása.

**Haladó Mount Point Opciók:**

- **`idmap`:** Speciális UID/GID újra-térképezés (override) egy konkrét csatolási ponthoz. Szintaxis: `típus:konténer_id:lemez_id:tartomány`.
  - **Példa 1:** `u:123:456:1` (A konténer 123-as UID-ja a fizikai lemezen a 456-os UID-t kapja).
  - **Példa 2:** `g:100:50:5` (A konténer 100-104-es csoportjai a lemezen az 50-54-es GID-eket kapják).
  - **Példa 3:** `passthrough` (A konténerbeli és a lemezen lévő azonosítók 1:1-ben megegyeznek).
- **`quota=1`:** User kvóták bekapcsolása (ZFS subvolume-okon **nem támogatott**).
- **`acl=1`:** Hozzáférés-vezérlési listák (POSIX ACL) engedélyezése.
- **`replicate=0` / `backup=0`:** Replikáció vagy mentés kihagyása.

### Hálózat (`net0` – `net9`)

Maximum **10** hálózati kártya definiálható.

- **`host-managed=1`:** A hálózati kártya (IP) konfigurációját a host kezeli, a konténeren belül **TILOS** saját DHCP klienst vagy Network Managert futtatni. (Alkalmazás-konténereknél hasznos).
- **Egyéb opciók:** `bridge`, `firewall`, `gw`/`gw6`, `ip`/`ip6`, `hwaddr` (MAC), `mtu`, `rate` (sávszélesség-korlát), `tag` (VLAN), `trunks`.

### Speciális Képességek (`features` paraméter)

Ezek az opciók kritikusak, ha összetett szolgáltatásokat (Docker, SMB, FUSE) futtatnál LXC-ben:

- **`nesting=1`:** Engedélyezi a `procfs` és `sysfs` tartalmát a konténer felé. **Kötelező**, ha a konténerben lévő systemd szolgáltatásokat izolálni akarod, vagy Dockert futtatsz.
- **`keyctl=1`:** A `keyctl()` system call engedélyezése (unprivileged gépeken alapból tiltva van). **Kötelező** a Docker vagy a `systemd-networkd` futtatásához unprivileged módban (anélkül végzetes hibát dobnak).
- **`fuse=1`:** FUSE fájlrendszerek engedélyezése. (**Veszély:** FUSE használata esetén I/O deadlock alakulhat ki a konténer fagyasztásakor, pl. snapshot vagy backup során, így használata **szigorúan ellenjavallt!**)
- **`mknod=1`:** Device node-ok létrehozásának engedélyezése ($\ge 5.3$ kernel kell hozzá, kísérleti funkció).
- **`mount=ext4;nfs`:** Konkrét fájlrendszertípusok felcsatolásának engedélyezése a konténerből.

### Automatikus indítás és Hook scriptek

**Indítás:**

```bash
pct set 100 -onboot 1
```

**Finomhangolás:** Start/Shutdown order (kisebb szám indul hamarabb, leállítás fordított sorrendben), Startup delay, Shutdown timeout (alapból **60 másodpercet** vár, mielőtt kilövi a konténert).

**`hookscript`:** Bármely életciklus-szakaszban egyéni scripteket hívhatsz meg (pl. `-hookscript local:snippets/hookscript.pl`).

## 11.5 Biztonság, AppArmor és Cgroups

**AppArmor:** A rendszerhívások (pl. `mount`) korlátozására szolgál. Hibakereséshez:

```bash
dmesg | grep apparmor
```

Teljesen kikapcsolható a konfigurációs fájlban az `lxc.apparmor.profile = unconfined` sorral, de ez éles környezetben (**production**) **óriási biztonsági rés**, mert konténerből való kitörést (**escape exploit**) eredményezhet.

**Cgroups (v2):** A Proxmox VE 9.0 óta csak a **cgroupv2** él. Ennek nagy előnye, hogy a RAM és a Swap limitje végre függetlenül állítható be. Azonban a v2-ben a fájlrendszer-kvóták (**quotas**) jelenleg **nem támogatottak!** Ha olyan régi OS sablont használsz (pl. CentOS 7, Ubuntu 16.10), ami nem ismeri a cgroup v2-t, a konténer **nem fog elindulni**. (**Megoldás:** OS frissítés, vagy systemd backport telepítése).

## 11.6 Vendég OS Konfiguráció módosítása (A `.pve-ignore` mágia)

A Proxmox indításkor automatikusan belenyúl a konténer fájljaiba, hogy beállítsa a hálózatot, hostnevet, DNS-t.

A beírások mindig a `# --- BEGIN PVE ---` és `# --- END PVE ---` közé kerülnek.

**Felülbírálás:** Ha meg akarod akadályozni, hogy a Proxmox felülírja mondjuk a `/etc/hosts` fájlodat, hozz létre egy üres fájlt a mellé `.pve-ignore.` előtaggal! Példa:

```bash
touch /etc/.pve-ignore.hosts
```

A teljes OS-szintű automatika kikapcsolható a konfigurációban: `ostype: unmanaged` (ilyenkor semmit nem módosít az indulásnál a Proxmox).

## 11.8 Biztonsági mentés (`vzdump`) és Helyreállítás (Restore)

A konténerekről a `vzdump` eszközzel lehet mentést készíteni. A kimentett archívumból a konfiguráció utólag is kinyerhető a `pvesm extractconfig` paranccsal.

Helyreállításnál (`pct restore`) két mód létezik:

- **"Simple" (Egyszerű) mód:** A mentésből származó Mount Point konfigot próbálja visszaállítani. Létrehozza a lemezeket a cél tárolón, kicsomagolja az adatokat, és beállítja a Bind/Device mountokat (anélkül, hogy oda adatokat bontana ki).
- **"Advanced" (Haladó) mód:** Ha a `pct restore` parancsnak kézzel megadod a `rootfs` vagy bármely `mpX` paramétert, a Proxmox teljesen figyelmen kívül hagyja a mentésben lévő tárolóbeállításokat, és az általad parancssorban megadott új útvonalakra, új tárolókra (Storage) bontja ki az adatokat.

## 11.9 Konténer menedzsment: A `pct` CLI parancsok

A grafikus felület mellett a `pct` parancs teljes hozzáférést ad.

**Konzol megnyitása (getty-n keresztül):**

```bash
pct console 100
```

**Közvetlen Root Shell (LXC Namespace belépés jelszó nélkül!):**

```bash
pct enter 100
```

**Konfiguráció megjelenítése:**

```bash
pct config 100
```

**Beállítások melegcseréje (Hot-plug):**

```bash
pct set 100 -net0 name=eth0,bridge=vmbr0,ip=192.168.15.147/24,gw=192.168.15.1
```

vagy RAM csökkentése:

```bash
pct set 100 -memory 512
```

(Amelyik opciót nem lehet hot-plugolni, az a GUI-n piros **"pending"** (függő) állapotba kerül, és csak újraindítás után lép életbe).

**Kötet (lemez) áthelyezése másik tárolóra:**

```bash
pct move-volume 100 mp0 masik-storage
```

**Kötet (lemez) átadása egy MÁSIK konténernek:**

```bash
pct move-volume 100 mp0 --target-vmid 200 --target-volume mp1
```

**Törlés (Purge):**

```bash
pct destroy 100 --purge
```

A `pct destroy 100 --purge` nemcsak a gépet törli, hanem kiszedi a tűzfalszabályokból, a replikációs jobokból, a backup ütemezésekből és a HA szabályokból is.

### Hibakeresés (Debugging) indulási problémáknál

Ha a konténer nem hajlandó elindulni, a következő parancsok adnak mélyreható debug logot:

```bash
pct start 100 --debug
# Vagy közvetlenül az lxc-start paranccsal:
lxc-start -n 100 -F -l DEBUG -o /tmp/lxc-100.log
```

(Az utóbbi parancs foreground (előtér) módban indul, leállításához egy másik terminálból kell kiadni a `pct stop 100` parancsot).

## 11.10 Migráció (Offline és Restart mód)

A konténereket (a VM-ekkel ellentétben) technikai korlátok miatt **nem lehet élőben migrálni** (**Live Migration**). Két opciód van:

- **Offline Migration:** Kikapcsolt állapotban. Ha a lemez lokális, átmásolja a hálózaton a cél node-ra.
- **Restart Migration:** Futó gépnél a `pct migrate <ctid> <target> --restart` kapcsolóval. A gép gyorsan leáll, átkerül a konfig, és azonnal elindul a célgépen. Mivel a konténerek bootolása ezredmásodpercekben mérhető, ez a módszer sokszor csak minimális, észrevehetetlen leállással (**downtime**) jár.

## 11.11 – 11.12 Nyers konfiguráció és Zárolások (Locks)

- **A konfigurációs fájl (`/etc/pve/lxc/<CTID>.conf`):** Egy `Kulcs: érték` alapú szövegfájl. Bár a Proxmox GUI és CLI elfedi, ide közvetlenül beírhatsz nyers, alacsony szintű LXC paramétereket is. Ezeket vagy `lxc.init_cmd: /sbin/my_own_init` vagy `lxc.init_cmd = /sbin/my_own_init` formátumban kell beírni, amit a rendszer egy az egyben átad az LXC motornak.
- **Pillanatképek (Snapshots):** A VM-ekhez hasonlóan a konfigurációs fájl alján, egy `[snapshotnév]` szekcióban tárolódik el a pillanatképkori állapot, a `parent` és a `snaptime` (Unix epoch) időbélyeggel.
- **Zárolások (Locks):** Replikáció, mentés (`vzdump`) vagy migráció során a konténer zárolást (**lock**) kap. Ha áramszünet miatt a zárolás beragad, feloldható:

```bash
pct unlock <CTID>
```

(De csak akkor használd, ha 100%-ig biztos vagy benne, hogy a zárolást kiváltó folyamat már nem fut a háttérben!)
