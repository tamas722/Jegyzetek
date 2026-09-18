# A Linux `/proc` Fájlrendszer Felfedezése

Bár a `/proc` könyvtár úgy tűnik, mintha mindig ott lenne a háttértáron, valójában ez egy virtuális fájlrendszer (procfs). A rendszer indításakor (bootoláskor) jön létre a memóriában, és leállításkor törlődik. Nem tartalmaz valódi fájlokat a lemezen, hanem valós idejű, rendkívül hasznos információkat szolgáltat az operációs rendszerről, a hardverről és a futó folyamatokról (processzekről).

## 1. A `/proc` Könyvtár Tartalma

Ha kiadod az `ls /proc` parancsot, több mint 150 fájlt és mappát fogsz látni (a futó folyamatok számától függően).

**Számozott könyvtárak (Process ID - PID):**

```plaintext
1     174    26    2902   4109   531
1004  176    2602  2907   41292  547
```

A számokkal elnevezett mappák az éppen futó folyamatok azonosítóit (PID) jelölik. Az `1`-es mappa például a legelsőként elindult folyamat (általában az `init` vagy a `systemd`) adatait rejti.

**Folyamat specifikus könyvtárak vizsgálata:**

Ha belépsz az egyik számozott mappába (pl. `ls /proc/411`), rengeteg fájlt találsz, amelyek az adott folyamat memóriatérképét, környezeti változóit és hálózati kapcsolatait írják le (pl. `cmdline`, `environ`, `fd`).

> **Fontos:** Ezeknek a fájloknak egy részét (pl. `cwd`, `root`, `exe` szimbolikus linkek) biztonsági okokból csak root (rendszergazda) jogosultsággal lehet olvasni. Normál felhasználóként a listázásuk "Permission denied" (Hozzáférés megtagadva) hibát eredményez.

## 2. A `/proc` Fájlok Különleges Jellemzői (0 bájtos méret)

Ha egy hosszú listázást kérsz (`ls -l /proc`), észre fogod venni, hogy a benne lévő normál szöveges fájlok (mint pl. a `cpuinfo` vagy a `meminfo`) fájlmérete 0 bájt.

- Egy hagyományos fájlrendszerben a 0 bájtos fájl azt jelenti, hogy a fájl teljesen üres.
- A `/proc` esetében azonban a fájlok virtuálisak. Ennek ellenére tartalmaznak adatot, csupán a virtuális jellegük miatt mutat a rendszer 0 bájtot. A tartalmuk abban a pillanatban generálódik le a memóriából, amikor megnyitod vagy kiolvasod őket.

## 3. Rendszerinformációk Kiolvasása (`cat` parancs)

Bár virtuálisak és 0 bájtosak, ezeket a fájlokat pontosan úgy olvashatod ki, mint bármelyik másik szöveges fájlt (például a `cat` paranccsal).

**Példa: CPU (Processzor) hardverinformációk lekérdezése:**

```bash
cat /proc/cpuinfo
```

**Példa kimenet (rövidítve):**

```plaintext
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 142
model name      : Intel(R) Core(TM) i5-7360U CPU @ 2.30GHz
stepping        : 9
cpu MHz         : 2303.998
cache size      : 4096 KB
physical id     : 0
siblings        : 1
core id         : 0
cpu cores       : 1
fpu             : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic...
```

**Egyéb hasznos fájlok a `/proc` mappában:**

Bár nem minden fájl tartalmaz az emberi szem számára könnyen értelmezhető adatokat (sok csak a fejlesztőknek vagy a szoftvereknek szól), a rendszergazdák számára a leghasznosabbak közé tartozik:

- `/proc/cpuinfo`: Processzor adatai.
- `/proc/meminfo`: Részletes memória- és swap használati statisztikák (ez a `free` és a `top` parancsok forrása is).

