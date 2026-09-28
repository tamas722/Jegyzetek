# Dockerfile referencia

A Docker automatikusan tud image-eket építeni egy Dockerfile utasításainak beolvasásával. A Dockerfile egy szöveges dokumentum, amely tartalmazza az összes parancsot, amelyet a felhasználó a parancssorban meghívhat egy image összeállításához. Ez az oldal leírja a Dockerfile-ban használható parancsokat.

## Áttekintés

A Dockerfile a következő utasításokat támogatja:

| Utasítás | Leírás |
| --- | --- |
| `ADD` | Helyi vagy távoli fájlok és könyvtárak hozzáadása. |
| `ARG` | Build-idő változók használata. |
| `CMD` | Alapértelmezett parancsok megadása. |
| `COPY` | Fájlok és könyvtárak másolása. |
| `ENTRYPOINT` | Alapértelmezett futtatható fájl megadása. |
| `ENV` | Környezeti változók beállítása. |
| `EXPOSE` | Annak leírása, hogy az alkalmazás mely portokon figyel. |
| `FROM` | Új build-szakasz létrehozása egy alapimage-ből. |
| `HEALTHCHECK` | A konténer állapotának ellenőrzése indításkor. |
| `LABEL` | Metaadatok hozzáadása egy image-hez. |
| `MAINTAINER` | Az image szerzőjének megadása. |
| `ONBUILD` | Utasítások megadása arra az esetre, amikor az image-et egy buildben használják. |
| `RUN` | Build-parancsok végrehajtása. |
| `SHELL` | Az image alapértelmezett shelljének beállítása. |
| `STOPSIGNAL` | A konténer kilépéséhez használandó rendszerhívás-szignál megadása. |
| `USER` | Felhasználói és csoport ID beállítása. |
| `VOLUME` | Kötet mountok létrehozása. |
| `WORKDIR` | Munkakönyvtár megváltoztatása. |

## Formátum

Íme a Dockerfile formátuma:

```dockerfile
# Comment
INSTRUCTION arguments
```

Az utasítás nem kis- és nagybetű-érzékeny. A konvenció azonban az, hogy NAGYBETŰVEL írják őket, hogy könnyebben megkülönböztethetők legyenek az argumentumoktól.

A Docker a Dockerfile utasításait sorrendben hajtja végre. A Dockerfile-nak `FROM` utasítással kell kezdődnie. Ez a parser direktívák, megjegyzések és globális hatókörű `ARG`-ok után lehet. A `FROM` utasítás megadja az alapimage-et, amelyből építesz. A `FROM`-ot csak egy vagy több `ARG` utasítás előzheti meg, amelyek a Dockerfile `FROM` soraiban használt argumentumokat deklarálják.

A BuildKit a `#`-tel kezdődő sorokat megjegyzésként kezeli, kivéve, ha a sor érvényes parser direktíva. A `#` jel a sor bármely más helyén argumentumként kerül kezelésre. Ez lehetővé teszi az ilyen állításokat:

```dockerfile
# Comment
RUN echo 'we are running some # of cool things'
```

A megjegyzéssorok eltávolításra kerülnek, mielőtt a Dockerfile utasításai végrehajtódnak. A következő példában a megjegyzés eltávolításra kerül, mielőtt a shell végrehajtja az `echo` parancsot.

```dockerfile
RUN echo hello \
# comment
world
```

A következő példa egyenértékű.

```dockerfile
RUN echo hello \
world
```

A megjegyzések nem támogatják a sorfolytonossági karaktereket.

> **Megjegyzés a whitespace-ről**
>
> A visszafelé kompatibilitás érdekében a megjegyzések (`#`) és utasítások (mint a `RUN`) előtti vezető whitespace figyelmen kívül marad, de nem ajánlott. A vezető whitespace ezekben az esetekben nem marad meg, ezért a következő példák egyenértékűek:
>
> ```dockerfile
>         # this is a comment-line
>     RUN echo hello
> RUN echo world
> ```
>
> ```dockerfile
> # this is a comment-line
> RUN echo hello
> RUN echo world
> ```
>
> Az utasítás-argumentumokban lévő whitespace azonban nem marad figyelmen kívül. A következő példa `hello world`-öt ír ki a megadott vezető whitespace-szel:
>
> ```dockerfile
> RUN echo "\
>      hello\
>      world"
> ```

## Parser direktívák

A parser direktívák opcionálisak, és befolyásolják a Dockerfile további sorainak kezelését. A parser direktívák nem adnak rétegeket a buildhez, és nem jelennek meg build-lépésekként. A parser direktívák a `# directive=value` formájú speciális megjegyzéstípusként vannak írva. Egy direktíva csak egyszer használható.

A következő parser direktívák támogatottak:

- `syntax`
- `escape`
- `check` (a Dockerfile v1.8.0 óta)

Miután egy megjegyzés, üres sor vagy builder utasítás feldolgozásra került, a BuildKit már nem keres parser direktívákat. Ehelyett mindent, ami parser direktívaként van formázva, megjegyzésként kezel, és nem próbálja érvényesíteni, hogy parser direktíva lehet-e. Ezért minden parser direktívának a Dockerfile tetején kell lennie.

A parser direktíva kulcsok, mint a `syntax` vagy `check`, nem kis- és nagybetű-érzékenyek, de a konvenció szerint kisbetűsek. A direktíva értékei kis- és nagybetű-érzékenyek, és a direktívához megfelelő esetben kell írni őket. Például a `#check=skip=jsonargsrecommended` érvénytelen, mert a check névnek Pascal case-t kell használnia, nem kisbetűt. Szintén konvenció, hogy bármely parser direktíva után egy üres sort kell beilleszteni. A sorfolytonossági karakterek nem támogatottak a parser direktívákban.

Ezen szabályok miatt a következő példák mind érvénytelenek:

Érvénytelen a sorfolytonosság miatt:

```dockerfile
# direc \
tive=value
```

Érvénytelen a kétszeri megjelenés miatt:

```dockerfile
# directive=value1
# directive=value2

FROM ImageName
```

Megjegyzésként kezelt, mert egy builder utasítás után jelenik meg:

```dockerfile
FROM ImageName
# directive=value
```

Megjegyzésként kezelt, mert egy nem parser direktíva megjegyzés után jelenik meg:

```dockerfile
# About my dockerfile
# directive=value
FROM ImageName
```

A következő `unknowndirective` megjegyzésként kezelt, mert nem ismert. Az ismert `syntax` direktíva megjegyzésként kezelt, mert egy nem parser direktíva megjegyzés után jelenik meg.

```dockerfile
# unknowndirective=value
# syntax=value
```

Nem sortörő whitespace megengedett egy parser direktívában. Ezért a következő sorok mind azonos módon kezeltek:

```dockerfile
#directive=value
# directive =value
#	directive= value
# directive = value
#	  dIrEcTiVe=value
```

### syntax

A `syntax` parser direktívával deklarálhatod a buildhez használandó Dockerfile szintaxis verziót. Ha nincs megadva, a BuildKit a Dockerfile frontend beépített verzióját használja. A szintaxis verzió deklarálása lehetővé teszi a legújabb Dockerfile verzió automatikus használatát anélkül, hogy frissítened kellene a BuildKit-et vagy a Docker Engine-t, vagy akár egyedi Dockerfile implementációt használnál.

A legtöbb felhasználó ezt a parser direktívát `docker/dockerfile:1`-re szeretné állítani, ami azt eredményezi, hogy a BuildKit a Dockerfile szintaxis legfrissebb stabil verzióját tölti le a build előtt.

```dockerfile
# syntax=docker/dockerfile:1
```

A parser direktíva működéséről további információért lásd: Custom Dockerfile syntax.

### escape

```dockerfile
# escape=\
```

Vagy

```dockerfile
# escape=`
```

Az `escape` direktíva beállítja a Dockerfile-ban a karakterek escape-eléséhez használt karaktert. Ha nincs megadva, az alapértelmezett escape karakter a `\`.

Az escape karakter mind a sorban lévő karakterek escape-elésére, mind az újsor escape-elésére szolgál. Ez lehetővé teszi, hogy egy Dockerfile utasítás több sorra terjedjen ki. Ne feledd, hogy függetlenül attól, hogy az escape parser direktíva benne van-e a Dockerfile-ban, az escape-elés nem történik meg egy `RUN` parancsban, kivéve a sor végén.

Az escape karakter `-re állítása különösen hasznos Windows-on, ahol a `\` a könyvtár elérési út elválasztója. A `` ` `` összhangban van a Windows PowerShell-lel.

Tekintsd a következő példát, amely nem nyilvánvaló módon hiúsulna meg Windows-on. A második sor végén lévő második `\` az újsor escape-eléseként értelmeződne, ahelyett, hogy az első `\`-ból való escape célpontja lenne. Hasonlóképpen, a harmadik sor végén lévő `\`, feltéve, hogy valóban utasításként kezelik, sorfolytonosságként kezelné. A Dockerfile eredménye az, hogy a második és harmadik sor egyetlen utasításnak számít:

```dockerfile
FROM microsoft/nanoserver
COPY testfile.txt c:\\
RUN dir c:\
```

Eredmény:

```powershell
PS E:\myproject> docker build -t cmd .

Sending build context to Docker daemon 3.072 kB
Step 1/2 : FROM microsoft/nanoserver
 ---> 22738ff49c6d
Step 2/2 : COPY testfile.txt c:\RUN dir c:
GetFileAttributesEx c:RUN: The system cannot find the file specified.
PS E:\myproject>
```

A fenti probléma egyik megoldása az lenne, ha a `/`-t használnánk mind a `COPY` utasítás, mind a `dir` célpontjaként. Ez a szintaxis azonban a legjobb esetben is zavaró, mivel nem természetes a Windows elérési útjaihoz, a legrosszabb esetben pedig hibára hajlamos, mivel nem minden Windows-parancs támogatja a `/`-t elérési út elválasztóként.

Az `escape` parser direktíva hozzáadásával a következő Dockerfile sikeresen lefut a várt módon, a Windows fájlelérési utak természetes platform-szemantikájával:

```dockerfile
# escape=`

FROM microsoft/nanoserver
COPY testfile.txt c:\
RUN dir c:\
```

Eredmény:

```powershell
PS E:\myproject> docker build -t succeeds --no-cache=true .

Sending build context to Docker daemon 3.072 kB
Step 1/3 : FROM microsoft/nanoserver
 ---> 22738ff49c6d
Step 2/3 : COPY testfile.txt c:\
 ---> 96655de338de
Removing intermediate container 4db9acbb1682
Step 3/3 : RUN dir c:\
 ---> Running in a2c157f842f5
 Volume in drive C has no label.
 Volume Serial Number is 7E6D-E0F7

 Directory of c:\

10/05/2016  05:04 PM             1,894 License.txt
10/05/2016  02:22 PM    DIR          Program Files
10/05/2016  02:14 PM    DIR          Program Files (x86)
10/28/2016  11:18 AM                62 testfile.txt
10/28/2016  11:20 AM    DIR          Users
10/28/2016  11:20 AM    DIR          Windows
           2 File(s)          1,956 bytes
           4 Dir(s)  21,259,096,064 bytes free
 ---> 01c7f3bef04f
Removing intermediate container a2c157f842f5
Successfully built 01c7f3bef04f
PS E:\myproject>
```

### check

```dockerfile
# check=skip=<checks|all>
# check=error=<boolean>
```

A `check` direktíva a build-ellenőrzések kiértékelésének konfigurálására szolgál. Alapértelmezésben minden ellenőrzés lefut, és a hibák figyelmeztetésként kezeltek.

Letilthatsz bizonyos ellenőrzéseket a `#check=skip=<check-name>` használatával. Több ellenőrzés kihagyásához vesszővel válaszd el őket:

```dockerfile
# check=skip=JSONArgsRecommended,StageNameCasing
```

Az összes ellenőrzés letiltásához használd a `#check=skip=all`-t.

Alapértelmezésben a sikertelen build-ellenőrzésekkel rendelkező buildek nulla státuszkóddal lépnek ki a figyelmeztetések ellenére. Ahhoz, hogy a build figyelmeztetés esetén meghiúsuljon, állítsd be a `#check=error=true`-t.

```dockerfile
# check=error=true
```

> **Megjegyzés**
>
> A `check` direktíva `error=true` opcióval való használatakor ajánlott a Dockerfile szintaxist egy adott verzióhoz rögzíteni. Ellenkező esetben a build elkezdhet meghiúsulni, amikor új ellenőrzések kerülnek hozzáadásra a jövőbeli verziókban.

A `skip` és `error` opciók kombinálásához használj pontosvesszőt az elválasztásukhoz:

```dockerfile
# check=skip=JSONArgsRecommended;error=true
```

Az összes elérhető ellenőrzés megtekintéséhez lásd a build checks referenciát. Ne feledd, hogy az elérhető ellenőrzések a Dockerfile szintaxis verziójától függenek. Annak biztosításához, hogy a legfrissebb ellenőrzéseket kapd, használd a `syntax` direktívát a Dockerfile szintaxis verziójának a legfrissebb stabil verzióra való megadásához.

## Környezeti helyettesítés

A környezeti változók (amelyeket az `ENV` utasítás deklarál) bizonyos utasításokban változókként is használhatók, amelyeket a Dockerfile értelmez. Az escape-elés szintén kezelt a változószerű szintaxis szó szerinti beillesztéséhez egy utasításba.

A környezeti változók a Dockerfile-ban `$variable_name` vagy `${variable_name}` formában jelöltek. Ezek egyenértékűen kezeltek, és a kapcsos zárójeles szintaxist általában a whitespace nélküli változónevek problémáinak kezelésére használják, mint a `${foo}_bar`.

A `${variable_name}` szintaxis néhány szabványos bash módosítót is támogat az alábbiak szerint:

- `${variable:-word}` azt jelzi, hogy ha a `variable` be van állítva és nem üres, akkor az eredmény az az érték lesz. Ha a `variable` nincs beállítva vagy üres, akkor a `word` lesz az eredmény.
- `${variable-word}` azt jelzi, hogy ha a `variable` be van állítva (még ha üres is), akkor az eredmény az az érték lesz. Ha a `variable` nincs beállítva, akkor a `word` lesz az eredmény.
- `${variable:+word}` azt jelzi, hogy ha a `variable` be van állítva és nem üres, akkor a `word` lesz az eredmény, egyébként az eredmény az üres string.
- `${variable+word}` azt jelzi, hogy ha a `variable` be van állítva (még ha üres is), akkor a `word` lesz az eredmény, egyébként az eredmény az üres string.

A következő változó-helyettesítések támogatottak a Dockerfile szintaxis egy kiadás előtti verziójában, amikor a `# syntax=docker/dockerfile-upstream:master` szintaxis direktívát használod a Dockerfile-odban:

- `${variable#pattern}` eltávolítja a `pattern` legrövidebb egyezését a `variable`-ból, a string elejétől keresve.

  ```bash
  str=foobarbaz echo ${str#f*b}     # arbaz
  ```

- `${variable##pattern}` eltávolítja a `pattern` leghosszabb egyezését a `variable`-ból, a string elejétől keresve.

  ```bash
  str=foobarbaz echo ${str##f*b}    # az
  ```

- `${variable%pattern}` eltávolítja a `pattern` legrövidebb egyezését a `variable`-ból, a string végétől visszafelé keresve.

  ```bash
  string=foobarbaz echo ${string%b*}    # foobar
  ```

- `${variable%%pattern}` eltávolítja a `pattern` leghosszabb egyezését a `variable`-ból, a string végétől visszafelé keresve.

  ```bash
  string=foobarbaz echo ${string%%b*}   # foo
  ```

- `${variable/pattern/replacement}` lecseréli a `pattern` első előfordulását a `variable`-ban a `replacement`-re

  ```bash
  string=foobarbaz echo ${string/ba/fo}  # fooforbaz
  ```

- `${variable//pattern/replacement}` lecseréli a `pattern` összes előfordulását a `variable`-ban a `replacement`-re

  ```bash
  string=foobarbaz echo ${string//ba/fo}  # fooforfoz
  ```

Minden esetben a `word` bármilyen string lehet, beleértve további környezeti változókat is.

