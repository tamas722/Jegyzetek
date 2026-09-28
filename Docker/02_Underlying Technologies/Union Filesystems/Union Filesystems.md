# Overlay Filesystem

Ez a dokumentum egy prototípust ír le a Linux overlay-fájlrendszer funkcionalitásának új megközelítéséhez (amelyet néha union-fájlrendszereknek is neveznek). Az overlay-fájlrendszer megpróbál egy olyan fájlrendszert bemutatni, amely egy fájlrendszer egy másikra való ráhelyezésének (overlaying) eredménye.

## Overlay objektumok

Az overlay-fájlrendszer megközelítése „hibrid”, mert a fájlrendszerben megjelenő objektumok nem mindig tűnnek ahhoz a fájlrendszerhez tartozónak. Sok esetben egy unióban elért objektum megkülönböztethetetlen az eredeti fájlrendszerből elért megfelelő objektumtól. Ez a legnyilvánvalóbb a `stat(2)` által visszaadott `st_dev` mezőből.

Míg a könyvtárak az overlay-fájlrendszer `st_dev` értékét jelentik, a nem könyvtár objektumok az alsó (lower) vagy felső (upper) fájlrendszer `st_dev` értékét jelenthetik, amely az objektumot szolgáltatja. Hasonlóképpen az `st_ino` csak az `st_dev`-vel kombinálva egyedi, és mindkettő megváltozhat egy nem könyvtár objektum élettartama alatt. Sok alkalmazás és eszköz figyelmen kívül hagyja ezeket az értékeket, és nem érinti őket.

Abban a speciális esetben, ha minden overlay réteg ugyanazon a mögöttes fájlrendszeren van, minden objektum az overlay-fájlrendszer `st_dev` értékét és a mögöttes fájlrendszer `st_ino` értékét jelenti. Ez az overlay mountot jobban megfelelteti a fájlrendszer-szkennereknek, és az overlay objektumok megkülönböztethetők lesznek az eredeti fájlrendszer megfelelő objektumaitól.

64 bites rendszereken, még ha nem is minden overlay réteg van ugyanazon a mögöttes fájlrendszeren, ugyanaz a megfelelő viselkedés érhető el a „xino” funkcióval. A „xino” funkció egy egyedi objektumazonosítót állít össze a valódi objektum `st_ino`-jából és egy mögöttes fsid számból. A „xino” funkció a magas inode-szám biteket használja az fsid-hez, mert a mögöttes fájlrendszerek ritkán használják a magas inode-szám biteket. Abban az esetben, ha a mögöttes inode-szám túlcsordul a magas xino bitekbe, az overlay-fájlrendszer visszaesik a nem xino viselkedésre arra az inode-ra.

A „xino” funkció engedélyezhető a `-o xino=on` overlay mount opcióval. Ha minden mögöttes fájlrendszer támogatja az NFS fájlkezelőket, az overlay-fájlrendszer objektumainak `st_ino` értéke nemcsak egyedi, hanem tartós is a fájlrendszer élettartama alatt. A `-o xino=auto` overlay mount opció csak akkor engedélyezi a „xino” funkciót, ha a tartós `st_ino` követelmény teljesül.

Az alábbi táblázat összefoglalja, mi várható a különböző overlay konfigurációkban.

### Inode tulajdonságok

| Konfiguráció | Persistent st_ino | Uniform st_dev | st_ino == d_ino | d_ino == i_ino [*] |
| --- | --- | --- | --- | --- |
| | dir | !dir | dir | !dir | dir | !dir | dir | !dir |
| All layers on same fs | Y | Y | Y | Y | Y | Y | Y | Y |
| Layers not on same fs, xino=off | N | N | Y | N | N | Y | N | Y |
| xino=on/auto | Y | Y | Y | Y | Y | Y | Y | Y |
| xino=on/auto, ino overflow | N | N | Y | N | N | Y | N | Y |

