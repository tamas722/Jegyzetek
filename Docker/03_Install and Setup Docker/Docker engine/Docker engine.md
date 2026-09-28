# A Docker Engine: A Konténerizáció Motorja

A Docker Engine a Docker platform központi, nyílt forráskódú szoftvere, amely a konténerek építéséért, futtatásáért és menedzseléséért felel. Klasszikus kliens-szerver architektúrára épül, és szorosan együttműködik a gazdagép (host) operációs rendszerével a folyamatok elszigetelése (isolation) érdekében.

## 1. A Docker Engine Három Fő Komponense

A Docker Engine nem egyetlen monolitikus program, hanem három egymással kommunikáló rétegből áll:

- **Docker Daemon (`dockerd`):** A háttérben futó, hosszan futó szolgáltatás (szerver). Ő végzi a tényleges "nehézemelést": figyeli az API kéréseket, és kezeli a Docker objektumokat.
- **REST API:** A démon által biztosított programozási felület. Ez határozza meg, hogy más programok (vagy maga a CLI) milyen HTTP kérésekkel tudnak kommunikálni a démonnal.
- **Docker CLI (`docker`):** A parancssori kliens, amellyel a felhasználó interakcióba lép. A CLI önmagában nem futtat konténereket; a kiadott parancsokat (pl. `docker run`, `docker build`) lefordítja REST API hívásokká, és elküldi a démonnak.

> **Megjegyzés:** A kliens és a démon futhat ugyanazon a gépen, de a kliens akár egy távoli Docker démonhoz is csatlakozhat a hálózaton keresztül.

## 2. A Docker Engine Által Kezelt Objektumok

A `dockerd` a következő négy alapvető entitás életciklusáért felel:

- **Images (Képek):** Csak olvasható sablonok (read-only templates), amelyekből a konténerek felépülnek.
- **Containers (Konténerek):** Az image-ek futtatható példányai (runnable instances). A konténer a legfelső, írható rétegben (writable layer) tárolja a futás közbeni ideiglenes változásokat.
- **Networks (Hálózatok):** Az elszigetelt hálózati réteg, amely lehetővé teszi, hogy a konténerek kommunikáljanak egymással, a gazdagéppel, vagy a külvilággal (pl. bridge, host, overlay hálózatok).
- **Volumes (Kötetek):** A perzisztens (állandó) adattárolás eszközei. Mivel a konténer törlésekor a benne lévő adatok elvesznek, a Volume-ok biztosítják, hogy az adatbázisok vagy fájlok megmaradjanak a gazdagép lemezén, teljesen függetlenítve a konténer életciklusától.

## 3. Az Architektúra a Motorháztető Alatt (Moduláris felépítés)

Bár korábban a Docker Engine csinált mindent egyben, ma már az OCI (Open Container Initiative) szabványok miatt a feladatok szét vannak bontva. A Docker Engine valójában karmesterként (orchestrator) működik más, alacsonyabb szintű eszközök felett:

- **`dockerd` (Docker Daemon):** Fogadja a felhasználói kérést, kezeli az image-eket (letöltés, kicsomagolás) és a hálózati/kötet beállításokat.
- **`containerd` (High-level Runtime):** A démon átadja neki a feladatot. A containerd felügyeli a konténerek életciklusát az adott gépen.
- **`containerd-shim`:** Egy kis köztes réteg, amely elválasztja a futó konténert a containerd-től, így a démon vagy a containerd újraindítható anélkül, hogy a futó konténerek leállnának (daemonless containers).
- **`runc` (Low-level OCI Runtime):** A legalsó réteg, amely ténylegesen "beszél" a Linux kernellel, és létrehozza a Namespace-eket (elszigetelés) és a Cgroup-okat (erőforrás-korlátozás).

## 4. A `docker run` Parancs Életciklusa

Mi történik pontosan, amikor kiadod a `docker run nginx` parancsot?

1. **Kérés:** A Docker CLI (`docker`) a háttérben egy REST API kérést küld a Docker Daemonnak (`dockerd`).
2. **Kép ellenőrzése:** A démon megnézi, hogy az `nginx` kép (image) megvan-e a helyi gyorsítótárban (cache).
3. **Letöltés (Pull):** Ha nincs meg, a démon letölti a megadott Registry-ből (pl. Docker Hub).
4. **Példányosítás:** A démon létrehoz egy új konténert a kép alapján (hozzáad egy írható réteget a tetejére).
5. **Erőforrások allokálása:** Létrehozza a szükséges hálózati interfészeket, beköti (mount) a kért köteteket (volumes), és meghatározza az IP-címet.
6. **Indítás:** A démon utasítja a `containerd`-t, hogy a `runc` segítségével ténylegesen indítsa el a folyamatot (process) a kernellel elkülönítve, és összeköti a standard kimeneteket (STDOUT, STDERR) a klienssel.
