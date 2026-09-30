# Docker Multi-stage Builds (Többlépcsős Építés)

A multi-stage build a Docker egyik leghatékonyabb funkciója a képfájlok (image-ek) méretének drasztikus csökkentésére és a Dockerfile olvashatóságának javítására. Bevezetése előtt a fejlesztőknek gyakran két külön Dockerfile-t kellett írniuk (egy ún. "Builder pattern" nevű trükköt használva), hogy elválasszák a fejlesztői/fordítói környezetet a végső, éles (production) környezettől.

## 1. Mi az a Multi-stage Build és hogyan működik?

A többlépcsős építés lényege, hogy egyetlen Dockerfile-on belül több `FROM` utasítást használhatsz.

- Minden egyes `FROM` utasítás egy új alapot (base image) határoz meg, és egy új szakaszt (stage) indít az építési folyamatban.
- **A legfontosabb képesség:** a `COPY --from=...` utasítással kiválaszthatod, hogy miket (milyen elkészült fájlokat, binárisokat) másolsz át egy korábbi szakaszból az újba.
- Minden, amit nem másolsz át (pl. a forráskód, a fordítóprogramok, a köztes fájlok és csomagkezelők), automatikusan hátramarad és nem kerül be a végső Docker image-be.

## 2. Példa: A klasszikus Go/Node.js probléma megoldása

Nézzünk egy tipikus fordítós nyelvet (pl. Golang). A fordításhoz szükségünk van a teljes Go környezetre (ami több száz MB). De a futtatáshoz már csak a lefordított bináris fájl kell (ami csak pár MB).

```dockerfile
# -----------------------------------------
# 1. SZAKASZ: Építés (Builder Stage)
# -----------------------------------------
# Elnevezzük ezt a szakaszt "builder"-nek az AS kulcsszóval
FROM golang:1.20-alpine AS builder

WORKDIR /app
# Bemásoljuk a forráskódot
COPY . .
# Lefordítjuk az alkalmazást egy "myapp" nevű binárissá
RUN go build -o myapp main.go

# -----------------------------------------
# 2. SZAKASZ: Éles környezet (Production Stage)
# -----------------------------------------
# Egy teljesen új, tiszta és apró alap-image-et indítunk
FROM alpine:latest

WORKDIR /root/
# A VARÁZSLAT: Csak a kész binárist másoljuk át a "builder" szakaszból!
COPY --from=builder /app/myapp .

CMD ["./myapp"]
```

**Eredmény:** A végső Docker image alapja az apró `alpine:latest` lesz, kiegészítve az egyetlen lefordított binárissal. A nehéz `golang` image és a forráskód a kukában landol, az image mérete gigabájtok helyett megabájtokra csökken!

## 3. Szakaszok Elnevezése (AS kulcsszó)

Alapértelmezésben a szakaszoknak nincs nevük, a Docker sorszámozza őket 0-tól kezdve. Hivatkozhatnál rájuk sorszámmal is: `COPY --from=0 /app/myapp .`

Azonban a dokumentáció szigorúan ajánlja az `AS <név>` használatát a `FROM` utasításnál (pl. `FROM ubuntu AS builder`), majd a név szerinti hivatkozást (`COPY --from=builder`), mert ez olvashatóvá és a jövőbeli módosításokkal szemben ellenállóvá teszi a fájlt.

## 4. Haladó Funkciók és Trükkök

### A) Építés megállítása egy adott szakasznál (--target)

Amikor elindítod a `docker build` parancsot, az alapértelmezésben a Dockerfile legvégéig fut. Azonban a `--target` kapcsolóval megmondhatod a Dockernek, hogy álljon meg egy adott szakasznál.

**Miért jó ez?** Mert egyetlen Dockerfile-ba beteheted a tesztelő, a fejlesztői (debug) és a produkciós környezetet is!

```bash
# Csak a builder szakaszt futtatja (pl. fejlesztéshez, teszteléshez)
docker build --target builder -t myapp:dev .
```

### B) Másolás külső Image-ből (Belső szakasz nélkül)

A `--from` paraméter nemcsak a Dockerfile-ban definiált szakaszok nevét fogadja el, hanem bármilyen létező Docker image nevét is! Ezzel közvetlenül áthúzhatsz eszközöket külső image-ekből anélkül, hogy telepítened kéne őket.

**Példa:** Másoljuk ki az Nginx alapértelmezett konfigurációját a hivatalos `nginx` image-ből:

```dockerfile
COPY --from=nginx:latest /etc/nginx/nginx.conf /my-custom-nginx.conf
```

