# Különbség a Hard link (Kemény link) és a Soft link (Szimbolikus link) között

## 1. Elméleti alapok

### Mi az a hard link?

A hard link egy fájl tükörmásolata a Linux vagy Unix rendszerben. Az eredeti fájl és a linkelt fájl ugyanazzal az inode-dal (indexcsomóponttal) rendelkezik. Mivel az inode közös, a hard linkek nem léphetik át a fájlrendszerek határait (nem hozhatsz létre hard linket egy másik partíción/csatolási ponton lévő fájlhoz). Ha törölsz egy hard linket, az eredeti fájl és a többi hard link érintetlen marad, a törlés csupán a fájlra mutató linkek számát (link count) csökkenti. A hard linkek a fájl tényleges tartalmát hordozzák.

### Mi az a soft link?

A soft link (vagy szimbolikus link) csupán egy hivatkozás egy fájlra. Leginkább a Windows "asztali parancsikonjaihoz" (desktop shortcuts) hasonlítható. Mivel ez csak egy hivatkozás, az inode-ja eltér a célfájlétól. A soft linkek átléphetik a fájlrendszerek határait. Ha törlöd az eredeti fájlt, az összes rá mutató soft link "eltörik" (hibát dob), mivel egy már nem létező fájlra fognak mutatni.

## 2. Különbségek (Összehasonlító táblázat)

| Hard link (Kemény link) | Soft link (Szimbolikus link) |
| --- | --- |
| Az eredeti fájl tükörmásolata | Hivatkozás (link) az eredeti fájlra |
| A link és az eredeti fájl ugyanazzal az inode-dal rendelkezik | A linknek eltérő az inode-ja, mint az eredeti fájlnak |
| Nem lépheti át a fájlrendszerek határait | Létrehozható különböző fájlrendszerek között is |
| Az adatokat akkor is megmutatja, ha az eredeti fájlt törlik | Hibát dob (eltörik), ha az eredeti fájlt törlik |
| Tartalmazza az eredeti fájl teljes tartalmát | Csak a forrásfájlra mutat, így nem tartalmazza a forrás adatait |
| Nem mutathat könyvtárakra (directories) | Mutathat könyvtárra is |
| Spórol az inode-okkal (mivel osztozik a forrás inode-ján) | Egy saját inode-ot foglal el, így csökkenti az elérhető inode-ok számát |
| Tárhelyet foglal, mivel egy tükörmásolat | Szinte egyáltalán nem foglal tárhelyet (csak a forrás útvonalát tartalmazza) |

## 3. Hogyan hozzunk létre hard linket?

Az `ln` parancsot kell használni, amit a forrás (eredeti fájlnév), majd a link neve követ. Az alábbi példában két hard linket (`link1`, `link2`) hozunk létre a `testdata` fájlhoz:

```bash
# cat testdata
This is test file with test data!

# ln testdata link1
# ln testdata link2

# ls -li
total 12
3921 -rw-r--r--. 3 root root 50 May 16 01:16 link1
3921 -rw-r--r--. 3 root root 50 May 16 01:16 link2
3921 -rw-r--r--. 3 root root 50 May 16 01:16 testdata
```

(Az `ls -li` kimenet első oszlopában látható, hogy mindhárom fájl ugyanazt az inode-ot (3921) kapta, és a méretük is megegyezik).

Ha töröljük az eredeti fájlt, a hard linkek továbbra is tartalmazzák az adatokat, mivel tükörmásolatok:

```bash
# rm testdata
rm: remove regular file `testdata'? y

# ls -li
total 8
3921 -rw-r--r--. 2 root root 50 May 16 01:16 link1
3921 -rw-r--r--. 2 root root 50 May 16 01:16 link2

# cat link1
This is test file with test data!
```

## 4. Hogyan hozzunk létre soft linket?

A soft link létrehozásához is az `ln` parancsot használjuk, de meg kell adni az `-s` (soft) opciót.

```bash
# ln -s testdata link1
# ln -s testdata link2

