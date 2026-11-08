## 1. Mi az a Docker, és milyen problémákat old meg?

### A konténer és a Container Repository fogalma

- **Konténer (Container):** Egy izolált futtatókörnyezet, amely tartalmazza magát az alkalmazást, annak pontos verziójú függőségeit, az indító szkripteket és a szükséges konfigurációt. Hordozható artefaktumként viselkedik, amely ugyanúgy fut bármelyik fejlesztői gépen vagy szerveren.
- **Container Repository (Registry):** Speciális tárolóhely a konténer-képfájlok (**image**-ek) számára, ahová a csomagokat fel lehet tölteni (**push**) és le lehet tölteni (**pull**).
  - **Publikus repository (Docker Hub):** Az alapértelmezett nyilvános tároló, ahol több százezer alkalmazás érhető el. Léteznek hivatalos képfájlok (**Official Images**) (pl. PostgreSQL, Redis, Jenkins), amelyeket a készítők és a Docker közösen tartanak karban, valamint közösségi/egyéni image-ek.
  - **Privát repository:** Vállalati környezetben használt zárt tárolók (pl. AWS ECR, GitLab Container Registry, Harbor), ahol a cégek a saját, belső fejlesztésű alkalmazásaik image-eit tárolják.

### Hogyan javítja a fejlesztési folyamatot (Development)?

- **Konténerek előtt:** Ha egy fejlesztői csapat például egy Node.js/JavaScript alkalmazáson dolgozott, amely PostgreSQL-t és Redist használt, minden egyes fejlesztőnek manuálisan kellett telepítenie a binárisokat a saját operációs rendszerére (Windows, macOS, különféle Linux disztribúciók). A telepítés lépései operációs rendszerenként eltértek, a többlépcsős konfiguráció miatt magas volt a hibalehetőség, és nehéz volt ugyanannak a szolgáltatásnak két eltérő verzióját egymás mellett futtatni.
- **Konténerekkel:** A szolgáltatásokat nem kell közvetlenül a gazda (host) operációs rendszerre telepíteni. Minden szolgáltatás saját, izolált Linux-alapú konténerben fut. Operációs rendszertől függetlenül ugyanaz az egyetlen Docker parancs letölti és elindítja a megfelelő verziójú szolgáltatást. Könnyedén futtatható párhuzamosan például egy PostgreSQL 9.6 és egy PostgreSQL 10.10 anélkül, hogy összeakadnának a könyvtáraik.

### Hogyan javítja a telepítési és üzemeltetési folyamatot (Deployment)?

- **Hagyományos folyamat:** A fejlesztők (Dev) előállították a binárist (pl. egy `.jar` fájlt), és írtak mellé egy szöveges dokumentációt az üzemeltetőknek (Ops), hogy milyen adatbázist, függőségeket és konfigurációt kell beállítani a szerveren. Ez gyakran vezetett félreértésekhez, hiányos leírásokhoz, verzióütközésekhez a szerveren és hosszas oda-vissza kommunikációhoz ("az én gépemen pedig működött").
- **Konténerizált folyamat:** A Dev és az Ops közösen csomagolja be az alkalmazást és a konfigurációt egyetlen Docker image-be. A célszerveren nem kell függőségeket telepíteni és környezetet konfigurálni: mindössze a **Docker futtatókörnyezetet (Docker Runtime)** kell egyszer feltelepíteni, utána bármelyik konténer egyetlen paranccsal letölthető és indítható.

## 2. Hogyan épül fel egy konténer? (Image vs. Container és a rétegek)

### Az Image-ek rétegzett felépítése (Layers)

Technikailag a konténerek egymásra épülő képfájl-rétegekből (**image layers**) állnak:

- **Linux Base Image (Alapréteg):** Legalul egy minimalista Linux disztribúció fájlrendszere található. Gyakran az **Alpine Linux**-ot használják, mivel rendkívül kicsi (néhány megabájt), így a teljes konténer mérete is kicsi marad.
- **Köztes rétegek (Intermediate Images):** A futtatáshoz szükséges könyvtárak, függőségek, futtatókörnyezetek.
- **Alkalmazás réteg és konfiguráció:** Legfelül helyezkedik el maga az alkalmazás és a konfigurációs adatok.

**Miért előnyös a rétegződés? (Gyakorlati példa):**
Amikor a terminálban először letöltöd a `postgres:9.6` képfájlt, a Docker különálló rétegekben (egyedi hash azonosítókkal jelölve) tölti le az image-et. Ha ezután elindítod a `postgres:10.10` verziót is, a Docker felismeri, hogy az alaprétegek megegyeznek a két verzió között. A közös rétegeket nem tölti le újra és nem tárolja duplán a lemezen, kizárólag a megváltozott felső rétegeket húzza le. Ez jelentős sávszélességet, időt és tárhelyet takarít meg.

### Image vs. Container – Mi a pontos különbség?

A két fogalmat gyakran keverik, pedig éles határ van köztük:

| Fogalom | Meghatározás | Állapot | Analógia (Programozás / OS) |
| :--- | :--- | :--- | :--- |
| **Docker Image** | A becsomagolt alkalmazás, függőségek és alapkonfiguráció. Egy statikus, csak olvasható (**read-only**) sablon/artefaktum, amely a Docker Hubon vagy a helyi lemezen pihen. | Nem fut (statikus) | Osztály (Class) vagy futtatható bináris fájl a lemezen |
| **Docker Container** | Az Image futó példánya. Amikor elindítasz egy image-et, a Docker létrehoz köré egy izolált futtatókörnyezetet saját virtuális, írható (**read-write**) fájlrendszerrel, környezeti változókkal és hálózati portokkal. | Futó vagy leállított folyamat | Példányosított objektum (Instance) vagy futó folyamat (Process) a memóriában |

**Kiegészítés (Hogyan működik ez a motorháztető alatt?):**
A Docker az úgynevezett **Union File System** (pl. OverlayFS / overlay2) technológiát használja. Az Image összes rétege **immutable** (megváltoztathatatlan, csak olvasható). Amikor a `docker run` paranccsal létrehozol egy konténert, a Docker egy vékony írható réteget (**Container Layer**) helyez a legfelső image-réteg tetejére. Minden fájl, amit a futó konténer létrehoz vagy módosít (pl. logok, ideiglenes fájlok), ebbe a felső rétegbe kerül (**Copy-on-Write** mechanizmus), így ugyanabból az alap image-ből tucatnyi konténert indíthatsz anélkül, hogy egymás fájljait felülírnák.

## 3. Docker vs. Virtuális Gépek (VM)

Ahhoz, hogy érthető legyen a különbség a Docker és a hagyományos virtualizáció (pl. Oracle VirtualBox, VMware, KVM/Proxmox VM) között, az operációs rendszer két fő rétegét kell megvizsgálni:

- **OS Kernel (Rendszermag):** Ez a réteg kommunikál közvetlenül a fizikai hardverrel (CPU, memória, lemez, hálózati kártya) és osztja el az erőforrásokat.
- **Applications Layer (Alkalmazási réteg / User Space):** A kernel felett futó fájlrendszer, rendszerkönyvtárak, csomagkezelők, grafikus felület és a felhasználói alkalmazások. (Például az Ubuntu, a Debian és a Fedora mind ugyanazt a Linux kernelt használják, csak a felette lévő alkalmazási rétegük tér el.)

### Mit virtualizál a Docker és mit a VM?

| Szempont | Docker Konténer | Virtuális Gép (VM) |
| :--- | :--- | :--- |
| **Mit virtualizál?** | Csak az **Alkalmazási réteget** (User Space). Nincs saját rendszermagja, a gazdagép (Host OS) Linux kernelét használja közösen a többi konténerrel. | A **teljes operációs rendszert** (saját Guest Kernel + saját Alkalmazási réteg). Nem a host kernelét használja, hanem sajátot bootol. |
| **Képfájl mérete** | Rendkívül kicsi (általában néhány tíz vagy száz megabájt). | Nagy (több gigabájt, mivel a teljes OS-t tartalmazza). |
| **Indítási sebesség** | Másodpercek (vagy ezredmásodpercek) alatt elindul, mivel a host kernel már fut, csak az alkalmazás folyamatát kell elindítani. | Percekig tarthat, mert minden indításkor nulláról kell bootolnia a vendég operációs rendszer kernelét. |
| **Kompatibilitás** | A Linux konténereknek Linux kernelre van szükségük. Régebbi Windows/macOS rendszereken nem tudtak natívan futni segédeszköz nélkül. | Teljes keresztkompatibilitás: bármilyen Host OS-en futtatható bármilyen más architektúrájú/típusú Guest OS (pl. Linux VM Windowson). |

**Kiegészítés (Linux Kernel technológiák és modern Windows/macOS működés):**

- **Linuxon (Natív működés):** A Docker valójában nem egy klasszikus hypervisor, hanem a Linux kernel két beépített funkciójára épül (hasonlóan az LXC konténerekhez):
  - **Namespaces (Névtérszeparáció):** Izolálja a konténert, hogy csak a saját folyamatait (`pid`), saját hálózati interfészét (`net`), saját csatolási pontjait (`mnt`) és saját hostnevét (`uts`) lássa.
  - **cgroups (Control Groups):** Korlátozza és méri, hogy egy adott konténer mennyi fizikai erőforrást (CPU, RAM, I/O) használhat fel a gazdagépen.
- **Windowson és macOS-en:** Mivel a Windows (NT kernel) és a macOS (XNU/Darwin kernel) nem Linux kernelt használ, a Docker Desktop a háttérben automatikusan elindít egy pehelysúlyú Linux virtuális gépet (modern Windowson a **WSL 2** alrendszert, macOS-en a natív **Virtualization.framework**-öt), és a Linux konténerek ennek a rejtett mini-VM-nek a kernelén futnak.

## 4. A Docker telepítése és környezeti előfeltételek

A videó két kiadást különít el:

- **Docker Community Edition (CE):** Ingyenes, nyíltabb közösségi verzió, amely tökéletes egyéni fejlesztéshez, tanuláshoz és a legtöbb alapinfrastruktúrához.
- **Docker Enterprise Edition (EE):** Nagyvállalati környezetre szánt, fizetős támogatással és extra menedzsment funkciókkal rendelkező kiadás.

### Telepítés Windows és macOS rendszerekre

**Előfeltételek:** 64 bites operációs rendszer, legalább 4 GB RAM, és a hardveres virtualizáció (Intel VT-x / AMD-V) engedélyezése a BIOS/UEFI-ben (Windowson a Feladatkezelő $\rightarrow$ Teljesítmény $\rightarrow$ CPU fülön ellenőrizhető).

- **Docker Desktop:** Mindkét rendszerre egyetlen grafikus telepítőcsomag érhető el (Stable csatorna), amely egyben tartalmazza a Docker Engine-t (háttérszolgáltatás), a Docker CLI klienst (parancssor) és a Docker Compose-t.
  > **Fontos megjegyzés többfelhasználós gépekhez:** Ha egy gépen több felhasználói fiók van bejelentkezve egyszerre, a Docker Desktop párhuzamos futtatása ütközést okozhat; fiókváltás előtt érdemes kilépni a Dockerből az első fiókban.
- **Docker Toolbox (Legacy rendszerekre):** A videó kitér a Windows 10 előtti és régi macOS rendszerekre, amelyek nem támogatták a modern beépített virtualizációt. Erre szolgált a Docker Toolbox, amely egy Oracle VirtualBox VM-et telepített a gép és a Docker közé.
  > *(Kiegészítés: A Docker Toolbox ma már hivatalosan kivezetett/elavult technológia, modern rendszereken kizárólag a WSL 2 alapú Docker Desktopot vagy natív Linuxot használunk.)*

### Telepítés Linux disztribúciókra

Linuxon három telepítési mód létezik:

1. **A Docker hivatalos csomagtárolójának (Repository) beállítása (Ajánlott):** Könnyen frissíthető és karbantartható a rendszer csomagkezelőjével (`apt`, `dnf`).
2. **Manuális csomagtelepítés (`.deb` / `.rpm`):** Csak akkor érdemes használni, ha a gépnek nincs internetkapcsolata (**air-gapped**), mert a frissítéseket kézzel kell kezelni.
3. **Automatizált teszt szkript (`get.docker.com`):** Kizárólag gyors tesztelésre való, éles vagy fejlesztői környezetbe nem ajánlott.

### A Repository-alapú telepítés lépései és parancsai Linuxon

**1. Csomaglista frissítése és a Docker GPG kulcs + Repository hozzáadása (Ubuntu/Debian példa):**

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
# A hivatalos GPG kulcs és a stable repository hozzáadása után:
sudo apt-get update
```

(Fedora rendszeren ugyanez a `dnf-plugins-core` telepítésével és a `sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo` paranccsal történik.)

**2. A legfrissebb Docker CE telepítése:**

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

**3. Elérhető verziók listázása és egy konkrét verzió telepítése (Éles/Production szervereken fontos!):**
Éles környezetben nem mindig a legfrissebb (`latest`), hanem egy konkrétan letesztelt Docker verziót szokás telepíteni.

```bash
# Elérhető verziók lekérdezése a beállított repository-ból (Ubuntu/Debian):
apt-cache madison docker-ce

