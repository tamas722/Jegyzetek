# 6. Fejezet: Adatbetöltés, Tárolás és Fájlformátumok

## 6.1 Adatok olvasása és írása szöveges formátumban

A pandas számos beépített függvényt kínál arra, hogy a legkülönfélébb szöveges és bináris formátumokat DataFrame objektummá alakítsa. Ezek közül a legfontosabb és leggyakrabban használt a `pd.read_csv`.

### Table 6-1. Szöveges és bináris adatbetöltő függvények a pandas-ban

| Függvény | Leírás (Description) |
| :--- | :--- |
| `read_csv` | Határolójelekkel elválasztott adatokat tölt be fájlból vagy URL-ből; alapértelmezett határoló a vessző (`,`). |
| `read_fwf` | Fix oszlopszélességű formátumot olvas be (itt nincsenek elválasztó karakterek). |
| `read_clipboard` | Olyan, mint a `read_csv`, de a vágólapról (clipboard) olvas. Nagyon hasznos weblapokról kimásolt táblázatok gyors beolvasásához. |
| `read_excel` | Adatokat olvas Excel XLS vagy XLSX fájlokból. |
| `read_hdf` | HDF5 formátumú (a pandas által írt) fájlokat olvas be. |
| `read_html` | Egy megadott HTML dokumentumban található összes táblázatot beolvassa. |
| `read_json` | JSON stringből, fájlból vagy URL-ből olvas be adatokat. |
| `read_feather`, `read_orc`, `read_parquet` | Különféle gyors bináris fájlformátumok beolvasása. |
| `read_pickle` | A Python beépített pickle formátumában elmentett pandas objektumokat tölti vissza. |
| `read_sql`, `read_sql_table` | SQL adatbázisokból olvas ki adatokat vagy teljes táblákat (az SQLAlchemy segítségével). |
| `read_xml` | Adatokat olvas be XML fájlokból. |

A valós adatok gyakran piszkosak. Emiatt a `read_csv` több mint **50 opcionális paraméterrel** rendelkezik! (Ezeket érdemes a hivatalos pandas dokumentációban keresgélni, ha valami nagyon speciális fájllal találkozol).

A paraméterek főbb kategóriái:

- **Indexelés:** Mely oszlopok legyenek az adat, és mi legyen az index (a sor-címke)?
- **Típus-kikövetkeztetés (Type inference):** A pandas magától kitalálja (infer), hogy egy oszlop szám, dátum vagy szöveg.
- **Dátumok értelmezése (Parsing):** Akár több oszlopból is össze tud rakni egy dátumot.
- **Hiányos vagy csúnya adatok kezelése:** Sorok kihagyása, kommentek mellőzése, ezres-elválasztók kezelése.

### Alapvető CSV olvasás és Indexelés

Kezdjük egy egyszerű CSV (Vesszővel elválasztott) fájllal. (A `!cat` egy Linux/Mac terminálparancs, ami csak simán kiírja a fájl tartalmát a képernyőre).

```python
In [10]: !cat examples/ex1.csv
a,b,c,d,message
1,2,3,4,hello
5,6,7,8,world
9,10,11,12,foo
```

A beolvasás végtelenül egyszerű:

```python
In [11]: df = pd.read_csv("examples/ex1.csv")
```

Mivel a fájl első sora tartalmazza a fejléceket (`a,b,c,d,message`), a pandas ezeket automatikusan megteszi oszlopneveknek.

**Mi van, ha nincs fejléc a fájlban?**

```python
In [13]: !cat examples/ex2.csv
1,2,3,4,hello
5,6,7,8,world
```

Két lehetőségünk van:

1. Megkérjük a pandast, hogy generáljon számokat oszlopneveknek: `header=None`
2. Mi magunk adjuk meg a neveket a `names` listával:

```python
In [15]: pd.read_csv("examples/ex2.csv", names=["a", "b", "c", "d", "message"])
Out[15]: 
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
...
```

**Index oszlop megadása:**
Ha azt akarjuk, hogy ne egy 0, 1, 2... sorszámozás legyen a sor-címkénk, hanem mondjuk a `message` oszlop tartalma, akkor az `index_col` paramétert kell használni:

```python
In [17]: pd.read_csv("examples/ex2.csv", names=names, index_col="message")
Out[17]: 
         a   b   c   d
message               
hello    1   2   3   4
world    5   6   7   8
foo      9  10  11  12
```

