# 7. Fejezet: Adattisztítás és Előkészítés

## 7.1 Hiányzó adatok kezelése (Handling Missing Data)

A hiányzó adatok nagyon gyakoriak a valós alkalmazásokban. A pandas egyik fő célja, hogy a hiányzó adatokkal való munkát a lehető legfájdalommentesebbé tegye. (Emlékezz: a leíró statisztikák, mint a `.mean()` és a `.sum()`, alapértelmezetten kizárják ezeket).

A pandas a numerikus (`float64`) adatoknál a `NaN` (Not a Number) lebegőpontos értéket használja a hiányzó adatok reprezentálására. Ezt nevezzük **sentinel** (jelző) értéknek.

```python
In [14]: float_data = pd.Series([1.2, -3.5, np.nan, 0])
In [15]: float_data
Out[15]: 
0    1.2
1   -3.5
2    NaN
3    0.0
dtype: float64
```

A pandas statisztikai konvenciók miatt ezeket az adatokat gyakran **NA**-nak (Not Available) nevezi. Fontos, hogy a beépített Python `None` értéket is automatikusan NA-ként (azaz `NaN`-ként) kezeli!

```python
In [17]: string_data = pd.Series(["aardvark", np.nan, None, "avocado"])
In [19]: string_data.isna()
Out[19]: 
0    False
1     True
2     True
3    False
dtype: bool
```

**Magyarázat:** A `.isna()` metódus (vagy a `pd.isna()`) egy logikai maszkot ad vissza, ami pontosan megmutatja, hol vannak a "lyukak" az adatainkban.

### Table 7-1. NA (Hiányzó adat) kezelő metódusok

| Metódus | Leírás (Description) |
| :--- | :--- |
| `dropna` | Kiszűri (eldobja) azokat a tengelycímkéket (sorokat vagy oszlopokat), amelyek hiányzó adatot tartalmaznak. A küszöbérték (hogy mennyi hiányzót toleráljon) állítható. |
| `fillna` | Kitölti a hiányzó adatokat egy megadott értékkel, vagy egy interpolációs módszerrel (pl. `"ffill"` - forward fill, vagy `"bfill"` - backward fill). |
| `isna` | Logikai (`True`/`False`) értékeket ad vissza, jelezve, hogy hol vannak a hiányzó (NA) értékek. |
| `notna` | Az `isna` tagadása (ellentéte): `True` a létező adatoknál, és `False` a hiányzóknál. |

### Hiányzó adatok kiszűrése (dropna)

Természetesen szűrhetsz manuálisan logikai indexeléssel is (`data[data.notna()]`), de a `.dropna()` sokkal kényelmesebb.

**Series esetén:** Egyszerűen eldobja az összes `NaN` értéket és az indexüket.

```python
In [23]: data = pd.Series([1, np.nan, 3.5, np.nan, 7])
In [24]: data.dropna()
Out[24]: 
0    1.0
2    3.5
4    7.0
dtype: float64
```

**DataFrame esetén (Sorok eldobása):**
Itt bonyolultabb a helyzet, mert nem dobhatsz el csak egyetlen cellát (akkor elcsúszna a táblázat). A `.dropna()` alapértelmezetten eldobja a teljes sort, ha akár csak egyetlen `NaN` is van benne!

```python
In [26]: data = pd.DataFrame([[1., 6.5, 3.], [1., np.nan, np.nan],
   ....:                      [np.nan, np.nan, np.nan], [np.nan, 6.5, 3.]])
In [28]: data.dropna()
Out[28]: 
     0    1    2
0  1.0  6.5  3.0
```

**Magyarázat:** Csak a 0. sor maradt meg, mert az összes többiben volt legalább egy `NaN`.

Ha csak azokat a sorokat akarod eldobni, amik teljesen üresek (minden oszlopuk `NaN`), használd a `how="all"` paramétert:

```python
In [29]: data.dropna(how="all")
Out[29]: 
     0    1    2
0  1.0  6.5  3.0
1  1.0  NaN  NaN
3  NaN  6.5  3.0
```

**DataFrame esetén (Oszlopok eldobása):**
Megadhatod az `axis="columns"` (vagy `axis=1`) paramétert, ha sorok helyett oszlopokat vizsgálsz.

```python
In [30]: data[4] = np.nan # Hozzáadunk egy teljesen üres oszlopot
In [32]: data.dropna(axis="columns", how="all")
# Kidobja a '4' nevű oszlopot, a többit megtartja.
```

**Küszöbérték megadása (`thresh`):**
Adatelemzésnél gyakran azt mondjuk: "Tarts meg minden sort, amiben van legalább N darab érvényes (nem hiányzó) adat". Ezt a `thresh` paraméterrel állíthatjuk be.

```python
In [36]: df
# 7 soros tábla, tele NaN-okkal

In [38]: df.dropna(thresh=2)
Out[38]: 
          0         1         2
2  0.092908       NaN  0.769023
3  1.246435       NaN -1.296221
4  0.274992  0.228913  1.352917
5  0.886429 -2.001637 -0.371843
6  1.669025 -0.438570 -0.539741
```

**Magyarázat:** A 0. és 1. sort kidobta, mert azokban csak 1-1 érvényes érték volt. A 2. sor megmaradt, mert abban pontosan 2 (a küszöb) érvényes érték van.

### Hiányzó adatok kitöltése (fillna)

Sokszor nem dobhatjuk ki az adatokat (mert túl sok infót veszítenénk). Ilyenkor be kell "tömnünk a lyukakat" (imputálás). A `.fillna()` erre a legfőbb eszköz.

**Kitöltés egyetlen konstanssal:**

```python
In [39]: df.fillna(0) # Minden NaN helyére 0 kerül
```

**Kitöltés oszloponként eltérő értékkel (Szótár átadása):**

```python
In [40]: df.fillna({1: 0.5, 2: 0})
```

**Magyarázat:** Az 1-es oszlopban lévő `NaN`-okat 0.5-re, a 2-es oszlopban lévőket 0-ra cseréli.

