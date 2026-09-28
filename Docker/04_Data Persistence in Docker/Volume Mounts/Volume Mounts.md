# Kötetek (Volumes)

A kötetek (volumes) a konténerek perzisztens adattárolói, amelyeket a Docker hoz létre és kezel. Létrehozhatsz egy kötetet explicit módon a `docker volume create` paranccsal, vagy a Docker létrehozhat egy kötetet a konténer vagy szolgáltatás létrehozása során.

Amikor létrehozol egy kötetet, az a Docker hoszt egy könyvtárában tárolódik. Amikor a kötetet egy konténerbe csatolod (mount), ez a könyvtár kerül a konténerbe. Ez hasonló a bind mountok működéséhez, azzal a különbséggel, hogy a köteteket a Docker kezeli, és el vannak különítve a hosztgép alapvető funkcionalitásától.

## Mikor használj köteteket

A kötetek a Docker konténerek által létrehozott és használt adatok perzisztálásának előnyben részesített mechanizmusa. Míg a bind mountok a hosztgép könyvtárszerkezetétől és operációs rendszerétől függenek, a köteteket teljesen a Docker kezeli. A kötetek jó választás az alábbi felhasználási esetekhez:

- A kötetek könnyebben menthetők vagy migrálhatók, mint a bind mountok.
- A köteteket Docker CLI parancsokkal vagy a Docker API-val kezelheted.
- A kötetek Linux és Windows konténereken is működnek.
- A kötetek biztonságosabban megoszthatók több konténer között.
- Az új kötetek tartalma előre feltölthető egy konténer vagy build által.
- Amikor az alkalmazásod nagy teljesítményű I/O-t igényel.

A kötetek nem jó választás, ha a fájlokat a hosztról kell elérned, mivel a kötetet teljesen a Docker kezeli. Használj bind mountokat, ha fájlokat vagy könyvtárakat kell elérned mind a konténerekből, mind a hosztról.

A kötetek gyakran jobb választás, mint az adatok közvetlen írása egy konténerbe, mert a kötet nem növeli az őt használó konténerek méretét. A kötet használata is gyorsabb; a konténer írható rétegébe való írás megköveteli, hogy egy storage driver kezelje a fájlrendszert. A storage driver egy union fájlrendszert biztosít, a Linux kernel használatával. Ez az extra absztrakció csökkenti a teljesítményt a kötetek használatához képest, amelyek közvetlenül a hoszt fájlrendszerébe írnak.

Ha a konténered nem perzisztens állapotadatokat generál, fontold meg egy `tmpfs` mount használatát, hogy elkerüld az adatok állandó tárolását bárhol, és növeld a konténer teljesítményét azáltal, hogy elkerülöd a konténer írható rétegébe való írást.

A kötetek `rprivate` (rekurzív privát) bind propagációt használnak, és a bind propagáció nem konfigurálható a kötetekhez.

## Egy kötet életciklusa

Egy kötet tartalma egy adott konténer életciklusán kívül létezik. Amikor egy konténer megsemmisül, az írható réteg is megsemmisül vele. A kötet használata biztosítja, hogy az adatok megmaradjanak, még akkor is, ha az őt használó konténer eltávolításra kerül.

Egy adott kötet egyszerre több konténerbe is csatolható. Amikor egyetlen futó konténer sem használ egy kötetet, a kötet továbbra is elérhető a Docker számára, és nem távolítódik el automatikusan. A nem használt köteteket a `docker volume prune` paranccsal távolíthatod el.

## Kötet csatolása meglévő adatok fölé

Ha egy nem üres kötetet csatolsz a konténer egy olyan könyvtárába, amelyben fájlok vagy könyvtárak léteznek, a meglévő fájlok elrejtődnek a mount által. Ez hasonló ahhoz, mintha fájlokat mentenél a `/mnt` könyvtárba egy Linux hoszton, majd egy USB meghajtót csatolnál a `/mnt`-be. A `/mnt` tartalma elrejtődne az USB meghajtó tartalma által, amíg az USB meghajtót le nem csatolják.

A konténereknél nincs egyszerű módja a mount eltávolításának, hogy újra felfedjék az elrejtett fájlokat. A legjobb megoldás az, ha újra létrehozod a konténert a mount nélkül.

