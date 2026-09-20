# Az LVM (Logical Volume Manager) Alapjai Linuxon

Az LVM (Logical Volume Manager) egy fejlett tárolókezelő rendszer a Linuxban. A legnagyobb előnye a hagyományos particionálással szemben a rugalmasság: lehetővé teszi a "partíciók" (logikai kötetek) méretének rendkívül egyszerű, menet közbeni növelését vagy csökkentését, valamint több fizikai lemez egyetlen nagy virtuális tárhellyé (pool) történő összevonását.

## 1. Az LVM Fő Komponensei

Az LVM három fő rétegre osztja a tárolást, amelyek egymásra épülnek:

- **PV - Physical Volume (Fizikai Kötet):** Maga a nyers, fizikai tárolóeszköz (pl. egy teljes merevlemez, mint a `/dev/sdb`, vagy egy partíció, mint a `/dev/sdb1`).

- **VG - Volume Group (Kötetcsoport):** Egy vagy több Physical Volume (PV) összevonásából létrehozott virtuális "tárolómedence" (pool). Ebből a medencéből lehet majd helyet osztani.

- **LV - Logical Volume (Logikai Kötet):** A Volume Groupból (VG) kihasított virtuális partíció. Erre kerül a fájlrendszer (pl. ext4), és ezt csatoljuk fel a rendszerbe.

## 2. A Lemez Előkészítése (Particionálás)

(Bár az LVM használható közvetlenül nyers lemezeken is, gyakori gyakorlat először partíciókat létrehozni a lemezen).

Példaként egy 16 GB-os új lemezt (`/dev/sdb`) két 8 GB-os partícióra osztunk a `parted` CLI eszközzel:

1. A lemez ellenőrzése: `lsblk`
2. A parted elindítása: `sudo parted /dev/sdb`
3. Parted parancsok a felületen belül:
   - `mklabel gpt` (GPT partíciós tábla létrehozása)
   - `unit GB` (Mértékegység átállítása GB-ra)
   - `mkpart primary 0 8GB` (Első 8 GB-os partíció létrehozása)
   - `mkpart primary 8GB 16GB` (Második 8 GB-os partíció létrehozása)
   - `print` (Ellenőrzés)
   - `quit` (Kilépés)

## 3. Fizikai Kötetek (PV) Létrehozása és Kezelése

Az LVM számára elsőként be kell "mutatnunk" az újonnan létrehozott partíciókat.

**PV létrehozása:**

```bash
sudo pvcreate /dev/sdb1
sudo pvcreate /dev/sdb2
```

**Aktív PV-k listázása és ellenőrzése:**

```bash
sudo pvs
```

(Egy PV eltávolításához a `sudo pvremove /dev/sdb1` parancs használható).

## 4. Kötetcsoport (VG) Létrehozása és Kezelése

A PV-kből most létrehozunk egy nagy közös "medencét" (Volume Group).

**VG létrehozása (pl. `foss_vg` néven) a két PV összevonásával:**

```bash
sudo vgcreate foss_vg /dev/sdb1 /dev/sdb2
```

**Aktív VG-k listázása:**

```bash
sudo vgs
```

**Kötetcsoport kibővítése:**

Ha később veszel egy új lemezt (pl. `/dev/sdb3`), hozzáadhatod a meglévő VG-hez, így növelve a kapacitását:

```bash
sudo vgextend foss_vg /dev/sdb3
```

## 5. Logikai Kötetek (LV) Létrehozása és Kezelése

A közös medencéből (VG) most már kiszabhatunk tetszőleges méretű logikai köteteket (LV), amelyeket partíciókként fogunk használni.

**LV létrehozása:**

(Szintaxis: `lvcreate -L [MÉRET_MB-ban] -n [NÉV] [VG_NEVE]`)

```bash
# 2048 MB-os kötet létrehozása 'foss_lv_01' néven a 'foss_vg'-ből
sudo lvcreate -L 2048 -n foss_lv_01 foss_vg

# Egy másik, 4096 MB-os kötet létrehozása 'foss_lv_02' néven
sudo lvcreate -L 4096 -n foss_lv_02 foss_vg
```

**Aktív LV-k listázása és részletei:**

```bash
sudo lvdisplay
```

## 6. Formázás, Felcsatolás és Fstab Bejegyzés

