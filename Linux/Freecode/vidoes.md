# Bevezetés a kurzusba

A kurzus oktatója **Bo K.**, célközönsége pedig azok a felhasználók (akár egyéni, akár vállalati), akiknek egyáltalán nincs, vagy csak korlátozott tapasztalatuk van a Linuxszal. A tananyag bemutatja a rendszergazdák és végfelhasználók által mindennap használt eszközöket és technikákat, a navigációt a főbb disztribúciókban, a rendszerbeállításokat, a grafikus felületeket, valamint a parancssori alapműveleteket és gyakori alkalmazásokat. A kurzus a Linux Foundation támogatásával jött létre (egy eredetileg szöveges anyag videósított változata).

> **Fontos:** A parancssorban megjelenő dollárjelet (`$`) nem kell begépelni, az csak azt jelzi, hogy az utána lévő szöveget a terminálba kell beírni.

## 1. fejezet: Linux disztribúció családok

A kurzus optimális elvégzéséhez legalább egy Linux disztribuciónak telepítve kell lennie. Bár több száz verzió létezik, a tananyag három fő disztribúció családra fókuszál.

### A Red Hat család

A család élén a **Red Hat Enterprise Linux (RHEL)** áll, tagjai közé tartozik a **CentOS**, a **CentOS Stream**, a **Fedora** és az **Oracle Linux**. A Fedora egy széles közösség által fejlesztett, több szoftvert tartalmazó változat, amely egyben a jövőbeli RHEL kiadások tesztplatformja (upstream) is. Mivel a normál CentOS 8 frissítései leálltak, a kurzus a RHEL-lel szinte teljesen megegyező **CentOS Stream**-et használja. A rendszerek többek között Intel x86, ARM, Itanium, PowerPC és IBM System Z hardvereket támogatnak. Csomagkezelésre az RPM-alapú **YUM** és **DNF** eszközöket használják. Vállalati környezetben rendkívül elterjedtek.

### A SUSE család

Ide tartozik a **SUSE Linux Enterprise Server (SLES)** és az **openSUSE**. A kurzus az ingyenesen elérhető openSUSE-t használja referenciaként, mivel az szinte mindenben megegyezik a SLES-sel (amely az openSUSE upstream verziója). Csomagkezelésre az RPM-alapú **Zypper**-t használják, a rendszeradminisztráció legfőbb eszköze pedig a **YaST** (Yet another Setup Tool). A SLES-t széles körben alkalmazzák például a kereskedelmi szektorban.

### A Debian család

A **Debian** egy tisztán nyílt forráskódú közösségi projekt, nem birtokolja egyetlen vállalat sem, fő fókusza pedig a stabilitás. Ez rendelkezik a messze legnagyobb szoftvertárolóval (repository). A Debian az upstream verziója az **Ubuntu**-nak, ami pedig az upstreamje a **Linux Mint**-nek. Az Ubuntu kiváló kompromisszumot kínál a stabilitás és az egyszerű használhatóság között. A kurzus az **Ubuntu LTS (Long-Term Support)** kiadást használja referenciaként. Ez a család a dpkg-alapú **APT** csomagkezelőt alkalmazza. Az Ubuntu nagyon népszerű a felhős (cloud) rendszerekben, és bár alapvetően Debianra és GNOME-ra épül, vizuálisan jelentősen eltér az alap Debiantól.


# 2. fejezet: A Linux filozófiája és fogalmai

A kurzus maximális kihasználásához javasolt egy Linux rendszer telepítése (akár fizikai, akár virtuális, esetleg dual-boot vagy live USB formájában). Ha elakadnál, a legtöbb dokumentáció már jelen van a rendszereden a **Man Pages** (kézikönyv oldalak) formájában. Bármilyen parancsról vagy programról a `man [téma]` parancs beírásával tudhatsz meg többet. A nyílt forráskódú közösségben gyakran használják a **"Foo"** kifejezést helyettesítő névként (pl. *"írd be a fájlnevet ide"*), nem kell ténylegesen "Foo" nevű fájlokat létrehozni.

## Alapfogalmak, amiket ismerned kell

