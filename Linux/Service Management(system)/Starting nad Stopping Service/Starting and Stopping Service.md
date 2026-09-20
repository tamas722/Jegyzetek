# Szolgáltatások és Démonok Kezelése Linuxon: systemd és SysVinit

A szolgáltatások (services) és démonok (daemons) olyan háttérben futó folyamatok, amelyek alapvető feladatokat látnak el a rendszerben (pl. webszerverek, adatbázisok, ütemezett feladatok). A Linux rendszerek ezek kezelésére ún. init rendszereket használnak.

- **systemd:** A modern Linux disztribúciók legelterjedtebb init rendszere. Párhuzamos indítást, igény alapú aktiválást és fejlett függőségkezelést biztosít. Fő eszköze a `systemctl`.
- **SysVinit:** Egy régebbi init rendszer, amely egyszerű szkripteket használ a szolgáltatások kezelésére. Ma már nagyrészt leváltotta a systemd, de néhány rendszeren még találkozhatunk vele. Fő eszközei a `service` és az `update-rc.d`.

## 1. Szolgáltatások kezelése systemd-vel (`systemctl`)

A `systemctl` a systemd elsődleges parancssori eszköze.

**Státusz ellenőrzése:**

```bash
sudo systemctl status <szolgáltatás_neve>
# Példa: sudo systemctl status apache2
```

**Indítás és Leállítás:**

```bash
sudo systemctl start <szolgáltatás_neve>
sudo systemctl stop <szolgáltatás_neve>
```

**Engedélyezés (automatikus indítás bootoláskor) és Tiltás:**

```bash
sudo systemctl enable <szolgáltatás_neve>
sudo systemctl disable <szolgáltatás_neve>
```

**Újraindítás (Restart) és Konfiguráció Újratöltése (Reload):**

(A reload leállás nélkül, röptében olvassa újra a konfigurációs fájlokat).

```bash
sudo systemctl restart <szolgáltatás_neve>
sudo systemctl reload <szolgáltatás_neve>
```

**Naplók (Logok) megtekintése:**

```bash
sudo journalctl -u <szolgáltatás_neve>
```

## 2. Szolgáltatások kezelése SysVinit-tel (`service`)

Bár ritkább, a régebbi rendszereken a `service` parancsot használjuk.

**Alapvető műveletek (Státusz, Indítás, Leállítás, Újraindítás):**

```bash
sudo service <szolgáltatás_neve> status
sudo service <szolgáltatás_neve> start
sudo service <szolgáltatás_neve> stop
sudo service <szolgáltatás_neve> restart
```

**Engedélyezés és Tiltás (bootoláskor):**

A SysVinit a runlevel (futási szint) könyvtárak frissítésével kezeli az automatikus indulást az `update-rc.d` paranccsal.

```bash
# Engedélyezés:
sudo update-rc.d <szolgáltatás_neve> defaults

# Tiltás:
sudo update-rc.d -f <szolgáltatás_neve> remove
```

## 3. Egyedi (Custom) Szolgáltatások Létrehozása

### A) Egyedi systemd szolgáltatás

1. **Fájl létrehozása:** Hozz létre egy `.service` fájlt az `/etc/systemd/system/` könyvtárban.

   ```bash
   sudo nano /etc/systemd/system/myservice.service
   ```

2. **Konfiguráció hozzáadása:**

   ```ini
   [Unit]
   Description=Saját egyedi szolgáltatás
   After=network.target

   [Service]
   ExecStart=/usr/local/bin/myscript.sh
   Restart=on-failure

   [Install]
   WantedBy=multi-user.target
   ```