[*] nfsd v3 readdirplus verifies d_ino == i_ino. i_ino is exposed via several /proc files, such as /proc/locks and /proc/self/fdinfo/<fd> of an inotify file descriptor.

## Upper and Lower

Az overlay-fájlrendszer két fájlrendszert kombinál – egy „felső” (upper) és egy „alsó” (lower) fájlrendszert. Amikor egy név mindkét fájlrendszerben létezik, a „felső” fájlrendszerben lévő objektum látható, míg az „alsó” fájlrendszerben lévő objektum vagy rejtett, vagy könyvtárak esetében a „felső” objektummal egyesül.

Helyesebb lenne „felső” és „alsó” **könyvtárfáról** beszélni, mint „fájlrendszerről”, mivel nagyon valószínű, hogy mindkét könyvtárfa ugyanabban a fájlrendszerben van, és nincs követelmény arra, hogy egy fájlrendszer gyökerét adják meg sem a felső, sem az alsó számára.

A Linux által támogatott fájlrendszerek széles skálája lehet az alsó fájlrendszer, de nem minden Linux által mountolható fájlrendszer rendelkezik az OverlayFS működéséhez szükséges funkciókkal. Az alsó fájlrendszernek nem kell írhatónak lennie. Az alsó fájlrendszer akár egy másik overlayfs is lehet. A felső fájlrendszer általában írható, és ha az, akkor támogatnia kell a `trusted.*` és/vagy `user.*` kiterjesztett attribútumok létrehozását, és érvényes `d_type`-ot kell biztosítania a readdir válaszokban, így az NFS nem megfelelő.

Két csak olvasható fájlrendszer csak olvasható overlay-je bármilyen fájlrendszertípust használhat.

## Directories

Az overlaying főként könyvtárakat érint. Ha egy adott név mind a felső, mind az alsó fájlrendszerben megjelenik, és valamelyikben nem könyvtárra utal, akkor az alsó objektum rejtett – a név csak a felső objektumra utal.

Ahol mind a felső, mind az alsó objektum könyvtár, ott egy egyesített könyvtár (merged directory) jön létre.

Mount időben a mount opciókként megadott két könyvtár, a `lowerdir` és az `upperdir`, egy egyesített könyvtárba kombinálódik:

```bash
mount -t overlay overlay -olowerdir=/lower,upperdir=/upper,\
workdir=/work /merged
```

A `workdir`-nek ugyanazon a fájlrendszeren lévő könyvtárnak kell lennie, mint az `upperdir`.

Ezután valahányszor egy lookup kérést intéznek egy ilyen egyesített könyvtárban, a lookup minden tényleges könyvtárban megtörténik, és a kombinált eredmény az overlay-fájlrendszerhez tartozó dentry-ban gyorsítótárazódik. Ha mindkét tényleges lookup könyvtárat talál, mindkettő tárolódik, és egy egyesített könyvtár jön létre, különben csak az egyik tárolódik: a felső, ha létezik, egyébként az alsó.

Csak a könyvtárakból származó névlisták egyesülnek. Más tartalom, például metaadatok és kiterjesztett attribútumok csak a felső könyvtárhoz vannak jelentve. Az alsó könyvtár ezen attribútumai rejtettek.

## Whiteouts and opaque directories

Annak érdekében, hogy támogassa az `rm` és `rmdir` műveleteket az alsó fájlrendszer megváltoztatása nélkül, az overlay-fájlrendszernek rögzítenie kell a felső fájlrendszerben, hogy a fájlokat eltávolították. Ez whiteoutokkal és opaque könyvtárakkal történik (a nem könyvtárak mindig opaque-ek).

A whiteout egy 0/0 eszközszámú karakteres eszközként vagy egy nulla méretű normál fájlként jön létre a `trusted.overlay.whiteout` xattr-rel.

Amikor egy whiteout található egy egyesített könyvtár felső szintjén, az alsó szinten lévő bármely megfelelő név figyelmen kívül marad, és maga a whiteout is rejtett.

