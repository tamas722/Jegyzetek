# A Docker Build Cache (Építési Gyorsítótár) Működése

Amikor a `docker build` (vagy `docker buildx build`) paranccsal felépítesz egy Docker image-et, a Docker sorról sorra olvassa a Dockerfile-t. A folyamat felgyorsítása érdekében a Docker egy gyorsítótárat (cache) használ, amely eltárolja az egyes utasítások (rétegek) eredményét. Ha egy utasítás és annak kontextusa nem változott a legutóbbi építés óta, a Docker nem hajtja végre újra, hanem a gyorsítótárból tölti be a már kész réteget (ezt hívják Cache Hit-nek).

## 1. A Cache Működési Elve és Logikája

A Docker a következő algoritmust használja a Dockerfile minden egyes lépésének kiértékelésekor:

1. A Docker megvizsgálja a gyorsítótárban lévő előző réteget (parent image).
2. Megnézi a következő utasítást a Dockerfile-ban.
3. Összehasonlítja az utasítást az összes olyan származtatott réteggel (child layer), amely ebből az alaprétegből készült.
4. Ha pontos egyezést talál, felhasználja a cache-elt réteget.
5. Ha nem talál egyezést, a cache "megtörik" (érvénytelenítődik), és az adott réteget, valamint az összes utána következő réteget teljesen újra kell építeni. Ezt hívják kaszkád-hatásnak vagy dominó-elvnek.

## 2. A Cache Érvénytelenítésének (Invalidation) Szabályai

A Docker különbözőképpen vizsgálja az utasításokat, hogy eldöntse, használhatja-e a cache-t:

- **ADD és COPY utasítások:**
  A Docker kiszámolja a bemásolandó fájlok tartalmának ellenőrzőösszegét (checksum, általában SHA256). Az utolsó módosítás idejét (mtime) vagy a hozzáférési időt (atime) a Docker nem veszi figyelembe.
  Ha a fájlok tartalmának ellenőrzőösszege akár egyetlen bájttal is eltér az előző buildhez képest, a cache érvénytelenítődik.

- **RUN és egyéb utasítások (pl. ENV, CMD, LABEL):**
  A Docker csak a parancs szövegét (stringjét) vizsgálja meg.
  **Példa:** Ha a `RUN apt-get update` szerepel a fájlban, a Docker csak azt nézi, hogy a szöveg változott-e. Nem tudja, hogy a távoli szerveren frissültek-e a csomagok. Ha a szöveg ugyanaz, a Docker a cache-ből tölti be a régi (esetleg elavult) réteget.

## 3. Legjobb Gyakorlatok a Cache Optimalizálására (Best Practices)

A kaszkád-hatás miatt a Dockerfile sorainak sorrendje kritikus fontosságú. **Alapszabály:** A ritkán változó dolgok kerüljenek felülre, a gyakran változók (pl. a forráskódod) pedig alulra!

### A) A "Függőségek" (Dependencies) trükkje (A legfontosabb szabály)

Ha egy `COPY . .` paranccsal mindent bemásolsz, majd lefuttatod a függőségek telepítését (pl. `npm install`), akkor ha akár egyetlen sor kódod is változik, a teljes `npm install` újra fog futni, ami lassú.

**Helytelen megoldás:**

```dockerfile
COPY . /app
RUN npm install   # Bármilyen fájl változik, ez a nehéz folyamat újra lefut!
```

**Helyes (Cache-optimalizált) megoldás:**

Először csak azokat a fájlokat másold be, amik a függőségek telepítéséhez kellenek (pl. `package.json` vagy `requirements.txt`).

```dockerfile
# 1. Csak a függőségleírókat másoljuk
COPY package*.json ./

# 2. Telepítjük a függőségeket (Ez csak akkor fut újra, ha a package.json megváltozott!)
RUN npm install

# 3. Csak ezután másoljuk be a gyakran változó forráskódot
COPY . .
```

### B) Parancsok Láncolása (Chaining) a RUN utasításnál

A csomagkezelők frissítését és a telepítést mindig egyetlen `RUN` utasításba kell láncolni `&&` jellel.

**Helytelen:**

```dockerfile
RUN apt-get update
RUN apt-get install -y python3   # Ha új csomagot adsz hozzá, az update nem fut le újra a cache miatt!
```

**Helyes:**

```dockerfile
RUN apt-get update && apt-get install -y \
    python3 \
    git \
    && rm -rf /var/lib/apt/lists/*
```

> **Extra tipp:** A cache méretének csökkentése érdekében a végén mindig töröld a csomagkezelő átmeneti gyorsítótárát.

## 4. Modern Cache Funkciók (BuildKit / Buildx)

Az újabb Docker verziók (amelyek alapértelmezetten a BuildKit motort használják) sokkal fejlettebb cache-kezelést tesznek lehetővé. A `--cache-from` és `--cache-to` zászlókkal a gyorsítótárat exportálhatod és importálhatod, ami CI/CD (Continuous Integration) környezetben hatalmas sebességnövekedést jelent.

- **Inline Cache:** A cache adatokat beágyazza magába a felépített Docker image-be.
- **Registry Cache:** A cache adatokat egy távoli registry-be (pl. Docker Hub) tölti fel különálló rétegként.

  ```bash
  docker buildx build --cache-to type=registry,ref=user/app:buildcache --cache-from type=registry,ref=user/app:buildcache .
  ```

- **Local Cache:** A cache-t a helyi lemezed egy specifikus mappájába menti (`type=local`).
- **GHA (GitHub Actions) Cache:** Kifejezetten a GitHub Actions gyorsítótárazó API-jához optimalizált cache backend (`type=gha`).

## 5. A Cache Kezelése és Törlése

### 1. A Cache mellőzése egy adott buildnél

Ha garantálni akarod, hogy a Docker mindent az alapoktól építsen újra (pl. egy `apt-get update` kikényszerítése miatt), használd a `--no-cache` opciót:

```bash
docker build --no-cache -t myapp:latest .
```

### 2. Építési gyorsítótár ürítése (Helyfelszabadítás)

A Docker build cache idővel rengeteg helyet foglalhat a merevlemezen. Az alábbi paranccsal törölheted a build cache-t:

```bash
# Biztonságos törlés (csak a már nem hivatkozott (dangling) cache rétegeket törli):
docker builder prune

# Teljes és agresszív törlés (Minden build cache-t töröl):
docker builder prune -a
```

> **Összegzés:** A Docker build cache megértése az alapja a professzionális és gyors Dockerfile írásának. A "legkevésbé változó rétegektől a leggyakrabban változók felé" elv betartásával percekkel csökkentheted a szoftvered build idejét (CI/CD pipeline-okban ez naponta akár órák megtakarítását is jelentheti).
