# `netstat` parancs – Példák

A `netstat` az egyik leggyakoribb hálózati parancs Linuxon. Ebben az útmutatóban hasznos példákat tanulhatsz a `netstat` használatára.

A `netstat` az egyik legnépszerűbb segédprogram a hálózati kapcsolatok figyelésére.

Lehetővé teszi a bejövő és kimenő kapcsolatok, a figyelő portok, a statisztikák és egyebek egyszerű megfigyelését.

Ebben az útmutatóban bemutatok néhányat a `netstat` parancs leggyakoribb példái közül Linuxon.

## 1. Az összes figyelő port megkeresése

Az összes port (TCP és UDP) megkereséséhez hozzá kell fűznöd a `-l` kapcsolót a `netstat` parancshoz:

```bash
netstat -l
```

## 2. Figyelő és nem figyelő portok listázása

Ha a rendszereden elérhető socketek listáját szeretnéd megkapni, használhatod a `-a` kapcsolót a `netstat` paranccsal:

```bash
netstat -a
```

Most térjünk rá a konkrétabbakra.

## 3. TCP figyelő portok megkeresése

Ha a TCP protokollt használó és figyelő állapotban lévő portokat szeretnéd listázni, a `-l` kapcsolót kell használnod a figyeléshez és a `-t` kapcsolót a TCP-kapcsolatokhoz:

```bash
netstat -lt
```

## 4. UDP figyelő portok megkeresése

A rendszereden lévő összes figyelő UDP-port listázásához hozzá kell fűznöd a `-l` és `-u` kapcsolókat a `netstat` parancshoz:

```bash
netstat -lu
```

## 5. Az összes TCP-portkapcsolat listázása

Ha minden TCP-kapcsolatot használó socketet listázni szeretnél, beleértve a figyelő és nem figyelő állapotúakat is, használd a `-at` kapcsolót a `netstat` paranccsal:

```bash
netstat -at
```

Szeretnéd tudni a különbséget a figyelő (listening) és a létrejött (established) állapot között?

- A **LISTENING** azt jelenti, hogy a socket bejövő kapcsolatokra vár.
- Az **ESTABLISHED** azt jelzi, hogy a socketnek létrejött kapcsolata van.

## 6. Az összes UDP-kapcsolat listázása

Ha minden UDP-t használó socketet listázni szeretnél, használhatod az `-a` és `-u` kapcsolók kombinációját:

```bash
netstat -au
```

## 7. Statisztikai összefoglaló kérése protokollonként

Ez a `netstat` egyik leghasznosabb funkciója, amely lehetővé teszi a létrejött kapcsolatok számának, az elküldött és fogadott üzenetek számának és még sok másnak a megismerését.

Ahhoz, hogy összefoglalót kapj minden protokollról, csak hozzá kell fűznöd a `-s` kapcsolót:

```bash
netstat -s
```

De mi van, ha adott protokollokra vonatkozó statisztikákat szeretnél? Íme, hogyan csináld.

## 8. Statisztikák kérése egy adott kapcsolathoz

Kezdjük a TCP-vel.

A TCP-kapcsolatok statisztikáinak megtekintéséhez csak a `-s` és `-t` kapcsolókat kell használnod a `netstat` paranccsal:

```bash
netstat -st
```

Hasonlóképpen, ha ugyanezt szeretnéd UDP-hez, a `-su` kapcsolót kell használnod:

```bash
netstat -su
```

## 9. Nyers hálózati statisztikák kérése

Ha szűrt adatok helyett nyers adatokat keresel, az könnyen előállítható a `-s` (statisztikákhoz) és a `--raw` (nyers adatokhoz) használatával:

```bash
netstat -s --raw
```

## 10. Szolgáltatások megkeresése PID-del

Ha hibaelhárítással foglalkozol, a szolgáltatás PID-jének megismerése nagyon hasznos lehet. A PID megszerzéséhez csak a `-p` kapcsolót kell használnod:

```bash
sudo netstat -p
```

## 11. Egy adott figyelő szolgáltatás megkeresése a hálózaton

Egy adott figyelő szolgáltatás megkereséséhez használhatod a `grep` parancsot, amely gyilkos kombinációt alkot hibaelhárítás közben.

