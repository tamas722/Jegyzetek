# A Linux Folyamat Szignálok (Process Signals) Megértése

A folyamatszignálok a Linux rendszerkezelés alapkövei. Lehetővé teszik a felhasználók és az operációs rendszer számára, hogy kommunikáljanak a futó folyamatokkal.

## 1. Mik azok a Folyamat Szignálok?

A folyamatszignálok szoftveres megszakítások (software interrupts), amelyek aszinkron események kezelésére szolgálnak. Úgy is felfoghatjuk őket, mint üzeneteket, amelyeket a folyamatok kapnak, hogy értesüljenek egy adott eseményről, vagy végrehajtsanak valamilyen kérést (például álljanak le).

## 2. A Leggyakoribb Linux Szignálok

### SIGTERM (Szignál 15)

A `SIGTERM` az udvarias (graceful) módja egy folyamat leállításának. Amikor egy folyamat megkapja ezt a szignált:

- Van ideje felszabadítani az erőforrásokat (cleanup).
- El tudja menteni az adatokat, ha szükséges.
- Rendezett, finom leállást hajt végre.

```bash
kill -15 <PID>
# (A kill parancs alapértelmezésben is a 15-ös szignált küldi, ha nem adunk meg mást).
```

### SIGKILL (Szignál 9)

A `SIGKILL` az erőszakos leállítási szignál:

- A folyamatok nem hagyhatják figyelmen kívül.
- Azonnal kilövi a folyamatot.
- Nem engedélyezi a takarítási és mentési műveleteket (adatvesztéshez vezethet).

```bash
kill -9 <PID>
```

### SIGHUP (Szignál 1)

Hagyományosan a terminálkapcsolat megszakadásának jelzésére szolgált (hangup), de ma leggyakrabban a következőkre használják:

- Konfigurációs fájlok újraolvasása (reload) anélkül, hogy a folyamatot teljesen le kellene állítani.
- Folyamatok újraindítása / alkalmazások újra-inicializálása.

```bash
kill -1 <PID>
```

## 3. Szignálok küldése és kezelése

A szignálok küldésének elsődleges eszköze a `kill` parancs.

**Alapvető szignálküldés:**

```bash
# SIGTERM (15) küldése az 1234-es PID-ű folyamatnak
kill 1234

# SIGHUP (1) küldése az 1234-es folyamatnak
kill -1 1234
```

**Minden elérhető szignál kilistázása a rendszeren:**

```bash
kill -l
```

**Folyamat leállítása név és felhasználó alapján (`killall`):**

```bash
# Az adott felhasználóhoz tartozó, meghatározott nevű folyamat összes példányának leállítása
killall -u username process_name
```

**Alkalmazások kezelése név alapján (`pkill`):**

```bash
# A Firefox finom leállítása
pkill -15 firefox

# A Firefox erőszakos leállítása (ha lefagyott és nem válaszol)
pkill -9 firefox
```

## 4. Bevált Gyakorlatok (Best Practices)

**1. Mindig használd először a SIGTERM-et!**

A `SIGKILL` (9) alkalmazása előtt mindig adj esélyt a programnak a normális leállásra:

```bash
# Első próbálkozás (SIGTERM)
kill <PID>

# Ha a folyamat pár másodperc múlva is makacsul fut
kill -9 <PID>
```

**2. Szolgáltatások konfigurációjának újraolvasása (Reload):**

Ahelyett, hogy leállítanál egy webszervert, küldj neki `SIGHUP` jelet a beállítások frissítéséhez:

```bash
# Keresd meg az Nginx PID-jét, és küldj neki 1-es szignált
kill -1 $(pidof nginx)
```

**3. Szignálok küldése Folyamatcsoportoknak (Process Groups):**

```bash
# Szignál küldése egy teljes folyamatcsoportnak (PGID)
kill -TERM -<PGID>
```

**4. Szignálok monitorozása (`strace`):**

Nyomon követheted, hogy egy adott folyamat milyen szignálokat kap:

```bash
strace -e signal -p <PID>
```

## 5. Hibaelhárítás (Troubleshooting)

**Zombi Folyamatok (Zombie Processes):**

A zombi folyamatok akkor jönnek létre, ha a szülő folyamat nem kezeli megfelelően a gyermekfolyamat leállását (nem olvassa ki az exit kódját). Ezeket nem lehet `kill -9`-cel megölni, mert már halottak. A megoldás a szülő folyamat leállítása vagy újraindítása.