# Konkrét verzió telepítése:
sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
```

**4. A telepítés ellenőrzése a `hello-world` tesztkonténerrel:**

```bash
sudo docker run hello-world
```

**Magyarázat:** Ez a parancs ellenőrzi, hogy fut-e a Docker démon. Mivel a `hello-world` image még nincs a gépen, letölti a Docker Hubról, elindítja egy konténerben, kiírja a megerősítő üzenetet a terminálba, majd a konténer le is áll.

**Kiegészítés (Két fontos Linuxos lépés, amit érdemes megtenni telepítés után):**

- **Szolgáltatás indítása:** Sok disztribúción (pl. Fedora/RHEL) a telepítés után a szolgáltatást külön engedélyezni és indítani kell:

```bash
sudo systemctl enable --now docker
```

- **Futtatás `sudo` nélkül:** Alapértelmezésben a Docker daemon egy `root` jogosultságú Unix socketen figyel, így minden parancs elé `sudo` kellene. Hogy a saját felhasználóddal közvetlenül tudd futtatni a `docker` parancsokat, add hozzá magad a `docker` csoporthoz (utána jelentkezz ki és vissza):

```bash
sudo usermod -aG docker $USER
```

## 5. Alapvető Docker parancsok és Hálózati Port Binding

Ebben a szekcióban a videó a PostgreSQL és a Redis képfájlokon keresztül mutatja be a konténerek teljes életciklusát.

### Képfájlok (Images) letöltése és listázása

**`docker pull`**
Letölt egy képfájlt a Docker Hubról (vagy más megadott registry-ből) a helyi gépre, anélkül, hogy elindítaná azt.

```bash
# Alapértelmezett (latest) verzió letöltése:
docker pull redis

# Konkrét verzió (tag) letöltése kettősponttal elválasztva:
docker pull postgres:9.6
```

**Hogyan működik a Tag (verziójelölés)?** A `név:tag` formátumban a kettőspont utáni rész határozza meg a verziót (pl. `redis:4.0`, `postgres:10.10`). Ha nem adsz meg taget, a Docker automatikusan a `:latest` címkéjű image-et húzza le.

**`docker images`**
Kilistázza az összes olyan Docker képfájlt, amely már le van töltve (vagy fel van építve) a helyi gépedre.

```bash
docker images
```

**Mit mutat a kimenet?**
- `REPOSITORY`: Az image neve (pl. `redis`, `postgres`).
- `TAG`: A verzió (pl. `latest`, `9.6`).
- `IMAGE ID`: Az image egyedi, 12 karakteres rövidített hash azonosítója.
- `CREATED`: Mikor készítették az adott image-et.
- `SIZE`: Mennyi helyet foglal a virtuális fájlrendszerrel együtt.

### Konténerek indítása, listázása és leállítása

**`docker run` (Előtérben / Attached módban)**
Létrehoz egy új konténert a megadott image-ből, és azonnal elindítja azt. Ha az image még nincs meg helyben, a `docker run` automatikusan végrehajt egy `docker pull`-t a háttérben, majd utána indítja el a konténert.

```bash
docker run redis
```

**Mi történik ilyenkor?** A konténer **Attached** (csatolt / előtérbeli) módban indul el. A terminálod lefoglalódik, közvetlenül látod benne a Redis szerver kimenetét (logjait). Ha ebben az ablakban nyomsz egy `Ctrl + C`-t (vagy bezárod a terminált), a folyamat megszakad, és a konténer leáll.

**`docker run -d` (Háttérben / Detached módban)**
A `-d` (`--detach`) kapcsolóval a konténer a háttérben indul el, így a terminálod azonnal felszabadul további parancsok kiadására.

```bash
docker run -d redis
```

**Mit ad vissza?** Csak egyetlen hosszú karakterláncot ír ki a képernyőre (pl. `838a4f9c...`), ami az újonnan elindított konténer egyedi azonosítója (**Container ID**).

**`docker ps`**
Kilistázza az aktuálisan futó konténereket.

```bash
docker ps
```

**Mit mutat a kimenet?**
- `CONTAINER ID`: A konténer egyedi azonosítója.
- `IMAGE`: Melyik képfájlból indult (pl. `redis:4.0`).
- `COMMAND`: Milyen belső indítóparancsot (entrypoint/cmd) futtat a konténer.
- `CREATED` és `STATUS`: Mikor jött létre, és mióta fut (pl. `Up 2 minutes`).
- `PORTS`: Milyen portokon figyel a konténer, és van-e port-összerendelés a gazdagéppel.
- `NAMES`: A konténer neve (ha nem adunk meg sajátot, a Docker generál egy véletlenszerű nevet, pl. `wizardly_visvesvaraya`).

**`docker ps -a`**
Kilistázza az összes konténert, beleértve a jelenleg futókat és a már leállított (**Exited**) konténereket is.

```bash
docker ps -a
```

**Mikor hasznos?** Ha tegnap leállítottál egy konténert, ma a sima `docker ps` üres listát ad. A `docker ps -a` paranccsal viszont megtalálod a leállított konténer ID-ját, így újra tudod indítani.

**`docker stop` és `docker start`**
Egy már létező konténer szabályos leállítására és újraindítására szolgálnak. Nem kell a teljes, 64 karakteres Container ID-t bemásolni, elég az első néhány egyedi karakterét (vagy a konténer nevét) megadni.

```bash
# Futó konténer leállítása (pl. a 838a kezdetű ID-val):
docker stop 838a

# Leállított konténer újraindítása:
docker start 838a
```

### Port Binding (-p) – Hogyan érjük el a konténert kívülről?

**A probléma a videóból**
Ha elindítasz két Redis konténert (`docker run -d redis` és `docker run -d redis:4.0`), a `docker ps` kimenetében a `PORTS` oszlopban mindkettőnél ugyanazt látod: `6379/tcp`.
Miért nem ütközik össze a két konténer, ha mindkettő a 6379-es porton figyel? Mert minden konténer saját, izolált virtuális hálózattal (saját IP-címmel és porttérrel) rendelkezik.
**Hogyan csatlakozik hozzájuk az alkalmazásunk a laptopról?** Sehogy! Alapértelmezésben a konténer portja kívülről (a gazdagép `localhost` címéről) nem érhető el, amíg nem hozunk létre egy **Port Binding**ot (port-összerendelést / port továbbítást) a gazdagép és a konténer között.

**A Port Binding működése**
A gazdagép (**Host**) egy szabad portját hozzárendeljük a konténer belső portjához.
- A gazdagépen nem lehet két konténert ugyanarra a host portra kötni (port ütközés: `Bind for 0.0.0.0:6000 failed: port is already allocated`).
- Viszont két különböző host portot (pl. 6000 és 6001) gond nélkül irányíthatunk két különböző konténer azonos belső portjára (6379).

```
[ Gazdagép (Host) ]                       [ Docker Izolált Hálózat ]
localhost:6000  ──────(Port Binding)────► [ Konténer 1 (redis:latest) ] -> Belső port: 6379
localhost:6001  ──────(Port Binding)────► [ Konténer 2 (redis:4.0)    ] -> Belső port: 6379
```

**`docker run -p`**
A `-p` (`--publish`) kapcsolóval hozzuk létre a port-összerendelést.
**Szintaxis (nagyon fontos a sorrend!):** `-p <HOST_PORT>:<CONTAINER_PORT>`

```bash
# 1. Az újabb Redis indítása a háttérben: a gép 6000-es portját kötjük a konténer 6379-es portjához
docker run -d -p 6000:6379 redis

