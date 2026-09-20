# A Linux Rendszernaplók Kezelése: journalctl és systemd-journald

A `systemd-journald` szolgáltatás a rendszer, a kernel és az alkalmazások naplóit gyűjti. A hagyományos, egyszerű szöveges naplókkal (pl. `/var/log/syslog`) ellentétben a journal strukturált, indexelt, bináris formátumban tárolja az adatokat, ami villámgyors és rendkívül precíz keresést tesz lehetővé.

## 1. Elméleti Alapok és Tárolás

**Tárolási helyek:**

- `/run/log/journal/`: Illékony (Volatile) tároló, csak a memóriában létezik, újraindításkor (boot) a tartalma elvész.
- `/var/log/journal/`: Perzisztens (Persistent) tároló, túléli az újraindítást. (Ha nem létezik, a `mkdir -p /var/log/journal` paranccsal és a journald újraindításával létrehozható).

**Indexelt Metaadatok (Mezők):**

Minden bejegyzés kulcs-érték párokat tartalmaz. A legfontosabb belső mezők, amelyekre azonnal szűrhetünk:

- `_SYSTEMD_UNIT`: A szolgáltatás (unit) neve (pl. `nginx.service`).
- `_PID`: Folyamatazonosító.
- `_BOOT_ID`: A rendszerindítás egyedi azonosítója.
- `PRIORITY`: Súlyossági szint (0: emerg - 7: debug).
- `_COMM`: A parancs neve.

## 2. Alapvető Parancsok és Navigáció

- **Teljes napló megtekintése:** `journalctl` (Kilépés a `q` betűvel, keresés a `/` jellel).
- **Legújabb elöl (Fordított sorrend):** `journalctl -r`
- **Csak az utolsó X sor:** `journalctl -n 100` (utolsó 100 sor)
- **Valós idejű követés (tail -f):** `journalctl -f`
- **Lapozó (pager) kikapcsolása:** `journalctl --no-pager` (Kiváló szkriptekhez, mert a kimenet egyben jelenik meg).

## 3. A Naplók Szűrése (A legfontosabb funkciók)

A `journalctl` igazi ereje a szűrők kombinálásában rejlik. A szűrők logikai ÉS (AND) kapcsolatként működnek.

**Szűrés Szolgáltatás (Unit) szerint (`-u`):**

```bash
journalctl -u nginx.service
```

**Szűrés Rendszerindítás (Boot) szerint (`-b`):**

- Aktuális boot: `journalctl -b`
- Előző boot: `journalctl -b -1`
- Összes boot munkamenet listázása: `journalctl --list-boots`

**Szűrés Idő szerint (`--since`, `--until`):**

```bash
journalctl --since "1 hour ago"
journalctl --since "2026-02-13 10:00" --until "2026-02-13 11:00"
journalctl --since today
```

**Szűrés Prioritás (Súlyosság) szerint (`-p`):**

A szintek: 0 (emerg), 1 (alert), 2 (crit), 3 (err), 4 (warning), 5 (notice), 6 (info), 7 (debug).

```bash
# Csak a hibák és annál súlyosabbak:
journalctl -p err

# Tartomány megadása (figyelmeztetéstől a vészhelyzetig):
journalctl -p warning..emerg
```

**Csak a Kernel naplók (`-k`):**

A hardverhibák és kernel pánikok elkülönítésére (független az alkalmazásoktól).

```bash
journalctl -k -b
```

**Közvetlen szűrés Strukturált Mezőkre:**

```bash
journalctl _PID=1234
journalctl _COMM=nginx
```

**Kombinált példa (Gyors hibakereséshez):**

(Az Nginx hibáinak lekérdezése a jelenlegi boot során, az elmúlt 30 percben):

```bash
journalctl -u nginx.service -b -p err --since "30 minutes ago"
```

## 4. Kimeneti Formátumok és Exportálás

A strukturált adatok miatt a naplók könnyen átalakíthatók gépi feldolgozásra.

