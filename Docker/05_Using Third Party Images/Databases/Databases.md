# Konténerizált adatbázisok használata

Ismerd meg, hogyan futtathatsz és kezelhetsz adatbázisokat hatékonyan konténerként.

A helyi konténerizált adatbázis használata rugalmasságot és egyszerű beállítást kínál, lehetővé téve, hogy szorosan tükrözd az éles környezeteket a hagyományos adatbázis-telepítések többletterhe nélkül. A Docker leegyszerűsíti ezt a folyamatot, lehetővé téve, hogy néhány paranccsal telepíts, kezelj és skálázz adatbázisokat izolált konténerekben.

Ebben az útmutatóban megtanulod, hogyan:

- Futtass helyi konténerizált adatbázist
- Érd el egy konténerizált adatbázis shelljét
- Csatlakozz egy konténerizált adatbázishoz a hosztról
- Csatlakozz egy konténerizált adatbázishoz egy másik konténerből
- Őrizd meg az adatbázis adatait egy kötetben
- Építs testreszabott adatbázis-image-et
- Használd a Docker Compose-t egy adatbázis futtatásához

Ez az útmutató a MySQL image-et használja példaként, de a fogalmak más adatbázis-image-ekre is alkalmazhatók.

## Előfeltételek

Az útmutató követéséhez telepítve kell lennie a Dockernek. A Docker telepítéséhez lásd: Get Docker.

## Helyi konténerizált adatbázis futtatása

A legnépszerűbb adatbázis-rendszerek, köztük a MySQL, a PostgreSQL és a MongoDB, rendelkeznek Docker Official Image-mel a Docker Hubon. Ezek az image-ek egy válogatott készlet, amelyek követik a legjobb gyakorlatokat, biztosítva, hogy hozzáférj a legújabb funkciókhoz és biztonsági frissítésekhez. A kezdéshez látogass el a Docker Hubra, és keress rá a számodra érdekes adatbázisra. Minden image oldala részletes utasításokat ad a konténer futtatásához, a beállítás testreszabásához és az adatbázis igényeidnek megfelelő konfigurálásához. Az ebben az útmutatóban használt MySQL image-ről további információért lásd a Docker Hub MySQL image oldalát.

Adatbázis-konténer futtatásához használhatod a Docker Desktop GUI-t vagy a CLI-t.

**CLI / GUI**

Ha CLI-t használva szeretnél konténert futtatni, futtasd a következő parancsot egy terminálban:

```bash
docker run --name my-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -e MYSQL_DATABASE=mydb -d mysql:latest
```

Ebben a parancsban:

- `--name my-mysql` a `my-mysql` nevet adja a konténerednek a könnyebb hivatkozás érdekében.
- `-e MYSQL_ROOT_PASSWORD=my-secret-pw` a MySQL root jelszavát `my-secret-pw`-re állítja. Cseréld le a `my-secret-pw`-t egy általad választott biztonságos jelszóra.
- `-e MYSQL_DATABASE=mydb` opcionálisan létrehoz egy `mydb` nevű adatbázist. A `mydb`-t lecserélheted a kívánt adatbázisnévre.
- `-d` a konténert leválasztott (detached) módban futtatja, ami azt jelenti, hogy a háttérben fut.
- `mysql:latest` meghatározza, hogy a MySQL image legújabb verzióját szeretnéd használni.

Annak ellenőrzéséhez, hogy a konténered fut, futtasd a `docker ps` parancsot egy terminálban.

## Egy konténerizált adatbázis shelljének elérése

Amikor egy adatbázis egy Docker konténeren belül fut, szükség lehet a shelljének elérésére az adatbázis kezeléséhez, parancsok végrehajtásához vagy adminisztratív feladatok elvégzéséhez. A Docker egyszerű módot kínál erre a `docker exec` parancs használatával. Ezenkívül, ha grafikus felületet részesítesz előnyben, használhatod a Docker Desktop GUI-ját.

