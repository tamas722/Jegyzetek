# 10. Fejezet: QEMU/KVM Virtuális Gépek (1. rész)

A **QEMU** (Quick Emulator) egy nyílt forráskódú hypervisor. A host (gazdagép) szemszögéből a QEMU egy egyszerű felhasználói (user) program, ami helyi erőforrásokat (lemezek, hálózat) kap, majd ezeket emulált hardverként továbbadja a virtuális gépnek (VM). A vendég (guest) operációs rendszer úgy látja ezeket, mintha valódi hardveren futna.

A **"KVM"** (Kernel-based Virtual Machine) azt jelenti, hogy a QEMU a Linux KVM moduljának segítségével, a processzor hardveres virtualizációs bővítményeit kihasználva fut. A Proxmox VE-ben a QEMU mindig megpróbálja betölteni a KVM modult, ezért a két fogalmat a dokumentáció gyakran szinonimaként használja. A QEMU folyamat (process) mindig **root** jogokkal fut, hogy hozzáférhessen a blokk- és PCI-eszközökhöz.

## 10.1 Emulált vs. Paravirtualizált eszközök

A QEMU képes szoftveresen emulálni a pontos mását létező hardvereknek (pl. IDE vagy SATA vezérlők, E1000 hálózati kártyák). Ez jó, mert módosítás nélkül futtathatók rajta a régi operációs rendszerek, de hatalmas a teljesítményigénye (CPU overhead).

Ennek kivédésére a QEMU támogatja a **virtio** szabványt (paravirtualizáció). Ha a vendég OS-ben van virtio driver, az felismeri, hogy QEMU-ban fut, és együttműködik a hypervisorral.

> **Tipp:** Mindig, amikor csak lehet, használj virtio eszközöket (pl. **VirtIO Block**, **VirtIO SCSI**, **VirtIO Network**)!
> Egy virtio hálózati kártya akár háromszor nagyobb átviteli sebességre képes, mint egy emulált E1000-es, a VirtIO lemezvezérlő pedig megduplázhatja a szekvenciális írási sebességet az IDE-hez képest.

## 10.2 Virtuális Gép Beállítások (VM Settings)

A GUI-n vagy a parancssorban beállított értékek határozzák meg a VM működését. **Óvatosan változtass**, mert egy rossz beállítás drasztikus teljesítményeséshez vezethet!

### 10.2.1 Általános és OS beállítások

- **Node:** A fizikai szerver, amin a VM futni fog.
- **VM ID:** Egyedi azonosítószám a teljes clusterben. (A 100 alatti ID-k a rendszer számára fenntartottak).
- **Name (Név):** Szabadon választható szöveges név.
- **Resource Pool:** A VM besorolása egy logikai csoportba.
- **Guest OS (Operációs rendszer):** Beállítása azért fontos, mert a Proxmox ez alapján optimalizál alacsony szintű paramétereket. Például a Windows elvárja, hogy a BIOS óra a helyi időt (Local time) mutassa, míg a Linux (Unix) alapú rendszerek az UTC-t.

### 10.2.3 Rendszer beállítások (System Settings)

- **Graphic Card (Kijelző) & SCSI Controller:** Itt választható ki a megjelenítő és a lemezvezérlő (lásd később).
- **QEMU Agent (Kritikus funkció):** Ha bepipálod, a Proxmox tudni fogja, hogy a VM-ben fut (vagy telepítve lesz) a **QEMU Guest Agent**. Ez lehetővé teszi, hogy a host tiszta leállítási parancsokat (`shutdown`) küldjön, konzisztens snapshotokat készítsen a fájlrendszer fagyasztásával (`fsfreeze`), és kiírja a vendég IP-címét a GUI-ra.
- **BIOS / UEFI:** A klasszikus **SeaBIOS** (alapértelmezett, kiváló a legtöbb esetben) vagy az **OVMF** (nyílt forráskódú UEFI). Az OVMF kötelező PCIe passthrough esetén (pl. videókártya átadás), és bizonyos OS-ek (pl. Windows 11) is megkövetelik.

**Machine Type (Gép típusa):** A virtuális alaplap architektúrája.

- **Intel 440FX:** Az alapértelmezett, régi, stabil szabvány.
- **Q35:** Fejlettebb chipset, virtuális PCIe busszal rendelkezik. Kötelező, ha PCIe hardvert (pl. GPU) akarsz átadni (passthrough) a gépnek.

**Machine Version (Gép verziója):** Minden gép típushoz verziók (pl. 8.1, 9.2) tartoznak.

- Linux esetén az **"Latest"** (legújabb) van érvényben, vagyis a Proxmox mindig a legfrissebbet használja.
- Windowsnál a létrehozáskori verzió fixen rögzül (**pinned**), mert a Windows a verzióváltás miatt azt hihetné, hogy hardvert cseréltek alatta (pl. a hálózati kártyák felsorolása megváltozhat, és eldobná az IP beállításokat).

**PVE Machine Revision (+pveX):** A Proxmox néha eszközöl módosításokat a QEMU verziókon (pl. `pc-q35-9.2+pve0`).

