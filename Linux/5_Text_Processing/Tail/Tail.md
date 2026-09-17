# A Linux `tail` Parancs Használata (Fájlok végének megjelenítése és követése)

Szerverproblémák hibaelhárításakor vagy naplófájlok (logok) elemzésekor gyakran nincs szükség a teljes fájl megnyitására, csak az utolsó néhány sorra. A `tail` parancs pontosan ezt csinálja: kiírja egy fájl végét (alapértelmezésben a legutolsó 10 sort), és képes valós időben követni a fájlhoz újonnan hozzáadott sorokat.

## 1. A `tail` Parancs Szintaxisa és Alapértelmezett Működése

**Szintaxis:**

```bash
tail [OPTION]... [FILE]...
```

(Ha nincs fájl megadva, vagy a fájl argumentum `-`, a `tail` a szabványos bemenetről olvas).

Ha opciók nélkül használod, a `tail` az utolsó 10 sort jeleníti meg, függetlenül a fájl méretétől:

```bash
tail /var/log/syslog
```

**Kimenet:**

```plaintext
Apr  7 08:15:01 server CRON[1423]: (root) CMD (command -v debian-sa1)
Apr  7 08:17:01 server CRON[1430]: (root) CMD (cd / && run-parts)
Apr  7 08:20:22 server systemd[1]: Starting Daily apt activities...
Apr  7 08:20:24 server systemd[1]: Started Daily apt activities.
Apr  7 08:25:01 server CRON[1445]: (root) CMD (command -v debian-sa1)
Apr  7 08:30:01 server CRON[1452]: (root) CMD (command -v debian-sa1)
Apr  7 08:32:11 server sshd[1460]: Accepted publickey for admin
Apr  7 08:35:01 server CRON[1478]: (root) CMD (command -v debian-sa1)
Apr  7 08:40:01 server CRON[1485]: (root) CMD (command -v debian-sa1)
Apr  7 08:45:01 server CRON[1492]: (root) CMD (command -v debian-sa1)
```

## 2. A sorok számának megadása (`-n`)

Az `-n` (`--lines`) opcióval pontosan megadható, hány sort szeretnél látni a fájl végéről:

```bash
tail -n 50 /var/log/syslog
```

> **Megjegyzés:** A GNU `tail` még elfogadja a régi `tail -50` rövidítést is, de a hivatalos kézikönyv ezt elavultnak tekinti. Szkriptekben mindig a `tail -n 50` formát használd.

**Kezdés egy adott sortól (A `+` jel használata)**

Ha a szám elé `+` jelet teszel, a `tail` nem a végétől számol vissza, hanem a megadott sortól kezdve a fájl legvégéig mindent kiír. Ez nagyon hasznos, ha például egy CSV fájl fejlécét (első sorait) akarod levágni:

```bash
# Kihagyja az első 4 sort, és az 5. sortól a végéig mindent kiír
tail -n +5 /etc/passwd
```

## 3. Megadott számú bájt kiíratása (`-c`)

Sorok helyett bájtok számát is megadhatod a `-c` (`--bytes`) opcióval:

```bash
tail -c 500 /var/log/auth.log
```

Használhatsz szorzókat (utótagokat) is: `b` (512), `K` (1024), `M` (1048576) stb. Például az utolsó 2 kibibájt (2048 bájt) kiírása:

```bash
tail -c 2K /var/log/auth.log
```

## 4. Fájlok valós idejű követése (`-f` és `-F`)

Az `-f` (`--follow`) opció nyitva tartja a fájlt, és a képernyőre írja az újonnan hozzáfűzött sorokat. Ez a leggyakoribb módja a logok valós idejű megfigyelésének. (A követés leállításához nyomj `Ctrl+C`-t).

```bash
tail -f /var/log/nginx/error.log
```

Kombinálhatod az `-n` opcióval, hogy megadd, hány régebbi sort mutasson a követés megkezdése előtt:

```bash
tail -n 50 -f /var/log/syslog
```

**Rotált (cserélt) logfájlok követése (`-F`)**

A logfájlokat a `logrotate` gyakran átnevezi és újat hoz létre helyettük. A sima `-f` a fájlleírót (file descriptor) követi, így forgatás után az eredeti (már átnevezett) fájlt figyelné tovább.