Ha még nincs futó adatbázis-konténered, lásd: Helyi konténerizált adatbázis futtatása.

**CLI / GUI**

Egy MySQL konténer termináljának eléréséhez CLI-vel a következő `docker exec` parancsot használhatod.

```bash
docker exec -it my-mysql bash
```

Ebben a parancsban:

- `docker exec` megmondja a Dockernek, hogy egy parancsot szeretnél végrehajtani egy futó konténerben.
- `-it` biztosítja, hogy az elért terminál interaktív legyen, így parancsokat írhatsz bele.
- `my-mysql` a MySQL konténered neve. Ha másképp nevezted el a konténert a futtatásakor, használd azt a nevet helyette.
- `bash` a parancs, amelyet a konténeren belül szeretnél futtatni. Ez megnyit egy bash shellt, amely lehetővé teszi, hogy interakcióba lépj a konténer fájlrendszerével és telepített alkalmazásaival.

A parancs végrehajtása után hozzáférést kapsz a MySQL konténeredben lévő bash shellhez, ahonnan közvetlenül kezelheted a MySQL szerveredet. A terminálba való visszatéréshez futtathatod az `exit` parancsot.

Miután elérted a konténer terminálját, futtathatsz bármilyen eszközt, amely elérhető abban a konténerben. A következő példa a `mysql` használatát mutatja a konténerben az adatbázisok listázásához.

```bash
mysql -u root -p
Enter password: my-secret-pw

mysql> SHOW DATABASES;

+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydb               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

## Csatlakozás konténerizált adatbázishoz a hosztról

A hosztgépről egy konténerizált adatbázishoz való csatlakozás magában foglalja a konténeren belüli port leképezését a hosztgéped egy portjára. Ez a folyamat biztosítja, hogy a konténeren belüli adatbázis elérhető legyen a hosztgép hálózatán keresztül. A MySQL esetében az alapértelmezett port a 3306. Ennek a portnak a közzétételével különféle adatbázis-kezelő eszközöket vagy alkalmazásokat használhatsz a hosztgépeden a MySQL adatbázisoddal való interakcióhoz.

Mielőtt elkezded, el kell távolítanod minden korábban futtatott konténert ehhez az útmutatóhoz. Egy konténer leállításához és eltávolításához:

- Egy terminálban futtasd a `docker rm --force my-mysql` parancsot a `my-mysql` nevű konténer eltávolításához.
- Vagy a Docker Desktop Dashboardon válaszd a Delete ikont a konténered mellett a Containers nézetben.

Ezután használhatod a Docker Desktop GUI-t vagy a CLI-t a konténer futtatásához a leképezett porttal.

**CLI / GUI**

Futtasd a következő parancsot egy terminálban.

```bash
docker run -p 3307:3306 --name my-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -e MYSQL_DATABASE=mydb -d mysql:latest
```

Ebben a parancsban a `-p 3307:3306` a hoszt 3307-es portját a konténer 3306-os portjára képezi le.

A port leképezésének ellenőrzéséhez futtasd a következő parancsot.

```bash
docker ps
```

Valami ilyesmit kell látnod:

```plaintext
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                               NAMES
6eb776cfd73c   mysql:latest   "docker-entrypoint.s…"   17 minutes ago   Up 17 minutes   33060/tcp, 0.0.0.0:3307->3306/tcp   my-mysql
```

Ezen a ponton a hosztgépeden futó bármely alkalmazás elérheti a konténerben lévő MySQL szolgáltatást a `localhost:3307` címen.

## Csatlakozás konténerizált adatbázishoz egy másik konténerből

Egy konténerizált adatbázishoz egy másik konténerből való csatlakozás gyakori forgatókönyv a mikroszolgáltatás-architektúrában és a fejlesztési folyamatok során. A Docker hálózati képességei megkönnyítik ennek a kapcsolatnak a létrehozását anélkül, hogy az adatbázist ki kellene tenni a hoszt hálózatára. Ezt úgy érik el, hogy mind az adatbázis-konténert, mind a hozzáférni kívánó konténert ugyanarra a Docker hálózatra helyezik.

Mielőtt elkezded, el kell távolítanod minden korábban futtatott konténert ehhez az útmutatóhoz. Egy konténer leállításához és eltávolításához:

- Egy terminálban futtasd a `docker rm --force my-mysql` parancsot a `my-mysql` nevű konténer eltávolításához.
- Vagy a Docker Desktop Dashboardon válaszd a Delete ikont a konténered mellett a Containers nézetben.

Hálózat létrehozásához és konténerek futtatásához rajta:

1. Futtasd a következő parancsot egy `my-network` nevű Docker hálózat létrehozásához.

   ```bash
   docker network create my-network
   ```

2. Futtasd az adatbázis-konténeredet, és add meg a hálózatot a `--network` opcióval. Ez a konténert a `my-network` hálózaton futtatja.

   ```bash
   docker run --name my-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -e MYSQL_DATABASE=mydb --network my-network -d mysql:latest
   ```

3. Futtasd a többi konténeredet, és add meg a hálózatot a `--network` opcióval. Ebben a példában egy phpMyAdmin konténert fogsz futtatni, amely csatlakozni tud az adatbázisodhoz.

   - Futtass egy phpMyAdmin konténert. Használd a `--network` opciót a hálózat megadásához, a `-p` opciót, hogy a hosztgépedről elérhesd a konténert, és a `-e` opciót egy szükséges környezeti változó megadásához ehhez az image-hez.

     ```bash
     docker run --name my-phpmyadmin -d --network my-network -p 8080:80 -e PMA_HOST=my-mysql phpmyadmin
     ```

4. Ellenőrizd, hogy a konténerek kommunikálni tudnak. Ebben a példában eléred a phpMyAdmin-t, és ellenőrzöd, hogy csatlakozik az adatbázishoz.
   - Nyisd meg a `http://localhost:8080` címet a phpMyAdmin konténered eléréséhez.
   - Jelentkezz be a `root` felhasználónévvel és a `my-secret-pw` jelszóval. Csatlakoznod kell a MySQL szerverhez, és látnod kell a listázott adatbázisodat.