A `pattern` egy glob minta, ahol a `?` bármely egyetlen karakterre illeszkedik, a `*` pedig bármennyi karakterre (beleértve a nullát is). A szó szerinti `?` és `*` illesztéséhez használj visszaperjel escape-et: `\?` és `\*`.

Egész változóneveket escape-elhetsz egy `\` hozzáadásával a változó elé: `\$foo` vagy `\${foo}`, például `$foo` és `${foo}` literálokká fordulnak le.

Példa (az elemzett reprezentáció a `#` után jelenik meg):

```dockerfile
FROM busybox
ENV FOO=/bar
WORKDIR ${FOO}   # WORKDIR /bar
ADD . $FOO       # ADD . /bar
COPY \$FOO /quux # COPY $FOO /quux
```

A környezeti változókat a Dockerfile következő utasításlistája támogatja:

- `ADD`
- `COPY`
- `ENV`
- `EXPOSE`
- `FROM`
- `LABEL`
- `STOPSIGNAL`
- `USER`
- `VOLUME`
- `WORKDIR`
- `ONBUILD` (a fenti támogatott utasítások egyikével kombinálva)

A környezeti változókat a `RUN`, `CMD` és `ENTRYPOINT` utasításokkal is használhatod, de ezekben az esetekben a változó-helyettesítést a parancs shell kezeli, nem a builder. Ne feledd, hogy az exec formot használó utasítások nem hívnak meg automatikusan parancs shellt. Lásd: Variable substitution.

A környezeti változó-helyettesítés ugyanazt az értéket használja minden változóhoz az egész utasítás során. Egy változó értékének megváltoztatása csak a későbbi utasításokban lép életbe. Tekintsd a következő példát:

```dockerfile
ENV abc=hello
ENV abc=bye def=$abc
ENV ghi=$abc
```

- A `def` értéke `hello` lesz
- A `ghi` értéke `bye` lesz

## .dockerignore fájl

A `.dockerignore` fájlt használhatod fájlok és könyvtárak kizárására a build kontextusból. További információért lásd: .dockerignore file.

## Shell és exec forma

A `RUN`, `CMD` és `ENTRYPOINT` utasításoknak két lehetséges formája van:

- `INSTRUCTION ["executable","param1","param2"]` (exec forma)
- `INSTRUCTION command param1 param2` (shell forma)

Az exec forma lehetővé teszi a shell string-manipuláció elkerülését, és a parancsok egy adott parancs shell vagy bármely más futtatható fájl használatával való meghívását. JSON tömb szintaxist használ, ahol a tömb minden eleme egy parancs, flag vagy argumentum.

A shell forma lazább, és a könnyű használatot, rugalmasságot és olvashatóságot hangsúlyozza. A shell forma automatikusan parancs shellt használ, míg az exec forma nem.

### Exec forma

Az exec forma JSON tömbként van értelmezve, ami azt jelenti, hogy dupla idézőjeleket (`"`) kell használnod a szavak körül, nem szimpla idézőjeleket (`'`).

```dockerfile
ENTRYPOINT ["/bin/bash", "-c", "echo hello"]
```

Az exec forma legjobban egy `ENTRYPOINT` utasítás megadására használható, kombinálva a `CMD`-vel az alapértelmezett argumentumok beállításához, amelyek futásidőben felülírhatók. További információért lásd: ENTRYPOINT.

### Változó-helyettesítés

Az exec forma használata nem hív meg automatikusan parancs shellt. Ez azt jelenti, hogy a normál shell-feldolgozás, mint a változó-helyettesítés, nem történik meg. Például a `RUN [ "echo", "$HOME" ]` nem kezeli a `$HOME` változó-helyettesítést.

Ha shell-feldolgozást szeretnél, akkor vagy használd a shell formát, vagy futtass egy shellt közvetlenül az exec formával, például: `RUN [ "sh", "-c", "echo $HOME" ]`. Amikor az exec formát használod és közvetlenül egy shellt futtatsz, mint a shell forma esetében, a shell végzi a környezeti változó-helyettesítést, nem a builder.

### Visszaperjelek

Az exec formában escape-elned kell a visszaperjeleket. Ez különösen releváns Windows-on, ahol a visszaperjel az elérési út elválasztója. A következő sor egyébként shell formaként lenne kezelve, mert nem érvényes JSON, és váratlan módon hiúsulna meg:

```dockerfile
RUN ["c:\windows\system32\tasklist.exe"]
```

A helyes szintaxis ehhez a példához:

```dockerfile
RUN ["c:\\windows\\system32\\tasklist.exe"]
```

### Shell forma

Az exec formával ellentétben a shell formát használó utasítások mindig parancs shellt használnak. A shell forma nem használja a JSON tömb formátumot, helyette egy szabályos string. A shell forma string lehetővé teszi az újsorok escape-elését az escape karakterrel (alapértelmezésben visszaperjel) egyetlen utasítás következő sorra való folytatásához. Ez megkönnyíti a hosszabb parancsok használatát, mert lehetővé teszi több sorra bontásukat. Például tekintsd ezt a két sort:

```dockerfile
RUN source $HOME/.bashrc && \
echo $HOME
```

Egyenértékűek a következő sorral:

```dockerfile
RUN source $HOME/.bashrc && echo $HOME
```

A shell formával heredokokat is használhatsz a támogatott parancsok felbontásához.

```dockerfile
RUN <<EOF
  source $HOME/.bashrc
  echo $HOME
EOF
```

A heredokokról további információért lásd: Here-documents.

### Más shell használata

A `SHELL` paranccsal megváltoztathatod az alapértelmezett shellt. Például:

```dockerfile
SHELL ["/bin/bash", "-c"]
RUN echo hello
```

További információért lásd: SHELL.

## FROM

```dockerfile
FROM [--platform=<platform>] <image> [AS <name>]
```

Vagy

```dockerfile
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
```

Vagy

```dockerfile
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
```

A `FROM` utasítás inicializál egy új build-szakaszt, és beállítja az alapimage-et a későbbi utasításokhoz. Mint ilyen, egy érvényes Dockerfile-nak `FROM` utasítással kell kezdődnie. Az image bármilyen érvényes image lehet.

- Az `ARG` az egyetlen utasítás, amely megelőzheti a `FROM`-ot a Dockerfile-ban. Lásd: Understand how ARG and FROM interact.
- A `FROM` többször is megjelenhet egyetlen Dockerfile-ban több image létrehozásához vagy egy build-szakasz másik függőségeként való használatához. Egyszerűen jegyezd meg az utolsó image ID-t, amelyet a commit kiadott minden új `FROM` utasítás előtt. Minden `FROM` utasítás törli az előző utasítások által létrehozott állapotot.
- Opcionálisan egy nevet adhatsz egy új build-szakasznak az `AS name` hozzáadásával a `FROM` utasításhoz. A név használható a későbbi `FROM <name>`, `COPY --from=<name>` és `RUN --mount=type=bind,from=<name>` utasításokban az ebben a szakaszban épített image-re való hivatkozáshoz.
- Egy korábbi build-szakasz alapként való használata egy későbbi szakaszhoz gyakori minta a közös alapkörnyezet megosztásához:

```dockerfile
FROM ubuntu AS base
RUN apt-get update && apt-get install -y shared-tooling

FROM base AS dev
RUN apt-get install -y dev-tooling

FROM base AS prod
COPY --from=build /app /app
```

A tag vagy digest értékek opcionálisak. Ha bármelyiket elhagyod, a builder alapértelmezésben a `latest` címkét feltételezi. A builder hibát ad vissza, ha nem találja a tag értékét.

Az opcionális `--platform` flag használható az image platformjának megadásához, ha a `FROM` többplatformos image-re hivatkozik. Például `linux/amd64`, `linux/arm64` vagy `windows/amd64`. Alapértelmezésben a build kérés célplatformja használatos. Globális build argumentumok használhatók ennek a flagnek az értékében, például az automatikus platform ARG-ok lehetővé teszik egy szakasz natív build platformra való kényszerítését (`--platform=$BUILDPLATFORM`), és annak használatát a célplatformra való cross-compile-hoz a szakaszon belül.

### Az ARG és FROM kölcsönhatásának megértése

A `FROM` utasítások támogatják azokat a változókat, amelyeket bármely `ARG` utasítás deklarál az első `FROM` előtt.

```dockerfile
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD  /code/run-app

FROM extras:${CODE_VERSION}
CMD  /code/run-extras
```

Az első `FROM` előtt deklarált `ARG` egy build-szakaszon kívül van, így nem használható semmilyen utasításban a `FROM` után. Az első `FROM` előtt deklarált `ARG` alapértelmezett értékének használatához használj egy érték nélküli `ARG` utasítást egy build-szakaszon belül:

```dockerfile
ARG VERSION=latest
FROM busybox:$VERSION
ARG VERSION
RUN echo $VERSION > image_version
```

## RUN

A `RUN` utasítás végrehajtja a parancsokat, hogy új réteget hozzon létre az aktuális image tetején. A hozzáadott réteg a Dockerfile következő lépésében használatos. A `RUN`-nak két formája van:

```dockerfile
# Shell form:
RUN [OPTIONS] <command> ...
# Exec form:
RUN [OPTIONS] [ "<command>", ... ]
```

A két forma közötti különbségekről további információért lásd: shell or exec forms.

A shell forma a leggyakrabban használt, és lehetővé teszi a hosszabb utasítások több sorra bontását, akár újsor escape-ekkel, akár heredokokkal:

```dockerfile
RUN <<EOF
apt-get update
apt-get install -y curl
EOF
```

A `RUN` utasítás elérhető `[OPTIONS]` opciói:

| Opció | Minimum Dockerfile verzió |
| --- | --- |
| `--device` | 1.27 |
| `--mount` | 1.2 |
| `--network` | 1.3 |
| `--security` | 1.20 |

### Cache érvénytelenítés RUN utasításokhoz

A `RUN` utasítások gyorsítótára nem érvénytelenítődik automatikusan a következő build során. Egy olyan utasítás gyorsítótára, mint a `RUN apt-get dist-upgrade -y`, újra felhasználásra kerül a következő build során. A `RUN` utasítások gyorsítótára érvényteleníthető a `--no-cache` flag használatával, például `docker build --no-cache`.

További információért lásd a Dockerfile Best Practices útmutatót.

A `RUN` utasítások gyorsítótára érvényteleníthető az `ADD` és `COPY` utasításokkal.

### RUN --device

> **Megjegyzés**
>
> Ez az opció BuildKit 0.20.0 vagy újabb verziót igényel.

```dockerfile
RUN --device=name,[required]
```

A `RUN --device` lehetővé teszi, hogy a build CDI eszközöket kérjen, hogy elérhetők legyenek a build-lépéshez.

> ⚠️ **Figyelmeztetés**
>
> A `--device` használatát a `device` entitlement védi, amelyet engedélyezni kell a `buildkitd` démon `--allow-insecure-entitlement device` flaggel vagy a `buildkitd` configban való indításakor, valamint egy build kéréshez a `--allow device` flaggel.

Az eszköz nevét a BuildKit-ben regisztrált CDI specifikáció adja meg.

A következő példában több eszköz van regisztrálva a `vendor1.com/device` gyártó CDI specifikációjában.

```yaml
cdiVersion: "0.6.0"
kind: "vendor1.com/device"
devices:
  - name: foo
    containerEdits:
      env:
        - FOO=injected
  - name: bar
    annotations:
      org.mobyproject.buildkit.device.class: class1
    containerEdits:
      env:
        - BAR=injected
  - name: baz
    annotations:
      org.mobyproject.buildkit.device.class: class1
    containerEdits:
      env:
        - BAZ=injected
  - name: qux
    annotations:
      org.mobyproject.buildkit.device.class: class2
    containerEdits:
      env:
        - QUX=injected
annotations:
  org.mobyproject.buildkit.device.autoallow: true
```

Az eszköznév formátuma rugalmas, és különféle mintákat fogad el több eszközkonfiguráció támogatásához:

- `vendor1.com/device`: az első eszköz kérése ehhez a gyártóhoz
- `vendor1.com/device=foo`: egy adott eszköz kérése
- `vendor1.com/device=*`: az összes eszköz kérése ehhez a gyártóhoz
- `class1`: eszközök kérése az `org.mobyproject.buildkit.device.class` annotáció alapján

> **Megjegyzés**
>
> Az annotációkat a CDI specifikáció a 0.6.0 verzió óta támogatja.

> **Megjegyzés**
>
> Az összes CDI specifikációban regisztrált eszköz automatikus engedélyezéséhez beállíthatod az `org.mobyproject.buildkit.device.autoallow` annotációt. Ezt az annotációt egy adott eszközhöz is beállíthatod.

#### Példa: CUDA-val működő LLaMA inferencia

Ebben a példában a `--device` flaget használjuk a `llama.cpp` inferencia futtatásához egy NVIDIA GPU eszközön keresztül CDI-n keresztül:

```dockerfile
# syntax=docker/dockerfile:1

FROM scratch AS model
ADD https://huggingface.co/bartowski/Llama-3.2-1B-Instruct-GGUF/resolve/main/Llama-3.2-1B-Instruct-Q4_K_M.gguf /model.gguf

FROM scratch AS prompt
COPY <<EOF prompt.txt
Q: Generate  a list of 10 unique biggest countries by population in JSON with their estimated poulation in 1900 and 2024. Answer only newline formatted JSON with keys "country", "population_1900", "population_2024" with 10 items.
A:
[
    {

EOF

FROM ghcr.io/ggml-org/llama.cpp:full-cuda-b5124
RUN --device=nvidia.com/gpu=all \
    --mount=from=model,target=/models \
    --mount=from=prompt,target=/tmp \
    ./llama-cli -m /models/model.gguf -no-cnv -ngl 99 -f /tmp/prompt.txt
```

### RUN --mount

```dockerfile
RUN --mount=[type=TYPE][,option=<value>[,option=<value>]...]
```

A `RUN --mount` lehetővé teszi olyan fájlrendszer mountok létrehozását, amelyeket a build elérhet. Ez használható:

- Bind mount létrehozására a hoszt fájlrendszerhez vagy más build-szakaszokhoz
- Build titkok vagy ssh-agent socketek eléréséhez
- Állandó csomagkezelési gyorsítótár használatához a build felgyorsítására

A támogatott mount típusok:

| Típus | Leírás |
| --- | --- |
| `bind` (alapértelmezett) | Kontextus-könyvtárak bind-mountolása (csak olvasható). |
| `cache` | Ideiglenes könyvtár csatolása gyorsítótár-könyvtárakhoz fordítók és csomagkezelők számára. |
| `tmpfs` | tmpfs csatolása a build konténerben. |
| `secret` | A build konténer hozzáférésének engedélyezése biztonságos fájlokhoz, mint privát kulcsok, anélkül, hogy beágyaznák őket az image-be vagy a build gyorsítótárba. |
| `ssh` | A build konténer hozzáférésének engedélyezése SSH kulcsokhoz SSH agenteken keresztül, jelmondatok támogatásával. |

#### RUN --mount=type=bind

Ez a mount típus lehetővé teszi fájlok vagy könyvtárak bindolását a build konténerhez. A bind mount alapértelmezésben csak olvasható.

| Opció | Leírás |
| --- | --- |
| `target`, `dst`, `destination`¹ | Mount útvonal. |
| `source` | Forrás útvonal a `from`-ban. Alapértelmezésben a `from` gyökere. |
| `from` | Build-szakasz, kontextus vagy image név a forrás gyökeréhez. Alapértelmezésben a build kontextus. |
| `rw`, `readwrite` | Írások engedélyezése a mounton. Az írt adatok elvetésre kerülnek a `RUN` utasítás befejezése után, és nem kerülnek commitolásra az image rétegbe. |

#### RUN --mount=type=cache

Ez a mount típus lehetővé teszi a build konténer számára, hogy gyorsítótárazza a könyvtárakat fordítók és csomagkezelők számára.