Átadhatsz egy listát is az `index_col`-nak (pl. `["key1", "key2"]`), ha több oszlopból akarsz egy hierarchikus indexet (**MultiIndex**) létrehozni.

### Szabálytalan határolójelek (Szóközök és Reguláris Kifejezések)

Sokszor az elválasztó nem egy sima vessző, hanem változó mennyiségű szóköz vagy tabulátor.

```python
In [21]: !cat examples/ex3.txt
            A         B         C
aaa -0.264438 -1.026059 -0.619500
bbb  0.927272  0.302904 -0.032399
```

Ezt sima CSV-ként nem lehet olvasni. Helyette a `sep` (separator) paraméternek átadunk egy **Reguláris Kifejezést (RegEx)**. A `\s+` jelentése: "Egy vagy több szóköz (vagy tabulátor)".

```python
In [22]: result = pd.read_csv("examples/ex3.txt", sep="\s+")
```

### "Szemét" sorok kihagyása

Gyakran vannak a fájl elején megjegyzések (kommentek), amiket nem akarunk beolvasni.
A `skiprows` paraméternek átadhatsz egy listát, hogy melyik sorokat (0-tól indexelve) hagyja ki a gép.

```python
In [25]: pd.read_csv("examples/ex4.csv", skiprows=[0, 2, 3])
```

### Hiányzó adatok (NaN) kezelése olvasáskor

Ez a valós adatelemzés egyik legtrükkösebb része. A fájlok sokféleképpen jelezhetik, hogy hiányzik egy adat (pl. üresen hagyják, beírják, hogy "NA", "NULL", vagy esetleg "-999").

Alapértelmezetten a pandas ismeri a leggyakoribb "üres" jelölőket (sentinels), és ezeket automatikusan `NaN` (Not a Number) értékre cseréli a memóriában.

```python
In [26]: !cat examples/ex5.csv
something,a,b,c,d,message
one,1,2,3,4,NA
two,5,6,,8,world
```

A 2. sor végén a "NA", és a 3. sor közepén a `,,` (üres érték a két vessző között) is automatikusan `NaN` lesz.

**Saját hiányzó érték jelölők megadása:**
Ha a te fájlodban valami más jelöli az ürességet (pl. "NULL"), hozzáadhatod a keresendő listához az `na_values` paraméterrel:

```python
In [30]: result = pd.read_csv("examples/ex5.csv", na_values=["NULL"])
```

**Alapértelmezett NA szabályok kikapcsolása:**
Néha baj, ha a pandas túl okos, és mondjuk a "NA" rövidítést (ami mondjuk Észak-Amerikát / North America jelenti a te adatodban) véletlenül hiányzó adatnak veszi.
A `keep_default_na=False` kikapcsolja a beépített keresőt, így a "NA" sima szöveg marad. Ilyenkor te magad (vagy oszloponként lebontva egy szótárral) adhatod meg az `na_values` paraméterben, hogy mit tekintsen üresnek.

```python
In [38]: sentinels = {"message": ["foo", "NA"], "something": ["two"]}
In [39]: pd.read_csv("examples/ex5.csv", na_values=sentinels, keep_default_na=False)
```

### Table 6-2. A legfontosabb pandas.read_csv argumentumok

