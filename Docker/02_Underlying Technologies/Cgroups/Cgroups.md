
# Hogy működik valójában a Docker? (Technical Deep Dive)

A konténerek megváltoztatták a szoftverfejlesztést, lecserélve a monolitikus architektúrákat mikroszolgáltatásokra (microservices).

A legfőbb különbség a VM (Virtuális Gép) és a Konténer között: A VM-ek hardvert virtualizálnak (mindegyik saját teljes operációs rendszerrel fut), míg a konténerek az operációs rendszert (OS-t) virtualizálják, megosztva a gazdagép (host) kernelét, ezáltal sokkal gyorsabbak és erőforrás-takarékosabbak.

## 1. A Docker Architektúrája

A Docker egy kliens-szerver modellt használ.

**Magas szintű (High-level) architektúra:**

- **Docker Client (Kliens):** A parancssori eszköz (CLI), amivel a felhasználó kommunikál (pl. `docker run`). REST API-n keresztül (UNIX socketen vagy hálózaton) küld utasításokat a démonnak.
- **Docker Daemon (`dockerd`):** A háttérben futó szolgáltatás, amely ténylegesen kezeli a konténereket, képeket, hálózatokat és köteteket.
- **Docker Registry:** A konténerképek tárolója (pl. Docker Hub).

**Alacsony szintű (Low-level) architektúra (Hogy indul el egy konténer?):**

1. A Docker Client szól a Docker Daemonnak (`dockerd`).
2. A `dockerd` nem maga futtatja a konténert, hanem átadja a feladatot a `containerd`-nek (ami egy szabványos ipari container runtime).
3. A `containerd` a shim API-n (runtime v2) keresztül kommunikál az OCI runtime-mal.
4. Az OCI runtime (legtöbbször a `runc`) az, ami ténylegesen létrehozza a konténert a Linux kernel funkcióinak (Namespaces, Cgroups) felhasználásával.

## 2. A Konténerek Lelke: A Linux Kernel Funkciói

Technikailag "Linux konténer" mint olyan nem is létezik a kernelben. A konténer valójában csak egy elszigetelt és korlátozott Linux folyamat (process). Ezt az elszigetelést két fő kernel funkció biztosítja:

### A) Névterek (Namespaces) – Az elszigetelés (Isolation)

A névterek elhitetik a folyamattal, hogy övé az egész rendszer. 6+1 fő névtér létezik:

- **PID:** Elszigeteli a folyamatazonosítókat (a konténerben futó program azt hiszi, ő az 1-es PID (init)).
- **NET:** Saját hálózati interfészek, IP címek, routing táblák.
- **MNT:** Fájlrendszer csatolási pontok (mount points) elszigetelése.
- **USER:** Felhasználói és csoport azonosítók (a konténerben lehet root, miközben a hoston normál felhasználó).
- **UTS:** Elszigeteli a hosztnevet (hostname).
- **IPC:** Folyamatok közötti kommunikáció (Inter-process communication) elszigetelése.

### B) Cgroups (Control Groups) – Az erőforrás-korlátozás (Limitation)

Megakadályozzák, hogy egy konténer feleméssze a gazdagép összes erőforrását.

- Korlátozzák a CPU, Memória, I/O és Hálózat használatát.
- **Rootless konténerek:** A cgroup v2 és a User névterek teszik lehetővé, hogy normál, jogosultság nélküli felhasználók is futtathassanak és kezelhessenek konténereket, növelve a biztonságot (a Docker és a RootlessKit segítségével).

## 3. Konténer Képek (Images) és Tárolás

A konténerkép (Image) egy megváltoztathatatlan (immutable), statikus fájl, amely tartalmazza a futtatható kódot és a függőségeket.

- **Rétegek (Layers) és CoW (Copy-on-Write):** Az image-ek rétegekből állnak (minden Dockerfile utasítás, pl. `RUN`, új réteget hoz létre). A rétegek csak olvashatók (read-only). Amikor egy konténer elindul, kap egy legfelső, írható réteget (writable layer). A CoW stratégia miatt a módosítások csak ezen az írható rétegen történnek (általában az `overlay2` storage driver használatával).
- **Docker Manifest:** JSON fájl, ami leírja a képfájlt (tagek, architektúra, digitális aláírás).
- **Multi-stage builds:** A Dockerfile-on belül több `FROM` utasítással optimalizálható a méret, így csak a lefordított fájlok kerülnek a végső képbe, a fordítóeszközök nem.
- **Scratch & Distroless:** A `FROM scratch` egy teljesen üres képfájl. A distroless képek csak az alkalmazást és annak futásidejű függőségeit tartalmazzák (nincs bennük shell vagy csomagkezelő), ami drasztikusan csökkenti a méretet és növeli a biztonságot.

## 4. Szabványok és az OCI (Open Container Initiative)

Az OCI egy nyílt iparági szabvány, amelyet a Docker, a CoreOS és mások hoztak létre, hogy egységesítsék a konténer formátumokat.

- **Két fő specifikáció:** Image-spec (Képfájl formátum) és Runtime-spec (Futási környezet és életciklus, pl. `config.json`).
- **Runtimes (Futási környezetek):**
  - **`runc`:** Az OCI referencia implementációja (Go és C nyelven). Ő végzi a piszkos munkát a kernellel.
  - **`crun`:** A Red Hat által fejlesztett, C-ben írt runtime (sokkal gyorsabb és kevesebb memóriát eszik, mint a `runc`).
  - **`youki` / `railcar`:** Rust nyelvű implementációk a memóriabiztonság és sebesség érdekében.

## 5. A CRI (Container Runtime Interface) és a Kubernetes

A Kubernetes (K8s) a konténer-hangszerelés (orchestration) királya.

- **Docker leválasztása (Deprecation):** A Kubernetes 1.20-tól kezdve kivezette a `dockershim`-et. A Kubernetes már nem közvetlenül a Dockert használja, hanem a CRI szabványon keresztül kommunikál a runtime-okkal (mint a `containerd` vagy a `CRI-O`). A Docker mint "közvetítő" kikerült a képből.
- **`containerd`:** Ipari szabványú runtime. A Docker is ezt használja a motorháztető alatt.
- **CLI eszközök:** A Kubernetes node-ok hibaelhárításához a Docker CLI helyett a `crictl` (CRI-kompatibilis), vagy a kísérleti, Docker-szerű `nerdctl` használatos.

## 6. Építés és Biztonság (Build & Security)

- **BuildKit / Buildx:** A Docker modern, gyorsítótár-hatékony, párhuzamos építőeszköze. (A `DOCKER_BUILDKIT=1` bekapcsolásával vagy a `docker buildx` használatával aktiválható).
- **Alternatív Builder-ek** (Docker démon nélkül is lehet építeni!): `ko` (Go appokhoz), `kaniko`, `buildah`, `podman`.
- **Biztonság:**
  - **SBOM (Software Bill of Materials):** A szoftver "összetevőinek" listája. Elengedhetetlen a sebezhetőségek (pl. a `trivy` vagy a `docker scout` által végzett) kereséséhez.
  - **Aláírás (Signing):** A Cosign / Sigstore segítségével digitálisan aláírhatók a képek, garantálva, hogy senki nem módosította őket a build és a futtatás között.

## Forrás
-[https://medium.com/@furkan.turkal/how-does-docker-actually-work-the-hard-way-a-technical-deep-diving-c5b8ea2f0422]