Az `-F` (ami a `--follow=name --retry` rövidítése) a fájlnevet követi. Érzékeli, ha az eredeti fájlt áthelyezték, és automatikusan újra megnyitja az azonos néven létrejött új fájlt.

```bash
tail -F /var/log/nginx/access.log
```

**Leállítás egy folyamat (Process) kilépésekor**

Ha egy logfájlt figyelsz, amit egy adott folyamat ír, megmondhatod a `tail`-nek, hogy automatikusan álljon le, amikor az a folyamat befejeződik (a `1234` helyére a valódi PID-t kell írni):

```bash
tail -f --pid=1234 /var/log/myapp.log
```

## 5. Több fájl vizsgálata egyszerre

Több fájl megadása esetén a `tail` mindegyikből kiírja az utolsó 10 sort, és egyértelmű fejlécekkel (`==> fájlnév <==`) választja el őket egymástól:

```bash
tail /var/log/syslog /var/log/auth.log
```

**Kimenet:**

```plaintext
==> /var/log/syslog <==
Apr  7 08:40:01 server CRON[1485]: (root) CMD (command -v debian-sa1)
Apr  7 08:45:01 server CRON[1492]: (root) CMD (command -v debian-sa1)
...

==> /var/log/auth.log <==
Apr  7 08:32:11 server sshd[1460]: Accepted publickey for admin
Apr  7 08:33:05 server sudo:   admin : TTY=pts/0 ; COMMAND=/usr/bin/apt update
...
```

(Az opciók, mint az `-n 20` vagy a `-f`, több fájl esetén is tökéletesen működnek. Valós idejű követésnél (`-f`) a `tail` mindig kiírja a fejlécet, amikor egy másik fájlba új adat érkezik).

## 6. A `tail` kombinálása más parancsokkal (Pipes)

A `tail` gyakran használatos csővezetékekben (pipes) más parancsokkal kombinálva:

**Logfájl valós idejű szűrése egy adott IP-címre:**

```bash
tail -f /var/log/nginx/access.log | grep 192.168.42.12
```

**Az 5 legtöbb CPU-t használó folyamat kiírása:**

A `ps` kimenetét a `sort` a 3. oszlop (CPU%) alapján növekvő sorrendbe rendezi, majd a `tail` leveszi a lista legvégéről a legmagasabb 5 értéket.

```bash
ps aux | sort -nk 3 | tail -n 5
```

**A 20. és a 30. sor közötti rész kivágása egy fájlból:**

A `tail` a 20. sortól a végéig mindent átad, majd a `head` ebből megtartja az első 11 sort (vagyis a 20-30. sorokat).

```bash
tail -n +20 /var/log/syslog | head -n 11
```

## 7. Gyors Referencia (Összefoglaló Táblázat)

| Parancs | Leírás |
| --- | --- |
| `tail file.log` | Az utolsó 10 sor megjelenítése. |
| `tail -n 50 file.log` | Az utolsó 50 sor megjelenítése. |
| `tail -f file.log` | Új sorok valós idejű követése. |
| `tail -F file.log` | Fájlnév szerinti követés (log rotáció esetén nem törik el). |
| `tail -n +5 file.txt` | A fájl kiírása az 5. sortól kezdve a végéig. |
| `tail -c 500 file.log` | Az utolsó 500 bájt megjelenítése. |
| `tail file1.log file2.log` | Az utolsó 10 sor megjelenítése mindkét fájlból (fejlécekkel). |
| `tail -f /syslog /auth.log` | Több fájl egyidejű, valós idejű követése. |