**Előre-kitöltés (Forward filling) interpolációval:**
Idősoroknál gyakori, hogy a hiányzó értéket az előző érvényes adattal töltjük ki (mintha "tovább élne" az előző állapot).

```python
In [45]: df.fillna(method="ffill")
```

Ha nem akarod, hogy egyetlen régi adat nagyon sokáig érvényes maradjon, limitálhatod a kitöltések számát a `limit=` paraméterrel:

```python
In [46]: df.fillna(method="ffill", limit=2)
```

**Magyarázat:** Maximum 2 egymást követő `NaN`-t hajlandó kitölteni az előző értékkel. Ha a 3. is `NaN`, az marad üresen.

**Kitöltés statisztikai értékkel (Átlag, Medián):**
Ez az egyik leggyakoribb egyszerű adat-imputációs módszer a gépi tanulásban.

```python
In [47]: data = pd.Series([1., np.nan, 3.5, np.nan, 7])
In [48]: data.fillna(data.mean())
Out[48]: 
0    1.000000
1    3.833333  # Az 1, 3.5 és 7 átlaga
2    3.500000
3    3.833333  # Szintén az átlag
4    7.000000
dtype: float64
```

### Table 7-2. A fillna függvény argumentumai

| Argumentum | Leírás (Description) |
| :--- | :--- |
| `value` | Skalár (konstans) érték vagy szótárszerű objektum (oszloponkénti leképezés), amivel a hiányzó értékeket kitöltjük. |
| `method` | Interpolációs metódus. `"bfill"` (hátrafelé kitöltés - backward fill) vagy `"ffill"` (előrefelé kitöltés - forward fill). Alapértelmezetten `None`. |
| `axis` | A tengely, amely mentén a kitöltés történik (`"index"` sorok mentén lefelé, vagy `"columns"` oszlopokon át). |
| `limit` | Interpoláció (előre/hátra) esetén a maximálisan kitölthető egymást követő (konzekutív) hiányzó elemek száma. |

## 7.2 Adat-transzformáció (Data Transformation)

A hiányzó adatok (`NaN`) kezelése után a második legfontosabb lépés az adatok szűrése, tisztítása és szerkezetük átalakítása.

### Másodpéldányok eltávolítása (Removing Duplicates)

Bármilyen okból kifolyólag előfordulhat, hogy egy DataFrame-ben azonos (duplikált) sorok találhatók.

```python
In [49]: data = pd.DataFrame({"k1": ["one", "two"] * 3 + ["two"],
   ....:                      "k2": [1, 1, 2, 3, 3, 4, 4]})
In [50]: data
Out[50]: 
    k1  k2
0  one   1
1  two   1
...
5  two   4
6  two   4
```

**Duplikációk ellenőrzése:** A `.duplicated()` egy logikai Series-t ad vissza, amelynél a `True` azt jelenti, hogy a sor teljesen megegyezik egy korábban már megjelent sorral (a 6. sor megegyezik az 5. sorral).

```python
In [51]: data.duplicated()
Out[51]: 
...
5    False
6     True
dtype: bool
```

**Duplikációk eldobása:** A `.drop_duplicates()` egy új DataFrame-et ad vissza, amiből ki vannak szűrve ezek a másodpéldányok.

```python
In [52]: data.drop_duplicates()
# A 6. sor eltűnik
```

Alapértelmezetten a metódus az összes oszlopot figyelembe veszi az összehasonlításnál. Ha csak bizonyos oszlopok alapján akarsz szűrni (pl. keresd meg, hol ismétlődik a `k1` kulcs, függetlenül a `k2` értékétől), használd a `subset` paramétert:

```python
In [55]: data.drop_duplicates(subset=["k1"])
Out[55]: 
    k1  k2  v1
0  one   1   0
1  two   1   1
```

**Magyarázat:** Csak a legelső `one` és a legelső `two` sor maradt meg.

Alapból a gép a legelső találatot tartja meg, és a többit eldobja. Ha a legutolsó találatot akarod megtartani, használd a `keep="last"` opciót:

```python
In [56]: data.drop_duplicates(["k1", "k2"], keep="last")
```

### Adatok transzformálása Függvénnyel vagy Leképezéssel (Mapping)

Gyakran előfordul, hogy egy oszlop értékei alapján egy új oszlopot akarsz létrehozni, ami a régi értékek valamilyen logikai leképezése.
Tegyük fel, van egy húsokat tartalmazó listánk, és mellé akarjuk írni, hogy az adott hús milyen állatból van.

**Készítünk egy szótárat (leképezést):**

```python
meat_to_animal = {
  "bacon": "pig",
  "pulled pork": "pig",
  "pastrami": "cow",
  "corned beef": "cow",
  "honey ham": "pig",
  "nova lox": "salmon"
}
```

A Series `.map()` metódusába bedobhatod ezt a szótárat, és ő automatikusan végigmegy az összes elemen, kikeresi a szótárból a párt, és beleteszi egy új Series-be:

```python
In [60]: data["animal"] = data["food"].map(meat_to_animal)
In [61]: data
Out[61]: 
          food  ounces  animal
0        bacon     4.0     pig
1  pulled pork     3.0     pig
2        bacon    12.0     pig
...
```

(Ha a leképezés bonyolultabb, szótár helyett átadhatsz a `.map()`-nek egy komplett `def` függvényt is).

### Értékek cseréje (Replacing Values)

A hiányzó adatok kitöltése (`fillna`) valójában az értékcsere egy speciális fajtája. Ha azonban általános értékeket akarsz cserélni (pl. egy hibakódot vagy egy placeholder-t `NaN`-ra), a `.replace()` sokkal rugalmasabb, mint a `map`.

```python
In [64]: data = pd.Series([1., -999., 2., -999., -1000., 3.])
```

Tegyük fel, hogy a `-999` a mérési hiba kódja, és a pandas által ismert `NaN`-ra akarjuk cserélni:

```python
In [66]: data.replace(-999, np.nan)
```

Több értéket is lecserélhetsz egyszerre (mindegyik `NaN` lesz):

```python
In [67]: data.replace([-999, -1000], np.nan)
```

