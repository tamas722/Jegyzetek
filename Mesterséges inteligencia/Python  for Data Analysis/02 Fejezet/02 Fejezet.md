# 2. Fejezet: Python nyelvi alapok, IPython és Jupyter Notebook (1. rész)

## 2.1 A Python Interpreter (Kódértelmező)

**Fő koncepció:** A Python egy interpretált (értelmezett) nyelv. Ez azt jelenti, hogy a kód nem fordul le előre egy futtatható fájllá (mint pl. C++ esetén egy `.exe`), hanem a Python interpreter (értelmező) a programot sorról sorra olvassa és hajtja végre.

### A standard (alap) Python környezet

Ha a parancssorba (terminálba) beírod, hogy `python`, elindul az interaktív Python környezet. Ezt a `>>>` (prompt) jelzi.

```python
>>> a = 5
>>> print(a)
5
```

**Sorról sorra:**

- `>>> a = 5`: Létrehozunk egy `a` nevű változót, és adunk neki egy `5`-ös értéket. A `>>>` azt jelzi, hogy ide gépelhetünk.
- `>>> print(a)`: Kiíratjuk az `a` változó értékét a képernyőre.

**Kilépés:** Az `exit()` paranccsal, vagy Linux/Mac esetén a `Ctrl-D` billentyűkombinációval léphetünk ki innen.

### Python scriptek (fájlok) futtatása

Ha írunk egy `hello_world.py` fájlt (amiben mondjuk a `print("Hello world")` kód van), azt a terminálból így tudjuk lefuttatni:

```bash
$ python hello_world.py
Hello world
```

(A `$` jel a terminál promptja, azt nem kell begépelni.)

### Az IPython (Interactive Python)

Az adatelemzők ritkán használják a sima `python` környezetet. Helyette az IPythont indítják el (az `ipython` paranccsal a terminálban), amely egy felokosított, kényelmesebb kódértelmező. A prompt itt nem `>>>`, hanem sorszámozott: `In [1]:`.

Egyik nagy előnye, hogy a `%run` paranccsal be tudunk tölteni és le tudunk futtatni külső `.py` fájlokat közvetlenül az IPythonon belül, és utána a fájlban lévő változókkal interaktívan dolgozhatunk tovább.

```python
In [1]: %run hello_world.py
Hello world
```

## 2.2 IPython alapok

### A kimenetek formázása (Pretty-printing)

Az IPython sokkal szebben, olvashatóbban jeleníti meg az adatokat (ezt hívják **pretty-printing**-nek), mint a standard Python `print()` függvénye. Nézzük meg egy példán:

```python
In [5]: import numpy as np
In [6]: data = [np.random.standard_normal() for i in range(7)]
In [7]: data
Out[7]: 
[-0.20470765948471295,
 0.47894333805754824,
 -0.5194387150567381,
 -0.55573030434749,
 1.9657805725027142,
 1.3934058329729904,
 0.09290787674371767]
```

**Sorról sorra:**

- `import numpy as np`: Behozzuk a NumPy könyvtárat numerikus számításokhoz.
- `data = [...]`: Létrehozunk egy `data` nevű Python listát. A lista belsejében egy ún. **list comprehension** (lista-kifejezés) fut: `for i in range(7)` miatt 7-szer lefut a ciklus, és minden lépésben az `np.random.standard_normal()` generál egy véletlen számot normál eloszlásból.
- `data`: Csak beírjuk a változó nevét (nem kell `print(data)`).
- `Out[7]:`: Az IPython automatikusan kiírja az eredményt. Figyeld meg: mivel a lista hosszú, az IPython okosan sortöréseket tesz a számok közé, hogy átlátható legyen. A sima Python egyetlen hosszú, kifolyó sorban írná ki az egészet.

### A Jupyter Notebook

A Jupyter a modern adattudomány alapvető eszköze. Ez egy webes felületű "füzet" (notebook), amelyben egyaránt keverhetünk:

- Futtatható Python kódokat.
- Formázott szövegeket (Markdown nyelv segítségével, ahogy ez a jegyzet is íródott).
- Grafikonokat, képeket és adatvizualizációkat (amelyek rögtön a kód alatt jelennek meg).