| Argumentum | Leírás (Description) |
| :--- | :--- |
| `path` | Az elérési út szövegként, URL, vagy fájl-objektum. |
| `sep` vagy `delimiter` | Karakter vagy reguláris kifejezés, ami elválasztja a cellákat egymástól (alapból: `,`). |
| `header` | Annak a sornak a sorszáma, amiből az oszlopneveket (fejlécet) kell venni. Ha nincs fejléc, legyen `None`. |
| `index_col` | Oszlopszám vagy név, amiből a sorok indexe (címkéje) lesz. Lehet lista is hierarchikus indexhez. |
| `names` | Oszlopnevek (fejlécek) listája, ha a fájl nem tartalmazza őket, vagy felül akarjuk írni. |
| `skiprows` | A fájl elején átugrandó sorok száma, **VAGY** a kihagyandó sorok indexeinek listája. |
| `na_values` | Azok a szövegek/számok, amiket `NaN` (hiányzó) értékként kell értelmezni. |
| `keep_default_na` | Használja-e a beépített "hiányzó érték felismerő" listát (Alapból `True`). |
| `comment` | Ha megadsz egy karaktert (pl. `#`), akkor az adott sorban ezután következő részt megjegyzésként kezeli és eldobja. |
| `parse_dates` | Megpróbálja a beolvasott adatokat dátum/idő formátummá alakítani. Lehet `True` (mindent megpróbál) vagy oszlopnevek listája. |
| `keep_date_col` | Ha több oszlopból fűzi össze a dátumot, megtartsa-e az eredeti oszlopokat is (Alapból `False`). |
| `converters` | Egy szótár, ami oszlopokhoz rendel hozzá egyedi Python függvényeket, hogy rögtön beolvasáskor átalakítsa az adatot. |
| `dayfirst` | Nemzetközi dátumformátum kényszerítése (pl. `7/6/2012` -> Június 7, nem pedig Július 6). |
| `date_parser` | Egyedi függvény a dátumok értelmezésére. |
| `nrows` | Beolvasandó sorok maximális száma (hasznos gigantikus fájlok előnézetéhez). |
| `iterator` | Ha `True`, egy olyan objektumot ad vissza, amivel darabokban (chunk) olvashatod be a fájlt. |
| `chunksize` | Iteráláskor az egy "darabban" beolvasandó sorok száma. |
| `skip_footer` | A fájl legvégéről kihagyandó sorok száma. |
| `encoding` | Szövegkódolás (pl. `"utf-8"`, ami az alapértelmezett, vagy `"iso-8859-1"` Windows/Magyar ékezetes fájlokhoz). |
| `thousands` | Ezres-elválasztó karakter megadása (pl. `,` vagy `.`), hogy a szöveges számokat számmá tudja konvertálni. |
| `decimal` | Tizedes-elválasztó karakter megadása (pl. `.`, de magyar fájloknál gyakran `,`). |
| `engine` | A CSV értelmező motor: `"c"` (nagyon gyors, alapértelmezett), `"python"` (lassabb, de okosabb bizonyos funkcióknál), `"pyarrow"` (új, modern és szupergyors). |

## Szöveges fájlok olvasása darabokban (Reading Text Files in Pieces)

Amikor nagyon nagy fájlokat (gigabájtok) dolgozol fel, vagy csak a megfelelő olvasási paramétereket próbálod kitalálni, nem érdemes a teljes fájlt a memóriába tölteni.

**Csak az első néhány sor beolvasása (`nrows`):**
Ezzel elkerülheted a teljes fájl beolvasását. Nagyon hasznos előnézethez!

```python
In [43]: pd.read_csv("examples/ex6.csv", nrows=5)
```

**Iterálás a fájlon keresztül darabokban (`chunksize`):**
A `chunksize` (darabméret) paraméter megadásával a pandas nem egy DataFrame-et, hanem egy speciális iterátort (`TextFileReader`) ad vissza. Ezen a fájlon egy `for` ciklussal végig tudsz menni, és mindig csak annyi sort tart a memóriában, amennyit megadtál.

Például iteráljunk végig a fájlon 1000 soros darabokban, és számoljuk össze az értékek gyakoriságát a "key" oszlopban:

```python
chunker = pd.read_csv("examples/ex6.csv", chunksize=1000)
tot = pd.Series([], dtype='int64')

for piece in chunker:
    # A darab 'key' oszlopának gyakoriságát hozzáadjuk az eddigi összeghez
    tot = tot.add(piece["key"].value_counts(), fill_value=0)

tot = tot.sort_values(ascending=False)
```

## Adatok kiírása szöveges formátumban (Writing Data to Text Format)

Ahogy beolvasni, úgy menteni is tudjuk az adatainkat. A DataFrame `.to_csv()` metódusa erre való.

```python
In [48]: data = pd.read_csv("examples/ex5.csv")

# Fájlba írás:
In [50]: data.to_csv("examples/out.csv")

# Nézzük meg a tartalmát a terminálban:
In [51]: !cat examples/out.csv
,something,a,b,c,d,message
0,one,1,2,3.0,4,
1,two,5,6,,8,world
...
```

(Látható, hogy az első oszlop az index lett (0, 1, 2).)

**Egyéb opciók mentésnél:**

- `sep`: Más elválasztó karakter használata (pl. pipe `|`).
- `na_rep`: Mivel a hiányzó értékek (`NaN`) alapértelmezetten üres stringként `""` jelennek meg, megadhatsz helyette egy `"NA"` vagy `"NULL"` szöveget.
- `index=False` és `header=False`: Kikapcsolható velük a sorcímkék (index) és az oszlopnevek (header) kiírása.
- `columns`: Megadhatod a kimenteni kívánt oszlopok listáját, tetszőleges sorrendben.