**Verziók kivezetése (Deprecation):** A QEMU 10.1-től kezdve a **6 évnél régebbi** géptípus-verziókat törlik (kb. 2 nagy Proxmox kiadásonként). Ha elavulás figyelmeztetést látsz a GUI-n, frissítened kell a gép verzióját egy újabbra, de előtte **KÖTELEZŐ a backup**, mert az OS újrahúzza a drivereket. Fontos: ha egy RAM-ot is tartalmazó snapshotot (állapotmentést) készítettél egy régi géptípus-verzióval, annak a verziója utólag **nem módosítható**; csak a snapshot betöltésével lehet kimenteni belőle az adatokat.

### 10.2.4 Hardveres Lemezek (Hard Disk)

**Busz / Vezérlő**
A Proxmox több vezérlőtípust emulál. A javaslat mindig a **VirtIO SCSI** vagy **VirtIO Block**.

- **IDE:** 1984-es PC/AT szabvány, max. 4 eszköz köthető rá. Csak 2003 előtti (nagyon régi) rendszerekhez ajánlott, amik mást nem támogatnak.
- **SATA:** 2003-as dizájn, picit gyorsabb, max. 6 eszközt kezel.
- **SCSI:** 1985-ös, szerver-szintű vezérlő (alapból LSI 53C895A), max. 14 eszköz.
- **VirtIO SCSI (AJÁNLOTT):** A Proxmox VE 7.3 óta minden új Linux VM alapértelmezése a **VirtIO SCSI single**. Teljesítményre van kihegyezve. Ennél a módnál minden egyes virtuális lemez megkapja a saját VirtIO SCSI vezérlőjét! Windows alá a telepítéskor a VirtIO ISO-ról kell betölteni hozzá a drivert.
- **VirtIO Block:** Régebbi, paravirtualizált blokkeszköz (`virtio-blk`), a VirtIO SCSI mára felülmúlta.

**Image Format (Lemezképek formátuma)**

- **`raw` (nyers):** Bitről-bitre másolat a lemezről (mint a `dd` parancs). Önmagában nem támogatja a thin provisioningot (vékony allokációt) és a snapshotokat, de ha a mögöttes tároló (pl. ZFS, Ceph, LVM-thin) támogatja ezeket, akkor ez a legjobb választás. Gyakran akár 10%-kal gyorsabb, mint a `qcow2`.
- **`qcow2` (QEMU image format):** Copy-on-write formátum. Beépítve támogatja a vékony allokációt és a snapshotokat. Fájl-alapú tárolókon (pl. NFS, ext4) kötelező ezt használni, ha snapshotokat akarsz készíteni.
- **VMware formátum:** Csak akkor használd, ha a lemezt át akarod vinni egy ESXi szerverre.

**Haladó Lemez-opciók**

- **Cache Mode (Gyorsítótár):** Az alapértelmezett a **No cache**. Ez a legjobb kompromisszum a biztonság és a sebesség között. A vendég OS akkor kapja meg a nyugtázást az írásról, amikor az adat fizikailag elérte a storage írási sorát (kihagyva a host memóriagyorsítótárát / page cache-t).
- **No backup / Skip replication:** Beállítható, hogy egy konkrét lemezt a Proxmox biztonsági mentő rendszere vagy a Storage Replication (tárhelyreplikáció) figyelmen kívül hagyjon.
- **Trim / Discard:** Ha a tárhely támogatja a vékony allokációt (**Thin provisioning**), a Discard opció engedélyezésével a vendég OS TRIM parancsai eljutnak a fizikai tárolóig, így a VM-ben törölt fájlok után felszabadul a fizikai hely a storage-on. **Feltétel:** VirtIO Block esetén a vendég kernelnek 5.0-nál újabbnak kell lennie.
- **SSD Emulation:** Ha azt akarod, hogy az operációs rendszer a meghajtót szilárdtest-meghajtóként (SSD) érzékelje a hagyományos forgólemez helyett. Ehhez **nem kötelező** fizikai SSD-nek lennie a gépben. VirtIO Block vezérlővel ez a funkció **NEM működik**.
- **IO Thread (Kritikus teljesítmény-opció):** Csak **VirtIO SCSI single** vagy **VirtIO Block** vezérlőkkel működik. Ha bekapcsolod, a QEMU a lemezműveleteket nem a fő eseményhurokban (vagy a vCPU szálakon) végzi, hanem egy dedikált I/O szálat hoz létre lemezenként. Ez masszív I/O terhelésnél megakadályozza a vendég OS lefagyását / akadását, mert a lemezműveletek nem blokkolják a processzormagokat.

### 10.2.5 Processzor (CPU) beállítások

**Sockets & Cores (Foglalatok és Magok):** A teljesítmény szempontjából majdnem mindegy, hogy 1 foglalatot (socket) adsz 4 maggal (core), vagy 2 foglalatot 2 maggal. Bizonyos Windows licencek viszont foglalat-alapúak, így ott érdemes a Socket értékét alacsonyan tartani. Általános szabályként a **Total cores = 2** egy jó kiindulópont ismeretlen terhelésnél.