- **Kernel:** A Linux operációs rendszer agya. Ez vezérli a hardvert, és biztosítja a kapcsolatot a hardver és az alkalmazások között. A legújabb és korábbi verziók a [kernel.org](https://kernel.org) oldalon találhatók.
- **Disztribúció (Distro):** Programok és a Linux kernel összessége, amelyek együttesen alkotnak egy Linux-alapú operációs rendszert (pl. RHEL, Fedora, Ubuntu, Gentoo).
- **Bootloader:** Az operációs rendszert indító (bootoló) program (pl. GRUB, ISOLINUX).
- **Service (Szolgáltatás):** A háttérben futó program (daemon) (pl. httpd, nfsd, ntpd).
- **File system (Fájlrendszer):** Fájlok tárolására és rendszerezésére szolgáló módszer (pl. ext3, ext4, FAT, XFS, Btrfs).
- **X Window System:** Szabványos eszközkészlet és protokoll a grafikus felhasználói felületek (GUI) felépítéséhez.
- **Desktop environment (Asztali környezet):** A grafikus felhasználói felület az operációs rendszer felett (pl. GNOME, KDE, XFCE, Fluxbox).
- **Command line (Parancssor):** Felület parancsok beírására.
- **Shell:** A parancssori értelmező, ami feldolgozza a bemenetet, és utasítja az operációs rendszert a feladatok elvégzésére (pl. bash, tcsh, zsh).

## A Linux disztribúciók felépítése

Egy teljes Linux disztribúció a **kernelből** és számos **szoftvereszközből** áll (fájlkezelés, felhasználókezelés, csomagkezelés). Bár a disztribúciók különböző kernelverziókat használhatnak (pl. a RHEL 8 egy rendkívül stabil, de régebbi, 4.18-as kernelt használ), a fejlesztők gyakran beépítik a legújabb kernel fejlesztéseket a régebbi verziókba is.

Egyéb alapvető eszközök egy disztribúcióban:

- C, C++ és Clang fordítók
- GDB hibakereső (debugger)
- Alaprendszeri könyvtárak (libraries)
- Grafikus megjelenítésért felelős alacsony szintű interfészek, és a magasabb szintű asztali környezet
- Csomag- és szoftverfrissítő rendszerek
- Előre telepített alkalmazások széles köre

## Disztribúciók felhasználási területei

A nagyvállalatok és kormányzati intézmények általában a főbb, kereskedelmileg támogatott disztribúciókat választják (**Red Hat**, **SUSE**, **Canonical/Ubuntu**). A **CentOS** és a **CentOS Stream** népszerű, ingyenes alternatívái a RHEL-nek, olyan szervezetek számára, amelyek nem igényelnek fizetős technikai támogatást (ráadásul binárisan kompatibilisek a RHEL-lel). Az **Ubuntu** és a **Fedora** a fejlesztők és az oktatás körében kedvelt, míg a **Scientific Linux** a kutatók körében. A kereskedelmi disztribútorok hosszú távú, fizetős támogatást, hardver/szoftver tanúsítványokat, valamint biztonsági és hibajavítási frissítéseket is kínálnak.

## Összegzés

A Linux erősen merít a Unix operációs rendszerből. Számos funkciót és szolgáltatást fájlokon és fájlszerű objektumokon keresztül ér el. A Linux egy teljes mértékben **többfeladatos (multitasking)**, **többfelhasználós (multi-user)** operációs rendszer, beépített hálózati funkciókkal és háttérben futó szolgáltatásokkal (daemonokkal).

# 3. fejezet: Linux alapok és rendszerindítás

## A rendszerindítási folyamat (Boot Process)

A rendszerindítás az a folyamat, amely a számítógép bekapcsolásától a felhasználói felület teljes betöltődéséig tart. Bár a folyamat mélyen technikai, az ismerete sokat segít a hibaelhárításban.

1. **BIOS / UEFI és POST:** Bekapcsoláskor a **BIOS** (Basic Input/Output System) inicializálja a hardvereket (képernyő, billentyűzet) és teszteli a memóriát (ez a **POST** – Power On Self Test). Az alaplapi CMOS-ból kiolvassa a dátumot és az időt.

2. **Bootloader (Rendszertöltő):** A vezérlés átkerül a bootloaderhez, amely a merevlemezen található (MBR/BIOS rendszereknél a merevlemez legelső 512 bájtos szektorában, UEFI rendszereknél az EFI partíción).
   - Leggyakoribb bootloaderek: **GRUB** (Grand Unified Bootloader), **ISOLINUX** (cserélhető adathordozókhoz), **U-Boot** (beágyazott eszközökhöz).
   - A bootloader betölti a kernelt és az ideiglenes RAM lemezt (**initramfs**) a memóriába, majd átadja a vezérlést.

3. **Kernel és Initramfs:** A kernel kicsomagolja magát, inicializálja a hardvereket és a beépített illesztőprogramokat.
   - Az **initramfs** egy ideiglenes fájlrendszer a memóriában, amely a végleges gyökérfájlrendszer (root file system) csatolásához (mount) szükséges programokat és drivereket tartalmazza.
   - A **udev** (user device) felel a jelenlévő eszközök felismeréséért és az illesztőprogramok betöltéséért. Ha a gyökérfájlrendszert hiba nélkül megtalálta, becsatolja, az initramfs pedig törlődik a RAM-ból.

4. **Az "init" folyamat (System V vs. systemd):** A kernel elindítja az `/sbin/init` folyamatot (ez minden más folyamat ősatyja).
   - Régebben a **System V (SysV)** módszert használták, amely szekvenciális (egymás utáni) "runlevel"-ek (futási szintek) és szkriptek sorozatából állt.
   - Ma már minden fő disztribúció a **systemd**-t használja. Ez sokkal gyorsabb, mert a folyamatokat agresszíven, párhuzamosan (egyszerre) indítja el, bonyolult szkriptek helyett egyszerű konfigurációs fájlokat használ. Az `/sbin/init` manapság valójában csak egy hivatkozás a systemd-re.

## Fájlrendszerek és partíciók (Kiemelt részletek)

A fájlrendszer úgy működik, mint egy hűtőszekrény: a polcok segítenek az élelmiszerek méret, forma és típus szerinti rendszerezésében. A fájlrendszer az önkényes adathalmazok ember által használható formában történő tárolásának és rendszerezésének módszere.

Linux alatt beszélhetünk hagyományos lemezes, flash tárolós, adatbázis és speciális célú fájlrendszerekről (pl. `ext4`, `xfs`, `btrfs`).

# Fájlrendszerek

## ext3 (Third Extended Filesystem)

Az ext3 egy régebbi, de történelmileg nagyon fontos Linux fájlrendszer, amely az ext2-t váltotta.

- **Naplózás (Journaling):** Ez volt az első széles körben elterjedt Linux fájlrendszer, ami bevezette a naplózást. Ez azt jelenti, hogy a lemezre írás előtt a rendszer feljegyzi a tervezett változtatásokat. Áramszünet vagy rendszerösszeomlás esetén nem sérül meg a teljes fájlrendszer, és a helyreállítás (`fsck`) másodpercek alatt lefut.
- **Korlátok:** Mára elavultnak számít. A maximális fájlmérete **2 TB**, egy partíció maximális mérete pedig **32 TB** lehetett.
- **Státusz:** Leváltotta az ext4, új rendszereknél már nem használják.

## ext4 (Fourth Extended Filesystem)

Jelenleg a legelterjedtebb és legstabilabb fájlrendszer a Linux világban (például az Ubuntu és a Debian alapértelmezettje).

- **Hatalmas kapacitás:** Támogatja a gigantikus méreteket. Egy partíció akár **1 exabájt** (1 millió terabájt) is lehet, egyetlen fájl mérete pedig elérheti a **16 TB**-ot.
- **Extents technológia:** A régi blokk-alapú tárolás helyett "extenteket" használ, ami azt jelenti, hogy a nagyméretű fájlokat folytonos blokkokban tárolja. Ez drasztikusan csökkenti a fájlok töredezettségét (fragmentációját) és gyorsítja az olvasást/írást.
- **Visszafelé kompatibilitás:** Egy ext3 fájlrendszert könnyedén fel lehet frissíteni ext4-re formázás és adatvesztés nélkül.
- **Felhasználás:** Általános célú asztali gépekhez, laptopokhoz és hagyományos szerverekhez a legmegbízhatóbb, "gondozásmentes" választás.

## XFS

Eredetileg a **Silicon Graphics (SGI)** fejlesztette ki nagyteljesítményű UNIX rendszerekhez, majd átkerült Linuxra. A **Red Hat Enterprise Linux (RHEL)** és a **CentOS** 7-es verziójától kezdve ez az alapértelmezett fájlrendszer.

- **Párhuzamos feldolgozás:** Az XFS extrém jól skálázódik. Képes a bemeneti/kimeneti (I/O) műveleteket szálakra bontani, így rengeteg párhuzamos írási/olvasási műveletet tud kezelni lassulás nélkül.
- **Nagy fájlok kezelése:** Kifejezetten a nagyméretű fájlokhoz (például adatbázisok, nagy médiafájlok, virtuális gépek lemezképei) van optimalizálva.
- **Különleges korlát (Shrink hiánya):** A legnagyobb hátránya, hogy az XFS partíciók méretét csak növelni lehet, csökkenteni (shrink) nem. Ha rosszul particionálsz, nem tudsz utólag helyet felszabadítani róla más partíciók számára (csak ha letörlöd és újraformázod).

## Btrfs (B-Tree Filesystem, gyakran "Butter FS"-nek ejtik)

Egy modern, új generációs fájlrendszer, amely a **ZFS** nyomdokaiba lép. A **Fedora** és a **SUSE/openSUSE** rendszerek alapértelmezett fájlrendszere.

- **Copy-on-Write (CoW):** Ha módosítasz egy fájlt, a rendszer nem írja felül az eredetit, hanem egy új blokkba menti a változást. Ez megvédi az adatokat a sérüléstől, ha írás közben áramszünet történik.
- **Pillanatképek (Snapshots):** Ez a Btrfs legnagyobb fegyvere. A CoW technológia miatt a másodperc töredéke alatt készíthető egy "fotó" a rendszer aktuális állapotáról, amely nem foglal extra helyet. Ha egy frissítés tönkreteszi a Linuxot, a boot menüből azonnal visszaállítható a rendszer a korábbi, működő állapotba.
- **Fejlett funkciók:** Beépített szoftveres RAID támogatással (több lemez összefűzése), transzparens adattömörítéssel (automatikusan tömöríti a fájlokat a háttérben, így helyet spórol), és önjavító funkciókkal rendelkezik.

## FAT / FAT32 (File Allocation Table)

Bár ez egy eredetileg Microsoft által fejlesztett (Windowsos) fájlrendszer, a videóban is szerepel, mert megkerülhetetlen a számítástechnikában.

- **Univerzális kompatibilitás:** Szinte minden operációs rendszer (Windows, macOS, Linux) és eszköz (okostévék, kamerák, autós rádiók) ismeri és tudja olvasni/írni. Ezért vannak a pendrive-ok gyárilag FAT32-re formázva.
- **Korlátok:** Nincs benne naplózás (könnyebben sérülnek az adatok), nem támogatja a Linuxos fájljogosultságokat, és a FAT32 legnagyobb hátránya, hogy egy fájl nem lehet nagyobb **4 GB**-nál (ezt az **exFAT** fájlrendszer oldotta meg később).
- **Linuxos felhasználás:** Leginkább az UEFI rendszerek EFI boot partíciója használja, illetve cserélhető adathordozókon találkozhatsz vele, magát a Linux rendszert soha nem telepítik FAT partícióra.

### Különbség a partíció és a fájlrendszer között

- **Partíció:** A lemez egy fizikailag összefüggő, logikailag elkülönített része (olyan, mint egy tároló konténer).
- **Fájlrendszer:** A fájlok tárolásának és megtalálásának módszere, amely a partíción belül helyezkedik el. (Bizonyos esetekben szimbolikus linkekkel egy fájlrendszer több partíción is átívelhet).
- **Biztonsági előny:** A merevlemez partíciókra osztásával az adatok szeparálhatók. Ha hiba történik, csak az érintett partíción lévő adatok sérülnek, a többi túléli.

### Különbségek a Linux és a Windows fájlrendszere között

- **Fa struktúra és meghajtóbetűk:** A Windows `C:`, `D:` stb. meghajtókat használ. A Linuxnak nincsenek meghajtóbetűjelei. Egyetlen, egységes fájlrendszer-hierarchia létezik, amely a gyökérből (`/`) indul. Minden más fizikai meghajtó vagy partíció ebbe a fába van becsatolva (mountolva) mint egy könyvtár.
- **Elválasztó karakter:** A Windows fordított perjelet (`\`), a Linux normál perjelet (`/`) használ az útvonalak elválasztására.
- **Kis- és nagybetűk érzékenysége (Case sensitivity):** A Linux fájlrendszerei megkülönböztetik a kis- és nagybetűket. A `boot`, `Boot` és `BOOT` három teljesen különálló mappát vagy fájlt jelent. Windows alatt ezek ugyanazt jelentik.
- **Cserélhető adathordozók (USB, CD):** Ezek modern Linuxokon automatikusan a `/run/media/felhasznalonev/cimke` (pl. `/run/media/student/Fedora/`) mappába csatolódnak, régebbi rendszereken a `/media` mappába.

### File System Hierarchy Standard (FHS)

A Linux Foundation által karbantartott szabvány, amely meghatározza a fájlrendszer alapvető felépítését. Ennek köszönhetően a felhasználók és fejlesztők anélkül válthatnak disztribúciót, hogy újra kellene tanulniuk a rendszer felépítését (pl. a felhasználói programok a `/usr` mappába kerülnek, a konfigurációk az `/etc`-be). Grafikus felületen (GNOME) is könnyen navigálhatunk a gyökérkönyvtárból kiindulva (pl. a Fájlkezelő "Számítógép" vagy "Computer" menüpontjából).

## Disztribúció választása és telepítés tervezése

Telepítés előtt számos tényezőt kell mérlegelni:

- **Funkció és szoftverek:** Webes szerver lesz, vagy asztali gép? Milyen szoftverek (pl. Apache, MySQL, Firefox) kellenek?
- **Hardver és tárhely:** Mennyi a hely (pl. beágyazott rendszereknél kritikus)? Milyen architektúra (x86, ARM, PPC)?
- **Frissítési ciklus:** Szükség van-e **LTS** (Long-Term Support) hosszútávú stabilitásra, vagy a legújabb "cutting edge" szoftverek kellenek?

### Telepítési folyamat és opciók

- **Automatizált telepítések:** A telepítők konfigurációs fájlokból teljesen automatizálhatók (**Kickstart** fájl a Red Hatnél, **AutoYaST** a SUSE-nál, **Preseed** a Debiannál).
- **Szoftverek és Biztonság:** A telepítő alapértelmezetten telepít egy asztali környezetet (ha olyat választottunk, pl. GNOME/KDE), és létrehoz egy root jelszót, valamint egy sima felhasználót (az Ubuntunál a root jelszavas belépés le van tiltva, és `sudo` kell a normál fiókból). Extra biztonsági keretrendszerek is települnek: **SELinux** (Red Hat/CentOS/Fedora esetén alapértelmezett) vagy **AppArmor** (Ubuntu esetén).
- **Biztonságos kipróbálás/telepítés:** Hogy ne veszítsünk adatot, használhatunk **Dual-boot**-ot (particionálással), **virtuális gépeket** (VMware, VirtualBox), vagy **Live CD/USB**-t (ami egyáltalán nem ír a lemezre).

### Példa telepítések tapasztalatai a kurzusból (Particionálási fókusszal)

- **Ubuntu (VMware-ben):** Gyors, szinte beavatkozást nem igénylő folyamat. A kurzuson bemutatott verzió nem csinált külön swap (lapozó) partíciót, hanem egy kb. 1,5 GB-os swap fájlt hozott létre. A fő partíció alapértelmezetten `ext4` fájlrendszerű lett. A `df -Th` paranccsal ellenőrizhető a fájlrendszerek típusa és mérete.
- **CentOS 8:** Itt már kézzel lett beállítva a partíció (Custom storage configuration). A Red Hat (és így a CentOS) alapértelmezett fájlrendszere az **XFS**, de az oktató átállította ezt `ext4`-re, mert az kompatibilisebb más rendszerekkel. Külön 1 GB-os Swap partíciót is létrehozott a 29 GB-os gyökér (`/`) mellé.
- **openSUSE:** Telepítésnél javasol egy partíciós sémát (itt 2 GB swap partíció és 28 GB gyökér `ext4`-re formázva, külön home partíció nélkül a puritánabb telepítés érdekében). Telepítés után azonnal hálózatból letölti és telepíti a frissítéseket.


# 4. fejezet: Grafikus felhasználói felület (GUI)

Ebben a fejezetben a Linux grafikus felületének alapvető használatát, a munkamenetek (session) kezelését, valamint a testreszabási lehetőségeket ismertetik. Bár a parancssor (CLI) hatékonyabb az ismétlődő feladatoknál, a grafikus felület (GUI) megkönnyíti a navigációt és a ritkábban használt funkciók elérését.

## X Window System és Wayland

- **X Window System (X11 vagy X):** Ez a grafikus rendszer hagyományos alapja a Linuxban (az 1980-as évek közepe óta létezik). A **Display Manager** (képernyőkezelő) elindítja az X servert, amely grafikus szolgáltatásokat nyújt az alkalmazások (kliensek) számára, és kezeli a grafikus bejelentkezést. Ha nem indul el automatikusan, a terminálból a `startx` paranccsal indítható.
- **Wayland:** Egy újabb, modernebb kijelző-kiszolgáló protokoll, amely lassan leváltja a régebbi, biztonsági és strukturális hiányosságokkal küzdő X11-et. Sok modern disztribúcióban (pl. RHEL 8, Fedora, CentOS Stream) már a Wayland az alapértelmezett, bár a felhasználó számára a működés vizuálisan szinte azonos az X11-gyel.

## Asztali környezetek (Desktop Environments)

Egy asztali környezet egy **munkamenet-kezelőből** (session manager) és egy **ablakkezelőből** (window manager) áll, amelyek együttesen alkotják a teljes felhasználói élményt (menük, ablakok, ikonok).

- **GNOME:** A legnépszerűbb környezet, amely alapértelmezett a Red Hat, Fedora, CentOS, SUSE, Ubuntu és Debian rendszerekben. Menüalapú navigációt kínál. Bár az alapja ugyanaz (GNOME), a különböző disztribúciók (pl. az Ubuntu oldalsó sávja vs. a CentOS letisztult asztala) vizuálisan eltérően szabhatják testre.
- **KDE:** Egy másik nagyon népszerű és történelmileg fontos környezet, amelyet gyakran a SUSE/openSUSE rendszerekkel használnak, és hagyományosan (bal alsó sarokból nyíló menüvel) jobban hasonlít a klasszikus Windows elrendezésre.

## Munkamenetek kezelése (Session Management)

A Linux egy valódi **többfelhasználós (multi-user)** rendszer, amely lehetővé teszi több felhasználó egyidejű jelenlétét.

- **Bejelentkezés / Kijelentkezés (Login / Logout):** A bejelentkezési képernyőn (greeter) kiválasztható a felhasználó. A kijelentkezés minden futó folyamatot leállít az adott munkamenetben, ezért előtte mindig menteni kell a munkát.
- **Felhasználóváltás (Switch User):** Ha a kijelentkezés helyett a felhasználóváltást választjuk, a rendszer zárolja a képernyőt, de a háttérben futni hagyja az összes alkalmazást és nyitott ablakot. Így egy másik felhasználó beléphet a saját fiókjába, majd visszaváltva pontosan onnan folytathatjuk, ahol abbahagytuk.
- **Képernyőzárolás (Lock Screen):** A jobb felső sarokban lévő menüből, vagy a **Super Key** (Windows gomb) + **L** (esetleg Super + Esc) billentyűkombinációval zárolható a gép. Ez nem állítja le a programokat.
- **Alvó mód / Felfüggesztés (Suspend / Sleep):** A rendszerállapotot a memóriába (RAM) menti, és lekapcsolja a hardver többi részét (ezzel áramot takarít meg). Mivel a modern Linux rendszerek villámgyorsan bootolnak, az alvó móddal megspórolt idő ma már elenyésző, de laptopoknál hasznos funkció.

## Testreszabás és Alkalmazások

- **Megjelenés:** A háttérkép (wallpaper) az asztalon jobb gombbal kattintva (Change Background) könnyen cserélhető, akár saját képre is.
- **Témák (Themes):** A teljes vizuális élményt (ablakok kerete, gombok kinézete, görgetősávok) a témák határozzák meg.
- **Gnome Tweaks:** Mivel az alapértelmezett beállítási menü (Settings) a GNOME-ban sokszor túl egyszerű (a "Quest for Simplicity" jegyében kivették a haladó funkciókat), érdemes telepíteni a **Gnome Tweaks** segédprogramot. Ezzel sokkal finomabban beállítható a rendszer (pl. téma csere, vagy az `Alt+F2` használata parancsok futtatására, esetleg a Caps Lock billentyű funkciójának megváltoztatása).
- **Alkalmazások indítása:** Az "Activities" vagy "Applications" menüből (általában a bal felső sarokból) indíthatók.
- **Alapértelmezett alkalmazások (Default Apps):** A rendszerbeállítások (Settings) → Részletek (Details) → Default Applications menüben állítható be, hogy melyik fájltípust melyik program nyissa meg (pl. beállítható a Firefox vagy Chrome böngészőnek, a VLC videólejátszónak). A Linux eleve rengeteg szoftverrel érkezik (LibreOffice, böngészők).

## Fájlkezelés (Nautilus)

A GNOME alapértelmezett fájlkezelője a **Nautilus** (gyakran csak "Files" vagy "Fájlok" néven jelenik meg az ikonja).

- **Saját könyvtár (Home):** Minden felhasználónak van egy saját könyvtára (pl. `/home/student`), ahol a rendszer alapértelmezetten létrehozza az alapvető mappákat (Dokumentumok, Képek, Letöltések, Asztal). A mentett fájlok alapból ide kerülnek.
- **Nézetek:** A fájlok listaként vagy ikonokként is megjeleníthetők (váltás: gombokkal a fejlécen, vagy `Ctrl+1` / `Ctrl+2`). Nagyítani/kicsinyíteni (Zoom) is lehet a fájlkezelőben az ikonokat.
- **Rejtett fájlok (Hidden files):** A Linuxban a rejtett fájlok (általában konfigurációs fájlok) neve ponttal kezdődik (pl. `.config`). Megjelenítésük: menüből vagy a `Ctrl+H` billentyűkombinációval.
- **Keresés:** A fájlkezelő beépített keresője rekurzívan keres az aktuális mappában és alkönyvtáraiban, ha elkezdünk gépelni a keresőmezőbe.
- **Törlés:** A fájl kijelölése után a törlés gomb (`Delete` vagy `Ctrl+Delete`) a **Kuka (Trash)** mappába teszi a fájlokat (amely a `~/.local/share/Trash/files` alatt található). A `Shift + Delete` billentyűkombináció véglegesen törli a fájlt, megkerülve a kukát.

# 5. fejezet: Rendszerkonfiguráció grafikus felületről

Ez a fejezet a Linux rendszerek alapvető beállításainak – dátum és idő, kijelző, hálózat, valamint a szoftvercsomagok kezelésének – grafikus felületen (GUI) történő elvégzését mutatja be.

## 1. Rendszerbeállítások (System Settings)

A legtöbb alapvető beállítás (képernyőfelbontás, hálózat, dátum/idő, felhasználók) elérhető a **System Settings** (Rendszerbeállítások) panelen.

- **Elérés:** A jobb felső sarokban található menüből a "fogaskerék" (vagy néhány disztribúciónál csavarhúzó/villáskulcs) ikonra kattintva érhető el.
- **Gnome Tweaks (Haladó beállítások):** Bár a Settings menü szándékosan leegyszerűsített, sok testreszabási lehetőséget elrejtettek. A **Gnome Tweaks** eszközzel módosíthatók a témák, a betűtípusok, a billentyűzetkiosztások és az automatikusan induló programok. Ezt a programot sokszor külön kell telepíteni.
- **Gnome Extensions:** A legújabb GNOME verziókban a kiegészítők (extensions) kezelését leválasztották a Tweaks-ről, és egy külön **"Gnome Extensions"** alkalmazásba tették át.

## 2. Kijelző (Display) beállításai

- A **"Settings → Devices → Displays"** menüpontban lehet módosítani a képernyő felbontását. Ha a választott felbontás nem működik, a rendszer egy rövid időkorlát (timeout) után automatikusan visszaáll az eredetire.
- **Több monitor:** Ha több monitort használunk, a rendszer alapértelmezetten egy nagy kiterjesztett asztalt hoz létre. Lehetőség van a képernyők sorrendjének változtatására (drag and drop), vagy a **"Tükrözés"** (Mirrored mode) bekapcsolására, ahol minden monitor ugyanazt mutatja.
- **Zárt forráskódú illesztők (NVIDIA/AMD):** Ha a rendszer proprietáris videókártya-drivert használ, annak gyakran van egy saját konfigurációs segédprogramja (amihez root jelszó kellhet), de ha lehetséges, az alapvető beállításokat érdemes továbbra is a GNOME beépített paneljén végezni.

## 3. Dátum és Idő

- **UTC (Coordinated Universal Time):** A Linux a belső időméréshez mindig az **UTC-t** (koordinált világidő) használja. A felhasználó felé mutatott időt kizárólag az időzóna (Time Zone) beállítása határozza meg.
- **NTP (Network Time Protocol):** A Linux disztribúciók mindig beépített, megbízható időszerverekkel kommunikálnak (NTP) az óra szinkronizálásához. Ehhez általában semmilyen manuális beállítás nem szükséges az **"Automatic Date & Time"** (On/Off) kapcsolón kívül.

## 4. Hálózatkezelés (Network Manager)

A **Network Manager** egy egységes, minden modern disztribúcióban jelen lévő intelligens eszköz a hálózatok (vezetékes, Wi-Fi, mobilnet, VPN) menedzselésére.

- **Vezetékes (Wired):** Általában automatikusan konfigurálódik, amint bedugják a kábelt, a **DHCP** (Dynamic Host Configuration Protocol) segítségével.
- **Wi-Fi:** A jobb felső sarokban megjelenik az elérhető hálózatok listája. Biztonságos hálózat esetén (lakat ikon) első alkalommal jelszót kér, amit a későbbi csatlakozásokhoz elment.
- **Speciális beállítások (IPv4/IPv6):** A "fogaskerék" ikonra kattintva beállítható fix IP cím, DNS szerver, Gateway, statikus routing, vagy akár megváltoztatható a hálózati kártya **MAC** (Media Access Control) címe is. A hálózati beállítások módosításához gyakran root (admin) jelszó szükséges.
- **VPN (Virtual Private Network):** A Network Manager alapból támogatja a legtöbb VPN protokollt (Cisco IPSec, Cisco AnyConnect/OpenConnect, Microsoft PPTP, OpenVPN).

## 5. Szoftverkezelés és Csomagkezelők (Package Managers)

A Linux rendszerekben a szoftverek **"csomagok"** (packages) formájában vannak. A telepítőprogramok legnagyobb előnye, hogy a **függőségeket** (dependencies) automatikusan megoldják. (Például: ha egy e-mail kliens titkosítást használ, a csomagkezelő automatikusan letölti hozzá az SSL/TLS titkosító könyvtárat is, anélkül, hogy a felhasználónak kellene keresgélnie.)

Bár a tapasztalt rendszergazdák a parancssort preferálják, minden rendszer rendelkezik grafikus szoftverközponttal (pl. **"Gnome Software"**, ami az okostelefonos alkalmazásboltokhoz hasonló).

### A két fő csomagkezelő rendszer

#### 1. Debian család (Ubuntu, Mint)

- **Alapok:** Az alacsony szintű csomagkezelő a **dpkg** (ez önmagában nem tölt le az internetről és nem old meg függőségeket). A magasabb szintű eszköz az **APT** (Advanced Package Tool).
- **Grafikus felületek:**
  - **Ubuntu Software:** Modern, felhasználóbarát "App Store" kinézetű felület.
  - **Synaptic Package Manager:** Egy régebbi, de nagyon átlátható és hatékony eszköz, amellyel rendkívül részletesen kereshetünk, telepíthetünk vagy távolíthatunk el csomagokat.

#### 2. Red Hat és SUSE család (Fedora, CentOS, openSUSE)

- **Alapok:** A csomagkezelő rendszer az **RPM** (Red Hat Package Manager).
- **Magas szintű eszközök:** A Red Hat/Fedora rendszerek a **dnf** (régebben **yum**) parancssori eszközt használják, míg a SUSE a **Zypper**-t.
- **Grafikus felület (SUSE):** A SUSE ikonikus rendszeradminisztrációs központja a **YaST** (Yet another Setup Tool). Ennek a szoftverkezelő moduljában csomagcsoportok (RPM groups) vagy kategóriák szerint is lehet böngészni és telepíteni szoftvereket. Bármilyen csomagkezelési művelethez a grafikus felületen root jelszó megadása szükséges.

# 5. fejezet: Rendszerkonfiguráció grafikus felületről

Ez a fejezet a Linux rendszerek alapvető beállításainak – dátum és idő, kijelző, hálózat, valamint a szoftvercsomagok kezelésének – grafikus felületen (GUI) történő elvégzését mutatja be.

## 1. Rendszerbeállítások (System Settings)

A legtöbb alapvető beállítás (képernyőfelbontás, hálózat, dátum/idő, felhasználók) elérhető a **System Settings** (Rendszerbeállítások) panelen.

- **Elérés:** A jobb felső sarokban található menüből a "fogaskerék" (vagy néhány disztribúciónál csavarhúzó/villáskulcs) ikonra kattintva érhető el.
- **Gnome Tweaks (Haladó beállítások):** Bár a Settings menü szándékosan leegyszerűsített, sok testreszabási lehetőséget elrejtettek. A **Gnome Tweaks** eszközzel módosíthatók a témák, a betűtípusok, a billentyűzetkiosztások és az automatikusan induló programok. Ezt a programot sokszor külön kell telepíteni.
- **Gnome Extensions:** A legújabb GNOME verziókban a kiegészítők (extensions) kezelését leválasztották a Tweaks-ről, és egy külön **"Gnome Extensions"** alkalmazásba tették át.

## 2. Kijelző (Display) beállításai

- A **"Settings → Devices → Displays"** menüpontban lehet módosítani a képernyő felbontását. Ha a választott felbontás nem működik, a rendszer egy rövid időkorlát (timeout) után automatikusan visszaáll az eredetire.
- **Több monitor:** Ha több monitort használunk, a rendszer alapértelmezetten egy nagy kiterjesztett asztalt hoz létre. Lehetőség van a képernyők sorrendjének változtatására (drag and drop), vagy a **"Tükrözés"** (Mirrored mode) bekapcsolására, ahol minden monitor ugyanazt mutatja.
- **Zárt forráskódú illesztők (NVIDIA/AMD):** Ha a rendszer proprietáris videókártya-drivert használ, annak gyakran van egy saját konfigurációs segédprogramja (amihez root jelszó kellhet), de ha lehetséges, az alapvető beállításokat érdemes továbbra is a GNOME beépített paneljén végezni.

## 3. Dátum és Idő

- **UTC (Coordinated Universal Time):** A Linux a belső időméréshez mindig az **UTC-t** (koordinált világidő) használja. A felhasználó felé mutatott időt kizárólag az időzóna (Time Zone) beállítása határozza meg.
- **NTP (Network Time Protocol):** A Linux disztribúciók mindig beépített, megbízható időszerverekkel kommunikálnak (NTP) az óra szinkronizálásához. Ehhez általában semmilyen manuális beállítás nem szükséges az **"Automatic Date & Time"** (On/Off) kapcsolón kívül.

## 4. Hálózatkezelés (Network Manager)

A **Network Manager** egy egységes, minden modern disztribúcióban jelen lévő intelligens eszköz a hálózatok (vezetékes, Wi-Fi, mobilnet, VPN) menedzselésére.

- **Vezetékes (Wired):** Általában automatikusan konfigurálódik, amint bedugják a kábelt, a **DHCP** (Dynamic Host Configuration Protocol) segítségével.
- **Wi-Fi:** A jobb felső sarokban megjelenik az elérhető hálózatok listája. Biztonságos hálózat esetén (lakat ikon) első alkalommal jelszót kér, amit a későbbi csatlakozásokhoz elment.
- **Speciális beállítások (IPv4/IPv6):** A "fogaskerék" ikonra kattintva beállítható fix IP cím, DNS szerver, Gateway, statikus routing, vagy akár megváltoztatható a hálózati kártya **MAC** (Media Access Control) címe is. A hálózati beállítások módosításához gyakran root (admin) jelszó szükséges.
- **VPN (Virtual Private Network):** A Network Manager alapból támogatja a legtöbb VPN protokollt (Cisco IPSec, Cisco AnyConnect/OpenConnect, Microsoft PPTP, OpenVPN).

## 5. Szoftverkezelés és Csomagkezelők (Package Managers)

A Linux rendszerekben a szoftverek **"csomagok"** (packages) formájában vannak. A telepítőprogramok legnagyobb előnye, hogy a **függőségeket** (dependencies) automatikusan megoldják. (Például: ha egy e-mail kliens titkosítást használ, a csomagkezelő automatikusan letölti hozzá az SSL/TLS titkosító könyvtárat is, anélkül, hogy a felhasználónak kellene keresgélnie.)

Bár a tapasztalt rendszergazdák a parancssort preferálják, minden rendszer rendelkezik grafikus szoftverközponttal (pl. **"Gnome Software"**, ami az okostelefonos alkalmazásboltokhoz hasonló).

### A két fő csomagkezelő rendszer

#### 1. Debian család (Ubuntu, Mint)

- **Alapok:** Az alacsony szintű csomagkezelő a **dpkg** (ez önmagában nem tölt le az internetről és nem old meg függőségeket). A magasabb szintű eszköz az **APT** (Advanced Package Tool).
- **Grafikus felületek:**
  - **Ubuntu Software:** Modern, felhasználóbarát "App Store" kinézetű felület.
  - **Synaptic Package Manager:** Egy régebbi, de nagyon átlátható és hatékony eszköz, amellyel rendkívül részletesen kereshetünk, telepíthetünk vagy távolíthatunk el csomagokat.

#### 2. Red Hat és SUSE család (Fedora, CentOS, openSUSE)

- **Alapok:** A csomagkezelő rendszer az **RPM** (Red Hat Package Manager).
- **Magas szintű eszközök:** A Red Hat/Fedora rendszerek a **dnf** (régebben **yum**) parancssori eszközt használják, míg a SUSE a **Zypper**-t.
- **Grafikus felület (SUSE):** A SUSE ikonikus rendszeradminisztrációs központja a **YaST** (Yet another Setup Tool). Ennek a szoftverkezelő moduljában csomagcsoportok (RPM groups) vagy kategóriák szerint is lehet böngészni és telepíteni szoftvereket. Bármilyen csomagkezelési művelethez a grafikus felületen root jelszó megadása szükséges.

# 6. fejezet: Gyakori alkalmazások (Common Applications)

A Linux rendszerek széles választékban kínálnak szoftvereket a mindennapi feladatok elvégzésére. Az egyik legnagyobb előny, hogy ezek a programok a csomagkezelőkön keresztül teljesen ingyenesen és biztonságosan telepíthetők.

## 1. Internetes alkalmazások

A webalapú kommunikációhoz és böngészéshez a Linux grafikus és szöveges (parancssori) alkalmazásokat is kínál.

- **Webböngészők:**
  - Grafikus böngészők: **Firefox** (sok disztribúció alapértelmezettje), **Google Chrome**, **Chromium** (a Chrome nyílt forráskódú alapja), **Epiphany** (újabb nevén GNOME Web), **Konqueror**, **Opera**.
  - Szöveges (terminálos) böngészők: **Lynx**, **Links**, **w3m** (ezek grafikus felület nélkül, tisztán szövegesen jelenítik meg a weboldalakat).
- **E-mail kliensek (IMAP és POP protokollokhoz):**
  - Grafikus e-mail kliensek: **Thunderbird**, **Evolution**, **Claws Mail**.
  - Szöveges e-mail kliensek: **mutt**, **mail**.
  - Webes kliensek: Természetesen a böngészőből elérhető szolgáltatások (Gmail, Yahoo, Office 365) is tökéletesen működnek.
- **Egyéb hálózati és kommunikációs szoftverek:** **FileZilla** (fájlátvitel / FTP), **Pidgin**, **HexChat** (IRC/cset alkalmazások), **Ekiga** (videókonferencia).

## 2. Irodai szoftvercsomagok (Office Suites)

A mindennapi irodai munkára a Linux a felhőalapú megoldások (Google Docs, Microsoft Office 365 webes verziója) mellett robusztus asztali alkalmazásokat is biztosít.

- **LibreOffice:** A legkiforrottabb és legelterjedtebb nyílt forráskódú irodai csomag, amely 2010-ben vált ki az OpenOffice-ból. Teljesen ingyenes.
  - **Részei:** Writer (szövegszerkesztő), Calc (táblázatkezelő), Impress (prezentációkészítő), Draw (vektorgrafika és diagramok).
  - **Kompatibilitás:** A LibreOffice képes olvasni és menteni a nem natív, Microsoft Office formátumokat (pl. `.docx`, `.xlsx`). Általában nagyon jó a konverzió, bár extrém bonyolult dokumentumoknál előfordulhatnak kisebb formázási eltérések.

## 3. Fejlesztői eszközök (Developer Tools)

A Linux a szoftverfejlesztők elsődleges rendszere, mivel az összes szükséges eszköz natívan, ingyenesen és integrálva elérhető rajta (szemben más operációs rendszerekkel, ahol ezek gyakran drágák vagy külön kell összevadászni őket).

- **Szövegszerkesztők kódoláshoz:** **Vim**, **Emacs** (fejlett, programozókra szabott szerkesztők).
- **Fordítók (Compilers):** Szinte az összes valaha létezett programozási nyelvhez elérhetők, beleértve a legmodernebbeket is (pl. Go, Rust, C/C++).
- **Hibakeresők (Debuggers):** **GDB** (GNU Debugger) és ennek különböző grafikus felületei, valamint teljesítménymérő és monitorozó szoftverek.
- **Integrált fejlesztői környezetek (IDE):** **Eclipse**, **Visual Studio Code** (amelyek ezeket az eszközöket egyetlen felületen fogják össze).

## 4. Multimédia alkalmazások (Zene és Videó)

A Linux rendszerek kiválóan alkalmasak tartalomfogyasztásra és -készítésre is, a böngészőben futó streaming szolgáltatások (Spotify, Pandora stb.) mellett dedikált programokkal is.

- **Zene- és hanglejátszók:** **Amarok**, **Audacity** (hangszerkesztő is egyben), **Rhythmbox**.
- **Videó- és filmlejátszók:** **VLC** (az egyik legnépszerűbb, mindent lejátszó szoftver), **MPlayer**, **Xine**, **Totem** (a GNOME beépített lejátszója).
- **Videószerkesztők:** **Kino**, **CinePaint**, **Blender** (professzionális 3D modellező és videószerkesztő), **FFmpeg** (rendkívül erős parancssori multimédia-kezelő).

## 5. Grafikai szerkesztők

- **GIMP (GNU Image Manipulation Program):** A Linux "Adobe Photoshopja". Egy hihetetlenül gazdag funkciókkal rendelkező képszerkesztő. Képes kezelni bármilyen képformátumot, támogatja a rétegeket (layers), csatornákat, hisztogramokat, és rengeteg szűrővel/bővítménnyel rendelkezik.
- **Egyéb grafikai eszközök:**
  - **EOG** (Eye of GNOME): Alapértelmezett, egyszerű képmegjelenítő.
  - **Inkscape:** Professzionális vektorgrafikus szerkesztő (mint az Adobe Illustrator).
  - **Convert** (ImageMagick): Erőteljes parancssori képalakító eszköz.
  - **Scribus:** Kiadványszerkesztő szoftver (Desktop Publishing).

  # Parancssori felület (CLI)

A parancssoros felület (CLI) elengedhetetlen a Linux rendszergazdák számára. Míg a grafikus felület az egyszerű feladatokat könnyíti meg, a parancssor a bonyolult feladatokat teszi lehetővé minimális erőforrás-igénnyel, kiváló szkriptelhetőséggel és távoli eléréssel (pl. `ssh felhasznalo@szerver.com` segítségével).

## Terminálok és Rendszervezérlés

- **Terminál emulátorok:** Ablakban futó parancssorok (**GNOME Terminal**, **xterm**, **KDE Konsole**).
- **Virtuális terminálok (VT):** Tiszta szöveges felületek, a `Ctrl+Alt+F1` – `F7` billentyűkkel érhetők el (a grafikus felület általában a VT1 vagy VT7 alatt fut). Fagyás esetén hibaelhárításra tökéletes.
- **GUI indítása/leállítása:** A `sudo systemctl stop gdm` leállítja, a `start gdm` elindítja a grafikus felületet.
- **Rendszerleállítás:** `shutdown -h now` (leállítás), `shutdown -r` (újraindítás).

## Alapvető parancsok és jogosultságok

- **A parancsok felépítése:** `parancs [opciók] [argumentumok]`.
- **sudo:** Rendszergazdai (root) jogosultságot ad az adott parancs idejére. (Beállításához az `/etc/sudoers.d/` alatt kell konfigurációs fájlt létrehozni).
- **Navigáció:**
  - `pwd` (jelenlegi mappa)
  - `cd` (belépés)
  - `cd ~` vagy `cd $HOME` (saját mappa)
  - `cd ..` (vissza egy szintet)
  - `cd -` (vissza az előző mappába)
  - A `pushd` és `popd` parancsokkal mappákat tehetünk egy "verembe", így könnyen ugrálhatunk köztük.
- **Útvonalak:** Az **abszolút útvonal** a gyökérből indul (pl. `/usr/local/bin`), a **relatív útvonal** a jelenlegi helyhez viszonyít.
- **Keresés:** A futtatható programok helyét a `which` vagy a `whereis` parancs mutatja meg.

## Fájlműveletek és Linkek

- **Olvasás:**
  - `cat` (kiírja az egészet)
  - `less` (lapozható)
  - `head` / `tail` (első/utolsó 10 sor, paraméterezhető pl. `-20`)
  - `tac` (visszafelé olvassa a fájlt)
- **Kezelés:**
  - `touch` (üres fájlt hoz létre vagy frissíti a módosítás idejét)
  - `mkdir` (mappa létrehozása)
  - `rmdir` (üres mappa törlése)
  - `mv` (áthelyezés/átnevezés)
- **Törlés:**
  - `rm -i` (interaktív törlés rákérdezéssel)
  - `rm -rf` (mappák és tartalmuk rekurzív, kérdés nélküli törlése – **nagyon veszélyes**)
- **Hard link** (`ln fájl1 fájl2`): Ugyanarra a fizikai adatra (**inode**) mutat. Ha az eredetit törlik, az adat elérhető marad a másikon keresztül. Nem működik eltérő partíciók között.
- **Szimbolikus / Soft link** (`ln -s fájl1 fájl3`): Hagyományos parancsikon. Külön inode, de az eredeti fájlra mutat. Eltérő fájlrendszereken is működik.

## Bemenet, kimenet és csövezés (Piping)

A Linux három alapvető adatfolyamot (stream) használ: **stdin (0)**, **stdout (1)**, **stderr (2)**.

- **Átirányítás:** A kimenetet a `>` jellel fájlba írhatjuk. A hibákat (stderr) a `2> hiba.log` menti el. A `2>&1` a hibákat a normál kimenetre irányítja.
- **Csövezés (`|`):** Egy parancs kimenetét átadja a következő parancs bemenetének (pl. `ls | less`). Ez a többmagos rendszereken rendkívül gyors, mert a folyamatok párhuzamosan, a memóriában dolgoznak átmeneti fájlok nélkül.

## Keresés (Locate és Find)

A fájlkereséshez a Linux helyettesítő karaktereket (wildcards) támogat: `?` (egy karakter), `*` (bármennyi), `[a-z]` (tartomány).

- **locate:** Nagyon gyors, mert egy adatbázisból dolgozik. Ha új fájl készül, nem látja azonnal, frissíteni kell a `sudo updatedb` paranccsal (kivételeket a `/etc/updatedb.conf` tartalmazza).
- **find:** Valós időben, rekurzívan keres a mappastruktúrában.
  - **Opciók:** `-name` (pontos név), `-iname` (kis/nagybetű független), `-type d` vagy `f` (mappa vagy fájl), `-size` (méret), `-mtime` (módosítás ideje).
  - A `-maxdepth 1` opcióval megakadályozható, hogy almappákba is belépjen.
  - A találatokon parancsot is végrehajthatunk: `find . -type f -exec grep "hiba" {} \;` (A `{}` jelöli a megtalált fájlt). A `-ok` opció ugyanez, de végrehajtás előtt engedélyt kér.

## Csomagkezelés

A csomagkezelők feloldják a programok telepítéséhez szükséges **függőségeket** (dependencies). Két szinten működnek: alacsony szinten (csak telepít, nem tölt le) és magas szinten (internetről letölt, függőséget felold).

| Család | Alacsony szintű eszköz (példák) | Magas szintű eszköz (példák) |
|---|---|---|
| **Debian / Ubuntu** | `dpkg` <br> `-l` (telepített csomagok listája) <br> `-L csomagnev` (csomag fájljai) <br> `-r` (törlés függőségekkel együtt hibát dob) | `apt` / `apt-get` / `apt-cache` <br> `apt-cache search szoftver` <br> `apt-get install szoftver` <br> `apt-get remove` |
| **Red Hat / CentOS / Fedora** | `rpm` <br> `-qa` (minden csomag) <br> `-qi` (infó) <br> `-ql` (fájllista) <br> `-e --test` (próbatörlés) | `dnf` (vagy régebben `yum`) <br> `dnf list` <br> `dnf install` <br> `dnf remove` |
| **SUSE / openSUSE** | `rpm` (megegyezik a Red Hattel) | `zypper` <br> `zypper search` <br> `zypper install` <br> `zypper remove` |


# 8. fejezet: Linux dokumentáció keresése (Finding Documentation)

A Linux rendszerek rengeteg beépített segítséget és dokumentációt tartalmaznak, amelyeket a disztribútorok egységesítve tesznek elérhetővé. A négy legfontosabb forrás: a **Man pages**, a **GNU info**, a **--help** opció, és a **help** parancs.

## 1. A Man pages (Kézikönyv oldalak)

A leggyakrabban használt dokumentációs forrás. Részletes leírást ad a programokról, konfigurációs fájlokról, rendszerhívásokról, könyvtárakról és a kernelről. Gyökerei az 1970-es évekbeli Unix rendszerekig nyúlnak vissza (a "man" a *manual* rövidítése).

- **Alapvető használat:** A `man [téma]` (pl. `man socket`) megnyitja a vonatkozó leírást. Mivel ezek hosszúak, a kimenet egy lapozó programon (általában a `less`-en) keresztül jelenik meg.
- **Navigáció:** Szóköz (lapozás), nyilak (görgetés), `q` (kilépés).
- **Szekciók (Fejezetek):** Egy témának több man oldala is lehet különböző kategóriákban (pl. parancsok, rendszerhívások). Ha egy konkrét szekciót akarsz látni: `man 7 socket` (a 7. fejezetet nyitja meg a socketekről).
- **Keresés a Man oldalak között:**
  - `man -f [téma]` vagy `whatis [téma]`: Kilistázza az összes man oldalt, aminek a nevében pontosan szerepel a keresett szó (és megmutatja, melyik szekciókban találhatók).
  - `man -k [téma]` vagy `apropos [téma]`: Kilistázza az összes olyan man oldalt, amelynek a leírásában (description) szerepel a keresett szó.
  - `man -a [téma]`: Sorban, egymás után megnyitja az összes találatot. Ha az egyikből kilépsz (`q`), felajánlja a következőt (ezt a `Ctrl+D` lenyomásával ugorhatod át).

## 2. A GNU Info rendszer

A GNU projekt által létrehozott, a man oldalakat alternatívaként leváltani hivatott dokumentációs formátum. Ez egy szabad formátumú, linkekkel (hiperhivatkozásokkal) összekötött rendszer, amely úgy működik, mint egy weboldal, bár még a World Wide Web (WWW) kora előtt tervezték.

- **Alapvető használat:** Az `info` parancs önmagában egy fő tartalomjegyzéket nyit meg, az `info [téma]` (pl. `info make`) pedig az adott téma kezdőoldalát (Node).
- **Felépítés (Csomópontok / Nodes):** Az info rendszer fejezetekből és alfejezetekből, úgynevezett csomópontokból áll. A linkeket (amelyek más csomópontokra mutatnak) csillag `*` vagy dupla kettőspont `::` jelöli.
- **Navigáció:**
  - **Nyilak / PageUp / PageDown:** Normál mozgás a szövegben.
  - **Enter:** Rákattintás egy linkre (menüpontra).
  - **n (Next):** Ugrás a következő csomópontra (fejezetre) azonos szinten.
  - **p (Previous):** Ugrás az előző csomópontra.
  - **u (Up):** Egy szinttel feljebb lépés (pl. alfejezetből a főfejezetbe).
  - **/ (Perjel):** Keresés indítása a szövegben.
  - **h (Help):** Megjeleníti a billentyűkombinációk súgóját.
  - **q (Quit):** Kilépés.

## 3. A --help vagy -h opció

A legtöbb parancsnak és alkalmazásnak van egy beépített, rövidített súgója, amely sokkal gyorsabban lekérhető, mint a man vagy az info megnyitása.

- **Használat:** `parancs --help` vagy `parancs -h` (pl. `man --help`). Ez csak kiírja a terminálba az alapvető opciókat (kapcsolókat) és a használat szintaktikáját.

## 4. A help parancs (Bash beépített parancsokhoz)

Amikor a parancssort (Bash shell) használjuk, bizonyos gyakori parancsokat (pl. `echo`, `cd`) a rendszer nem a merevlemezről (fájlrendszerből) tölt be, hanem a Bash saját beépített (built-in) verzióját futtatja a sebesség és az erőforrás-hatékonyság érdekében. (Ezek a beépített parancsok minimálisan eltérhetnek a hagyományos bináris verzióktól).

- Ezekhez a beépített parancsokhoz **nem** a `--help` opciót, hanem a `help` parancsot kell használni.
- **Használat:** `help [parancs]` (pl. `help cd` vagy `help echo`). Ez pont úgy működik a beépített parancsoknál, mint a `--help` a külső programoknál.

## 5. Egyéb dokumentációs források

A fentieken túl a Linux rendszereken rendelkezésre állnak még:

- A grafikus asztali környezet beépített súgója (Desktop help system).
- Az adott telepített csomagok saját dokumentációi (amelyek hagyományosan a `/usr/share/doc/` könyvtárban találhatók).
- Az internetes, online források (amelyekből rengeteg érhető el a Linuxhoz).


# 10. fejezet: Processzusok (Folyamatok) kezelése

A **processzus (process)** egy vagy több kapcsolódó feladat (szál / thread) futó példánya a számítógépen. Fontos megkülönböztetni a **programtól** vagy a **parancstól**: egyetlen parancs kiadása egyszerre több folyamatot is elindíthat. A folyamatok rendszererőforrásokat (CPU, memória, hálózat) használnak, ezek megfelelő elosztásáért (allokáció) a Linux kernel felel.

## 1. Folyamatok típusai és állapotai

- **Típusok:**
  - **Interaktív:** Olyan folyamatok, amelyekhez felhasználói beavatkozás szükséges (pl. egy terminálablak).
  - **Nem interaktív (Háttérfolyamatok):** A shelltől leválasztva futnak, így a terminál szabad marad más feladatokra.
- **Állapotok** (A kernel ütemezője – Scheduler – irányítja):
  - **Running (Futó):** Éppen utasításokat hajt végre a CPU-n, vagy a Run Queue-ban várakozik a saját időszeletére (Time slice).
  - **Sleeping (Alvó):** A Wait Queue-ban várakozik, amíg egy esemény be nem következik (pl. a felhasználó leüt egy billentyűt, vagy befejeződik egy lemezolvasás).
  - **Zombie (Zombi):** A folyamat már befejeződött (halott), de a szülő folyamat (parent process) még nem kérdezte le az állapotát, így továbbra is látszik a rendszer folyamatlistájában, anélkül, hogy ténylegesen futna.

## 2. PID (Process ID) és azonosítók

A rendszer minden folyamatot egy egyedi azonosítóval, a **PID**-vel (Process ID) tart nyilván.

- A PID-k folyamatosan növekvő sorrendben kerülnek kiosztásra (A **PID 1** mindig az `init` vagy `systemd` folyamat).
- **Felhasználói azonosítók (UID)** és **Csoport azonosítók (GID):** A folyamatok ahhoz a felhasználóhoz (és csoporthoz) tartoznak, aki elindította őket. Megkülönböztetünk valós (Real – RUID) és effektív (Effective – EUID) azonosítókat, amelyek a jogosultságokat szabályozzák.

## 3. Folyamatok prioritása (Niceness)

Mivel egy CPU mag egyszerre csak egy feladatot tud elvégezni, a folyamatok prioritással rendelkeznek.

- **Nice érték (Niceness):** A Linuxban a prioritást a "niceness" (udvariasság) határozza meg. Skálája: **-20-tól +19-ig** terjed.
- **Fontos szabály:** A skála fordított. A **-20** a legmagasabb prioritás (a folyamat "nem udvarias", azonnal CPU-t követel), a **+19** a legalacsonyabb prioritás ("nagyon udvarias", átengedi a CPU-t másoknak). Az alapértelmezett érték a **0**.
- **Módosítás:**
  - `renice [érték] [PID]` (pl. `renice +5 3077`). A folyamat prioritásának csökkentését (niceness növelését) bárki megteheti a saját folyamataival.
  - A prioritás növelését (a niceness érték csökkentését mínusz irányba) biztonsági okokból kizárólag a **root** (`sudo`) felhasználó végezheti el.
  - (Grafikus felületen a **System Monitor** programból jobb kattintással, a *Change Priority* menüben is állítható.)

## 4. Rendszerterhelés (Load Average)

A rendszer pillanatnyi terheltségét a `w`, `top` vagy `uptime` parancsokkal lehet ellenőrizni.

- A kimenet 3 számot mutat (pl. `0.45, 0.17, 0.12`). Ezek a rendszer CPU kihasználtságát jelentik az elmúlt **1, 5 és 15 percben**.
- **Értelmezés:** Egy CPU mag esetén az **1.0** azt jelenti, hogy a processzor 100%-osan, de optimálisan ki van használva. Az 1.0 feletti érték túlterhelést jelent (a folyamatoknak várniuk kell a CPU-ra).
- **Többmagos rendszerek:** Ha 4 magos processzorod van, akkor a **4.0** érték jelenti a 100%-os kihasználtságot (a kapott load average számot mindig el kell osztani a CPU magok számával).

## 5. Folyamatok megfigyelése (top és ps)

- **ps (Process Status):** Pillanatkép a folyamatokról.
  - `ps` (csak az aktuális terminál folyamatai)
  - `ps -f` (teljes formátum, mutatja a szülő azonosítót, a PPID-t is)
  - `ps -l` (hosszú formátum, mutatja a niceness/prioritás értékeket is)
  - `ps -ef` vagy `ps aux` (a rendszer összes futó folyamatának részletes listája, az `aux` a CPU/Memória százalékot is mutatja).
  - `pstree`: Fa-struktúrában mutatja a folyamatokat, így jól látszanak a szülő-gyermek kapcsolatok.
- **top:** Valós idejű, folyamatosan frissülő (2-3 másodpercenként) monitorozó eszköz.
  - Mutatja a Load Average-t, a CPU és Memória (fizikai és Swap) használatot.
  - Interaktív parancsok a `top`-on belül: `1` (lebontás CPU magokra), `h` (segítség/help), `q` (kilépés), ezenkívül menet közben lehet vele folyamatokat kilőni vagy prioritást változtatni.

## 6. Folyamatok vezérlése (Előtér és Háttér)

Egy hosszú ideig futó folyamat (pl. biztonsági mentés) blokkolhatja a terminált. Ezt háttérbe lehet küldeni.

- **Indítás eleve háttérben:** Ha a parancs végére egy "és" jelet (`&`) teszünk, a folyamat a háttérben indul el (pl. `updatedb &`).
- **Futó folyamat felfüggesztése:** `Ctrl + Z` (a folyamat megáll, és visszakapjuk a terminált).
- **Folyamat kilövése (Megszakítás):** `Ctrl + C`.
- **Mozgatás:** A `bg` (background) parancs a felfüggesztett folyamatot elindítja a háttérben. Az `fg` (foreground) parancs visszahozza a háttérből a terminál előterébe.
- **Ellenőrzés:** A `jobs` parancs kilistázza a jelenlegi terminálhoz tartozó háttérfolyamatokat.
- **Kilövés parancssorból:** `kill -SIGKILL [PID]` vagy röviden `kill -9 [PID]`. (Mindenki csak a saját folyamatát lőheti ki, másét csak root jogosultsággal).

## 7. Időzítés (at, cron, sleep)

- **at:** Egyszeri, jövőbeli végrehajtás. Megadható neki, hogy egy parancsot fusson le egy megadott időpontban (akkor is, ha nem ülünk a gép előtt).
- **cron:** Rendszeres, ismétlődő feladatok (background jobs) időzítője.
  - A beállításokat a **cron table** tartalmazza, amelyet a `crontab -e` paranccsal lehet szerkeszteni.
  - Minden sor 6 mezőből áll: **Perc Óra Nap(Hónap) Hónap Nap(Hét) Parancs**.
- **sleep:** Felfüggeszti (szünetelteti) a következő parancs végrehajtását egy adott ideig.
  - **Szintaxis:** `sleep [szám][s/m/h/d]`. Például: `sleep 10m` (10 perces szünetet tart). Gyakran használják szkriptekben, például ha várni kell egy lemez felcsatolására a mentés megkezdése előtt. (Különbség: a `sleep` várakoztatja a végrehajtást, az `at` egy konkrét jövőbeli időpontban indítja el).


  # A ps parancs extra opciói és kimeneti jelölései

- **Kernel folyamatok jelölése:** Ha a `ps -ef` kimenetében egy folyamat neve szögletes zárójelek között van (pl. `[kworker]`), az azt jelenti, hogy az egy közvetlenül a kernelben futó háttérfolyamat (nem a felhasználó indította), és általában nagyon magas prioritással (pl. `-20`) fut.
- **`ps -u [felhasználónév]`:** Csak egy adott felhasználó folyamatait listázza ki.
- **`ps -eF`** (Nagy F betűvel): Még egy lépéssel tovább megy, mint a `-ef`, és minden egyes szálról (thread) külön sort jelenít meg.
- **`ps axo [attribútumok]`:** Ezzel teljesen testreszabható, hogy a `ps` pontosan milyen oszlopokat (attribútumokat) mutasson, ha saját riportot akarunk készíteni.

## Folyamatok és szálak (Threads) jelölései

- A `pstree` parancs kimenetében a rendszer a folyamatokhoz tartozó szálakat (threads) kapcsos zárójelek `{}` között jeleníti meg.
- A `jobs -l` parancs annyiban tud többet a sima `jobs`-nál, hogy a háttérben futó feladatok mellé kiírja a pontos **PID**-t (Process ID) is.

## A top parancs 3. sorának (CPU) pontos dekódolása

A `top` kimenetében a CPU sor nagyon rövid rövidítéseket használ, amelyeket a videó pontosan megmagyaráz:

- **us (User):** Felhasználói folyamatokra fordított idő.
- **sy (System/Kernel):** A kernel által használt idő.
- **ni (Nice):** Alacsonyabb prioritású (nice) folyamatokra fordított idő.
- **id (Idle):** Üresjárat (ha a Load average magas, ennek az értéknek alacsonynak kell lennie).
- **wa (Wait):** I/O (pl. merevlemez) műveletekre való várakozással töltött idő.
- **hi (Hardware Interrupts):** Hardveres megszakítások.
- **si (Software Interrupts):** Szoftveres megszakítások.
- **st (Steal time):** Virtuális gépeknél (VM) mutatja, hogy a hypervisor mennyi időt "lopott el" más rendszerek számára.

## Real-time (Valós idejű) prioritás

A videó tesz egy fontos elméleti megkülönböztetést: a Linuxban adhatunk **"real-time"** (extrém magas) prioritást egy folyamatnak (pl. bejövő adatok rögzítése), de ez **nem** keverendő össze a **"Hard Real-Time"** fogalmával, ami egy sokkal szigorúbb, garantált időablakon belüli végrehajtást jelent (ez utóbbit a hagyományos Linux kernelek nem tudják alapból).

## Grafikus Rendszerfigyelő (Gnome System Monitor) extra fülei

A **Processes** (Folyamatok) fül mellett a program tartalmaz egy **Resources** (Erőforrások) fület, amely folyamatosan rajzolt grafikonokon mutatja a terhelést, valamint egy **File Systems** (Fájlrendszerek) fület, ahol egy kattintással ellenőrizhető a csatolt partíciók telítettsége.

# 11. fejezet (Eredetileg: Chapter 10) – Fájlműveletek és Architektúra

A Linux (és minden Unix-szerű rendszer) egyik legfőbb alapelve, hogy **"Minden egy fájl"**. Legyen szó egyszerű szöveges dokumentumokról, vagy olyan hardvereszközökről, mint a hangkártya, a nyomtató vagy a merevlemez, a rendszer ugyanazokon az I/O (Input/Output) olvasási és írási műveleteken keresztül kommunikál velük. Emiatt a szövegszerkesztők kiemelten fontos szerepet kapnak a rendszer üzemeltetésében.

## 1. Fájlrendszer Architektúra és Csatolás (Mountolás)

A fájlrendszer hierarchiája egy fejjel lefelé fordított fához hasonlít, amelynek a törzse a **Gyökérkönyvtár** (Root directory), melynek jele a perjel `/`.

> **Fontos különbségtétel:** A gyökérkönyvtár (`/`) **nem** azonos a root felhasználóval, sem annak saját mappájával (`/root`).

### Fájlrendszer típusok és particionálás

A Linux számos natív fájlrendszert támogat (`ext3`, `ext4`, `btrfs`), és képes más operációs rendszerek fájlrendszereit is kezelni (Windows NTFS, FAT, Mac OS).

A merevlemezt különböző logikai részekre, **partíciókra** bontják. Az elkülönítésnek (pl. külön partíció a rendszerfájloknak `/` és a felhasználói adatoknak `/home`) biztonsági okai vannak: ha egy partíció (például a naplófájlokat gyűjtő `/var`) megtelik és elfogy rajta a hely, az nem omlasztja össze az egész operációs rendszert. A grafikus particionálásra jó példa a videóban látott **GParted** segédprogram.

### Mountolás (Csatolás)

Ahhoz, hogy egy fájlrendszert (vagy akár egy pendrive-ot) használni tudjunk, be kell csatolnunk a fa-struktúrába, egy **csatolási pontra** (Mount point).

- A csatolási pont általában egy üres mappa. Ha olyan mappába csatolunk valamit, amiben már vannak fájlok, az eredeti fájlok láthatatlanná/elérhetetlenné válnak, amíg a fájlrendszert le nem választjuk.
- **Felcsatolás:** `sudo mount /dev/sda5 /home` (becsatolja az `sda5` nevű partíciót a `/home` mappába).
- **Leválasztás:** `sudo umount /home` (Figyelem, a parancs `umount`, nem `unmount`!).
- **Automatikus csatolás:** Hogy a rendszer minden induláskor felcsatolja a lemezeket, az `/etc/fstab` (File System Table) konfigurációs fájlt kell szerkeszteni.
- **Ellenőrzés:** A `df -Th` parancs megmutatja a csatolt rendszereket, a típusukat (`ext4`, `tmpfs` stb.) és a szabad helyet.

## 2. Hálózati Fájlrendszerek (NFS és CIFS)

A hálózaton keresztül megosztott mappák többféle technológiával csatolhatók fel a helyi gépre. A vállalatoknál gyakran egy központi szerveren vannak a `/home` mappák, így a felhasználó bármelyik gépre is lép be, a saját fájljait és beállításait kapja meg.

- **NFS (Network File System):** Unix/Linux rendszerek közötti standard fájlmegosztó (eredetileg a Sun Microsystems fejlesztette).
- **CIFS (Samba):** Microsoft Windows alapú hálózatokkal való megosztáshoz használatos.

## 3. A Fájlrendszer Hierarchia (Legfontosabb könyvtárak)

- **`/home`:** A normál felhasználók saját könyvtárai.
- **`/root`:** A szuperfelhasználó (rendszergazda) saját könyvtára.
- **`/bin` és `/sbin`:** Az alapvető, rendszerindításhoz és alapműködéshez szükséges futtatható programok (binaries) helye (pl. `ls`, `cat`, `rm`). Az `/sbin` a rendszergazdai (system binaries) parancsokat tartalmazza. (Modern rendszereken ezek gyakran csak szimbolikus linkek a `/usr/bin` mappákra).
- **`/usr`:** A nem létfontosságú programok, dokumentációk, felhasználói szoftverek helye.
- **`/dev` (Devices):** A hardvereszközök reprezentációja. Itt találhatók a lemezek (pl. `/dev/sda1`), vagy olyan speciális eszközök, mint a `/dev/random` (véletlenszám-generátor).
- **`/var` (Variable):** A folyamatosan változó méretű fájlok otthona. Ide kerülnek a rendszernaplók (`/var/log`), nyomtatási sorok (`/var/spool`), vagy a webszerverek adatai (`/var/www`).
- **`/etc`:** Rendszerszintű konfigurációs fájlok tárolója. Futtatható (bináris) programokat **nem** tartalmaz. (Itt van pl. a DNS feloldást végző `/etc/resolv.conf`, vagy a felhasználói fiókokat kezelő `passwd` és `shadow` fájlok).
- **`/boot`:** A rendszerindításhoz szükséges fájlok helye:
  - `vmlinuz`: Maga a tömörített Linux kernel.
  - `initramfs` (vagy `initrd`): Az ideiglenes gyökérfájlrendszer.
  - A GRUB bootloader konfigurációja (`/boot/grub2`).
- **`/lib` (vagy `/lib64`):** Megosztott programkönyvtárak (Shared libraries), amelyekre a `/bin`-ben lévő programoknak szükségük van a működéshez. A kernel modulok (driverek) a `/lib/modules/` alatt találhatók.
- **`/run` és `/media`:** Modern rendszereken a cserélhető meghajtók (pendrive-ok) ide csatolódnak be automatikusan (pl. `/run/media/felhasznalo/PENDRAJV_NEVE`).
- **`/mnt`:** Hagyományosan a kézzel (ideiglenesen) felcsatolt partíciók helye.

### A Pseudo Fájlrendszer (`/proc`)

Ez a könyvtár **nem létezik a merevlemezen**, csak a memóriában (RAM-ban) jön létre futásidőben. A kernelről és a rendszer hardveres állapotáról nyújt dinamikus információt.

- Tartalmazza minden futó folyamat (PID) saját virtuális mappáját.
- A `cat /proc/cpuinfo` vagy `cat /proc/meminfo` paranccsal pontos képet kaphatunk a hardverről a merevlemez használata nélkül.

## 4. Fájlok Összehasonlítása és Foltozása (diff és patch)

- **`diff`:** Két szöveges fájl közötti különbséget mutatja meg sorról sorra. Bináris fájlok összehasonlítására a `cmp` parancs való.
- **`diff3`:** Három fájlt tud egyszerre összehasonlítani (egy közös alapfájlt véve referenciának, amiből a másik kettő származik).
- **`patch`:** Szoftverkódok frissítésekor nem az egész fájlt küldik el újra, hanem csak a `diff` által generált "különbségfájlt" (Deltát), ez az ún. **Patch**. A `patch` parancs ezt ráilleszti a régi fájlra, módosítva azt. Sokkal kisebb méretű, ezért nagyon hatékony.

## 5. Fájltípusok megállapítása (`file` parancs)

A Windowssal ellentétben a Linux **nem** hagyatkozik a fájlkiterjesztésekre (pl. `.exe` vagy `.txt`). A kiterjesztés itt csak a felhasználó kényelmét szolgálja. Ha tudni akarjuk, mi van egy fájlban, a `file` parancsot kell használni (pl. `file dokumentum`). Ez megvizsgálja a fájl belső struktúráját (a tartalmát), és megmondja, hogy az egy szöveges dokumentum, kép, futtatható program, vagy szkript.

## 6. Adatok mentése és szinkronizálása (`rsync`)

A hagyományos másoló paranccsal (`cp`) szemben az `rsync` egy sokkal intelligensebb és robusztusabb eszköz a fájlok és könyvtárak szinkronizálására és biztonsági mentésére.

- **Hatékonyság:** Az `rsync` ellenőrzi, hogy a célhelyen létezik-e már a fájl. Csak azokat a fájlokat, vagy a fájloknak csak azokat a részeit másolja át, amelyek ténylegesen megváltoztak (Delta transfer), így gigabájtokat és rengeteg időt spórol.
- **Hálózati másolás:** Szemben a `cp`-vel, az `rsync` képes egy másik hálózati gépre (vagy gépről) is másolni (`rsync forras.txt felhasznalo@gep:/cel/mappa`).
- **Gyakori kapcsolók:** `rsync -avz` (`a` = archív mód, minden jogosultságot megőriz; `v` = verbose, bőbeszédű kimenet; `z` = tömöríti az adatokat hálózati küldés közben).
- **Biztonsági funkció:** Mivel egy rosszul megadott paraméter (`/` jelek megléte vagy hiánya a mappa neve után) nagy károkat okozhat, mindig érdemes a `--dry-run` opcióval futtatni élesben indulás előtt. Ez csak megmutatja, mi történne, de nem módosít semmit.

## 7. Adattömörítés

A lemezterület megtakarítása és a gyorsabb hálózati másolás érdekében a Linux többféle tömörítési eljárást támogat.

A videó az alábbiakat említi (a leggyengébbtől és leggyorsabbtól a leghatékonyabb felé haladva):

- **`gzip`** (a klasszikus, kiterjesztése: `.gz`)
- **`bzip2`** (jobb tömörítési arány, de lassabb, kiterjesztése: `.bz2`)
- **`xz`** (nagyon magas tömörítési arány, lassú, kiterjesztése: `.xz`)

> Az egyre jobb tömörítési hatékonyság egyre több CPU időt igényel a becsomagoláshoz, a kicsomagolás ideje viszont szinte mindegyiknél megegyezik.

# 12. fejezet: Szövegszerkesztők (Text Editors)

A Linux rendszergazdák számára a szövegszerkesztők használata mindennapos feladat (pl. konfigurációs fájlok módosítása, bash szkriptek írása, kódolás).

> **Fontos szabály:** A hagyományos irodai szövegszerkesztők (mint a Microsoft Word vagy a LibreOffice Writer) **nem** alkalmasak ezekre a feladatokra, mert bár a felhasználó nem látja, formázási kódokat adnak a dokumentumokhoz, ami a konfigurációs fájlokat és a szkripteket azonnal működésképtelenné teszi.

## 1. Fájl létrehozása szerkesztő nélkül

Ha csak egy nagyon rövid (egy-két soros) fájlt kell létrehozni, az közvetlenül a parancssorból is megoldható anélkül, hogy megnyitnánk egy dedikált szerkesztőprogramot. Ez szkriptelésnél különösen hasznos.

- **Az `echo` parancs használatával:**
  - `echo "elso sor" > file.txt` (Létrehozza a fájlt. Ha már létezik, felülírja a tartalmát).
  - `echo "masodik sor" >> file.txt` (A dupla nyíl `>>` hozzáfűzi az új sort a fájl legvégéhez, a régi adatok megőrzésével).
- **A `cat` és az átirányítás használatával:**
  - A `cat > file.txt` paranccsal a terminálba ezután begépelt szöveg (akár több sor is) közvetlenül a fájlba íródik. A folyamat a `Ctrl+D` lenyomásával állítható le és menthető el.

## 2. Egyszerű szövegszerkesztők

Kezdők számára a legkönnyebben megtanulható eszközök, amelyek nem igényelnek előképzettséget.

- **Nano:** Egy rendkívül egyszerű, tiszta parancssoros (szöveges) szerkesztő.
  - **Indítás:** `nano fajlnev.txt`
  - A képernyő alján mindig látható egy súgó a billentyűkombinációkkal. (A menüben látható `^` jel a Ctrl billentyűt jelenti).
  - **Legfontosabb parancsok:** `Ctrl+O` (Mentés/Write Out), `Ctrl+X` (Kilépés), `Ctrl+R` (Másik fájl tartalmának beolvasása/beszúrása), `Ctrl+C` (Kurzor pozíciójának megjelenítése).
- **Gedit (és Kate/Kwrite):** Ezek grafikus felületű szerkesztők (A Gedit a GNOME, a Kate a KDE asztali környezet alapértelmezettje). Vizuálisan a Windows-os Jegyzettömbre (Notepad) hasonlítanak, de sokkal fejlettebbek (pl. kódkiemelés, pluginok). **Hátrányuk**, hogy kizárólag grafikus felületen futnak, így egy távoli szerveren parancssoros elérés (SSH) esetén nem használhatók.

## 3. Haladó szövegszerkesztők (Vi/Vim és Emacs)

A Unix/Linux világ két legrégebbi és legrobosztusabb eszköze. Bár a betanulási görbéjük meredek (nehezek a kezdőknek), a profik kizárólag ezeket használják, mert egerészés nélkül, a billentyűzetről vezérelve villámgyors munkát tesznek lehetővé.

### A) Vi / Vim (Vi IMproved)

Minden egyes Unix és Linux rendszeren (még a Mac OS-en is) gyárilag jelen van. Ha egy távoli szerveren semmilyen más eszköz nincs, a Vi biztosan működni fog. A rendszerben általában a `vim` (a modernizált változat) van telepítve, amelyet gyakran ráaliasoltak a `vi` parancsra.

A Vi **nem használ egeret**, mindent a billentyűzetről kell vezérelni. Legfőbb jellemzője a **három különböző mód** megléte (Kezdők gyakori hibája, hogy elvesztik a fonalat, épp melyik módban vannak):

1. **Parancs mód (Command mode):** Ez az alapértelmezett mód a fájl megnyitásakor. Itt a billentyűleütések parancsokként értelmeződnek, nem gépelt szövegként.
   - **Navigáció** (Nyilak helyett): `h` (balra), `j` (le), `k` (fel), `l` (jobbra). `w` (ugrás a következő szó elejére), `$` (ugrás a sor végére).
   - **Keresés:** `/keresett_szo` (előre keresés a dokumentumban), `?keresett_szo` (hátra keresés). Ugrás a találatok között: `n` (következő), `N` (előző találat).
   - **Szerkesztés parancs módban:** `x` (egy karakter törlése), `dd` (a teljes sor törlése/kivágása), `yy` (egy sor másolása), `p` (a kivágott/másolt rész beillesztése).

2. **Beszúrás mód (Insert mode):** Ebben a módban írhatjuk ténylegesen a szöveget a képernyőre.
   - **Belépés** a parancs módból: Az `i` betűvel (beszúrás a kurzor előtt), vagy az `a` betűvel (hozzáfűzés a kurzor után).
   - **Kilépés:** Az `Esc` gombbal mindig visszajutunk a Parancs módba. Ezt gyakran kell használni.

3. **Sor/Parancssor mód (Line mode):** Parancs módból a kettőspont `:` lenyomásával érhető el. Ide írhatók be a fájlműveleti parancsok (Enterrel lezárva).
   - `:w` (Mentés/Write).
   - `:q` (Kilépés/Quit).
   - `:wq` (Mentés és kilépés).
   - `:q!` (Kilépés mentés nélkül, erőszakosan, ha elrontottunk valamit).
   - `:! [parancs]` (Egy külső Linux parancs futtatása a szerkesztőből való kilépés nélkül. Például a `:! wc %` parancs megszámolja a szavakat a jelenlegi – `%` jellel hivatkozott – fájlban).

> **Oktató:** A terminálból a `vimtutor` paranccsal egy kiváló, 7 leckés beépített oktatóprogram indul.

### B) Emacs

Az Emacs a Vi legnagyobb történelmi riválisa. Nem használ különböző módokat (mint a Vi), ehelyett egyetlen módban fut, a funkciókat pedig a kombinációs gombok (Ctrl, Alt/Meta, Esc) folyamatos lenyomásával és betűkkel lehet elérni. Hatalmas tudású, rendkívül testreszabható program (lehet belőle e-mailezni vagy kódokat is futtatni). Mivel rendkívül sokat kell a Ctrl gombot használni (ami a normál billentyűzeteken a bal alsó sarokban, "kisujjtörő" helyen van), a profi Emacs felhasználók a gépükön át szokták állítani a ritkán használt Caps Lock gombot Ctrl funkciójúra.

- **Oktatóprogram:** Az Emacson belül a `Ctrl+h` majd a `t` lenyomásával indul a beépített Tutorial.
- **Gyakori parancsok:**
  - `Ctrl+x Ctrl+c` (Mentés és Kilépés).
  - `Ctrl+x Ctrl+s` (Sima mentés).
  - `Ctrl+x Ctrl+w` (Mentés másként / Write as).
  - `Ctrl+s` (Keresés).
  - `Ctrl+k` (Sor törlése a kurzortól a sor végéig).
  - `Ctrl+w` (Kijelölt rész kivágása), `Ctrl+y` (Kivágott rész beillesztése / Yank).
  - `Ctrl+x 2` (Az ablak vízszintes felosztása, így két fájl is látható és szerkeszthető egyszerre. A két ablakrész között a `Ctrl+x o` paranccsal lehet ugrálni). A `Ctrl+x 1` visszavált az egy ablakos nézetre.

  # 13. fejezet: Felhasználói környezet (User Environment)

A Linux, mint igazi **többfelhasználós** operációs rendszer, lehetővé teszi, hogy minden felhasználó testreszabja a saját munkakörnyezetét (környezeti változók, aliasok, jogosultságok), miközben a rendszergazda (root) szigorúan kontrollálja a rendszer egészét.

## 1. Felhasználók és Csoportok azonosítása

- A rendszerben minden felhasználót egy egyedi szám, a **UID** (User ID) azonosít (a normál felhasználók a modern rendszereken az **1000-es** UID-től indulnak).
- A jogosultságok kezelésének megkönnyítése érdekében a felhasználókat **csoportokba** foglalják (**GID** – Group ID). Egy felhasználónak van egy "elsődleges" (primary) csoportja (ami alapértelmezetten a saját nevével és UID-jével egyezik meg), és tartozhat további kiegészítő csoportokba is.
- **Azonosító parancsok:**
  - `whoami` (Csak kiírja a jelenlegi felhasználónevet).
  - `who` (Kilistázza, hogy pontosan kik vannak bejelentkezve a gépre. A `who -a` részletesebb információt ad).
  - `id` (Részletesen megmutatja az aktuális, vagy egy argumentumként megadott felhasználó UID és GID azonosítóit).

## 2. Bash Indítási fájlok (Startup files)

Amikor a parancssor (Bash shell) elindul, konfigurációs fájlokat olvas be a rendszer beállításához (pl. milyen legyen a prompt, hol keresse a futtatható programokat). Két szint létezik:

- **Globális (Rendszerszintű):** Az `/etc/profile` fájlt minden teljes bejelentkezéskor (login shell) beolvassa a rendszer (ez minden felhasználóra érvényes).
- **Saját (Felhasználói):** A saját könyvtárban lévő rejtett fájlok, a következő sorrendben: `~/.bash_profile`, `~/.bash_login`, `~/.profile`. A rendszer azt értékeli ki, amelyiket legelőször megtalálja a három közül, a többit figyelmen kívül hagyja.
- **`~/.bashrc` szerepe:** Amikor asztali környezetből egyszerűen nyitsz egy új terminálablakot, a rendszer nem végez teljes bejelentkezést, hanem csak a `~/.bashrc` fájlt olvassa be. Ezért, ha a mindennapi munkához saját parancsokat, promptot vagy rövidítéseket (aliasokat) akarsz készíteni, azokat ide érdemes beírni. (Sok disztribúcióban a login fájlok eleve úgy vannak megírva, hogy betöltsék a `.bashrc`-t).

## 3. Aliasok (Parancs rövidítések)

Az `alias` paranccsal hosszú, bonyolult parancsokat lehet egy rövid, egyedi névre átkeresztelni a gyorsabb gépelés érdekében. Ezeket is a `~/.bashrc`-be érdemes menteni.

- **Létrehozás:** `alias update='sudo dnf update -y'` (Figyelem: az `=` jel körül **nem** lehet szóköz, az egész parancsot pedig idézőjelbe kell tenni!).
- **Lekérdezés:** Sima `alias` beírásával listázza az aktív aliasokat.
- **Törlés:** `unalias update`

## 4. Környezeti Változók (Environment Variables)

Ezek olyan, a memóriában tárolt nagybetűs "karakterláncok", amelyeket a programok és a shell folyamatosan használnak az alapvető működéshez.

- **Megtekintés:** Az `env`, `set` vagy `export` parancsokkal kilistázható az összes beállított változó.
- **Legfontosabb változók:**
  - `$HOME`: A felhasználó saját könyvtárának útvonala. A `cd` parancs önmagában is ide visz vissza, és a tilde `~` karakter is ezt helyettesíti (pl. `cd ~`).
  - `$PATH`: Kettősponttal elválasztott mappák listája. Amikor beírsz egy parancsot, a Linux ezekben a mappákban (pl. `/bin`, `/usr/bin`) keresi sorban a futtatható programot. Ha kiegészítenéd a saját mappáddal: `export PATH=$PATH:$HOME/bin`.
  - `$SHELL`: Az aktuálisan használt parancsértelmező útvonala (pl. `/bin/bash`).
  - `$PS1`: A terminál prompt kinézetét határozza meg (a felirat a kurzor előtt). Olyan karaktereket tartalmazhat, mint `\u` (user), `\h` (host), `\w` (working directory).
- **Exportálás:** Ahhoz, hogy egy újonnan létrehozott változót ne csak a jelenlegi shell, hanem az abból indított alfolyamatok (child processes) és szkriptek is lássanak, használni kell az `export VALTOZO_NEV=ertek` parancsot.

## 5. Parancselőzmények (History) és Gyorsbillentyűk

A Bash folyamatosan elmenti a terminálba beírt parancsokat a `~/.bash_history` nevű rejtett fájlba. (Ezek viselkedése is szabályozható változókkal, pl. a `$HISTSIZE` határozza meg az eltárolt sorok számát, alapértelmezetten 500-at).

- A `history` parancs kilistázza a korábban kiadott parancsokat. A **Fel/Le nyilakkal** simán léptethetünk köztük.
- **`Ctrl + R` (Reverse Search):** Intelligens, visszafelé keresés az előzményekben. Lenyomása után, ahogy elkezded gépelni a szót (pl. `nano`), azonnal felkínálja a legutóbbi pontosan illeszkedő parancsot.
- **Hasznos billentyűkombinációk (Shortcuts) a terminálban:**
  - `Ctrl + A` (Ugrás a sor legelejére).
  - `Ctrl + E` (Ugrás a sor legvégére).
  - `Ctrl + K` (A kurzortól a sor végéig mindent kitöröl/kivág).
  - `Ctrl + U` (A sor elejétől a kurzorig mindent kitöröl).
  - `Ctrl + L` (Képernyő letörlése, megegyezik a `clear` paranccsal).
  - `Ctrl + C` (Félbehagyja az írást, vagy megszakítja a futó parancsot).

## 6. Jogosultságok, Tulajdonjogok és Sudo

Linux alatt minden fájlnak van egy **Tulajdonosa** (Owner/User) és egy **Csoportja** (Group). Emellett háromféle alapjog létezik: **Olvasás** (read), **Írás** (write), **Futtatás** (execute). A jogosultságokat a rendszer 3 entitásra bontja: a Tulajdonosra (`u`), a Csoportra (`g`), és Mindenki másra a gépen (`o` – others/world).

- **Sudo vs. Su:** Biztonsági okokból **soha** nem szabad bejelentkezve maradni, és sima felhasználóként dolgozni a teljes jogú rendszergazda (root) fiókkal. Ehelyett a normál felhasználónak (aki benne van a sudoers fájlban) a `sudo [parancs]` formátumot kell használnia, ami az aktuális egyetlen parancsra átmeneti rendszergazdai jogot ad a saját jelszava beírása után. (A `su` vagy `su -` parancs teljesen átváltana a root környezetre, ami tévedésekhez és a rendszer letörléséhez vezethet).
- **Felhasználók és csoportok kezelése** (Kizárólag root/sudo joggal!):
  - `useradd -m [nev]` (Felhasználó létrehozása. Az `-m` opció kötelező ahhoz, hogy a `/home` mappában saját könyvtára is létrejöjjön a `/etc/skel` sablonjai alapján).
  - `passwd [nev]` (A felhasználó jelszavának beállítása).
  - `userdel -r [nev]` (Felhasználó törlése. Az `-r` opcióval a saját mappája és fájljai is azonnal törlődnek).
  - `groupadd`, `groupdel` (Csoportok létrehozása és törlése).
  - `usermod -aG [csoport] [nev]` (Felhasználó hozzáadása egy kiegészítő csoporthoz. A `-aG` opció (append) **nagyon fontos**, mert e nélkül a parancs az összes korábbi csoportjából kiléptetné a felhasználót!).
- **Tulajdonos váltás (`chown` és `chgrp`):**
  - A `sudo chown uj_felhasznalo fajlnev` megváltoztatja a fájl/mappa tulajdonosát.
  - A `sudo chown uj_felhasznalo:uj_csoport fajlnev` mindkettőt egyszerre váltja.
  - A `chgrp uj_csoport fajlnev` kifejezetten csak a csoportot cseréli.
- **Jogosultságok módosítása (`chmod`):**
  - **Szöveges mód:** pl. `chmod o-w fajl` (Elveszi (mínusz) mindenki mástól az írási jogot). Vagy `chmod u+x fajl` (Futtatási jogot ad a tulajdonosnak).
  - **Numerikus (Oktális) mód:** A leggyakoribb, gyors számszerűsített módszer. Kiszámolása: **Olvasás = 4, Írás = 2, Futtatás = 1**. Ezeket az értékeket össze kell adni külön a Tulajdonos, a Csoport, és a Többiek esetében.
    - (Példák: a **7** = 4+2+1, azaz teljes `rwx` jog; a **6** = 4+2, azaz írás/olvasás `rw-`; az **5** = 4+1, azaz olvasás és futtatás `r-x`).
  - **Gyakorlati használat:** `chmod 755 fajl` (A tulajdonosnak 7 = `rwx` joga, a csoportnak 5 = `r-x` joga, és mindenki másnak is 5 = `r-x` joga lesz. Teljesen letiltani a többieket így lehet: `chmod 700 fajl`).

  # 14. fejezet (Eredetileg: Chapter 13) – Szöveges adatok manipulálása

Minden Linux rendszergazda, fejlesztő és felhasználó számára elengedhetetlen a szöveges fájlok hatékony parancssori kezelése. Ez sokkal gyorsabb, mint grafikus felületen matatni, és a legfőbb előnye, hogy kiválóan **szkriptelhető** (automatizálható).

## 1. Fájlok tartalmának olvasása és egyesítése

- **`cat` (Concatenate):** Fájlok kiíratása a terminálra, vagy fájlok összefűzése.
  - **Olvasás:** `cat file.txt` (Hosszabb fájloknál a szöveg azonnal kigördül a képernyőről).
  - **Összefűzés:** `cat file1.txt file2.txt > file3.txt` (A két fájl tartalmát egyesíti a harmadikban).
- **`tac`:** A `cat` parancs pontosan fordítva. Ugyanúgy írja ki a fájlt, de az utolsó sorral kezdi, és az elsővel fejezi be. (Figyelem: sorok szintjén fordít, a szavakat nem írja visszafelé!).
- **`less`:** Hatalmas (akár sok gigabájtos log) fájlok megnyitására való. Gyors, mert nem tölti be az egész fájlt a memóriába, hanem oldalanként (Page by Page) jeleníti meg. Felfelé és lefelé is lehet benne görgetni a nyilakkal, `q`-val lehet kilépni belőle (a man oldalak is ezt használják a háttérben).
- **`head` és `tail`:** Egy fájl legelső vagy legutolsó sorait mutatja meg.
  - Alapértelmezésben 10 sort írnak ki. (pl. `head -15 file.txt` vagy `tail -n 15 file.txt` módosítja ezt a számot).
  - **`tail -f file.log` (Follow):** Ez az egyik leghasznosabb rendszergazdai parancs. Folyamatosan "figyeli" a fájlt, és valós időben írja ki a terminálra, ahogy az új sorok (pl. hibaüzenetek) belekerülnek a logba.

## 2. Szövegszűrés és módosítás (Stream eszközök)

A `sed` és az `awk` olyan történelmi Unix eszközök, amelyek sokkal kevesebb memóriát használnak és sokkal gyorsabbak, mint egy komplex Python vagy Perl szkript megírása ugyanarra a feladatra. Rendszerindításkor vagy vészhelyzetben felbecsülhetetlenek.

- **`sed` (Stream Editor):** Szövegfolyam-szerkesztő. Általában cserék (substitutions) elvégzésére használják úgy, hogy magát a fájlt nem kell megnyitni a szövegszerkesztőben.
  - **Szintaxis:** `sed -e 's/eredeti/uj/g' file.txt > out.txt`
  - A fenti példában az `s` jelöli a cserét (substitute). Kicseréli az *eredeti* szót az *uj* szóra. A `g` a végén (global) azt jelenti, hogy egy soron belül **minden** előfordulást cseréljen, ne csak a legelsőt.
  - Rendkívül veszélyes funkciója az *in-place* szerkesztés, ami beavatkozás nélkül módosítja és felülírja az eredeti fájlt, ezért tesztelésnél a kimenetet mindig inkább egy új fájlba irányítjuk (`> out.txt`).
- **`awk`:** Nem is parancs, hanem egy önálló programozási nyelv (a 70-es évekből) adatkinyerésre és jelentések készítésére. Akkor zseniális, ha a fájl táblázatszerűen, oszlopokból (mezőkből) áll.
  - **Példa:** A `ps -ef | awk '{print $1, $2}'` parancs csak a folyamatok futtatóját (1. oszlop) és a PID-t (2. oszlop) írja ki a nagyon széles `ps` kimenetből.
  - Egyéni elválasztó is megadható: `awk -F: '{print $1}' /etc/passwd` (mivel a jelszófájlban a kettőspont az oszlopelválasztó, ez a parancs tisztán kilistázza a rendszer összes felhasználójának nevét).

## 3. Keresés és Reguláris Kifejezések (Regex)

A **Reguláris Kifejezések** (Regular Expressions) olyan speciális karakterláncok, amelyekkel nem konkrét szavakra, hanem logikai mintákra kereshetünk. (Pl. "keress egy olyan sort, ami S betűvel kezdődik, benne van a *hiba* szó, és számmal végződik"). Ezt a szabványos rendszert használja a `grep`, a `sed`, az `awk`, a `find` és a `vi` is.

- **Legfontosabb Regex metakarakterek:**
  - `^` : A sor elejét jelöli (pl. `^hello` = pontosan "hello"-val kezdődő sor).
  - `$` : A sor végét jelöli (pl. `end$` = "end"-re végződő sor).
  - `.` (Pont): Pontosan egy darab tetszőleges karakter.
  - `*` (Csillag): Az előtte álló karakter 0 vagy többszöri ismétlődése.
  - `[abc]` : A szögletes zárójelen belüli karakterek bármelyike (a, b vagy c). A `[A-Z]` az összes nagybetűt jelenti.
- **`grep`:** Szöveges fájlokban vagy a futó parancsok kimenetében keres Regex minták alapján. (Gyakori opciók: `grep -i` a kis/nagybetű független keresés, a `grep -v` pedig az inverz: azokat a sorokat mutatja meg, amikben **nincs** benne a keresett szó).
- **`strings`:** Ez a parancs arra jó, hogy ember által olvasható szavakat (printable strings) keressen és szedjen ki lefordított, bináris (futtatható) fájlokból.

## 4. Fájlok rendezése, tagolása és egyesítése

- **`sort`:** Ábécé sorrendbe (vagy a `-r` opcióval visszafelé) rendezi a fájl sorait. Képes megadott oszlopok alapján is rendezni.
- **`uniq`:** Törli a duplikált, ismétlődő sorokat egy fájlból. **Kritikus szabály:** a `uniq` csak az egymás után közvetlenül következő ismétlődéseket veszi észre, ezért a fájlt először mindig ABC sorrendbe kell rendezni! (pl. `sort file.txt | uniq`).
  - A `sort -u` parancs önmagában elvégzi ezt a két lépést.
  - A `uniq -c` (count) opció megszámolja, hányszor ismétlődött egy sor.
- **`cut`:** Kiszűr bizonyos oszlopokat egy fájlból (a default elválasztó a Tab). Példa: `ls -l | cut -d " " -f 3` (A kimenetet a szóközök `" "` alapján vágja fel, és csak a 3. oszlopot írja ki).
- **`paste` és `join`:** Két különálló fájl tartalmának vízszintes (oszloponkénti) egyesítésére valók.
  - A `paste` egyszerűen a két fájl oszlopait fűzi össze egymás mellé, mintha egy Excel táblát ragasztanál össze, sorról sorra.
  - A `join` intelligensebb: két fájlt egyesít úgy, hogy keres bennük egy közös mezőt (pl. egy telefonszámot mindkét fájlban), és az alapján társítja (relációs adatbázis módjára) az adatokat.
- **`split`:** Felvág egy hatalmas fájlt kisebb, egyenlő (alapból 1000 soros) darabokra. Az új fájlok nevéhez általában egy betűkből álló előtagot (prefix) ragaszt (pl. ha egy szótárt vágunk fel, a darabok `dictionaryxaa`, `dictionaryxab` stb. néven jönnek létre).

## 5. Hasznos apróságok (Karakterek és Kimenetek)

- **`tr` (Translate):** Karakterek halmazának cseréje vagy törlése. Csak a standard bemenetről tud olvasni. (Példa, csupa nagybetűssé alakítás: `cat file.txt | tr 'a-z' 'A-Z'`).
- **`wc` (Word Count):** Megszámolja egy fájl tartalmát. A kimenetben alapból három szám van: a sorok (`-l`), a szavak (`-w`), és a karakterek (`-c`) száma. (Pl. a `ls -l | wc -l` megmondja, hány fájl van egy mappában).
- **`tee` (T-elágazás):** A standard kimenet (stdout) kettéosztása. Az adatokat egyszerre jeleníti meg a monitoron, **ÉS** egyidejűleg el is menti egy megadott fájlba. Példa: `ls -l | tee lista.txt` (így azonnal látjuk a képernyőn a kimenetet, de le is mentettük magunknak).

# 15. fejezet (Eredetileg: Chapter 14) – Hálózati műveletek

Ebben a fejezetben a hálózati alapfogalmakat, az IP-címzést, a névfeloldást és a legfontosabb hálózati diagnosztikai és fájlátviteli parancsokat ismertetik.

## 1. Hálózati alapfogalmak és IP címek

A hálózat számítógépek és eszközök csoportja, amelyek kommunikálnak egymással és erőforrásokat (pl. nyomtatók, adatbázisok) osztanak meg. A kommunikáció **adatcsomagok** (packets) formájában történik. Minden hálózaton lévő eszköznek rendelkeznie kell egy egyedi azonosítóval, az **IP-címmel** (Internet Protocol address).

- **IPv4 vs. IPv6:**
  - **IPv4:** A régebbi, de még mindig domináns szabvány. 32 bites címeket használ, ami "csak" kb. 4,3 milliárd egyedi címet tesz lehetővé (pl. `192.168.1.1`). (Ezt A, B, C, D, E osztályokra, úgynevezett class-okra bontják a hálózat és a hosztok azonosítása alapján).
  - **IPv6:** Az újabb szabvány, amely 128 bites címeket használ (pl. `2001:0db8:85a3::8a2e:0370:7334`), ami gyakorlatilag végtelen számú cím kiosztását teszi lehetővé. Az átállás folyamatos, de a két protokoll kompatibilitási kihívásai miatt lassú.
- **NAT (Network Address Translation):** Mivel az IPv4 címek elfogytak, a NAT technológiát használják. Ez teszi lehetővé, hogy egy egész otthoni vagy vállalati belső hálózat (több tucat belső IP-címmel rendelkező gép) egyetlen, kifelé mutató "publikus" IP-cím mögé bújva kommunikáljon az internettel.
- **DHCP (Dynamic Host Configuration Protocol):** Automatikusan oszt ki és konfigurál IP-címeket a hálózatra csatlakozó gépeknek (így nem kell azokat manuálisan/statikusan beállítani a klienseken).
- **A "Localhost":** A `127.0.0.1` IP-cím egy speciális visszacsatolási (loopback) cím, amely a hálózaton mindig magára az aktuális gépre mutat.

## 2. Névfeloldás (DNS)

A **DNS** (Domain Name System) egy telefonkönyvhöz hasonlóan lefordítja a nehezen megjegyezhető IP-címeket ember által olvasható domain nevekre (pl. a `104.9.58.51` címet lefordítja a `whitehouse.gov`-ra).

- **Keresési sorrend:** A Linux először mindig a lokális `/etc/hosts` fájlt ellenőrzi (ide kézzel is beírhatunk IP – Név párosításokat). Ha itt nem találja a nevet, akkor fordul a hálózati DNS szerverhez (amelynek beállítása a modern rendszereken a `systemd-resolved` szolgáltatással, régebben az `/etc/resolv.conf` fájllal történik).
- **Névfeloldást tesztelő parancsok:**
  - `host linuxfoundation.org` (A legegyszerűbb kimenet).
  - `nslookup linuxfoundation.org`
  - `dig linuxfoundation.org` (Ez adja a legrészletesebb, technikai kimenetet a szerver válaszidejével együtt).

## 3. Hálózati Interfészek és Diagnosztika (Parancsok)

Bár a hálózati kapcsolatok a grafikus Network Manager-rel, vagy a terminálos `nmtui` programmal is beállíthatók, az alacsony szintű diagnosztikához a következő parancsokat használják:

- **`ifconfig`:** A régebbi, de vizuálisan átláthatóbb parancs a hálózati csatolók (hálózati kártyák MAC címe, IP címek) lekérdezésére. Sok modern rendszerből már hiányzik (a `net-tools` csomagban található).
- **`ip` (IP Route2):** A modern, rendkívül erős eszköz, amely leváltotta az `ifconfig`-ot.
  - `ip addr show` (Vagy röviden `ip a`): Megmutatja az IP-címeket és a hálózati kártyákat.
  - `ip -br a` (Brief): Rövidített, a szemnek sokkal átláthatóbb kimenetet ad egyetlen sorban.
  - `ip route show` (Vagy röviden `ip r`): Megmutatja az útválasztási (routing) táblát és az alapértelmezett átjárót (gateway).
- **`ping`:** Teszteli, hogy egy adott gép (IP vagy domain) elérhető-e a hálózaton. ICMP adatcsomagokat küld, és méri a válaszidőt. (A Windows-al ellentétben a Linuxos `ping` a végtelenségig fut; a `ping -c 4` opció beállítja, hogy csak 4 csomagot küldjön, különben a `Ctrl+C`-vel kell megszakítani).
- **`traceroute`:** Megmutatja a csomag által megtett teljes útvonalat (a közbeeső routereket és csomópontokat/hop-okat) a célgépig. Kiváló hálózati akadályok és lassulások lokalizálására.
- **`netstat -r`:** Kilistázza az aktív kapcsolatokat és a routing táblát.
- **`nmap`:** Hálózatfelderítő eszköz, ami (többek között) megmutatja egy hálózaton lévő gép nyitott portjait.

## 4. Webböngészés a terminálból

Előfordul, hogy egy szerveren egyáltalán nincs grafikus felület, vagy nem férünk hozzá, de mégis le kell töltenünk valamit az internetről.

- **Terminálos (szöveges) webböngészők:** `lynx`, `links`, `w3m`. Ezek képek nélkül, kizárólag a weboldalak szövegét és hiperhivatkozásait jelenítik meg a parancssorban.
- **`wget`:** Kiváló eszköz fájlok parancssori letöltésére. Képes megszakadt letöltések folytatására, jelszóval védett fájlok letöltésére, vagy akár egy teljes weboldal struktúrájának (rekurzív) lementésére is. Használat: `wget http://weboldal.com/file.zip`.
- **`curl`:** Hasonló a `wget`-hez, de míg a `wget` alapértelmezetten lementi a letöltést egy fájlba, a `curl` a weboldal forráskódját (vagy a lekért adatot) közvetlenül a terminál standard kimenetére (stdout) írja ki. Ha fájlba akarunk menteni, a `-o` opció kell hozzá: `curl -o mentes.html http://pelda.com`.

## 5. Távoli hozzáférés és Biztonságos Másolás (SSH és SCP)

A Linux szervereket a világ minden táján távolról, a parancssorból üzemeltetik, ehhez erős titkosításra van szükség.

- **`ssh` (Secure Shell):** Kriptográfiailag titkosított protokoll, amely teljes mértékben kiváltotta a régi, jelszavakat nyílt szövegként küldő Telnetet.
  - **Használat:** `ssh felhasznalo@192.168.1.100`. (Bejelentkezés egy távoli gépre).
  - Képes jelszó nélküli (publikus/privát kulcspáron alapuló) gyors bejelentkezésre is.
  - Egyetlen parancs is futtatható a távoli gépen bejelentkezés nélkül (pl. `ssh felhasznalo@szerver "ls -l"`).
- **`scp` (Secure Copy):** Fájlok másolása hálózaton keresztül gépek között, az SSH biztonságos, titkosított csatornáján keresztül. Használata nagyon hasonlít a hagyományos `cp` másolóparancsra, kiegészítve a hálózati elérhetőséggel.
  - **Fájl feltöltése** a helyi gépről a szerverre: `scp helyi_fajl.txt felhasznalo@192.168.1.100:/tavoli/mappa/`
  - **Mappa letöltése** a szerverről a helyi gépre: `scp -r felhasznalo@szerver:/tavoli/mappa/ /helyi/temp/` (A `-r` opció itt is a rekurzív, tehát a teljes mappastruktúra másolását jelenti).
