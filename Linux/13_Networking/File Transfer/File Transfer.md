# A Linux `ftp` parancs használata fájlátvitelhez

**Szerző:** Dejan Panovski
**Frissítve:** 2026. március 6.
**7 perc olvasás**

## Ezen az oldalon

- Gyors referencia
- Mielőtt elkezdenéd
- FTP-kapcsolat létrehozása
- Átviteli módok: Bináris vs ASCII
- Fájlok letöltése
  - Egyetlen fájl letöltése
  - Több fájl letöltése
- Fájlok feltöltése
  - Egyetlen fájl feltöltése
  - Több fájl feltöltése
- Passzív mód
- A kapcsolat lezárása
- Gyakori FTP-parancsok
- Hibaelhárítás
- GYIK
- Következtetés

Az FTP (File Transfer Protocol) egy szabványos hálózati protokoll, amelyet fájlok távoli szerverre és onnan történő átvitelére használnak. Bár az FTP-t nagyrészt felváltották a biztonságos alternatívák, a parancssori `ftp` kliens továbbra is hasznos a régi szerverek eléréséhez és automatizált átvitelekhez ellenőrzött környezetekben.

Ez az útmutató elmagyarázza, hogyan kell használni a Linux `ftp` parancsát egy távoli szerverhez való csatlakozáshoz, fájlok letöltéséhez és feltöltéséhez.

## Gyors referencia

Nyomtatható gyors referenciáért lásd az FTP cheatsheet-et.

| Parancs | Leírás |
| --- | --- |
| `ftp hostname` | Csatlakozás FTP-szerverhez |
| `get file` | Egyetlen fájl letöltése |
| `put file` | Egyetlen fájl feltöltése |
| `mget *.txt` | Több fájl letöltése |
| `mput *.txt` | Több fájl feltöltése |
| `lcd /path` | Helyi könyvtár megváltoztatása |
| `cd /path` | Távoli könyvtár megváltoztatása |
| `ls` | Távoli könyvtár tartalmának listázása |
| `binary` | Váltás bináris átviteli módra |
| `ascii` | Váltás ASCII átviteli módra |
| `passive` | Passzív mód be/ki |
| `prompt` | Fájlonkénti megerősítés be/ki az `mget`/`mput`-hoz |
| `bye` vagy `quit` | A kapcsolat lezárása |

## Mielőtt elkezdenéd

Az FTP-forgalom, beleértve a hitelesítő adatokat is, nyílt szövegként továbbítódik, és nincs titkosítva. Biztonságos fájlátvitelhez használj SCP-t vagy SFTP-t.

A fájlok átviteléhez legalább olvasási jogosultsággal kell rendelkezned a forrásfájlon, és írási jogosultsággal a célrendszeren.

Nagy fájlok átvitelekor ajánlott az `ftp` parancsot egy `screen` vagy `tmux` munkamenetben futtatni.

Az a könyvtár, ahonnan az `ftp` parancsot futtatod, a helyi munkakönyvtár.

## FTP-kapcsolat létrehozása

Távoli FTP-szerverhez való csatlakozáshoz futtasd az `ftp` parancsot a szerver IP-címével vagy domainnevével:

```bash
ftp 192.168.42.77
```

Ha a kapcsolat létrejött, egy üdvözlő üzenet jelenik meg, és a rendszer kéri az FTP-felhasználónevedet:

```plaintext
220---------- Welcome to Pure-FTPd [privsep] [TLS] ----------
220-You are user number 1 of 50 allowed.
220-Local time is now 21:35. Server port: 21.
220-This is a private system - No anonymous login
220-IPv6 connections are also welcome on this server.
220 You will be disconnected after 15 minutes of inactivity.
Name (192.168.42.77:localuser): linuxize
```

A felhasználónév megadása után írd be a jelszavadat a promptnál:

```plaintext
Password:
```

Ha a hitelesítő adatok helyesek, a szerver egy megerősítő üzenetet és az `ftp>` promptot jeleníti meg:

```plaintext
230 OK. Current restricted directory is /
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

> **Info:** Ha az FTP-szerver elfogad névtelen kapcsolatokat, használd az `anonymous` felhasználónevet és az e-mail címedet jelszóként.

## Átviteli módok: Bináris vs ASCII

Az FTP két átviteli módot támogat:

- **Bináris mód:** A fájlokat bájt-ról bájt-ra, módosítás nélkül továbbítja. Ezt használd képekhez, archívumokhoz, futtatható fájlokhoz és bármilyen nem szöveges fájlhoz.
- **ASCII mód:** A sorvégeket alakítja át a rendszerek között (pl. `\r\n` Windows-on `\n`-re Linuxon). Ezt csak egyszerű szöveges fájlokhoz használd.

A legtöbb modern FTP-kliens alapértelmezésben bináris módra áll. A módok közötti váltáshoz az `ftp>` promptnál:

```bash
binary
```

```bash
ascii
```

Mindig bináris módot használj nem szöveges fájlok átviteléhez. Egy bináris fájl ASCII módban történő átvitele korrumpálja azt.

## Fájlok letöltése

Miután bejelentkeztél, a jelenlegi munkakönyvtárad a távoli felhasználó home könyvtára. A fájlok abba a helyi könyvtárba töltődnek le, ahonnan az `ftp` parancsot indítottad.

Ha más helyi könyvtárba szeretnél fájlokat letölteni, használd az `lcd` parancsot:

```bash
lcd ~/ftp_downloads
```

### Egyetlen fájl letöltése

Használd a `get` parancsot egy fájl letöltéséhez a távoli szerverről:

```bash
get backup.zip
```

**Kimenet:**

```plaintext
200 PORT command successful
150-Connecting to port 60609
150 6516.9 kbytes to download
226-File successfully transferred
226 2.356 seconds (measured here), 2.70 Mbytes per second
6673256 bytes received in 2.55 seconds (2.49 Mbytes/s)
```

### Több fájl letöltése

Használd az `mget` parancsot több fájl letöltéséhez. Megadhatsz egyedi fájlneveket, vagy használhatsz helyettesítő karaktereket:

```bash
mget backup1.zip backup2.zip
```

Alapértelmezésben az `mget` minden fájlnál megerősítést kér:

```plaintext
mget backup1.zip? y
200 PORT command successful
150 Connecting to port 52231
226-File successfully transferred
226 0.000 seconds (measured here), 31.51 Kbytes per second
14 bytes received in 0.00058 seconds (23.6 kbytes/s)
mget backup2.zip? y
200 PORT command successful
150-Connecting to port 59179
150 7.2 kbytes to download
226-File successfully transferred
226 0.000 seconds (measured here), 16.68 Mbytes per second
7415 bytes received in 0.011 seconds (661 kbytes/s)
```

A fájlonkénti megerősítés kikapcsolásához futtasd a `prompt` parancsot az `mget` használata előtt:

```bash
prompt
```

**Kimenet:**

```plaintext
Interactive mode off.
```

A megerősítések újbóli engedélyezéséhez futtasd újra a `prompt` parancsot.

## Fájlok feltöltése

### Egyetlen fájl feltöltése

Használd a `put` parancsot egy fájl feltöltéséhez a távoli szerverre:

```bash
put image.jpg
```

**Kimenet:**

```plaintext
200 PORT command successful
150 Connecting to port 34583
226-File successfully transferred
226 0.849 seconds (measured here), 111.48 Kbytes per second
96936 bytes sent in 0.421 seconds (225 kbytes/s)
```

Ha egy fájlt más helyi könyvtárból szeretnél feltölteni, használd annak abszolút elérési útját.

### Több fájl feltöltése

Használd az `mput` parancsot több fájl feltöltéséhez:

```bash
mput image1.jpg image2.jpg
```

Az `mget`-hez hasonlóan az `mput` parancs is megerősítést kér minden fájlnál. A `prompt` paranccsal kapcsolhatod ki ezt a viselkedést.

## Passzív mód

Alapértelmezésben az FTP aktív módot használ, ahol a szerver kezdeményezi az adatkapcsolatot vissza a klienshez. Ez gyakran meghiúsul, ha a kliens NAT-router vagy tűzfal mögött van, mert a bejövő kapcsolat blokkolva van.

A passzív mód ezt úgy oldja meg, hogy a kliens kezdeményezi mind a vezérlő, mind az adatkapcsolatot. A passzív mód be/ki kapcsolásához:

```bash
passive
```

**Kimenet:**

```plaintext
Passive mode on.
```

Ha „connection refused” vagy időtúllépési hibákat tapasztalsz az átvitelek során, a passzív módra váltás általában megoldja a problémát.

## A kapcsolat lezárása

Amikor végeztél, zárd le az FTP-kapcsolatot a `bye` vagy `quit` paranccsal:

```bash
quit
```

**Kimenet:**

```plaintext
221-Goodbye. You uploaded 0 and downloaded 6544 kbytes.
221 Logout.
```

## Gyakori FTP-parancsok

| Parancs | Leírás |
| --- | --- |
| `help` vagy `?` | Az összes elérhető FTP-parancs listázása |
| `cd directory` | Távoli könyvtár megváltoztatása |
| `lcd directory` | Helyi könyvtár megváltoztatása |
| `ls` | Fájlok listázása a jelenlegi távoli könyvtárban |
| `pwd` | A jelenlegi távoli munkakönyvtár kiírása |
| `mkdir directory` | Könyvtár létrehozása a távoli szerveren |
| `delete file` | Fájl törlése a távoli szerveren |
| `rmdir directory` | Könyvtár törlése a távoli szerveren |
| `rename old new` | Fájl átnevezése a távoli szerveren |
| `status` | A jelenlegi kapcsolat és átviteli beállítások megjelenítése |
| `hash` | Hash-jel megjelenítésének be/ki kapcsolása az átvitelek során |

## Hibaelhárítás

- **Connection refused vagy timed out:** Lehet, hogy az FTP-szerver nem fut, a portot tűzfal blokkolja, vagy az IP-cím helytelen. Ellenőrizd, hogy a szerver a 21-es porton figyel-e, és hogy a tűzfalad engedélyezi-e a kimenő kapcsolatokat felé.
- **`ftp: connect: No route to host`:** Hálózati szintű probléma akadályozza a kapcsolatot. Ellenőrizd a hálózati kapcsolatodat és a DNS-feloldást.
- **Az átvitel sikeres csatlakozás után lefagy vagy időtúllép:** Ez szinte mindig passzív mód probléma. A szerver megpróbál adatkapcsolatot nyitni vissza a kliensedhez, de egy NAT-router vagy tűzfal blokkolja. Futtasd a `passive` parancsot az `ftp>` promptnál a passzív módra váltáshoz.
- **A letöltött fájl korrupt vagy rossz méretű:** A fájl valószínűleg ASCII módban lett átvive bináris mód helyett. Futtasd a `binary` parancsot nem szöveges fájlok letöltése előtt.
- **Az `mget` vagy `mput` minden fájlnál megerősítést kér:** Futtasd a `prompt` parancsot az interaktív megerősítés kikapcsolásához. Ez különösen hasznos sok fájl egyszerre történő letöltésekor vagy feltöltésekor.

## GYIK

- **Biztonságos még az FTP használata?**
  Az FTP nyílt szövegként továbbítja az adatokat és a hitelesítő adatokat. Csak megbízható hálózatokon vagy nem érzékeny adatokhoz szabad használni. Biztonságos átvitelhez használj SFTP-t vagy SCP-t.

- **Mi a különbség az FTP és az SFTP között?**
  Az FTP egy titkosítatlan protokoll, amely a 21-es portot használja. Az SFTP SSH-n (22-es port) keresztül fut, és titkosítja az összes adatot és hitelesítő adatot. A hasonló nevek ellenére teljesen különböző protokollok.

- **Hogyan tudok fájlokat átvinni anélkül, hogy mindegyiknél megkérdeznének?**
  Futtasd a `prompt` parancsot az `ftp>` promptnál az interaktív mód kikapcsolásához. Ezután az `mget` és `mput` az összes egyező fájlt megerősítés nélkül átviszi.

- **Mi a különbség az aktív és a passzív FTP mód között?**
  Aktív módban a szerver csatlakozik vissza a klienshez az adatátvitelhez, ami gyakran meghiúsul tűzfalak vagy NAT mögött. Passzív módban a kliens kezdeményezi mindkét kapcsolatot, így kompatibilisebb a modern hálózati beállításokkal.

  ## Forrás
  -[https://linuxize.com/post/how-to-use-linux-ftp-command-to-transfer-files/]

  # Az `rsync` parancs Linuxon – Példákkal

**Szerző:** Dejan Panovski
**Frissítve:** 2026. február 23.
**7 perc olvasás**

## Ezen az oldalon

- Az Rsync telepítése
  - Rsync telepítése Ubuntu, Debian és származékaik rendszerére
  - Rsync telepítése Fedora, RHEL és származékaik rendszerére
- Az Rsync parancs szintaxisa
- Rsync opciók
- Alapvető használat
  - Könyvtár tükrözése `--delete`-tel
- Próba futtatás (Dry Run)
- Távoli szinkronizálás
- Fájlok és könyvtárak kizárása
- Gyors referencia
- Hibaelhárítás
- GYIK
- Következtetés

Az `rsync` egy gyors, sokoldalú parancssori segédprogram fájlok és könyvtárak szinkronizálására két hely között távoli shellen vagy távoli Rsync démonon keresztül. Gyors növekményes fájlátvitelt biztosít azáltal, hogy csak a forrás és a cél közötti különbségeket továbbítja.

Az Rsync-et használhatod adatok tükrözésére, növekményes biztonsági mentésekhez `--link-dest`-tel, fájlok másolására rendszerek között, valamint az `scp`, `sftp` és `cp` parancsok kiváló alternatívájaként.

Ez az útmutató gyakorlati példákon és a leggyakoribb Rsync opciók részletes magyarázatán keresztül mutatja be az Rsync használatát.

## Az Rsync telepítése

Az Rsync a legtöbb Linux disztribúción és macOS-en előre telepítve van. Ha hiányzik, telepítsd a disztribúciód csomagkezelőjével.

### Rsync telepítése Ubuntu, Debian és származékaik rendszerére

```bash
sudo apt install rsync
```

### Rsync telepítése Fedora, RHEL és származékaik rendszerére

```bash
sudo dnf install rsync
```

## Az Rsync parancs szintaxisa

Az Rsync parancsok a következő szerkezetet követik:

```bash
# Helyi -> Helyi:
rsync [OPTION]... [SRC]... DEST

# Helyi -> Távoli:
rsync [OPTION]... [SRC]... [USER@]HOST:DEST

