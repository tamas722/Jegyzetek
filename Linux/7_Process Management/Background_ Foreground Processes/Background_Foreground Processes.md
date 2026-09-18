# Linux Parancsok Futtatása a Háttérben és az Előtérben

Amikor egy hosszan futó feladatot vagy parancsot indítasz, nem feltétlenül érdemes megvárni, amíg befejeződik és addig blokkolni a terminált. A Linux lehetővé teszi, hogy a folyamatokat a háttérbe (background) küldd, így a terminál szabad marad más feladatok elvégzésére. Ezeket a folyamatokat később bármikor visszahozhatod az előtérbe (foreground).

## 1. Parancs indítása közvetlenül a háttérben (`&`)

Ha már előre tudod, hogy egy parancs sokáig fog futni, a legegyszerűbb, ha eleve a háttérben indítod el. Ehhez mindössze egy és jelet (`&`) kell tenni a parancs végére.

**Szintaxis:**

```bash
your_command &
```

**Példa:** A `sleep` (várakozás) parancs háttérbe küldése 60 másodpercre:

```bash
sleep 60 &
```

Amikor a háttérben futó parancs befejeződik, a terminál egy ehhez hasonló üzenettel jelez:

```plaintext
[1]+  Done                    sleep 60
```

## 2. Már futó folyamat háttérbe küldése (`Ctrl+Z` és `bg`)

Ha elindítottál egy programot, és csak utána jössz rá, hogy a háttérben kellett volna futtatni, semmi gond. Egy már futó folyamatot is a háttérbe lehet küldeni.

1. Először nyomd meg a `Ctrl+Z` billentyűkombinációt. Ez felfüggeszti (suspend) a futó folyamatot.
2. Ezután add ki a `bg` (background) parancsot. Ez a felfüggesztett folyamatot elindítja a háttérben.

**Példa folyamat:**

```bash
sleep 60
^Z
[1]+  Stopped                 sleep 60

bg
[1]+ sleep 60 &
```

## 3. A háttérben futó folyamatok listázása (`jobs`)

Ha tudni szeretnéd, hogy milyen parancsok futnak éppen a háttérben, a `jobs` parancsot kell használnod.

Indítsunk el néhány programot a háttérben:

```bash
firefox &
gedit &
vim &
```

Most listázzuk ki őket:

```bash
jobs
```

**Kimenet:**

```plaintext
[1]   Running                 firefox &
[2]-  Running                 gedit &
[3]+  Stopped                 vim
```

**A kimenet értelmezése:**

- `[1]`, `[2]`, `[3]`: Ezek a feladatazonosítók (job ID-k). Ezekre hivatkozva tudod kezelni az adott folyamatot.
- `+` jel: Az utoljára elindított vagy előtérbe hozott feladatot (aktuális job) jelöli.
- `-` jel: Az utolsó előtti feladatot jelöli.

## 4. Folyamat visszahozása az előtérbe (`fg`)

A háttérbe küldéshez a `bg` parancsot használtuk, az előtérbe (foreground) hozáshoz pedig az `fg` parancsot kell alkalmazni.

**A legutóbbi feladat előtérbe hozása:**

Ha csak simán kiadod az `fg` parancsot, az a `jobs` listában `+` jellel megjelölt (legutolsó) feladatot hozza vissza a terminálba.

```bash
fg
```

**Konkrét feladat előtérbe hozása a Job ID alapján:**

Ha egy specifikus feladatot szeretnél visszahozni, add meg a feladatazonosítót (Job ID) az `fg` parancs után:

```bash
fg 1
```

(Ez a parancs az `[1]`-es azonosítójú folyamatot – a mi példánkban a Firefoxot – hozza az előtérbe).

## Bónusz Tipp: A `nohup` parancs

Az `&` jellel háttérbe küldött folyamatok leállnak, ha kilépsz a terminálból vagy megszakad az SSH kapcsolatod. Ha azt szeretnéd, hogy a folyamat a kijelentkezés után is tovább fusson a háttérben, használd a `nohup` (no hangup) parancsot a parancsod előtt:

```bash
nohup your_command &
```