> **Tipp a túlterhelésről (Overprovisioning):** Nyugodtan adhatsz több virtuális magot a VM-eknek összesen, mint ahány fizikai magod van (pl. négy darab 4 magos VM egy 8 magos szerveren), a Linux kernel ütemezője megoldja az elosztást. Azt viszont a Proxmox letiltja, hogy egyetlen VM több magot kapjon, mint a fizikai processzormagok száma, mert a kontextusváltások (**context switch**) miatt az drasztikusan belassítaná a gépet.

**CPU Limit (CPU korlátok és ütemezés)**

- **`cpulimit`:** Százalékos érték lebegőpontos formátumban (pl. az `1.0` a 100%, a `4.0` a 400%). Ha egy 8 magos VM-nél nem akarod, hogy a host gép mind a 8 magját teljesen, 100%-osan lefoglalja, beállíthatod a limitet `4.0`-ra. Így a VM továbbra is 8 vCPU-t lát, de a fizikai processzoridőből maximum 4 teljes magnyi kapacitást (400%) kap meg.
- **`cpuunits` (vagy CPU weight):** Relatív súlyozás (alapértelmezetten 100, vagy régi cgroup v1 esetén 1024). Ha a 100-as VM-nek meghagyod a 100-at, a 200-as VM-nek pedig 200-ra állítod, akkor processzor-szűkület (terhelés) esetén a 200-as gép kétszer annyi CPU időt fog kapni a kerneltől.
- **`affinity`:** Itt megadható (pl. `0-1,8-11`), hogy a VM mely fizikai magokon futhat (**CPU pinning**). Ez nem biztonsági funkció, és a QEMU periféria/IO szálakra nem vonatkozik. Karbantartása és a migrációk szempontjából problémás lehet, így csak nagyon indokolt esetben ajánlott a használata.

**CPU Type (Processzortípus) és Migrációs stratégiák**
A QEMU képes 486-ostól a legújabb Xeonig bármit emulálni.

- **`host` típus:** A VM pontosan megkapja a fizikai processzor összes létező parancskészletét (flagjét). Ez adja a legmagasabb teljesítményt. A bökkenő: ha a VM-et élőben átmigrálod egy másik (akár csak picit régebbi mikroarchitektúrájú) szerverre a clusterben, a gép **összeomlik**, mert útközben eltűnnek alóla a processzor-utasítások. Csak azonos hardverű clusternél vagy önálló gépnél használd!
- **Virtuális típusok (Proxmox 8 default = `x86-64-v2-AES`):** Ezek olyan "közös nevezők", amiket minden modern CPU ismer. A 2020-ban meghatározott x86-64-ABI szintek:
  - `kvm64` (v1): Pentium 4 szint, nagyon régi, elavult, a PVE régen ezt használta alapként.
  - `x86-64-v2`: Nehalem (Intel) / Opteron G3 (AMD) szint (SSE4.2, popcnt).
  - `x86-64-v2-AES`: Westmere szint (az AES titkosítás miatt gyorsabb hálózat/VPN kezelés). A Proxmox új GUI alapértelmezése.
  - `x86-64-v3`: Haswell / EPYC szint (AVX, AVX2).
  - `x86-64-v4`: Skylake / EPYC Genoa (AVX-512).
- **Custom CPU Models (Egyedi CPU-k):** A Datacenter szintjén létrehozhatsz saját CPU modellt a `/etc/pve/virtual-guest/cpu-models.conf` fájlban, és finomhangolhatod, mely flageket kapcsolod be/ki. A kiosztásához `Mapping.Use` jogosultság kell.

**Meltdown / Spectre CPU flagek (Biztonság)**
Ha nem `host` CPU típust használsz, bizonyos sebezhetőségek (Meltdown, Spectre v1/v2/v4) hardveres ellenszereit kézzel kell beállítanod az "Extra CPU Flags" alatt, hogy a vendég operációs rendszer is védve legyen (és persze a Host gépen frissített microcode-ra is szükség van).

- **Intel:** `pcid` (csökkenti a KPTI / Meltdown javítás miatti teljesítményesést), `spec-ctrl` (Spectre v1/v2 javítás támogatása IBRS nélkül), `ssbd` (Spectre v4 javítás).
- **AMD:** `ibpb` (Spectre v1/v2 retpoline nélkül), `virt-ssbd` (Spectre v4 virtuális védelem), `amd-ssbd` (hardveres védelem), `amd-no-ssb` (új CPU-knál jelzi a VM-nek, hogy a host egyáltalán nem is sebezhető, ne pazaroljon erőforrást a védekezésre).

**Windows specifikus beállítások (Intel):** A `cet-ibt` és `cet-ss` (Control-flow Enforcement) flagek Windows 11 alatt ki vannak kapcsolva, mert jelenleg akadályozzák a bootolást a Virtualization-based security (VBS) használatakor. Ugyanígy a `level=30` CPU szint opció (a CPUDID lekérdezések limitálása) egy ismert workaround a Windows Hyper-V boot hibáira új Intel hostokon, ha a típus `host` vagy `max`.

