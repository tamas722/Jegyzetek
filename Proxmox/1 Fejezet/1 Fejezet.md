# 1. A Proxmox VE Architektúrája és Alapkoncepciói

A Proxmox VE egy teljesen nyílt forráskódú, Debian GNU/Linux alapú platform, amely két különböző virtualizációs technológiát ötvöz a maximális rugalmasság érdekében:

- **KVM (Kernel-based Virtual Machine):** A teljes hardveres virtualizációért felel (pl. Windows, OPNsense vagy bármilyen egyedi kernelű OS futtatásához).
- **LXC (Linux Containers):** Pehelysúlyú, OS-szintű virtualizáció, amely megosztja a hoszt kernelét, így minimális erőforrás-igénnyel futtat Linuxos szolgáltatásokat.

## 1.1 Központosított Menedzsment (Multi-Master Dizájn)

A rendszer legfontosabb tervezési döntése a **"Multi-Master"** architektúra. Nincs egyetlen, kitüntetett menedzsment szerver (Single-Point-of-Failure); a clusterbe kötött fizikai gépek bármelyike képes a teljes rendszert irányítani.

- **`pmxcfs` (Proxmox Cluster File System):** Egy adatbázis-vezérelt (SQLite) fájlrendszer, amely a konfigurációs fájlokat tárolja. A **Corosync** protokoll ezt a fájlrendszert valós időben replikálja a cluster összes gépe között. Az adatok a fizikai lemezen is tárolódnak, de a működéshez a RAM-ban is él egy másolat, aminek a maximális mérete $128$ MiB (ez több ezer VM konfigurációjához is elegendő).

> **Gyakorlati példa:** Ha egy rack szekrényben a HP DL360-as és a Dell R530-as szervereidet egy clusterbe kötöd, és az egyik gép webes felülete (GUI) valamiért elérhetetlenné válik (pl. hálózati kártya hiba a menedzsment porton), egyszerűen belépsz a másik gép IP-címén a portálra, és onnan zavartalanul tudod menedzselni a kiesett gép épen maradt virtuális hálózatát vagy leállítani a rajta futó szolgáltatásokat.

### REST API

A Proxmox VE teljes egésze egy **JSON sémával definiált RESTful API**-ra épül. Ez azt jelenti, hogy minden, amit a webes felületen vagy a parancssorban (CLI) csinálsz, ezen az API-n keresztül fut. Ennek a legnagyobb előnye a gyakorlatban, hogy a rendszeredet rendkívül könnyen integrálhatod külső automatizációs eszközökkel (pl. **Terraform**, **Ansible**) vagy saját fejlesztésű szoftverekkel.

### Szerepkör-alapú adminisztráció és Hitelesítés

A Proxmox **granuláris** (nagyon finoman hangolható) jogosultságkezelést használ. Különböző hitelesítési forrásokat (**Realm**) lehet bekötni, mint például a Microsoft **Active Directory (AD)**, **LDAP**, a szabványos Linux **PAM**, vagy a Proxmox saját beépített hitelesítési szervere. Egy felhasználóhoz vagy csoporthoz specifikus objektumokat (pl. csak egy adott VM-et) és szerepköröket lehet rendelni.

## 1.2 és 1.7 Hiperkonvergens Infrastruktúra (HCI) és Tárolás

A Proxmox VE egyetlen platformba integrálja a számítási (compute), a hálózati és a tárolási erőforrásokat.

- **Lokális tárolók:** LVM, ext4 könyvtárak (Directory), valamint a ZFS, ami szoftveres RAID-et és fejlett kötetkezelést biztosít közvetlenül a hypervisoron.
- **Hálózati / Megosztott tárolók:** iSCSI, NFS, CIFS, illetve elosztott rendszerek, mint a Ceph (RBD és CephFS).

> **Gyakorlati példa az élő migrációra (Live Migration):** Ha a virtuális gépek lemezképei egy megosztott tárolón (például egy dedikált NAS-on, vagy a szervereken átívelő Ceph clusterben) csücsülnek, a futó VM-eket nulla másodperc leállással mozgathatod a fizikai hostok között, hiszen csak a RAM tartalmát kell a hálózaton átmásolni, magát a sok tíz gigabájtos lemezképet nem.

## 1.3 Biztonsági mentés (Backup & Restore)

A rendszer integrált mentési eszköze a **`vzdump`**, amely képes konzisztens snapshot (pillanatkép) alapú mentéseket készíteni a futó VM-ekről, akár NFS vagy iSCSI tárolókon is. Emellett a **Proxmox Backup Server (PBS)** integráció révén deduplikált, inkrementális, kliens-oldalon titkosított mentések is készíthetők.

## 1.4 Magas Rendelkezésre Állás (HA Cluster)