**Használata:**

1. Terminálban kiadjuk a `jupyter notebook` parancsot.
2. Ez elindít egy helyi webszervert, és a böngészőnkben (pl. Chrome) megnyílik a Jupyter felülete (általában a `http://localhost:8888` címen).
3. A "New" -> "Python 3" gombbal hozhatunk létre új jegyzetfüzetet.
4. A kódokat a cellákba (code cells) írjuk. Futtatni a `Shift + Enter` billentyűkombinációval lehet őket.

**Fájlformátuma:** `.ipynb` (Interactive Python Notebook).

> **Fontos:** Ha csak bezárod a böngészőfület, a háttérben a Python még futni fog. A rendes leállításhoz a **File -> Close and Halt** opciót kell választani.

### Tab Completion (Tabulátoros kiegészítés)

Ez az egyik leghasznosabb funkció, ami rengeteg gépelést és hibakeresést spórol meg nekünk. Ha gépelés közben megnyomod a `Tab` billentyűt, az IPython (és a Jupyter) megpróbálja kitalálni és kiegészíteni, amit írni akarsz.

**1. Változók kiegészítése:**

```python
In [1]: an_apple = 27
In [2]: an_example = 42
In [3]: an<TAB>
# Ekkor felugrik egy menü a lehetőségekkel:
# an_apple   an_example   any
```

**2. Objektumok metódusainak (függvényeinek) listázása:**
(Saját példa az érthetőségért) Tegyük fel, van egy listád, de nem emlékszel, milyen beépített parancsokkal lehet módosítani.

```python
In [3]: b = [1, 2, 3]
In [4]: b.<TAB>
# A pont (.) beírása és a Tab megnyomása után az IPython kilistázza a lista metódusait:
# append()   count()   insert()   reverse()
# clear()    extend()  pop()      sort()
# copy()     index()   remove()
```

**3. Modulok tartalmának felfedezése:**
Ugyanez működik betöltött könyvtárakra is.

```python
In [1]: import datetime
In [2]: datetime.<TAB>
# Megmutatja a datetime modulban lévő osztályokat és változókat:
# date       MAXYEAR    timedelta
# datetime   MINYEAR    timezone
```

> **Fontos megjegyzés:** A Pythonban a speciális vagy "rejtett" metódusok neve alulvonással `_` kezdődik. Ezeket az IPython alapesetben elrejti, hogy ne zavarja össze a kezdőket. Ha ezeket is látni akarod, a pont után be kell gépelni egy alulvonást, pl.: `b._<TAB>`.

**Egyéb helyek, ahol a Tab működik:**

- **Fájlútvonalak:** Ha elkezdesz írni egy elérési utat macskakörmök között (pl. `dataset = pd.read_csv('adatok/adat...<TAB>`), a Tab kiegészíti a mappádban lévő fájl nevével.
- **Függvény argumentumok (paraméterek):** Ha egy függvény zárójelén belül vagy, a Tab megmutatja, milyen paramétereket lehet megadni a függvénynek.

- # 2. Fejezet: Python nyelvi alapok (2. rész)

## 1. Introspekció (Introspection)

**Fő koncepció:** Az IPythonban egy változó vagy függvény elé/mögé tett kérdőjel (`?`) segítségével azonnal hasznos információkat (típus, belső dokumentáció, forráskód helye) kaphatunk az adott objektumról. Ezt hívjuk **objektum-introspekciónak** (önvizsgálatnak).

**Példa egy egyszerű lista lekérdezésére:**

```python
In [1]: b = [1, 2, 3]
In [2]: b?
Type:       list
String form: [1, 2, 3]
Length:     3
Docstring:
Built-in mutable sequence.
...
```

**Magyarázat:** Létrehozunk egy `b` nevű listát. A `b?` parancs hatására az IPython kiírja, hogy ez egy `list` típusú objektum, 3 elem hosszú, és megmutatja a hozzá tartozó belső dokumentációt (Docstring).

**Példa függvény lekérdezésére:**

