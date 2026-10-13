# 1. Bevezetés a gépi tanulásba (Machine Learning)

## 1.1 Mi az a gépi tanulás?

A gépi tanulás (Machine Learning, ML) a számítástechnika azon ága, ahol nem kézzel programozzuk le az összes logikai feltételt, hanem mintákból és adatokból építünk statisztikai modelleket a problémák megoldására.

A folyamat két alapvető lépésből áll:

- **Adatgyűjtés:** Egy adott jelenséget leíró adathalmaz összeállítása (ez származhat természetes mérésekből, kézi címkézésből vagy szimulációkból).
- **Modellépítés:** Egy algoritmus segítségével statisztikai modellt illesztünk az adatokra, amely képes új, korábban nem látott bemenetek esetén is érvényes kimenetet adni.

## 1.2 A tanulási formák típusai

A feladat jellege és az elérhető adatok alapján négy fő tanulási kategóriát különböztetünk meg:

1. Felügyelt tanulás (Supervised Learning)
2. Félig felügyelt tanulás (Semi-Supervised Learning)
3. Felügyelet nélküli tanulás (Unsupervised Learning)
4. Megerősítéses tanulás (Reinforcement Learning)

### 1.2.1 Felügyelt tanulás (Supervised Learning)

A felügyelt tanulás során a tanító adathalmaz minden egyes bemeneti mintájához tartozik egy előre ismert kimeneti érték, azaz egy **címke** (label vagy ground truth). A cél egy olyan függvénykapcsolat megtanulása, amely az új bemenetekhez a lehető legpontosabban rendeli hozzá a helyes címkét.

#### Matematikai formalizmus és jelölésrendszer

**Adathalmaz:**

$$\Large D = \{(x_i, y_i)\}_{i=1}^N$$

Ahol:

- $N \in \mathbb{N}$  A tanítópéldák száma az adathalmazban.
- $i \in \{1, 2, \dots, N\}$: Az adott példa indexe (alsó index jelöli).

**Jellemzővektor (Feature Vector):**

Minden minta egy `D`-dimenziós vektor a valós térben:

$$\large x_i \in \mathbb{R}^D$$


Komponensenként felírva:

$$\large x_i = \begin{bmatrix} x_i^{(1)} \\ x_i^{(2)} \\ \vdots \\ x_i^{(D)} \end{bmatrix}$$


A felső index zárójelben, $x^{(j)}$, a vektor $j$-edik attribútumát (jellemzőjét) jelöli ($j \in \{1, 2, \dots, D\}$).

**Konzisztencia-elv:** A jellemzővektor struktúrája és jelentése minden példában szigorúan kötött. Ha az $x_1^{(2)}$ elem a testtömeget tárolja, akkor az összes $x_k^{(2)}$ ($k \in \{1, 2, \dots, N\}$) elemnek a testtömeget kell tartalmaznia.

**Címke ($y_i$):**

A kimenet típusa határozza meg a gépi tanulási feladat jellegét:

- **Osztályozás (Classification):** A címke egy véges diszkrét halmaz eleme:

  $$y_i \in \{1, 2, \dots, C\}, \quad C \in \mathbb{N}, \quad i \in \{1, 2, \dots, N\}$$

  (Kétosztályos esetben gyakran: $y_i \in \{0, 1\}$ vagy $y_i \in \{-1, +1\}$).

- **Regresszió (Regression):** A címke folytonos valós szám:

  $$y_i \in \mathbb{R}, \quad i \in \{1, 2, \dots, N\}$$

- **Strukturált kimenet:** Összetettebb struktúrák (vektor, mátrix, gráf, szekvencia).

**A feladat matematikai célja:**

Egy olyan hipotézisfüggvény vagy modell megtalálása:

$$f: \mathbb{R}^D \rightarrow \mathcal{Y}, \quad D \in \mathbb{N}$$

amely tetszőleges új $x \in \mathbb{R}^D$ vektorra olyan $\hat{y} = f(x)$ becslést ad, amelyre a valódi címkétől való eltérés minimális.

#### Részletes számpélda: Ingatlanár-becslés és vevőminősítés