# Távoli -> Helyi:
rsync [OPTION]... [USER@]HOST:SRC... [DEST]
```

- **OPTION:** Az Rsync opciók (részletesen lentebb).
- **SRC:** Forrás elérési út.
- **DEST:** Cél elérési út.
- **USER:** Távoli felhasználónév.
- **HOST:** Távoli hosztnév vagy IP-cím.

## Rsync opciók

| Opció | Leírás |
| --- | --- |
| `-a`, `--archive` | Archív mód; egyenértékű a `-rlptgoD`-vel. Rekurzívan szinkronizálja a könyvtárakat, és megőrzi a szimbolikus linkeket, módosítási időket, csoportokat, tulajdonjogot és jogosultságokat. |
| `-z`, `--compress` | Tömöríti az adatokat az átvitel során. Csak lassú kapcsolatokon használd; gyors hálózatokon szükségtelen többletterhet ad. |
| `-P` | Egyenértékű a `--partial --progress`-szel. Folyamatjelzőt mutat, és megtartja a részben átvitt fájlokat. Hasznos nagy fájlokhoz lassú vagy instabil kapcsolatokon. |
| `--delete` | Törli a célhelyről azokat a fájlokat, amelyek már nem léteznek a forrásnál. Tükrözéshez hasznos. |
| `--dry-run` | Szimulálja az átvitelt anélkül, hogy bármilyen változtatást végezne. Ezzel előnézheted, mit tenne az rsync, mielőtt valóban futtatnád. |
| `-v`, `--verbose` | Részletes információkat ír ki minden átvitt fájlról. |
| `-q`, `--quiet` | Elnyomja a nem hibaüzeneteket. |
| `-e` | Megadja a használandó távoli shellt. Alapértelmezésben SSH. |

## Alapvető használat

Az Rsync legegyszerűbb felhasználási esete egyetlen fájl helyi másolása. Íme egy példa:

```bash
rsync -a /opt/filename.zip /tmp/
```

A parancsot futtató felhasználónak olvasási jogosultsággal kell rendelkeznie a forráshelyen, és írási jogosultsággal a célhelyen.

Ha elhagyod a fájlnevet a cél elérési útból, a fájl az aktuális névvel másolódik. Ha más néven szeretnéd menteni a fájlt, add meg az új nevet a cél elérési útban:

```bash
rsync -a /opt/filename.zip /tmp/newfilename.zip
```

Az Rsync igazi ereje a könyvtárak szinkronizálásakor mutatkozik meg. Az alábbi példa bemutatja, hogyan készíts helyi biztonsági mentést weboldal fájlokról:

```bash
rsync -a /var/www/domain.com/public_html/ /var/www/domain.com/public_html_backup/
```

Ha a célkönyvtár nem létezik, az Rsync létrehozza azt.

Az Rsync másképp kezeli a forrás elérési utakat, ha azok perjellel (`/`) végződnek:

- **Perjellel:** Az Rsync csak a könyvtár tartalmát másolja a célkönyvtárba.
- **Perjel nélkül:** Az Rsync magát a forráskönyvtárat másolja a célkönyvtárba.

### Könyvtár tükrözése `--delete`-tel

Használd a `--delete` opciót, hogy a cél a forrás pontos tükörképe legyen. A célhelyen lévő azon fájlok, amelyek már nem léteznek a forrásnál, törlődnek:

```bash
rsync -a --delete /var/www/domain.com/public_html/ /var/www/domain.com/public_html_backup/
```

> ⚠️ **Figyelmeztetés:** A `--delete` véglegesen eltávolítja a fájlokat a célhelyről. Mindig futtasd először `--dry-run`-nal, hogy ellenőrizd, mi fog törlődni.

## Próba futtatás (Dry Run)

Mielőtt bármilyen Rsync parancsot futtatnál, amely adatokat módosít, különösen a `--delete`-et használókat, használd a `--dry-run`-t az átvitel szimulálásához anélkül, hogy bármilyen változtatást végeznél:

```bash
rsync -a --dry-run /src_directory/ /dst_directory/
```

Az Rsync kiírja azokat a fájlokat, amelyeket átvinne vagy törölne, de semmit sem érint a lemezen. Ez a legbiztonságosabb módja annak, hogy ellenőrizd a parancsot, mielőtt valóban végrehajtanád.

## Távoli szinkronizálás

Az Rsync-kel történő távoli adatátvitelhez mind a forrás-, mind a célgépen telepítve kell lennie. Az Rsync alapértelmezésben SSH-t használ távoli shellként.

A következő példában egy könyvtárat viszünk át helyiről távoli gépre:

```bash
rsync -a /opt/media/ remote_user@remote_host_or_ip:/opt/media/
```

> **Info:** Ha nem állítottál be jelszó nélküli SSH-bejelentkezést a távoli gépre, a rendszer kérni fogja a felhasználói jelszót. Az SSH kapcsolódási kapcsolókról és hitelesítési opciókról lásd az `ssh` parancs útmutatóját.

Adatok átviteléhez távoli gépről helyire használd a távoli helyet forrásként:

```bash
rsync -a remote_user@remote_host_or_ip:/opt/media/ /opt/media/
```

Ha a távoli hoszton az SSH a 22-es alapértelmezettől eltérő porton figyel, add meg a portot a `-e` opcióval:

```bash
rsync -a -e "ssh -p 2322" /opt/media/ remote_user@remote_host_or_ip:/opt/media/
```

Nagy átvitelekhez ajánlott az `rsync` parancsot egy `screen` munkamenetben futtatni, vagy a `-P` opciót használni a folyamatjelző megjelenítéséhez és a megszakadt átvitelek folytatásához:

```bash
rsync -a -P remote_user@remote_host_or_ip:/opt/media/ /opt/media/
```

## Fájlok és könyvtárak kizárása

Kétféleképpen zárhatók ki fájlok és könyvtárak az rsync átvitelből. Az első a `--exclude` opció használata, és az elérési utak megadása a parancssorban.

Fájlok vagy könyvtárak kizárásakor azoknak a forráshelyhez viszonyított relatív elérési útját kell használni.

A következő példa kizárja a `node_modules` és a `tmp` könyvtárakat:

```bash
rsync -a --exclude=node_modules --exclude=tmp /src_directory/ /dst_directory/
```

A második lehetőség a `--exclude-from` használata, és a kizárandó elérési utak listázása egy fájlban:

```bash
rsync -a --exclude-from='/exclude-file.txt' /src_directory/ /dst_directory/
```

`/exclude-file.txt`

```plaintext
node_modules
tmp
```

## Gyors referencia

| Feladat | Parancs |
| --- | --- |
| Fájl másolása helyben | `rsync -a /src/file.zip /dest/` |
| Könyvtár szinkronizálása helyben | `rsync -a /src/ /dest/` |
| Tükrözés (törölt fájlok eltávolítása) | `rsync -a --delete /src/ /dest/` |
| Próba futtatás (csak előnézet) | `rsync -a --dry-run /src/ /dest/` |
| Átvitel távolira | `rsync -a /src/ user@host:/dest/` |
| Átvitel távoliról | `rsync -a user@host:/src/ /dest/` |
| Folyamatjelző megjelenítése | `rsync -a -P /src/ /dest/` |
| Egyedi SSH port használata | `rsync -a -e "ssh -p 2222" /src/ user@host:/dest/` |
| Könyvtár kizárása | `rsync -a --exclude=dir /src/ /dest/` |
| Kizárás fájlból | `rsync -a --exclude-from=list.txt /src/ /dest/` |
| Tömörítés átvitel közben | `rsync -a -z /src/ user@host:/dest/` |

Nyomtatható gyors referenciáért lásd az Rsync cheatsheet-et.

## Hibaelhárítás

- **Permission denied (Engedély megtagadva):**
  Győződj meg arról, hogy az olvasási jogosultságok be vannak állítva a forráson, és az írási jogosultságok a célon. Távoli átviteleknél ellenőrizd, hogy van SSH-hozzáférésed a távoli hoszthoz. Add hozzá a `-v`-t a részletes kimenethez a probléma diagnosztizálásához.

- **No route to host / Connection refused:**
  Ellenőrizd a hálózati kapcsolatodat, a tűzfalszabályokat (az SSH portnak nyitva kell lennie), és erősítsd meg a helyes hosztnevet vagy IP-címet. Teszteld a kapcsolatot `ping`-gel, vagy közvetlenül SSH-zz be a hosztra `ssh user@host` paranccsal.

- **Protocol version mismatch:**
  Ez általában akkor fordul elő, ha az egyik oldal régebbi rsync-verziót futtat. Először frissítsd az rsync-et a régebbi gépen, és ellenőrizd, hogy mindkét oldal kompatibilis verziókon van-e.

- **Files skipped or not transferred (Fájlok kihagyva vagy nem átvive):**
  Add hozzá a `-v`-t a részletek megtekintéséhez. Ellenőrizd az aktív `--exclude` szabályokat, a záró perjel különbségeket vagy a symlink-problémákat. Használd a `--dry-run`-t az átvitel előnézetéhez változtatások nélkül.

- **Slow transfers (Lassú átvitelek):**
  Kerüld a `-z`-t gyors hálózatokon; a tömörítés többletterhet ad. Nagy fájlokhoz lassú vagy megbízhatatlan kapcsolatokon használd a `-P`-t a folyamatjelző megjelenítéséhez és a megszakadt átvitelek folytatásához.

- **Vanished files error:**
  Ez akkor fordul elő, amikor a forrásfájlok megváltoznak az átvitel során. Futtasd újra a parancsot, vagy használd a `--ignore-errors`-t a tranziens hibák átlépéséhez.

## GYIK

- **Mit jelent a záró perjel a forráskönyvtáron?**
  Záró perjellel (`rsync -a /src/ /dest/`) az rsync a `src` tartalmát másolja a `dest`-be. Záró perjel nélkül (`rsync -a /src /dest/`) az rsync magát az `src` könyvtárat másolja a `dest`-be, ami `/dest/src/`-et eredményez. Ez az egyik leggyakoribb félreértésforrás az rsync használatakor.

- **Hogyan nézhetem meg előre, mit fog tenni az rsync anélkül, hogy bármit átvinne?**
  Add hozzá a `--dry-run`-t bármely rsync parancshoz. Az Rsync kiírja azokat a fájlokat, amelyeket átvinne vagy törölne, anélkül hogy bármit módosítana a lemezen.

- **Hogyan tarthatom szinkronban a célt, és hogyan távolíthatom el a forrásból törölt fájlokat?**
  Használd a `--delete`-et: `rsync -a --delete /src/ /dest/`. Mindig futtasd először `--dry-run`-nal, hogy megerősítsd, mely fájlok fognak törlődni.

- **Hogyan folytathatok egy megszakadt átvitelt?**
  Használd a `-P` opciót (`--partial --progress`). Megtartja a részben átvitt fájlokat, és folytatja azokat a következő futtatáskor: `rsync -a -P user@host:/src/ /dest/`.

- **Mi a különbség az rsync és az scp között?**
  Az `scp` feltétel nélkül másolja a fájlokat. Az `rsync` csak a forrás és a cél közötti különbségeket továbbítja, így sokkal hatékonyabb ismételt szinkronizálásokhoz vagy nagy könyvtárakhoz. Az `rsync` emellett támogat kizárásokat, próba futtatásokat és tükrözést, amelyekkel az `scp` nem rendelkezik.

## Következtetés

Az Rsync a szabványos eszköz a hatékony fájlszinkronizáláshoz Linuxon, helyben vagy SSH-n keresztül. Használd a `-a`-t archív módhoz, a `--dry-run`-t az alkalmazás előtti ellenőrzéshez, és a `--delete`-et óvatosan tükrözéskor.

További részletekért lásd az Rsync User’s Manualt vagy az Rsync cheatsheet-et.

Ha bármilyen kérdésed van, nyugodtan hagyj megjegyzést alább.

## Forrás
-[https://linuxize.com/post/how-to-use-rsync-for-local-and-remote-data-transfer-and-synchronization/]

# `scp` parancs – Példák

Íme néhány gyakorlati és alapvető `scp` parancs példa, amely bemutatja, hogyan lehet biztonságosan fájlokat másolni távoli Linux rendszerek között.

Valószínűleg már ismered a Linux `cp` parancsát, amelyet fájlok másolására használnak Linuxon. A `cp` parancs azonban csak a helyi gépen használható.

De mi a helyzet a fájlok másolásával egy távoli Linux rendszerről? Fájlok másolásához egy távoli szerverről vagy egy távoli szerverre használhatod az `scp` nevű parancsot.

> **Érdekesség:** Az `scp` a „Secured Copy” rövidítése.

## Az `scp` parancs szintaxisa

Az `scp` parancs szintaxisa és használata hasonló a `cp` parancséhoz, amint azt hamarosan látni fogod ezekben az `scp` parancs példákban.

De először nézzük meg az `scp` parancs szintaxisát:

```bash
scp <opciók> forrás_útvonal cél_útvonal
```

A másolandó fájl eredetétől függően a forrás lehet a kliens vagy a szerver. Ha a forrás a kliens, akkor a cél a szerver, és fordítva.

De ez túl egyszerűnek hangzik, nem igaz? Az `scp` parancs fő fókusza a helyes formátum használata a forrás- vagy célútvonal formátumához.

Egy tipikus elérési út egy fájlhoz vagy könyvtárhoz a szerveren ebben a formátumban van ábrázolva:

```plaintext
<felhasználónév>@<rendszer_ip_címe>:<fájl/könyvtár_elérési_útja>
```

Valójában használhatod a `user@hostname` formát is az `scp` parancsban, de valószínűbb, hogy „Could not resolve hostname” hibát kapsz.

Ez azért van, mert a hosztnév IP-címhez való hozzárendelése a szerver hálózatának konfigurációjától függ. A tényleges IP-cím használata megkímél ettől a problémától.

## Amire figyelni kell az `scp` parancs használata előtt

Íme néhány dolog, amire szükséged van, mielőtt használhatnád az `scp` parancsot a Linux rendszereden:

- Győződj meg arról, hogy az SSH telepítve van és fut mind a kliens, mind a szerver gépeken, mivel az `scp` parancs belsőleg SSH-t használ. Ha az SSH nincs telepítve, telepítsd a `sudo apt install ssh` paranccsal Debian-alapú gépeken, vagy a disztribúciód megfelelő csomagkezelőjével.
- Ismerned kell a távoli rendszer IP-címét vagy hosztnevét (ha ugyanazon az alhálózaton van).
- Ismerned kell a távoli rendszer felhasználónevét és jelszavát. A rendszer kérni fogja a távoli hoszt jelszavát. A fájlátvitel csak akkor kezdődik el, amikor megadod a távoli hoszt helyes jelszavát.

## Az `scp` parancs használata Linuxon: 10 gyakorlati példa

Most, hogy láttuk az `scp` parancs szintaxisát és a szerver elérési útjának megadásának formátumát, nézzük meg, hogyan kell használni az `scp` parancsot.

### 1. Fájl másolása a távoli rendszerre az `scp` paranccsal

Egyetlen fájl helyi gépről távoli hosztra másolásához add meg a fájl elérési útját forrásútvonalként, és a távoli hoszton lévő elérési utat, ahová a fájlt másolni kell.

```bash
scp abc.txt xyz@<ip_address_of_xyz>:/home/xyz/Desktop
```

### 2. Fájl másolása a távoli rendszerről az `scp` paranccsal

Egy fájl távoli rendszerről a helyi rendszerre másolása nagyjából ugyanaz. Csak meg kell adnod a fájl teljes elérési útját a távoli rendszeren és az elérési utat a helyi rendszeren.

Tehát ahhoz, hogy egy fájlt a távoli rendszerről az aktuális könyvtárba másolj, egyszerűen használd a parancsot a következő módon:

```bash
scp xyz@<ip_address_of_xyz>:/remote/user/home/abc.txt .
```

### 3. Több fájl másolása az `scp` paranccsal

Több fájl helyi gépről hosztra másolásához egyszerűen add meg a fájlok nevét forrásútvonalként.

```bash
scp abc.txt def.txt xyz@<ip_address_of_xyz>:/home/xyz/Desktop
```

### 4. Egy teljes könyvtár másolása az `scp` paranccsal

Egy könyvtár másolása az `scp`-vel szintén ugyanaz, mint a `cp` parancs esetében. Használhatod a `-r` opciót, és megadhatod a mappa nevét forrásútvonalként. Ezt rekurzív módnak nevezik.

```bash
scp -r ~/Desktop/test xyz@<ip_address_of_xyz>:/home/xyz/Desktop
```

Több könyvtárat is megadhatsz, és egyetlen parancsban másolhatod őket.

### 5. A fájlátvitel részleteinek megtekintése a kimenetben

Ha kíváncsi vagy arra, mi történik a másolás során a háttérben, használhatod a `-v` opciót, hogy lássd az összes végrehajtott folyamatot, beleértve a hibakeresést, a kilépési állapotot, a titkosítást, a visszafejtést stb. Ezt verbose módnak nevezik.

```bash
scp -v abc.txt xyz@<ip_address_of_xyz>:/home/xyz/Desktop
```

### 6. Fájlok másolása két távoli hoszt között

Eddig láttad, hogyan lehet helyi gépről távoli gépre másolni.

Ahhoz, hogy fájlokat másolj egyik távoli számítógépről a másikra, add meg mind a forrás, mind a cél elérési útját a távoli számítógép formátumában, ahogyan fentebb tárgyaltuk.

```bash
scp abc@<ip_address_of_abc>:<path_of_file_or_folder> xyz@<ip_address_of_xyz>:<path_of_file_or_folder>
```

Mondanunk sem kell, hogy itt mindkét rendszer jelszavát ismerned kell.

### 7. Fájlok másolása tömörítéssel [csökkenti az átviteli időt]

Ha megpróbálsz egy csomó fájlt együtt másolni, az több időt vesz igénybe, mint egyetlen, az összes fájl méretének összegével egyenértékű fájl átvitele.

Ha úgy érzed, hogy növelned kell a fájlátvitel sebességét, tömörítheted a fájlokat a `-C` opcióval, és átviheted őket.

A legjobb az, hogy a tömörítés csak az átvitel során történik. Az átvitel befejezése után a fájlok eredeti formájukban tárolódnak. Nem fogsz archív fájlt látni, hanem az összes egyedi fájlt, ahogyan a távoli szerveren voltak.

```bash
scp -C abc.txt xyz@<ip_address_of_xyz>:/home/xyz/Desktop
```

### 8. A fájlátvitel sávszélességének korlátozása

Ha úgy érzed, hogy a fájlok átvitele a sávszélesség nagy részét használja, csökkentheted a fájlátvitel sávszélességét a `-l` opcióval, amelyet az új sávszélesség értéke követ.

```bash
scp -l 800 abc.txt xyz@<ip_address_of_xyz>:/home/xyz/Desktop
```

Vegye figyelembe, hogy a parancsban megadott sávszélesség kilobit per másodpercben van, de a fájlok átvitele közben az `scp` parancs kimenete az átviteli sebességet kilobyte per másodpercben fogja mutatni.

Mivel 1 byte = 8 bit, a parancs paraméterében szereplő 800 kilobit per másodperc 100 kilobyte per másodperccé válik a parancs kimenetében.

### 9. A másolt fájlok eredeti attribútumainak megőrzése

Ha meg kell őrizned az eredeti fájlattribútumokat, például a fájljogosultságokat és a tulajdonjogot, a fájl időbélyegeit stb., használd a `-p` opciót.

```bash
scp -p abc.txt xyz@<ip_address_of_xyz>:/home/xyz/Desktop
```

### 10. Az `scp` parancs kimenetének elrejtése

Ha nem vagy elégedett az `scp` parancs hosszú kimenetével, elkerülheted a parancs kimenetét a `-q` opcióval. Ezt quiet módnak nevezik.

Még a fájlmásolás folyamatát is elrejti.

```bash
scp -q abc.txt xyz@<ip_address_of_xyz>:/home/xyz/Desktop
```

A fenti példákban láttad, hogyan lehet fájlokat másolni a távoli hosztra a helyi gépről. Ha fájlokat kell másolnod a távoli hosztról a helyi gépre, írd be a távoli hosztról másolandó fájl/mappa elérési útját forrásútvonalként, és a helyi gépen lévő elérési utat, ahová a fájlt/mappát be kell illeszteni, célútvonalként.

## Következtetés

Az `scp` paranccsal kapcsolatos további részletekért kérjük, olvasd el a man oldalát.

Remélem, mostanra megértetted, hogyan lehet a legjobban kihasználni az `scp` parancsot fájlok biztonságos másolásához számítógépek között. Személy szerint az `rsync` parancsot részesítem előnyben az `scp`-vel szemben, mert az `rsync`-nek több funkciója van.

Azonban, mint a `docker cp` parancs, az `scp` is hasonló szintaxist követ, mint a közönséges `cp` parancs. És ez vonzó választássá teszi sok Linux felhasználó számára.

Ha bármilyen problémába ütközöl, nyugodtan tegyél fel kérdéseket az alábbi megjegyzés szekcióban. És ha ismersz más nagyszerű `scp` parancs tippet, oszd meg velünk itt.

## Forrás 
-[https://linuxhandbook.com/scp-command/]

# A `wget` használata Linuxon – Példákkal

**Szerző:** Dejan Panovski
**Frissítve:** 2026. augusztus 14.
**11 perc olvasás**

## Ezen az oldalon

- A Wget telepítése
  - Ubuntu, Debian és származékaik
  - Fedora, RHEL és származékaik
- A Wget parancs szintaxisa
- Fájl letöltése
- Mentés más néven
- Letöltés egy adott könyvtárba
- Letöltés folytatása
- Letöltési sebesség korlátozása
- Több fájl letöltése
- Letöltés a háttérben
- Rekurzív letöltés és weboldal-tükrözés
- Hitelesítés
  - HTTP hitelesítés
  - FTP hitelesítés
- Hasznos opciók
  - URL létezésének ellenőrzése
  - Letöltés csak ha újabb
  - Egyedi HTTP fejlécek
  - Újrapróbálkozások és időtúllépések beállítása
  - User Agent megváltoztatása
  - Tanúsítvány-ellenőrzés kihagyása
  - Cookie-k használata
  - Proxy használata
  - Letöltés és átirányítás egy másik parancsba
  - Szerver által javasolt fájlnév használata
- Hibaelhárítás
- Gyors referencia
- GYIK
- Következtetés

A GNU Wget egy parancssori segédprogram fájlok letöltésére a webről. Támogatja a HTTP, HTTPS és FTP protokollokat, és olyan funkciókat kínál, mint a rekurzív letöltések, a sávszélesség-korlátozás, a folytatás támogatása és a háttérműködés.

Ez az útmutató gyakorlati példákon és a leggyakoribb opciók részletes magyarázatán keresztül mutatja be a `wget` parancs használatát.

## A Wget telepítése

A Wget a legtöbb Linux disztribúción előre telepítve van. Az ellenőrzéshez futtasd:

```bash
wget --version
```

Ha nincs telepítve, használd a disztribúciód csomagkezelőjét.

### Ubuntu, Debian és származékaik

```bash
sudo apt install wget
```

### Fedora, RHEL és származékaik

```bash
sudo dnf install wget
```

## A Wget parancs szintaxisa

A `wget` parancs általános szintaxisa:

```bash
wget [OPTIONS] [URL]
```

- **OPTIONS** – A letöltési viselkedést szabályozó kapcsolók
- **URL** – A letöltendő fájl vagy erőforrás címe

## Fájl letöltése

A legegyszerűbb formájában a `wget` letölti az adott URL-en lévő erőforrást az aktuális munkakönyvtárba:

```bash
wget https://releases.ubuntu.com/noble/ubuntu-24.04.4-live-server-amd64.iso
```

A letöltés során a `wget` megjeleníti a folyamatjelzőt a fájlnévvel, fájlmérettel, letöltési sebességgel és a hátralévő becsült idővel együtt. A letöltés befejezése után a fájl az aktuális könyvtárba kerül.

Ha már létezik azonos nevű fájl, a `wget` `.N` (egy szám) utótagot fűz a fájlnévhez, hogy elkerülje a felülírást. Ha inkább kihagynád a letöltést ahelyett, hogy számozott másolatot készítenél, használd a `-nc` (no clobber) opciót:

```bash
wget -nc https://ftp.gnu.org/gnu/wget/wget-1.25.0.tar.gz
```

**Kimenet:**

```plaintext
File ‘wget-1.25.0.tar.gz’ already there; not retrieving.
```

Az összes kimenet elnyomásához használd a `-q` (quiet) opciót:

```bash
wget -q https://example.com/file.tar.gz
```

A `-q` opció a folyamatjelzőt is elrejti. Ha csendes futtatást szeretnél, amely mégis megmutatja az átvitel állapotát, kombináld a `--show-progress` opcióval:

```bash
wget -q --show-progress https://ftp.gnu.org/gnu/wget/wget-1.25.0.tar.gz
```

**Kimenet:**

```plaintext
wget-1.25.0.tar.gz  100%[===================>]   5.02M  1.71MB/s    in 2.9s
```

Az alapértelmezett kimenet és a teljes csend közötti középútként a `-nv` (no verbose) csak a hibákat és egy egysoros összefoglalót ír ki minden fájlhoz.

## Mentés más néven

A letöltött fájl adott néven való mentéséhez használd a `-O` opciót:

```bash
wget -O latest-hugo.zip https://github.com/gohugoio/hugo/archive/refs/heads/master.zip
```

Ez `latest-hugo.zip` néven menti a fájlt az eredeti neve helyett.

## Letöltés egy adott könyvtárba

Alapértelmezésben a `wget` az aktuális könyvtárba menti a fájlokat. Más helyre mentéshez használd a `-P` opciót:

```bash
wget -P /mnt/iso https://releases.ubuntu.com/noble/ubuntu-24.04.4-live-server-amd64.iso
```

## Letöltés folytatása

Egy megszakadt letöltés folytatásához használd a `-c` (continue) opciót:

```bash
wget -c https://releases.ubuntu.com/noble/ubuntu-24.04.4-live-server-amd64.iso
```

**Kimenet:**

```plaintext
--2026-08-14 16:43:26--  https://releases.ubuntu.com/noble/ubuntu-24.04.4-live-server-amd64.iso
Resolving releases.ubuntu.com (releases.ubuntu.com)... 91.189.91.107, 185.125.190.37, 91.189.91.108, ...
Connecting to releases.ubuntu.com (releases.ubuntu.com)|91.189.91.107|:443... connected.
HTTP request sent, awaiting response... 206 Partial Content
Length: 3405469696 (3.2G), 3403237819 (3.2G) remaining [application/x-iso9660-image]
Saving to: ‘ubuntu-24.04.4-live-server-amd64.iso’
```

A `206 Partial Content` állapot és a hátralévő érték megerősítik, hogy a szerver onnan folytatta, ahol az előző kísérlet abbamaradt. A `200 OK` válasz itt azt jelentené, hogy a szerver figyelmen kívül hagyta a tartománykérést, és az átvitel nulláról indult újra.

Ez hasznos nagy fájlok letöltésekor megbízhatatlan kapcsolaton.

## Letöltési sebesség korlátozása

Hogy megakadályozd a `wget`-et abban, hogy az összes elérhető sávszélességet feleméssze, használd a `--limit-rate` opciót. Fűzz hozzá `k`-t kilobájtokhoz, `m`-et megabájtokhoz vagy `g`-t gigabájtokhoz:

```bash
wget --limit-rate=2m https://example.com/large-file.tar.gz
```

Ez 2 MB/másodpercre korlátozza a letöltési sebességet.

## Több fájl letöltése

Több fájl egyszerre történő letöltéséhez hozz létre egy szöveges fájlt soronként egy URL-lel, és add át a `wget`-nek a `-i` opcióval:

`urls.txt`

```plaintext
https://ftp.gnu.org/gnu/wget/wget-1.25.0.tar.gz
https://ftp.gnu.org/gnu/hello/hello-2.12.2.tar.gz
https://ftp.gnu.org/gnu/make/make-4.4.1.tar.gz
```

```bash
wget -i urls.txt
```

A Wget sorban dolgozza fel a listát, és egy `FINISHED` blokkal zár, amely jelentést ad arról, hány fájlt töltött le és a teljes átviteli időről.

Az URL-ek szabványos bemenetről való beolvasásához használd a `-i -` opciót.

## Letöltés a háttérben

A letöltés háttérfolyamatként való futtatásához használd a `-b` opciót:

```bash
wget -b https://releases.ubuntu.com/noble/ubuntu-24.04.4-live-server-amd64.iso
```

**Kimenet:**

```plaintext
Continuing in background, pid 77466.
Output will be written to ‘wget-log’.
```

A kimenet a `wget-log` fájlba íródik az aktuális könyvtárban. Az állapot figyeléséhez használd a `tail` parancsot:

```bash
tail -f wget-log
```

**Kimenet:**

```plaintext
--2026-08-14 16:42:43--  https://releases.ubuntu.com/noble/ubuntu-24.04.4-live-server-amd64.iso
Resolving releases.ubuntu.com (releases.ubuntu.com)... 91.189.91.108, 91.189.91.109, 91.189.91.107, ...
Connecting to releases.ubuntu.com (releases.ubuntu.com)|91.189.91.108|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3405469696 (3.2G) [application/x-iso9660-image]
Saving to: ‘ubuntu-24.04.4-live-server-amd64.iso’

     0K .......... .......... .......... .......... ..........  0%  229K 4h1m
