# 1. Fejezet: Bevezetés (Preliminaries)

## 1.1 Miről szól ez a könyv?

A könyv fókuszában a strukturált adatok Pythonban történő manipulálása, feldolgozása, tisztítása és elemzése (ún. **data wrangling** vagy **munging**) áll. Bár a címben szerepel az "adatelemzés" (data analysis), a könyv nem a statisztikai vagy gépi tanulási módszertanokra, hanem a mindennapi munkához szükséges Python programozásra, könyvtárakra és eszközökre fókuszál.

**Milyen adatokat fogunk használni? (Strukturált adatok)**

- Táblázatos (spreadsheet-szerű) adatok (pl. SQL táblák, CSV fájlok), ahol az oszlopok különböző típusúak lehetnek (szöveg, szám, dátum).
- Többdimenziós tömbök (mátrixok).
- Egymáshoz kulcsokkal (primary/foreign key) kapcsolódó táblák.
- Idősoros adatok (egyenletes vagy egyenetlen térközzel).

## 1.2 Miért a Python az adatelemzéshez?

A Python 1991-es megjelenése óta az egyik legnépszerűbb nyelvvé vált, különösen a tudományos számítástechnika és az adattudomány (Data Science) területén. Ennek főbb okai:

- **A Python mint "ragasztó" nyelv (Glue language):** A Python kiválóan alkalmas C, C++ vagy FORTRAN kódok integrálására. Gyakori, hogy a teljesítmény-kritikus matematikai számítások mélyén gyors C kód fut, amit a Python csak összefog ("összeragaszt").
- **A "Kétnyelvű" probléma megoldása:** Korábban a cégek gyakran R-t vagy SAS-t használtak a kutatásra és prototípus-gyártásra, majd Java-t vagy C++-t az éles (produkciós) rendszerhez. A Python mindkettőre kiváló, így elég egyetlen nyelvet és környezetet fenntartani.

**Mikor NE használjunk Pythont?**

- Mivel interpretált nyelv, lassabb, mint a lefordított nyelvek (pl. C++). Ha a mikroszekundumos késleltetés (low-latency) kritikus (pl. magas frekvenciájú tőzsdei kereskedésnél), a Python nem a legjobb választás.
- Nagyon intenzív, több CPU szálat igénylő (multithreaded) feladatoknál a Python **GIL (Global Interpreter Lock)** mechanizmusa akadályt jelenthet, mert ez megakadályozza, hogy az interpreter egyszerre több Python utasítást hajtson végre.

## 1.3 Alapvető Python könyvtárak (Eszköztár)

A könyv az alábbi alapvető eszközökre épít:

- **NumPy:** A numerikus számítások alapja. Hatékony többdimenziós tömböket (ndarray) biztosít, és eszközt nyújt a gyors matematikai műveletekhez.
- **pandas:** A könyv főszereplője. Táblázatos adatszerkezeteket (DataFrame és Series) kínál, amelyekkel intuitív módon lehet adatokat szűrni, tisztítani, csoportosítani és manipulálni.
- **matplotlib:** A legnépszerűbb 2D adatvizualizációs és grafikonkészítő könyvtár.
- **IPython és Jupyter:** Interaktív fejlesztői környezetek. Lehetővé teszik a "futtatás-felfedezés" (execute-explore) munkafolyamatot, a Jupyter notebookok pedig kód, szöveg és grafikonok együttes megjelenítését is biztosítják.
- **SciPy:** Tudományos számításokhoz (pl. integrálás, lineáris algebra, statisztika, jelfeldolgozás) használt csomag.
- **scikit-learn:** A Python elsőszámú általános gépi tanulási (Machine Learning) könyvtára.
- **statsmodels:** Klasszikus statisztikai és ökonometriai modellezéshez (pl. regressziók, varianciaanalízis, idősorok) használt eszköz.

## 1.4 Telepítés és beállítás

