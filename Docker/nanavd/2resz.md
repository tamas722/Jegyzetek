# 1. Linux Kernel alapok a motorháztető alatt (Amit minden DevOps interjún megkérdeznek)

A kezdő szinten elegendő annyit tudni, hogy „a konténer a host kernelét használja”. DevOps mérnökként viszont pontosan értened kell azt a három Linux kernel funkciót, amelyekből a konténer valójában felépül (hiszen a Linux kernelben nincs olyan objektum, hogy „konténer” – csak izolált és korlátozott folyamatok léteznek).

## A) Linux Namespaces (Izoláció – „Mit láthat a folyamat?”)

A Namespaces (névterek) felelnek azért, hogy a konténerben futó folyamat úgy érezze, egy saját, önálló operációs rendszerben fut. Amikor a Docker elindít egy konténert, a kernel az alábbi névtereket hozza létre számára:

| Namespace | Kernel Flag | Mit izolál a konténer számára? |
|---|---|---|
| **PID** | `CLONE_NEWPID` | Folyamatazonosítók: A konténer fő folyamata a konténeren belül a PID 1 azonosítót kapja, miközben a gazdagépen (Host) egy teljesen más, magas PID-en (pl. PID 48210) fut. A konténer nem látja a host többi folyamatát. |
| **NET** | `CLONE_NEWNET` | Hálózati stack: Saját virtuális hálózati interfészek (`eth0`, `lo`), saját IP-cím, saját útválasztási tábla (routing table), saját porttér és saját iptables tűzfalszabályok. |
| **MNT** | `CLONE_NEWNS` | Csatolási pontok (Mount points): Saját gyökérfájlrendszer (`/`) és csatolási tábla, így a konténer nem látja a gazdagép partícióit. |
| **UTS** | `CLONE_NEWUTS` | Hostnév és domain név: Lehetővé teszi, hogy minden konténernek saját hostneve legyen (amit a `hostname` parancs kiír). |
| **IPC** | `CLONE_NEWIPC` | Folyamatok közötti kommunikáció: Izolált megosztott memóriaterületek (Shared Memory) és POSIX üzenetsorok. |
| **USER** | `CLONE_NEWUSER` | Felhasználói és csoportazonosítók (UID/GID): Lehetővé teszi a Rootless működést, ahol a konténeren belüli root (UID 0) a fizikai gazdagépen egy alacsony jogosultságú felhasználóra (pl. UID 100000) van leképezve. |
| **CGROUP** | `CLONE_NEWCGROUP` | Control Group gyökérkönyvtár: Elrejti a gazdagép teljes cgroup hierarchiáját a konténer elől. |

### Profi hibakeresési trükk (nsenter) – Hogyan lépj be egy konténer hálózatába, ha nincs benne semmilyen parancs?

Éles környezetben a konténerekben nincs telepítve sem `tcpdump`, sem `ip`, sem `netstat`, sőt gyakran még `/bin/sh` sem (pl. Distroless image-ek). Mivel azonban a konténer hálózata csak egy Linux Namespace a gazdagépen, a hostról a `nsenter` paranccsal beléphetsz a konténer hálózati névterébe úgy, hogy közben a gazdagépre telepített diagnosztikai programokat használod:

```bash
# 1. Lekérdezzük a futó konténer gazdagépbeli (Host) PID-jét:
PID=$(docker inspect -f '{{.State.Pid}}' my-app)

# 2. A hostról belépünk a konténer hálózati (-n) névterébe, és lefuttatjuk a hoston lévő tcpdump vagy ss parancsot:
sudo nsenter -t $PID -n ss -tulpn
sudo nsenter -t $PID -n tcpdump -i eth0 port 3000
```

## B) Control Groups – cgroups (Erőforrás-korlátozás – „Mennyit fogyaszthat a folyamat?”)

Míg a Namespaces azt szabályozza, hogy **mit lát** a konténer, addig a cgroups (Control Groups) azt korlátozza, hogy **mennyi** fizikai erőforrást (CPU, RAM, Disk I/O, PID szám) használhat fel. (A modern Linux rendszerek – pl. Fedora, Ubuntu 22.04+, Debian 11+ – már az egységesített cgroups v2 hierarchiát használják a `/sys/fs/cgroup` alatt).

A két legfontosabb erőforrás viselkedése túllépés esetén (**Klasszikus interjúkérdés!**):

