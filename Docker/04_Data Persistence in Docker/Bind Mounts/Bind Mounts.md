# Bind mountok

Amikor bind mountot használsz, a hosztgépen lévő fájl vagy könyvtár a hosztról a konténerbe csatolódik. Ezzel szemben, amikor kötetet (volume) használsz, egy új könyvtár jön létre a Docker tárolási könyvtárán belül a hosztgépen. A Docker létrehozza és karbantartja ezt a tárolási helyet, de a konténerek közvetlenül, szabványos fájlrendszer-műveletekkel férnek hozzá.

## Mikor használj bind mountokat

A bind mountok az alábbi típusú felhasználási esetekhez megfelelőek:

- Forráskód vagy build artefaktumok megosztása a Docker hoszton lévő fejlesztői környezet és egy konténer között.
- Amikor fájlokat akarsz létrehozni vagy generálni egy konténerben, és azokat a hoszt fájlrendszerén szeretnéd megőrizni.
- Konfigurációs fájlok megosztása a hosztgépről a konténerekbe. Így biztosít a Docker alapértelmezésben DNS-feloldást a konténereknek, azzal, hogy a hosztgépről a `/etc/resolv.conf`-ot minden konténerbe csatolja.

A bind mountok buildekhez is elérhetők: bind mountolhatod a forráskódot a hosztról a build konténerbe egy projekt teszteléséhez, linteléséhez vagy fordításához.

## Bind mountolás meglévő adatok fölé

Ha egy fájlt vagy könyvtárat bind mountolsz a konténer egy olyan könyvtárába, amelyben fájlok vagy könyvtárak léteznek, a meglévő fájlok elrejtődnek a mount által. Ez hasonló ahhoz, mintha fájlokat mentenél a `/mnt` könyvtárba egy Linux hoszton, majd egy USB meghajtót csatolnál a `/mnt`-be. A `/mnt` tartalma elrejtődne az USB meghajtó tartalma által, amíg az USB meghajtót le nem csatolják.

A konténereknél nincs egyszerű módja a mount eltávolításának, hogy újra felfedjék az elrejtett fájlokat. A legjobb megoldás az, ha újra létrehozod a konténert a mount nélkül.

## Megfontolások és korlátozások

- A bind mountok alapértelmezésben írási hozzáféréssel rendelkeznek a hoszt fájljaihoz.
- A bind mountok használatának egyik mellékhatása, hogy a konténerben futó folyamatokon keresztül megváltoztathatod a hoszt fájlrendszerét, beleértve fontos rendszerfájlok vagy könyvtárak létrehozását, módosítását vagy törlését. Ez a képesség biztonsági vonatkozásokkal járhat. Például hatással lehet a hosztrendszeren lévő nem Docker-folyamatokra.
- A `readonly` vagy `ro` opcióval megakadályozhatod, hogy a konténer írjon a mountba.
- A bind mountok a Docker démon hosztjához jönnek létre, nem a klienshez.
- Ha távoli Docker démont használsz, nem hozhatsz létre bind mountot a kliensgépen lévő fájlok eléréséhez egy konténerben.
- A Docker Desktop esetében a démon egy Linux VM-en belül fut, nem közvetlenül a natív hoszton. A Docker Desktop beépített mechanizmusokkal rendelkezik, amelyek átlátszóan kezelik a bind mountokat, lehetővé téve a natív hoszt fájlrendszer-útvonalak megosztását a virtuális gépen futó konténerekkel.
- A bind mountokkal rendelkező konténerek erősen kötődnek a hoszthoz.
- A bind mountok a hosztgép fájlrendszerének egy adott könyvtárszerkezetére támaszkodnak. Ez a függőség azt jelenti, hogy a bind mountokkal rendelkező konténerek meghiúsulhatnak, ha egy másik hoszton futnak ugyanazon könyvtárszerkezet nélkül.

## Szintaxis

Bind mount létrehozásához használhatod a `--mount` vagy a `--volume` flaget.

```bash
docker run --mount type=bind,src=<host-path>,dst=<container-path>
docker run --volume <host-path>:<container-path>
```

Általában a `--mount` az előnyben részesített. A fő különbség az, hogy a `--mount` flag explicitebb, és támogatja az összes elérhető opciót.

Ha a `--volume`-t használod egy olyan fájl vagy könyvtár bind mountolásához, amely még nem létezik a Docker hoszton, a Docker automatikusan létrehozza a könyvtárat a hoszton a számodra. Mindig könyvtárként jön létre. Ha a Docker démonnak nincs jogosultsága a forráskönyvtár létrehozásához, hozd létre a konténer indítása előtt.

