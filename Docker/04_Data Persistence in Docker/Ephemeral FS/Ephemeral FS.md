# Konténeradatok perzisztálása

## Magyarázat

Amikor egy konténer elindul, a kép (image) által biztosított fájlokat és konfigurációt használja. Minden konténer képes fájlokat létrehozni, módosítani és törölni anélkül, hogy ez hatással lenne bármely más konténerre. Amikor a konténert törlik, ezek a fájlváltozások is törlődnek.

Bár a konténereknek ez a rövid életű természete nagyszerű, kihívást jelent, amikor az adatokat meg szeretnéd őrizni. Például, ha újraindítasz egy adatbázis-konténert, valószínűleg nem egy üres adatbázissal szeretnéd kezdeni. Szóval, hogyan őrzöd meg a fájlokat?

## Konténerkötetek (Container volumes)

A kötetek (volumes) egy tárolási mechanizmus, amely lehetővé teszi az adatok megőrzését egy egyedi konténer életciklusán túl. Gondolj rá úgy, mint egy parancsikonra vagy symlinkre a konténeren belülről a konténeren kívülre.

Példaként képzeld el, hogy létrehozol egy `log-data` nevű kötetet.

```bash
docker volume create log-data
```

Amikor a következő paranccsal elindítasz egy konténert, a kötet a `/logs` alá csatolódik (mountolódik) a konténerbe:

```bash
docker run -d -p 80:80 -v log-data:/logs docker/welcome-to-docker
```

Ha a `log-data` kötet nem létezik, a Docker automatikusan létrehozza a számodra.

Amikor a konténer fut, minden fájl, amelyet a `/logs` mappába ír, ebben a kötetben tárolódik, a konténeren kívül. Ha törlöd a konténert, és ugyanazt a kötetet használva új konténert indítasz, a fájlok még mindig ott lesznek.

> **Fájlok megosztása kötetek használatával**
>
> Ugyanazt a kötetet több konténerhez is csatolhatod, hogy fájlokat ossz meg a konténerek között. Ez hasznos lehet olyan forgatókönyvekben, mint a naplóaggregáció, adatpipeline-ok vagy más eseményvezérelt alkalmazások.

## Kötetek kezelése

A köteteknek saját életciklusuk van a konténereken túl, és meglehetősen nagyra nőhetnek, a használt adatok típusától és alkalmazásoktól függően. A következő parancsok hasznosak lesznek a kötetek kezeléséhez:

- `docker volume ls` – az összes kötet listázása
- `docker volume rm <volume-name-or-id>` – egy kötet eltávolítása (csak akkor működik, ha a kötet nincs egyetlen konténerhez sem csatolva)
- `docker volume prune` – az összes nem használt (nem csatolt) kötet eltávolítása

## Próbáld ki

Ebben az útmutatóban gyakorolni fogod a kötetek létrehozását és használatát egy Postgres konténer által létrehozott adatok megőrzésére. Amikor az adatbázis fut, fájlokat tárol a `/var/lib/postgresql` könyvtárba. A kötet ide csatolásával képes leszel a konténert többször újraindítani az adatok megtartása mellett.

### Kötetek használata

1. Töltsd le és telepítsd a Docker Desktopot.

2. Indíts el egy konténert a Postgres image használatával a következő paranccsal:

   ```bash
   docker run --name=db -e POSTGRES_PASSWORD=secret -d -v postgres_data:/var/lib/postgresql postgres:18
   ```

   Ez elindítja az adatbázist a háttérben, jelszóval konfigurálja, és egy kötetet csatol ahhoz a könyvtárhoz, ahol a PostgreSQL megőrzi az adatbázisfájlokat.

3. Csatlakozz az adatbázishoz a következő paranccsal:

   ```bash
   docker exec -ti db psql -U postgres
   ```

4. A PostgreSQL parancssorban futtasd a következőt egy adatbázistábla létrehozásához és két rekord beszúrásához:

   ```sql
   CREATE TABLE tasks (
       id SERIAL PRIMARY KEY,
       description VARCHAR(100)
   );
   INSERT INTO tasks (description) VALUES ('Finish work'), ('Have fun');
   ```

5. Ellenőrizd, hogy az adatok benne vannak az adatbázisban a következő futtatásával a PostgreSQL parancssorban:

   ```sql
   SELECT * FROM tasks;
   ```

   A kimenetnek valahogy így kell kinéznie:

   ```plaintext
    id | description
   ----+-------------
     1 | Finish work
     2 | Have fun
   (2 rows)
   ```

6. Lépj ki a PostgreSQL shellből a következő parancs futtatásával:

   ```plaintext
   \q
   ```

7. Állítsd le és távolítsd el az adatbázis-konténert. Ne feledd, hogy bár a konténer törölve lett, az adatok a `postgres_data` kötetben maradnak.

   ```bash
   docker stop db
   docker rm db
   ```

8. Indíts el egy új konténert a következő parancs futtatásával, csatolva ugyanazt a kötetet a megőrzött adatokkal:

   ```bash
   docker run --name=new-db -d -v postgres_data:/var/lib/postgresql postgres:18
   ```

   Talán észrevetted, hogy a `POSTGRES_PASSWORD` környezeti változó elhagyásra került. Azért, mert ez a változó csak egy új adatbázis bootstrapelésekor használatos.

9. Ellenőrizd, hogy az adatbázisban még mindig megvannak a rekordok a következő parancs futtatásával:

   ```bash
   docker exec -ti new-db psql -U postgres -c "SELECT * FROM tasks"
   ```

## Kötet tartalmának megtekintése

A Docker Desktop Dashboard lehetővé teszi bármely kötet tartalmának megtekintését, valamint a kötetek exportálását, importálását, kiürítését, törlését és klónozását.

1. Nyisd meg a Docker Desktop Dashboardot, és navigálj a Volumes nézetre. Ebben a nézetben látnod kell a `postgres_data` kötetet.

2. Válaszd ki a `postgres_data` kötet nevét.

3. A **Stored Data** fül a kötet tartalmát mutatja, és lehetővé teszi a fájlok közötti navigálást. A **Container in-use** fül a kötetet használó konténer nevét, az image nevét, a konténer által használt portszámot és a célt (target) jeleníti meg. A cél egy elérési út a konténeren belül, amely hozzáférést biztosít a kötet fájljaihoz. Az **Exports** fül lehetővé teszi a kötet exportálását. Egy fájlra duplán kattintva megtekintheted a tartalmát és módosíthatod.

4. Bármely fájlra jobb gombbal kattintva mentheted vagy törölheted.

## Kötetek eltávolítása

A kötet eltávolítása előtt nem szabad egyetlen konténerhez sem csatolva lennie. Ha még nem távolítottad el az előző konténert, tedd meg a következő paranccsal (a `-f` először leállítja a konténert, majd eltávolítja):

```bash
docker rm -f new-db
```

Néhány módszer a kötetek eltávolítására:

- Válaszd a **Delete Volume** opciót egy kötetnél a Docker Desktop Dashboardon.
- Használd a `docker volume rm` parancsot:

  ```bash
  docker volume rm postgres_data
  ```

- Használd a `docker volume prune` parancsot az összes nem használt kötet eltávolításához:

  ```bash
  docker volume prune
  ```
