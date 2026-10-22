# 8. Fejezet: Data Wrangling (Adatelőkészítés): Összekapcsolás, Kombinálás és Átalakítás

## 8.1 Hierarchikus Indexelés (Hierarchical Indexing)

**Fő koncepció:** A hierarchikus indexelés lehetővé teszi, hogy egy tengelyen (sorokban vagy oszlopokban) több (kettő vagy több) index szint (level) legyen. Ez egy zseniális trükk arra, hogy magasabb dimenziós adatokat (pl. 3D vagy 4D adatokat) egy alacsonyabb dimenziós, lapos (pl. 1D Series vagy 2D DataFrame) formában kezelj!

Vegyünk egy példát: hozzunk létre egy Series-t úgy, hogy az indexnek egy listákból álló listát adunk át.

```python
In [11]: data = pd.Series(np.random.uniform(size=9),
   ....:                  index=[["a", "a", "a", "b", "b", "c", "c", "d", "d"],
   ....:                         [1, 2, 3, 1, 3, 1, 2, 2, 3]])
In [12]: data
Out[12]: 
a  1    0.929616
   2    0.316376
   3    0.183919
b  1    0.204560
   3    0.567725
c  1    0.595545
   2    0.964515
d  2    0.653177
   3    0.748907
dtype: float64
```

**Magyarázat:** Amit itt látsz, az egy MultiIndex-szel rendelkező Series "csinosított" nézete. Ahol "lyukat" (üres helyet) látsz a bal szélő oszlopban (pl. a 2-es és 3-as szám mellett), az azt jelenti, hogy "vedd a közvetlenül felette lévő címkét" (az `a`-t).
Itt a külső szint az "a, b, c, d", a belső szint az "1, 2, 3".

### Részleges indexelés (Partial indexing)

A hierarchikus indexelés igazi ereje abban rejlik, hogy nagyon tömören választhatjuk ki az adatok egy részhalmazát (partial indexing) a külső szintek alapján:

```python
# Csak a 'b' csoporthoz tartozó adatok kiválasztása:
In [14]: data["b"]
Out[14]: 
1    0.204560
3    0.567725
dtype: float64

# Szeletelés a külső szinten (inkluzív!):
In [15]: data["b":"c"]

# Választás a belső szintről (Minden olyan adat, ahol a belső index 2):
In [17]: data.loc[:, 2]
Out[17]: 
a    0.316376
c    0.964515
d    0.653177
dtype: float64
```

### Unstack és Stack (Átrendezés 2D-be)

A hierarchikus adatokat könnyedén szétteríthetjük (`unstack`) egy 2D-s DataFrame-mé, ahol a belső index-szintből oszlopok lesznek!

```python
In [18]: data.unstack()
Out[18]: 
          1         2         3
a  0.929616  0.316376  0.183919
b  0.204560       NaN  0.567725
c  0.595545  0.964515       NaN
d       NaN  0.653177  0.748907
```

(Természetesen ennek az ellentéte is létezik: az oszlopokat visszacsomagolhatjuk egy belső indexszintre a `.stack()` metódussal).

### Hierarchikus Indexelés a DataFrame-nél

DataFrame esetén a sorok (index) **ÉS** az oszlopok (columns) is rendelkezhetnek több szinttel! Még neveket is adhatunk ezeknek a szinteknek (ezt nem szabad összekeverni az egyes sorcímkékkel).

```python
In [20]: frame = pd.DataFrame(np.arange(12).reshape((4, 3)),
   ....:                      index=[["a", "a", "b", "b"], [1, 2, 1, 2]],
   ....:                      columns=[["Ohio", "Ohio", "Colorado"],
   ....:                               ["Green", "Red", "Green"]])

In [22]: frame.index.names = ["key1", "key2"]
In [23]: frame.columns.names = ["state", "color"]
In [24]: frame
Out[24]: 
state      Ohio     Colorado
color     Green Red    Green
key1 key2                   
a    1        0   1        2
     2        3   4        5
b    1        6   7        8
     2        9  10       11
```