##Forrás
-[https://linuxhandbook.com/run-process-background/]
# Linux Folyamatok Kezelése: Előtér (Foreground) és Háttér (Background)

Rendszergazdaként vagy Linux felhasználóként gyakran előfordul, hogy egy hosszú ideig futó parancsot vagy szkriptet futtatsz. Ha nem akarod, hogy ez a folyamat blokkolja a terminálodat, vagy ha szeretnéd biztonságosan bezárni az SSH munkamenetet anélkül, hogy a futó feladat megszakadna, a folyamatokat a háttérbe (background) kell küldened.

## 1. Már futó parancs felfüggesztése és háttérbe küldése (`Ctrl+Z`, `bg`, `fg`)

Ha már elindítottál egy parancsot, amely lefoglalja a terminált, a megszakítása nélkül is a háttérbe küldheted.

1. **A folyamat felfüggesztése:** Nyomd meg a `Ctrl+Z` billentyűkombinációt. Ez megállítja (Stopped) a folyamatot.

   ```bash
   $ sleep 10000
   ^Z
   [1]+  Stopped                 sleep 10000
   ```

2. **A feladatok (jobs) listázása:** A `jobs` parancs kilistázza az aktuális shell háttérben lévő vagy felfüggesztett folyamatait (az `-l` opció a PID-t is mutatja).

   ```bash
   $ jobs -l
   [1]+  1650 Stopped                 sleep 10000
   ```

3. **A folyamat folytatása a háttérben (`bg`):** A `bg` paranccsal a felfüggesztett feladatot elindíthatod a háttérben. Ha több feladatod van, a `%` jellel hivatkozhatsz a Job ID-ra.

   ```bash
   $ bg %1
   [1]+ sleep 10000 &
   ```

   Most a `jobs -l` kiírja, hogy az állapota már "Running" (Fut).

4. **A folyamat visszahozása az előtérbe (`fg`):** Az `fg` paranccsal (és a Job ID-val) a háttérben futó vagy felfüggesztett feladatot újra az előtérbe (foreground) hozhatod.

   ```bash
   $ fg %1
   ```

## 2. Parancs indítása eleve a háttérben (`&`)

Ha már az elején tudod, hogy a parancs a háttérben kell, hogy fusson, egyszerűen tegyél egy és jelet (`&`) a parancs végére:

```bash
$ sleep 10000 &
[1] 1900
```

Ezzel a folyamat azonnal a háttérben indul el, a terminál pedig szabadon használható marad.

```bash
$ jobs -l
[1]+  1900 Running                 sleep 10000 &
```

## 3. A terminál bezárása a háttérfolyamat megtartásával (`disown`, `nohup`)

Alapértelmezésben, ha bezárod a terminált vagy megszakad az SSH kapcsolatod, a rendszer egy SIGHUP (hangup) jelet küld a shellből indított folyamatoknak, amik emiatt leállnak. Ennek elkerülésére két beépített módszer létezik.

### A) A `disown` parancs

Ha egy parancs már fut a háttérben (az `&` vagy a `bg` segítségével), de most be akarod zárni a terminált:

```bash
$ disown %1
```

(Ha csak egy feladat van, a `%1` elhagyható).

Ez a parancs eltávolítja a folyamatot a shell saját "feladatlistájából" (jobs table), így a terminál bezárásakor a folyamat nem kap leállási jelet, hanem zavartalanul fut tovább.

A `jobs -l` parancs ezután már nem fogja mutatni, de a `ps` paranccsal továbbra is megtalálod:

```bash
$ ps aux | grep sleep
linuxco+    1650  0.0  0.0   8084   524 pts/0    S    12:27   0:00 sleep 10000
```

Ha később mégis le akarod állítani, a `kill` paranccsal és a PID (Process ID) megadásával teheted meg:

```bash
$ kill 1650
```

### B) A `nohup` parancs

A `nohup` (no hangup) eleve arra utasítja a programot induláskor, hogy hagyja figyelmen kívül a SIGHUP jelet. A kimenetét (ha nem irányítod át máshova) egy `nohup.out` nevű fájlba fogja menteni.

```bash
$ nohup sleep 10000 &
[1] 1908
nohup: ignoring input and appending output to 'nohup.out'
```

## 4. A `screen` parancs használata (Haladó megoldás)

A fenti beépített parancsok mellett gyakori megoldás a `screen` szoftvercsomag használata, amely teljes virtuális terminál munkameneteket képes a háttérben futtatni. (Hasonló a `tmux` szoftverhez).

1. **Új screen session indítása:** Az `-S` opcióval nevet is adhatsz a munkamenetnek.

   ```bash
   $ screen -S mycommand
   ```

2. **A parancs futtatása:** Az újonnan megnyíló (üres) képernyőn indítsd el a hosszú szkriptet.

   ```bash
   $ /path/to/myscript.sh
   ```

3. **Lekapcsolódás (Detach) a screenről:** Nyomd meg a `Ctrl+A` billentyűkombinációt, majd engedd el, és nyomd meg a `D` betűt.

   Ezzel visszatérsz az eredeti terminálodba, de a screen a háttérben továbbra is zavartalanul fut. (A terminált most már nyugodtan bezárhatod).

4. **Aktív screenek listázása:**

   ```bash
   $ screen -ls
   There is a screen on:
       2741.mycommand	(04/08/2021 01:13:24 PM)	(Detached)
   ```

5. **Visszacsatlakozás (Reattach) a screenhez:** A `-r` opcióval és a processz azonosítóval (PID, a fenti listából) visszatérhetsz a futó folyamathoz:

   ```bash
   $ screen -r 2741
   ```

   ##Forrás
   -[https://linuxconfig.org/understanding-foreground-and-background-linux-processes]
