# CCNA Bővített Jegyzet – 39. Lecke: Network Time Protocol (NTP)

Ebben a leckében rátérünk az "IP Services" (IP Szolgáltatások) modul első vizsgatémájára: a hálózati időszerverekre, azaz az **NTP**-re (Network Time Protocol).

Miért olyan fontos, hogy a routereink, switcheink órája percre pontosan ugyanazt az időt mutassa? Képzeld el, hogy a hálózatban éjjel 1-kor történt egy kritikus OSPF kábelszakadás. Ha az egyik router 2008-as dátumot mutat a naplóiban (Syslog), a másik pedig 2020-ast, ember legyen a talpán, aki össze tudja fésülni a két eszköz ezer soros hibanaplóját (Logs) egy folyamattá. Az NTP feladata tehát az egységes, milliszekundum-pontosságú, összehasonlítható hibakeresési (Troubleshooting) időbélyegek garantálása!

## 1. Az Óra manuális beállítása (Szoftver és Hardver)

Ha valamiért nem akarunk NTP-t használni, a routereken az időt kézzel is beállíthatjuk. Itt azonban figyelembe kell venni, hogy a Cisco routerekben **KÉT, egymástól független óra** ketyeg!

- **Software Clock (Szoftver óra):** Az operációs rendszer (IOS) belső ideje. Ha a router újraindul, ez elfelejti az időt, és a hardveres órából tölti be magát újra.
  - Megnézés: `show clock detail`
  - Beállítás (Privileged EXEC módból!): `clock set 15:30:00 27 dec 2020`
- **Hardware Calendar (Hardver óra):** A fizikai alaplapi óra (mint a BIOS-ban a gombelem). Ez túléli az áramszünetet is.
  - Megnézés: `show calendar`
  - Beállítás (Privileged EXEC módból!): `calendar set 15:30:00 27 dec 2020`

### A két óra szinkronizálása

Mivel a két óra eltérhet, manuálisan rá tudjuk erőltetni az egyiket a másikra:

- `clock update-calendar`: A szoftveres órát beírja a hardverbe.
- `clock read-calendar`: A hardveres órát betölti a szoftverbe.

### Időzóna (Timezone) és Nyári Időszámítás (DST) beállítása

(Fontos! Az időzónát és a nyári időszámítást **Globális Konfigurációs módból** (Config) kell beállítani, mert ezek bekerülnek a Running Config fájlba!).

```text
clock timezone JST 9
```

(Példa a japán időzónára, ami a GMT-hez képest +9 óra. Az alapértelmezett időzóna a UTC!).

```text
clock summer-time EDT recurring 2 Sunday March 02:00 1 Sunday Nov 02:00
```

(Példa egy ismétlődő, kanadai nyári időszámításra, ami megmondja, mikor tekerje előre és hátra az órát).

## 2. Az NTP (Network Time Protocol) Architektúrája

A manuális beállítás nem skálázható, és a gépek órái idővel elcsúsznak. Erre találták ki az NTP-t, amely **UDP 123-as porton** (ezt tudni kell!) kommunikál, és kliens/szerver alapokon tartja szinkronban a bolygó hálózatait.

### A Stratum (Réteg) Hierarchia

Az NTP szervereket pontosságuk szerint egy 1-től 15-ig terjedő "Stratum" skálán rangsorolják. (A 16-os Stratum már halott / elérhetetlen szervert jelent, ahhoz a gép sosem fog szinkronizálni!).

- **Stratum 0 (Reference Clocks - Referencia órák):** Ezek maguk a fizikai, brutálisan drága atomórák, és a GPS műholdak. (A Cisco eszközöd sosem fog Stratum 0-hoz csatlakozni).
- **Stratum 1 (Primary Servers - Elsődleges szerverek):** Olyan NTP szerverek (pl. a Google-nél vagy egy katonai bázison), amelyek **KÖZVETLENÜL** rá vannak kötve egy atomórára!
- **Stratum 2, 3, 4 (Secondary Servers):** Egy Stratum 2-es szerver onnan kapja az idejét, hogy csatlakozik egy Stratum 1-es szerverhez. Egy Stratum 3-as a 2-estől kéri, és így tovább lefelé, egészen 15-ig. (Minden ugrással a Stratum nő 1-gyel, és a pontosság picikét romlik).

### A 3 Működési Mód Cisco routereken

- **Client Mode (Kliens Mód):** A router bekéri az időt egy külső szervertől. (Boson Extra: Ennek a módnak a pontos hivatalos neve a **Static Client Mode**).
- **Server Mode (Szerver Mód):** Amint egy router (Kliensként) megkapja az időt egy szervertől, automatikusan **Ő MAGA IS szerverré** (Stratum + 1) válik, és tovább tudja osztani az időt az alatta lévő többi routernek! (Természetesen külső forrás nélkül is megmondhatod a gépnek, hogy mostantól te vagy a szerver).
- **Symmetric Active Mode (Szimmetrikus Aktív / Peer Mód):** Két azonos (vagy hasonló) Stratum szinten lévő router "haverként" össze tud kapcsolódni, hogy ellenőrizzék egymást, és ha valamelyikük elveszítené az internetet, a másik kisegíti őt. (Nincs klasszikus főnök-beosztott viszony).

