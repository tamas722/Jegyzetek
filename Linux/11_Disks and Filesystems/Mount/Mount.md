# A Linux `mount` és `umount` Parancsok (Fájlrendszerek Csatolása)

A Linux fájlrendszer-hierarchiája egy fához hasonlít, amelynek a gyökérkönyvtára a `/` (root). A `mount` parancs lehetővé teszi további fájlrendszerek (pl. partíciók, USB meghajtók, hálózati megosztások, ISO képfájlok) csatlakoztatását ehhez a fához egy adott csatolási ponton (mount point).

## 1. A `mount` Parancs Alapszintaxisa

```bash
mount -t [típus] [eszköz] [könyvtár]
```

- **[típus]:** A fájlrendszer típusa (opcionális, pl. `ext4`, `xfs`, `vfat`, `ntfs`).
- **[eszköz]:** A csatolni kívánt fizikai vagy logikai eszköz (pl. `/dev/sdb1`).
- **[könyvtár]:** A célkönyvtár, azaz a csatolási pont (pl. `/mnt/media`).

(Amíg a fájlrendszer fel van csatolva, a célkönyvtár eredeti tartalma ideiglenesen láthatatlanná válik, és a könyvtár a felcsatolt fájlrendszer gyökerét fogja mutatni).

## 2. Kilépési Kódok (Exit Status)

A `mount` parancs lefutása után a kernel az alábbi kódok egyikével tér vissza:

| Kód | Jelentés |
| --- | --- |
| 0 | Siker. |
| 1 | Helytelen parancshívás vagy elégtelen jogosultság. |
| 2 | Rendszerhiba. |
| 4 | Belső mount hiba (bug). |
| 8 | A műveletet a felhasználó megszakította. |
| 16 | Probléma az `/etc/mtab` fájl írásával vagy zárolásával. |
| 32 | Csatolási hiba (Mount failure). |
| 64 | Legalább egy csatolás sikerült, de nem mindegyik. |

## 3. Fontosabb `mount` Opciók

| Opció | Leírás |
| --- | --- |
| `-a` | Felcsatolja az `/etc/fstab` fájlban felsorolt összes fájlrendszert. |
| `-l` | Kilistázza a jelenleg felcsatolt fájlrendszereket a címkéikkel (labels) együtt. |
| `-L [címke]` | A partíció csatolása a megadott címke (label) alapján. |
| `-o [opciók]` | Egyedi csatolási opciók megadása vesszővel elválasztva (pl. `ro`, `loop`). |
| `-r` | Csak olvasható (Read-only) módban csatolja fel a fájlrendszert. |
| `-t [típus]` | Meghatározza a fájlrendszer típusát (pl. `ext4`, `iso9660`). |
| `-M` (`--move`) | Egy már felcsatolt fájlrendszer áthelyezése egy másik csatolási pontra. |
| `-v` | Bőbeszédű (verbose) mód, kiírja, hogy éppen mit csinál a parancs. |

## 4. Gyakorlati Példák

**Felcsatolt fájlrendszerek listázása:**

```bash
mount           # Az összes listázása
mount -t ext4   # Csak az ext4 típusúak listázása
```

**Alapvető csatolás (pl. egy új merevlemez-partíció):**

```bash
sudo mount /dev/sdb1 /mnt/media
```

**USB meghajtó manuális csatolása:**

Ha az automatikus csatolás sikertelen, keressük meg az eszközt (`fdisk -l`), hozzuk létre a mappát, majd csatoljuk:

```bash
sudo mkdir -p /media/usb-drive
sudo mount /dev/sdb1 /media/usb-drive
```

**CD-ROM csatolása:**

A CD-k szabványos fájlrendszere az `iso9660`. Csak olvasható módban (`-o ro`) kell csatolni:

```bash
sudo mount -t iso9660 -o ro /dev/cdrom /mnt
```

**ISO képfájl csatolása (Loop device):**

Egy ISO fájl tartalmának böngészéséhez virtuális loop eszközként kell felcsatolni:

```bash
sudo mount /image.iso /media/iso-file -o loop
```

**Hálózati meghajtó (NFS) csatolása:**

(Előfeltétel: telepített NFS kliens).

```bash
sudo mkdir -p /media/nfs
# Ideiglenes csatolás IP cím/hosztnév és útvonal megadásával:
sudo mount remote.server:/dir /media/nfs
```

## 5. Az `/etc/fstab` használata (Automatikus Csatolás)

A rendszer újraindítása utáni automatikus csatoláshoz a meghajtókat az `/etc/fstab` fájlban kell rögzíteni.

**Felépítése:** `<fájlrendszer> <csatolási_pont> <típus> <opciók> <dump> <pass>`

**Példa egy NFS megosztás fstab bejegyzésére:**

```plaintext
remote.server:/dir /media/nfs  nfs      defaults    0       0
```

