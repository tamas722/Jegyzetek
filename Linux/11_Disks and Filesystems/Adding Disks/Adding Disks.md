# Új lemez (Disk) hozzáadása egy meglévő Linux szerverhez

Rendszergazdaként gyakori feladat a kapacitásbővítés vagy egy meghibásodott lemez cseréje, ami új, nyers (raw) merevlemezek konfigurálását igényli. Ez a leírás az `fdisk` parancssori segédprogramot használja, amely a Linux rendszerek alapvető eszköze a merevlemezek és partíciók megtekintésére és kezelésére.

> **Megjegyzés:** Ez az útmutató új partíció létrehozására fókuszál, maximum 2TB-os méretig. 2TB felett a `parted` vagy `gdisk` használata, illetve a GPT partíciós tábla javasolt.

## 1. A jelenlegi lemezek és partíciók listázása

Mielőtt bármit módosítanál, azonosítanod kell az új, még particionálatlan lemezt. Ehhez listázd ki a rendszer összes lemezét:

```bash
fdisk -l
```

A kimenetben keresd meg az újonnan hozzáadott eszközt.

- Fizikai lemezeknél ez általában `/dev/sdb`, `/dev/sdc` stb.
- Virtuális környezetben (pl. AWS, KVM) gyakran `/dev/xvdb`, `/dev/xvdc` vagy `/dev/vdb` formátumú. A példában a `/dev/xvdc` azonosítót használjuk.

## 2. A lemez particionálása (`fdisk` interaktív mód)

Indítsd el az `fdisk` segédprogramot a kiválasztott lemezen (vigyázz, hogy a megfelelő eszközt add meg!):

```bash
fdisk /dev/xvdc
```

**A leggyakrabban használt `fdisk` parancsok** (az interaktív menüben):

| Parancs | Leírás |
| --- | --- |
| `n` | Új partíció létrehozása (New) |
| `p` | A jelenlegi partíciós tábla kiíratása (Print) |
| `d` | Partíció törlése (Delete) |
| `q` | Kilépés a változtatások mentése nélkül (Quit) |
| `w` | Változtatások kiírása a lemezre és kilépés (Write) |

**Az új partíció létrehozásának lépései az `fdisk`-en belül:**

1. **Új partíció:** Nyomd meg az `n` gombot.
2. **Partíció típusa:** Válaszd a Primary (Elsődleges) partíciót (általában a `p` gomb). Alapértelmezésben legfeljebb 4 elsődleges partíció hozható létre.
3. **Partíció száma:** Válaszd ki a számot. Javasolt az alapértelmezett 1-es érték használata.
4. **Első szektor (First sector):** Kérni fogja a kezdő szektort. Új lemez esetén hagyd alapértelmezetten (csak nyomj egy Entert).
5. **Utolsó szektor vagy méret (Last sector or size):** Megadhatod az utolsó szektort, de sokkal egyszerűbb a méretet megadni. Mindig tegyél egy `+` jelet a méret elé (pl. `+20G` egy 20 GB-os partícióhoz). Ha a teljes lemezt fel akarod használni, csak nyomj egy Entert az alapértelmezett értékhez.
6. **Mentés:** Nyomd meg a `w` gombot a partíciós tábla kiírásához és a kilépéshez.

Ezzel létrejött a partíció, amelynek az azonosítója `/dev/xvdc1` (a lemez neve + a partíció száma) lesz.

## 3. A lemez formázása (Fájlrendszer létrehozása)

Ahhoz, hogy a partícióra adatokat lehessen írni, létre kell hozni rajta egy fájlrendszert (pl. ext4 vagy xfs). Ehhez az `mkfs` (Make File System) parancsot használjuk az újonnan létrehozott partíción (`/dev/xvdc1`):

```bash
mkfs.ext4 /dev/xvdc1
```

## 4. A partíció csatolása (Mount)

Miután a formázás befejeződött, a partíciót csatolni kell a Linux fájlrendszerfájába egy csatolási ponton (mount point) keresztül.

Hozz létre egy mappát (ha még nem létezik), például a `/data` mappát, majd csatold oda a partíciót:

```bash
mkdir -p /data
mount /dev/xvdc1 /data
```

## 5. Állandó csatolás beállítása (`/etc/fstab`)

A `mount` paranccsal végzett csatolás egy újraindítás után elveszik. Ahhoz, hogy a lemez a rendszer minden indításakor automatikusan felcsatolódjon, be kell írni az `/etc/fstab` (File System Table) fájlba.

Nyisd meg a fájlt egy szerkesztővel (pl. `nano /etc/fstab`), és add hozzá az alábbi sort a fájl végéhez:

```plaintext
/dev/xvdc1    /data    ext4    defaults    0   0
```