## 3. NTP Konfiguráció (Cisco IOS)

Lássuk a parancsokat, hogyan építünk fel egy pontos hálózatot! (Minden parancs Globális Konfigurációs módból indul).

**Kliens Mód (Kapcsolódás egy külső szerverhez):**

```text
Router(config)# ntp server 216.239.35.0
Router(config)# ntp server 216.239.35.4 prefer
```

(Az IP-címek helyett beírhattuk volna, hogy `time.google.com` is, de ahhoz kéne egy működő DNS szerver beállítás. A `prefer` szó a végén azt jelenti, hogy ha a router mindkét szervert eléri, de választania kell, a 4-es végűt részesítse előnyben).

**Az NTP Szomszédok/Státusz Ellenőrzése:**

- `show ntp associations`: Megmutatja, kiket adtál meg szervernek. (Ha a szerver IP címe mellett van egy `*` csillag, az azt jelenti: Ezt választottam, tőle veszem az időt! A `+` jel azt jelenti, hogy tartaléknak használható).
- `show ntp status`: Megmutatja, hogy a gépünk "szinkronizálva" (synchronized) van-e, és kiírja, hogy mi lett a **SAJÁT** gépünk Stratum szintje. (Ha a Google Stratum 1-éhez kapcsolódtál, a te géped Stratum 2-es lett!).

**A Loopback Interfész (Mint Forrás) Használata:**
Hogy a router mindig egy stabil, soha el nem tűnő IP-címen ossza tovább az időt, érdemes Loopback interfészt használni forrásként:

```text
Router(config)# ntp source loopback0
```

**Szerver Mód (Saját magad kinevezése "Főnöknek"):**
Mi van, ha nincs Interneted a cégben, de azt akarod, hogy az 50 routerből legalább az R1 legyen a főnök, akire mindenki más szinkronizál?

```text
Router(config)# ntp master
```

(Ezzel a router kinevezi magát főnöknek. A rendszer saját maga felé fog egy "Loopback" címet (`127.127.1.1`) generálni referenciának. Az alapértelmezett, gyári Stratum értéke a parancsnak 8. Ha akarod, ezt módosíthatod is: `ntp master 4`).

**Symmetric Active Mode (Peer / Haver kapcsolat):**
Az R2 és az R3 szimmetrikusan akarják védeni egymást. (Ehhez be kell állítaniuk egymást "peer"-ként).

```text
Router(config)# ntp peer 10.1.1.1
```

## 4. NTP Hitelesítés (Authentication) és DAD Emlékeztető

Ahhoz, hogy megakadályozzuk, hogy egy hacker betegyél egy kamu NTP szervert a hálózatba (és a gépeinket pl. 1999-re állítsa, megbénítva a logolást vagy a titkosítási kulcsokat), jelszavas hitelesítést (**Authentication**) használhatunk. Ez kliensen és szerveren is **4 lépésből** áll!

1. Bekapcsoljuk a védelmet: `ntp authenticate`
2. Létrehozzuk a jelszót (MD5 titkosítással): `ntp authentication-key 1 md5 TitkosJelszó`
3. Megbízhatónak jelöljük a kulcsot (Ezt nagyon szeretik kifelejteni!): `ntp trusted-key 1`
4. Ráerőltetjük a kulcsot a szerver kapcsolatra (Csak kliensen!): `ntp server 10.0.12.1 key 1`

**Hardveres naptár szinkronizáció**
Ha beállítottuk az NTP-t, az automatikusan beállítja a Szoftveres órát. De a hardvert nem piszkálja! Ahhoz, hogy az NTP ráerőszakolja a pontos időt a Hardver Órára (Calendar) is, ki kell adni a következőt:

```text
Router(config)# ntp update-calendar
```

## Kvíz a 39. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod a Network Time Protocol parancsaiból és hierarchiájából!

### Kérdés 1

Az alábbi parancsok közül melyik utasítja a routert arra, hogy olvassa ki az időt a Hardver Naptárból (Calendar), és állítsa be annak alapján a saját Szoftver Óráját (Clock)?

- A) `clock update-calendar`
- B) `ntp update-calendar`
- C) `clock read-calendar`
- D) `calendar update-clock`

**Helyes Válasz: C**

**Magyarázat:** Amikor a "read-calendar" szót látod, az az jelenti, hogy "olvasd be a naptárat magadba". Tehát a Calendar (hardver) adja az időt a Clocknak (szoftver). A `clock update-calendar` pont a fordítottját csinálná (a szoftver írná felül a hardvert).

### Kérdés 2

Melyik paranccsal tudod beállítani a router Időzónáját (Time zone)?

