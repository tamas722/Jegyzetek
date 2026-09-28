# Rendszererőforrás-használat szabályozása a `ulimit` paranccsal

A `ulimit` egy beépített shell parancs, amelyet erőforrások megjelenítésére, kiosztására és korlátozására terveztek. Ismerd meg, hogyan kell használni a `ulimit` parancsot Linuxon.

**2020. április 25. · Szerző: Christopher Murray · 4 perc olvasás**

Ebben a kezdő útmutatóban megismerkedsz a `ulimit` paranccsal Linuxon. Megtudod, hogyan használhatod a rendszererőforrás-kiosztások szabályozására.

## Mi az a `ulimit` Linuxon?

A `ulimit` egy beépített shell parancs, amelyet erőforrások megjelenítésére, kiosztására és korlátozására terveztek. Elengedhetetlen, hogy bármely rendszer szabályozni tudja az ilyen típusú vezérléseket.

Ez a fajta szabályozás globális, csoport- és felhasználói szinten érvényesíthető. Amellett, hogy biztosítja a feladatok zökkenőmentes feldolgozását, megakadályozza, hogy a nem kívánt folyamatok felemésszék a rendszer erőforrásait, például a RAM-ot és a CPU-teljesítményt.

A `ulimit` egy biztonsági konfigurációs fájlhoz kapcsolódik. A pontos hely változhat, de általában valami ilyesmi: `/etc/security/limits.conf`. A `ulimit` lehetővé teszi, hogy ezt a konfigurációt gyorsan szerkesszük.

## Soft vs. Hard limitek

Felhasználóként valójában módosíthatod a `ulimit` beállításaidat.

Talán azon gondolkodsz, miért érdemes egyáltalán limitet beállítani, ha a felhasználó módosíthatja. Itt jönnek képbe a soft és hard limitek.

Tehát adminisztrátori szemszögből lehet, hogy inkább azt szeretnéd, hogy a felhasználód egy bizonyos érték körül mozogjon. Ez lenne a soft limit (mondjuk 25).

Aztán létrehozhatsz egy hard limitet, amelyet az a felhasználó nem léphet túl (50). A felhasználó jogosult lenne a limitjét 25-ről 50-ig növelni.

## A `ulimit` használata Linuxon

### A `ulimit` parancs

Íme a `ulimit` parancs szintaxisa:

```bash
ulimit <options>
```

### Az összes limit megjelenítése bármely felhasználóhoz

Mindenféle limitet megjeleníthetsz egy megadott felhasználóhoz ilyen módon:

```bash
ulimit -a user_name
```

Az `-a` kapcsoló megjeleníti az összes opciót és azok konfigurációját a megadott felhasználónevedhez.

Ha elhagyod a `user_name`-t, a saját limitjeidet mutatja. Hadd mutassam meg a számomra alapértelmezésben beállított limiteket:

```plaintext
christopher@linux-handbook:~$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 31503
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 31503
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

Az alapértelmezett értékeid természetesen eltérhetnek az enyémektől. Ez a nézet egy leírást, a hozzárendelt kapcsolót (amely a limitek módosítására használható) és a konfigurációt jeleníti meg.

### Hard és soft limitek megjelenítése

Lehetőség van ezeknek a limiteknek a külön-külön való megtekintésére is egy kapcsolóval.

A soft limitek megjelenítéséhez használd a `-S` opciót:

```bash
ulimit -S
```

A hard limitek megjelenítéséhez használd a `-H` opciót:

```bash
ulimit -H
```

Hasznosabb ezeket kombinálni a fenti konkrét kapcsolókkal. Tehát, ha ellenőrizni szeretnéd a felhasználói folyamatok maximális számának hard limitjét, ezt írnád be:

```plaintext
christopher@linux-handbook:~$ ulimit -Hu
31503
```

### A limitek módosítása (ideiglenesen)

Most változtassuk meg ezt az értéket 31500-ra demonstrációs célból, és ellenőrizzük újra a hard limitet.

```plaintext
christopher@linux-handbook:~$ ulimit -u 31500
christopher@linux-handbook:~$ ulimit -Hu
31500
```

Érdemes megjegyezni, hogy a jogosultságaid által engedélyezett bármely változtatás csak ideiglenesen íródik, és az aktuális shelledre hat.

Ennek megerősítéséhez kiléptem a shellből, és létrehoztam egy új terminált, és visszakaptam az eredeti alapértelmezett értéket.

```plaintext
christopher@linux-handbook:~$ ulimit -Hu
31503
```

A következő szakasz bemutatja, hogyan lehet a változtatásokat véglegesíteni.

### A `ulimit` végleges módosítása

Ahogy a cikk elején említettem, a `ulimit`-et a `/etc/security/limits.conf` szabályozza. Ha bármely felhasználó `ulimit`-jein végleges változtatásokat szeretnél végrehajtani, rootként kell módosítanod a biztonsági fájlt.

Szerkesztéskor a következő négy elemet kell megadnod:

```plaintext
<domain> <type> <item> <value>
```

Íme a szöveg, amelyet a fájlhoz fűztem, hogy beállítsam a folyamatok számának hard limitjét a `christopher` felhasználóhoz (azaz a saját fiókomhoz):

```plaintext
christopher    hard    nproc   2000
```

Ne feledd, hogy bevált gyakorlat, hogy ne engedélyezd a root fiókot, hacsak nem vagy teljesen tisztában a lehetséges következményekkel. Ezt egy virtuális gépen tettem, hogy neked ne kelljen megtenned.

```plaintext
christopher@linux-handbook:~$ su
Password: 
root@linux-handbook:/home/christopher# nano /etc/security/limits.conf 
root@linux-handbook:/home/christopher# exit
exit
christopher@linux-handbook:~$ ulimit -u
20000
```

Amint láthatod, a `christopher` limitje 20000-re változott.

Honnan tudod, milyen kulcsszót kell használni a `/etc/security/limits.conf` fájl szerkesztéséhez? Íme egy táblázat, amely tartalmazza a lehetséges elem-kulcsszavakat és azok leírásait:

| Elem kulcsszó | Leírás |
| --- | --- |
| core | a core fájl méretét korlátozza (KB) |
| data | maximális adatméret (KB) |
| fsize | maximális fájlméret (KB) |
| memlock | maximális zárolt memóriacím-terület (KB) |
| nofile | maximális nyitott fájlleírók száma |
| rss | maximális rezidens halmaz mérete (KB) |
| stack | maximális veremméret (KB) |
| cpu | maximális CPU-idő (MIN) |
| nproc | maximális folyamatszám |
| as | címtér-korlát (KB) |
| maxlogins | maximális bejelentkezések száma ehhez a felhasználóhoz |
| maxsyslogins | maximális bejelentkezések száma a rendszeren |
| priority | a felhasználói folyamat futtatásának prioritása |
| locks | a felhasználó által birtokolható fájlzárak maximális száma |
| sigpending | maximális függőben lévő szignálok száma |
| msgqueue | POSIX üzenetsorok által használt maximális memória (bájt) |
| nice | a maximálisan megengedett nice prioritás értékek: [-20, 19] |
| rtprio | maximális valós idejű prioritás |
| chroot | root módosítása könyvtárra (Debian-specifikus) |

| Limit típus | Leírás |
| --- | --- |
| hard | hard limit |
| soft | soft limit |
| – | mind hard, mind soft limit |

### A `ulimit` módosítása csoportokhoz

Egy csoportházirend módosítása nagyon hasonló ahhoz, amit az előző szakaszban láttál, de a csoport neve elé egy `@` szimbólumot kell tenned.

Íme egy példa, ahol a `student` csoport maximális bejelentkezéseinek számát 4-re állítottam a `/etc/security/limits.conf` fájl szerkesztésével:

```plaintext
@student    -   maxlogins   4
```

## Következtetés

Remélem, tetszett ez a gyors bevezető a `ulimit` Linux parancsba.

Ha bármilyen megjegyzésed vagy kérdésed van, hagyd alább. Ha bármilyen javaslatod van a fedni kívánt témákra, nyugodtan hagyd azokat is.

## Forrás
-[https://linuxhandbook.com/ulimit-command/]


# Felhasználói limitek ellenőrzése és beállítása a `ulimit` Linux paranccsal

**2025. szeptember 22. · Szerző: Luke Reynolds**

A `ulimit` parancsot Linuxon arra használják, hogy korlátozzák az egyes felhasználók által elfogyasztható rendszererőforrások mennyiségét.

Akár szándékos, akár véletlen, egyetlen felhasználó is felemésztheti az összes rendelkezésre álló rendszererőforrást, például a RAM-memóriát vagy a lemezterületet. A Linux rendszered jellegétől függően érdemes lehet a felhasználóidat csak arra korlátozni, amire valóban szükségük lehet.

Ez egy alapvető finomhangolás, amelyet általában virtuális privát szervereken vagy más megosztott hosting helyzetekben végeznek, ahol egyetlen szervert több tucat felhasználónak adnak bérbe. Ebben az útmutatóban példákon keresztül megtanulod, hogyan kell használni a `ulimit` parancsot Linuxon. Kövesd az alábbiakat, hogy megismerd a paranccsal használható különféle opciókat.

Ebben az útmutatóban megtanulod:

- Hogyan kell használni a `ulimit` parancsot Linuxon

## Szoftverkövetelmények és Linux parancssori konvenciók

| Kategória | Követelmények, konvenciók vagy használt szoftververzió |
| --- | --- |
| Rendszer | Bármely Linux disztribúció |
| Szoftver | `ulimit` |
| Egyéb | Emelt szintű hozzáférés a Linux rendszeredhez rootként vagy a `sudo` paranccsal |
| Konvenciók | `#` – a megadott linux parancsokat root jogosultsággal kell végrehajtani, akár közvetlenül root felhasználóként, akár a `sudo` parancs használatával |
| | `$` – a megadott linux parancsokat normál, nem privilegizált felhasználóként kell végrehajtani |

