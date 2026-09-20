# A Linux `df` Parancs (Lemezterület Ellenőrzése)

A `df` (disk free) parancs megmutatja a felcsatolt fájlrendszerek használt és szabad lemezterületét. Ez az elsődleges eszköz, ha a lemezmegtelési riasztásokat vizsgáljuk, vagy telepítés előtt ellenőrizzük a szabad helyet.

## 1. Alapszintaxis és Oszlopok

**Szintaxis:** `df [OPCIÓK] [FÁJLRENDSZER...]`

Ha argumentum nélkül futtatjuk, kilistázza az összes felcsatolt fájlrendszert. Ha egy konkrét fájlrendszert vagy csatolási pontot adunk meg (pl. `df /`), csak azt mutatja.

**A kimenet oszlopai:**

- **Filesystem:** A fájlrendszer / eszköz neve (pl. `/dev/nvme0n1p3`).
- **1K-blocks / Size:** A fájlrendszer teljes mérete (alapértelmezetten 1K blokkokban).
- **Used:** A felhasznált terület.
- **Available (Avail):** A szabad terület.
- **Use%:** A felhasznált terület százalékosan.
- **Mounted on:** A könyvtár (csatolási pont), ahová a fájlrendszer csatolva van.

## 2. Mértékegységek és Formázás

Alapértelmezésben a `df` 1K-s blokkokban mutatja a méreteket, ami nehezen olvasható.

- **`-h` (Human-readable):** Ember számára olvasható formátum (KB, MB, GB). 1024-es váltószámmal dolgozik (kibibyte, mebibyte). Ez a leggyakrabban használt kapcsoló (`df -h`).
- **`-H` (`--si`):** Szintén olvasható formátum, de 1000-es váltószámmal. (Ezért mutat a 240 GB-osként árult bolti merevlemez a `df -h` paranccsal kevesebbet, de a `df -H` paranccsal pontosan a gyári értéket).
- **`-k`:** 1K blokkok kényszerítése (rendszereken ez az alapértelmezett).
- **`-B` (`--block-size`):** Egyedi blokkméret megadása.
  - Pl. `df -BG` (Gigabájtokban mutatja az értékeket, "G" utótaggal).
  - Pl. `df -B1G` (Csak a számokat mutatja, utótag nélkül).

> **Megjegyzés:** A macOS BSD alapú `df` parancsa ismeri a `-m` kapcsolót, de a Linuxos GNU `df` nem! Helyette használd a `-BM` kapcsolót a Megabájtokhoz.

## 3. Fájlrendszer Típusok és Szűrés

- **`-T`:** Hozzáad egy "Type" oszlopot a kimenethez, amely megmutatja a fájlrendszer típusát (pl. `ext4`, `vfat`, `tmpfs`).
- **`-t [típus]` (Filter in):** Csak a megadott típusú fájlrendszereket listázza. Pl.: `df -h -t ext4`
- **`-x [típus]` (Filter out):** Kizárja a megadott típust. Pl. a memórialapú pszeudo fájlrendszerek elrejtése: `df -h -x tmpfs`
- **`-a` (`--all`):** Megmutatja az összes (akár 0 bájtos, pszeudo) fájlrendszert is, amiket a `df` alapból elrejt.
- **`-l` (`--local`):** Csak a helyi fájlrendszereket mutatja. Kihagyja a hálózati megosztásokat (pl. NFS, CIFS), ami megakadályozza, hogy a parancs lefagyjon, ha egy távoli szerver nem válaszol.

## 4. Összesítés és Inode-ok

- **`--total`:** A kimenet aljához ad egy "total" sort, amely összegzi a listázott fájlrendszereket.

  **Miért nem egyezik meg pontosan a használt + szabad hely a teljes (total) hellyel?** Mert az ext2/3/4 fájlrendszerek alapértelmezetten területet (általában 5%-ot) tartanak fenn a privilegizált (root) folyamatok számára, hogy a rendszer ne omoljon össze, ha a lemez betelik. A `df` ezt beleszámolja a teljes méretbe, de a "szabad" helybe nem.

