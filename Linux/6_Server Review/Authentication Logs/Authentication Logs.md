# A Linux Bejelentkezési Előzmények (Login History) Ellenőrzése

Akár több felhasználós a rendszered, akár nem, fontos tisában lenni azzal, hogy ki, honnan és mikor lépett be a szerverre. Ez különösen hasznos a biztonság szempontjából, hiszen a botok folyamatosan próbálnak betörni a nyitott Linux szerverekre.

## 1. A Bejelentkezéseket Tároló Naplófájlok (Logok)

A Linux minden bejelentkezési eseményt rögzít. Ezek az információk alapvetően három fájlban tárolódnak:

- `/var/log/wtmp` – Az utolsó (sikeres) bejelentkezési munkamenetek naplója.
- `/var/run/utmp` – A jelenleg is futó (aktív) bejelentkezések naplója.
- `/var/log/btmp` – A sikertelen (hibás) bejelentkezési kísérletek naplója.

## 2. A `last` Parancs Használata (Sikeres bejelentkezések)

### Összes bejelentkezési előzmény listázása

A `last` parancs a `/var/log/wtmp` fájlt olvassa ki, és megmutatja a sikeres belépéseket.

```bash
last
```

**Kimenet példa:**

```plaintext
abhi     pts/0        202.91.87.115    Wed Mar 13 13:31   still logged in
root     pts/0        202.91.87.115    Wed Mar 13 13:30 - 13:31  (00:00)
servesha pts/0        125.20.97.117    Tue Mar 12 12:07 - 14:25  (02:17)
...
wtmp begins Mon Mar  4 13:35:54 2019
```

**Magyarázat:**

- Látható a felhasználónév, a kapcsolat típusa (a `pts/0` SSH kapcsolatra utal), az IP-cím, a dátum/idő, valamint az időtartam (vagy a "still logged in" felirat, ha még bent van).
- Az utolsó sor (`wtmp begins...`) mutatja, mikor jött létre a naplófájl. Ennél régebbi adatokat a parancs nem tud mutatni.

### Egy konkrét felhasználó előzményei

Ha csak egy adott felhasználó belépéseire vagy kíváncsi, írd a nevét a parancs után:

```bash
last servesha
```

### Hosztnevek helyett IP-címek megjelenítése (`-i`)

Előfordulhat, hogy a `last` alapértelmezésben hosztneveket mutat (főleg alhálózatokon). Az `-i` opcióval kikényszerítheted a numerikus IP-címek megjelenítését:

```bash
last -i
```

### Csak az utolsó N darab bejelentkezés listázása (`-N`)

Mivel a lista hatalmas lehet, korlátozhatod a kiírt sorok számát. Például, ha csak az utolsó 5 bejelentkezést szeretnéd látni:

```bash
last -5
```

(Ezt kombinálhatod a felhasználónévvel is: `last -5 servesha`).

## 3. Sikertelen bejelentkezési kísérletek ellenőrzése (`lastb`)

Ez a legfontosabb rész a biztonság szempontjából. A sikertelen próbálkozásokat a `lastb` paranccsal lehet kilistázni (amely a `/var/log/btmp` fájlt olvassa). Ez a parancs tulajdonképpen egyenértékű a `last -f /var/log/btmp` futtatásával.

```bash
lastb
```

**Kimenet példa:**

```plaintext
root     ssh:notty    218.92.0.158     Wed Mar 13 14:34 - 14:34  (00:00)
sindesi  ssh:notty    59.164.69.10     Wed Mar 13 14:34 - 14:34  (00:00)
root     ssh:notty    218.92.0.158     Wed Mar 13 14:34 - 14:34  (00:00)
```

**Magyarázat a védekezéshez:**

A hibás bejelentkezés lehet egy elgépelt jelszó egy legitim felhasználótól, de ha egy ismeretlen IP-címről rengeteg próbálkozást látsz (pl. a `root` felhasználóval), az valószínűleg egy bot, ami brute-force (nyerserős) jelszótörést próbál végrehajtani a szervereden.