## Gyakran használt opciók

A fork bombák, az olyan parancsok, mint a `yes`, és a hibás programok könnyen hatalmas mennyiségű rendszererőforrást használhatnak fel nagyon rövid idő alatt. Ez kihat az összes többi felhasználó élményére a rendszeren, csak azért, mert egy ember visszaél a rendszererőforrásokkal (akár szándékosan, akár nem).

A `ulimit` parancs a megoldás erre a problémára. Nézd meg az alábbi `ulimit` példák közül néhányat, hogy lásd, hogyan lehet korlátozni a felhasználóid rendszererőforrás-fogyasztását.

## A `ulimit` parancs alapvető példái Linuxon

> **TUDTAD?**
> Az alább látható összes limit csak az aktuális bash shell munkamenetre vonatkozik. Ahhoz, hogy rendszerszintű, állandó változtatást hajts végre, módosítanod kell a `/etc/security/limits.conf` fájlt. Megmutatjuk, hogyan kell ezt tenni egy későbbi szakaszban.

- Használd a `ulimit` parancsot a `-a` opcióval, hogy lásd a felhasználód összes limitjének listáját a rendszeren, vagy csak a `ulimit`-et önmagában, hogy egy gyors összefoglalót láss.

  ```bash
  $ ulimit -a
  ```

- A felhasználó által indítható folyamatok maximális számának korlátozásához használd a `-u` opciót. Ez a parancs 10 folyamatra korlátozza őket. Ez gondoskodik az olyan dolgokról, mint a fork bombák.

  ```bash
  $ ulimit -u 10
  ```

  Ha megpróbáljuk túllépni ezt a limitet, hibába ütközünk:

  ```plaintext
  bash: fork: retry: Resource temporarily unavailable
  ```

- A `-f` opció lehetővé teszi a felhasználó által létrehozható fájl méretének korlátozását. Ez a parancs 100 KB-os vagy kisebb fájlokra korlátozza a felhasználót.

  ```bash
  $ ulimit -f 100
  ```

  És íme, mi történik, ha most megpróbáljuk túllépni a limitet.

  ```bash
  $ cat /dev/zero > file
  File size limit exceeded (core dumped)
  $ ls -lh file
  -rw-rw-r--. 1 linuxconfig linuxconfig 100K Feb 21 18:27 file 
  ```

- A `ulimit`-tel az is lehetséges, hogy korlátozzuk egy folyamat számára elérhető maximális virtuális memória mennyiségét. Használd a `-v` opciót ennek eléréséhez.

  ```bash
  $ ulimit -v 1000
  ```

