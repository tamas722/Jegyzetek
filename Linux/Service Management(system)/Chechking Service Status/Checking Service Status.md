# Szolgáltatáskezelés

`service` `systemctl` `systemd`

## 1. Áttekintés

A szolgáltatások olyan programok, amelyek gyakran a háttérben futnak. A szolgáltatások másik jellemzője, hogy általában a rendszer inicializálási folyamata során indulnak el. Amikor elindulnak, az init program, amely a rendszer inicializálásáért felelős, szintén elindítja ezeket a szükséges szolgáltatásokat. Ráadásul az init programnak többféle implementációja létezik. Ezen implementációk többsége biztosít módot az összes szolgáltatás és azok állapotának listázására.

Ebben az útmutatóban megvizsgáljuk, hogyan listázhatjuk a szolgáltatásokat és azok állapotát a népszerű szolgáltatáskezelőkben. Azt is megnézzük, hogyan találhatjuk meg egy szolgáltatás állapotát, ha nem ismerjük a nevét.

## 2. SysV inicializálás

Ez a régebbi inicializálási folyamat, amely egykor elterjedt volt. A szolgáltatásokat inicializáló shell szkriptek indítják el, amelyek a `/etc/init.d` könyvtárban találhatók.

### 2.1. A `/etc/init.d` könyvtár

Tehát, ha nem ismerjük egy SysV alatti szolgáltatás pontos nevét, az egyik módja annak megtalálására, hogy egy szkriptet keresünk a `/etc/init.d` könyvtárban:

```bash
$ ls -al /etc/init.d | grep ssh
-rwxr-xr-x    1 root     root          3939 Mar 30  2022 ssh
```

Itt listáztuk a `/etc/init.d` könyvtár összes fájlját, és a kimenetet a `grep` paranccsal és az `ssh` karakterlánccal szűrtük. Valóban, a kimenetben láthatjuk, hogy van egy `ssh` nevű szkript. Ez bizonyíték arra, hogy a rendszerünkön van SSH szolgáltatás. Ráadásul gyakran lekérdezhetjük a szolgáltatás állapotát a `status` opcióval:

```bash
$ /etc/init.d/ssh status
sshd is running.
```

Valóban, a szkript kiírta, hogy a szolgáltatás fut.

### 2.2. A `service` parancs

A `service` parancs egy felület a szolgáltatások kezeléséhez. Valójában a `service` is egy shell szkript. A háttérben azokat a `/etc/init.d` könyvtárban lévő shell szkripteket használja, amelyeket korábban láttunk. Sőt, van egy hasznos opciója, a `--status-all`, amely kiírja az összes rendszerszolgáltatás listáját és azok állapotát.

A `--status-all` opció az összes `/etc/init.d` alatti szkriptet lefuttatja a `status` opcióval, és az eredményt a szabványos kimenetre írja:

```bash
$ service --status-all
 [ - ]  nginx
 [ + ]  ssh
 [ ? ]  ubuntu-fan
 ...
```

Amint láthatjuk, a `--status-all` opció kiírta a szolgáltatások listáját. Minden szolgáltatás előtt egy speciális jel van zárójelben:

- `[+]`: fut
- `[-]`: nem fut
- `[?]`: a `status` opció nincs implementálva

Tehát a fenti példában van egy `nginx` szolgáltatás, amely nem fut, egy `ssh` szolgáltatás, amely fut, és egy `ubuntu-fan` szolgáltatás, amelynek az inicializálási szkriptjében nincs implementálva a `status` opció.

Fontos, hogy a `service` parancs fejlődött, hogy a systemd inicializálási rendszerrel is működjön, amelyet a következő szakaszokban vizsgálunk meg. Valójában a SysV inicializálási rendszert nagyrészt felváltotta a systemd. Ennek ellenére továbbra is használhatjuk a `service` parancsot egy szolgáltatás állapotának megkeresésére. Ez azért igaz, mert a `service` ellenőrzi, hogy a rendszer systemd-vel indult-e, és meghívja a megfelelő systemd parancsot az állapot lekéréséhez.

## 3. A systemd rendszerkezelő

Jelenleg a legtöbb Linux disztribúció a systemd-t használja. A systemd kezelő speciális konfigurációs fájlokat használ shell szkriptek helyett. Ezen fájlok neve a `.service` karakterlánccal végződik. Ez megkülönbözteti őket más egységektől, amelyeket a systemd kezel, mint például a socketek, mountok és mások.

### 3.1. Konfigurációs fájlok

Érdekes módon a systemd sok könyvtárat vizsgál át, hogy megtalálja a szolgáltatás konfigurációs fájlokat. Példák erre a `/etc/systemd/system` és a `/run/systemd/system`. Hogy képet kapjunk a tartalomról, listázzuk a `/etc/systemd/system` könyvtárat:

```bash
$ ls /etc/systemd/system | grep .service$
...
nginx.service
snap.lxd.activate.service
snap.lxd.daemon.service
sshd.service
syslog.service
vmtoolsd.service
```

Itt a kimenetet úgy szűrtük, hogy csak a `.service` utótaggal végződő fájlokat jelenítse meg. Ebben az esetben a fájl neve megegyezik a szolgáltatás nevével.

### 3.2. A `systemctl list-units` parancs

Ahelyett, hogy könyvtárakban keresnénk egy szolgáltatást, jobb módszer a `systemctl` parancs használata. A `list-units` opció hasznos a systemd által kezelt egységek kiírásához:

```bash
$ systemctl list-units --type=service | grep ssh
  ssh.service                                    loaded active running OpenBSD Secure Shell server
```

Itt a `--type` opciót használtuk, hogy csak a `service` típusú egységeket írjuk ki. Továbbá a `grep` paranccsal és az `ssh` karakterlánccal szűrtük a szolgáltatásokat. Valóban, van egy `ssh.service` egység, amely be van töltve, aktív és fut. Ráadásul a parancs kiírta a szolgáltatás rövid leírását.

### 3.3. A `systemctl status` parancs

Egy alternatív mód a `systemctl` parancs `status` opciójának használata:

```bash
$ systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2023-04-07 16:03:37 EEST; 2 days ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 1262 (sshd)
      Tasks: 1 (limit: 1053)
     Memory: 4.6M
     CGroup: /system.slice/ssh.service
             └─1262 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups

Apr 07 16:11:34 handsome-tattler sshd[1746]: Accepted publickey for ubuntu from 172.27.192.1 port 56487 ssh2: RSA SHA256:mUySiSiRXHAP2ZJWNJOWhmnZwW95e1apeRPpf818Zbg
Apr 07 16:11:34 handsome-tattler sshd[1746]: pam_unix(sshd:session): session opened for user ubuntu by (uid=0)
...
```

Láthatjuk, hogy a parancs sok információt írt ki a szabványos kimenetre:

- ha a szolgáltatás be van töltve a memóriába, megkapjuk az egység konfigurációs fájl elérési útját és az aktuális betöltési állapotot (`enabled`), ami azt jelenti, hogy a rendszer indításakor elindul
- aktiválási (`active`) és futási (`running`) állapot a kezdés dátumával együtt
- kézikönyv információ
- folyamat ID
- feladatok
- memóriahasználat
- cgroup, amelyhez a szolgáltatás tartozik
- szolgáltatás kimeneti naplója

Abban az esetben, ha ismerjük a folyamat ID-t és nem a szolgáltatás nevét, akkor is használhatjuk a `status` opciót:

```bash
$ systemctl status 1262
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2023-04-07 16:03:37 EEST; 3 days ago
...
```

Valóban, láthatjuk, hogy a `status` opció tökéletesen működik a szolgáltatás folyamat ID-jával.

## 4. Az OpenRC szolgáltatáskezelő

Az OpenRC egy másik szolgáltatáskezelő, amely általában a Gentoo disztribúciókban található. Ráadásul telepíthető a Debian tárolókból is.

Az OpenRC szolgáltatáskezelőnek van egy `rc-status` parancsa, amely kiírja az összes szolgáltatás állapotát:

```bash
$ rc-status --all
...
Runlevel: sysinit
 udev                                                                                                                  [  stopped  ]
 procps                                                                                                                [  stopped  ]
 networking                                                                                                            [  stopped  ]
 hwclock.sh                                                                                                            [  stopped  ]
 kmod                                                                                                                  [  stopped  ]
 cgroups                                                                                                               [  stopped  ]
Runlevel: shutdown
Runlevel: recovery
...
```

Itt láthatjuk, hogy a szolgáltatások a rendszerindítási szint (runlevel) szerint vannak csoportosítva, amelyen a rendszer elindítja őket. Továbbá láthatjuk a szolgáltatás állapotát zárójelben.

Egy másik parancs, amelyet egy szolgáltatás állapotának lekérésére használhatunk, az `rc-service` parancs a `status` opcióval:

```bash
$ rc-service ssh status
sshd is running.
```

Ebben a példában egy szolgáltatás nevét adtuk meg, hogy lekérjük az állapotát. Tehát még mindig előre kell ismernünk a szolgáltatás pontos nevét.

## 5. A rendszer inicializálási módjának meghatározása

Nem triviális meghatározni, hogyan inicializálódik a rendszer. A probléma arra vezethető vissza, hogy meghatározzuk, az init program systemd, SysV vagy valami más-e. Az egyik módja, hogy megkeressük az init program fájlt, és ellenőrizzük, hogy egy másik fájlra mutató link-e. Általában az init program a `/sbin/init` helyen található:

```bash
$ ls -al /sbin/init
lrwxrwxrwx    1 root     root            20 Mar  2 14:58 /sbin/init -> /lib/systemd/systemd
```

Itt láthatjuk, hogy az init program a systemd-re mutat. Tehát ebben az esetben a rendszerünk systemd-t használ.

Egy másik módja, hogy ellenőrizzük az init folyamatot a `pstree` paranccsal. A `pstree` parancs egy fát ír ki a rendszer futó folyamataival. A fa gyökere az init folyamat:

```bash
$ pstree
systemd─┬─ModemManager───2*[{ModemManager}]
        ├─accounts-daemon───2*[{accounts-daemon}]
        ├─2*[agetty]
        ├─atd
...
```

Itt a `pstree` parancs egyértelműen jelzi, hogy az init folyamat a systemd-t futtatja.

## 6. Következtetés

Ebben az útmutatóban megtanultuk, hogyan találhatjuk meg egy szolgáltatás állapotát, ha nem ismerjük a nevét. Különösen három rendszert vizsgáltunk meg:

- SysV
- systemd
- OpenRC

Végül két módot láttunk annak meghatározására, hogyan inicializálódik egy Linux rendszer.

##Forrás
-[https://www.baeldung.com/linux/initialization-managers-service-status]