Több node-os cluster esetén a beépített HA menedzser automatikusan érzékeli a hardveres meghibásodásokat. Nincs szükség külső eszközökre (mint pl. a Pacemaker); a rendszer **watchdog-alapú fencing** (elszigetelő) mechanizmust használ, hogy a kiesett node-on futó szolgáltatásokat biztonságosan újraindítsa egy túlélő, egészséges szerveren.

## 1.5 és 1.6 Hálózat és Tűzfal

A Proxmox VE a natív Linux hálózati stacket (**Linux Bridge**) használja. Úgy képzeld el, mintha minden virtuális gép egy virtuális patch kábellel lenne bedugva egy szoftveres switch-be.

- **Haladó hálózatok:** Támogatja a VLAN-okat (802.1q) és a hálózati kártyák összefogását (link aggregation / bonding). Kiterjedt (pl. kollégiumi vagy sok-bérlős) infrastruktúrákhoz a Software Defined Network (SDN) stack nyújt VXLAN és EVPN alapú zóna-izolációt.
- **Tűzfal:** Interfész szinten szűrhetők a hálózati csomagok, és komplex biztonsági csoportok (Security Groups) is létrehozhatók.

## Az 1. fejezetben szereplő natív Linux parancsok

Mivel a dokumentum első fejezete az architekturális áttekintésről szól, a mindennapi üzemeltetés (pl. VM indítás) parancsai majd a későbbi fejezetekben jönnek elő. A hivatalos dokumentáció 1. fejezete az alábbi konkrét terminál-parancsokat listázza, amelyek a nyílt forráskódú dokumentáció és a GUI lokalizációjához (fordításához), illetve a szükséges perl-csomagok telepítéséhez köthetők.

**Csomagok telepítése Debian/Ubuntu rendszereken:**

```bash
apt-get install perl liblocale-po-perl libjson-perl
```

(Ez a parancs telepíti azokat a Debian csomagokat, amelyek a `.po` nyelvi fájlok fordításához és `.js` formátumba alakításához szükségesek.)

**A nyelvi Git tároló klónozása és inicializálása:**

```bash
git clone git://git.proxmox.com/git/proxmox-i18n.git
cd proxmox-i18n
make init-<LANG>.po
```

(A `<LANG>` helyére a nyelv kódját kell beírni, pl. `hu` a magyarhoz. A `make` parancs legenerálja az alapértelmezett nyelvi sablont.)

**A nyelvi fájl szerkesztése:**

```bash
poedit <LANG>.po
```

(A Poedit egy elterjedt GUI eszköz, de a fájl bármilyen szövegszerkesztővel (pl. `vim`, `nano`) szerkeszthető.)

**A lefordított fájl átalakítása Proxmox kompatibilis JavaScript fájllá:**

```bash
./po2js.pl -t pve xx.po >pve-lang-xx.js
```

(Egy Perl script, ami a lefordított szótárat olyan formába önti, amit az ExtJS webes felület futtatáskor értelmezni tud.)

**A csomag fordítása (Debian package készítése):**

```bash
make deb
```

**Egy fontos extra parancs (mentés készítése), amit a szöveg futólag említ:**
Bár a szintaxist az 1. fejezet még nem fejti ki, a `vzdump` az az alapvető parancs, ami a teljes KVM és LXC mentési logikát hajtja a motorháztető alatt.

> **Gyakorlati példa:** `vzdump 100 --mode snapshot` – ez a parancs élőben, leállás nélkül menti le a 100-as azonosítójú VM-et.


### Nyílt forráskód és licencelés

A Proxmox **GNU Affero General Public License (AGPLv3)** alatt jelenik meg. A dokumentáció kiemeli, hogy a **vendor lock-in** (szállítói röghöz kötés) elkerülése és a forráskód teljes átláthatósága garantálja a magas biztonságot.

### Támogatási csatornák

Felsorolja, hogy hol lehet segítséget kérni:

- **Proxmox VE Wiki**
- közösségi fórum
- levelezőlisták fejlesztőknek és felhasználóknak
- hivatalos hibajegykezelő (**Bug Tracker**)
- fizetős kereskedelmi támogatás (**Enterprise Support**), ami garantált válaszidőt ad a fejlesztőktől

### A projekt története

Egy történelmi áttekintés arról, hogy a projekt **2007-ben** indult, eredetileg **OpenVZ**-t használt konténerekre (amit a **4.0-s verzióban** cseréltek **LXC**-re), a **Proxmox Backup Server (PBS) 2020-ban** jelent meg, **2022-ben** érkezett az **Offline Mirror** eszköz, a **8.1-es verziótól** pedig a **Software Defined Network (SDN)** alapértelmezetten települ.