**Magyarázat:** Az oszlopoknál a "state" (Ohio, Colorado) a legfelső szint, alatta van a "color". A soroknál a "key1" (a, b) a külső szint, alatta a "key2" (1, 2).

Oszlopok kiválasztása is működik részlegesen a legfelső szintről:

```python
In [26]: frame["Ohio"]
Out[26]: 
color      Green  Red
key1 key2            
a    1         0    1
     2         3    4
b    1         6    7
     2         9   10
```

### Szintek újrarendezése és rendezése (Reordering and Sorting Levels)

Néha a szintek rossz sorrendben vannak felépítve (pl. a belső szinttel akarsz csoportosítani). A `.swaplevel()` metódussal két megadott szintet felcserélhetsz egymással (az adatok értékei változatlanok maradnak, csak a címkék cserélnek helyet):

```python
In [27]: frame.swaplevel("key1", "key2")
Out[27]: 
state      Ohio     Colorado
color     Green Red    Green
key2 key1                   
1    a        0   1        2
2    a        3   4        5
1    b        6   7        8
2    b        9  10       11
```

A `sort_index()` alapértelmezetten a legkülső szinttől befelé haladva rendez ABC-sorrendbe. De megadhatod neki a `level=` paraméterrel, hogy csak egy konkrét belső szint alapján rendezze a táblázatot:

```python
In [28]: frame.sort_index(level=1)  # A belső (számos) szint alapján rendez
```

> **Fontos teljesítmény-tipp:** A hierarchikusan indexelt objektumokból történő adatlekérdezés (selection) sokkal gyorsabb, ha az index szigorúan lexikografikusan (ABC/növekvő) sorrendbe van rendezve a legkülső szinttől kezdve (pl. `sort_index(level=0)` használatával).

### Összegző Statisztikák Szintek (Levels) Alapján

A DataFrame-ek (és Series-ek) leíró statisztikáinál (mint a `.sum()`, `.mean()`) megadhatod a `level` opciót. Ez a háttérben egy `groupby` (csoportosítás) műveletet hajt végre a megadott szinten.

```python
# Sorok szerinti összegzés a belső (key2) szint alapján:
In [30]: frame.groupby(level="key2").sum()
Out[30]: 
state  Ohio     Colorado
color Green Red    Green
key2                    
1         6   8       10
2        12  14       16

# Oszlopok szerinti összegzés a belső (color) szint alapján:
In [31]: frame.groupby(level="color", axis="columns").sum()
Out[31]: 
color      Green  Red
key1 key2            
a    1         2    1
     2         8    4
b    1        14    7
     2        20   10
```

### Indexelés DataFrame oszlopokkal (set_index és reset_index)

Nagyon gyakori feladat adatelőkészítésnél, hogy egy beolvasott táblázat bizonyos adat-oszlopait akarjuk áthelyezni az indexbe (sor-címkévé), vagy épp fordítva: az indexet akarjuk normál oszloppá visszaminősíteni.

**Oszlopból Index (`set_index`):**
A `set_index` új DataFrame-et hoz létre, ahol a megadott oszlop(ok) hierarchikus indexszé válnak.

```python
In [32]: frame = pd.DataFrame({"a": range(7), "b": range(7, 0, -1),
   ....:                       "c": ["one", "one", "one", "two", "two", "two", "two"],
   ....:                       "d": [0, 1, 2, 0, 1, 2, 3]})
In [33]: frame
Out[33]: 
   a  b    c  d
0  0  7  one  0
1  1  6  one  1
...

In [34]: frame2 = frame.set_index(["c", "d"])
In [35]: frame2
Out[35]: 
       a  b
c   d      
one 0  0  7
    1  1  6
    2  2  5
two 0  3  4
    1  4  3
...
```

(Alapértelmezetten a felhasznált oszlopokat a gép törli az adatmezőkből. Ha mégis bent akarod tartani őket, használd a `drop=False` paramétert).

