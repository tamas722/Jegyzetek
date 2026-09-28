# Arch rendszerindítási folyamat (Arch boot process)

Az Arch Linux elindításához (bootolásához) be kell állítani egy Linux-kompatibilis rendszerindítót (boot loadert). A rendszerindító felelős a kernel és az initramfs betöltéséért, mielőtt elindítaná a boot folyamatot. Az eljárás meglehetősen eltérő a BIOS és az UEFI rendszerek esetében.

## Firmware típusok

A firmware a legelső program, amely a rendszer bekapcsolásakor lefut.

> **Tipp:** A BIOS és az UEFI szavakat gyakran a firmware helyett használják. Ne tévessze össze a Linux firmware-rel.

### UEFI

A Unified Extensible Firmware Interface (UEFI) támogatja mind a partíciós tábla, mind a fájlrendszerek olvasását. Az UEFI nem indít semmilyen rendszerindító kódot a Master Boot Recordból (MBR), függetlenül attól, hogy az létezik-e vagy sem; a rendszerindítás ehelyett az NVRAM-ban található boot bejegyzéseken alapul.

Az UEFI specifikáció előírja a FAT12, FAT16 és FAT32 fájlrendszerek támogatását (lásd az UEFI specifikáció 2.11-es verziójának 13.3.1.1-es szakaszát), de bármelyik konform gyártó opcionálisan hozzáadhatja további fájlrendszerek támogatását; például a HFS+ vagy az APFS támogatását egyes Apple firmware-ekben. Az UEFI implementációk az optikai lemezekhez használt ISO 9660-at is támogatják.

Az UEFI EFI alkalmazásokat indít el, pl. rendszerindítókat (boot loaders), rendszerindítás-kezelőket (boot managers), UEFI shell-t stb. Ezek az alkalmazások általában fájlként vannak tárolva az EFI rendszerpartíción (EFI system partition). Minden gyártó a saját fájljait az EFI rendszerpartíción belül, a `/EFI/vendor_name` könyvtárban tárolhatja. Az alkalmazások az NVRAM-hoz adott boot bejegyzés hozzáadásával vagy az UEFI shellből indíthatók.

Az UEFI specifikáció a Kompatibilitást Támogató Modullal (Compatibility Support Module - CSM) támogatja a hagyományos BIOS-os rendszerindítást. Ha a CSM engedélyezve van az UEFI-ben, az UEFI minden meghajtóhoz CSM boot bejegyzéseket generál. Ha egy CSM boot bejegyzést választanak ki indításra, az UEFI CSM megpróbál a meghajtó MBR bootstrap kódjából bootolni.

> **Megjegyzés:** Az Intel fokozatosan megszünteti a CSM támogatását, így a jövőben nem feltétlenül lehet erre a funkcióra támaszkodni.

### BIOS

A BIOS (Basic Input-Output System) az esetek többségében magán az alaplapon lévő flash memóriában tárolódik, és független a rendszer tárhelyétől. Eredetileg az IBM PC-hez hozták létre a hardver inicializálásának és a rendszerindítási folyamatnak a kezelésére, de 2010 óta fokozatosan felváltotta az UEFI, amely nem szenved ugyanezektől a technikai korlátoktól.

## Rendszer-inicializálás

A rendszer bekapcsolásakor lefut a bekapcsolási önteszt (power-on self-test - POST). Lásd még: Modern CPUs have a backstage cast (A modern CPU-knak van egy háttércsapatuk) - Hugo Landau.

### UEFI

A POST után az UEFI inicializálja a rendszerindításhoz szükséges hardvereket (lemez, billentyűzetvezérlők stb.).

A firmware kiolvassa az NVRAM-ban lévő boot bejegyzéseket, hogy meghatározza, melyik EFI alkalmazást indítsa el, és honnan (pl. melyik lemezről és partícióról).

Egy boot bejegyzés lehet egyszerűen csak egy lemez. Ebben az esetben a firmware keres egy EFI rendszerpartíciót az adott lemezen, és megpróbál találni egy EFI alkalmazást a tartalék rendszerindítási útvonalon: `\EFI\BOOT\BOOTx64.EFI` (`BOOTIA32.EFI` az IA32 (32 bites) UEFI-vel rendelkező rendszereken). Így működnek az UEFI-vel bootolható cserélhető adathordozók.

A firmware elindítja az EFI alkalmazást.

- Ez lehet egy boot loader vagy maga az Arch kernel egy EFI boot stub használatával.
- Lehet valamilyen más EFI alkalmazás, például az UEFI shell, vagy egy boot manager, mint a systemd-boot vagy a rEFInd.

Ha a Biztonságos indítás (Secure Boot) engedélyezve van, a boot folyamat aláírás alapján ellenőrzi az EFI bináris hitelességét.

> **Megjegyzés:** Néhány UEFI rendszer csak a tartalék rendszerindítási útvonalról (fallback boot path) tud bootolni.

### Többszörös rendszerindítás (Multibooting)

