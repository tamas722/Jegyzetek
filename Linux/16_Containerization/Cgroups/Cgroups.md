# Cgroups jegyzet – Red Hat Enterprise Linux 7

## 1. Bevezetés

A control groups (cgroups) a Linux kernel egyik funkciója, amely lehetővé teszi, hogy hierarchikusan csoportosított processzek között erőforrásokat osszunk el (CPU idő, memória, hálózati sávszélesség stb.). A rendszergazda így finoman szabályozhatja az erőforrások kiosztását, priorizálását, megtagadását és monitorozását.

- **Cél:** alkalmazások és felhasználók közötti erőforrás-elosztás javítása.
- **Régi megközelítés:** minden processz hasonló erőforrást kapott, amit a niceness értékkel lehetett módosítani; a sok processzből álló alkalmazások aránytalanul több erőforrást kaptak.
- **RHEL 7 újdonság:** az erőforrás-kezelés a processz szintről az alkalmazás szintjére került; a cgroup hierarchiák a systemd unit fával kapcsolódnak össze. Így `systemctl` parancsokkal vagy unit fájlok módosításával kezelhetők.
- **Régi eszköz:** a `libcgroup` csomag `cgconfig` parancsa deprecated, nem ajánlott, mert ütközhet a systemd alapértelmezett hierarchiájával. Kivétel: olyan vezérlők, amiket a systemd még nem támogat (pl. `net_prio`).
- **Fő vezérlők:** cpu, memory, blkio.

## 2. Alapértelmezett cgroup hierarchiák

A systemd automatikusan létrehoz egy slice – scope – service unit hierarchiát, és a fontos kernel erőforrás-vezérlőket a `/sys/fs/cgroup/` alá mountolja.

> ⚠️ **Figyelmeztetés:** a libcgroup eszközöket soha ne használd a systemd által mountolt alapértelmezett hierarchiák módosítására – váratlan viselkedést okoz. A libcgroup a jövőben eltűnik.

### Systemd unit típusok

Minden processz a systemd init gyermeke. Három, erőforrás-kezelésre használt unit típus:

- **Service** – systemd által indított processz vagy processzcsoport. Név: `name.service`
- **Scope** – külsőleg létrehozott processzek csoportja (pl. user sessionök, konténerek, VM-ek). Név: `name.scope`
- **Slice** – hierarchikusan szervezett unitok csoportja; maga nem tartalmaz processzt. Név: `parent-name.slice` (a `-` a path elválasztó). Root slice: `-.slice`

A service/scope/slice unitok közvetlenül a cgroup fában lévő objektumokra képeződnek. Pl. `ex.service` a `test-waldo.slice`-ban → `test.slice/test-waldo.slice/ex.service/`.

**Alapértelmezett slice-ok:**

- `-.slice` – root slice
- `system.slice` – rendszer-szolgáltatások
- `user.slice` – felhasználói sessionök
- `machine.slice` – VM-ek és Linux konténerek

Minden user session külön scope unitba kerül, a VM-ek és konténerek is. Minden felhasználó kap egy implicit subslice-t. A rendszergazda új slice-okat definiálhat.

**Példa cgroup fára:**

```plaintext
├─1 /usr/lib/systemd/systemd ...
├─user.slice
│ └─user-1000.slice
│   └─session-1.scope
│     ├─11459 gdm-session-worker ...
│     ...
└─system.slice
  ├─systemd-journald.service
  │ └─422 /usr/lib/systemd/systemd-journald
  ├─bluetooth.service
  ...
```

A service-ek és scope-ok tartalmaznak processzeket, a slice-ok nem. Kivétel: PID 1 a `-.slice`-ban van (ez implicit a fa gyökere).

A service és slice unitok konfigurálhatók perzisztens unit fájlokkal vagy dinamikusan API hívásokkal. A scope unitok csak dinamikusan hozhatók létre. A dinamikusan létrehozott unitok tranziensek – leálláskor/újraindításkor eltűnnek.

## 3. Erőforrás-vezérlők a Linux kernelben

A resource controller (cgroup subsystem) egy erőforrást reprezentál (CPU, memória). A kerneltől függően több vezérlő van; a systemd automatikusan mountolja őket. Lista: `/proc/cgroups` vagy `lssubsys`.

**RHEL 7-ben alapértelmezett vezérlők:**

- **blkio** – blokk-eszköz I/O korlátok
- **cpu** – CPU scheduler hozzáférés (cpuacct-tal egy mounton)
- **cpuacct** – CPU használati riportok (cpu-val egy mounton)
- **cpuset** – CPU-k és memória node-ok hozzárendelése
- **devices** – eszközhozzáférés engedélyezése/tiltása
- **freezer** – processzek felfüggesztése/folytatása
- **memory** – memória korlátok és riportok
- **net_cls** – hálózati csomagok címkézése classid-del (tc, iptables)
- **perf_event** – perf tool monitorozás
- **hugetlb** – nagy méretű virtuális memória lapok és korlátok

