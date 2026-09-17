# 3.6 Átirányítások

Egy parancs végrehajtása előtt a bemenete és kimenete átirányítható egy speciális jelölés segítségével, amelyet a shell értelmez. Az átirányítás lehetővé teszi a parancsok fájlleíróinak (file handles) duplikálását, megnyitását, lezárását, más fájlokra való hivatkozását, és megváltoztathatja, hogy a parancs mely fájlokból olvas és melyekbe ír.

Amikor az `exec` beépített paranccsal együtt használják, az átirányítások az aktuális shell végrehajtási környezetének fájlleíróit módosítják.

A következő átirányítási operátorok megelőzhetnek egy egyszerű parancsot, megjelenhetnek bárhol benne, vagy követhetik is azt.

Az átirányítások a megjelenésük sorrendjében, balról jobbra haladva kerülnek feldolgozásra.

Minden olyan átirányítás, amelyet egy fájlleíró száma előzhet meg, megelőzhető egy `{varname}` (változónév) formátumú szóval is.

Ebben az esetben, minden átirányítási operátornál – a `>&-` és `<&-` kivételével – a shell lefoglal egy 10-nél nagyobb vagy egyenlő fájlleírót, és hozzárendeli a `{varname}`-hez.

Ha a `{varname}` a `>&-` vagy `<&-` operátort előzi meg, a `varname` értéke határozza meg a lezárandó fájlleírót.

Ha a `{varname}` meg van adva, az átirányítás a parancs hatókörén túl is megmarad, ami lehetővé teszi a shell programozó számára, hogy a fájlleíró élettartamát manuálisan kezelje az `exec` beépített parancs használata nélkül.

Ezt a viselkedést a `varredir_close` shell opció szabályozza (lásd: A `Shopt` beépített parancs).

A következő leírásokban, ha a fájlleíró száma el van hagyva, és az átirányítási operátor első karaktere `<`, az átirányítás a szabványos bemenetre (standard input, 0-s fájlleíró) vonatkozik.

Ha az átirányítási operátor első karaktere `>`, az átirányítás a szabványos kimenetre (standard output, 1-es fájlleíró) vonatkozik.

A következő leírásokban az átirányítási operátort követő szó – hacsak nincs másképp jelezve – alávetődik a kapcsos zárójel-kiterjesztésnek (brace expansion), tilde kiterjesztésnek, paraméter- és változó-kiterjesztésnek, parancs-helyettesítésnek, aritmetikai kiterjesztésnek, az idézőjelek eltávolításának (quote removal), fájlnév-kiterjesztésnek és szétvágásnak (word splitting).

Ha a kiterjesztés eredménye egynél több szó, a Bash hibát jelez.

Az átirányítások sorrendje lényeges.

Például a parancs:

```bash
ls > dirlist 2>&1
```

mind a szabványos kimenetet (1-es fájlleíró), mind a szabványos hibakimenetet (2-es fájlleíró) a `dirlist` fájlba irányítja, míg a parancs:

```bash
ls 2>&1 > dirlist
```

csak a szabványos kimenetet irányítja a `dirlist` fájlba, mivel a szabványos hibakimenet a szabványos kimenet másolata lett azelőtt, hogy a szabványos kimenetet átirányították volna a `dirlist` fájlba.

A Bash bizonyos fájlneveket speciálisan kezel, amikor azokat átirányításokban használják, amint azt a következő táblázat mutatja.

Ha az operációs rendszer, amelyen a Bash fut, biztosítja ezeket a speciális fájlokat, akkor a Bash azokat használja; ellenkező esetben belsőleg emulálja őket az alábbiakban leírt viselkedéssel.

| Fájlnév | Viselkedés |
| --- | --- |
| `/dev/fd/fd` | Ha az `fd` egy érvényes egész szám, duplikálja az `fd` fájlleírót. |
| `/dev/stdin` | A 0-s fájlleírót duplikálja. |
| `/dev/stdout` | Az 1-es fájlleírót duplikálja. |
| `/dev/stderr` | A 2-es fájlleírót duplikálja. |
| `/dev/tcp/host/port` | Ha a `host` egy érvényes gépnév (hostname) vagy internetcím, és a `port` egy egész portszám vagy szolgáltatásnév, a Bash megkísérli megnyitni a megfelelő TCP socketet. |
| `/dev/udp/host/port` | Ha a `host` egy érvényes gépnév vagy internetcím, és a `port` egy egész portszám vagy szolgáltatásnév, a Bash megkísérli megnyitni a megfelelő UDP socketet. |

