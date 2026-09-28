# `docker container run`

## Leírás

Új konténer létrehozása és futtatása egy image-ből.

## Használat

```bash
docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

## Álnév (Aliases)

```bash
docker run
```

## Leírás

A `docker run` parancs egy parancsot futtat egy új konténerben, szükség esetén letölti az image-et, és elindítja a konténert.

Egy leállított konténert az összes korábbi változásával együtt újraindíthatsz a `docker start` paranccsal. A `docker ps -a` paranccsal megtekintheted az összes konténer listáját, beleértve a leállítottakat is.

## Opciók

| Opció | Alapértelmezés | Leírás |
| --- | --- | --- |
| `--add-host` | | Egyedi host-to-IP leképezés hozzáadása (`host:ip`) |
| `--annotation` | | API 1.43+ Annotáció hozzáadása a konténerhez (átadva az OCI runtime-nak) |
| `-a`, `--attach` | | Csatlakozás a STDIN, STDOUT vagy STDERR-hez |
| `--blkio-weight` | | Blokk I/O (relatív súly), 10 és 1000 között, vagy 0 a letiltáshoz (alapértelmezés 0) |
| `--blkio-weight-device` | | Blokk I/O súly (relatív eszközsúly) |
| `--cap-add` | | Linux képességek hozzáadása |
| `--cap-drop` | | Linux képességek eltávolítása |
| `--cgroup-parent` | | Opcionális szülő cgroup a konténerhez |
| `--cgroupns` | | API 1.41+ Használandó cgroup névtér (`host`\|`private`)<br>`'host'`: A konténer futtatása a Docker hoszt cgroup névterében<br>`'private'`: A konténer futtatása a saját privát cgroup névterében<br>`''`: A démon `default-cgroupns-mode` opciója által konfigurált cgroup névtér használata (alapértelmezett) |
| `--cidfile` | | A konténer ID kiírása a fájlba |
| `--cpu-count` | | CPU szám (csak Windows) |
| `--cpu-percent` | | CPU százalék (csak Windows) |
| `--cpu-period` | | CPU CFS (Completely Fair Scheduler) periódus korlátozása |
| `--cpu-quota` | | CPU CFS (Completely Fair Scheduler) kvóta korlátozása |
| `--cpu-rt-period` | | API 1.25+ CPU valós idejű periódus korlátozása mikroszekundumban |
| `--cpu-rt-runtime` | | API 1.25+ CPU valós idejű futásidő korlátozása mikroszekundumban |
| `-c`, `--cpu-shares` | | CPU részesedések (relatív súly) |
| `--cpus` | | API 1.25+ CPU-k száma |
| `--cpuset-cpus` | | CPU-k, amelyeken a végrehajtás engedélyezett (0-3, 0,1) |
| `--cpuset-mems` | | MEM-ek, amelyeken a végrehajtás engedélyezett (0-3, 0,1) |
| `-d`, `--detach` | | A konténer futtatása a háttérben és a konténer ID kiírása |
| `--detach-keys` | | A konténer leválasztásához használt billentyűsorozat felülírása |
| `--device` | | Hoszt eszköz hozzáadása a konténerhez |
| `--device-cgroup-rule` | | Szabály hozzáadása a cgroup engedélyezett eszközlistájához |
| `--device-read-bps` | | Olvasási sebesség korlátozása (bájt/másodperc) egy eszközről |
| `--device-read-iops` | | Olvasási sebesség korlátozása (IO/másodperc) egy eszközről |
| `--device-write-bps` | | Írási sebesség korlátozása (bájt/másodperc) egy eszközre |
| `--device-write-iops` | | Írási sebesség korlátozása (IO/másodperc) egy eszközre |
| `--dns` | | Egyedi DNS szerverek beállítása |
| `--dns-option` | | DNS opciók beállítása |
| `--dns-search` | | Egyedi DNS keresési domainek beállítása |
| `--domainname` | | Konténer NIS domainnév |
| `--entrypoint` | | Az image alapértelmezett ENTRYPOINT-jának felülírása |
| `-e`, `--env` | | Környezeti változók beállítása |
| `--env-file` | | Környezeti változók fájljának beolvasása |
| `--expose` | | Port vagy porttartomány közzététele |
| `--gpus` | | API 1.40+ Hozzáadandó GPU eszközök (`'all'` az összes GPU átadásához) |
| `--group-add` | | További csoportok hozzáadása a csatlakozáshoz |
| `--health-cmd` | | Az állapot ellenőrzéséhez futtatandó parancs |
| `--health-interval` | | Az ellenőrzések közötti idő (ms\|s\|m\|h) (alapértelmezés 0s) |
| `--health-retries` | | Az egészségtelen állapot jelentéséhez szükséges egymást követő hibák száma |
| `--health-start-interval` | | API 1.44+ Az ellenőrzések közötti idő a kezdő időszakban (ms\|s\|m\|h) (alapértelmezés 0s) |
| `--health-start-period` | | API 1.29+ Kezdő időszak a konténer inicializálásához, mielőtt a health-retries visszaszámlálás elkezdődik (ms\|s\|m\|h) (alapértelmezés 0s) |
| `--health-timeout` | | Egy ellenőrzés futtatásának maximális ideje (ms\|s\|m\|h) (alapértelmezés 0s) |
| `--help` | | Használat kiírása |
| `-h`, `--hostname` | | Konténer hosztneve |
| `--init` | | API 1.25+ Init futtatása a konténeren belül, amely továbbítja a szignálokat és begyűjti a folyamatokat |
| `-i`, `--interactive` | | A STDIN nyitva tartása akkor is, ha nincs csatolva |
| `--io-maxbandwidth` | | Maximális I/O sávszélesség korlát a rendszermeghajtóhoz (csak Windows) |
| `--io-maxiops` | | Maximális IOps korlát a rendszermeghajtóhoz (csak Windows) |
| `--ip` | | IPv4 cím (pl. 172.30.100.104) |
| `--ip6` | | IPv6 cím (pl. 2001:db8::33) |
| `--ipc` | | Használandó IPC mód |
| `--isolation` | | Konténer izolációs technológia |
| `-l`, `--label` | | Metaadatok beállítása a konténeren |
| `--label-file` | | Soronként tagolt címkefájl beolvasása |
| `--link` | | Link hozzáadása egy másik konténerhez |
| `--link-local-ip` | | Konténer IPv4/IPv6 link-local címei |
| `--log-driver` | | A konténer naplózási drivere |
| `--log-opt` | | Naplózási driver opciók |
| `--mac-address` | | Konténer MAC címe (pl. 92:d0:c6:0a:29:33) |
| `-m`, `--memory` | | Memória korlát |
| `--memory-reservation` | | Memória soft limit |
| `--memory-swap` | | Swap korlát egyenlő a memória plusz swap-pal: `'-1'` a korlátlan swap engedélyezéséhez |
| `--memory-swappiness` | -1 | A konténer memória swappiness hangolása (0-100) |
| `--mount` | | Fájlrendszer mount csatolása a konténerhez |
| `--name` | | Név hozzárendelése a konténerhez |
| `--network` | | Konténer csatlakoztatása egy hálózathoz |
| `--network-alias` | | Hálózatra kiterjedő álnév hozzáadása a konténerhez |
| `--no-healthcheck` | | A konténer által meghatározott HEALTHCHECK letiltása |
| `--oom-kill-disable` | | OOM Killer letiltása |
| `--oom-score-adj` | | A hoszt OOM preferenciáinak hangolása (-1000 és 1000 között) |
| `--pid` | | Használandó PID névtér |
| `--pids-limit` | | A konténer pids limitjének hangolása (-1 a korlátlanhoz) |
| `--platform` | | API 1.32+ Platform beállítása, ha a szerver többplatformos |
| `--privileged` | | Kiterjesztett jogosultságok adása a konténernek |
| `-p`, `--publish` | | A konténer portjának/portjainak közzététele a hoszton |
| `-P`, `--publish-all` | | Az összes közzétett port közzététele véletlenszerű portokra |
| `--pull` | missing | Image letöltése futtatás előtt (always, missing, never) |
| `-q`, `--quiet` | | A pull kimenet elnyomása |
| `--read-only` | | A konténer gyökérfájlrendszerének csak olvashatóként való csatolása |
| `--restart` | no | Újraindítási szabályzat a konténer kilépésekor |
| `--rm` | | A konténer és a hozzá tartozó névtelen kötetek automatikus eltávolítása kilépéskor |
| `--runtime` | | A konténerhez használandó runtime |
| `--security-opt` | | Biztonsági opciók |
| `--shm-size` | | A `/dev/shm` mérete |
| `--sig-proxy` | true | A fogadott szignálok proxizása a folyamatnak |
| `--stop-signal` | | A konténer leállításához küldendő szignál |
| `--stop-timeout` | | API 1.25+ Időtúllépés (másodpercben) a konténer leállításához |
| `--storage-opt` | | Storage driver opciók a konténerhez |
| `--sysctl` | | Sysctl opciók |
| `--tmpfs` | | tmpfs könyvtár csatolása |
| `-t`, `--tty` | | Pszeudo-TTY kiosztása |
| `--ulimit` | | Ulimit opciók |
| `--umask` | | API 1.56+ Umask beállítása a konténerhez |
| `--use-api-socket` | | Docker API socket és a szükséges hitelesítés bind mountolása |
| `-u`, `--user` | | Felhasználónév vagy UID (formátum: `<name\|uid>[:<group\|gid>]`) |
| `--userns` | | Használandó felhasználói névtér |
| `--uts` | | Használandó UTS névtér |
| `-v`, `--volume` | | Kötet bind mountolása |
| `--volume-driver` | | Opcionális kötet driver a konténerhez |
| `--volumes-from` | | Kötetek csatolása a megadott konténer(ek)ből |
| `-w`, `--workdir` | | Munkakönyvtár a konténeren belül |

## Példák

### Név hozzárendelése (`--name`)

A `--name` flag lehetővé teszi egy egyedi azonosító megadását a konténerhez. A következő példa egy `test` nevű konténert futtat az `nginx:alpine` image használatával leválasztott módban.

```bash
docker run --name test -d nginx:alpine
```

```plaintext
4bed76d3ad428b889c56c1ecc2bf2ed95cb08256db22dc5ef5863e1d03252a19
```

```bash
docker ps
```

```plaintext
CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS                  PORTS     NAMES
4bed76d3ad42   nginx:alpine   "/docker-entrypoint.…"   1 second ago   Up Less than a second   80/tcp    test
```

A konténerre más parancsokkal is hivatkozhatsz név szerint. Például a következő parancsok leállítják és eltávolítják a `test` nevű konténert:

```bash
docker stop test
test
```

```bash
docker rm test
test
```

Ha nem adsz meg egyedi nevet a `--name` flaggel, a démon véletlenszerűen generált nevet rendel a konténerhez, például `vibrant_cannon`. Az egyedileg meghatározott név használatának előnye, hogy könnyen megjegyezhető azonosítója lesz a konténernek.

Továbbá, ha a konténert egy felhasználó által definiált bridge hálózathoz csatlakoztatod, az ugyanazon a hálózaton lévő többi konténer DNS-en keresztül hivatkozhat a konténerre név szerint.

```bash
docker network create mynet
```

```plaintext
cb79f45948d87e389e12013fa4d969689ed2c3316985dd832a43aaec9a0fe394
```

```bash
docker run --name test --net mynet -d nginx:alpine
```

```plaintext
58df6ecfbc2ad7c42d088ed028d367f9e22a5f834d7c74c66c0ab0485626c32a
```

```bash
docker run --net mynet busybox:latest ping test
```

```plaintext
PING test (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.073 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.411 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.319 ms
64 bytes from 172.18.0.2: seq=3 ttl=64 time=0.383 ms
...
```

### Konténer ID rögzítése (`--cidfile`)

Az automatizálás segítéséhez a Docker a konténer ID-t kiírhatja egy általad választott fájlba. Ez hasonló ahhoz, ahogy egyes programok kiírják a folyamat ID-jüket egy fájlba (láthattad őket PID fájlokként):

```bash
docker run --cidfile /tmp/docker_test.cid ubuntu echo "test"
```

Ez létrehoz egy konténert, és kiírja a `test` szöveget a konzolra. A cidfile flag hatására a Docker megpróbál létrehozni egy új fájlt, és kiírni bele a konténer ID-t. Ha a fájl már létezik, a Docker hibát ad vissza. A Docker bezárja ezt a fájlt, amikor a `docker run` kilép.

### PID beállítások (`--pid`)

```plaintext
--pid=""  : Set the PID (Process) Namespace mode for the container,
             'container:<name|id>': joins another container's PID namespace
             'host': use the host's PID namespace inside the container
```

Alapértelmezésben minden konténernek engedélyezve van a PID névtér.

A PID névtér a folyamatok elkülönítését biztosítja. A PID névtér eltávolítja a rendszerfolyamatok nézetét, és lehetővé teszi a folyamat ID-k újrafelhasználását, beleértve a PID 1-et is.

Bizonyos esetekben szeretnéd, hogy a konténered megossza a hoszt folyamat névterét, lehetővé téve a konténeren belüli folyamatok számára, hogy lássák a rendszer összes folyamatát. Például építhetsz egy konténert olyan hibakereső eszközökkel, mint a `strace` vagy a `gdb`, de ezeket az eszközöket a konténeren belüli folyamatok hibakeresésekor szeretnéd használni.

#### Példa: `htop` futtatása egy konténeren belül

A `htop` futtatásához egy olyan konténerben, amely megosztja a hoszt folyamat névterét:

1. Futtass egy alpine konténert a `--pid=host` opcióval:

   ```bash
   docker run --rm -it --pid=host alpine
   ```

2. Telepítsd a `htop`-ot a konténerben:

   ```bash
   / # apk add --quiet htop
   ```

3. Hívd meg a `htop` parancsot.

   ```bash
   / # htop
   ```

#### Példa: Csatlakozás egy másik konténer PID névteréhez

Egy másik konténer PID névteréhez való csatlakozás hasznos lehet annak a konténernek a hibakereséséhez.

1. Indíts egy konténert, amely egy Redis szervert futtat:

   ```bash
   docker run --rm --name my-nginx -d nginx:alpine
   ```

2. Futtass egy Alpine konténert, amely a `--pid` névteret a `my-nginx` konténerhez csatolja:

   ```bash
   docker run --rm -it --pid=container:my-nginx \
     --cap-add SYS_PTRACE \
     --security-opt seccomp=unconfined \
     alpine
   ```

3. Telepítsd a `strace`-t az Alpine konténerben:

   ```bash
   / # apk add strace
   ```

4. Csatlakozz az 1-es folyamathoz, a `my-nginx` konténer folyamat ID-jéhez:

   ```bash
   / # strace -p 1
   strace: Process 1 attached
   ```

### Névtér-átképezés letiltása egy konténerhez (`--userns`)

Ha engedélyezed a felhasználói névtereket a démonon, minden konténer alapértelmezésben engedélyezett felhasználói névterekkel indul. Egy adott konténer felhasználói névtér-átképezésének letiltásához beállíthatod a `--userns` flaget `host`-ra.

```bash
docker run --userns=host hello-world
```

A `host` az egyetlen érvényes érték a `--userns` flaghez.

További információért lásd: Isolate containers with a user namespace.

### UTS beállítások (`--uts`)

```plaintext
--uts=""  : Set the UTS namespace mode for the container
            'host': use the host's UTS namespace inside the container
```

Az UTS névtér a hosztnév és a domain beállítására szolgál, amely látható az adott névtérben futó folyamatok számára. Alapértelmezésben minden konténernek, beleértve a `--network=host`-tal rendelkezőket is, saját UTS névtere van. A `--uts` `host`-ra állítása azt eredményezi, hogy a konténer ugyanazt az UTS névteret használja, mint a hoszt.

> **Megjegyzés**
>
> A Docker nem engedélyezi a `--hostname` és `--domainname` flagek kombinálását a `--uts=host`-tal. Ez megakadályozza, hogy a hoszt UTS névterében futó konténerek megpróbálják megváltoztatni a hoszt konfigurációját.

Érdemes lehet megosztani az UTS névteret a hoszttal, ha szeretnéd, hogy a konténer hosztneve változzon, ahogy a hoszt hosztneve változik. Egy fejlettebb felhasználási eset a hoszt hosztnevének megváltoztatása egy konténerből.

### IPC beállítások (`--ipc`)

```plaintext
--ipc="MODE"  : Set the IPC mode for the container
```

A `--ipc` flag a következő értékeket fogadja el:

| Érték | Leírás |
| --- | --- |
| `""` | A démon alapértelmezésének használata. |
| `"none"` | Saját privát IPC névtér, a `/dev/shm` nincs csatolva. |
| `"private"` | Saját privát IPC névtér. |
| `"shareable"` | Saját privát IPC névtér, más konténerekkel való megosztás lehetőségével. |
| `"container:<name-or-ID>"` | Csatlakozás egy másik („shareable”) konténer IPC névteréhez. |
| `"host"` | A hosztrendszer IPC névterének használata. |

Ha nincs megadva, a démon alapértelmezése használatos, amely vagy `"private"` vagy `"shareable"` lehet, a démon verziójától és konfigurációjától függően.

A System V interprocess communication (IPC) névterek a nevesített megosztott memóriaszegmensek, szemaforok és üzenetsorok elkülönítését biztosítják.

A megosztott memóriaszegmenseket az inter-process kommunikáció memóriasebességgel való gyorsítására használják, csövek vagy a hálózati verem helyett. A megosztott memóriát általában adatbázisok és egyedi építésű (jellemzően C/OpenMPI, C++/boost könyvtárakat használó) nagy teljesítményű alkalmazások használják a tudományos számítástechnika és a pénzügyi szolgáltatások iparágaiban. Ha ezeket az alkalmazásokat több konténerre bontják, szükség lehet a konténerek IPC mechanizmusainak megosztására, a „shareable” mód használatával a fő (azaz „donor”) konténerhez, és a `"container:<donor-name-or-ID>"` használatával a többi konténerhez.

### Konténer jogosultságok eszkalálása (`--privileged`)

A `--privileged` flag a következő képességeket adja a konténernek:

- Engedélyezi az összes Linux kernel képességet
- Letiltja az alapértelmezett seccomp profilt
- Letiltja az alapértelmezett AppArmor profilt
- Letiltja a SELinux folyamat címkét
- Hozzáférést ad az összes hoszt eszközhöz
- A `/sys`-t olvasás-írásra állítja
- A cgroups mountokat olvasás-írásra állítja

Más szóval, a konténer ezután szinte mindent megtehet, amit a hoszt. Ez a flag azért létezik, hogy speciális felhasználási eseteket tegyen lehetővé, mint a Docker futtatása Dockerben.

> ⚠️ **Figyelmeztetés**
>
> A `--privileged` flaget óvatosan használd. A `--privileged`-dzsel rendelkező konténer nem biztonságosan sandboxolt folyamat. Az ebben a módban lévő konténerek root shellt szerezhetnek a hoszton, és átvehetik az irányítást a rendszer felett.
>
> A legtöbb felhasználási esetben ez a flag nem lehet az előnyben részesített megoldás. Ha a konténered eszkalált jogosultságokat igényel, előnyben kell részesítened a szükséges engedélyek explicit megadását, például egyedi kernel képességek hozzáadásával a `--cap-add`-dal.
>
> További információért lásd: Runtime privilege and Linux capabilities

A következő példa nem működik, mert alapértelmezésben a Docker elutasítja a legtöbb potenciálisan veszélyes kernel képességet, beleértve a `CAP_SYS_ADMIN`-t (amely a fájlrendszerek csatolásához szükséges).

```bash
docker run -t -i --rm ubuntu bash
root@bc338942ef20:/# mount -t tmpfs none /mnt
mount: permission denied
```

Működik, ha hozzáadod a `--privileged` flaget:

```bash
docker run -t -i --privileged ubuntu bash
root@50e3f57e16e6:/# mount -t tmpfs none /mnt
root@50e3f57e16e6:/# df -h
Filesystem      Size  Used Avail Use% Mounted on
none            1.9G     0  1.9G   0% /mnt
```

### Munkakönyvtár beállítása (`-w`, `--workdir`)

```bash
docker run -w /path/to/dir/ ubuntu pwd
```

A `-w` opció a megadott könyvtárban futtatja a végrehajtott parancsot, ebben a példában a `/path/to/dir/`-ben. Ha az útvonal nem létezik, a Docker létrehozza a konténeren belül.

### Storage driver opciók beállítása konténerenként (`--storage-opt`)

```bash
docker run -it --storage-opt size=120G fedora /bin/bash
```

Ez (a `size`) a konténer fájlrendszerének méretét 120G-ra korlátozza létrehozáskor. Ez az opció csak a `btrfs`, `overlay2`, `windowsfilter` és `zfs` storage driverekhez érhető el.

Az `overlay2` storage driver esetében a `size` opció csak akkor érhető el, ha a mögöttes fájlrendszer `xfs`, és `pquota` mount opcióval van csatolva. Ezekben a feltételekben bármilyen méretet átadhatsz, amely kisebb, mint a mögöttes fájlrendszer mérete.

A `windowsfilter`, `btrfs` és `zfs` storage drivereknél nem adhatsz át a Default BaseFS Size-nál kisebb méretet.

### tmpfs csatolása (`--tmpfs`)

A `--tmpfs` flag lehetővé teszi egy tmpfs mount létrehozását.

A `--tmpfs`-nak átadható opciók megegyeznek a Linux `mount -t tmpfs -o` parancsával. A következő példa egy üres tmpfs-t csatol a konténerbe a `rw, noexec, nosuid, size=65536k` opciókkal.

```bash
docker run -d --tmpfs /run:rw,noexec,nosuid,size=65536k my_image
```

További információért lásd: tmpfs mounts.

### Kötet csatolása (`-v`)

```bash
docker  run  -v $(pwd):$(pwd) -w $(pwd) -i -t  ubuntu pwd
```

A fenti példa az aktuális könyvtárat csatolja a konténerbe ugyanazon az útvonalon a `-v` flag használatával, beállítja munkakönyvtárként, majd futtatja a `pwd` parancsot a konténeren belül.

A Docker Engine 23-as verziójától kezdve relatív útvonalakat használhatsz a hoszton.

```bash
docker  run  -v ./content:/content -w /content -i -t  ubuntu pwd
```

A fenti példa az aktuális könyvtárban lévő `content` könyvtárat csatolja a konténerbe a `/content` útvonalon a `-v` flag használatával, beállítja munkakönyvtárként, majd futtatja a `pwd` parancsot a konténeren belül.

```bash
docker run -v /doesnt/exist:/foo -w /foo -i -t ubuntu bash
```

Amikor egy bind-mounted kötet hosztkönyvtára nem létezik, a Docker automatikusan létrehozza ezt a könyvtárat a hoszton a számodra. A fenti példában a Docker létrehozza a `/doesnt/exist` mappát a konténer indítása előtt.

### Kötet csatolása csak olvashatóként (`--read-only`)

```bash
docker run --read-only -v /icanwrite busybox touch /icanwrite/here
```

A köteteket a `--read-only` flaggel kombinálva szabályozhatod, hová ír fájlokat a konténer. A `--read-only` flag a konténer gyökérfájlrendszerét csak olvashatóként csatolja, megtiltva az írásokat a megadott köteteken kívüli helyekre.

```bash
docker run -t -i -v /var/run/docker.sock:/var/run/docker.sock -v /path/to/static-docker-binary:/usr/bin/docker busybox sh
```

A Docker Unix socket és a statikusan linkelt Docker bináris bind mountolásával (lásd: get the Linux binary) teljes hozzáférést adsz a konténernek a hoszt Docker démonjának létrehozásához és manipulálásához.

Windows-on Windows-stílusú útvonalszemantikával kell megadnod az útvonalakat.

```powershell
PS C:\> docker run -v c:\foo:c:\dest microsoft/nanoserver cmd /s /c type c:\dest\somefile.txt
Contents of file

PS C:\> docker run -v c:\foo:d: microsoft/nanoserver cmd /s /c type d:\somefile.txt
Contents of file
```

A következő példák meghiúsulnak Windows-alapú konténerek használatakor, mivel a kötet vagy bind mount célja a konténeren belül a következők egyike kell legyen: nem létező vagy üres könyvtár; vagy a C:-től eltérő meghajtó. Továbbá a bind mount forrásának helyi könyvtárnak kell lennie, nem fájlnak.

```powershell
net use z: \\remotemachine\share
docker run -v z:\foo:c:\dest ...
docker run -v \\uncpath\to\directory:c:\dest ...
docker run -v c:\foo\somefile.txt:c:\dest ...
docker run -v c:\foo:c: ...
docker run -v c:\foo:c:\existing-directory-with-contents ...
```

A kötetekről mélyreható információért lásd: manage data in containers

### Bind mountok vagy kötetek hozzáadása a `--mount` flaggel

A `--mount` flag lehetővé teszi kötetek, hosztkönyvtárak és tmpfs mountok csatolását egy konténerbe.

A `--mount` flag támogatja a