Egy könyvtár opaque lesz a `trusted.overlay.opaque` xattr „y”-ra állításával. Ahol a felső fájlrendszer opaque könyvtárat tartalmaz, az alsó fájlrendszerben lévő bármely azonos nevű könyvtár figyelmen kívül marad.

Egy opaque könyvtárnak nem szabad whiteoutokat tartalmaznia, mert azok nem szolgálnak semmilyen célt. Egy merge könyvtár, amely a `trusted.overlay.whiteout` xattr-rel rendelkező normál fájlokat tartalmaz, továbbá meg kell jelölni a `trusted.overlay.opaque` xattr „x”-re állításával magán a merge könyvtáron. Erre azért van szükség, hogy elkerüljük a `trusted.overlay.whiteout` ellenőrzésének overheadjét az összes bejegyzésen a readdir során a gyakori esetben.

## readdir

Amikor egy `readdir` kérést intéznek egy egyesített könyvtárhoz, a felső és alsó könyvtárakat külön-külön olvassák, és a névlisták a nyilvánvaló módon egyesülnek (a felsőt először olvassák, majd az alsót – a már létező bejegyzések nem kerülnek újra hozzáadásra). Ez az egyesített névlista a `struct file`-ban gyorsítótárazódik, és így addig marad, amíg a fájlt nyitva tartják. Ha a könyvtárat két folyamat egyszerre nyitja meg és olvassa, mindegyiknek külön gyorsítótára lesz. Egy seekdir a könyvtár elejére (0 offset), amelyet egy readdir követ, a gyorsítótár elvetését és újraépítését okozza.

Ez azt jelenti, hogy az egyesített könyvtár változásai nem jelennek meg, amíg a könyvtárat olvassák. Ez valószínűleg nem tűnik fel sok programnak.

A seek offsetek szekvenciálisan vannak hozzárendelve, amikor a könyvtárakat olvassák. Így ha:

- egy könyvtár egy részét olvasod
- megjegyzel egy offsetet, és bezárod a könyvtárat
- később újra megnyitod a könyvtárat
- a megjegyzett offsetre seekelsz

akkor kevés korreláció lehet a régi és az új helyek között a fájlnevek listájában, különösen, ha bármi megváltozott a könyvtárban.

A nem egyesített könyvtárakon a readdir egyszerűen a mögöttes könyvtár (felső vagy alsó) által kezelt.

## renaming directories

Amikor egy könyvtárat átneveznek, amely az alsó rétegen vagy egyesített (azaz a könyvtár nem a felső rétegen jött létre eredetileg), az overlayfs két különböző módon kezelheti:

- **EXDEV hiba visszaadása:** ez a hiba a `rename(2)` által kerül visszaadásra, amikor egy fájlt vagy könyvtárat fájlrendszer-határokon keresztül próbálnak mozgatni. Ezért az alkalmazások általában fel vannak készülve ennek a hibának a kezelésére (például az `mv(1)` rekurzívan másolja a könyvtárfát). Ez az alapértelmezett viselkedés.
- Ha a „redirect_dir” funkció engedélyezve van, akkor a könyvtár copy up-olódik (de a tartalom nem). Ezután a `trusted.overlay.redirect` kiterjesztett attribútum az eredeti hely elérési útjára van állítva az overlay gyökerétől. Végül a könyvtár az új helyre mozog.

Számos módja van a „redirect_dir” funkció hangolásának.

**Kernel config opciók:**

- `OVERLAY_FS_REDIRECT_DIR`:
  - Ha ez engedélyezve van, akkor a redirect_dir alapértelmezésben be van kapcsolva.
- `OVERLAY_FS_REDIRECT_ALWAYS_FOLLOW`:
  - Ha ez engedélyezve van, akkor a redirecteket alapértelmezésben mindig követik. Ennek engedélyezése kevésbé biztonságos konfigurációt eredményez. Csak akkor engedélyezd ezt az opciót, ha aggódsz a visszafelé kompatibilitás miatt olyan kernelekkel, amelyek rendelkeznek a redirect_dir funkcióval, és követik a redirecteket akkor is, ha ki van kapcsolva.