### C) Egy korábbi szakasz használata új bázisként

Használhatsz egy szakaszt közös alapként (base), amiből utána több különböző végcél (pl. dev és prod) is elágazik. Ez csökkenti a kódismétlést a Dockerfile-ban.

```dockerfile
# KÖZÖS ALAP
FROM alpine:latest AS base
RUN apk add --no-cache curl vim

# FEJLESZTŐI ÁG (Örökli a base-t)
FROM base AS development
RUN apk add --no-cache gcc make
CMD ["make", "run"]

# PRODUKCIÓS ÁG (Szintén a base-ből indul, a gcc/make nem lesz benne)
FROM base AS production
COPY ./myapp /app/
CMD ["/app/myapp"]
```

### D) Scratch használata (A legkisebb létező image)

Sokszor a végső szakasznál a `FROM scratch` utasítást használják (főleg C, C++, Rust vagy Go nyelveknél). A `scratch` egy speciális, teljesen üres image a Dockerben. Nincsenek benne Linux könyvtárak, nincs shell, egyszerűen semmi. Ha a programod statikusan van fordítva (minden függőségét tartalmazza a bináris), a `FROM scratch` használatával az image-ed pontosan bájt-ra akkora lesz, mint maga a futtatható fájlod. Nincs ennél biztonságosabb és kisebb konténer.

# 5. A BuildKit Párhuzamosítása (A Láthatatlan Gyorsítás)

A modern Docker (a BuildKit motor) nem egyszerűen fentről lefelé olvassa a szekvenciális szakaszokat, hanem egy függőségi gráfot (dependency graph) épít fel belőlük.

- **Felesleges szakaszok kihagyása:** Ha a `--target` kapcsolóval célzol meg egy szakaszt, a Docker okos: teljesen figyelmen kívül hagyja (ki sem értékeli) azokat a korábbi vagy későbbi szakaszokat, amelyekre a célodnak nincs szüksége (nincs belőlük `COPY --from=...`).
- **Párhuzamos építés (Concurrency):** Ha a Dockerfile-odban több olyan szakasz van, amik nem függenek egymástól (pl. az egyik a frontendet építi Node.js-sel, a másik a backendet Golanggel), a BuildKit ezeket egyszerre, párhuzamosan fogja felépíteni, drasztikusan csökkentve a teljes build időt. A végén pedig egy harmadik szakaszban összefoghatod a két külön szál eredményét.

# 6. A "Közös Csapda": Az ARG használata a FROM előtt

A multi-stage buildeknél a leggyakoribb hiba a globális változók (ARG) hibás használata. A dokumentáció külön felhívja erre a figyelmet.

Ha egy `ARG` változót a legelső `FROM` utasítás előtt definiálsz, azt az összes `FROM` sor használhatja (például a verziószámok dinamikus megadására). **DE!** Ez az `ARG` a szakaszokon (a `FROM` alatti részeken) belül már nem lesz látható, kivéve, ha újra deklarálod!

**Hibás megoldás:**

```dockerfile
ARG VERSION=latest
FROM alpine:$VERSION
# ITT HIBA LESZ: a $VERSION változó itt bent már üres!
RUN echo "A verzió: $VERSION"
```

**Helyes (Dokumentáció szerinti) megoldás:**

```dockerfile
# 1. Globális deklaráció (A FROM-ok használhatják)
ARG VERSION=latest

FROM alpine:$VERSION
# 2. Újbóli deklaráció érték nélkül! Így már a RUN is látja.
ARG VERSION
RUN echo "A verzió: $VERSION"
```

# 7. Külső Aliasként történő paraméterezés (Dinamikus FROM)

Multi-stage buildeknél előfordulhat, hogy a base image-et külső paraméterként (Build argumentumként) szeretnéd megadni, így ugyanaz a Dockerfile tud építeni pl. Debian és Alpine alapokra is.

```dockerfile
# Alapértelmezésben a "golang:alpine" lesz a "build_base", de ez buildeléskor felülírható
ARG BUILD_IMAGE=golang:alpine
FROM ${BUILD_IMAGE} AS build_base

WORKDIR /app
COPY . .
RUN go build -o myapp main.go

# A második szakasznál is használhatsz dinamikus alapot
ARG RUN_IMAGE=alpine:latest
FROM ${RUN_IMAGE}
COPY --from=build_base /app/myapp /myapp
```

Építéskor így tudod felülírni a fenti szakaszokat:

```bash
docker build --build-arg BUILD_IMAGE=golang:1.20-buster --build-arg RUN_IMAGE=debian:buster-slim .
```