Alapértelmezésben a `--mount` nem hoz létre automatikusan könyvtárat, ha a megadott mount útvonal nem létezik a hoszton. Ehelyett hibát produkál:

```bash
docker run --mount type=bind,src=/dev/noexist,dst=/mnt/foo alpine
docker: Error response from daemon: invalid mount config for type "bind": bind source path does not exist: /dev/noexist.
```

A `bind-create-src` opcióval automatikusan létrehozhatod a forráskönyvtárat a hoszton, ha az nem létezik:

```bash
docker run --mount type=bind,src=/home/user/mydir,dst=/mnt/foo,bind-create-src alpine
```

### Opciók a `--mount`-hoz

A `--mount` flag több kulcs-érték párt tartalmaz, vesszővel elválasztva, mindegyik egy `<key>=<value>` tuple-t alkotva. A kulcsok sorrendje nem jelentős.

```bash
docker run --mount type=bind,src=<host-path>,dst=<container-path>[,<key>=<value>...]
```

Érvényes opciók a `--mount type=bind`-hez:

| Opció | Leírás |
| --- | --- |
| `source`, `src` | A fájl vagy könyvtár helye a hoszton. Ez lehet abszolút vagy relatív útvonal. |
| `destination`, `dst`, `target` | Az elérési út, ahová a fájl vagy könyvtár a konténerbe csatolódik. Abszolút útvonalnak kell lennie. |
| `readonly`, `ro` | Ha jelen van, a bind mount csak olvashatóként csatolódik a konténerbe. |
| `bind-propagation` | Ha jelen van, megváltoztatja a bind propagációt. |
| `bind-create-src` | Automatikusan létrehozza a forráskönyvtárat a hoszton, ha az nem létezik. Alapértelmezésben a `--mount` hibát produkál, ha a forrásútvonal nem létezik a démonon. |

**Példa:**

```bash
docker run --mount type=bind,src=.,dst=/project,ro,bind-propagation=rshared
```

### Opciók a `--volume`-hoz

A `--volume` vagy `-v` flag három mezőből áll, kettőspont karakterekkel (`:`) elválasztva. A mezőknek helyes sorrendben kell lenniük.

```bash
docker run -v <host-path>:<container-path>[:opts]
```

Az első mező a hoszton lévő útvonal, amelyet a konténerbe kell bind mountolni. A második mező az elérési út, ahová a fájl vagy könyvtár a konténerbe csatolódik.

A harmadik mező opcionális, és egy vesszővel elválasztott opciólista. Érvényes opciók a `--volume`-hoz bind mounttal:

| Opció | Leírás |
| --- | --- |
| `readonly`, `ro` | Ha jelen van, a bind mount csak olvashatóként csatolódik a konténerbe. |
| `z`, `Z` | SELinux címkézést konfigurál. Lásd: Configure the SELinux label |
| `rprivate` (alapértelmezett) | A bind propagációt `rprivate`-re állítja ehhez a mounthez. Lásd: Configure bind propagation. |
| `private` | A bind propagációt `private`-re állítja ehhez a mounthez. Lásd: Configure bind propagation. |
| `rshared` | A bind propagációt `rshared`-re állítja ehhez a mounthez. Lásd: Configure bind propagation. |
| `shared` | A bind propagációt `shared`-re állítja ehhez a mounthez. Lásd: Configure bind propagation. |
| `rslave` | A bind propagációt `rslave`-re állítja ehhez a mounthez. Lásd: Configure bind propagation. |
| `slave` | A bind propagációt `slave`-re állítja ehhez a mounthez. Lásd: Configure bind propagation. |

**Példa:**

```bash
docker run -v .:/project:ro,rshared
```

## Konténer indítása bind mounttal

Tekintsünk egy esetet, amikor van egy `source` könyvtárad, és amikor a forráskódot lefordítod, az artefaktumok egy másik könyvtárba, a `source/target/`-be mentődnek. Szeretnéd, hogy az artefaktumok elérhetők legyenek a konténerben a `/app/` alatt, és szeretnéd, hogy a konténer hozzáférjen egy új buildhez minden alkalommal, amikor a forráskódot a fejlesztői hoszton lefordítod. Használd a következő parancsot a `target/` könyvtár bind mountolásához a konténerbe a `/app/` alatt. Futtasd a parancsot a `source` könyvtárból. A `$(pwd)` alparancs az aktuális munkakönyvtárra bővül Linux vagy macOS hosztokon. Ha Windows-on vagy, lásd még: Path conversions on Windows.

A következő `--mount` és `-v` példák ugyanazt az eredményt produkálják. Nem futtathatod mindkettőt, hacsak nem távolítod el a `devtest` konténert az első futtatása után.

**`--mount`:**

```bash
docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  nginx:latest
```