```python
In [3]: print?
Docstring:
print(value, ..., sep=' ', end='\n', file=sys.stdout, flush=False)
...
```

**Magyarázat:** A `print?` kiírja a beépített `print` függvény pontos paramétereit és leírását.

**Keresés a névtérben (Wildcard keresés):**
A kérdőjelet a `*` (joker/wildcard) karakterrel kombinálva kereshetünk is a parancsok között:

```python
In [9]: import numpy as np
In [10]: np.*load*?
np.__loader__
np.load
np.loads
np.loadtxt
```

**Magyarázat:** A `np.*load*?` kilistázza a NumPy könyvtár összes olyan függvényét vagy változóját, amelynek a nevében szerepel a "load" szó.

## 2. A Python nyelv alapjai (Language Semantics)

A Python tervezésekor a legfőbb szempont az olvashatóság, az egyszerűség és az egyértelműség volt. Sokan úgy tartják, hogy a Python kód szinte olyan, mint egy "futtatható pszeudokód".

### Behúzások (Indentation), nem kapcsos zárójelek

A C++, Java vagy R nyelvekkel ellentétben a Python **nem használ kapcsos zárójeleket** `{}` a kódblokkok (pl. ciklusok, feltételek) határolására. Ehelyett szóközöket (behúzást) és kettőspontot (`:`) használ.

> **Jó tanács:** Mindig 4 szóközt használj a behúzáshoz, sose Tab-ot!

```python
for x in array:
    if x < pivot:
        less.append(x)
    else:
        greater.append(x)
```

Pontosvesszőt (`;`) sem kell tenni a sorok végére (bár lehet vele több parancsot egy sorba írni, de ez Pythonban kerülendő, mert rontja az olvashatóságot).

### Változók és argumentum-átadás (Nagyon fontos!)

Amikor egy változónak értéket adsz (pl. `a = [1, 2, 3]`), valójában nem magát az adatot tárolod a változóban, hanem létrehozol egy **hivatkozást (referenciát)** a memóriában lévő objektumra.

```python
In [8]: a = [1, 2, 3]
In [9]: b = a
In [10]: b
Out[10]: [1, 2, 3]
```

Más nyelvekkel ellentétben a `b = a` parancs **NEM másolja le a listát!** Az `a` és a `b` most pontosan ugyanarra a memóriaterületre mutat. Ha módosítod az egyiket, változik a másik is:

```python
In [11]: a.append(4)
In [12]: b
Out[12]: [1, 2, 3, 4]
```

> 📍 **[KÉP HELYE: Figure 2-5. Two references for the same object]**

### Erősen típusos nyelv (Strong types)

Bár a változóknak megadáskor nem kell típust adni (dinamikus típusadás), maga a memóriában lévő objektum pontosan tudja, hogy ő milyen típusú. A Python **nem végez automatikus konverziót** (mint pl. a JavaScript).

```python
In [21]: "5" + 5
TypeError: can only concatenate str (not "int") to str
```

**Magyarázat:** Megpróbálunk egy szöveges (String) 5-öst összeadni egy szám (Integer) 5-össel. A Python hibát (`TypeError`) dob, mert nem hajlandó a szöveget titokban számmá alakítani.

### Kacsatípusosság (Duck typing)

Gyakran nem az érdekel minket, hogy egy objektum pontosan milyen típusú (lista, tuple, set), hanem csak az, hogy mit tud csinálni (pl. végig lehet-e iterálni rajta egy for-ciklussal). *"Ha úgy totyog, mint egy kacsa, és úgy hápog, mint egy kacsa, akkor az egy kacsa."*

```python
In [33]: def isiterable(obj):
   ....:     try:
   ....:         iter(obj)
   ....:         return True
   ....:     except TypeError: # not iterable
   ....:         return False
```

**Magyarázat:** Létrehozunk egy függvényt, ami megpróbálja az `iter()` függvényt meghívni az objektumon. Ha sikerül, visszaadja, hogy `True` (iterálható). Ha az objektum ezt nem tudja, `TypeError` hibát dob, amit elkapunk, és visszaadjuk, hogy `False`. (A szövegek és a listák `True`-t, a sima számok `False`-ot adnak).