**Modul opciók** (a `/sys/module/overlay/parameters/`-en keresztül is módosíthatók):

- `redirect_dir=BOOL`: Lásd a fenti `OVERLAY_FS_REDIRECT_DIR` kernel config opciót.
- `redirect_always_follow=BOOL`: Lásd a fenti `OVERLAY_FS_REDIRECT_ALWAYS_FOLLOW` kernel config opciót.
- `redirect_max=NUM`: Az abszolút redirectben lévő bájtok maximális száma (alapértelmezés: 256).

**Mount opciók:**

- `redirect_dir=on`: A redirectek engedélyezve vannak.
- `redirect_dir=follow`: A redirectek nem jönnek létre, de követve vannak.
- `redirect_dir=nofollow`: A redirectek nem jönnek létre és nincsenek követve.
- `redirect_dir=off`: Ha a `redirect_always_follow` engedélyezve van a kernel/modul configban, ez az „off” „follow”-ra fordul le, egyébként „nofollow”-ra.

Amikor az NFS export funkció engedélyezve van, minden copy up-olt könyvtár indexelve van az alsó inode fájlkezelője által, és a felső könyvtár fájlkezelője a `trusted.overlay.upper` kiterjesztett attribútumban tárolódik az index bejegyzésen. Egy egyesített könyvtár lookup-jakor, ha a felső könyvtár nem egyezik az indexben tárolt fájlkezelővel, az annak a jele, hogy több felső könyvtár lehet átirányítva ugyanahhoz az alsó könyvtárhoz. Ebben az esetben a lookup hibát ad vissza, és figyelmeztet egy lehetséges inkonzisztenciára.

Mivel az alsó réteg redirectjei nem ellenőrizhetők az indexszel, az NFS export támogatás engedélyezése egy felső réteg nélküli overlay-fájlrendszeren megköveteli a redirect follow kikapcsolását (pl. `redirect_dir=nofollow`).

## Non-directories

A nem könyvtár objektumok (fájlok, symlinkek, speciális eszközfájlok stb.) a felső vagy alsó fájlrendszerből jelennek meg, ahogy megfelelő. Amikor egy fájlt az alsó fájlrendszerben olyan módon érnek el, amely írási hozzáférést igényel, például írásra való megnyitás, valamilyen metaadat módosítása stb., a fájl először az alsó fájlrendszerből a felső fájlrendszerbe másolódik (`copy_up`). Ne feledd, hogy egy hard-link létrehozása is `copy_up`-ot igényel, bár természetesen egy symlink létrehozása nem.

A `copy_up` kiderülhet, hogy szükségtelen, például ha a fájlt olvasás-írásra nyitják meg, de az adatokat nem módosítják.

A `copy_up` folyamat először megbizonyosodik arról, hogy a tartalmazó könyvtár létezik a felső fájlrendszerben – létrehozva azt és a szülőket, ahogy szükséges. Ezután létrehozza az objektumot ugyanazokkal a metaadatokkal (tulajdonos, mód, mtime, symlink-cél stb.), majd ha az objektum fájl, az adatokat az alsóról a felső fájlrendszerbe másolja. Végül bármely kiterjesztett attribútum copy up-olódik.

Miután a `copy_up` befejeződött, az overlay-fájlrendszer egyszerűen közvetlen hozzáférést biztosít az újonnan létrehozott fájlhoz a felső fájlrendszerben – a fájlon végzett jövőbeli műveleteket az overlay-fájlrendszer alig veszi észre (bár a fájl nevén végzett műveletet, például az átnevezést vagy a törlést természetesen észreveszi és kezeli).

## Permission model

Az overlay-fájlrendszer eltárolja azokat a hitelesítő adatokat, amelyeket az alsó vagy felső fájlrendszerek elérésekor használnak.

A régi mount api-ban a `mount(2)`-t hívó task hitelesítő adatai tárolódnak el. Az új mount api-ban a superblockot a `fsconfig(2)` `FSCONFIG_CMD_CREATE` parancsán keresztül létrehozó task hitelesítő adatai tárolódnak el.

A v6.15 kerneltől kezdve lehetséges az `override_creds` mount opció használata, amely a hívó task hitelesítő adatainak rögzítését eredményezi. Ne feledd, hogy az `override_creds` csak az új mount api-val használva értelmes, mivel a régi mount api egyetlen `mount(2)` syscallban kombinálja az opciók beállítását és a superblock létrehozását.

Az engedélyellenőrzés az overlay-fájlrendszerben a következő elveket követi:

- az engedélyellenőrzésnek UGYANAZT az eredményt KELL visszaadnia a copy up előtt és után
- az overlay mountot létrehozó task NEM KAPHAT további jogosultságokat
- a task[*] TOVÁBBI jogosultságokat KAPHAT az overlay-en keresztül, a mögöttes alsó vagy felső fájlrendszereken való közvetlen hozzáféréshez képest

Ezt úgy érik el, hogy minden hozzáférésnél két engedélyellenőrzést végeznek:

- ellenőrzik, hogy az aktuális task engedélyezett-e a helyi DAC (tulajdonos, csoport, mód és posix acl), valamint MAC-ellenőrzések alapján
- ellenőrzik, hogy az eltárolt hitelesítő adatok engedélyezettek-e valódi műveletre az alsó vagy felső rétegen a mögöttes fájlrendszer-engedélyek alapján, ismét MAC-ellenőrzésekkel együtt

Az (a) ellenőrzés biztosítja a konzisztenciát (1), mivel a tulajdonos, csoport, mód és posix acl-ek copy up-olódnak. Másrészt a szerver által érvényesített engedélyek (amelyeket például az NFS használ) figyelmen kívül maradhatnak (3).

A (b) ellenőrzés biztosítja, hogy egyetlen task se kapjon olyan jogosultságokat a mögöttes rétegekhez, amelyekkel az eltárolt hitelesítő adatok nem rendelkeznek (2). Ez azt is jelenti, hogy lehetséges olyan beállításokat létrehozni, ahol a konzisztencia-szabály (1) nem teljesül; általában azonban az eltárolt hitelesítő adatok elegendő jogosultsággal rendelkeznek az összes művelet végrehajtásához.

A modell bemutatásának másik módja párhuzam vonása a következők között:

```bash
mount -t overlay overlay -olowerdir=/lower,upperdir=/upper,... /merged
```

és:

```bash
cp -a /lower /upper
mount --bind /upper /merged
```

Az eredményül kapott hozzáférési engedélyeknek ugyanazoknak kell lenniük. A különbség a másolás idejében van (igény szerinti vs. előzetes).

## Idmapped mounts

Maga az overlay mount idmapped mounttá alakítható, ha idmappinget alkalmazunk rá a `mount_setattr(2)` és a `MOUNT_ATTR_IDMAP` segítségével, csakúgy, mint más idmapped mountokat támogató fájlrendszereknél.

A mount idmapping csak azt változtatja meg, hogyan jelennek meg és értelmeződnek az overlay inode-ok tulajdonjoga és engedélyei a hívó számára. Nem változtatja meg, hogyan fér hozzá az overlayfs a mögöttes rétegekhez: azokat továbbra is az eltárolt mountoló hitelesítő adataival érik el a saját mountjaikon keresztül, amelyek maguk is idmappedek lehetnek. Az overlay mount idmapping és bármely réteg idmapping összeáll; egy mögöttes id először a vonatkozó réteg idmapping szerint, majd az overlay mount idmapping szerint képeződik le.

## Multiple lower layers

Több alsó réteg adható meg a kettőspont („:”) elválasztó karakterként való használatával a könyvtárnevek között. Például:

```bash
mount -t overlay overlay -olowerdir=/lower1:/lower2:/lower3 /merged
```

Ahogy a példa mutatja, az `upperdir=` és a `workdir=` elhagyható. Ebben az esetben az overlay csak olvasható lesz.

A megadott alsó könyvtárak a jobb szélsőtől kezdve balra haladva lesznek egymásra halmozva. A fenti példában a `lower1` lesz a felső, a `lower2` a középső és a `lower3` az alsó réteg.

> **Megjegyzés:** a kettőspontokat tartalmazó könyvtárnevek alsó rétegként való megadásához a kettőspontokat egyetlen visszaperjellel kell escape-elni. Például:

```bash
mount -t overlay overlay -olowerdir=/a\:lower\:\:dir /merged
```

A v6.8 kernelverzió óta a kettőspontokat tartalmazó könyvtárnevek a `lowerdir+` mount opciókkal és az új mount api `fsconfig` syscalljával is konfigurálhatók alsó rétegként. Például:

```c
fsconfig(fs_fd, FSCONFIG_SET_STRING, "lowerdir+", "/a:lower::dir", 0);
```

Ez utóbbi esetben az alsó réteg könyvtárneveiben lévő kettőspontok oktális karakterekként (072) lesznek escape-elve, amikor a `/proc/self/mountinfo`-ban megjelennek.

## Metadata only copy up

Amikor a „metacopy” funkció engedélyezve van, az overlayfs csak metaadatokat másol (szemben a teljes fájllal), amikor egy metaadat-specifikus műveletet, például `chown`/`chmod`-ot hajtanak végre. Egy ebben az állapotban lévő felső fájl a `trusted.overlayfs.metacopy` xattr-rel van megjelölve, ami azt jelzi, hogy a felső fájl nem tartalmaz adatot. Az adatok később másolódnak fel, amikor a fájlt WRITE műveletre nyitják meg. Miután az alsó fájl adatai copy up-olódtak, a `trusted.overlayfs.metacopy` xattr eltávolításra kerül a felső fájlból.

Más szóval, ez egy késleltetett adat copy up művelet, és az adatok akkor másolódnak fel, amikor ténylegesen módosítani kell az adatokat.

Több módja van e funkció engedélyezésének/letiltásának. Egy `CONFIG_OVERLAY_FS_METACOPY` config opció beállítható/kikapcsolható, hogy alapértelmezésben engedélyezze/letiltja ezt a funkciót. Vagy engedélyezhető/letiltható modul betöltéskor a `metacopy=on/off` modul paraméterrel. Végül van egy mount opciónkénti `metacopy=on/off` is, amely mountonként engedélyezi/letiltja ezt a funkciót.

Ne használd a `metacopy=on`-t nem megbízható felső/alsó könyvtárakkal. Különben lehetséges, hogy egy támadó létrehoz egy kézzel készített fájlt megfelelő REDIRECT és METACOPY xattr-ekkel, és hozzáférést nyer az alsó fájlhoz, amelyre a REDIRECT mutat. Ez nem lehet lehetséges helyi rendszeren, mivel a `trusted.` xattr-ek beállítása `CAP_SYS_ADMIN`-t igényel. De lehetséges lehet nem megbízható rétegeknél, például pendrive-ról.

> **Megjegyzés:** a `redirect_dir={off|nofollow|follow[*]}` és az `nfs_export=on` mount opciók ütköznek a `metacopy=on`-nal, és hibát eredményeznek.

[*] A `redirect_dir=follow` csak akkor ütközik a `metacopy=on`-nal, ha az `upperdir=...` meg van adva.

## Data-only lower layers

A „metacopy” funkció engedélyezésével egy overlayfs normál fájl akár három különböző rétegből származó információ kompozíciója lehet:

- metaadatok a felső rétegben lévő fájlból
- `st_ino` és `st_dev` objektumazonosító egy alsó rétegben lévő fájlból
- adatok egy másik alsó rétegben (mélyebben) lévő fájlból

