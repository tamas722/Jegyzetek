# A Linux Szolgáltatások (Services) és Démonok Kezelése: systemd és SysVinit

A szolgáltatások (services) és démonok (daemons) olyan háttérben futó folyamatok, amelyek alapvető funkciókat látnak el a rendszerben (pl. webkiszolgálók, adatbázisok, hálózati kezelés, ütemezett feladatok). A démonok specifikusan olyan szolgáltatások, amelyeket felhasználói beavatkozás (felügyelet) nélküli futásra terveztek.

A Linux rendszerek különböző "init" (inicializáló) rendszereket használnak ezen folyamatok kezelésére:

- **systemd:** A legelterjedtebb, modern init rendszer. Párhuzamos indítást, igény szerinti (on-demand) aktiválást és fejlett függőségkezelést biztosít.
- **SysVinit:** Egy régebbi, egyszerű indítószkripteken alapuló rendszer. Bár nagyrészt felváltotta a systemd, néhány disztribúcióban még mindig találkozhatunk vele.

## 1. Szolgáltatások Kezelése a systemd (`systemctl`) segítségével

A `systemctl` a systemd elsődleges parancssoros eszköze.

**Státusz ellenőrzése:**

```bash
sudo systemctl status apache2
```

**Indítás és Leállítás:**

```bash
sudo systemctl start apache2
sudo systemctl stop apache2
```

**Engedélyezés (Automatikus indulás bootoláskor) és Tiltás:**

```bash
sudo systemctl enable apache2
sudo systemctl disable apache2
```

**Újraindítás és Újratöltés (Reload):**

(A reload újraolvassa a konfigurációs fájlt anélkül, hogy leállítaná a futó folyamatot, míg a restart teljesen leállítja és újraindítja azt).

```bash
sudo systemctl restart apache2
sudo systemctl reload apache2
```

**Szolgáltatás naplóinak (logjainak) megtekintése:**

```bash
sudo journalctl -u apache2
```

## 2. Szolgáltatások Kezelése a SysVinit (`service`) segítségével

A régebbi rendszereken a `service` parancs biztosítja az alapvető funkciókat.

**Státusz, Indítás és Leállítás:**

```bash
sudo service apache2 status
sudo service apache2 start
sudo service apache2 stop
```

**Újraindítás:**

```bash
sudo service apache2 restart
```

**Engedélyezés és Tiltás (SysVinit környezetben):**

Ehhez a futási szintek (runlevels) könyvtárait kell frissíteni az `update-rc.d` paranccsal.

- Engedélyezés (Indulás bootkor):

  ```bash
  sudo update-rc.d apache2 defaults
  ```

- Tiltás:

  ```bash
  sudo update-rc.d -f apache2 remove
  ```

## 3. Egyedi (Saját) Szolgáltatások Létrehozása

### A) Saját systemd szolgáltatás létrehozása

1. Hozd létre a Service fájlt:

   ```bash
   sudo nano /etc/systemd/system/myservice.service
   ```

2. Add hozzá a konfigurációt:

   ```ini
   [Unit]
   Description=My Custom Service
   After=network.target

   [Service]
   ExecStart=/usr/local/bin/myscript.sh
   Restart=on-failure

   [Install]
   WantedBy=multi-user.target
   ```