Ha egy fájlrendszer szerepel az `fstab`-ban, a csatoláshoz elég csak az egyik paramétert (az eszközt vagy a könyvtárat) megadni, a többit a parancs kiolvassa a fájlból:

```bash
sudo mount /media/nfs
```

**Csatolás Normál (Nem-root) Felhasználóként:**

Csak a root csatolhat fájlrendszereket, KIVÉVE, ha az `/etc/fstab` fájlban szerepel a `user` vagy `users` opció.

- `user`: Bármely felhasználó felcsatolhatja, de csak ő csatolhatja le.
- `users`: Bármely felhasználó felcsatolhatja, és bárki más is lecsatolhatja.

Példa fstab sor: `/dev/cdrom /cd iso9660 ro,user,noauto,unhide`

## 6. A Lecsatolás: Az `umount` Parancs

A fájlrendszerek biztonságos leválasztására (lecsatolására) az `umount` parancs szolgál. (Figyelem: A parancs neve `umount`, nem `unmount`!)

**Alapszintaxis:** (A csatolási pont vagy az eszköznév is megadható)

```bash
sudo umount /media/usb-drive
# vagy
sudo umount /dev/sdb1
```

**Foglalt (Busy) fájlrendszerek kezelése:**

Ha a lecsatolás sikertelen, mert egy program épp használja a lemezt, a `fuser` paranccsal kiderítheted, melyik folyamat (PID) tartja fogva:

```bash
fuser -m /media/usb-drive
```

**Lusta lecsatolás (Lazy Unmount):**

Ha nem akarod manuálisan leállítani a folyamatokat, a `--lazy` opció azonnal leválasztja a fájlrendszert a könyvtárfáról, de fizikailag csak akkor csatolja le, ha az összes folyamat befejezte a használatát:

```bash
sudo umount --lazy /dev/sdb1
```

**Erőszakos lecsatolás (Forced Unmount):**

Csak végszükség esetén (pl. megszakadt hálózati kapcsolatnál) használandó, mert adatvesztést vagy adatsérülést okozhat!

```bash
sudo umount -f /media/usb-drive
```