A fájl megnyitásának vagy létrehozásának sikertelensége az átirányítás meghiúsulását okozza.

A 9-nél nagyobb fájlleírókat használó átirányításokat óvatosan kell használni, mivel ütközhetnek a shell által belsőleg használt fájlleírókkal.

**Átirányítási típusok:**

- Bemenet átirányítása
- Kimenet átirányítása
- Átirányított kimenet hozzáfűzése
- Szabványos kimenet és szabványos hibakimenet átirányítása
- Szabványos kimenet és szabványos hibakimenet hozzáfűzése
- Here dokumentumok
- Here sztringek
- Fájlleírók duplikálása
- Fájlleírók áthelyezése
- Fájlleírók megnyitása olvasásra és írásra

## 3.6.1 Bemenet átirányítása

A bemenet átirányítása megnyitja azt a fájlt (amelynek neve a `word` kiterjesztéséből származik) olvasásra az `n` fájlleírón, vagy a szabványos bemeneten (0-s fájlleíró), ha az `n` nincs megadva.

A bemenet átirányításának általános formátuma:

```bash
[n]<word
```

## 3.6.2 Kimenet átirányítása

A kimenet átirányítása megnyitja azt a fájlt (amelynek neve a `word` kiterjesztéséből származik) írásra az `n` fájlleírón, vagy a szabványos kimeneten (1-es fájlleíró), ha az `n` nincs megadva.

Ha a fájl nem létezik, létrejön; ha létezik, nullára (üresre) csonkolódik.

A kimenet átirányításának általános formátuma:

```bash
[n]>[|]word
```

Ha az átirányítási operátor `>`, és a `set` beépített parancs `noclobber` opciója engedélyezve van, az átirányítás meghiúsul, ha a `word` kiterjesztéséből származó nevű fájl már létezik és hagyományos (regular) fájl.

Ha az átirányítási operátor `>|`, vagy az operátor `>` és a `set` beépített parancs `noclobber` opciója nincs engedélyezve, a Bash akkor is megkísérli az átirányítást, ha a `word` által megnevezett fájl létezik.

## 3.6.3 Átirányított kimenet hozzáfűzése

A kimenet ily módon történő átirányítása megnyitja azt a fájlt (amelynek neve a `word` kiterjesztéséből származik) hozzáfűzésre az `n` fájlleírón, vagy a szabványos kimeneten (1-es fájlleíró), ha az `n` nincs megadva.

Ha a fájl nem létezik, létrejön.

A kimenet hozzáfűzésének általános formátuma:

```bash
[n]>>word
```

## 3.6.4 Szabványos kimenet és szabványos hibakimenet átirányítása

Ez a szerkezet mind a szabványos kimenetet (1-es fájlleíró), mind a szabványos hibakimenetet (2-es fájlleíró) abba a fájlba irányítja, amelynek neve a `word` kiterjesztéséből származik.

A szabványos kimenet és a szabványos hibakimenet átirányításának két formátuma van:

```bash
&>word
```

és

```bash
>&word
```

A két forma közül az első a preferált. Ez szemantikailag egyenértékű ezzel:

```bash
>word 2>&1
```

A második forma használatakor a `word` nem bomolhat ki egy számmá vagy `-`-szá. Ha mégis, akkor más átirányítási operátorok lépnek érvénybe (lásd alább a Fájlleírók duplikálása részt) kompatibilitási okokból.

## 3.6.5 Szabványos kimenet és szabványos hibakimenet hozzáfűzése

Ez a szerkezet mind a szabványos kimenetet (1-es fájlleíró), mind a szabványos hibakimenetet (2-es fájlleíró) ahhoz a fájlhoz fűzi hozzá, amelynek neve a `word` kiterjesztéséből származik.