- **CPU korlátozás (`--cpus="1.5"`):** A CPU egy **összenyomható (compressible)** erőforrás. Ha a konténer több CPU-időt kér, mint amennyi a limitje (`cpu.max`), a konténer **NEM** áll le, csupán a Linux kernel ütemezője (CFS) visszafogja (CPU Throttling), így a válaszidő megnő.
- **Memória korlátozás (`--memory="512m"`):** A RAM egy **nem összenyomható (incompressible)** erőforrás. Ha a konténer memóriahasználata eléri az 512 MB-os határt (`memory.max`), a Linux kernel **OOM Killer** (Out-Of-Memory Killer) mechanizmusa azonnal közbelép, és egy `SIGKILL` (9-es) jellel kíméletlenül lelövi a konténer folyamatát.

**Hogyan ismered fel az OOM Killt?** A leállt konténer kilépési kódja **Exit Code 137** lesz (128 + 9 [SIGKILL] = 137), és a `docker inspect` kimenetében az `"OOMKilled": true` mező látható!

## C) Union Filesystem (OverlayFS / overlay2)

A Docker alapértelmezett tárolómotorja Linuxon az `overlay2`, amely rétegekből állítja össze a konténer fájlrendszerét:

- **lowerdir (Image rétegek):** A letöltött képfájl egymásra épülő, csak olvasható (Read-Only) rétegei. Ezeket több futó konténer is közösen használhatja.
- **upperdir (Container réteg):** Amikor elindítod a konténert, kap egy vékony, írható-olvasható (Read-Write) felső réteget.
- **merged (Egyesített nézet):** Ez az a virtuális csatolási pont, amit a konténeren belül `/` gyökérkönyvtárként látsz.

**Copy-on-Write (CoW) buktató:** Ha a konténer módosítani akar egy fájlt, amely az alsó (lowerdir) rétegben van, az OverlayFS először teljes egészében felmásolja a fájlt az upperdir rétegbe, és ott módosítja. Ezért lassú, ha nagy fájlokat (pl. adatbázisokat) közvetlenül a konténer fájlrendszerében módosítunk: a **Docker Volume** teljesen megkerüli az OverlayFS-t, és közvetlenül a host fájlrendszersebességével ír a lemezre!

## D) A Docker Daemon és az OCI (Open Container Initiative) Folyamatlánc

Amikor kiadod a `docker run` parancsot, valójában nem maga a `dockerd` futtatja a konténert, hanem egy szabványosított komponenslánc:

```text
[ docker CLI ]
      │ (REST API hívás a /var/run/docker.sock Unix socketen keresztül)
      ▼
[ dockerd (Docker Daemon) ]  --> Kezeli az image-eket, hálózatokat, volume-okat
      │ (gRPC híváson keresztül átadja a vezérlést)
      ▼
[ containerd ]               --> Magas szintű, iparági szabvány konténer-életciklus kezelő démon
      │ (Minden indított konténerhez létrehoz egy felügyelő folyamatot)
      ▼
[ containerd-shim ]          --> Fenntartja a konténer STDIN/STDOUT csatornáit és figyeli a kilépési kódot
      │ (Meghívja az OCI futtatókörnyezetet, ami létrehozza a Namespace-eket és cgroupokat, majd kilép)
      ▼
[ runc (OCI Runtime) ]       --> Létrehozza a konténert a Linux kernelben, majd átadja a helyét az alkalmazásnak
      │
      ▼
[ Alkalmazás folyamat (pl. node server.js) ] (A containerd-shim közvetlen gyermekfolyamataként fut!)
```

**Miért fontos ezt érteni?**

- Mivel a futó konténer szülőfolyamata a `containerd-shim` (és nem a `dockerd`), a Docker démon leállása vagy frissítése során a konténerek tovább futhatnak leállás nélkül (ha az `/etc/docker/daemon.json` fájlban be van kapcsolva a `"live-restore": true`).
- A Kubernetes (K8s) pontosan ezért hagyta el a `dockerd` használatát: a Kubernetes közvetlenül a `containerd`-vel kommunikál, kihagyva a felesleges középső Docker réteget.

# 2. Production-Grade Dockerfile és Image Optimalizálás

Egy állásinterjún a próbafeladat szinte mindig tartalmaz egy „Írj egy Dockerfile-t ehhez az alkalmazáshoz!” kérést. Az alábbi 5 szempont dönti el, hogy Junior vagy Medior/Profi megoldást adsz-e be.