**Indexből Oszlop (`reset_index`):**
Ez pontosan a `set_index` fordítottja. A hierarchikus (vagy sima) index szinteket fogja, és átteszi őket normál adatoszlopokként a táblázatba, a sor-címkéket pedig visszaállítja egy sima 0, 1, 2... sorszámozásra.

```python
In [37]: frame2.reset_index()
Out[37]: 
     c  d  a  b
0  one  0  0  7
1  one  1  1  6
2  one  2  2  5
3  two  0  3  4
...
```

## Adathalmazok kombinálása és egyesítése (Combining and Merging)

Gyakori, hogy a munkánkhoz szükséges adatok több különböző táblázatban vannak szétszórva. A pandas három fő módszert biztosít ezek egyesítésére:

- **`pandas.merge`:** Sorok összekapcsolása (join) egy vagy több közös kulcs alapján (mint az SQL-ben).
- **`pandas.concat`:** Objektumok egymás mellé vagy egymás alá fűzése (stacking) egy tengely mentén.
- **`combine_first`:** Hiányzó adatok (lyukak) kitöltése egy táblában egy másik tábla értékeivel (splicing).

### Adatbázis-stílusú DataFrame egyesítések (Joins / pd.merge)

A `pd.merge` a fő belépési pont, ha relációs adatbázisokhoz hasonló join műveleteket akarsz végezni.

**Egyszerű példa (Több az egyhez / Many-to-one kapcsolat):**
Van egy fő táblánk (`df1`), ahol az "a" és "b" kulcsok többször is szerepelnek, és egy szótártáblánk (`df2`), ahol minden kulcs csak egyszer szerepel.

```python
In [38]: df1 = pd.DataFrame({"key": ["b", "b", "a", "c", "a", "a", "b"],
   ....:                     "data1": pd.Series(range(7), dtype="Int64")})
In [39]: df2 = pd.DataFrame({"key": ["a", "b", "d"],
   ....:                     "data2": pd.Series(range(3), dtype="Int64")})

In [43]: pd.merge(df1, df2, on="key")
Out[43]: 
  key  data1  data2
0   b      0      1
1   b      1      1
2   b      6      1
3   a      2      0
4   a      4      0
5   a      5      0
```

**Magyarázat:** A pandas összekapcsolta a két táblát a "key" oszlop alapján. Figyeld meg, hogy a "c" és a "d" kulcsok eltűntek! Ennek oka, hogy a `.merge()` alapértelmezetten **BELSŐ (INNER) JOIN**-t csinál, azaz csak azokat a sorokat tartja meg, amiknek a kulcsa mindkét táblázatban megtalálható (metszet).

**Eltérő oszlopnevek megadása:**
Ha a két táblázatban más néven hívják a közös kulcsot (pl. `lkey` és `rkey`), a `left_on` és `right_on` paramétereket kell használni:

```python
In [46]: pd.merge(df3, df4, left_on="lkey", right_on="rkey")
```

**Külső (OUTER) JOIN használata:**
Ha nem akarunk elveszíteni semmit (tehát azokat a kulcsokat is látni akarjuk, amik csak az egyik táblában vannak meg), akkor a `how="outer"` opciót használjuk. Ami nem egyezik, amellé `NaN` / `<NA>` kerül.

```python
In [47]: pd.merge(df1, df2, how="outer")
Out[47]: 
  key  data1  data2
0   b      0      1
...
6   c      3   <NA>
7   d   <NA>      2
```

### Table 8-1. Join típusok a how argumentummal

| Opció | Viselkedés (Behavior) |
| :--- | :--- |
| `how="inner"` | Alapértelmezett. Csak azokat a kulcsokat használja, amik **MIND KÉT** táblában megvannak (Metszet). |
| `how="left"` | A bal oldali tábla **MINDEN** kulcsát megtartja. |
| `how="right"` | A jobb oldali tábla **MINDEN** kulcsát megtartja. |
| `how="outer"` | A két tábla összes létező kulcsát megtartja (Unió). |