Vagy adhatsz nekik külön-külön új értéket két listával, esetleg szótárral:

```python
# Két lista átadása (A -999-ből NaN lesz, a -1000-ből 0):
In [68]: data.replace([-999, -1000], [np.nan, 0])

# VAGY, ami sokkal elegánsabb, szótár átadása:
In [69]: data.replace({-999: np.nan, -1000: 0})
```

> **Tipp:** Ne keverd össze ezt a `data.str.replace`-szel, ami a szövegeken belüli (rész)szövegek cseréjére szolgál!

### Tengelycímkék (Indexek) átnevezése (Renaming Axis Indexes)

Ahogy a táblázat értékeit, úgy az indexeit és oszlopneveit is átalakíthatjuk.
Mivel az index objektumoknak is van `.map()` metódusa, rájuk húzhatunk egy függvényt (pl. ami mindent nagybetűssé tesz, és csak az első 4 karaktert tartja meg):

```python
In [71]: def transform(x):
   ....:     return x[:4].upper()

# Helyben felülírjuk az indexet:
In [73]: data.index = data.index.map(transform)
In [74]: data
Out[74]: 
      one  two  three  four
OHIO    0    1      2     3
COLO    4    5      6     7
NEW     8    9     10    11
```

Ha nem akarod felülírni az eredeti táblázatot, hanem egy új, átnevezett DataFrame-et szeretnél visszakapni (ami a leggyakoribb eset), a `.rename()` metódust érdemes használni:

```python
# FONTOS: A str.title minden szó első betűjét nagyra veszi, a str.upper csupa nagybetűt csinál
In [75]: data.rename(index=str.title, columns=str.upper)
Out[75]: 
      ONE  TWO  THREE  FOUR
Ohio    0    1      2     3
Colo    4    5      6     7
New     8    9     10    11
```

A `.rename()` arra is tökéletes, hogy csak bizonyos oszlopokat vagy sorokat nevezz át, szótár segítségével:

```python
In [76]: data.rename(index={"OHIO": "INDIANA"},
   ....:             columns={"three": "peekaboo"})
```

### Diszkretizálás és Binning (Kategorizálás)

Adatelemzésnél folytonos adatokat (pl. életkor, jövedelem, árak) gyakran diszkrét "vödrökbe" (bins) kell osztani. Például egy kísérlet résztvevőit korcsoportokba osztjuk:

```python
In [77]: ages = [20, 22, 25, 27, 21, 23, 37, 31, 61, 45, 41, 32]

# Megadjuk a vödrök HATÁRAIT (18-25-ig, 25-35-ig stb.)
In [78]: bins = [18, 25, 35, 60, 100]

# Elvégezzük a darabolást a pd.cut függvénnyel:
In [79]: age_categories = pd.cut(ages, bins)
```

A `pd.cut` egy speciális **Categorical** objektumot ad vissza. Matematikai zárójelek mutatják a kategóriákat: a `(18, 25]` azt jelenti, hogy 18 felett (nyílt, exkluzív), egészen 25-ig bezárólag (zárt, inkluzív).

```python
In [84]: pd.value_counts(age_categories)
Out[84]: 
(18, 25]     5
(25, 35]     3
(35, 60]     3
(60, 100]    1
```

(Ha a zárójelek irányát meg akarod fordítani, használd a `right=False` opciót).

Ha nem a csúnya matematikai zárójeleket akarod látni, saját címkéket is adhatsz a vödröknek:

```python
In [86]: group_names = ["Youth", "YoungAdult", "MiddleAged", "Senior"]
In [87]: pd.cut(ages, bins, labels=group_names)
Out[87]: 
['Youth', 'Youth', 'Youth', 'YoungAdult', 'Youth', ..., 'Senior', 'MiddleAged', 'MiddleAged', 'YoungAdult']
```

Ha nem te akarod megadni a határokat, hanem csak azt mondod, hogy "Darabold ezt az adatot 4 egyenlő hosszúságú (equal-length) részre!", adj át egy egész számot a határok helyett:

```python
In [89]: pd.cut(data, 4, precision=2)
```

**Kvantilis alapú binning (`pd.qcut`):**
A `cut` a tartományt egyenlő méretekre vágja (így lehet, hogy az egyik vödörbe 100 ember kerül, a másikba csak 1). A `qcut` viszont kvantiliseket használ, azaz garantálja, hogy minden vödörbe nagyjából ugyanannyi ember (adat) kerüljön.

```python
In [90]: data = np.random.standard_normal(1000)
In [91]: quartiles = pd.qcut(data, 4, precision=2) # 4 vödör (negyedek)

In [93]: pd.value_counts(quartiles)
Out[93]: 
(-2.96, -0.68]     250
(-0.68, -0.026]    250
(-0.026, 0.62]     250
(0.62, 3.93]       250
```

### Kiugró értékek (Outliers) felismerése és szűrése

Az outlierek szűrése nagyrészt a NumPy-tól megtanult logikai tömbműveletekből áll.
Van egy véletlenszámokkal teli 1000 soros, 4 oszlopos táblázatunk.
Keressük meg azokat a sorokat a 2-es nevű oszlopban, ahol az érték abszolút értéke meghaladja a 3-at!

```python
In [97]: col = data[2]
In [98]: col[col.abs() > 3]
Out[98]: 
41    -3.399312
136   -3.745356
Name: 2, dtype: float64
```

Most keressük meg az összes olyan sort a teljes táblázatban, ahol **BÁRMELYIK** oszlopban (`any`) van 3-nál nagyobb (vagy -3-nál kisebb) érték:

```python
# Logikai DataFrame-et ad (True/False): data.abs() > 3
# Ráhúzzuk az any(axis="columns") szűrőt: ha a sorban van legalább egy True, a sor marad.
In [99]: data[(data.abs() > 3).any(axis="columns")]
```

Ha ezeket az extrém értékeket nem kidobni akarjuk, hanem "le akarjuk vágni" (capping / clipping) mondjuk pontosan 3-ra és -3-ra (ez minimalizálja az outlierek hatását a statisztikákra):

```python
In [100]: data[data.abs() > 3] = np.sign(data) * 3
```