Tegyük fel, hogy egy HTTPS szolgáltatást keresek figyelő állapotban, ami a következő paranccsal tehető meg:

```bash
sudo netstat -apl | grep -w https
```

Szeretnéd tudni, hogyan hozhatod ki a többet a `grep`-ből? Olvasd el a témáról szóló részletes útmutatónkat.

## 12. Hálózati interfészek tranzakcióinak megjelenítése

A `netstat` segédprogram arra is használható, hogy listázza az elérhető hálózati interfészeket és megmutassa mindegyik tranzakcióit.

Ehhez csak hozzá kell fűznöd a `-i` kapcsolót a `netstat`-hez:

```bash
netstat -i
```

## 13. A hálózat folyamatos figyelése a `netstat` paranccsal

Ha folyamatosan szeretnéd figyelni a hálózatot, megteheted a `-c` opcióval:

```bash
netstat -c
```

Használhatsz megfelelő kapcsolókat, például a `-lt`-t a `-c`-vel, és folyamatosan figyelni fogja a figyelő TCP-kapcsolatokat:

```bash
netstat -ltc
```

Elég hasznos, igaz?

## További információ a hálózatkezelésről Linuxon

Ha most kezdted a karrieredet vagy tanulmányaidat a hálózatkezelés terén, van egy részletes útmutatónk a legalapvetőbb hálózati parancsokról.

Többet szeretnél tudni a portokról? Erről is gondoskodtunk.

Ez volt a részemről. És ha bármilyen kérdésed van, vagy tippjeid vannak kezdőknek, oszd meg értékes tudásodat a megjegyzésekben.

