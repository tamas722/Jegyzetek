# 5. Fejezet: Ismerkedés a pandas könyvtárral

## Bevezetés

A **pandas** olyan adatszerkezeteket és eszközöket tartalmaz, amelyek a Pythonban történő adattisztítást és elemzést gyorssá és kényelmessé teszik. Gyakran használjuk a NumPy-jal, a SciPy-jal, a matplotlib-bel és gépi tanulási (scikit-learn) könyvtárakkal együtt.

**Mi a különbség a NumPy és a pandas között?**
Míg a NumPy-t homogén (azonos típusú) numerikus tömbökön végzett műveletekre tervezték, a pandas kifejezetten **táblázatos és heterogén** (sokféle típust tartalmazó) adatok kezelésére lett kitalálva.

**Importálási konvenciók:**
Ahogy a NumPy-nál az `np`, úgy a pandas-nál a `pd` a szabványos rövidítés:

```python
In [1]: import numpy as np
In [2]: import pandas as pd
```

Mivel a két legfontosabb adatszerkezetét (a `Series`-t és a `DataFrame`-et) folyton használni fogjuk, ezeket érdemes közvetlenül is beimportálni:

```python
In [3]: from pandas import Series, DataFrame
```

## 5.1 Bevezetés a pandas adatszerkezeteibe

A pandas két "igásló" (workhorse) adatszerkezetre épül: a **Series**-re és a **DataFrame**-re.

### A Series (Egydimenziós adatsor)

**Fő koncepció:** A Series egy egydimenziós, tömbszerű objektum. Tartalmaz egy adatsort (értékeket), és egy hozzájuk kapcsolódó címkesort, amit **indexnek** nevezünk.

**Létrehozás egyszerű listából:**

```python
In [14]: obj = pd.Series([4, 7, -5, 3])
In [15]: obj
Out[15]: 
0    4
1    7
2   -5
3    3
dtype: int64
```

**Magyarázat:** Létrehoztunk egy Series-t négy számból. Mivel nem adtuk meg, mi legyen az index (a címke), a pandas automatikusan generált egy 0-tól induló sorszámozást (0, 1, 2, 3). A bal oszlop az index, a jobb oszlop maga az érték.

Az értékeket és az indexet külön is lekérdezhetjük:

```python
In [16]: obj.array
Out[16]: 
<PandasArray>
[4, 7, -5, 3]
Length: 4, dtype: int64

In [17]: obj.index
Out[17]: RangeIndex(start=0, stop=4, step=1)
```

**Létrehozás egyéni indexszel (címkékkel):**
Gyakran érdemes beszédes címkéket rendelni az adatokhoz:

```python
In [18]: obj2 = pd.Series([4, 7, -5, 3], index=["d", "b", "a", "c"])
In [19]: obj2
Out[19]: 
d    4
b    7
a   -5
c    3
dtype: int64

In [20]: obj2.index
Out[20]: Index(['d', 'b', 'a', 'c'], dtype='object')
```

**Értékek lekérése, módosítása és szűrése címkék alapján:**
A Series pontosan úgy indexelhető, mint egy szótár, de támogatja a NumPy-szerű maszkolást (szűrést) is.

```python
In [21]: obj2["a"]         # Egy konkrét elem lekérése
Out[21]: -5

In [22]: obj2["d"] = 6     # Érték módosítása

In [23]: obj2[["c", "a", "d"]]  # Több elem lekérése egyszerre (lista átadásával)
Out[23]: 
c    3
a   -5
d    6
dtype: int64
```

**Vektorizált (tömbösített) matematikai műveletek használata:**

```python
In [24]: obj2[obj2 > 0]    # Csak a pozitív számok szűrése
Out[24]: 
d    6
b    7
c    3
dtype: int64

In [25]: obj2 * 2          # Minden elem szorzása kettővel
Out[25]: 
d    12
b    14
a   -10
c     6
dtype: int64

In [26]: import numpy as np
In [27]: np.exp(obj2)      # NumPy függvény (e^x) alkalmazása a Series-en
Out[27]: 
d     403.428793
b    1096.633158
a       0.006738
c      20.085537
dtype: float64
```

> **Fontos szabály:** Amikor NumPy matematikai műveleteket vagy szűrést végzünk, a pandas mindig megőrzi az index-érték kapcsolatot! A címke (pl. "d") sosem válik el a hozzá tartozó számtól.

**A Series mint szótár (Dictionary):**
Gondolhatsz úgy a Series-re, mint egy fix hosszúságú, rendezett szótárra. Ugyanúgy működik rajta az `in` kulcsszó:

```python
In [28]: "b" in obj2
Out[28]: True
In [29]: "e" in obj2
Out[29]: False
```

Ha az adataink alapból egy Python szótárban vannak, közvetlenül Series-t csinálhatunk belőle:

```python
In [30]: sdata = {"Ohio": 35000, "Texas": 71000, "Oregon": 16000, "Utah": 5000}
In [31]: obj3 = pd.Series(sdata)
In [32]: obj3
Out[32]: 
Ohio      35000
Texas     71000
Oregon    16000
Utah       5000
dtype: int64

# És vissza is alakíthatjuk:
In [33]: obj3.to_dict()
Out[33]: {'Ohio': 35000, 'Texas': 71000, 'Oregon': 16000, 'Utah': 5000}
```

**Hiányzó adatok (NaN - Not a Number) kezelése:**
Ha szótárból készítünk Series-t, de mi magunk mondjuk meg, mik legyenek az indexek, a pandas megpróbálja párosítani őket.

```python
In [34]: states = ["California", "Ohio", "Oregon", "Texas"]
In [35]: obj4 = pd.Series(sdata, index=states)
In [36]: obj4
Out[36]: 
California        NaN
Ohio          35000.0
Oregon        16000.0
Texas         71000.0
dtype: float64
```

**Magyarázat:** Mivel a szótárban nem volt "California", a pandas egy `NaN` (hiányzó adat) jelzőt tett a helyére. "Utah" pedig hiába volt a szótárban, mivel nem kértük a `states` listában, kimaradt.

Hiányzó adatok keresésére a `pd.isna` és `pd.notna` (vagy a példány metódusok) valók:

```python
In [37]: pd.isna(obj4)   # Igaz, ha az adat hiányzik
Out[37]: 
California     True
Ohio          False
Oregon        False
Texas         False
dtype: bool

In [39]: obj4.isna()     # Ugyanez, de a Series saját metódusaként meghívva
```

**Automatikus adatigazítás (Data Alignment):**
Ez az egyik legkirályabb funkciója a pandasnak. Ha két Series-t összeadsz, a pandas automatikusan a címkék alapján párosítja össze őket (mint egy OUTER JOIN adatbázisoknál).

```python
In [42]: obj3 + obj4
Out[42]: 
California         NaN
Ohio           70000.0
Oregon         32000.0
Texas         142000.0
Utah               NaN
dtype: float64
```

**Magyarázat:** A gép megtalálta Ohiot mindkét helyen (35000 + 35000 = 70000). Ahol az egyiknél hiányzott az adat (pl. California, Utah), az összeg is `NaN` lett.

**Elnevezések és index módosítás:**
A Series-nek és magának az index-oszlopnak is adhatunk nevet:

```python
In [43]: obj4.name = "population"
In [44]: obj4.index.name = "state"
In [45]: obj4
Out[45]: 
state
California        NaN
Ohio          35000.0
Oregon        16000.0
Texas         71000.0
Name: population, dtype: float64
```

Egy meglévő Series indexe helyben (in-place) lecserélhető egy új listával:

```python
In [47]: obj.index = ["Bob", "Steve", "Jeff", "Ryan"]
```

### A DataFrame (Adattábla)

**Fő koncepció:** A DataFrame egy kétdimenziós, téglalap alakú adattábla. Oszlopok rendezett gyűjteménye, ahol minden oszlop más-más típusú lehet (szám, szöveg, logikai). Képzeld el úgy, mint az Excelt, vagy egy olyan szótárat, ahol a kulcsok az oszlopnevek, az értékek pedig Series objektumok, amelyek mind ugyanazon az indexen (sorcímkéken) osztoznak.

