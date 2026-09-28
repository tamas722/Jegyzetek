# `docker image ls`

## Leírás

Képek listázása.

## Használat

```bash
docker image ls [OPTIONS] [REPOSITORY[:TAG]]
```

## Álnév (Aliases)

```bash
docker image list
docker images
```

## Leírás

Az alapértelmezett `docker images` megmutatja az összes legfelső szintű képet, azok tárolóját (repository), címkéit (tags) és méretét.

A Docker képeknek köztes rétegei (intermediate layers) vannak, amelyek növelik az újrafelhasználhatóságot, csökkentik a lemezhasználatot és felgyorsítják a `docker build`-et azáltal, hogy minden lépés gyorsítótárazható. Ezek a köztes rétegek alapértelmezésben nem jelennek meg.

A címkézetlen (dangling) képek szintén rejtettek alapértelmezésben. Használd a `-a` (`--all`) flaget a köztes rétegek és a dangling képek megjelenítéséhez.

A `SIZE` az image és az összes szülőimage által elfoglalt kumulatív terület. Ez az a lemezterület is, amelyet a `docker save` paranccsal létrehozott Tar fájl tartalma használ.

Egy image többször is szerepel a listában, ha több repository-neve vagy címkéje van. Ez az egyetlen image (amely az egyező `IMAGE ID`-ról azonosítható) csak egyszer számít bele a feltüntetett `SIZE`-ba.

## Opciók

| Opció | Alapértelmezés | Leírás |
| --- | --- | --- |
| `-a`, `--all` | | Az összes image megjelenítése (alapértelmezésben a köztes és dangling képek rejtettek) |
| `--digests` | | Digestek megjelenítése |
| `-f`, `--filter` | | Kimenet szűrése a megadott feltételek alapján |
| `--format` | | Kimenet formázása egyedi sablonnal:<br>`'table'`: Kimenet táblázatos formátumban oszlopfejlécekkel (alapértelmezett)<br>`'table TEMPLATE'`: Kimenet táblázatos formátumban a megadott Go sablon használatával<br>`'json'`: Kiírás JSON formátumban<br>`'TEMPLATE'`: Kimenet a megadott Go sablon használatával.<br>A sablonokkal való kimenetformázásról további információért lásd: https://docs.docker.com/go/formatting/ |
| `--no-trunc` | | A kimenet ne legyen csonkolva |
| `-q`, `--quiet` | | Csak az image ID-k megjelenítése |
| `--tree` | | API 1.47+ kísérleti (CLI) Többplatformos képek listázása faként (KÍSÉRLETI) |

## Példák

### A legutóbb létrehozott képek listázása

```bash
docker images
```

```plaintext
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
<none>                    <none>              77af4d6b9913        19 hours ago        1.089 GB
committ                   latest              b6fa739cedf5        19 hours ago        1.089 GB
<none>                    <none>              78a85c484f71        19 hours ago        1.089 GB
docker                    latest              30557a29d5ab        20 hours ago        1.089 GB
<none>                    <none>              5ed6274db6ce        24 hours ago        1.089 GB
postgres                  9                   746b819f315e        4 days ago          213.4 MB
postgres                  9.3                 746b819f315e        4 days ago          213.4 MB
postgres                  9.3.5               746b819f315e        4 days ago          213.4 MB
postgres                  latest              746b819f315e        4 days ago          213.4 MB
```

### Képek listázása név és címke szerint

A `docker images` parancs elfogad egy opcionális `[REPOSITORY[:TAG]]` argumentumot, amely a listát az argumentumnak megfelelő képekre korlátozza. Ha megadod a `REPOSITORY`-t, de nem a `TAG`-et, a `docker images` parancs az adott repository összes képét listázza.

Például a `java` repository összes képének listázásához futtasd a következő parancsot:

```bash
docker images java
```