- A) `clock set timezone ...` (Privileged EXEC módból)
- B) `ntp timezone ...` (Globális Konfigurációs módból)
- C) `clock timezone ...` (Privileged EXEC módból)
- D) `clock timezone ...` (Globális Konfigurációs módból)

**Helyes Válasz: D**

**Magyarázat:** Szemben a `clock set` paranccsal (amit Privileged módból adunk ki, és újrainduláskor elvész), az időzóna a Running Config szerves része! Tehát a globális konfigurációs módból (Config) kell kiadni a `clock timezone` parancsot. (A B hibás, mert az NTP csakis UTC-ben dolgozik, nem foglalkozik időzónákkal).

### Kérdés 3

Kiadod a `show ntp associations` parancsot az R1-es routeren. A kimenetben (a táblázat alatt) azt látod, hogy a Referencia Óra címe: `127.127.1.1`. Melyik parancsot adták ki az R1-es routeren, hogy ezt elérjék?

- A) `ntp master 9`
- B) `ntp server 127.127.1.1`
- C) `ntp master` (Vagyis `ntp master 8`)
- D) `ntp peer 127.127.1.1`

**Helyes Válasz: A**

**Magyarázat:** Ha a referencia óra `127.127.1.1`, az azt jelenti, hogy a gép saját magát jelölte ki mesternek! Ez kizárólag az `ntp master` parancs kiadásával lehetséges. A videóban láthattuk, hogy ennél a kérdésnél a gép Stratum szintje 8 volt (a `show ntp status` kimenetben). Mivel a hivatkozott (szülő) referencia óra (`127.127.1.1`) szintjét mutatta a feladat táblázata "st" oszlopa 8-asként, ez azt jelenti, hogy az alapértelmezett helyett a Stratum 9-et paraméterezték fel a parancsnak (mert a gép maga Stratum 9-es, így a belső óra 8-as). A B és a D azért hibás, mert loopback címeket nem lehet kézzel Server/Peer-ként megadni.

### Kérdés 4

Melyik paranccsal teszed a routeredet egy konkrét külső időszerver Kliensévé?

- A) `ntp peer 216.239.35.0`
- B) `ntp master`
- C) `ntp server 216.239.35.0`
- D) `ntp client`

**Helyes Válasz: C**

**Magyarázat:** A hierarchikus (Mester->Szolga) kapcsolódáshoz az `ntp server` parancsot használjuk. Ennek hatására a mi routerünk a megadott IP-cím Kliense lesz! (A D parancs nem is létezik, az A parancs pedig szimmetrikus (egyenrangú) módba tenné a gépet).

### Kérdés 5

Az alábbi lépések/parancsok közül melyik **NÉGY** elengedhetetlen ahhoz, hogy egy NTP Kliensen sikeresen engedélyezzük az NTP Hitelesítést (Authentication) és csatlakozzunk a Szerverhez? (Válassz NÉGYET!)

- A) `ntp authentication-key 1 md5 <jelszó>`
- B) `ntp authenticate`
- C) `ntp trusted-key 1`
- D) `ntp server <IP> key 1`
- E) `ntp master key 1`
- F) `ntp update-calendar`

**Helyes Válasz: A, B, C és D**

**Magyarázat:** A hitelesítés 4 aranyparancsa:
1. Bekapcsolni a védelmet (B).
2. Megírni a jelszót az 1-es kulcshoz (A).
3. Bízni az 1-es kulcsban (C).
4. A Kliensnek, amikor beköti a szervert, kötelező jeleznie, hogy ezt a kulcsot fogja használni (D).
Az F parancs a hardver órára vonatkozik, az E parancsnak pedig semmi értelme.

### Bónusz Kérdés (Boson ExSim vizsgakérdés - A "Static Client" Kifejezés)

Amikor a Globális Konfigurációs módban kiadod az `ntp server` parancsot egy Cisco routeren, a hálózati működési módok közül **MELYIKBE** kerül bele az eszköz? Válaszd a legjobb választ!

- A) Static Client Mode (Statikus Kliens Mód)
- B) Symmetric Active Mode (Szimmetrikus Aktív Mód)
- C) Broadcast Client Mode (Szórásos Kliens Mód)
- D) Server Mode (Szerver Mód)
- E) Authentication (Hitelesítés)

**Helyes Válasz: A**

**Magyarázat:** Amikor manuálisan (`ntp server <IP>`) megadod, hogy egy fix IP-című szerverhez akarod kötni a gépet, az a legegyszerűbb, klasszikus Kliens működés. Ennek az egyirányú (kérdezz-felelek) folyamatnak a Boson feladat által hivatkozott (és a Cisco dokumentációkban szereplő) szakmai neve a **"Static Client Mode"**. (Jeremy nem említette meg külön ezt a szakkifejezést a fővideóban, mert ritkán kérdezik ezen a néven, de itt rávilágított: A Broadcast client egy másik, fejlettebb mód, ahol a router nem szólítja meg az NTP szervert, hanem csak "fülel", várva, hogy a szerver szórja az időt (Broadcast)).