**NUMA és vCPU Hot-plug (Meleg cserélés)**

- **NUMA:** Modern, több foglalatos szervereknél (pl. 2 db processzor) a memória lokálisan tartozik egy-egy fizikai processzorhoz. Ha a NUMA opciót bekapcsolod (és a host ténylegesen NUMA architektúrájú), a Proxmox tudni fogja, hogyan ossza el optimálisan a szálakat és a RAM-ot. Alapszabály: a VM "Socket" értékét állítsd egyezőre a host fizikai NUMA node-jainak számával. (A NUMA opció elengedhetetlen feltétele a magok és a RAM üzem közbeni, **"hot-plug"** cseréjének is).
- **vCPU hot-plug:** Egy modern, futó VM-hez menet közben is hozzáadhatsz újabb processzormagokat (a lecsökkentés viszont gépfüggő és nem garantált). A Proxmox beállításaiban a `Core * Socket` adja a maximális "bedugható" (plugged) magok számát, a `vcpus` opció pedig azt jelöli, hogy induláskor valójában mennyit használ a VM.

### 10.2.6 Memória beállítások

A RAM allokáció lehet fix (**Memory = Minimum Memory**), vagy Dinamikus (**Ballooning**).

- **Fixed Memory (Fix memória):** A Proxmox annyi RAM-ot foglal le, amennyit megadsz. Még ilyenkor is települ a gépbe egy "Ballooning Device" (lufi eszköz), aminek a fő feladata csupán a statisztika szolgáltatása (megmondja a Proxmoxnak, mennyi a valós RAM használat odabent). Ha valamiért zavar, a konfigban kiiktatható a `balloon: 0` paraméterrel.
- **Automatic Memory Allocation (Auto-ballooning):** Ha a Minimum memory alacsonyabb, mint a Memory (a maximális keret), a Proxmox dinamikusan játszik a kiosztott memóriával.
  - **Hogyan működik?** A host célja alapértelmezetten, hogy a fizikai szerver RAM-jának csak **80%-a** legyen lefoglalva. Ha ez feljebb kúszik, a host szól a VM-ben futó "lufi" drivernek, hogy "fújódjon fel". A lufi lefoglalja a memóriát a VM kernelétől, majd a lapokat átadja a hostnak. Amikor a host felszabadul, a lufi leereszt, és visszaadja a RAM-ot a virtuális gépnek.
- **Shares (Súlyozás):** Hasonlóan a CPU-hoz, itt is lehet prioritást adni. Ha több gép akarja visszaszerezni a szabad RAM-ot, az a gép kap belőle többet, aminek a Shares értéke magasabb (az alapértelmezés az **1000**). Pl. egy 3000-es értékű VM háromszor annyi memóriát kap vissza az osztozkodásból, mint az 1000-es.

> **Szabály:** Soha ne add oda az összes RAM-ot a VM-eknek! Mindig hagyj legalább **1 GB** memóriát szabadon a Proxmox host operációs rendszerének! Windows vendégek esetén a ballooning drivert külön kell telepíteni (VirtIO ISO), de kritikus rendszereknél teljesítménycsökkenést okozhat, ezért ott a használata nem ajánlott.

**AMD SEV / Memóriatitkosítás (10.2.7)**
**Kísérleti funkció!** AMD EPYC (7002/7003+) rendszereken hardveres (AES-128) titkosítást lehet alkalmazni a VM-ek memóriájára, sőt a CPU regisztereire (SEV-ES) is, ami megakadályozza, hogy a hypervisor (a host rendszer) vagy bármely más folyamat belelásson a virtuális gép RAM-jába.

- **Korlátozások:** Élő migráció, snapshotok és PCI passthrough jelenleg **nem működnek SEV alatt**.
- **Engedélyezés:** A PVE kernel paraméterében (`kvm_amd.sev=1`), a konfiguráció pedig a `qm set <vmid> -amd-sev type=std...` paranccsal történik.

### 10.2.8 Hálózati eszközök (Network Device)

Egy VM-hez akár több hálózati kártya (NIC) is hozzáadható.

**Kártyamodellek:**

- **VirtIO (paravirtualized):** (AJÁNLOTT) A legmagasabb teljesítményt és a legkisebb CPU terhelést adja. (Windows alá driver kell).
- **Intel E1000:** Az alapértelmezett kompatibilitási modell (virtuális Gigabit). Ha a gép nem ismeri a VirtIO-t, ez a legjobb.
- **Realtek 8139:** Elavult (100 Mbit/s), kizárólag nagyon régi (2002 előtti) OS-ekhez.
- **`vmxnet3`:** VMware paravirtualizált kártya. Csak ESXi-ből importált gépeknél érdemes meghagyni.

A MAC-címet a rendszer véletlenszerűen generálja.

**Hálózati Módok (Bridge vs. NAT)**