## 1. Layer Caching (Réteg-gyorsítótárazás) és .dockerignore

- **Build Context és .dockerignore:** Amikor kiadod a `docker build .` parancsot, a Docker CLI az aktuális mappa (`.`) teljes tartalmát becsomagolja és elküldi a Docker démonnak. Ha nincs `.dockerignore` fájlod, beküldi a több száz megabájtos `.git` mappát, a helyi `node_modules`-t vagy `target/` könyvtárat, és a lokális `.env` titkaidat is!
- **Cache érvénytelenítés (Cache Invalidation):** A Docker minden Dockerfile utasításból külön réteget csinál. Ha egy réteg tartalma megváltozik (pl. egy `COPY` utasításnál módosult egyetlen fájl), az utána következő összes réteg cache-e érvénytelenné válik, és újra lefut.
- **Szabály:** Mindig a legritkábban változó utasításokat (OS csomagok, majd `package.json` / `pom.xml` / `requirements.txt` és a függőségek letöltése) tedd előre, és a leggyakrabban változó forráskódot (`COPY src/ ./src/`) a legvégére!

## 2. Multi-Stage Builds (Többlépcsős építés)

A Multi-Stage build lehetővé teszi, hogy egyetlen Dockerfile-on belül több `FROM` blokkot használj:

- Az első (**builder**) szakaszban egy nagy méretű, fordítóprogramokat (TypeScript compiler, Maven, GCC, Go SDK) tartalmazó image-ben lefordítod a kódot.
- A második (**production / runner**) szakaszban egy tiszta, minimalista image-ből indulsz, és a `COPY --from=builder` utasítással kizárólag a kész, lefordított állományokat emeled át. A fordítóprogramok, tesztfájlok és dev-függőségek nem kerülnek bele a végső image-be!

## 3. Base Image választási szempontok

- **`ubuntu` / `debian` (Teljes image-ek, 80–120 MB+):** Túl nagyok, sok felesleges segédprogramot (és ezzel potenciális biztonsági rést / CVE-t) tartalmaznak. Csak akkor használd, ha speciális rendszerkönyvtárakra van szükséged.
- **`*-slim` (pl. `node:20-bookworm-slim`, `python:3.12-slim`, 40–70 MB):** A legjobb általános választás éles környezetbe! Minimalista Debian alap, de a szabványos `glibc` C-könyvtárat használja, így minden előre fordított csomag hibátlanul és gyorsan fut rajta.
- **`*-alpine` (pl. `node:20-alpine`, 5–40 MB):** Rendkívül kicsi, de a `glibc` helyett a `musl libc` könyvtárat használja. **Buktató:** Python adatfeldolgozó csomagoknál (`pandas`, `numpy`) az Alpine nem tudja használni az előre lefordított `manylinux` csomagokat, ezért forrásból próbálja lefordítani őket, ami 10 perces buildidőt és óriási image-et eredményez! Emellett a musl DNS-feloldója sajátosan működik, ami nagy terhelésű Kubernetes környezetben DNS-hibákat okozhat.
- **`gcr.io/distroless/*` vagy `scratch` (2–20 MB):** A Google Distroless képfájljai még parancsértelmezőt (`/bin/sh`) és csomagkezelőt sem tartalmaznak, kizárólag a futtatókörnyezetet (pl. Go binárisokhoz vagy Java JRE-hez). Ha egy támadó feltöri az alkalmazást (RCE), nem tud shell parancsokat futtatni a konténerben.

## 4. PID 1, Signal Handling és a Graceful Shutdown

Amikor a Docker (vagy a Kubernetes) le akar állítani egy konténert, a következő történik:

1. Küld egy `SIGTERM` jelet a konténerben futó PID 1 (legelső) folyamatnak. Ez egy udvarias kérés: „Kérlek, fejezd be az éppen futó HTTP kérések kiszolgálását, zárd le az adatbázis-kapcsolatokat, és állj le!”
2. Vár 10 másodpercet (Kubernetesben alapból 30 másodpercet).
3. Ha a folyamat még mindig fut, küld egy `SIGKILL` jelet, ami azonnal, adatmentés nélkül kilövi a konténert.

**Miért veszélyes a Shell Form a Dockerfile-ban?**

