# 4. Fejezet: NumPy alapok: Tömbök és Vektorizált számítások

## Bevezetés: Mi az a NumPy és miért ilyen gyors?

A **NumPy** (Numerical Python) a Python numerikus számításainak alapcsomagja. Bár önmagában nem végez bonyolult statisztikai elemzéseket (arra ott a pandas), az adatcserének és a gyors matematikai műveleteknek ez a szabványos (lingua franca) eszköze.

**Miért 10-100x gyorsabb a NumPy, mint a hagyományos Python listák?**

- **Memóriakezelés:** A NumPy az adatokat egyetlen folyamatos (összefüggő) memóriablokkban tárolja, függetlenül a beépített Python objektumoktól.
- **C-nyelvű motor:** A háttérben futó, C-ben írt algoritmusok anélkül tudnak ezen a memórián műveleteket végezni, hogy a Python lassú típusellenőrzésével (type checking) kéne bajlódniuk.
- **Vektorizáció:** A műveleteket a teljes tömbön egyszerre (kötegelve) hajtja végre, így nincs szükség lassú Python `for` ciklusokra.

### Teljesítmény-teszt (Sebesség összehasonlítása)

Nézzük meg egy egymillió elemből álló NumPy tömb és egy normál Python lista szorzásának sebességkülönbségét a Jupyter `%timeit` (időmérő) varázsparancsával:

```python
In [7]: import numpy as np
In [8]: my_arr = np.arange(1_000_000)
In [9]: my_list = list(range(1_000_000))

In [10]: %timeit my_arr2 = my_arr * 2
715 us +- 13.2 us per loop (mean +- std. dev. of 7 runs, 1000 loops each)

In [11]: %timeit my_list2 = [x * 2 for x in my_list]
48.8 ms +- 298 us per loop (mean +- std. dev. of 7 runs, 10 loops each)
```

**Magyarázat:** A NumPy (`my_arr * 2`) mikroszekundumok (us) alatt végez, míg a Python lista (`[x * 2 ...]`) milliszekundumok (ms) alatt. A NumPy itt nagyságrendekkel (kb. 70-szer) gyorsabb!

> **Jó gyakorlat:** A könyv mindig az `import numpy as np` konvenciót használja. Sose használd a `from numpy import *` parancsot, mert felülírhatja a beépített Python függvényeket, pl. a `max`-ot!

## 4.1 A NumPy ndarray: Egy többdimenziós tömb objektum

Az **ndarray** (N-dimensional array) egy gyors, rugalmas tároló, ami lehetővé teszi, hogy egész adatblokkokon végezz matematikai műveleteket pontosan olyan egyszerű szintaxissal, mintha csak két sima számot adnál össze.

**Alapvető tömbműveletek:**

```python
In [12]: import numpy as np
In [13]: data = np.array([[1.5, -0.1, 3], [0, -3, 6.5]])
In [14]: data
Out[14]: 
array([[ 1.5, -0.1,  3. ],
       [ 0. , -3. ,  6.5]])

In [15]: data * 10
Out[15]: 
array([[ 15.,  -1.,  30.],
       [  0., -30.,  65.]])

In [16]: data + data
Out[16]: 
array([[  3. ,  -0.2,   6. ],
       [  0. ,  -6. ,  13. ]])
```

**Magyarázat:**
- Létrehoztunk egy 2D-s tömböt (egy listákból álló listát konvertáltunk).
- A `data * 10` a tömb minden egyes elemét megszorozza 10-zel (nem kell hozzá `for` ciklus!).
- A `data + data` pedig celláról cellára összeadja az elemeket (pl. `1.5 + 1.5 = 3.0`).

**Az ndarray két legfontosabb attribútuma (tulajdonsága):**
Minden tömb homogén (csak azonos típusú elemeket tartalmazhat).

```python
In [17]: data.shape
Out[17]: (2, 3)
In [18]: data.dtype
Out[18]: dtype('float64')
```

**Magyarázat:**
- A `.shape` (alak) egy tuple, ami megmondja a dimenziók méretét: itt 2 sor és 3 oszlop.
- A `.dtype` (data type) az elemek típusát mutatja: itt 64-bites lebegőpontos számok.

### Tömbök (ndarrays) létrehozása

A legkézenfekvőbb módszer az `np.array` használata, amely bármilyen szekvenciát (pl. listát) tömbbé alakít.

```python
In [19]: data1 = [6, 7.5, 8, 0, 1]
In [20]: arr1 = np.array(data1)
In [21]: arr1
Out[21]: array([6. , 7.5, 8. , 0. , 1. ])

In [22]: data2 = [[1, 2, 3, 4], [5, 6, 7, 8]]
In [23]: arr2 = np.array(data2)
In [25]: arr2.ndim
Out[25]: 2
In [26]: arr2.shape
Out[26]: (2, 4)
```

**Magyarázat:** A `data2` egy "lista a listában", így a NumPy automatikusan felismeri, hogy ez egy 2-dimenziós tömb (`.ndim = 2`), aminek az alakja (`.shape`) 2 sor és 4 oszlop.

**Speciális tömb-létrehozó függvények:**
Gyakran van szükség előre kitöltött (pl. csupa nulla) tömbökre. Ilyenkor a shape-et egy tuple formájában adjuk meg.

```python
In [29]: np.zeros(10)
Out[29]: array([0., 0., 0., 0., 0., 0., 0., 0., 0., 0.])

In [30]: np.zeros((3, 6))  # 3 sor, 6 oszlop csupa nullával
Out[30]: 
array([[0., 0., 0., 0., 0., 0.],
       [0., 0., 0., 0., 0., 0.],
       [0., 0., 0., 0., 0., 0.]])

In [31]: np.empty((2, 3, 2))  # Helyfoglalás értékadás nélkül
In [32]: np.arange(15)        # A Python 'range' NumPy megfelelője
Out[32]: array([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14])
```

> **Fontos Figyelmeztetés:** Az `np.empty` nem nullákat ad vissza, hanem inicializálatlan memóriaterületet, ami bármilyen ott maradt "szemét" értéket tartalmazhat! Csak akkor használd, ha utána biztosan feltöltöd adatokkal.

#### Table 4-1. Fontos NumPy tömblétrehozó függvények

| Függvény | Leírás (Description) |
| :--- | :--- |
| `array` | Bemeneti adatot (lista, tuple) ndarray-é alakít, kitalálja a típust (vagy explicite megadható). Alapértelmezetten lemásolja az adatot. |
| `asarray` | Bemenetet alakít ndarray-é, de nem másol, ha a bemenet már eleve egy ndarray. |
| `arange` | Olyan, mint a beépített `range`, de listák helyett ndarray-t ad vissza. |
| `ones`, `ones_like` | Csupa 1-esből álló tömböt hoz létre. A `ones_like` egy másik tömb alakját és típusát veszi alapul, és azt tölti fel 1-esekkel. |
| `zeros`, `zeros_like` | Ugyanaz, mint a `ones`, csak csupa 0-val. |
| `empty`, `empty_like` | Új tömböt hoz létre memóriafoglalással, de nem tölti fel értékekkel (gyorsabb, mint a `zeros`, de szemétadatok lehetnek benne). |
| `full`, `full_like` | Létrehoz egy tömböt, és minden elemét feltölti a megadott értékkel (fill value). |
| `eye`, `identity` | Egy $N \times N$-es egységmátrixot (identitás mátrix) hoz létre (1-esek az átlón, mindenhol máshol 0). |

### Adattípusok az ndarray-k számára (Data Types / dtype)

