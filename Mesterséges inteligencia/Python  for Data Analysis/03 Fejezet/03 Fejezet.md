# 3. Fejezet: Beépített adatszerkezetek, függvények és fájlok

## 3.1 Adatszerkezetek és Szekvenciák (Data Structures and Sequences)

A Python adatszerkezetei egyszerűek, de rendkívül erősek. A leggyakrabban használt szekvencia (sorozat) típusok a **tuple**, a **list** és a **dict** (szótár). Most az első kettőt nézzük meg.

### Tuple (Többszörös / Kötött sorozat)

**Fő koncepció:** A tuple egy fix hosszúságú, módosíthatatlan (immutable) Python objektumsorozat. Ha egyszer létrehoztad, a mérete és a benne lévő elemek sorrendje kőbe van vésve.

**Létrehozás:**
Leggyakrabban kerek zárójelekkel `()` vagy csak simán vesszővel elválasztva hozzuk létre.

```python
In [2]: tup = (4, 5, 6)
In [3]: tup
Out[3]: (4, 5, 6)

# Zárójelek nélkül is működik:
In [4]: tup = 4, 5, 6
In [5]: tup
Out[5]: (4, 5, 6)
```

Bármilyen iterálható dolgot (pl. szöveget) át lehet alakítani tuple-lé a `tuple()` függvénnyel:

```python
In [6]: tuple([4, 0, 2])
Out[6]: (4, 0, 2)

In [7]: tup = tuple('string')
In [8]: tup
Out[8]: ('s', 't', 'r', 'i', 'n', 'g')
```

**Indexelés és beágyazás:**
Az elemeket (ugyanúgy, mint C-ben vagy Java-ban) 0-tól kezdve indexeljük, szögletes zárójellel `[]`. Tuple-ökbe tehetünk más tuple-öket is (beágyazás).

```python
In [9]: tup[0]
Out[9]: 's'

In [10]: nested_tup = (4, 5, 6), (7, 8)
In [11]: nested_tup
Out[11]: ((4, 5, 6), (7, 8))

In [12]: nested_tup[0]
Out[12]: (4, 5, 6)
In [13]: nested_tup[1]
Out[13]: (7, 8)
```

**Módosíthatatlanság (Immutability):**
A tuple elemeit nem lehet kicserélni.

```python
In [14]: tup = tuple(['foo', [1, 2], True])
In [15]: tup[2] = False
# TypeError: 'tuple' object does not support item assignment
```

**Kivétel:** Ha a tuple belsejében egy módosítható (mutable) objektum van (például egy lista), akkor annak a belsejét megváltoztathatod!

```python
In [16]: tup[1].append(3)
In [17]: tup
Out[17]: ('foo', [1, 2, 3], True)
```

**Magyarázat:** A tuple továbbra is ugyanarra a listára mutat, ezt a "mutatást" nem lehet megváltoztatni. De magába a listába bele lehet tenni egy új elemet (a 3-ast).

**Összefűzés és többszörözés:**

```python
In [18]: (4, None, 'foo') + (6, 0) + ('bar',)
Out[18]: (4, None, 'foo', 6, 0, 'bar')

In [19]: ('foo', 'bar') * 4
Out[19]: ('foo', 'bar', 'foo', 'bar', 'foo', 'bar', 'foo', 'bar')
```

**Magyarázat:** A `+` új, hosszabb tuple-t csinál. A `* 4` pedig egymás után négyszer fűzi össze a tuple tartalmát (az elemek nem másolódnak, csak a referenciák).

**Kicsomagolás (Unpacking):**
Ha egy tuple értékeit külön változókba akarod tenni, a Python automatikusan "kicsomagolja" neked:

```python
In [20]: tup = (4, 5, 6)
In [21]: a, b, c = tup
In [22]: b
Out[22]: 5
```

Működik beágyazott tuple-ökkel is:

```python
In [23]: tup = 4, 5, (6, 7)
In [24]: a, b, (c, d) = tup
In [25]: d
Out[25]: 7
```

**Változók megcserélése (Swap):**
Sok nyelvben ehhez kéne egy `tmp` (ideiglenes) változó. Pythonban a tuple kicsomagolás miatt ez egy sor:

```python
In [26]: a, b = 1, 2
In [29]: b, a = a, b
In [30]: a
Out[30]: 2
In [31]: b
Out[31]: 1
```

**Ciklusos kicsomagolás:**
Gyakori trükk tuple-ök listájánál:

```python
In [32]: seq = [(1, 2, 3), (4, 5, 6), (7, 8, 9)]
In [33]: for a, b, c in seq:
   ....:     print(f'a={a}, b={b}, c={c}')
```

**A `*rest` szintaxis (maradék elemek):**
Ha csak az első pár elem kell, a többit beleteheted egy listába a csillag `*` segítségével.

```python
In [34]: values = 1, 2, 3, 4, 5
In [35]: a, b, *rest = values
In [36]: a
Out[36]: 1
In [37]: b
Out[37]: 2
In [38]: rest
Out[38]: [3, 4, 5]
```

> **Jó gyakorlat:** Ha a maradék adatra nincs szükséged, a `rest` helyett használd az alulvonás `*_` nevet: `a, b, *_ = values`.

**Tuple metódusok:**
Mivel nem lehet módosítani, kevés metódusa van. A legfontosabb a `.count()`, ami megszámolja, hányszor szerepel egy érték:

```python
In [40]: a = (1, 2, 2, 2, 3, 4, 2)
In [41]: a.count(2)
Out[41]: 4
```

### Listák (List)