Mivel minden operációs rendszer vagy gyártó a saját fájljait karbantarthatja az EFI rendszerpartíción belül anélkül, hogy a többit befolyásolná, az UEFI használatával történő multi-booting csupán egy adott operációs rendszer rendszerindítójának megfelelő EFI alkalmazás elindítását jelenti. Ezzel megszűnik az a szükséglet, hogy egy boot loader lánc-betöltési (chain-loading) mechanizmusaira kelljen támaszkodni egy másik operációs rendszer betöltéséhez.

### BIOS

A POST után a BIOS inicializálja a rendszerindításhoz szükséges hardvereket (lemez, billentyűzetvezérlők stb.).

A BIOS elindítja a BIOS lemezsorrendjében szereplő első lemez első 440 bájtját (a Master Boot Record bootstrap kódterületét).

A boot loader MBR boot kódban lévő első szakasza ezután elindítja a második szakasz kódját (ha van) a következők valamelyikéből:

- az MBR utáni következő lemezszektorokból, azaz az ún. MBR utáni résből (post-MBR gap - csak MBR partíciós tábla esetén),
- egy partíció vagy egy partíció nélküli lemez kötet-rendszerindító rekordjából (Volume Boot Record - VBR),
- a GRUB esetében egy GPT partícionált lemezen – egy GRUB-specifikus BIOS boot partícióból (ezt használják a GPT-ben nem létező post-MBR rés helyett).

Elindul a tényleges boot loader.

A boot loader ezután betölt egy operációs rendszert lánc-betöltéssel (chain-loading) vagy az operációs rendszer kernelének közvetlen betöltésével.

## Rendszerindító (Boot loader)

A boot loader egy szoftver, amelyet a firmware (UEFI vagy BIOS) indít el. Feladata a kernel betöltése a kívánt kernel paraméterekkel és az esetleges külső initramfs képfájlokkal.

Egy rendszerindítás-kezelő (boot manager) a boot opciók menüjét jeleníti meg, vagy valamilyen más módot biztosít a boot folyamat szabályozására – azaz csupán más EFI végrehajtható fájlokat futtat.

UEFI esetén magát a kernelt az UEFI közvetlenül is elindíthatja az EFI boot stub segítségével. Egy különálló boot loader vagy boot manager továbbra is használható a kernel paraméterek rendszerindítás előtti szerkesztésére.

A 32 bites IA32 UEFI-vel rendelkező rendszerek olyan boot loadert igényelnek, amely támogatja a vegyes módú rendszerindítást (mixed mode booting).

> **Figyelmeztetés:** Az Arch sikeres elindításához a boot loadernek hozzá kell férnie a kernelhez és az initramfs képfájl(ok)hoz, amelyek jellemzően a `/boot` könyvtárban találhatók. Ez azt jelenti, hogy a boot loadernek mindent támogatnia kell a blokkeszközöktől kezdve a halmozott blokkeszközökön át (LVM, RAID, dm-crypt, LUKS stb.) egészen addig a fájlrendszerig, amelyen a kernel(ek) és az initramfs képfájl(ok) találhatók.
>
> Mivel szinte egyetlen boot loader sem támogatja az ilyen halmozott blokkeszközöket, és mivel a fájlrendszerek bevezethetnek olyan új funkciókat, amelyeket még egyetlen boot loader sem támogat (pl. fscrypt titkosított könyvtárak), egy különálló `/boot` partíció használata egy univerzálisan támogatott fájlrendszerrel, például a FAT32-vel, gyakran megvalósíthatóbb megoldás.

### Funkciók összehasonlítása (Feature comparison)

> **Megjegyzés:** Mivel a GPT az UEFI specifikáció része, minden UEFI boot loader támogatja a GPT lemezeket. A GPT használata BIOS rendszereken is lehetséges, akár "hibrid bootolással" (Hybrid MBR), akár az új csak-GPT (GPT-only) protokollal. Ez a protokoll azonban bizonyos BIOS implementációknál problémákat okozhat.
>
> Mivel a Secure Boot az UEFI specifikáció része, minden UEFI boot loader támogatja azt, bár némelyik korlátozásokkal.