Ha egy üres kötetet csatolsz a konténer egy olyan könyvtárába, amelyben fájlok vagy könyvtárak léteznek, ezek a fájlok vagy könyvtárak alapértelmezésben propagálódnak (másolódnak) a kötetbe. Hasonlóképpen, ha elindítasz egy konténert, és megadsz egy kötetet, amely még nem létezik, létrejön egy üres kötet a számodra. Ez jó módja annak, hogy előre feltöltsd az adatokat, amelyekre egy másik konténernek szüksége van.

Annak megakadályozásához, hogy a Docker a konténer meglévő fájljait egy üres kötetbe másolja, használd a `volume-nocopy` opciót, lásd: Options for --mount.

## Nevesített és névtelen kötetek

Egy kötet lehet nevesített vagy névtelen. A névtelen kötetek véletlenszerű nevet kapnak, amely garantáltan egyedi egy adott Docker hoszton. A nevesített kötetekhez hasonlóan a névtelen kötetek is megmaradnak, még akkor is, ha eltávolítod az őket használó konténert, kivéve, ha a `--rm` flaget használod a konténer létrehozásakor, amely esetben a konténerhez társított névtelen kötet megsemmisül. Lásd: Remove anonymous volumes.

Ha egymás után több konténert hozol létre, amelyek mind névtelen köteteket használnak, minden konténer létrehozza a saját kötetét. A névtelen köteteket nem használják újra vagy osztják meg automatikusan a konténerek között. Egy névtelen kötet két vagy több konténer közötti megosztásához a névtelen kötetet a véletlenszerű kötet ID-jével kell csatolnod.

## Szintaxis

Egy kötet `docker run` paranccsal való csatolásához használhatod a `--mount` vagy a `--volume` flaget.

```bash
docker run --mount type=volume,src=<volume-name>,dst=<mount-path>
docker run --volume <volume-name>:<mount-path>
```

Általában a `--mount` az előnyben részesített. A fő különbség az, hogy a `--mount` flag explicitebb, és támogatja az összes elérhető opciót.

A `--mount`-ot kell használnod, ha:

- Kötet driver opciókat akarsz megadni
- Egy kötet alkönyvtárát akarod csatolni
- Egy kötetet egy Swarm szolgáltatásba akarsz csatolni

### Opciók a `--mount`-hoz

A `--mount` flag több kulcs-érték párt tartalmaz, vesszővel elválasztva, mindegyik egy `<key>=<value>` tuple-t alkotva. A kulcsok sorrendje nem jelentős.

```bash
docker run --mount type=volume[,src=<volume-name>],dst=<mount-path>[,<key>=<value>...]
```

Érvényes opciók a `--mount type=volume`-hoz:

| Opció | Leírás |
| --- | --- |
| `source`, `src` | A mount forrása. Nevesített köteteknél ez a kötet neve. Névtelen köteteknél ez a mező elhagyható. |
| `destination`, `dst`, `target` | Az elérési út, ahová a fájl vagy könyvtár a konténerbe csatolódik. |
| `volume-subpath` | Egy elérési út a köteten belüli alkönyvtárhoz, amelyet a konténerbe kell csatolni. Az alkönyvtárnak léteznie kell a kötetben, mielőtt a kötetet egy konténerbe csatolják. Lásd: Mount a volume subdirectory. |
| `readonly`, `ro` | Ha jelen van, a kötet csak olvashatóként csatolódik a konténerbe. |
| `volume-nocopy` | Ha jelen van, a célhelyen lévő adatok nem másolódnak a kötetbe, ha a kötet üres. Alapértelmezésben a célhely tartalma a csatolt kötetbe másolódik, ha az üres. |
| `volume-opt` | Többször is megadható, egy kulcs-érték párt vesz fel, amely az opció nevéből és értékéből áll. |

**Példa:**

```bash
docker run --mount type=volume,src=myvolume,dst=/data,ro,volume-subpath=/foo
```

### Opciók a `--volume`-hoz

A `--volume` vagy `-v` flag három mezőből áll, kettőspont karakterekkel (`:`) elválasztva. A mezőknek helyes sorrendben kell lenniük.

```bash
docker run -v [<volume-name>:]<mount-path>[:opts]
```

Nevesített kötetek esetében az első mező a kötet neve, és egyedi egy adott hosztgépen. Névtelen kötetek esetében az első mező elhagyható. A második mező az elérési út, ahová a fájl vagy könyvtár a konténerbe csatolódik.