Használd a `docker inspect devtest` parancsot annak ellenőrzéséhez, hogy a bind mount helyesen jött létre. Keresd a `Mounts` szakaszt:

```json
"Mounts": [
    {
        "Type": "bind",
        "Source": "/tmp/source/target",
        "Destination": "/app",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
],
```

Ez azt mutatja, hogy a mount egy bind mount, a helyes forrást és célt mutatja, hogy a mount olvasás-írás, és hogy a propagáció `rprivate`-re van állítva.

Állítsd le és távolítsd el a konténert:

```bash
docker container rm -fv devtest
```

## Mountolás a konténer egy nem üres könyvtárába

Ha egy könyvtárat bind mountolsz a konténer egy nem üres könyvtárába, a könyvtár meglévő tartalma elrejtődik a bind mount által. Ez előnyös lehet, például amikor az alkalmazásod egy új verzióját szeretnéd tesztelni új image építése nélkül. Ez azonban meglepő is lehet, és ez a viselkedés eltér a kötetekétől.

Ez a példa szándékosan szélsőséges, de lecseréli a konténer `/usr/` könyvtárának tartalmát a hosztgép `/tmp/` könyvtárára. A legtöbb esetben ez egy nem működő konténert eredményezne.

A `--mount` és `-v` példák ugyanazt a végeredményt adják.

**`--mount`:**

```bash
docker run -d \
  -it \
  --name broken-container \
  --mount type=bind,source=/tmp,target=/usr \
  nginx:latest
```

```plaintext
docker: Error response from daemon: oci runtime error: container_linux.go:262:
starting container process caused "exec: \"nginx\": executable file not found in $PATH".
```

A konténer létrejön, de nem indul el. Távolítsd el:

```bash
docker container rm broken-container
```

## Csak olvasható bind mount használata

Néhány fejlesztői alkalmazásnál a konténernek írnia kell a bind mountba, így a változások visszapropagálódnak a Docker hosztra. Máskor a konténernek csak olvasási hozzáférésre van szüksége.

Ez a példa módosítja az előzőt, de a könyvtárat csak olvasható bind mountként csatolja, hozzáadva a `ro`-t az (alapértelmezésben üres) opciólistához, a konténeren belüli mount pont után. Ha több opció van jelen, vesszővel válaszd el őket.

A `--mount` és `-v` példák ugyanazt az eredményt adják.

**`--mount`:**

```bash
docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app,readonly \
  nginx:latest
```

Használd a `docker inspect devtest` parancsot annak ellenőrzéséhez, hogy a bind mount helyesen jött létre. Keresd a `Mounts` szakaszt:

```json
"Mounts": [
    {
        "Type": "bind",
        "Source": "/tmp/source/target",
        "Destination": "/app",
        "Mode": "ro",
        "RW": false,
        "Propagation": "rprivate"
    }
],
```

Állítsd le és távolítsd el a konténert:

```bash
docker container rm -fv devtest
```

## Rekurzív mountok

Amikor egy olyan útvonalat bind mountolsz, amely maga is tartalmaz mountokat, azok az almountok alapértelmezésben szintén belekerülnek a bind mountba. Ez a viselkedés konfigurálható a `bind-recursive` opcióval a `--mount`-hoz. Ez az opció csak a `--mount` flaggel támogatott, a `-v`-vel vagy `--volume`-mal nem.

Ha a bind mount csak olvasható, a Docker Engine a legjobb erőfeszítéssel megpróbálja az almountokat is csak olvashatóvá tenni. Ezt nevezik rekurzív csak olvasható mountoknak. A rekurzív csak olvasható mountok Linux kernel 5.12-es vagy újabb verzióját igénylik. Ha régebbi kernelverziót futtatsz, az almountok automatikusan olvasás-írásként csatolódnak alapértelmezésben. Ha megpróbálod az almountokat csak olvashatóra állítani egy 5.12-nél korábbi kernelverzióban, a `bind-recursive=readonly` opció használatával, az hibát eredményez.

A `bind-recursive` opció támogatott értékei:

| Érték | Leírás |
| --- | --- |
| `enabled` (alapértelmezett) | A csak olvasható mountok rekurzívan csak olvashatóvá válnak, ha a kernel v5.12 vagy újabb. Ellenkező esetben az almountok olvasás-írás. |
| `disabled` | Az almountok figyelmen kívül maradnak (nem kerülnek bele a bind mountba). |
| `writable` | Az almountok olvasás-írás. |
| `readonly` | Az almountok csak olvashatók. Kernel v5.12 vagy újabb szükséges. |

## Bind propagáció konfigurálása