**Létrehozás szótárból (ahol az értékek egyforma hosszú listák):**

```python
data = {"state": ["Ohio", "Ohio", "Ohio", "Nevada", "Nevada", "Nevada"],
        "year": [2000, 2001, 2002, 2001, 2002, 2003],
        "pop": [1.5, 1.7, 3.6, 2.4, 2.9, 3.2]}

frame = pd.DataFrame(data)

In [50]: frame
Out[50]: 
    state  year  pop
0    Ohio  2000  1.5
1    Ohio  2001  1.7
2    Ohio  2002  3.6
3  Nevada  2001  2.4
4  Nevada  2002  2.9
5  Nevada  2003  3.2
```

> 📍 **[KÉP HELYE: Figure 5-1. How pandas DataFrame objects look in Jupyter]**
>
> **Megjegyzés:** A Jupyter Notebook egy szép, HTML alapú, kényelmesen olvasható táblázatot renderel belőle.

Nagy táblázatoknál a `.head()` az első 5, a `.tail()` az utolsó 5 sort mutatja meg:

```python
In [51]: frame.head()
In [52]: frame.tail()
```

**Oszlopok sorrendjének meghatározása és új (üres) oszlop hozzáadása:**

```python
In [53]: pd.DataFrame(data, columns=["year", "state", "pop"])
# Az oszlopok most a megadott sorrendben jelennek meg.

In [54]: frame2 = pd.DataFrame(data, columns=["year", "state", "pop", "debt"])
In [55]: frame2
Out[55]: 
   year   state  pop debt
0  2000    Ohio  1.5  NaN
1  2001    Ohio  1.7  NaN
...
```

**Magyarázat:** A `debt` (adósság) oszlop nem létezett a bemeneti `data` szótárban, ezért a pandas létrehozta, és feltöltötte `NaN` (hiányzó) értékekkel.

**Oszlopok lekérdezése (Series-ként):**

```python
In [57]: frame2["state"]      # Szótár-stílusú hozzáférés (Ez mindig működik)

In [58]: frame2.year          # Attribútum-stílusú (dot) hozzáférés
```

> **Fontos:** A "pont" (dot) stílusú hozzáférés (`frame2.year`) csak akkor működik, ha az oszlopnév egy érvényes Python változónév (pl. nincs benne szóköz) és nem ütközik egy beépített DataFrame metódus nevével. A `frame2["state"]` stílus mindig biztonságos!

**Sorok lekérdezése pozíció alapján (`loc` és `iloc`):**

```python
In [59]: frame2.loc[1]     # Sor kiválasztása a CÍMKE (index neve) alapján
Out[59]: 
year      2001
state     Ohio
pop        1.7
debt       NaN
Name: 1, dtype: object

In [60]: frame2.iloc[2]    # Sor kiválasztása a POZÍCIÓ (hányadik sor) alapján
```

**Oszlopok módosítása és új oszlopok létrehozása:**

```python
In [61]: frame2["debt"] = 16.5                    # Minden sor 16.5 lesz
In [63]: frame2["debt"] = np.arange(6.)           # 0.0-tól 5.0-ig kapják az értéket
```

Ha egy új (hiányos) Series-t adsz át egy oszlopnak, az indexek mentén tökéletesen illeszkedni fog, és `NaN` kerül oda, amire nem adtál meg értéket:

```python
In [65]: val = pd.Series([-1.2, -1.5, -1.7], index=["two", "four", "five"])
In [66]: frame2["debt"] = val
```

Új oszlop létrehozása nagyon egyszerű, csak értéket kell rendelni egy nem létező oszlophoz. Oszlop törléséhez a `del` kulcsszót használjuk:

```python
In [68]: frame2["eastern"] = frame2["state"] == "Ohio" # Új logikai oszlop
In [70]: del frame2["eastern"]                         # Oszlop törlése
```

> **Figyelem:** Új oszlopot **NEM** hozhatsz létre `frame2.eastern = ...` szintaxissal! Erre mindenképp a szögletes zárójeles módszert kell használni. Továbbá, az indexelt oszlop (`frame2["state"]`) csak egy nézetet ad vissza, nem másolatot. Ha helyben módosítod, a DataFrame is változik!

**Létrehozás beágyazott szótárból:**
Ha egy szótárban újabb szótárak vannak, a pandas a külső kulcsokból oszlopokat, a belső kulcsokból (index) sorokat csinál.

```python
In [72]: populations = {"Ohio": {2000: 1.5, 2001: 1.7, 2002: 3.6},
   ....:                "Nevada": {2001: 2.4, 2002: 2.9}}
In [73]: frame3 = pd.DataFrame(populations)
In [74]: frame3
Out[74]: 
      Ohio  Nevada
2000   1.5     NaN
2001   1.7     2.4
2002   3.6     2.9
```

**Transzponálás (Sorok és oszlopok felcserélése):**
Pont, mint a NumPy-nál, a `.T` elvégzi a transzponálást:

```python
In [75]: frame3.T
Out[75]: 
        2000  2001  2002
Ohio     1.5   1.7   3.6
Nevada   NaN   2.4   2.9
```

**Konvertálás tiszta NumPy tömbbé:**
A DataFrame `to_numpy()` metódusa eltünteti a címkéket, és csak a nyers értékeket adja vissza egy 2D NumPy mátrix formájában. Ha vegyes típusok voltak a táblázatban, a NumPy olyan típust választ (pl. `object`), ami minden elemet be tud fogadni.

```python
In [82]: frame3.to_numpy()
Out[82]: 
array([[1.5, nan],
       [1.7, 2.4],
       [3.6, 2.9]])
```

#### Table 5-1. Lehetséges bemeneti adatok a DataFrame létrehozásához

| Típus (Type) | Megjegyzések (Notes) |
| :--- | :--- |
| **2D ndarray** | Egy NumPy adatmátrix; opcionálisan átadhatók hozzá a sor- és oszlopcímkék. |
| **Tömbökből, listákból vagy tuple-ökből álló Szótár (Dict)** | Minden szekvencia egy oszloppá válik a DataFrame-ben. Minden szekvenciának egyforma hosszúnak kell lennie. |
| **NumPy strukturált (record) tömb** | Úgy kezeli, mint a "Tömbökből álló szótár" esetet. |
| **Series objektumokból álló Szótár** | Minden Series egy oszloppá válik; a Series-ek indexei unióba (egybe) olvadnak, ez lesz a táblázat sor-indexe (kivéve, ha explicite mást adunk meg). |
| **Szótárakból álló Szótár (Nested dict)** | Minden belső szótár egy oszloppá válik; a belső kulcsok uniójából lesz a sor-index (ugyanúgy, mint a Series-ek esetén). |
| **Szótárakból vagy Series-ekből álló Lista** | Minden listaelem egy sorrá válik a DataFrame-ben. A szótárkulcsok vagy Series indexek uniója lesz az oszlopok neve (fejléc). |
| **Listákból vagy tuple-ökből álló Lista** | Úgy kezeli, mint a "2D ndarray" esetet. |
| **Egy másik DataFrame** | Átveszi az eredeti DataFrame adatait, indexeit és oszlopait, hacsak nem adunk meg explicit újakat. |
| **NumPy MaskedArray** | Hasonló a "2D ndarray"-hez, de a maszkolt értékek `NaN` (hiányzó) értékként jelennek meg az eredményben. |

## Az Index objektumok (Index Objects)

**Fő koncepció:** A pandas Index objektumai felelősek a tengelycímkék (pl. a DataFrame oszlopnevei vagy a Series sor-címkéi) és egyéb metaadatok tárolásáért. Bármilyen listát is adsz át címkeként a pandas-nak, az a háttérben automatikusan egy Index objektummá alakul.

```python
In [84]: obj = pd.Series(np.arange(3), index=["a", "b", "c"])
In [85]: index = obj.index
In [86]: index
Out[86]: Index(['a', 'b', 'c'], dtype='object')

In [87]: index[1:]
Out[87]: Index(['b', 'c'], dtype='object')
```