### Módosítható (Mutable) és nem módosítható (Immutable) objektumok

- **Mutable (Módosítható):** Listák, szótárak (dictionary), NumPy tömbök. Bármikor megváltoztatható az elemük (pl. `a_list[2] = 5`).
- **Immutable (Nem módosítható):** Szövegek (String) és Tuple-ök. Létrehozás után a tartalmuk kőbe van vésve.

```python
In [51]: a_tuple = (3, 5, (4, 5))
In [52]: a_tuple[1] = "four"
# TypeError: 'tuple' object does not support item assignment
```

## 3. Táblázatok: Függvények és Operátorok

### A szövegben bemutatott beépített függvények és metódusok

| Függvény / Metódus | Leírás és működés (Mit csinál?) | Példa a kódból |
| :--- | :--- | :--- |
| `print()` | Kiírja az átadott értéket a képernyőre (alapértelmezetten a konzolra/terminálba). | `print("Hello world")` |
| `.append()` | Lista metódus; hozzáad egy új elemet a meglévő lista legvégéhez (helyben módosít, nem hoz létre új listát). | `a.append(4)` |
| `type()` | Visszaadja egy adott objektum vagy változó pontos Python típusát (pl. `int`, `str`, `list`). | `type(a)` |
| `isinstance()` | Ellenőrzi, hogy egy objektum egy adott típushoz (vagy típusok halmazához) tartozik-e. `True`-t vagy `False`-ot ad vissza. | `isinstance(a, int)` |
| `getattr()` | Egy objektum attribútumát vagy metódusát kéri le dinamikusan, a neve alapján (szöveges formában). Hasznos általános (generic) kód írásakor. | `getattr(a, "split")` |
| `iter()` | Ellenőrzi, hogy egy objektum támogatja-e az iterációs protokollt (azaz lehet-e rajta pl. for ciklussal végigmenni). | `iter(obj)` |
| `list()` | Egy új, üres listát hoz létre, vagy egy meglévő (iterálható) objektumból készít egy új, független listamásolatot. | `c = list(a)` |

### Table 2-1. Bináris operátorok

Itt találhatók a Pythonban használható legfontosabb matematikai és logikai operátorok.

> **Fontos különbség:** Az `==` azt vizsgálja, hogy két változó értéke megegyezik-e. Az `is` azt vizsgálja, hogy a két változó a memóriában fizikailag ugyanaz az objektum-e.

| Operáció (Operation) | Leírás (Description) |
| :--- | :--- |
| `a + b` | Hozzáadja `a`-t `b`-hez (összeadás) |
| `a - b` | Kivonja `b`-t `a`-ból (kivonás) |
| `a * b` | Megszorozza `a`-t `b`-vel (szorzás) |
| `a / b` | Elosztja `a`-t `b`-vel (osztás - eredménye lebegőpontos szám) |
| `a // b` | Egész osztás: Elosztja `a`-t `b`-vel, és eldobja a törtrészt (maradékot) |
| `a ** b` | Hatványozás: `a`-t a `b`-edik hatványra emeli |
| `a & b` | Igaz (`True`), ha `a` és `b` is Igaz; Egész számoknál bitenkénti ÉS (AND) művelet |
| `a \| b` | Boole-értékeknél Igaz, ha `a` vagy `b` Igaz; Egész számoknál bitenkénti VAGY (OR) művelet |
| `a ^ b` | Boole-értékeknél Igaz, ha `a` vagy `b` Igaz, de nem mindkettő; Egész számoknál bitenkénti KIZÁRÓ-VAGY (XOR) |
| `a == b` | Igaz (`True`), ha `a` értéke egyenlő `b` értékével |
| `a != b` | Igaz (`True`), ha `a` értéke nem egyenlő `b` értékével |
| `a < b`, `a <= b` | Igaz (`True`), ha `a` kisebb (kisebb vagy egyenlő), mint `b` |
| `a > b`, `a >= b` | Igaz (`True`), ha `a` nagyobb (nagyobb vagy egyenlő), mint `b` |
| `a is b` | Igaz (`True`), ha `a` és `b` ugyanarra a Python objektumra hivatkozik a memóriában |
| `a is not b` | Igaz (`True`), ha `a` és `b` különböző Python objektumokra hivatkozik |