**Fő koncepció:** A listák változó hosszúságúak, és a tartalmuk bármikor módosítható (mutable). Szögletes zárójelekkel `[]` vagy a `list()` függvénnyel hozhatók létre.

```python
In [42]: a_list = [2, 3, 7, None]
In [43]: tup = ("foo", "bar", "baz")
In [44]: b_list = list(tup)
In [45]: b_list
Out[45]: ['foo', 'bar', 'baz']
```

**Módosítás:**

```python
In [46]: b_list[1] = "peekaboo"
In [47]: b_list
Out[47]: ['foo', 'peekaboo', 'baz']
```

**Generátorok "kiterítése" listává:**

```python
In [48]: gen = range(10)
In [50]: list(gen)
Out[50]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**Elemek hozzáadása és törlése:**

- `.append()`: Elem hozzáadása a legvégére.
- `.insert(index, elem)`: Elem beszúrása egy adott pozícióra.

```python
In [51]: b_list.append("dwarf")
In [52]: b_list
Out[52]: ['foo', 'peekaboo', 'baz', 'dwarf']

In [53]: b_list.insert(1, "red")
In [54]: b_list
Out[54]: ['foo', 'red', 'peekaboo', 'baz', 'dwarf']
```

> **Fontos megjegyzés a teljesítményről:** Az `insert` lassú (számításigényes), mert a beszúrt elemtől jobbra lévő összes többi elemet a gépnek arrébb kell tolnia a memóriában. Ha sokat kell az elejére beszúrni, használd inkább a `collections.deque` szerkezetet.

- `.pop(index)`: Eltávolítja és visszaadja az adott indexű elemet (az `insert` ellentéte).
- `.remove(érték)`: Megkeresi az adott értéket, és az első találatot kitörli.

```python
In [55]: b_list.pop(2)
Out[55]: 'peekaboo'

In [57]: b_list.append("foo")
In [59]: b_list.remove("foo")
In [60]: b_list
Out[60]: ['red', 'baz', 'dwarf', 'foo']
```

**Keresés a listában (`in`):**

```python
In [61]: "dwarf" in b_list
Out[61]: True
In [62]: "dwarf" not in b_list
Out[62]: False
```

> **Teljesítmény tipp:** A listában való keresés viszonylag lassú, mert a gép az elejétől a végéig végignézi a listát (lineáris keresés). Ha sokszor kell ellenőrizni, hogy valami benne van-e, érdemes Set-et (halmazt) vagy Dictionary-t (szótárat) használni.

**Listák összefűzése (`+` vs `.extend`):**

```python
In [63]: [4, None, "foo"] + [7, 8, (2, 3)]
Out[63]: [4, None, 'foo', 7, 8, (2, 3)]

In [64]: x = [4, None, "foo"]
In [65]: x.extend([7, 8, (2, 3)])
```

> **Teljesítmény tipp:** A `+` operátor minden alkalommal egy új listát hoz létre a memóriában, ami sok elem esetén lassú. A `.extend()` helyben módosítja a meglévő listát, így sokkal gyorsabb, ha iterációk során akarunk építeni egy listát!

**Rendezés (`.sort`):**
A `.sort()` helyben (in-place) rendezi a listát.

```python
In [67]: a = [7, 2, 5, 1, 3]
In [68]: a.sort()
In [69]: a
Out[69]: [1, 2, 3, 5, 7]
```

Átadhatunk egy `key` paramétert is, amivel megmondjuk, mi alapján rendezzen. Például a beépített `len` (hosszúság) függvénnyel a rövidebb szavaktól a hosszabbak felé:

```python
In [70]: b = ["saw", "small", "He", "foxes", "six"]
In [71]: b.sort(key=len)
In [72]: b
Out[72]: ['He', 'saw', 'six', 'small', 'foxes']
```

### Szeletelés (Slicing)

A szekvenciákból (listák, tuple-ök, szövegek) a `[start:stop]` szintaxissal tudunk darabokat kivágni.
**Alapszabály:** A `start` index beletartozik az eredménybe, a `stop` index viszont **MÁR NEM**.

```python
In [73]: seq = [7, 2, 3, 7, 5, 6, 0, 1]
In [74]: seq[1:5]
Out[74]: [2, 3, 7, 5]
```

**Magyarázat:** A 1-es, 2-es, 3-as és 4-es indexű elemeket vágja ki. (A kimenet hossza mindig: `stop - start`, azaz `5-1=4`).

A szeletet le is cserélhetjük (assignment):

```python
In [75]: seq[3:5] = [6, 3]
In [76]: seq
Out[76]: [7, 2, 3, 6, 3, 6, 0, 1]
```

Ha elhagyod a `start` vagy `stop` indexet, alapértelmezetten a lista legelejét / legvégét jelenti:

```python
In [77]: seq[:5]     # Az elejétől az 5. indexig (de azt már nem beleértve)
Out[77]: [7, 2, 3, 6, 3]

In [78]: seq[3:]     # A 3. indextől a legvégéig
Out[78]: [6, 3, 6, 0, 1]
```

**Negatív indexelés:**
A lista végétől számol visszafelé (a legutolsó elem a `-1`).

```python
In [79]: seq[-4:]
Out[79]: [3, 6, 0, 1]