# 2. A régebbi (4.0-s) Redis indítása: a gép 6001-es portját kötjük a konténer 6379-es portjához
docker run -d -p 6001:6379 redis:4.0
```

**Mit látunk ezután a `docker ps` kimenetében?**
- `0.0.0.0:6000->6379/tcp` (Az első Redis elérhető a `localhost:6000` címen)
- `0.0.0.0:6001->6379/tcp` (A második Redis elérhető a `localhost:6001` címen)

**Kiegészítés (Hálózati és biztonsági részletek a `-p` kapcsolóhoz):**
Amikor kiadod a `-p 6000:6379` parancsot, a Docker a Linux hoston (iptables / nftables DNAT szabályokkal és a `docker-proxy` folyamattal) minden hálózati interfészen (`0.0.0.0`) megnyitja a 6000-es portot! Ez azt jelenti, hogy ha a géped egy helyi hálózaton van, és nincs tűzfalad, más gépek is elérhetik a Redis konténeredet a géped IP-címén keresztül.
Ha azt szeretnéd, hogy a konténer kizárólag a saját gépedről (`localhost`) legyen elérhető, megadhatod a loopback IP-címet is a kapcsolóban:

```bash
docker run -d -p 127.0.0.1:6000:6379 redis
```

## 6. Hibakeresés (Debugging) és Életciklus-kezelés

Amikor a konténerek `-d` (detached) módban a háttérben futnak, nem látjuk a kimenetüket a terminálban. Ha egy alkalmazás nem tud csatlakozni az adatbázishoz vagy hibára fut, a következő parancsokkal deríthetjük ki az okát.

### Konténerek elnevezése (`--name`)

Ahogy a videóban is látszott, amikor többször futtatunk `docker run` parancsokat különböző opciókkal, rengeteg véletlenszerű nevű konténer jön létre, és nehéz fejben tartani, melyik ID vagy random név melyik verzióhoz tartozik. A `--name` kapcsolóval egyedi, beszédes nevet adhatunk a konténernek:

```bash
# Régebbi verzió indítása beszédes névvel:
docker run -d -p 6001:6379 --name redis-older redis:4.0

# Legújabb verzió indítása beszédes névvel:
docker run -d -p 6000:6379 --name redis-latest redis
```

Innentől kezdve minden Docker parancsban (`stop`, `start`, `logs`, `exec`) használhatjuk a `redis-older` vagy `redis-latest` nevet a Container ID helyett!

### Naplófájlok lekérdezése (`docker logs`)

Kiírja a háttérben futó (vagy akár már leállt) konténer szabványos kimenetét (`stdout` és `stderr`).

```bash
# Lekérdezés Container ID alapján:
docker logs 838a

# Lekérdezés a konténer neve alapján:
docker logs redis-older
```

**Kiegészítés (Hasznos `docker logs` kapcsolók a gyakorlatban):**
- `docker logs -f redis-older`: A `-f` (`--follow`) kapcsoló a Linuxos `tail -f`-hez hasonlóan élőben streameli az új logsorokat a képernyőre.
- `docker logs --tail 50 redis-older`: Csak az utolsó 50 sort írja ki (nagyon hasznos, ha a konténer már napok óta fut és több gigabájtnyi logot termelt).

### Belépés a futó konténer termináljába (`docker exec -it`)

Előfordul, hogy a logok nem elegendőek: szeretnénk belépni a futó konténer belsejébe, körülnézni a virtuális fájlrendszerében, ellenőrizni egy konfigurációs fájlt vagy a környezeti változókat. Erre szolgál a `docker exec`.

```bash
# Belépés Container ID alapján:
docker exec -it 838a /bin/bash

# Vagy a konténer neve alapján:
docker exec -it redis-latest /bin/bash
```

**A kapcsolók pontos jelentése:**
- `exec`: Parancs végrehajtása egy már futó konténerben.
- `-i` (`--interactive`): Nyitva tartja a szabványos bemenetet (STDIN), így tudsz gépelni a konténernek.
- `-t` (`--tty`): Lefoglal egy pszeudo-terminált (TTY), így kapsz egy normális parancssori promptot (pl. `root@838a4f9c:/data#`). A kettőt mindig együtt, `-it` formában használjuk interaktív belépéshez.
- `/bin/bash` (vagy `/bin/sh`): A program, amit elindítunk a konténeren belül (maga a parancsértelmező shell).

**A videóban bemutatott műveletek a konténeren belül:**
Miután a kurzor megváltozott (`root@<container_id>:/data#`), a konténer saját, izolált Linux fájlrendszerében vagyunk:

```bash
ls          # Fájlok listázása az aktuális könyvtárban
pwd         # Aktuális munkakönyvtár kiírása (pl. /data)
cd /        # Navigálás a konténer gyökérkönyvtárába
env         # A konténeren belüli környezeti változók (Environment Variables) kilistázása
exit        # Kilépés a konténer termináljából vissza a gazdagépre (a konténer ettől tovább fut!)
```

**Fontos megjegyzés a minimalista image-ekről:** Mivel a konténerek apró Linux alapokra (pl. Alpine vagy Debian Slim) épülnek, a legtöbb megszokott Linuxos segédprogram (`curl`, `vim`, `nano`, `ping`, `ip`) nincs bennük előre telepítve.
> *(Kiegészítés: Ha egy konténer annyira minimalista – pl. Alpine alapú –, hogy még a `/bin/bash` sincs benne telepítve, a parancs hibát fog dobni. Ilyenkor `/bin/sh`-t kell használni: `docker exec -it <név> /bin/sh`.)*

### `docker run` vs. `docker start` – A legfontosabb különbség

A videó végén külön hangsúlyt kap a két indítóparancs közötti különbség, mert ez az egyik leggyakoribb hibaforrás:

- **`docker run` → Image-ből hoz létre ÚJ konténert:**
  - Mindig egy Image nevet vár paraméterként.
  - Itt adod meg a konténer összes tulajdonságát (`-d`, `-p 6000:6379`, `--name redis-latest`).
  - Ha tízszer futtatod le a `docker run` parancsot, tíz különálló konténert hozol létre a gépeden.
- **`docker start` → Már LÉTEZŐ, leállított konténert indít újra:**
  - Mindig egy Container ID-t vagy Konténer nevet vár paraméterként (nem image-et!).
  - Nem kell (és nem is lehet) újra megadni neki a `-p` vagy `--name` kapcsolókat, mert a konténer megőrzi azokat a beállításokat, amelyekkel eredetileg a `docker run` során létrehozták.

```bash
# Létrehozás (csak egyszer kell kiadni):
docker run -d -p 6000:6379 --name redis-latest redis

# Ha később leállítod:
docker stop redis-latest

# Amikor másnap újra szükséged van rá (megőrzi a 6000-es portot és a háttérben futást):
docker start redis-latest
```

**Kiegészítés (Konténerek és Image-ek törlése – Takarítás):**
A videóban az előadó megemlíti, hogy idővel sok leállított konténer és régi image halmozódik fel a gépen (például a port-próbálkozások során), amelyeket érdemes letakarítani, de a konkrét törlő parancsokat nem gépelte be. Ezek a következők:

- **Leállított konténer törlése (`docker rm`):**

```bash
docker rm <container_id_vagy_név>
# Futó konténer azonnali, kényszerített törlése:
docker rm -f <container_id_vagy_név>
```