A **dtype** (metadata, azaz adat az adatról) mondja meg a NumPy-nak, hogyan kell értelmezni a memóriában lévő nyers bájtokat. Ennek köszönhető, hogy a NumPy könnyen tud külső C vagy FORTRAN kódokkal kommunikálni.

```python
In [33]: arr1 = np.array([1, 2, 3], dtype=np.float64)
In [34]: arr2 = np.array([1, 2, 3], dtype=np.int32)
```

A típusok neve általában a típusfajtából (`int`, `float`) és az elemenként felhasznált bitek számából (8, 16, 32, 64) áll. Egy normál lebegőpontos szám 64 bitet (8 bájtot) foglal, ezért a neve `float64`.

#### Table 4-2. NumPy adattípusok

| Típus (Type) | Kód | Leírás (Description) |
| :--- | :--- | :--- |
| `int8`, `uint8` | `i1`, `u1` | Előjeles (signed) és előjel nélküli (unsigned) 8-bites (1 bájt) egész számok. (Az unsigned csak pozitív számokat tárolhat, pl. 0-255-ig). |
| `int16`, `uint16` | `i2`, `u2` | 16-bites egész számok. |
| `int32`, `uint32` | `i4`, `u4` | 32-bites egész számok. |
| `int64`, `uint64` | `i8`, `u8` | 64-bites egész számok. |
| `float16` | `f2` | Fél-pontosságú (Half-precision) lebegőpontos szám. |
| `float32` | `f4` | Szimpla pontosságú lebegőpontos szám (kompatibilis a C `float`-tal). |
| `float64` | `f8` | Dupla pontosságú lebegőpontos szám (kompatibilis a C `double`-lel és a Python normál `float`-jával). |
| `float128` | `f16` | Kiterjesztett pontosságú lebegőpontos szám. |
| `complex64`, `128`, `256` | `c8`, `c16`, `c32` | Komplex számok. |
| `bool` | `?` | Logikai típus (`True` vagy `False`). |
| `object` | `O` | Bármilyen általános Python objektum. |
| `string_` | `S` | Fix hosszúságú ASCII szövegtípus. Pl. egy 10 hosszú szöveghez: `'S10'`. |
| `unicode_` | `U` | Fix hosszúságú Unicode típus. |

### Típuskonverzió (Casting / .astype())

Egy tömb adattípusát az `.astype()` metódussal változtathatjuk meg. **Fontos:** A konverzió mindig egy teljesen új tömböt (másolatot) hoz létre a memóriában, még akkor is, ha a típus nem változott!

**Egészből lebegőpontosba (int -> float):**

```python
In [37]: arr = np.array([1, 2, 3, 4, 5])
In [39]: float_arr = arr.astype(np.float64)
In [40]: float_arr
Out[40]: array([1., 2., 3., 4., 5.])
```

**Lebegőpontosból egészbe (float -> int) TÖRVÉSSEL!**

```python
In [42]: arr = np.array([3.7, -1.2, -2.6, 0.5, 12.9, 10.1])
In [44]: arr.astype(np.int32)
Out[44]: array([ 3, -1, -2,  0, 12, 10], dtype=int32)
```

**Magyarázat:** Lebegőpontos szám egésszé alakításakor a NumPy **nem kerekít**, hanem egyszerűen levágja (truncates) a tizedesrészt! A `3.7`-ből `3` lesz, a `-2.6`-ból `-2`.

**Szövegből (String) számmá alakítás:**
Adatelemzésnél gyakori, hogy a számok szövegként vannak beolvasva (pl. egy CSV fájlból). Az `astype` ezt is képes kezelni:

```python
In [45]: numeric_strings = np.array(["1.25", "-9.6", "42"], dtype=np.string_)
In [46]: numeric_strings.astype(float)
Out[46]: array([  1.25,  -9.6 ,  42.  ])
```

> **Tipp:** Beírhattuk egyszerűen a `float` szót a `np.float64` helyett, mert a NumPy okosan lefordítja a beépített Python típusokat a saját típusaira. Később a pandas-ban ez még egyszerűbb lesz, ott intuitívabban működik a stringek konverziója, mint a natív NumPy `string_` típusánál.

**Konvertáláskor akár egy másik létező tömb típusát is átadhatjuk:**

```python
In [47]: int_array = np.arange(10)
In [48]: calibers = np.array([.22, .270, .357, .380, .44, .50], dtype=np.float64)
In [49]: int_array.astype(calibers.dtype)
Out[49]: array([0., 1., 2., 3., 4., 5., 6., 7., 8., 9.])
```
### 1. Matematikai műveletek tömbökkel (Arithmetic with NumPy Arrays)

**Fő koncepció:** A tömbök azért olyan fontosak, mert lehetővé teszik a kötegelt (batch) adatfeldolgozást anélkül, hogy `for` ciklusokat kellene írnunk. Ezt a NumPy világában **vektorizációnak** (vectorization) hívják.

Bármilyen aritmetikai (matematikai) művelet két egyenlő méretű tömb között elemenként (element-wise) hajtódik végre:

```python
In [52]: arr = np.array([[1., 2., 3.], [4., 5., 6.]])
In [54]: arr * arr
Out[54]: 
array([[ 1.,  4.,  9.],
       [16., 25., 36.]])

In [55]: arr - arr
Out[55]: 
array([[0., 0., 0.],
       [0., 0., 0.]])
```

**Magyarázat:** A szorzás és a kivonás is celláról cellára történik (pl. az első elem `1. * 1. = 1.`, a második `2. * 2. = 4.`).

Skalárokkal (sima számokkal) végzett műveletek esetén a NumPy a számot a tömb minden egyes elemére propagálja (kiterjeszti):

```python
In [56]: 1 / arr
Out[56]: 
array([[1.    , 0.5   , 0.3333],
       [0.25  , 0.2   , 0.1667]])

In [57]: arr ** 2
Out[57]: 
array([[ 1.,  4.,  9.],
       [16., 25., 36.]])
```

**Magyarázat:** Az `1 / arr` minden elemből reciprokot csinál, az `arr ** 2` pedig minden elemet négyzetre emel.

Azonos méretű tömbök összehasonlítása egy logikai (Boolean) tömböt eredményez:

```python
In [58]: arr2 = np.array([[0., 4., 1.], [7., 2., 12.]])
In [60]: arr2 > arr
Out[60]: 
array([[False,  True, False],
       [ True, False,  True]])
```

**Magyarázat:** A gép elemenként megnézi, hogy az `arr2` értéke nagyobb-e az `arr` értékénél (pl. `0 > 1` hamis, de `4 > 2` igaz).

### 2. Alapvető indexelés és szeletelés (Basic Indexing and Slicing)

Az egydimenziós tömbök a felszínen úgy viselkednek, mint a normál Python listák.

```python
In [61]: arr = np.arange(10)
In [63]: arr[5]
Out[63]: 5

In [64]: arr[5:8]
Out[64]: array([5, 6, 7])
```

Ha egy szeletnek egyetlen értéket adunk meg (pl. egy skalárt), az kiterjed (broadcast) a teljes kiválasztásra:

```python
In [65]: arr[5:8] = 12
In [66]: arr
Out[66]: array([ 0,  1,  2,  3,  4, 12, 12, 12,  8,  9])
```

> **KULCSFONTOSSÁGÚ SZABÁLY: A NumPy szeletek VIEW-k (Nézetek)!**
> A beépített Python listáktól eltérően, a NumPy tömbökből kivágott szeletek (slices) **nem másolatok**. A szelet valójában egy "ablak" (nézet) az eredeti tömbre. Ha módosítod a szeletet, az eredeti tömb is megváltozik!

