# A folyamat- és szálprioritások megértése Linuxon

## 1. Bevezetés

A több futtatható feladatot támogató rendszereken minden feladathoz hozzá van rendelve egy „prioritás” vagy rangsor, amely meghatározza, milyen gyakran kapja meg a CPU-t. Az elv az, hogy a magasabb prioritású futtatható feladatok a lower prioritásúak előtt kapják meg a CPU-t, és az azonos prioritású feladatok round-robin módon kapják meg a CPU-t.

Első ránézésre tehát egy folyamat prioritása meglehetősen egyszerű fogalomnak tűnik. A feladat prioritásának nyomon követése és kezelése azonban meglehetően bonyolult, mert a Linux kernelnek különböző felhasználási eseteket kell kiszolgálnia. Például különböző ütemezési szabályzatokkal (azaz eltérő időzítési megkötésekkel és/vagy eltérő feladatkiválasztási kritériumokkal) rendelkező feladatokat kell ütemeznie, ideiglenesen növelheti egy alacsony prioritású feladat prioritását egy magas prioritású feladat blokkolásának feloldásához, vagy számos más olyan forgatókönyv lehet, ahol pusztán egy rangsor vagy prioritás adása egy feladathoz nem eredményez optimális összteljesítményt.

Továbbá számos, a feladatok prioritásának megfigyelésére és/vagy megváltoztatására használt felhasználói térbeli eszköz más UNIX-változatokkal is működik. Így ezek nem feltétlenül a Linux kernel által használt pontos prioritási értékeket mutatják. Ezen eszközök eredményeinek helyes értelmezéséhez meg kell értenünk a kapcsolatot az ezen eszközök által mutatott prioritási értékek és a kernel által belsőleg fenntartott tényleges prioritási értékek között.

Ez a cikk célja, hogy elmagyarázza a Linux kernel által belsőleg fenntartott és használt különböző prioritási értékeket, valamint azt, hogyan kell értelmezni a gyakori eszközök által mutatott prioritási értékeket.

## 2. Linux ütemezési szabályzatok/osztályok

Mielőtt a feladatprioritásokat vizsgálnánk, röviden leírom a Linux kernel által támogatott különböző ütemezési osztályokat/szabályzatokat. Egy ütemezési osztály vagy szabályzat határozza meg, hogyan választódik ki a CPU-n következőként futtatandó feladat. Azt is meghatározza, hogy a kiválasztott feladat mennyi ideig kapja meg a CPU-t, ha a feladat nem blokkolódik vagy önként adja fel a CPU-t. A Linux ütemezője mind valós idejű, mind nem valós idejű feladatokra működik. A valós idejű feladatoknak egy adott időkereten belül kell válaszolniuk egy eseményre, a legtöbb esetben a lehető leghamarabb.

Valós idejű feladatokhoz 3 szabályzat támogatott: `SCHED_RR`, `SCHED_FIFO` és `SCHED_DEADLINE`.

A `SCHED_RR` (azaz Round Robin) és a `SCHED_FIFO` (azaz First In First Out) a POSIX valós idejű szabályzatoknak felel meg. Ezen szabályzatok alá tartozó feladatok prioritásuk sorrendjében kerülnek kiválasztásra, azaz ha több `SCHED_FIFO` vagy `SCHED_RR` feladat van, a magasabb prioritású feladatok kerülnek először kiválasztásra.

A másik valós idejű szabályzatnál, azaz a `SCHED_DEADLINE`-nál (határidő) a feladatok a határidejük alapján kerülnek kiválasztásra.

Ez azt jelenti, hogy amikor több `SCHED_DEADLINE` feladat van, a legkorábbi határidejű feladat kerül először kiválasztásra. A `SCHED_DEADLINE` feladatok esetében a prioritás nem befolyásolja a feladatkiválasztást.

Nem valós idejű feladatokhoz 3 szabályzat támogatott: `SCHED_OTHER`, `SCHED_BATCH` és `SCHED_IDLE`. Ezek közül a `SCHED_OTHER`-t szinte minden esetben használják. A `SCHED_BATCH` és a `SCHED_IDLE` speciális eset szabályzatai a nem valós idejű feladatoknak. A `SCHED_BATCH` feladatok esetében az ütemező feltételezi, hogy ezek a feladatok CPU-igényesek, és nem igényelnek alacsony késleltetésű ütemezést az ébresztési eseményekre válaszul.

A `SCHED_IDLE` olyan feladatokhoz használatos, amelyek csak akkor kapnak CPU-t, ha nincs más futtatni való.

Az ütemezési szabályzatoknak egymás között is vannak prioritásaik a következő (magasabbtól az alacsonyabbig) sorrendben:

```plaintext
SCHED_DEADLINE > SCHED_RR/SCHED_FIFO > SCHED_OTHER > SCHED_BATCH > SCHED_IDLE
```

Amint a fentiekből látható, a `SCHED_RR` és a `SCHED_FIFO` feladatok azonos prioritással rendelkeznek, és ezen szabályzatok feladatai prioritásuk sorrendjében kapják meg a CPU-t.

Egy feladat a szülőjétől örökli az ütemezési szabályzatát, és ha szükséges, az ütemezési szabályzat a megfelelő rendszerhívásokkal megváltoztatható.

Ettől a ponttól kezdve a `SCHED_OTHER` ütemezési szabályzattal rendelkező feladatokat **normál feladatoknak**, a `SCHED_FIFO` és `SCHED_RR` ütemezési szabályzattal rendelkező feladatokat **valós idejű feladatoknak**, a `SCHED_DEADLINE` ütemezési szabályzattal rendelkező feladatokat pedig **határidős feladatoknak** fogom nevezni. A többi ütemezési osztályra szükség szerint explicit módon hivatkozom majd.

## 3. Feladatprioritások a Linux kernelben

A Linux külön prioritási tartományokat használ a normál és a valós idejű feladatokhoz. A normál feladatokhoz egy **-20 és +19** közötti prioritási tartományt (vagy nice értéket) használ. Az alacsonyabb nice érték magasabb prioritásnak felel meg. Más szóval a feladat kevésbé „kedves” a rendszer többi feladatával. A valós idejű feladatokhoz egy **0 és 99** közötti prioritási tartományt használ. Ebben az esetben a magasabb szám magasabb prioritást jelez.

Mivel a határidős feladatok nem prioritásuk alapján kerülnek kiválasztásra, ezekhez nincs megadva prioritási tartomány. Minden határidős feladat **-1** prioritást kap.

Belsőleg a Linux kernel egyetlen tartományt (**0-139**) használ, amely leképezi a fenti 2 tartományt (az alábbi ábrán látható módon). Ebben a tartományban az alacsonyabb érték magasabb prioritást jelent.

Amint korábban említettük, a kernelnek különböző, a feladat prioritásától függő felhasználási eseteket kell kiszolgálnia. Mindezen felhasználási esetek nem szolgálhatók ki hatékonyan pusztán osztályalapú prioritási értékekkel. Ezért minden feladathoz a kernel 4 típusú prioritást tart fenn a feladat `task_struct` struktúrájában.

Ezek a prioritások a következők:

- `static_prio`
- `rt_priority`
- `normal_prio`
- `prio`

A `static_prio` és az `rt_priority` az ütemezési osztályon alapuló prioritási értékeket tükrözik. A `normal_prio` meghatározott forgatókönyvekben használatos (az alábbiakban leírva), a `prio` pedig az effektív prioritás, amelyet a kernel ténylegesen használ az ütemezési döntéseiben.

### 3.1. `static_prio`

Ez a normál feladatokhoz használt prioritási tartományt képezi le, és a feladat nice értéke szerinti prioritás.

A nice érték és a `static_prio` közötti kapcsolat a következőképpen fejezhető ki:

```plaintext
static_prio = 120 + nice
```

Tehát a -20 és +19 közötti nice értékek esetén a `static_prio` a 100 és 139 közötti tartományban van.

Egy felhasználói feladat `static_prio` értéke a keletkezésekor (fork, exec stb.) kerül beállításra, a kernel azt saját magától nem változtatja meg. A felhasználó a megfelelő rendszerhívásokkal módosíthatja.

Alapértelmezés szerint a kernel feladatai (kthreads) a 0 nice értéknek megfelelő `static_prio`-val jönnek létre. Bizonyos kthread-ek esetében a kernel később megváltoztathatja a `static_prio`-t. Például a worker szálak esetében a kthread-ek először az alapértelmezett `static_prio`-val jönnek létre, majd később a kernel a `worker_pool` attribútumban (`pool.attr.nice`) megadott nice értéknek megfelelően megváltoztatja a `static_prio`-t. Ez a worker kthread-ek létrehozása után, de a worker pool-hoz való csatolásuk előtt történik.

A kernel részéről történő ideiglenes prioritásnövelés hiányában a normál feladatok effektív prioritása (azaz `prio`) megegyezik a `static_prio`-jukkal.

Normál feladatok esetében a `static_prio` hatékonyan meghatározza, hogy egy futtatható feladat milyen gyakran és mennyi ideig kapja meg a CPU-t. A `static_prio` nem befolyásolja a valós idejű vagy határidős feladatok CPU-idejét. De befolyásolja az összes feladat terhelési súlyát. A CPU terhelését a futási sorában lévő összes feladat terhelési súlyának felhasználásával határozzák meg.

### 3.2. `rt_priority`

Ez a valós idejű feladatokhoz használt prioritási tartományt képezi le, és a valós idejű prioritást jelzi.

A Linux által támogatott valós idejű prioritási tartomány a következő képlettel képeződik le a belső prioritási tartományra:

```plaintext
MAX_RT_PRIO-1 – rt_priority
```

Itt a `MAX_RT_PRIO` 100. Tehát a 0 és 99 közötti `rt_priority` a 99 és 0 közötti belső prioritási értékre képeződik le. Emlékezni kell arra, hogy a magas `rt_priority` érték magas prioritást jelent, és a kernelben az alacsony prioritási érték jelent magas prioritást. Innen a fenti összefüggés. A kernel részéről történő ideiglenes prioritásnövelés hiányában egy valós idejű feladat effektív prioritása (azaz `prio`) a fenti összefüggés szerint kapcsolódik az `rt_priority`-jához.

Az `rt_priority` csak a valós idejű feladatok prioritását befolyásolja, és a normál feladatok esetében figyelmen kívül marad. A feladat `rt_priority`-ja a létrehozásakor (fork, exec stb.) kerül beállításra, és később a megfelelő rendszerhívásokkal módosítható.

### 3.3. `normal_prio`

Ez egy feladat prioritását jelzi a kernel részéről történő ideiglenes prioritásnövelés nélkül. Normál feladatok esetében megegyezik a `static_prio`-val, RT feladatok esetében pedig közvetlenül kapcsolódik az `rt_priority`-hoz a következőképpen:

```plaintext
normal_prio = MAX_RT_PRIO-1 – rt_priority
```

A `normal_prio` fő célja, hogy megakadályozza a CPU éhezését más feladatok esetében egy vagy több megnövelt prioritású feladat gyermekei miatt.

A `normal_prio` hiányában egy prioritásnövelt feladat gyermekei is megnövelt prioritást kapnának, és ez CPU éhezést okozna más feladatok esetében. Az ilyen helyzet elkerülése érdekében a kernel fenntartja egy feladat `normal_prio`-ját. Az elágaztatott feladatok általában az effektív `prio`-jukat a szülő `normal_prio`-jára állítják, és így nem kapnak megnövelt prioritást.

### 3.4. `prio`

Ez egy feladat effektív prioritása, és minden ütemezéssel kapcsolatos döntéshozatalban használatos.

Amint korábban említettük, értéke 0 és 139 között van, és az alacsonyabb érték magasabb prioritást jelent. Általában a normál feladatok esetében a `static_prio`-tól, a valós idejű feladatok esetében az `rt_priority`-tól függ, de bizonyos speciális esetekben (mint az RT mutex, a megnövelt RCU olvasási oldali kritikus szakaszok stb.) a kernel megnövelheti egy feladat `prio`-ját anélkül, hogy megváltoztatná a `static_prio`-ját vagy az `rt_priority`-ját.

Az effektív prioritás révén a kernel 2 dolgot ér el:

1. Le tudja képezni a különböző ütemezési osztályok sokféle prioritási tartományát egyetlen tartományba, és ezt a tartományt használhatja ugyanazon végeredmény eléréséhez.
2. Ideiglenes prioritásnövelés esetén a kernel vissza tud térni a szokásos prioritási értékekhez, amint az ideiglenes prioritásnövelés szükségessége megszűnik.

Ha most leképezzük a Linux kernel által fenntartott különböző prioritási értékeket a különböző ütemezési szabályzatok különböző prioritási tartományaira, a következő ábrát kapjuk:

## 4. Hogyan értelmezzük a gyakori eszközök által mutatott prioritásokat

A diagnosztikai eszközök, mint a `top`, `ps` stb., a folyamat prioritási értékeit a procfs-ból szerzik be, de nem a procfs-ból olvasott prioritási értékeket mutatják. Ennek az az oka, hogy ezen eszközök némelyike más Unix-változatokkal is működik, és így a kimenetük megfelel a UNIX szabványnak, ami eltérhet a Linux-specifikus értelmezéstől. Tehát ahhoz, hogy ezen eszközök kimenetét helyesen értelmezzük a Linux kontextusában, világosan meg kell értenünk a kapcsolatot az ezen eszközök által mutatott prioritási értékek, a kernel által fenntartott prioritási értékek és a Linux által támogatott prioritási értékek között. Mivel ezek az eszközök a procfs-t használják, először nézzük meg, milyen prioritási értékeket exportál a kernel a procfs-ba, és hol.

### 4.1. A procfs-ban látható feladatprioritások

A feladat prioritása a procfs-ban is látható, de ugyanaz a prioritási érték nem mindenhol jelenik meg. Ezért fontos megérteni, hogy mely/milyen prioritási értékek láthatók a procfs-ban.

Az egyik hely, ahol a feladat prioritása ellenőrizhető, a `/proc/<pid>/sched` fájl.

Ebben a fájlban a „prio :” sor jelzi a feladat effektív prioritását (azaz `task_struct.prio`). Például:

```text
# cat /proc/2189/sched

ibus-engine-sim (2189, #threads: 3)

se.exec_start                            :   15866601445.863836
.................
.................
prio                                     :   120
.................
```

Egy másik hely, ahol a feladat prioritása ellenőrizhető, a `/proc/<pid>/stat` fájl.

Itt a 18. oszlop és a 40. oszlop ad prioritási értékeket. A 18. oszlopban látható érték:

```plaintext
p->prio – MAX_RT_PRIO
```

Mind RT, mind nem RT feladatok esetében.

A 40. oszlop egy feladat `rt_priority`-ját adja, és így csak RT feladatok esetében hasznos.

Vegyünk például néhány RT és nem RT feladatot, és nézzük meg, mit látunk a prioritási értékekre a procfs-ban.

#### 4.1.1. 1. példa

Nem RT feladat alapértelmezett prioritással (azaz `nice == 0`)

```text
# cat /proc/2189/sched

ibus-engine-sim (2189, #threads: 3)

se.exec_start                            :       15866601445.863836
...................................
...................................
policy                                   :       0
prio                                     :       120
...................................
```

Nézzük meg, mit látunk a `/proc/2189/stat` 18. és 40. oszlopában

```text
# cat /proc/2189/stat | awk '{ print $18 "," $40 }'
20,0
```

A 18. oszlop 20 (azaz `prio – MAX_RT_PRIO` vagy 120 – 100)

#### 4.1.2. 2. példa

Nem RT feladat `nice -10` értékkel.

Először változtassuk meg a feladat nice értékét:

```text
# renice -n -10 -p 2189
2189 (process ID) old priority 0, new priority -10
```

Most ellenőrizzük a procfs prioritási mezőit

```text
# cat /proc/2189/sched

ibus-engine-sim (2189, #threads: 3)

se.exec_start                          :        15868269398.011337
...............................
...............................
policy                                 :        0
prio                                   :        110
...............................

# cat /proc/2189/stat | awk '{ print $18 "," $40 }'
10,0
```

Amint látható, a nice érték 10-zel való csökkentése 120-ról 110-re csökkentette az effektív prioritását. Ha emlékszünk arra, hogy az effektív prioritás (azaz `task_stuct.prio`) alacsonyabb értéke magasabb prioritást jelent, láthatjuk, hogy gyakorlatilag növeljük ennek a folyamatnak a prioritását.

Mivel ez egy nem RT feladat, a `proc/<pid>/stat` 40. oszlopa lényegtelen.

Nézzünk néhány példát RT feladatokkal.

#### 4.1.3. 3. példa: RT feladat `rt_priority 50` értékkel

```text
# cat /proc/16/sched

idle_inject/0 (16, #threads: 1)

se.exec_start                     :        237.537238
..............................
..............................
policy                            :        2
prio                              :        49
..............................
```

Nézzük meg a `/proc/16/stat` fájlt

```text
# cat /proc/16/stat | awk '{ print $18 "," $40 }'
-51,50
```

A fenti részletből láthatjuk, hogy ennél az RT feladatnál az `rt_priority` 50, az effektív prioritás pedig 49 (azaz `MAX_RT_PRIORITY – 1 – rt_priority`). Ebben az esetben is a fenti 18. oszlopban lévő érték megfelel a (`task_struct.prio – MAX_RT_PRIO` azaz 49 – 100) értéknek.

Utolsó példaként változtassuk meg ennek a feladatnak az RT prioritását, és nézzük meg, hogyan változnak az értékek a procfs-ban:

#### 4.1.4. 4. példa: RT feladat `rt_priority 69` értékkel

Először változtassuk meg az RT prioritást

```text
# chrt -p 69 16
```

Ezután ellenőrizzük a procfs-t:

```text
# cat /proc/16/sched

idle_inject/0 (16, #threads: 1)

se.exec_start                 :         237.537238
............................
............................
policy                        :         2
prio                          :         30
.............................

# cat /proc/16/stat | awk '{ print $18 "," $40 }'
-70,69
```

Láthatjuk ugyanazt a kapcsolatot a procfs bejegyzések között.

Amint korábban említettük, néha a kernel ideiglenesen megnöveli egy feladat effektív prioritását, és ebben az esetben az `rt_priority`-ja vagy `static_priority`-ja nem lesz közvetlenül leképezett kapcsolatban az effektív prioritásával (azaz `prio`). De itt elsősorban egy feladat effektív prioritása és a `/proc/<pid>/stat` 18. oszlopában látható prioritása közötti kapcsolat érdekel minket, mert ezt az értéket használják a releváns eszközök (a következő szakaszban leírva) egy feladat prioritásának jelzésére.

### 4.2. A `ps`, `top` stb. segédprogramok által mutatott feladatprioritások

A `top` a prioritást a `/proc/<pid>/stat` 18. oszlopában látható módon mutatja mind RT, mind nem RT feladatok esetében. Az egyetlen kivétel, hogy nagyobb negatív számok esetében a prioritási érték helyett csak az „rt” karakterláncot mutatja, jelezve, hogy a feladat RT feladat.

Néhány példa látható alább:

```text
# cat /proc/16/stat | awk '{ print $18 "," $40 }'
-51,50

# cat /proc/6/stat | awk '{ print $18 "," $40 }'
0,0

#top
.................................
2 root  20   0   0  0  0  S    0.0    0.0    0:08.00    kthreadd
3 root   0  -20  0  0  0  I    0.0    0.0    0:00.00    rcu_gp
..................................
6 root   0  -20  0  0  0  I    0.0    0.0    0:00.00    netns
.................................
16 root -51   0  0  0  0  S    0.0    0.0    0:00.00    idle_inject/0
```

A `ps` esetében 2 fő opció van:

- A `ps -o priority` a `/proc/<pid>/stat` 18. oszlopa szerinti prioritást mutatja
- a `ps -o pri` pedig a prioritást.

A következő képlet szerint:

```plaintext
priority = 39 – a proc/<pid>/stat 18. oszlopából olvasott prioritás
```

Példaként ellenőrizhetjük a korábbi példában a `top` esetében szereplő pid-eket (6 és 16):

```text
# ps -ax -opri,priority,comm | grep netns
 39    0 netns

# ps -ax -opri,priority,comm | grep idle_inject
 90   -51 idle_inject/0
```

Az `-opri`-ban mutatott prioritás követi azt a konvenciót, hogy a magasabb érték magasabb prioritást jelent.

## 5. Következtetés

Ez a cikk leírta a kernel által egy feladathoz fenntartott különböző prioritási értékeket. Leírta azt is, hogyan lehet a különböző eszközök, mint a `ps`, `top` stb. által mutatott prioritási értékeket leképezni a kernel/ütemező által belsőleg a döntéshozatalában használt tényleges prioritási értékekre. Ez az információ felhasználható az ütemezési viselkedés elemzésekor egy vagy több CPU-n, a `top`, `ps` stb. adatainak felhasználásával. Hasznos akkor is, ha valaki a `ps`, `top` stb. által mutatott prioritásokat szeretné összevetni ugyanazon rendszer vmcore-jában látott különböző prioritásokkal.

##Forrás:
-[https://blogs.oracle.com/linux/task-priority]
-[https://www.itsmarttricks.com/how-to-manipulate-process-priority-in-linux-using-nice-and-renice-commands/]