**Magyarázat:** Lekértük a Series indexét, ami egy önálló Index típusú objektum, és ugyanúgy szeletelhető (slicing), mint a normál listák vagy NumPy tömbök.

> **Fontos szabály: Az Index módosíthatatlan (Immutable)!**
> Nem változtathatod meg az Index egy konkrét elemét közvetlenül:

```python
index[1] = "d"  # TypeError (Hibát dob!)
```

Miért jó ez? Mert a módosíthatatlanság (immutability) miatt tökéletesen biztonságos megosztani ugyanazt az Index objektumot több különböző Series vagy DataFrame között, anélkül, hogy félni kéne attól, hogy valaki véletlenül elrontja az összeset.

```python
In [88]: labels = pd.Index(np.arange(3))
In [90]: obj2 = pd.Series([1.5, -2.5, 0], index=labels)
In [92]: obj2.index is labels
Out[92]: True
```

**Az Index olyan, mint egy halmaz (Set), de megenged duplikációkat:**
Ellenőrizhetjük, hogy egy oszlopnév vagy címke benne van-e az indexben az `in` kulcsszóval:

```python
In [94]: frame3.columns
Out[94]: Index(['Ohio', 'Nevada'], dtype='object', name='state')

In [95]: "Ohio" in frame3.columns
Out[95]: True

In [96]: 2003 in frame3.index
Out[96]: False
```

Ellentétben a normál Python halmazokkal (set), a pandas Index tartalmazhat azonos (duplikált) címkéket:

```python
In [97]: pd.Index(["foo", "foo", "bar", "bar"])
Out[97]: Index(['foo', 'foo', 'bar', 'bar'], dtype='object')
```

Ha egy ilyen duplikált címkére hivatkozol a lekérdezésnél, a pandas az összes egyező találatot vissza fogja adni.

### Table 5-2. Hasznos Index metódusok és tulajdonságok

| Metódus/Tulajdonság | Leírás (Description) |
| :--- | :--- |
| `append()` | Összefűzi egy másik Index objektummal, egy új Indexet hozva létre. |
| `difference()` | Halmaz-különbség: kiszámítja a különbséget (ami az egyikben benne van, a másikban nincs) Indexként. |
| `intersection()` | Halmaz-metszet: a közös elemeket adja vissza. |
| `union()` | Halmaz-unió: a két Index összes elemét egyesíti. |
| `isin()` | Logikai (Boolean) tömböt ad: minden elemről megmondja, benne van-e a megadott gyűjteményben. |
| `delete()` | Új Indexet hoz létre az `i`-edik pozíción lévő elem törlésével. |
| `drop()` | Új Indexet hoz létre a megadott érték(ek) törlésével. |
| `insert()` | Új Indexet hoz létre egy új elem beszúrásával az `i`-edik pozícióra. |
| `is_monotonic` | Igaz (`True`), ha minden elem nagyobb vagy egyenlő az előzőnél (monoton növekvő). |
| `is_unique` | Igaz (`True`), ha az Indexben nincs semmilyen duplikált érték. |
| `unique()` | Kiszámítja és visszaadja az Index egyedi (duplikációmentes) értékeinek tömbjét. |

## 5.2 Alapvető funkciók (Essential Functionality)

Ebben a részben a Series és DataFrame adatokkal való alapvető interakciók mechanikáját nézzük meg.

### Újraindexelés (Reindexing)

A `reindex` egy nagyon fontos metódus a pandasban. Azt jelenti, hogy létrehozunk egy új objektumot, amiben az adatok átrendeződnek úgy, hogy illeszkedjenek egy újonnan megadott indexhez.

**Példa egy Series-en:**

```python
In [98]: obj = pd.Series([4.5, 7.2, -5.3, 3.6], index=["d", "b", "a", "c"])
In [100]: obj2 = obj.reindex(["a", "b", "c", "d", "e"])
In [101]: obj2
Out[101]: 
a   -5.3
b    7.2
c    3.6
d    4.5
e    NaN
dtype: float64
```

**Magyarázat:** Az eredeti `obj` indexeit átrendeztük abc sorrendbe. Mivel az "e" címkéhez nem tartozott eredetileg adat, a pandas okosan berakott a helyére egy `NaN` (hiányzó) értéket.

**Előre-kitöltés (Forward-filling):**
Idősoros adatoknál gyakori, hogy egy új index bevezetésénél nem `NaN`-t akarunk, hanem azt, hogy a legutolsó ismert érték "folyjon tovább" előre. Erre való a `method="ffill"`.

```python
In [102]: obj3 = pd.Series(["blue", "purple", "yellow"], index=[0, 2, 4])
In [104]: obj3.reindex(np.arange(6), method="ffill")
Out[104]: 
0      blue
1      blue    # <- Előre kitöltötte az előző (0) értékkel
2    purple
3    purple    # <- Előre kitöltötte az előző (2) értékkel
4    yellow
5    yellow    # <- Előre kitöltötte az előző (4) értékkel
dtype: object
```

**Újraindexelés DataFrame-nél (Sorok és Oszlopok):**
A DataFrame-nél mind a sorok, mind az oszlopok újraindexelhetők.
Ha csak egy listát adsz át, akkor a sorokat rendezi át:

```python
In [105]: frame = pd.DataFrame(np.arange(9).reshape((3, 3)),
   .....:                      index=["a", "c", "d"],
   .....:                      columns=["Ohio", "Texas", "California"])
In [107]: frame2 = frame.reindex(index=["a", "b", "c", "d"])
```

Az oszlopokat a `columns` kulcsszóval lehet újraindexelni (sorrendet cserélni, újat hozzáadni vagy eldobni):

```python
In [109]: states = ["Texas", "Utah", "California"]
In [110]: frame.reindex(columns=states)
Out[110]: 
   Texas  Utah  California
a      1   NaN           2
c      4   NaN           5
d      7   NaN           8
```

**Magyarázat:** Csak a "Texas", "Utah" és "California" oszlopokat tartottuk meg. Mivel az eredetiben "Utah" nem volt, `NaN` lett. Az "Ohio" oszlop pedig kimaradt, ezért teljesen eltűnt az eredményből.

Ugyanezt meg lehet csinálni az `axis="columns"` paraméter megadásával is:

```python
In [111]: frame.reindex(states, axis="columns")
```

> **Gyakorlati tipp:** Sok esetben a `reindex` helyett egyszerűbb a `.loc` használatával "kivágni" az új sorrendet, de az csak akkor működik, ha az összes megadott címke **MÁR LÉTEZIK** a táblában. Ha új címkét adsz meg a `.loc`-ban, az hibát fog dobni, míg a `reindex` szépen berak egy `NaN`-t.

```python
In [112]: frame.loc[["a", "d", "c"], ["California", "Texas"]] # Újrarendezés loc-kal
```

#### Table 5-3. A reindex függvény argumentumai

| Argumentum | Leírás (Description) |
| :--- | :--- |
| `labels` | Az új szekvencia, amit indexként akarunk használni. Lehet Index példány vagy lista. |
| `index` | Ezt a szekvenciát használja új sor-indexként (címkeként). |
| `columns` | Ezt a szekvenciát használja új oszlop-címkeként. |
| `axis` | A tengely, amit újra akarunk indexelni: `"index"` (sorok) vagy `"columns"` (oszlopok). |
| `method` | Interpolációs (kitöltési) módszer. Az `"ffill"` (forward fill) előre, a `"bfill"` (backfill) hátrafelé tölti ki a hiányzó adatokat. |
| `fill_value` | Helyettesítő érték, amit a `NaN` helyett akarunk berakni az új címkékhez (alapértelmezetten marad a `NaN`). |
| `limit` | Ha `ffill` vagy `bfill` módszert használunk, megadja a maximálisan kitölthető hiányzó elemek számát. |
| `tolerance` | A maximális távolság, amíg egy pontatlan (inexact) egyezést elfogadunk az interpolációnál. |
| `level` | MultiIndex (többszintű index) esetén megadja, hogy melyik szinten végezzük az újraindexelést. |
| `copy` | Ha `True` (alapértelmezett), mindenképp másolja az adatokat a memóriában. Ha `False`, nem másol, amennyiben a régi és új index megegyezik. |