- **ROSSZ (Shell form):** `CMD node server.js`
  A Docker ezt a háttérben így futtatja le: `/bin/sh -c "node server.js"`. Emiatt a PID 1 folyamat a `/bin/sh` lesz, a `node` pedig csak a gyermekfolyamata (PID 2). A `/bin/sh` **nem továbbítja** a `SIGTERM` jelet a gyermekfolyamatainak! Eredmény: a Node.js nem tudja, hogy le kellene állnia, a konténer 10 másodpercig nem csinál semmit, majd a `SIGKILL` adatvesztéssel lelövi.
- **JÓ (Exec form + tini):** `CMD ["node", "server.js"]`
  Itt nincs köztes shell, közvetlenül a megadott folyamat indul el. Ha mégis szeretnél egy valódi, könnyűsúlyú init folyamatot a PID 1 helyére (ami begyűjti az árva/zombi folyamatokat és tisztán továbbítja a jeleket), használd a Docker beépített `--init` kapcsolóját, vagy építsd be a `tini`-t az image-be!

## Minta: Iparági szabványú, Multi-Stage Dockerfile és .dockerignore

### 1. A `.dockerignore` fájl (mindig a Dockerfile mellett):

```text
.git
.gitignore
.env
.env.*
node_modules
npm-debug.log
dist
coverage
Dockerfile*
compose*.yaml
docker-compose*.yml
README.md
```

### 2. A Multi-Stage Dockerfile (példa: TypeScript / Node.js alkalmazás):

```dockerfile
# ==========================================
# 1. STAGE: Builder (Fordítási környezet)
# ==========================================
FROM node:20-bookworm-slim AS builder

WORKDIR /build

# Először csak a csomagleírókat másoljuk a Layer Cache kihasználásához
COPY package.json package-lock.json ./

# Telepítjük az összes függőséget (beleértve a TypeScript fordítót is)
# Az npm ci garantálja a package-lock.json szerinti determinisztikus telepítést
RUN npm ci

# Most másoljuk be a forráskódot, és lefordítjuk (pl. TS -> JS a ./dist mappába)
COPY tsconfig.json ./
COPY src/ ./src/
RUN npm run build

# Leszedjük a devDependencies csomagokat, hogy csak a futáshoz szükségesek maradjanak
RUN npm prune --production


# ==========================================
# 2. STAGE: Production Runner (Végső image)
# ==========================================
FROM node:20-bookworm-slim AS runner

# Telepítjük a tini-t (PID 1 init rendszer a tiszta leálláshoz és zombi-kezeléshez)
RUN apt-get update && apt-get install -y --no-install-recommends tini \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Környezet beállítása production módra
ENV NODE_ENV=production

# Átmásoljuk a builder stage-ből KIZÁRÓLAG a szűrt node_modules-t és a lefordított kódot,
# egyből a korlátozott jogú "node" felhasználó (UID 1000) tulajdonába adva
COPY --from=builder --chown=node:node /build/node_modules ./node_modules
COPY --from=builder --chown=node:node /build/dist ./dist
COPY --from=builder --chown=node:node /build/package.json ./

# Biztonság: elhagyjuk a root jogosultságot!
USER node

EXPOSE 3000

# A tini lesz a PID 1, ami elindítja és menedzseli a node folyamatot (Exec form!)
ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["node", "dist/server.js"]
```

# 3. Haladó Docker Hálózatkezelés és a Tűzfal-csapda

## Hogyan működik a bridge hálózat a Linuxban?

Amikor létrehozol egy konténert egy bridge hálózaton:

1. A Docker létrehoz egy virtuális switch-et (Linux Bridge interfészt, alapból `docker0`, egyéni hálózatnál `br-xxxx` néven) a gazdagépen.
2. Létrehoz egy virtuális Ethernet kábelpárt (**veth pair**): a kábel egyik végét (`eth0`) bedugja a konténer saját NET névterébe, a másik végét (`vethXXXX`) pedig a gazdagép Linux bridge interfészébe.
3. A kimenő és bejövő (`-p`) forgalmat a Linux kernel `iptables` / `nftables` szabályai irányítják (NAT tábla, PREROUTING és POSTROUTING MASQUERADE láncok).

## A „Docker + UFW / Firewalld” biztonsági csapda (Nagyon fontos!)

