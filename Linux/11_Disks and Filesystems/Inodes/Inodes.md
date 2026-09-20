# ext4 Index Node-ok (Inode-ok)

A hagyományos UNIX fájlrendszerekben az inode tárolja a fájlhoz tartozó összes metaadatot (időbélyegek, blokktérképek, kiterjesztett attribútumok stb.), nem pedig a könyvtárbejegyzés. Az ext4 teljesítményoptimalizálási okokból (csalva egy kicsit) a fájltípus másolatát a könyvtárbejegyzésben is eltárolja.

Az inode tábla a `struct ext4_inode` elemek lineáris tömbje. Az inode checksum (ellenőrzőösszeg) a fájlrendszer UUID-je, az inode száma és maga az inode struktúra alapján kerül kiszámításra. Nincs 0-s sorszámú inode.

## Inode keresése és elhelyezkedése

Minden blokkcsoport `sb->s_inodes_per_group` számú inode-ot tartalmaz. Egy adott inode fizikai helyének kiszámítása az alábbi képletekkel történik:

- **Blokkcsoport (Block group):** `bg = (inode_num - 1) / sb->s_inodes_per_group`
- **Index a csoporton belül:** `index = (inode_num - 1) % sb->s_inodes_per_group`
- **Bájt eltolás (Offset):** `offset = index * sb->s_inode_size`

## Az `ext4_inode` struktúra felépítése

| Eltolás | Méret | Név | Leírás |
| --- | --- | --- | --- |
| 0x0 | `__le16` | `i_mode` | Fájl mód (jogosultságok és típus). Lásd az `i_mode` táblázatot. |
| 0x2 | `__le16` | `i_uid` | A tulajdonos UID-jének alsó 16 bitje. |
| 0x4 | `__le32` | `i_size_lo` | A fájlméret (bájtban) alsó 32 bitje. |
| 0x8 | `__le32` | `i_atime` | Utolsó hozzáférési idő (másodpercben az Epoch óta). Ha az EA_INODE flag be van állítva, a kiterjesztett attribútum értékének checksumját tárolja. |
| 0xC | `__le32` | `i_ctime` | Utolsó inode módosítási idő. Ha az EA_INODE flag be van állítva, az attribútum hivatkozásszámának (refcount) alsó 32 bitje. |
| 0x10 | `__le32` | `i_mtime` | Utolsó adat módosítási idő. Ha az EA_INODE be van állítva, a kiterjesztett attribútumot birtokló inode száma. |
| 0x14 | `__le32` | `i_dtime` | Törlési idő (másodpercben az Epoch óta). |
| 0x18 | `__le16` | `i_gid` | A csoport GID-jének alsó 16 bitje. |
| 0x1A | `__le16` | `i_links_count` | Hard linkek száma. (Az ext4 alapból max. 65 000 hard linket engedélyez. DIR_NLINK funkcióval, ha eléri a határt, ez az érték 1-re áll, jelezve a korlátlan almappákat). |
| 0x1C | `__le32` | `i_blocks_lo` | "Blokkok" számának alsó 32 bitje. A huge_file és az EXT4_HUGE_FILE_FL flagtól függően 512 bájtos vagy fájlrendszer-blokkokat jelent. |
| 0x20 | `__le32` | `i_flags` | Inode flagek. Lásd az `i_flags` táblázatot. |
| 0x24 | 4 bájt | `i_osd1` | OS-függő adatok (Linux: Inode verzió / EA_INODE esetén refcount felső 32 bit). |
| 0x28 | 60 bájt | `i_block[15]` | Blokktérkép vagy extent fa. |
| 0x64 | `__le32` | `i_generation` | Fájl verzió (NFS-hez). |
| 0x68 | `__le32` | `i_file_acl_lo` | Kiterjesztett attribútum blokk alsó 32 bitje. |
| 0x6C | `__le32` | `i_size_high` / `i_dir_acl` | Fájl/könyvtár méret felső 32 bitje. (ext2/3-ban `i_dir_acl` néven ismert). |
| 0x70 | `__le32` | `i_obso_faddr` | (Elavult) fragment cím. |
| 0x74 | 12 bájt | `i_osd2` | OS-függő adatok 2. (Linux: blokkszám felső 16 bit, UID/GID felső 16 bit, checksum alsó 16 bit). |
| 0x80 | `__le16` | `i_extra_isize` | Az eredeti ext2 inode-on (128 bájt) túli extra inode mezők mérete (alapértelmezetten 32 bájt). |
| 0x82 | `__le16` | `i_checksum_hi` | Inode checksum felső 16 bitje. |
| 0x84 | `__le32` | `i_ctime_extra` | Extra módosítási idő bitek (szub-szekundumos pontosság). |
| 0x88 | `__le32` | `i_mtime_extra` | Extra adatmódosítási idő bitek. |
| 0x8C | `__le32` | `i_atime_extra` | Extra hozzáférési idő bitek. |
| 0x90 | `__le32` | `i_crtime` | Fájl létrehozási ideje (másodpercben az Epoch óta). |
| 0x94 | `__le32` | `i_crtime_extra` | Extra fájl létrehozási idő bitek. |
| 0x98 | `__le32` | `i_version_hi` | Verziószám felső 32 bitje. |
| 0x9C | `__le32` | `i_projid` | Projekt ID. |