# 2. Fejezet: Python nyelvi alapok (3. rész)

## 1. Skalár típusok (Scalar Types)

A Python rendelkezik egy szűk készlettel a számok, szövegek, logikai (Igaz/Hamis) értékek, valamint dátumok és idők kezelésére. Ezeket az "egyetlen értéket" tartalmazó típusokat nevezzük **skalár típusoknak**.

### Table 2-2. Standard Python skalár típusok

| Típus (Type) | Leírás (Description) |
| :--- | :--- |
| `None` | A Python "null" (üres/semmi) értéke. (A `None` objektumból csak egyetlen egy létezik a memóriában.) |
| `str` | Szöveges (String) típus; Unicode (univerzális) karaktereket tárol. |
| `bytes` | Nyers bináris adatok tárolására. |
| `float` | Dupla pontosságú lebegőpontos szám (törtszám). A Pythonban nincs külön "double" típus. |
| `bool` | Logikai Igaz (`True`) vagy Hamis (`False`) érték. |
| `int` | Tetszőleges hosszúságú (pontosságú) egész szám. |

### Numerikus típusok (Számok)

A két legfontosabb számtípus az `int` (egész) és a `float` (tört). A Pythonban az egész számok bármilyen nagyok lehetnek.

```python
In [53]: ival = 17239871
In [54]: ival ** 6
Out[54]: 26254519291092456596965462913230729701102721
```

**Magyarázat:** Létrehozunk egy nagy egész számot, majd a `**` operátorral a 6. hatványra emeljük. A Python hiba nélkül kezeli az extrém nagy számokat is.

```python
In [55]: fval = 7.243
In [56]: fval2 = 6.78e-5
```

**Magyarázat:** A float számokat leírhatjuk tizedesponttal, vagy tudományos (exponenciális) formátumban is (a `6.78e-5` jelentése $6.78 \times 10^{-5}$, azaz $0.0000678$).

**Osztás:**

```python
In [57]: 3 / 2
Out[57]: 1.5

In [58]: 3 // 2
Out[58]: 1
```

**Magyarázat:**
- A sima osztás (`/`) mindig lebegőpontos számot (float) ad eredményül.
- A dupla perjel (`//`, ún. **floor division**) eldobja a törtrészt, és csak az egész eredményt adja vissza (C-stílusú osztás).

### Szövegek (Strings)

Szövegeket szimpla (`'`) vagy dupla (`"`) idézőjelekkel is megadhatunk.

```python
a = 'one way of writing a string'
b = "another way"
```

Többsoros szövegekhez tripla idézőjelet (`"""` vagy `'''`) használunk:

```python
c = """
This is a longer string that
spans multiple lines
"""
In [60]: c.count("\n")
Out[60]: 3
```

**Magyarázat:** A szöveg megadásakor az enterek (sortörések) is bekerülnek a változóba. A `.count("\n")` megszámolja az "új sor" (`\n`) karaktereket a szövegben, amiből 3 darab van (egy az első tripla idézőjel után, egy a `lines` szó végén, egy pedig az utolsó tripla idézőjel előtt).

**A szövegek módosíthatatlanok (immutable):**

```python
In [61]: a = "this is a string"
In [62]: a[10] = "f"
# TypeError: 'str' object does not support item assignment
```

**Magyarázat:** Nem cserélhetjük le egy szöveg adott karakterét. Ha változtatni akarunk, új szöveget kell generálnunk, például a `.replace()` metódussal:

```python
In [63]: b = a.replace("string", "longer string")
In [64]: b
Out[64]: 'this is a longer string'
In [65]: a
Out[65]: 'this is a string'
```

**Magyarázat:** A `.replace(mit, mire)` egy vadonatúj szöveget hoz létre, és azt menti a `b` változóba. Az eredeti `a` változó érintetlen marad.