Az újonnan létrehozott LV-t pontosan úgy kell kezelni, mint egy hagyományos partíciót.

**Formázás ext4 fájlrendszerrel:**

A logikai kötetek útvonala általában `/dev/VG_NEVE/LV_NEVE` formátumú.

```bash
sudo mkfs.ext4 -m 0 /dev/foss_vg/foss_lv_01
```

**Felcsatolás (Mount):**

```bash
sudo mount /dev/foss_vg/foss_lv_01 /mnt
```

**Állandó felcsatolás (`/etc/fstab`):**

Nyisd meg szerkesztésre a `sudo nano /etc/fstab` fájlt, és add hozzá az alábbi sort:

```plaintext
/dev/foss_vg/foss_lv_01    /mnt    ext4    defaults    0    2
```

Mentés után mindig olvastasd újra a konfigurációt a systemd-vel:

```bash
sudo systemctl daemon-reload
```

## 7. Logikai Kötet Kiterjesztése és Törlése

Az LVM legnagyobb ereje a menet közbeni méretezés.

**Logikai kötet (LV) méretének növelése:**

```bash
# A kötet fizikai kiterjesztése (pl. +1200 MB-tal)
sudo lvextend -L +1200 /dev/foss_vg/foss_lv_01

# A fájlrendszer KITERJESZTÉSE, hogy felismerje az új méretet (ext2/ext3/ext4 esetén!)
sudo resize2fs /dev/foss_vg/foss_lv_01
```

**Logikai kötet (LV) törlése:**

(Fontos: Törlés előtt a kötetet le kell csatolni (`umount`), és biztosnak kell lenni abban, hogy nincs rajta fontos adat).

```bash
sudo lvremove /dev/foss_vg/foss_lv_01
```

