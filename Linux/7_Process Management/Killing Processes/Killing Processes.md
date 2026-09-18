# A Linux `kill` Parancs Használata (Folyamatok Leállítása és Szignálok Küldése)

Néha az alkalmazások nem válaszolnak, vagy túl sok rendszererőforrást fogyasztanak, és a normál bezárás már nem működik. A `kill` parancs azáltal fejezi be ezeket a folyamatokat, hogy egy szignált küld nekik – a finom leállítási kéréstől (graceful shutdown) az azonnali, erőszakos kilövésig. Bár a nevéből azt hihetnénk, hogy csak "gyilkol", valójában bármilyen szignál elküldésére alkalmas.

> **Megjegyzés:** A legtöbb shellben, pl. a Bash-ben vagy a Zsh-ban a `kill` egy beépített parancs. A viselkedése kissé eltérhet a `/bin/kill` önálló futtatható állományétól.

## 1. A `kill` Parancs Szintaxisa és a Leggyakoribb Szignálok

**Szintaxis:**

```bash
kill [OPTIONS] [PID]...
```

(Ha nem adunk meg szignált, az alapértelmezett a `-15`, azaz a `SIGTERM`).

**A 3 legfontosabb szignál:**

- **1 (SIGHUP - Hangup):** A démonok és háttérszolgáltatások gyakran ezt használják a konfigurációs fájlok újraolvasására (reload) anélkül, hogy leállnának.
- **9 (SIGKILL - Kill):** Egy folyamat azonnali, erőszakos kilövése.
- **15 (SIGTERM - Terminate):** Egy folyamat finom (udvarias) leállítása.

A szignálokat többféleképpen is megadhatod. Az alábbi három parancs pontosan ugyanazt csinálja (újraolvasásra utasítja a folyamatot):

```bash
kill -1 PID_NUMBER
kill -SIGHUP PID_NUMBER
kill -HUP PID_NUMBER
```

> **Tipp:** A `kill -l` paranccsal kilistázhatod a rendszer összes elérhető szignálját.

## 2. A folyamatok leállítása (SIGTERM vs. SIGKILL)

- **Aranyszabály:** Mindig próbáld meg először a `SIGTERM` (`-15`) szignált! Ez időt hagy a programnak arra, hogy elmentse az adatait, lezárja a fájlokat, és biztonságosan kilépjen. A `SIGKILL` (`-9`) a "végső megoldás", mert azonnal megöli a programot, aminek adatvesztés vagy sérült fájlok lehetnek a következményei.

- Először meg kell találnod a folyamat azonosítóját (PID). Ezt a `top`, a `ps` vagy a `pidof` paranccsal teheted meg.

  Például a Firefox PID-jének megkeresése:

  ```bash
  pidof firefox
  # Kimenet pl.: 6263 6199 6142 6076
  ```

- Ezután leállíthatod őket (az alapértelmezett, 15-ös szignállal):

  ```bash
  kill 6263 6199 6142 6076
  ```

- Ha nem reagál, jöhet az erőszakos kilövés:

  ```bash
  kill -9 6263 6199 6142 6076
  ```

**Okos egysoros megoldás:** A keresést és a leállítást kombinálhatod is:

```bash
kill -9 $(pidof firefox)
```

## 3. Speciális Használati Esetek

**Konfiguráció újraolvasása (SIGHUP - 1)**

Ha például az Nginx webszervert szeretnéd utasítani, hogy olvassa be újra a beállításait:

1. Olvasd ki a fő folyamat (master process) azonosítóját:

   ```bash
   cat /var/run/nginx.pid
   # Kimenet pl.: 30251
   ```

2. Küldd el az 1-es szignált (rendszergazdai jogokkal):

   ```bash
   sudo kill -1 30251
   ```

**Létezik-e a folyamat? (`-0`)**

A 0-ás szignál segítségével úgy tudod ellenőrizni, hogy egy adott PID létezik-e (és van-e jogod hozzáférni), hogy közben semmilyen valódi szignált nem küldesz neki:

```bash
kill -0 PID
```

**Háttérfeladatok (Background Jobs) leállítása**

Ha a terminálodon a háttérben futnak feladatok, azokat a Job ID alapján is leállíthatod a `%` jel használatával (nem kell a PID-t kikeresni).

A `jobs` paranccsal megnézheted a sorszámot (pl. `[1]`), majd:

```bash
kill %1
```

## 4. Alternatívák a `kill` parancsra (Leállítás név alapján)

Ha nem akarsz bajlódni a PID-k kikeresésével, használhatod a nevek alapján működő eszközöket:

- **`pkill`:** Részleges névegyezés alapján állít le folyamatokat. Ugyanazokat a kapcsolókat fogadja, mint a `kill`.

  ```bash
  pkill firefox
  ```

- **`killall`:** Kizárólag a pontos névegyezés esetén állítja le a folyamatokat.

  ```bash
  killall firefox
  ```

## 5. Gyakori Hibák és Hibaelhárítás

- **`kill: (PID): No such process`:** A folyamat már kilépett (vagy beírtál egy rossz PID-t).
- **`kill: (PID): Operation not permitted`:** Nincs jogosultságod leállítani a folyamatot (mert pl. a root vagy egy másik felhasználó indította). Használj `sudo`-t a parancs előtt.
- **A folyamat nem áll le a `kill -15` után:** A folyamat ignorálja a szignált, vagy beakadt egy rendszerhívásba. Használd a `kill -9`-et az erőszakos kilövéshez.