3. Töltsd újra a systemd konfigurációt, engedélyezd és indítsd el:

   (A `daemon-reload` kötelező minden új vagy módosított unit fájl után!)

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable myservice
   sudo systemctl start myservice
   ```

### B) Saját SysVinit szolgáltatás létrehozása

1. Hozd létre az Init szkriptet:

   ```bash
   sudo nano /etc/init.d/myservice
   ```

2. Add hozzá a szkript tartalmát (Case ágakkal):

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

3. Tedd futtathatóvá, engedélyezd és indítsd el:

   ```bash
   sudo chmod +x /etc/init.d/myservice
   sudo update-rc.d myservice defaults
   sudo service myservice start
   ```

## 4. Haladó Szolgáltatáskezelés (systemd)

**Maszkolás (Mask) és Feloldás (Unmask):**

A maszkolás erősebb a `disable`-nél: egy maszkolt szolgáltatást még manuálisan sem lehet elindítani, mivel a fájlját a `/dev/null`-hoz linkeli a rendszer.

```bash
sudo systemctl mask apache2
sudo systemctl unmask apache2
```

**Konfigurációs fájlok szerkesztése közvetlenül:**

```bash
sudo systemctl edit --full apache2
# Mentés után kötelező a frissítés:
sudo systemctl daemon-reload
```

**Függőségek kezelése (Dependency Management):**

A `.service` fájlokban a `[Unit]` szekcióban adható meg:

- `After=`: Csak az adott szolgáltatás (pl. hálózat) elindulása után induljon.
- `Requires=`: Kötelező (szigorú) függőség. Ha az itt megadott szolgáltatás leáll, ez is leáll.
- `Wants=`: Gyenge függőség (ajánlott, de nem kötelező).

## 5. Hibaelhárítás és Legjobb Gyakorlatok

**Logok és Részletes Hibák Elemzése:**

Ha egy szolgáltatás elszáll, a `status` lekérdezése mellett a `journalctl` adja a legtöbb információt.

```bash
sudo systemctl status apache2
# Részletes (kiterjesztett) rendszernaplók megtekintése a végéről:
sudo journalctl -xe
```

**Biztonsági mentés (Backup) és Automatizálás:**

A fontos konfigurációs fájlokról érdemes másolatot tartani. Ezt egy cron job segítségével automatizálhatjuk is.

```bash
sudo crontab -e
```

Add hozzá az alábbi sort, hogy minden éjjel 2:00-kor készüljön egy másolat:

```plaintext
0 2 * * * cp /etc/systemd/system/myservice.service /backup/
```

##Forrás
-[https://dev.to/iaadidev/how-to-manage-services-in-linux-systemd-and-sysvinit-essentials-devops-prerequisite-8-1jop]

# A Linux Szolgáltatások (Services) Listázása a `systemctl` paranccsal

A Linux számítógépek működése nagymértékben támaszkodik a háttérben futó feladatokra, amelyeket szolgáltatásoknak (services) vagy démonoknak (daemons) hívunk.

A modern Linux disztribúciók túlnyomó többsége (Ubuntu, Fedora, Arch stb.) a systemd rendszert használja ezen folyamatok kezelésére. A systemd a legelső folyamat (process), amely elindul a gép bekapcsolásakor, a szolgáltatások beállításait pedig ún. "unit" (egység) fájlokban tárolja.

> **Tipp:** Annak ellenőrzésére, hogy a rendszered systemd alapú-e, megnézheted a folyamatfa legtetejét a `pstree | head -5` paranccsal. Ha a `systemd` van legfelül, akkor jó helyen jársz.

## 1. Futó szolgáltatások listázása

A szolgáltatások listázására a `systemctl` parancsot használjuk. Szűrhetünk kifejezetten a szolgáltatásokra (`--type=service`) és azok állapotára (pl. `--state=running`).

**Parancs a futó szolgáltatások listázására:**

```bash
systemctl --type=service --state=running
```

(Mivel ez az alapértelmezett viselkedés, a fenti parancs egyenértékű a `systemctl list-units --type=service --state=running` paranccsal).

A kimenet egy táblázat, amely (hosszú lista esetén) a `less` lapozóban nyílik meg (kilépés a `Q` gombbal, oldalra görgetés a Jobb/Bal nyilakkal).

**Az oszlopok jelentése:**

- **Unit:** A szolgáltatás vagy démon neve (a "unit fájl" alapján).
- **Load:** A betöltési állapot (pl. loaded, not-found, bad-setting, error, masked).
- **Active:** A szolgáltatás átfogó állapota (active, reloading, inactive, failed, activating, deactivating).
- **SUB:** A részletesebb al-állapot (dead, exited, failed, inactive, running).
- **Description:** A szolgáltatás rövid leírása.

## 2. Keresés és szűrés más állapotokra

**Keresés egy adott szolgáltatásra (`grep` használatával):**

Ha csak az SSH szolgáltatásra (`sshd`) vagy kíváncsi a futó folyamatok között:

```bash
systemctl --type=service --state=running | grep ssh
```

**Sikertelen (failed) szolgáltatások listázása:**

```bash
systemctl --type=service --state=failed
```

**Több állapot egyidejű lekérdezése:**

Az állapotokat vesszővel elválasztva (szóköz nélkül!) is megadhatod. Az alábbi parancs azokat a szolgáltatásokat listázza, amelyek vagy leálltak (`exited`), vagy hibára futottak (`failed`):

```bash
systemctl --type=service --state=failed,exited
```

## 3. Telepített Unit (Egység) Fájlok Listázása

Ha nemcsak az éppen futó szolgáltatásokat szeretnéd látni, hanem a számítógépre telepített összes unit fájlt, a `list-unit-files` opciót kell használnod.

**Csak az engedélyezett (enabled - bootoláskor automatikusan induló) fájlok listázása:**

```bash
systemctl list-unit-files --state=enabled
```

**Az összes telepített unit fájl listázása (állapottól függetlenül):**

```bash
systemctl list-unit-files
```

**Csak a letiltott (disabled) vagy hibás (failed) unit fájlok listázása:**

```bash
systemctl list-unit-files --state=disabled,failed
```

## 4. Részletes információk lekérdezése egyetlen szolgáltatásról

Ha egy adott szolgáltatás (pl. az `sshd`) részletes állapotára, hibáira vagy memóriahasználatára vagy kíváncsi, a `status` opciót kell használnod.

**Parancs:**

```bash
systemctl status sshd
```

**A kimenet a következő hasznos információkat tartalmazza:**

- A szolgáltatás neve és leírása, valamint egy színkódolt pötty (zöld = fut, piros = nem fut).
- A betöltött unit fájl pontos elérési útja (path).
- Mióta fut a szolgáltatás (uptime).
- A Process ID (PID) – a futó folyamat azonosítója.
- Az egyidejűleg futó példányok száma.
- A szolgáltatás által felhasznált memória és CPU idő.
- A legutóbbi rendszernapló (log) bejegyzések, amelyek a szolgáltatáshoz tartoznak (nagyon hasznos hibaelhárításnál, ha valami nem indul el).

##Forrás:
-[https://www.howtogeek.com/839285/how-to-list-linux-services-with-systemctl/]
-[https://medium.com/@thesureshvadde/service-management-in-linux-a-comprehensive-guide-cb4c7e81dfa9]