## 4. További források (1.4)

**Telepített dokumentáció:**

- `systemd.resource-control(5)`, `systemd.unit(5)`, `systemd.slice(5)`, `systemd.scope(5)`, `systemd.service(5)`
- kernel-doc csomag (Optional channel): `/usr/share/doc/kernel-doc-<ver>/Documentation/cgroups/` – blkio, cpuacct, cpuset, devices, freezer, memory, net_cls
- CPU: `sched-rt-group.txt`, `sched-bwc.txt`

**Online:** RHEL 7 System Administrator's Guide, systemd D-Bus API

## 5. Control groups használata (2. fejezet)

A systemd szempontjából a cgroup egy system unithez kötött, unit fájllal konfigurálható. Az erőforrás-kezelés lehet tranziens vagy perzisztens.

### 5.1 Tranziens cgroup létrehozása – `systemd-run`

```plaintext
systemd-run --unit=name --scope --slice=slice_name command
```

- `--unit=name` – unit neve (egyedi futásidőben); ha nincs, automatikus
- `--scope` – scope unit service helyett (szinkron futás)
- `--slice=slice_name` – meglévő slice-ba helyezés (alap: system.slice)
- `command` – a futtatandó parancs a végén
- Egyéb: `--description`, `--remain-after-exit`, `--machine` (konténer)

**Példa:**

```plaintext
systemd-run --unit=toptest --slice=test top -b
→ Running as unit toptest.service
```

### 5.2 Perzisztens cgroup

Automatikus indításhoz: `systemctl enable` → unit fájl a `/usr/lib/systemd/system/`-ben. A cgroup perzisztens módosításához a unit fájlt kell módosítani.

### 5.3 Cgroup eltávolítása

Tranziens cgroup automatikusan felszabadul, ha a processzek befejeződnek. `--remain-after-exit` esetén marad.

- Leállítás: `systemctl stop name.service`
- Processzek kill: `systemctl kill name.service --kill-who=PID,... --signal=signal` (alap: SIGTERM)
- Perzisztens cgroup felszabadítása: `systemctl disable name.service` + konfig fájl törlése

### 5.4 Cgroup módosítása

Minden perzisztens unitnak van unit konfig fájlja a `/usr/lib/systemd/system/`-ben. Módosítás manuálisan vagy `systemctl set-property`-vel.

**`systemctl set-property`:**

```plaintext
systemctl set-property name parameter=value
```

- Azonnal alkalmazódik, a unit fájlba íródik (reboot után is megmarad)
- `--runtime` – csak tranziens
- Több property egyszerre is megadható

**Példa:**

```plaintext
systemctl set-property httpd.service CPUShares=600 MemoryLimit=500M
systemctl set-property --runtime httpd.service CPUShares=600 MemoryLimit=500M
```

### 5.5 Unit fájlok módosítása

A systemd service unit fájlok magas szintű paramétereket kínálnak, amelyek a kernel cgroup vezérlőkkel kommunikálnak.

**CPU kezelés:**

- `CPUShares=value` – CPU share-ek száma (alap: 1024). Növelés → több CPU idő. Automatikusan bekapcsolja a `CPUAccounting`-ot. A `cpu.shares` cgroup paramétert vezérli.
- `CPUQuota=value` – CPU idő kvóta százalékban (egy CPU összidejéhez viszonyítva). >100% → több CPU. A `cpu.max` (unified) és `cpu.cfs_quota_us` (legacy) attribútumot vezérli. Automatikusan bekapcsolja a `CPUAccounting`-ot.

**Példa CPU:**

```plaintext
/etc/systemd/system/httpd.service.d/cpu.conf:
[Service]
CPUShares=1500
CPUQuota=20%
→ systemctl daemon-reload && systemctl restart httpd.service
```

**Memória kezelés:**

- `MemoryLimit=value` – max memória (K, M, G, T suffix). `MemoryAccounting` szükséges. A `memory.limit_in_bytes` paramétert vezérli.

**Példa:**

```plaintext
[Service]
MemoryLimit=1G
```

**Block IO kezelés (`BlockIOAccounting` szükséges):**

- `BlockIOWeight=value` – 10–1000, alap: 1000
- `BlockIODeviceWeight=device_name value` – eszközönkénti súly (10–1000)
- `BlockIOReadBandwidth=device_name value` – olvasási sávszélesség korlát (K/M/G/T, alap: byte/s)
- `BlockIOWriteBandwidth=device_name value` – írási sávszélesség korlát