**Több a többhöz (Many-to-many) egyesítés:**
Ha a kulcs mindkét táblázatban többször is szerepel (pl. a bal oldalon 3-szor van "b", a jobb oldalon 2-szer van "b"), akkor a pandas a **Descartes-szorzatot** (Cartesian product) veszi: $3 \times 2 = 6$ darab "b" sor lesz a végeredményben!

**Több oszlopos (kompozit) kulcs:**
Ha az azonosításhoz több oszlop kell egyszerre (pl. Vezetéknév + Keresztnév), adj át egy listát az `on` paraméternek:

```python
In [57]: pd.merge(left, right, on=["key1", "key2"], how="outer")
```

**Ütköző (azonos nevű) oszlopok átnevezése:**
Ha két táblázatnak vannak olyan azonos nevű oszlopai, amik nem az összekapcsolási kulcsok, a pandas automatikusan egy `_x` és `_y` utótagot ragaszt a nevükre a végeredményben. Ezt felülírhatod a `suffixes` paraméterrel:

```python
In [59]: pd.merge(left, right, on="key1", suffixes=("_left", "_right"))
```

### Table 8-2. A pandas.merge függvény argumentumai

| Argumentum | Leírás (Description) |
| :--- | :--- |
| `left` / `right` | A bal és a jobb oldalon lévő (egyesítendő) DataFrame-ek. |
| `how` | A join típusa: `"inner"`, `"outer"`, `"left"`, vagy `"right"`. Alapból `"inner"`. |
| `on` | Az összekapcsoláshoz használt oszlopnév(ek). Mindkét táblában benne kell lennie. |
| `left_on` / `right_on` | Ha az oszlopnevek eltérnek a bal és jobb táblában, itt lehet őket külön megadni. |
| `left_index` / `right_index` | Ha `True`, a tábla normál adatoszlopa helyett a sor-címkéit (indexét) használja kulcsként. |
| `sort` | Ha `True`, a végeredményt ABC sorrendbe rendezi a kulcsok alapján (alapból `False`). |
| `suffixes` | Tuple (pl. `("_x", "_y")`), amit a gép a nem-kulcs, de azonos nevű oszlopok végére ragaszt. |
| `indicator` | Ha `True`, hozzáad egy `_merge` nevű extra oszlopot a táblához, ami megmondja, hogy a sor adatai honnan származnak (`left_only`, `right_only`, vagy `both`). |

### Egyesítés az Index alapján (Merging on Index)

Néha a kulcs, amivel összekötöznéd a két táblát, az egyik táblázatban normál oszlop, a másikban viszont a táblázat indexe (sor-címkéje). Ilyenkor a `right_index=True` (vagy `left_index=True`) paraméterrel utasíthatod a `.merge()`-t, hogy az indexben keresse a kulcsokat.

```python
# A left1 táblának van egy "key" nevű normál oszlopa (értékek: a, b, c)
# A right1 táblának a sor-indexei (címkéi) az "a" és "b"

In [64]: pd.merge(left1, right1, left_on="key", right_index=True)
Out[64]: 
  key  value  group_val
0   a      0        3.5
2   a      2        3.5
3   a      3        3.5
1   b      1        7.0
4   b      4        7.0
```

> **Tipp:** Mivel ez egy many-to-one kapcsolat volt – és a jobboldali index egyedi –, a pandas megtartotta a bal oldali táblázat eredeti sor-indexeit (0, 2, 3, 1, 4), nem dobta el őket!

Ha mindkét táblázatnál az index alapján akarsz egyesíteni, egyszerűen írd be, hogy `left_index=True, right_index=True`.

### A `.join()` metódus: A kényelmes rövidítés

Mivel az index alapján történő egyesítés nagyon gyakori, a DataFrame-ek rendelkeznek egy kényelmi `.join()` metódussal.
A `.join()` alapértelmezetten a két tábla indexei alapján végez **LEFT JOIN**-t (ellentétben a `pd.merge` INNER JOIN-jával)!