A szerző a **Miniconda** használatát javasolja (amely a conda csomagkezelő minimalista verziója) a **conda-forge** csatornával. Ez biztosítja, hogy minden függőség a megfelelő verzióban kerüljön a gépünkre. (Megjegyzés: a könyv a Python 3.10-es verzióját használja alapértelmezetten).

### Kódpéldák és magyarázatok

A fejezet végén a szerző bemutat néhány alapvető importálási és beállítási konvenciót, amelyeket az egész könyv során használni fog.

#### 1. Kimeneti formázás beállítása (Display Options)

Amikor a Jupyter Notebookban adatokat íratunk ki, a pandas és a NumPy alapértelmezett beállításai néha túl sok vagy túl kevés adatot mutatnak, esetleg nehezen olvasható formátumban (pl. exponenciális számábrázolás). Az alábbi kód ezt teszi szebbé.

```python
import numpy as np
import pandas as pd

pd.options.display.max_columns = 20
pd.options.display.max_rows = 20
pd.options.display.max_colwidth = 80
np.set_printoptions(precision=4, suppress=True)
```

**Sorról sorra történő magyarázat:**

- `import numpy as np`: Beimportálja a NumPy könyvtárat a programba, és ad neki egy rövidebb álnév/aliast (`np`). Ez egy iparági standard, így nem kell mindig leírni, hogy `numpy.függvény`.
- `import pandas as pd`: Beimportálja a pandas könyvtárat `pd` néven. Ez szintén univerzális konvenció.
- `pd.options.display.max_columns = 20`: Utasítja a pandast, hogy ha kiíratunk egy nagy táblázatot (DataFrame-et), maximum 20 oszlopot jelenítsen meg a képernyőn (a többit pontokkal `...` helyettesíti, hogy ne follyon ki a monitorról).
- `pd.options.display.max_rows = 20`: Ugyanez sorokra; maximum 20 sort fog kinyomtatni a táblázatból (általában az első és utolsó 10-et).
- `pd.options.display.max_colwidth = 80`: Egyetlen oszlop maximális szélességét 80 karakterben határozza meg. Ha egy cellában ennél hosszabb szöveg van, azt levágja.
- `np.set_printoptions(precision=4, suppress=True)`: A NumPy tömbök megjelenítését formázza meg. A `precision=4` azt jelenti, hogy a lebegőpontos számokat (törtszámokat) 4 tizedesjegyre kerekítve mutassa. A `suppress=True` pedig elnyomja a tudományos (exponenciális) jelölést (pl. `1.5e-4` helyett `0.00015`-öt fog kiírni), így sokkal olvashatóbbak lesznek a számok.

#### 2. Importálási konvenciók (Import Conventions)

A Python adat-ökoszisztémájában szigorú, közösség által elfogadott rövidítéseket (aliasokat) használunk. A rossz gyakorlat (pl. `from numpy import *`) elkerülése végett mindig prefixekkel hivatkozunk a modulokra.

```python
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns
import statsmodels as sm
```

**Sorról sorra történő magyarázat:**

- `import numpy as np`: A NumPy hivatkozása `np`-ként. Példa a használatra később: `np.array([1, 2, 3])`.
- `import matplotlib.pyplot as plt`: A matplotlib grafikonrajzoló modulját (`pyplot`) `plt` néven importáljuk. Ezzel fogunk ábrákat készíteni. Példa: `plt.plot(x, y)`.
- `import pandas as pd`: A pandas hivatkozása `pd`-ként.
- `import seaborn as sns`: A seaborn egy haladó, szebb grafikonokat készítő vizualizációs könyvtár, amit `sns` néven szokás importálni (a rövidítés a Samuel Norman Seaborn nevű fiktív karakterre utal, amiből a könyvtár a nevét kapta).
- `import statsmodels as sm`: A statisztikai modellező könyvtár importálása `sm` néven.