In [80]: seq[-6:-2]
Out[80]: [3, 6, 3, 6]
```

Itt érdemes megnézni a könyv ábráját, ami tökéletesen elmagyarázza, hogyan is néznek ki ezek az indexek "dobozokként" felfogva.

> 📍 **[KÉP HELYE: Figure 3-1. Illustration of Python slicing conventions]**

**Lépésköz (Step) használata:**
Két kettőspont után megadhatod, hogy hányasával lépkedjen:

```python
In [81]: seq[::2]
Out[81]: [7, 3, 3, 0]
```

**Zseniális trükk:** Ha a lépésköz `-1`, az megfordítja (reverse) a teljes listát vagy szöveget!

```python
In [82]: seq[::-1]
Out[82]: [1, 0, 6, 3, 6, 3, 2, 7]
```

## 3.1 Adatszerkezetek és Szekvenciák

### Szótár (Dictionary / dict)

**Fő koncepció:** A szótár (más nyelvekben hash map vagy asszociatív tömb) valószínűleg a legfontosabb beépített Python adatszerkezet. Kulcs-érték (key-value) párokat tárol. Minden kulcshoz tartozik egy érték, így a kulcs ismeretében azonnal (nagyon gyorsan) lekérhető, módosítható vagy törölhető az adat. Kapcsos zárójelekkel `{}` és kettősponttal `:` hozzuk létre.

**Létrehozás, elérés és módosítás:**

```python
In [83]: empty_dict = {}
In [84]: d1 = {"a": "some value", "b": [1, 2, 3, 4]}
In [85]: d1
Out[85]: {'a': 'some value', 'b': [1, 2, 3, 4]}

In [86]: d1[7] = "an integer"
In [87]: d1
Out[87]: {'a': 'some value', 'b': [1, 2, 3, 4], 7: 'an integer'}
In [88]: d1["b"]
Out[88]: [1, 2, 3, 4]
```

**Magyarázat:** Létrehoztunk egy `d1` nevű szótárat. A `d1[7] = "an integer"` sorral beszúrtunk egy új elemet (ahol a kulcs a 7-es szám). A `d1["b"]` pedig visszaadja a "b" kulcshoz tartozó listát.

**Keresés és törlés:**

```python
In [89]: "b" in d1
Out[89]: True

In [94]: del d1[5]
In [96]: ret = d1.pop("dummy")
In [97]: ret
Out[97]: 'another value'
```

**Magyarázat:** Az `in` kulcsszóval ellenőrizhetjük, hogy egy kulcs létezik-e a szótárban. Törölni a `del` paranccsal vagy a `.pop()` metódussal lehet. A `.pop()` előnye, hogy nemcsak törli a kulcsot, hanem vissza is adja a törölt értéket (ezt mentettük a `ret` változóba).

**Iterálás a szótáron (Kulcsok, Értékek, Párok):**

```python
In [99]: list(d1.keys())
Out[99]: ['a', 'b', 7]

In [100]: list(d1.values())
Out[100]: ['some value', [1, 2, 3, 4], 'an integer']

In [101]: list(d1.items())
Out[101]: [('a', 'some value'), ('b', [1, 2, 3, 4]), (7, 'an integer')]
```

**Magyarázat:** A `.keys()` a kulcsokat, a `.values()` az értékeket, az `.items()` pedig a kettőt együtt (2-elemű tuple-ök formájában) adja vissza. A sorrend megegyezik a beszúrás sorrendjével.

**Szótárak egyesítése (`.update`):**

```python
In [102]: d1.update({"b": "foo", "c": 12})
In [103]: d1
Out[103]: {'a': 'some value', 'b': 'foo', 7: 'an integer', 'c': 12}
```

**Magyarázat:** Az `.update()` beolvaszt egy másik szótárat. **Figyelem:** Ez helyben (in-place) módosít! Mivel a "b" kulcs már létezett, a régi értékét felülírta az új "foo" értékkel.

**Szótárak létrehozása szekvenciákból:**
Ha van két listánk (egyik a kulcsok, másik az értékek), a `zip()` és a `dict()` segítségével elegánsan párosíthatjuk őket:

```python
In [104]: tuples = zip(range(5), reversed(range(5)))
In [106]: mapping = dict(tuples)
In [107]: mapping
Out[107]: {0: 4, 1: 3, 2: 2, 3: 1, 4: 0}
```

**Alapértelmezett értékek (Default values) kezelése:**
Gyakran előfordul, hogy le akarunk kérni egy értéket, de ha nem létezik, adni akarunk egy alapértéket.

Hosszú (if-else) megoldás helyett a `.get()` használata:

```python
# Helyett:
# if key in some_dict: value = some_dict[key]
# else: value = default_value

# Használd ezt:
value = some_dict.get(key, default_value)
```

Ha nemcsak lekérni, de beállítani is akarunk alapértéket, a `.setdefault()` vagy a `collections.defaultdict` a legjobb barátunk. (Például szavakat csoportosítunk kezdőbetű szerint):

```python
In [115]: from collections import defaultdict
In [116]: by_letter = defaultdict(list)
In [117]: for word in words:
   .....:     by_letter[word[0]].append(word)
```

**Magyarázat:** A `defaultdict(list)` azt jelenti, hogy ha egy kulcs (pl. "a") még nem létezik, automatikusan létrehoz hozzá egy üres listát `[]`, amibe rögtön tudunk `.append()`-elni.

**Érvényes szótárkulcsok (Hashability):**
Bár a szótár értéke bármi lehet, a kulcsnak módosíthatatlannak (immutable) kell lennie (pl. szám, string, tuple). Ezt a tulajdonságot nevezzük "hash-elhetőségnek" (hashability). A `hash()` függvénnyel ellenőrizheted:

```python
In [118]: hash("string")
Out[118]: 3634226001988967898