### Elemek eldobása egy tengelyről (Dropping Entries)

Ha ki akarsz törölni egy sort vagy oszlopot, a `.drop()` metódus a legkényelmesebb eszköz.

> **Fontos:** a `.drop()` mindig egy új objektumot ad vissza, tehát nem módosítja az eredeti adatot, hacsak nem mented rá, vagy nem használod az `inplace=True` paramétert!

**Series esetén:**

```python
In [113]: obj = pd.Series(np.arange(5.), index=["a", "b", "c", "d", "e"])
In [115]: new_obj = obj.drop("c")       # Egyetlen elem eldobása
In [117]: obj.drop(["d", "c"])          # Több elem eldobása (listaként átadva)
Out[117]: 
a    0.0
b    1.0
e    4.0
dtype: float64
```

**DataFrame esetén:**
Létrehozunk egy 4x4-es próba-táblázatot amerikai államokkal és számokkal:

```python
In [118]: data = pd.DataFrame(np.arange(16).reshape((4, 4)),
   .....:                     index=["Ohio", "Colorado", "Utah", "New York"],
   .....:                     columns=["one", "two", "three", "four"])
```

**Sorok eldobása (Axis 0 - alapértelmezett):**
Ha csak egy listát adsz a `drop()`-nak (vagy az `index=` paramétert használod), akkor a sorokat törli.

```python
In [120]: data.drop(index=["Colorado", "Ohio"])
Out[120]: 
          one  two  three  four
Utah        8    9     10    11
New York   12   13     14    15
```

**Oszlopok eldobása (Axis 1):**
Ha oszlopokat akarsz törölni, használnod kell a `columns=` paramétert, **VAGY** meg kell mondanod a gépnek, hogy az 1-es tengelyen (oszlopokon) keresse a címkét az `axis=1` vagy `axis="columns"` segítségével.

```python
# Ezek teljesen egyenértékűek:
In [121]: data.drop(columns=["two"])
In [122]: data.drop("two", axis=1)

In [123]: data.drop(["two", "four"], axis="columns")
Out[123]: 
          one  three
Ohio        0      2
Colorado    4      6
Utah        8     10
New York   12     14
```

## Indexelés, Kiválasztás és Szűrés (Indexing, Selection, and Filtering)

### Series indexelés

A Series indexelése (`obj[...]`) nagyon hasonlóan működik a NumPy tömbök indexeléséhez, azzal a nagy különbséggel, hogy itt nemcsak sorszámokat (integereket), hanem az index címkéit (label) is használhatod.

```python
In [124]: obj = pd.Series(np.arange(4.), index=["a", "b", "c", "d"])
In [125]: obj
Out[125]: 
a    0.0
b    1.0
c    2.0
d    3.0
dtype: float64

# Választás címke alapján:
In [126]: obj["b"]
Out[126]: 1.0

# Választás sorszám (pozíció) alapján:
In [127]: obj[1]
Out[127]: 1.0

# Szeletelés (Slicing) pozíciók alapján:
In [128]: obj[2:4]
Out[128]: 
c    2.0
d    3.0
dtype: float64

# Több elem kiválasztása egy listával:
In [129]: obj[["b", "a", "d"]]
Out[129]: 
b    1.0
a    0.0
d    3.0
dtype: float64

In [130]: obj[[1, 3]]
Out[130]: 
b    1.0
d    3.0
dtype: float64

# Logikai szűrés (Boolean filtering):
In [131]: obj[obj < 2]
Out[131]: 
a    0.0
b    1.0
dtype: float64
```

Bár a fenti `[]` módszer működik, a pandas-ban a **javasolt és legbiztonságosabb** módja az értékek lekérésének a speciális `.loc` és `.iloc` operátorok használata.

**Miért?** Mert ha a Series indexed alapból számokat tartalmaz, a sima `[]` operátor megzavarodhat, hogy most a címkére gondolsz, vagy a pozícióra.

- **`.loc`:** Kizárólag címke (label) alapján keres.
- **`.iloc`:** Kizárólag pozíció (integer sorszám) alapján keres.

```python
In [132]: obj.loc[["b", "a", "d"]]
Out[132]: 
b    1.0
a    0.0
d    3.0
dtype: float64
```

**Szeletelés címkékkel (`loc`): FONTOS KÜLÖNBSÉG!**
A normál Python szeleteléstől (ahol az utolsó elem már nem kerül bele az eredménybe) eltérően, a `.loc` segítségével történő címke-alapú szeletelésnél a végpont **is beletartozik** az eredménybe (inclusive)!

```python
In [141]: obj2.loc["b":"c"]
Out[141]: 
b    2
c    3
dtype: int64

# Természetesen értéket is adhatunk egy ilyen szeletnek:
In [142]: obj2.loc["b":"c"] = 5
```

> **Tipp kezdőknek:** A `loc` és `iloc` nem metódusok, hanem attribútumok, ezért szögletes zárójelet `[]` kell utánuk tenni, nem kerek zárójelet `()`!

### DataFrame indexelése (Sima szögletes zárójellel `[]`)

Ha egy DataFrame neve után sima szögletes zárójelet teszel, azzal alapvetően **oszlopokat** tudsz lekérdezni:

```python
In [144]: data = pd.DataFrame(np.arange(16).reshape((4, 4)),
   .....:                     index=["Ohio", "Colorado", "Utah", "New York"],
   .....:                     columns=["one", "two", "three", "four"])
In [146]: data["two"]
Out[146]: 
Ohio        1
Colorado    5
Utah        9
New York   13
Name: two, dtype: int64

In [147]: data[["three", "one"]]
Out[147]: 
          three  one
Ohio          2    0
Colorado      6    4
Utah         10    8
New York     14   12
```

Van azonban két speciális eset, amikor a sima `[]` nem oszlopot, hanem sorokat választ ki kényelmi okokból:

**1. Szeletelés (Slicing):**

```python
In [148]: data[:2]
Out[148]: 
          one  two  three  four
Ohio        0    1      2     3
Colorado    4    5      6     7
```

**2. Logikai szűrés (Boolean array):**

```python
In [149]: data[data["three"] > 5]
Out[149]: 
          one  two  three  four
Colorado    4    5      6     7
Utah        8    9     10    11
New York   12   13     14    15
```

**Értékek tömeges módosítása logikai DataFrame alapján:**

```python
# Hol kisebb az adat 5-nél? True/False mátrixot ad.
In [150]: data < 5

# Ahol a feltétel igaz, ott felülírjuk 0-val:
In [151]: data[data < 5] = 0
```

### Kiválasztás DataFrame-ből `loc` és `iloc` segítségével

Mivel a DataFrame kétdimenziós, a `.loc` és `.iloc` használatával egyszerre választhatunk ki sorokat és oszlopokat vesszővel elválasztva: `data.loc[sorok, oszlopok]`.

#### 1. Kiválasztás címke alapján (`.loc`)

```python
# Egyetlen sor kiválasztása (Series-t ad vissza):
In [154]: data.loc["Colorado"]
Out[154]: 
one      0
two      5
three    6
four     7
Name: Colorado, dtype: int64

# Több sor kiválasztása:
In [155]: data.loc[["Colorado", "New York"]]

# Sorok ÉS oszlopok együttes kiválasztása:
In [156]: data.loc["Colorado", ["two", "three"]]
Out[156]: 
two      5
three    6
Name: Colorado, dtype: int64
```

#### 2. Kiválasztás pozíció/sorszám alapján (`.iloc`)

```python
# A 3. sor kiválasztása (indexelés 0-tól indul!):
In [157]: data.iloc[2]

# A 3. és 2. sor kiválasztása ebben a sorrendben:
In [158]: data.iloc[[2, 1]]

# Konkrét sor és konkrét oszlopok kiválasztása számok alapján:
In [159]: data.iloc[2, [3, 0, 1]]
Out[159]: 
four    11
one      8
two      9
Name: Utah, dtype: int64
```