```python
In [78]: left2.join(right2, how="outer")
```

Sőt, a `.join()`-nak akár egy DataFrame-ekből álló listát is átadhatsz, és ő mindet szépen összekapcsolja (feltéve, hogy a tábláknak azonosak az indexei, de nincsenek bennük ütköző nevű oszlopok).

```python
In [82]: left2.join([right2, another])
```
## Összefűzés (Concatenation / Stacking) egy tengely mentén

A `pd.merge` a közös kulcsok (azonosítók) alapján kapcsolja össze az adatokat. A `pd.concat` ezzel szemben fizikailag "egymáshoz ragasztja" vagy egymásra pakolja a táblázatokat, anélkül, hogy megpróbálná a sorok tartalmát összepárosítani.

### Összefűzés Series-eknél

Vegyünk három Series-t, amelyeknek nincs közös indexük.

```python
In [87]: s1 = pd.Series([0, 1], index=["a", "b"], dtype="Int64")
In [88]: s2 = pd.Series([2, 3, 4], index=["c", "d", "e"], dtype="Int64")
In [89]: s3 = pd.Series([5, 6], index=["f", "g"], dtype="Int64")
```

Ha átadod őket egy listában a `pd.concat`-nak, alapértelmezetten (az `axis="index"` mentén) egyetlen hosszú Series-t csinál belőlük (egymás alá fűzi őket):

```python
In [93]: pd.concat([s1, s2, s3])
Out[93]: 
a    0
b    1
c    2
d    3
e    4
f    5
g    6
dtype: Int64
```

Ha azt mondod neki, hogy `axis="columns"`, akkor nem egymás alá, hanem egymás mellé rakja őket, amiből így egy DataFrame lesz. (Ami nem létezett az adott sorban, abból `NaN` / üres hely lesz).

```python
In [94]: pd.concat([s1, s2, s3], axis="columns")
Out[94]: 
      0     1     2
a     0  <NA>  <NA>
b     1  <NA>  <NA>
c  <NA>     2  <NA>
d  <NA>     3  <NA>
e  <NA>     4  <NA>
f  <NA>  <NA>     5
g  <NA>  <NA>     6
```

Ha csak azokat a sorokat akarod megtartani, amik minden összefűzött objektumban léteznek (metszet), használd a `join="inner"` paramétert:

```python
In [96]: s4 = pd.concat([s1, s3]) # 'a', 'b', 'f', 'g'
In [98]: pd.concat([s1, s4], axis="columns", join="inner")
Out[98]: 
   0  1
a  0  0
b  1  1
# Az 'f' és 'g' eltűnt, mert az 's1'-ben nem voltak benne!
```

### Az eredeti forrás azonosítása (keys)

Miután összefűztél 5 különböző táblázatot egymás alá, jó lenne tudni, hogy egy adott sor melyik eredeti táblázatból jött. Ezt a `keys` paraméterrel oldhatod meg, ami létrehoz egy hierarchikus (külső) indexet:

```python
In [99]: result = pd.concat([s1, s1, s3], keys=["one", "two", "three"])
In [100]: result
Out[100]: 
one    a    0
       b    1
two    a    0
       b    1
three  f    5
       g    6
dtype: Int64
```

(Ha oszlopok mentén (`axis="columns"`) fűzöl össze, akkor ezek a "kulcsok" az új DataFrame oszlopnevei lesznek).

### Összefűzés DataFrame-eknél

A fenti logika tökéletesen működik DataFrame-ekre is.

```python
In [103]: df1 = pd.DataFrame(np.arange(6).reshape(3, 2), index=["a", "b", "c"], columns=["one", "two"])
In [104]: df2 = pd.DataFrame(5 + np.arange(4).reshape(2, 2), index=["a", "c"], columns=["three", "four"])

In [107]: pd.concat([df1, df2], axis="columns", keys=["level1", "level2"])
Out[107]: 
  level1      level2     
     one  two  three  four
a      0    1    5.0   6.0
b      2    3    NaN   NaN
c      4    5    7.0   8.0
```