**Szövegek szeletelése (Slicing) és listává alakítása:**

```python
In [69]: s = "python"
In [70]: list(s)
Out[70]: ['p', 'y', 't', 'h', 'o', 'n']
In [71]: s[:3]
Out[71]: 'pyt'
```

**Magyarázat:** A `list()` betűnként darabolja a szöveget. A `s[:3]` szintaxis (szeletelés) az első 3 karaktert vágja ki (a 0., 1., 2. indexűeket).

**Speciális karakterek (Escape) és nyers (Raw) szövegek:**
A `\` (visszaper) arra való, hogy speciális karaktereket jelezzünk vele (pl. `\n` = új sor). Ha magát a `\` jelet akarjuk leírni, duplázni kell: `\\`.

```python
In [72]: s = "12\\34"
In [73]: print(s)
12\34
```

Ha sok visszaperünk van (pl. Windows mappaútvonalak), tehetünk egy `r` (raw = nyers) betűt az idézőjel elé, ami kikapcsolja a speciális karakterek értelmezését:

```python
In [74]: s = r"this\has\no\special\characters"
In [75]: s
Out[75]: 'this\\has\\no\\special\\characters'
```

**Szövegek összefűzése (Concatenation):**

```python
In [76]: a = "this is the first half "
In [77]: b = "and this is the second half"
In [78]: a + b
Out[78]: 'this is the first half and this is the second half'
```

**Magyarázat:** A `+` jellel a szövegek egyszerűen egymáshoz tapadnak.

**Szövegformázás (String formatting) - `.format()`:**

```python
In [79]: template = "{0:.2f} {1:s} are worth US${2:d}"
In [80]: template.format(88.46, "Argentine Pesos", 1)
Out[80]: '88.46 Argentine Pesos are worth US$1'
```

**Magyarázat:** Készítünk egy "sablont", ahol a kapcsos zárójelek helyére értékek kerülnek majd.
- `{0:.2f}`: A 0. argumentumot (88.46) lebegőpontos számként (`f`), 2 tizedesjegy pontossággal (`.2`) írja ki.
- `{1:s}`: Az 1. argumentumot ("Argentine Pesos") sima szövegként (`s`) illeszti be.
- `{2:d}`: A 2. argumentumot (1) pontos egész számként (`d`) illeszti be.

**Szövegformázás (f-strings) - Az új és ajánlott módszer (Python 3.6+):**
Sokkal kényelmesebb, mert közvetlenül a kapcsos zárójelekbe írhatjuk a változókat, ha a szöveg elé egy `f` betűt teszünk.

```python
In [81]: amount = 10
In [82]: rate = 88.46
In [83]: currency = "Pesos"
In [84]: result = f"{amount} {currency} is worth US${amount / rate}"
In [85]: f"{amount} {currency} is worth US${amount / rate:.2f}"
Out[85]: '10 Pesos is worth US$0.11'
```

**Magyarázat:** A kapcsos zárójelek között a Python azonnal kiértékeli a matematikai műveletet (`amount / rate`), a `:.2f` pedig kerekíti az eredményt két tizedesjegyre.

### Bájt (Bytes) és Unicode

A modern Pythonban a normál szöveg (`str`) Unicode-ban tárolódik (így tud kezelni ékezeteket, emojikat). Ha ezt ki akarjuk írni mondjuk egy fájlba, gyakran át kell alakítani bináris (bájt) adatsorrá az `.encode()` segítségével. Visszafelé pedig a `.decode()` kell.

```python
In [86]: val = "español"
In [88]: val_utf8 = val.encode("utf-8")
In [89]: val_utf8
Out[89]: b'espa\xc3\xb1ol'
```

**Magyarázat:** Az `encode` átalakította a szöveget nyers bájtokká (`b` betű jelzi az elején). Az `ñ` betűből két speciális hexadecimális bájt (`\xc3\xb1`) lett a memóriában. Visszaalakítás:

```python
In [91]: val_utf8.decode("utf-8")
Out[91]: 'español'
```

### Logikai értékek (Booleans)

Értékük `True` (Igaz) vagy `False` (Hamis). Az `and` (és), `or` (vagy), illetve `not` (tagadás) kulcsszavakkal kombinálhatjuk őket.

```python
In [95]: True and True
Out[95]: True
In [96]: False or True
Out[96]: True
In [99]: a = True
In [101]: not a
Out[101]: False
```

Számmá alakítva a `True` 1, a `False` pedig 0:

```python
In [97]: int(False)
Out[97]: 0
In [98]: int(True)
Out[98]: 1
```

### Típuskonverzió (Type casting)

A beépített típusneveket függvényként használva konvertálhatunk értékeket:

```python
In [103]: s = "3.14159"
In [104]: fval = float(s)    # Szövegből tört
In [106]: int(fval)          # Törtből egész (levágja a törtrészt: 3)
In [107]: bool(fval)         # Törtből logikai (Minden nem nulla szám True lesz)
In [108]: bool(0)            # A nulla mindig False
```

### A None objektum

A `None` egy speciális típus, ami az "ürességet" jelöli.

```python
In [109]: a = None
In [110]: a is None
Out[110]: True
```

Gyakran használják függvényeknél alapértelmezett (default) értékként, jelezve, hogy az argumentum opcionális:

```python
def add_and_maybe_multiply(a, b, c=None):
    result = a + b
    if c is not None:
        result = result * c
    return result