**Szeletelés (Slicing) `loc` és `iloc` használatával:**
Mindkettő működik szeletekkel is. **Figyelem:** A `.loc` a címkénél belevonja a végpontot is!

```python
In [161]: data.loc[:"Utah", "two"]   # Az elejétől a "Utah" sorig, a "two" oszlopban
Out[161]: 
Ohio        0
Colorado    5
Utah        9
Name: two, dtype: int64

In [162]: data.iloc[:, :3][data.three > 5] # Minden sor, az első 3 oszlop, és egy logikai szűrő ráhúzva
```

### Table 5-4. DataFrame indexelési opciók

| Típus (Type) | Megjegyzések (Notes) |
| :--- | :--- |
| `df[column]` | Egy vagy több oszlop kiválasztása a DataFrame-ből. Kényelmi kivételek: Ha logikai tömböt kap, a sorokat szűri. Ha szeletet kap (pl. `[:2]`), a sorokat szeleteli. |
| `df.loc[rows]` | Egyetlen sor vagy sorok egy halmazának kiválasztása címke (label) alapján. |
| `df.loc[:, cols]` | Oszlop vagy oszlopok kiválasztása címke alapján. |
| `df.loc[rows, cols]` | Sor(ok) és oszlop(ok) együttes kiválasztása címkék alapján. |
| `df.iloc[rows]` | Sor vagy sorok kiválasztása pozíció (integer) alapján. |
| `df.iloc[:, cols]` | Oszlop vagy oszlopok kiválasztása pozíció alapján. |
| `df.iloc[rows, cols]` | Sor(ok) és oszlop(ok) együttes kiválasztása pozíció alapján. |
| `df.at[row, col]` | Egy egyetlen konkrét cella (skalár) kiválasztása címke alapján (gyorsabb, mint a `loc`). |
| `df.iat[row, col]` | Egy egyetlen konkrét cella kiválasztása pozíció alapján (gyorsabb, mint az `iloc`). |
| `reindex` metódus | Sorok vagy oszlopok újrarendezése új címkék megadásával. |

### Az egész-számos indexelés csapdái (Integer indexing pitfalls)

Ha egy Series vagy DataFrame indexe számokból áll (pl. 0, 1, 2), a kezdők gyakran belefutnak egy hibába. Nézzük meg a `ser[-1]` esetét! A normál Python listáknál ez a legutolsó elemet jelenti. De nézzük mi történik a pandas-ban:

```python
In [164]: ser = pd.Series(np.arange(3.))
In [165]: ser
Out[165]: 
0    0.0
1    1.0
2    2.0
dtype: float64

In [166]: ser[-1]
# KeyError: -1  (HIBÁT DOB!)
```

**Magyarázat:** Mivel a `ser` indexe számokból (0, 1, 2) áll, a pandas megpróbálja megkeresni azt a címkét, aminek a neve -1. Ilyen címke nincs, ezért hibát dob. Nem hajlandó megtippelni, hogy te most pozícióra gondoltál-e.

Ezzel szemben, ha az index nem számokból áll (pl. betűkből), akkor a pandas tudja, hogy a -1 csakis pozíció lehet, és visszaadja a legutolsó elemet:

```python
In [168]: ser2 = pd.Series(np.arange(3.), index=["a", "b", "c"])
In [169]: ser2[-1]
Out[169]: 2.0
```

**A megoldás:** Ennek az ambivalenciának az elkerülése végett mindig használd az `.iloc` operátort, ha pozíció alapján akarsz keresni!

```python
In [170]: ser.iloc[-1]
Out[170]: 2.0
```

### A láncolt indexelés csapdája (SettingWithCopyWarning)

A `.loc` és `.iloc` használható értékek megváltoztatására (assignment) is a DataFrame-en belül. De nagyon vigyázni kell arra, hogyan írjuk le a kódot!

Ezek a helyes, helyben történő módosítások:

```python
In [172]: data.loc[:, "one"] = 1                # A "one" oszlop minden értéke 1 lesz
In [174]: data.iloc[2] = 5                      # A 3. sor minden értéke 5 lesz
In [176]: data.loc[data["four"] > 5] = 3        # Logikai szűrés alapján módosítás
```

**A gyakori kezdő hiba (Láncolt indexelés / Chained indexing):**
Tegyük fel, hogy ahol a "three" oszlop értéke 5, ott a "three" oszlop értékét át akarjuk írni 6-ra.
Sokan így írják le (két egymás utáni szögletes zárójellel):

```python
In [177]: data.loc[data.three == 5]["three"] = 6
# SettingWithCopyWarning: 
# A value is trying to be set on a copy of a slice from a DataFrame.
# Try using .loc[row_indexer,col_indexer] = value instead
```

**Magyarázat:** Ez a híres **SettingWithCopyWarning**. A probléma az, hogy a `data.loc[data.three == 5]` egy köztes, átmeneti DataFrame másolatot (vagy nézetet) hozott létre a memóriában, és te ennek a másolatnak a "three" oszlopát írtad át 6-ra, az eredeti `data` táblázat pedig érintetlen (módosítatlan) maradt!

**A helyes megoldás:**
Kerüld el a láncolt indexelést! Ha módosítani akarsz, azt egy **egyetlen** `.loc` híváson belül kell megtenned, ahol megadod a sort és az oszlopot is, vesszővel elválasztva:

```python
In [180]: data.loc[data.three == 5, "three"] = 6
```

Így a pandas pontosan tudja, hogy az eredeti DataFrame melyik celláiba kell beleírnia a 6-ost.

## Matematikai műveletek és adatigazítás (Arithmetic and Data Alignment)

**Fő koncepció:** A pandas egyik legokosabb (és legkényelmesebb) funkciója, hogy ha különböző indexszel (címkékkel) rendelkező objektumokat adsz össze, ő megpróbálja automatikusan összeilleszteni (align) őket.

Ha bármelyik index-pár nem egyezik meg pontosan, az eredmény indexe a két index **uniója** (összessége) lesz.

**Példa két Series összeadására:**

```python
In [182]: s1 = pd.Series([7.3, -2.5, 3.4, 1.5], index=["a", "c", "d", "e"])
In [183]: s2 = pd.Series([-2.1, 3.6, -1.5, 4, 3.1],
   .....:                index=["a", "c", "e", "f", "g"])

In [186]: s1 + s2
Out[186]: 
a    5.2
c    1.1
d    NaN
e    0.0
f    NaN
g    NaN
dtype: float64
```

**Magyarázat:**
- Az "a", "c" és "e" címkék mindkét sorozatban szerepeltek, így a pandas simán összeadta a hozzájuk tartozó értékeket (pl. "a": `7.3 + (-2.1) = 5.2`).
- A "d" csak az `s1`-ben volt benne, az "f" és a "g" pedig csak az `s2`-ben. Ezeknél a pandas nem tudott mit mihez adni, ezért bevezetett egy hiányzó értéket (`NaN`). Ezek a hiányzó értékek a további matematikai műveletek során is tovább fognak terjedni.

**Példa DataFrame-ek összeadására:**
A DataFrame-eknél az illesztés mind a sorokon, mind az oszlopokon megtörténik!

```python
In [187]: df1 = pd.DataFrame(np.arange(9.).reshape((3, 3)), columns=list("bcd"),
   .....:                    index=["Ohio", "Texas", "Colorado"])
In [188]: df2 = pd.DataFrame(np.arange(12.).reshape((4, 3)), columns=list("bde"),
   .....:                    index=["Utah", "Ohio", "Texas", "Oregon"])

In [191]: df1 + df2
Out[191]: 
            b   c     d   e
Colorado  NaN NaN   NaN NaN
Ohio      3.0 NaN   6.0 NaN
Oregon    NaN NaN   NaN NaN
Texas     9.0 NaN  12.0 NaN
Utah      NaN NaN   NaN NaN
```

