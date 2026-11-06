# 2. Fejezet: A Proxmox VE Telepítése (Installing Proxmox VE)

A telepítési folyamat bár egyszerűnek tűnik, a motorháztető alatt rengeteg olyan paramétert lehet finomhangolni, amelyek később meghatározzák a teljes szerver teljesítményét. A hivatalos lemezkép (ISO) egy teljes Debian GNU/Linux alaprendszert és a Proxmox VE csomagokat is tartalmazza, de meglévő Debianra is felhúzható a rendszer.

## 2.1 Rendszerkövetelmények (System Requirements)

A dokumentáció élesen elválasztja a tesztkörnyezetet a produkciós (éles) környezettől.

**Minimum (csak tesztelésre):**
64-bites x86 (Intel VT / AMD-V flaggel) vagy ARM64 (ARMv8-A) processzor, $1$ GB RAM, merevlemez, és $1$ db hálózati kártya (NIC).

**Ajánlott (éles szervertermi környezetbe):**

- **Processzor:** Intel 64 / AMD64 (VT/AMD-V) vagy 64-bites ARM (ARMv9-A). A **PCI(e) passthrough**-hoz (pl. videókártya vagy fizikai HBA kártya átadásához a VM-nek) a BIOS-ban be kell kapcsolni az **IOMMU** (Intel VT-d vagy AMD-Vi) támogatást.
- **Memória:** Alapból $2$ GB a Proxmox szolgáltatásainak, plusz a VM-ek memóriaigénye. Ha **ZFS**-t vagy **Ceph**-et használsz, minden $1$ TB nyers (RAW) tárhely után további $1$ GB RAM ajánlott a gyorsítótárazáshoz.
- **Hálózat:** Redundáns Multi-Gbit hálózati kártyák, külön portokkal a menedzsment, a cluster (**Corosync**) és a tároló forgalom számára.
- **Tárolás:**
  - **Hardveres RAID használata esetén** (pl. az OS számára): Akkumulátoros írási gyorsítótárral (BBU) rendelkező vezérlő (pl. a Dell R530-as szerverekben a PERC H330/H730-as vezérlők RAID módja) megfelelő ext4 vagy XFS partíciókhoz.
  - **ZFS vagy Ceph használata esetén:** A hardveres RAID **szigorúan kerülendő**. A RAID vezérlőt **HBA (IT) módba** kell flashelni vagy kapcsolni, hogy a ZFS közvetlenül, beavatkozás nélkül érje el a nyers lemezeket. A **PLP (Power-Loss-Protection)** funkcióval rendelkező Enterprise SSD-k használata erősen ajánlott.
- **ARM64 támogatás:** A rendszer natívan támogatja az ARM64-et, de csak UEFI-n keresztül bootolva, ACPI hardverleírással. A Raspberry Pi-szerű, csak Device-Tree-t használó eszközök **nem támogatottak**. ARM-on nincs SeaBIOS, minden VM UEFI-n keresztül bootol. Fontos, hogy az élő migráció (Live Migration) csak azonos architektúrájú (x86 $\rightarrow$ x86 vagy arm64 $\rightarrow$ arm64) szerverek között működik.

A gép processzor- és I/O-teljesítményének gyors tesztelésére a dokumentáció a következő beépített parancsot ajánlja:

```bash
pveperf
```

### 2.1.5 Támogatott webböngészők

A grafikus felület eléréséhez a **Firefox**, **Chrome**, illetve az aktuális **Edge** és **Safari** (az adott év kiadásai, vagy ESR verziók) támogatottak. Ha mobilról lépsz be, a Proxmox automatikusan egy könnyített, érintőképernyőre optimalizált felületet ad.

## 2.2 Telepítő média előkészítése

A letöltött hibrid ISO fájl kiírható CD/DVD-re, vagy USB pendrive-ra (ez utóbbi a javasolt). A **UNetbootin** használata tilos, mert nem kompatibilis a Proxmox ISO-jával.