## `i_mode` értékek (Jogosultságok és Fájltípusok)

| Érték | Leírás | Érték | Fájltípus (Egymást kizáróak) |
| --- | --- | --- | --- |
| 0x1 | S_IXOTH (Mások futtathatják) | 0x1000 | S_IFIFO (FIFO / Csővezeték) |
| 0x2 | S_IWOTH (Mások írhatják) | 0x2000 | S_IFCHR (Karakteres eszköz) |
| 0x4 | S_IROTH (Mások olvashatják) | 0x4000 | S_IFDIR (Könyvtár) |
| 0x8 | S_IXGRP (Csoport futtathatja) | 0x6000 | S_IFBLK (Blokk eszköz) |
| 0x10 | S_IWGRP (Csoport írhatja) | 0x8000 | S_IFREG (Normál fájl) |
| 0x20 | S_IRGRP (Csoport olvashatja) | 0xA000 | S_IFLNK (Szimbolikus link) |
| 0x40 | S_IXUSR (Tulajdonos futtathatja) | 0xC000 | S_IFSOCK (Socket) |
| 0x80 | S_IWUSR (Tulajdonos írhatja) | | |
| 0x100 | S_IRUSR (Tulajdonos olvashatja) | | |
| 0x200 | S_ISVTX (Sticky bit) | | |
| 0x400 | S_ISGID (Set GID) | | |
| 0x800 | S_ISUID (Set UID) | | |

## `i_flags` (Inode flagek)

| Érték | Leírás |
| --- | --- |
| 0x1 / 0x2 / 0x4 | Biztonságos törlés / Megőrzés / Tömörítés (Nincsenek implementálva) |
| 0x8 | EXT4_SYNC_FL - Minden írás szinkron történik. |
| 0x10 | EXT4_IMMUTABLE_FL - A fájl megváltoztathatatlan (Immutable). |
| 0x20 | EXT4_APPEND_FL - A fájlhoz csak hozzáfűzni lehet. |
| 0x40 | EXT4_NODUMP_FL - A dump(1) segédprogram ne mentse. |
| 0x80 | EXT4_NOATIME_FL - A hozzáférési idő (atime) nem frissül. |
| 0x800 | EXT4_ENCRYPT_FL - Titkosított inode. |
| 0x1000 | EXT4_INDEX_FL - A könyvtár hash-elt indexeket használ. |
| 0x4000 | EXT4_JOURNAL_DATA_FL - A fájladatokat mindig a journalon keresztül kell írni. |
| 0x10000 | EXT4_DIRSYNC_FL - A könyvtárbejegyzések szinkron íródnak. |
| 0x20000 | EXT4_TOPDIR_FL - A könyvtárhierarchia teteje. |
| 0x40000 | EXT4_HUGE_FILE_FL - Hatalmas fájl (Huge file). |
| 0x80000 | EXT4_EXTENTS_FL - Az inode extent-eket használ blokktérkép helyett. |
| 0x100000 | EXT4_VERITY_FL - Verity által védett fájl. |
| 0x200000 | EXT4_EA_INODE_FL - Az inode nagy kiterjesztett attribútumot tárol az adatblokkjaiban. |
| 0x10000000 | EXT4_INLINE_DATA_FL - Az inode inline (beágyazott) adatot tartalmaz. |
| 0x20000000 | EXT4_PROJINHERIT_FL - A gyermekek öröklik a projekt ID-t. |
| 0x40000000 | EXT4_CASEFOLD_FL - Kis/nagybetűre nem érzékeny (case-insensitive) könyvtárkeresés. |