**Magyarázat:** A gép létrehozott egy új táblázatot, aminek a sorai és az oszlopai a két eredeti tábla uniójából állnak. Ha összeadsz két olyan táblát, aminek semmi közös nincs a címkéiben, akkor a végeredmény egy csupa `NaN`-ból álló táblázat lesz.

### Matematikai metódusok kitöltő értékekkel (fill values)

Nagyon gyakori, hogy nem akarjuk, hogy a hiányzó illesztésekből `NaN` legyen. Ilyenkor a `+` operátor helyett a `.add()` metódust használjuk, amihez megadhatunk egy kitöltő értéket (`fill_value`).

```python
# Létrehozunk két táblát, és direkt berakunk az egyikbe egy NaN-t:
In [197]: df1 = pd.DataFrame(np.arange(12.).reshape((3, 4)), columns=list("abcd"))
In [198]: df2 = pd.DataFrame(np.arange(20.).reshape((4, 5)), columns=list("abcde"))
In [199]: df2.loc[1, "b"] = np.nan

# Sima összeadás (tele lesz NaN-nal):
In [202]: df1 + df2

# Összeadás kitöltő értékkel (0-val helyettesíti a hiányzókat):
In [203]: df1.add(df2, fill_value=0)
Out[203]: 
      a     b     c     d     e
0   0.0   2.0   4.0   6.0   4.0
1   9.0   5.0  13.0  15.0   9.0
2  18.0  20.0  22.0  24.0  14.0
3  15.0  16.0  17.0  18.0  19.0
```

### Table 5-5. Rugalmas matematikai metódusok

Ezeknek a metódusoknak mind létezik egy `r` betűvel kezdődő párja (pl. `radd`), ami megfordítja az argumentumok sorrendjét (például az `1 / df1` egyenértékű a `df1.rdiv(1)`-gyel).

| Metódus | Leírás (Description) |
| :--- | :--- |
| `add`, `radd` | Összeadás (`+`) |
| `sub`, `rsub` | Kivonás (`-`) |
| `div`, `rdiv` | Osztás (`/`) |
| `floordiv`, `rfloordiv` | Egész osztás (maradék eldobása) (`//`) |
| `mul`, `rmul` | Szorzás (`*`) |
| `pow`, `rpow` | Hatványozás (`**`) |

(Ezeket a kitöltő értékeket a `reindex` metódusnál is ugyanígy lehet használni a `fill_value=0` argumentummal).

### Műveletek DataFrame és Series között (Broadcasting)

Ahogy a NumPy-ban egy 2D-s mátrixból kivonhatunk egy 1D-s vektort (és az kiterjed minden sorra), úgy ez a pandas-ban is lehetséges.

```python
In [211]: frame = pd.DataFrame(np.arange(12.).reshape((4, 3)),
   .....:                      columns=list("bde"),
   .....:                      index=["Utah", "Ohio", "Texas", "Oregon"])
In [212]: series = frame.iloc[0]  # Kivesszük az első sort egy Series-be

In [215]: frame - series
Out[215]: 
          b    d    e
Utah    0.0  0.0  0.0
Ohio    3.0  3.0  3.0
Texas   6.0  6.0  6.0
Oregon  9.0  9.0  9.0
```

**Magyarázat:** Alapértelmezés szerint a pandas a Series indexét (címkéit) a DataFrame oszlopaihoz (columns) igazítja, majd a műveletet soronként lefelé hajtja végre mindenkin. Tehát minden egyes sorból kivonta a "Utah" sor értékeit.

Mi van, ha nem soronként, hanem oszloponként akarjuk elvégezni a műveletet? Például kivonni egy adott oszlopot a táblázat összes oszlopából?
Ezt a sima `-` operátorral nem lehet, a `.sub()` metódust kell használni az `axis="index"` (vagy `axis=0`) megadásával:

```python
In [219]: series3 = frame["d"]    # Kivesszük a 'd' oszlopot

In [222]: frame.sub(series3, axis="index")
Out[222]: 
          b    d    e
Utah   -1.0  0.0  1.0
Ohio   -1.0  0.0  1.0
Texas  -1.0  0.0  1.0
Oregon -1.0  0.0  1.0
```

**Magyarázat:** Itt megmondtuk a gépnek, hogy az igazítást a sor-indexeken (`axis="index"`) végezze el, és a kivonást oszlopokon keresztül hajtsa végre.

## Függvények alkalmazása és leképezése (Function Application and Mapping)

### 1. NumPy ufunc-ok használata

A pandas objektumokon közvetlenül is működnek a NumPy univerzális függvényei (mint pl. az abszolút érték).

```python
In [224]: frame
# ... random számok (negatívok is)

In [225]: np.abs(frame)
# ... ugyanaz a tábla, de minden szám pozitívra váltott
```

### 2. Saját függvény alkalmazása a sorokon vagy oszlopokon (`.apply`)

Nagyon gyakori, hogy írunk egy saját függvényt, amit rá akarunk dobni a táblázat minden egyes oszlopára vagy sorára. Erre való az `apply` metódus.

```python
# Írunk egy függvényt, ami kiszámolja a különbséget a max és min érték között
In [226]: def f1(x):
   .....:     return x.max() - x.min()

# Ráhúzzuk a DataFrame oszlopaira (alapértelmezés)
In [227]: frame.apply(f1)
Out[227]: 
b    1.802165
d    1.684034
e    2.689627
dtype: float64
```

Ha oszlopok helyett soronként akarjuk alkalmazni, átadjuk az `axis="columns"` argumentumot (amit úgy érdemes értelmezni, hogy "alkalmazd az oszlopokon keresztül"):

```python
In [228]: frame.apply(f1, axis="columns")
```

> **Megjegyzés:** Az `apply` akkor is működik, ha a függvényed nem egyetlen számot, hanem egy komplett új Series-t ad vissza. Ilyenkor az eredmény egy DataFrame lesz.

### 3. Saját függvény alkalmazása ELEMENKÉNT (`.applymap` és `.map`)

Ha nem oszlopokra, hanem a táblázat minden egyes cellájára (elemére) külön-külön akarsz egy függvényt alkalmazni (például megformázni a számokat 2 tizedesjegyes szövegként):

- DataFrame esetén: `.applymap()`
- Series esetén: `.map()`

```python
In [231]: def my_format(x):
   .....:     return f"{x:.2f}"

In [232]: frame.applymap(my_format)
Out[232]: 
            b     d      e
Utah    -0.20  0.48  -0.52
Ohio    -0.56  1.97   1.39
Texas    0.09  0.28   0.77
Oregon   1.25  1.01  -1.30

# Ugyanez csak egyetlen oszlopra (ami egy Series):
In [233]: frame["e"].map(my_format)
```

## Rendezés és Rangsorolás (Sorting and Ranking)

### Rendezés index/címke alapján (`sort_index`)

Létrehoz egy új objektumot, ami lexiszikografikusan (ABC vagy növekvő sorrendbe) rendezi a sor- vagy oszlopcímkéket.

```python
In [234]: obj = pd.Series(np.arange(4), index=["d", "a", "b", "c"])
In [236]: obj.sort_index()
Out[236]: 
a    1
b    2
c    3
d    0
dtype: int64
```

DataFrame esetén megadhatod, hogy az oszlop-neveket (`axis="columns"`) akarod-e betűrendbe tenni, és hogy csökkenő sorrendet kérsz-e (`ascending=False`):

```python
In [241]: frame.sort_index(axis="columns", ascending=False)
```

### Rendezés értékek alapján (`sort_values`)

Adatelemzésnél leginkább maguk az adatok (az értékek) alapján akarunk rendezni (pl. "Rendezd növekvőbe az árakat").

**Series esetén:**

```python
In [242]: obj = pd.Series([4, 7, -3, 2])
In [243]: obj.sort_values()
Out[243]: 
2   -3
3    2
0    4
1    7
dtype: int64
```

> **Fontos:** Ha hiányzó adat (`NaN`) van a sorozatban, az alapértelmezetten mindig a legvégére kerül. Ha az elejére akarod tenni, használd az `na_position="first"` opciót.