- **Letöltött Image törlése (`docker rmi`):** (Csak akkor törölhető, ha már egyetlen – akár leállított – konténer sem használja!)

```bash
docker rmi <image_id_vagy_név:tag>
```

- **Minden használaton kívüli erőforrás** (leállított konténerek, árva hálózatok, nem használt image-ek) takarítása egyetlen paranccsal:

```bash
docker system prune
```

## 7. Parancs Összefoglaló Puskázó (Cheat Sheet)

| Parancs | Mit csinál? |
| :--- | :--- |
| `docker pull <image>:<tag>` | Letölt egy megadott verziójú képfájlt a Docker Hubról a helyi gépre (indítás nélkül). |
| `docker images` | Kilistázza a helyi gépen tárolt összes Docker image-et és azok méretét. |
| `docker run <image>:<tag>` | Létrehoz és elindít egy új konténert az előtérben (attached mód, lefoglalja a terminált). |
| `docker run -d <image>:<tag>` | Létrehoz és elindít egy új konténert a háttérben (detached mód). |
| `docker run -d -p <host>:<cont> --name <név> <image>` | Új konténer indítása a háttérben, port-összerendeléssel és egyedi konténernévvel. |
| `docker ps` | Kilistázza a jelenleg futó konténereket. |
| `docker ps -a` | Kilistázza az összes konténert (a futókat és a már leállítottakat is). |
| `docker stop <id_vagy_név>` | Szabályosan leállít egy futó konténert. |
| `docker start <id_vagy_név>` | Újraindít egy korábban már létrehozott, de leállított konténert az eredeti beállításaival. |
| `docker logs <id_vagy_név>` | Kiírja a konténer által generált naplóüzeneteket (logokat). |
| `docker exec -it <id_vagy_név> /bin/bash` | Interaktív terminált (shellt) nyit egy futó konténer belsejében hibakereséshez. |
| `docker rm <id_vagy_név>` | Véglegesen töröl egy leállított konténert. |
| `docker rmi <image_id_vagy_név>` | Töröl egy letöltött képfájlt a helyi gépről. |



# 1. Demo Projekt Áttekintés – A Docker a teljes fejlesztési folyamatban

A modern szoftverfejlesztésben a Docker a helyi fejlesztéstől az éles szerverre történő telepítésig minden lépésben jelen van:

```text
[1. Helyi Fejlesztés (Laptop)]
   Node.js app (Hoston fut)  <--->  MongoDB + Mongo Express (Docker konténerekben)
         │
         ▼ (git commit & push - Kód + Dockerfile + docker-compose.yml)
[2. Git Repository]
         │
         ▼ (Triggereli a CI pipeline-t)
[3. CI Szerver (pl. Jenkins)]
   - Lefordítja/összecsomagolja a JS alkalmazást (npm build)
   - A Dockerfile alapján felépíti a Docker Image-et (docker build)
   - Feltölti az Image-et a privát tárolóba (docker push)
         │
         ▼
[4. Privát Docker Registry (pl. AWS ECR)]
   Tárolja a saját alkalmazásunk verziózott képfájljait (pl. my-app:1.0, my-app:1.1)
         │
         ▼ (docker-compose up a szerveren)
[5. Fejlesztői / Teszt Szerver (Dev Server)]
   - Lehúzza a my-app:1.0 image-et a Privát Registry-ből (AWS ECR)
   - Lehúzza a hivatalos mongo és mongo-express image-et a publikus Docker Hubról
   - Egy közös Docker hálózatban elindítja mindhárom konténert
```

# 2. Helyi fejlesztés konténerekkel és a Docker Hálózatok (docker network)

## A demóprojekt felépítése

A példaalkalmazás egy egyszerű felhasználói profilszerkesztő weboldal, amely három komponensből áll:

- **Frontend + Backend (index.html + server.js):** Egy Node.js (Express) szerver, amely a 3000-es porton figyel. Amíg nincs adatbázis mögötte, az oldal frissítésekor (F5) minden módosítás elvész.
- **mongo (Adatbázis konténer):** A hivatalos MongoDB NoSQL adatbázis konténer, amely alapértelmezetten a 27017-es porton figyel.
- **mongo-express (Webes Admin UI konténer):** Egy webes grafikus felület a MongoDB-hez (alapértelmezetten a 8081-es porton figyel), hogy ne kelljen parancssorból lekérdezni az adatbázist, hanem böngészőből lássuk a létrejövő adatbázisokat és kollekciókat (táblákat).

## Miért van szükség saját Docker hálózatra (Docker Network)?

A Docker egyik legfontosabb koncepciója az izolált hálózatkezelés:

- **Kommunikáció a Docker hálózaton BELÜL:** Ha két konténert (pl. `mongodb` és `mongo-express`) ugyanabba az egyéni Docker hálózatba helyezünk, akkor közvetlenül a konténer nevén keresztül (pl. `mongodb`) látják egymást. Nincs szükség a gazdagép `localhost` címére vagy külső port-továbbításra.
- **Kommunikáció a Docker hálózaton KÍVÜLRŐL (a Host gépről):** Amíg a Node.js alkalmazásunkat nem csomagoltuk konténerbe (hanem simán a laptopunkon futtatjuk `node server.js` paranccsal), addig a Node.js-nek és a böngészőnknek kívülről, a `localhost` + nyitott Host port (`-p`) kombinációval kell csatlakoznia a konténerekhez.

## Hálózati és indító parancsok lépésről lépésre

### 1. A szükséges képfájlok letöltése

```bash
docker pull mongo
docker pull mongo-express
```

### 2. Docker hálózatok listázása és új hálózat létrehozása

A Docker telepítéskor automatikusan létrehoz néhány alaphálózatot (`bridge`, `host`, `none`).

```bash
# Meglévő Docker hálózatok kilistázása:
docker network ls

# Saját izolált hálózat létrehozása a projektünknek "mongo-network" néven:
docker network create mongo-network
```

> **Kiegészítés (Miért nem elég az alapértelmezett bridge hálózat?):**
> Amikor elindítasz egy konténert a `--net` (vagy `--network`) kapcsoló nélkül, a Docker az alapértelmezett `bridge` hálózatba teszi. Az alapértelmezett bridge hálózaton azonban nincs automatikus belső DNS-névfeloldás – ott a konténerek csak egymás belső IP-címével (pl. `172.17.0.2`) érnék el egymást, ami újraindításkor változhat. Amikor viszont létrehozol egy saját hálózatot (`docker network create mongo-network`), a Docker elindít rajta egy beépített DNS-szervert (`127.0.0.11`), amely automatikusan feloldja a `--name` kapcsolóval megadott konténerneveket a megfelelő belső IP-címre.

### 3. A MongoDB konténer indítása környezeti változókkal (`-e`) és hálózattal (`--net`)

A hivatalos Docker Hub dokumentáció alapján a `mongo` image indításkor környezeti változókból (Environment Variables) olvassa ki a kezdeti adminisztrátori felhasználónevet és jelszót.