Nézzük meg a fenti formális jelöléseket egy gyakorlati példán keresztül, ahol 3 lakás adatait rögzítettük ($N=3$, így $i \in \{1, 2, 3\}$).

Minden lakást $D = 3$ darab jellemzővel írunk le ($j \in \{1, 2, 3\}$):

- $x^{(1)}$: Alapterület (m²)
- $x^{(2)}$: Szobák száma (db)
- $x^{(3)}$: Távolság a belvárostól (km)

A jellemzővektor tehát:

$$x = \begin{bmatrix} x^{(1)} \\ x^{(2)} \\ x^{(3)} \end{bmatrix} \in \mathbb{R}^3$$

Tegyük fel, hogy az adathalmazunk a következő mérésekből áll:

1. lakás: 50 m², 2 szoba, 3 km távolság.

```math
\large
x_1 = \begin{bmatrix} 50 \\ 2 \\ 3 \end{bmatrix}
```

2. lakás: 75 m², 3 szoba, 8 km távolság.

```math
\large
x_2 = \begin{bmatrix} 75 \\ 3 \\ 8 \end{bmatrix}
```

3. lakás: 35 m², 1 szoba, 1 km távolság.

```math
\large
x_3 = \begin{bmatrix} 35 \\ 1 \\ 1 \end{bmatrix}
```

##### 1. Eset: Regressziós feladat (Ingatlanár becslése)

A kimenet itt egy folytonos valós szám ($y_i \in \mathbb{R}$, millió Ft-ban):

- $y_1 = 45$
- $y_2 = 62$
- $y_3 = 38$

A tanítóhalmaz formális felírása:

```math
D_{\text{regr}} = \left\{
\left( \begin{bmatrix} 50 \\ 2 \\ 3 \end{bmatrix}, 45 \right),
\left( \begin{bmatrix} 75 \\ 3 \\ 8 \end{bmatrix}, 62 \right),
\left( \begin{bmatrix} 35 \\ 1 \\ 1 \end{bmatrix}, 38 \right)
\right\}
```





#### 2. Eset: Bináris osztályozási feladat (Befektetési ajánlás)

Tegyük fel, hogy kategorizálni szeretnénk a lakásokat: megéri megvenni kiadásra ($\large 1$), vagy nem éri meg ($\large 0$):

- $\large y_1 = 1$ (megéri)
- $\large y_2 = 0$ (túl messze van / drága)
- $\large y_3 = 1$ (könnyen kiadható kis garzon)

A címkehalmaz: $\large y_i \in \{0, 1\}, \quad i \in \{1, 2, 3\}$.

```math
\large
D_{\text{class}} = \left\{
\left( \begin{bmatrix} 50 \\ 2 \\ 3 \end{bmatrix}, 1 \right),
\left( \begin{bmatrix} 75 \\ 3 \\ 8 \end{bmatrix}, 0 \right),
\left( \begin{bmatrix} 35 \\ 1 \\ 1 \end{bmatrix}, 1 \right)
\right\}
```

##### Következtetés (Inference)

Ha érkezik egy új lakás:

```math
\large
x_{\text{új}} = \begin{bmatrix} 60 \\ 2 \\ 2 \end{bmatrix}
```

A betanított modell feladata kiszámítani az $\large f(x_{\text{új}})$ értéket, ami regressziónál egy becsült ár (pl. $\large 52.4 \text{ M Ft}$), osztályozásnál pedig egy döntési érték vagy valószínűség (pl. $\large P(y=1 \mid x_{\text{új}}) = 0.87$).

## 1.2.2 Felügyelet nélküli tanulás (Unsupervised Learning)

A felügyelet nélküli tanulás esetében a gép nem kap előre megadott "helyes válaszokat", azaz az adatokhoz nem tartoznak címkék.

**Matematikai háttér és jelölés:**

Az adathalmaz formálisan:

```math
\large
D = \{x_i\}_{i=1}^{N}, \quad N \in \mathbb{N}, \quad i \in \{1, 2, \dots, N\}
```