##Forrás
-[https://linuxize.com/post/linux-tail-command/]

# A Linux `tail` Parancs Használata és Példák (Kiegészítés)

A `cat` parancs az egész fájlt kiírja a képernyőre, ami nagy fájlok esetén nem praktikus. Ilyenkor jön jól a `tail` parancs, amely egy vagy több fájl legutolsó részét jeleníti meg. Emellett a leggyakrabban naplófájlok (logok) valós idejű megfigyelésére használják.

## 1. Alapszintaxis és Alapértelmezett Működés

**Szintaxis:**

```bash
tail [options] [files]
```

Ha nem adsz meg semmilyen opciót, a `tail` alapértelmezés szerint az utolsó 10 sort írja ki a fájlból. (Az üres sorok is érvényes sornak számítanak. Ha a fájl rövidebb 10 sornál, akkor az egészet kiírja).

```bash
tail agatha.txt
```

**Kimenet:**

```plaintext
Giant's Bread
The Floating Admiral
The Sittaford Mystery
Peril at End House
Lord Edgware Dies
Murder on the Orient Express
Unfinished Portrait
Why Didn't They Ask Evans?
Three Act Tragedy
Death in the Clouds
```

## 2. Gyakorlati Példák és Opciók

### 1. Az utolsó N darab sor kiíratása (`-n`)

Ha nem az alapértelmezett 10 sort, hanem egy specifikus számú sort akarsz látni a fájl végéről, használd az `-n` opciót (vagy egyszerűen csak kötőjelet és a számot).

Például az utolsó 5 sor megjelenítése:

```bash
tail -n 5 agatha.txt
# VAGY a rövidebb forma:
tail -5 agatha.txt
```

**Kimenet:**

```plaintext
Murder on the Orient Express
Unfinished Portrait
Why Didn't They Ask Evans?
Three Act Tragedy
Death in the Clouds
```

### 2. Az összes sor kiíratása az N. sortól kezdve (`+` jel)

Ha nem a végétől akarsz visszaszámolni, hanem a fájl elejétől a megadott sortól kezdve mindent ki akarsz írni a fájl végéig, használj `+` jelet a szám előtt.

Például a 7. sortól kezdve az összes sor kiírása:

```bash
tail -n +7 agatha.txt
```

**Kimenet:**

```plaintext
The Big Four
The Mystery of the Blue Train
The Seven Dials Mystery
The Murder at the Vicarage
... (és a többi sor a fájl végéig)
```

### 3. Több fájl egyidejű olvasása

A `tail` parancsnak egyszerre több fájlt is átadhatsz. Alapértelmezésben a kimeneteket a fájlneveket tartalmazó fejlécek (`==> fájlnév <==`) választják el.

Utolsó 3 sor kiírása két fájlból:

```bash
tail -n3 sherlock.txt agatha.txt
```

**Kimenet:**

```plaintext
==> sherlock.txt <==
The Adventure of the Noble Bachelor
The Adventure of the Beryl Coronet
The Adventure of the Copper Beeches

==> agatha.txt <==
Why Didn't They Ask Evans?
Three Act Tragedy
Death in the Clouds
```

> **Tipp:** A fejléceket a `-q` (quiet) opcióval eltüntetheted a kimenetből.

### 4. Fájlok valós idejű követése (Logok monitorozása, `-f` és `-F`)

Ha egy fájl folyamatosan bővül (pl. egy logfájl), a `tail` képes az újonnan hozzáadott sorokat azonnal a képernyőre írni.

Először kiírja az utolsó 10 sort, majd folyamatosan frissíti a kimenetet.

```bash
tail -f <log-file>
```

> **Tipp:** Ha a kis `-f` helyett nagy `-F` opciót használsz, a `tail` várni fog a fájl létrejöttére, ha az még nem létezik, és utána kezdi el a valós idejű követést.

### 5. A `tail` parancs használata csővezetékekkel (Pipes)

A `tail` kimenete és bemenete szabadon kombinálható más parancsokkal.

Például, ha egy mappában túl sok fájl van, és te csak a 3 legutóbb módosított fájlt szeretnéd látni:

```bash
ls -ltr | tail -n3
```

(Magyarázat: Az `ls -ltr` időrendben kilistázza a fájlokat úgy, hogy a legújabbak vannak legalul. A `tail -n3` ebből a listából csak a legalsó hármat tartja meg).

## Bónusz Tipp: A `tail` kimenetének sorszámozása

Gyakran hasznos, ha a sorok számozva vannak (így láthatod például a fájl teljes sorszámát). A `tail` parancsnak nincs beépített sorszámozó funkciója, de az `nl` (number lines) paranccsal és egy csővezetékkel (`|`) ez könnyen megoldható.

Először sorszámozzuk a fájl teljes tartalmát az `nl` paranccsal, majd ezt adjuk át a `tail`-nek, hogy csak az utolsó 3 sort mutassa:

```bash
nl <filename> | tail -3
```

##Forrás
-[https://linuxhandbook.com/tail-command/]

##Forrás
-[https://labex.io/tutorials/linux-linux-tail-command-file-end-display-214303]