- **Részletes (minden metaadat):** `journalctl -o verbose`
- **JSON formátum (szkriptekhez, feldolgozáshoz):** `journalctl -o json`
- **Szép, olvasható JSON:** `journalctl -o json-pretty`

**Exportálás fájlba (irányítással):**

```bash
journalctl -u nginx.service --since "1 hour ago" --no-pager > nginx_error.log
```

## 5. Karbantartás és Lemezterület (Vacuum)

A Journald könnyen felemésztheti a lemezterületet, ha nincs korlátozva. Az automatikus korlátokat a `/etc/systemd/journald.conf` fájlban lehet beállítani (pl. `SystemMaxUse=500M`).

**Lemezhasználat ellenőrzése:**

```bash
journalctl --disk-usage
```

**Kézi helyfelszabadítás (Vacuum):**

```bash
# 7 napnál régebbi naplók törlése
sudo journalctl --vacuum-time=7d

# A napló méretének visszavágása 200 MB-ra
sudo journalctl --vacuum-size=200M

# Csak az utolsó 5 archivált fájl megtartása
sudo journalctl --vacuum-files=5
```

## 6. Jogosultságok és Biztonság

Alapértelmezésben a normál felhasználók csak a saját munkamenetük (session) naplóit láthatják.

A teljes rendszernapló (kernel, szolgáltatások, más felhasználók) olvasásához root jogosultság, vagy a `systemd-journal` csoport tagsága szükséges.

**Felhasználó hozzáadása a jogosult csoporthoz** (sudo nélkül is olvashatja a naplót):

```bash
sudo usermod -aG systemd-journal username
```

## 7. Gyors Referencia Táblázat (Cheatsheet)