##Forrás
-[https://phoenixnap.com/kb/linux-mount-command]

# A Linux Fájlrendszer Hierarchia: A `/mnt` Könyvtár és a Csatolás Kezelése

## 1. A `/mnt` Könyvtár Szerepe

A `/mnt` egy általános csatolási pont (mount point), amely alá fájlrendszereket vagy hardvereszközöket csatolhatunk.

- **Hagyományos használat:** Régebben ide kerültek az állandó csatolási pontok alkönyvtárai (pl. `/mnt/cdrom`, `/mnt/floppy`). (Megjegyzés: Egyes disztribúciók, mint a Debian a gyökérben lévő `/cdrom` és `/floppy` mappákat, míg a RedHat/Mandrake a `/mnt` alattiakat preferálta).

- **Modern szabvány (FSSTND 2.3+):** A szabványosítás óta a `/mnt` kifejezetten a rendszergazda általi ideiglenes csatolásokra szolgál.

- **Fontos:** Telepítőprogramok sosem használhatják a `/mnt`-t; nekik egy külön, a rendszer által épp nem használt ideiglenes mappát kell létrehozniuk.

## 2. A Csatolás (Mounting) Elmélete és Gyakorlata

A UNIX/Linux rendszerekben minden fájl egyetlen, hatalmas könyvtárfában (directory tree) helyezkedik el. Ahhoz, hogy egy új fájlrendszert (pl. egy lemezt) használni tudjunk, "be kell illeszteni" ebbe a fába egy meglévő mappán (a csatolási ponton) keresztül. Ezt a folyamatot hívjuk csatolásnak.

**Alapparancsok:**

A `mount` parancs két fő argumentumot vár: az eszközfájlt (device file), amely a lemez nyers adataihoz ad hozzáférést, és a csatolási pontot (mount point), amelyen keresztül a fájlok elérhetővé válnak.

```bash
sudo mount /dev/hda2 /home 
sudo mount /dev/hda3 /usr
```

**Fájlrendszer típusának megadása (`-t`):**

Bár a `mount` gyakran kitalálja a fájlrendszer típusát, néha manuálisan kell megadni a `-t` (type) kapcsolóval:

```bash
sudo mount -t msdos /dev/fd0 /floppy
```

**Írásvédett (Read-only) csatolás (`-r`):**

Ha nem akarunk (vagy nem tudunk, pl. CD-ROM esetén) írni az eszközre, az `-r` kapcsolóval csak olvasható módban csatolhatunk. Ez megakadályozza, hogy a kernel frissítse a fájlok hozzáférési idejét (inode access times), és letilt minden írási kísérletet.

```bash
sudo mount -r /dev/cdrom /mnt/cdrom
```

> **Mi történik, ha egy nem üres mappába csatolunk?**
> A csatolási pontnak léteznie kell, de nem muszáj üresnek lennie. Amikor ráillesztjük az új fájlrendszert, a mappában lévő eredeti fájlok ideiglenesen elérhetetlenné válnak név alapján (amíg a lecsatolás meg nem történik). Ez hasznos trükkök forrása is lehet (pl. a boot korai szakaszában a `/tmp` átmenetileg máshová mutathat, amíg a végleges `/var` fel nem csatolódik).

## 3. A Gyökér (Root) Fájlrendszer és az Automatikus Csatolás

Hogyan csatolódik fel a legelső fájlrendszer (a root `/`), ha nincs még fa, amire csatlakozhatna? Ezt a kernel a rendszerindítás (boot) során "mágikusan" végzi el (a LILO vagy az `rdev` beállításai alapján).

- A gyökér fájlrendszer először csak olvasható (read-only) módban csatolódik fel.
- Az indító szkriptek lefuttatják az `fsck` (file system check) parancsot az adatok épségének ellenőrzésére.

  > **Szigorú szabály:** Az `fsck`-t TILOS írhatóként felcsatolt fájlrendszeren futtatni, mert adatvesztést okozhat! Mivel a root ekkor még read-only, az ellenőrzés biztonságos.

- Ha nincs hiba, a rendszer újracsatolja (remount) a gyökeret írható-olvasható (read-write) módban.

A többi fájlrendszer, aminek bootoláskor automatikusan el kell indulnia, az `/etc/fstab` fájlban van definiálva.

## 4. A Lecsatolás (`umount`) és a Lemezes Gyorsítótár (Disk Caching)

Amikor egy fájlrendszerre már nincs szükség, le kell csatolni az `umount` paranccsal. (Történelmi érdekesség: az "unmount" szóból az 'n' betű a 70-es években rejtélyes módon eltűnt a UNIX-ból, így a parancs neve `umount`).

Argumentumként megadható az eszköz neve VAGY a csatolási pont:

```bash
umount /dev/hda2
# vagy
umount /usr
```

> ⚠️ **Kritikus szabály hordozható adathordozóknál:**
> Soha ne húzz ki lemezt/pendrive-ot anélkül, hogy lecsatolnád! A Linux lemezes gyorsítótárat (disk caching) használ. Még ha úgy is tűnik, hogy a fájlmásolás befejeződött, az adatok fizikailag lehet, hogy csak a lecsatolás pillanatában íródnak ki a lemezre. A korai kihúzás az adatok megsemmisülését (garbled data) eredményezheti.

## 5. Biztonság és Normál Felhasználók Csatolási Jogai

Alapértelmezésben csak a root (rendszergazda) végezhet csatolási műveleteket. Ha bárki csatolhatna bármit, könnyen betölthetnének egy trójai programot (pl. egy fertőzött `/bin/sh` fájlt egy cserélhető lemezről).

Ha mégis engedélyezni kell a normál felhasználóknak (pl. floppyk, pendrive-ok) a csatolást, 4 út létezik:

1. **Kiadni a root jelszót:** Súlyos biztonsági rés, kerülendő.
2. **A `sudo` használata:** Kicsit jobb, szabályozható, de még mindig admin jogokat ad.
3. **Az `mtools` csomag használata:** MS-DOS lemezek manipulálása tényleges felcsatolás nélkül (ritka).
4. **Az `/etc/fstab` beállítása (A legjobb megoldás):** Részletes szabályokat hozhatunk létre.

**Példa normál felhasználók csatolásának engedélyezésére az `/etc/fstab` fájlban:**

```plaintext
/dev/fd0    /floppy    msdos    user,noauto    0    0
```

**Az oszlopok jelentése és az opciók magyarázata:**

- **Oszlopok:** 1. Eszközfájl, 2. Csatolási pont, 3. Fájlrendszer, 4. Opciók, 5. Dump gyakoriság, 6. fsck ellenőrzési sorrend.

- **`noauto` opció:** Megakadályozza, hogy a rendszer bootoláskor (`mount -a`) automatikusan próbálja felcsatolni az eszközt (hiszen lehet, hogy nincs is bedugva).

- **`user` opció:** Engedélyezi, hogy BÁRMELY normál felhasználó felcsatolhassa a lemezt (pl. `mount /floppy` paranccsal).

  **Beépített biztonság:** A `user` opció automatikusan letiltja a programok futtatását (execution) és a setuid biteket a felcsatolt meghajtóról, hogy megelőzze a trójai támadásokat.

- **Tipp MS-DOS fájlrendszerekhez:** Érdemes a `uid`, `gid` és `umask` opciókat is megadni a `user` mellé, hogy elkerüljük azt, hogy alapértelmezésben mindenki a rendszeren olvasási jogot kapjon az adott lemezhez.

##Forrás
-[https://tldp.org/LDP/Linux-Filesystem-Hierarchy/html/mnt.html]