```

**Magyarázat:** Ha a felhasználó megadja a `c` értékét (tehát az `is not None`), akkor megszorozzuk az összeget `c`-vel.

### Dátumok és idők (datetime modul)

A `datetime` a Python beépített időkezelője.

```python
In [113]: from datetime import datetime, date, time
In [114]: dt = datetime(2011, 10, 29, 20, 30, 21)
In [115]: dt.day
Out[115]: 29
In [116]: dt.minute
Out[116]: 30
```

**Magyarázat:** Létrehoztunk egy időbélyeget (Év, Hónap, Nap, Óra, Perc, Másodperc). Az attribútumokat (pl. `dt.day`) külön le tudjuk kérni.

**Szöveggé formázás és szövegből beolvasás:**

```python
In [119]: dt.strftime("%Y-%m-%d %H:%M")
Out[119]: '2011-10-29 20:30'

In [120]: datetime.strptime("20091031", "%Y%m%d")
Out[120]: datetime.datetime(2009, 10, 31, 0, 0)
```

**Magyarázat:** A `.strftime` (format time) szöveget készít a dátumból a megadott minta alapján (`%Y`=4 számjegyű év, `%m`=hónap stb.). A `.strptime` (parse time) fordítva: a szöveget alakítja vissza `datetime` objektummá.

**Dátumok módosítása (`.replace`) és matematikai műveletek (`timedelta`):**
A `datetime` is immutable, így a `.replace()` egy új objektumot hoz létre:

```python
In [121]: dt_hour = dt.replace(minute=0, second=0)
In [122]: dt_hour
Out[122]: datetime.datetime(2011, 10, 29, 20, 0)
```

Ha kivonunk egymásból két dátumot, egy `timedelta` (időkülönbség) objektumot kapunk:

```python
In [124]: dt2 = datetime(2011, 11, 15, 22, 30)
In [125]: delta = dt2 - dt
In [126]: delta
Out[126]: datetime.timedelta(days=17, seconds=7179)
```

**Magyarázat:** Eltelt 17 nap és 7179 másodperc. Ezt a deltát hozzá is lehet adni dátumokhoz (`dt + delta`), ami kitolja az időt a jövőbe.

## 2. Vezérlési szerkezetek (Control Flow)

### if, elif, else (Feltételek)

A Python alapvető elágazás-kezelője.

```python
x = -5
if x < 0:
    print("It's negative")
elif x == 0:
    print("Equal to zero")
elif 0 < x < 5:
    print("Positive but smaller than 5")
else:
    print("Positive and larger than or equal to 5")