**Pendrive elkészítése Linuxon:**
Keresd meg a pendrive eszköznevét, majd írd ki bithelyesen (az adatok törlődnek):

```bash
lsblk
dd bs=1M conv=fdatasync if=./proxmox-ve_*.iso of=/dev/XYZ
```

(Az `XYZ` helyére a megfelelő azonosító, pl. `sdb` kerül).

**macOS-en:**

```bash
hdiutil convert proxmox-ve_*.iso -format UDRW -o proxmox-ve_*.dmg
diskutil list
diskutil unmountDisk /dev/diskX
sudo dd if=proxmox-ve_*.dmg bs=1M of=/dev/rdiskX
```

(Az `rdiskX` használata a `diskX` helyett felgyorsítja az írást).

**Windowson:** Etcher vagy Rufus ajánlott, utóbbinál feltétlenül a **DD módot** kell választani.

## 2.3 A Proxmox VE telepítője és Haladó Opciók

A telepítő törli a céllemezen lévő összes adatot. A boot menüben az alábbi opciók választhatók:

- **Install Proxmox VE (Graphical):** A normál grafikus varázsló.
- **Install Proxmox VE (Terminal UI):** Karakteres felület (Ncurses), hasznos régi vagy extrém új hardvereknél, ha a grafikus driver hibára futna. (Létezik Serial Console verzió is, ha teljesen "headless" a gép).
- **Advanced Options $\rightarrow$ Rescue Boot:** Egy már telepített, de boot-hibás (pl. GRUB problémás) Proxmox megmentéséhez a lemezen lévő rendszert az ISO kernelével bootolja be.
- **Advanced Options $\rightarrow$ Test Memory:** memtest86+ futtatása (csak x86-on, Secure Boot nélkül).

A grafikus (vagy terminálos) varázsló lépései között található a Céllemez (Target harddisk) kiválasztása. Itt az **Options** gombra kattintva érhetők el a kritikus haladó beállítások.

**Haladó LVM Opciók (ext4 vagy XFS esetén):**
A rendszer a `pve` Volume Groupot, és a `root`, `data` (LVM-thin), valamint `swap` Logical Volume-okat hozza létre.

- **`hdsize`:** A lemezből felhasznált maximális méret. A maradék üresen marad további manuális partícionálásra.
- **`swapsize`:** A swap mérete (alapértelmezetten a RAM méretével megegyezik, de min. $4$ GB, max. $8$ GB). Ha $0$-ra állítod, nem lesz swap.
- **`maxroot`:** A gyökérfájlrendszer (`/`) maximális mérete (alapértelmezetten a `hdsize` $1/4$-e, max $96$ GB).
- **`maxvz`:** Az LVM-thin VM-tároló maximális mérete.
- **`minfree`:** A `pve` Volume Groupban szabadon hagyott hely (snapshotokhoz kellhet normál LVM esetén, bár LVM-thin-nél nem kritikus). $128$ GB-nál nagyobb lemez esetén $16$ GB, alatta a lemez $1/8$-a.

**Haladó ZFS Opciók:**
Ha nincs hardveres RAID vezérlő, a ZFS beépített szoftveres RAID szintjei választhatók (RAID0, RAID1, RAID10, RAIDZ-1, 2, 3).

- **`ashift`:** A 4K szektoros lemezek miatt ezt $12$-re érdemes állítani az optimális teljesítményhez.
- **`compress`:** A tömörítés bekapcsolása (LZ4 a javasolt).
- **ARC max size:** A ZFS memóriahasználatának (Adaptive Replacement Cache) limitálása, hogy hagyjon memóriát a VM-eknek.

A ZFS teljesítménye jelentősen javítható egy dedikált **ZIL (ZFS Intent Log)** SSD hozzáadásával a telepítés után:

```bash
zpool add <pool-name> log </dev/path_to_fast_ssd>
```