| Opció | Leírás |
| --- | --- |
| `id` | Opcionális ID különálló/különböző gyorsítótárak azonosításához. Alapértelmezésben a `target` értéke. |
| `target`, `dst`, `destination`¹ | Mount útvonal. |
| `ro`, `readonly` | Csak olvasható, ha be van állítva. |
| `sharing` | Egy a `shared`, `private` vagy `locked` közül. Alapértelmezésben `shared`. A `shared` gyorsítótár mount több író által egyidejűleg használható. A `private` új mountot hoz létre, ha több író van. A `locked` szünetelteti a második írót, amíg az első el nem engedi a mountot. |
| `from` | Build-szakasz, kontextus vagy image név a gyorsítótár mount alapjaként. Alapértelmezésben üres könyvtár. |
| `source` | Alútvonal a `from`-ban a csatoláshoz. Alapértelmezésben a `from` gyökere. |
| `mode` | Fájl mód az új gyorsítótár-könyvtárhoz oktálisban. Alapértelmezés 0755. |
| `uid` | Felhasználói ID az új gyorsítótár-könyvtárhoz. Alapértelmezés 0. |
| `gid` | Csoport ID az új gyorsítótár-könyvtárhoz. Alapértelmezés 0. |

A gyorsítótár-könyvtárak tartalma a builder hívások között megmarad anélkül, hogy érvénytelenítené az utasítás-gyorsítótárat. A gyorsítótár mountokat csak jobb teljesítmény érdekében szabad használni. A buildnek működnie kell a gyorsítótár-könyvtár bármely tartalmával, mivel egy másik build felülírhatja a fájlokat, vagy a GC tisztíthatja, ha több tárhelyre van szükség.

**Példa: Go csomagok gyorsítótárazása**

```dockerfile
# syntax=docker/dockerfile:1
FROM golang
RUN --mount=type=cache,target=/root/.cache/go-build \
  go build ...
```

**Példa: apt csomagok gyorsítótárazása**

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu
RUN rm -f /etc/apt/apt.conf.d/docker-clean; echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
  --mount=type=cache,target=/var/lib/apt,sharing=locked \
  apt-get update && apt-get --no-install-recommends install -y gcc
```

Az Apt-nek exkluzív hozzáférésre van szüksége az adataihoz, ezért a gyorsítótárak a `sharing=locked` opciót használják, amely biztosítja, hogy az ugyanazt a gyorsítótár mountot használó több párhuzamos build várjon egymásra, és ne férjenek hozzá ugyanazokhoz a gyorsítótár-fájlokhoz egyszerre. A `sharing=private`-et is használhatod, ha szeretnéd, hogy minden build egy másik gyorsítótár-könyvtárat hozzon létre ebben az esetben.

#### RUN --mount=type=tmpfs

Ez a mount típus lehetővé teszi tmpfs csatolását a build konténerben.

| Opció | Leírás |
| --- | --- |
| `target`, `dst`, `destination`¹ | Mount útvonal. |
| `size` | Felső korlát megadása a fájlrendszer méretére. |

#### RUN --mount=type=secret

Ez a mount típus lehetővé teszi a build konténer számára, hogy hozzáférjen titkos értékekhez, mint tokenek vagy privát kulcsok, anélkül, hogy beágyazná őket az image-be.

Alapértelmezésben a titok fájlként van csatolva. A titkot környezeti változóként is csatolhatod az `env` opció beállításával.

| Opció | Leírás |
| --- | --- |
| `id` | A titok ID-ja. Alapértelmezésben a célútvonal basename-je. |
| `target`, `dst`, `destination` | A titok csatolása a megadott útvonalra. Alapértelmezésben `/run/secrets/` + `id`, ha nincs beállítva és az `env` sincs beállítva. |
| `env` | A titok csatolása környezeti változóhoz fájl helyett, vagy mindkettőhöz. (a Dockerfile v1.10.0 óta) |
| `required` | Ha `true`-ra van állítva, az utasítás hibát ad, amikor a titok nem elérhető. Alapértelmezésben `false`. |
| `mode` | Fájl mód a titokfájlhoz oktálisban. Alapértelmezés 0400. |
| `uid` | Felhasználói ID a titokfájlhoz. Alapértelmezés 0. |
| `gid` | Csoport ID a titokfájlhoz. Alapértelmezés 0. |

> **Megjegyzés**
>
> Windows konténereken a titok egyetlen fájlként, csak olvasható bind mountként kerül átadásra (nincs tmpfs). Explicit cél szükséges, mert nincs alapértelmezett `/run/secrets/` hely, pl. `--mount=type=secret,id=mysecret,target=C:/path/to/secret`. Használj előre dőlt perjelet a célban: a Dockerfile escape karakter (alapértelmezésben `\`) egyébként felemészti a visszaperjeleket az értelmezés során. A `mode`, `uid` és `gid` opciók nem támogatottak Windows-on, és figyelmen kívül maradnak. A titok egy ideiglenes fájlba kerül írásra, amely explicit ACL-lel korlátozott a SYSTEM-re, az Administrators csoportra és a BuildKit démon fiókra, majd a lépés után eltávolításra kerül; nyílt szövegként kerül írásra, ezért fontold meg a BitLocker-t a nyugalmi titkosításhoz. Mivel a fájl nem kerül megadásra a konténer alapértelmezett felhasználójának, a RUN lépésnek adminisztrátorként kell futnia (pl. `USER ContainerAdministrator`) a titok olvasásához. A titok értéke nem marad meg az image-ben; azonban egy üres placeholder könyvtár maradhat a célútvonalon az eredményül kapott rétegben, ami a Windows bind mountok velejáró tulajdonsága.

**Példa: hozzáférés S3-hoz**

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3
RUN pip install awscli
RUN --mount=type=secret,id=aws,target=/root/.aws/credentials \
  aws s3 cp s3://... ...
```

```bash
docker buildx build --secret id=aws,src=$HOME/.aws/credentials .
```

**Példa: Mount környezeti változóként**

A következő példa az `API_KEY` titkot veszi, és ugyanazzal a névvel rendelkező környezeti változóként csatolja.

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
RUN --mount=type=secret,id=API_KEY,env=API_KEY \
    some-command --token-from-env $API_KEY
```

Feltételezve, hogy az `API_KEY` környezeti változó be van állítva a build környezetben, ezt a következő paranccsal építheted:

```bash
docker buildx build --secret id=API_KEY .
```

#### RUN --mount=type=ssh

Ez a mount típus lehetővé teszi a build konténer számára, hogy SSH kulcsokhoz férjen hozzá SSH agenteken keresztül, jelmondatok támogatásával.

| Opció | Leírás |
| --- | --- |
| `id` | Az SSH agent socket vagy kulcs ID-ja. Alapértelmezésben `"default"`. |
| `target`, `dst`, `destination` | SSH agent socket útvonal. Alapértelmezésben `/run/buildkit/ssh_agent.${N}`. |
| `required` | Ha `true`-ra van állítva, az utasítás hibát ad, amikor a kulcs nem elérhető. Alapértelmezésben `false`. |
| `mode` | Fájl mód a sockethez oktálisban. Alapértelmezés 0600. |
| `uid` | Felhasználói ID a sockethez. Alapértelmezés 0. |
| `gid` | Csoport ID a sockethez. Alapértelmezés 0. |

**Példa: hozzáférés GitLabhoz**

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
RUN apk add --no-cache openssh-client
RUN mkdir -p -m 0700 ~/.ssh && ssh-keyscan gitlab.com >> ~/.ssh/known_hosts
RUN --mount=type=ssh \
  ssh -q -T git@gitlab.com 2>&1 | tee /hello
# "Welcome to GitLab, @GITLAB_USERNAME_ASSOCIATED_WITH_SSHKEY" should be printed here
# with the type of build progress is defined as `plain`.
```

```bash
eval $(ssh-agent)
ssh-add ~/.ssh/id_rsa
(Input your passphrase here)
docker buildx build --ssh default=$SSH_AUTH_SOCK .
```

A hoszton lévő `*.pem` fájl elérési útját is megadhatod közvetlenül a `$SSH_AUTH_SOCK` helyett. A jelmondattal rendelkező pem fájlok azonban nem támogatottak.

### RUN --network

```dockerfile
RUN --network=TYPE
```

A `RUN --network` lehetővé teszi annak szabályozását, hogy a parancs mely hálózati környezetben fusson.

A támogatott hálózati típusok:

| Típus | Leírás |
| --- | --- |
| `default` (alapértelmezett) | Futtatás az alapértelmezett hálózatban. |
| `none` | Futtatás hálózati hozzáférés nélkül. |
| `host` | Futtatás a hoszt hálózati környezetében. |

#### RUN --network=default

Egyenértékű a flag egyáltalán nem való megadásával, a parancs a build alapértelmezett hálózatában fut.

#### RUN --network=none

A parancs hálózati hozzáférés nélkül fut (a `lo` még mindig elérhető, de erre a folyamatra izolált)

**Példa: külső hatások izolálása**

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.6
ADD mypackage.tgz wheels/
RUN --network=none pip install --find-links wheels mypackage
```

A pip csak a tarfile-ban biztosított csomagokat tudja telepíteni, amelyeket egy korábbi build-szakasz szabályozhat.

#### RUN --network=host

A parancs a hoszt hálózati környezetében fut (hasonlóan a `docker build --network=host`-hoz, de utasításonkénti alapon)

> ⚠️ **Figyelmeztetés**
>
> A `--network=host` használatát a `network.host` entitlement védi, amelyet engedélyezni kell a `buildkitd` démon `--allow-insecure-entitlement network.host` flaggel vagy a `buildkitd` configban való indításakor, valamint egy build kéréshez a `--allow network.host` flaggel.

### RUN --security

```dockerfile
RUN --security=<sandbox|insecure>
```

Az alapértelmezett biztonsági mód a `sandbox`. A `--security=insecure` esetén a builder a parancsot sandbox nélkül, nem biztonságos módban futtatja, ami lehetővé teszi emelt jogosultságokat igénylő folyamatok futtatását (pl. containerd). Ez egyenértékű a `docker run --privileged` futtatásával.

> ⚠️ **Figyelmeztetés**
>
> A funkció eléréséhez a `security.insecure` entitlementet engedélyezni kell a `buildkitd` démon `--allow-insecure-entitlement security.insecure` flaggel vagy a `buildkitd` configban való indításakor, valamint egy build kéréshez a `--allow security.insecure` flaggel.

Az alapértelmezett sandbox mód aktiválható a `--security=sandbox`-dal, de ez no-op.

**Példa: entitlementek ellenőrzése**

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu
RUN --security=insecure cat /proc/self/status | grep CapEff
```

```plaintext
#84 0.093 CapEff:	0000003fffffffff
```

## CMD

A `CMD` utasítás beállítja a parancsot, amelyet egy image-ből való konténer futtatásakor kell végrehajtani.

A `CMD` utasításokat shell vagy exec formában adhatod meg:

- `CMD ["executable","param1","param2"]` (exec forma)
- `CMD ["param1","param2"]` (exec forma, mint az ENTRYPOINT alapértelmezett paraméterei)
- `CMD command param1 param2` (shell forma)

Egy Dockerfile-ban csak egy `CMD` utasítás lehet. Ha több `CMD`-t sorolsz fel, csak az utolsó lép életbe.

A `CMD` célja, hogy alapértelmezéseket biztosítson egy futó konténerhez. Ezek az alapértelmezések tartalmazhatnak egy futtatható fájlt, vagy elhagyhatják a futtatható fájlt, amely esetben egy `ENTRYPOINT` utasítást is meg kell adnod.

Ha szeretnéd, hogy a konténered minden alkalommal ugyanazt a futtatható fájlt futtassa, akkor fontold meg az `ENTRYPOINT` használatát a `CMD`-vel kombinálva. Lásd: ENTRYPOINT. Ha a felhasználó argumentumokat ad meg a `docker run`-hoz, azok felülírják a `CMD`-ben megadott alapértelmezést, de továbbra is az alapértelmezett `ENTRYPOINT`-ot használják.

Ha a `CMD`-t az `ENTRYPOINT` utasítás alapértelmezett argumentumainak biztosítására használják, mind a `CMD`, mind az `ENTRYPOINT` utasítást exec formában kell megadni.

> **Megjegyzés**
>
> Ne keverd össze a `RUN`-t a `CMD`-vel. A `RUN` valójában futtat egy parancsot és commitolja az eredményt; a `CMD` nem hajt végre semmit build időben, hanem az image szándékolt parancsát adja meg.

## LABEL

```dockerfile
LABEL <key>=<value> [<key>=<value>...]
```

A `LABEL` utasítás metaadatokat ad egy image-hez. A `LABEL` egy kulcs-érték pár. A `LABEL` értékében lévő szóközök beillesztéséhez használj idézőjeleket és visszaperjeleket, ahogy a parancssori értelmezésben tennéd. Néhány használati példa:

```dockerfile
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```

Egy image-nek több címkéje is lehet. Egyetlen sorban több címkét is megadhatsz. A Docker 1.10 előtt ez csökkentette a végső image méretét, de ez már nem így van. Továbbra is választhatod több címke megadását egyetlen utasításban, a következő két mód egyikével:

```dockerfile
LABEL multi.label1="value1" multi.label2="value2" other="value3"
```

```dockerfile
LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
```

> **Megjegyzés**
>
> Ügyelj arra, hogy dupla idézőjeleket használj, ne szimpla idézőjeleket. Különösen, amikor string-interpolációt használsz (pl. `LABEL example="foo-$ENV_VAR"`), a szimpla idézőjelek a stringet úgy veszik, ahogy van, a változó értékének kicsomagolása nélkül.

Az alapimage-ekben (a `FROM` sorban lévő image-ek) található címkék öröklődnek az image-edbe. Ha egy címke már létezik, de más értékkel, a legutóbb alkalmazott érték felülírja a korábban beállított értéket.

Egy többlépcsős buildben a köztes szakaszokból származó címkék csak akkor vannak jelen a végső image-ben, ha a végső szakasz közvetlenül vagy közvetve azokon alapul (a `FROM`-on keresztül). Azokból a szakaszokból származó címkék, amelyekre csak `COPY --from`-tal vagy `RUN --mount=from=`-tal hivatkozol, nem kerülnek bele a kimeneti image-be. A végső `FROM` utasításban megadott alapimage címkéi mindig öröklődnek.

Egy image címkéinek megtekintéséhez használd a `docker image inspect` parancsot. A `--format` opcióval csak a címkéket jelenítheted meg;

```bash
docker image inspect --format='{{json .Config.Labels}}' myimage
```

```json
{
  "com.example.vendor": "ACME Incorporated",
  "com.example.label-with-value": "foo",
  "version": "1.0",
  "description": "This text illustrates that label-values can span multiple lines.",
  "multi.label1": "value1",
  "multi.label2": "value2",
  "other": "value3"
}
```

## MAINTAINER (elavult)

```dockerfile
MAINTAINER <name>
```

A `MAINTAINER` utasítás beállítja a generált image-ek Author mezőjét. A `LABEL` utasítás ennek sokkal rugalmasabb verziója, és helyette azt kell használnod, mivel lehetővé teszi bármilyen szükséges metaadat beállítását, és könnyen megtekinthető, például a `docker inspect`-tal. A `MAINTAINER` mezőnek megfelelő címke beállításához használhatod:

```dockerfile
LABEL org.opencontainers.image.authors="SvenDowideit@home.org.au"
```

Ez aztán a `docker inspect`-tal látható lesz a többi címkével együtt.

## EXPOSE

```dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

Az `EXPOSE` utasítás tájékoztatja a Docker-t, hogy a konténer futásidőben a megadott hálózati portokon figyel. Megadhatod, hogy a port TCP-n vagy UDP-n figyel-e, és az alapértelmezés a TCP, ha nem adsz meg protokollt.

Az `EXPOSE` utasítás valójában nem teszi közzé a portot. Ez egyfajta dokumentációként működik az image-et építő személy és a konténert futtató személy között arról, hogy mely portokat szándékoznak közzétenni. A port közzétételéhez a konténer futtatásakor használd a `-p` flaget a `docker run`-on egy vagy több port közzétételéhez és leképezéséhez, vagy a `-P` flaget az összes közzétett port közzétételéhez és magasabb rendű portokra való leképezéséhez.

Alapértelmezésben az `EXPOSE` TCP-t feltételez. UDP-t is megadhatsz:

```dockerfile
EXPOSE 80/udp
```

Mind TCP-n, mind UDP-n való közzétételhez két sort kell megadnod:

```dockerfile
EXPOSE 80/tcp
EXPOSE 80/udp
```

Ebben az esetben, ha a `-P`-t használod a `docker run`-nal, a port egyszer TCP-hez és egyszer UDP-hez lesz közzétéve. Ne feledd, hogy a `-P` egy ideiglenes magas rendű hoszt portot használ a hoszton, így a TCP és UDP nem ugyanazt a portot használja.

Az `EXPOSE` beállításoktól függetlenül felülírhatod őket futásidőben a `-p` flag használatával. Például

```bash
docker run -p 80:80/tcp -p 80:80/udp ...
```

A hosztrendszeren való porttovábbítás beállításához lásd a `-P` flag használatát. A `docker network` parancs támogatja hálózatok létrehozását a konténerek közötti kommunikációhoz anélkül, hogy specifikus portokat kellene közzétenni vagy publikálni, mivel a hálózathoz csatlakozó konténerek bármely porton kommunikálhatnak egymással. Részletes információért lásd a funkció áttekintését.

## ENV

```dockerfile
ENV <key>=<value> [<key>=<value>...]
```

Az `ENV` utasítás beállítja a `<key>` környezeti változót a `<value>` értékre. Ez az érték a build-szakasz összes későbbi utasításának környezetében lesz, és sokban inline is helyettesíthető. Az érték más környezeti változókra is értelmezésre kerül, így az idézőjel karakterek eltávolításra kerülnek, ha nincsenek escape-elve. A parancssori értelmezéshez hasonlóan idézőjelek és visszaperjelek használhatók a szóközök értékeken belüli beillesztéséhez.

Példa:

```dockerfile
ENV MY_NAME="John Doe"
ENV MY_DOG=Rex\ The\ Dog
ENV MY_CAT=fluffy
```

Az `ENV` utasítás lehetővé teszi több `<key>=<value> ...` változó egyidejű beállítását, és az alábbi példa ugyanazt a nettó eredményt adja a végső image-ben:

```dockerfile
ENV MY_NAME="John Doe" MY_DOG=Rex\ The\ Dog \
    MY_CAT=fluffy
```

Az `ENV` használatával beállított környezeti változók megmaradnak, amikor a konténert az eredményül kapott image-ből futtatják. Az értékeket a `docker inspect`-tal tekintheted meg, és a `docker run --env <key>=<value>`-vel változtathatod meg.

Egy szakasz örökli az összes környezeti változót, amelyet a szülő szakasza vagy bármely ős szakasz `ENV`-vel beállított. További információért lásd a multi-stage builds szakaszt a kézikönyvben.

A környezeti változó megmaradása váratlan mellékhatásokat okozhat. Például az `ENV DEBIAN_FRONTEND=noninteractive` beállítása megváltoztatja az `apt-get` viselkedését, és összezavarhatja az image-ed felhasználóit.

Ha egy környezeti változóra csak a build során van szükség, és nem a végső image-ben, fontold meg egy érték beállítását egyetlen parancshoz:

```dockerfile
RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y ...
```

Vagy használd az `ARG`-ot, amely nem marad meg a végső image-ben:

```dockerfile
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y ...
```

> **Megjegyzés: Alternatív szintaxis**
>
> Az `ENV` utasítás egy alternatív szintaxist is lehetővé tesz: `ENV <key> <value>`, elhagyva a `=`-t. Például:
>
> ```dockerfile
> ENV MY_VAR my-value
> ```
>
> Ez a szintaxis nem teszi lehetővé több környezeti változó beállítását egyetlen `ENV` utasításban, és zavaró lehet. Például a következő egyetlen környezeti változót (`ONE`) állít be `"TWO= THREE=world"` értékkel:
>
> ```dockerfile
> ENV ONE TWO= THREE=world
> ```
>
> Az alternatív szintaxis a visszafelé kompatibilitás érdekében támogatott, de a fenti okok miatt nem ajánlott, és egy jövőbeli kiadásban eltávolítható.

## ADD

Az `ADD`-nak két formája van. Az utóbbi forma szükséges a szóközt tartalmazó elérési utakhoz.

```dockerfile
ADD [OPTIONS] <src> ... <dest>
ADD [OPTIONS] ["<src>", ... "<dest>"]
```

Az elérhető `[OPTIONS]` opciók:

| Opció | Minimum Dockerfile verzió |
| --- | --- |
| `--keep-git-dir` | 1.1 |
| `--checksum` | 1.6 |
| `--chmod` | 1.2 |
| `--chown` | |
| `--link` | 1.4 |
| `--unpack` | 1.17 |
| `--exclude` | 1.19 |

Az `ADD` utasítás új fájlokat vagy könyvtárakat másol a `<src>`-ből, és hozzáadja azokat az image fájlrendszeréhez a `<dest>` útvonalon. A fájlok és könyvtárak másolhatók a build kontextusból, egy távoli URL-ről vagy egy Git repository-ból.

Az `ADD` és `COPY` utasítások funkcionálisan hasonlóak, de kissé eltérő célokat szolgálnak. Tanulj többet az ADD és COPY közötti különbségekről.

### Forrás

Az `ADD`-dal több forrásfájlt vagy könyvtárat is megadhatsz. Az utolsó argumentumnak mindig a célnak kell lennie. Például két fájl, a `file1.txt` és a `file2.txt` hozzáadásához a build kontextusból a `/usr/src/things/`-be a build konténerben:

```dockerfile
ADD file1.txt file2.txt /usr/src/things/
```

Ha több forrásfájlt adsz meg, akár közvetlenül, akár helyettesítő karakterrel, akkor a célnak könyvtárnak kell lennie (perjellel `/` kell végződnie).

Távoli helyről származó fájlok hozzáadásához megadhatsz egy URL-t vagy egy Git repository címét forrásként. Például:

```dockerfile
ADD https://example.com/archive.zip /usr/src/things/
ADD git@github.com:user/repo.git /usr/src/things/
```

A BuildKit felismeri a `<src>` típusát, és ennek megfelelően dolgozza fel.

- Ha a `<src>` egy helyi fájl vagy könyvtár, a könyvtár tartalma a megadott célba másolódik. Lásd: Adding files from the build context.
- Ha a `<src>` egy helyi tar archívum, akkor kicsomagolásra és kibontásra kerül a megadott célba. Lásd: Adding local tar archives.
- Ha a `<src>` egy URL, az URL tartalma letöltésre és a megadott célba helyezésre kerül. Lásd: Adding files from a URL.
- Ha a `<src>` egy Git repository, a repository a megadott célba klónozódik. Lásd: Adding files from a Git repository.

#### Fájlok hozzáadása a build kontextusból

Minden relatív vagy helyi útvonal, amely nem `http://`, `https://` vagy `git@` protokoll előtaggal kezdődik, helyi fájlútvonalnak számít. A helyi fájlútvonal a build kontextushoz viszonyított. Például, ha a build kontextus az aktuális könyvtár, az `ADD file.txt /` a `./file.txt` fájlt adja hozzá a build konténer fájlrendszerének gyökeréhez.

Egy vezető perjellel rendelkező vagy a build kontextuson kívülre navigáló forrásútvonal megadása, mint az `ADD ../something /something`, automatikusan eltávolítja a szülőkönyvtár-navigációt (`../`). A forrásútvonalban lévő záró perjelek szintén figyelmen kívül maradnak, így az `ADD something/ /something` egyenértékű az `ADD something /something`-tel.

Ha a forrás egy könyvtár, a könyvtár tartalma másolódik, beleértve a fájlrendszer-metaadatokat is. Maga a könyvtár nem másolódik, csak a tartalma. Ha alkönyvtárakat tartalmaz, ezek is másolódnak, és egyesülnek a célban lévő meglévő könyvtárakkal. Bármely ütközés a hozzáadott tartalom javára oldódik fel, fájlonkénti alapon, kivéve, ha egy könyvtárat próbálsz másolni egy meglévő fájlra, amely esetben hiba merül fel.

Ha a forrás egy fájl, a fájl és metaadatai a célba másolódnak. A fájljogosultságok megmaradnak. Ha a forrás egy fájl, és egy azonos nevű könyvtár létezik a célban, hiba merül fel.

Ha egy Dockerfile-t stdin-en keresztül adsz át a buildhez (`docker build - < Dockerfile`), nincs build kontextus. Ebben az esetben csak az `ADD` utasítást használhatod távoli fájlok másolására. Egy tar archívumot is átadhatsz stdin-en keresztül: (`docker build - < archive.tar`), az archívum gyökerében lévő Dockerfile és az archívum többi része lesz a build kontextusa.

#### Mintázatillesztés

Helyi fájlok esetében minden `<src>` tartalmazhat helyettesítő karaktereket, és az illesztés a Go `filepath.Match` szabályai szerint történik.

Például az összes `.png`-vel végződő fájl és könyvtár hozzáadásához a build kontextus gyökerében:

```dockerfile
ADD *.png /dest/
```

A következő példában a `?` egyetlen karaktert helyettesítő helyettesítő karakter, amely illeszkedik pl. az `index.js`-re és az `index.ts`-re.

```dockerfile
ADD index.?s /dest/
```

Amikor speciális karaktereket (mint `[` és `]`) tartalmazó fájlokat vagy könyvtárakat adsz hozzá, ezeket az útvonalakat a Golang szabályai szerint kell escape-elned, hogy megakadályozd, hogy illesztési mintaként kezeljék őket. Például egy `arr[0].txt` nevű fájl hozzáadásához használd a következőt;

```dockerfile
ADD arr[[]0].txt /dest/
```

#### Helyi tar archívumok hozzáadása

Amikor egy helyi tar archívumot használsz az `ADD` forrásaként, és az archívum egy felismert tömörítési formátumban van (gzip, bzip2, xz vagy zstd, vagy tömörítetlen), az archívum kicsomagolásra és kibontásra kerül a megadott célba. A helyi tar archívumok alapértelmezésben kibontásra kerülnek, lásd az [ADD --unpack flag] részt.

Amikor egy könyvtárat kicsomagolnak, ugyanaz a viselkedése, mint a `tar -x`-nek. Az eredmény a következők uniója:

- Bármi, ami a célútvonalon létezett, és
- A forrásfa tartalma, az ütközések a hozzáadott tartalom javára oldódnak fel, fájlonkénti alapon.

> **Megjegyzés**
>
> Az, hogy egy fájlt felismert tömörítési formátumként azonosítanak-e vagy sem, kizárólag a fájl tartalma alapján történik, nem a fájl neve alapján. Például, ha egy üres fájl véletlenül `.tar.gz`-re végződik, ez nem kerül felismerésre tömörített fájlként, és nem generál semmilyen kicsomagolási hibaüzenetet, hanem a fájl egyszerűen a célba másolódik.

#### Fájlok hozzáadása URL-ről

Abban az esetben, ha a forrás egy távoli fájl URL, a cél 600-as jogosultságokkal rendelkezik. Ha a HTTP válasz tartalmaz egy `Last-Modified` fejlécet, akkor a fejlécből származó időbélyeg kerül felhasználásra a célfájl mtime-jának beállításához. Azonban, mint bármely más, az `ADD` során feldolgozott fájl esetében, az mtime nem kerül bele annak meghatározásába, hogy a fájl megváltozott-e, és a gyorsítótárat frissíteni kell-e.

Ha a távoli fájl egy tar archívum, az archívum alapértelmezésben nem kerül kibontásra. Az archívum letöltéséhez és kibontásához használd az [ADD --unpack flag] részt.

Ha a cél záró perjellel végződik, akkor a fájlnevet az URL útvonalából következtetik. Például az `ADD http://example.com/foobar /` létrehozná a `/foobar` fájlt. Az URL-nek nem triviális útvonallal kell rendelkeznie, hogy megfelelő fájlnév legyen felfedezhető (`http://example.com` nem működik).

Ha a cél nem záró perjellel végződik, a célútvonal lesz a letöltött fájl fájlneve az URL-ről. Például az `ADD http://example.com/foo /bar` létrehozná a `/bar` fájlt.

Ha az URL-fájljaid hitelesítéssel védettek, a `RUN wget`, `RUN curl` vagy egy másik eszköz használata szükséges a konténeren belül, mivel az `ADD` utasítás nem támogatja a hitelesítést.

#### Titkok

A `HTTP_AUTH_HEADER_<host>` és `HTTP_AUTH_TOKEN_<host>` titkokat használhatod a távoli források hitelesítő adatainak beállításához. További információért lásd: Build secrets.

#### Fájlok hozzáadása Git repository-ból

Ha egy Git repository-t szeretnél használni az `ADD` forrásaként, hivatkozhatsz a repository HTTP vagy SSH címére forrásként. A repository a megadott célba klónozódik az image-ben.

```dockerfile
ADD https://github.com/user/repo.git /mydir/
```

URL-töredékekkel megadhatsz egy adott ágat, címkét, commitot vagy alkönyvtárat. Például a buildkit repository v0.14.1 címkéjének docs könyvtárának hozzáadásához:

```dockerfile
ADD git@github.com:moby/buildkit.git#v0.14.1:docs /buildkit-docs
```

A Git URL-töredékekről további információért lásd: URL fragments.

Amikor Git repository-ból adsz hozzá, a fájlok jogosultsági bitjei 644. Ha egy fájl a repository-ban rendelkezik a futtatható bittel, a jogosultságai 755-re lesznek állítva. A könyvtárak jogosultságai 755-re vannak állítva.

Amikor Git repository-t használsz forrásként, a repository-nak elérhetőnek kell lennie a build kontextusból. Egy repository SSH-n keresztüli hozzáadásához, akár nyilvános, akár privát, át kell adnod egy SSH kulcsot a hitelesítéshez. Például, adott a következő Dockerfile:

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
ADD git@git.example.com:foo/bar.git /bar
```

Ennek a Dockerfile-nak az építéséhez add át a `--ssh` flaget a `docker build`-nek, hogy csatolja az SSH agent socketet a buildhez. Például:

```bash
docker build --ssh default .
```

A titkokkal való építésről további információért lásd: Build secrets.

### Cél

Ha a célútvonal egy előre dőlt perjellel kezdődik, abszolút útvonalként értelmeződik, és a forrásfájlok a megadott célba másolódnak az aktuális build-szakasz gyökeréhez viszonyítva.

```dockerfile
# create /abs/test.txt
ADD test.txt /abs/
```

A záró perjelek jelentősek. Például az `ADD test.txt /abs` létrehoz egy fájlt a `/abs`-on, míg az `ADD test.txt /abs/` létrehozza a `/abs/test.txt`-t.

Ha a célútvonal nem kezdődik vezető perjellel, akkor a build konténer munkakönyvtárához viszonyítottan értelmeződik.

```dockerfile
WORKDIR /usr/src/app
# create /usr/src/app/rel/test.txt
ADD test.txt rel/
```

Ha a cél nem létezik, létrejön, az összes hiányzó könyvtárral együtt az útvonalában.

Ha a forrás egy fájl, és a cél nem záró perjellel végződik, a forrásfájl a célútvonalra lesz írva fájlként.

### ADD --keep-git-dir

```dockerfile
ADD [--keep-git-dir=<boolean>] <src> ... <dir>
```

Amikor a `<src>` egy távoli Git repository HTTP vagy SSH címe, a BuildKit a Git repository tartalmát az image-hez adja hozzá, alapértelmezésben kizárva a `.git` könyvtárat.

A `--keep-git-dir=true` flag lehetővé teszi a `.git` könyvtár megőrzését.

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
ADD --keep-git-dir=true https://github.com/moby/buildkit.git#v0.10.1 /buildkit
```

### ADD --checksum

```dockerfile
ADD [--checksum=<hash>] <src> ... <dir>
```

A `--checksum` flag lehetővé teszi egy távoli Git vagy HTTP erőforrás ellenőrzőösszegének ellenőrzését:

- Git források esetében az ellenőrzőösszeg a commit SHA. Lehet a teljes commit SHA, vagy illeszkedhet az előtagra (1 vagy több karakter).
- HTTP források esetében az ellenőrzőösszeg a SHA-256 tartalom digest, `sha256:<hash>` formátumban. A SHA-256 az egyetlen támogatott hash algoritmus.