**Magyarázat:** A `np.sign(data)` egy mátrixot ad, ami 1, ha a szám pozitív, és -1, ha negatív. Szorozva 3-mal ez pontosan beállítja a limitet 3-ra vagy -3-ra, anélkül, hogy az előjel megváltozna.

## 7.2 Adat-transzformáció (Folytatás)

### Permutáció (Megkeverés) és Véletlenszerű Mintavételezés (Random Sampling)

Gyakran előfordul, hogy egy adathalmaz sorait véletlenszerűen össze kell kevernünk (pl. neurális hálózatok betanítása előtt), vagy egy véletlenszerű mintát kell vennünk belőle.

**Megkeverés (Permutation):**
A `numpy.random.permutation` függvény egy adott számhoz (vagy a DataFrame hosszához) legenerál egy véletlenszerű sorrendet. Ezt utána átadhatjuk az `iloc`-nak (vagy a `take` függvénynek), hogy átrendezze a táblázatot.

```python
In [103]: df = pd.DataFrame(np.arange(5 * 7).reshape((5, 7)))
In [105]: sampler = np.random.permutation(5)
In [106]: sampler
Out[106]: array([3, 1, 4, 2, 0])  # A véletlenszerű sorrend

# Átrendezés a 'sampler' alapján:
In [108]: df.iloc[sampler]
Out[108]: 
    0   1   2   3   4   5   6
3  21  22  23  24  25  26  27
1   7   8   9  10  11  12  13
4  28  29  30  31  32  33  34
2  14  15  16  17  18  19  20
0   0   1   2   3   4   5   6
```

**Közvetlen Mintavétel (Sampling):**
Sokkal egyszerűbb és gyorsabb a beépített `.sample()` metódust használni, amivel közvetlenül kérhetsz $N$ darab véletlen sort.

```python
# Visszatevés NÉLKÜLI (without replacement) húzás, azaz egy sor csak egyszer szerepelhet:
In [112]: df.sample(n=3)

# Visszatevéses (with replacement) húzás, ahol egy sor többször is kihúzható:
In [113]: choices = pd.Series([5, 7, -1, 6, 4])
In [114]: choices.sample(n=10, replace=True)
```

### Dummy (Indikátor) Változók Létrehozása Gépi Tanuláshoz

A gépi tanulási modellek (pl. lineáris regresszió) nem tudnak mit kezdeni a szöveges kategóriákkal (mint a "kutya", "macska", "madár"). Ezeket a kategóriákat át kell alakítani úgynevezett indikátor (dummy) mátrixszá. Ha egy oszlopban $k$ különböző érték van, akkor ebből $k$ darab új oszlop lesz, tele 0-kkal és 1-esekkel (ahol 1 jelenti, hogy az adott sor abba a kategóriába tartozik).
Ezt hívják **One-Hot Encoding**-nak is.

Erre a legjobb eszköz a `pd.get_dummies()`:

```python
In [115]: df = pd.DataFrame({"key": ["b", "b", "a", "c", "a", "b"], "data1": range(6)})
In [116]: df
Out[116]: 
  key  data1
0   b      0
1   b      1
2   a      2
3   c      3
4   a      4
5   b      5

In [117]: pd.get_dummies(df["key"])
Out[117]: 
   a  b  c
0  0  1  0
1  0  1  0
2  1  0  0
3  0  0  1
4  1  0  0
5  0  1  0
```

Gyakran érdemes a generált oszlopnevek elé egy előtagot (`prefix`) tenni, hogy később egyértelmű legyen, honnan jöttek, majd hozzáilleszteni (`join`) az eredeti adatokhoz:

```python
In [118]: dummies = pd.get_dummies(df["key"], prefix="key")
In [119]: df_with_dummy = df[["data1"]].join(dummies)
In [120]: df_with_dummy
Out[120]: 
   data1  key_a  key_b  key_c
0      0      0      1      0
1      1      0      1      0
2      2      1      0      0
```

**Több kategória egyetlen mezőben (pl. Film Műfajok):**
Trükkösebb a helyzet, ha egy cellában több kategória is van, egy határolóval (pl. `"Action|Adventure|Sci-Fi"`). Erre a pandas a `.str.get_dummies()` metódust biztosítja.

```python
# A MovieLens adatbázis 'genres' (műfaj) oszlopa: "Animation|Children's|Comedy"
In [124]: dummies = movies["genres"].str.get_dummies("|")
In [125]: dummies.iloc[:5, :6] # Az első 5 film és 6 műfaj megjelenítése
Out[125]: 
   Action  Adventure  Animation  Children's  Comedy  Crime
0       0          0          1           1       1      0
1       0          1          0           1       0      0
...
```

> **Tipp:** Nagyon nagy adathalmazoknál ez a módszer lelassulhat, olyankor érdemes közvetlenül NumPy tömbökre optimalizált függvényt írni.

Kombinálhatjuk a `get_dummies()`-t a korábban megismert `cut` függvénnyel is, hogy a folytonos adatainkat (pl. életkorokat) kategóriákba, majd 0/1 mátrixba konvertáljuk.

## 7.3 Kiterjesztett Adattípusok (Extension Data Types)

Bár ez haladóbb téma, nagyon fontos megérteni, mert a modern pandas fejlesztések erre épülnek.

Eredetileg a pandas a NumPy-ra épült, ami nagyszerű volt a számoknál, de a szövegekkel vagy a hiányzó (`NaN`) adatokkal rendelkező egész számokkal / logikai értékekkel szenvedett.
Ha egy `int` típusú oszlopban megjelent egy `NaN`, a pandas kénytelen volt az egész oszlopot `float64`-é alakítani, mert az `int` nem tudott `NaN`-t tárolni.

Ennek kiküszöbölésére a pandas létrehozta az **Extension (Kiterjesztett) Adattípusokat**. Ha ilyet használsz, a hiányzó értékeket a speciális `pd.NA` jelző (és a `<NA>` szimbólum) reprezentálja, a típus pedig nem változik át lebegőpontossá!