| Név | Firmware BIOS | Firmware UEFI | Partíciós tábla MBR | Partíciós tábla GPT | Multi-boot | Fájlrendszerek | Megjegyzések |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Clover | Igen | Igen | Nem | Igen | Igen | Bővíthető² ⁵ | Képes UEFI-t emulálni hagyományos BIOS rendszereken. |
| EFI boot stub | – | Igen¹ | Igen | Igen | – | Firmware-től örökölt² | A kernel egy érvényes EFI végrehajtható fájl, amelyet az UEFI vagy egy másik UEFI boot loader közvetlenül elindíthat. |
| GRUB | Igen | Igen³ | Igen | Igen | Igen | Beépített | Támogatja a RAID-et, LUKS-t és LVM-et (de a vékonyan lefoglalt köteteket nem). |
| Limine | Igen | Igen³ | Igen | Igen | Igen | Korlátozott | |
| rEFInd | Nem | Igen | Igen | Igen | Igen⁴ | Bővíthető² ⁵ | Támogatja a kernelek és paraméterek automatikus felismerését explicit konfiguráció nélkül, és támogatja a fastbootot. |
| Syslinux | Igen | Részleges¹ | Igen | Igen | Részleges | Korlátozott | Nincs támogatás bizonyos fájlrendszer funkciókhoz. Csak ahhoz a fájlrendszerhez fér hozzá, amelyre telepítették. |
| systemd-boot | Nem | Igen³ | Manuális | Igen | Igen⁴ | Bővíthető² ⁵ | Csak abból az ESP-ből (vagy XBOOTLDR partícióból) tud binárisokat indítani, amelyre telepítve van. Automatikusan felismeri az `esp/EFI/Linux/` mappába helyezett egyesített kernelképfájlokat. |
| Unified kernel image | – | Igen³ | Igen | Igen | – | Firmware-től örökölt² | `systemd-stub(7)`, egy kernel, initramfs és kernel parancssor egy EFI végrehajtható fájlba csomagolva, amely közvetlenül az UEFI firmware-ből vagy más boot loaderből tölthető be. |
| GRUB Legacy | Igen | Nem | Igen | Nem | Igen | Korlátozott | Megszűnt a GRUB javára. |
| LILO | Igen | Nem | Igen | Részleges | Igen | Korlátozott | Megszűnt korlátozásai miatt (pl. Btrfs, GPT, RAID, titkosítás). |

¹ Bár a bináris aláírható a Secure Boot számára, nem végez további ellenőrzést, ezzel megtörve a bizalmi láncot (chain of trust).

² A fájlrendszer támogatás a firmware-ből öröklődik. Az UEFI specifikáció előírja a FAT12, FAT16 és FAT32 támogatását, de a gyártók további fájlrendszereket is támogathatnak (pl. Apple Mac firmware a HFS+ fájlrendszert). Ha a firmware biztosít felületet UEFI illesztőprogramok betöltésére, további fájlrendszerek is hozzáadhatók.

³ Támogatja a vegyes módú bootolást (64 bites x86_64 kernel bootolása 32 bites IA32 UEFI-n).

⁴ Boot manager. Csak más EFI alkalmazásokat tud indítani.

⁵ Támogatja az UEFI fájlrendszer-illesztőprogramok betöltését.

## Kernel

A boot loader betölti a kernelt tartalmazó `vmlinux` képfájlt.

A kernel alacsony szinten (kerneltér / kernelspace) működik, és a gép hardvere, valamint a programok közötti interakciót végzi. A kernel kezdetben végrehajtja a hardverek számbavételét és inicializálását, mielőtt továbblépne a felhasználói térbe (userspace).

## initramfs

Az initramfs (initial RAM file system - kezdeti RAM fájlrendszer) képfájl egy cpio archívum, amely biztosítja a korai felhasználói tér (early userspace) számára a kései felhasználói tér (late userspace) sikeres elindításához szükséges fájlokat. Ez túlnyomórészt az összes kernel modult, felhasználói terű eszközt, a hozzájuk tartozó könyvtárakat, és olyan támogató fájlokat jelent (mint az udev szabályok), amelyek a gyökérfájlrendszer felkutatásához, eléréséhez és csatolásához szükségesek. Az initramfs koncepciójával még összetettebb felépítések is kezelhetők, mint például külső meghajtóról való bootolás, halmozott eszközök (logikai kötetek, szoftveres RAID-ek, tömörítés, titkosítás) vagy egy apró SSH szerver futtatása a korai felhasználói térben a gyökérfájlrendszer távoli feloldásához vagy karbantartási feladatokhoz.

A modulok többségét az init folyamat későbbi szakaszaiban tölti be az udev, miután átváltottak a gyökérfájlrendszerre.

A folyamat a következő:

1. A `/` alatti gyökérfájlrendszer egy üres rootfs-ként indul, amely a tmpfs vagy ramfs egy speciális példánya. Ez az ideiglenes gyökérfájlrendszer, ahová az initramfs képfájlok kicsomagolásra kerülnek.
2. A kernel kicsomagolja a beépített initramfs-ét az ideiglenes gyökérbe. Az Arch Linux hivatalosan támogatott kernelei egy üres archívumot használnak beépített initramfs-ként (ez az alapértelmezett a Linux fordításakor).
3. A kernel a boot loader által átadott parancssorban megadott sorrendben kicsomagolja a külső initramfs képfájlokat, felülírva a beágyazott initramfs-ből származó vagy a korábban kicsomagolt fájlokat. Több initramfs képfájl is kombinálható egyetlen fájlban.
4. Ha az első initramfs kép tömörítetlen, kicsomagolása után a kernel CPU mikrokód frissítéseket és ACPI tábla frissítéseket fog keresni a `/kernel/x86/microcode/` illetve a `/kernel/firmware/acpi/` könyvtárakban.
5. A CPU mikrokód és ACPI tábla frissítések feldolgozása után a kernel folytatja a többi (ha van) initramfs képfájl kicsomagolását.