```bash
# Zombi folyamatok azonosítása (A 'Z' állapot keresése)
ps aux | grep 'Z'

# A zombi folyamat szülőjének (PPID) megkeresése
ps -o ppid= -p <ZOMBIE_PID>
```

**Jogosultsági hibák:**

Csak a saját folyamataidnak küldhetsz szignált. Ha egy folyamat másé (pl. root), jogosultsági hibát kapsz.

```bash
# Ellenőrizd, ki a folyamat tulajdonosa
ps -o user= -p <PID>

# Használj sudo-t a privilegizált folyamatok leállításához
sudo kill <PID>
```

## 6. Haladó Szignál Koncepciók

**Valós idejű szignálok (Real-time Signals):**

A Linux biztosít valós idejű szignálokat (`SIGRTMIN`-től `SIGRTMAX`-ig), amelyeket az alkalmazások specifikus, egyedi célokra használhatnak.

```bash
# Valós idejű szignál küldése
kill -RTMIN+1 <PID>
```

**Szignál Maszkolás (Signal Masking):**

A folyamatok képesek ideiglenesen blokkolni (maszkolni) bizonyos szignálokat, hogy egy kritikus művelet közben ne szakítsák meg őket.

```bash
# Egy folyamat által blokkolt szignálok lekérdezése
cat /proc/<PID>/status | grep SigBlk
```