```dockerfile
ADD --checksum=be1f38e https://github.com/moby/buildkit.git#v0.26.2 /
ADD --checksum=sha256:24454f830cdb571e2c4ad15481119c43b3cafd48dd869a9b2945d1036d1dc68d https://mirrors.edge.kernel.org/pub/linux/kernel/Historic/linux-0.01.tar.gz /
```

### ADD --chmod

Lásd: COPY --chmod.

### ADD --chown

Lásd: COPY --chown.

### ADD --link

Lásd: COPY --link.

### ADD --unpack

```dockerfile
ADD [--unpack=<bool>] <src> ... <dir>
```

A `--unpack` flag szabályozza, hogy automatikusan kicsomagolja-e a tar archívumokat (beleértve a tömörített formátumokat, mint a gzip, bzip2, xz vagy zstd), amikor hozzáadja őket az image-hez. A helyi tar archívumok alapértelmezésben kibontásra kerülnek, míg a távoli tar archívumok (ahol a `src` egy URL) letöltésre kerülnek kicsomagolás nélkül.

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
# Download and unpack archive.tar.gz into /download:
ADD --unpack=true https://example.com/archive.tar.gz /download
# Add local tar without unpacking:
ADD --unpack=false my-archive.tar.gz .
```

### ADD --exclude

Lásd: COPY --exclude.

## COPY

A `COPY`-nak két formája van. Az utóbbi forma szükséges a szóközt tartalmazó elérési utakhoz.

```dockerfile
COPY [OPTIONS] <src> ... <dest>
COPY [OPTIONS] ["<src>", ... "<dest>"]
```

Az elérhető `[OPTIONS]` opciók:

| Opció | Minimum Dockerfile verzió |
| --- | --- |
| `--from` | |
| `--chmod` | 1.2 |
| `--chown` | |
| `--link` | 1.4 |
| `--parents` | 1.20 |
| `--exclude` | 1.19 |

A `COPY` utasítás új fájlokat vagy könyvtárakat másol a `<src>`-ből, és hozzáadja azokat az image fájlrendszeréhez a `<dest>` útvonalon. A fájlok és könyvtárak másolhatók a build kontextusból, build-szakaszból, nevesített kontextusból vagy egy image-ből.

Az `ADD` és `COPY` utasítások funkcionálisan hasonlóak, de kissé eltérő célokat szolgálnak. Tanulj többet az ADD és COPY közötti különbségekről.

### Forrás

A `COPY`-val több forrásfájlt vagy könyvtárat is megadhatsz. Az utolsó argumentumnak mindig a célnak kell lennie. Például két fájl, a `file1.txt` és a `file2.txt` másolásához a build kontextusból a `/usr/src/things/`-be a build konténerben:

```dockerfile
COPY file1.txt file2.txt /usr/src/things/
```

Ha több forrásfájlt adsz meg, akár közvetlenül, akár helyettesítő karakterrel, akkor a célnak könyvtárnak kell lennie (perjellel `/` kell végződnie).

A `COPY` elfogad egy `--from=<name>` flaget, amely lehetővé teszi a forrás helyének egy build-szakasz, kontextus vagy image megadását. A következő példa fájlokat másol egy `build` nevű szakaszból:

```dockerfile
FROM golang AS build
WORKDIR /app
RUN --mount=type=bind,target=. go build -o /myapp ./cmd

COPY --from=build /myapp /usr/bin/
```

A nevesített forrásokból való másolásról további információért lásd a `--from` flaget.

#### Másolás a build kontextusból

Amikor forrásfájlokat másolsz a build kontextusból, az útvonalak a kontextus gyökeréhez viszonyítottan értelmeződnek.

Egy vezető perjellel rendelkező vagy a build kontextuson kívülre navigáló forrásútvonal megadása, mint a `COPY ../something /something`, automatikusan eltávolítja a szülőkönyvtár-navigációt (`../`). A forrásútvonalban lévő záró perjelek szintén figyelmen kívül maradnak, így a `COPY something/ /something` egyenértékű a `COPY something /something`-tel.

Ha a forrás egy könyvtár, a könyvtár tartalma másolódik, beleértve a fájlrendszer-metaadatokat is. Maga a könyvtár nem másolódik, csak a tartalma. Ha alkönyvtárakat tartalmaz, ezek is másolódnak, és egyesülnek a célban lévő meglévő könyvtárakkal. Bármely ütközés a hozzáadott tartalom javára oldódik fel, fájlonkénti alapon, kivéve, ha egy könyvtárat próbálsz másolni egy meglévő fájlra, amely esetben hiba merül fel.

Ha a forrás egy fájl, a fájl és metaadatai a célba másolódnak. A fájljogosultságok megmaradnak. Ha a forrás egy fájl, és egy azonos nevű könyvtár létezik a célban, hiba merül fel.

Ha egy Dockerfile-t stdin-en keresztül adsz át a buildhez (`docker build - < Dockerfile`), nincs build kontextus. Ebben az esetben csak a `COPY` utasítást használhatod fájlok másolására más szakaszokból, nevesített kontextusokból vagy image-ekből, a `--from` flag használatával. Egy tar archívumot is átadhatsz stdin-en keresztül: (`docker build - < archive.tar`), az archívum gyökerében lévő Dockerfile és az archívum többi része lesz a build kontextusa.

Amikor Git repository-t használsz build kontextusként, a másolt fájlok jogosultsági bitjei 644. Ha egy fájl a repository-ban rendelkezik a futtatható bittel, a jogosultságai 755-re lesznek állítva. A könyvtárak jogosultságai 755-re vannak állítva.

#### Mintázatillesztés

Helyi fájlok esetében minden `<src>` tartalmazhat helyettesítő karaktereket, és az illesztés a Go `filepath.Match` szabályai szerint történik.

Például az összes `.png`-vel végződő fájl és könyvtár hozzáadásához a build kontextus gyökerében:

```dockerfile
COPY *.png /dest/
```

A következő példában a `?` egyetlen karaktert helyettesítő helyettesítő karakter, amely illeszkedik pl. az `index.js`-re és az `index.ts`-re.

```dockerfile
COPY index.?s /dest/
```

Amikor speciális karaktereket (mint `[` és `]`) tartalmazó fájlokat vagy könyvtárakat adsz hozzá, ezeket az útvonalakat a Golang szabályai szerint kell escape-elned, hogy megakadályozd, hogy illesztési mintaként kezeljék őket. Például egy `arr[0].txt` nevű fájl hozzáadásához használd a következőt;

```dockerfile
COPY arr[[]0].txt /dest/
```

### Cél

Ha a célútvonal egy előre dőlt perjellel kezdődik, abszolút útvonalként értelmeződik, és a forrásfájlok a megadott célba másolódnak az aktuális build-szakasz gyökeréhez viszonyítva.

```dockerfile
# create /abs/test.txt
COPY test.txt /abs/
```

A záró perjelek jelentősek. Például a `COPY test.txt /abs` létrehoz egy fájlt a `/abs`-on, míg a `COPY test.txt /abs/` létrehozza a `/abs/test.txt`-t.

Ha a célútvonal nem kezdődik vezető perjellel, akkor a build konténer munkakönyvtárához viszonyítottan értelmeződik.

```dockerfile
WORKDIR /usr/src/app
# create /usr/src/app/rel/test.txt
COPY test.txt rel/
```

Ha a cél nem létezik, létrejön, az összes hiányzó könyvtárral együtt az útvonalában.

Ha a forrás egy fájl, és a cél nem záró perjellel végződik, a forrásfájl a célútvonalra lesz írva fájlként.

### COPY --from

Alapértelmezésben a `COPY` utasítás fájlokat másol a build kontextusból. A `COPY --from` flag lehetővé teszi fájlok másolását egy image-ből, egy build-szakaszból vagy egy nevesített kontextusból helyette.

```dockerfile
COPY [--from=<image|stage|context>] <src> ... <dest>
```

Egy build-szakaszból való másoláshoz egy többlépcsős buildben add meg annak a szakasznak a nevét, amelyből másolni szeretnél. A szakaszneveket az `AS` kulcsszóval adod meg a `FROM` utasítással.

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine AS build
COPY . .
RUN apk add clang
RUN clang -o /hello hello.c

FROM scratch
COPY --from=build /hello /
```

Fájlokat közvetlenül nevesített kontextusokból (amelyeket `--build-context <name>=<source>`-szal adsz meg) vagy image-ekből is másolhatsz. A következő példa egy `nginx.conf` fájlt másol a hivatalos Nginx image-ből.

```dockerfile
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```

A `COPY --from` forrásútvonala mindig annak az image-nek vagy szakasznak a fájlrendszer gyökeréből oldódik fel, amelyet megadsz.

### COPY --chmod

```dockerfile
COPY [--chmod=<perms>] <src> ... <dest>
```

A `--chmod` flag támogatja az oktális jelölést (pl. `755`, `644`) és a szimbolikus jelölést (pl. `+x`, `g=u`). A szimbolikus jelölés (a Dockerfile 1.14 verziója óta hozzáadva) akkor hasznos, ha az oktális nem elég rugalmas. Például az `u=rwX,go=rX` a könyvtárakat 755-re, a fájlokat 644-re állítja, miközben megőrzi a már futtatható fájlok futtatható bitjét. (A nagy `X` azt jelenti, hogy „csak akkor futtatható, ha könyvtár, vagy már futtatható”.)

A szimbolikus jelölés szintaxisáról további információért lásd a `chmod(1)` kézikönyvet.

Példák oktális jelöléssel:

```dockerfile
COPY --chmod=755 app.sh /app/
COPY --chmod=644 file.txt /data/
ARG MODE=440
COPY --chmod=$MODE . .
```

Példák szimbolikus jelöléssel:

```dockerfile
COPY --chmod=+x script.sh /app/
COPY --chmod=u=rwX,go=rX . /app/
COPY --chmod=g=u config/ /config/
```

A `--chmod` flag nem támogatott Windows konténerek építésekor.

### COPY --chown

```dockerfile
COPY [--chown=<user>:<group>] <src> ... <dest>
```

Beállítja a másolt fájlok tulajdonjogát. A flag nélkül a fájlok 0-s UID-dal és GID-del jönnek létre.

A flag elfogad felhasználóneveket, csoportneveket, UID-kat vagy GID-ket bármilyen kombinációban. Ha csak egy felhasználót adsz meg, a GID ugyanarra a numerikus értékre lesz állítva, mint az UID.

```dockerfile
COPY --chown=55:mygroup files* /somedir/
COPY --chown=bin files* /somedir/
COPY --chown=1 files* /somedir/
COPY --chown=10:11 files* /somedir/
COPY --chown=myuser:mygroup --chmod=644 files* /somedir/
```

Amikor numerikus ID-k helyett neveket használsz, a BuildKit azokat a konténer gyökérfájlrendszerében lévő `/etc/passwd` és `/etc/group` segítségével oldja fel. Ha ezek a fájlok hiányoznak, vagy nem tartalmazzák a megadott neveket, a build meghiúsul. A numerikus ID-k nem igénylik ezt a feloldást.

A `--chown` flag nem támogatott Windows konténerek építésekor.

### COPY --link

```dockerfile
COPY [--link[=<boolean>]] <src> ... <dest>
```

Ennek a flagnek a engedélyezése a `COPY` vagy `ADD` parancsokban lehetővé teszi fájlok másolását fokozott szemantikával, ahol a fájljaid a saját rétegükön maradnak függetlenül, és nem érvénytelenítődnek, amikor az előző rétegeken lévő parancsok megváltoznak.

Amikor a `--link`-et használod, a forrásfájljaid egy üres célkönyvtárba másolódnak. Az a könyvtár egy réteggé alakul, amely az előző állapotod tetejére van linkelve.

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
COPY --link /foo /bar
```

Egyenértékű két build végrehajtásával:

```dockerfile
FROM alpine
```

és

```dockerfile
FROM scratch
COPY /foo /bar
```

és mindkét image összes rétegének összeolvasztásával.

#### A --link használatának előnyei

Használd a `--link`-et a már felépített rétegek újrafelhasználásához a későbbi buildekben `--cache-from`-mal, még akkor is, ha az előző rétegek megváltoztak. Ez különösen fontos a többlépcsős buildeknél, ahol egy `COPY --from` utasítás korábban érvénytelenítődött, ha bármely előző parancs ugyanabban a szakaszban megváltozott, ami a köztes szakaszok újbóli felépítését okozta. A `--link`-kel az előző build által generált réteg újrafelhasználásra kerül, és az új rétegek tetejére olvad. Ez azt is jelenti, hogy könnyedén rebase-elheted az image-eidet, amikor az alapimage-ek frissítéseket kapnak, anélkül, hogy a teljes buildet újra kellene futtatnod. Az ezt támogató backendekben a BuildKit elvégezheti ezt a rebase műveletet anélkül, hogy bármely réteget pusholni vagy pullolni kellene a kliens és a regisztrum között. A BuildKit felismeri ezt az esetet, és csak új image manifestet hoz létre, amely a megfelelő sorrendben tartalmazza az új és régi rétegeket.

Ugyanaz a viselkedés, ahol a BuildKit elkerülheti az alapimage letöltését, akkor is előfordulhat, ha a `--link`-et használod, és nincs más parancs, amely hozzáférést igényelne az alapimage fájljaihoz. Ebben az esetben a BuildKit csak a `COPY` parancsok rétegeit építi fel, és azokat közvetlenül a regisztrumba pusholja az alapimage rétegeinek tetejére.

#### Inkompatibilitások a --link=false használatával

A `--link` használatakor a `COPY`/`ADD` parancsok nem olvashatnak fájlokat az előző állapotból. Ez azt jelenti, hogy ha az előző állapotban a célkönyvtár egy szimbolikus linket tartalmazó útvonal volt, a `COPY`/`ADD` nem tudja követni. A végső image-ben a `--link`-kel létrehozott célútvonal mindig csak könyvtárakat tartalmazó útvonal lesz.

Ha nem támaszkodsz a célútvonalban lévő szimbolikus linkek követésének viselkedésére, a `--link` használata mindig ajánlott. A `--link` teljesítménye egyenértékű vagy jobb, mint az alapértelmezett viselkedés, és sokkal jobb feltételeket teremt a gyorsítótár újrafelhasználásához.

Amikor egy útvonalat egy alkönyvtárba másolsz, a `--link` mindig a fájlrendszer gyökeréből másol. Egy könyvtár másolásakor a meglévő mód felülírásra kerül a másolt útvonal új módjával. Ha egy könyvtárhoz konkrét módra van szükséged, például a megengedőbb `/tmp` könyvtárhoz, lehet, hogy el kell kerülnöd a `--link` használatát, fel kell bontanod a másolást alapösszetevőire, vagy a `--chmod`-ot kell használnod annak biztosítására, hogy a felülíró könyvtár ugyanazokat a jogosultságokat tartalmazza.

### COPY --parents

```dockerfile
COPY [--parents[=<boolean>]] <src> ... <dest>
```

A `--parents` flag megőrzi a szülőkönyvtárakat a `src` bejegyzésekhez. Ez a flag alapértelmezésben `false`.

```dockerfile
# syntax=docker/dockerfile:1
FROM scratch

COPY ./x/a.txt ./y/a.txt /no_parents/
COPY --parents ./x/a.txt ./y/a.txt /parents/

# /no_parents/a.txt
# /parents/x/a.txt
# /parents/y/a.txt
```

Ez a viselkedés hasonló a Linux `cp` segédprogram `--parents` vagy az `rsync --relative` flagjéhez.

Az Rsync-hez hasonlóan lehetséges korlátozni, hogy mely szülőkönyvtárak kerüljenek megőrzésre, ha egy pontot és egy perjelet (`./`) illesztesz be a forrásútvonalba. Ha ilyen pont létezik, csak az utána lévő szülőkönyvtárak kerülnek megőrzésre. Ez különösen hasznos lehet a szakaszok közötti másolásoknál `--from`-mal, ahol a forrásútvonalaknak abszolútnak kell lenniük.

```dockerfile
# syntax=docker/dockerfile:1
FROM scratch