> **Tipp:** Lista helyett egy Python szótárat (`{"level1": df1, "level2": df2}`) is átadhatsz, a pandas automatikusan használni fogja a szótárkulcsokat a hierarchikus indexhez.

### Index eldobása (ignore_index)

Ha a táblázatok sor-indexeiben (pl. 0, 1, 2) semmilyen hasznos információ nincs, és egymás alá akarod őket másolni, használd az `ignore_index=True` paramétert. Így a pandas nem tartja meg a régi indexeket, hanem generál egy teljesen új, egységes 0, 1, 2... sorszámozást a végeredménynek.

### Table 8-3. A pandas.concat függvény argumentumai

| Argumentum | Leírás (Description) |
| :--- | :--- |
| `objs` | A pandas objektumok (Series vagy DataFrame) listája vagy szótára. Ez az egyetlen kötelező argumentum. |
| `axis` | A tengely, ami mentén ragasztunk. Alapból `axis="index"` (egymás alá). |
| `join` | `"outer"` (alapértelmezett, unió) vagy `"inner"` (metszet) a másik tengelyen lévő indexek kezelésére. |
| `keys` | Értékek, amiket az egyes eredeti objektumokhoz rendel (hierarchikus indexként). |
| `levels` | Ha `keys` van megadva, konkrét hierarchikus szinteket lehet vele definiálni. |
| `names` | Nevek a létrehozott hierarchikus index-szintek számára (ha van `keys` vagy `levels`). |
| `verify_integrity` | Ha `True`, megvizsgálja, hogy az új tengelyen lettek-e duplikált címkék, és hibát dob, ha igen (alapból `False`). |
| `ignore_index` | Ha `True`, eldobja a régi indexeket, és újat generál (0-tól a teljes hosszig). |

### Átfedő adatok kombinálása (Folt-hátán-folt / combine_first)

Van egy nagyon specifikus probléma, amit sem a `merge`, sem a `concat` nem tud megoldani: Képzeld el, hogy van két táblázatod. Az elsőben (`df1`) sok adat hiányzik (`NaN`). A második táblázatban (`df2`) szintén vannak adatok, néhol ugyanarra a dologra (ugyanazon az indexen).
A feladat: Készíts egy táblázatot a `df1` adataiból, de ahol a `df1`-ben `NaN` van, azt "foltozd be" a `df2`-ben lévő adatokkal!

Hagyományosan ezt a NumPy `np.where(pd.isna(a), b, a)` megoldásával lehetne, de az nem ellenőrzi az indexeket. Erre való a `.combine_first()` metódus!

```python
In [115]: a = pd.Series([np.nan, 2.5, 0.0, 3.5, 4.5, np.nan], index=["f", "e", "d", "c", "b", "a"])
In [116]: b = pd.Series([0., np.nan, 2., np.nan, np.nan, 5.], index=["a", "b", "c", "d", "e", "f"])

In [120]: a.combine_first(b)
Out[120]: 
a    0.0  # Az 'a'-ban NaN volt, így átvette a 'b' táblából a 0.0-t.
b    4.5  # Az 'a'-ban volt érték (4.5), így megtartotta azt.
c    3.5  # Az 'a'-ban volt érték (3.5).
d    0.0  # Az 'a'-ban volt érték (0.0).
e    2.5  # Az 'a'-ban volt érték (2.5).
f    5.0  # Az 'a'-ban NaN volt, így átvette a 'b'-ből az 5.0-t.
dtype: float64
```

DataFrame-ek esetén a `combine_first` oszlopról oszlopra csinálja meg ugyanezt:

```python
In [123]: df1
Out[123]: 
     a    b   c
0  1.0  NaN   2
1  NaN  2.0   6
2  5.0  NaN  10
3  NaN  6.0  14

In [124]: df2
Out[124]: 
     a    b
0  5.0  NaN
1  4.0  3.0
2  NaN  4.0
3  3.0  6.0
4  7.0  8.0

In [125]: df1.combine_first(df2)
Out[125]: 
     a    b     c
0  1.0  NaN   2.0
1  4.0  2.0   6.0
2  5.0  4.0  10.0
3  3.0  6.0  14.0
4  7.0  8.0   NaN
```

**Magyarázat:**

- A 0. sorban a "b" oszlop mindkét táblában `NaN` volt, így az is maradt.
- Az 1. sorban az "a" oszlop a `df1`-ben `NaN` volt, ezért megkapta a `df2`-ből a 4.0-t.
- A 4. sor egyáltalán nem is létezett a `df1`-ben, de a `.combine_first()` kibővítette a táblázatot (uniót csinált az indexekből és az oszlopnevekből), és átemelte az adatokat a `df2`-ből. (Ahol a `df2`-nek nem volt "c" oszlopa, ott természetesen `NaN` lett).

- ## Átalakítás és Pivotálás (Reshaping and Pivoting)

Az adatok elrendezését gyakran kell átszervezni attól függően, hogy milyen elemzést vagy vizualizációt szeretnénk csinálni rajtuk. Erre a pandas a hierarchikus indexelésre építve két alapvető műveletet kínál: a `stack`-et és az `unstack`-et.

### Átalakítás Hierarchikus Indexeléssel (Stack / Unstack)

Képzeljünk el egy 2D-s DataFrame-et (táblázatot), aminek a sorai államok ("Ohio", "Colorado"), az oszlopai pedig számok ("one", "two", "three").

```python
In [127]: data
Out[127]: 
number    one  two  three
state                    
Ohio        0    1      2
Colorado    3    4      5
```

#### 1. `stack()`: "Oszlopokból Sorok"

A `stack` (egymásra pakolás) "bepörgeti" a táblázat oszlopait a sorokba, mint egy új, belső hierarchikus index-szintet, és ezzel az egész 2D-s táblázatot egy nagyon hosszú 1D-s Series-zé alakítja.

```python
In [128]: result = data.stack()
In [129]: result
Out[129]: 
state     number
Ohio      one       0
          two       1
          three     2
Colorado  one       3
          two       4
          three     5
dtype: int64
```

#### 2. `unstack()`: "Sorokból Oszlopok"

Ez a `stack` pontos ellentéte. Fog egy hierarchikus indexet (sor-címkét), és "kiteríti" oszlopokká. Alapértelmezetten mindig a legbelső szintet (itt a `number`-t) teríti ki, de megmondhatod neki a szint nevét (vagy sorszámát), ha a külsőt akarod oszloppá tenni:

```python
# Alapértelmezett (visszaadja az eredeti táblázatot):
In [130]: result.unstack()

# A külső ('state') szint kiterítése (Ebből lesznek az oszlopok):
In [132]: result.unstack(level="state")
Out[132]: 
state   Ohio  Colorado
number                
one        0         3
two        1         4
three      2         5
```

> **Tipp:** Az `unstack` művelet sokszor bevezethet `NaN` hiányzó értékeket a táblázatba, ha az adathalmaz "hiányos" volt, tehát nem volt meg minden lehetséges sor-oszlop kombináció.

### Pivotálás "Hosszúból" "Széles" Formátumba (`pivot`)

Sok adatbázisban és CSV fájlban a többes idősorokat (pl. gazdasági mutatókat) úgynevezett **hosszú (long)** vagy **halmozott (stacked)** formátumban tárolják.
Ebben a formátumban nincs külön oszlop az inflációnak vagy a GDP-nek. Ehelyett van egy `date` (dátum) oszlop, egy `item` (típus) oszlop, és egyetlen `value` (érték) oszlop.

```python
In [155]: long_data[:5]
Out[155]: 
        date     item     value
0 1959-01-01  realgdp  2710.349
1 1959-01-01     infl     0.000
2 1959-01-01    unemp     5.800
3 1959-04-01  realgdp  2778.801
4 1959-04-01     infl     2.340
```