##Forrás
-[https://www.redhat.com/en/blog/linux-proc-filesystem]

# A Linux/Unix Folyamatok (Processes) Alapjai

A folyamat (process) egy program aktív végrehajtását jelenti. Amikor elindítasz egy alkalmazást, futtatsz egy programot vagy kiadsz egy parancsot, a Linux egy folyamatot kezdeményez. Egyetlen parancs egyetlen folyamatot hoz létre, de egy komplex alkalmazás több folyamatot is indíthat a feladatai elvégzéséhez.

A Linux rendszerekben a folyamatok általában egy már meglévő (szülő) folyamatból történő "leágazással" (forking) jönnek létre.

## 1. Folyamattípusok a Linuxban

**Szülő és Gyermek folyamat (Parent and Child Process):**

A gyermek folyamat egy olyan folyamat, amelyet egy másik (a szülő) hozott létre. Unix-szerű rendszereken ez általában a `fork` rendszerhívással (system call) történik (szemben a Windows NT `spawn` módszerével). A szülő folyamat az, amely egy vagy több ilyen gyermek (al-) folyamatot indít.

**Zombi és Árva folyamat (Zombie and Orphan Process):**

- **Zombi (Zombie):** Olyan folyamat, amely már befejezte a futását, de a bejegyzése még mindig ott van a folyamattáblában (process table). Ez azért történik, mert a szülő folyamata még nem olvasta ki a gyermek kilépési státuszát (exit status).
- **Árva (Orphan):** Olyan folyamat, amely azután is tovább fut, hogy a szülő folyamata már befejeződött vagy leállították. Létrejöhet szándékosan vagy véletlenül is.

**Démon folyamat (Daemon Process):**

Ezek a rendszerhez szorosan kötődő háttérfolyamatok. Gyakran root jogosultsággal futnak, és más folyamatok kéréseit szolgálják ki. A háttérben futva várnak bizonyos eseményekre vagy feladatokra. A normál folyamatokkal ellentétben a démonoknak nincs szükségük vezérlő terminálra (controlling terminal) a működéshez.

## 2. A Folyamatok Kezelésének Alapparancsai

| Parancs | Leírás és Használat | Példa |
| --- | --- | --- |
| `top` | Valós idejű, dinamikus listát ad a rendszerben éppen futó folyamatokról és az erőforrás-használatról. | `top` |
| `bg` | Egy felfüggesztett (megállított) feladat folytatása a háttérben (background). | `bg jobname` |
| `fg` | Egy háttérben lévő vagy felfüggesztett feladat visszahozása az előtérbe (foreground). | `fg jobname` |
| `ps` | Pillanatkép készítése a jelenleg futó folyamatokról (hasonló a `top`-hoz, de statikus). Egy adott folyamat (PID) állapotának lekérdezésére is jó. | `ps ux`, `ps PID` |
| `nice` | Új folyamat indítása specifikus prioritással (nice érték). Az értékek -20 (legmagasabb prioritás) és 19 (legalacsonyabb prioritás) között mozoghatnak. | `nice -10 parancs` |
| `kill` | Folyamatok leállítása vagy szignálok küldése a Folyamatazonosító (PID) alapján. Lehet finom (graceful) vagy erőszakos leállítás. | `kill PID` |

## 3. Rendszererőforrás-ellenőrző parancsok (Extra)

Bár nem szigorúan a folyamatkezeléshez tartoznak, az erőforrások ellenőrzése szorosan összefügg a folyamatok működésével:

**`df` (Disk Free):**

Megjeleníti a szabad lemezterületet (Hard Disk) a csatolt fájlrendszereken.

```bash
df
```

**`free`:**

Megjeleníti a rendszer szabad és használt memóriájának (RAM) teljes mennyiségét (valamint a swap terület állapotát).

```bash
free
```

##Forrás
-[https://www.scaler.com/topics/linux-process/]
-[https://www.kernel.org/doc/html/latest/filesystems/proc.html]