> **Fontos:** a blkio vezérlő nem támogatja a buffered write-ot – csak direct I/O-ra hat. A buffered read támogatott, a `BlockIOReadBandwidth` mindkét módban működik.

**Példa:**

```plaintext
[Service]
BlockIODeviceWeight=/home/jdoe 750
BlockIOReadBandwidth=/var/log 5M
```

**Egyéb erőforrások:**

- `DeviceAllow=device_name options` – eszközhozzáférés (r, w, m)
- `DevicePolicy=value` – strict / closed / auto (alap)
- `Slice=slice_name` – unit elhelyezése slice-ban (alap: system.slice); scope-oknál nem működik
- `ExecStartPost=command` – pl. swap tiltása: `memory.memsw.limit_in_bytes` beállítása

**Példa swap tiltás:**

```plaintext
ExecStartPost=/bin/bash -c "echo 1G > /sys/fs/cgroup/memory/system.slice/example.service/memory.memsw.limit_in_bytes"
```

### 5.6 Információ lekérése

- `systemctl list-units` – aktív unitok listája (oszlopok: UNIT, LOAD, ACTIVE, SUB, DESCRIPTION)
  - `--all` – inaktívak is
  - `-t slice` / `-t service,masked` – típus szerint
  - `systemctl list-unit-files` – telepített unit fájlok
- `systemd-cgls` – cgroup hierarchia megjelenítése (processzekkel)
  - `systemd-cgls name` – adott controller részfája
  - `systemctl status name` – unit részletei, cgroup subtree
- `machinectl` – Linux konténerek monitorozása
- `cat /proc/PID/cgroup` – processz cgroup hovatartozása
- `systemd-cgtop` – dinamikus erőforrás-használat (CPU, Memory, IO), mint a top

### 5.7 További források (2.5)

- Man: `systemd-run(1)`, `systemctl(1)`, `systemd-cgls(1)`, `systemd-cgtop(1)`, `machinectl(1)`, `systemd.kill(5)`
- kernel-doc: mint az 1.4-ben

## 6. libcgroup eszközök (3. fejezet)

A libcgroup csomag deprecated; csak olyan vezérlőkhöz használd, amiket a systemd még nem támogat (pl. `net_prio`). Ne használd az alapértelmezett vezérlőkhöz.

**Telepítés:**

```plaintext
yum install libcgroup libcgroup-tools
modprobe netprio_cgroup   # net_prio modul betöltése
```

### 6.1 Hierarchia mountolása

`cgconfig` service: a `/etc/cgconfig.conf` mount szekcióját módosítva perzisztensen mountolható. A service alapból nem indul; indításkor alkalmazza a konfigot, leállításkor unmountol.

```plaintext
mount {
    net_prio = /sys/fs/cgroup/net_prio;
}
→ systemctl restart cgconfig.service
```

**mount paranccsal (tranziens):**

```plaintext
mkdir /sys/fs/cgroup/net_prio
mount -t cgroup -o net_prio none /sys/fs/cgroup/net_prio
lssubsys -am   # ellenőrzés
```

### 6.2 Hierarchia unmountolása

- `cgconfig.conf`-ból a mount bejegyzés törlése + service restart
- vagy: `umount /sys/fs/cgroup/controller_name`

> **Figyelem:** csak a saját magad által mountolt hierarchiákat unmountold; alapértelmezett vezérlő detacholása rendszer-újraindítást igényelhet.

### 6.3 Cgroup létrehozása

```plaintext
cgcreate -t uid:gid -a uid:gid -g controllers:path
```

- `-t` – tasks pseudo-file tulajdonos (processz hozzáadás)
- `-a` – egyéb pseudo-file-ok tulajdonosa (erőforrás módosítás)
- `-g` – hierarchia + path (controller lista, majd `:path`)

Vagy `mkdir /sys/fs/cgroup/controller/name/child_name`

### 6.4 Cgroup törlése

```plaintext
cgdelete controllers:path
cgdelete -r controllers:path   # rekurzív
```

Törléskor a processzek a szülő csoportba kerülnek.

### 6.5 Cgroup paraméterek beállítása

`cgconfig.conf` Groups szekciója (perzisztens):

```plaintext
group name {
    [perm { task { uid=...; gid=...; } admin { uid=...; gid=...; } }]
    controller {
        param_name = param_value;
    }
}
```

Restart: `systemctl stop cgconfig` → szerkesztés → `systemctl start cgconfig`

**`cgset` (tranziens):**

```plaintext
cgset -r parameter=value path_to_cgroup
cgset --copy-from source target
```