A „lower data” fájl bármely alsó rétegben lehet, kivéve a legfelső alsó réteget.

A legfelső alsó réteg alatt tetszőleges számú legalsóbb réteg definiálható „data-only” alsó rétegként, dupla kettőspont („::”) elválasztók használatával. Egy normál alsó réteg nem lehet egy data-only réteg alatt, így az egyszeres kettőspont elválasztók nem megengedettek a dupla kettőspont („::”) elválasztóktól jobbra.

Például:

```bash
mount -t overlay overlay -olowerdir=/l1:/l2:/l3::/do1::/do2 /merged
```

A „data-only” alsó rétegekben lévő fájlok elérési útjai nem láthatók az egyesített overlayfs könyvtárakban, és a „data-only” alsó rétegekben lévő fájlok metaadatai és `st_ino`/`st_dev` értékei nem láthatók az overlayfs inode-okban.

Csak a „data-only” alsó rétegekben lévő fájlok adatai lehetnek láthatók, amikor a felette lévő egyik alsó rétegben lévő „metacopy” fájl „redirect”-je a „data-only” alsó rétegben lévő „lower data” fájl abszolút elérési útjára mutat.

A `metacopy=on` explicit engedélyezése helyett elegendő legalább egy data-only réteget megadni az adatok data-only rétegre való átirányításának engedélyezéséhez. Ebben az esetben a metacopy más formái elutasításra kerülnek. Megjegyzés: így a data-only rétegek a `userxattr`-ral együtt használhatók, amely esetben gondos figyelmet kell fordítani a `user.overlay.redirect` xattr módosításához szükséges jogosultságokra a visszaélések megelőzése érdekében.

A v6.8 kernelverzió óta a „data-only” alsó rétegek a `datadir+` mount opciókkal és az új mount api `fsconfig` syscalljával is hozzáadhatók. Például:

```c
fsconfig(fs_fd, FSCONFIG_SET_STRING, "lowerdir+", "/l1", 0);
fsconfig(fs_fd, FSCONFIG_SET_STRING, "lowerdir+", "/l2", 0);
fsconfig(fs_fd, FSCONFIG_SET_STRING, "lowerdir+", "/l3", 0);
fsconfig(fs_fd, FSCONFIG_SET_STRING, "datadir+", "/do1", 0);
fsconfig(fs_fd, FSCONFIG_SET_STRING, "datadir+", "/do2", 0);
```

## Specifying layers via file descriptors

A v6.13 kernel óta az overlayfs támogatja a rétegek megadását fájl leírókon keresztül a path-ként való megadás mellett. Ez a funkció elérhető a `datadir+`, `lowerdir+`, `upperdir` és `workdir+` mount opciókhoz az új mount api `fsconfig` syscalljával:

```c
fsconfig(fs_fd, FSCONFIG_SET_FD, "lowerdir+", NULL, fd_lower1);
fsconfig(fs_fd, FSCONFIG_SET_FD, "lowerdir+", NULL, fd_lower2);
fsconfig(fs_fd, FSCONFIG_SET_FD, "lowerdir+", NULL, fd_lower3);
fsconfig(fs_fd, FSCONFIG_SET_FD, "datadir+", NULL, fd_data1);
fsconfig(fs_fd, FSCONFIG_SET_FD, "datadir+", NULL, fd_data2);
fsconfig(fs_fd, FSCONFIG_SET_FD, "workdir", NULL, fd_work);
fsconfig(fs_fd, FSCONFIG_SET_FD, "upperdir", NULL, fd_upper);
```

## fs-verity support

Egy alsó fájl metaadat copy up-ja során, ha a forrásfájl fs-verity-vel rendelkezik, és az overlay verity támogatás engedélyezve van, akkor az alsó fájl digestje hozzáadódik a `trusted.overlay.metacopy` xattr-he

## Forrás
-[https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html]