A bind propagáció alapértelmezésben `rprivate` mind a bind mountok, mind a kötetek esetében. Csak bind mountokhoz konfigurálható, és csak Linux hosztgépeken. A bind propagáció egy haladó téma, és sok felhasználónak soha nem kell konfigurálnia.

A bind propagáció arra utal, hogy egy adott bind-mountban létrehozott mountok propagálódhatnak-e a mount replikáira. Tekintsünk egy `/mnt` mount pontot, amely a `/tmp`-re is csatolva van. A propagációs beállítások szabályozzák, hogy egy `/tmp/a`-n lévő mount elérhető lenne-e a `/mnt/a`-n is. Minden propagációs beállításnak van egy rekurzív megfelelője. A rekurzió esetében tekintsük, hogy a `/tmp/a` a `/foo`-ként is csatolva van. A propagációs beállítások szabályozzák, hogy a `/mnt/a` és/vagy `/tmp/a` létezne-e.

> **Megjegyzés**
>
> A mount propagáció nem működik a Docker Desktoppal.

| Propagációs beállítás | Leírás |
| --- | --- |
| `shared` | Az eredeti mount al-mountjai ki vannak téve a replika mountoknak, és a replika mountok al-mountjai is propagálódnak az eredeti mountba. |
| `slave` | Hasonló a `shared` mounthez, de csak egy irányban. Ha az eredeti mount kitet egy al-mountot, a replika mount látja azt. Ha azonban a replika mount tesz ki egy al-mountot, az eredeti mount nem látja azt. |
| `private` | A mount privát. A benne lévő al-mountok nincsenek kitéve a replika mountoknak, és a replika mountok al-mountjai nincsenek kitéve az eredeti mountnak. |
| `rshared` | Ugyanaz, mint a `shared`, de a propagáció kiterjed az eredeti vagy replika mount pontokon belül beágyazott mount pontokra is. |
| `rslave` | Ugyanaz, mint a `slave`, de a propagáció kiterjed az eredeti vagy replika mount pontokon belül beágyazott mount pontokra is. |
| `rprivate` | Az alapértelmezett. Ugyanaz, mint a `private`, ami azt jelenti, hogy az eredeti vagy replika mount pontokon belül bárhol lévő mount pontok egyik irányban sem propagálódnak. |

Mielőtt bind propagációt állítanál be egy mount ponton, a hoszt fájlrendszerének már támogatnia kell a bind propagációt.

A bind propagációról további információért lásd a Linux kernel dokumentációját a shared subtree-ról.

A következő példa a `target/` könyvtárat kétszer csatolja a konténerbe, és a második mount beállítja mind a `ro` opciót, mind az `rslave` bind propagációs opciót.

A `--mount` és `-v` példák ugyanazt az eredményt adják.

**`--mount`:**

```bash
docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  --mount type=bind,source="$(pwd)"/target,target=/app2,readonly,bind-propagation=rslave \
  nginx:latest
```

Most, ha létrehozod a `/app/foo/`-t, a `/app2/foo/` is létezik.

## SELinux címke konfigurálása

Ha SELinux-t használsz, hozzáadhatod a `z` vagy `Z` opciókat a hoszt fájl vagy könyvtár SELinux címkéjének módosításához, amelyet a konténerbe csatolsz. Ez magára a hosztgépen lévő fájlra vagy könyvtárra hat, és a Docker hatókörén kívüli következményei lehetnek.

- A `z` opció azt jelzi, hogy a bind mount tartalma több konténer között megosztott.
- A `Z` opció azt jelzi, hogy a bind mount tartalma privát és nem megosztott.

Rendkívüli óvatossággal használd ezeket az opciókat. Egy rendszerkönyvtár, például a `/home` vagy `/usr` bind mountolása a `Z` opcióval működésképtelenné teszi a hosztgépedet, és lehet, hogy kézzel kell újracímkézned a hosztgép fájljait.

> **Fontos**
>
> Amikor bind mountokat használsz szolgáltatásokkal, a SELinux címkék (`:Z` és `:z`), valamint a `:ro` figyelmen kívül maradnak. Lásd: moby/moby #32579 a részletekért.

Ez a példa a `z` opciót állítja be, hogy megadja, több konténer oszthatja a bind mount tartalmát:

A SELinux címke módosítása nem lehetséges a `--mount` flaggel.

```bash
docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app:z \
  nginx:latest
```

## Bind mount használata Docker Compose-zal

Egyetlen Docker Compose szolgáltatás bind mounttal így néz ki:

```yaml
services:
  frontend:
    image: node:lts
    volumes:
      - type: bind
        source: ./static
        target: /opt/app/static
volumes:
  myapp:
```

A bind típusú kötetek Compose-zal való használatáról további információért lásd: Compose reference on the volumes top-level element és Compose reference on the volume attribute.