```python
# Hagyományos (NumPy-alapú) viselkedés - Átalakul float64-re:
In [133]: s = pd.Series([1, 2, 3, None])
In [134]: s
Out[134]: 
0    1.0
1    2.0
2    3.0
3    NaN
dtype: float64

# Kiterjesztett (Extension) típus használata:
In [136]: s = pd.Series([1, 2, 3, None], dtype=pd.Int64Dtype())
# VAGY string rövödítéssel (A NAGYBETŰ fontos!): dtype="Int64"
In [137]: s
Out[137]: 
0       1
1       2
2       3
3    <NA>
dtype: Int64
```

A kiterjesztett típusok a szövegeknél is remekelnek. Az `object` helyett használhatod a `StringDtype` (vagy `"string"`) típust, amely kevesebb memóriát eszik és gyorsabb:

```python
In [143]: s = pd.Series(['one', 'two', None, 'three'], dtype=pd.StringDtype())
In [144]: s
Out[144]: 
0      one
1      two
2     <NA>
3    three
dtype: string
```

**Konverzió:** Az `.astype()` metódussal bármikor átkonvertálhatsz egy hagyományos oszlopot az új kiterjesztett típusokra (pl. `.astype("Int64")`).

### Table 7-3. pandas kiterjesztett adattípusok

Ezek mind támogatják a `<NA>` (null) értékeket anélkül, hogy float-ra változnának.

| Kiterjesztett típus | Leírás (Description) |
| :--- | :--- |
| `BooleanDtype` | `"boolean"` - Nullázható logikai értékek (`True` / `False` / `NA`). |
| `CategoricalDtype` | `"category"` - Kategórikus adatok. |
| `DatetimeTZDtype` | Dátum-idő típus, időzónával kiegészítve. |
| `Float32Dtype`, `Float64Dtype` | `"Float32"`, `"Float64"` - Lebegőpontos számok (Az első betű nagy!). |
| `Int8`...`Int64Dtype` | `"Int8"` - `"Int64"` - Nullázható előjeles egész számok (Nagybetűs `Int`!). |
| `UInt8`...`UInt64Dtype` | `"UInt8"` - `"UInt64"` - Nullázható előjel nélküli (csak pozitív) egész számok. |

## 7.4 Szövegmanipuláció (String Manipulation)

A Python eleve kiváló a szövegfeldolgozásban. Először nézzük meg a Python beépített string metódusait, utána pedig azt, hogyan alkalmazható ez a pandasban.

**Beépített Python string operációk:**

- `.split(elválasztó)`: Darabokra (listára) vágja a szöveget a megadott elválasztó mentén.
- `.strip()`: Levágja a felesleges szóközöket és entereket a szöveg két végéről.
- `.join(lista)`: Összefűzi a listában található szövegeket úgy, hogy közéjük teszi azt a stringet, amin meghívtad.

**Gyakori "recept" (egy list comprehension-nel kombinálva):**

```python
In [151]: val = "a,b,  guido"
# Felvágjuk a vesszőknél, és minden darab végéről eltüntetjük a szóközöket:
In [153]: pieces = [x.strip() for x in val.split(",")]
In [154]: pieces
Out[154]: ['a', 'b', 'guido']

# Utána összefűzzük őket egy dupla kettősponttal:
In [157]: "::".join(pieces)
Out[157]: 'a::b::guido'
```

**Keresés a szövegben:**

- `in`: Boolean választ ad, hogy benne van-e. (Pl. `"guido" in val` -> `True`).
- `.find()` és `.index()`: Visszaadják a keresett szó első betűjének helyét (indexét). A különbség, hogy ha nem találják, a `find` `-1`-et ad vissza, az `index` viszont `ValueError` hibát dob és megállítja a programot.
- `.count()`: Megszámolja a részlet előfordulását.
- `.replace()`: Kicserél egy részletet egy másikra (vagy kitörli, ha az új részlet egy üres `""` string).

### Table 7-4. Python beépített string metódusok

| Metódus | Leírás (Description) |
| :--- | :--- |
| `count` | Visszaadja, hányszor szerepel (nem átfedésben) a részlet a szövegben. |
| `endswith` | Igaz (`True`), ha a szöveg a megadott végződéssel (suffix) végződik. |
| `startswith` | Igaz (`True`), ha a szöveg a megadott kezdettel (prefix) indul. |
| `join` | Delimiterként (elválasztóként) szolgál egy lista/szekvencia összefűzésénél. |
| `index` | Visszaadja az első találat kezdő indexét. Hibát dob, ha nincs benne. |
| `find` | Ugyanaz, mint az `index`, de hiba helyett `-1`-et ad, ha nincs találat. |
| `rfind` | Jobbról (hátulról) keresi az első (azaz az utolsó) találatot. `-1`-et ad, ha nincs. |
| `replace` | Kicseréli a szöveg adott részleteit egy másik szövegre. |
| `strip`, `rstrip`, `lstrip` | Eltávolítja a szóközöket/entereket mindkét (`strip`), jobb (`rstrip`) vagy bal (`lstrip`) oldalról. |
| `split` | Feldarabolja a szöveget a megadott elválasztó (delimiter) alapján. |
| `lower` | Minden betűt kisbetűssé alakít. |
| `upper` | Minden betűt NAGYBETŰSSÉ alakít. |
| `casefold` | Erőteljesebb a `lower`-nél; úgy alakít kisbetűssé, hogy a speciális regionális karakterek is összehasonlíthatóvá váljanak. |
| `ljust`, `rjust` | Balra vagy jobbra igazítja a szöveget, és kitölti szóközökkel egy megadott minimum szélességig. |


## 7.4 Szövegmanipuláció (Folytatás)

### Reguláris Kifejezések (Regular Expressions / RegEx)

A **Reguláris Kifejezések** lehetővé teszik, hogy mintázatokat (patterns) keressünk a szövegben. Ez a Python beépített `re` moduljával történik. Képzeld el úgy, mint egy szuper-intelligens "Keresés és Csere" (Ctrl+F) funkciót.

