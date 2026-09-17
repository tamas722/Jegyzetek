# Linux Könyvtárszerkezet (Filesystem Hierarchy Standard)

A Linux fájlrendszer hierarchiája (FHS) határozza meg a könyvtárak struktúráját és célját, amely a legtöbb Linux (és UNIX-szerű) rendszerben azonos.

## Alapkönyvtárak és Funkcióik

- **`/` (Root):** A gyökérkönyvtár. Minden fájl és mappa ebből ágazik ki (pl. `/home/user`).

  > A `rm -rf /` parancs mindent törölne a gépről (általában a `--no-preserve-root` opció nélkül le van tiltva).

- **`/bin` (Binaries):** Alapvető végrehajtható programok és shell parancsok (pl. `ls`, `cp`, `cd`), amelyek minden felhasználó számára elérhetők.

- **`/dev` (Device files):** Speciális és eszközfájlok (virtuális fájlok).

  - Példák: `/dev/null` (megsemmisítő), `/dev/zero` (végtelen nullák), `/dev/random` (véletlenszerű értékek).

- **`/etc` (Configuration files):** A rendszer és a szolgáltatások központi konfigurációs fájljai (pl. jelszófájl, hálózati beállítások).

- **`/usr` (User binaries and program data):** Felhasználói programok, binárisok, könyvtárak. (A legtöbb tartalom csak olvasható).

  - `/usr/bin`: Alapvető felhasználói parancsok.
  - `/usr/sbin`: Rendszergazdai parancsok.
  - `/usr/lib`: Rendszerkönyvtárak.
  - `/usr/share`: Dokumentációk, man oldalak.

- **`/home` (User personal data):** A felhasználók saját, személyes mappái (pl. `/home/alice`). Ide kerülnek a személyes fájlok és beállítások. Két különböző felhasználó nem férhet hozzá egymás könyvtáraihoz.

- **`/lib` (Shared libraries):** A `/bin` és `/sbin` mappákban lévő programok futtatásához szükséges megosztott könyvtárak (kódok).

- **`/sbin` (System binaries):** Hasonló a `/bin`-hez, de az itt lévő programokat csak a root vagy sudo joggal rendelkező felhasználók futtathatják (rendszergazdai parancsok).

- **`/tmp` (Temporary files):** Ideiglenes fájlok tárolására. Újraindításkor tartalma törlődik.

- **`/var` (Variable data files):** Változó adatok, például rendszer naplófájlok (logok), gyorsítótárak. Nem törlődik automatikusan. (pl. `/var/log/wtmp`).

- **`/boot` (Boot files):** A rendszer indításához (boot) szükséges fájlok (kernel, LILO/Grub boot image fájlok).

- **`/proc` (Process and kernel files):** Jelenleg futó folyamatok és a kernel paraméterei (virtuális fájlok, pl. `/proc/cpuinfo`, `/proc/meminfo`). A rendszer futásidejű információinak lekérésére szolgál.

- **`/opt` (Optional software):** Harmadik féltől származó, a disztribúció tárolójában nem szereplő opcionális szoftverek telepítési helye.

- **`/root` (The home directory of the root):** A root (rendszergazda) felhasználó saját "home" mappája. Nem keverendő össze a `/` gyökérkönyvtárral!

- **`/media` (Mount point for removable media):** A cserélhető adathordozók (USB, SD kártya, DVD) automatikus csatolási pontja.

- **`/mnt` (Mount directory):** A fájlrendszerek manuális csatolására (mountolására) használt könyvtár a rendszergazdák számára.

- **`/srv` (Service data):** A rendszer által nyújtott szolgáltatások adatai (pl. webtárhely / webszerver adatai).
##Forrás
-[https://linuxhandbook.com/linux-directory-structure/]
-[https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/4/html/reference_guide/s1-filesystem-fhs]