COPY --parents ./x/./y/*.txt /parents/

# Build context:
# ./x/y/a.txt
# ./x/y/b.txt
#
# Output:
# /parents/y/a.txt
# /parents/y/b.txt
```

A `**` helyettesítő karakter bármennyi útvonalkomponensre illeszkedik, beleértve a nullát is, és rekurzívan illeszthet fájlokat a könyvtárszinteken keresztül:

```dockerfile
# syntax=docker/dockerfile:1
FROM scratch

COPY --parents ./src/**/*.txt /parents/

# Build context:
# ./src/a.txt
# ./src/x/b.txt
# ./src/x/y/c.txt
#
# Output:
# /parents/src/a.txt
# /parents/src/x/b.txt
# /parents/src/x/y/c.txt
```

Ne feledd, hogy a `--parents` flag megadása nélkül bármely fájlnév-ütközés meghiúsítja a Linux `cp` műveletet egy explicit hibaüzenettel (`cp: will not overwrite just-created './x/a.txt' with './y/a.txt'`), ahol a Buildkit csendesen felülírja a célfájlt a célban.

Bár lehetséges megőrizni a könyvtárszerkezetet csak egyetlen `src` bejegyzésből álló `COPY` utasításokhoz, általában előnyösebb a rétegszámot a lehető legalacsonyabban tartani az eredményül kapott image-ben. Ezért a `--parents` flaggel a Buildkit képes több `COPY` utasítást együtt csomagolni, a könyvtárszerkezet érintetlenül hagyásával.

### COPY --exclude

```dockerfile
COPY [--exclude=<path> ...] <src> ... <dest>
```

A `--exclude` flag lehetővé teszi egy elérésiút-kifejezés megadását a kizárandó fájlokhoz.

Az elérésiút-kifejezés ugyanazt a formátumot követi, mint a `<src>`, támogatva a helyettesítő karaktereket és a Go `filepath.Match` szabályai szerinti illesztést. Például az összes „hom”-mal kezdődő fájl hozzáadásához, kizárva a `.txt` kiterjesztésű fájlokat:

```dockerfile
# syntax=docker/dockerfile:1
FROM scratch

COPY --exclude=*.txt hom* /mydir/
```

A `--exclude` opciót többször is megadhatod egy `COPY` utasításhoz. A megadott `--exclude` minták bármelyikének megfelelő fájlok nem kerülnek másolásra, még akkor sem, ha az útvonaluk megegyezik a `<src>`-ben megadott mintával. Az összes „hom”-mal kezdődő fájl hozzáadásához, kizárva a `.txt` vagy `.md` kiterjesztésű fájlokat:

```dockerfile
# syntax=docker/dockerfile:1
FROM scratch

COPY --exclude=*.txt --exclude=*.md hom* /mydir/
```

## ENTRYPOINT

Az `ENTRYPOINT` lehetővé teszi egy konténer konfigurálását, amely futtatható fájlként fut.

Az `ENTRYPOINT`-nak két lehetséges formája van:

- Az exec forma, amely az előnyben részesített forma:

  ```dockerfile
  ENTRYPOINT ["executable", "param1", "param2"]
  ```

- A shell forma:

  ```dockerfile
  ENTRYPOINT command param1 param2
  ```

A különböző formákról további információért lásd: Shell and exec form.

A következő parancs elindít egy konténert az `nginx`-ből annak alapértelmezett tartalmával, a 80-as porton figyelve:

```bash
docker run -i -t --rm -p 80:80 nginx
```

A `docker run <image>` parancssori argumentumai az exec forma `ENTRYPOINT` összes eleme után fűződnek hozzá, és felülírják a `CMD`-vel megadott összes elemet.

Ez lehetővé teszi argumentumok átadását a belépési pontnak, azaz a `docker run <image> -d` átadja a `-d` argumentumot a belépési pontnak. Az `ENTRYPOINT` utasítást a `docker run --entrypoint` flaggel írhatod felül.

Az `ENTRYPOINT` shell formája figyelmen kívül hagyja a `CMD`-t vagy a `docker run` parancssori argumentumait. Az `ENTRYPOINT`-odat a `/bin/sh -c` alparancsaként indítja, amely nem továbbítja a szignálokat. Ez azt jelenti, hogy a futtatható fájl nem lesz a konténer PID 1-je, és nem fog Unix szignálokat kapni. Ebben az esetben a futtatható fájlod nem kap SIGTERM-et a `docker stop <container>`-től.

Csak az utolsó `ENTRYPOINT` utasítás lesz hatással a Dockerfile-ban.

### Exec forma ENTRYPOINT példa

Az `ENTRYPOINT` exec formáját használhatod meglehetősen stabil alapértelmezett parancsok és argumentumok beállítására, majd a `CMD`-t további, valószínűleg megváltoztatandó alapértelmezések beállítására.

Amikor az exec forma `ENTRYPOINT`-ot a `CMD`-vel kombinálod, használd a `CMD` exec formáját is. A `CMD` shell formájának használata azt eredményezi, hogy `/bin/sh -c`-be csomagolódik, ami azt jelenti, hogy az `ENTRYPOINT` egy shell meghívást kap argumentumként a csupasz parancs és paraméterek helyett. Lásd: Understand how CMD and ENTRYPOINT interact.

```dockerfile
FROM ubuntu
ENTRYPOINT ["top", "-b"]
CMD ["-c"]
```

Amikor futtatod a konténert, láthatod, hogy a `top` az egyetlen folyamat:

```bash
docker run -it --rm --name test  top -H
```

```plaintext
top - 08:25:00 up  7:27,  0 users,  load average: 0.00, 0.01, 0.05
Threads:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:   2056668 total,  1616832 used,   439836 free,    99352 buffers
KiB Swap:  1441840 total,        0 used,  1441840 free.  1324440 cached Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
    1 root      20   0   19744   2336   2080 R  0.0  0.1   0:00.04 top
```

Az eredmény további vizsgálatához használhatod a `docker exec`-et:

```bash
docker exec -it test ps aux
```

```plaintext
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  2.6  0.1  19752  2352 ?        Ss+  08:24   0:00 top -b -H
root         7  0.0  0.1  15572  2164 ?        R+   08:25   0:00 ps aux
```

És kecsesen kérheted a `top` leállítását a `docker stop test`-tel.

A következő Dockerfile az `ENTRYPOINT` használatát mutatja az Apache előtérben (azaz PID 1-ként) való futtatásához:

```dockerfile
FROM debian:stable
RUN apt-get update && apt-get install -y --force-yes apache2
EXPOSE 80 443
VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

Ha egy indító szkriptet kell írnod egyetlen futtatható fájlhoz, biztosíthatod, hogy a végső futtatható fájl megkapja a Unix szignálokat az `exec` és `gosu` parancsok használatával:

```bash
#!/usr/bin/env bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```

Végül, ha extra takarítást kell végezned (vagy más konténerekkel kell kommunikálnod) leállításkor, vagy több futtatható fájlt koordinálsz, lehet, hogy biztosítanod kell, hogy az `ENTRYPOINT` szkript megkapja a Unix szignálokat, továbbítsa azokat, majd végezzen még némi munkát:

```bash
#!/bin/sh
# Note: I've written this using sh so it works in the busybox container too

# USE the trap if you need to also do manual cleanup after the service is stopped,
#     or need to start multiple services in the one container
trap "echo TRAPed signal" HUP INT QUIT TERM

# start service in background here
/usr/sbin/apachectl start

echo "[hit enter key to exit] or run 'docker stop <container>'"
read

# stop service and clean up here
echo "stopping apache"
/usr/sbin/apachectl stop

echo "exited $0"
```

Ha ezt az image-et a `docker run -it --rm -p 80:80 --name test apache` paranccsal futtatod, akkor megvizsgálhatod a konténer folyamatait a `docker exec`-kel vagy a `docker top`-tal, majd kérheted a szkriptet az Apache leállítására:

```bash
docker exec -it test ps aux
```

```plaintext
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  0.0   4448   692 ?        Ss+  00:42   0:00 /bin/sh /run.sh 123 cmd cmd2
root        19  0.0  0.2  71304  4440 ?        Ss   00:42   0:00 /usr/sbin/apache2 -k start
www-data    20  0.2  0.2 360468  6004 ?        Sl   00:42   0:00 /usr/sbin/apache2 -k start
www-data    21  0.2  0.2 360468  6000 ?        Sl   00:42   0:00 /usr/sbin/apache2 -k start
root        81  0.0  0.1  15572  2140 ?        R+   00:44   0:00 ps aux
```

```bash
docker top test
```

```plaintext
PID                 USER                COMMAND
10035               root                {run.sh} /bin/sh /run.sh 123 cmd cmd2
10054               root                /usr/sbin/apache2 -k start
10055               33                  /usr/sbin/apache2 -k start
10056               33                  /usr/sbin/apache2 -k start
```

```bash
/usr/bin/time docker stop test
```

```plaintext
test
real	0m 0.27s
user	0m 0.03s
sys	0m 0.03s
```

> **Megjegyzés**
>
> Az `ENTRYPOINT` beállítást felülírhatod a `--entrypoint`-tal, de ez csak a binárist tudja beállítani az exec-hez (nem használódik `sh -c`).

### Shell forma ENTRYPOINT példa

Megadhatsz egy egyszerű stringet az `ENTRYPOINT`-hoz, és az a `/bin/sh -c`-ben fog végrehajtódni. Ez a forma shell-feldolgozást használ a shell környezeti változók helyettesítéséhez, és figyelmen kívül hagyja a `CMD`-t vagy a `docker run` parancssori argumentumait. Annak biztosításához, hogy a `docker stop` helyesen szignálja bármely hosszan futó `ENTRYPOINT` futtatható fájlt, ne feledd `exec`-kel indítani:

```dockerfile
FROM ubuntu
ENTRYPOINT exec top -b
```

Amikor ezt az image-et futtatod, az egyetlen PID 1 folyamatot fogod látni:

```bash
docker run -it --rm --name test top
```

```plaintext
Mem: 1704520K used, 352148K free, 0K shrd, 0K buff, 140368121167873K cached
CPU:   5% usr   0% sys   0% nic  94% idle   0% io   0% irq   0% sirq
Load average: 0.08 0.03 0.05 2/98 6
  PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
    1     0 root     R     3164   0%   0% top -b
```

Amely tisztán kilép a `docker stop`-ra:

```bash
/usr/bin/time docker stop test
```

```plaintext
test
real	0m 0.20s
user	0m 0.02s
sys	0m 0.04s
```

Ha elfelejtesz `exec`-et hozzáadni az `ENTRYPOINT` elejéhez:

```dockerfile
FROM ubuntu
ENTRYPOINT top -b
CMD -- --ignored-param1
```

Akkor futtathatod (nevet adva neki a következő lépéshez):

```bash
docker run -it --name test top --ignored-param2
```

```plaintext
top - 13:58:24 up 17 min,  0 users,  load average: 0.00, 0.00, 0.00
Tasks:   2 total,   1 running,   1 sleeping,   0 stopped,   0 zombie
Cpu(s): 16.7 us, 33.3 sy,  0.0 ni, 50.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   1990.8 total,   1354.6 free,    231.4 used,    404.7 buff/cache
MiB Swap:   1024.0 total,   1024.0 free,      0.0 used.   1639.8 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
    1 root      20   0    2612    604    536 S   0.0   0.0   0:00.02 sh
    6 root      20   0    5956   3188   2768 R   0.0   0.2   0:00.00 top
```

A `top` kimenetéből láthatod, hogy a megadott `ENTRYPOINT` nem PID 1.

Ha ezután futtatod a `docker stop test`-et, a konténer nem fog tisztán kilépni – a stop parancs kénytelen lesz SIGKILL-t küldeni az időtúllépés után:

```bash
docker exec -it test ps waux
```

```plaintext
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.4  0.0   2612   604 pts/0    Ss+  13:58   0:00 /bin/sh -c top -b --ignored-param2
root         6  0.0  0.1   5956  3188 pts/0    S+   13:58   0:00 top -b
root         7  0.0  0.1   5884  2816 pts/1    Rs+  13:58   0:00 ps waux
```

```bash
/usr/bin/time docker stop test
```

```plaintext
test
real	0m 10.19s
user	0m 0.04s
sys	0m 0.03s
```

### A CMD és ENTRYPOINT kölcsönhatásának megértése

Mind a `CMD`, mind az `ENTRYPOINT` utasítások meghatározzák, milyen parancs hajtódik végre egy konténer futtatásakor. Néhány szabály írja le az együttműködésüket.

- A Dockerfile-nak legalább a `CMD` vagy `ENTRYPOINT` parancsok egyikét meg kell adnia.
- Az `ENTRYPOINT`-ot akkor kell meghatározni, amikor a konténert futtatható fájlként használod.
- A `CMD`-t az `ENTRYPOINT` parancs alapértelmezett argumentumainak meghatározására vagy egy ad-hoc parancs konténerben való végrehajtására kell használni.
- A `CMD` felülírásra kerül, amikor a konténert alternatív argumentumokkal futtatod.

Az alábbi táblázat megmutatja, milyen parancs hajtódik végre a különböző `ENTRYPOINT` / `CMD` kombinációkhoz:

| | No ENTRYPOINT | ENTRYPOINT exec_entry p1_entry | ENTRYPOINT ["exec_entry", "p1_entry"] |
| --- | --- | --- | --- |
| No CMD | error, not allowed | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry |
| CMD ["exec_cmd", "p1_cmd"] | exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry exec_cmd p1_cmd |
| CMD exec_cmd p1_cmd | /bin/sh -c exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry /bin/sh -c exec_cmd p1_cmd |

> **Megjegyzés**
>
> Ha a `CMD` az alapimage-ből van definiálva, az `ENTRYPOINT` beállítása visszaállítja a `CMD`-t üres értékre. Ebben a forgatókönyvben a `CMD`-t az aktuális image-ben kell definiálni, hogy értéke legyen.

## VOLUME

```dockerfile
VOLUME ["/data"]
```

A `VOLUME` utasítás létrehoz egy mount pontot a megadott névvel, és megjelöli azt, hogy külsőleg csatolt köteteket tartalmaz a natív hosztról vagy más konténerekről. Az érték lehet egy JSON tömb, `VOLUME ["/var/log/"]`, vagy egy egyszerű string több argumentummal, mint a `VOLUME /var/log` vagy `VOLUME /var/log /var/db`. További információért/példákért és a Docker kliensen keresztüli mountolási utasításokért lásd: Share Directories via Volumes dokumentáció.

A `docker run` parancs inicializálja az újonnan létrehozott kötetet bármely olyan adattal, amely az alapimage-ben a megadott helyen létezik. Például tekintsd a következő Dockerfile részletet:

```dockerfile
FROM ubuntu
RUN mkdir /myvol
RUN echo "hello world" > /myvol/greeting
VOLUME /myvol
```

Ez a Dockerfile egy olyan image-et eredményez, amely hatására a `docker run` létrehoz egy új mount pontot a `/myvol`-on, és átmásolja a `greeting` fájlt az újonnan létrehozott kötetbe.

### Megjegyzések a kötetek megadásához

Tartsd szem előtt a következőket a Dockerfile-ban lévő kötetekkel kapcsolatban.

- **Kötetek Windows-alapú konténereken:** Windows-alapú konténerek használatakor a kötet céljának a konténeren belül a következők egyikének kell lennie:
  - nem létező vagy üres könyvtár
  - a C:-től eltérő meghajtó
- **A kötet megváltoztatása a Dockerfile-on belül:** Ha bármely build-lépés megváltoztatja a kötetön belüli adatokat, miután deklarálta, ezek a változtatások elvetésre kerülnek a régi builder használatakor. A Buildkit használatakor a változtatások megőrzésre kerülnek.
- **JSON formázás:** A lista JSON tömbként van értelmezve. A szavakat dupla idézőjelekkel (`"`) kell körülvenni, nem szimpla idézőjelekkel (`'`).
- **A hosztkönyvtár a konténer futásidőben van deklarálva:** A hosztkönyvtár (a mountpont) természeténél fogva hosztfüggő. Ez az image hordozhatóságának megőrzését szolgálja, mivel egy adott hosztkönyvtár elérhetősége nem garantálható minden hoszton. Ezért nem csatolhatsz hosztkönyvtárat a Dockerfile-on belül. A `VOLUME` utasítás nem támogatja a host-dir paraméter megadását. A mountpontot a konténer létrehozásakor vagy futtatásakor kell megadnod.

