# A Linux Swap (Lapozóterület) Kezelése és Beállítása

A Linux a fizikai memóriát (RAM) memórialapokra (pages) osztja. A lapozás (swapping) az a folyamat, amikor a rendszer egy memórialapot a merevlemezen lévő, előre konfigurált területre (swap space) másol, hogy memóriát szabadítson fel. A fizikai memória és a swap terület együttes mérete adja a rendszer virtuális memóriáját.

A swap használható a memória kiterjesztésére (az OOM - Out of Memory folyamatleállások elkerülésére), valamint a lemezre történő hibernálás (suspend-to-disk) biztosítására.

## A Swap állapotának ellenőrzése

```bash
# Aktív swap területek listázása
swapon --show

# Fizikai memória és swap használat együttes megjelenítése
free -h
```

## 1. Swap Partíció (Swap partition)

A swap partíció egy dedikált lemezterület. Létrehozásához (a particionáló szoftverekben) MBR esetén a 82-es típus, GPT esetén a Linux swap GUID (0657FD6D...) használandó.

**Létrehozás és aktiválás:**

```bash
# Swap terület formázása a partíción (MINDEN ADAT ELVÉSZ RAJTA!)
sudo mkswap /dev/sdxy

# Swap partíció bekapcsolása
sudo swapon /dev/sdxy
```

**Kikapcsolás (Deaktiválás):**

```bash
# Adott swap terület kikapcsolása
sudo swapoff /dev/sdxy

# AZ ÖSSZES aktív swap kikapcsolása
sudo swapoff -a
```

**Automatikus csatolás bootoláskor (`/etc/fstab`):**

A fájlba a partíció UUID-jét érdemes beírni (amit a `blkid` paranccsal tudsz lekérdezni).

```plaintext
UUID=device_UUID none swap defaults 0 0
```

## 2. Swap Fájl (Swap file)

Ha nincs dedikált partíciónk, vagy szűkös a hely (pl. SSD-ken), a swap fájl sokkal rugalmasabb alternatíva. Bármikor létrehozható, átméretezhető vagy törölhető.

**Fájlrendszerek támogatása Swap fájlhoz:**

| Fájlrendszer | Támogatott? |
| --- | --- |
| Btrfs, ext4, F2FS, NTFS3, XFS, JFS | Igen (Megjegyzés: A Btrfs egyedi eljárást igényel!) |
| Bcachefs, NILFS2, ZFS | Nem |

**Létrehozás és aktiválás (pl. 4 GB-os fájl):**

```bash
# Fájl létrehozása és formázása egy lépésben
sudo mkswap -U clear --size 4G --file /swapfile

# Swap fájl bekapcsolása
sudo swapon /swapfile
```

**Automatikus csatolás (`/etc/fstab`):**

> ⚠️ **Fontos:** A swap fájlt a fájlrendszerbeli elérési útjával kell megadni, NEM UUID-vel vagy LABEL-lel!

```plaintext
/swapfile none swap defaults 0 0
```

**Swap fájl eltávolítása:**

```bash
sudo swapoff /swapfile
sudo rm -f /swapfile
# Ezután töröld a vonatkozó sort az /etc/fstab fájlból is.
```

## 3. Teljesítményhangolás (Performance Tuning)

A swap műveletek jóval lassabbak, mint a RAM közvetlen elérése. Az alábbi kernel paraméterekkel szabályozható a rendszer memóriakezelési stratégiája.

### A. Swappiness (Lapozási hajlandóság)

A `vm.swappiness` paraméter határozza meg, hogy a kernel mennyire preferálja a swap használatát a fájl-gyorsítótárak (file cache) ürítésével szemben, amikor fogy a memória.

**Értéktartomány:** 0 - 200 (Alapértelmezett: 60).

- **Alacsony érték (pl. 1-10):** A kernel kerüli a swap használatát, inkább a megnyitott fájlok memóriáját szabadítja fel (asztali rendszereknél jobb reszponzivitást adhat).
- **Magas érték (100+):** A kernel előszeretettel használja a swapot. (ZRAM vagy gyors SSD esetén hasznos).

**Beállítás menete:**

```bash
# Jelenlegi érték lekérdezése
sysctl vm.swappiness

# Ideiglenes beállítás (újraindításig él)
sudo sysctl -w vm.swappiness=35
```

**Végleges beállítás:** Hozz létre egy fájlt pl. `/etc/sysctl.d/99-swappiness.conf` néven:

```plaintext
vm.swappiness = 35
```

### B. VFS Cache Pressure (Gyorsítótár nyomás)

A `vm.vfs_cache_pressure` (Alapértelmezett: 100) azt szabályozza, milyen gyorsan ürítse a kernel a fájlrendszeri (inode/dentry) gyorsítótárakat.

- Asztali gépeken gyakran javasolják az érték 50-re csökkentését, hogy a fájlrendszer böngészése gyors maradjon.
- Sok apró, ritkán használt fájl esetén a magasabb érték a jobb.

### C. Swap Prioritás és Striping (Round-robin)

Ha több swap területtel rendelkezel (pl. egy gyors NVMe-n és egy lassabb HDD-n), prioritásokat oszthatsz ki nekik 0-tól 32767-ig. A rendszer mindig a legmagasabb prioritású területet használja először.

**Fstab beállítás (`pri=` paraméterrel):**

```plaintext
UUID=gyors_lemez_uuid none swap defaults,pri=100 0 0
UUID=lassu_lemez_uuid none swap defaults,pri=10  0 0
```

**Ideiglenes bekapcsolás prioritással:**

```bash
sudo swapon --priority 100 /dev/sda1
```

**Striping (Sávosítás):** Nincs szükség RAID-re a swap gyorsításához. Ha több, ugyanolyan sebességű lemezen lévő swap partíciónak azonos prioritást adsz, a kernel automatikusan "round-robin" (felváltva író) módszerrel, párhuzamosan fogja használni őket, ami növeli a teljesítményt.

##Forrás
-[https://wiki.archlinux.org/title/Swap]
-[https://linuxconfig.org/how-to-increase-swap-space-on-linux]