> **Biztonsági tipp:** Ilyen esetekben érdemes telepíteni és beállítani a `Fail2Ban` nevű szoftvert, amely automatikusan kitiltja (ban) azokat az IP-címeket, ahonnan túl sok hibás próbálkozás érkezik.

##Forrás
-[https://linuxhandbook.com/linux-login-history/]

# A Linux Hitelesítési Naplók (Authentication Logs) Felügyelete és Elemzése

A Linux hitelesítési naplók nem csupán a szerverhez való hozzáférés rögzítésére szolgálnak, hanem kulcsfontosságúak a mintázatok megértésében, az esetleges behatolások azonosításában és a rendszerbiztonság növelésében.

## 1. A Hitelesítési Naplók Vizsgálata

A naplófájlok valós idejű megfigyelésével (a `tail -f` paranccsal) nyomon követhetjük a sikeres és sikertelen belépéseket, valamint a munkamenetek bontását.

**Ubuntu / Debian rendszereken:**

```bash
sudo tail -f /var/log/auth.log
```

**Fedora / Red Hat / CentOS rendszereken:**

```bash
sudo tail -f /var/log/secure
```

**Alternatív megoldás a systemd használatával** (kizárólag az SSH eseményekre szűrve):

```bash
journalctl -u ssh
```

**Példa egy sikertelen bejelentkezési kísérlet (brute-force) naplóbejegyzésére:**

```plaintext
Feb 10 15:45:14 ubuntu-lts sshd[47343]: Failed password for root from 103.106.189.143 port 33990 ssh2
```

Látható a "Ki" (`root`), "Mi" (`Failed password`), "Mikor" (`Feb 10 15:45:14`), "Honnan" (`103.106.189.143`) és "Hogyan" (`ssh2`).

## 2. A Naplók Strukturálása (JSON formátumba alakítás)

A szabványos Syslog formátum strukturálatlan, ami megnehezíti az automatizált elemzést. A megoldás a naplók elemzése (parsing) és strukturált JSON formátumba alakítása egy olyan eszközzel, mint például a Vector.

**Példa a Vector konfigurációra** (`/etc/vector/vector.yaml`):

A Vector beolvassa a fájlt, reguláris kifejezésekkel (regex) kinyeri az IP-címet, felhasználónevet, portot, majd JSON-ként adja tovább.

```yaml
sources:
  auth_logs:
    type: file
    include:
      - /var/log/auth.log

transforms:
  parse_auth_log:
    inputs:
      - auth_logs
    type: remap
    source: |
      . |= parse_linux_authorization!(.message)
      .event = { "category": "authentication" }
      
      if contains(.message, "Failed password") {
          .event.name = "failed password"
          .context = parse_regex!(.message, r'Failed password for (invalid user )?(?P<username>\w+) from (?P<ip_address>[\d.]+) port (?P<port>\d+) (?P<protocol>\w+)')
      }
      # (Itt további szabályok következnek a többi eseménytípushoz...)

sinks:
  print:
    type: console
    inputs:
      - parse_auth_log
    encoding:
      codec: json
```

A Vector elindítása után (`sudo vector`) a kimenet egy jól kereshető JSON objektum lesz:

```json
{
  "appname": "sshd",
  "context": {
    "ip_address": "180.101.88.228",
    "port": "55488",
    "stage": "preauth",
    "username": "root"
  },
  "event": {
    "category": "authentication",
    "name": "disconnect user"
  },
  "file": "/var/log/auth.log",
  "message": "Disconnected from authenticating user root 180.101.88.228 port 55488 [preauth]"
}
```

## 3. Logok Elemzése és Vizualizációja (Dashboardok és SQL)

Miután a strukturált logokat egy központi naplókezelőbe (pl. Better Stack, ELK) továbbítottuk (a Vector http sink-jével), SQL lekérdezésekkel készíthetünk kimutatásokat.

**Sikertelen belépések egyszerű szűrése:**

```plaintext
event.name:"failed password"
```

**Sikertelen belépések összesítése felhasználónevek szerint (SQL példa):**

Megmutatja, melyik felhasználónevekkel próbálnak a leggyakrabban betörni:

```sql
SELECT username as "User", countMerge(events_count) AS "Attempts"
FROM {{source}}
WHERE {{time}} BETWEEN {{start_time}} AND {{end_time}}
AND event_name = 'failed password'
AND username IS NOT NULL
GROUP BY username
ORDER BY "Attempts" DESC
```

**Sikertelen belépések összesítése IP-címek szerint (SQL példa):**

```sql
SELECT ip_address as "IP Address", countMerge(events_count) AS "Attempts"
FROM {{source}}
WHERE {{time}} BETWEEN {{start_time}} AND {{end_time}}
AND event_name = 'failed password'
AND ip_address IS NOT NULL
GROUP BY ip_address
ORDER BY "Attempts" DESC
```

**Sikeres belépések listázása (SQL példa):**

```sql
SELECT {{time}} as "Timestamp", username as "User", ip_address as "IP Address", auth_method as "Auth method"
FROM {{source}}
WHERE {{time}} BETWEEN {{start_time}} AND {{end_time}}
  AND event_name = 'successful login'
ORDER BY "Timestamp" DESC
```

## 4. Riasztások (Alerts) Beállítása

A megfigyelés mellett riasztásokat is be kell állítani a gyanús eseményekre.

Például, egy riasztás beállítása arra az esetre, ha valaki sikeresen belép root-ként:

**Szűrési feltétel a riasztáshoz:**

```plaintext
event.name="successful login" context.username=root
```

Ha a találatok száma nagyobb, mint 0, a rendszer azonnal e-mail értesítést küld.

**További javasolt riasztási esetek:**

- Gyakori hitelesítési hibák valós (létező) felhasználónevek esetén (szivárgásra utalhat).
- Sikeres belépések szokatlan vagy nem engedélyezett IP-címekről.
- Sikeres belépések jelszóval (ha az SSH kulcsos azonosítás az elvárt).
- Túl sok root hozzáférési próbálkozás (brute-force jelenléte).

## 5. A Szerver Védelme és Keményítése (Taking Action)

A naplózás csak a problémát mutatja meg. A szerver védelme érdekében az alábbi gyakorlati lépéseket érdemes megtenni:

- **Jelszavas bejelentkezés tiltása:** Használj kizárólag SSH kulcsokat.
- **SSH port megváltoztatása:** A 22-es alapértelmezett port megváltoztatása jelentősen csökkenti az automatizált botok támadásait.
- **Tűzfal és Fail2Ban használata:** A Fail2Ban automatikusan kitiltja (ban) azokat az IP-címeket, ahonnan túl sok hibás próbálkozás érkezik.
- **Kétfaktoros hitelesítés (MFA) vagy Port Knocking beállítása.**

### Gyakorlati Példa: A root bejelentkezés tiltása SSH-n keresztül

A legfontosabb biztonsági lépés, hogy megakadályozzuk a közvetlen root hozzáférést.

1. Nyisd meg az SSH konfigurációs fájlt szerkesztésre:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Keresd meg a `PermitRootLogin` sort, vedd ki előle a komment jelet (`#`), és állítsd `no`-ra:

   ```plaintext
   PermitRootLogin no
   ```

3. Mentsd el a fájlt, majd indítsd újra az SSH szolgáltatást a változások érvényesítéséhez:

   ```bash
   sudo systemctl restart sshd
   ```

4. Ellenőrzésképpen próbálj meg root-ként belépni:

   ```bash
   ssh root@<your_server_ip>
   ```

   **Kimenet:** A szerver el fogja utasítani a kapcsolatot:

   ```plaintext
   Received disconnect from <your_server_ip> port 22:2: Too many authentication failures
   Disconnected from <your_server_ip> port 22
   ```

##Forrás
-[https://betterstack.com/community/guides/logging/monitoring-linux-auth-logs/]