- **Bridged mode (Híd):** Az alapértelmezett. A VM kap egy **tap** (virtuális patch kábel) interfészt, ami bekötődik a Proxmox hídjába (pl. `vmbr0`). A VM úgy működik, mintha bedugták volna a fizikai helyi hálózatba (LAN).
- **NAT mode:** Csak parancssorból/API-ból érhető el. A QEMU egy saját beépített DHCP servert és routert ad a VM-nek (`10.0.2.0/24` tartományban), és a kimenő forgalom a host gép IP-jén megy ki (**MASQUERADE**). Rettenetesen lassú, csak nagyon speciális tesztkörnyezetbe való.

Az MTU (Maximum Transmission Unit) mérete felülírható, vagy az `mtu=1` beállítással átörökíthető a mögöttes Linux hídról (csak VirtIO kártyán).

**Multiqueue (Többszörös sorbaállítás)**
Ha a VirtIO kártyát használod, a hálózati csomagok feldolgozása alapból egyetlen szálon (a QEMU host kernel `vhost` drivere által) történik. Hatalmas bejövő hálózati terhelésnél (pl. egy router vagy reverse proxy VM esetén) ez a szál szűk keresztmetszet lehet.
A Multiqueue bekapcsolásával több virtuális CPU is részt vehet a hálózati megszakítások feldolgozásában.

- **Beállítása:** Érdemes a VM vCPU-jainak számával megegyező értéket beírni.
- **Vendég oldali konfiguráció (Linux):** Az `ethtool -L ensX combined <vCPU_szám>` paranccsal kell engedélyezni a többcsatornás működést a VM-ben.
- **Vendég oldali konfiguráció (Windows):** A VirtIO driver telepítése után az eszközkezelőben a hálózati kártya beállításainál a "Receive Side Scaling" opciót "Enabled"-re kell állítani, és a "Maximum number of RSS Queues" értékét a vCPU-k számára kell felhúzni.

> **Figyelmeztetés:** A Multiqueue megnöveli a host CPU terhelését, így tényleg csak extrém hálózati terhelésű gépeknél kapcsold be!

### 10.2.9 Kijelző (Display) és VNC Clipboard

A QEMU többféle videókártyát emulál a webes konzol megjelenítéséhez.

- **`std`:** A jelenlegi alapértelmezett kártya a legtöbb modern rendszerhez (Bochs VBE).
- **`qxl`:** Paravirtualizált QXL kártya, amely bekapcsolja a SPICE távoli asztali protokollt (ami jobb minőséget, hangot és USB megosztást is nyújt a sima HTML5 noVNC-hez képest).
- **`virtio-gl` (VirGL):** Virtuális 3D GPU. Ez a zseniális megoldás képes a 3D grafikai műveleteket áthárítani a host gép fizikai videókártyájára, anélkül, hogy a teljes fizikai kártyát (PCI passthrough) át kéne adni. Így több VM is osztozhat a host fizikai GPU-ján. Ehhez a Proxmox node-on külön csomagokat kell telepíteni:

```bash
apt install libgl1 libegl1
```

- **`cirrus`:** Nagyon régi (Windows XP korabeli) emuláció, kerülendő.
- **`serial0` (Serial Terminal):** A VGA kimenet teljesen letiltásra kerül, és a Proxmox Webes Konzolja egy szöveges, soros porti ablakot nyit a géphez. Szöveges (headless) Linux szervereknél ajánlott.

**Vágólap (Clipboard) a Webes Konzolban**
A noVNC (HTML5) webes konzolban is lehetséges a host (a saját géped) és a VM közötti "Copy-Paste" művelet (vágólap megosztás).

- Ehhez a VM Display opciói között a `clipboard` beállítást `vnc`-re kell állítani (`qm set <vmid> -vga <displaytype>,clipboard=vnc`).
- A vendég operációs rendszerben **KÖTELEZŐ** feltelepíteni a SPICE Guest Tools-t (Linuxon a csomag neve: `spice-vdagent`). Ha ez nincs fent, a vágólap nem fog működni.

> **Figyelem:** Mivel ez a háttérben a `qemu-vdagent` eszközt használja, az ezzel bekapcsolt VM-ek élő migrációja (Live Migration) csak **10.1-nél frissebb** QEMU/Machine verziókon támogatott.

## 10.2 Virtuális Gép Beállítások (folytatás)

### 10.2.10 USB Passthrough (USB átadás)

Kétféleképpen adhatsz át USB eszközt a virtuális gépnek:

- **Host USB passthrough:** A fizikai szerverbe dugott eszközt adod át a VM-nek, vagy a Vendor/Product ID (pl. `0123:abcd`), vagy a fizikai Busz/Port (pl. `1-2.3.4`) alapján.
  > **Kritikus korlátozás:** Ezzel a módszerrel a VM élő migrációja (**Live Migration**) lehetetlenné válik, hiszen a fizikai hardver csak az adott hoston létezik!
- **SPICE USB passthrough:** Lehetővé teszi, hogy a saját (kliens) gépedbe dugott USB eszközt (pl. pendrive, hardverkulcs) a SPICE kliensen keresztül dinamikusan, hálózaton át add be a virtuális gépnek.

### 10.2.11 BIOS és UEFI (OVMF)

