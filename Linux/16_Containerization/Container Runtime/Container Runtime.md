# Konténer futtatókörnyezetek (Container Runtimes)

> **Megjegyzés:** A Dockershim a Kubernetes projekt 1.24-es kiadásától eltávolításra került. További részletekért olvasd el a Dockershim Removal FAQ-ot.

Telepítened kell egy konténer futtatókörnyezetet a klaszter minden egyes node-jára, hogy a Podok ott futni tudjanak. Ez az oldal összefoglalja, mi mindent kell tenni, és leírja a node-ok beállításával kapcsolatos feladatokat.

A Kubernetes 1.37 megköveteli, hogy olyan futtatókörnyezetet használj, amely megfelel a Container Runtime Interface (CRI) szabványnak.

További információért lásd a CRI verziótámogatást.

Ez az oldal áttekintést nyújt arról, hogyan kell használni néhány gyakori konténer futtatókörnyezetet a Kubernetes-szel.

- containerd
- CRI-O
- Docker Engine
- Mirantis Container Runtime

> **Megjegyzés:**
> A v1.24 előtti Kubernetes-kiadások tartalmaztak egy közvetlen integrációt a Docker Engine-nel, egy `dockershim` nevű komponens használatával. Ez a speciális közvetlen integráció már nem része a Kubernetes-nek (ezt az eltávolítást a v1.20-as kiadás részeként jelentették be). Elolvashatod a Check whether Dockershim removal affects you oldalt, hogy megértsd, ez az eltávolítás hogyan érinthet téged. Ha a dockershim használatáról szeretnél migrálni, lásd a Migrating from dockershim oldalt.
>
> Ha a v1.37-től eltérő Kubernetes-verziót futtatsz, nézd meg annak a verziónak a dokumentációját.

## Előfeltételek telepítése és konfigurálása

### Hálózati konfiguráció

Alapértelmezésben a Linux kernel nem engedélyezi, hogy IPv4-csomagok legyenek routolva az interfészek között. A legtöbb Kubernetes klaszter-hálózati implementáció megváltoztatja ezt a beállítást (ha szükséges), de néhány elvárhatja, hogy az adminisztrátor tegye meg helyettük. (Néhány más sysctl paraméter beállítását is elvárhatja, kernelmodulok betöltését stb.; nézd meg a konkrét hálózati implementációd dokumentációját.)

#### IPv4 csomagtovábbítás engedélyezése

Az IPv4 csomagtovábbítás manuális engedélyezéséhez:

```bash
# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

Ellenőrizd, hogy a `net.ipv4.ip_forward` 1-re van állítva:

```bash
sysctl net.ipv4.ip_forward
```

### cgroup illesztőprogramok (cgroup drivers)

Linuxon a control groupokat arra használják, hogy korlátozzák a processzekhez kiosztott erőforrásokat.

Mind a kubeletnek, mind a mögöttes konténer futtatókörnyezetnek együtt kell működnie a control groupokkal, hogy érvényesítse a podok és konténerek erőforrás-kezelését, és beállítsa az olyan erőforrásokat, mint a cpu/memória kérések és limitek. A control groupokkal való együttműködéshez a kubeletnek és a konténer futtatókörnyezetnek egy cgroup illesztőprogramot kell használnia. Kritikus, hogy a kubelet és a konténer futtatókörnyezet ugyanazt a cgroup illesztőprogramot használja, és ugyanúgy legyen konfigurálva.

Két cgroup illesztőprogram érhető el:

- `cgroupfs`
- `systemd`

#### cgroupfs illesztőprogram

A `cgroupfs` illesztőprogram a kubelet alapértelmezett cgroup illesztőprogramja. Amikor a `cgroupfs` illesztőprogramot használják, a kubelet és a konténer futtatókörnyezet közvetlenül a cgroup fájlrendszerrel működik együtt a cgroupok konfigurálásához.

A `cgroupfs` illesztőprogram nem ajánlott, ha a systemd az init rendszer, mert a systemd egyetlen cgroup kezelőt vár el a rendszeren. Továbbá, ha cgroup v2-t használsz, a `systemd` cgroup illesztőprogramot használd a `cgroupfs` helyett.

#### systemd cgroup illesztőprogram

Amikor a systemd-t választják egy Linux disztribúció init rendszereként, az init folyamat létrehoz és fogyaszt egy root control groupot (cgroup), és cgroup kezelőként működik.

A systemd szorosan integrálódik a cgroupokkal, és minden systemd unithez egy cgroupot oszt ki. Ennek eredményeként, ha a systemd-t használod init rendszerként a `cgroupfs` illesztőprogrammal, a rendszer két különböző cgroup kezelőt kap.

Két cgroup kezelő két nézetet eredményez a rendszerben elérhető és használatban lévő erőforrásokról. Bizonyos esetekben azok a node-ok, amelyek `cgroupfs`-t használnak a kubelethez és a konténer futtatókörnyezethez, de systemd-t a többi folyamathoz, instabillá válnak erőforrásnyomás alatt.

Az instabilitás enyhítésének megközelítése az, hogy a systemd-t használjuk cgroup illesztőprogramként a kubelethez és a konténer futtatókörnyezethez, amikor a systemd a választott init rendszer.

A systemd cgroup illesztőprogramként való beállításához szerkeszd a `KubeletConfiguration` `cgroupDriver` opcióját, és állítsd `systemd`-re. Például:

```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
...
cgroupDriver: systemd
```

Ha a systemd-t állítod be cgroup illesztőprogramként a kubelethez, a systemd-t kell beállítanod cgroup illesztőprogramként a konténer futtatókörnyezethez is. A részletekért lásd a konténer futtatókörnyezeted dokumentációját. Például:

- containerd
- CRI-O

A Kubernetes 1.37-ben, a `KubeletCgroupDriverFromCRI` feature gate engedélyezésével és egy olyan konténer futtatókörnyezettel, amely támogatja a `RuntimeConfig` CRI RPC-t, a kubelet automatikusan felismeri a megfelelő cgroup illesztőprogramot a futtatókörnyezetből, és figyelmen kívül hagyja a kubelet konfigurációján belüli `cgroupDriver` beállítást.

A konténer futtatókörnyezetek régebbi verziói (konkrétan a containerd 1.y és az alattiak) azonban nem támogatják a `RuntimeConfig` CRI RPC-t, és nem biztos, hogy helyesen válaszolnak erre a lekérdezésre, így a kubelet visszaesik a saját `--cgroup-driver` flagjében lévő érték használatára.

A Kubernetes 1.38-ban ez a visszaesési viselkedés megszűnik, és a containerd régebbi verziói hibát fognak jelezni az újabb kubeletekkel.

> ⚠️ **Figyelem:**
> Egy klaszterhez csatlakozott node cgroup illesztőprogramjának megváltoztatása érzékeny művelet. Ha a kubelet már létrehozott Podokat az egyik cgroup illesztőprogram szemantikájával, a konténer futtatókörnyezet másik cgroup illesztőprogramra váltása hibákat okozhat, amikor megpróbálja újra létrehozni a Pod sandboxot az ilyen meglévő Podokhoz. A kubelet újraindítása nem biztos, hogy megoldja az ilyen hibákat.
>
> Ha van olyan automatizálásod, amely megvalósíthatóvá teszi, cseréld le a node-ot egy másikra a frissített konfigurációval, vagy telepítsd újra automatizálással.

### CRI verziótámogatás

A konténer futtatókörnyezetednek támogatnia kell a container runtime interface v1-es verzióját.

A Kubernetes a v1.26-tól kezdve csak a CRI API v1-es verziójával működik. Ha egy konténer futtatókörnyezet nem támogatja a v1-es API-t, a kubelet nem regisztrál node-ként.

## Konténer futtatókörnyezetek

> **Megjegyzés:** Ez a szakasz harmadik féltől származó projektekre mutató linkeket tartalmaz, amelyek a Kubernetes által megkövetelt funkcionalitást biztosítanak. A Kubernetes projekt szerzői nem felelősek ezekért a projektekért, amelyek ábécé sorrendben vannak felsorolva. Ha hozzá szeretnél adni egy projektet ehhez a listához, olvasd el a content guide-ot, mielőtt változtatást nyújtanál be. További információ.

### containerd

Ez a szakasz ismerteti a containerd CRI futtatókörnyezetként való használatához szükséges lépéseket.

A containerd rendszerre való telepítéséhez kövesd a getting started with containerd útmutatót. Térj vissza ehhez a lépéshez, miután létrehoztál egy érvényes `config.toml` konfigurációs fájlt.

- Linux
- Windows

Ezt a fájlt a `/etc/containerd/config.toml` elérési úton találod.

Linuxon a containerd alapértelmezett CRI socketje a `/run/containerd/containerd.sock`. Windows-on az alapértelmezett CRI végpont a `npipe://./pipe/containerd-containerd`.