- A felhasználó által egyszerre nyitva tartható fájlok számának (fájlleírók) korlátozásához használd a `-n` opciót.

  ```bash
  $ ulimit -n 10
  ```

  Amikor ezt a limitet túllépik, a következőhöz hasonló hibát kapsz:

  ```plaintext
  bash: command: Too many open files
  ```

  Ha a „too many open files” hibába ütközöd, lehet, hogy módosítanod kell ezeket a limiteket.

Mostanra már érted, hogy bármelyik opciót használhatod újfajta limitek létrehozására. Használd a `ulimit --help`-et vagy csak a `ulimit -a`-t, hogy lásd a kiszabható limitek típusait és a hozzájuk tartozó parancsopciókat.

> **MEGJEGYZÉS**
> Mindig használhatod a `man` parancsot, hogy többet olvass a `ulimit` parancsról és annak hivatalos dokumentációjáról. Kattints az előző linkre, hogy lásd, hogyan kell megnyitni bármely parancs kézikönyvoldalát Linux rendszeren.

## Haladó használat

Eddig „soft” limiteket szabtunk ki a `ulimit` paranccsal. Ezek olyan limitek, amelyeket maguk a felhasználók hoznak létre, és ameddig a „hard” limit engedi, addig növelhetik őket. „Hard” limit létrehozásához szerkeszteni fogjuk a `/etc/security/limits.conf` fájlt.

1. Használj root jogosultságokat a `/etc/security/limits.conf` fájl megnyitásához. Ebben a példában a `nano`-t fogjuk használni, de használhatsz más szövegszerkesztőt is, ha szeretnél.

   ```bash
   $ sudo nano /etc/security/limits.conf
   ```

2. A fájl alján néhány hard limitet szabunk ki a felhasználóinkra, amelyek rendszerszinten érvényesek. Az általános szintaxis így néz ki:

   ```plaintext
   [domain] [type] [item] [value]
   ```

   A `[domain]` lehet felhasználónév, csoportnév vagy helyettesítő karakter. A `[type]` lehet „soft” vagy „hard” attól függően, hogy melyik fajta limitet szeretnéd érvényesíteni. Az `[item]` az, amit korlátozni szeretnél, például „cpu” vagy „nproc” (folyamatok száma) stb. A `[value]` az a szám, amelyet érvényesíteni szeretnél.

   Mindez az információ a fájlban is össze van foglalva, arra az esetre, ha elfelejtenéd.

3. Példaként próbáljunk meg egy limitet beállítani a „corporate” csoportban lévő felhasználók által indítható folyamatok számára. Ennek a sornak a fájlhoz adása csak legfeljebb 10 folyamat indítását tenné lehetővé a csoport felhasználóinak.

   ```plaintext
   @corporate        hard    nproc           20
   ```

4. Egy másik példaként korlátozzuk a `linuxconfig` felhasználó által használható fájlméret mennyiségét. Ez 10 MB-ra korlátozza a felhasználót. A számot 10000-ként kell beírnunk, mivel kilobájtban kell lennie.

   ```plaintext
   linuxconfig        hard    fsize           10000
   ```

Használd a `/etc/security/limits.conf` fájlban leírt limit-típusok bármelyikét, hogy limiteket szabj ki felhasználókra, csoportokra vagy a teljes rendszer minden felhasználójára.

## Záró gondolatok

Ebben az útmutatóban mindent megtanultunk a `ulimit` parancsról Linuxon. A `ulimit` parancs elengedhetetlen a több felhasználóval rendelkező rendszerekhez, hogy biztosítsuk, mindenki tisztességesen játszik, és csak a rendszererőforrások rá eső részét használja.

Amikor a limitjeidnek tartósnak kell lenniük, szerkesztheted a `ulimit` konfigurációs fájlt a fentiek szerint, hogy rendszerlimiteket szabj ki felhasználókra, csoportokra vagy a rendszer minden fiókjára.

## Forrás
-[https://linuxconfig.org/limit-user-environment-with-ulimit-linux-command]
-[https://www.dummies.com/article/technology/computers/operating-systems/linux/10-linux-troubleshooting-tips-274301/]