A QEMU alapértelmezetten a **SeaBIOS**-t használja (csak x86-on), ami a legtöbb esetben tökéletes. Azonban bizonyos rendszerek (pl. Windows 11) vagy a PCIe GPU passthrough megkövetelhetik az UEFI használatát. Erre szolgál az **OVMF** (Open Virtual Machine Firmware).

**EFI Disk:** Az OVMF-nek szüksége van egy dedikált **"EFI Disk"**-re a boot sorrend és a beállítások tárolásához. Ebből csak egy lehet, és a mentések/snapshotok részét képezi. A modern gépeknél a **`4m`** verziót kell használni (ez támogatja a Secure Boot-ot), és be kell kapcsolni a `pre-enrolled-keys=1` opciót, hogy a Microsoft szabványos kulcsai bekerüljenek.

> **Secure Boot tanúsítványok lejárata (Kritikus!):** A 2011-es Microsoft tanúsítványok **2026 júniusában lejárnak**. A Proxmox (a `pve-edk2-firmware 4.2025.05-1` verziójától) már a 2023-as kulcsokat is tartalmazza (`ms-cert=2023k` jelölés). Régebbi gépeknél a GUI-n a **Disk Action $\rightarrow$ Enroll Updated Certificates** menüponttal kell frissíteni őket.
> **Figyelem, BitLocker használata esetén:** Mielőtt a kulcsokat frissíted, a Windowsban rendszergazdaként ki kell adni a `manage-bde -protectors -disable C:` parancsot, különben a következő bootnál a Windows kérni fogja a BitLocker helyreállítási kulcsot.

### 10.2.12 TPM (Trusted Platform Module)

A TPM (pl. Windows 11-hez) egy **`tpmstate`** kötetként adható a géphez. Szintén `raw` formátumú, és utólag **nem módosítható**, csak törölhető. Két verziója van: **v1.2** és **v2.0**. Ha nincs specifikus ok a régire, mindig a **v2.0** használandó. Mivel ez szoftveresen emulált és a fájlrendszeren tárolódik, fizikai biztonságot (**tamper-resistance**) valójában nem nyújt a host-oldali hozzáféréssel szemben.

### 10.2.13 – 10.2.16 További Speciális Hardverek

- **Inter-VM shared memory (`ivshmem`):** A gazdagép és a VM (vagy több VM) közötti közvetlen RAM megosztást teszi lehetővé (pl. a **Looking Glass** projekthez az alacsony késleltetésű képernyőtükrözéshez).

```bash
qm set <vmid> -ivshmem size=32,name=foo
```

- **Audio Device:** Hang megosztása SPICE kliensen keresztül. Támogatott modellek: `ich9-intel-hda`, `intel-hda`, vagy öreg rendszerekhez az `AC97`.
- **VirtIO RNG (Random Number Generator):** Hardveres véletlenszám-generátor, ami a hostról (preferáltan a `/dev/urandom`-ból) ad entrópiát a VM-nek. Megakadályozza, hogy a vendég rendszer a bootolás során megakadjon **"entrópia-éhezés"** (**entropy starvation**) miatt. Szabályozható a másodpercenként átvihető adatmennyiség (pl. `1 KiB/s`).
- **Virtiofs:** A VM és a host közötti rendkívül gyors, közvetlen fájlrendszer-megosztás hálózati stack (NFS/CIFS) nélkül. **Linux $\ge$ 5.4** támogatja. A hoston a `virtiofsd` csomag telepítése szükséges.
  > **Korlátozások:** Nem támogatja a memória hot-plugot, az élő migrációt, a snapshotokat és a hibernálást. A Windows az ACL (jogosultság) kezelést itt nem érti, így Windows VM-eknél az ACL-t ki kell kapcsolni.

### 10.2.17 – 10.2.19 Indítási sorrend, Automatikus Indítás és QEMU Guest Agent

- A QEMU számára megadható, honnan próbáljon bootolni (pl. `boot: order=scsi0;net0`). **Fontos:** az újabb SeaBIOS és OVMF verziók csak azokat a lemezeket inicializálják bootoláskor, amelyek a boot listán (pipával jelölve) szerepelnek.
- **Auto Start / Shutdown:** Beállítható a bootolási sorrend (**Start/Shutdown order**), a csúsztatás (**Startup delay**), és a leállítási időtúllépés (**Shutdown timeout**, alapból **180 mp**). A leállítás fordított sorrendben történik. A High Availability (HA) által kezelt gépekre ezek a szabályok **nem vonatkoznak!**
- **QEMU Guest Agent (QGA):** A VM-en belül futó démon, ami kommunikál a Proxmox hosttal. Kritikus funkciói:
  - Az IP-címek megjelenítése a GUI-n.
  - Backup és snapshot készítésekor a fájlrendszer "befagyasztása" (`guest-fsfreeze-freeze`) a konzisztencia érdekében. Windows alatt ez néha összeakadhat az MS SQL VSS írójával, ilyenkor a fagyasztás kikapcsolható a GUI-n, de ez inkonzisztens mentéseket eredményezhet.
  - Automatikus TRIM (**Run guest-trim**) a Storage migrációk után a vékony allokáció fenntartásához.