```python
In [67]: arr_slice = arr[5:8]
In [69]: arr_slice[1] = 12345
In [70]: arr
Out[70]: array([    0,     1,     2,     3,     4,    12, 12345,    12,     8,     9])

In [71]: arr_slice[:] = 64
In [72]: arr
Out[72]: array([ 0,  1,  2,  3,  4, 64, 64, 64,  8,  9])
```

**Magyarázat:** A NumPy hatalmas adatokkal dolgozik. Ha minden szeletelés memóriamásolással járna, a gép pillanatok alatt összeomlana. Ezért alapértelmezetten mindent "helyben" tart. Ha tényleg egy független másolatot akarsz, az `.copy()` metódust kell használnod (pl. `arr[5:8].copy()`).

#### Többdimenziós tömbök indexelése

Egy 2D-s tömbnél az első index a sort, a második az oszlopot jelenti. Kétféleképpen is hivatkozhatunk egy cellára, de a vesszős megoldás a standard:

```python
In [73]: arr2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
In [75]: arr2d[0][2]
Out[75]: 3
In [76]: arr2d[0, 2]
Out[76]: 3
```

> 📍 **[KÉP HELYE: Figure 4-1. Indexing elements in a NumPy array]**
>
> Segítség az ábrához: A NumPy-ban a 0-s tengely (axis 0) mindig a sorokat, az 1-es tengely (axis 1) az oszlopokat jelenti.

Ha egy 3 dimenziós tömbnél kihagyod az utolsó indexeket, az eredmény egy alacsonyabb dimenziós tömb (pl. egy teljes 2D-s mátrix) lesz:

```python
In [77]: arr3d = np.array([[[1, 2, 3], [4, 5, 6]], [[7, 8, 9], [10, 11, 12]]])
In [79]: arr3d[0]
Out[79]: 
array([[1, 2, 3],
       [4, 5, 6]])
```

### 3. Indexelés szeletekkel (Slicing)

A szeletelés több dimenzióban is működik. Ilyenkor vesszővel elválasztva adod meg, hogy melyik tengelyen mennyit akarsz vágni.

```python
In [91]: arr2d
Out[91]: 
array([[1, 2, 3],
       [4, 5, 6],
       [7, 8, 9]])

In [92]: arr2d[:2]
Out[92]: 
array([[1, 2, 3],
       [4, 5, 6]])
```

**Magyarázat:** A `[:2]` az első dimenzióra (sorokra) vonatkozik, így kiválasztotta az első két sort.

```python
In [93]: arr2d[:2, 1:]
Out[93]: 
array([[2, 3],
       [5, 6]])
```

**Magyarázat:** Kiválasztjuk az első két sort (`:2`), **MAJD** ezekből a sorokból a második oszloptól a végéig tartó részt (`1:`).

> 📍 **[KÉP HELYE: Figure 4-2. Two-dimensional array slicing]**

Ha egy szeletnek értéket adunk, az az egész kijelölt területre kiterjed:

```python
In [98]: arr2d[:2, 1:] = 0
```

**Magyarázat:** A jobb felső 2x2-es blokk minden elemét kinulláztuk.

### 4. Logikai indexelés (Boolean Indexing)

Nagyon gyakori, hogy egy adatbázisban egy bizonyos feltétel alapján akarunk sorokat szűrni.
Vegyünk egy nevek tömbjét és egy adatmátrixot, ahol minden névhez tartozik egy sor az adatmátrixban:

```python
In [100]: names = np.array(["Bob", "Joe", "Will", "Bob", "Will", "Joe", "Joe"])
In [101]: data = np.array([[4, 7], [0, 2], [-5, 6], [0, 0], [1, 2], [-12, -4], [3, 4]])
```

Szűrjük ki a "Bob" nevű illetőhöz tartozó sorokat! A `names == "Bob"` összehasonlítás egy `True`/`False` tömböt ad:

```python
In [104]: names == "Bob"
Out[104]: array([ True, False, False,  True, False, False, False])

In [105]: data[names == "Bob"]
Out[105]: 
array([[4, 7],
       [0, 0]])
```

**Magyarázat:** A logikai tömb "maszkként" (mask) működik. Csak azokat a sorokat adja vissza a `data` mátrixból, ahol a maszk értéke `True` (itt a legelső és a negyedik sor).

Kombinálhatod a logikai szűrést oszlop-kiválasztással is:

```python
In [106]: data[names == "Bob", 1:]  # Bob sorai, de csak a 2. oszloptól
Out[106]: 
array([[7],
       [0]])
```

**Tagadás (Nem Bob):** Használhatod a `!=` operátort, vagy a `~` (tilde) jelet, ami megfordítja a `True`/`False` értékeket.

```python
In [110]: data[~(names == "Bob")]
```

**Több feltétel (ÉS / VAGY):**
A normál Python `and` és `or` kulcsszavak **NEM MŰKÖDNEK** a NumPy logikai tömbjeivel. Helyettük a bitenkénti `&` (és) valamint a `|` (vagy) jeleket kell használni.

```python
In [113]: mask = (names == "Bob") | (names == "Will")
In [115]: data[mask]
```

> **Fontos különbség:** A logikai indexelés **MINDIG LÉTREHOZ EGY MÁSOLATOT** a memóriában (nem úgy, mint a sima szeletelés)!

**Értékek tömeges módosítása feltétel alapján:**
Villámgyorsan beállíthatjuk például az összes negatív számot nullára:

```python
In [116]: data[data < 0] = 0
```

### 5. Cifra / Trükkös Indexelés (Fancy Indexing)

A "Fancy Indexing" a NumPy szakkifejezése arra, amikor egész számokat tartalmazó tömbökkel (vagy listákkal) indexelünk.

```python
In [120]: arr = np.zeros((8, 4))
In [121]: for i in range(8):
   .....:     arr[i] = i
```

Ez készített egy 8x4-es mátrixot, ahol minden sorban a sorszám van.
Ha csak bizonyos sorokat akarunk lekérni egy kívánt sorrendben, egyszerűen átadunk egy listát a sorok indexeivel:

```python
In [123]: arr[[4, 3, 0, 6]]
Out[123]: 
array([[4., 4., 4., 4.],
       [3., 3., 3., 3.],
       [0., 0., 0., 0.],
       [6., 6., 6., 6.]])
```

**Koordináta-alapú kiválasztás (Több listával):**
Ha két (vagy több) listát adsz át, az **NEM** egy téglalap alakú régiót fog kivágni! Helyette a listák elemeit párosítja (mint a `zip`), és konkrét pontokat / cellákat választ ki.

```python
In [125]: arr = np.arange(32).reshape((8, 4))
In [127]: arr[[1, 5, 7, 2], [0, 3, 1, 2]]
Out[127]: array([ 4, 23, 29, 10])
```

**Magyarázat:** A gép vette az `(1, 0)`, az `(5, 3)`, a `(7, 1)` és a `(2, 2)` koordinátájú elemeket, és ezt adta vissza egy 1D-s tömbben.

**Hogyan vágjunk ki téglalapot?**
Ha te valójában az 1, 5, 7, 2 soroknak és a 0, 3, 1, 2 oszlopoknak a metszetét (egy kisebb mátrixot) akartad kiválasztani, azt két lépésben kell megtenned:

```python
In [128]: arr[[1, 5, 7, 2]][:, [0, 3, 1, 2]]
Out[128]: 
array([[ 4,  7,  5,  6],
       [20, 23, 21, 22],
       [28, 31, 29, 30],
       [ 8, 11,  9, 10]])
```