## USER

```dockerfile
USER <user>[:<group>]
```

vagy

```dockerfile
USER UID[:<GID>]
```

A `USER` utasítás beállítja a felhasználónevet (vagy UID-t) és opcionálisan a felhasználói csoportot (vagy GID-t), amelyet alapértelmezett felhasználóként és csoportként kell használni az aktuális szakasz hátralévő részében. A megadott felhasználó a `RUN` utasításokhoz és futásidőben a releváns `ENTRYPOINT` és `CMD` parancsok futtatásához használatos.

> Ne feledd, hogy amikor csoportot adsz meg a felhasználóhoz, a felhasználónak csak a megadott csoporttagsága lesz. Bármely más konfigurált csoporttagság figyelmen kívül marad.

> ⚠️ **Figyelmeztetés**
>
> Amikor a felhasználónak nincs elsődleges csoportja, akkor az image (vagy a következő utasítások) a root csoporttal fognak futni.

Windows-on a felhasználót először létre kell hozni, ha nem beépített fiók. Ez a `net user` paranccsal tehető meg, amelyet a Dockerfile részeként hívnak meg.

```dockerfile
FROM microsoft/windowsservercore
# Create Windows user in the container
RUN net user /add patrick
# Set it for subsequent commands
USER patrick
```

## WORKDIR

```dockerfile
WORKDIR /path/to/workdir
```

A `WORKDIR` utasítás beállítja a munkakönyvtárat bármely `RUN`, `CMD`, `ENTRYPOINT`, `COPY` és `ADD` utasításhoz, amelyek követik azt a Dockerfile-ban. Ha a `WORKDIR` nem létezik, létrejön, még akkor is, ha nem használják semmilyen későbbi Dockerfile utasításban.

A `WORKDIR` utasítás többször is használható egy Dockerfile-ban. Ha relatív útvonalat adsz meg, az az előző `WORKDIR` utasítás útvonalához viszonyított lesz. Például:

```dockerfile
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

A végső `pwd` parancs kimenete ebben a Dockerfile-ban `/a/b/c` lenne.

A `WORKDIR` utasítás fel tudja oldani a korábban `ENV`-vel beállított környezeti változókat. Csak a Dockerfile-ban explicit módon beállított környezeti változókat használhatod. Például:

```dockerfile
ENV DIRPATH=/path
WORKDIR $DIRPATH/$DIRNAME
RUN pwd
```

A végső `pwd` parancs kimenete ebben a Dockerfile-ban `/path/$DIRNAME` lenne.

Ha nincs megadva, az alapértelmezett munkakönyvtár a `/`. A gyakorlatban, ha nem `scratch`-ből építesz Dockerfile-t (`FROM scratch`), a `WORKDIR` valószínűleg az általad használt alapimage által van beállítva.

Ezért, hogy elkerüld a nem szándékos műveleteket ismeretlen könyvtárakban, a legjobb gyakorlat a `WORKDIR` explicit beállítása.

## ARG

```dockerfile
ARG <name>[=<default value>] [<name>[=<default value>]...]
```

Az `ARG` utasítás definiál egy változót, amelyet a felhasználók build időben átadhatnak a buildernek a `docker build` paranccsal a `--build-arg <varname>=<value>` flag használatával. Ez a változó használható a későbbi utasításokban, mint a `FROM`, `ENV`, `WORKDIR` és mások, a `${VAR}` vagy `$VAR` sablon szintaxissal. Ez a változó az összes későbbi `RUN` utasításnak is átadódik build-idő környezeti változóként.

Az `ENV`-vel ellentétben az `ARG` változó nem ágyazódik be az image-be, és nem érhető el a végső konténerben.

> ⚠️ **Figyelmeztetés**
>
> Nem ajánlott build argumentumokat használni titkok, például felhasználói hitelesítő adatok, API tokenek stb. átadására. A build argumentumok láthatók a `docker history` parancsban és a max módú provenance attestationökben, amelyek alapértelmezésben az image-hez vannak csatolva, ha a Buildx GitHub Actions-t használod, és a GitHub repository-d nyilvános.
>
> A titkok biztonságos használatáról az image-ek építésekor a RUN --mount=type=secret szakaszban olvashatsz.

Egy Dockerfile egy vagy több `ARG` utasítást tartalmazhat. Például a következő érvényes Dockerfile:

```dockerfile
FROM busybox
ARG user1
ARG buildno
# ...
```

### Alapértelmezett értékek

Egy `ARG` utasítás opcionálisan tartalmazhat alapértelmezett értéket:

```dockerfile
FROM busybox
ARG user1=someuser
ARG buildno=1
# ...
```

Ha egy `ARG` utasításnak van alapértelmezett értéke, és nincs érték átadva build időben, a builder az alapértelmezést használja.

### Hatókör

Egy `ARG` változó attól a sortól lép életbe, amelyen deklarálják a Dockerfile-ban. Például tekintsd ezt a Dockerfile-t:

```dockerfile
FROM busybox
USER ${username:-some_user}
ARG username
USER $username
# ...
```

Egy felhasználó ezt a fájlt a következő hívással építi:

```bash
docker build --build-arg username=what_user .
```

- A `USER` utasítás a 2. sorban a `some_user` fallback-re értékelődik, mert a `username` változó még nincs deklarálva.
- A `username` változó a 3. sorban van deklarálva, és attól a ponttól kezdve elérhető a Dockerfile utasításban való hivatkozáshoz.
- A `USER` utasítás a 4. sorban `what_user`-re értékelődik, mivel ezen a ponton a `username` argumentumnak `what_user` értéke van, amelyet a parancssorban adtak át. Egy `ARG` utasítás általi definíciója előtt bármely változó használata üres stringet eredményez.

Egy build-szakaszon belül deklarált `ARG` változót automatikusan örökölnek az azon a szakaszon alapuló más szakaszok. A nem kapcsolódó build-szakaszok nem férnek hozzá a változóhoz. Egy argumentum több különálló szakaszban való használatához minden szakasznak tartalmaznia kell az `ARG` utasítást, vagy mindkettőnek egy közös alapszakaszon kell alapulnia ugyanabban a Dockerfile-ban, ahol a változó deklarálva van.

További információért lásd: variable scoping.

### ARG változók használata

Használhatsz `ARG` vagy `ENV` utasítást a `RUN` utasítás számára elérhető változók megadásához. Az `ENV` utasítással definiált környezeti változók mindig felülírják az azonos nevű `ARG` utasítást. Tekintsd ezt a Dockerfile-t egy `ENV` és `ARG` utasítással.

```dockerfile
FROM ubuntu
ARG CONT_IMG_VER
ENV CONT_IMG_VER=v1.0.0
RUN echo $CONT_IMG_VER
```

Ezután tegyük fel, hogy ez az image ezzel a paranccsal épül:

```bash
docker build --build-arg CONT_IMG_VER=v2.0.1 .
```

Ebben az esetben a `RUN` utasítás a `v1.0.0`-t használja a felhasználó által átadott `v2.0.1` `ARG` beállítás helyett. Ez a viselkedés hasonló egy shell szkripthez, ahol egy lokálisan hatókörű változó felülírja az argumentumként átadott vagy a környezetből örökölt változókat a definíciójának pontjától.

A fenti példát használva, de más `ENV` specifikációval hasznosabb interakciókat hozhatsz létre az `ARG` és `ENV` utasítások között:

```dockerfile
FROM ubuntu
ARG CONT_IMG_VER
ENV CONT_IMG_VER=${CONT_IMG_VER:-v1.0.0}
RUN echo $CONT_IMG_VER
```

Az `ARG` utasítással ellentétben az `ENV` értékek mindig megmaradnak az épített image-ben. Tekintsd a `--build-arg` flag nélküli `docker build`-et:

```bash
docker build .
```

Ezt a Dockerfile példát használva a `CONT_IMG_VER` még mindig megmarad az image-ben, de az értéke `v1.0.0` lenne, mivel ez az alapértelmezés a 3. sorban az `ENV` utasítás által.

A változó-kiterjesztési technika ebben a példában lehetővé teszi, hogy argumentumokat adj át a parancssorból, és azokat az `ENV` utasítás kihasználásával megőrizd a végső image-ben. A változó-kiterjesztés csak a Dockerfile utasítások korlátozott halmazához támogatott.

### Előre definiált ARG-ok

A Dockernek van egy sor előre definiált `ARG` változója, amelyeket a Dockerfile-ban megfelelő `ARG` utasítás nélkül is használhatsz.

- `HTTP_PROXY`
- `http_proxy`
- `HTTPS_PROXY`
- `https_proxy`
- `FTP_PROXY`
- `ftp_proxy`
- `NO_PROXY`
- `no_proxy`
- `ALL_PROXY`
- `all_proxy`

Ezek használatához add át őket a parancssorban a `--build-arg` flaggel, például:

```bash
docker build --build-arg HTTPS_PROXY=https://my-proxy.example.com .
```

Alapértelmezésben ezek az előre definiált változók ki vannak zárva a `docker history` kimenetéből. Kizárásuk csökkenti annak kockázatát, hogy véletlenül érzékeny hitelesítési információkat szivárogtassanak ki egy `HTTP_PROXY` változóban.

Például tekintsd a következő Dockerfile építését `--build-arg HTTP_PROXY=http://user:pass@proxy.lon.example.com` használatával

```dockerfile
FROM ubuntu
RUN echo "Hello World"
```

Ebben az esetben a `HTTP_PROXY` változó értéke nem érhető el a `docker history`-ban, és nincs gyorsítótárazva. Ha a helyet megváltoztatnád, és a proxy szervered `http://user:pass@proxy.sfo.example.com`-ra változna, egy későbbi build nem eredményez cache miss-t.

Ha felül kell írnod ezt a viselkedést, akkor megteheted egy `ARG` utasítás hozzáadásával a Dockerfile-hoz a következőképpen:

```dockerfile
FROM ubuntu
ARG HTTP_PROXY
RUN echo "Hello World"
```

Ennek a Dockerfile-nak az építésekor a `HTTP_PROXY` megmarad a `docker history`-ban, és az értékének megváltoztatása érvényteleníti a build gyorsítótárat.

### Automatikus platform ARG-ok a globális hatókörben

Ez a funkció csak a BuildKit backend használatakor érhető el.

A BuildKit támogat egy előre definiált `ARG` változókat tartalmazó készletet, amelyek információt tartalmaznak a buildet végző node platformjáról (build platform) és az eredményül kapott image platformjáról (target platform). A célplatform a `--platform` flaggel adható meg a `docker build`-en.

A következő `ARG` változók automatikusan beállításra kerülnek:

- `TARGETPLATFORM` – a build eredményének platformja. Pl. `linux/amd64`, `linux/arm/v7`, `windows/amd64`.
- `TARGETOS` – a `TARGETPLATFORM` OS komponense
- `TARGETARCH` – a `TARGETPLATFORM` architektúra komponense
- `TARGETVARIANT` – a `TARGETPLATFORM` változat komponense
- `BUILDPLATFORM` – a buildet végző node platformja.
- `BUILDOS` – a `BUILDPLATFORM` OS komponense
- `BUILDARCH` – a `BUILDPLATFORM` architektúra komponense
- `BUILDVARIANT` – a `BUILDPLATFORM` változat komponense

Ezek az argumentumok a globális hatókörben vannak definiálva, így nem automatikusan elérhetők a build-szakaszokon belül vagy a `RUN` parancsaidhoz. Ezen argumentumok egyikének a build-szakaszon belüli eléréséhez definiáld újra érték nélkül.

Például:

```dockerfile
FROM alpine
ARG TARGETPLATFORM
RUN echo "I'm building for $TARGETPLATFORM"
```

### BuildKit beépített build arg-ok

| Arg | Típus | Leírás |
| --- | --- | --- |
| `BUILDKIT_BUILD_NAME` | String | A build nevének felülírása a `buildx history` parancsban és a Docker Desktop Builds nézetben. |
| `BUILDKIT_CACHE_MOUNT_NS` | String | Opcionális cache ID névtér beállítása. |
| `BUILDKIT_CONTEXT_KEEP_GIT_DIR` | Bool | A Git kontextus .git könyvtárának megőrzésének kiváltása. |
| `BUILDKIT_GIT_ADVICE` | Bool | Git tanácsüzenetek megjelenítése a BuildKit által kezelt Git műveletekből. Alapértelmezésben `false`. |
| `BUILDKIT_INLINE_CACHE2` | Bool | Inline cache metaadatok az image konfigba vagy sem. |
| `BUILDKIT_MULTI_PLATFORM` | Bool | Determinisztikus kimenet választása a többplatformos kimenettől függetlenül vagy sem. |
| `BUILDKIT_SANDBOX_HOSTNAME` | String | A hosztnév beállítása (alapértelmezés `buildkitsandbox`) |
| `BUILDKIT_SYNTAX` | String | Frontend image beállítása. Állítsd `dockerfile.v0`-ra a Dockerfile `# syntax=` direktíva figyelmen kívül hagyásához és a beépített frontend használatához helyette. |
| `SOURCE_DATE_EPOCH` | Int | A létrehozott image és rétegek Unix időbélyegének beállítása. További információ a reproducible builds-től. A Dockerfile 1.5, BuildKit 0.11 óta támogatott |

**Példa: .git könyvtár megőrzése**

Amikor Git kontextust használsz, a `.git` könyvtár nem marad meg a checkoutokon. Hasznos lehet megtartani, ha git információkat szeretnél lekérni a build során:

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
WORKDIR /src
RUN --mount=target=. \
  make REVISION=$(git rev-parse HEAD) build