Adatbázisokhoz ez tökéletes, de elemzéshez nagyon kényelmetlen. Sokkal jobb lenne, ha lenne külön "realgdp", "infl" és "unemp" oszlopunk. Pontosan ezt csinálja a `.pivot()` metódus!

```python
In [156]: pivoted = long_data.pivot(index="date", columns="item", values="value")
In [157]: pivoted.head()
Out[157]: 
item        infl   realgdp  unemp
date                             
1959-01-01  0.00  2710.349    5.8
1959-04-01  2.34  2778.801    5.1
```

**Magyarázat paraméterenként:**

- `index="date"`: Ez az oszlop lesz az új, "széles" táblázatunk sor-címkéje (az Y tengely).
- `columns="item"`: Ezekből az értékekből jönnek létre az új oszlopnevek (az X tengely).
- `values="value"`: Ebből az oszlopból veszi ki a gép a konkrét számokat, amik a táblázat celláiba kerülnek.

> **Megjegyzés:** A `.pivot(...)` valójában egy kényelmi rövidítés arra, ha csinálnánk egy `.set_index(["date", "item"])`-et, majd rögtön utána egy `.unstack("item")`-et.

### Pivotálás "Szélesből" "Hosszú" Formátumba (`melt`)

A `pivot` pontos ellentéte (inverze) a `pd.melt()`. Ahelyett, hogy egyetlen oszlopból csinálna sokat (szélesítené a táblát), több oszlopot (és a bennük lévő adatokat) "beleolvaszt" két új, szűk oszlopba (`variable` és `value`), így a táblázat sokkal hosszabb lesz.

Képzeljük el ezt a "széles" DataFrame-et:

```python
In [166]: df = pd.DataFrame({"key": ["foo", "bar", "baz"],
   .....:                    "A": [1, 2, 3],
   .....:                    "B": [4, 5, 6],
   .....:                    "C": [7, 8, 9]})
In [167]: df
Out[167]: 
   key  A  B  C
0  foo  1  4  7
1  bar  2  5  8
2  baz  3  6  9
```

Az "A", "B" és "C" csak adat-oszlopok, de a "key" egy csoport-azonosító. Amikor elolvasztjuk (`melt`) a táblát, meg kell mondanunk, hogy melyik a csoport-azonosító (a horgony).

```python
In [168]: melted = pd.melt(df, id_vars="key")
In [169]: melted
Out[169]: 
   key variable  value
0  foo        A      1
1  bar        A      2
2  baz        A      3
3  foo        B      4
4  bar        B      5
5  baz        B      6
6  foo        C      7
7  bar        C      8
8  baz        C      9
```

**Magyarázat:** A gép megtartotta a "key" oszlopot (`id_vars`), a többi (A, B, C) oszlop nevét bepakolta egy új, hosszú `variable` oszlopba, az oszlopokban lévő számokat pedig a mellette lévő `value` oszlopba tette. Az eredeti 3 sorból egy 9 soros "hosszú" táblázat lett!

Később ebből a "melted" (megolvasztott) táblázatból egy `pivot`-tal pillanatok alatt vissza lehet kapni az eredeti "széles" formát:

```python
In [170]: reshaped = melted.pivot(index="key", columns="variable", values="value")
# Utána egy reshaped.reset_index() teljesen visszaadja az eredeti 'df'-et.
```

## 8.4 Összegzés (Conclusion)

Ezzel be is fejeztük a 8. fejezetet! Most már professzionálisan tudunk adatokat betölteni (load), tisztítani (clean), egyesíteni (merge/join), és átszervezni-forgatni (reshape/pivot) a pandas segítségével. Ezek a "Data Wrangling" készségek jelentik az adatelemzői munka alapját. A következő fejezetben az adatok vizualizációjával (ábrázolásával) fogunk megismerkedni a matplotlib könyvtár segítségével.