```

Mivel a napló nem terminál, a `wget` pontokkal jelzi a folyamatot a szokásos sáv helyett. Minden pontsor 50 KB. Ha a naplót a `wget-log`-tól eltérő helyre szeretnéd küldeni, add hozzá a `-o` opciót:

```bash
wget -b -o ~/downloads.log https://releases.ubuntu.com/noble/ubuntu-24.04.4-live-server-amd64.iso
```

## Rekurzív letöltés és weboldal-tükrözés

Egy teljes weboldal offline böngészéshez való letöltéséhez használd a `-m` (mirror) opciót a `-k` (linkek átalakítása) és a `-p` (oldal kellékeinek letöltése) opciókkal kombinálva:

```bash
wget -m -k -p https://example.com
```

- `-m` – Tükrözési mód: engedélyezi a rekurzív letöltést, az időbélyegzést és a végtelen mélységet
- `-k` – A letöltött oldalak linkjeit helyi fájlokra mutatóvá alakítja
- `-p` – Letölti az összes erőforrást, amely az egyes oldalak megjelenítéséhez szükséges (CSS, képek, JavaScript)

A rekurzív mélység korlátozásához használd a `-l` opciót:

```bash
wget -r -l 2 https://example.com
```

Ez legfeljebb 2 szint mélységig tölti le az oldalakat a kezdő URL-től.

Ha egyetlen könyvtárat szeretnél, és nem az oldal többi részét, add hozzá a `--no-parent` opciót. Ez megakadályozza, hogy a `wget` a kezdő útvonal fölé mászó linkeket kövessen:

```bash
wget -r --no-parent https://example.com/docs/manuals/
```

A `--no-parent` nélkül egyetlen visszalink a `/docs/`-ra elegendő ahhoz, hogy behúzza azt a könyvtárat és mindent alatta.

Adott fájltípusok elfogadásához vagy elutasításához rekurzív letöltés során:

```bash
wget -r --accept=jpg,png,gif https://example.com/gallery/
```

```bash
wget -r --reject=mp4,avi https://example.com/media/
```

## Hitelesítés

### HTTP hitelesítés

Egy HTTP alap hitelesítést igénylő szerverről való letöltéshez használd a `--user` és `--password` opciókat:

```bash
wget --user=admin --password=secret https://example.com/protected/file.tar.gz
```

Tokenalapú hitelesítéshez add át a tokent egyedi fejlécként:

```bash
wget --header="Authorization: Bearer YOUR_TOKEN" https://api.example.com/data.json
```

### FTP hitelesítés

Jelszóval védett FTP-szerverről való letöltéshez:

```bash
wget --ftp-user=FTP_USERNAME --ftp-password=FTP_PASSWORD ftp://ftp.example.com/file.tar.gz
```

> ⚠️ **Figyelmeztetés:** Kerüld a valódi jelszavak, tokenek vagy munkamenet-cookie-k beírását olyan parancsokba, amelyek a shell előzményeibe mentődhetnek. Használj korlátozott jogosultságú ideiglenes fájlokat, verziókezelésből kizárt környezeti fájlokat, vagy interaktív promptokat, mint a `--ask-password`, amikor csak lehetséges.

## Hasznos opciók

### URL létezésének ellenőrzése

A `--spider` opció ellenőrzi, hogy egy URL elérhető-e a fájl letöltése nélkül:

```bash
wget --spider https://releases.ubuntu.com/noble/ubuntu-24.04.4-live-server-amd64.iso
```

**Kimenet:**

```plaintext
Spider mode enabled. Check if remote file exists.
--2026-08-14 16:21:02--  https://releases.ubuntu.com/noble/ubuntu-24.04.4-live-server-amd64.iso
Resolving releases.ubuntu.com (releases.ubuntu.com)... 185.125.190.37, 185.125.190.40, 91.189.91.107, ...
Connecting to releases.ubuntu.com (releases.ubuntu.com)|185.125.190.37|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3405469696 (3.2G) [application/x-iso9660-image]
Remote file exists.
```

Az utolsó sor az, amit ellenőrizni kell. Ha az URL hibás, ugyanaz a parancs másképp végződik:

```plaintext
HTTP request sent, awaiting response... 404 Not Found
Remote file does not exist -- broken link!!!
```

A `--spider` sikertelenség esetén nem nulla kilépési állapotot is beállít, ami használhatóvá teszi olyan szkriptekben, amelyek egy tükör ellenőrzését végzik egy nagy letöltés megkezdése előtt.

### Letöltés csak ha újabb

A `-N` (timestamping) opció csak akkor tölti le a fájlt, ha a távoli verzió újabb, mint a helyi másolat:

```bash
wget -N https://example.com/data.csv
```

### Egyedi HTTP fejlécek

Egyedi fejlécek küldéséhez a kéréssel használd a `--header` opciót:

```bash
wget --header="Accept: application/json" https://api.example.com/endpoint
```

A `--header` többször is átadható több fejléc hozzáadásához.

### Újrapróbálkozások és időtúllépések beállítása

Az újrapróbálkozási viselkedés és a kapcsolódási időtúllépések szabályozásához:

```bash
wget --tries=5 --timeout=30 https://example.com/file.tar.gz
```

- `--tries=N` – Legfeljebb N alkalommal próbálkozik újra (alapértelmezés: 20)
- `--timeout=N` – A DNS-, kapcsolódási és olvasási időtúllépést N másodpercre állítja

Hogy akkor is újrapróbálkozzon, ha a kapcsolatot elutasítják:

```bash
wget --tries=5 --retry-connrefused https://example.com/file.tar.gz
```

### User Agent megváltoztatása

Egyes szerverek blokkolják az alapértelmezett `wget` user agentről érkező kéréseket. Más user agent sztring küldéséhez használd a `-U` opciót:

```bash
wget -U "Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0" https://example.com/
```

### Tanúsítvány-ellenőrzés kihagyása

Érvénytelen vagy önaláírt SSL-tanúsítvánnyal rendelkező hosztról való letöltéshez használd a `--no-check-certificate` opciót:

```bash
wget --no-check-certificate https://self-signed.example.com/file.tar.gz
```

> ⚠️ **Figyelmeztetés:** A tanúsítvány-ellenőrzés kihagyása letilt egy fontos biztonsági ellenőrzést. Csak akkor használd ezt az opciót, ha megbízol a távoli hosztban és megérted a kockázatot.

### Cookie-k használata

A szerver által küldött állandó cookie-k fájlba mentéséhez használd a `--save-cookies` opciót:

```bash
wget --save-cookies cookies.txt https://example.com/login
```

Sok bejelentkezési rendszer munkamenet-cookie-kat használ. Azok mentéséhez is add hozzá a `--keep-session-cookies` opciót:

```bash
wget --save-cookies cookies.txt --keep-session-cookies https://example.com/login
```

A cookie-k fájlból való küldéséhez a következő kérésekkel használd a `--load-cookies` opciót:

```bash
wget --load-cookies cookies.txt https://example.com/protected/file.tar.gz
```

Ez hasznos, ha az erőforrás hitelesített cookie-t igényel. Kombináld a `--save-cookies`, `--keep-session-cookies` és `--load-cookies` opciókat, hogy ugyanazt a munkamenetet tartsd fenn több `wget` hívás között.

### Proxy használata

Proxy szerveren keresztüli letöltéshez add át a proxy címét a `-e` opcióval:

```bash
wget -e use_proxy=yes -e https_proxy=http://proxy.example.com:8080 https://example.com/file.tar.gz
```

A proxy környezeti változókat a `wget` hívása előtt is exportálhatod:

```bash
export http_proxy=http://proxy.example.com:8080
export https_proxy=http://proxy.example.com:8080
wget https://example.com/file.tar.gz
```

Ha a proxy hitelesítést igényel, add hozzá a `--proxy-user` és `--proxy-password` opciókat:

```bash
wget --proxy-user=USER --proxy-password=PASS https://example.com/file.tar.gz
```

### Letöltés és átirányítás egy másik parancsba

Egy fájl letöltéséhez és közvetlenül egy másik parancsba való átirányításához anélkül, hogy lemezre mentenéd, használd a `-O -` opciót:

> ⚠️ **Figyelmeztetés:** Csak olyan forrásokból irányíts át archívumokat, amelyekben megbízol. Szoftverletöltéseknél először ellenőrizd az ellenőrzőösszegeket vagy aláírásokat, és csomagold ki egy előkészítő könyvtárba, mielőtt a fájlokat egy webgyökérbe vagy rendszerútvonalba mozgatnád.

```bash
wget -q -O - "https://wordpress.org/latest.tar.gz" | tar -xzf - -C /var/www
```

Ez letölti a legújabb WordPress archívumot, és közvetlenül a `/var/www`-be csomagolja ki.

### Szerver által javasolt fájlnév használata

Egyes szerverek egy `Content-Disposition` fejlécet küldenek egy javasolt fájlnévvel. Alapértelmezésben a `wget` figyelmen kívül hagyja ezt, és az utolsó útvonalszegmens után nevezi el a fájlt:

```bash
wget "https://codeload.github.com/gohugoio/hugo/tar.gz/refs/tags/v0.128.0"
```

**Kimenet:**

```plaintext
Saving to: ‘v0.128.0’
```

Ez a név nem hasznos. A `--content-disposition` opcióval a `wget` a szerver által javasolt nevet használja helyette:

```bash
wget --content-disposition "https://codeload.github.com/gohugoio/hugo/tar.gz/refs/tags/v0.128.0"
```

**Kimenet:**

```plaintext
Saving to: ‘hugo-0.128.0.tar.gz’
```

## Hibaelhárítás

- **Unable to resolve host address:**
  A DNS-feloldás meghiúsult. Ellenőrizd, hogy a hosztnév helyes-e, és hogy a DNS-beállításaid működnek-e. Próbáld ki a `ping hostname` paranccsal.

- **HTTP request sent, awaiting response… 403 Forbidden:**
  A szerver blokkolja a kérést. Próbáld megváltoztatni a user agentet a `-U` opcióval, vagy ellenőrizd, hogy a szerver igényel-e hitelesítést.

- **ERROR: cannot verify certificate:**
  A szerver SSL-tanúsítványa érvénytelen vagy lejárt. Ha megbízol a hosztban, használd a `--no-check-certificate` opciót. Ellenkező esetben frissítsd a rendszer CA-tanúsítványait a `sudo apt install ca-certificates` vagy `sudo dnf install ca-certificates` paranccsal.

- **Download speed is extremely slow:**
  Ellenőrizd a kapcsolatodat, és hogy a távoli szerver lassítja-e. Próbálj meg egy másik tükröt, ha elérhető. Használd a `-c` opciót a folytatáshoz, ha a letöltés elakad.

- **wget: command not found:**
  A Wget nincs telepítve. Telepítsd a `sudo apt install wget` vagy `sudo dnf install wget` paranccsal a disztribúciódnak megfelelően.

## Gyors referencia

Nyomtatható gyors referenciáért lásd a Wget Cheatsheet-et.

| Parancs | Leírás |
| --- | --- |
| `wget URL` | Fájl letöltése |
| `wget -O name.txt URL` | Mentés adott fájlnévvel |
| `wget -P /path URL` | Mentés adott könyvtárba |
| `wget -nc URL` | Letöltés kihagyása, ha a fájl már létezik |
| `wget -q --show-progress URL` | Csendes kimenet folyamatjelzővel |
| `wget -c URL` | Megszakadt letöltés folytatása |
| `wget --limit-rate=2m URL` | Letöltési sebesség korlátozása |
| `wget -i urls.txt` | URL-ek letöltése fájlból |
| `wget -b URL` | Letöltés a háttérben |
| `wget -m -k -p URL` | Weboldal tükrözése offline böngészéshez |
| `wget -r --no-parent URL/dir/` | Rekurzív letöltés felfelé lépés nélkül |
| `wget --spider URL` | URL létezésének ellenőrzése letöltés nélkül |
| `wget -N URL` | Letöltés csak ha a távoli fájl újabb |
| `wget --tries=5 --timeout=30 URL` | Újrapróbálkozások és időtúllépés beállítása |
| `wget --header="Auth: token" URL` | Egyedi HTTP fejléc küldése |
| `wget -q -O - URL \| cmd` | Letöltés átirányítása egy másik parancsba |

## GYIK

- **Mi a különbség a wget és a curl között?**
  Mindkettő fájlokat tölt le a webről. A Wget a rekurzív letöltésekre, a weboldal-tükrözésre és a nagy átvitelek folytatására specializálódott. A curl több protokollt támogat, és jobban alkalmas API-interakciókra és adatküldésre. A Wget alapértelmezésben fájlba ment, míg a curl a szabványos kimenetre ír. Lásd a curl vs wget összehasonlítást egymás melletti példákért, vagy a curl parancs példák útmutatót a curl önálló használatához.

- **Hogyan tölthetek le egy teljes könyvtárat egy webszerverről?**
  Használd a `wget -r --no-parent URL/directory/` parancsot a könyvtár rekurzív letöltéséhez anélkül, hogy a szülőbe lépne. Add hozzá a `--accept` vagy `--reject` opciókat a fájltípusok szűréséhez.

- **Folytathatja a wget egy részben letöltött fájlt?**
  Igen. Használd a `-c` kapcsolót. A Wget egy Range fejlécet küld, hogy csak a hátralévő bájtokat kérje. Ha a szerver nem támogatja a tartománykéréseket, a letöltés az elejéről indul.

- **Hogyan próbálhatok újra automatikusan egy sikertelen letöltést?**
  A Wget alapértelmezésben legfeljebb 20 alkalommal próbálkozik újra. A `--tries=N` opcióval módosíthatod a korlátot. Add hozzá a `--retry-connrefused` opciót, hogy akkor is újrapróbálkozzon, ha a szerver elutasítja a kapcsolatot. A `--waitretry=N` opció lineáris visszalépést használ, egy másodperctől kezdve, és legfeljebb N másodpercig növelve az újrapróbálkozások között.

- **Hogyan tölthetek le bejelentkezést igénylő fájlokat?**
  HTTP alap hitelesítéshez használd a `--user` és `--password` opciókat. Tokenalapú API-khoz add át a tokent a `--header="Authorization: Bearer TOKEN"` opcióval. FTP-hez használd a `--ftp-user` és `--ftp-password` opciókat.

## Következtetés

A Wget egy sokoldalú eszköz fájlok letöltéséhez, weboldalak tükrözéséhez és átvitelek automatizálásához a parancssorból. Kombináld olyan opciókkal, mint a `-c` a folytatáshoz, a `--limit-rate` a sávszélesség-szabályozáshoz, és a `-m -k -p` az offline weboldal-másolatokhoz.

## Forrás
-[https://linuxize.com/post/wget-command-examples/]
