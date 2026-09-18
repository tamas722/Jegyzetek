# Systemd Szolgáltatás (Service) Létrehozása Linuxon

A legtöbb modern Linux disztribúció (Ubuntu, Fedora, RHEL, Debian, Arch, stb.) a systemd-t használja alapértelmezett init rendszerként. Egy szolgáltatás (service) egy olyan háttérfolyamat, amely automatikusan elindul vagy leáll bizonyos körülmények (pl. rendszerindítás, hálózat csatlakozása) hatására.

A systemd szolgáltatásokat egy speciális formátumú konfigurációs fájllal (ún. unit fájl) határozzuk meg, amelynek a kiterjesztése `.service`.

## 1. A Systemd Szolgáltatásfájl Szerkezete

Minden `.service` fájl három kötelező és logikailag elkülönített szekcióból áll. A megjegyzéseket `#` (hash) jellel lehet jelölni.

### A `[Unit]` szekció

Ez a szekció tartalmazza a szolgáltatás leírását és függőségeit (mikor és mihez képest induljon).

- `Description=`: A szolgáltatás ember által olvasható címe / leírása.
- `After=`: Megadja, hogy a szolgáltatás melyik másik szolgáltatás(ok) vagy cél(ok) (target) sikeres elindulása után indulhat (pl. `After=network.target`).
- `Before=`: Megadja, hogy a szolgáltatásnak melyik másik szolgáltatás előtt kell elindulnia.

### A `[Service]` szekció

Ez határozza meg a tényleges végrehajtást és a folyamat viselkedését.

- `ExecStart=`: A parancs vagy szkript abszolút útvonala, amelyet a szolgáltatás indításakor futtatni kell.
- `ExecReload=`: (Opcionális) A parancs a szolgáltatás újraolvasásához (reload). Főleg adatbázisoknál hasznos, ahol jobb finoman újraolvasni a konfigurációt, mint erőszakosan újraindítani a folyamatot.
- `Type=`: A folyamat indulási típusa. Lehet `simple`, `forking`, `oneshot` (egyszer lefutó szkriptekhez), `notify`, `idle` stb.
- `Restart=`: Meghatározza, hogy mikor kell újraindítani a szolgáltatást. Lehetőségek: `no`, `on-success`, `on-failure`, `always`, stb.

### Az `[Install]` szekció

Ez a szekció kezeli a szolgáltatás engedélyezését (enable / disable), azaz azt, hogy milyen rendszerállapotba települjön be.

- `WantedBy=`: Hasonló a klasszikus "runlevel"-ekhez. Általában a `multi-user.target` (többfelhasználós mód, hálózattal) vagy a `default.target` (amikor a felhasználó már bejelentkezhet) a leggyakoribb.
- `RequiredBy=`: Erős függőséget jelent. Ha az itt megadott cél/szolgáltatás elindul, ez a szolgáltatás kötelezően elindul; ha ez elbukik, a cél is elbukik.

## 2. Példa 1: Systemd Szolgáltatás ROOT felhasználónak

Ha a szolgáltatás futtatásához és engedélyezéséhez rendszergazdai (root) jogok kellenek (pl. a rendszer indulásakor lefutó szkript).

**Fájl helye:** A root jogosultságot igénylő egyedi unit fájlokat mindig az `/etc/systemd/system/` könyvtárba kell tenni.

1. Hozzuk létre a fájlt: `/etc/systemd/system/update-on-boot.service`

   ```ini
   [Unit]
   Description=Csomaglistak frissitese inditaskor
   After=multi-user.target

   [Service]
   # A szkript futtatása (a bash és a szkript abszolút útvonalával)
   ExecStart=/usr/bin/bash /root/.scripts/sys-update.sh
   Type=simple

   [Install]
   WantedBy=multi-user.target
   ```

2. A systemd konfiguráció újraolvasása (Kötelező minden új fájl vagy módosítás után!):

   ```bash
   sudo systemctl daemon-reload
   ```

3. A szolgáltatás engedélyezése (Automatikus indulás beállítása):

   ```bash
   sudo systemctl enable update-on-boot.service
   ```

4. Státusz ellenőrzése (hogy engedélyezve van-e):

   ```bash
   sudo systemctl is-enabled update-on-boot.service
   # Elvárt kimenet: enabled
   ```

## 3. Példa 2: Systemd Szolgáltatás NORMÁL felhasználónak

A systemd képes normál felhasználók saját (user-level) szolgáltatásait is futtatni. Ilyenkor nincs szükség sudo jogosultságokra, de más könyvtárat és parancsokat kell használni.

**Fájl helye:** A felhasználói unit fájlok a `~/.config/systemd/user/` könyvtárba kerülnek. (Ha nem létezik, hozd létre a `mkdir -p ~/.config/systemd/user` paranccsal).

1. Hozzuk létre a fájlt: `~/.config/systemd/user/scoreboard.service`

   (Példa: Egy szkript, amely a rendszer leállítása előtt lementi az uptime-ot).

   ```ini
   [Unit]
   Description=Log uptime in scoreboard
   DefaultDependencies=no
   Before=shutdown.target

   [Service]
   Type=oneshot
   ExecStart=/usr/bin/bash /home/pratham/.scripts/big-uptime.sh
   TimeoutStartSec=0

   [Install]
   WantedBy=shutdown.target
   ```