Az Initramfs képfájlok az Arch preferált módszerei a korai felhasználói tér beállítására, és az `mkinitcpio`, `dracut` vagy `booster` segítségével generálhatók.

### Futtatás initramfs nélkül

> **Megjegyzés:** Általában nem ajánlott initramfs nélkül futtatni a rendszert. Ez a felépítés törékeny, és extra hibaelhárítást igényelhet.

A kernel képes közvetlenül is lemezpartícióról bootolni, bár az initramfs hiánya jelentős mennyiségű korlátozást von maga után.

A hivatalosan támogatott kernelek korlátozásai:

- Csak a Btrfs és Ext4 fájlrendszer-illesztőprogramok vannak beépítve, más fájlrendszerek nem használhatók gyökérpartícióként.
- Csak a virtio és SCSI/SATA/AHCI illesztőprogramok vannak beépítve, más típusú tárolók (NVMe, USB, device mapper stb.) nem használhatók.

Általános korlátozások:

- A GPT partíciók automatikus csatolása nem használható, a root kernel paraméter mindig szükséges.
- A gyökér tartós blokkeszköz elnevezése (persistent naming) a PARTUUID-re és PARTLABEL-re korlátozódik.
- A rootflags csatolási opciói korlátozottak (pl. a noatime nem működik), de a kívánt opciók a későbbi boot szakaszokban alkalmazhatók az fstab-on keresztül.
- A `systemd-gpt-auto-generator(8)` nem használható és problémákat okoz. (Tiltsa le a `systemd.gpt_auto=no` paraméterrel).
- LVM vagy titkosítás nem használható a gyökérkötethez.
- A korai mikrokód betöltés nem lehetséges (kivéve ha külön initramfs fájlban van).

## Korai felhasználói tér (Early userspace)

A korai felhasználói tér, más néven az initramfs fázis, a rootfs-ben zajlik, és az initramfs által biztosított fájlokból áll. A korai felhasználói tér azzal kezdődik, hogy a kernel végrehajtja az `/init` binárist PID 1-ként.

Bár a korai felhasználói tér funkciója konfigurálható, fő célja a rendszer elindítása odáig, hogy hozzáférjen a tényleges gyökérfájlrendszerhez. Ez magában foglalja a következőket:

- A `systemd-modules-load(8)` betölti a kernel modulokat (pl. a valódi gyökérfájlrendszer csatolásához szükséges blokkeszköz modulokat).
- Beállítja a tárolási vermet (storage stack), amelyen a gyökérfájlrendszer elhelyezkedhet (pl. dm-crypt, dm-verity, mdadm, LVM, systemd-repart stb.).
- Kezeli a valódi gyökérfájlrendszer dekódolását, ha szükséges.
- Feloldja a tartós blokkeszköz neveket valódi eszközökre az udev segítségével.
- Betölti a DRM modult, mivel a korai KMS alapértelmezés szerint engedélyezett.

A korai felhasználói tér végső szakaszában a valódi gyökér a `/sysroot/` alá csatolódik, majd a rendszer átvált rá a `systemctl switch-root` használatával (systemd-alapú initramfs esetén) vagy a `switch_root(8)` használatával (busybox-alapú initramfs esetén). A kései felhasználói tér a valódi gyökérfájlrendszerből indított init program futtatásával veszi kezdetét.

## Kései felhasználói tér (Late userspace)

A kései felhasználói tér indítását az init folyamat végzi. Az Arch hivatalosan a systemd-t használja, amely az egységek (units) és szolgáltatások (services) koncepciójára épül, de az itt leírt funkcionalitás nagyrészt átfedésben van más init rendszerekkel is.

### getty

Az init folyamat virtuális terminálonként egyszer meghívja a `getty`-t (jellemzően hatot). A getty inicializálja az egyes terminálokat, és megvédi őket a jogosulatlan hozzáféréstől. Amikor megadják a felhasználónevet és jelszót, a getty ellenőrzi azokat az `/etc/passwd` és `/etc/shadow` fájlok alapján, majd meghívja a `login(1)` programot.

### Bejelentkezés (Login)

A `login` program megkezdi a felhasználó munkamenetét környezeti változók beállításával, és elindítja a felhasználó shelljét (parancsértelmezőjét) az `/etc/passwd` alapján. Sikeres bejelentkezés után a login program közvetlenül a bejelentkezési shell elindítása előtt megjeleníti az `/etc/motd` (a nap üzenete) tartalmát. Ez jó hely arra, hogy emlékeztesse a felhasználókat a helyi irányelvekre.

### Parancsértelmező (Shell)

Miután a felhasználó shellje elindult, jellemzően futtat egy futásidejű konfigurációs fájlt (például `bashrc`), mielőtt egy promptot jelenít meg a felhasználónak. Ha a fiók úgy van beállítva, hogy bejelentkezéskor elindítsa az X-et, a futásidejű konfigurációs fájl meghívja a `startx` vagy `xinit` programot. (Ugorjon a Grafikus munkamenet (Xorg) részre).

### Kijelzőkezelő (Display manager)