Sok kezdő rendszergazda elköveti azt a hibát, hogy egy publikus VPS/Cloud szerveren elindít egy adatbázist `-p 27017:27017` kapcsolóval, majd bekapcsolja az Ubuntu `ufw` tűzfalát (`ufw deny 27017`), azt gondolva, hogy az adatbázis védve van.

**A valóság:** A Docker `-p` (`0.0.0.0:27017->27017`) kapcsolója közvetlenül az `iptables` PREROUTING (NAT) láncába szúrja be a porttovábbítási szabályát, ami **hamarabb fut le**, mint az `ufw` INPUT lánca! Így a Docker teljesen megkerüli a host tűzfalát, és a 27017-es port a világ felé nyitva áll!

**A helyes védekezés (3 szinten):**

1. **Soha ne használj `-p` (`ports:`) publikálást belső adatbázisokon!** Ha az alkalmazás és az adatbázis is konténerben fut, a belső Docker hálózaton `-p` nélkül is elérik egymást.
2. Ha a gazdagépről mégis el akarod érni a portot (pl. SSH-tunnelen keresztül), kösd kifejezetten a loopback interfészre: `-p 127.0.0.1:27017:27017`.
3. Éles szerveren a külvilág felé kizárólag egyetlen Reverse Proxy (Traefik, Nginx, Caddy) konténer nyithatja meg a 80 és 443 portokat.

# 4. Production-Ready compose.yaml (Minden egyben sablon)

Az alábbi `compose.yaml` (Docker Compose V2 szintaxis) megmutatja, hogyan néz ki a videóban szereplő alkalmazás akkor, ha valódi éles követelményeknek kell megfelelnie:

- **Hálózati szegmentáció:** A `frontend-net` hálózaton csak a Reverse Proxy és a Node.js app van; a `backend-net` egy teljesen elzárt (`internal: true`, még internetkapcsolattal sem rendelkező!) belső hálózat a Node.js app és a MongoDB között. Tehát a Reverse Proxy közvetlenül még csak nem is látja az adatbázist!
- **Log rotáció (`logging`):** Alapértelmezésben a Docker a végtelenségig gyűjti a konténerek JSON logjait a `/var/lib/docker/containers/` alatt, amíg be nem telik a szerver lemeze és le nem áll az egész gép. A `max-size` és `max-file` beállítása kötelező!
- **Biztonsági korlátozások (`cap_drop`, `read_only`, `security_opt`):** Minimalizálja a konténer jogosultságait.
- **Erőforrás-limitek (`deploy.resources.limits`):** Megvédi a szervert attól, hogy egy memóriaszivárgás (memory leak) miatt az egész gazdagép lefagyjon.