(A példában a `sys.stdout` fájl-objektumot használtuk kimenetként, így az adat nem a lemezre ment, hanem egyből kiíródott a konzolra).

```python
In [56]: data.to_csv(sys.stdout, index=False, columns=["a", "b", "c"])
a,b,c
1,2,3.0
5,6,
```

## Egyéb határoló formátumok (A csv modul használata)

Néha a fájl annyira rosszul van formázva (pl. minden adat idézőjelek `"1"` között van), hogy a `pd.read_csv` összezavarodik. Ilyenkor nyúlhatunk a Python beépített `csv` moduljához.

```python
In [57]: !cat examples/ex7.csv
"a","b","c"
"1","2","3"
"1","2","3"
```

A beépített olvasó (`csv.reader`) iterátorként működik, és sorként egy listát ad vissza, eltüntetve az idézőjeleket:

```python
In [58]: import csv
In [59]: f = open("examples/ex7.csv")
In [60]: reader = csv.reader(f)

In [61]: for line in reader:
   ....:     print(line)
['a', 'b', 'c']
['1', '2', '3']
['1', '2', '3']
```

Ezekből a listákból utána Python szótár-kifejezésekkel (comprehension) mi magunk is összerakhatunk egy adatszerkezetet.

> **Gyakorlati tanács:** Csak nagyon extrém esetben írj saját beolvasót. A `pd.read_csv` paramétereivel szinte minden megoldható!

### Table 6-3. CSV dialektus (Dialect) opciók

Ezeket a Python beépített `csv` moduljánál lehet használni.

| Argumentum | Leírás (Description) |
| :--- | :--- |
| `delimiter` | Egykarakteres szöveg a mezők elválasztásához (alapból `,`). |
| `lineterminator` | Sor-záró karakter írásnál (olvasásnál a modul automatikusan felismeri a rendszert). |
| `quotechar` | Idézőjel karakter (alapból `"`). |
| `quoting` | Idézőjel-szabály. Pl. `QUOTE_MINIMAL` (csak a speciális karaktert tartalmazókat), `QUOTE_ALL` (mindent), `QUOTE_NONE` (semmit). |
| `skipinitialspace` | Figyelmen kívül hagyja a szóközöket az elválasztó után. |
| `doublequote` | Ha `True`, megduplázza a mezőn belüli idézőjeleket. |
| `escapechar` | Szöveg, amivel a határolójelet "menekítjük" (escape), ha a quoting be van tiltva. |

## JSON adatok olvasása

A JSON (JavaScript Object Notation) a webes API-k standard adatformátuma. Szabadabb formátumú, mint a CSV; leginkább beágyazott Python szótárakra és listákra hasonlít.

A Python beépített `json` moduljával a `json.loads` szövegből Python objektumot (szótárat) csinál, a `json.dumps` pedig Python objektumból JSON szöveget.

```python
In [68]: import json
In [69]: result = json.loads(obj) # obj egy JSON szöveg
```

A pandas beépített `pd.read_json` függvénye automatikusan Series vagy DataFrame objektummá tud alakítani speciális szerkezetű JSON fájlokat. Alapértelmezetten feltételezi, hogy a JSON tömb minden eleme egy sor (row) a táblázatban.

```python
In [75]: !cat examples/example.json
[{"a": 1, "b": 2, "c": 3},
 {"a": 4, "b": 5, "c": 6},
 {"a": 7, "b": 8, "c": 9}]

In [76]: data = pd.read_json("examples/example.json")
```

DataFrame-ből JSON-t csinálni a `.to_json()` metódussal lehet.

## XML és HTML: Web Scraping (Adatextrakció weblapokról)

A weblapokon nagyon sok adat szép `<table>` tagek közé (HTML táblázatokba) van rejtve. A pandas `read_html` függvénye ezeket az oldalakat letölti, megkeresi bennük a táblázatokat, és egy DataFrame-ekből álló listát ad vissza.

Mivel a háttérben külső modulokat használ, előtte telepíteni kell őket:

```bash
conda install lxml beautifulsoup4 html5lib
```

**Példa:** olvassuk be az amerikai bankcsődökről szóló HTML táblázatot:

```python
In [80]: tables = pd.read_html("examples/fdic_failed_bank_list.html")
In [81]: len(tables) # Kiderült, hogy 1 db táblázatot talált
Out[81]: 1

In [82]: failures = tables[0] # Kivesszük a DataFrame-et a listából
In [83]: failures.head()
Out[83]: 
                      Bank Name             City  ST   CERT \
0                   Allied Bank         Mulberry  AR     91   
1  The Woodbury Banking Company         Woodbury  GA  11297   
...
```

Ezután a megszerzett adatokkal normál adatbázisként dolgozhatunk. Például a `pd.to_datetime()` segítségével dátummá alakíthatjuk a "Closing Date" (Zárás dátuma) oszlopot, és kiszámolhatjuk, melyik évben hány bankcsőd volt:

```python
In [84]: close_timestamps = pd.to_datetime(failures["Closing Date"])
In [85]: close_timestamps.dt.year.value_counts()
Out[85]: 
2010    157
2009    140
2011     92
...
```

## XML fájlok feldolgozása `lxml.objectify` segítségével

Az XML egy másik nagyon gyakori strukturált adatformátum, ami a HTML-hez hasonló, de általánosabb, hierarchikusan egymásba ágyazott (nested) adatok tárolására való.

A példában a New York-i közlekedési vállalat (MTA) vonatainak teljesítményadatait dolgozzuk fel egy XML fájlból. A fájlban `<INDICATOR>` tagek (címkék) között vannak az egyes rekordok.

**Az `lxml` könyvtár használata manuális beolvasásra:**

```python
In [86]: from lxml import objectify
In [87]: path = "datasets/mta_perf/Performance_MNR.xml"
In [88]: with open(path) as f:
   ....:     parsed = objectify.parse(f)
In [89]: root = parsed.getroot()
```

**Magyarázat:** Az `objectify.parse()` beolvassa a fájlt egy memóriastruktúrába, a `.getroot()` pedig visszaadja a dokumentum legfelső (gyökér) elemét.

Készítsünk a fa struktúrából egy adatszótárakból álló listát, amiből már tudunk DataFrame-et építeni:

```python
data = []
skip_fields = ["PARENT_SEQ", "INDICATOR_SEQ", "DESIRED_CHANGE", "DECIMAL_PLACES"]

for elt in root.INDICATOR:
    el_data = {}
    for child in elt.getchildren():
        if child.tag in skip_fields:
            continue
        el_data[child.tag] = child.pyval
    data.append(el_data)
```

**Magyarázat sorról sorra:**

- `data = []`: Létrehozunk egy üres listát a végső adatoknak.
- `skip_fields = [...]`: Ezeket a tageket (oszlopokat) nem akarjuk beolvasni, kihagyjuk őket.
- `for elt in root.INDICATOR:`: Végigmegyünk a gyökérelem összes `<INDICATOR>` nevű "gyerek" elemén (ezek a konkrét sorok/rekordok).
- `el_data = {}`: Minden rekordnak készítünk egy üres szótárat.
- `for child in elt.getchildren():`: Végigmegyünk a rekord belsejében lévő mezőkön (pl. `<AGENCY_NAME>`, `<PERIOD_YEAR>`).
- `if child.tag in skip_fields: continue`: Ha a mező neve benne van a kihagyandó listában, ugorjunk a következőre.
- `el_data[child.tag] = child.pyval`: A szótárba beletesszük a mező nevét (`.tag`) kulcsként, és a benne lévő értéket (`.pyval`) értékként.
- `data.append(el_data)`: A kész rekordot (szótárat) beletesszük a nagy listába.

Ebből már könnyű táblázatot csinálni:

```python
In [91]: perf = pd.DataFrame(data)
```

**A modern, egysoros megoldás (pandas beépített XML olvasója):**
Bár a fenti kód jól mutatja a logikát, a modern pandas-ban van már dedikált függvény erre, ami egy sorban megoldja az egészet!

```python
In [93]: perf2 = pd.read_xml(path)
In [94]: perf2.head()
# Visszaadja a tökéletesen formázott DataFrame-et!
```

## 6.2 Bináris adatformátumok (Binary Data Formats)

Szöveges (CSV) fájlok helyett sokkal gyorsabb és helytakarékosabb, ha bináris formátumban tároljuk az adatokat a lemezen.

### 1. A Python Pickle formátuma