### 10.2.20 SPICE Kiegészítők

Ha a Display típus SPICE (`qxl`), két extra funkció aktiválható:

- **Folder Sharing (Mappamegosztás):** Egy helyi mappa megosztása a vendéggel WebDAV-on keresztül (`http://localhost:9843`). A VM-ben a `spice-webdavd` csomag telepítése szükséges. (**Kísérleti funkció!**)
- **Video Streaming:** A gyorsan frissülő képernyőterületeket videostreamként kódolja, ami csökkenti a hálózati sávszélességet.

## 10.3 – 10.6 Migráció, Klónozás és Sablonok (Templates)

### Migráció (Live és Offline)

- **Online (Live) Migration:** A VM leállása nélkül helyezi át a gépet egy másik node-ra. Ehhez azonos CPU architektúra, stabil cluster hálózat, és nem-lokális hardverek (pl. PCI átadás nélküli gép) szükségesek. A RAM tartalmát aszinkron módon másolja át, majd az utolsó milliszekundumos különbségnél egy észrevehetetlen pillanatra megállítja a gépet (**pause**), és átvált a cél node-ra. A hálózati kapcsolatok megtartása érdekében a rendszer megpróbálja átvinni a **conntrack** (tűzfal állapot) táblákat is (**Conntrack State Migration**), de ez SNAT (masquerading) esetén nem garantált.
- **Offline Migration:** Ha a lemez megosztott, a gép egyszerűen átkerül. Ha a lemez lokális (pl. LVM-thin), az Offline migráció a lemez tartalmát is átmásolja a hálózaton a célgép azonos nevű lokális tárolójára.

### Klónozás és Sablonok

- **Full Clone (Teljes klón):** Független, önálló VM. Fizikailag átmásolja a lemezt, ami lassú, de a cél tárhely és a formátum szabadon változtatható.
- **Linked Clone (Csatolt klón):** Villámgyors klónozás a Copy-on-write elvre építve. Az új gép csak a változásokat (deltákat) írja egy új fájlba, a meglévő, módosítatlan adatokat egy **Sablonból (Template)** olvassa.
- **Virtual Machine Templates:** Egy VM-et bármikor Sablonná konvertálhatsz. Ez egy írásvédett (**read-only**) állapot. **Fontos:** A sablont nem lehet elindítani (mert az módosítaná a lemezt), és addig nem törölhető a sablon, amíg létezik belőle akár egyetlen Linked Clone is!
- **VM Generation ID (`vmgenid`):** Egy **128 bites** egyedi azonosító, amit a QEMU ad át a vendég OS-nek. Ebből a vendég (főleg a Windows Active Directory és adatbázisok) érzékelni tudja, ha egy korábbi snapshotból lett visszaállítva, vagy klónozták, így elkerülhetők a replikációs / idő-eltolódási anomáliák.

## 10.7 Importálás idegen rendszerekből

Virtuális gépeket importálhatsz VMware ESXi-ből vagy más hypervisorokból:

- **Native Import Wizard:** A GUI-n keresztül közvetlenül importálható a VM egy hozzáadott ESXi tárolóról (ESXi 6.5 – 8.0 verziókkal tesztelve, de a vSAN nem támogatott).
- **OVA/OVF Import:** OVF konfigurációs fájlok és `vmdk`/`qcow2` lemezképek feltöltése és importálása. Ha az OVF fájlból hiányzik a SCSI vezérlő típusa (ami gyakori), manuálisan kell átállítani **Virtio SCSI**-ra, különben a gép nem fog bootolni OVMF (UEFI) alatt. A parancssori eszköz ehhez a `qm importovf`.

## 10.8 Cloud-Init Támogatás

A **Cloud-Init** a virtualizációs szabvány (OpenStack-ből ered) egy VM korai inicializálására (hálózat beállítása, SSH kulcsok átadása, jelszó beállítása, lemez átméretezése). A Proxmox egy virtuális CD-ROM-ot hoz létre a Cloud-Init adatokkal, amit a vendég OS induláskor beolvas.

- **Linux:** Alapértelmezett formátum a **`nocloud`**. Javasolt SSH kulcsos bejelentkezést használni a jelszavas helyett (mert a jelszó visszafejthető a Cloud-Init adatokból).
- **Windows (Cloudbase-Init):** A formátumot **`configdrive2`**-re kell állítani, és az operációs rendszer típust `Windows`-ra (`ostype`). Kézzel kell telepíteni a **Cloudbase-Init** eszközt. Gyakran használják együtt a Windows **Sysprep** eszközével (`sysprep.exe /generalize /oobe /unattend:Unattend.xml`), ami letakarítja az egyedi gépazonosítókat, így egy tökéletesen tiszta Windows Template-et kaphatsz a klónozáshoz.

## 10.9 PCI(e) Passthrough (Hardver átadás)

Lehetővé teszi, hogy egy fizikai PCIe kártyát (pl. videókártyát, HBA vezérlőt) natívan, QEMU emuláció nélkül átadj a VM-nek. Ez minimális késleltetést (**latency**) és maximális teljesítményt ad, de a kártya onnantól a gazdagép (host) számára elérhetetlenné válik.