```

**Magyarázat:** A kód fentről lefelé fut. Amelyik feltétel először `True` (igaz) lesz, annak a blokkja fut le, a többit a gép átugorja. Különlegesség, hogy Pythonban matematikai módon láncolhatjuk a feltételeket: `0 < x < 5` teljesen érvényes kód!

**Láncolt és "Rövidzáras" (short-circuit) kiértékelés:**

```python
In [130]: a = 5; b = 7
In [131]: c = 8; d = 4
In [132]: if a < b or c > d:
   .....:     print("Made it")
```

**Magyarázat:** Az `or` (vagy) műveletnél a gép balról jobbra halad. Mivel az `a < b` (5 < 7) IGAZ, a rendszer tudja, hogy a teljes kifejezés biztosan igaz lesz, így a `c > d` feltételt meg sem vizsgálja (ez a rövidzár).

**Láncolás extrém példa:**

```python
In [133]: 4 > 3 > 2 > 1
Out[133]: True
```

### for ciklusok (for loops)

Egy gyűjtemény (pl. lista, tuple) elemein megy végig.

```python
sequence = [1, 2, None, 4, None, 5]
total = 0
for value in sequence:
    if value is None:
        continue
    total += value
```

**Magyarázat:** A `continue` parancs azt mondja, hogy ugorjunk a ciklus következő iterációjára (lépésére), az alatta lévő részt hagyjuk ki. Itt: ha az elem `None`, nem adja hozzá a `total` összeghez, hanem ugrik a következő elemre.

A `break` parancs azonnal megszakítja a teljes ciklust:

```python
sequence = [1, 2, 0, 4, 6, 5, 2, 1]
total_until_5 = 0
for value in sequence:
    if value == 5:
        break
    total_until_5 += value
```

**Magyarázat:** Amint a ciklus meglátja az 5-ös számot, teljesen kilép, és nem ad hozzá több számot a `total`-hoz. (Ha több egymásba ágyazott `for` ciklusunk van, a `break` csak a legbelsőt töri meg).

### while ciklusok

Addig fut, amíg a feltétele Igaz (`True`).

```python
x = 256
total = 0
while x > 0:
    if total > 500:
        break
    total += x
    x = x // 2
```

**Magyarázat:** Minden lépésben hozzáadjuk az `x`-et a `total`-hoz, majd az `x`-et elosztjuk kettővel. Ha a `total` eléri az 500-at, megszakítjuk a ciklust.

### pass

A `pass` a "ne csinálj semmit" (no-op) parancs. Mivel a Python a behúzásokra támaszkodik, egy `if` vagy `for` blokk nem maradhat üresen. Ide rakjuk a `pass`-t, ha később akarjuk megírni a kódot.

```python
if x < 0:
    print("negative!")
elif x == 0:
    # TODO: put something smart here
    pass
else:
    print("positive!")
```

### range függvény

A `range` egyenletesen elosztott egész számok generálására szolgál. Változatai: `range(vég)`, `range(kezdet, vég)`, `range(kezdet, vég, lépésköz)`.

```python
In [136]: list(range(10))
Out[136]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

In [137]: list(range(0, 20, 2))
Out[137]: [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

In [138]: list(range(5, 0, -1))
Out[138]: [5, 4, 3, 2, 1]
```

**Magyarázat:** Figyeld meg, hogy a végső szám sosem kerül bele a kimenetbe! (A `range(10)` csak 9-ig megy).

Memóriagazdálkodás szempontjából a `range` zseniális:

```python
In [141]: total = 0
In [142]: for i in range(100_000):
   .....:     # % is the modulo operator
   .....:     if i % 3 == 0 or i % 5 == 0:
   .....:         total += i
In [143]: print(total)
2333316668
```

**Magyarázat:** Bár a `range(100_000)` százezer számot jelent, a gép nem hoz létre a memóriában egy 100 000 elemű listát! A számokat mindig csak röptében (futásidőben) generálja le az adott cikluslépésnél, így a memóriahasználata minimális marad. A `100_000` megadásában az alulvonás csak az ezreselválasztó (olvashatóságot segíti), a gép `100000`-ként kezeli. A `%` (modulo) a maradékos osztás operátora: ha a 3-mal vett osztás maradéka 0 (tehát osztható 3-mal), akkor hozzáadja az összeghez.