A harmadik mező opcionális, és egy vesszővel elválasztott opciólista. Érvényes opciók a `--volume`-hoz adatkötettel:

| Opció | Leírás |
| --- | --- |
| `readonly`, `ro` | Ha jelen van, a kötet csak olvashatóként csatolódik a konténerbe. |
| `volume-nocopy` | Ha jelen van, a célhelyen lévő adatok nem másolódnak a kötetbe, ha a kötet üres. Alapértelmezésben a célhely tartalma a csatolt kötetbe másolódik, ha az üres. |

**Példa:**

```bash
docker run -v myvolume:/data:ro
```

## Kötetek létrehozása és kezelése

A bind mounttal ellentétben köteteket bármely konténer hatókörén kívül is létrehozhatsz és kezelhetsz.

**Kötet létrehozása:**

```bash
docker volume create my-vol
```

**Kötetek listázása:**

```bash
docker volume ls
```

```plaintext
local               my-vol
```

**Kötet vizsgálata:**

```bash
docker volume inspect my-vol
```

```json
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
```

**Kötet eltávolítása:**

```bash
docker volume rm my-vol
```

## Konténer indítása kötettel

Ha egy konténert egy még nem létező kötettel indítasz, a Docker létrehozza a kötetet a számodra. A következő példa a `myvol2` kötetet a konténer `/app/` könyvtárába csatolja.

A következő `-v` és `--mount` példák ugyanazt az eredményt produkálják. Nem futtathatod mindkettőt, hacsak nem távolítod el a `devtest` konténert és a `myvol2` kötetet az első futtatása után.

**`--mount`:**

```bash
docker run -d \
  --name devtest \
  --mount source=myvol2,target=/app \
  nginx:latest
```

Használd a `docker inspect devtest` parancsot annak ellenőrzéséhez, hogy a Docker létrehozta a kötetet, és az helyesen csatolódott. Keresd a `Mounts` szakaszt:

```json
"Mounts": [
    {
        "Type": "volume",
        "Name": "myvol2",
        "Source": "/var/lib/docker/volumes/myvol2/_data",
        "Destination": "/app",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```

Ez azt mutatja, hogy a mount egy kötet, a helyes forrást és célt mutatja, és hogy a mount olvasás-írás.

Állítsd le a konténert, és távolítsd el a kötetet. Ne feledd, hogy a kötet eltávolítása külön lépés.

```bash
docker container stop devtest

docker container rm devtest

docker volume rm myvol2
```

## Kötet használata Docker Compose-zal

A következő példa egyetlen Docker Compose szolgáltatást mutat egy kötettel:

```yaml
services:
  frontend:
    image: node:lts
    volumes:
      - myapp:/home/node/app
volumes:
  myapp:
```

A `docker compose up` első futtatása létrehoz egy kötetet. A Docker ugyanazt a kötetet használja újra, amikor a parancsot később futtatod.

Létrehozhatsz egy kötetet közvetlenül a Compose-on kívül a `docker volume create` paranccsal, majd hivatkozhatsz rá a `compose.yaml`-ben a következőképpen:

```yaml
services:
  frontend:
    image: node:lts
    volumes:
      - myapp:/home/node/app
volumes:
  myapp:
    external: true
```

A kötetek Compose-zal való használatáról további információért lásd a Volumes szakaszt a Compose specifikációban.

## Szolgáltatás indítása kötetekkel

Amikor elindítasz egy szolgáltatást, és meghatározol egy kötetet, minden szolgáltatáskonténer a saját helyi kötetét használja. Egyik konténer sem tudja megosztani ezeket az adatokat, ha a `local` kötet drivert használod. Néhány kötet driver azonban támogatja a megosztott tárolást.

A következő példa egy `nginx` szolgáltatást indít négy replikával, amelyek mindegyike egy `myvol2` nevű helyi kötetet használ.

```bash
docker service create -d \
  --replicas=4 \
  --name devtest-service \
  --mount source=myvol2,target=/app \
  nginx:latest
```

Használd a `docker service ps devtest-service` parancsot annak ellenőrzéséhez, hogy a szolgáltatás fut:

```bash
docker service ps devtest-service
```

```plaintext
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
4d7oz1j85wwn        devtest-service.1   nginx:latest        moby                Running             Running 14 seconds ago
```

A szolgáltatás eltávolításával leállíthatod a futó feladatokat:

```bash
docker service rm devtest-service
```

A szolgáltatás eltávolítása nem távolítja el a szolgáltatás által létrehozott köteteket. A kötet eltávolítása külön lépés.

## Kötet feltöltése konténer használatával

Ha elindítasz egy konténert, amely új kötetet hoz létre, és a konténerben fájlok vagy könyvtárak vannak a csatolandó könyvtárban, például `/app/`, a Docker a könyvtár tartalmát a kötetbe másolja. A konténer ezután csatolja és használja a kötetet, és a kötetet használó más konténerek is hozzáférnek az előre feltöltött tartalomhoz.

Ennek bemutatására a következő példa elindít egy `nginx` konténert, és feltölti az új `nginx-vol` kötetet a konténer `/usr/share/nginx/html` könyvtárának tartalmával. Ez az a hely, ahol az Nginx tárolja az alapértelmezett HTML tartalmát.

A `--mount` és `-v` példák ugyanazt a végeredményt adják.

**`--mount`:**

```bash
docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html \
  nginx:latest
```

Miután futtattad valamelyiket a példák közül, futtasd a következő parancsokat a konténerek és kötetek tisztításához. Ne feledd, hogy a kötet eltávolítása külön lépés.

```bash
docker container stop nginxtest

docker container rm nginxtest

docker volume rm nginx-vol
```

## Csak olvasható kötet használata

Néhány alkalmazásnál a konténernek írnia kell a kötetbe. Máskor a konténernek csak olvasási hozzáférésre van szüksége az adatokhoz. Több konténer csatolhatja ugyanazt a kötetet. Egyetlen kötetet egyszerre csatolhatsz olvasás-írásként néhány konténerhez és csak olvashatóként másokhoz.

A következő példa megváltoztatja az előzőt. A könyvtárat csak olvasható kötetként csatolja, hozzáadva a `ro`-t az (alapértelmezésben üres) opciólistához, a konténeren belüli mount pont után. Ha több opció van jelen, vesszővel választhatod el őket.

A `--mount` és `-v` példák ugyanazt az eredményt adják.

**`--mount`:**

```bash
docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly \
  nginx:latest
```

Használd a `docker inspect nginxtest` parancsot annak ellenőrzéséhez, hogy a Docker helyesen hozta létre a csak olvasható mountot. Keresd a `Mounts` szakaszt:

```json
"Mounts": [
    {
        "Type": "volume",
        "Name": "nginx-vol",
        "Source": "/var/lib/docker/volumes/nginx-vol/_data",
        "Destination": "/usr/share/nginx/html",
        "Driver": "local",
        "Mode": "",
        "RW": false,
        "Propagation": ""
    }
],
```

Állítsd le és távolítsd el a konténert, majd távolítsd el a kötetet. A kötet eltávolítása külön lépés.

```bash
docker container stop nginxtest

docker container rm nginxtest

docker volume rm nginx-vol
```

## Kötet alkönyvtárának csatolása

Amikor egy kötetet egy konténerbe csatolsz, megadhatod a kötet egy alkönyvtárát a `volume-subpath` paraméterrel a `--mount` flaghez. A megadott alkönyvtárnak léteznie kell a kötetben, mielőtt megpróbálod a konténerbe csatolni; ha nem létezik, a mount meghiúsul.

A `volume-subpath` megadása hasznos, ha egy kötetnek csak egy adott részét szeretnéd megosztani egy konténerrel. Tegyük fel például, hogy több konténered fut, és minden konténer naplóit egy megosztott kötetben szeretnéd tárolni. Létrehozhatsz egy alkönyvtárat minden konténerhez a megosztott kötetben, és az alkönyvtárat a konténerbe csatolhatod.

A következő példa létrehoz egy `logs` kötetet, és elindítja az `app1` és `app2` alkönyvtárakat a kötetben. Ezután elindít két konténert, és a `logs` kötet egyik alkönyvtárát csatolja mindegyik konténerbe. Ez a példa feltételezi, hogy a konténerekben lévő folyamatok a `/var/log/app1` és `/var/log/app2` helyre írják a naplóikat.