## Forrás 
-[https://linuxhandbook.com/netstat-command/]

# A Linux `netstat` parancs – Portok, folyamatok és protokollok

A Linux `netstat` parancsa kincsesbányányi információt ad a hálózati kapcsolataidról, a használatban lévő portokról és az azokat használó folyamatokról. Ismerd meg, hogyan kell használni.

## Portok, folyamatok és protokollok

A hálózati socketek vagy csatlakoztatva vannak, vagy egy kapcsolatra várnak. A kapcsolatok olyan hálózati protokollokat használnak, mint a Transport Control Protocol (TCP) vagy a User Datagram Protocol (UDP). Internet Protocol címeket és hálózati portokat használnak a kapcsolatok létrehozásához.

A „socket” szó talán egy fizikai csatlakozási pont képét idézheti fel egy vezeték vagy kábel számára, de ebben a kontextusban a socket egy szoftverkonstrukció, amelyet egy hálózati adatkapcsolat egyik végének kezelésére használnak.

A socketeknek két fő állapotuk van: vagy csatlakoztatva vannak és egy folyamatban lévő hálózati kommunikációt tesznek lehetővé, vagy egy bejövő kapcsolatra várnak, hogy az csatlakozzon hozzájuk. Vannak más állapotok is, például amikor egy socket éppen egy távoli eszközön lévő kapcsolat létrehozásának félidejében van, de a tranziens állapotokat félretéve a socketet tekintheted csatlakoztatottnak vagy várakozónak (amit gyakran figyelőnek, azaz listeningnek neveznek).

A figyelő socketet szervernek, a figyelő sockethez kapcsolódást kérő socketet kliensnek nevezik. Ezeknek a neveknek semmi közük a hardverhez vagy a számítógépek szerepéhez. Egyszerűen meghatározzák az egyes socketek szerepét a kapcsolat mindkét végén.

A `netstat` parancs lehetővé teszi, hogy felfedezd, mely socketek csatlakoztatottak és melyek figyelnek. Ez azt jelenti, hogy megmondja, mely portok vannak használatban, és mely folyamatok használják őket. Meg tudja mutatni az útválasztási táblákat és a hálózati interfészeidről és multicast kapcsolataidról szóló statisztikákat.

A `netstat` funkcionalitását az idők során különböző Linux segédprogramokban replikálták, mint az `ip` és az `ss`. Még mindig érdemes ismerni ezt a hálózatelemző parancsok ősatyját, mert minden Linux és Unix-szerű operációs rendszeren elérhető, sőt Windows-on és Mac-en is.

Íme, hogyan kell használni, teljes példaparancsokkal.

## Az összes socket listázása a `netstat`-tal

Az `-a` (all) opció hatására a `netstat` megmutatja az összes csatlakoztatott és várakozó socketet. Ez a parancs hosszú listát eredményezhet, ezért a `less`-be irányítjuk.

```bash
netstat -a | less
```

A lista tartalmaz TCP (IP), TCP6 (IPv6) és UDP socketeket.

A terminálablakban való tördelés kissé megnehezíti annak megértését, mi történik. Íme néhány szakasz abból a listából:

```plaintext
Active Internet connections (servers and established)

Proto    Recv-Q    Send-Q    Local Address          Foreign Address       State
TCP      0         0         localhost:domain       0.0.0.0:*             LISTEN
TCP      0         0         0.0.0.0:ssh            0.0.0.0:*             LISTEN
TCP      0         0         localhost:ipp          0.0.0.0:*             LISTEN
TCP      0         0         localhost:smtp         0.0.0.0:*             LISTEN
TCP6     0         0         [::]:ssh               [::]:*                LISTEN
TCP6     0         0         ip6-localhost:ipp      [::]:*                LISTEN

Active UNIX domain sockets (servers and established)

Proto    RefCnt    Flags    Type       State        I-Node    Path
unix     24        [ ]      DGRAM                   12831     /run/systemd/journal/dev-log
unix     2         [ ACC ]  STREAM     LISTENING    24747     @/tmp/dbus-zH6clYmvw8
unix     2         [ ]      DGRAM                   26372     /run/user/1000/system/notify
unix     2         [ ]      DGRAM                   23382     /run/user/121/system/notify
unix     2         [ ACC ]  SEQPACKET  LISTENING    12839     /run/udev/control
```

Az „Active Internet” szakasz a csatlakoztatott külső kapcsolatokat és a távoli kapcsolatkérésekre figyelő helyi socketeket listázza. Vagyis azokat a hálózati kapcsolatokat listázza, amelyek létrejöttek (vagy létre fognak jönni) külső eszközökkel.

A „UNIX domain” szakasz a csatlakoztatott és figyelő belső kapcsolatokat listázza. Más szóval azokat a kapcsolatokat listázza, amelyek a számítógépeden belül jöttek létre különböző alkalmazások, folyamatok és az operációs rendszer elemei között.

Az „Active Internet” oszlopai:

- **Proto:** Az e socket által használt protokoll (például TCP vagy UDP).
- **Recv-Q:** A fogadási sor. Ezek a bejövő bájtok, amelyeket fogadtak és puffereltek, várva, hogy az e kapcsolatot használó helyi folyamat beolvassa és feldolgozza őket.
- **Send-Q:** A küldési sor. Ez mutatja azokat a bájtokat, amelyek készen állnak a küldési sorból való elküldésre.
- **Local address:** A kapcsolat helyi végének címadatai. Alapértelmezésben a `netstat` a helyi hosztnevet mutatja a címhez, és a szolgáltatás nevét a port számára.
- **Foreign address:** A kapcsolat távoli végének címe és portszáma.
- **State:** A helyi socket állapota. UDP socketek esetében ez általában üres. Lásd az állapottáblázatot lentebb.

TCP-kapcsolatoknál a state érték a következők egyike lehet:

- **LISTEN:** Csak szerveroldalon. A socket kapcsolatkérésre vár.
- **SYN-SENT:** Csak kliensoldalon. Ez a socket kapcsolatkérést tett, és arra vár, hogy elfogadják-e.
- **SYN-RECEIVED:** Csak szerveroldalon. Ez a socket egy kapcsolat-nyugtázásra vár, miután elfogadott egy kapcsolatkérést.
- **ESTABLISHED:** Szerver és kliens. Működő kapcsolat jött létre a szerver és a kliens között, lehetővé téve az adatátvitelt a kettő között.
- **FIN-WAIT-1:** Szerver és kliens. Ez a socket egy kapcsolatbontási kérésre vár a távoli socketről, vagy egy korábban erről a socketről küldött kapcsolatbontási kérés nyugtázására.
- **FIN-WAIT-2:** Szerver és kliens. Ez a socket egy kapcsolatbontási kérésre vár a távoli socketről.
- **CLOSE-WAIT:** Szerver és kliens. Ez a socket egy kapcsolatbontási kérésre vár a helyi felhasználótól.
- **CLOSING:** Szerver és kliens. Ez a socket egy kapcsolatbontási kérés nyugtázására vár a távoli socketről.
- **LAST-ACK:** Szerver és kliens. Ez a socket a távoli socketnek küldött kapcsolatbontási kérésének nyugtázására vár.
- **TIME-WAIT:** Szerver és kliens. Ez a socket nyugtázást küldött a távoli socketnek, hogy tudassa vele, megkapta a távoli socket bontási kérését. Most arra vár, hogy megbizonyosodjon arról, hogy a nyugtázást megkapták.
- **CLOSED:** Nincs kapcsolat, így a socket megszűnt.

A „Unix domain” oszlopai:

- **Proto:** Az e socket által használt protokoll. „unix” lesz.
- **RefCnt:** Hivatkozási szám. Az ehhez a sockethez kapcsolódó csatolt folyamatok száma.
- **Flags:** Ez általában `ACC`, ami a `SO_ACCEPTON`-t képviseli, vagyis a socket kapcsolatkérésre vár. A `SO_WAITDATA`, `W`-ként megjelenítve, azt jelenti, hogy van adat, ami olvasásra vár. A `SO_NOSPACE`, `N`-ként megjelenítve, azt jelenti, hogy nincs hely adat írására a socketbe (azaz a küldési puffer megtelt).
- **Type:** A socket típusa. Lásd a típustáblázatot lentebb.
- **State:** A socket állapota. Lásd az állapottáblázatot lentebb.
- **I-Node:** Az ehhez a sockethez társított fájlrendszer-inode.
- **Path:** A socket fájlrendszerbeli elérési útja.

A Unix domain socket típusa a következők egyike lehet:

- **DGRAM:** A socket datagram módban van használatban, rögzített hosszúságú üzeneteket használva. A datagramok nem garantáltan megbízhatóak, sorrendezettek vagy duplikátummentesek.
- **STREAM:** Ez a socket egy stream socket. Ez a mindennapi „normál” socketkapcsolat-típus. Ezeket a socketeket a csomagok megbízható, sorrendezett (sorrendben történő) kézbesítésének biztosítására tervezték.
- **RAW:** Ez a socket nyers socketként van használatban. A nyers socketek az OSI-modell hálózati szintjén működnek, és nem hivatkoznak a szállítási szint TCP- és UDP-fejléceire.
- **RDM:** Ez a socket egy megbízhatóan kézbesített üzenetkapcsolat egyik végén található.
- **SEQPACKET:** Ez a socket szekvenciális csomag socketként működik, ami egy másik eszköz a megbízható, sorrendezett és duplikátummentes csomagkézbesítés biztosítására.
- **PACKET:** Nyers interfész-hozzáférési socket. A csomagsocketeket nyers csomagok fogadására vagy küldésére használják az eszközmeghajtó (azaz adatkapcsolati réteg) szintjén az OSI-modellben.

A Unix domain socket állapota a következők egyike lehet:

- **FREE:** Ez a socket nincs kiosztva.
- **LISTENING:** Ez a socket bejövő kapcsolatkérésekre figyel.
- **CONNECTING:** Ez a socket csatlakozás folyamatában van.
- **CONNECTED:** Kapcsolat jött létre, és a socket képes adatot fogadni és továbbítani.
- **DISCONNECTING:** A kapcsolat megszüntetésének folyamatában van.

Hű, ez sok információ! A `netstat` opciók közül sok így vagy úgy finomítja az eredményeket, de nem változtatja meg túlságosan a tartalmat. Nézzük meg.

## Socketek listázása típus szerint a `netstat` paranccsal

A `netstat -a` parancs több információt nyújthat, mint amennyit látni szeretnél. Ha csak a TCP socketeket szeretnéd vagy kell látnod, használhatod a `-t` (TCP) opciót, hogy a megjelenítést csak a TCP socketekre korlátozd.

```bash
netstat -at | less
```

A kimenet nagymértékben lecsökken. A listázott néhány socket mind TCP socket.

A `-u` (UDP) és `-x` (UNIX) opciók hasonló módon viselkednek, az eredményeket a parancssorban megadott socket típusára korlátozva. Íme a `-u` (UDP) opció használatban:

```bash
netstat -au | less
```

Csak UDP socketek vannak listázva.

## Socketek listázása állapot szerint

A figyelő vagy várakozó állapotban lévő socketek megtekintéséhez használd a `-l` (listening) opciót.

```bash
netstat -l | less
```

A listázott socketek azok, amelyek figyelő állapotban vannak.

Ez kombinálható a `-t` (TCP), `-u` (UDP) és `-x` (UNIX) opciókkal, hogy tovább szűkítsd a érdeklődésre számot tartó socketeket. Keressünk figyelő TCP socketeket:

```bash
netstat -lt | less
```

Most csak TCP figyelő socketeket látunk.

## Hálózati statisztikák protokollonként

Egy protokoll statisztikáinak megtekintéséhez használd a `-s` (statistics) opciót, és add át a `-t` (TCP), `-u` (UDP) vagy `-x` (UNIX) opciókat. Ha csak a `-s` (statistics) opciót használod önmagában, az összes protokoll statisztikáit látni fogod. Nézzük a TCP protokoll statisztikáit.

```bash
netstat -st | less
```

A TCP-kapcsolatok statisztikáinak gyűjteménye jelenik meg a `less`-ben.

## Folyamatnevek és PID-ek megjelenítése

Hasznos lehet látni egy socketet használó folyamat PID-jét (folyamatazonosítóját), együtt a folyamat nevével. A `-p` (program) opció pontosan ezt teszi. Nézzük meg, mik a PID-ek és folyamatnevek azon folyamatok esetében, amelyek egy figyelő állapotban lévő TCP socketet használnak. A `sudo`-t használjuk, hogy megbizonyosodjunk arról, hogy minden elérhető információt megkapunk, beleértve minden olyan információt, amely általában root jogosultságot igényelne.

```bash
sudo netstat -p -at
```

Íme a kimenet formázott táblázatban:

```plaintext
Active Internet connections (servers and established)

Proto    Recv-Q    Send-Q    Local Address          Foreign Address    State     PID/Program Name
tcp      0         0         localhost:domain       0.0.0.0:*          LISTEN    6927/systemd-resolv
tcp      0         0         0.0.0.0:ssh            0.0.0.0:*          LISTEN    751/sshd
tcp      0         0         localhost:ipp          0.0.0.0:*          LISTEN    7687/cupsd
tcp      0         0         localhost:smtp         0.0.0.0:*          LISTEN    1176/master
tcp6     0         0         [::]:ssh               [::]:*             LISTEN    751/sshd
tcp6     0         0         ip6-localhost:ipp      [::]:*             LISTEN    7687/cupsd
tcp6     0         0         ip6-localhost:smtp     [::]:*             LISTEN    1176/master
```

Kaptunk egy extra oszlopot „PID/program name” néven. Ez az oszlop az egyes socketeket használó folyamat PID-jét és nevét listázza.

## Numerikus címek listázása

Egy másik lépés, amellyel eltávolíthatunk némi kétértelműséget, hogy a helyi és távoli címeket IP-címekként jelenítjük meg a feloldott domain- és hosztnevek helyett. Ha a `-n` (numeric) opciót használjuk, az IPv4-címek pontozott decimális formátumban jelennek meg:

```bash
sudo netstat -an | less
```

Az IP-címek numerikus értékként jelennek meg. A portszámok is láthatók, kettősponttal (`:`) elválasztva az IP-címtől.

A `127.0.0.1` IP-cím azt mutatja, hogy a socket a helyi számítógép loopback címéhez van kötve. A `0.0.0.0` IP-címet úgy tekintheted, mint a helyi címek „alapértelmezett útvonalát”, és a távoli címek „bármely IP-címét”. A `::`-ként megjelenő IPv6-címek szintén mind nulla címek.

A listázott portok könnyen ellenőrizhetők, hogy lássuk, mi a szokásos céljuk:

- **22:** Ez a Secure Shell (SSH) figyelő port.
- **25:** Ez a Simple Mail Transfer Protocol (SMTP) figyelő port.
- **53:** Ez a Domain Name System (DNS) figyelő port.
- **68:** Ez a Dynamic Host Configuration Protocol (DHCP) figyelő port.
- **631:** Ez a Common UNIX Printing System (CUPS) figyelő port.

## Az útválasztási tábla megjelenítése

A `-r` (route) opció megjeleníti a kernel útválasztási tábláját.

```bash
sudo netstat -r
```

Íme a kimenet rendezett táblázatban:

```plaintext
Kernel IP routing table

Destination    Gateway        Genmask          Flags    MSS    Window    irtt    Iface
default        Vigor.router   0.0.0.0          UG       0      0         0       enp0s3
link-local     0.0.0.0        255.255.0.0      U        0      0         0       enp0s3
192.168.4.0    0.0.0.0        255.255.255.0    U        0      0         0       enp0s3
```

És íme, mit jelentenek az oszlopok:

- **Destination:** A célhálózat vagy célgazdagép-eszköz (ha a cél nem egy hálózat).
- **Gateway:** Az átjáró címe. Csillag (`*`) jelenik meg itt, ha nincs beállítva átjárócím.
- **Genmask:** Az útvonal alhálózati maszkja.
- **Flags:** Lásd a jelzőtáblázatot lentebb.
- **MSS:** Az alapértelmezett Maximum Segment Size a TCP-kapcsolatokhoz ezen az útvonalon – ez a legnagyobb adatmennyiség, amely egy TCP-szegmensben fogadható.
- **Window:** Az alapértelmezett ablaksméret a TCP-kapcsolatokhoz ezen az útvonalon, jelezve a csomagok számát, amelyek átvihetők és fogadhatók, mielőtt a fogadási puffer megtelik. A gyakorlatban a csomagokat a fogadó alkalmazás fogyasztja el.
- **irtt:** Az Initial Round Trip Time. Ezt az értéket a kernel hivatkozza a TCP-paraméterek dinamikus módosításához olyan távoli kapcsolatok esetében, amelyek lassan válaszolnak.
- **Iface:** Az a hálózati interfész, amelyről az ezen az útvonalon küldött csomagok továbbításra kerülnek.

A flags érték a következők egyike lehet:

- **U:** Az útvonal aktív.
- **H:** A cél egy hoszt, és ez az egyetlen lehetséges cél ezen az útvonalon.
- **G:** Használd az átjárót.
- **R:** Az útvonal visszaállítása dinamikus útválasztáshoz.
- **D:** Dinamikusan telepítette az útválasztási démon.
- **M:** Az útválasztási démon módosította, amikor Internet Control Message Protocol (ICMP) csomagot kapott.
- **A:** Az `addrconf` telepítette, az automatikus DNS- és DHCP-konfigurációs fájlgenerátor.
- **C:** Gyorsítótár-bejegyzés.
- **!:** Útvonal elutasítása.

## Egy folyamat által használt port megkeresése

Ha a `netstat` kimenetét a `grep`-en keresztül irányítjuk, kereshetünk egy folyamatot név szerint, és azonosíthatjuk az általa használt portot. A korábban használt `-a` (all), `-n` (numeric) és `-p` (program) opciókat használjuk, és az „sshd” kifejezésre keresünk.

```bash
sudo netstat -anp | grep "sshd"
```

A `grep` megtalálja a célkarakterláncot, és látjuk, hogy az `sshd` démon a 22-es portot használja.

Természetesen ezt fordítva is megtehetjük. Ha a `:22`-re keresünk, megtudhatjuk, melyik folyamat használja azt a portot, ha egyáltalán használja valamelyik.

```bash
sudo netstat -anp | grep ":22"
```

Ezúttal a `grep` megtalálja a `:22` célkarakterláncot, és látjuk, hogy az ezt a portot használó folyamat az `sshd` démon, 751-es folyamatazonosítóval.

## A hálózati interfészek listázása

Az `-i` (interfaces) opció megjeleníti a `netstat` által felfedezhető hálózati interfészek táblázatát.

```bash
sudo netstat -i
```

Íme a kimenet olvashatóbb formában:

```plaintext
Kernel Interface table

Iface    MTU      RX-OK     RX-ERR   RX-DRP   RX-OVR   TX-OK     TX-ERR   TX-DRP   TX-OVR   Flg
enp0s3   1500     4520671   0        0        0        4779773   0        0        0        BMRU
lo       65536    30175     0        0        0        30175     0        0        0        LRU
```

Ez az, amit az oszlopok jelentenek:

- **Iface:** Az interfész neve. Az `enp0s3` interfész a külvilág felé vezető hálózati interfész, a `lo` interfész pedig a loopback interfész. A loopback interfész lehetővé teszi, hogy a folyamatok a számítógépen belül kommunikáljanak egymással hálózati protokollok segítségével, még akkor is, ha a számítógép nincs hálózathoz csatlakoztatva.
- **MTU:** A Maximum Transmission Unit (MTU). Ez a legnagyobb „csomag”, amely elküldhető. Egy fejlécből áll, amely útválasztási és protokolljelzőket, valamint egyéb metaadatokat tartalmaz, plusz a ténylegesen szállított adatokat.
- **RX-OK:** A fogadott csomagok száma, hibák nélkül.
- **RX-ERR:** A fogadott csomagok száma, hibákkal. Azt szeretnénk, hogy ez a lehető legalacsonyabb legyen.
- **RX-DRP:** Az elejtett (azaz elveszett) csomagok száma. Ezt is a lehető legalacsonyabbnak szeretnénk.
- **RX-OVR:** A fogadáskor túlcsordulás miatt elveszett csomagok száma. Ez általában azt jelenti, hogy a fogadási puffer megtelt, és nem tudott több adatot fogadni, de több adat érkezett, és el kellett dobni. Minél alacsonyabb ez a szám, annál jobb, és a nulla a tökéletes.
- **TX-OK:** Az elküldött csomagok száma, hibák nélkül.
- **TX-ERR:** Az elküldött csomagok száma, hibákkal. Azt szeretnénk, hogy ez nulla legyen.
- **TX-DRP:** Az elküldéskor elejtett csomagok száma. Ideálisan nullának kell lennie.
- **TX-OVR:** Az elküldéskor túlcsordulás miatt elveszett csomagok száma. Ez általában azt jelenti, hogy a küldési puffer megtelt, és nem tudott több adatot fogadni, de több adat állt készen az átvitelre, és el kellett dobni.
- **Flg:** Jelzők. Lásd a jelzőtáblázatot lentebb.

A jelzők a következőket képviselik:

- **B:** Broadcast cím van használatban.
- **L:** Ez az interfész egy loopback eszköz.
- **M:** Minden csomag fogadása (azaz promiscuous módban). Semmi sincs szűrve vagy eldobva.
- **O:** Az Address Resolution Protocol (ARP) ki van kapcsolva ezen az interfészen.
- **P:** Ez egy Point-to-Point (PPP) kapcsolat.
- **R:** Az interfész fut.
- **U:** Az interfész aktív.

## Multicast csoporttagságok listázása

Egyszerűen fogalmazva, a multicast átvitel lehetővé teszi, hogy egy csomagot csak egyszer küldjenek el, függetlenül a címzettek számától. Olyan szolgáltatásoknál, mint például a videóstreamelés, ez hatalmas mértékben növeli a hatékonyságot a küldő szempontjából.

A `-g` (groups) opció hatására a `netstat` listázza az egyes interfészeken lévő socketek multicast csoporttagságát.

```bash
sudo netstat -g
```

Az oszlopok meglehetősen egyszerűek:

- **Interface:** Annak az interfésznek a neve, amelyen keresztül a socket továbbít.
- **RefCnt:** A hivatkozási szám, amely a sockethez kapcsolódó folyamatok száma.
- **Group:** A multicast csoport neve vagy azonosítója.

## Forrás
-[https://www.howtogeek.com/513003/how-to-use-netstat-on-linux/]