A szabványos kimenet és szabványos hibakimenet hozzáfűzésének formátuma:

```bash
&>>word
```

Ez szemantikailag egyenértékű ezzel:

```bash
>>word 2>&1
```

(lásd alább a Fájlleírók duplikálása részt).

## 3.6.6 Here dokumentumok

Ez a típusú átirányítás arra utasítja a shellt, hogy az aktuális forrásból olvasson bemenetet addig, amíg egy olyan sort nem olvas, amely csak a határolót (delimiter) tartalmazza (záró szóközök nélkül).

Az addig a pontig beolvasott összes sor egy parancs szabványos bemenetévé (vagy az `n` fájlleíróvá, ha az `n` meg van adva) válik.

A here dokumentumok formátuma:

```bash
[n]<<[-]word
here-document
delimiter
```

A shell nem végez paraméter- és változó-kiterjesztést, parancs-helyettesítést, aritmetikai kiterjesztést, sem fájlnév-kiterjesztést a `word`-ön.

Ha a `word` bármely része idézőjelben van, a `delimiter` a `word` idézőjeleinek eltávolítása utáni eredmény lesz, és a here dokumentum sorai nem lesznek kiterjesztve.

Ha a `word` nincs idézőjelben, a `delimiter` maga a `word`, és a here dokumentum szövege hasonlóan lesz kezelve, mint egy dupla idézőjeles sztring: a here dokumentum minden sora alávetődik a paraméter-kiterjesztésnek, parancs-helyettesítésnek és aritmetikai kiterjesztésnek, a `\newline` karaktersorozatot szó szerint kezeli, és a `\` karaktert kell használni a `\`, `$`, és `` ` `` karakterek escape-eléséhez; azonban a dupla idézőjeleknek nincs speciális jelentésük.

Ha az átirányítási operátor `<<-`, a shell eltávolítja a vezető tabulátor karaktereket a bemeneti sorokból és a határolót (delimiter) tartalmazó sorból is.

Ez lehetővé teszi, hogy a shell szkripteken belüli here dokumentumok természetes módon legyenek behúzva.

Ha a határoló (delimiter) nincs idézőjelben, a `\<újsor>` szekvencia sorfolytatásként lesz kezelve: a két sor egyesül, és a visszaperjel-újsor eltávolításra kerül.

Ez a here dokumentum beolvasása közben történik, a záró határoló ellenőrzése előtt, így az egyesített sorok formálhatják a záró határolót.

## 3.6.7 Here sztringek

A here dokumentumok egy változata, a formátuma:

```bash
[n]<<< word
```

A `word` alávetődik a tilde kiterjesztésnek, paraméter- és változó-kiterjesztésnek, parancs-helyettesítésnek, aritmetikai kiterjesztésnek, és az idézőjelek eltávolításának.

Fájlnév-kiterjesztés és szétvágás (word splitting) nem történik.

Az eredmény egyetlen sztringként, egy újsor karakterrel kiegészítve kerül átadásra a parancs szabványos bemenetére (vagy az `n` fájlleíróra, ha az `n` meg van adva).

## 3.6.8 Fájlleírók duplikálása

Az átirányítási operátor

```bash
[n]<&word
```

a bemeneti fájlleírók duplikálására szolgál.

Ha a `word` egy vagy több számjegyre terjed ki, az `n` fájlleíró annak a fájlleírónak a másolata lesz.

Átirányítási hibát jelent, ha a `word`-ben lévő számjegyek nem egy bemenetre megnyitott fájlleírót határoznak meg.

Ha a `word` értéke `-`, az `n` fájlleíró lezárásra kerül.

Ha az `n` nincs megadva, ez a szabványos bemenetet (0-s fájlleíró) használja.

Az operátor

```bash
[n]>&word
```

hasonlóan a kimeneti fájlleírók duplikálására szolgál.

Ha az `n` nincs megadva, ez a szabványos kimenetet (1-es fájlleíró) használja.

Átirányítási hibát jelent, ha a `word`-ben lévő számjegyek nem egy kimenetre megnyitott fájlleírót határoznak meg.