> **Tipp:** A regex mintákat mindig nyers Python szövegként (raw string) írjuk le, egy `r` betűvel az elején, pl. `r"minta"`. Így a gép nem próbálja meg értelmezni a speciális backslash `\` karaktereket.

**Feldarabolás változó hosszúságú szóközök/tabulátorok mentén:**

```python
In [165]: import re
In [166]: text = "foo    bar\t baz  \tqux"

# A \s+ jelentése: "Egy vagy több whitespace (szóköz/tab/enter) karakter"
In [167]: re.split(r"\s+", text)
Out[167]: ['foo', 'bar', 'baz', 'qux']
```

**Regex objektum előfordítása (Kompilálás):**
Ha ugyanazt a mintát sokszor (pl. egy millió soros táblázatban) akarod használni, érdemes először lefordítani a `re.compile()` segítségével. Ez rengeteg processzoridőt (CPU) spórol meg!

```python
In [168]: regex = re.compile(r"\s+")
In [169]: regex.split(text)
Out[169]: ['foo', 'bar', 'baz', 'qux']
```

**Minták megkeresése és kinyerése:**
Nézzünk egy igazi példát: szedjük ki az e-mail címeket egy szövegből. A minta `r"[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,4}"` azt jelenti, hogy: "valami név @ valami domain . valami 2-4 betűs kiterjesztés".

```python
text = """Dave dave@google.com
Steve steve@gmail.com
Rob rob@gmail.com
Ryan ryan@yahoo.com"""

pattern = r"[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,4}"

# A re.IGNORECASE miatt nem fog számítani a kis- és nagybetű!
regex = re.compile(pattern, flags=re.IGNORECASE)
```

Három fő keresőfüggvény létezik:

- **`findall`:** Megkeresi az ÖSSZES találatot, és visszaadja őket egy listában.

```python
In [172]: regex.findall(text)
Out[172]: 
['dave@google.com',
 'steve@gmail.com',
 'rob@gmail.com',
 'ryan@yahoo.com']
```

- **`search`:** Megkeresi a legelső találatot bárhol a szövegben, és visszaad egy speciális `Match` objektumot (ami megmondja, hányadik karakternél kezdődik és végződik).
- **`match`:** Olyan, mint a `search`, de CSAK akkor ad találatot, ha a minta a szöveg legelső karakterénél kezdődik. Mivel a mi szövegünk a "Dave " szóval kezdődik, ez `None`-t ad.

**Szövegcsere (Helyettesítés):**
A `.sub(mire, hol)` metódussal lecserélheted a találatokat (mint cenzúrázásnál).

```python
In [177]: print(regex.sub("REDACTED", text))
# Dave REDACTED
# Steve REDACTED
...
```

**Zárójelek és Csoportosítás (Grouping):**
Ha a regex mintába kerek zárójeleket `()` teszel, a gép részekre (csoportokra) osztja a találatot (pl. felhasználónév, domain, kiterjesztés).

```python
# Zárójeleket tettünk a @ és a . jel köré
In [178]: pattern = r"([A-Z0-9._%+-]+)@([A-Z0-9.-]+)\.([A-Z]{2,4})"
In [179]: regex = re.compile(pattern, flags=re.IGNORECASE)

In [182]: regex.findall(text)
Out[182]: 
[('dave', 'google', 'com'),
 ('steve', 'gmail', 'com'),
 ('rob', 'gmail', 'com'),
 ('ryan', 'yahoo', 'com')]
```

Ennek óriási haszna van cserénél is: a `\1`, `\2` az első és második csoportra hivatkozik!

```python
In [183]: print(regex.sub(r"Username: \1, Domain: \2, Suffix: \3", text))
# Dave Username: dave, Domain: google, Suffix: com
```

### Table 7-5. Reguláris Kifejezés (Regex) metódusok

| Metódus | Leírás (Description) |
| :--- | :--- |
| `findall` | Visszaadja a szövegben lévő összes találatot (amik nem fedik át egymást) egy listában. |
| `finditer` | Ugyanaz, mint a `findall`, de egy iterátort ad vissza lista helyett. |
| `match` | Csak a szöveg legelején keresi a mintát. Ha talál, ad egy `Match` objektumot, különben `None`. |
| `search` | Végigpásztázza a szöveget a legelső találatig, és visszaadja a `Match` objektumot. Bárhol lehet a szövegben. |
| `split` | Feldarabolja a szöveget ott, ahol megtalálja a mintát. |
| `sub`, `subn` | Kicseréli az összes (`sub`) vagy az első `n` darab (`subn`) találatot a megadott új szövegre. A `\1`, `\2` szimbólumokkal hivatkozhatunk a talált csoportokra. |

### Szöveges (String) függvények a pandas-ban (A `.str` attribútum)

Adattisztításnál hatalmas probléma, hogy a szöveges oszlopokban gyakran vannak hiányzó (`NaN`) adatok. Ha egy sima Python `.map()` függvénnyel rájuk eresztesz egy string metódust, a program összeomlik, mert a `NaN` (ami egy float szám) nem szöveg!

A pandas erre találta ki a `.str` attribútumot. Ez lehetővé teszi, hogy vektorizáltan hajts végre string műveleteket a teljes oszlopon, miközben a `NaN` értékeket elegánsan átugorja!

```python
In [184]: data = {"Dave": "dave@google.com", "Steve": "steve@gmail.com",
   .....:         "Rob": "rob@gmail.com", "Wes": np.nan}
In [185]: data = pd.Series(data)
```

Nézzük meg, kinek van "gmail" a címében a `.str.contains` segítségével:

```python
In [188]: data.str.contains("gmail")
Out[188]: 
Dave     False
Steve     True
Rob       True
Wes        NaN     # <- Átugrotta, és NaN maradt a hibadobás helyett!
dtype: object
```

**Regex használata a pandas `.str`-en belül:**
Bármelyik `.str` metódus érti a reguláris kifejezéseket!

```python
# Kinyerjük az e-mail címeket csoportosítva:
In [193]: data.str.findall(pattern, flags=re.IGNORECASE)
Out[193]: 
Dave     [(dave, google, com)]
Steve    [(steve, gmail, com)]
...
```

Ha rá akarunk hivatkozni a lista legelső elemére, vagy a csoport (tuple) egy bizonyos elemére, a `.str` attribútum ezt is tudja (láncolva is használható!):

```python
# Kiveszi a listából a nulladik elemet (ami a tuple)
In [194]: matches = data.str.findall(pattern, flags=re.IGNORECASE).str[0]

