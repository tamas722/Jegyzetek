# Konténerek izolálása felhasználói névtérrel (user namespace)

A Linux névterek (namespaces) izolációt biztosítanak a futó folyamatok számára, korlátozva hozzáférésüket a rendszererőforrásokhoz anélkül, hogy a futó folyamat tudatában lenne a korlátozásoknak. A Linux névterekről további információért lásd: Linux namespaces.

A legjobb módja annak, hogy megakadályozzuk a konténeren belüli jogosultságkiterjesztési (privilege-escalation) támadásokat, ha a konténer alkalmazásait nem privilegizált felhasználóként futtatjuk. Azoknál a konténereknél, amelyek folyamatainak root felhasználóként kell futniuk a konténeren belül, ezt a felhasználót átképezhetjük egy kevésbé privilegizált felhasználóra a Docker hoszton. Az átképezett felhasználóhoz egy UID-tartományt rendelünk, amely a névtéren belül normál UID-ként működik 0-tól 65536-ig, de a hosztgépen magán nincs jogosultsága.

> **Megjegyzés**
> A `userns-remap` használatával a Docker démon továbbra is rootként fut. Ahhoz, hogy mind a démont, mind a konténereket root jogosultság nélkül futtassuk, lásd helyette a Rootless módot.

## Az átképezésről és a alárendelt felhasználói és csoportazonosítókról

Magát az átképezést két fájl kezeli: a `/etc/subuid` és a `/etc/subgid`. Mindkét fájl ugyanúgy működik, de az egyik a felhasználói ID-tartományra, a másik a csoport ID-tartományra vonatkozik. Tekintsük a következő bejegyzést a `/etc/subuid`-ben:

```plaintext
testuser:231072:65536
```

Ez azt jelenti, hogy a `testuser` hozzá van rendelve a 231072-es alárendelt felhasználói ID-tartományhoz, és a következő 65536 egész számhoz sorban. A 231072-es UID a névtéren belül (ebben az esetben a konténeren belül) 0-s UID-ként (root) van leképezve. A 231073-as UID 1-ként van leképezve, és így tovább. Ha egy folyamat megpróbálja kiterjeszteni a jogosultságát a névtéren kívülre, a folyamat a hoszton egy nem privilegizált, magas számú UID-ként fut, amely még egy valódi felhasználóhoz sem képeződik le. Ez azt jelenti, hogy a folyamatnak egyáltalán nincsenek jogosultságai a hosztrendszeren.

> **Megjegyzés**
> Lehetséges több alárendelt tartományt hozzárendelni egy adott felhasználóhoz vagy csoporthoz úgy, hogy több nem átfedő leképezést adunk hozzá ugyanahhoz a felhasználóhoz vagy csoporthoz a `/etc/subuid` vagy `/etc/subgid` fájlban. Ebben az esetben a Docker csak az első öt leképezést használja, összhangban a kernel azon korlátozásával, hogy csak öt bejegyzés lehet a `/proc/self/uid_map` és `/proc/self/gid_map` fájlokban.

Amikor a Docker-t a `userns-remap` funkció használatára konfigurálod, opcionálisan megadhatsz egy meglévő felhasználót és/vagy csoportot, vagy megadhatod a `default` értéket. Ha a `default`-ot adod meg, létrejön egy `dockremap` nevű felhasználó és csoport, amelyet erre a célra használnak.

> ⚠️ **Figyelmeztetés**
> Néhány disztribúció nem adja hozzá automatikusan az új csoportot a `/etc/subuid` és `/etc/subgid` fájlokhoz. Ha ez a helyzet, manuálisan kell szerkesztened ezeket a fájlokat, és nem átfedő tartományokat kell hozzárendelned. Ez a lépés az Előfeltételekben van lefedve.

Nagyon fontos, hogy a tartományok ne fedjék át egymást, hogy egy folyamat ne nyerhessen hozzáférést egy másik névtérben. A legtöbb Linux disztribúción a rendszereszközök kezelik a tartományokat, amikor felhasználókat adsz hozzá vagy távolítasz el.