```bash
docker run -d \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  --name mongodb \
  --net mongo-network \
  mongo
```

A kapcsolók részletes magyarázata:

- `-d`: Háttérben (detached módban) futtatja a konténert.
- `-p 27017:27017`: A gazdagép 27017-es portját összeköti a konténer 27017-es portjával (hogy a gazdagépen futó Node.js alkalmazás elérje az adatbázist a `localhost:27017` címen).
- `-e KEY=VALUE`: Környezeti változó (environment variable) átadása a konténernek. Itt állítjuk be, hogy a root user neve `admin`, a jelszava pedig `password` legyen.
- `--name mongodb`: A konténer neve `mongodb` lesz (ezen a néven fogja keresni a `mongo-express`).
- `--net mongo-network` (vagy `--network mongo-network`): Csatlakoztatja a konténert az általunk létrehozott hálózathoz.
- `mongo`: A használt képfájl neve.
- `\`: Többsoros parancs tördelése Linux/macOS terminálban az olvashatóságért.

### 4. A Mongo Express (Web UI) konténer indítása

A `mongo-express` konténernek meg kell adnunk a MongoDB adminisztrátori belépési adatait, valamint azt, hogy melyik szervernéven találja az adatbázist.

```bash
docker run -d \
  -p 8081:8081 \
  -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
  -e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
  -e ME_CONFIG_MONGODB_SERVER=mongodb \
  --net mongo-network \
  --name mongo-express \
  mongo-express
```

> **Kritikus beállítás:** A `-e ME_CONFIG_MONGODB_SERVER=mongodb` sorban az érték (`mongodb`) pontosan megegyezik az előző lépésben elindított adatbázis-konténer `--name` paraméterével! Ez kizárólag azért működik, mert mindkét konténer a `--net mongo-network` hálózatban fut.

Miután elindult, a böngészőben a `http://localhost:8081` címen érhető el a grafikus felület, ahol a videóban létrehozzák a `user-account` nevű adatbázist és azon belül a `users` kollekciót.

### 5. Csatlakozás a Node.js kódból (server.js) és a naplók követése

Amíg a Node.js a laptopon (a Dockeren kívül) fut, a `server.js` fájlban a `MongoClient` a `localhost:27017` címet használja:

```javascript
// Kapcsolódási URI, amikor a Node.js a gazdagépen (Host) fut:
let mongoUrlLocal = "mongodb://admin:password@localhost:27017";
```

**Élő hibakeresés a `docker logs` kapcsolóival:**

Amikor a weboldalon módosítjuk a profilt, a MongoDB konténer naplójában valós időben ellenőrizhetjük a beérkező kapcsolatokat:

```bash
# Csak az utolsó néhány naplósor lekérdezése (pl. utolsó 10 sor):
docker logs mongodb --tail 10

# A napló élő streamelése (folyamatos figyelés, kilépés: Ctrl+C):
docker logs mongodb -f
```

# 3. Docker Compose – Több szolgáltatás (konténer) együttes vezérlése

## Miért van szükség a Docker Compose-ra?

Ahogy az előző fejezetben látszott, már két konténer indításához is hosszú, sokparaméteres `docker run` parancsokat kellett begépelni, és külön létre kellett hozni a hálózatot. Ha egy alkalmazás 5–10 mikroszolgáltatásból áll, a parancssori indítás átláthatatlanná és hibalehetőséggé válik.

A Docker Compose lehetővé teszi, hogy az összes konténert, azok képfájljait, portjait és környezeti változóit egyetlen strukturált, deklaratív YAML fájlban (pl. `docker-compose.yml` vagy a videóban `mongo.yaml`) írjuk le.

## Hogyan képezzük le a docker run parancsokat Docker Compose fájlra?

A videóban létrehozott `mongo.yaml` fájl teljes tartalma és felépítése:

```yaml
version: '3'
services:
  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password

  mongo-express:
    image: mongo-express
    ports:
      - 8080:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb
```

> **Megjegyzés a portokhoz:** A videóban a `mongo-express` külső portját a Compose fájlban `8080:8081`-re állította, így a laptopon a `http://localhost:8080` címen érhető el, amit a Docker a konténer belső 8081-es portjára továbbít.

A YAML fájl kulcsszavainak magyarázata:

- `version: '3'`: A Docker Compose fájlséma verziója.
- `services:`: Itt soroljuk fel az indítandó konténereket. (A YAML fájlokban a behúzás / indentáció szintaktikai jelentőségű: a szolgáltatások nevei egy szinten vannak, a beállításaik pedig eggyel beljebb!)
- `mongodb:` és `mongo-express:`: A szolgáltatások (services) nevei. A Docker Compose ezekből generálja a konténernevet, és ugyanezen a néven (`mongodb`) tudnak egymásra hivatkozni a belső hálózaton!
- `image:`: Melyik képfájlból készüljön a konténer (megadható verzió is, pl. `mongo:4.2`).
- `ports:`: A `-p` kapcsoló megfelelője (`HOST_PORT:CONTAINER_PORT`).
- `environment:`: A `-e` kapcsoló megfelelője (környezeti változók listája).

## Hová tűnt a hálózat (`--net`) beállítása a Compose fájlból?

A Docker Compose egyik legnagyobb előnye, hogy automatikusan létrehoz egy közös, izolált hálózatot az egy fájlban szereplő összes szolgáltatás számára!

Ha a mappánk neve például `my-app`, a Compose létrehoz egy `my-app_default` nevű hálózatot.

A konténereknek pedig automatikusan előtaggal és sorszámmal ellátott nevet ad (pl. `my-app_mongodb_1` és `my-app_mongo-express_1`), miközben a hálózaton belül a rövid szolgáltatásnéven (`mongodb`) is elérhetővé teszi őket.

## Docker Compose parancsok

### `docker-compose up` (Környezet elindítása)

```bash
# Egyéni nevű YAML fájl (mongo.yaml) futtatása a -f (--file) kapcsolóval:
docker-compose -f mongo.yaml up

# Kiegészítés: Háttérben (detached módban) történő indítás:
docker-compose -f mongo.yaml up -d
```

Mi történik indításkor?

- Létrehozza a közös hálózatot (`Creating network "my-app_default" with the default driver`).
- Létrehozza és párhuzamosan elindítja a két konténert.
- Összefésülve (színekkel elkülönítve) kiírja mindkét konténer naplóját a terminálba.
- **Indítási sorrend (Race condition a logokban):** Mivel a két konténer egyszerre indul, a `mongo-express` gyorsabban felállhat, mint maga az adatbázis. Emiatt a logokban az első néhány másodpercben `Connection refused` hiba látható, majd amint a MongoDB készen áll a kapcsolatok fogadására, a `mongo-express` sikeresen rácsatlakozik.

### `docker-compose down` (Környezet leállítása és törlése)

```bash
docker-compose -f mongo.yaml down
```

Mit csinál pontosan a `down`?

- Leállítja az összes szolgáltatást (`Stopping...`).
- Törli a leállított konténereket (`Removing...`).
- Törli az automatikusan létrehozott hálózatot (`Removing network my-app_default`).