```plaintext
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
java                8                   308e519aac60        6 days ago          824.5 MB
java                7                   493d82594c15        3 months ago        656.3 MB
java                latest              2711b1d6f3aa        5 months ago        603.9 MB
```

A `[REPOSITORY[:TAG]]` értéknek pontos egyezésnek kell lennie. Ez azt jelenti, hogy például a `docker images jav` nem egyezik a `java` image-vel.

Ha mind a `REPOSITORY`, mind a `TAG` meg van adva, csak az adott repository-nak és címkének megfelelő képek kerülnek listázásra. A `java` repository összes `8` címkéjű helyi képének megtalálásához használhatod:

```bash
docker images java:8
```

```plaintext
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
java                8                   308e519aac60        6 days ago          824.5 MB
```

Ha semmi sem egyezik a `REPOSITORY[:TAG]`-gel, a lista üres.

```bash
docker images java:0
```

```plaintext
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```

### A teljes hosszúságú image ID-k listázása (`--no-trunc`)

```bash
docker images --no-trunc
```

```plaintext
REPOSITORY                    TAG                 IMAGE ID                                                                  CREATED             SIZE
<none>                        <none>              sha256:77af4d6b9913e693e8d0b4b294fa62ade6054e6b2f1ffb617ac955dd63fb0182   19 hours ago        1.089 GB
committest                    latest              sha256:b6fa739cedf5ea12a620a439402b6004d057da800f91c7524b5086a5e4749c9f   19 hours ago        1.089 GB
<none>                        <none>              sha256:78a85c484f71509adeaace20e72e941f6bdd2b25b4c75da8693efd9f61a37921   19 hours ago        1.089 GB
docker                        latest              sha256:30557a29d5abc51e5f1d5b472e79b7e296f595abcf19fe6b9199dbbc809c6ff4   20 hours ago        1.089 GB
<none>                        <none>              sha256:0124422dd9f9cf7ef15c0617cda3931ee68346455441d66ab8bdc5b05e9fdce5   20 hours ago        1.089 GB
<none>                        <none>              sha256:18ad6fad340262ac2a636efd98a6d1f0ea775ae3d45240d3418466495a19a81b   22 hours ago        1.082 GB
<none>                        <none>              sha256:f9f1e26352f0a3ba6a0ff68167559f64f3e21ff7ada60366e2d44a04befd1d3a   23 hours ago        1.089 GB
tryout                        latest              sha256:2629d1fa0b81b222fca63371ca16cbf6a0772d07759ff80e8d1369b926940074   23 hours ago        131.5 MB
<none>                        <none>              sha256:5ed6274db6ceb2397844896966ea239290555e74ef307030ebb01ff91b1914df   24 hours ago        1.089 GB
```

### Image digestek listázása (`--digests`)

A v2 vagy újabb formátumot használó image-eknek van egy tartalomcímezhető azonosítójuk, amelyet digestnek neveznek. Amíg az image generálásához használt bemenet változatlan, a digest értéke kiszámítható. Az image digest értékek listázásához használd a `--digests` flaget:

```bash
docker images --digests
```

```plaintext
REPOSITORY                         TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
localhost:5000/test/busybox        <none>              sha256:cbbf2f9a99b47fc460d422812b6a5adff7dfee951d8fa2e4a98caa0382cfbdbf   4986bf8c1536        9 weeks ago         2.43 MB
```

Amikor egy 2.0-s registrybe pusholsz vagy onnan pullolsz, a push vagy pull parancs kimenete tartalmazza az image digestet. Pullolhatsz digest érték használatával. Hivatkozhatsz digest alapján a `create`, `run` és `rmi` parancsokban, valamint a Dockerfile-ban a `FROM` image hivatkozásban is.

### Szűrés (`--filter`)

A szűrő flag (`-f` vagy `--filter`) formátuma `"key=value"`. Ha több szűrő van, akkor több flaget adj át (pl. `--filter "foo=bar" --filter "bif=baz"`).

A jelenleg támogatott szűrők:

- `dangling` (boolean - true vagy false)
- `label` (`label=<key>` vagy `label=<key>=<value>`)
- `before` (`<image-name>[:<tag>]`, `<image id>` vagy `<image@digest>`) - az adott id vagy hivatkozás előtt létrehozott képek szűrése
- `since` (`<image-name>[:<tag>]`, `<image id>` vagy `<image@digest>`) - az adott id vagy hivatkozás óta létrehozott képek szűrése
- `reference` (egy image hivatkozás mintája) - azon képek szűrése, amelyek hivatkozása megegyezik a megadott mintával

#### Címkézetlen (dangling) képek megjelenítése

```bash
docker images --filter "dangling=true"
```

```plaintext
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              8abc22fbb042        4 weeks ago         0 B
<none>              <none>              48e5f45168b9        4 weeks ago         2.489 MB
<none>              <none>              bf747efa0e2f        4 weeks ago         0 B
<none>              <none>              980fe10e5736        12 weeks ago        101.4 MB
<none>              <none>              dea752e4e117        12 weeks ago        101.4 MB
<none>              <none>              511136ea3c5a        8 months ago        0 B
```

Ez a címkézetlen képeket jeleníti meg, amelyek az image-fa levelei (nem köztes rétegek). Ezek a képek akkor keletkeznek, amikor egy image új buildje elveszi a `repo:tag`-et az image ID-tól, így az `<none>:<none>` vagy címkézetlen marad. Figyelmeztetés jelenik meg, ha megpróbálsz eltávolítani egy image-et, amelyet jelenleg egy konténer használ. Ez a flag lehetővé teszi a kötegelt tisztítást.

Használhatod a `docker rmi`-vel együtt:

```bash
docker rmi $(docker images -f "dangling=true" -q)
```

```plaintext
8abc22fbb042
48e5f45168b9
bf747efa0e2f
980fe10e5736
dea752e4e117
511136ea3c5a
```

A Docker figyelmeztet, ha olyan konténerek léteznek, amelyek ezeket a címkézetlen képeket használják.

#### Adott címkével rendelkező képek megjelenítése

A `label` szűrő a képeket egy címke jelenléte alapján, vagy egy címke és egy érték alapján egyezteti.

A következő szűrő a `com.example.version` címkével rendelkező képeket egyezteti, függetlenül annak értékétől.

```bash
docker images --filter "label=com.example.version"
```

```plaintext
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
match-me-1          latest              eeae25ada2aa        About a minute ago   188.3 MB
match-me-2          latest              dea752e4e117        About a minute ago   188.3 MB
```

A következő szűrő a `com.example.version` címkével és `1.0` értékkel rendelkező képeket egyezteti.

```bash
docker images --filter "label=com.example.version=1.0"
```

```plaintext
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
match-me            latest              511136ea3c5a        About a minute ago   188.3 MB
```

Ebben a példában a `0.1` értékkel üres halmazt ad vissza, mert nem található egyezés.

```bash
docker images --filter "label=com.example.version=0.1"
```

```plaintext
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
```

#### Képek szűrése idő szerint

A `before` szűrő csak az adott ID-vel vagy hivatkozással rendelkező image előtt létrehozott képeket mutatja. Például, ha ezek a képek vannak:

```bash
docker images
```

```plaintext
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
image1              latest              eeae25ada2aa        4 minutes ago        188.3 MB
image2              latest              dea752e4e117        9 minutes ago        188.3 MB
image3              latest              511136ea3c5a        25 minutes ago       188.3 MB
```

A `before` szűrővel ez lenne az eredmény:

```bash
docker images --filter "before=image1"
```

```plaintext
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
image2              latest              dea752e4e117        9 minutes ago        188.3 MB
image3              latest              511136ea3c5a        25 minutes ago       188.3 MB
```

A `since` szűrővel ez lenne az eredmény:

```bash
docker images --filter "since=image3"
```