- $\large N$: A tanítópéldák összesített száma ($\large N \in \mathbb{N}$).
- $\large i$: Az adott példa sorszáma, indexe ($\large i \in \{1, 2, \dots, N\}$).
- $\large x_i$: A bemeneti jellemzővektor (feature vector), ahol $\large x_i \in \mathbb{R}^D, \quad D \in \mathbb{N}$.

**Fontos különbség:** Itt hiányzik az $\large y_i$ érték (a címke), amely a felügyelt tanulásnál jelen volt.

**A tanulás célja:**

A modell bemenetként megkapja az $\large x$ jellemzővektort, és átalakítja azt egy másik vektorrá vagy egy értékké, amellyel egy adott gyakorlati probléma megoldhatóvá válik.

**Alkalmazási példák és számpéldák (a szöveg alapján):**

### Klaszterezés (Clustering)

A modell felismeri az adatok belső struktúráját, és minden $\large x$ vektorhoz visszaad egy "klaszter azonosítót" (egy csoport azonosítóját).

**Példa:** Az adathalmaz vásárlók adatait tartalmazza. A modell felismeri, hogy két fő csoport van. A bemeneti

```math
\large
x = \begin{bmatrix} 25 \text{ éves} \\ 100000 \text{ Ft költés} \end{bmatrix}
```

vektorra a modell visszaadja, hogy ez a vásárló az $\large 1$-es azonosítójú (fiatal sokat költő) csoportba tartozik.

### Dimenziócsökkentés (Dimensionality reduction)

A kimenet egy olyan új jellemzővektor, amelynek kevesebb jellemzője (eleme) van, mint az eredeti bemeneti `x` vektornak, mégis megtartja a lényegi információkat.

**Példa:** Egy háromdimenziós bemeneti vektorból

```math
\large
x = \begin{bmatrix} 10 \\ 5 \\ 8 \end{bmatrix}
```

a modell létrehoz egy kétdimenziós transzformált vektort

```math
\large
x' = \begin{bmatrix} 7.5 \\ 4.2 \end{bmatrix}
```

ami számításigényesebb algoritmusoknál gyorsítja a feldolgozást.

### Kiugró értékek (anomáliák) detektálása (Outlier detection)

A kimenet egy valós szám, amely azt mutatja meg, hogy az adott $\large x$ példa mennyire tér el az adathalmaz "tipikus" (átlagos) példáitól.

**Példa:** Egy bankkártyás fizetési tranzakció ($\large x$) vizsgálatakor a modell kimenete $\large 0.1$ (tipikus tranzakció) vagy $\large 0.95$ (nagyon szokatlan, valószínűleg csalás).

## 1.2.3 Félig felügyelt tanulás (Semi-Supervised Learning)

**Az adathalmaz összetétele:** Ez a módszer címkézett és címkézetlen példákat is tartalmaz vegyesen.

**Mennyiségi arányok:** A gyakorlatban a címkézetlen adatok mennyisége általában sokkal nagyobb, mint a címkézett adatoké (mivel nyers adatot könnyű gyűjteni, de felcímkézni drága és időigényes).

**A tanulás célja:** Pontosan ugyanaz, mint a felügyelt tanulásnál (egy jól prediktáló modell építése).

**A koncepció lényege:** A rengeteg címkézetlen adat felhasználásával az algoritmus a remények szerint egy jobb modellt tud kiszámítani/létrehozni, mintha kizárólag a kis mennyiségű címkézett adatot használná fel.

## 1.2.4 Megerősítéses tanulás (Reinforcement Learning)

Bár erről korábban már volt szó, az új szövegrészlet alapján a pontosított, formális definíció a következő:

**A rendszer elemei és működése:** Egy ágens (agent) interakcióba lép a környezettel. A környezet pillanatnyi állapotát (state) egy jellemzővektorként érzékeli.

**Akciók és következmények:** Az ágens különböző akciókat (actions) hajt végre. Ezek az akciók két dolgot eredményeznek:

- Jutalmakat (rewards).
- Állapotátmeneteket (state transitions - a környezet egy új állapotba kerül).

**A tanulás célja:** Egy irányelv (policy) megtanulása. Az irányelv egy függvény, amely az aktuális állapotvektort ráképezi (hozzárendeli) arra az optimális akcióra, amely a hosszú távon várható átlagos jutalmat maximalizálja.