##Forrás
-[https://itsfoss.com/lvm-guide/]

# Az LVM (Logical Volume Manager) Teljes Kézikönyve

Az LVM (Logical Volume Manager) egy alternatív, rugalmas tárolókezelési mechanizmus a hagyományos, statikus particionálással szemben. LVM használatakor a nyers lemezeket és partíciókat egy közös tárolómedencévé vonjuk össze, amelyből utána tetszőleges méretű és könnyen átméretezhető "logikai köteteket" (virtuális partíciókat) hasíthatunk ki.

> **Kivétel:** A `/boot` partícióhoz nem használható logikai kötet, mivel a legtöbb bootloader (pl. GRUB, systemd-boot) nem képes olvasni az LVM struktúrát.

## Hagyományos Particionálás vs. LVM

Az LVM három fő komponensből áll. Ebből kettő feleltethető meg a hagyományos rendszereknek:

| Hagyományos Particionálás | LVM Komponens |
| --- | --- |
| Lemezek (Disks) | Kötetcsoportok (Volume Groups - VG) |
| Partíciók (Partitions) | Logikai Kötetek (Logical Volumes - LV) |

(A harmadik komponens a Fizikai Kötet (PV), amely a nyers építőelemet jelenti, közvetlen hagyományos megfelelője nincs).

## 1. Fizikai Kötetek (Physical Volumes - PV)

A fizikai kötetek jelentik az LVM építőelemeit. Egy fizikai kötet lehet egy nyers, particionálatlan lemez (pl. `/dev/sdc`) vagy egy hagyományos partíció (pl. `/dev/sdd1`). Ezek inicializálása teszi lehetővé, hogy az LVM kezelni tudja őket.

(A PV-ket kezelő segédprogramok neve mindig `pv`-vel kezdődik).

**Fizikai kötetek létrehozása (inicializálása):**

Megadhatsz egyetlen nyers lemezt, vagy akár több partíciót is egyszerre:

```bash
sudo pvcreate /dev/sdc
sudo pvcreate /dev/sdd1 /dev/sdd2
```

**Elérhető fizikai kötetek listázása:**

Különböző szintű részletességgel kérhetők le az adatok:

- `sudo pvs` (A legátláthatóbb, táblázatos nézet)
- `sudo pvscan` (Rövid összefoglaló)
- `sudo pvdisplay` (Minden metaadat és részlet megjelenítése)

**Fizikai kötet eltávolítása:**

Törli az LVM címkét a lemezről, így az ismét normál meghajtóként viselkedik:

```bash
sudo pvremove /dev/sdd2
```

## 2. Kötetcsoportok (Volume Groups - VG)

A kötetcsoportok jelentik a következő absztrakciós szintet: fizikai kötetek (PV) összevonásából létrejött közös tárolómedencék (storage pools).

(A VG-ket kezelő segédprogramok neve mindig `vg`-vel kezdődik).

**Kötetcsoport létrehozása:**

Első argumentumként a VG nevét (pl. `lvm_tutorial`), majd a felhasználni kívánt fizikai köteteket kell megadni:

```bash
sudo vgcreate lvm_tutorial /dev/sdc /dev/sdd1
```

**Kötetcsoportok listázása:**

```bash
sudo vgs
# Bővebb információkhoz: sudo vgdisplay vagy sudo vgscan
```

**Egy adott VG-hez tartozó fizikai kötetek (PV-k) listázása:**

```bash
# A PV-k neveinek lekérdezése
sudo pvdisplay -S vgname=lvm_tutorial -C -o pv_name

# A PV-k számának lekérdezése
sudo vgdisplay -S vgname=lvm_tutorial -C -o pv_count
```

**Kötetcsoport kiterjesztése (Bővítés):**

Ha új lemezt vagy partíciót adsz a géphez, hozzáadhatod a meglévő VG-hez:

```bash
sudo vgextend lvm_tutorial /dev/sdd2
```

(A `vgextend` automatikusan PV-vé inicializálja az eszközt, ha az még nem volt az).

**Kötetcsoport csökkentése (PV eltávolítása):**

Kivesz egy vagy több fizikai kötetet a medencéből (csak akkor működik, ha a kivenni kívánt lemezen nincs aktív logikai kötet adata):

```bash
sudo vgreduce lvm_tutorial /dev/sdc /dev/sdd1
```

**Kötetcsoport törlése:**

```bash
sudo vgremove lvm_tutorial
```

## 3. Logikai Kötetek (Logical Volumes - LV)

A logikai kötetek felelnek meg a hagyományos partícióknak. A kötetcsoport (VG) szabad kapacitásából hasítjuk ki őket. Előnyük, hogy menet közben is növelhetők vagy csökkenthetők.

(Az LV-ket kezelő segédprogramok neve mindig `lv`-vel kezdődik).

**Logikai kötet létrehozása:**

A `-L` kapcsolóval a méretet (GB, MB, KB), az `-n` kapcsolóval a nevet határozzuk meg, végül megadjuk a forrás kötetcsoportot.

```bash
sudo lvcreate -L 5GB -n lv1 lvm_tutorial
```

**Formázás és Felcsatolás:**

A létrehozott logikai kötet elérési útja mindig `/dev/<vg_neve>/<lv_neve>` formátumú (a mi esetünkben: `/dev/lvm_tutorial/lv1`).

```bash
# Formázás ext4 fájlrendszerrel
sudo mkfs.ext4 /dev/lvm_tutorial/lv1

# Felcsatolás a /mnt könyvtárba
sudo mount -t ext4 /dev/lvm_tutorial/lv1 /mnt
```

**Logikai kötet átméretezése (Növelés):**

Először érdemes ellenőrizni, hogy van-e elég szabad hely a kötetcsoportban:

```bash
sudo vgs -S vgname=lvm_tutorial -o vg_free
```

A logikai kötet méretének növelése (pl. 2 GB-tal a `+` jel használatával):

```bash
sudo lvresize -L +2GB lvm_tutorial/lv1
# (Ugyanezt a feladatot az 'lvextend' parancs is elvégzi)
```

A fájlrendszer kiterjesztése az új méretre (élőben, lecsatolás nélkül is működik ext4 esetén):

```bash
sudo resize2fs /dev/lvm_tutorial/lv1
```

> **Tipp:** A menet közbeni - hot - átméretezést nem minden fájlrendszer támogatja. Erre a célra az ext4 és az xfs a legmegbízhatóbb.

**Logikai kötet törlése:**

Törlés előtt a kötetet mindig le kell csatolni. A parancs jóváhagyást kér a végleges törléshez.

```bash
sudo lvremove lvm_tutorial/lv1
```
##Forrás
-[https://linuxhandbook.com/lvm-guide/]