#### A systemd cgroup illesztőprogram konfigurálása

A `systemd` cgroup illesztőprogram használatához a `/etc/containerd/config.toml` fájlban a `runc`-kal állítsd be a következő konfigurációt a containerd verziód alapján:

**Containerd 1.x verziók:**

```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

**Containerd 2.x verziók:**

```toml
[plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc]
  ...
  [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options]
    SystemdCgroup = true
```

A systemd cgroup illesztőprogram ajánlott, ha cgroup v2-t használsz.

> **Megjegyzés:**
> Ha a containerd-t csomagból telepítetted (például RPM vagy .deb), előfordulhat, hogy a CRI integrációs plugin alapértelmezésben le van tiltva.
>
> A CRI támogatást engedélyezned kell ahhoz, hogy a containerd-t a Kubernetes-szel használhasd. Győződj meg arról, hogy a `cri` nincs benne a `disabled_plugins` listában a `/etc/containerd/config.toml` fájlon belül; ha módosítottad azt a fájlt, indítsd újra a containerd-t is.
>
> Ha a kezdeti klaszter-telepítés után vagy egy CNI telepítése után konténer-összeomlási ciklusokat tapasztalsz, a csomaggal szállított containerd konfiguráció inkompatibilis konfigurációs paramétereket tartalmazhat. Fontold meg a containerd konfiguráció visszaállítását a `containerd config default > /etc/containerd/config.toml` paranccsal a getting-started.md-ben leírtak szerint, majd állítsd be a fenti konfigurációs paramétereket ennek megfelelően.
>
> Ha alkalmazod ezt a változtatást, győződj meg arról, hogy újraindítod a containerd-t:

```bash
sudo systemctl restart containerd
```

A Kubernetes v1.28-ban engedélyezheted a cgroup illesztőprogram automatikus felismerését alpha funkcióként. További részletekért lásd a systemd cgroup driver oldalt.

#### A sandbox (pause) kép felülírása

A containerd konfigurációdban felülírhatod a sandbox képet a következő konfiguráció beállításával:

```toml
[plugins."io.containerd.grpc.v1.cri"]
  sandbox_image = "registry.k8s.io/pause:3.10"