> **Fontos szabály:** Csakúgy, mint a logikai indexelés, a Fancy Indexing is **MINDIG MÁSOLATOT KÉSZÍT** (nem nézetet ad) az adatokról, ha egy új változóba mentjük! De ha közvetlenül értéket rendelünk hozzá (pl. `arr[[1, 5, 7, 2], [0, 3, 1, 2]] = 0`), az módosítja a tömböt.


### 6. Tömbök transzponálása és tengelyek cseréje (Transposing and Swapping Axes)

**Fő koncepció:** A transzponálás (a sorok és oszlopok felcserélése) a tömbök átalakításának egy speciális formája. A szeleteléshez hasonlóan ez is csak egy **nézetet (view)** ad vissza, tehát nem másolja le az adatokat a memóriában.

Minden NumPy tömbnek van egy `.T` attribútuma (tulajdonsága), ami a transzponáltját adja vissza:

```python
In [132]: arr = np.arange(15).reshape((3, 5))
In [133]: arr
Out[133]: 
array([[ 0,  1,  2,  3,  4],
       [ 5,  6,  7,  8,  9],
       [10, 11, 12, 13, 14]])

In [134]: arr.T
Out[134]: 
array([[ 0,  5, 10],
       [ 1,  6, 11],
       [ 2,  7, 12],
       [ 3,  8, 13],
       [ 4,  9, 14]])
```

**Magyarázat:** A `reshape((3, 5))` a 15 számot egy 3 soros, 5 oszlopos mátrixba rendezte. Az `arr.T` ezt megfordította: a sorokból oszlopok lettek, így az eredmény egy 5x3-as mátrix.

**Mátrixszorzás:**
Adatelemzésnél és gépi tanulásnál nagyon gyakori a belső mátrixszorzat kiszámítása (pl. $X^T X$). Erre az `np.dot()` függvényt vagy a Python új `@` operátorát használhatjuk:

```python
In [135]: arr = np.array([[0, 1, 0], [1, 2, -2], [6, 3, 2], [-1, 0, -1], [1, 0, 1]])
In [137]: np.dot(arr.T, arr)
Out[137]: 
array([[39, 20, 12],
       [20, 14,  2],
       [12,  2, 10]])

# Ugyanez rövidebben az @ operátorral:
In [138]: arr.T @ arr
Out[138]: 
array([[39, 20, 12],
       [20, 14,  2],
       [12,  2, 10]])
```

**Tengelyek felcserélése (`swapaxes`):**
A sima `.T` a transzponálás egy speciális esete. Ha többdimenziós (pl. 3D vagy 4D) tömbbel dolgozol, a `.swapaxes(tengely1, tengely2)` metódussal pontosan megadhatod, hogy melyik két tengelyt akarod megcserélni:

```python
In [140]: arr.swapaxes(0, 1)
```

**Magyarázat:** Kicseréli a 0-s tengelyt (sorokat) az 1-es tengellyel (oszlopokkal). Ez is csak egy nézetet (view) ad vissza, memóriamásolás nem történik.

## 4.2 Pszeudovéletlenszám-generálás (Pseudorandom Number Generation)

A `numpy.random` modul kiegészíti a beépített Python `random` modult, és lehetővé teszi, hogy hatalmas tömböket töltsünk fel másodpercek töredéke alatt véletlenszerű (statisztikai eloszlásból származó) számokkal.

```python
In [141]: samples = np.random.standard_normal(size=(4, 4))
In [142]: samples
Out[142]: 
array([[-0.2047,  0.4789, -0.5194, -0.5557],
       [ 1.9658,  1.3934,  0.0929,  0.2817],
       [ 0.769 ,  1.2464,  1.0072, -1.2962],
       [ 0.275 ,  0.2289,  1.3529,  0.8864]])
```

**Magyarázat:** Létrehoztunk egy 4x4-es mátrixot standard normál eloszlású (0 várható értékű, 1 szórású) véletlenszámokkal.

**Miért jobb ez, mint a sima Python random? (Teljesítmény)**
A beépített Python modul egyszerre csak **EGY** számot tud generálni. Ha 1 millió szám kell, a NumPy nagyságrendekkel (több mint tízszer) gyorsabb:

```python
# Sima Python random:
In [145]: %timeit samples = [normalvariate(0, 1) for _ in range(N)]
1.04 s +- 11.4 ms per loop

# NumPy random:
In [146]: %timeit np.random.standard_normal(N)
21.9 ms +- 155 us per loop
```

**Véletlenszám-generátor (RNG) objektum létrehozása:**
Modern NumPy-ban (1.17 verzió felett) a legjobb gyakorlat, ha egy elszigetelt, dedikált generátor objektumot hozunk létre (így nem zavarjuk össze a többi programrészt, ami szintén véletlenszámokat használ).

```python
In [147]: rng = np.random.default_rng(seed=12345)
In [148]: data = rng.standard_normal((2, 3))
```

**Magyarázat:** A `seed` (mag) határozza meg a generátor kezdőállapotát. Ha ugyanazt a seed-et adod meg, a "véletlen" számok mindig hajszálpontosan ugyanazok lesznek (ezért hívjuk őket pszeudo-véletlennek). Ez elengedhetetlen a reprodukálható adatelemzéshez!

### Table 4-3. NumPy véletlenszám-generátor (RNG) metódusok

| Metódus | Leírás (Description) |
| :--- | :--- |
| `permutation` | Egy szekvencia véletlenszerű összekeverését (permutációját) adja vissza új tömbként. |
| `shuffle` | Helyben (in-place) megkever (összeráz) egy szekvenciát. |
| `uniform` | Számokat húz egyenletes (uniform) eloszlásból. |
| `integers` | Véletlen egész számokat generál egy megadott mettől-meddig tartományban. |
| `standard_normal` | Standard normál eloszlásból húz mintát (átlag=0, szórás=1). |
| `binomial` | Binomiális eloszlásból húz mintát. |
| `normal` | Normál (Gauss) eloszlásból húz mintát (megadható átlag és szórás). |
| `beta`, `chisquare`, `gamma` | Béta, Khí-négyzet és Gamma eloszlásokból húz mintát. |

## 4.3 Univerzális Függvények (Universal Functions / ufuncs)

**Fő koncepció:** Az univerzális függvények (röviden **ufunc**-ok) olyan függvények, amelyek elemenként (element-wise) végeznek el gyors számításokat a NumPy tömbökön. Képzeld el őket úgy, mint a sima matematikai függvények gyors, tömbösített (vektorizált) változatát.

### Egyváltozós (Unary) ufunc-ok

Egyetlen tömböt várnak bemenetként. Például gyökvonás (`sqrt`) vagy exponenciális függvény (`exp`):

```python
In [150]: arr = np.arange(10)
In [152]: np.sqrt(arr)
Out[152]: 
array([0.    , 1.    , 1.4142, 1.7321, 2.    , 2.2361, 2.4495, 2.6458,
       2.8284, 3.    ])

In [153]: np.exp(arr)
Out[153]: 
array([   1.    ,    2.7183,    7.3891,   20.0855,   54.5982,  148.4132,
        403.4288, 1096.6332, 2980.958 , 8103.0839])
```

### Kétváltozós (Binary) ufunc-ok

Két tömböt várnak, és egy harmadikat adnak vissza eredményként (pl. nemenkénti maximumkeresés):

```python
In [154]: x = rng.standard_normal(8)
In [155]: y = rng.standard_normal(8)

In [158]: np.maximum(x, y)
Out[158]: 
array([-0.467 ,  0.6489,  0.7888, -1.2567,  2.3474,  1.399 ,  1.3223,
        0.9022])
```