**Haladó BTRFS Opciók (Technológiai előzetes):**
`compress`: Támogatott a `zlib`, `lzo`, vagy `zstd`.

**A hálózati beállítások (Management Network Configuration):**
A telepítő során egyetlen statikus IP-címet, hálózati maszkot, átjárót és DNS szervert kell megadni, amely a gép menedzsment interfészéhez (`vmbr0`) kapcsolódik. IPv4 és IPv6 is támogatott, de dual-stack módot (mindkettőt) csak a telepítés után, a konfigurációs fájlból lehet beállítani.

**Hibaelhárítás: A `nomodeset` kernel paraméter**
Ha a telepítő lefagy grafikus hibák miatt (régi vagy extrém új GPU-k), a GRUB menüben nyomj egy `e` betűt a Terminal UI bejegyzésen, keresd meg a `linux` szóval kezdődő sort, és a végére szóközzel elválasztva írd be: `nomodeset`. Indítás a `Ctrl-X` vagy `F10` megnyomásával.

### Jelszó és e-mail megadása a telepítőben (2.3-as rész)

A `root` felhasználó jelszavának legalább **8 karakterből** kell állnia (de **12 ajánlott**). Az e-mail címet a rendszer a fontos értesítésekhez használja, például csomagfrissítések, sikertelen biztonsági mentések, HA (magas rendelkezésre állás) események és cron hibák küldésére.

### Hibaelhárítás telepítésnél (Installation Failure)

Ha a telepítés bármilyen okból meghiúsul, a **`CTRL+ALT+F2`** billentyűkombinációval a második TTY (virtuális terminál) ablakra váltva lehet megnézni a konkrét hibaüzeneteket és a logokat.

### 2.3.1 Hozzáférés a menedzsment felülethez a telepítés után

Miután a gép újraindult, a **`https://<IP>:8006`** címen éred el a felületet. A belépéshez a `root` felhasználót és a **Linux PAM** standard authentication (**Realm**) modult kell használni. A dokumentáció kiemeli, hogy első belépés után érdemes azonnal ellenőrizni az IP-t, a gépnevet, az időzónát és a tűzfal beállításait, valamint feltölteni az előfizetési kulcsot (ha van), vagy átállítani a csomagtárolókat (**repository**-kat).

### ZFS további haladó beállításai (2.3.3-as rész)

A korábban említett memórialimit, tömörítés és `ashift` mellett két ZFS opció maradt ki a telepítőből:

- **`checksum`:** Itt adható meg, milyen ellenőrzőösszeg-algoritmust használjon az `rpool`.
- **`copies`:** A ZFS adatmásolat paramétere, ami adatduplikációt jelent ugyanazon a lemezen belül, de a dokumentáció figyelmeztet, hogy ez **nem helyettesíti a fizikai redundanciát** (pl. a RAID-et).

### BTRFS további haladó beállítása (2.3.4-es rész)

A tömörítésen túl itt is beállítható a `hdsize` a lemezterület limitálására, így hagyhatsz üres helyet, ha később mondjuk egy hagyományos swap partíciót szeretnél létrehozni (mivel BTRFS alatt **nem jön létre automatikusan swap**).

## 2.4 Automatizált telepítés (Unattended Installation)

Nagyobb infrastrukturális kiépítéseknél a Proxmox automatikusan, emberi beavatkozás nélkül is telepíthető egy "answer file" (válaszfájl) segítségével, amiben filter szabályok alapján dől el, melyik lemezre és hálózati kártyára kerüljön a rendszer. Erre utólag jól illeszthetők Ansible automatizációk.

## 2.5 Telepítés meglévő Debian rendszerre

A Proxmox VE tulajdonképpen egy Debian csomaghalmaz. Bár nem ajánlott kezdőknek, haladó rendszeradminisztrátorok a hálózat és a tárhelyek kézi beállítása után felhúzhatják egy tiszta Debianra is:

```bash
apt-get update
apt-get install proxmox-ve
```