Ha a `word` értéke `-`, az `n` fájlleíró lezárásra kerül.

Különleges esetként, ha az `n` ki van hagyva, és a `word` nem bomlik ki egy vagy több számjeggyé vagy `-`-szá, akkor ez a szabványos kimenetet és a szabványos hibakimenetet a korábban leírtak szerint irányítja át.

## 3.6.9 Fájlleírók áthelyezése

Az átirányítási operátor

```bash
[n]<&digit-
```

a `digit` (számjegy) fájlleírót az `n` fájlleíróra helyezi át, vagy a szabványos bemenetre (0-s fájlleíró), ha az `n` nincs megadva.

A `digit` a duplikálás után lezárásra kerül az `n`-en.

Hasonlóképpen, az átirányítási operátor

```bash
[n]>&digit-
```

a `digit` (számjegy) fájlleírót az `n` fájlleíróra helyezi át, vagy a szabványos kimenetre (1-es fájlleíró), ha az `n` nincs megadva.

## 3.6.10 Fájlleírók megnyitása olvasásra és írásra

Az átirányítási operátor

```bash
[n]<>word
```

a `word` kiterjesztéséből származó nevű fájlt nyitja meg mind olvasásra, mind írásra az `n` fájlleírón, vagy a 0-s fájlleírón, ha az `n` nincs megadva.

Ha a fájl nem létezik, létrejön.