# ls -li
total 4
3921 lrwxrwxrwx. 1 root root  8 May 16 01:26 link1 -> testdata
3925 lrwxrwxrwx. 1 root root  8 May 16 01:26 link2 -> testdata
3923 -rw-r--r--. 1 root root 34 May 16 01:25 testdata
```

(Itt látható, hogy a soft linkek különböző inode-okat (3921, 3925) kaptak, mint az eredeti fájl (3923). A méretük is sokkal kisebb, és az `ls` kimenetének utolsó oszlopában látszik, hogy hova mutatnak: `-> testdata`).

Ha töröljük az eredeti fájlt, a soft linkek "eltörnek", és nem lehet őket többé használni:

```bash
# cat link2
This is test file with test data!

# rm -f testdata

# cat link2
cat: link2: No such file or directory
```

## 5. Hogyan azonosítsuk a hard és a soft linkeket?

- **Soft link:** Nagyon könnyű azonosítani. Az `ls -l` (vagy `ll`) kimenetében a jogosultságokat jelző sáv első karaktere egy `l` betű (pl. `lrwxrwxrwx`). Emellett az utolsó oszlopban egy nyíl (`->`) is egyértelműen mutatja a forrásfájl nevét.

- **Hard link:** Kicsit bonyolultabb. Használni kell az `ls -i` parancsot, és manuálisan megkeresni a duplikált inode számokat. Alternatív megoldásként a `find` paranccsal és a `-samefile` opcióval listáztathatjuk ki egy adott fájl összes hard linkjét:

```bash
# find /path -xdev -samefile testdata
```

(Ez a parancs végigszkenneli a `/path` könyvtárat, és kilistázza azokat a fájlokat, amelyeknek az inode-ja megegyezik a `testdata` fájléval.)

##Forrás
-[https://kerneltalks.com/commands/difference-between-hard-link-and-soft-link/]


# Hard linkek és Soft (Szimbolikus) linkek a Linuxban

Minden fájl a Linux fájlrendszerben egyetlen "hard linkkel" (kemény hivatkozással) indul, amely összeköti a fájlnevet a fájlrendszeren tárolt tényleges adattal.

## 1. Hard linkek (Kemény linkek)

Ha létrehozunk egy új hard linket egy fájlhoz, az azt jelenti, hogy egy új fájlnevet hozunk létre, amely hajszálpontosan ugyanarra az adatra mutat, mint az eredeti fájlnév.

**Létrehozás és példa:**

Tegyük fel, hogy van egy fájlunk `link_test` néven, benne a "hello world" szöveggel.

```bash
[tcarrigan@server demo]$ ls -l
total 4
-rw-rw-r--. 1 tcarrigan tcarrigan 12 Aug 29 14:27 link_test
```

(Figyeld meg a második oszlopban a linkek számát: jelenleg 1).

**Hard link létrehozásának szintaxisa:** `ln (eredeti fájl útvonala) (új fájl útvonala)`

```bash
[tcarrigan@server demo]$ ln link_test /tmp/link_new
```

Ha most mindkét fájlt megnézzük:

```bash
[tcarrigan@server demo]$ ls -l link_test /tmp/link_new 
-rw-rw-r--. 2 tcarrigan tcarrigan 12 Aug 29 14:27 link_test
-rw-rw-r--. 2 tcarrigan tcarrigan 12 Aug 29 14:27 /tmp/link_new
```

(A linkek száma mindkettőnél 2-re nőtt).

Ha kiolvasod az új link tartalmát, az eredeti adatot fogod látni:

```bash
[tcarrigan@server demo]$ cat /tmp/link_new 
hello world
```

**Fontos tulajdonságok:**

- Bármelyik fájlnevet is módosítod, a másik azonnal tükrözi a változásokat (hiszen ugyanaz az adat). A jogosultságok, tulajdonos, időbélyegek és a tartalom teljesen megegyeznek.

- Ha az eredeti fájlt törlöd, az adat nem vész el, mert a másodlagos hard linken keresztül továbbra is elérhető. Az adat csak akkor törlődik a meghajtóról, ha az összes rá mutató hard linket törölték.

- **Azonosítás:** Ha nem vagy biztos benne, hogy két fájl hard link-e, használd az `ls -i` (vagy `ls -li`) parancsot. A hard linkek osztoznak az inode számon (a lenti példában 2730074):

  ```bash
  [tcarrigan@server demo]$ ls -li link_test /tmp/link_new 
  2730074 -rw-rw-r--. 2 tcarrigan tcarrigan 12 Aug 29 14:27 link_test
  2730074 -rw-rw-r--. 2 tcarrigan tcarrigan 12 Aug 29 14:27 /tmp/link_new
  ```

**A Hard linkek korlátai (Hard limits):**

- Kizárólag normál (regular) fájlokhoz hozhatók létre (könyvtárakhoz vagy speciális fájlokhoz nem).
- Nem léphetik át a fájlrendszerek határait. A hard linknek ugyanazon a partíción/fájlrendszeren kell lennie, mint az eredeti fájlnak.

## 2. Soft linkek (Szimbolikus linkek)

A soft link egy speciális fájl, amely csupán egy másik meglévő fájl nevére (útvonalára) mutat. Ezek átléphetik a fájlrendszerek határait, és könyvtárakhoz vagy speciális fájlokhoz is létrehozhatók.

**Soft link létrehozásának szintaxisa (`-s` opcióval):**

`ln -s (mutatni kívánt fájl útvonala) (új fájl útvonala)`

**Példa:**

Létrehozunk egy soft linket (`/tmp/soft_link_new`) egy meglévő fájlhoz (`soft_link_test`):

```bash
[tcarrigan@server demo]$ ln -s /home/tcarrigan/demo/soft_link_test /tmp/soft_link_new
```

Ha kilistázzuk őket:

```bash
[tcarrigan@server demo]$ ls -l soft_link_test /tmp/soft_link_new 
-rw-rw-r--. 1 tcarrigan tcarrigan 17 Aug 30 11:59 soft_link_test
lrwxrwxrwx. 1 tcarrigan tcarrigan 35 Aug 30 12:09 /tmp/soft_link_new -> /home/tcarrigan/demo/soft_link_test
```

(Az `l` betű jelzi a sor elején, hogy linkről van szó, és a `->` mutatja az eredeti fájl útvonalát).

Az új soft link kiolvasása az eredeti szöveget adja vissza:

```bash
[tcarrigan@server demo]$ cat /tmp/soft_link_new 
soft Hello world
```

**A Soft linkek hátrányai (Adatvesztés és kavarodás):**

- Ha az eredeti fájlt törlöd, a soft link "eltörik" (ezt lógó vagy árva soft linknek – dangling soft link – hívják).

- Ha azonban valaki később létrehoz egy új fájlt pontosan ugyanazzal a névvel, mint a törölt eredeti fájl, az "eltört" soft link hirtelen ismét működni kezd, de már az új fájlra fog mutatni – függetlenül attól, hogy ez volt-e a szándékod vagy sem.

## Összegzés (Alapszabály)

Bár a koncepciók elsőre nehéznek tűnhetnek, a lényeg a következőképpen foglalható össze:

- A **Hard link** mindig egy fájlnevet kapcsol össze egy tárolóeszközön lévő **TÉNYLEGES ADATTAL**.
- A **Soft link** mindig egy fájlnevet kapcsol össze egy **MÁSIK FÁJLNÉVVEL**, amely végül a tárolóeszközön lévő adatra mutat.

##Forrás
-[https://www.redhat.com/en/blog/linking-linux-explained]