- **`-i` (Inodes):** Lemezterület helyett az Inode-ok használatát mutatja.

  **Mi az az Inode?** Egy adatstruktúra, ami a fájlok metaadatait (méret, tulajdonos, jogosultságok) tárolja. Minden fájl 1 inode-ot fogyaszt. Ha egy fájlrendszerből kifogynak az inode-ok (pl. rengeteg nagyon kicsi fájl miatt), nem tudsz új fájlt létrehozni, hiába van még több gigabájtnyi szabad hely a lemezen!

## 5. Egyedi Kimenet és Szkriptelés (POSIX mód)

- **`--output=...`:** Egyedi oszlopok kiválasztása. Paraméterek vesszővel elválasztva: `source`, `fstype`, `size`, `used`, `avail`, `pcent`, `target` (csatolási pont), stb.

  Pl.: `df -h -t ext4 --output=source,size,used,avail,pcent`

- **`-P` (`--portability`):** POSIX kompatibilis kimenet. Megakadályozza, hogy a túl hosszú eszköznevek miatt a kimenet két sorba törjön. Szkriptek írásánál (pl. `awk`-val történő feldolgozásnál) kötelező!

  Pl. a root fájlrendszer foglaltsági százalékának kinyerése: `df -P / | awk 'NR==2 {print $5}'`

- **`--sync`:** Kikényszeríti a fájlrendszer szinkronizálását, mielőtt beolvasná a méreteket. Nagy fájlmásolások után pontosabb azonnali eredményt ad.

## 6. `df` vs. `du` (Amikor a lemez tele van, de mégsem)

- A `df` a fájlrendszer szintjén, a metaadatok alapján méri a helyet (gyors).
- A `du` végigjárja a könyvtárfát, és összeadja a látható fájlok méretét.

**Hibaelhárítás:** Ha a `df` szerint a lemez 100%-on van, de a `du` szerint sok szabad hely kéne legyen, akkor egy már letörölt fájlt egy folyamat még mindig nyitva tart.

- **Megkeresésük:** `sudo lsof +L1`
- **Megoldás:** A fájlt fogva tartó szolgáltatás/folyamat újraindítása vagy leállítása, ami után felszabadul a hely.

## 7. Gyors Referencia Táblázat (Cheatsheet)

| Parancs | Leírás |
| --- | --- |
| `df` | Az összes felcsatolt fájlrendszer mutatása |
| `df /` | Egy adott fájlrendszer/könyvtár mutatása |
| `df -h` | Ember számára olvasható méretek (1024-es hatvány, pl. GB) |
| `df -H` | Ember számára olvasható méretek (1000-es hatvány) |
| `df -k` | Méretek 1K blokkokban (alapértelmezett) |
| `df -BG` | Méretek 1 Gigabájtos blokkokban |
| `df -T` | Fájlrendszer típusok (Type) megjelenítése |
| `df -t ext4` | Csak az ext4 fájlrendszerek mutatása |
| `df -x tmpfs` | A tmpfs fájlrendszerek kihagyása (elrejtése) |
| `df -a` | Pszeudo (0 bájtos) fájlrendszerek megjelenítése is |
| `df -h -l` | Csak helyi (local) fájlrendszerek mutatása (hálózatiak nélkül) |
| `df -h --total` | Összesítő sor megjelenítése a kimenet végén |
| `df -i` | Inode használat megjelenítése |
| `df -ih /` | Inode használat ember által olvasható formátumban |
| `df --output=source,size,pcent` | Egyedi oszlopok kiválasztása |
| `df -P /` | POSIX egysoros kimenet szkriptekhez |
| `df --sync -h` | Szinkronizálás kikényszerítése az olvasás előtt |

##Forrás
-[https://linuxize.com/post/how-to-check-disk-space-in-linux-using-the-df-command/]
-[https://docs.kernel.org/filesystems/index.html]