```

```bash
docker build --build-arg BUILDKIT_CONTEXT_KEEP_GIT_DIR=1 https://github.com/user/repo.git#main
```

### A build gyorsítótárazásra gyakorolt hatás

Az `ARG` változók nem maradnak meg az épített image-ben, ahogy az `ENV` változók. Az `ARG` változók azonban hasonló módon befolyásolják a build gyorsítótárat. Ha egy Dockerfile definiál egy `ARG` változót, amelynek értéke eltér egy korábbi buildtől, akkor egy „cache miss” történik az első használatakor, nem a definíciójakor. Különösen az `ARG` utasítást követő összes `RUN` utasítás implicit módon használja az `ARG` változót (környezeti változóként), így cache miss-t okozhat. Az összes előre definiált `ARG` változó mentes a gyorsítótárazás alól, kivéve, ha van megfelelő `ARG` utasítás a Dockerfile-ban.

Például tekintsd ezt a két Dockerfile-t:

```dockerfile
FROM ubuntu
ARG CONT_IMG_VER
RUN echo $CONT_IMG_VER
```

```dockerfile
FROM ubuntu
ARG CONT_IMG_VER
RUN echo hello
```

Ha a parancssorban megadod a `--build-arg CONT_IMG_VER=<value>`-t, mindkét esetben a 2. sor specifikációja nem okoz cache miss-t; a 3. sor okoz cache miss-t. Az `ARG CONT_IMG_VER` hatására a `RUN` sor úgy azonosításra kerül, mint a `CONT_IMG_VER=<value> echo hello` futtatása, így ha a `<value>` megváltozik, cache miss-t kapsz.

Tekintsd a másik példát ugyanazon parancssor alatt:

```dockerfile
FROM ubuntu
ARG CONT_IMG_VER
ENV CONT_IMG_VER=$CONT_IMG_VER
RUN echo $CONT_IMG_VER
```

Ebben a példában a cache miss a 3. sorban történik. A miss azért történik, mert a változó értéke az `ENV`-ben hivatkozik az `ARG` változóra, és az a változó a parancssoron keresztül megváltozik. Ebben a példában az `ENV` parancs hatására az image tartalmazza az értéket.

Ha egy `ENV` utasítás felülír egy azonos nevű `ARG` utasítást, mint ez a Dockerfile:

```dockerfile
FROM ubuntu
ARG CONT_IMG_VER
ENV CONT_IMG_VER=hello
RUN echo $CONT_IMG_VER
```

A 3. sor nem okoz cache miss-t, mert a `CONT_IMG_VER` értéke egy konstans (`hello`). Ennek eredményeként a `RUN`-on (4. sor) használt környezeti változók és értékek nem változnak a buildek között.

## ONBUILD

```dockerfile
ONBUILD INSTRUCTION
```

Az `ONBUILD` utasítás hozzáad az image-hez egy trigger utasítást, amelyet később, amikor az image-et egy másik build alapjaként használják, végre kell hajtani. A trigger a downstream build kontextusában hajtódik végre, mintha közvetlenül a downstream Dockerfile `FROM` utasítása után illesztették volna be.

Ez hasznos, ha olyan image-et építesz, amelyet más image-ek építésének alapjaként fognak használni, például egy alkalmazás-építési környezetet vagy egy démont, amely felhasználóspecifikus konfigurációval testreszabható.

Például, ha az image-ed egy újrafelhasználható Python alkalmazás-építő, akkor az alkalmazás forráskódját egy adott könyvtárba kell hozzáadni, és szükség lehet egy build szkript meghívására utána. Nem hívhatod meg egyszerűen az `ADD`-ot és `RUN`-t most, mert még nem férsz hozzá az alkalmazás forráskódjához, és az minden alkalmazás buildnél más lesz. Egyszerűen adhatnál az alkalmazásfejlesztőknek egy sablon Dockerfile-t, hogy másolják be az alkalmazásukba, de ez nem hatékony, hibára hajlamos és nehezen frissíthető, mert összekeveredik az alkalmazásspecifikus kóddal.

A megoldás az `ONBUILD` használata előzetes utasítások regisztrálására, amelyeket később, a következő build-szakaszban kell futtatni.

Íme, hogyan működik:

- Amikor egy `ONBUILD` utasítással találkozik, a builder hozzáad egy triggert az épített image metaadataihoz. Az utasítás egyébként nem befolyásolja az aktuális buildet.
- A build végén az összes trigger listája az image manifestben tárolódik, az `OnBuild` kulcs alatt. Ezek a `docker inspect` paranccsal vizsgálhatók meg.
- Később az image-et egy új build alapjaként használhatják, a `FROM` utasítással. A `FROM` utasítás feldolgozásának részeként a downstream builder keresi az `ONBUILD` triggereket, és azokat a regisztrálásuk sorrendjében hajtja végre. Ha bármelyik trigger meghiúsul, a `FROM` utasítás megszakad, ami viszont a build meghiúsulását okozza. Ha az összes trigger sikeres, a `FROM` utasítás befejeződik, és a build a szokásos módon folytatódik.
- A triggerek a végrehajtás után törlődnek a végső image-ből. Más szóval nem öröklődnek az „unoka” buildekbe.

Például hozzáadhatsz valami ilyesmit:

```dockerfile
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
```

### Másolás vagy mountolás szakaszból, image-ből vagy kontextusból

A Dockerfile szintaxis 1.11 verziója óta használhatod az `ONBUILD`-ot olyan utasításokkal, amelyek fájlokat másolnak vagy mountolnak más szakaszokból, image-ekből vagy build kontextusokból. Például:

```dockerfile
# syntax=docker/dockerfile:1.11
FROM alpine AS baseimage
ONBUILD COPY --from=build /usr/bin/app /app
ONBUILD RUN --mount=from=config,target=/opt/appconfig ...
```

Ha a `from` forrása egy build-szakasz, a szakaszt abban a Dockerfile-ban kell definiálni, ahol az `ONBUILD` kiváltódik. Ha egy nevesített kontextus, azt a kontextust át kell adni a downstream buildnek.

### ONBUILD korlátozások

- Az `ONBUILD` utasítások láncolása `ONBUILD ONBUILD` használatával nem megengedett.
- Az `ONBUILD` utasítás nem válthat ki `FROM` vagy `MAINTAINER` utasításokat.

## STOPSIGNAL

```dockerfile
STOPSIGNAL signal
```

A `STOPSIGNAL` utasítás beállítja azt a rendszerhívás-szignált, amelyet a konténernek a kilépéshez küldenek. Ez a szignál lehet egy szignálnév `SIG<NAME>` formátumban, például `SIGKILL`, vagy egy előjel nélküli szám, amely megfelel a kernel syscall táblájában lévő pozíciónak, például `9`. Az alapértelmezés a `SIGTERM`, ha nincs definiálva.

A `STOPSIGNAL` a `docker stop` által küldött szignálra vonatkozik (és a Docker démonra, amikor egy konténert leállít). Nem befolyásolja a billentyűparancsokkal, például a `Ctrl+C`-vel küldött szignálokat, amelyek közvetlenül a folyamatnak küldenek `SIGINT`-et, függetlenül a `STOPSIGNAL` beállítástól.

Az image alapértelmezett stopsignalja konténerenként felülírható a `--stop-signal` flaggel a `docker run` és `docker create` parancsokon.

## HEALTHCHECK

A `HEALTHCHECK` utasításnak két formája van:

- `HEALTHCHECK [OPTIONS] CMD command` (a konténer állapotának ellenőrzése egy parancs futtatásával a konténeren belül)
- `HEALTHCHECK NONE` (az alapimage-ből örökölt bármely healthcheck letiltása)

A `HEALTHCHECK` utasítás megmondja a Docker-nek, hogyan teszteljen egy konténert annak ellenőrzésére, hogy még mindig működik-e. Ez felismerhet olyan eseteket, mint egy végtelen ciklusba ragadt webszerver, amely képtelen új kapcsolatokat kezelni, annak ellenére, hogy a szerverfolyamat még mindig fut.

Amikor egy konténernek van megadott healthcheckje, annak van egy egészségi állapota a normál állapotán kívül. Ez az állapot kezdetben `starting`. Valahányszor egy health check sikeres, `healthy` lesz (bármilyen állapotban is volt korábban). Egy bizonyos számú egymást követő hiba után `unhealthy` lesz.

A `CMD` előtt megjelenhető opciók:

- `--interval=DURATION` (alapértelmezés: 30s)
- `--timeout=DURATION` (alapértelmezés: 30s)
- `--start-period=DURATION` (alapértelmezés: 0s)
- `--start-interval=DURATION` (alapértelmezés: 5s)
- `--retries=N` (alapértelmezés: 3)

A health check először `interval` másodperccel a konténer indítása után fut, majd ismét `interval` másodperccel minden előző ellenőrzés befejezése után. A kezdő időszakban a health checkek `start interval` gyakorisággal futnak helyette.

Ha az ellenőrzés egyetlen futása hosszabb, mint `timeout` másodperc, akkor az ellenőrzés meghiúsultnak tekintendő. Az ellenőrzést végző folyamat hirtelen leállításra kerül egy `SIGKILL`-lel.

`retries` egymást követő hiba szükséges a health checkhez, hogy a konténer `unhealthy`-nek tekinthető legyen.

A `start period` inicializálási időt biztosít azoknak a konténereknek, amelyeknek időre van szükségük a bootstrapeléshez. A próba meghiúsulása ebben az időszakban nem számít bele a maximális újrapróbálkozások számába. Azonban, ha egy health check sikeres a kezdő időszakban, a konténer `started`-nek tekintendő, és minden egymást követő hiba beleszámít a maximális újrapróbálkozások számába.

A `start interval` a health checkek közötti idő a kezdő időszakban. Ez az opció Docker Engine 25.0 vagy újabb verziót igényel.

Egy Dockerfile-ban csak egy `HEALTHCHECK` utasítás lehet. Ha többet sorolsz fel, akkor csak az utolsó `HEALTHCHECK` lép életbe.

A `CMD` kulcsszó utáni parancs lehet egy shell parancs (pl. `HEALTHCHECK CMD /bin/check-running`) vagy egy exec tömb (mint más Dockerfile parancsoknál; lásd pl. `ENTRYPOINT` a részletekért).

A parancs kilépési állapota jelzi a konténer egészségi állapotát. A lehetséges értékek:

- `0`: siker – a konténer egészséges és használatra kész
- `1`: egészségtelen – a konténer nem működik megfelelően
- `2`: fenntartott – ne használd ezt a kilépési kódot

Például annak ellenőrzéséhez, hogy egy webszerver körülbelül öt percenként képes-e kiszolgálni az oldal főoldalát három másodpercen belül:

```dockerfile
HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

A sikertelen próbák hibakeresésének segítéséhez a parancs által az stdoutra vagy stderrre írt bármely kimeneti szöveg (UTF-8 kódolású) az egészségi állapotban tárolódik, és a `docker inspect`-tal lekérdezhető. Az ilyen kimenetet röviden kell tartani (jelenleg csak az első 4096 bájt kerül tárolásra).

Amikor egy konténer egészségi állapota megváltozik, egy `health_status` esemény generálódik az új állapottal.

## SHELL

```dockerfile
SHELL ["executable", "parameters"]
```

A `SHELL` utasítás lehetővé teszi a parancsok shell formájához használt alapértelmezett shell felülírását. Az alapértelmezett shell Linuxon `["/bin/sh", "-c"]`, Windows-on `["cmd", "/S", "/C"]`. A `SHELL` utasítást JSON formában kell írni egy Dockerfile-ban.

A `SHELL` utasítás különösen hasznos Windows-on, ahol két gyakran használt és meglehetősen különböző natív shell van: a `cmd` és a `powershell`, valamint alternatív shellek, beleértve az `sh`-t is.

A `SHELL` utasítás többször is megjelenhet. Minden `SHELL` utasítás felülírja az összes előző `SHELL` utasítást, és befolyásolja az összes későbbi utasítást. Például:

```dockerfile
FROM microsoft/windowsservercore

# Executed as cmd /S /C echo default
RUN echo default

# Executed as cmd /S /C powershell -command Write-Host default
RUN powershell -command Write-Host default

# Executed as powershell -command Write-Host hello
SHELL ["powershell", "-command"]
RUN Write-Host hello

# Executed as cmd /S /C echo hello
SHELL ["cmd", "/S", "/C"]
RUN echo hello
```

A következő utasításokat befolyásolhatja a `SHELL` utasítás, amikor a shell formájukat használják egy Dockerfile-ban: `RUN`, `CMD` és `ENTRYPOINT`.

A következő példa egy gyakori minta Windows-on, amely egyszerűsíthető a `SHELL` utasítás használatával:

```dockerfile
RUN powershell -command Execute-MyCmdlet -param1 "c:\foo.txt"
```

A builder által meghívott parancs a következő lesz:

```dockerfile
cmd /S /C powershell -command Execute-MyCmdlet -param1 "c:\foo.txt"
```

Ez két okból nem hatékony. Először is, egy szükségtelen `cmd.exe` parancsprocesszor (azaz shell) kerül meghívásra. Másodszor, minden `RUN` utasítás a shell formában egy extra `powershell -command` előtagot igényel a parancs előtt.

Ennek hatékonyabbá tételéhez két mechanizmus egyike alkalmazható. Az egyik a `RUN` parancs JSON formájának használata, például:

```dockerfile
RUN ["powershell", "-command", "Execute-MyCmdlet", "-param1 \"c:\\foo.txt\""]
```

Bár a JSON forma egyértelmű és nem használja a szükségtelen `cmd.exe`-t, több bőbeszédűséget igényel dupla idézőjelekkel és escape-eléssel. Az alternatív mechanizmus a `SHELL` utasítás és a shell forma használata, ami természetesebb szintaxist eredményez a Windows felhasználók számára, különösen az escape parser direktívával kombinálva:

```dockerfile
# escape=`

FROM microsoft/nanoserver
SHELL ["powershell","-command"]
RUN New-Item -ItemType Directory C:\Example
ADD Execute-MyCmdlet.ps1 c:\example\
RUN c:\example\Execute-MyCmdlet -sample 'hello world'
```

Eredmény:

```powershell
PS E:\myproject> docker build -t shell .

Sending build context to Docker daemon 4.096 kB
Step 1/5 : FROM microsoft/nanoserver
 ---> 22738ff49c6d
Step 2/5 : SHELL powershell -command
 ---> Running in 6fcdb6855ae2
 ---> 6331462d4300
Removing intermediate container 6fcdb6855ae2
Step 3/5 : RUN New-Item -ItemType Directory C:\Example
 ---> Running in d0eef8386e97


    Directory: C:\


Mode         LastWriteTime              Length Name
----         -------------              ------ ----
d-----       10/28/2016  11:26 AM              Example


 ---> 3f2fbf1395d9
Removing intermediate container d0eef8386e97
Step 4/5 : ADD Execute-MyCmdlet.ps1 c:\example\
 ---> a955b2621c31
Removing intermediate container b825593d39fc
Step 5/5 : RUN c:\example\Execute-MyCmdlet 'hello world'
 ---> Running in be6d8e63fe75
hello world
 ---> 8e559e9bf424
Removing intermediate container be6d8e63fe75
Successfully built 8e559e9bf424
PS E:\myproject>
```

A `SHELL` utasítás arra is használható, hogy módosítsa a shell működésének módját. Például a `SHELL cmd /S /C /V:ON|OFF` használatával Windows-on a késleltetett környezeti változó-kiterjesztés szemantikája módosítható.

A `SHELL` utasítás Linuxon is használható, ha alternatív shellre van szükség, például `zsh`, `csh`, `tcsh` és mások.

## Here-Documents

A here-documents lehetővé teszik a későbbi Dockerfile-sorok átirányítását a `RUN` vagy `COPY` parancsok bemenetére. Ha egy ilyen parancs here-documentet tartalmaz, a Dockerfile a következő sorokat addig tekinti ugyanazon parancs részének, amíg egy csak a here-doc határolót tartalmazó sort nem talál.

### Példa: Többsoros szkript futtatása

```dockerfile
# syntax=docker/dockerfile:1
FROM debian
RUN <<EOT bash
  set -ex
  apt-get update
  apt-get install -y vim
EOT
```

Ha a parancs csak egy here-documentet tartalmaz, annak tartalma az alapértelmezett shell-lel kerül kiértékelésre.

```dockerfile
# syntax=docker/dockerfile:1
FROM debian
RUN <<EOT
  mkdir -p foo/bar
EOT
```

Alternatívaként shebang fejléc használható egy értelmező meghatározásához.

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.6
RUN <<EOT
#!/usr/bin/env python
print("hello world")
EOT
```

Bonyolultabb példák több here-documentet is használhatnak.

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
RUN <<FILE1 cat > file1 && <<FILE2 cat > file2
I am
first
FILE1
I am
second
FILE2
```

### Példa: Inline fájlok létrehozása

A `COPY` utasításoknál a forrás paramétert helyettesítheted egy here-doc indikátorral, hogy a here-document tartalmát közvetlenül egy fájlba írd. A következő példa egy `greeting.txt` fájlt hoz létre `hello world` tartalommal egy `COPY` utasítással.

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
COPY <<EOF greeting.txt
hello world
EOF
```

A szokásos here-doc változó-kiterjesztési és tab-eltávolítási szabályok érvényesek. A következő példa egy kis Dockerfile-t mutat, amely egy `hello.sh` szkriptfájlt hoz létre egy `COPY` utasítással és egy here-documenttel.

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
ARG FOO=bar
COPY <<-EOT /script.sh
  echo "hello ${FOO}"
EOT
ENTRYPOINT ash /script.sh
```

Ebben az esetben a `script` fájl „hello bar”-t ír ki, mert a változó a `COPY` utasítás végrehajtásakor kerül kiterjesztésre.

```bash
docker build -t heredoc .
docker run heredoc
hello bar
```

Ha ehelyett a here-document `EOT` szó bármely részét idézőjelbe tennéd, a változó nem kerülne kiterjesztésre build időben.

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
ARG FOO=bar
COPY <<-"EOT" /script.sh
  echo "hello ${FOO}"
EOT
ENTRYPOINT ash /script.sh
```

Ne feledd, hogy az `ARG FOO=bar` itt felesleges, és eltávolítható. A változó futásidőben kerül értelmezésre, amikor a szkriptet meghívják:

```bash
docker build -t heredoc .
docker run -e FOO=world heredoc
hello world
```