| Cél | Parancs |
| --- | --- |
| Összes napló megtekintése | `journalctl` |
| Legújabbak elöl | `journalctl -r` |
| Utolsó 100 sor | `journalctl -n 100` |
| Naplók valós idejű követése | `journalctl -f` |
| Egy szolgáltatás naplói | `journalctl -u nginx.service` |
| Szolgáltatás naplói (jelenlegi boot) | `journalctl -u nginx.service -b` |
| Csak a jelenlegi boot | `journalctl -b` |
| Előző boot | `journalctl -b -1` |
| Összes boot listázása | `journalctl --list-boots` |
| Adott időpont óta | `journalctl --since "2026-02-13 10:00"` |
| Relatív időszűrő | `journalctl --since "1 hour ago"` |
| Szűrés prioritás (hiba) szerint | `journalctl -p err` |
| Prioritási tartomány | `journalctl -p warning..emerg` |
| Csak kernelnaplók | `journalctl -k` |
| Szűrés PID alapján | `journalctl _PID=1234` |
| JSON kimenet | `journalctl -o json` |
| Lapozó (pager) kikapcsolása | `journalctl --no-pager` |
| Lemezhasználat ellenőrzése | `journalctl --disk-usage` |
| Naplók tisztítása idő alapján | `sudo journalctl --vacuum-time=7d` |
| Naplók tisztítása méret alapján | `sudo journalctl --vacuum-size=200M` |

 ##Forrás
 -[https://uptimerobot.com/knowledge-hub/logging/journalctl-explained-how-to-view-and-analyze-systemd-logs/]

 # A systemd Journal és a journalctl Használata

A systemd journal egy központosított naplózó rendszer, amely összegyűjti és kezeli a kernel, a szolgáltatások és a felhasználói folyamatok naplóit (logjait). A hagyományos, szétszórt és egyszerű szöveges naplók helyett a journal a `journald` démon segítségével bináris, strukturált és indexelt formátumban tárolja az adatokat. Ennek az adatbázisnak az olvasására és szűrésére szolgál a `journalctl` parancssori eszköz.

## 1. Előfeltétel: A Rendszeridő Helyes Beállítása (`timedatectl`)

Mivel a journald a naplókat binárisan rögzíti, képes azokat bármilyen időzónában megjeleníteni. Hogy a naplók időbélyege (timestamp) pontos legyen, a `timedatectl` paranccsal érdemes beállítani az időzónát.

- Elérhető időzónák listázása: `timedatectl list-timezones`
- Időzóna beállítása (pl. Budapest): `sudo timedatectl set-timezone Europe/Budapest`
- Aktuális idő és státusz lekérdezése: `timedatectl status`

## 2. Alapvető Használat és Kimeneti Beállítások

- **Az összes napló megtekintése:** `journalctl` (A legrégebbi elöl, lapozóba töltve).
- **Kimenet UTC idő szerint:** `journalctl --utc`
- **Lapozó (pager) kikapcsolása:** `journalctl --no-pager` (Hasznos szkriptekhez vagy `grep`-hez).
- **Hosszú sorok teljes megjelenítése levágás nélkül:** `journalctl -a`
- **Hosszú sorok csonkolása (ellipszis):** `journalctl --no-full`

**Kimeneti Formátumok (`-o` / `--output`):**

A bináris tárolás miatt a naplók tetszőleges formátumba exportálhatók:

- `journalctl -o json` (Szkriptekhez és gépi feldolgozáshoz)
- `journalctl -o json-pretty` (Ember által könnyebben olvasható JSON)
- További formátumok: `short` (alapértelmezett syslog stílus), `verbose` (minden rejtett metaadat megjelenítése), `cat` (csak az üzenet), `export`.

## 3. Valós Idejű Követés és Sorok Száma (tail alternatívák)

- **Az utolsó N sor megjelenítése:** `journalctl -n 20` (Megmutatja az utolsó 20 sort).
- **Élő követés (Real-time / Follow):** `journalctl -f` (Hasonló a `tail -f` parancshoz, folyamatosan frissül).

## 4. Szűrési Lehetőségek (Filtering)

A `journalctl` igazi ereje a pontos és indexelt szűrésekben rejlik.

### A) Szűrés Rendszerindítás (Boot) Szerint (`-b`)

- Aktuális boot: `journalctl -b`
- Boot munkamenetek listázása: `journalctl --list-boots` (Megmutatja az azonosítókat és az időintervallumokat).
- Előző bootok: `journalctl -b -1` (Az előző), `journalctl -b -2` (Kettővel ezelőtti).
- Boot azonosító (Boot ID) alapján: `journalctl -b caf0524a1d394ce0bdbcff75b94444fe`

### B) Szűrés Időtartam Szerint (`--since`, `--until`)

Formátum: `ÉÉÉÉ-HH-NN ÓÓ:PP:MM`, vagy relatív kulcsszavak (pl. `today`, `yesterday`, `1 hour ago`).

```bash
journalctl --since "2015-01-10 17:15:00"
journalctl --since 09:00 --until "1 hour ago"
journalctl --since yesterday
```

### C) Szűrés Szolgáltatás (Unit) és Elérési Út Szerint

- Adott systemd unit lekérdezése: `journalctl -u nginx.service`
- Több szolgáltatás kombinálása (pl. webszerver és PHP): `journalctl -u nginx.service -u php-fpm.service --since today`
- Futtatható állomány (Executable path) alapján: `journalctl /usr/bin/bash`

### D) Szűrés Prioritás (Súlyosság) Szerint (`-p`)

A szabványos syslog szinteket használja (0-tól 7-ig). A megadott szintet és a nála súlyosabbakat listázza.

Szintek: 0: emerg, 1: alert, 2: crit, 3: err, 4: warning, 5: notice, 6: info, 7: debug.

```bash
# Csak a hibák és kritikus események:
journalctl -p err -b
```

### E) Szűrés Kernel, PID, UID és Egyéb Mezők Szerint

- Kernel naplók (dmesg alternatíva): `journalctl -k` (vagy `--dmesg`). Szűrhető régebbi bootokra is: `journalctl -k -b -5`.
- Folyamatazonosító (PID): `journalctl _PID=8088`
- Felhasználó (UID) vagy Csoport (GID): `journalctl _UID=33`
- Rendelkezésre álló mezőértékek listázása (pl. mik a létező GID-k a naplóban): `journalctl -F _GID`

## 5. Lemezterület, Takarítás (Vacuum) és Perzisztencia

### Perzisztens (Állandó) Naplózás Bekapcsolása

Alapértelmezésben a naplók sokszor csak a memóriában (illékony, `/run/log/journal/`) tárolódnak, és újraindításkor elvesznek. A perzisztens tároláshoz létre kell hozni a megfelelő mappát:

```bash
sudo mkdir -p /var/log/journal
sudo systemd-tmpfiles --create --prefix /var/log/journal
sudo systemctl restart systemd-journald
```

(Az `/etc/systemd/journald.conf` fájlban a `Storage=persistent` beállítással is garantálható a működés).

### Lemezhasználat és Takarítás

- Jelenlegi lemezhasználat ellenőrzése: `journalctl --disk-usage`
- Takarítás méret alapján (Vacuum): `sudo journalctl --vacuum-size=1G` (Minden régi naplót töröl, amíg a méret 1 GB alá nem csökken).
- Takarítás idő alapján: `sudo journalctl --vacuum-time=1years` (vagy pl. `2weeks`).
- Konfigurációs limitek (Limits): Az `/etc/systemd/journald.conf` fájlban beállítható automatikus korlátok: `SystemMaxUse=`, `SystemKeepFree=`, `SystemMaxFileSize=`.

## 6. Hibaelhárítás és Jogosultságok (Permissions)

Ha a `journalctl` futtatásakor "Permission denied" (Hozzáférés megtagadva) hibát kapsz, vagy nem látsz minden naplót, annak oka, hogy alapértelmezésben csak a root látja a rendszerszintű eseményeket.

**Megoldás (sudo nélküli teljes hozzáféréshez):**

Add hozzá a felhasználódat a `systemd-journal` csoporthoz (majd jelentkezz ki és vissza):

```bash
sudo usermod -aG systemd-journal yourusername
```

**Sikertelen szolgáltatások (Failed services) extra vizsgálata:**

Ha egy szolgáltatás (pl. Nginx) elszállt, a kibővített hibakereséshez használd a `-xe` (extra + end) kapcsolókat:

```bash
journalctl -xe
```

**SSH hozzáférések figyelése:**

```bash
journalctl -u ssh.service -f
# Vagy szűrés sikertelen jelszavakra:
journalctl -u ssh.service | grep "Failed password"
```

## 7. Összehasonlítás: dmesg vs. journalctl

Míg a `dmesg` a hagyományos eszköz a kernelnaplókhoz, a `journalctl` sokkal átfogóbb megoldást nyújt:

| Funkció | dmesg | journalctl (különösen a `-k` kapcsolóval) |
| --- | --- | --- |
| Hatókör (Scope) | Csak a Kernel gyűrűs puffere (ring buffer) | Kernel + Rendszerszolgáltatások + Felhasználói (userland) naplók |
| Perzisztencia | Újraindításkor (vagy ha a puffer megtelik) törlődik | Perzisztens (ha engedélyezve van a `/var/log/journal` alatt) |
| Metaadatok | Nincs strukturált metaadat | Gazdag, indexelt metaadatok (Unit, PID, UID stb.) |
| Szűrési képességek | Nagyon korlátozott | Kiterjedt (Idő, Boot ID, Szolgáltatás, Prioritás stb.) |
| Kimeneti formátumok | Nyers, egyszerű szöveg | JSON, export, short, verbose, stb. |
| Jogosultságok | A teljes kimenethez általában sudo kell | root vagy a `systemd-journal` csoport tagsága szükséges |

##Forrás
-[https://www.digitalocean.com/community/tutorials/how-to-use-journalctl-to-view-and-manipulate-systemd-logs]