Ez az átképezés átlátszó a konténer számára, de némi konfigurációs bonyolultságot vezet be olyan helyzetekben, amikor a konténernek hozzá kell férnie a Docker hoszton lévő erőforrásokhoz, például bind mountokhoz a fájlrendszer olyan területeire, ahová a rendszerfelhasználó nem tud írni. Biztonsági szempontból a legjobb elkerülni ezeket a helyzeteket.

## Előfeltételek

- Az alárendelt UID- és GID-tartományokat egy meglévő felhasználóhoz kell társítani, még akkor is, ha a társítás implementációs részlet. A felhasználó birtokolja a névterekkel ellátott tárolási könyvtárakat a `/var/lib/docker/` alatt. Ha nem szeretnél meglévő felhasználót használni, a Docker létrehozhat egyet neked és használhatja azt. Ha meglévő felhasználónevet vagy felhasználói ID-t szeretnél használni, annak már léteznie kell. Ez általában azt jelenti, hogy a vonatkozó bejegyzéseknek benne kell lenniük a `/etc/passwd` és `/etc/group` fájlokban, de ha más hitelesítési hátteret használsz, ez a követelmény másképp fordulhat le.

  Ennek ellenőrzéséhez használd az `id` parancsot:

  ```bash
  id testuser
  ```

  ```plaintext
  uid=1001(testuser) gid=1001(testuser) groups=1001(testuser)
  ```

- A névtér-átképezés kezelésének módja a hoszton két fájl használatával történik, a `/etc/subuid` és a `/etc/subgid`. Ezeket a fájlokat általában automatikusan kezelik, amikor felhasználókat vagy csoportokat adsz hozzá vagy távolítasz el, de néhány disztribúción manuálisan kell kezelned ezeket a fájlokat.

  Minden fájl három mezőt tartalmaz: a felhasználó felhasználónevét vagy ID-jét, amelyet egy kezdő UID vagy GID követ (amely a névtéren belül UID vagy GID 0-ként van kezelve), és a felhasználó számára elérhető UID-k vagy GID-k maximális számát. Például a következő bejegyzés esetén:

  ```plaintext
  testuser:231072:65536
  ```

  Ez azt jelenti, hogy a `testuser` által indított névtérrel ellátott folyamatok a hoszt 231072-es UID-jától (amely a névtéren belül UID 0-nak tűnik) a 296607-ig (231072 + 65536 - 1) vannak birtokolva. Ezek a tartományok nem fedhetik át egymást, hogy biztosítsák, hogy a névtérrel ellátott folyamatok ne férhessenek hozzá egymás névtereihez.

- A felhasználó hozzáadása után ellenőrizd a `/etc/subuid` és `/etc/subgid` fájlokat, hogy a felhasználódnak van-e bejegyzése mindegyikben. Ha nincs, hozzá kell adnod, ügyelve az átfedés elkerülésére.

- Ha a Docker által automatikusan létrehozott `dockremap` felhasználót szeretnéd használni, ellenőrizd a `dockremap` bejegyzést ezekben a fájlokban a Docker konfigurálása és újraindítása után.

- Ha vannak olyan helyek a Docker hoszton, ahol a nem privilegizált felhasználónak írnia kell, ennek megfelelően módosítsd ezen helyek jogosultságait. Ez akkor is igaz, ha a Docker által automatikusan létrehozott `dockremap` felhasználót szeretnéd használni, de a jogosultságokat csak a Docker konfigurálása és újraindítása után módosíthatod.

- A `userns-remap` engedélyezése hatékonyan maszkolja a meglévő kép- és konténerrétegeket, valamint más Docker-objektumokat a `/var/lib/docker/`-on belül. Ez azért van, mert a Dockernek módosítania kell ezen erőforrások tulajdonjogát, és valójában egy alkönyvtárban tárolja őket a `/var/lib/docker/`-on belül. A legjobb ezt a funkciót egy új Docker-telepítésen engedélyezni, nem egy meglévőn.