Ezen a ponton a `my-network` konténerhálózaton futó bármely alkalmazás elérheti a konténerben lévő MySQL szolgáltatást a `my-mysql:3306` címen.

## Adatbázis-adatok megőrzése kötetben

Az adatbázis-adatok Docker kötetben való megőrzése szükséges ahhoz, hogy az adataid túléljék a konténer újraindításait és eltávolítását. A Docker kötet lehetővé teszi, hogy az adatbázisfájlokat a konténer írható rétegén kívül tárold, lehetővé téve a konténer frissítését, az alapok cseréjét és az adatok megosztását anélkül, hogy elveszítenéd azokat. Íme, hogyan csatolhatsz egy kötetet az adatbázis-konténeredhez a Docker CLI vagy a Docker Desktop GUI használatával.

Mielőtt elkezded, el kell távolítanod minden korábban futtatott konténert ehhez az útmutatóhoz. Egy konténer leállításához és eltávolításához:

- Egy terminálban futtasd a `docker rm --force my-mysql` parancsot a `my-mysql` nevű konténer eltávolításához.
- Vagy a Docker Desktop Dashboardon válaszd a Delete ikont a konténered mellett a Containers nézetben.

Ezután használhatod a Docker Desktop GUI-t vagy a CLI-t a konténer futtatásához kötettel.

**CLI / GUI**

Az adatbázis-konténer kötettel való futtatásához add meg a `-v` opciót a `docker run` parancsodban, megadva egy kötetnevet és az elérési utat, ahol az adatbázis az adatait a konténeren belül tárolja. Ha a kötet nem létezik, a Docker automatikusan létrehozza a számodra.