**Magyarázat:** A gép elemenként (indexenként) összehasonlította az `x` és az `y` tömböt, és mindig a nagyobbat tette be a végső tömbbe.

### Több visszatérési érték (`modf`)

Ritka, de van olyan ufunc, ami több tömböt is visszaad. Az `np.modf` szétválasztja a számok egész és tizedes részét:

```python
In [161]: remainder, whole_part = np.modf(arr)
In [162]: remainder
Out[162]: array([ 0.5146, -0.1079, -0.7909,  0.2474, -0.718 , -0.4084,  0.6237])
In [163]: whole_part
Out[163]: array([ 4., -8., -0.,  2., -6., -0.,  8.])
```

### Helyben (in-place) módosítás az `out` argumentummal

Ha nem akarsz új tömböt létrehozni a memóriában az eredménynek, az ufunc eredményét közvetlenül beleírhatod egy létező tömbbe:

```python
In [165]: out = np.zeros_like(arr)
In [167]: np.add(arr, 1, out=out)
Out[167]: array([ 5.5146, -7.1079,  0.2091,  3.2474, -5.718 ,  0.5916,  9.6237])
```

**Magyarázat:** A gép hozzáadott 1-et az `arr` minden eleméhez, és az eredményt rögtön a nullákkal teli `out` tömbbe írta, anélkül, hogy harmadik tömböt készített volna a memóriában.

### Table 4-4. Néhány egyváltozós (Unary) ufunc

| Függvény | Leírás (Description) |
| :--- | :--- |
| `abs`, `fabs` | Kiszámítja az abszolút értéket elemenként (egész, tört vagy komplex számokon). |
| `sqrt` | Négyzetgyökvonás elemenként (ugyanaz, mint `arr ** 0.5`). |
| `square` | Négyzetre emelés elemenként (ugyanaz, mint `arr ** 2`). |
| `exp` | Kiszámítja az $e^x$ értéket elemenként. |
| `log`, `log10`, `log2`, `log1p` | Természetes logaritmus (e alapú), 10-es alapú, 2-es alapú, és log(1 + x). |
| `sign` | Előjel függvény: 1 (pozitív), 0 (nulla), vagy -1 (negatív). |
| `ceil` | "Plafon": Felfelé kerekít a legközelebbi egész számra. |
| `floor` | "Padló": Lefelé kerekít a legközelebbi egész számra. |
| `rint` | A legközelebbi egészre kerekít (de megtartja a lebegőpontos dtype-ot). |
| `modf` | Szétválasztja a törtrészt és az egészt két külön tömbbe. |
| `isnan` | Logikai tömböt ad vissza: Igaz (`True`), ha az elem NaN (Not a Number, pl. hiányzó adat). |
| `isfinite`, `isinf` | Logikai tömb: Igaz (`True`), ha az elem véges, illetve végtelen. |
| `cos`, `sin`, `tan` stb. | Trigonometrikus és hiperbolikus függvények. |
| `arccos`, `arcsin` stb. | Inverz trigonometrikus függvények. |
| `logical_not` | A logikai NEM (tagadás) megfelelője (ugyanaz, mint a `~arr`). |

### Table 4-5. Néhány kétváltozós (Binary) ufunc

| Függvény | Leírás (Description) |
| :--- | :--- |
| `add` | Két tömb azonos indexű elemeit összeadja. |
| `subtract` | Kivonja a második tömböt az elsőből. |
| `multiply` | Összeszorozza a tömböket elemenként. |
| `divide`, `floor_divide` | Osztás, illetve egész osztás (maradék eldobása). |
| `power` | Az első tömb elemeit a második tömbben megadott értékű hatványra emeli. |
| `maximum`, `fmax` | Elemenkénti maximumkeresés (a nagyobb értéket adja vissza). Az `fmax` figyelmen kívül hagyja a NaN (hiányzó) értékeket. |
| `minimum`, `fmin` | Elemenkénti minimumkeresés (az `fmin` figyelmen kívül hagyja a NaN-t). |
| `mod` | Modulo (maradékos osztás, megadja az osztás maradékát). |
| `copysign` | A második tömb előjeleit "rámásolja" az első tömb értékeire. |
| `greater`, `less`, `equal` stb. | Elemenkénti összehasonlító operátorok (`>`, `<`, `==`, `<=`, `>=`, `!=`), amik logikai (`True`/`False`) tömböt adnak vissza. |
| `logical_and`, `or`, `xor` | Elemenkénti bitwise logikai operátorok (ÉS, VAGY, KIZÁRÓ VAGY). |


## Tömbközpontú programozás (Array-Oriented Programming)

A NumPy tömbök lehetővé teszik, hogy a legtöbb adatfeldolgozási feladatot tömör tömb-kifejezésekkel végezzük el, ahelyett, hogy `for` ciklusokat írnánk. A ciklusok tömb-kifejezésekre való lecserélését **vektorizációnak** (vectorization) hívjuk. A vektorizált műveletek nagyságrendekkel gyorsabbak a tiszta Python kódnál.

Vegyünk egy egyszerű példát: értékeljük ki a $\sqrt{x^2 + y^2}$ függvényt egy szabályos érték-rácson (grid). Az `np.meshgrid` függvény két 1D-s tömbből létrehoz két 2D-s mátrixot, ami az összes lehetséges (x, y) koordinátapárt tartalmazza.

```python
In [169]: points = np.arange(-5, 5, 0.01) # 1000 egyenletesen elosztott pont
In [170]: xs, ys = np.meshgrid(points, points)
In [171]: ys
Out[171]: 
array([[-5.  , -5.  , -5.  , ..., -5.  , -5.  , -5.  ],
       [-4.99, -4.99, -4.99, ..., -4.99, -4.99, -4.99],
       [-4.98, -4.98, -4.98, ..., -4.98, -4.98, -4.98],
       ...,
       [ 4.97,  4.97,  4.97, ...,  4.97,  4.97,  4.97],
       [ 4.98,  4.98,  4.98, ...,  4.98,  4.98,  4.98],
       [ 4.99,  4.99,  4.99, ...,  4.99,  4.99,  4.99]])
```

**Magyarázat:** Létrehoztunk egy 1D-s listát -5 és 5 között 0.01-es lépésközzel. A `meshgrid` ebből egy 2D-s koordináta-rendszert csinált (az `xs` az X, az `ys` az Y koordinátákat tartalmazza minden rácspontra).

Ezután maga a függvény kiszámítása csak egyetlen sor:

```python
In [172]: z = np.sqrt(xs ** 2 + ys ** 2)
In [173]: z
Out[173]: 
array([[7.0711, 7.064 , 7.0569, ..., 7.0499, 7.0569, 7.064 ],
       [7.064 , 7.0569, 7.0499, ..., 7.0428, 7.0499, 7.0569],
       [7.0569, 7.0499, 7.0428, ..., 7.0357, 7.0428, 7.0499],
       ...,
       [7.0499, 7.0428, 7.0357, ..., 7.0286, 7.0357, 7.0428],
       [7.0569, 7.0499, 7.0428, ..., 7.0357, 7.0428, 7.0499],
       [7.064 , 7.0569, 7.0499, ..., 7.0428, 7.0499, 7.0569]])
```

A kiszámolt mátrixot a `matplotlib` segítségével vizualizálhatjuk képpé:

```python
In [174]: import matplotlib.pyplot as plt
In [175]: plt.imshow(z, cmap=plt.cm.gray, extent=[-5, 5, -5, 5])
Out[175]: <matplotlib.image.AxesImage at 0x7f624ae73b20>
In [176]: plt.colorbar()
Out[176]: <matplotlib.colorbar.Colorbar at 0x7f6253e43ee0>
In [177]: plt.title("Image plot of $\sqrt{x^2 + y^2}$ for a grid of values")
Out[177]: Text(0.5, 1.0, 'Image plot of $\\sqrt{x^2 + y^2}$ for a grid of values')
```