> **Fontos következmény:** Mivel a `down` parancs törli a konténereket, a következő `up` parancs teljesen új konténereket hoz létre. Emiatt minden adat (a `user-account` adatbázis és a `users` kollekció), amit a MongoDB konténerben létrehoztunk, elvész! (Ezt fogja megoldani a Docker Volumes a 7. fejezetben).

> **Kiegészítés (Docker Compose V1 vs. V2 és a depends_on használata):**
>
> - **`docker-compose` vs. `docker compose`:** A videóban még a régebbi, különálló Python-alapú `docker-compose` (kötőjeles, V1) parancsot használják, és a fájl elején szerepel a `version: '3'`. A modern Docker verziókban a Compose már közvetlenül a Docker CLI-be épített Go-alapú plugin (Compose V2), így kötőjel nélkül, `docker compose -f mongo.yaml up` formában is hívható, és a `version:` sor ma már elhagyható a YAML fájl elejéről. Ha a fájlod neve pontosan `docker-compose.yml` (vagy `compose.yaml`), akkor a `-f <fájlnév>` kapcsolót sem kell kiírnod, elég a `docker compose up`!
> - **Indítási sorrend szabályozása (`depends_on`):** A videóban említett várakozási logika a YAML fájlban a `depends_on` kulcsszóval adható meg a `mongo-express` alatt:
>
> ```yaml
> mongo-express:
>   image: mongo-express
>   depends_on:
>     - mongodb
> ```
>
> (Haladó tipp: A sima `depends_on` csak megvárja, amíg a `mongodb` konténer elindul, de azt nem, hogy az adatbázis-motor ténylegesen készen álljon a TCP kapcsolatokra. Valódi várakozáshoz a `mongodb`-hez egy `healthcheck:` blokkot, a `depends_on`-hoz pedig `condition: service_healthy` feltételt szoktunk írni.)

# 4. Dockerfile – Saját Docker Image építése

Miután elkészült a Node.js alkalmazásunk, ahhoz, hogy szerverre telepítsük vagy CI/CD pipeline-ban (pl. Jenkins) futtassuk, magát a Node.js alkalmazást is be kell csomagolnunk egy saját Docker Image-be. Ennek a tervrajza (blueprintje) a Dockerfile.

## Hogyan épülnek egymásra az Image rétegek? (Öröklődés)

Minden Docker image egy másik alap image-re (Base Image) épül. Nem kell nulláról Linuxot és Node.js-t telepítenünk, hanem használhatjuk a hivatalos `node` image-et. Ha megnézzük a Docker Hubon a `node:13-alpine` image saját Dockerfile-ját, láthatjuk az öröklődési láncot:

```text
[ Saját Image: my-app:1.0 ]        --> Dockerfile: FROM node:13-alpine
        ▲
[ Hivatalos Image: node:13-alpine ] --> Dockerfile: FROM alpine:3.10 (telepíti rá a Node.js-t és az npm-et)
        ▲
[ Linux Alap Image: alpine:3.10 ]   --> Minimalista (~5 MB-os) Linux fájlrendszer
```

## A projekt mappaszerkezete és a Dockerfile felépítése

A videóban az előadó először a gyökérkönyvtárból másol be mindent (`COPY . /home/app`), de rájön, hogy így a Dockerfile és a `mongo.yaml` is bekerül a konténerbe, amire ott semmi szükség. Ezért a forráskódot egy külön `app/` almappába szervezi:

```text
projekt-mappa/
├── Dockerfile          <-- A képfájl építési tervrajza (pontosan így kell elnevezni, nagy D-vel, kiterjesztés nélkül!)
├── mongo.yaml          <-- A Docker Compose konfiguráció
└── app/                <-- Csak az alkalmazás futásához szükséges fájlok
    ├── index.html
    ├── server.js
    ├── package.json
    ├── images/
    └── node_modules/
```

A végleges, javított Dockerfile tartalma:

```dockerfile
FROM node:13-alpine

ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PWD=password

RUN mkdir -p /home/app

COPY ./app /home/app

CMD ["node", "/home/app/server.js"]
```

## A Dockerfile utasításainak részletes magyarázata

| Utasítás | Mikor fut le? | Hol fut le? | Részletes működés és szabályok |
|---|---|---|---|
| `FROM` | Build (építés) idején | Konténer image | Minden Dockerfile első sora (kivéve az `ARG`-ot). Meghatározza az alap képfájlt (`node:13-alpine`), így a konténerünkben már előre telepítve lesz a Linux és a Node.js. |
| `ENV` | Build idején (és megmarad futásidőben) | Konténeren belül | Környezeti változókat állít be a konténeren belül. (Megjegyzés: Gyakran jobb a környezeti változókat kívülről, a `docker-compose.yml`-ben átadni, mert akkor változtatáskor nem kell újraépíteni az image-et, de alapértelmezett értékként a Dockerfile-ban is megadható.) |
| `RUN` | Build idején (Image készítésekor) | Konténeren belül | Tetszőleges Linux shell parancsot futtat le az image rétegének építése közben (itt: létrehozza a `/home/app` könyvtárat a konténer virtuális fájlrendszerében, nem a laptopodon!). Egy Dockerfile-ban tetszőleges számú `RUN` utasítás lehet. |
| `COPY` | Build idején | Hostról -> Konténerbe | Fájlokat vagy mappákat másol be a gazdagépedről (Host) az épülő konténer képfájlba. Szintaxis: `COPY <forrás_a_hoston> <cél_a_konténerben>`. (Ha `RUN cp ...`-t írnál, az a konténeren belül keresné a forrásfájlt, ezért kell a `COPY` utasítás!) |
| `CMD` | Run idején (Amikor a konténer elindul!) | Konténeren belül | Az Entrypoint (belépési pont) parancs, amely akkor fut le, amikor a felépített image-ből elindítasz egy konténert (`docker run` vagy `docker start`). Egy Dockerfile-ban csak egyetlen érvényes `CMD` lehet! |

## Képfájl építése (docker build), hibakeresés és törlés parancsai

### 1. Az Image felépítése (docker build)

```bash
docker build -t my-app:1.0 .
```

A paraméterek magyarázata:

- `-t my-app:1.0` (`--tag`): Nevet (`my-app`) és verziócímkét (`1.0`) ad az elkészülő képfájlnak.
- `.` (a pont a végén!): Rendkívül fontos! Ez adja meg a Dockerfile helyét (és az úgynevezett Build Context könyvtárat, ahonnan a Docker a `COPY` parancs fájljait elérheti). A `.` az aktuális munkakönyvtárat jelenti.

### 2. A videóban bemutatott hibakeresési és takarítási folyamat (rm és rmi)

A videóban az első buildelésnél a `CMD ["node", "server.js"]` sorban lemaradt a `/home/app/` útvonal, ezért a `docker run my-app:1.0` parancs `Error: Cannot find module '/server.js'` hibával leállt.

Mivel a Dockerfile módosítása után az image-et újra kell építeni (`docker build`), az előadó letörli a hibás image-et. Igen ám, de egy image-et nem lehet letörölni (`docker rmi`), amíg létezik belőle egy (akár leállított) konténer!