**DataFrame esetén:**
Meg kell mondanod a `sort_values`-nak, hogy melyik oszlop(ok) adatai alapján rendezzen.

```python
In [247]: frame = pd.DataFrame({"b": [4, 7, -3, 2], "a": [0, 1, 0, 1]})

# Egyetlen oszlop alapján rendezés:
In [249]: frame.sort_values("b")

# Több oszlop alapján rendezés (Ha az "a" oszlopban döntetlen van, a "b" oszlop alapján dönt):
In [250]: frame.sort_values(["a", "b"])
Out[250]: 
   b  a
2 -3  0
0  4  0
3  2  1
1  7  1
```

### Rangsorolás (`rank`)

A rangsorolás megadja, hogy egy adat hányadik lenne a sorrendben (1-től kezdve), ha sorba lennének állítva.
**Probléma:** mi van, ha két elem értéke megegyezik (holtverseny / tie)? A pandas alapértelmezetten a holtversenyben lévő elemek átlagos rangját adja nekik.

```python
In [251]: obj = pd.Series([7, -5, 7, 4, 2, 0, 4])
In [252]: obj.rank()
Out[252]: 
0    6.5   # A két 7-es a 6. és 7. helyen állna, ezért megkapják az átlagot (6.5)
1    1.0   # A -5 a legkisebb, így övé az 1. hely
2    6.5
...
dtype: float64
```

Ha nem akarunk tizedestörteket (átlagot), használhatjuk a `method="first"` szabályt. Ez azt mondja: aki a táblázatban hamarabb jelent meg, az kapja a jobb (kisebb sorszámú) helyezést!

```python
In [253]: obj.rank(method="first")
```

DataFrame-nél a rangsorolást elvégezhetjük sorokon lefelé (alapértelmezett), vagy oszlopokon keresztül is a `rank(axis="columns")` segítségével.

### Table 5-6. Holtverseny-feloldó (Tie-breaking) módszerek a rank függvénynél

| Módszer (`method=...`) | Leírás (Description) |
| :--- | :--- |
| `"average"` | Alapértelmezett: Az egyenlő csoportba eső elemek megkapják az érintett helyezések számtani átlagát. |
| `"min"` | A csoport minden tagja megkapja az elérhető legkisebb (legjobb) helyezést. |
| `"max"` | A csoport minden tagja megkapja az elérhető legnagyobb (legrosszabb) helyezést. |
| `"first"` | Olyan sorrendben osztja ki a helyezéseket, amilyen sorrendben a gép megtalálta őket az adatok között. |
| `"dense"` | Hasonló a `"min"`-hez, de a rangok mindig szigorúan 1-gyel nőnek a csoportok között, nem ugorja át a felhasznált helyezéseket. |

## Tengely indexek duplikált címkékkel (Axis Indexes with Duplicate Labels)

Bár sok pandas függvény (mint a `reindex`) megköveteli, hogy a címkék egyediek legyenek, ez nem kötelező érvényű szabály a pandas-ban. Létrehozhatunk olyan Series-t vagy DataFrame-et, ahol az index ismétlődő értékeket tartalmaz.

```python
In [258]: obj = pd.Series(np.arange(5), index=["a", "a", "b", "b", "c"])
In [259]: obj
Out[259]: 
a    0
a    1
b    2
b    3
c    4
dtype: int64

# Az is_unique tulajdonság megmondja, hogy az index egyedi-e:
In [260]: obj.index.is_unique
Out[260]: False
```

**Fontos különbség az adatlekérésnél:**
Ha egy olyan címkére hivatkozol, ami többször is szerepel, a pandas egy Series-t ad vissza. Ha a címke egyedi, akkor egy egyszerű skalár (szám) értéket kapsz. Emiatt a kódod bonyolultabbá válhat, ha nem vagy biztos benne, hogy az indexed egyedi-e.

```python
In [261]: obj["a"]
Out[261]: 
a    0
a    1
dtype: int64

In [262]: obj["c"]
Out[262]: 4
```

(Ugyanez a logika érvényes a DataFrame-ek sorainak `loc` segítségével történő lekérdezésére is.)

## 5.3 Leíró statisztikák összegzése és kiszámítása (Summarizing and Computing Descriptive Statistics)

A pandas objektumok fel vannak szerelve egy sor beépített matematikai és statisztikai metódussal. Ezek többsége úgynevezett **redukciós vagy összegző statisztika**, ami egyetlen értékké (pl. összeg, átlag) sűríti egy Series adatait, vagy egy DataFrame sorait/oszlopait. Hatalmas előnyük a NumPy-jal szemben, hogy automatikusan kezelik a hiányzó adatokat (`NaN`).

```python
In [267]: df = pd.DataFrame([[1.4, np.nan], [7.1, -4.5],
   .....:                    [np.nan, np.nan], [0.75, -1.3]],
   .....:                   index=["a", "b", "c", "d"],
   .....:                   columns=["one", "two"])
In [268]: df
Out[268]: 
    one  two
a  1.40  NaN
b  7.10 -4.5
c   NaN  NaN
d  0.75 -1.3
```

A `.sum()` metódus alapértelmezetten oszloponként (lefelé) adja össze az értékeket, figyelmen kívül hagyva a `NaN`-okat.

```python
In [269]: df.sum()
Out[269]: 
one    9.25
two   -5.80
dtype: float64
```

Ha soronként (keresztbe) akarjuk összeadni, használjuk az `axis="columns"` vagy `axis=1` argumentumot:

```python
In [270]: df.sum(axis="columns")
Out[270]: 
a    1.40
b    2.60
c    0.00
d   -0.55
dtype: float64
```

**Magyarázat:** A "c" sorban mindkét adat `NaN` volt, így az összeg 0 lett. Ahol legalább egy szám volt, ott a `NaN` értéket 0-ként kezelte a gép.

Ha azt akarod, hogy a hiányzó értékek ne legyenek átugorva (tehát ha egy sorban/oszlopban van `NaN`, akkor az eredmény is legyen `NaN`), használd a `skipna=False` opciót:

```python
In [272]: df.sum(axis="columns", skipna=False)
Out[272]: 
a     NaN
b    2.60
c     NaN
d   -0.55
dtype: float64
```

Bizonyos függvényeknél (mint az átlag: `.mean()`) legalább egy érvényes adat kell a számításhoz, különben az eredmény `NaN` marad:

```python
In [273]: df.mean(axis="columns")
Out[273]: 
a    1.400
b    1.300
c      NaN
d   -0.275
dtype: float64
```

### Table 5-7. Opciók a redukciós (összegző) metódusokhoz

| Opció | Leírás (Description) |
| :--- | :--- |
| `axis` | A tengely, ami mentén a redukciót végezzük. `"index"` a DataFrame sorain (lefelé), `"columns"` az oszlopain (keresztbe) történő számításhoz. |
| `skipna` | Kizárja (átugorja) a hiányzó értékeket. Alapértelmezetten `True`. |
| `level` | Ha hierarchikus indexünk van (MultiIndex), ezen a szinten végzi el a csoportosított redukciót. |

Vannak indirekt statisztikák is, amik nem magát az értéket, hanem annak indexét (címkéjét) adják vissza. Ilyen az `idxmin` és `idxmax`:

```python
In [274]: df.idxmax()
Out[274]: 
one    b
two    d
dtype: object
```

**Magyarázat:** A "one" oszlopban a legnagyobb érték a "b" sorban van, a "two" oszlopban pedig a "d" sorban.

A `.describe()` egy zseniális metódus, ami egyetlen lépésben kiszámít egy sor statisztikai alapértéket (darabszám, átlag, szórás, minimum, maximum, kvartilisek):

```python
In [276]: df.describe()
Out[276]: 
            one       two
count  3.000000  2.000000
mean   3.083333 -2.900000
std    3.493685  2.262742
min    0.750000 -4.500000
25%    1.075000 -3.700000
50%    1.400000 -2.900000
75%    4.250000 -2.100000
max    7.100000 -1.300000
```