> 📍 **[KÉP HELYE: Figure 4-3. Plot of function evaluated on a grid]**
>
> **Tipp:** A megnyitott ablakokat a `plt.close("all")` paranccsal zárhatod be.

### Feltételes logika kifejezése tömbműveletként (np.where)

A `numpy.where` függvény a Python `x if condition else y` kifejezésének vektorizált verziója.

Tegyük fel, van két tömbünk és egy logikai tömbünk:

```python
In [180]: xarr = np.array([1.1, 1.2, 1.3, 1.4, 1.5])
In [181]: yarr = np.array([2.1, 2.2, 2.3, 2.4, 2.5])
In [182]: cond = np.array([True, False, True, True, False])
```

Ha ott akarunk értéket venni az `xarr`-ból, ahol a `cond` Igaz, különben az `yarr`-ból, list comprehension-nel ez lassú és többdimenziós tömbökkel nem működik. Az `np.where` azonnal megoldja:

```python
In [185]: result = np.where(cond, xarr, yarr)
In [186]: result
Out[186]: array([1.1, 2.2, 1.3, 1.4, 2.5])
```

Az `np.where` paraméterei lehetnek egyszerű számok (skalárok) is. Adattisztításnál gyakran cserélünk ki értékeket egy feltétel alapján. Például cseréljük le egy mátrix összes pozitív számát 2-re, a negatívokat -2-re:

```python
In [187]: arr = rng.standard_normal((4, 4))
In [189]: arr > 0
Out[189]: 
array([[ True,  True,  True, False],
       [False, False,  True,  True],
       [False, False,  True,  True],
       [False,  True, False, False]])

In [190]: np.where(arr > 0, 2, -2)
Out[190]: 
array([[ 2,  2,  2, -2],
       [-2, -2,  2,  2],
       [-2, -2,  2,  2],
       [-2,  2, -2, -2]])
```

Csak a pozitívakat 2-re, a többi maradjon az eredeti `arr` értéke:

```python
In [191]: np.where(arr > 0, 2, arr) # Csak a pozitív értékeket állítja 2-re
```

## Matematikai és Statisztikai Metódusok

A tömbök adatain aggregációkat (vagy redukciókat) végezhetünk, mint a szum (`sum`), átlag (`mean`) vagy szórás (`std`). Ezeket használhatjuk a tömb metódusaként (`arr.mean()`) vagy top-level függvényként (`np.mean(arr)`).

```python
In [192]: arr = rng.standard_normal((5, 4))
In [194]: arr.mean()
Out[194]: -0.08719744457434529
In [196]: arr.sum()
Out[196]: -1.743948891486906
```

**Az `axis` (tengely) argumentum:**
Irányíthatjuk, hogy a statisztika melyik tengely mentén számolódjon ki (így az eredmény egy dimenzióval kisebb lesz).

```python
In [197]: arr.mean(axis=1)  # Kiszámolja az átlagot oszlopokon KERESZTÜL (soronként)
Out[197]: array([ 0.109 ,  0.3281,  0.165 , -0.6672, -0.3709])

In [198]: arr.sum(axis=0)   # Kiszámolja az összeget a sorokon LEFELÉ (oszloponként)
Out[198]: array([-1.6292,  1.0399, -0.3344, -0.8203])
```

**Halmozott / Kumulatív függvények** (pl. `cumsum`, `cumprod`) nem egyetlen értékké redukálnak, hanem azonos méretű tömböt adnak vissza a részeredményekkel:

```python
In [199]: arr = np.array([0, 1, 2, 3, 4, 5, 6, 7])
In [200]: arr.cumsum()
Out[200]: array([ 0,  1,  3,  6, 10, 15, 21, 28])
```

Mátrixoknál:

```python
In [201]: arr = np.array([[0, 1, 2], [3, 4, 5], [6, 7, 8]])
In [203]: arr.cumsum(axis=0) # Oszloponként adja össze (lefelé)
Out[203]: 
array([[ 0,  1,  2],
       [ 3,  5,  7],
       [ 9, 12, 15]])
```

### Table 4-6. Alapvető tömb-statisztikai metódusok

| Metódus | Leírás (Description) |
| :--- | :--- |
| `sum` | A tömb (vagy egy tengelye mentén lévő) elemeinek összege. (Üres tömbre 0-t ad). |
| `mean` | Aritmetikai átlag. (Üres tömbre NaN-t ad). |
| `std`, `var` | Szórás (standard deviation) és variancia. |
| `min`, `max` | Minimum és maximum érték. |
| `argmin`, `argmax` | A minimum és maximum értékek indexeit adja vissza. |
| `cumsum` | Kumulatív (halmozott) összeg, 0-tól indulva. |
| `cumprod` | Kumulatív szorzat, 1-től indulva. |

### Metódusok logikai tömbökhöz

A logikai (Boolean) értékeket a statisztikai függvények automatikusan számokká alakítják (`True`=1, `False`=0). A `.sum()` a leggyakoribb módja annak, hogy megszámoljuk az Igaz értékeket:

```python
In [205]: arr = rng.standard_normal(100)
In [206]: (arr > 0).sum() # Pozitív értékek száma
Out[206]: 48
In [207]: (arr <= 0).sum() # Nem-pozitív értékek száma
Out[207]: 52
```

Két nagyon hasznos logikai vizsgáló függvény:
- `any()`: Igaz, ha legalább egy elem Igaz a tömbben.
- `all()`: Igaz, ha minden elem Igaz a tömbben.

```python
In [208]: bools = np.array([False, False, True, False])
In [209]: bools.any()
Out[209]: True
In [210]: bools.all()
Out[210]: False
```

### Rendezés (Sorting)

A NumPy tömbök a `.sort()` metódussal helyben (in-place) rendezhetők.

```python
In [211]: arr = rng.standard_normal(6)
In [213]: arr.sort()
In [214]: arr
Out[214]: array([-0.7208, -0.6839, -0.0824, -0.0548,  0.0773,  1.1206])
```

Többdimenziós tömböknél megadhatod a tengelyt (`axis`), ami mentén a rendezést végre kell hajtani:

```python
In [215]: arr = rng.standard_normal((5, 3))
In [217]: arr.sort(axis=0) # Sorokon lefelé (az oszlopok elemeit) rendezi
In [219]: arr.sort(axis=1) # Oszlopokon keresztül (a sorok elemeit) rendezi
```

Ha nem helyben akarod módosítani a tömböt, hanem egy rendezett másolatot akarsz, használd a top-level `np.sort()` függvényt:

```python
In [222]: sorted_arr2 = np.sort(arr2)
```

### Egyedi (Unique) értékek és egyéb halmaz-logika

Az egydimenziós tömbökre a NumPy beépített halmazműveletekkel rendelkezik. A leggyakrabban használt az `np.unique`, ami egy rendezett tömböt ad vissza a duplikációk kiszűrése után.

```python
In [224]: names = np.array(["Bob", "Will", "Joe", "Bob", "Will", "Joe", "Joe"])
In [225]: np.unique(names)
Out[225]: array(['Bob', 'Joe', 'Will'], dtype='<U4')
```

(Az `np.unique` sokkal gyorsabb, mint a tiszta Python alternatíva: `sorted(set(names))`.)

Az `np.in1d` leellenőrzi, hogy egy tömb elemei benne vannak-e egy másik szekvenciában, és logikai tömbbel tér vissza:

```python
In [229]: values = np.array([6, 0, 0, 3, 2, 5, 6])
In [230]: np.in1d(values, [2, 3, 6])
Out[230]: array([ True, False, False,  True,  True, False,  True])
```

### Table 4-7. Tömbös halmazműveletek (Array set operations)

| Metódus | Leírás (Description) |
| :--- | :--- |
| `unique(x)` | Kiszámítja `x` egyedi (duplikációmentes) elemeit, és rendezve adja vissza. |
| `intersect1d(x, y)` | Metszet: `x` és `y` közös elemei, rendezve. |
| `union1d(x, y)` | Unió: a két tömb elemeinek összessége (egyediként), rendezve. |
| `in1d(x, y)` | Logikai tömb: jelzi, hogy az `x` egyes elemei megtalálhatók-e az `y`-ban. |
| `setdiff1d(x, y)` | Különbség: azok az elemek `x`-ben, amik nincsenek benne `y`-ban. |
| `setxor1d(x, y)` | Szimmetrikus különbség: elemek, amik valamelyik tömbben benne vannak, de nem mindkettőben. |

## 4.5 Fájl be- és kimenet tömbökkel (File Input and Output)

A NumPy képes tömböket közvetlenül a lemezre (disk) menteni és onnan betölteni. A beépített bináris formátumok gyorsak és hatékonyak.

Az `np.save` és `np.load` az alapvető függvények. A mentés alapértelmezetten tömörítetlen nyers bináris formátumban történik, `.npy` kiterjesztéssel:

```python
In [231]: arr = np.arange(10)
In [232]: np.save("some_array", arr) # Ha nincs .npy kiterjesztés, automatikusan hozzáadja
In [233]: np.load("some_array.npy")
Out[233]: array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
```

Ha több tömböt is egyetlen fájlba (archívumba) akarsz menteni, használd az `np.savez` függvényt, ahol a tömböket kulcsszavas paraméterként adod át. A fájl `.npz` kiterjesztést kap.

```python
In [234]: np.savez("array_archive.npz", a=arr, b=arr)
In [235]: arch = np.load("array_archive.npz")
In [236]: arch["b"] # Szótár-szerűen lehet lekérni az egyes tömböket belőle
Out[236]: array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
```

Ha jól tömöríthető adatokkal dolgozol, a kisebb fájlméret érdekében érdemes az `np.savez_compressed` függvényt használni:

```python
In [237]: np.savez_compressed("arrays_compressed.npz", a=arr, b=arr)
```

## 4.6 Lineáris Algebra (Linear Algebra)

A mátrixszorzások, felbontások, determinánsok és egyéb műveletek kulcsfontosságúak az array-alapú könyvtáraknál. Míg a `*` operátor elemenkénti (element-wise) szorzást végez, a valódi mátrixszorzáshoz a `dot` függvényre (vagy az `@` operátorra) van szükség.

```python
In [241]: x = np.array([[1., 2., 3.], [4., 5., 6.]])
In [242]: y = np.array([[6., 23.], [-1, 7], [8, 9]])

# Mátrixszorzás .dot() metódussal:
In [245]: x.dot(y)
Out[245]: 
array([[ 28.,  64.],
       [ 67., 181.]])

# Mátrixszorzás top-level függvénnyel:
In [246]: np.dot(x, y)
Out[246]: 
array([[ 28.,  64.],
       [ 67., 181.]])

# Mátrixszorzás 2D és 1D tömb között (az eredmény egy 1D tömb) az @ operátorral:
In [247]: x @ np.ones(3)
Out[247]: array([  6.,  15.])
```

A `numpy.linalg` modul tartalmazza a lineáris algebrai "nehézfegyverzetet", mint például az inverz számítást (`inv`) és a felbontásokat. Számítsuk ki egy $X^T X$ mátrix inverzét:

```python
In [248]: from numpy.linalg import inv, qr
In [249]: X = rng.standard_normal((5, 5))
In [250]: mat = X.T @ X # Mátrixszorzat a transzponálttal
In [251]: inv(mat)      # A mátrix inverze
Out[251]: 
array([[  3.4993,   2.8444,   3.5956, -16.5538,   4.4733],
       [  2.8444,   2.5667,   2.9002, -13.5774,   3.7678],
       [  3.5956,   2.9002,   4.4823, -18.3453,   4.7066],
       [-16.5538, -13.5774, -18.3453,  84.0102, -22.0484],
       [  4.4733,   3.7678,   4.7066, -22.0484,   6.0525]])

# Egy mátrix és az inverzének szorzata az egységmátrixot (Identity matrix) adja:
In [252]: mat @ inv(mat)
Out[252]: 
array([[ 1.,  0., -0.,  0., -0.],
       [ 0.,  1.,  0.,  0., -0.],
       [ 0., -0.,  1., -0., -0.],
       [ 0., -0.,  0.,  1., -0.],
       [ 0., -0.,  0., -0.,  1.]])
```

### Table 4-8. Gyakran használt numpy.linalg függvények

| Függvény | Leírás (Description) |
| :--- | :--- |
| `diag` | Visszaadja egy négyzetes mátrix átlóját (diagonálisát) egy 1D-s tömbként, VAGY egy 1D-s tömbből csinál négyzetes mátrixot, ahol a tömb elemei az átlón vannak (máshol 0). |
| `dot` | Mátrixszorzás. |
| `trace` | Kiszámítja az átlón lévő elemek összegét. |
| `det` | Kiszámítja a mátrix determinánsát. |
| `eig` | Kiszámítja egy négyzetes mátrix sajátértékeit (eigenvalues) és sajátvektorait (eigenvectors). |
| `inv` | Kiszámítja egy négyzetes mátrix inverzét. |
| `pinv` | Kiszámítja egy mátrix Moore-Penrose pszeudoinverzét. |
| `qr` | Kiszámítja a QR felbontást (QR decomposition). |
| `svd` | Kiszámítja a szinguláris érték felbontást (Singular Value Decomposition). |
| `solve` | Megoldja az $Ax = b$ lineáris egyenletrendszert az $x$-re, ahol az $A$ egy négyzetes mátrix. |
| `lstsq` | Kiszámítja az $Ax = b$ egyenletrendszer legkisebb négyzetes (least-squares) megoldását. |

## 4.7 Példa: Véletlenszerű bolyongás (Random Walks)

A **véletlenszerű bolyongás** (olyan sorozat, ahol minden lépésben véletlenszerűen lépünk egyet előre vagy hátra) tökéletes példa a tömbműveletek hasznosítására.

Kezdjük egy egyszerű bolyongással: a 0 pozícióból indulunk, és minden lépésben 1-et vagy -1-et lépünk, egyenlő eséllyel.

### A hagyományos (Tiszta Python) megoldás

Így nézne ki egy 1000 lépéses bolyongás, ha a beépített `random` modult és egy egyszerű `for` ciklust használnánk:

```python
import random
position = 0
walk = [position]
nsteps = 1000
for _ in range(nsteps):
    step = 1 if random.randint(0, 1) else -1
    position += step
    walk.append(position)
```

**Magyarázat:**

- Beállítjuk a kezdő pozíciót 0-ra, és beletesszük a `walk` nevű listába.
- A `for` ciklus 1000-szer lefut.
- A `random.randint(0, 1)` ad egy 0-t vagy egy 1-et. Ha 1, akkor a `step` értéke 1 lesz, ha 0 (ami hamisként értékelődik ki), akkor -1.
- Hozzáadjuk a lépést az aktuális pozícióhoz, és az új pozíciót hozzáfűzzük a listához.