- Ugyanígy, ha letiltod a `userns-remap`-et, nem férhetsz hozzá a engedélyezése közben létrehozott erőforrásokhoz.

- Ellenőrizd a felhasználói névterek korlátozásait, hogy megbizonyosodj arról, hogy a felhasználási eseted lehetséges.

## A `userns-remap` engedélyezése a démonon

Elindíthatod a `dockerd`-t a `--userns-remap` flaggel, vagy követheted ezt az eljárást a démon `daemon.json` konfigurációs fájllal való konfigurálásához. A `daemon.json` módszer ajánlott. Ha a flaget használod, használd a következő parancsot mintaként:

```bash
dockerd --userns-remap="testuser:testuser"
```

1. Szerkeszd a `/etc/docker/daemon.json` fájlt. Feltételezve, hogy a fájl korábban üres volt, a következő bejegyzés engedélyezi a `userns-remap`-et a `testuser` nevű felhasználó és csoport használatával. A felhasználót és a csoportot ID vagy név alapján is megcímezheted. Csak akkor kell megadnod a csoport nevét vagy ID-jét, ha az eltér a felhasználó nevétől vagy ID-jétől. Ha mind a felhasználó, mind a csoport nevét vagy ID-jét megadod, kettősponttal (`:`) válaszd el őket. A következő formátumok mind működnek az értékhez, feltételezve, hogy a `testuser` UID-ja és GID-je 1001:

   - `testuser`
   - `testuser:testuser`
   - `1001`
   - `1001:1001`
   - `testuser:1001`
   - `1001:testuser`

   ```json
   {
     "userns-remap": "testuser"
   }
   ```

   > **Megjegyzés**
   > A `dockremap` felhasználó használatához és a Docker általi létrehozásához állítsd az értéket `default`-ra a `testuser` helyett.

2. Mentsd el a fájlt, és indítsd újra a Docker-t.

3. Ha a `dockremap` felhasználót használod, ellenőrizd, hogy a Docker létrehozta-e az `id` paranccsal.

   ```bash
   id dockremap
   ```

   ```plaintext
   uid=112(dockremap) gid=116(dockremap) groups=116(dockremap)
   ```

4. Ellenőrizd, hogy a bejegyzés hozzá lett-e adva a `/etc/subuid` és `/etc/subgid` fájlokhoz:

   ```bash
   grep dockremap /etc/subuid
   ```

   ```plaintext
   dockremap:231072:65536
   ```

   ```bash
   grep dockremap /etc/subgid
   ```

   ```plaintext
   dockremap:231072:65536
   ```

   Ha ezek a bejegyzések nincsenek jelen, szerkeszd a fájlokat root felhasználóként, és rendelj hozzá egy kezdő UID-t és GID-t, amely a legmagasabb hozzárendelt érték plusz az eltolás (ebben az esetben 65536). Ügyelj arra, hogy ne engedj átfedést a tartományokban.

5. Ellenőrizd, hogy a korábbi képek nem elérhetők a `docker image ls` paranccsal. A kimenetnek üresnek kell lennie.

6. Indíts el egy konténert a `hello-world` képből.

   ```bash
   docker run hello-world
   ```