```

Lehet, hogy újra kell indítanod a containerd-t is, miután frissítetted a konfigurációs fájlt: `systemctl restart containerd`.

### CRI-O

Ez a szakasz tartalmazza a CRI-O konténer futtatókörnyezetként való telepítéséhez szükséges lépéseket.

A CRI-O telepítéséhez kövesd a CRI-O Install Instructions útmutatót.

#### cgroup illesztőprogram

A CRI-O alapértelmezésben a systemd cgroup illesztőprogramot használja, ami valószínűleg jól működik számodra. A `cgroupfs` cgroup illesztőprogramra való váltáshoz vagy szerkeszd a `/etc/crio/crio.conf` fájlt, vagy helyezz el egy drop-in konfigurációt a `/etc/crio/crio.conf.d/02-cgroup-manager.conf` fájlban, például:

```toml
[crio.runtime]
conmon_cgroup = "pod"
cgroup_manager = "cgroupfs"
```

Meg kell jegyezned a megváltozott `conmon_cgroup`-ot is, amelyet `pod` értékre kell állítani, amikor a CRI-O-t `cgroupfs`-sel használod. Általában szükséges, hogy a kubelet (általában kubeadm-en keresztül történő) és a CRI-O cgroup illesztőprogram-konfigurációja szinkronban legyen.

A Kubernetes v1.28-ban engedélyezheted a cgroup illesztőprogram automatikus felismerését alpha funkcióként. További részletekért lásd a systemd cgroup driver oldalt.

A CRI-O esetében a CRI socket alapértelmezésben a `/var/run/crio/crio.sock`.

#### A sandbox (pause) kép felülírása

A CRI-O konfigurációdban beállíthatod a következő konfigurációs értéket:

```toml
[crio.image]
pause_image="registry.k8s.io/pause:3.10"
```

Ez a konfigurációs opció támogatja az élő konfiguráció-újratöltést a változtatás alkalmazásához: `systemctl reload crio` vagy SIGHUP küldése a crio folyamatnak.

### Docker Engine

> **Megjegyzés:**
> Ezek az utasítások feltételezik, hogy a `cri-dockerd` adaptert használod a Docker Engine Kubernetes-szel való integrálásához.

1. Minden node-on telepítsd a Docker-t a Linux disztribúciódhoz az Install Docker Engine útmutató szerint.
2. Telepítsd a `cri-dockerd`-t, követve a dokumentáció telepítési szakaszában lévő utasításokat.

A `cri-dockerd` esetében a CRI socket alapértelmezésben a `/run/cri-dockerd.sock`.

### Mirantis Container Runtime

A Mirantis Container Runtime (MCR) egy kereskedelmi forgalomban kapható konténer futtatókörnyezet, amely korábban Docker Enterprise Edition néven volt ismert.

A Mirantis Container Runtime-ot a Kubernetes-szel a nyílt forráskódú `cri-dockerd` komponens használatával használhatod, amely az MCR-hez van csomagolva.

Ha többet szeretnél megtudni a Mirantis Container Runtime telepítéséről, látogass el az MCR Deployment Guide oldalra.

Ellenőrizd a `cri-docker.socket` nevű systemd unitot, hogy megtudd a CRI socket elérési útját.

#### A sandbox (pause) kép felülírása

A `cri-dockerd` adapter elfogad egy parancssori argumentumot annak meghatározásához, hogy melyik konténerképet használja a Pod infrastruktúra-konténerként („pause image”). A használandó parancssori argumentum a `--pod-infra-container-image`.

## Forrás
-[https://kubernetes.io/docs/setup/production-environment/container-runtimes/]

# Mi az a konténer futtatókörnyezet (container runtime)?

A konténer futtatókörnyezetek speciális szoftverkomponensek, amelyek konténereket hajtanak végre és kezelnek a hosztrendszereken. Ezek a futtatókörnyezetek könnyűsúlyú, hordozható környezetek biztosításával garantálják a környezeti konzisztenciát, amelyek magukba foglalják az alkalmazásokat és azok függőségeit. A konténerizáció szabványosítása megszünteti a gyakori „az én gépemen működik” problémát, és lehetővé teszi, hogy az alkalmazások különböző környezetekben fussanak, miközben megőrzik a hoszt operációs rendszertől való elkülönülést. Mióta a Docker 2013-as indulásával széles körű elterjedtséget nyert, a technológia megalapozta a mai felhőnatív architektúrákat.

## Hogyan működnek a konténer futtatókörnyezetek?

A konténer futtatókörnyezetek úgy működnek, hogy izolált környezeteket hoznak létre az alkalmazások számára a hoszt operációs rendszer beépített funkcióival. A virtuális gépekkel ellentétben – amelyek virtualizálják a hardvert, és minden példányhoz teljes operációs rendszert igényelnek – a konténerek az operációs rendszer szintjén virtualizálva növelik a hatékonyságot, és megosztják a hoszt kerneljét. A kernelmegosztás ideálissá teszi a konténereket mikroszolgáltatások telepítéséhez és skálázható, nagy teljesítményű alkalmazások eléréséhez.

Amikor elindítasz egy konténert, a futtatókörnyezet letöli a konténerképet, beállítja a szükséges namespaces-eket és cgroupokat az izolációhoz és az erőforrás-szabályozáshoz, majd elindítja az alkalmazásfolyamatot abban a környezetben. A különböző platformok és eszközök közötti kompatibilitás biztosítása érdekében a legtöbb modern futtatókörnyezet követi az Open Container Initiative (OCI) szabványokat. A Docker és más iparági vezetők ezeket a szabványokat az ökoszisztéma konzisztenciájának fenntartása érdekében hozták létre. A végrehajtás olyan technológiákra támaszkodik, mint a namespaces, amelyek körülbelül 2002 óta részei a Linux kernelnek.

## A konténer futtatókörnyezetek alapvető funkciói és felelősségei

### Konténerek végrehajtása

A futtatókörnyezetek négy különálló fázison keresztül kezelik a végrehajtást:

- **Konténer létrehozása:** Konténerkörnyezetek létrehozása a megadott képek alapján, izolált namespaces-ek és erőforrás-kiosztások beállítása.
- **Alkalmazás indítása:** Konténerizált alkalmazások indítása és a szükséges futásidejű konfigurációk létrehozása.
- **Életciklus-kezelés:** Konténerek állapotának figyelése, a sikertelen folyamatok automatikus újraindítása és a skálázási műveletek kezelése.
- **Erőforrás-takarítás:** Konténerek szabályos leállítása és a rendszererőforrások felszabadítása, miután a folyamatok befejeződtek.

### Interakció a hoszt operációs rendszerrel

A konténerizoláció biztosítja, hogy a konténerizált alkalmazások ne zavarják a hosztrendszert vagy más konténereket. A futtatókörnyezetek az izolációt olyan Linux kernel-funkciók kihasználásával érik el, mint a namespaces a folyamatizolációhoz és a cgroupok az erőforrás-kezeléshez. A mögöttes architektúra fenntartja a rendszer stabilitását, miközben lehetővé teszi, hogy több konténer biztonságosan fusson ugyanazon a hoszton. Az izoláció fenntartása létfontosságú, mivel a vállalatok gyakran a biztonságot nevezik meg legfőbb kihívásukként.

### Erőforrás-kiosztás és -kezelés

A hatékony erőforrás-kezelés megakadályozza, hogy bármely egyetlen konténer monopolizálja a rendszererőforrásokat azáltal, hogy limiteket érvényesít a CPU-ra, a memóriára és az I/O-ra. A szabályozott erőforrás-elosztás lehetővé teszi, hogy több alkalmazás hatékonyan fusson megosztott infrastruktúrán, ideálissá téve a konténerizációt a modern többbérlős felhőkörnyezetekhez.

## Konténer futtatókörnyezetek vs. konténer motorok (container engines)

A konténer motorok átfogó konténerkezelési platformokat biztosítanak, míg a konténer futtatókörnyezetek kifejezetten a konténerek végrehajtására összpontosítanak. A motorok szélesebb felelősségi köröket kezelnek, mint például a képek építése, terjesztése, hálózatkezelés és orchestráció. A futtatókörnyezetek az egyes konténerfolyamatok indításának és kezelésének alapvető feladatát látják el.

A Docker nem kizárólag egy konténer futtatókörnyezet – ez egy teljes konténerizációs platform, amely tartalmaz egy futtatókörnyezet-komponenst is. A Docker Engine több eszközt kombinál a képek építéséhez, a hálózatok kezeléséhez és a konténerek orchestrálásához, és a containerd-et használja mögöttes futtatókörnyezeti motorként. A különbség ismerete tisztázza a konténerökoszisztéma különböző rétegeit.

> **Mi az a konténer motor (container engine)?**
> A konténer motor egy szoftvereszköz, amely automatizálja az alkalmazások izolált, könnyűsúlyú környezetekben – úgynevezett konténerekben – való futtatásának folyamatát.

## A konténer futtatókörnyezetek típusai

A konténer futtatókörnyezetek általában három fő kategóriába sorolhatók funkcionalitásuk és absztrakciós szintjük alapján:

| Futtatókörnyezet kategória | Absztrakciós szint | Alapvető funkcionalitás | Legjobb felhasználás |
| --- | --- | --- | --- |
| Alacsony szintű | Minimális / közvetlen | Közvetlenül az OS kernellel működik együtt (namespaces/cgroups) a konténervégrehajtáshoz. | Finomhangolt erőforrás-szabályozás és nagy teljesítményű végrehajtás. |
| Magas szintű | Mérsékelt / kezelt | Képkezelést, hálózatkezelést és API-kat ad hozzá az alapvető végrehajtás tetejére. | Átfogó kezelés és szabványos fejlesztői környezetek. |
| Specializált | Kontextus-specifikus | Niche platformokra vagy konkrét orchestrátorokra, például a Kubernetesre szabva. | Edge computing, IoT és optimalizált K8s telepítések. |

## Népszerű konténer futtatókörnyezet eszközök

A megfelelő konténer futtatókörnyezet kiválasztása a konkrét felhasználási esetedtől, biztonsági követelményeidtől és infrastruktúra-igényeidtől függ. Ezek az eszközök képviselik a termelési környezetekben legszélesebb körben elfogadott megoldásokat:

| Eszköz | Absztrakciós szint | Infrastruktúra-igények | Biztonsági profil | Legjobb felhasználási eset |
| --- | --- | --- | --- | --- |
| Docker, Podman | Magas szintű | Átfogó kezelés fejlesztői/termelési munkafolyamatokhoz. | Docker: Integrált javítások. Podman: Rootless biztonság. | Helyi fejlesztés és CI/CD pipeline-ok. |
| containerd, CRI-O | Kezelési réteg | Iparági szabványú backendek K8s-hez és Dockerhez. | Biztonságos tervezés, valamint rendszeres sebezhetőség-keresés. | Termelési szintű K8s és Docker backendek. |
| runC | Alacsony szintű | Olyan forgatókönyvek, amelyek közvetlen interakciót igényelnek a konténerfolyamatokkal. | Alapvető, mivel a hoszt OS biztonsági funkcióira támaszkodik. | Nagy kontrollt és testreszabást igénylő forgatókönyvek. |
| CRI-O (K8s-specifikus) | Specializált | Kubernetesre és Edge computingra szabva. | Kubernetes-specifikus fenyegetésvédelemmel tervezve. | K8s telepítések és IoT/Edge alkalmazások. |

- **Fejlesztői környezetekhez:** A Docker biztosítja a legátfogóbb eszközkészletet integrált build, szállítási és futtatókörnyezeti képességekkel.
- **Termelési Kuberneteshez:** A CRI-O optimalizált teljesítményt és biztonságot kínál, kifejezetten Kubernetes telepítésekhez tervezve.
- **Biztonságra összpontosító telepítésekhez:** A Podman daemon nélküli architektúrája csökkenti a támadási felületet, miközben megőrzi a Docker-kompatibilitást.
- **Minimális többletterheléshez:** A containerd és a runC könnyűsúlyú, fókuszált futtatókörnyezeti funkcionalitást biztosít további platformfunkciók nélkül.

A széles választék a konténerizáción belüli fejlődő, sokszínű igényeket tükrözi. A nyílt forráskódú közösségek ezeket az igényeket a sokféle telepítési stratégia és üzemeltetési követelmény számára szabott megoldások építésével elégítik ki. A szabványok tovább fejlődnek, amint azt a 2024 februári OCI Image and Distribution Specifications 1.1.0 kiadása is mutatja.

Míg az egyes eszközök konkrét előnyöket nyújtanak, a több rendszer kezelése gyakran láthatósági résekhez vezet. A Wiz átfogó, egységes felhőbiztonsági platformot kínál, amely a teljes környezetedben összevonja a konténer- és Kubernetes-biztonságot, felülmúlva az önálló futtatókörnyezeteket azáltal, hogy a futásidejű adatokat a felhőinfrastruktúra kockázatával korrelálja.

## Milyen biztonsági vonatkozásai vannak a konténer futtatókörnyezeteknek?

A konténerbiztonság olyan egyedi kockázatokkal foglalkozik, amelyek nem léteznek a hagyományos infrastruktúrában. A statikus virtuális gépekkel ellentétben a konténerek megosztják a kernel erőforrásait, és dinamikus környezetekben működnek, új támadási vektorokat hozva létre, amelyek speciális védelmi stratégiákat igényelnek.

**A biztonsági kihívások közé tartoznak:**

- **Nagyobb támadási felület:** A konténerizált környezetek jelentősen nagyobb támadási felületet jelentenek, mint a hagyományos munkaterhelések, a konténerek és azok mögöttes képeinek pusztán nagy száma miatt.
- **Megosztott kernelarchitektúra:** Mivel a konténerek megosztják a hoszt kerneljét, a hoszt önmagában való biztosítása nem elegendő. Egy sebezhetőség a konténerbeállításban az egész hosztrendszert kompromittálhatja. Létfontosságú biztonságos, izolált konfigurációkat fenntartani minden konténerhez a teljes rendszer integritásának és biztonságának biztosítása érdekében.
- **Dinamikus környezetek:** A konténerizált munkaterhelések rendkívül dinamikus természete láthatósági kihívásokat jelent, megnehezítve a hagyományos monitorozóeszközök számára a nyomon követést.

### Bevált gyakorlatok biztonságos konténertelepítéshez

A holisztikus biztonsági stratégia elfogadása az egyetlen módja a konténertelepítések védelmének. Egy hatékony stratégia mindent felölel a képlétrehozástól a futásidejű monitorozásig, beleértve:

- **Képek biztonságossá tétele:** Konténerképek vizsgálata a sebezhetőségek azonosításához és eltávolításához a telepítés előtt.
- **Regisztrumok biztonságossá tétele:** Hozzáférés-vezérlés és képek aláírása a konténerregisztrumokban.
- **Telepítés biztonságossá tétele:** A hoszt alapoperációs rendszer megerősítése, robusztus tűzfalszabályok bevezetése, és olyan konténer-orchestrátorok használata, amelyek biztonságos API-végpontokat biztosítanak és szerepalapú hozzáférés-vezérlést (RBAC) érvényesítenek. A konténerek konfigurálása a minimálisan szükséges jogosultsággal való futáshoz. Kerüld a konténerek root vagy magas jogosultsággal való futtatását, és biztosítsd, hogy a képek ne tartalmazzanak beágyazott titkokat.
- **Konténertevékenység monitorozása:** Granuláris monitorozás bevezetése a konténerek állapotába és viselkedésébe való láthatóság érdekében.

Szilárdan kell értened a konténerizolációt, a hálózatbiztonságot és a konténerkörnyezetekhez kapcsolódó sebezhetőségeket egy erős, átfogó biztonsági pozíció kiépítéséhez. A mélyreható feltáráshoz merülj el a részletekben a konténer futtatókörnyezet biztonsági blogbejegyzésünkkel.

## Wiz konténerekhez és Kuberneteshez

A biztonságos konténertelepítés megköveteli mind a futásidejű alapok, mind a felmerülő biztonsági kihívások megértését. Ahogy a konténerizált alkalmazások küldetéskritikus infrastruktúrává válnak, átfogó láthatóságra lesz szükséged a konténerkörnyezeteiden keresztül – a fejlesztéstől a termelésig.

A Wiz átfogó konténer- és Kubernetes-biztonsági megoldást kínál azoknak a csapatoknak, amelyek javítani szeretnék biztonsági pozíciójukat. Eszközkészletünk holisztikus megközelítést nyújt a konténerek és a Kubernetes biztonságossá tételéhez, biztosítva, hogy alkalmazásaid védve maradjanak a build időtől a valós időig. A Wiz biztonságos konténerképeket is kínál a WizOS-szal, amelyet folyamatosan közel nulla CVE-vel tartunk karban, hogy csökkentsük az örökölt sebezhetőségeket és az ellátási lánc kockázatát.

## Forrás
-[https://www.wiz.io/academy/container-security/container-runtimes]
-[https://opensource.com/article/21/9/container-runtimes]