# Kiveszi a tuple-ből az 1-es indexű (második) elemet (a domaint)
In [196]: matches.str.get(1)
Out[196]: 
Dave     google
Steve     gmail
...
```

(Ugyanígy szeletelhetünk is szövegeket tömegesen, pl. `data.str[:5]` minden szövegből levágja az első 5 karaktert).

A legprofibb megoldás e-mail címek (és egyéb csoportosított regex minták) szétszedésére az `.str.extract`. Ez azonnal egy gyönyörű DataFrame-et csinál a három regex-csoportból!

```python
In [198]: data.str.extract(pattern, flags=re.IGNORECASE)
Out[198]: 
           0       1    2
Dave    dave  google  com
Steve  steve   gmail  com
Rob      rob   gmail  com
Wes      NaN     NaN  NaN
```

### Table 7-6. A pandas Series `.str` metódusai

| Metódus | Leírás (Description) |
| :--- | :--- |
| `cat` | Összefűzi a szövegeket egy opcionális delimiterrel (elválasztóval). |
| `contains` | Visszaad egy `True`/`False` tömböt, ha a szöveg tartalmazza a mintát/regex-et. |
| `count` | Megszámolja a minta előfordulásait az adott sorban. |
| `extract` | Használ egy csoportosító `()` regex-et, és kinyeri az adatokat egy új DataFrame-be (minden csoport egy oszlop lesz). |
| `endswith` | Igaz (`True`), ha a szöveg adott mintával végződik. |
| `startswith` | Igaz (`True`), ha a szöveg adott mintával kezdődik. |
| `findall` | Keresi a minta összes előfordulását (listát ad vissza cellánként). |
| `get` | Kiveszi az `i`-edik elemet (legyen az lista vagy tuple a cellában). |
| `isalnum` / `isalpha` / `isdigit` | Beépített Python ellenőrzők (szám-betű, csak betű, csak szám-e a string). |
| `islower` / `isupper` | Kisbetűs / Nagybetűs ellenőrzés. |
| `join` | Összefűzi az elemeket a megadott elválasztóval. |
| `len` | Kiszámítja az egyes szövegek hosszát (karakterszámot). |
| `lower`, `upper` | Átalakítja kis- vagy nagybetűssé a szövegeket. |
| `match` | A `re.match` megfelelője: ellenőrzi, hogy a szöveg eleje megegyezik-e a mintával. |
| `pad` / `center` | Szóközöket ad a szöveg bal, jobb (vagy `center` esetén mindkét) oldalához egy bizonyos hosszig. |
| `repeat` | Megsokszorozza a szöveget (pl. `s.str.repeat(3)` háromszor egymás után írja le a szöveget a cellában). |
| `replace` | Kicseréli a mintát/regex-et egy másik szövegre. |
| `slice` | Felszeleteli a cellákban lévő szövegeket. |
| `split` | Feldarabolja a szövegeket a határoló/regex mentén (listákat csinál a cellákba). |
| `strip` / `rstrip` / `lstrip` | Eltávolítja a szóközöket/entereket mindkét (`strip`), jobb (`rstrip`) vagy bal (`lstrip`) oldalról. |

## 7.5 Kategórikus Adatok (Categorical Data)

### Háttér és Motiváció

Nagyon gyakori, hogy egy táblázat egyik oszlopa csak egy szűk halmazból származó (tehát nagyon sokszor ismétlődő) értékeket tartalmaz. Például egy "Gyümölcs" oszlopban az `'apple'` és `'orange'` szavak ismétlődnek milliószor.

Ha ezeket normál Python szövegként (string) tároljuk a memóriában, az rettentően sok helyet foglal, és a rajtuk végzett műveletek (pl. csoportosítás) lassúak.

Az adattárházakban erre a megoldás a **dimenziótáblák** és a **szótár alapú kódolás** (dictionary-encoded representation) használata. Ilyenkor magukat a konkrét szövegeket (`"apple"`, `"orange"`) csak egyszer tároljuk el egy kis "szótárban" (ezeket hívjuk kategóriáknak vagy szinteknek / levels), a hatalmas fő táblázatba pedig csak a hozzájuk tartozó apró egész számú kódokat (0, 1) írjuk be!

### A Categorical kiterjesztett típus a pandas-ban

A pandas a fenti tömörítési technikát a **Categorical** kiterjesztett típussal valósítja meg.

Vegyünk egy adathalmazt, amiben sokszor ismétlődnek a gyümölcsnevek szövegként:

```python
In [212]: df
Out[212]: 
   basket_id   fruit  count    weight
0          0   apple     11  1.564438
1          1  orange      5  1.331256
2          2   apple     12  2.393235
...
```

Konvertáljuk át a "fruit" oszlopot `category` típusúvá az `.astype()` metódussal:

```python
In [213]: fruit_cat = df['fruit'].astype('category')
In [214]: fruit_cat
Out[214]: 
0     apple
1    orange
2     apple
...
Name: fruit, dtype: category
Categories (2, object): ['apple', 'orange']
```

Bár a képernyőn továbbra is a szövegeket látod, a háttérben a pandas most már apró számokkal dolgozik! Két fontos attribútuma van: a `.categories` (ami a szótár, pl. `'apple'`, `'orange'`) és a `.codes` (ami az adattömb, tele 0-kkal és 1-esekkel).

```python
In [217]: c = fruit_cat.array
In [218]: c.codes
Out[218]: array([0, 1, 0, 0, 0, 1, 0, 0], dtype=int8)
```

Létrehozhatsz egy kategórikus tömböt közvetlenül az apró kódokból is (ha megmondod hozzá a kategóriákat), és még sorrendiséget is megadhatsz nekik (pl. egy kérdőíven: "Rossz" < "Közepes" < "Jó").

```python
In [228]: ordered_cat = pd.Categorical.from_codes([0, 1, 2, 0, 0, 1],
   .....:                                         ['foo', 'bar', 'baz'],
   .....:                                         ordered=True)