**Követelmények és Host beállítások:**

- Az alaplapnak és a CPU-nak támogatnia kell a hardveres megszakítás-újrairányítást (**IOMMU / VT-d / AMD-Vi**).
- A BIOS-ban engedélyezni kell az IOMMU-t. (Intel esetén korábban grub paraméter is kellett: `intel_iommu=on`).
- Az `iommu=pt` (passthrough) kernel paraméter javítja a host I/O teljesítményét a DMA kérések gyorsításával.
- Betöltendő kernel modulok (`/etc/modules-load.d/vfio.conf`): `vfio`, `vfio_iommu_type1`, `vfio_pci`. (Ezután `update-initramfs -u -k all`).
- A fizikai kártyának megfelelő IOMMU csoportba kell esnie.

**GPU átadás (VGA Passthrough):**
Ha a videókártyát elsődleges megjelenítőként (`x-vga=on`) akarod használni, a gép Machine Type-ja kötelezően **`q35`** legyen, és **OVMF**-et (UEFI) használj. A `rombar=on` alapértelmezett, de hibás kártyáknál külső `romfile=<path>` is beadható.

> **Tipp:** Ha a host kernel befoglalja a kártyát, a driverét tiltólistára kell tenni (**blacklist**), vagy a `vfio-pci` modulhoz soft-dependency-t kell beállítani.

**SR-IOV (Single-Root I/O Virtualization):**
Olyan hardveres képesség (tipikusan hálózati kártyáknál), amivel egy fizikai kártya több **VFs**-t (Virtual Function) hoz létre. Így egy 10G-s kártya hardveres tulajdonságait (pl. checksum offloading) szétoszthatod több VM között úgy, hogy közvetlenül a VM-be csatolod be a virtuális kártyát (SR-IOV) PCI eszközként.

**Mediated Devices (vGPU / GVT-g):**
Hasonló az SR-IOV-hez, de kimondottan Intel (**GVT-g**) vagy NVIDIA (**GRID vGPU**) kártyákhoz. A fizikai eszköz nem darabolódik PCI eszközökre a hoston, hanem az illesztőprogram a `/sys/bus/pci/devices/.../mdev_supported_types` alatt létrehoz felosztható példányokat. Konfiguráció:

```bash
qm set VMID -hostpci0 00:02.0,mdev=i915-GVTg_V5_4
```

**vIOMMU (Nested Virtualization):**
Ha egy virtuális gépben futtatott virtuális gépnek (Level-2) is akarsz PCI-t átadni, emulálhatsz egy vIOMMU-t (Intel vagy VirtIO) az elsődleges VM-nek:

```bash
qm set VMID -machine q35,viommu=virtio
```

## 10.10 – 10.15 Hookscriptek, Hibernálás, Resource Mapping és a `qm` parancs

- **Hookscripts:** A VM életciklus-állapotaiba (pl. indulás előtt, leállítás után) beékelődő egyedi Perl/Shell scriptek (pl. a `local:snippets/hookscript.pl` formában beállítva).
- **Hibernálás (Suspend to disk):** A `qm suspend ID --todisk` lementi a VM RAM-jának tartalmát a lemezre (egy dedikált **State** tárolóra, ami alapból a `vmstatestorage`), így a VM a következő indulásnál onnan folytatja, ahol abbahagyta.
- **Resource Mapping:** A cluster szintjén (a **Datacenter $\rightarrow$ Resource Mappings** alatt) elnevezhetsz egy adott PCI (vagy USB) eszközt. Pl. a `0000:01:00.0` PCI ID a `node1`-en és a `0000:02:00.0` a `node2`-n is `"MyGPU"` néven fut. Így ha a VM HA (magas rendelkezésre állás) miatt átvándorol a másik gépre, a Proxmox a név alapján megtalálja a megfelelő hardvert a célgépen.

**VM kezelés (`qm`):**

- Törlés az összes HA, replikációs és mentési szabállyal együtt:

```bash
qm destroy 300 --purge
```

- Lock levétele (ha áramszünet miatt a VM zárolva maradt):

```bash
qm unlock <vmid>
```

- Lemez áthelyezése másik VM-hez (fizikailag is átnevezi a lemezt a storage-on):

```bash
qm move-disk 300 scsi1 --target-vmid 400 --target-disk scsi3
```

## 10.14 Konfigurációs fájlok (`/etc/pve/qemu-server/<vmid>.conf`)

A VM-ek konfigurációja egy egyszerű kulcs-érték (`OPTION: value`) struktúrájú fájl. Ha a gépről Snapshot (Pillanatkép) készül, az aktuális állapot és a hierarchia a konfigurációs fájl alján, egy `[snapshotnév]` szekcióban (`parent` és `snaptime` megadásával) tárolódik el. A fájl közvetlen kézi szerkesztése lehetséges, de az üzem közbeni változtatásokhoz (**hot-plug**) a GUI vagy a `qm set` parancs használata ajánlott.