# 1.3 Hogyan működik a felügyelt tanulás? (Gyakorlati áttekintés)

Ez a fejezet a gépi tanulás leggyakrabban használt típusának, a felügyelt tanulásnak a teljes folyamatát mutatja be egy konkrét algoritmuson, a Támogatóvektor-gépen (Support Vector Machine - SVM) keresztül.

> **Megjegyzés:** A bemásolt szövegedben az OCR olvasó néhány matematikai jelet hibásan konvertált – pl. a mínusz jelből ≠, a nagyobb-egyenlőből Ø lett. Ezeket a jegyzetben természetesen a helyes matematikai formában írtam le!

## 1. Adatgyűjtés és előkészítés

A folyamat mindig adatgyűjtéssel kezdődik, amely (Bemenet, Kimenet) párokból áll.

- **Bemenet (Input):** Bármi lehet (email, kép, szenzoradat).
- **Kimenet (Output / Címke):** Lehet szám, kategória (spam / nem spam), vektor (pl. koordináták) vagy szekvencia.

**Példa: Spam szűrés**

Összegyűjtünk 10 000 emailt. Ezeket emberi erővel felcímkézzük: `spam` vagy `not_spam`.

## 2. Jellemzőkinyerés (Feature Extraction) és Kódolás

A gép nem tud közvetlenül szöveget olvasni, ezért a nyers adatot jellemzővektorokká (feature vectors), a kategóriákat pedig számokká kell alakítani.

**A "Szózsák" (Bag of Words) módszer:**

Veszünk egy fix szótárat (pl. 20 000 angol szót ábécésorrendben). Minden egyes emailből egy 20 000 dimenziós vektort csinálunk:

- Az 1. pozíció értéke 1, ha az "a" szó szerepel a levélben, különben 0.
- A 2. pozíció értéke 1, ha az "aaron" szó szerepel, különben 0.
- ...és így tovább a 20 000. szóig (pl. "zulu").

Így az adathalmazunk 10 000 darab, egyenként 20 000 dimenziós vektorból áll.

**Címkék kódolása (az SVM algoritmushoz):**

Az SVM algoritmusa megköveteli, hogy a kimeneti kategóriákat speciális számértékekké alakítsuk:

- Pozitív osztály (Spam): $\large y = +1$
- Negatív osztály (Nem spam): $\large y = -1$

## 3. Az SVM Modell és a Döntési Határ

Az SVM algoritmus minden 20 000 dimenziós vektort egy pontként kezel egy 20 000 dimenziós térben. A feladata, hogy húzzon egy képzeletbeli "vonalat" (több dimenzióban ezt hipersíknak nevezzük), amely elválasztja a $\large +1$-es (spam) és a $\large -1$-es (nem spam) pontokat. Ezt az elválasztó felületet **döntési határnak** (decision boundary) hívjuk.

**A hipersík matematikai egyenlete**

A hipersíkot két paraméter határozza meg: egy $\large w$ vektor (súlyvektor, dimenziója megegyezik a bemenetével) és egy $\large b$ valós szám (eltolás vagy bias). A síkon fekvő pontokra igaz:

```math
\large
w \cdot x - b = 0
```

Ahol a $\large w \cdot x$ a két vektor skaláris szorzata:

```math
\large
w \cdot x = w^{(1)} \cdot x^{(1)} + w^{(2)} \cdot x^{(2)} + \dots + w^{(D)} \cdot x^{(D)}
```

(Ahol $\large D$ a dimenziók száma, a mi esetünkben 20 000.)

**Hogyan prediktál a modell?**

Ha bejön egy új $\large x$ email vektor, a betanított modell ($\large f(x)$) a következőképpen dönti el, hogy spam-e:

```math
\large
y = \mathrm{sign}(w^{*} \cdot x - b^{*})
```

**Magyarázat a sign operátorhoz:** A "sign" egy előjel-függvény. Kiszámoljuk a zárójeles kifejezést: ha az eredmény pozitív szám, a függvény kimenete $\large +1$ (Spam). Ha negatív szám, a kimenete $\large -1$ (Nem spam).