```plaintext
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
image1              latest              eeae25ada2aa        4 minutes ago        188.3 MB
image2              latest              dea752e4e117        9 minutes ago        188.3 MB
```

#### Képek szűrése hivatkozás szerint

A `reference` szűrő csak azokat a képeket mutatja, amelyek hivatkozása megegyezik a megadott mintával.

```bash
docker images
```

```plaintext
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              e02e811dd08f        5 weeks ago         1.09 MB
busybox             uclibc              e02e811dd08f        5 weeks ago         1.09 MB
busybox             musl                733eb3059dce        5 weeks ago         1.21 MB
busybox             glibc               21c16b6787c6        5 weeks ago         4.19 MB
```

A `reference` szűrővel ez lenne az eredmény:

```bash
docker images --filter=reference='busy*:*libc'
```

```plaintext
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             uclibc              e02e811dd08f        5 weeks ago         1.09 MB
busybox             glibc               21c16b6787c6        5 weeks ago         4.19 MB
```

Több `reference` szűrővel (akár A, akár B egyezés):

```bash
docker images --filter=reference='busy*:uclibc' --filter=reference='busy*:glibc'
```

```plaintext
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             uclibc              e02e811dd08f        5 weeks ago         1.09 MB
busybox             glibc               21c16b6787c6        5 weeks ago         4.19 MB
```

### Kimenet formázása (`--format`)

A formázási opció (`--format`) szépen kiírja a konténer kimenetét egy Go sablon használatával.

A Go sablonhoz érvényes helyőrzők:

| Helyőrző | Leírás |
| --- | --- |
| `.ID` | Image ID |
| `.Repository` | Image repository |
| `.Tag` | Image tag |
| `.Digest` | Image digest |
| `.CreatedSince` | Az image létrehozása óta eltelt idő |
| `.CreatedAt` | Az image létrehozásának ideje |
| `.Size` | Image lemezméret |

A `--format` opció használatakor az image parancs vagy pontosan a sablon által deklarált adatokat adja ki, vagy a `table` direktíva használatakor oszlopfejléceket is tartalmaz.

A következő példa egy fejlécek nélküli sablont használ, és az ID és Repository bejegyzéseket kettősponttal (`:`) elválasztva adja ki az összes image-hez:

```bash
docker images --format "{{.ID}}: {{.Repository}}"
```

```plaintext
77af4d6b9913: <none>
b6fa739cedf5: committ
78a85c484f71: <none>
30557a29d5ab: docker
5ed6274db6ce: <none>
746b819f315e: postgres
746b819f315e: postgres
746b819f315e: postgres
746b819f315e: postgres
```

Az összes image repository-val és taggel való listázásához táblázatos formátumban használhatod:

```bash
docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
```

```plaintext
IMAGE ID            REPOSITORY                TAG
77af4d6b9913        <none>                    <none>
b6fa739cedf5        committ                   latest
78a85c484f71        <none>                    <none>
30557a29d5ab        docker                    latest
5ed6274db6ce        <none>                    <none>
746b819f315e        postgres                  9
746b819f315e        postgres                  9.3
746b819f315e        postgres                  9.3.5
746b819f315e        postgres                  latest
```

Az összes image JSON formátumban való listázásához használd a `json` direktívát:

```bash
docker images --format json
```

```json
{"Containers":"N/A","CreatedAt":"2021-03-04 03:24:42 +0100 CET","CreatedSince":"5 days ago","Digest":"\u003cnone\u003e","ID":"4dd97cefde62","Repository":"ubuntu","SharedSize":"N/A","Size":"72.9MB","Tag":"latest","UniqueSize":"N/A"}
{"Containers":"N/A","CreatedAt":"2021-02-17 22:19:54 +0100 CET","CreatedSince":"2 weeks ago","Digest":"\u003cnone\u003e","ID":"28f6e2705743","Repository":"alpine","SharedSize":"N/A","Size":"5.61MB","Tag":"latest","UniqueSize":"N/A"}
```