Adatbázis-konténer kötettel való futtatásához, majd az adatok megmaradásának ellenőrzéséhez:

1. Futtasd a konténert, és csatold a kötetet.

   ```bash
   docker run --name my-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -e MYSQL_DATABASE=mydb -v my-db-volume:/var/lib/mysql -d mysql:latest
   ```

   Ez a parancs a `my-db-volume` nevű kötetet a konténer `/var/lib/mysql` könyvtárába csatolja.

2. Hozz létre néhány adatot az adatbázisban. Használd a `docker exec` parancsot a `mysql` futtatásához a konténeren belül, és hozz létre egy táblát.

   ```bash
   docker exec my-mysql mysql -u root -pmy-secret-pw -e "CREATE TABLE IF NOT EXISTS mydb.mytable (column_name VARCHAR(255)); INSERT INTO mydb.mytable (column_name) VALUES ('value');"
   ```

   Ez a parancs a konténerben lévő `mysql` eszközt használja egy `mytable` nevű tábla létrehozásához egy `column_name` nevű oszloppal, majd beszúr egy `value` értéket.

3. Állítsd le és távolítsd el a konténert. Kötet nélkül az általad létrehozott tábla elveszne a konténer eltávolításakor.

   ```bash
   docker rm --force my-mysql
   ```

4. Indíts el egy új konténert a kötettel csatolva. Ezúttal nem kell környezeti változókat megadnod, mivel a konfiguráció a kötetben van mentve.

   ```bash
   docker run --name my-mysql -v my-db-volume:/var/lib/mysql -d mysql:latest
   ```

5. Ellenőrizd, hogy az általad létrehozott tábla még mindig létezik. Használd újra a `docker exec` parancsot a `mysql` futtatásához a konténeren belül.

   ```bash
   docker exec my-mysql mysql -u root -pmy-secret-pw -e "SELECT * FROM mydb.mytable;"
   ```

   Ez a parancs a konténerben lévő `mysql` eszközt használja az összes rekord kiválasztásához a `mytable` táblából.

   Valami ilyesmit kell látnod:

   ```plaintext
   column_name
   value
   ```

Ezen a ponton bármely MySQL konténer, amely a `my-db-volume`-ot csatolja, hozzáférhet és mentheti a megőrzött adatokat.

## Testreszabott adatbázis-image építése

Az adatbázis-image testreszabása lehetővé teszi, hogy további konfigurációt, szkripteket vagy eszközöket illessz be az alap adatbázis-szerver mellé. Ez különösen hasznos egy olyan Docker image létrehozásához, amely megfelel a konkrét fejlesztői vagy éles környezeti igényeidnek. A következő példa felvázolja, hogyan építs és futtass egy egyedi MySQL image-et, amely tartalmaz egy tábla-inicializáló szkriptet.

Mielőtt elkezded, el kell távolítanod minden korábban futtatott konténert ehhez az útmutatóhoz. Egy konténer leállításához és eltávolításához:

- Egy terminálban futtasd a `docker rm --force my-mysql` parancsot a `my-mysql` nevű konténer eltávolításához.
- Vagy a Docker Desktop Dashboardon válaszd a Delete ikont a konténered mellett a Containers nézetben.

Az egyedi image építéséhez és futtatásához:

1. Hozz létre egy Dockerfile-t.

   - Hozz létre egy `Dockerfile` nevű fájlt a projekted könyvtárában. Ebben a példában a Dockerfile-t egy általad választott üres könyvtárban hozhatod létre. Ez a fájl határozza meg, hogyan épüljön fel az egyedi MySQL image-ed.
   - Add hozzá a következő tartalmat a Dockerfile-hoz.

     ```dockerfile
     # syntax=docker/dockerfile:1

     # Use the base image mysql:latest
     FROM mysql:latest

     # Set environment variables
     ENV MYSQL_DATABASE mydb

     # Copy custom scripts or configuration files from your host to the container
     COPY ./scripts/ /docker-entrypoint-initdb.d/
     ```

     Ebben a Dockerfile-ban beállítottad a MySQL adatbázis nevének környezeti változóját. A `COPY` utasítással egyedi konfigurációs fájlokat vagy szkripteket is hozzáadhatsz a konténerhez. Ebben a példában a hosztod `./scripts/` könyvtárából származó fájlok a konténer `/docker-entrypoint-initdb.d/` könyvtárába másolódnak. Ebben a könyvtárban a `.sh`, `.sql` és `.sql.gz` szkriptek végrehajtódnak, amikor a konténert először indítják el. A Dockerfile-okról további részletekért lásd a Dockerfile referenciát.

   - Hozz létre egy szkriptfájlt egy tábla inicializálásához az adatbázisban. A Dockerfile-t tartalmazó könyvtárban hozz létre egy `scripts` nevű alkönyvtárat, majd hozz létre egy `create_table.sql` nevű fájlt a következő tartalommal.

     ```sql
     CREATE TABLE IF NOT EXISTS mydb.myothertable (
       column_name VARCHAR(255)
     );

     INSERT INTO mydb.myothertable (column_name) VALUES ('other_value');
     ```

   Most a következő könyvtárszerkezettel kell rendelkezned.

   ```plaintext
   ├── your-project-directory/
   │ ├── scripts/
   │ │ └── create_table.sql
   │ └── Dockerfile
   ```

2. Építsd fel az image-edet.

   - Egy terminálban váltsd át a könyvtárat arra a könyvtárra, ahol a Dockerfile-od található.
   - Futtasd a következő parancsot az image építéséhez.

     ```bash
     docker build -t my-custom-mysql .
     ```

     Ebben a parancsban a `-t my-custom-mysql` az új image-edet `my-custom-mysql`-ként címkézi (nevezi el). A parancs végén lévő pont (`.`) az aktuális könyvtárat adja meg a build kontextusaként, ahol a Docker keresi a Dockerfile-t és a buildhez szükséges egyéb fájlokat.

3. Futtasd az image-edet úgy, ahogy a Helyi konténerizált adatbázis futtatása részben tetted. Ezúttal az image-ed nevét add meg a `mysql:latest` helyett. Továbbá már nem kell megadnod a `MYSQL_DATABASE` környezeti változót, mivel azt most a Dockerfile-od definiálja.

   ```bash
   docker run --name my-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d my-custom-mysql
   ```

4. Ellenőrizd, hogy a konténered fut a következő paranccsal.

   ```bash
   docker ps
   ```

   Valami ilyesmit kell látnod:

   ```plaintext
   CONTAINER ID   IMAGE              COMMAND                  CREATED        STATUS          PORTS                 NAMES
   f74dcfdb0e59   my-custom-mysql   "docker-entrypoint.s…"    2 hours ago    Up 51 minutes   3306/tcp, 33060/tcp   my-mysql
   ```

5. Ellenőrizd, hogy az inicializáló szkripted lefutott-e. Futtasd a következő parancsot egy terminálban a `myothertable` tábla tartalmának megjelenítéséhez.

   ```bash
   docker exec my-mysql mysql -u root -pmy-secret-pw -e "SELECT * FROM mydb.myothertable;"
   ```

   Valami ilyesmit kell látnod:

   ```plaintext
   column_name
   other_value
   ```

Bármely konténer, amelyet a `my-custom-mysql` image-eddel futtatsz, a tábla inicializálva lesz az első indításkor.

## Docker Compose használata egy adatbázis futtatásához

A Docker Compose egy eszköz többkonténeres Docker alkalmazások definiálásához és futtatásához. Egyetlen paranccsal konfigurálhatod az alkalmazásod összes szolgáltatását (mint adatbázisok, webalkalmazások stb.), és kezelheted őket. Ebben a példában létrehozol egy Compose fájlt, és használod egy MySQL adatbázis-konténer és egy phpMyAdmin konténer futtatásához.