2. A systemd konfiguráció újraolvasása (Felhasználói módban a `--user` kapcsolóval):

   ```bash
   systemctl --user daemon-reload
   ```

3. A szolgáltatás engedélyezése:

   ```bash
   systemctl --user enable scoreboard.service
   ```

4. Státusz ellenőrzése:

   ```bash
   systemctl --user is-enabled scoreboard.service
   # Elvárt kimenet: enabled
   ```

##Forrás
-[https://linuxhandbook.com/create-systemd-services/]

# Professzionális Systemd Szolgáltatás Készítése Linuxon (2026-os Protokoll)

A systemd sokkal több, mint egy egyszerű "init" rendszer: egy egységes keretrendszer a folyamatok, naplók és rendszerállapotok kezelésére. Egy alapvető unit fájl megírása egyszerű, de egy production-grade (éles környezetbe szánt) szolgáltatás megköveteli, hogy tisztában legyünk a függőségekkel, a környezeti változókkal, a biztonsági homokozóval (sandboxing) és a hibaezrelékekkel.

## 1. A "Restart Trap" (Újraindítási Csapda) és a Megoldás

Ha egy szolgáltatás hiba miatt leáll, gyakran beállítják a `Restart=always` opciót. Ha azonban elfelejtünk egy késleltetést (`RestartSec=30`), és a szolgáltatás azonnal összeomlik, a systemd másodpercenként akár ezer alkalommal is megpróbálhatja újraindítani. Ez 100%-ra terhelheti a CPU-t, és órák alatt gigabájtosra duzzaszthatja a logfájlokat. **Mindig használj `RestartSec`-et!**

## 2. Gyakorlati Példa: Ideiglenes Fájlok Auditálása

### 1. lépés: A logikai szkript elkészítése

Hozzunk létre egy Bash szkriptet, amely kitakarítja a 7 napnál régebbi fájlokat a `/tmp` könyvtárból:

```bash
sudo nano /usr/local/bin/temp-audit.sh
```

A szkript tartalma:

```bash
#!/bin/bash
echo "Initiating Temp Audit Protocol..."
find /tmp -type f -mtime +7 -delete
echo "Cleanup complete at $(date)"
```

Tegyük futtathatóvá:

```bash
sudo chmod +x /usr/local/bin/temp-audit.sh
```

### 2. lépés: A Unit Fájl Megírása

A rendszer szintű unit fájlokat az `/etc/systemd/system/` könyvtárban tároljuk:

```bash
sudo nano /etc/systemd/system/temp-audit.service
```

**Production-grade konfiguráció:**

```ini
[Unit]
Description=FOSSLinux Temporary File Audit Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/temp-audit.sh
Restart=on-failure
RestartSec=30
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

- `After=network.target`: Biztosítja, hogy a hálózati réteg felálljon mielőtt a szolgáltatás elindul.
- `RestartSec=30`: Ez a védőháló (anti-DoS), ami 30 másodpercet vár az újraindítási kísérlet előtt hiba esetén.

### 3. lépés: Telepítés, Újratöltés és Indítás

Minden unit fájl létrehozása vagy módosítása után kötelező frissíteni a systemd-t:

```bash
sudo systemctl daemon-reload
sudo systemctl start temp-audit.service
```

Státusz ellenőrzése:

```bash
systemctl status temp-audit.service
```

### 4. lépés: Véglegesítés (Autostart)

Hogy a szolgáltatás minden rendszerindításkor fusson:

```bash
sudo systemctl enable temp-audit.service
```

### 5. lépés: Hibaelhárítás a Journal segítségével

A systemd minden kimenetet (STDOUT/STDERR) automatikusan a journalba irányít:

```bash
journalctl -u temp-audit.service --no-hostname --since "5 min ago"
```

## 3. A Környezeti Változók Csapdája (Environment vs. EnvironmentFile)

A systemd szolgáltatások egy "tiszta" (clean) környezetben futnak. Nem öröklik meg a felhasználói `.bashrc`-ből a `PATH`, `AWS_REGION` vagy egyéb változókat.

- **Egyedi változókhoz:** Használd a `Environment=` direktívát a `[Service]` szekcióban:

  ```ini
  Environment="PATH=/usr/local/bin:/usr/bin:/bin"
  Environment="DB_PORT=5432"
  ```

- **Titkokhoz és tömeges változókhoz:** Használd az `EnvironmentFile=` opciót, amely egy külső `.env` fájlra mutat (pl. `/etc/default/` vagy `/etc/sysconfig/`), így a konfiguráció és a kód tiszta marad.

## 4. Biztonsági Izoláció és Homokozó (Hardening)

Éles környezetben soha ne engedjünk egy szolgáltatásnak teljes hozzáférést a rendszerhez, ha arra nincs szüksége. A beépített direktívák kumulatív (összeadódó) módon szűkítik a szolgáltatás világát:

- `ProtectHome=yes`: Láthatatlanná teszi a `/home`, `/root` és `/run/user` könyvtárakat a folyamat számára.
- `ProtectSystem=strict`: Read-only (csak olvasható) módon csatolja fel az egész fájlrendszert, kivéve az explicit módon engedélyezett helyeket (pl. `/tmp`).
- `PrivateTmp=yes`: Izolált `/tmp` könyvtárat biztosít a szolgáltatásnak, megakadályozva a szimlinkes támadásokat.

##Forrás
-[https://www.fosslinux.com/111815/a-guide-to-creating-linux-services-with-systemd.htm]