> **Professzionális tipp:** A `/dev/xvdc1` eszköznév helyett érdemes a partíció UUID-jét használni az `fstab`-ban, amit a `blkid` paranccsal tudsz lekérdezni, így elkerülhetők az eszköznevek megváltozásából adódó csatolási hibák.
>
> ##Forrás
> -[https://www.tecmint.com/add-new-disk-to-an-existing-linux/]
>
> # Új Lemez Hozzáadása Linuxhoz (Parancssori Útmutató)

Amikor egy új lemezt adsz egy meglévő Linux rendszerhez, négy fő lépést kell végrehajtanod: azonosítás, particionálás, fájlrendszer létrehozása (formázás) és végleges felcsatolás (mount).

## 1. A Lemez Azonosítása (`fdisk -l`)

Mielőtt bármit csinálnál, meg kell találnod, hogy a rendszer milyen néven (eszközútvonalon) azonosítja az új meghajtót.

```bash
sudo fdisk -l
```

Keresd meg a particionálatlan, megfelelő méretű lemezt. A név a hardver típusától függ:

- **SATA/SCSI/USB:** `/dev/sda`, `/dev/sdb`, `/dev/sdc` stb.
- **NVMe SSD:** `/dev/nvme0n1`, `/dev/nvme1n1` stb. (Itt a partíciók neve `/dev/nvme0n1p1` formátumú lesz).
- **Virtuális lemez:** `/dev/xvdc`, `/dev/vdb` stb.

(A példákban a lemezre `/dev/sdX`-ként fogunk hivatkozni, ezt helyettesítsd a sajátoddal, pl. `/dev/sdb`).

## 2. A Lemez Particionálása (`fdisk`)

Indítsd el az `fdisk` eszközt a nyers lemezen:

```bash
sudo fdisk /dev/sdX
```

Az interaktív menü lépései egyetlen, a teljes lemezt elfoglaló partíció létrehozásához:

1. `n` (New partition / Új partíció).
2. `p` (Primary / Elsődleges partíció).
3. **Partition number:** Enter (Alapértelmezett, 1).
4. **First sector:** Enter (Alapértelmezett, a lemez legeleje).
5. **Last sector:** Enter (Alapértelmezett, a lemez legvége. Ha nem a teljes lemezt akarod használni, írj be egy méretet, pl. `+50G`).
6. `w` (Write / Változtatások kiírása a lemezre és kilépés).

## 3. A Partíció Formázása (`mkfs`)

A particionálás után egy fájlrendszert kell varázsolni az új partícióra (figyelj a számra a végén: `/dev/sdX1`!). A Linux számára az `ext4` a leginkább ajánlott fájlrendszer.

```bash
sudo mkfs -t ext4 /dev/sdX1
```

(Ha a lemezt Windows géppel is meg akarod osztani, használj `ntfs`-t, ha pedig pendrive, az `exFAT` a legjobb választás).

## 4. Végleges Felcsatolás (Permanent Mount)

Ahhoz, hogy a lemez újraindítás után is elérhető maradjon, az `/etc/fstab` fájlban kell rögzítenünk a felcsatolást.

**Miért UUID-t használjunk és ne eszköznevet (pl. `/dev/sdb1`)?**

Az eszköznevek újraindítások között megváltozhatnak (attól függően, milyen sorrendben ismeri fel őket a gép). Az UUID (Univerzális Egyedi Azonosító) a fájlrendszerhez kötődik és sosem változik meg, így garantálja a biztos felcsatolást.

**Lépések:**

1. **Hozd létre a csatolási pontot (mount point):** Ebbe a könyvtárba lesz elérhető a lemez tartalma.

   ```bash
   sudo mkdir -p /media/disk
   ```

2. **Keresd meg a partíció UUID-jét:**

   ```bash
   sudo blkid /dev/sdX1
   # Jegyezd fel az UUID értékét (idézőjelek nélkül).
   ```

3. **Szerkeszd az `/etc/fstab` fájlt:**

   ```bash
   sudo nano /etc/fstab
   ```

   Add hozzá a fájl végéhez ezt a sort (az UUID-t és a könyvtárat cseréld a sajátodra):

   ```plaintext
   UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890 /media/disk ext4 defaults 0 2
   ```

4. **Teszteld az `/etc/fstab` bejegyzést!**

   Mielőtt újraindítod a gépet, mindenképp futtasd le ezt a parancsot:

   ```bash
   sudo mount -a
   ```

   (Ha a parancs nem ad vissza semmilyen hibát, a konfiguráció tökéletes. Ha hibát dob, azonnal javítsd az `fstab` fájlt, különben a rendszer esetleg nem fog elindulni!)

## Gyakori Kérdések (FAQ)

- **Hogyan lehet biztonságosan eltávolítani a lemezt?**

  Először válaszd le a partíciót: `sudo umount /media/disk`. Ha beírtad az `/etc/fstab` fájlba, onnan is töröld a sort, mielőtt fizikailag kihúzod a meghajtót.

- **Miért nem csatolódik fel a lemezem újraindítás után?**


##Forrás
-[https://linuxconfig.org/how-to-add-new-disk-to-existing-linux-system]
  Ez általában az `/etc/fstab` fájl hibáját jelenti. Ellenőrizd, hogy az UUID stimmel-e (a `sudo blkid` paranccsal), létezik-e a csatolási könyvtár, és teszteld a `sudo mount -a` paranccsal.