A konténerek Docker Compose-zal való futtatásához:

1. Hozz létre egy Docker Compose fájlt.

   - Hozz létre egy `compose.yaml` nevű fájlt a projekted könyvtárában. Ez a fájl határozza meg a szolgáltatásokat, hálózatokat és köteteket.
   - Add hozzá a következő tartalmat a `compose.yaml` fájlhoz.

     ```yaml
     services:
       db:
         image: mysql:latest
         environment:
           MYSQL_ROOT_PASSWORD: my-secret-pw
           MYSQL_DATABASE: mydb
         ports:
           - 3307:3306
         volumes:
           - my-db-volume:/var/lib/mysql

       phpmyadmin:
         image: phpmyadmin/phpmyadmin:latest
         environment:
           PMA_HOST: db
           PMA_PORT: 3306
           MYSQL_ROOT_PASSWORD: my-secret-pw
         ports:
           - 8080:80
         depends_on:
           - db

     volumes:
       my-db-volume:
     ```

   Az adatbázis-szolgáltatásnál:
   - `db` a szolgáltatás neve.
   - `image: mysql:latest` meghatározza, hogy a szolgáltatás a Docker Hub legújabb MySQL image-ét használja.
   - `environment` listázza a MySQL által az adatbázis inicializálásához használt környezeti változókat, mint a root jelszó és az adatbázisnév.
   - `ports` a hoszt 3307-es portját a konténer 3306-os portjára képezi le, lehetővé téve, hogy a hosztgépedről csatlakozz az adatbázishoz.
   - `volumes` a `my-db-volume`-ot a konténeren belüli `/var/lib/mysql`-hoz csatolja az adatbázis adatainak megőrzéséhez.

   Az adatbázis-szolgáltatáson kívül van egy phpMyAdmin szolgáltatás is. Alapértelmezésben a Compose egyetlen hálózatot állít be az alkalmazásodhoz. Minden szolgáltatás minden konténere csatlakozik az alapértelmezett hálózathoz, és elérhető más konténerek számára azon a hálózaton, valamint felfedezhető a szolgáltatás nevével. Ezért a `PMA_HOST` környezeti változóban megadhatod a szolgáltatás nevét, a `db`-t, hogy csatlakozz az adatbázis-szolgáltatáshoz. A Compose-ról további részletekért lásd a Compose fájl referenciát.

2. Futtasd a Docker Compose-t.

   - Nyiss egy terminált, és váltsd át a könyvtárat arra a könyvtárra, ahol a `compose.yaml` fájlod található.
   - Futtasd a Docker Compose-t a következő paranccsal.

     ```bash
     docker compose up
     ```

   Most elérheted a phpMyAdmin-t a `http://localhost:8080` címen, és csatlakozhatsz az adatbázisodhoz a `root` felhasználónévvel és a `my-secret-pw` jelszóval.

   A konténerek leállításához nyomd meg a `ctrl+c`-t a terminálban.

Most a Docker Compose-zal elindíthatod az adatbázisodat és alkalmazásodat, csatolhatsz köteteket, konfigurálhatsz hálózatkezelést és még sok mást, mindezt egyetlen paranccsal.

## Összefoglaló

Ez az útmutató bemutatta a konténerizált adatbázisok használatának alapjait, különös tekintettel a MySQL-re, hogy növelje a rugalmasságot, az egyszerű beállítást és a konzisztenciát a fejlesztői környezeteidben. Az útmutatóban tárgyalt felhasználási esetek nemcsak egyszerűsítik a fejlesztési munkafolyamataidat, hanem felkészítenek a haladóbb adatbázis-kezelési és telepítési forgatókönyvekre is, biztosítva, hogy az adatvezérelt alkalmazásaid robusztusak és skálázhatók maradjanak.
