# Naplóelemzés Linuxon a journalctl Paranccsal (Kezdő Útmutató)

A modern Linux disztribúciók többsége a systemd-t használja. A systemd egyik fő funkciója a naplók (logok) gyűjtése és elemzése.

- **journald:** A systemd démona, amely különböző forrásokból (pl. syslog) gyűjti a naplókat, és azokat bináris formátumban tárolja.
- **journalctl:** A parancssori eszköz (CLI), amellyel olvashatjuk, szűrhetjük és elemezhetjük a journald által gyűjtött naplókat.

> 💡 **Tipp:** A legtöbb rendszerszintű napló megtekintéséhez root (sudo) jogosultság szükséges. Ha normál felhasználóként futtatod a `journalctl`-t, csak a saját felhasználói folyamataid naplóit fogod látni.

## 1. Alapvető Olvasás és Navigáció

A `journalctl` önmagában futtatva kronológiai sorrendben (legrégebbi elöl) mutatja meg a teljes naplót. A háttérben a `less` programot használja a megjelenítéshez.

**Navigáció a naplón belül (a `less` billentyűzete):**

- **Szóköz:** Egy oldal lefelé.
- **b:** Egy oldal felfelé.
- **g:** Ugrás a legelső sorra.
- **G:** Ugrás a legutolsó sorra.
- **/szöveg:** Keresés a "szöveg"-re. (`n`: következő találat, `N`: előző találat).
- **q:** Kilépés a nézetből.

**Lapozó (pager) kikapcsolása:**

Ha a kimenetet egyben akarod látni (pl. szkriptekhez), használd a `--no-pager` opciót:

```bash
journalctl --no-pager
```

**Fordított sorrend (legújabb elöl):**

```bash
journalctl -r
```

## 2. Valós Idejű Követés és Rövidítések

**Csak az utolsó N sor megjelenítése:**

(Példa: a legutóbbi 25 sor)

```bash
journalctl -n 25
```

**Valós idejű, élő követés (Real-time / Follow mode):**

(Hasonló a `tail -f` parancshoz. Kilépés: `Ctrl+C`)

```bash
journalctl -f
```

**Naplók megjelenítése UTC időzóna szerint:**

(Ha a szerver helyi időre van állítva, de te UTC-ben akarod látni)

```bash
journalctl --utc
```

## 3. A Naplók Szűrése (Filtering)

A `journalctl` legnagyobb ereje a precíz szűrési lehetőségekben rejlik. A szűrők tetszőlegesen kombinálhatók.

### A) Szűrés Szolgáltatás (Service) alapján

Ha csak egy adott systemd szolgáltatás (pl. ssh) naplóira vagy kíváncsi:

```bash
sudo journalctl -u ssh
```

### B) Szűrés Boot Munkamenet alapján

A journald minden újraindítást külön munkamenetként (session) kezel.

- Az összes boot munkamenet listázása:

  ```bash
  journalctl --list-boots
  ```

- Szűrés egy adott bootra: (A `0` az aktuális, a `-1` a legutóbbi újraindítás előtti állapot, és így tovább).

  ```bash
  journalctl -b 0    # Jelenlegi munkamenet
  journalctl -b -2   # Kettővel ezelőtti boot
  ```

### C) Szűrés Idő (Time) alapján

Használhatsz természetes nyelvi szavakat (`today`, `yesterday`, `now`), pontos dátumokat, vagy relatív időt (pl. `-1h20min`).

```bash
# Tegnaptól mostanáig
journalctl --since=yesterday --until=now

# Pontos dátumtól
journalctl --since "2020-07-10"

# Dátum és idő intervallum
journalctl --since "2020-07-10 15:10:00" --until "2020-07-12"
```

### D) Szűrés Kernel, PID, UID alapján

- Csak a kernel naplói (dmesg alternatíva): `journalctl -k`
- Szűrés Folyamatazonosítóra (PID): `journalctl _PID=1234`

(Ugyanígy szűrhetünk `_UID` vagy `_GID` mezőkre is).

### E) Szűrők Kombinálása (Példa)

Lássuk az SSH szolgáltatás tegnapi naplóit UTC idő szerint:

```bash
sudo journalctl -u ssh --since=yesterday --utc
```

## 4. Extra Információk és Hibakeresés

A fejlesztők és rendszergazdák egyik kedvenc "gyors" parancsa hibakeresésre:

```bash
journalctl -xe
```

- **`-e`:** Ugrás a napló legvégére (azaz a legfrissebb eseményekhez).
- **`-x`:** Bővített (eXtra) információk megjelenítése, ha elérhető (pl. hibaokok magyarázata és webes hivatkozások a megoldáshoz).

**Csak a Hibák (Errors) megjelenítése:**

A systemd szabványos prioritási (súlyossági) kódokat használ 0-tól (emerg) 7-ig (debug). A hiba (error) kódja a 3.

```bash
# Csak az aktuális boot (-b) hibáinak (-p 3) megjelenítése extra infóval (-x)
journalctl -p 3 -xb

# Szűrés egy adott tartományra (pl. figyelmeztetéstől az infóig: 4-6)
journalctl -p 4..6 -b0
# vagy
journalctl -p warning..info -b0
```