## 4. A Tanulás mint Optimalizációs Probléma

A gép feladata megtalálni az optimális $\large w^{*}$ és $\large b^{*}$ paramétereket. A gépek kiválóak függvények optimalizálásában, megadott feltételek (korlátok) mellett.

**Mik a feltételek?**

Azt akarjuk, hogy a meglévő 10 000 példánkat helyesen osztályozza a modell:

```math
\large
\begin{aligned}
w \cdot x_i - b &\ge 1 \quad \text{ha } y_i = +1 \\
w \cdot x_i - b &\le -1 \quad \text{ha } y_i = -1
\end{aligned}
```

Ezt a két feltételt egyetlen elegáns, kompakt egyenletté lehet összevonni:

```math
\large
y_i (w \cdot x_i - b) \ge 1 \qquad (\text{minden } i = 1, \dots, N \text{ példára})
```

**Mi a célfüggvény (Mit akarunk elérni)?**

Nem elég egy akármilyen elválasztó vonal. Azt a vonalat keressük, amely a legszélesebb "üres sávot" hagyja a két osztály között. Ezt a távolságot nevezzük **margónak** (margin). A széles margó biztosítja, hogy a modell a jövőbeli, új adatokon is jól teljesítsen (jó általánosítás).

Geometriailag a két osztályhoz legközelebbi pontok egy-egy párhuzamos hipersíkon fekszenek:

```math
\large
\begin{aligned}
w \cdot x - b &= 1 \\
w \cdot x - b &= -1
\end{aligned}
```

A két sík közötti távolság (a margó mérete) pontosan $\large \dfrac{2}{\|w\|}$.

Ahhoz, hogy ezt a távolságot maximalizáljuk, a tört nevezőjét, azaz a $\large w$ vektor Euklideszi normáját (hosszát) kell minimalizálnunk. A norma képlete:

```math
\large
\|w\| = \sqrt{\sum_{j=1}^{D} \left(w^{(j)}\right)^2}
```

A végső optimalizációs feladat tehát:

> Minimalizáld a $\large \|w\|$ értéket, azzal a feltétellel, hogy $\large y_i(w \cdot x_i - b) \ge 1$ minden példára.

<img width="872" height="717" alt="Képernyőkép 2026-10-12 214642" src="https://github.com/user-attachments/assets/37cb5d9e-0736-4259-8c38-f7225252f745" />


> Itt szerepeljen a könyvből a *Figure 1: An example of an SVM model for two-dimensional feature vectors.* című ábra, amely vizuálisan mutatja a két dimenziós teret, a kék/narancs pontokat, a `wx - b = 0` döntési határt, és a margót jelképező két párhuzamos síkot.

## 5. Finomítások és konklúzió

- **Lineáris vs. Nem-lineáris:** Az alap SVM egy egyenes/sík döntési határt hoz létre (lineáris modell). Azonban úgynevezett "kernelek" (kernels) segítségével a döntési határ tetszőlegesen görbült, nem-lineáris alakzat is lehet.
- **Tökéletlenség a valóságban:** Zajos adatok, címkézési hibák vagy extrém kiugró értékek (outliers) miatt a két osztály gyakran nem választható szét hibátlanul. Ilyenkor az SVM kap egy "büntető hiperparamétert", amely engedélyez némi tévesztést a tanítás során a robusztusabb modell érdekében.

**Az algoritmusok értékelési szempontjai a gyakorlatban:**

- **Pontosság:** Ezt a döntési határ alakja (és annak matematikai kiszámítása) határozza meg.
- **Tanítási sebesség:** Mennyi idő felépíteni a modellt.
- **Predikciós sebesség:** Mennyi idő alatt dönt egy új adatról.
- **Gyakori kompromisszum:** Sokszor egy picit pontatlanabb algoritmust választunk, ha az cserébe sokkal gyorsabban tanítható vagy gyorsabban hoz döntést éles környezetben.

  ## Példa levezetéssel:

  Képzeljük el, hogy összesen két darab emailünk van (`N=2`), és a szótárunk csak két szóból áll (`D=2`). Ez a végtelenül leegyszerűsített "játék" feladat tökéletesen megmutatja az SVM belső matematikáját.

