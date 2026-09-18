# A Linux `uptime` Parancs és a "Load Average" (Rendszerterhelés) Megértése

Az `uptime` egy rendkívül egyszerű, de alapvető Linux parancs. Megmutatja, hogy a rendszer mióta fut folyamatosan, az aktuális időt, a bejelentkezett felhasználók számát, és – ami a legfontosabb – a rendszer átlagos terhelését (load average) az elmúlt 1, 5 és 15 percben.

## 1. Az `uptime` Parancs Alapvető Használata

Ha a parancsot opciók nélkül futtatod:

```bash
uptime
```

Egyetlen soros kimenetet fogsz kapni, amely hasonló ehhez:

```plaintext
 16:13:00 up 2 days,  8:18,  1 user,  load average: 1.19, 1.54, 1.51
```

**A kimenet elemeinek magyarázata:**

- **16:13:00:** A rendszer aktuális ideje.
- **up 2 days, 8:18:** Az "uptime" (működési idő). Ez a rendszer 2 napja, 8 órája és 18 perce fut folyamatosan.
- **1 user:** Az aktuálisan bejelentkezett felhasználók (munkamenetek) száma. (Ezt a `/var/run/utmp` fájlból olvassa ki).
- **load average: 1.19, 1.54, 1.51:** Az átlagos terhelés az elmúlt 1, 5 és 15 percben.

## 2. Az `uptime` Parancs Kapcsolói (Options)

Bár az alapértelmezett kimenet elég informatív, a parancs testreszabható:

- **`-p` (`--pretty`):** Szebb, emberi fogyasztásra alkalmasabb (pretty) formátumban írja ki, és csak a működési időt jeleníti meg:

  ```bash
  uptime -p
  # Kimenet: up 2 days, 8 hours, 47 minutes
  ```

- **`-s` (`--since`):** Megmutatja a pontos dátumot és időpontot, amikor a rendszer elindult (bootolt):

  ```bash
  uptime -s
  # Kimenet: 2022-02-14 07:54:21
  ```

- **`-V` (`--version`):** Verzióinformáció megjelenítése.
- **`-h` (`--help`):** Súgó megjelenítése.

> **Bónusz Tipp:** Ha szeretnéd visszamenőleg látni a leghosszabb működési időket, telepítheted az `uptimed` démont (pl. `sudo apt install uptimed`), majd az `uprecords` paranccsal listázhatod a történelmi rekordokat.

## 3. Mi az a Load Average (Rendszerterhelés)?

A "Load" (Terhelés) a Linuxban azoknak a folyamatoknak (processzeknek) a száma, amelyek éppen a processzort (CPU-t) használják, VAGY arra várnak, hogy használhassák (futásra kész / runnable állapotban vannak), vagy éppen I/O műveletre (pl. lemezírásra) várnak (megszakíthatatlan / uninterruptable állapot).

- Ha a rendszer teljesen üresjárati (idle), a load értéke 0.
- Minden futó vagy várakozó folyamat 1-gyel növeli a load értékét.

Mivel a pillanatnyi terhelés másodpercenként drasztikusan változhat, a Linux az átlagos terhelést (Load Average) jeleníti meg három tizedestört formájában, balról jobbra haladva:

- **Első érték:** Átlagos terhelés az elmúlt 1 percben.
- **Második érték:** Átlagos terhelés az elmúlt 5 percben.
- **Harmadik érték:** Átlagos terhelés az elmúlt 15 percben.

### A CPU magok (Cores) és a Load Average kapcsolata

A load average érték önmagában nem mondja meg, hogy túl van-e terhelve a gép. Ezt mindig a processzor(ok) magjainak (cores) számához viszonyítva kell értelmezni!

- **1 magos CPU esetén:** Egy 1.00-ás load azt jelenti, hogy a CPU 100%-os kihasználtsággal működik. Nincs üresjárat, de még nincs is torlódás. (Ha az érték 1 fölé megy, a folyamatoknak már várakozniuk kell).
- **2 magos CPU esetén:** Egy 1.00-ás load azt jelenti, hogy az egyik mag 100%-on pörög, míg a másik teljesen üres (tehát a CPU kihasználtsága összesen csak kb. 50%). Itt a 2.00-ás load jelenti a 100%-os terhelést.
- **4 magos CPU esetén (Quad-core):** A 1.00-ás load mindössze 25%-os CPU használatot jelent. A 100%-os maximális kihasználtságot a 4.00-ás érték jelenti.

**Általános szabály:** Ha a load average (különösen az 5 és 15 perces átlag) tartósan meghaladja a CPU magok számát, akkor a rendszer túlterhelt, a folyamatok feltorlódtak, és a gép lassulni fog.

## 4. Egyéb eszközök a Load Average ellenőrzésére

Az `uptime` parancson kívül a terhelés más, részletesebb eszközökkel is vizsgálható:

- **`top` parancs:**

  Bebuildelt feladatkezelő, amely valós időben frissíti a futó folyamatokat és a rendszerterhelést. A fejlécében ugyanúgy ott van az uptime információ és a három load average érték:

  ```bash
  top
  ```

- **`glances` eszköz:**

  Egy fejlett rendszerfelügyeleti eszköz (külön kell telepíteni: `sudo apt-get install glances`), amely a `top`-nál átláthatóbb és részletesebb nézetet ad, és gyakran vizuálisan is (színekkel) jelzi, hogy a rendszerterhelés, illetve a rendelkezésre álló CPU magok aránya megfelelő-e:

  ```bash
  glances
  ```

  ##Forrás
  -[https://www.digitalocean.com/community/tutorials/load-average-in-linux]
  -[https://linuxhandbook.com/uptime-command/]
  -[https://www.tecmint.com/linux-uptime-command-examples/]