In [120]: hash((1, 2, [2, 3])) # fails because lists are mutable
# TypeError: unhashable type: 'list'
```

**Magyarázat:** A lista nem lehet kulcs, mert módosítható. Ha listát akarsz kulcsként használni, alakítsd át tuple-lé!

### Halmaz (Set)

**Fő koncepció:** A halmaz egy rendezetlen, egyedi (duplikáció nélküli) elemekből álló gyűjtemény. Képzeld el úgy, mint egy szótárat, aminek csak kulcsai vannak, értékei nincsenek.

**Létrehozás:** A `set()` függvénnyel vagy kapcsos zárójelekkel `{}`.

```python
In [124]: set([2, 2, 2, 1, 3, 3])
Out[124]: {1, 2, 3}

In [125]: {2, 2, 2, 1, 3, 3}
Out[125]: {1, 2, 3}
```

**Magyarázat:** Bár sok kettest és hármast adtunk meg, a halmaz automatikusan eldobja a másolatokat!

**Halmazműveletek (Unió, Metszet):**

```python
In [126]: a = {1, 2, 3, 4, 5}
In [127]: b = {3, 4, 5, 6, 7, 8}

# Unió (Minden elem a két halmazból)
In [128]: a.union(b)
In [129]: a | b
Out[129]: {1, 2, 3, 4, 5, 6, 7, 8}

# Metszet (Csak a közös elemek)
In [130]: a.intersection(b)
In [131]: a & b
Out[131]: {3, 4, 5}
```

> **Tipp:** A műveleteknek van "in-place" megfelelője is, pl. az `a |= b` felülírja az `a` halmazt az unió eredményével. Ez memóriatakarékos nagyon nagy halmazoknál.

#### Table 3-1. Python halmaz (set) műveletek

| Függvény (Function) | Alternatív szintaxis | Leírás (Description) |
| :--- | :--- | :--- |
| `a.add(x)` | N/A | Hozzáadja az `x` elemet az `a` halmazhoz |
| `a.clear()` | N/A | Teljesen kiüríti az `a` halmazt |
| `a.remove(x)` | N/A | Eltávolítja az `x` elemet az `a` halmazból |
| `a.pop()` | N/A | Eltávolít egy tetszőleges (véletlenszerű) elemet, és visszaadja. Ha üres, `KeyError`-t dob |
| `a.union(b)` | `a \| b` | Unió: Minden elem a két halmazból |
| `a.update(b)` | `a \|= b` | Beállítja `a` tartalmát az unióra (helyben) |
| `a.intersection(b)` | `a & b` | Metszet: Csak azok az elemek, amik `a`-ban ÉS `b`-ben is benne vannak |
| `a.intersection_update(b)` | `a &= b` | Beállítja `a` tartalmát a metszetre (helyben) |
| `a.difference(b)` | `a - b` | Különbség: Elemek, amik `a`-ban benne vannak, de `b`-ben nem |
| `a.difference_update(b)` | `a -= b` | Beállítja `a` tartalmát a különbségre (helyben) |
| `a.symmetric_difference(b)` | `a ^ b` | Szimmetrikus különbség: Elemek, amik vagy `a`-ban, vagy `b`-ben vannak, de nem mindkettőben |
| `a.symmetric_difference_update(b)` | `a ^= b` | Beállítja `a` tartalmát a szimmetrikus különbségre (helyben) |
| `a.issubset(b)` | `<=` | Igaz (`True`), ha `a` minden eleme benne van `b`-ben (részhalmaz) |
| `a.issuperset(b)` | `>=` | Igaz (`True`), ha `b` minden eleme benne van `a`-ban (bővített halmaz) |
| `a.isdisjoint(b)` | N/A | Igaz (`True`), ha `a`-nak és `b`-nek egyetlen közös eleme sincs (diszjunkt) |

### Beépített Szekvencia Függvények (Built-In Sequence Functions)

#### 1. `enumerate`

Amikor egy listán mész végig (`for` ciklus), gyakran kell nemcsak maga az elem, hanem annak indexe (sorszáma) is. Erre való az `enumerate`:

```python
for index, value in enumerate(collection):
    # csinálj valamit az értékkel és az indexszel
```

#### 2. `sorted`

Egy új, rendezett listát ad vissza bármilyen iterálható objektumból (a szövegeket betűrendbe rakja).

```python
In [145]: sorted([7, 1, 2, 6, 0, 3, 2])
Out[145]: [0, 1, 2, 2, 3, 6, 7]

In [146]: sorted("horse race")
Out[146]: [' ', 'a', 'c', 'e', 'e', 'h', 'o', 'r', 'r', 's']
```

#### 3. `zip`

A `zip` "összecipzáraz" (párosít) több listát vagy szekvenciát úgy, hogy az első elemeket egy tuple-be rakja, majd a másodikat stb.

```python
In [147]: seq1 = ["foo", "bar", "baz"]
In [148]: seq2 = ["one", "two", "three"]
In [149]: zipped = zip(seq1, seq2)
In [150]: list(zipped)
Out[150]: [('foo', 'one'), ('bar', 'two'), ('baz', 'three')]
```

> **Fontos:** Ha nem egyforma hosszúak a listák, a `zip` a legrövidebbnél megáll!

Tökéletes több lista egyidejű iterálására:

```python
In [153]: for index, (a, b) in enumerate(zip(seq1, seq2)):
   .....:     print(f"{index}: {a}, {b}")