Ezt a listát ábrázolhatjuk egy grafikonon:

```python
In [255]: plt.plot(walk[:100])
```

> 📍 **[KÉP HELYE: Figure 4-4. A simple random walk]**

### A NumPy (Vektorizált) megoldás

Ha jobban belegondolsz, ez a bolyongás valójában nem más, mint a véletlenszerű lépések **kumulatív összege** (halmozott összege). A NumPy-val a 1000 lépést egyszerre (batch műveletként) legenerálhatjuk, elkerülve a lassú `for` ciklust!

```python
In [256]: nsteps = 1000
In [257]: rng = np.random.default_rng(seed=12345) # Új véletlenszám-generátor
In [258]: draws = rng.integers(0, 2, size=nsteps)
In [259]: steps = np.where(draws == 0, 1, -1)
In [260]: walk = steps.cumsum()
```

**Sorról sorra történő magyarázat:**

- `rng = ...`: Létrehozunk egy generátort egy fix kezdőértékkel (seed), hogy az eredmény reprodukálható legyen.
- `draws = rng.integers(0, 2, size=nsteps)`: Egyszerre legenerálunk 1000 darab véletlen számot (0-t vagy 1-et). Emlékezz: az felső határ (2) már nincs benne!
- `steps = np.where(draws == 0, 1, -1)`: Ahol a `draws` értéke 0, ott a lépés 1 lesz, minden más esetben (tehát ha 1) -1 lesz. Ez megadja az 1000 darab lépést egyetlen tömbben.
- `walk = steps.cumsum()`: A `cumsum()` (halmozott összeg) kiszámítja a bolyongás teljes útvonalát minden egyes pontban. (Pl. lépések: `[1, 1, -1]`, útvonal: `[1, 2, 1]`).

Ebből a tömbből azonnal lekérhetjük a legalacsonyabb és legmagasabb pontot:

```python
In [261]: walk.min()
Out[261]: -8

In [262]: walk.max()
Out[262]: 50
```

**Keresztezési idő (First crossing time) kiszámítása:**
Kíváncsiak vagyunk, mikor (hányadik lépésben) távolodott el a bolyongás először legalább 10 lépésnyire a nullától (akár pozitív, akár negatív irányban).

```python
In [263]: (np.abs(walk) >= 10).argmax()
Out[263]: 155
```

**Magyarázat:**

- Az `np.abs(walk) >= 10` veszi az útvonal abszolút értékét, és létrehoz egy logikai (`True`/`False`) tömböt, ahol `True` van ott, ahol elértük vagy túlléptük a 10-et.
- Az `.argmax()` normál esetben a legnagyobb érték indexét adja vissza. Mivel a logikai tömbben a `True` a legnagyobb érték (1), az `argmax` visszadja az első `True` érték indexét, ami pontosan az az időpont, amit keresünk! (Megjegyzés: Ez a módszer végigpásztázza az egész tömböt, így bár nagyon elegáns, memória- és CPU-igényes is lehet).

### Sok bolyongás szimulálása egyszerre (2D tömbökkel)

Mi van, ha nem 1, hanem mondjuk 5000 különböző véletlenszerű bolyongást szeretnénk szimulálni, mindegyiket 1000 lépéssel? Ha átadunk egy tuplet a `size` paraméternek, a NumPy azonnal legenerálja az egészet egy 2D-s mátrixban!

```python
In [264]: nwalks = 5000
In [265]: nsteps = 1000
In [266]: draws = rng.integers(0, 2, size=(nwalks, nsteps)) # 0 vagy 1
In [267]: steps = np.where(draws > 0, 1, -1)
In [268]: walks = steps.cumsum(axis=1)
In [269]: walks
Out[269]: 
array([[  1,   2,   3, ...,  22,  23,  22],
       [  1,   0,  -1, ..., -50, -49, -48],
       [  1,   2,   3, ...,  50,  49,  48],
       ...,
       [ -1,  -2,  -1, ..., -10,  -9, -10],
       [ -1,  -2,  -3, ...,   8,   9,   8],
       [ -1,   0,   1, ...,  -4,  -3,  -2]])
```

**Magyarázat:** A logika teljesen ugyanaz, de a `draws` most egy 5000x1000-es mátrix. A trükk a `cumsum(axis=1)`: ez megmondja a NumPy-nak, hogy a halmozott összeget soronként (az 1-es tengely mentén) számolja ki, azaz mind az 5000 bolyongást külön-külön értékelje ki!

Megkereshetjük az összes bolyongás (tehát az 5 millió lépés) abszolút minimumát és maximumát:

```python
In [270]: walks.max()
Out[270]: 114

In [271]: walks.min()
Out[271]: -120
```

**Keresztezési idő kiszámítása több bolyongásnál:**
Keressük meg, hogy ezek a bolyongások átlagosan mikor érték el a 30-as (vagy -30-as) határt. De vigyázat: nem mindegyik bolyongás ért el 30-ig! Először ki kell szűrnünk azokat, amelyek egyáltalán elérték.

```python
In [272]: hits30 = (np.abs(walks) >= 30).any(axis=1)
In [273]: hits30
Out[273]: array([False,  True,  True, ...,  True, False,  True])

In [274]: hits30.sum() # Hány darab érte el a 30-at?
Out[274]: 3395
```

**Magyarázat:** Az `.any(axis=1)` megnézi, hogy soronként (bolyongásonként) van-e legalább egy `True` (azaz elérte-e a 30-at). A 5000-ből 3395-ször történt ez meg.

Most használjuk ezt a logikai maszkot (`hits30`), hogy kinyerjük ezeket a bolyongásokat, és kiszámoljuk a keresztezési időket:

```python
In [275]: crossing_times = (np.abs(walks[hits30]) >= 30).argmax(axis=1)
In [276]: crossing_times
Out[276]: array([201, 491, 283, ..., 219, 259, 541])

In [277]: crossing_times.mean()
Out[277]: 500.5699558173785
```

**Magyarázat:**

- `walks[hits30]`: Kivágja azt a 3395 sort a mátrixból, ami sikeresen elérte a határt.
- `>= 30`: Újra megnézzük, hol lépik át a határt.
- `.argmax(axis=1)`: Soronként kikeresi az **ELSŐ** olyan indexet (lépést), ahol átlépték a 30-at.
- A legvégén a `.mean()` veszi ennek a 3395 számnak az átlagát.

Ha nem csak +1 / -1 lépéseket akarunk, hanem mondjuk normál eloszlású folytonos értékeket, azt is könnyedén megtehetjük:

```python
In [278]: draws = 0.25 * rng.standard_normal((nwalks, nsteps))
```

> **Fontos mérnöki megjegyzés a memóriáról:**
> Ez a vektorizált megoldás elképesztően gyors, de cserébe létrehoz a memóriában egy `nwalks * nsteps` méretű mátrixot. 5000 x 1000 = 5 millió elemű tömb még könnyedén elfér a gépben, de mondjuk milliárdos nagyságrendű szimulációknál (ahol a RAM szűk keresztmetszet) más, memória-takarékosabb iteratív megközelítést kellene alkalmazni.

## 4.8 Összegzés (Conclusion)

Ezzel befejeztük a NumPy alapok feldolgozását. A könyv további részeiben a hangsúly a **pandas** könyvtáron lesz, hogy adatokat tudjunk betölteni, tisztítani és formázni. Ugyanakkor a pandas mélyen a NumPy tömb-alapú logikájára (vektorizáció, logikai indexelés) épül, így ez a most megszerzett gondolkodásmód végig elengedhetetlen lesz.