3. **A systemd frissítése és a szolgáltatás elindítása:**

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable myservice
   sudo systemctl start myservice
   ```

### B) Egyedi SysVinit szolgáltatás

1. **Szkript létrehozása:**

   ```bash
   sudo nano /etc/init.d/myservice
   ```

2. **Szkript tartalma (kötelező LSB fejléccel):**

   ```bash
   #!/bin/sh
   ### BEGIN INIT INFO
   # Provides:          myservice
   # Required-Start:    $network
   # Required-Stop:     $network
   # Default-Start:     2 3 4 5
   # Default-Stop:      0 1 6
   # Short-Description: My Custom Service
   ### END INIT INFO

   case "$1" in
       start)
           echo "Starting myservice"
           /usr/local/bin/myscript.sh &
           ;;
       stop)
           echo "Stopping myservice"
           pkill -f /usr/local/bin/myscript.sh
           ;;
       *)
           echo "Usage: /etc/init.d/myservice {start|stop}"
           exit 1
           ;;
   esac
   exit 0
   ```

3. **Futtathatóvá tétel és engedélyezés:**

   ```bash
   sudo chmod +x /etc/init.d/myservice
   sudo update-rc.d myservice defaults
   sudo service myservice start
   ```

## 4. Haladó Szolgáltatáskezelés (systemd)

**Szolgáltatások Maszkolása (Masking):**

A maszkolás erősebb a tiltásnál (disable). Egy maszkolt szolgáltatást még manuálisan sem lehet elindítani (a `/dev/null`-ra linkeli a fájlt).

```bash
sudo systemctl mask <szolgáltatás_neve>
sudo systemctl unmask <szolgáltatás_neve>
```

**Konfigurációs fájlok szerkesztése:**

A szolgáltatás paramétereinek közvetlen és biztonságos szerkesztése:

```bash
sudo systemctl edit --full <szolgáltatás_neve>
# A mentés után mindig futtasd:
sudo systemctl daemon-reload
```

**Függőségek (Dependencies) kezelése:**

A `.service` fájl `[Unit]` szekciójában szabályozható, hogy a szolgáltatás miután, vagy mivel együtt induljon:

- `After=`: Csak a megadott cél/szolgáltatás után indul.
- `Requires=`: Kötelező függőség (ha az itt megadott leáll, ez is leáll).
- `Wants=`: Gyenge függőség (ajánlott, de nem kötelező).

## 5. Hibaelhárítás és Bevált Gyakorlatok

**Hibaelhárítás (Troubleshooting):**

Ha egy szolgáltatás elszáll vagy hibásan indul, ezek a parancsok segítenek a diagnosztikában:

```bash
# Általános hibaüzenetek és állapot:
sudo systemctl status <szolgáltatás_neve>

# Részletes rendszernapló a hibák kiderítésére (az eXtra részletes End, azaz legutóbbi üzenetek):
sudo journalctl -xe

# Csak az adott szolgáltatás naplójának megtekintése:
sudo journalctl -u <szolgáltatás_neve>
```

**Biztonsági mentés (Backup) automatizálása:**

A kritikus konfigurációs fájlokat érdemes rendszeresen menteni. Például egy napi Cron job beállítása (`sudo crontab -e`):

```bash
# Minden éjjel 2:00-kor másolatot készít a service fájlról
0 2 * * * cp /etc/systemd/system/myservice.service /backup/
```

##Forrás
-[https://dev.to/iaadidev/how-to-manage-services-in-linux-systemd-and-sysvinit-essentials-devops-prerequisite-8-1jop]

# A Linux Szolgáltatáskezelés (Service Management) Mesterfokon: systemctl

A Linux szolgáltatások (services) olyan háttérfolyamatok, amelyek alapvető rendszerfunkciókat biztosítanak. Folyamatosan futnak, kritikus feladatokat látnak el, és lehetővé teszik az alkalmazások hatékony működését. A modern Linux ökoszisztéma elsődlegesen a systemd-t használja init rendszerként ezeknek a szolgáltatásoknak a kezelésére.

## 1. A Linux Szolgáltatások Típusai és Életciklusa

A szolgáltatások strukturált életciklust követnek (állapotuk lehet pl. active, inactive, failed, running). Minden szolgáltatás viselkedését, indítási feltételeit és függőségeit egy úgynevezett unit fájl határozza meg.

| Szolgáltatás Típusa (Type) | Leírás | Példa |
| --- | --- | --- |
| Rendszer (System) | Alapvető, magszintű rendszerműveletek. | Hálózatkezelés (Network management) |
| Felhasználói (User) | Felhasználó-specifikus háttérfolyamatok. | Asztali környezet (Desktop environment) |
| Hálózati (Network) | Internetes és hálózatfüggő feladatok. | SSH, Webszerverek (pl. Apache, Nginx) |

**Alapvető parancsok a szolgáltatások azonosítására:**

```bash
# Az összes aktív rendszerszolgáltatás listázása
systemctl list-units --type=service