## 5. Lemezhasználat (Disk Usage) Ellenőrzése

A journald képes megőrizni a debug szintű naplókat is, amik idővel komoly helyet (akár több gigabájtot) foglalhatnak el a `/var/log/journal` könyvtárban.

**A naplók által elfoglalt lemezterület lekérdezése:**

```bash
journalctl --disk-usage
```

(Ha a méret túl nagy, a naplók tisztíthatók a `--vacuum-size=` vagy `--vacuum-time=` parancsokkal).

##Forrás
-[https://linuxhandbook.com/journalctl-command/]

# Rendszernaplók Kezelése Linuxon: journalctl, dmesg és /var/log

Egy modern (systemd-alapú) Linux rendszeren a naplóelemzés (log analysis) három logikai rétegre oszlik. A szolgáltatások hibáinál a `journalctl` a kiindulópont, a hardveres vagy kernel szintű problémáknál a `dmesg`, míg a hosszú távú biztonsági auditokhoz a hagyományos `/var/log` fájlok használatosak.

## 1. A Modern Szabvány: journalctl (Systemd Journal)

A `systemd-journald` bináris, strukturált formátumban tárolja a naplókat, ami villámgyors és precíz szűrést tesz lehetővé, a metaadatok (UID, GID, környezet) megőrzésével.

**Legfontosabb parancsok és szűrők:**

- **Szűrés szolgáltatás (Unit) alapján (`-u`):**

  A leggyakoribb feladat az incidenskezelésben a zaj kiszűrése.

  ```bash
  journalctl -u ssh.service -n 5   # Az SSH szolgáltatás utolsó 5 sora
  ```

- **Valós idejű követés és részletes magyarázat (`-f` és `-xe`):**

  Élő hibakereséshez és a hibaok (root cause) részletes kifejtéséhez:

  ```bash
  journalctl -u nginx.service -f
  journalctl -xe
  ```

- **Szűrés Időablak alapján (`--since`, `--until`):**

  A "log-fáradtság" elkerülése érdekében pontosítsd az időt:

  ```bash
  journalctl --since "1 hour ago"
  journalctl --since "2026-05-03 12:00:00" --until "2026-05-03 13:00:00"
  ```

- **Szűrés Rendszerindítás (Boot) alapján (`-b`):**

  Egy újraindítás után visszanézheted, mi okozta a fagyást a korábbi munkamenetben:

  ```bash
  journalctl -b -1  # Az előző boot naplói
  ```

- **Perzisztencia (Állandóság):** Néhány disztribúció (pl. Debian) alapból illékony (`/run/log/journal/`) tárolót használ. Éles környezetben a `/var/log/journal/` mappa létrehozásával vagy a `journald.conf` módosításával kötelező bekapcsolni az állandó (persistent) naplózást, hogy a logok túléljék az újraindítást.

## 2. Kernel Diagnosztika: A dmesg (Ring Buffer)

Ha a probléma hardveres vagy egy kernelmodult érint, a `journalctl` nem mindig látja a teljes képet. A `dmesg` a kernel gyűrűs pufferét (ring buffer) olvassa, rögzítve a boot folyamat legkorábbi szakaszait és a hardveres megszakításokat (interrupts).

**Mikor használjuk?** I/O hibák, memóriahibák (OOM killer), meghajtó (driver) inicializálási hibák, SCSI/NVMe hibakódok keresésére.

**Példa a lemez-események szűrésére:**

```bash
dmesg | grep -i "sda" | tail -n 5
```

> **Pro-Tipp ("Tainted Kernel"):** Ha a `dmesg` kimenetében a "Tainted" (Szennyezett) szót látod, az azt jelenti, hogy a kernel egy zárt forráskódú (proprietary) drivert töltött be, vagy nem támogatott műveletet hajtott végre. Ez jelentősen megnehezítheti a hivatalos hibakeresést.

## 3. A Hagyományos Réteg: A /var/log Könyvtár

Bár a systemd a domináns, sok szolgáltatás továbbra is egyszerű szöveges fájlokba ír a `/var/log` alatt (általában az `rsyslog` démon segítségével). Ez biztosítja a hosszú távú, ember által olvasható archívumot.

**A legfontosabb auditálandó fájlok:**

- **`auth.log`** (vagy RedHat rendszereken `secure`): A biztonsági auditok központja. Nyomon követi a bejelentkezési kísérleteket és a `sudo` használatot. Ide kell nézni brute-force támadás gyanúja esetén.

- **`syslog`** (vagy `messages`): A "mindent bele" napló. Olyan alkalmazáshibák kerülnek ide, amiket nem irányítottak dedikált fájlba.

- **`kern.log`:** A kernelüzenetek szöveges tükörképe. Szemben a `dmesg`-gel (ami egy idő után felülírja a régi adatokat), a `kern.log` a fájlrotáció révén hosszú távon is megőrzi a történetet.

## 4. Naplórotáció és Tárhelykezelés (logrotate)

A nem menedzselt naplók a szerverek lemezmegtelésének első számú okozói.

**Két szinten kell védekezni:**

1. **Systemd Journal:** A `journald.conf` fájlban be kell állítani a `SystemMaxUse` értéket (pl. a tárhely maximum 10%-a).

2. **Szöveges logok (logrotate):** Az `/etc/logrotate.d/` alatti konfigurációk felelnek a régi szöveges logok tömörítéséért és törléséért.

   - Éles, nagy forgalmú szervereken a napi rotáció helyett méretalapú (size-based) rotációt érdemes használni, hogy egy hirtelen hibaáradat ne töltse meg a lemezt órák alatt.
   - Használd a `compress` és `delaycompress` opciókat, hogy a legutóbbi log azonnal olvasható maradjon, de a régebbiek (akár 90%-kal kisebb) gzip formátumba kerüljenek.

## 5. Példa: A Rendszermérnöki Hibaelhárítási Munkafolyamat

Amikor egy webszerver (pl. Nginx vagy Apache) nem indul el, a professzionális megközelítés lépcsőzetes:

1. **Állapotellenőrzés:** `systemctl status nginx` (Ha a kimenet csonkolt, lépj a következőre).

2. **Környezeti hiba keresése:** `journalctl -u nginx.service --since "5 minutes ago"` (Ez általában azonnal megmutatja a parser hibát okozó elgépelést a konfigurációs fájlban).

3. **Terhelés alatti összeomlás vizsgálata:** Ha a szolgáltatás elindul, de terhelés alatt leáll, irány a `dmesg`. Keresd az OOM (Out-Of-Memory) killer bejegyzéseit, amelyek azt mutatják, hogy a kernel lőtte ki a folyamatot memóriahiány miatt.

4. ##Forrás
5. -[https://www.fosslinux.com/8984/how-to-check-system-logs-on-linux-complete-usage-guide.htm]

6. # A Linux Kernel Napló Kezelése: dmesg

A `dmesg` (diagnostic messages) a Linux rendszerek egyik legfontosabb hibaelhárító eszköze. Segítségével a kernel üzenetpufferét (ring buffer) olvashatjuk ki. Minden olyan esemény, ami a kernellel vagy a hardverrel kapcsolatos (pl. új lemez csatlakoztatása, rendszer alvó állapotba helyezése, illesztőprogramok betöltése), itt kerül rögzítésre.

(A napló olvasásához alapértelmezésben nincsen szükség rendszergazdai/root jogosultságra).

## 1. Alapvető Használat és Navigáció

A parancs önmagában futtatva kilistázza a kernel teljes naplóját, időrendi sorrendben:

```bash
dmesg
```

**Időbélyegek az alapértelmezett nézetben:**

Az alapértelmezett kimenetben minden sor elején egy szám található szögletes zárójelek között (pl. `[64283.356724]`). Ez a rendszer (pontosabban a kernel) elindulása óta eltelt időt mutatja másodperc.ezredmásodperc formátumban.

**Lapozás a kimenetben:**

Mivel a `dmesg` kimenete rendkívül hosszú lehet, érdemes lapozó (pager) programba irányítani a kimenetet, hogy kényelmesen görgethessünk benne:

```bash
dmesg | more
```

## 2. Ember által olvasható Időbélyegek (Timestamps)

A boot óta eltelt másodpercek helyett sokkal praktikusabb hagyományos dátum- és időformátumot kérni. Ezt az ISO formátum kikényszerítésével tehetjük meg:

```bash
dmesg --time-format=iso
```

**Példa a kimenetre:**

```plaintext
2021-03-14T13:49:21,917805+11:00 sd 3:0:0:0: [sda] Starting disk
```

## 3. Milyen információkat találunk a naplóban?

A `dmesg` naplóban a bejegyzéseket gyakran egy "címke" (tag) előzi meg, amely jelzi, hogy melyik kernelmodul vagy illesztőprogram (driver) küldte az üzenetet. Néhány tipikus esemény, amit itt érdemes keresni:

- **Hardvereszközök csatlakoztatása / Felismerése:**

  ```plaintext
  [ 2.006753] usb 1-6: Product: USB Optical Mouse
  ```

  (Hasznos, ha meg akarjuk tudni, hogy a gép felismerte-e az imént bedugott pendrive-ot vagy egeret).

- **Energiagazdálkodás (Alvó állapot / Ébredés):**

  ```plaintext
  [97931.332991] ACPI: Preparing to enter system sleep state S3
  [97931.746603] ACPI: Low-level resume complete
  ```

- **CPU és Processzor állapotok:**

  ```plaintext
  [97931.707220] smpboot: CPU 1 is now offline
  ```

- **Illesztőprogramok (Drivers) üzenetei és hibái:**

  Hang- és videókártyák, hálózati csatolók inicializálása (pl. `snd_hda_intel`).

- **Kritikus rendszerhibák:**

  Lefagyott folyamatok (pl. `INFO: task blocked for more than 120 seconds`).

  ##Forrás
  -[https://linuxconfig.org/what-is-dmesg-and-how-do-i-use-it]