**A PID trükkjei (Jó tudni):**

- Ha a **PID = 0**: A szignál a jelenlegi shell folyamatcsoportjának (PGID) összes tagjához eljut.
- Ha a **PID = -1**: A szignál minden folyamathoz eljut, amihez a felhasználónak joga van (kivéve a rendszer alapfolyamatait, mint a PID 1). Óvatosan használandó!

##Forrás
-[https://linuxize.com/post/kill-command-in-linux/]

# A Linux `kill` Parancs Használata és Gyakorlati Példák

Amikor egy folyamat (process) túl sok rendszererőforrást emészt fel, vagy egyszerűen nem válaszol, a leggyorsabb megoldás a leállítására a `kill` parancs használata.

## 1. Alapszintaxis

A `kill` parancs maximális kihasználásához ismerni kell az alapvető szintaxisát:

```bash
kill [opciók] [szignál] PID
```

- **[opciók]:** A `kill` parancs alapértelmezett viselkedésének finomhangolása.
- **[szignál]:** A folyamat leállítására vagy befolyásolására használni kívánt szignál (jelzés). Ha nem adod meg, az alapértelmezett a `SIGTERM` (15).
- **PID:** A leállítani kívánt folyamat azonosítója (Process ID).

## 2. A `kill` Parancs Opciói

| Opció | Leírás |
| --- | --- |
| `-s` | Meghatározza a küldendő szignált név vagy szám alapján (pl. `-s SIGINT` vagy `-s 2`). Ezzel felülbírálható az alapértelmezett `SIGTERM`. |
| `-q` | A szignált a `sigqueue(3)` használatával küldi el a `kill(2)` helyett, lehetővé téve egy extra egész szám (integer) érték elküldését a szignállal együtt. |
| `-l` | Kilistázza az összes elérhető szignál nevét és a hozzájuk tartozó numerikus értéket. |
| `-L` | Szép, emberi olvasásra alkalmas (táblázatos) formátumban listázza a szignálokat. |

## 3. A Legfontosabb Szignálok Listája

| Szignál Neve | Érték | Leírás |
| --- | --- | --- |
| `SIGHUP` | 1 | Kapcsolat megszakadása (Hangup). Folyamatok újraindítására vagy konfigurációjuk újraolvasására használják. |
| `SIGINT` | 2 | Megszakítás (Interrupt). Ez a szignál küldődik, amikor lenyomod a `Ctrl+C` gombot. |
| `SIGQUIT` | 3 | Kilépés (Quit). Célja a folyamat leállítása egy core dump (memóriakép) létrehozásával. |
| `SIGILL` | 4 | Érvénytelen (Illegal) utasítás, ha a folyamat valami futtathatatlant próbál végrehajtani. |
| `SIGTERM` | 15 | Befejezés (Termination). Ez a `kill` parancs által küldött alapértelmezett szignál a finom leállításhoz. |
| `SIGKILL` | 9 | Kilövés (Kill). Erőszakos szignál, amely azonnal, takarítás nélkül megszakítja a folyamatot. |
| `SIGSTOP` | 19 | Megállítás (Stop). A folyamat szüneteltetésére (felfüggesztésére) szolgál. |
| `SIGCONT` | 18 | Folytatás (Continue). Egy szüneteltetett folyamat újraindítására használatos. |
| `SIGTSTP` | 20 | Terminál általi megállítás. Ez a szignál küldődik, amikor lenyomod a `Ctrl+Z` gombot. |
| `SIGCHLD` | 17 | Gyermek (Child) szignál. A szülő folyamat kapja, ha egy gyermekfolyamata leáll vagy befejeződik. |

## 4. Gyakorlati Példák

### 1. Egy folyamat egyszerű leállítása

A parancshoz csak a folyamatazonosítót (PID) kell megadni. Például a 78210-es PID-jű (pl. Slack) folyamat leállítása:

```bash
kill 78210
```

Több folyamatot is leállíthatsz egyszerre, ha a PID-ket szóközzel elválasztva adod meg:

```bash
kill PID1 PID2 PIDn
```

### 2. Folyamat leállítása egy specifikus szignállal (`-s`)

Az esetek többségében az alapértelmezett (15-ös) szignál elég, de ha specifikus jelet akarsz küldeni (pl. a 9-es `SIGKILL`-t), használd az `-s` flaget:

```bash
kill -s SIGKILL 3532
# (Megadhatod számmal is: kill -s 9 3532)
```

### 3. Az elérhető szignálok listázása (`-L`)

A szép, formázott listához használd az `-L` opciót:

```bash
kill -L
```

(Ha a shell beépített `kill` parancsa esetleg ismétlődő adatokat ír ki, érdemes meghívni a parancs külső, önálló verzióját: `/usr/bin/kill -L`).

### 4. A szignál nevének megkeresése a száma alapján (`-l`)

Ha ismersz egy szignálszámot, de nem tudod a nevét, az `-l` flaggel visszafejtheted:

Például a 18-as számú szignál nevének lekérdezése:

```bash
kill -l 18
# Kimenet: CONT
```

##Forrás
-[https://linuxhandbook.com/kill-command/]