7. Ellenőrizd, hogy létezik egy névtérrel ellátott könyvtár a `/var/lib/docker/`-on belül, amely a névtérrel ellátott felhasználó UID-jával és GID-jével van elnevezve, az adott UID és GID birtokolja, és nem csoport- vagy világ-olvasható. Néhány alkönyvtárat továbbra is a root birtokol, és különböző jogosultságokkal rendelkezik.

   ```bash
   sudo ls -ld /var/lib/docker/231072.231072/
   ```

   ```plaintext
   drwx------ 11 231072 231072 11 Jun 21 21:19 /var/lib/docker/231072.231072/
   ```

   ```bash
   sudo ls -l /var/lib/docker/231072.231072/
   ```

   ```plaintext
   total 14
   drwx------ 5 231072 231072 5 Jun 21 21:19 aufs
   drwx------ 3 231072 231072 3 Jun 21 21:21 containers
   drwx------ 3 root   root   3 Jun 21 21:19 image
   drwxr-x--- 3 root   root   3 Jun 21 21:19 network
   drwx------ 4 root   root   4 Jun 21 21:19 plugins
   drwx------ 2 root   root   2 Jun 21 21:19 swarm
   drwx------ 2 231072 231072 2 Jun 21 21:21 tmp
   drwx------ 2 root   root   2 Jun 21 21:19 trust
   drwx------ 2 231072 231072 3 Jun 21 21:19 volumes
   ```

   A könyvtárlistád némi eltérést mutathat, különösen, ha más konténer-tároló-illesztőprogramot használsz, mint az `aufs`.

   Az átképezett felhasználó által birtokolt könyvtárakat használják a `/var/lib/docker/` alatti ugyanazon könyvtárak helyett, és a nem használt verziók (mint például a példában szereplő `/var/lib/docker/tmp/`) eltávolíthatók. A Docker nem használja őket, amíg a `userns-remap` engedélyezve van.

## Névtér-átképezés letiltása egy konténerhez

Ha engedélyezed a felhasználói névtereket a démonon, minden konténer alapértelmezésben engedélyezett felhasználói névterekkel indul. Bizonyos helyzetekben, például privilegizált konténereknél, le kell tiltanod a felhasználói névtereket egy adott konténerhez. Lásd a felhasználói névtér ismert korlátozásait ezen korlátozások némelyikéhez.

A felhasználói névterek letiltásához egy adott konténerhez add hozzá a `--userns=host` flaget a `docker container create`, `docker container run` vagy `docker container exec` parancshoz.

Van egy mellékhatás ennek a flagnek a használatakor: a felhasználói átképezés nem lesz engedélyezve ahhoz a konténerhez, de mivel a csak olvasható (kép) rétegek meg vannak osztva a konténerek között, a konténer fájlrendszerének tulajdonjoga továbbra is át lesz képezve.

Ez azt jelenti, hogy a teljes konténer fájlrendszer a `--userns-remap` démon konfigurációban megadott felhasználóhoz fog tartozni (a fenti példában 231072). Ez váratlan viselkedéshez vezethet a konténeren belüli programoknál. Például a `sudo` (amely ellenőrzi, hogy a binárisai a 0-s felhasználóhoz tartoznak-e) vagy a setuid flaggel rendelkező binárisok.

## A felhasználói névtér ismert korlátozásai

A következő szabványos Docker-funkciók összeférhetetlenek a felhasználói névterekkel engedélyezett Docker démon futtatásával:

- PID vagy NET névterek megosztása a hoszttal (`--pid=host` vagy `--network=host`).
- Külső (kötet- vagy tároló-) illesztőprogramok, amelyek nem ismerik vagy nem képesek a démon felhasználói leképezéseit használni.
- A `--privileged` mód flag használata a `docker run`-on anélkül, hogy a `--userns=host`-ot is megadnád.

A felhasználói névterek egy fejlett funkció, és más képességekkel való koordinációt igényelnek. Például, ha kötetek vannak csatolva a hosztról, a fájltulajdonjogot előre el kell rendezni, ha olvasási vagy írási hozzáféréssel kell rendelkezned a kötet tartalmához.

Míg a felhasználói névtérrel ellátott konténerfolyamaton belüli root felhasználó a szuperfelhasználó sok elvárt jogosultságával rendelkezik a konténeren belül, a Linux kernel belső tudás alapján korlátozásokat szab, hogy ez egy felhasználói névtérrel ellátott folyamat. Egy figyelemre méltó korlátozás az `mknod` parancs használatának képtelensége. Az eszközlétrehozás engedélye megtagadva a konténeren belül, amikor a root felhasználó futtatja.

## Forrás
-[https://docs.docker.com/engine/security/userns-remap/]
-[https://man7.org/linux/man-pages/man7/namespaces.7.html]