```bash
docker volume create logs
docker run --rm \
  --mount src=logs,dst=/logs \
  alpine mkdir -p /logs/app1 /logs/app2
docker run -d \
  --name=app1 \
  --mount src=logs,dst=/var/log/app1,volume-subpath=app1 \
  app1:latest
docker run -d \
  --name=app2 \
  --mount src=logs,dst=/var/log/app2,volume-subpath=app2 \
  app2:latest
```

Ezzel a beállítással a konténerek a naplóikat a `logs` kötet külön alkönyvtáraiba írják. A konténerek nem férhetnek hozzá a másik konténer naplóihoz.

## Adatok megosztása gépek között

Hibaterű alkalmazások építésekor szükség lehet arra, hogy ugyanazon szolgáltatás több replikáját konfiguráld úgy, hogy hozzáférjenek ugyanazokhoz a fájlokhoz.

Több módja van ennek elérésére az alkalmazások fejlesztése során. Az egyik, hogy logikát adsz az alkalmazásodhoz, hogy fájlokat tároljon egy felhő objekttároló rendszerben, mint az Amazon S3. A másik, hogy köteteket hozol létre egy olyan driverrel, amely támogatja a fájlok külső tárolórendszerbe, például NFS-be vagy Amazon S3-ba való írását.

A kötet driverek lehetővé teszik, hogy absztraháld a mögöttes tárolórendszert az alkalmazáslogikától. Például, ha a szolgáltatásaid egy NFS driverrel rendelkező kötetet használnak, frissítheted a szolgáltatásokat egy másik driver használatára. Például az adatok felhőben való tárolásához, az alkalmazáslogika megváltoztatása nélkül.

## Kötet driver használata

Amikor egy kötetet a `docker volume create` paranccsal hozol létre, vagy amikor elindítasz egy konténert, amely egy még nem létrehozott kötetet használ, megadhatsz egy kötet drivert. A következő példák az `rclone/docker-volume-rclone` kötet drivert használják, először egy önálló kötet létrehozásakor, majd egy olyan konténer indításakor, amely új kötetet hoz létre.

> **Megjegyzés**
>
> Ha a kötet drivered vesszővel elválasztott listát fogad el opcióként, a külső CSV parser elől escape-elned kell az értéket. Egy `volume-opt` escape-eléséhez tedd dupla idézőjelek (`"`) közé, és az egész mount paramétert tedd szimpla idézőjelek (`'`) közé.
>
> Például a `local` driver mount opciókat fogad el vesszővel elválasztott listaként az `o` paraméterben. Ez a példa a lista helyes escape-elését mutatja.
>
> ```bash
> docker service create \
>   --mount 'type=volume,src=<VOLUME-NAME>,dst=<CONTAINER-PATH>,volume-driver=local,volume-opt=type=nfs,volume-opt=device=<nfs-server>:<nfs-path>,"volume-opt=o=addr=<nfs-address>,vers=4,soft,timeo=180,bg,tcp,rw"'
>   --name myservice \
>   IMAGE
> ```

### Kezdeti beállítás

A következő példa feltételezi, hogy két node-od van, amelyek közül az első egy Docker hoszt, és SSH-val tud csatlakozni a második node-hoz.

A Docker hoszton telepítsd az `rclone/docker-volume-rclone` plugint:

```bash
docker plugin install --grant-all-permissions rclone/docker-volume-rclone --aliases rclone
```

### Kötet létrehozása kötet driver használatával

Ez a példa az `1.2.3.4` hoszt `/remote` könyvtárát csatolja egy `rclonevolume` nevű kötetbe. Minden kötet drivernek nulla vagy több konfigurálható opciója lehet, mindegyiket egy `-o` flaggel adod meg.

```bash
docker volume create \
  -d rclone \
  --name rclonevolume \
  -o type=sftp \
  -o path=remote \
  -o sftp-host=1.2.3.4 \
  -o sftp-user=user \
  -o "sftp-password=$(cat file_containing_password_for_remote_host)"
```

Ez a kötet most már csatolható a konténerekbe.

### Konténer indítása, amely kötetet hoz létre kötet driver használatával

> **Megjegyzés**
>
> Ha a kötet driver megköveteli, hogy opciókat adj át, a `--mount` flaget kell használnod a kötet csatolásához, és nem a `-v`-t.

```bash
docker run -d \
  --name rclone-container \
  --mount type=volume,volume-driver=rclone,src=rclonevolume,target=/app,volume-opt=type=sftp,volume-opt=path=remote, volume-opt=sftp-host=1.2.3.4,volume-opt=sftp-user=user,volume-opt=-o "sftp-password=$(cat file_containing_password_for_remote_host)" \
  nginx:latest
```