A legegyszerűbb módszer adatok elmentésére (szerializációjára) a Python beépített `pickle` modulja. A pandas objektumok mind rendelkeznek `to_pickle` metódussal.

**Mentés fájlba:**

```python
In [95]: frame = pd.read_csv("examples/ex1.csv")
In [97]: frame.to_pickle("examples/frame_pickle")
```

**Visszatöltés a memóriába:**

```python
In [98]: pd.read_pickle("examples/frame_pickle")
```

> **FONTOS FIGYELMEZTETÉS:** A Pickle **kizárólag rövid távú tárolásra** ajánlott! Csak a Python érti meg (más nyelvek, pl. R vagy Java nem), és nem garantált, hogy egy ma elmentett Pickle fájlt 3 év múlva, egy újabb pandas verzióval is meg tudsz majd nyitni. Ha megbízható tárolás kell, használj CSV-t, Parquet-t vagy HDF5-öt.

### 2. Microsoft Excel fájlok olvasása és írása

Az adatelemzők élete nagyrészt az Excel fájlok körül forog. A pandas kiválóan kezeli a régi (`.xls`) és új (`.xlsx`) formátumokat, de a háttérben szüksége van az `xlrd` (régi) vagy `openpyxl` (új) csomagokra. (Ha nincsenek meg: `conda install openpyxl xlrd`).

**Olvasás az `ExcelFile` osztállyal (gyorsabb, ha több munkalapot olvasol):**

```python
In [101]: xlsx = pd.ExcelFile("examples/ex1.xlsx")
In [102]: xlsx.sheet_names      # Megmutatja a munkalapok nevét
Out[102]: ['Sheet1']

# Munkalap beolvasása DataFrame-ként:
In [103]: xlsx.parse(sheet_name="Sheet1")

# Megadhatjuk, hogy melyik oszlop legyen az index:
In [104]: xlsx.parse(sheet_name="Sheet1", index_col=0)
```

**Egyszerű, egysoros olvasás (ha csak egy lap kell):**

```python
In [105]: frame = pd.read_excel("examples/ex1.xlsx", sheet_name="Sheet1")
```

**Írás Excel formátumba:**
Létre kell hozni egy `ExcelWriter` objektumot, és beleírni a DataFrame-et a `.to_excel()` segítségével.

```python
In [107]: writer = pd.ExcelWriter("examples/ex2.xlsx")
In [108]: frame.to_excel(writer, "Sheet1")
In [109]: writer.save()

# Vagy a rövidebb megoldás (a Writer kikerülésével):
In [110]: frame.to_excel("examples/ex2.xlsx")
```

### 3. HDF5 formátum használata (Hatalmas adatokhoz)

A HDF5 (Hierarchical Data Format) egy tudományos standard. Szuper választás olyan gigantikus tömbökhöz/táblázatokhoz, amik nem férnek be a gép memóriájába (RAM). Támogatja a beépített tömörítést, és lehetővé teszi, hogy a fájl egy nagyon apró részletét olvasd csak be a lemezről, anélkül, hogy az egészet betöltenéd. (Szükséges csomag hozzá: `conda install pytables`)

**Mentés HDF5 formátumba (Szótár-szerű API-val):**

```python
In [113]: frame = pd.DataFrame({"a": np.random.standard_normal(100)})
In [114]: store = pd.HDFStore("examples/mydata.h5")
In [115]: store["obj1"] = frame        # Teljes tábla mentése
In [116]: store["obj1_col"] = frame["a"] # Csak egy oszlop mentése
```

Itt a `store` úgy működik, mint egy mappa a fájlon belül. Később ugyanígy szótárként hivatkozva le is kérhetjük: `store["obj1"]`.

**A "Table" (Tábla) formátum és a Lekérdezések (Queries):**
A `HDFStore` kétféleképpen tud tárolni: `"fixed"` (alapértelmezett, de egyszerű) és `"table"` (kicsit lassabb írásnál, de sokkal okosabb).
A `"table"` formátum támogatja, hogy úgy keress a fájlban, mint egy SQL adatbázisban!

```python
# Írás "table" formátumban a .put() metódussal:
In [119]: store.put("obj2", frame, format="table")

# Lekérdezés (Query) - Csak a 10 és 15 közötti indexeket kérjük le a lemezről:
In [120]: store.select("obj2", where=["index >= 10 and index <= 15"])
Out[120]: 
           a
10  1.007189
...
15  0.886429

# Fájl bezárása a végén:
In [121]: store.close()
```