Ezenkívül az init úgy is konfigurálható, hogy a getty helyett egy kijelzőkezelőt indítson el egy adott virtuális terminálon. Ehhez manuálisan engedélyezni kell a hozzá tartozó systemd szolgáltatásfájlt. A kijelzőkezelő ezután elindít egy grafikus munkamenetet.

### Grafikus munkamenet (Xorg)

Az `xinit` futtatja a felhasználó `xinitrc` futásidejű konfigurációs fájlját, amely normál esetben egy ablakkezelőt vagy egy asztali környezetet indít el. Amikor a felhasználó végzett és kilép, az `xinit`, a `startx`, a shell és a `login` ebben a sorrendben terminálódik, visszatérve a `getty`-hez vagy a kijelzőkezelőhöz.

##Forrás:
-[https://wiki.archlinux.org/title/Arch_boot_process#Boot_loader]

# A Linux Rendszertöltők (Boot Loaderek) Alapjai: GRUB, LILO és Syslinux

A Linux bootolási (rendszerindítási) folyamata egy kritikus szoftverkomponensre, a rendszertöltőre (boot loader) támaszkodik. E nélkül a Linux kernel sosem indulna el.

## 1. Mi a Rendszertöltő Feladata és Hol Található?

A boot loader "berúgja" a Linux indítási folyamatát az alábbi lépésekkel:

- Betölti a Linux kernelt a memóriába (RAM).
- Betölti az kezdeti RAM lemezt (initrd - initial RAM disk) az ideiglenes fájlrendszerrel.
- Átadja az irányítást a kernelnek, hogy elindíthassa az init rendszert (pl. systemd).

**Hol helyezkedik el a Boot Loader?**

- **Hagyományos BIOS rendszereken (MBR):** A lemez legelső, 512 bájtos szektorában, a Master Boot Record-ban (MBR) kap helyet. A BIOS öntesztje (POST) után átadja a vezérlést az itt található kódnak.

- **Modern UEFI rendszereken (ESP):** A boot loader egy dedikált EFI Rendszerpartíción (EFI System Partition - ESP) található, és az UEFI firmware innen indítja el.

## 2. A Linux Rendszertöltők Rövid Története

- **1992:** Megjelenik a LILO (LInux LOader), az első széles körben használt rendszertöltő.
- **1995:** Elindul a GRUB (GRand Unified Bootloader) projekt (a FreeBSD kódja alapján).
- **1999:** A GRUB Legacy képessé válik Windows és más OS-ek indítására is.
- **2007:** Megkezdődik a modern GRUB 2 fejlesztése.
- **2013:** A legtöbb disztribúcióban a GRUB 2 hivatalosan is leváltja a régi GRUB Legacy-t.
- **2015:** A LILO utolsó frissítése; a projekt eléri az élettartama végét (EOL).

## 3. LILO (LInux LOader) – Az Eredeti

Werner Almesberger fejlesztette 1992-ben. Hatalmas előnye volt, hogy kódja elég kicsi volt ahhoz, hogy elférjen a szűkös MBR-ben. Bár a 90-es években egyeduralkodó volt, mára elavulttá vált.

**Miért tűnt el a LILO? (Hátrányok)**

- **Nincs UEFI támogatás:** Csak a hagyományos (Legacy) BIOS rendszerekkel működik.
- **Nincs modern fájlrendszer támogatás:** Nem tud Linuxot bootolni ext4, Btrfs vagy XFS rendszerekről.
- **Körülményes frissítés:** Bármilyen kernel- vagy lemezmódosítás után a LILO-t manuálisan újra kellett építeni és kiírni az MBR-be.
- **Nincs automatikus felismerés:** Minden operációs rendszert kézzel kellett beállítani.

## 4. GRUB (GRand Unified Bootloader) – A Modern Szabvány

A GRUB azért jött létre, hogy egy rugalmasabb, több operációs rendszert támogató (Windows, BSD, Linux) rendszert adjon. A ma használt GRUB 2 az abszolút iparági szabvány a modern Linux gépeken.

**A GRUB 2 legfőbb újításai:**

- Natív UEFI támogatás.
- **Szkriptelhető konfiguráció:** Saját parancsnyelvvel rendelkezik, amivel dinamikusan állíthatók össze a menük.
- **Moduláris és kicsi alapkód:** Az alap boot loader mindössze ~192 KB. Az extra funkciók (pl. fájlrendszer-meghajtók) csak igény szerint, modulként töltődnek be.
- **Titkosított boot:** Képes feloldani a LUKS titkosított partíciókat a kernel betöltése előtt.

**A GRUB Boot Folyamata (Fázisok / Stages):**

- **Stage 1:** A BIOS/UEFI beolvassa a boot szektort (MBR vagy ESP) és végrehajtja a parányi GRUB kódot.
- **Stage 1.5:** Ez az átmeneti fázis pont annyi adatot tartalmaz, ami elegendő a fájlrendszerek olvasásához és a következő fázis betöltéséhez.
- **Stage 2:** A fő komponens. Itt töltődnek be a modulok, a menürendszer (amit a képernyőn látunk), és ez a fázis tölti be a Linux kernelt.

## 5. Syslinux – A Pehelysúlyú Bajnok

A Syslinux (készítője Noel Kölker) egy rendkívül kompakt rendszertöltő család, amelyet kifejezetten eltávolítható adathordozókhoz (USB, CD) és szűkös memóriájú környezetekhez terveztek.

**Fő felhasználási területek (és a Syslinux változatok):**

- **SYSLINUX:** Live Linux indítása pendrive-ról (FAT fájlrendszerekhez).
- **ISOLINUX:** Linux indítása optikai lemezről (CD/DVD), ahol a képfájlokat a RAM-ba tölti.
- **PXELINUX:** Hálózati bootolás (PXE) lemez nélküli munkaállomásokhoz.
- **EXTLINUX:** Syslinux változat ext2/3/4 partíciókhoz.

**Miben különbözik a GRUB-tól?**

- **Minimalista dizájn:** Nem moduláris, nincs interaktív parancssora. Beállításait egyszerű, sima szöveges fájlokból olvassa.
- **Előnyei:** Rendkívül kicsi (< 1 MB), nem igényel külön partíciót, végtelenül egyszerű a konfigurálása, minimális függőséggel rendelkezik.

## 6. Összehasonlító Táblázat (Melyiket mikor használjuk?)

| Rendszertöltő | LILO | GRUB (GRUB 2) | Syslinux |
| --- | --- | --- | --- |
| Hatókör (Boot mód) | Csak Legacy BIOS | BIOS és UEFI | Főként eltávolítható adathordozók |
| Komplexitás | Egyszerű | Fejlett funkciók, moduláris | Nagyon egyszerű |
| Rugalmasság | Korlátozott | Rendkívül testreszabható | Közepes rugalmasság |
| Gyakori felhasználás | Nagyon régi (Legacy) rendszerek | Modern Linux telepítések (Szerver/Asztali) | Live USB/CD boot, PXE hálózati boot |

 **Összegzés:** Ha modern rendszert telepítesz, szinte biztosan a GRUB (GRUB 2) lesz az alapértelmezett és legjobb választás. Ha Live USB telepítőt készítesz, ott a háttérben valószínűleg a Syslinux (vagy ISOLINUX) dolgozik. A LILO pedig ma már inkább csak a Linux történelemkönyveibe való.

 ##Forrás
-[https://thelinuxcode.com/what-is-a-boot-loader/]

# Mi az a GRUB Bootloader Linuxon?

**Szerző:** Marko Aleksic
**Közzétéve:** 2025. december 15.
**Témák:** linux

A bootloader (rendszertöltő) az operációs rendszer egyik alapvető összetevője, amely segít a hardver inicializálásában és a sikeres rendszerindítási folyamathoz szükséges szoftverkomponensek betöltésében. Ha egy számítógépen több operációs rendszer van, a bootloaderek egy felhasználói felületet biztosítanak, amely lehetővé teszi annak kiválasztását, hogy melyik operációs rendszert töltse be.

Ez a cikk bemutatja a GRUB bootloadert, és elmagyarázza annak funkcióját és célját egy Linux rendszeren belül.

## Mi az a GRUB?

A GRUB (más néven GNU GRUB vagy GNU Grand Unified Bootloader) egy bootloader és boot manager Linuxhoz és más Unix-alapú operációs rendszerekhez. A GRUB a BIOS hardvertesztjeinek befejezése után indul el, és a Master Boot Record-ból (MBR) töltődik be. Betöltés után a GRUB átveszi a rendszer irányítását, és betölti a Linux kernelt.

Jelenleg két GRUB verzió érhető el Linuxhoz:

- **GRUB Legacy (GRUB 1).** Ez az elavult GRUB verzió még néhány régebbi Linux disztribúción megtalálható.
- **GRUB 2.** Ez a GRUB verzió támogatja az új PC architektúrákat (mint az ARM) és fájlrendszereket (mint a ReiserFS), a RAID környezeteket és az LVM-et. A jelenlegi disztribúciók többsége a GRUB 2-t használja. Például alább a Linux Mint-ben található GRUB menü látható:

GRUB 2 a Linux Mint-ben.

### Bootloader vs. Boot Manager

A GRUB egyszerre működik bootloaderként és boot managerként. Ez a két komponens különböző funkciókat lát el a rendszer indítása során.

- **A bootloader** betölti a rendszermagot (kernel) egy tárolóeszközről (merevlemezről vagy SSD-ről) a RAM-ba. A bootloaderek a hardver inicializálási feladataiban is segítenek, például a memória beállításában és a CPU konfigurálásában.
- **A boot manager** egy felhasználói felületet biztosít a felhasználónak a boot konfiguráció kezeléséhez és annak kiválasztásához, hogy melyik rendszert indítsa el. A felület lehet egy egyszerű alapértelmezett GRUB menü, vagy testre szabható egy adott disztribúcióhoz, pl. Kali Linux:

GRUB a Kali Linuxban.

## Mit csinál a GRUB?

Miután a BIOS betölti a GRUB-ot a memóriába, a GRUB egy menüt jelenít meg a felhasználónak a boot opciókkal. Ezek az opciók tartalmazzák a rendszeren telepített operációs rendszereket és a speciális boot opciókat, például a biztonságos módot (safe mode).

Miután a felhasználó kiválaszt egy opciót, vagy lejár a választásra szánt idő, a GRUB betölti a megfelelő kernelt. A folyamat sikeres befejezéséhez a GRUB a következő paramétereket veszi figyelembe:

- **BOOT_IMAGE.** Ez a paraméter adja meg a Linux Kernel helyét.
- **initrd (initial ramdisk).** Egy séma egy ideiglenes gyökérfájlrendszer RAM-ba történő betöltésére.
- **root.** A gyökérfájlrendszer helye.
- **quiet.** Egy konfigurációs opció a rendszerrel kapcsolatos kimenet elrejtésére a boot folyamat során.
- **ro.** Ez a paraméter azt adja meg, hogy a fájlrendszert csak olvasható módban kell csatolni, hogy lehetővé tegye a konzisztencia-ellenőrző programok (fsck) működését.
- **splash.** A betöltőképernyő (splash screen), amelyet az OS indítása közben kell megjeleníteni.

## GRUB Boot Folyamat

A GRUB boot folyamat két vagy három szakaszból áll:

1. **Stage 1.** A kezdeti GRUB szakasz közvetlenül azután indul, hogy a BIOS betölti a GRUB-ot az MBR-ből. A GRUB az MBR-ben lévő kis helyet használja az inicializálási kód betöltésére, amely elindítja a következő, összetettebb szakaszt.
2. **Stage 1.5.** Ez az opcionális szakasz teszi lehetővé a Stage 2 fájl dinamikus olvasását. A GRUB több fájlrendszert támogat, és telepíti azt, amelyik megfelel annak a fájlrendszernek, amelyen a Stage 2 fájl található. Ha a Stage 1.5 hiányzik a GRUB telepítése során, a Stage 2 statikusan töltődik be.
3. **Stage 2.** A GRUB boot folyamat utolsó szakasza a leghosszabb és legösszetettebb. Ez a szakasz tartalmazza a rendszerindításhoz elengedhetetlen összes lépést, beleértve:
   - A boot menü megjelenítése.
   - Az OS kernel betöltése.
   - A kernel inicializálása.
   - A user-space inicializálása.

## GRUB Funkciók

A GRUB egy sokoldalú és robusztus bootloader, amely megfelel a modern operációs rendszerek követelményeinek. Alább a GRUB legfontosabb funkcióinak listája:

- **Moduláris architektúra.** A GRUB moduláris felépítése megkerüli a BIOS korlátait, és lehetővé teszi több funkció kínálását.
- **Rugalmas konfiguráció.** A GRUB konfigurációs fájljának szerkesztésével a felhasználók testre szabhatják rendszerük különféle aspektusait, beleértve az alapértelmezett boot bejegyzést, a kernel paramétereket és az időtúllépési időtartamokat.
- **Multi-boot támogatás.** A GRUB boot menüt biztosít, megkönnyítve a több operációs rendszert használó felhasználók számára annak kiválasztását, hogy melyik operációs rendszert futtassák.
- **Dinamikus felismerés.** A GRUB dinamikusan felismeri a telepített operációs rendszereket, és boot bejegyzéseket generál.
- **Chainloading.** A GRUB integrálható más bootloaderekkel, például a Windows Boot Managerrel és az NTLDR-rel.
- **Fájlrendszer támogatás.** Különféle fájlrendszerek támogatása (mint az ext2, ext3, ext4, Btrfs, HFS+, NTFS, FAT és ReiserFS) biztonságos választássá teszi a GRUB-ot bármely UNIX-alapú OS futtatásához.
- **Szkriptnyelv.** Azok a felhasználók, akiknek egyedi boot konfigurációkra és automatizálásra van szükségük, használhatják a GRUB szkriptnyelvét.
- **Jelszóvédelem.** A GRUB fokozza a rendszerbiztonságot azáltal, hogy jelszóvédelmet biztosít a bootloader menü számára.
- **Rescue mód.** A rescue mód segít a felhasználóknak a boot-tal kapcsolatos problémák elhárításában és javításában.
- **Legacy BIOS és UEFI támogatás.** A GRUB működik mind a hagyományos BIOS-szal, mind a modern UEFI firmware-rel.
- **Hálózati boot.** A GRUB támogatja a PXE boot-ot, lehetővé téve a hálózati számítógépek szerver általi indítását.

## Hogyan lehet hozzáférni a GRUB-hoz és szerkeszteni azt?

A GRUB konfigurációjának módosítása és felületének testreszabása a konfigurációs fájl szerkesztésével történik. A GRUB konfigurálása a rendszer terminálalkalmazásán keresztül történik, és adminisztrátori jogosultságokat igényel.

### A GRUB Konfigurációs Fájl Szerkesztése

> **Megjegyzés:** A GRUB helytelen konfigurálása megakadályozhatja a rendszer elindítását. A konfigurációs fájl szerkesztése előtt győződjön meg arról, hogy friss rendszermentésekkel rendelkezik.

A GRUB konfigurálásának megkezdéséhez:

1. Nyissa meg a konfigurációs fájlt egy szövegszerkesztőben, például a Nano-ban:

   ```bash
   sudo nano /etc/default/grub
   ```

   A GRUB konfigurációs fájljának szerkesztése Ubuntu-n.

   A fájl olyan paramétereket tartalmaz, amelyek a GRUB viselkedését szabályozzák. Íme néhány gyakran szerkesztett beállítás:

   - **GRUB_DEFAULT.** Numerikus indexet vagy menübejegyzés-címet állít be alapértelmezett boot bejegyzésként.
   - **GRUB_TIMEOUT.** Meghatározza, hogy a GRUB hány másodpercig mutassa a boot menüt, mielőtt betöltené az alapértelmezett opciót.
   - **GRUB_HIDDEN_TIMEOUT.** Elrejti vagy megjeleníti a GRUB menüt.
   - **GRUB_CMDLINE_LINUX.** Meghatározza a kernel parancssori paramétereit.
   - **GRUB_DISABLE_OS_PROBER.** Letiltja a dinamikus OS-felismerést.

2. Amikor befejezte a fájl módosítását, mentse el a fájlt, és lépjen ki a szerkesztőből. Ezután használja a következő parancsot a változtatások alkalmazásához a terminálban:

   ```bash
   sudo update-grub
   ```

   Ez a parancs átvizsgálja a `/etc/default/grub` módosításait, és alkalmazza azokat a `grub.cfg`-re, amelyet a GRUB a boot folyamat során használ.

3. Indítsa újra a számítógépet a változtatások érvénybe lépéséhez.

### A GRUB Felületének Szerkesztése

A konfigurációs fájl a GRUB megjelenését is megváltoztathatja a rendszeren. Kövesse az alábbi lépéseket a GRUB testreszabásához:

1. Nyissa meg a konfigurációs fájlt egy szövegszerkesztőben:

   ```bash
   sudo nano /etc/default/grub
   ```

2. Keresse meg azokat a paramétereket, amelyek a GRUB boot manager megjelenését határozzák meg:

   - **GRUB_BACKGROUND.** Ez a paraméter lehetővé teszi egy egyedi háttérkép beállítását. Használja a következő szintaxist:

     ```plaintext
     GRUB_BACKGROUND="[path-to-the-image]/[filename].jpg"
     ```

   - **GRUB_THEME:** Ha az Ön disztribúciója támogatja a GRUB témákat, itt adja meg a témafájl elérési útját.

   Mentse el a fájlt, és lépjen ki.

3. Frissítse a GRUB-ot:

   ```bash
   sudo update-grub
   ```

A testreszabások a következő újraindításkor lépnek életbe.

### GRUB Customizer

A GRUB Customizer egy GUI eszköz a GRUB megjelenésének és hangulatának megváltoztatására anélkül, hogy manuálisan szerkesztené a konfigurációs fájlokat. Telepítse a terminálon keresztül az alábbi lépéseket követve:

1. Adja hozzá a GRUB Customizer tárolót:

   ```bash
   sudo add-apt-repository ppa:danielrichter2007/grub-customizer
   ```

   A GRUB Customizer tároló hozzáadása Ubuntu-hoz.

   Nyomjon Entert a tároló hozzáadásának megerősítéséhez, és várja meg a folyamat befejezését.

2. Frissítse a tároló információit:

   ```bash
   sudo apt update
   ```

3. Telepítse a GRUB Customizer-t:

   ```bash
   sudo apt install grub-customizer
   ```

   A GRUB Customizer telepítése Ubuntu-n.

   Írjon be Y-t, és nyomjon Entert a telepítés megkezdéséhez.

4. Indítsa el a programot a következő beírásával:

   ```bash
   grub-customizer
   ```

   Megjelenik a GRUB Customizer GUI ablaka.

   A GRUB Customizer GUI ablaka.

Íme a GRUB Customizer néhány értékes funkciója:

- Egyszerű GUI felület a boot bejegyzések átrendezéséhez a boot menüben és az alapértelmezett boot bejegyzés beállításához.
- Személyre szabási lehetőségek a GRUB menü háttérképének és általános megjelenésének megváltoztatásához.
- Lehetőségek az időtúllépési érték beállításához.
- Grafikus mód a kernel paraméterek és a GRUB környezeti változóinak módosításához.
- Lehetőségek a GRUB telepítéséhez egy adott partícióra vagy eszközre.

## Következtetés

A cikk elolvasása után többet kell tudnia a GRUB-ról, annak funkcióiról, és arról, hogyan konfigurálhatja igényeinek megfelelően. A cikk bemutatta a GRUB Customizer-t is, egy GUI eszközt a GRUB konfiguráció szerkesztéséhez.

Következő lépésként tanulja meg, hogyan lehet a Grub Rescue-t használni a Linux boot hiba javításához.

##Forrás
-[https://phoenixnap.com/kb/what-is-grub]