```

#### 4. `reversed`

Megfordít egy szekvenciát. (Ez egy generátor, így ha látni akarjuk az eredményt, pl. listává kell alakítani).

```python
In [154]: list(reversed(range(10)))
Out[154]: [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```

### Comprehensions (Listagenerátorok / Kifejezések)

A Python egyik legjellegzetesebb, leggyakrabban használt nyelvi eleme. Lehetővé teszi, hogy egyetlen sorban hozzunk létre, szűrjünk meg és transzformáljunk listákat (vagy szótárakat, halmazokat).

**Alapképlet (Lista):**
`[kifejezés for érték in gyűjtemény if feltétel]`

**Példa:** Adott egy lista, vegyük azokat a szavakat, amik hosszabbak 2 karakternél, és tegyük mindet NAGYBETŰSSÉ:

```python
In [155]: strings = ["a", "as", "bat", "car", "dove", "python"]
In [156]: [x.upper() for x in strings if len(x) > 2]
Out[156]: ['BAT', 'CAR', 'DOVE', 'PYTHON']
```

(Ugyanez sima `for` ciklussal legalább 4 sor kód lenne!)

**Halmaz (Set) és Szótár (Dict) Comprehensions:**

- **Set:** Szögletes zárójel `[]` helyett kapcsos `{}` kell hozzá. Gyűjtsük ki a szavak egyedi hosszúságait:

```python
In [157]: unique_lengths = {len(x) for x in strings}
In [158]: unique_lengths
Out[158]: {1, 2, 3, 4, 6}
```

- **Dict:** Itt a kulcs-érték párt kettősponttal adjuk meg `kulcs: érték` formában. Készítsünk egy térképet a szóhoz és a lista-indexéhez:

```python
In [160]: loc_mapping = {value: index for index, value in enumerate(strings)}
In [161]: loc_mapping
Out[161]: {'a': 0, 'as': 1, 'bat': 2, 'car': 3, 'dove': 4, 'python': 5}
```

**Beágyazott Listagenerátorok (Nested List Comprehensions):**
Kifejezetten hasznos "kilapításra" (flattening), amikor egy listában listák (vagy tuple-ök) vannak, és egyetlen egydimenziós listát akarunk belőlük csinálni.

```python
In [168]: some_tuples = [(1, 2, 3), (4, 5, 6), (7, 8, 9)]
In [169]: flattened = [x for tup in some_tuples for x in tup]
In [170]: flattened
Out[170]: [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**Magyarázat:** Bár egy sorba van írva, a logika pontosan követi a hagyományos beágyazott `for` ciklusok sorrendjét.
1. Először: `for tup in some_tuples` (Vegyünk egy tuple-t)
2. Utána: `for x in tup` (Menjünk végig a benne lévő x számokon)
3. És az elején az `x` az az érték, amit a végső listába beletesz.

> **Vigyázat:** Ha a `[]` zárójeleket máshogy használjuk, akkor nem kilapítjuk a listát, hanem létrehozunk egy újabb listákból álló listát! Példa:

```python
In [172]: [[x for x in tup] for tup in some_tuples]
Out[172]: [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
```

## 3.2 Függvények (Functions)

A függvények a kód rendszerezésének és újrahasznosításának legfontosabb eszközei. **Alapszabály:** ha egy kódblokkot (vagy valami nagyon hasonlót) egynél többször le kell írnod, érdemes belőle függvényt írni.

### Alapszintaxis (`def` és `return`)

```python
In [173]: def my_function(x, y):
   .....:     return x + y

In [174]: my_function(1, 2)
Out[174]: 3
```

**Magyarázat:** A `def` kulcsszóval definiáljuk a függvényt, utána jön a neve, majd zárójelben a paraméterek. A `return` mondja meg, mit adjon vissza a függvény eredményként.

Ha a Python úgy ér a függvény végére, hogy nem találkozik `return` utasítással, akkor automatikusan a `None` értéket adja vissza:

```python
In [177]: def function_without_return(x):
   .....:     print(x)

In [178]: result = function_without_return("hello!")
hello!
In [179]: print(result)
None
```

### Pozicionális vs. Kulcsszavas (Keyword) argumentumok

A paramétereknek adhatunk alapértelmezett értéket, ezeket hívjuk kulcsszavas argumentumoknak.

```python
def my_function2(x, y, z=1.5):
    if z > 1:
        return z * (x + y)
    else:
        return z / (x + y)
```

Híváskor a pozicionális paramétereket (`x`, `y`) **KÖTELEZŐ** megadni. A `z`-t nem muszáj, ha nem adjuk meg, `1.5` lesz.

```python
In [181]: my_function2(5, 6, z=0.7)
Out[181]: 0.06363636363636363
In [182]: my_function2(3.14, 7, 3.5)
Out[182]: 35.49
```

> **Fontos szabály:** Függvényhíváskor a kulcsszavas argumentumoknak (ahol kiírod a nevét, pl. `z=...`) mindig a pozicionálisok után kell következniük!

### Névterek és Hatókör (Namespaces and Scope)

A változók élettartamát a névterek határozzák meg. Ami egy függvényen belül születik (helyi/local namespace), az a függvény végén megsemmisül.

Ha egy listát a függvényen kívül hozunk létre (globális/global namespace), a függvény képes módosítani azt (mivel a lista mutable):

```python
In [184]: a = []
In [185]: def func():
   .....:     for i in range(5):
   .....:         a.append(i)

In [186]: func()
In [187]: a
Out[187]: [0, 1, 2, 3, 4]
```

Ha viszont a függvényen belül akarsz új értéket adni (nem csak módosítani) egy kinti változónak, használnod kell a `global` kulcsszót (bár a szerző szerint ezt érdemes kerülni, és inkább osztályokat/objektumokat használni helyette).

### Több érték visszaadása (Returning Multiple Values)

A Python egyik legjobb funkciója, hogy egy függvény látszólag egyszerre több mindent is vissza tud adni.

```python
def f():
    a = 5
    b = 6
    c = 7
    return a, b, c

a, b, c = f()
```

**Magyarázat:** A valóságban a függvény csak egy dolgot ad vissza: egy 3 elemű tuple-t (`5, 6, 7`). Az utolsó sor ezt a tuple-t csomagolja ki automatikusan három külön változóba.

### A függvények is objektumok (Functions Are Objects)

Mivel a függvények is csak adatok a memóriában, átadhatod őket listákban vagy más függvények argumentumaként. Ez különösen adattisztításnál zseniális.

Tegyük fel, van egy csúnya adathalmazunk:

```python
In [193]: states = ["   Alabama ", "Georgia!", "Georgia", "georgia", "FlOrIda",
   .....:           "south   carolina##", "West virginia?"]
```

Ahelyett, hogy egy gigantikus függvényt írnánk, készíthetünk egy listát a műveletekből (függvényekből), amiket végre akarunk hajtani rajtuk:

```python
import re

def remove_punctuation(value):
    return re.sub("[!#?]", "", value)

# Lista a FÜGGVÉNYEKRŐL! Nem hívjuk meg őket, csak a nevüket adjuk át.
clean_ops = [str.strip, remove_punctuation, str.title]

def clean_strings(strings, ops):
    result = []
    for value in strings:
        for func in ops:
            value = func(value) # Itt hívjuk meg sorban a függvényeket
        result.append(value)
    return result
```

Ez a "funkcionális" programozási minta hihetetlenül rugalmassá teszi a kódot!

Használhatjuk a beépített `map()` függvényt is, ami egy adott függvényt ráhúz egy lista minden egyes elemére:

```python
In [198]: for x in map(remove_punctuation, states):
   .....:     print(x)
```

### Névtelen (Lambda) Függvények

Ezek egyetlen sorból álló, gyors, "eldobható" függvények. Azért hívjuk őket névtelennek, mert nem adunk nekik nevet a `def` kulcsszóval, csak a `lambda` kulcsszót használjuk.

```python
# Hagyományos:
def short_function(x):
    return x * 2

# Lambda:
equiv_anon = lambda x: x * 2
```

Adatelemzésnél rengetegszer használjuk, amikor egy másik függvény vár egy transzformációs szabályt, és lusták vagyunk egy teljes `def` blokkot írni neki.

Példa egy lista egyedi betűszám szerinti rendezésére:

```python
In [204]: strings = ["foo", "card", "bar", "aaaa", "abab"]
In [205]: strings.sort(key=lambda x: len(set(x)))
In [206]: strings
Out[206]: ['aaaa', 'foo', 'abab', 'bar', 'card']
```

**Magyarázat:** A `sort` megkapja a lambdát mint rendezési kulcsot. A lambda azt mondja: *"Csinálj a szóból egy halmazt (ez kiszűri a duplikált betűket, pl az 'aaaa' csak {'a'} lesz), és add vissza ennek a halmaznak a hosszát (ami itt 1)"*.

### Generátorok (Generators)

A Pythonban szinte minden iterálható (listák, szótárak, fájl sorai). Amikor megírsz egy `for` ciklust, a Python a háttérben létrehoz egy iterátort.

Egy hagyományos függvény egyszer lefut, kiszámolja a teljes eredményt (pl. egy hatalmas listát), beteszi a memóriába, és visszaadja. A **generátor** ezzel szemben "felfüggeszti" magát, és mindig csak **EGYETLEN** elemet számol ki és ad vissza (a `yield` kulcsszóval), amikor kérjük tőle. Ezzel rengeteg memóriát spórol!

```python
def squares(n=10):
    print(f"Generating squares from 1 to {n ** 2}")
    for i in range(1, n + 1):
        yield i ** 2
```

Amikor meghívod, a kód még nem fut le:

```python
In [213]: gen = squares()
In [214]: gen
Out[214]: <generator object squares at 0x7fefe437d120>
```

Csak akkor indul el és kezd el generálni, amikor elkezdünk "kérni" belőle (pl. `for` ciklussal):

```python
In [215]: for x in gen:
   .....:     print(x, end=" ")
Generating squares from 1 to 100
1 4 9 16 25 36 49 64 81 100 
```

**Generátor kifejezések (Generator expressions):**
Pont úgy néz ki, mint egy lista-comprehension, csak szögletes `[]` helyett kerek `()` zárójel van körülötte. Nagyon memóriatakarékos!

```python
In [216]: gen = (x ** 2 for x in range(100))

# Függvényargumentumként még a külső zárójel is elhagyható:
In [218]: sum(x ** 2 for x in range(100))
Out[218]: 328350
```

### Az itertools modul

Ez egy beépített könyvtár, ami zseniális generátorokat tartalmaz adatfeldolgozáshoz.

A `groupby` például csoportosít egymás utáni elemeket egy adott függvény (kulcs) alapján:

```python
In [220]: import itertools
In [222]: names = ["Alan", "Adam", "Wes", "Will", "Albert", "Steven"]
In [223]: for letter, names in itertools.groupby(names, lambda x: x[0]):
   .....:     print(letter, list(names))
A ['Alan', 'Adam']
W ['Wes', 'Will']
A ['Albert']
S ['Steven']
```

#### Table 3-2. Néhány hasznos itertools függvény

| Függvény (Function) | Leírás (Description) |
| :--- | :--- |
| `chain(*iterables)` | Összefűz iterátorokat. Ha az első kiürül, zökkenőmentesen folytatja a másodikkal, és így tovább. |
| `combinations(iterable, k)` | Legenerálja a szekvencia elemeiből képezhető összes lehetséges `k` elemű kombinációt (a sorrend nem számít, ismétlődés nincs). |
| `permutations(iterable, k)` | Legenerálja az összes lehetséges `k` elemű permutációt (itt a sorrend is számít). |
| `groupby(iterable[, keyfunc])` | Minden egyedi kulcshoz (amit a `keyfunc` generál) ad egy `(kulcs, al-iterátor)` párost. |
| `product(*iterables, repeat=1)` | Kiszámolja a bemeneti iterátorok Descartes-szorzatát (Cartesian product) tuple-ök formájában. Olyan, mint egy beágyazott `for`-ciklus. |

### Hibák és Kivételkezelés (Errors and Exception Handling)

Adatelemzésnél gyakori, hogy "rossz" (hibás formátumú) adatot kapunk, és nem akarjuk, hogy emiatt az egész program összeomoljon. Ezt a `try` / `except` blokkal kerülhetjük el.

```python
def attempt_float(x):
    try:
        return float(x)
    except:
        return x
```

**Magyarázat:** A gép megpróbálja végrehajtani a `try` blokkot (számmá alakítani a bemenetet). Ha sikerül, visszaadja. Ha bármilyen hiba történik (Exception), a program nem áll le hibaüzenettel, hanem átugrik az `except` blokkra, és visszaadja az eredeti, alakítatlan értéket.

**Specifikus hibák elkapása:**
A fenti kód veszélyes, mert minden hibát elnyom (még azokat is, amik mondjuk egy elgépelésből adódnak). Jobb, ha pontosan megmondjuk, milyen hibára számítunk (pl. `ValueError` vagy `TypeError`):

```python
def attempt_float(x):
    try:
        return float(x)
    except (TypeError, ValueError):
        return x
```

**`finally` és `else`:**
A `finally` blokk mindenképpen lefut a legvégén, függetlenül attól, hogy volt-e hiba vagy sem (általában erőforrások, pl. nyitott fájlok bezárására használjuk).

Az `else` blokk csak akkor fut le, ha a `try` blokk hibátlanul (kivétel nélkül) lefutott.

```python
f = open(path, mode="w")
try:
    write_to_file(f)
except:
    print("Failed")
else:
    print("Succeeded")
finally:
    f.close()
```

### Kivételek (Hibák) az IPythonban (Exceptions in IPython)

Ha lefuttatsz egy scriptet (`%run` paranccsal) vagy végrehajtasz egy kódsort az IPythonban, és az hibára (Exception) fut, az IPython alapértelmezetten egy nagyon részletes, teljes visszakövetést (call stack trace / **traceback**) ír ki.

Ami zseniális az IPythonban a sima Pythonnal szemben, hogy néhány sornyi környezetet (context) is megmutat a kódodból ott, ahol a hiba történt. Így azonnal látod a hibás kódsort, anélkül, hogy meg kéne nyitnod a fájlt.

```python
In [10]: %run examples/ipython_bug.py
---------------------------------------------------------------------------
AssertionError                            Traceback (most recent call last)
/home/wesm/code/pydata-book/examples/ipython_bug.py in <module>()
     13     throws_an_exception()
     14 
---> 15 calling_things()

/home/wesm/code/pydata-book/examples/ipython_bug.py in calling_things()
     11 def calling_things():
     12     works_fine()
---> 13     throws_an_exception()
     14 
     15 calling_things()

/home/wesm/code/pydata-book/examples/ipython_bug.py in throws_an_exception()
      7     a = 5
      8     b = 6
----> 9     assert(a + b == 10)
     10 
     11 def calling_things():

AssertionError: 
```

Ezt a részletességet szabályozhatod az `%xmode` "magic" paranccsal (a `Plain`-től a nagyon bőbeszédű `Verbose`-ig). Ha pedig interaktívan be akarsz lépni a hibakeresésbe (**postmortem debugging**), használhatod a `%debug` vagy `%pdb` parancsokat.

## 3.3 Fájlok és az Operációs Rendszer (Files and the OS)

Adatelemzésnél a legtöbbször a `pandas.read_csv`-t fogod használni adatbeolvasásra, de alapvető fontosságú érteni, hogyan nyit meg és kezel fájlokat maga a Python.

### Fájl megnyitása és olvasása

A beépített `open()` függvényt használjuk a fájl elérési útjával (path).

```python
In [233]: path = "examples/segismundo.txt"
In [234]: f = open(path, encoding="utf-8")
```

> **Jó gyakorlat:** Mindig add meg az `encoding="utf-8"` paramétert, mert az alapértelmezett kódolás Windowson és Mac/Linux rendszereken eltérő lehet!

Alapból a fájl csak olvasható (`"r"`) módban nyílik meg. A megnyitott fájlobjektumon végig lehet iterálni (pl. egy `for` ciklussal), mintha egy lista lenne, ami soronként olvassa a fájlt:

```python
for line in f:
    print(line)
```

Mivel a sorok végén ott marad a láthatatlan "új sor" (`\n` vagy EOL) karakter, gyakran használjuk az `.rstrip()` string metódust ezek eltávolítására:

```python
In [235]: lines = [x.rstrip() for x in open(path, encoding="utf-8")]
```

### Fájl bezárása és a with blokk

Ha az `open()` függvénnyel nyitsz meg egy fájlt, neked kell gondoskodnod a bezárásáról, hogy visszaadd az erőforrást az operációs rendszernek:

```python
In [237]: f.close()
```

Sokkal jobb és biztonságosabb a `with` kulcsszó használata (ezt hívják **Context Manager**-nek). Ha ezen belül nyitod meg a fájlt, a blokk végén a Python automatikusan bezárja azt, még akkor is, ha közben valamilyen hiba (Exception) történt a kódban!

```python
In [238]: with open(path, encoding="utf-8") as f:
   .....:     lines = [x.rstrip() for x in f]
# Ide érve (a blokkból kilépve) az 'f' fájl már be is záródott.
```

#### Table 3-3. Python fájlkezelési módok (File modes)

| Mód | Leírás (Description) |
| :--- | :--- |
| `r` | Csak olvasható (Read-only) mód (Ez az alapértelmezett). |
| `w` | Csak írható (Write-only) mód. Új fájlt hoz létre. **Veszély:** Ha a fájl már létezik, azonnal és visszavonhatatlanul letörli a tartalmát! |
| `x` | Csak írható mód, új fájlt hoz létre. De ha a fájl már létezik, hibát dob (biztonságosabb, mint a `w`). |
| `a` | Hozzáillesztés (Append). A fájl végére ír anélkül, hogy a régit törölné. Létrehozza a fájlt, ha nem létezik. |
| `r+` | Olvasásra és írásra is megnyitja a fájlt. |
| `b` | Bináris mód (pl. `"rb"` vagy `"wb"`). Nem próbálja a bájtokat Unicode szöveggé alakítani (pl. képek vagy nyers adatok olvasásakor kell). |
| `t` | Szöveges mód (Text). Automatikusan dekódolja a bájtokat Unicode-ra. Ez az alapértelmezett. |

### Fájlok olvasása, pozicionálása és írása

A `.read(darab)` beolvas egy adott mennyiségű karaktert a fájlból (vagy bájtot, ha bináris módban vagyunk).
A `.tell()` megmondja, pontosan hol tartunk (hányadik bájtnál) a fájlban.

```python
In [239]: f1 = open(path)
In [240]: f1.read(10)      # Beolvas 10 karaktert
Out[240]: 'Sueña el r'
In [243]: f1.tell()        # Megnézzük a pozíciót
Out[243]: 11
```

**Magyarázat:** Olvastunk 10 karaktert, de a `.tell()` 11-et mutat. Miért? Mert az `ñ` (ny) betű egy speciális karakter, ami az UTF-8 kódolásban két bájtot foglal el!

A `.seek(pozíció)` paranccsal a kurzort oda ugorathatjuk a fájlban, ahova akarjuk (bájt alapján).

```python
In [247]: f1.seek(3)       # Visszaugrunk a 3. bájthoz
Out[247]: 3
In [248]: f1.read(1)
Out[248]: 'ñ'
```

**Írás fájlba:**
Új fájlt hozunk létre (vagy felülírjuk) `"w"` móddal, és a `.writelines()` segítségével egy iterátorból egyből beleírjuk azokat a sorokat, amik hosszabbak 1 karakternél (kiszűrjük az üres sorokat).

```python
In [253]: with open("tmp.txt", mode="w") as handle:
   .....:     handle.writelines(x for x in open(path) if len(x) > 1)
```

#### Table 3-4. Fontos Python fájl metódusok és attribútumok

| Metódus/Attribútum | Leírás (Description) |
| :--- | :--- |
| `read([size])` | Beolvassa az adatot a fájlból. Ha megadjuk a `size`-t, akkor csak annyi karaktert (vagy bájtot) olvas be. |
| `readable()` | Igaz (`True`), ha a fájl olvasható módban lett megnyitva. |
| `readlines([size])` | Beolvassa a fájlt, és visszaad egy listát, amiben a fájl sorai az elemek. |
| `write(string)` | Beleírja a megadott stringet (szöveget) a fájlba. |
| `writable()` | Igaz (`True`), ha a fájl írható módban lett megnyitva. |
| `writelines(strings)` | Egy szekvenciát (pl. listát) kap, és annak elemeit beleírja a fájlba. |
| `close()` | Bezárja a fájlobjektumot. |
| `flush()` | Kiüríti a belső I/O buffert, és kényszeríti a lemezre történő kiírást. |
| `seek(pos)` | A kurzort az adott (egész szám) bájt pozícióra mozgatja. |
| `seekable()` | Igaz (`True`), ha a fájl támogatja a `seek` műveletet (random access). |
| `tell()` | Visszaadja a jelenlegi kurzorpozíciót (bájtban számolva). |
| `closed` | Attribútum: Igaz (`True`), ha a fájl már be van zárva. |
| `encoding` | Attribútum: Megmutatja, milyen kódolást használ a gép a fájl értelmezéséhez (pl. `utf-8`). |

### Bájtok és Unicode a fájlokkal

Amikor simán, szöveges módban (text mode) nyitsz meg egy fájlt, a Python automatikusan dekódolja a fájlban lévő nyers bájtokat Unicode (`str`) szöveggé.
Ha bináris módban (`"rb"`) nyitod meg, akkor pontosan azt kapod, ami a merevlemezen van: nyers bájtokat.

```python
# Szöveges mód
In [258]: with open(path) as f:
   .....:     chars = f.read(10)
In [259]: chars
Out[259]: 'Sueña el r'

# Bináris mód
In [261]: with open(path, mode="rb") as f:
   .....:     data = f.read(10)
In [262]: data
Out[262]: b'Sue\xc3\xb1a el '
```

**Veszély a `seek`-kel szöveges fájlok esetén:**
Mivel az UTF-8 változó hosszúságú (egy karakter lehet 1, de akár 4 bájt is), a `.seek()` használata nagyon veszélyes! Ha pont egy olyan helyre (bájtra) ugrasz, ami egy kettő bájtból álló karakter "kellős közepe", és utána olvasni próbálsz, a Python összeomlik egy `UnicodeDecodeError` hibával, mert nem tudja értelmezni a "félig elvágott" karaktert.

Erre figyelni kell nem-ASCII karakterek (pl. magyar ékezetek, spanyol `ñ`) kezelésekor!