**Rövidebb megoldás (A pandas beépített HDF5 funkciói):**
Nem muszáj mindig `HDFStore` objektumot csinálni, működik az egyszerű beépített metódus is:

```python
# Mentés:
In [122]: frame.to_hdf("examples/mydata.h5", "obj3", format="table")

# Betöltés szűrővel:
In [123]: pd.read_hdf("examples/mydata.h5", "obj3", where=["index < 5"])
```

> **FONTOS FIGYELMEZTETÉS A HDF5-RŐL:**
> A HDF5 **NEM adatbázis**. Kifejezetten "write-once, read-many" (egyszer megírod, sokszor olvasod) adatokhoz találták ki. Ha egyszerre többen (vagy több szálon futó kód) próbál beírni a fájlba, a fájl menthetetlenül megsérülhet / korruptálódhat! (Ha elosztott vagy felhő alapú rendszerben dolgozol – pl. Amazon S3 –, a HDF5 helyett az Apache Parquet formátum a javasolt).
>
> ## Web API-k használata (Interacting with Web APIs)

Számos weboldal és szolgáltatás biztosít nyilvános API-t (Alkalmazásprogramozási Felület), amelyeken keresztül JSON vagy más formátumú adatfolyamok (data feeds) érhetők el. Pythonban a legelterjedtebb és leginkább ajánlott mód ezek elérésére a `requests` csomag (telepítése: `conda install requests`).

**Adatok lekérése egy API-ról:**
A példában a pandas GitHub tárolójának (repository) utolsó 30 hibajegyét (issue) kérjük le egy HTTP GET kéréssel.

```python
In [126]: import requests
In [127]: url = "https://api.github.com/repos/pandas-dev/pandas/issues"
In [128]: resp = requests.get(url)
In [129]: resp.raise_for_status()
In [130]: resp
Out[130]: <Response [200]>

In [131]: data = resp.json()
In [132]: data[0]["title"]
Out[132]: 'REF: make copy keyword non-stateful'
```

**Sorról sorra történő magyarázat:**

- `requests.get(url)`: Elküld egy HTTP GET kérést a megadott webcímre, és a választ a `resp` változóba menti. A `<Response [200]>` azt jelenti, hogy a kérés sikeres volt.
- `resp.raise_for_status()`: Jó gyakorlat! Ellenőrzi, hogy történt-e valamilyen HTTP hiba (pl. 404 Not Found vagy 500 Server Error). Ha igen, a program leáll egy kivétellel, így nem próbálunk meg hibás adatot feldolgozni.
- `resp.json()`: A válaszban kapott nyers JSON szöveget automatikusan beépített Python objektummá (szótárrá vagy listává) alakítja. Jelen esetben egy szótárakból álló listát kapunk.
- `data[0]["title"]`: A lista első eleméből (ami egy szótár) lekérjük a "title" (cím) kulcshoz tartozó értéket.

**A kapott JSON átalakítása DataFrame-mé:**
Mivel a `data` egy szótárakból álló lista, közvetlenül átadhatjuk a `pd.DataFrame` konstruktornak, és a `columns` paraméterrel kiválaszthatjuk, hogy a rengeteg JSON mezőből melyik oszlopokra van szükségünk.

```python
In [133]: issues = pd.DataFrame(data, columns=["number", "title", "labels", "state"])
In [134]: issues
Out[134]: 
    number                                              title  \
0    48062                REF: make copy keyword non-stateful   
1    48061                              STYLE: upgrade flake8   
2    48060  DOC: "Creating a Python environment" in "Creat...   
...
```

**Magyarázat:** A pandas a szótárak kulcsait megfeleltette a kért oszlopneveknek, és egy gyönyörű, strukturált adattáblát készített a webes adatokból.

## 6.4 Interakció adatbázisokkal (Interacting with Databases)

Üzleti környezetben az adatok ritkán vannak egyszerű szöveges vagy Excel fájlokban. Sokkal gyakoribbak a relációs adatbázisok (pl. SQL Server, PostgreSQL, MySQL vagy SQLite). A pandas kiváló eszközökkel rendelkezik az SQL lekérdezések eredményeinek DataFrame-be történő betöltéséhez.

**Adatbázis létrehozása és adatok beszúrása (Sima Python módszer):**
A beépített `sqlite3` modult használjuk egy helyi adatbázis teszteléséhez.