# Egy adott szolgáltatás állapotának lekérdezése
systemctl status ssh.service

# Egy szolgáltatás függőségeinek (dependencies) megtekintése
systemctl list-dependencies network.service
```

## 2. A systemctl Parancsok Alapjai (Command Guide)

A `systemctl` a legfontosabb parancssori segédprogram a systemd szolgáltatások kezelésére. Teljes kontrollt biztosít a háttérfolyamatok felett.

| Parancs | Funkció | Használati eset (Példa) |
| --- | --- | --- |
| `start` | Szolgáltatás elindítása. | Egy leállított szolgáltatás futtatásának megkezdése. |
| `stop` | Futó szolgáltatás leállítása. | A szolgáltatás folyamatainak befejezése (terminálása). |
| `restart` | Szolgáltatás újraindítása. | Konfigurációs módosítások érvényesítése. |
| `status` | Szolgáltatás állapotának lekérdezése. | A szolgáltatás "egészségének" (health) ellenőrzése. |
| `enable` | Automatikus indulás (autostart) beállítása. | A szolgáltatás perzisztens aktiválása bootoláskor. |
| `disable` | Automatikus indulás eltávolítása. | A bootoláskori automatikus indulás megakadályozása. |

**Gyakorlati parancspéldák:**

```bash
sudo systemctl start ssh             # SSH szolgáltatás indítása
sudo systemctl stop apache2          # Apache webszerver leállítása
sudo systemctl restart mysql         # MySQL adatbázis újraindítása
sudo systemctl status nginx          # Nginx állapotának ellenőrzése
sudo systemctl enable postgresql     # PostgreSQL automatikus indítása bootoláskor
sudo systemctl disable bluetooth     # Bluetooth automatikus indításának tiltása
```

## 3. Hibaelhárítási Stratégiák (Troubleshooting)

A hatékony hibakereséshez (debugging) meg kell érteni a szolgáltatások állapotát, és tudni kell elemezni a naplókat, valamint az erőforrás-használatot.

**Hibaelhárítási folyamat:** Probléma észlelése ➔ Állapot ellenőrzése ➔ Naplók elemzése ➔ Konfiguráció ellenőrzése ➔ Újraindítás / Újrakonfigurálás.

**Gyakori hibaelhárítási technikák és parancsok:**

| Technika | Parancs | Cél / Funkció |
| --- | --- | --- |
| Állapot (Status) | `systemctl status` | A szolgáltatás általános állapotának és legutóbbi üzeneteinek ellenőrzése. |
| Rendszernaplók (Logs) | `journalctl` | A hibaüzenetek és a futási előzmények részletes elemzése. |
| Erőforrás-figyelés | `top` / `htop` | Teljesítménybeli szűk keresztmetszetek (bottlenecks) azonosítása. |
| Függőségek (Dependencies) | `systemctl list-dependencies` | Szolgáltatások közötti konfliktusok vagy hiányzó feltételek megoldása. |

**Diagnosztikai parancspéldák:**

```bash
# Részletes állapotinformáció kérése
sudo systemctl status nginx

# A szolgáltatás-specifikus naplók (logok) megtekintése
sudo journalctl -u ssh.service

# Egy szolgáltatás függőségeinek ellenőrzése
systemctl list-dependencies mysql.service

# Rendszererőforrások monitorozása valós időben
top

# Annak ellenőrzése, hogy egy szolgáltatás engedélyezve van-e (autostart)
sudo systemctl is-enabled postgresql
```

##Forrás
-[https://labex.io/tutorials/linux-how-to-master-linux-service-management-with-systemctl-392864]