A beállítható értékek függhetnek a hierarchiában feljebb beállítottaktól.

### 6.6 Processz mozgatása cgroupba

```plaintext
cgclassify -g controllers:path_to_cgroup pidlist
```

- `-g` nélkül a `/etc/cgrules.conf` alapján
- `--sticky` – gyermek processzek is maradnak
- `cgred` service (`cgrulesengd`) – automatikus mozgatás a `cgrules.conf` alapján

**`cgrules.conf` formák:**

- `user subsystems control_group;`
- `user:command subsystems control_group;`

**Jelölések:**

- `@` – csoport (pl. `@admins`)
- `\*` – minden
- `%` – ugyanaz, mint az előző sorban

**Példa:**

```plaintext
maria    net_prio    /usergroup/staff
maria:ftp devices     /usergroup/staff/ftp
@adminstaff net_prio /admingroup
@labstaff   %        %
```

### 6.7 Processz indítása cgroupban

```plaintext
cgexec -g controllers:path_to_cgroup command arguments
```

- `--sticky` – gyermekek maradnak

### 6.8 Információ lekérése

- `cat /proc/cgroups` – elérhető vezérlők
- `lssubsys -m controllers` – mount pontok
- `lscgroup` – cgroupok listája; `lscgroup cpuset:adminusers` – szűrés
- `cgget -r parameter list_of_cgroups` – paraméterek
- `cgget -g cpuset /` – összes paraméter

### 6.9 További források (3.9)

Man: `cgclassify(1)`, `cgclear(1)`, `cgconfig.conf(5)`, `cgconfigparser(8)`, `cgcreate(1)`, `cgdelete(1)`, `cgexec(1)`, `cgget(1)`, `cgsnapshot(1)`, `cgred.conf(5)`, `cgrules.conf(5)`, `cgrulesengd(8)`, `cgset(1)`, `lscgroup(1)`, `lssubsys(1)`

## 7. Alkalmazási példák (4. fejezet)

### 7.1 Adatbázis I/O priorizálása

Két adatbázis KVM guestben: egy magas, egy alacsony prioritású. Erőforrás-allokáció nélkül a két DB egyenlő I/O-t kap (a low priority indulásakor ~45 időpontban). Cgroupokkal 10:1 arány érhető el.

**Eljárás:**

```plaintext
systemctl set-property db1.service BlockIOAccounting=true
systemctl set-property db2.service BlockIOAccounting=true
systemctl set-property db1.service BlockIOWeight=1000
systemctl set-property db2.service BlockIOWeight=100
```

**Eredmény:** a két DB I/O throughputja 10:1 arányban oszlik meg. Alternatíva: block device I/O throttling a low priority DB-nél.

### 7.2 Hálózati prioritások fájlmegosztó szolgáltatásokhoz

A `net_prio` subsystem használatával (libcgroup, mert systemd nem támogatja):

1. `netprio_cgroup` modul betöltése
2. `cgconfig.conf` mount bejegyzés + cgroupok létrehozása (`samba_low`, `nfs_high`)
3. `cgrules.conf` bejegyzések a service-ekhez
4. `cgred` service indítása
5. smb és nfs service-ek indítása, ellenőrzés:

```plaintext
cat /sys/fs/cgroup/net_prio/samba_low/tasks
cat /sys/fs/cgroup/net_prio/nfs_high/tasks
```

Az NFS forgalma most magasabb prioritású, mint a Samba-é.

Hasonlóan használható kliens alkalmazásokhoz (pl. Firefox) is.

## Összefoglaló kulcspontok

| Téma | Lényeg |
| --- | --- |
| Cgroup | Kernel funkció erőforrás-elosztásra hierarchikus processzcsoportok között |
| systemd integráció | slice/scope/service unitok, systemctl, unit fájlok |
| Tranziens | systemd-run, futásidőben eltűnik |
| Perzisztens | unit fájl módosítás, systemctl set-property |
| Fő paraméterek | CPUShares, CPUQuota, MemoryLimit, BlockIO*, Device*, Slice |
| Monitorozás | systemctl list-units, systemd-cgls, systemd-cgtop, cat /proc/PID/cgroup |
| libcgroup | deprecated, csak nem támogatott vezérlőkhöz (pl. net_prio) |
| libcgroup parancsok | cgcreate, cgdelete, cgset, cgclassify, cgexec, cgget, lscgroup, lssubsys |
| Példák | DB I/O priorizálás (BlockIOWeight), hálózati prioritás (net_prio) |

## Forrás
-[https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/resource_management_guide/legal-notice]
-[https://www.man7.org/linux/man-pages/man7/cgroups.7.html]