(Ha a táblázat nem számokat, hanem szövegeket tartalmaz, a `.describe()` teljesen más adatokat ad: az elemek számát, az egyedi elemek számát, a leggyakoribb elemet és annak előfordulási gyakoriságát).

### Table 5-8. Leíró és összegző statisztikák

| Metódus | Leírás (Description) |
| :--- | :--- |
| `count` | A nem-NA (létező) értékek száma. |
| `describe` | Kiszámít egy teljes leíró statisztikai összefoglalót. |
| `min`, `max` | Minimum és maximum értékek. |
| `argmin`, `argmax` | A min. és max. értékek pozícióját (számát) adja meg (DataFrame-en nem elérhető). |
| `idxmin`, `idxmax` | A min. és max. értékek címkéjét (index nevét) adja meg. |
| `quantile` | Kiszámítja a megadott (0-tól 1-ig terjedő) kvantilist (alapból 0.5, ami a medián). |
| `sum` | Az értékek összege. |
| `mean` | Az értékek számtani átlaga. |
| `median` | Az értékek mediánja (az 50. percentilis). |
| `mad` | Az átlagtól való átlagos abszolút eltérés (Mean absolute deviation). |
| `prod` | Az értékek szorzata. |
| `var` | A minta varianciája (szórásnégyzete). |
| `std` | A minta szórása (Standard deviation). |
| `skew` | A minta ferdesége (harmadik momentum). |
| `kurt` | A minta csúcsossága / lapultsága (kurtosis - negyedik momentum). |
| `cumsum` | Kumulatív (halmozott) összeg. |
| `cummin`, `cummax` | Kumulatív minimum vagy maximum érték. |
| `cumprod` | Kumulatív szorzat. |
| `diff` | Az első aritmetikai különbség kiszámítása (nagyon hasznos idősoroknál, pl. előző naphoz képesti változás). |
| `pct_change` | Százalékos változás kiszámítása. |

## Korreláció és Kovariancia (Correlation and Covariance)

Egyes statisztikai mutatók oszloppárokból számítódnak. Tegyük fel, van két DataFrame-ünk részvényárakkal és kereskedési volumenekkel.

A `pct_change()` kiszámítja a napi százalékos változást:

```python
In [281]: returns = price.pct_change()
```

A Series `.corr()` és `.cov()` metódusai kiszámítják két adatsor (például a Microsoft és az IBM hozamai) közötti korrelációt és kovarianciát:

```python
In [283]: returns["MSFT"].corr(returns["IBM"])
Out[283]: 0.49976361144151144

In [284]: returns["MSFT"].cov(returns["IBM"])
Out[284]: 8.870655479703546e-05
```

Ha ugyanezt a DataFrame-en hívod meg paraméterek nélkül, akkor kiszámolja az összes oszlop egymáshoz viszonyított (páronkénti) korrelációs vagy kovariancia mátrixát:

```python
In [286]: returns.corr()
Out[286]: 
          AAPL      GOOG       IBM      MSFT
AAPL  1.000000  0.407919  0.386817  0.389695
GOOG  0.407919  1.000000  0.405099  0.465919
IBM   0.386817  0.405099  1.000000  0.499764
MSFT  0.389695  0.465919  0.499764  1.000000
```

**Magyarázat:** Látható, hogy minden részvény önmagával 1.0 (100%) korrelációt mutat.

A `.corrwith()` metódussal egy DataFrame összes oszlopát összehasonlíthatod egy külső Series-zel (pl. korrelál-e a piac összes részvénye az IBM-mel), **VAGY** egy másik DataFrame-mel (pl. az árak változásának korrelációja a volumennel oszlopról oszlopra):

```python
In [289]: returns.corrwith(volume)
Out[289]: 
AAPL   -0.075565
GOOG   -0.007067
IBM    -0.204849
MSFT   -0.092950
dtype: float64
```

## Egyedi értékek, Gyakoriságok és Tagság (Unique Values, Value Counts, and Membership)

Ezek a metódusok egydimenziós Series-eken működnek, és információt adnak a bennük lévő értékek halmazáról.

### 1. unique

Visszaadja a Series-ben található összes egyedi értéket (duplikációk nélkül) egy tömbben. (Nem rendezi őket sorba!)

```python
In [290]: obj = pd.Series(["c", "a", "d", "a", "a", "b", "b", "c", "c"])
In [291]: uniques = obj.unique()
In [292]: uniques
Out[292]: array(['c', 'a', 'd', 'b'], dtype=object)
```

### 2. value_counts

Kiszámítja a gyakoriságot, azaz megmondja, melyik érték hányszor szerepel. Az eredmény egy Series, ami kényelmi okokból a leggyakoribb elemtől csökkenő sorrendben van rendezve.

```python
In [293]: obj.value_counts()
Out[293]: 
c    3
a    3
b    2
d    1
dtype: int64
```

### 3. isin

Logikai (Boolean) ellenőrzés halmaz-tagságra. Igazat ad, ha az adott elem benne van a megadott listában. Tökéletes adatok szűrésére.

```python
In [296]: mask = obj.isin(["b", "c"])
In [297]: mask
Out[297]: 
0     True
1    False
...
dtype: bool

In [298]: obj[mask] # Leszűrjük a Series-t csak 'b' és 'c' értékekre
Out[298]: 
0    c
5    b
6    b
7    c
8    c
dtype: object
```

### Table 5-9. Egyediséget, darabszámot és halmaz-tagságot vizsgáló metódusok

| Metódus | Leírás (Description) |
| :--- | :--- |
| `isin` | Kiszámít egy logikai tömböt, ami megmondja, hogy a Series/DataFrame minden egyes értéke benne van-e az átadott listában/szekvenciában. |
| `get_indexer` | Kiszámítja az egyedi elemek tömbjéhez tartozó index-pozíciókat (hasznos adattisztításnál és join-szerű műveleteknél). |
| `unique` | Egyedi (nem ismétlődő) értékek tömbjét adja vissza, a megjelenés sorrendjében. |
| `value_counts` | Visszaad egy Series-t, ahol az index az egyedi érték, az érték pedig a darabszám (gyakoriság). A darabszám szerint csökkenő sorrendbe rendezve adja vissza. |

**Gyakoriság kiszámítása a teljes DataFrame-en:**
Ha van egy DataFrame-ed több oszloppal, és az egész táblázatra akarsz gyakoriságot (hisztogramot) számolni, a `pd.value_counts` függvényt adhatod át az `apply` metódusnak:

```python
In [303]: data = pd.DataFrame({"Qu1": [1, 3, 4, 3, 4],
   .....:                      "Qu2": [2, 3, 1, 2, 3],
   .....:                      "Qu3": [1, 5, 2, 4, 4]})

In [306]: result = data.apply(pd.value_counts).fillna(0)
In [307]: result
Out[307]: 
   Qu1  Qu2  Qu3
1  1.0  1.0  1.0
2  0.0  2.0  1.0
3  2.0  2.0  0.0
4  2.0  0.0  2.0
5  0.0  0.0  1.0
```

**Magyarázat:** A sorok címkéi (1, 2, 3, 4, 5) a táblázatban fellelhető összes lehetséges értéket jelentik. Az oszlopokban pedig látszik, hogy az adott érték hányszor fordult elő abban az oszlopban (a `fillna(0)` a hiányzó értékeket nullára cserélte).

Létezik egy dedikált `DataFrame.value_counts()` metódus is, ami nem cellánként, hanem teljes soronként (tuple-ként) számolja a gyakoriságot, azaz megmondja, hány darab teljesen megegyező sor van a táblázatban.

```python
In [308]: data = pd.DataFrame({"a": [1, 1, 1, 2, 2], "b": [0, 0, 1, 0, 0]})
In [310]: data.value_counts()
Out[310]: 
a  b
1  0    2
2  0    2
1  1    1
dtype: int64
```

**Magyarázat:** Az `(1, 0)` sor kétszer szerepel, a `(2, 0)` sor is kétszer, az `(1, 1)` sor pedig csak egyszer.