##Forrás
-[https://www.gnu.org/software/bash/manual/html_node/Redirections.html]

# Bemenet, Kimenet és Hiba Átirányítás Linuxban

A Linux parancsok egy bemenetet kapnak, amiből valamilyen kimenetet generálnak. Három fő adatfolyam (data stream) vesz részt ebben a folyamatban.

## 1. Az Alapvető Adatfolyamok (Data Streams)

| Adatfolyam | Leírás | Alapértelmezés | Stream ID (File Descriptor) |
| --- | --- | --- | --- |
| Standard input (stdin) | A bemeneti adatok forrása | Billentyűzetről bevitt szöveg | 0 |
| Standard output (stdout) | A parancs sikeres eredménye / kimenete | Megjelenik a képernyőn | 1 |
| Standard error (stderr) | A hibaüzenetek kimenete | Szintén megjelenik a képernyőn | 2 |

Az átirányítás (redirection) azt jelenti, hogy ezeket az adatfolyamokat eltérítjük a szokásos céljuktól (pl. képernyő) egy fájlba vagy egy másik parancsba.

## 2. Kimenet Átirányítása (Output / stdout Redirection)

A `>` jellel a kimenetet egy fájlba irányíthatjuk. Ha a fájl nem létezik, a shell létrehozza. (A célfájl még azelőtt létrejön, mielőtt a parancs egyáltalán lefutna).

```bash
command > file
```

**Példa:**

```bash
abhishek@linuxhandbook:~$ ls > output.txt
abhishek@linuxhandbook:~$ cat output.txt 
appstxt
new.txt
output.txt
static-ip.txt
```

(Látható, hogy az `output.txt` már szerepel a listában, mert az átirányítás célfájljának kész kell lennie a parancs lefutása előtt).

**Felülírás helyett hozzáfűzés (Append):**

Ha a célfájl már létezik, a `>` operátor törli a tartalmát (clobber) és felülírja azt. Ha az eredeti tartalmat meg akarjuk tartani, és a végére fűzni az újat, a `>>` jelet kell használni:

```bash
command >> file
```

> **Tipp:** A felülírást az aktuális sessionben a `set -C` paranccsal le is lehet tiltani.

## 3. Csővezeték Átirányítás (Pipe Redirection)

A pipe (`|`) a leggyakoribb átirányítás, amely az egyik parancs kimenetét (stdout) a következő parancs bemenetévé (stdin) teszi a memórián (buffer) keresztül.

```bash
command_1 | command_2
```

Tetszőleges számú parancs láncolható össze vele:

```bash
command_1 | command_2 | command_3 | command_4
```

**Példa:** Számoljuk meg a látható fájlokat az aktuális mappában:

```bash
abhishek@linuxhandbook:~$ ls -1 | wc -l
4
```

(Az `ls -1` kilistázza a fájlokat egymás alá, a pipe átadja a listát a `wc -l` parancsnak, ami megszámolja a sorokat).

**Fontos szabály:** Az adatfolyam (stdout/stdin) mindig egy nyers adatdarab / szöveg, nem pedig fájlnevek.

Például, a `find` parancs kimenetét (a megtalált fájlnevek szövegét) nem lehet közvetlenül átadni az `mv` parancsnak (mivel az fájlokat, mint objektumokat/argumentumokat vár, nem szöveget):

```bash
# EZ NEM FOG MŰKÖDNI
find . -type f -name "*.txt" | mv destination_directory
```

Ezt az `xargs` (vagy az `exec`) segítségével kell megoldani, ami a szöveget argumentummá alakítja:

```bash
find . -type f -name "*.txt" | xargs -t -I{} mv {} ../new_dir
```

## 4. Bemenet Átirányítása (Input / stdin Redirection)

A `<` operátorral egy fájl tartalmát adhatjuk át bemenetként egy parancsnak:

```bash
command < file
```

A bemenet átirányítását ritkábban használjuk, mert a legtöbb Linux parancs (pl. `head`) közvetlenül is elfogad fájlneveket argumentumként (pl. `head filename.txt`, a `<` használata nélkül).

Ugyanakkor hasznos, ha a program kifejezetten a stdin-t várja (és elkerülhető vele a `cat` parancs felesleges használata).

**Példa:** Szöveg kisbetűsről nagybetűsre konvertálása a `tr` paranccsal:

```bash
tr a-z A-Z < filename.txt
```

(Sokan ezt helytelenül `cat filename.txt | tr a-z A-Z` formában írják meg).

## 5. Átirányítások Kombinálása

A különböző átirányítások (stdin, stdout, pipe) szabadon kombinálhatóak a szükségleteknek megfelelően.

**Példa:** Listázzuk a `.txt` fájlokat, számoljuk meg őket, majd az eredményt mentsük egy fájlba:

```bash
ls *.txt | wc -l > count.txt
```

## 6. Hiba Átirányítása (Error / stderr Redirection)

Alapértelmezésben a `>` operátor az `1>` rövidítése, azaz csak az stdout (1-es stream) kimenetre vonatkozik.

A hibaüzenetek (stderr) fájlba irányításához kifejezetten a 2-es stream ID-t kell használni (`2>` vagy hozzáfűzéshez `2>>`).

**Csak a hibaüzenet fájlba írása:**

```bash
abhishek@linuxhandbook:~$ ls fffff 2> error.txt
abhishek@linuxhandbook:~$ cat error.txt 
ls: cannot access 'fffff': No such file or directory
```

**A normál kimenet (stdout) és a hibaüzenet (stderr) szétválasztása két külön fájlba:**

```bash
abhishek@linuxhandbook:~$ ls -l new.txt ffff > output.txt 2> error.txt 
abhishek@linuxhandbook:~$ cat output.txt 
-rw-rw-r-- 1 abhishek abhishek 0 May  5 10:25 new.txt
abhishek@linuxhandbook:~$ cat error.txt 
ls: cannot access 'ffff': No such file or directory
```

**A normál kimenet és a hibaüzenet ugyanabba a fájlba irányítása (manuális hozzáfűzéssel):**

```bash
abhishek@linuxhandbook:~$ ls -l new.txt fff 2>> combined.txt >> combined.txt 
```

**Az ajánlott mód a stdout és stderr egyesítésére (`2>&1`):**

A `2>&1` jelentése: "irányítsd a stderr-t ugyanoda, ahová az stdout megy".

```bash
abhishek@linuxhandbook:~$ ls -l new.txt fff > output.txt 2>&1
abhishek@linuxhandbook:~$ cat output.txt 
ls: cannot access 'fff': No such file or directory
-rw-rw-r-- 1 abhishek abhishek 0 May  5 10:25 new.txt
```

##Forrás
-[https://linuxhandbook.com/redirection-linux/]