### Szolgáltatás létrehozása, amely NFS kötetet hoz létre

A következő példa bemutatja, hogyan hozhatsz létre NFS kötetet egy szolgáltatás létrehozásakor. A `10.0.0.10`-et használja NFS szerverként és a `/var/docker-nfs`-t az NFS szerveren exportált könyvtárként. Ne feledd, hogy a megadott kötet driver a `local`.

**NFSv3:**

```bash
docker service create -d \
  --name nfs-service \
  --mount 'type=volume,source=nfsvolume,target=/app,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/var/docker-nfs,volume-opt=o=addr=10.0.0.10' \
  nginx:latest
```

**NFSv4:**

```bash
docker service create -d \
  --name nfs-service \
  --mount 'type=volume,source=nfsvolume,target=/app,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/var/docker-nfs,"volume-opt=o=addr=10.0.0.10,rw,nfsvers=4,async"' \
  nginx:latest
```

## CIFS/Samba kötetek létrehozása

Közvetlenül csatolhatsz egy Samba megosztást a Dockerben anélkül, hogy mount pontot konfigurálnál a hoszton.

```bash
docker volume create \
	--driver local \
	--opt type=cifs \
	--opt device=//uxxxxx.your-server.de/backup \
	--opt o=addr=uxxxxx.your-server.de,username=uxxxxxxx,password=*****,file_mode=0777,dir_mode=0777 \
	--name cifs-volume
```

Az `addr` opció szükséges, ha hosztnevet adsz meg IP helyett. Ez lehetővé teszi a Docker számára a hosztnév keresését.

## Blokk tárolóeszközök

Csatolhatsz egy blokk tárolóeszközt, például egy külső meghajtót vagy egy meghajtópartíciót egy konténerbe. A következő példa bemutatja, hogyan hozhatsz létre és használhatsz egy fájlt blokk tárolóeszközként, és hogyan csatolhatod a blokk eszközt konténerkötetként.

> **Fontos**
>
> A következő eljárás csak egy példa. Az itt bemutatott megoldás nem ajánlott általános gyakorlatként. Ne próbáld meg ezt a megközelítést, hacsak nem vagy biztos abban, hogy mit csinálsz.

### Hogyan működik a blokk eszközök csatolása

A motorháztető alatt a `--mount` flag a `local` storage drivert használva meghívja a Linux `mount` syscallt, és változatlanul továbbítja az általad átadott opciókat. A Docker nem implementál semmilyen további funkcionalitást a Linux kernel által támogatott natív mount funkciók tetején.

Ha ismered a Linux `mount` parancsát, a `--mount` opciókat úgy tekintheted, mint amelyek a következő módon kerülnek továbbításra a `mount` parancsnak:

```bash
mount -t <mount.volume-opt.type> <mount.volume-opt.device> <mount.dst> -o <mount.volume-opts.o>
```

Ennek további magyarázatához tekintsd a következő `mount` parancs példát. Ez a parancs a `/dev/loop5` eszközt a `/external-drive` elérési útra csatolja a rendszeren.

```bash
mount -t ext4 /dev/loop5 /external-drive
```

A következő `docker run` parancs hasonló eredményt ér el a futtatott konténer szempontjából. Egy konténer futtatása ezzel a `--mount` opcióval ugyanúgy állítja be a mountot, mintha az előző példa `mount` parancsát hajtottad volna végre.

```bash
docker run \
  --mount='type=volume,dst=/external-drive,volume-driver=local,volume-opt=device=/dev/loop5,volume-opt=type=ext4'
```

A `mount` parancsot nem futtathatod közvetlenül a konténeren belül, mert a konténer nem tud hozzáférni a `/dev/loop5` eszközhöz. Ezért használja a `docker run` parancs a `--mount` opciót.

### Példa: Blokk eszköz csatolása konténerbe

A következő lépések létrehoznak egy ext4 fájlrendszert, és csatolják egy konténerbe. A rendszered fájlrendszer-támogatása az általad használt Linux kernel verziójától függ.

1. Hozz létre egy fájlt, és ossz ki neki helyet:

   ```bash
   fallocate -l 1G disk.raw
   ```

2. Építs fájlrendszert a `disk.raw` fájlra:
