# Az Elérhető (Available) Memória Ellenőrzése Linuxon: 5 Módszer

A Linuxban az "elérhető" (vagy "szabad" / unused) memória azt a memóriamennyiséget jelenti, amelyet az operációs rendszer vagy az alkalmazások jelenleg nem használnak. Ennek az értéknek a figyelése elengedhetetlen, amikor el kell dönteni, hogy a rendszer elbír-e egy újabb erőforrás-igényes folyamatot (processzt).

Íme 5 hatékony módszer a szabad memória ellenőrzésére:

## 1. A `free` parancs használata

A `free` parancs a leggyorsabb módja a memóriaállapot lekérdezésének. Az `-m` (megabytes) opcióval a fizikai memória teljes (total), használt (used), megosztott (shared) és elérhető (available) mennyiségét mutatja meg megabájtban.

**Parancs:**

```bash
free -m
```

A kimenetben az "available" oszlop mutatja meg azt a memóriamennyiséget (pl. 837 MB), ami azonnal felhasználható az új folyamatok számára.

## 2. A `top` parancs használata

A `top` parancs egy valós idejű, dinamikus áttekintést nyújt a futó folyamatokról és a rendszer erőforrás-használatáról.

**Parancs:**

```bash
top
```

Amikor a parancs fut, a fejlécben található "Mem" sorban látható a teljes, a szabad (free), a használt és a pufferelt memória mennyisége. Emellett lent a listában az is látszik, hogy az egyes folyamatok pontosan mennyi memóriát fogyasztanak (a `%MEM` oszlopban).

## 3. A `vmstat` parancs használata

A `vmstat` (Virtual Memory Statistics) parancs az `-s` (statistics) opcióval részletes listát ad a rendszer memóriájának állapotáról, beleértve a teljes (total), használt (used), aktív, inaktív és elérhető (available) memóriát.

**Parancs:**

```bash
vmstat -s
```

**Példa kimeneti sor a szabad memóriára:**

```plaintext
   232716 K free memory
```

(Az értékeket alapértelmezés szerint kilobájtban - KB - jeleníti meg).

## 4. A `/proc/meminfo` fájl olvasása (`cat`)

A Linux a `/proc/meminfo` virtuális fájlban tárolja a legalapvetőbb és legrészletesebb memóriainformációkat. A legtöbb monitorozó eszköz is ebből a fájlból nyeri ki az adatokat.

**Parancs:**

```bash
cat /proc/meminfo
```

A kimenet egy hosszú lista lesz, amelyben a `MemAvailable:` sor (pl. `MemAvailable: 902193 kB`) mutatja meg a rendszer rendelkezésére álló szabad memóriát.

## 5. A Rendszerfigyelő (System Monitor) használata - Grafikus (GUI) módszer

Ha asztali környezetet használsz, a parancssor helyett a beépített Rendszerfigyelő (System Monitor) alkalmazást is használhatod.

1. Nyisd meg az alkalmazások menüt, és keress rá a **System Monitor** (Rendszerfigyelő) kifejezésre.
2. Kattints a **Resources** (Erőforrások) fülre.
3. A "Memory and Swap History" (Memória és cserehely) grafikon alatt pontosan láthatod a memóriahasználatot (pl. `1.2 GB (60.0%) of 2.0 GB`), ami alapján kiszámolható, hogy 0.8 GB elérhető memória maradt.

 ##Forrás
 -[https://itslinuxfoss.com/5-ways-check-available-memory-linux/]

# Szabad (Free) vs. Elérhető (Available) Memória a Linuxban

A Linux memóriakezelésének megértése kritikus fontosságú. Amikor a rendszer memóriahasználatát vizsgáljuk, sokan megijednek, ha azt látják, hogy a "szabad" memória szinte nulla. A Linux azonban szándékosan működik így. Nagyon fontos különbséget tenni a **Free (Szabad)** és az **Available (Elérhető)** memória között.

## 1. A Különbség Megértése

- **Free (Szabad) Memória:** Ez az a memória (RAM), ami jelenleg egyáltalán semmit sem csinál. Különösen szerverkörnyezetben a szabad memóriát úgy is tekinthetjük, mint elpazarolt (wasted) memória. Miután a folyamatok elindultak és a rendszer már egy ideje fut (magas uptime), ennek a számnak ideális esetben nagyon kicsinek kell lennie.

- **Available (Elérhető) Memória:** Ez a memóriamennyiség áll rendelkezésre új vagy meglévő folyamatok számára anélkül, hogy a rendszernek a lassabb Swap-hez (cserehelyhez) kellene nyúlnia. Az "Available" érték tartalmazza a "Free" memóriát PLUSZ azokat a puffereket és gyorsítótárakat (buffers/cache), amelyeket az operációs rendszer bármikor, teljesítményveszteség nélkül fel tud szabadítani, ha egy alkalmazásnak szüksége van rá.

**Összegzés:** A "Free" memória kihasználatlanul áll, míg az "Available" memória aktívan használt (pl. fájlrendszer gyorsítótárazására), de szükség esetén azonnal átcsoportosítható. Mindig az "Available" értéket kell figyelni, ha arra vagy kíváncsi, mennyi memóriád maradt!

## 2. Memória ellenőrzése a `free` paranccsal

A leggyakoribb eszköz a memória ellenőrzésére a `free` parancs. A `-h` (human-readable) opcióval olvasható formátumban (MB, GB) kapjuk meg az adatokat:

```bash
free -h
```

**Kimenet értelmezése (Példa két szerver alapján):**

- **"A" Szerver (Kiváló memóriahasználat):**

  Ha a 60 GB-os szerveren a `free` (szabad) memória <1%, az `available` (elérhető) pedig 13 GB, az azt jelenti, hogy a rendszer a memóriája majdnem 100%-át hasznosítja (pl. gyorsítótárazásra), de van bőven tartaléka új folyamatok indítására.

- **"B" Szerver (Elpazarolt erőforrások):**

  Ha egy másik 60 GB-os szerveren több hónapnyi futás után is van 30 GB `free` (szabad) memória, az azt jelenti, hogy a memória több mint 50%-a szó szerint semmit nem csinál, teljesen el van pazarolva.

> **Megjegyzés:** A Linux kernel a legritkábban használt memórialapokat akkor is áthelyezheti a Swap területre, ha van még elérhető memória. Ezt opportunista swappingnak hívják.

## 3. Mit kell mérlegelnie egy Rendszergazdának?

Amikor a memóriastatisztikákat elemzed, ne vonj le elhamarkodott következtetéseket csak a "Free" oszlop alapján. Tedd fel magadnak a következő kérdéseket:

- Lassítja a szervert a swap (cserehely) használata?
- Várható a forgalom/munkaterhelés növekedése a közeljövőben?
- Érdemes-e a memóriát csökkenteni (downgrade) a "B" szervernél, mivel a fele üresen áll?
- Beállítható-e a "B" szerver (pl. az adatbázis szoftver) úgy, hogy több memóriát használjon gyorsítótárazásra (buffers/cache) a teljesítmény növelése érdekében?

## 4. Egyéb Hasznos Eszközök a Memória Elemzésére

A `free` parancs mellett az alábbi eszközök is kiválóak a memóriahasználat és a rendszer teljesítményének megfigyelésére:

- **`top`** – Átfogó, folyamatosan frissülő rendszerkép.
- **`htop`** – Interaktív, vizuálisan fejlettebb folyamatkezelő (a `top` alternatívája).
- **`atop`** – Haladó Linux szerver teljesítményelemző.
- **`Glances` és `nmon`** – További modern, sokrétű alternatívák a `top`/`htop` helyett.
- **`vmstat`** – Részletes statisztika a rendszermemóriáról, folyamatokról, megszakításokról, lapozásról és a blokk I/O-ról.
- **`cat /proc/meminfo`** – A kernel nyers memóriastatisztikáinak közvetlen olvasása.

##Forrás
-[https://linuxblog.io/free-vs-available-memory-in-linux/]