##Forrás
-[https://www.ceos3c.com/linux/understanding-linux-process-signals-a-complete/]

# A Linux Folyamat Szignálok (Process Signals) és Jelentésük

A Linux szignálokat használ a folyamatokkal való interakcióhoz és azok állapotának meghatározásához. A rendszer POSIX megbízható és valós idejű (real-time) szignálokat alkalmaz, melyek közül az előbbieket tekintjük a "szabványos" (standard) szignáloknak.

Számos program a glibc könyvtárra épül, és olyan függvényeket használ – mint például a `kill(2)` –, hogy szignált küldjön egy folyamatnak, egy folyamatcsoportnak, vagy akár a rendszer összes folyamatának. Egy folyamat (a programozásától függően) eldöntheti, hogy figyelmen kívül hagyja (ignore) a szignált, vagy egy úgynevezett szignálkezelő (signal handler) rutin segítségével valamilyen specifikus műveletet hajt végre a beérkezésekor.

## Szignálok Referenciatáblázata

Az alábbi táblázat tartalmazza a legfontosabb szabványos Linux szignálokat, azok numerikus azonosítóját és rövid leírását:

| Szignál neve | Érték | Leírás és Funkció |
| --- | --- | --- |
| `SIGHUP` | 1 | A felhasználó terminálkapcsolata megszakadt (hangup). Sok háttérfolyamat (démon) ezt a szignált használja a konfigurációs fájlok újraolvasására. |
| `SIGINT` | 2 | Megszakítás (Interrupt). Gyakran a `CTRL+C` lenyomásával generálódik. Általában a rendszererőforrások (pl. ideiglenes fájlok) megfelelő takarításával jár. |
| `SIGQUIT` | 3 | Hasonló a `SIGINT`-hez, de általában a `CTRL+\` lenyomására jön létre, és sokszor nem végez takarítást kilépés előtt. |
| `SIGILL` | 4 | Érvénytelen (illegal) utasítás. A folyamat "szemetet" próbált futtatni, vagy privilegizált utasítást kísérelt meg végrehajtani. |
| `SIGTRAP` | 5 | Hibakeresők (debuggerek) által használt szignál (töréspontokhoz). |
| `SIGABRT` | 6 | A folyamat meghívta az `abort()` függvényt (szándékos, programozott összeomlás). |
| `SIGIOT` | 6 | Eredetileg a PDP-11 "iot" utasítása generálta, Linuxon a `SIGABRT` szignálnak felel meg. |
| `SIGBUS` | 7 | Hasonló a szegmentálási hibához, de fizikai, érvénytelen memóriacím használatakor lép fel (Bus error). |
| `SIGEMT` | - | Emulátor csapda, bizonyos nem implementált utasítások végrehajtásakor. |
| `SIGFPE` | 8 | Lebegőpontos kivétel (Floating-point exception), és egyéb végzetes aritmetikai hibák (pl. nullával osztás vagy túlcsordulás). |
| `SIGKILL` | 9 | Egy folyamat erőszakos, azonnali leállítása. Szigorúbb a `SIGTERM`-nél, a folyamat nem tudja lekezelni vagy figyelmen kívül hagyni. |
| `SIGUSR1` | 10 | Fejlesztők számára fenntartott szignál egyedi funkciók (akciók) meghatározására. |
| `SIGSEGV` | 11 | Szegmentálási hiba (Segmentation fault) / hozzáférési hiba. Akkor történik, ha a program érvénytelen (nem hozzá tartozó) memóriaterületet próbál elérni. |
| `SIGUSR2` | 12 | Hasonló a `SIGUSR1`-hez, egy második fejlesztői szignál. |
| `SIGPIPE` | 13 | Megszakadt csővezeték (Broken pipe), pipe-okkal és FIFO speciális fájlokkal kapcsolatos hiba. |
| `SIGALRM` | 14 | Valós idejű (vagy óra) időzítő lejárta (pl. az `alarm()` függvény használatakor). |
| `SIGTERM` | 15 | Leállítási kérelem (Terminate). Ez az alapértelmezett módja egy folyamat finom, szabályos leállításának. |
| `SIGSTKFLT` | 16 | Veremhiba (Stack fault). Akkor küldik a folyamatnak, ha veremtúlcsordulás (overflow) vagy alulcsordulás (underflow) történik. |
| `SIGCHLD` | 17 | A szülő folyamatnak küldött jel, amikor a gyermekfolyamata leáll. |
| `SIGCLD` | - | Elavult jelzés, a `SIGCHLD` váltotta fel. |
| `SIGCONT` | 18 | Egy korábban megállított (felfüggesztett) folyamat folytatására szolgáló szignál. |
| `SIGSTOP` | 19 | Folyamat azonnali megállítása (felfüggesztése). A folyamat nem tudja lekezelni és nem hagyhatja figyelmen kívül. |
| `SIGTSTP` | 20 | Interaktív megállítási kérelem (pl. `CTRL+Z` a terminálban). A folyamat képes figyelmen kívül hagyni, ellentétben a `SIGSTOP`-pal. |
| `SIGTTIN` | 21 | Akkor jön létre, ha egy háttérben futó folyamat olvasni próbál a terminálból (ami nem lehetséges). |
| `SIGTTOU` | 22 | Hasonló a `SIGTTIN`-hez, de akkor történik, ha a háttérfolyamat kimenetet próbál írni a terminálra. |
| `SIGURG` | 23 | Sürgős szignál "sávonkívüli" (out-of-band) hálózati adatokhoz, speciális kezelést igényel. |
| `SIGXCPU` | 24 | A folyamat túllépte a számára engedélyezett CPU időkorlátot. |
| `SIGXFSZ` | 25 | A folyamat túllépte a számára engedélyezett maximális fájlméretet (soft limit). |
| `SIGVTALRM` | 26 | Virtuális időzítő riasztása; az aktuális folyamat által felhasznált CPU időzítő lejárta. |
| `SIGPROF` | 27 | Kód profilozáshoz (profiling) használt szignál (a folyamat és a rendszer CPU idejének mérésére). |
| `SIGWINCH` | 28 | Az (aktív) ablak átméretezésekor küldött eseményjelző. |
| `SIGIO` | 29 | Tájékoztató szignál arról, hogy egy fájlleíró (file descriptor) készen áll pl. egy bemeneti/kimeneti (I/O) műveletre. |
| `SIGPOLL` | - | System V szignál, nagyon hasonló a `SIGIO`-hoz. |
| `SIGPWR` | 30 | Kizárólag az init (elsődleges) folyamat használja, tipikusan hardverprobléma (pl. áramszünet) jelzésére. |
| `SIGINFO` | - | Információkérés (let the process share information); a folyamat megoszthatja vele a státuszát. |
| `SIGLOST` | - | Erőforrás elvesztése (pl. zárolás/lock elvesztése egy NFS fájlrendszeren). |
| `SIGSYS` | 31 | Érvénytelen (rossz) paraméterek átadása egy rendszerhívásnak (syscall). |
| `SIGUNUSED` | 31 | Nem használt |


##Forrás
-[https://linux-audit.com/processes/linux-process-signals/]