```yaml
services:
  # 1. Reverse Proxy (Egyetlen belépési pont a külvilág felől)
  reverse-proxy:
    image: traefik:v3.0
    command:
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
    ports:
      - "80:80"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - frontend-net
    restart: unless-stopped
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  # 2. Saját Backend Alkalmazásunk
  my-app:
    build:
      context: .
      dockerfile: Dockerfile
    image: ghcr.io/my-org/my-app:1.2.0
    restart: unless-stopped
    # Nincs "ports:"! Kívülről közvetlenül nem érhető el, csak a Traefik proxyn keresztül:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.example.com`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=3000"
    # Környezeti változók külső .env fájlból (nem égetjük be a YAML-be a jelszavakat!)
    environment:
      - MONGO_HOST=mongodb
      - MONGO_USER=${MONGO_USER}
      - MONGO_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password
    # Csak akkor indul el, ha a MongoDB már ténylegesen válaszol a pingre!
    depends_on:
      mongodb:
        condition: service_healthy
    networks:
      - frontend-net
      - backend-net
    # Security Hardening: csak olvasható gyökérfájlrendszer és eldobott Linux kernel képességek
    read_only: true
    tmpfs:
      - /tmp
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges:true
    # Erőforrás-korlátok (cgroups)
    deploy:
      resources:
        limits:
          cpus: "0.50"
          memory: 256M
        reservations:
          memory: 128M
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  # 3. Adatbázis (Teljesen elzárva a külvilágtól)
  mongodb:
    image: mongo:7.0.8
    restart: unless-stopped
    environment:
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_USER}
      - MONGO_INITDB_ROOT_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - mongo-data:/data/db
    networks:
      - backend-net
    # Valódi állapotellenőrzés (Healthcheck)
    healthcheck:
      test: ["CMD", "mongosh", "--quiet", "--eval", "db.adminCommand('ping').ok"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 1G
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "5"

# Hálózati szeparáció
networks:
  frontend-net:
    driver: bridge
  backend-net:
    driver: bridge
    internal: true   # Teljesen zárt belső hálózat, nincs kimenő internet sem!

# Perzisztens tároló
volumes:
  mongo-data:
    driver: local

# Titkok biztonságos becsatolása fájlként (/run/secrets/db_password)
secrets:
  db_password:
    file: ./secrets/db_password.txt
```

# 5. Automatizált CI/CD Pipeline és DevSecOps (GitHub Actions példa)

A munkahelyeken soha nem a saját laptopodon adod ki a `docker build` és `docker push` parancsokat, hanem egy CI/CD pipeline (GitHub Actions, GitLab CI, Jenkins) végzi el ezt minden `git push` után, kiegészítve egy automatikus biztonsági sérülékenység-vizsgálattal (Trivy).

Így néz ki egy komplett, éles `.github/workflows/docker-ci.yml` pipeline, amely felépíti az image-et, átvizsgálja biztonsági résekre, és ha tiszta, feltölti a GitHub Container Registry-be (`ghcr.io`):

```yaml
name: Build, Scan and Push Docker Image

on:
  push:
    branches: [ "main" ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-scan-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      security-events: write

    steps:
      - name: 1. Forráskód letöltése (Checkout)
        uses: actions/checkout@v4

      - name: 2. Docker Buildx beállítása (haladó cache támogatáshoz)
        uses: docker/setup-buildx-action@v3

      - name: 3. Image felépítése helyben a biztonsági teszthez
        uses: docker/build-push-action@v5
        with:
          context: .
          load: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: 4. DevSecOps: Image ellenőrzése Trivy CVE szkennerrel
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          format: 'table'
          # Ha javítható CRITICAL vagy HIGH sérülékenység van az image-ben,
          # a pipeline hibával (exit 1) megáll, és NEM tölti fel a hibás image-et!
          exit-code: '1'
          ignore-unfixed: true
          vuln-type: 'os,library'
          severity: 'CRITICAL,HIGH'

      - name: 5. Bejelentkezés a Container Registry-be (GHCR)
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: 6. Sikeresen ellenőrzött Image feltöltése (Push)
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
          cache-from: type=gha
```

# 6. A DevOps Ökoszisztéma Térképe – Mire épül a munka a Docker felett?

Ha Junior/Medior DevOps, Cloud vagy Platform Engineer pozícióra pályázol, a Docker az alap, amelyre a következő technológiai rétegek épülnek. Érdemes úgy felépítened a portfóliódat, hogy minden rétegből legyen egy-egy gyakorlati tapasztalatod:

| Réteg | Iparági Szabvány Eszközök | Mi a feladata a munkafolyamatban? |
|---|---|---|
| **1. Operációs rendszer és Hálózat** | Linux (Debian/RHEL/Fedora), Bash, TCP/IP, DNS, TLS, iptables/nftables, SSH | Minden konténer és felhős virtuális gép alapja. Ha nem érted a DNS-t, a subneteket, a routolást vagy a Linux jogosultságokat, nem fogod tudni debuggolni a konténereket sem. |
| **2. Konténerizáció** | Docker, Podman, containerd, Buildx, Trivy | Az alkalmazások szabványos, izolált OCI képfájlokba csomagolása és biztonsági ellenőrzése. |
| **3. CI/CD Pipeline-ok** | GitHub Actions, GitLab CI, Jenkins | A tesztelés, a Docker image építés, a biztonsági szkennelés és a verziózás teljes automatizálása minden git commit után. |
| **4. Konténer-orkesztráció** | Kubernetes (K8s / K3s), Helm, Kustomize | Konténerek tucatjainak automatikus elosztása több szerver (Node) között, öngyógyítás (Self-healing), automatikus skálázás, terheléselosztás (Service/Ingress) és leállásmentes frissítés (Rolling Update). |
| **5. GitOps (Continuous Delivery)** | ArgoCD, Flux | A Kubernetes klaszter kívánt állapotát egy Git repository-ban tároljuk. Az ArgoCD figyeli a Git-et, és automatikusan szinkronizálja a klasztert a Git-ben lévő YAML/Helm leírókkal. |

 [Docker & OCI Rendszerarchitektúra](https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Docker/nanavd/interactive.html)