A helyes törlési sorrend parancsai:

```bash
# 1. Megkeressük a my-app image-ből létrejött (leállított vagy futó) konténert:
docker ps -a | grep my-app

# 2. Ha a konténer még fut, először leállítjuk:
docker stop <container_id>

# 3. Töröljük magát a konténert (rm = remove container):
docker rm <container_id>

# 4. Most már törölhetjük a képfájlt (rmi = remove image):
docker rmi <image_id_vagy_my-app:1.0>

# 5. Újraépítjük a javított Dockerfile-ból az image-et és elindítjuk:
docker build -t my-app:1.0 .
docker run my-app:1.0
```

### 3. Belépés az Alpine-alapú konténerbe (/bin/sh a /bin/bash helyett!)

Amikor az előadó be akar lépni a futó `my-app:1.0` konténerbe a szokásos `docker exec -it <container_id> /bin/bash` paranccsal, a Docker hibát dob: `exec: "/bin/bash": stat /bin/bash: no such file or directory`.

Miért? Mert a `node:13-alpine` képfájl az Alpine Linuxon alapul, amely a kis méret érdekében nem tartalmazza a `bash` shellt, csak az alapértelmezett POSIX `sh` (Bourne shell) parancsértelmezőt!

```bash
# Helyes belépés Alpine alapú konténerekbe:
docker exec -it <container_id> /bin/sh

# A konténeren belül ellenőrizhetjük a Dockerfile-ban beállított dolgokat:
env                # Látszik a MONGO_DB_USERNAME=admin és MONGO_DB_PWD=password
ls -l /home/app    # Látszanak a COPY ./app /home/app paranccsal bemásolt fájlok
exit               # Kilépés a konténerből
```

> **Kiegészítés (Dockerfile Best Practices a gyakorlatban):**
>
> - **`WORKDIR` használata:** Ahelyett, hogy `RUN mkdir -p /home/app` és `CMD ["node", "/home/app/server.js"]` teljes elérési utakat írnánk, a Dockerfile-ban a `WORKDIR /home/app` utasítást szoktuk használni. Ez automatikusan létrehozza a mappát (ha még nem létezik), és belép oda, így minden utána következő `RUN`, `COPY` és `CMD` parancs már alapból abban a mappában fut (`COPY ./app .` és `CMD ["node", "server.js"]`).
> - **`.dockerignore` fájl:** A videóban a `node_modules` mappát is átmásolta a hostról a konténerbe. A valóságban a `node_modules` mappát soha nem másoljuk át a hostról (mert lehetnek benne OS-specifikus, pl. Windowsra/macOS-re lefordított C++ binárisok, amik az Alpine Linux konténerben elhasalnak). Helyette készítünk egy `.dockerignore` fájlt (beleírva a `node_modules` és `.git` mappákat), a Dockerfile-ban pedig először csak a `package.json`-t másoljuk be, és a konténeren belül futtatjuk le a `RUN npm install` parancsot!
> - **`ENTRYPOINT` vs. `CMD`:** A videó megemlíti az `ENTRYPOINT`-ot is (a hivatalos Node image-ben látszott). A különbség az, hogy a `CMD`-ben megadott parancsot a `docker run <image> <másik_parancs>` futtatásakor könnyedén felülírhatod, míg az `ENTRYPOINT` fixen lefut, és a `CMD` (vagy a `docker run` végén megadott argumentum) paraméterként adódik át neki.

# 5. Privát Docker Repository (AWS ECR) – Képfájlok feltöltése

Miután a fejlesztői gépen (vagy a Jenkins CI szerveren) felépült a `my-app:1.0` image, fel kell töltenünk egy központi, privát tárolóba (Container Registry), ahonnan a szervereink letölthetik. A videóban ehhez az AWS ECR (Amazon Elastic Container Registry) szolgáltatást használják (de ugyanezen az elven működik a GitHub Container Registry, a GitLab Registry, a Harbor vagy a Nexus is).

Az AWS ECR sajátossága: Itt minden alkalmazásnak (image-nek) külön repository-t hozunk létre (tehát a repository neve maga a `my-app` lesz), és ezen a repository-n belül tároljuk az adott alkalmazás különböző verziócímkéit (`1.0`, `1.1`, `1.2` – AWS ECR-ben alapértelmezetten akár 1000 taget).

## Képfájlok elnevezési konvenciója (Image Naming) a Registry-kben

Miért tudtuk eddig egyszerűen azt írni, hogy `docker pull mongo:4.2`, és miért kell egy privát registry-nél egy hosszú URL-szerű nevet megadni?

```text
Általános felépítés:   <registryDomain> / <imageName_vagy_repo> : <tag>

1. Docker Hub (Rövidítés):  mongo:4.2
   Ami a háttérben lefut:   docker.io/library/mongo:4.2

2. Privát Registry (AWS):   664527484861.dkr.ecr.eu-central-1.amazonaws.com/my-app:1.0
                            └──────────────────────┬──────────────────────┘ └──┬─┘ └┬┘
                                            Registry Domain                   Repo Tag
```

Ha nem adsz meg domain nevet az image neve előtt (pl. csak annyit írsz: `docker push my-app:1.0`), a Docker mindig az alapértelmezett publikus Docker Hubra (`docker.io`) próbálja feltölteni! Ahhoz, hogy a Docker tudja, melyik privát szerverre kell küldenie a fájlt, az image nevébe bele kell címkézni (`docker tag`) a cél registry pontos címét.

## A feltöltés (Push) 3 lépése és parancsai

### 1. Bejelentkezés a privát Registry-be (docker login)

Mivel a tároló privát, először hitelesítenünk kell magunkat (akár a saját laptopunkról, akár a Jenkins szerverről töltünk fel).

```bash
# Általános bejelentkezés bármilyen Docker Registry-be:
docker login <registry_domain>

# AWS ECR specifikus bejelentkezés (az AWS CLI generálja a ideiglenes tokent a docker login-nak):
# (A videóban használt AWS CLI v1 parancs):
$(aws ecr get-login --no-include-email --region eu-central-1)

# (Kiegészítés - A modern AWS CLI v2 parancs ugyanezen célra):
aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.eu-central-1.amazonaws.com
```

Sikeres hitelesítés után a terminál kiírja: `Login Succeeded`. Ezt csak egyszer kell megtenni az adott munkamenetben, utána szabadon végezhető `pull` és `push`.

### 2. A helyi Image átcímkézése (docker tag)

A helyi `my-app:1.0` képfájlunknak adunk egy új nevet (aliast), amely tartalmazza az AWS ECR repository teljes címét.

**Szintaxis:** `docker tag <HELYI_IMAGE>:<TAG> <TELJES_REGISTRY_URI>:<TAG>`

```bash
docker tag my-app:1.0 664527484861.dkr.ecr.eu-central-1.amazonaws.com/my-app:1.0
```

Mit csinál a `docker tag`? Nem másolja le fizikailag a fájlokat (nem foglal dupla helyet a lemezen!), csupán létre