```python
In [135]: import sqlite3
In [136]: query = """
   .....: CREATE TABLE test
   .....: (a VARCHAR(20), b VARCHAR(20),
   .....:  c REAL,        d INTEGER
   .....: );"""
In [137]: con = sqlite3.connect("mydata.sqlite")
In [138]: con.execute(query)
In [139]: con.commit()
```

**Magyarázat:** Létrehozunk egy kapcsolatot (`connect`) egy `mydata.sqlite` nevű fájlhoz. A `query` változóban lévő SQL paranccsal létrehozunk egy `test` nevű táblát 4 oszloppal (szövegek, valós szám, egész szám). Az `execute` lefuttatja, a `commit` pedig véglegesíti a műveletet a lemezen.

```python
In [140]: data = [("Atlanta", "Georgia", 1.25, 6),
   .....:         ("Tallahassee", "Florida", 2.6, 3),
   .....:         ("Sacramento", "California", 1.7, 5)]
In [141]: stmt = "INSERT INTO test VALUES(?, ?, ?, ?)"
In [142]: con.executemany(stmt, data)
In [143]: con.commit()
```

**Magyarázat:** A `data` változó egy tuple-ökből álló lista (ezek lesznek a sorok). A `stmt` (statement) egy paraméteres SQL beszúró parancs. Az `executemany` segítségével a teljes listát egyszerre írjuk be az adatbázisba.

**Adatok lekérdezése és DataFrame-be alakítása (Sima Python módszer):**

```python
In [144]: cursor = con.execute("SELECT * FROM test")
In [145]: rows = cursor.fetchall()
In [146]: rows
Out[146]: 
[('Atlanta', 'Georgia', 1.25, 6),
 ('Tallahassee', 'Florida', 2.6, 3),
 ('Sacramento', 'California', 1.7, 5)]

In [147]: cursor.description
Out[147]: 
(('a', None, None, None, None, None, None),
 ('b', None, None, None, None, None, None),
 ...)

In [148]: pd.DataFrame(rows, columns=[x[0] for x in cursor.description])
Out[148]: 
             a           b     c  d
0      Atlanta     Georgia  1.25  6
1  Tallahassee     Florida  2.60  3
2   Sacramento  California  1.70  5
```

**Magyarázat:**

- A `fetchall()` lekéri a lekérdezés összes sorát, de ez csak egy listát ad a nyers adatokról, oszlopnevek nélkül.
- Az oszlopneveket a `cursor.description` metaadataiból kell kinyerni (itt egy list comprehension-t `[x[0] for x in ...]` használunk, ami kiválogatja a tuple-ök legelső elemét, azaz a nevet).
- Ezután tudjuk átadni az adatokat és az oszlopneveket a `pd.DataFrame` konstruktornak. Ez a folyamat (munging) elég fárasztó lenne minden lekérdezésnél.

**A modern, pandas módszer (SQLAlchemy használatával):**
A fenti manuális oszlop-kivonatolás helyett a pandas az SQLAlchemy könyvtárra támaszkodik, amely elfedi a különböző SQL adatbázisok (MySQL, PostgreSQL stb.) közötti különbségeket.

```python
In [149]: import sqlalchemy as sqla
In [150]: db = sqla.create_engine("sqlite:///mydata.sqlite")
In [151]: pd.read_sql("SELECT * FROM test", db)
Out[151]: 
             a           b     c  d
0      Atlanta     Georgia  1.25  6
1  Tallahassee     Florida  2.60  3
2   Sacramento  California  1.70  5
```

**Magyarázat:**

- `sqla.create_engine(...)`: Létrehoz egy univerzális adatbázis-motort a megadott kapcsolati string alapján.
- `pd.read_sql(query, connection)`: Ez a legfontosabb függvény. Egyszerűen átadsz neki egy SQL lekérdezést (vagy táblanevet) és az adatbázis kapcsolatot. A pandas a háttérben elvégzi az összes munkát (lekérdezés, oszlopnevek kinyerése, típusok konvertálása), és egy tökéletesen formázott DataFrame-et ad vissza.

## 6.5 Összegzés (Conclusion)

Az adatok elérése és beolvasása általában az első lépés az adatelemzési folyamatban. Ebben a fejezetben megismertük a legfontosabb eszközöket, a fájlok (CSV, Excel), bináris adatok (HDF5), webes API-k (JSON) és relációs adatbázisok (SQL) kezeléséhez.