## 1. Az adatok és a jelölések felfrissítése

Két adatrekordunk van, amelyeket kódoltunk:

- $\large N = 2$: A tanítópéldák száma.
- $\large D = 2$: A jellemzők (dimenziók) száma.
- $\large x_i$: A jellemzővektor (az email matematikailag leírva). Két számból áll: az első a "nyeremény" szó gyakorisága, a második az "ingyen" szó gyakorisága.
- $\large y_i \in \{+1, -1\}$: A címke ($\large +1$ a spam, $\large -1$ a normál levél).

A tanítóhalmazunk ($\large D_{\text{train}}$):

1. példa (Spam, mert mindkét gyanús szóból 2-2 db van benne):

```math
\large
x_1 = \begin{bmatrix} 2 \\ 2 \end{bmatrix}, \qquad y_1 = +1
```

2. példa (Nem spam, mert egyik szó sincs benne):

```math
\large
x_2 = \begin{bmatrix} 0 \\ 0 \end{bmatrix}, \qquad y_2 = -1
```

## 2. A betanított modell paraméterei

Az SVM tanulási algoritmusa már lefutott, és "megtalálta" nekünk az optimális döntési határt (a hipersíkot, ami $\large D=2$ esetén csak egy egyenes). Ennek az egyenesnek két paramétere van:

- $\large w$ (Súlyvektor / Weight vector): Ugyanannyi dimenziója van, mint az adatoknak ($\large D=2$). Ez határozza meg az egyenes dőlését.
- $\large b$ (Eltolás / Bias): Egyetlen valós szám. Ez határozza meg, hol metszi az egyenes a tengelyeket.

Tegyük fel, hogy az algoritmus a következő paramétereket dobta ki:

```math
\large
w = \begin{bmatrix} 0.5 \\ 0.5 \end{bmatrix}, \qquad b = 1
```

## 3. Az optimalizációs feltételek ellenőrzése

Az SVM legfőbb szabálya, hogy minden pontnak a megfelelő térfélen kell lennie, méghozzá elég messze a vonaltól. A kompakt képlet, aminek minden $\large i$ példára teljesülnie kell:

```math
\large
y_i (w \cdot x_i - b) \ge 1
```

Emlékeztető: a $\large w \cdot x$ a két vektor skaláris szorzata: $\large w^{(1)} \cdot x^{(1)} + w^{(2)} \cdot x^{(2)}$.

**Ellenőrizzük az 1. példát ($\large x_1$, spam):**

- Kiszámoljuk a belső részt: $\large w \cdot x_1 - b = (0.5 \cdot 2 + 0.5 \cdot 2) - 1 = (1 + 1) - 1 = 1$
- Megszorozzuk a címkével ($\large y_1 = +1$): $\large (+1) \cdot 1 = 1$
- A feltétel: $\large 1 \ge 1$. Teljesül!

**Ellenőrizzük a 2. példát ($\large x_2$, nem spam):**

- Kiszámoljuk a belső részt: $\large w \cdot x_2 - b = (0.5 \cdot 0 + 0.5 \cdot 0) - 1 = 0 - 1 = -1$
- Megszorozzuk a címkével ($\large y_2 = -1$): $\large (-1) \cdot (-1) = 1$
- A feltétel: $\large 1 \ge 1$. Ez is teljesül!

## 4. Predikció egy teljesen új adaton (Inference)

Bejön a rendszerbe egy 3. levél, amit még sosem látott a modell. A jellemzőkinyerés után ez a vektora:

```math
\large
x_{\text{új}} = \begin{bmatrix} 3 \\ 1 \end{bmatrix}
```

(Tehát a "nyeremény" 3-szor, az "ingyen" 1-szer szerepel benne).

A predikció képlete:

```math
\large
y = \mathrm{sign}(w \cdot x_{\text{új}} - b)
```

$\large \mathrm{sign}()$: Előjel operátor. Ha a zárójelben pozitív szám van, a kimenet $\large +1$, ha negatív, akkor $\large -1$.

A számítás lépései:

1. $\large w \cdot x_{\text{új}} = (0.5 \cdot 3) + (0.5 \cdot 1) = 1.5 + 0.5 = 2$
2. Kivonjuk a $\large b$-t: $\large 2 - 1 = 1$
3. Alkalmazzuk a sign függvényt: $\large \mathrm{sign}(1) = +1$

A gép döntése: $\large +1$, azaz ez a levél **Spam**.

## 5. A margó (Margin) kiszámítása

A gépi tanulás célja az volt, hogy a két osztály (spam és nem spam) közötti "üres sáv" a lehető legszélesebb legyen. A margó szélességének képlete:

```math
\large
\text{Távolság} = \frac{2}{\|w\|}
```

$\large \|w\|$ (Euklideszi norma): A $\large w$ vektor hossza a térben. Képlete:

```math
\large
\|w\| = \sqrt{\left(w^{(1)}\right)^2 + \left(w^{(2)}\right)^2}
```

Számoljuk ki a mi súlyvektorunk hosszát:

```math
\large
\|w\| = \sqrt{0.5^2 + 0.5^2} = \sqrt{0.25 + 0.25} = \sqrt{0.5} \approx 0.707
```

A két elválasztó sík közötti teljes távolság:

```math
\large
\text{Távolság} = \frac{2}{0.707} \approx 2.82
```

Az SVM algoritmus azért választotta ezt a $\large w$ vektort, mert matematikailag be tudta bizonyítani, hogy ennél szélesebb margót ($\large 2.82$-nél nagyobbat) nem lehet húzni a mi $\large x_1$ és $\large x_2$ pontunk közé úgy, hogy a feltételek érvényesek maradjanak.

# 1.4 Miért működik a modell az új (korábban nem látott) adatokon?

A gépi tanulás igazi értéke az **általánosítási képesség** – vagyis az, hogy a modell nem magolja be a válaszokat, hanem egy új, eddig sosem látott adatra is helyes predikciót ad. A szöveg ezt geometriai és statisztikai okokkal magyarázza.

## A geometriai és statisztikai okok

**A tér kettéosztása:** Ahogy a korábbi (1. ábrán látható) példában is szerepelt, a döntési határ két külön altérre (térfélre) vágja a teret. Az egyik oldalon a pozitív, a másikon a negatív példák találhatók.

**Hasonló a hasonlóhoz húz:** Ha a tanítóadatokat véletlenszerűen, egymástól függetlenül és azonos módszerrel gyűjtöttük, akkor feltételezhetjük, hogy a jövőbeli adatok is ugyanezt a mintát követik. Emiatt egy újonnan érkező negatív példa a térben nagy valószínűséggel a korábbi negatív példák közelében (azok "szomszédságában") fog landolni, egy új pozitív példa pedig a többi pozitív mellett.

**A hibák esélye:** Mivel az új adatok a saját fajtájukhoz közel bukkannak fel, a korábban meghúzott döntési határ ezeket is jó eséllyel helyesen választja szét. Természetesen előfordulnak majd szokatlan helyen lévő pontok, ahol a modell téved, de mivel ezek ritkább, kisebb valószínűségű esetek, a helyes döntések száma jóval magasabb lesz a hibákénál.

## A tanítóhalmaz mérete és az SVM stratégiája

**Több adat = Kevesebb meglepetés:** Intuitívan belátható, hogy minél nagyobb a tanító adathalmazunk, annál sűrűbben lefedjük a teret. Minél több példát láttunk, annál valószínűtlenebb, hogy egy teljesen új, a korábbiaktól extrém módon eltérő (térben távol lévő) adattal találkozunk.

**A legnagyobb margó jelentősége:** Itt kap igazi értelmet az SVM optimalizációs célja. Az algoritmus direkt úgy húzza meg a döntési határt, hogy a lehető legnagyobb legyen a margó (az üres sáv a vonal körül). Célzottan a lehető legmesszebbre teszi a vonalat mindkét osztály pontjaitól. Ez az extra "biztonsági távolság" biztosítja, hogy ha egy új pont kicsit kijjebb is esik a megszokott halmazból, még mindig bőven a saját térfelén maradjon, így minimalizálva a tévedés valószínűségét.