**Összesített flagek:** 0x705BDFFF (Felhasználó számára látható), 0x604BC0FF (Felhasználó által módosítható).

## Inode Méretek és Extra Hely

Az ext2 és ext3 rendszerekben az inode struktúra és rekord mérete fix 128 bájt volt (`EXT2_GOOD_OLD_INODE_SIZE`). Az ext4 formázáskor nagyobb lemezes inode rekordokat foglalhat le (alapértelmezetten 256 bájt).

A rekord méretét a szuperblokk `s_inode_size` mezője tárolja.

Az eredeti 128 bájton túli, az `ext4_inode` által ténylegesen felhasznált extra bájtok számát az `i_extra_isize` mező rögzíti (jelenleg a struktúra 160 bájtos, így `i_extra_isize = 32`).

Az inode struktúra vége és az inode rekord vége közötti extra hely kiterjesztett attribútumok (Extended Attributes) tárolására használható.

## Időbélyegek (Timestamps) és a 2038-as Probléma

Az első 128 bájtban a `ctime`, `atime`, `mtime` és `dtime` időbélyegek 32 bites előjeles egész számként vannak eltárolva, amelyek 2038 januárjában túlcsordulnának. Ha a fájlrendszer nem támogatja az `orphan_file` funkciót, az árva inode-ok a `dtime` mezőt túlterhelik (overload), és az árva lista következő elemére mutató mutatóként használják.

Ha az inode mérete (`s_inode_size`) nagyobb mint 128 bájt, és az `i_extra_isize` elegendő helyet biztosít, a `ctime`, `atime`, `mtime` és a létrehozási idő (`crtime`) mezők 64 bitre szélesednek.

Az extra 32 bites mezők (`i_[cma]time_extra`) alsó 2 bitje az epoch (korszak) kiterjesztésére szolgál, míg a felső 30 bit a nanoszekundumos pontosságot adja. Ezzel az időbélyegek 2446 májusáig válnak érvényessé.

A `dtime` (törlési idő) nem került kiszélesítésre.

## 64-bites Idő Dekódolása (Epoch Bitek)

| Extra epoch bitek | 32-bites idő MSB-je (Legnagyobb helyiértékű bit) | Korrekció (32-ből 64 bitre) | Dekódolt 64-bites tv_sec tartomány | Érvényes Dátumtartomány |
| --- | --- | --- | --- | --- |
| 0 | 0 | 0 | -0x80000000 – -0x00000001 | 1901-12-13 – 1969-12-31 |
| 0 | 0 | 0 | 0x00000000 – 0x07fffffff | 1970-01-01 – 2038-01-19 |
| 0 | 1 | 0x100000000 | 0x080000000 – 0x0ffffffff | 2038-01-19 – 2106-02-07 |
| 0 | 1 | 0x100000000 | 0x100000000 – 0x17fffffff | 2106-02-07 – 2174-02-25 |
| 1 | 0 | 0x200000000 | 0x180000000 – 0x1ffffffff | 2174-02-25 – 2242-03-16 |
| 1 | 0 | 0x200000000 | 0x200000000 – 0x27fffffff | 2242-03-16 – 2310-04-04 |
| 1 | 1 | 0x300000000 | 0x280000000 – 0x2ffffffff | 2310-04-04 – 2378-04-22 |
| 1 | 1 | 0x300000000 | 0x300000000 – 0x37fffffff | 2378-04-22 – 2446-05-10 |

##Forrás
-[https://www.kernel.org/doc/html/latest/filesystems/ext4/inodes.html]
-[https://labex.io/lesson/inodes]