In [229]: ordered_cat
Out[229]: 
['foo', 'bar', 'baz', 'foo', 'foo', 'bar']
Categories (3, object): ['foo' < 'bar' < 'baz']
```

### Számítások Kategórikus adatokkal

A `pd.qcut` függvény (amit az előző részben tanultunk a kvantilisek szerinti daraboláshoz) valójában egy kategórikus objektumot ad vissza!

Képzeljük el, hogy egy feladathoz értelmes neveket ("Q1", "Q2" stb.) adunk a 4 negyed (kvartilis) vödrének:

```python
In [236]: bins = pd.qcut(draws, 4, labels=['Q1', 'Q2', 'Q3', 'Q4'])
In [237]: bins
Out[237]: 
['Q1', 'Q4', 'Q1', 'Q2', 'Q2', ..., 'Q3', 'Q3', 'Q2', 'Q3', 'Q2']
Length: 1000
Categories (4, object): ['Q1' < 'Q2' < 'Q3' < 'Q4']
```

Mivel ez az objektum emlékszik a sorrendiségre (`'Q1'` kisebb, mint `'Q2'`), a pandas ezt fel tudja használni a statisztikák, például a `groupby` (csoportosítás) során.

**Teljesítmény-növekedés (Miért használjuk ezt?):**
Nézzünk meg egy 10 millió elemű oszlopot szövegként, majd konvertáljuk `category` típusra.

- **Memóriahasználat:** A szöveges (`object`) verzió ~600 MB RAM-ot eszik, míg a kategórikus csupán ~10 MB-ot!
- **Sebesség:** Ha ezen a 10 millió soron meghívod a `value_counts()` gyakoriságszámlálót, a szöveges verziónak ~840 ms kell hozzá, a kategórikus verziónak viszont csak 30 ms!

### Kategórikus Metódusok (A `.cat` attribútum)

Ahogy a szövegeknél a `.str`, a dátumoknál a `.dt`, úgy a kategórikus adatoknál a pandas a `.cat` attribútumon keresztül biztosítja a speciális metódusokat. Ezen keresztül érheted el a kódokat és a kategóriákat anélkül, hogy a nyers array-hez nyúlnál.

```python
In [254]: cat_s.cat.codes
Out[254]: 
0    0
1    1
...

In [255]: cat_s.cat.categories
Out[255]: Index(['a', 'b', 'c', 'd'], dtype='object')
```

**Kategóriák bővítése:**
Gyakran megesik, hogy tudod, hogy a "Kiváló" minősítés is létezik, csak pont a te adathalmazodban nem szerepelt egyszer sem. Ha később gépi tanulást fogsz végezni rajta, be kell állítanod, hogy a pandas ismerje ezt a kategóriát is! Erre való a `.set_categories()`.

```python
In [256]: actual_categories = ['a', 'b', 'c', 'd', 'e']
In [257]: cat_s2 = cat_s.cat.set_categories(actual_categories)
```

Innentől kezdve a `value_counts()` ki fogja írni az "e" kategóriát is 0-s darabszámmal!

**Nem használt kategóriák eltávolítása:**
Ennek az ellentéte, amikor leszűrtél egy nagy adathalmazt, és olyan kategóriák maradtak a memóriában (a szótárban), amik már biztosan nincsenek benne a szűrt adatokban.

```python
In [263]: cat_s3.cat.remove_unused_categories()
```

### Table 7-7. Kategórikus metódusok a pandas Series-hez

Ezek mind a `.cat.` előtagon keresztül érhetők el.

| Metódus | Leírás (Description) |
| :--- | :--- |
| `add_categories` | Új, eddig nem használt kategóriákat fűz hozzá a már meglévők végéhez. |
| `as_ordered` | Beállítja, hogy a kategóriák között legyen sorrendiség. |
| `as_unordered` | Eltávolítja a sorrendiséget a kategóriák közül. |
| `remove_categories` | Eltávolít specifikus kategóriákat. (Az ilyen értékű adatok a cellákban `NaN`-ná válnak). |
| `remove_unused_categories` | Kitörli a memóriából azokat a kategóriákat, amik már egyszer sem szerepelnek az adathalmazban. |
| `rename_categories` | Átnevezi a kategóriákat egy új listára. A kategóriák száma nem változhat. |
| `reorder_categories` | Átnevezés, de egyben beállítja, hogy a megadott lista szerinti rendezett (ordered) kategória jöjjön létre. |
| `set_categories` | Kicseréli a teljes kategória-halmazt egy újra (lehet újakat is hozzáadni, és eltávolítani meglévőket). |

### Dummy változók készítése a modellezéshez

A fejezet korábbi részében megismert One-Hot Encoding (`pd.get_dummies()`) természetesen a `category` típusú oszlopokon is tökéletesen és gyorsan működik. Létrehoz egy DataFrame-et, amiben annyi oszlop lesz, ahány kategória a szótárban szerepelt (akár van belőle konkrét adat, akár nincs!), és 1-esekkel/0-kal jelzi a jelenlétet.

```python
In [264]: cat_s = pd.Series(['a', 'b', 'c', 'd'] * 2, dtype='category')
In [265]: pd.get_dummies(cat_s)
Out[265]: 
   a  b  c  d
0  1  0  0  0
1  0  1  0  0
2  0  0  1  0
...
```

## 7.6 Összegzés (Conclusion)

A hatékony adatelőkészítés drasztikusan javítja a termelékenységedet: sokkal több időd marad magára az elemzésre, ha már profin és gyorsan tudod kitisztítani az adatokat. Ebben a fejezetben megtanultuk a hiányzó értékek kezelését, a vektorizált értékcseréket, a kiterjesztett típusokat és a memóriakímélő kategórikus adatokat. A következő (8.) fejezetben azzal foglalkozunk majd, hogyan kombináljuk (merge/join) és csoportosítsuk (groupby) a pandas objektumokat.
