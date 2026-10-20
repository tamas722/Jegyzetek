# 3.1 Lineáris Regresszió (Linear Regression)

A lineáris regresszió célja, hogy a bemeneti változók (jellemzők) súlyozott összeadásával – azaz egy lineáris kombinációval – megjósoljon egy folytonos célértéket (például egy lakás árát, egy hőmérsékletet, vagy egy részvény árfolyamát).

## 3.1.1 A probléma meghatározása és a Modell

Mint minden felügyelt tanulásnál, itt is egy felcímkézett adathalmazból $\{(x_i, y_i)\}_{i=1}^{N}$ indulunk ki.

A gép feladata egy $f_{w,b}(x)$ függvény (modell) felépítése, amely a következőképpen néz ki:

```math
\large
f_{w,b}(x) = w \cdot x + b
```

**A paraméterek jelentése:**

- $x$ (Bemenet): A jellemzővektor (pl. a lakás adatai).
- $w$ (Súlyvektor): A gép által megtanult paraméterek, amik megmondják, melyik jellemző mennyire "húzza" lefelé vagy felfelé az eredményt.
- $b$ (Bias/Eltolás): Egy alapérték. Ha minden bemenet 0 lenne, ennyi lenne a becslés. (Gondolj rá úgy, mint egy lakás alapárára).

### 🔢 1. Számpélda: Hogyan számol a lineáris modell?

Képzeljük el, hogy lakásárakat ($y$, millió Forintban) akarunk megjósolni. Két jellemzőnk van ($D=2$):

- $x^{(1)}$: Alapterület (m²)
- $x^{(2)}$: Távolság a belvárostól (km)

Tegyük fel, a gép már "megtanulta" (mindjárt megnézzük, hogyan) az optimális paramétereket:

- Súlyok: $w = [0.6,\ -2.0]$ (Minden extra m² növeli az árat 0.6 millióval, de minden km a központtól csökkenti 2 millióval).
- Eltolás: $b = 10$ (Alapár).

Érkezik egy új lakás: 60 m²-es, és 5 km-re van a központtól. A bemeneti vektor: $x = [60, 5]$.

Számoljuk ki a becslést (a vektorok skaláris szorzatával):

```math
\large
\begin{aligned}
f(x) &= (0.6 \cdot 60) + (-2.0 \cdot 5) + 10 \\
f(x) &= 36 - 10 + 10 = 36
\end{aligned}
```

A modell predikciója tehát **36 millió Ft**.

<img width="506" height="392" alt="1" src="https://github.com/user-attachments/assets/12fbc738-cc23-45f6-acc4-182946bfbf91" />

 Ez az ábra azt vizualizálja, ahogy egydimenziós bemenet esetén (pl. csak az alapterület) a modell egy szimpla egyenest húz az adatpontok közé.

**Különbség az SVM-hez képest:**

Míg az SVM-nél (osztályozás) egy olyan egyenest kerestünk, ami elválasztja a pontokat és a lehető legmesszebb van tőlük, a lineáris regressziónál egy olyan egyenest keresünk, ami a lehető legközelebb simul az összes pontunkhoz.

## 3.1.2 A megoldás: A Költség és a Veszteség minimalizálása

Ahhoz, hogy a gép megtalálja a legjobb $w$ és $b$ értékeket, definiálnunk kell, mit jelent az, hogy "rossz becslés". Erre szolgál a **Veszteségfüggvény** (Loss function).

A lineáris regresszió a **Négyzetes hibát** (Squared error loss) használja:

```math
\large
\text{Veszteség} = \left(f(x_i) - y_i\right)^2
```

Ezt az összes példánkra kiszámoljuk, majd átlagoljuk. Ez adja a teljes modell **Költségfüggvényét** (Cost function), amit **Tapasztalati kockázatnak** (Empirical risk) is hívnak:

```math
\large
J(w, b) = \frac{1}{N} \sum_{i=1}^{N} \left(f_{w,b}(x_i) - y_i\right)^2
```

Az algoritmus célja ezt a $J$ értéket a minimálisra csökkenteni.

### 🔢 2. Számpélda: A Veszteség és a Költség kiszámítása

Maradjunk a lakásos példánál, és nézzük meg, hogyan bünteti a gép a saját hibáit! Legyen egy apró adathalmazunk 2 lakással ($N=2$):

**1. lakás** (Valódi ár: $y_1 = 40$): A gépünk az előbb látott módon becsült rá egyet, mondjuk $f(x_1) = 36$.

- Hiba: $36 - 40 = -4$.
- Négyzetes Veszteség: $(-4)^2 = 16$.

**2. lakás** (Valódi ár: $y_2 = 25$): A gép becslése $f(x_2) = 28$.

- Hiba: $28 - 25 = 3$.
- Négyzetes Veszteség: $3^2 = 9$.

A Költségfüggvény értéke (az átlagos hiba):

```math
\large
J = \frac{16 + 9}{2} = 12.5
```

A gép most elkezdi módosítgatni a $w$ és $b$ paramétereit (kicsit növeli, kicsit csökkenti őket), hogy ez a $12.5$-ös szám a lehető legközelebb menjen a nullához.

**Miért pont négyzetre emelünk (és miért nem pl. abszolút értéket veszünk)?**

A szerző megjegyzi, hogy használhatnánk szimpla abszolút különbséget is ($|f(x_i) - y_i|$). Azonban a négyzetre emelésnek ($x^2$) két hatalmas előnye van:

1. **Felnagyítja a nagy hibákat:** Ha a gép 1 milliót téved, a büntetés $1^2 = 1$. Ha 5 milliót téved, a büntetés $5^2 = 25$. A négyzetes hiba szigorúan rákényszeríti az egyenest, hogy egyetlen ponttól se távolodjon el extrém módon, különben a büntetés az egekbe szökik.
2. **Matematikailag könnyen deriválható (Smoothness):** Adrien-Marie Legendre (1705) rájött, hogy a négyzetes függvény folytonos és "sima" U-alakú görbe. Az abszolút érték grafikája egy V-alak, aminek a csúcsánál "törés" van (ott nem deriválható).

### 🔢 3. Számpélda: Optimalizáció deriválással (A zárt formula)

A könyv utolsó bekezdése említi, hogy a négyzetes forma miatt a gradiens kiszámolható és egyenlővé tehető nullával.

A 2. fejezet Láncszabályát alkalmazva a veszteségfüggvényünk: $(f(x) - y)^2$.

A külső függvény deriváltja: $2 \cdot (f(x) - y)$.

Ez egy egyszerű szorzás, nincsenek benne törések.

Ha a teljes költségfüggvény gradiensét (deriváltjait) kiszámoljuk, és az egyenletet egyenlővé tesszük $0$-val (hiszen a minimumot keressük, ahol a függvény meredeksége vízszintes), egy sima, lineáris egyenletrendszert kapunk.

A gépnek nem is kell próbálkoznia! Egy pontos algebrai egyenlettel (úgynevezett **zárt formulával** / closed form solution) egyetlen mátrixszorzásos lépésben megkapja a tökéletes $w^{*}$ és $b^{*}$ paramétereket.

## Miért Lineáris? (A Túltanulás / Overfitting veszélye)

Érthető a kérdés: miért ragaszkodunk az egyenes vonalhoz? Miért nem húzunk egy kacskaringós, 10-edfokú polinomot a pontjainkra, ami minden egyes pontot tökéletesen érint?

A válasz az **Overfitting (Túltanulás)**.

Egy komplex modell (pl. ami $x^{10}$-nel dolgozik) tökéletesen, 0 hibával (Költség = 0) bemagolja a tanítóadatokat. Viszont olyan vadul fog ugrálni a pontok között, hogy ha bejön egy új adat, hatalmasat fog tévedni rajta. A lineáris regresszió "merevsége" egy beépített fék, ami megakadályozza, hogy a modell túltanulja a zajt.

<img width="453" height="376" alt="2" src="https://github.com/user-attachments/assets/d56e25c8-b365-401a-9508-7f0d8ef849a7" />

 Ez az ábra tökéletesen bemutatja, ahogy a 10-edfokú (piros) polinom kanyarogva érint minden kék pontot, de a legelső új $x_{\text{new}}$ pontnál a becslés teljesen elszáll az értelmezhető tartományból.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez](https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/03%20Fejezet/polinomoverfitting.html)

👉 [Kattints ide az interaktív HTML widget megtekintéséhez](https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/03%20Fejezet/PolynomialCurveFitting.html)

 # 3.2 Logisztikus Regresszió (Logistic Regression)

A legfontosabb dolog, amit rögtön az elején tisztázni kell: a nevével ellentétben a logisztikus regresszió **nem regressziós, hanem egy osztályozó (classification) algoritmus**. A nevét a statisztikából hozta, mivel a matematikai formulája nagyon hasonlít a lineáris regresszióhoz. Bár a könyv a bináris (kétosztályos) problémán keresztül mutatja be, természetesen kiterjeszthető többosztályos feladatokra is.

## 3.2.1 A probléma meghatározása

A lineáris regressziónál láttuk, hogy a kimenetet egy lineáris kombinációval számoljuk: $\mathbf{w} \cdot \mathbf{x} + b$.

Ezzel a bináris osztályozásnál (ahol a címke csak $0$ vagy $1$ lehet) egy óriási probléma van: a $\mathbf{w} \cdot \mathbf{x} + b$ képlet eredménye mínusz végtelentől plusz végtelenig bármi lehet. Ezt nem tudjuk egyértelműen $0$-nak vagy $1$-nek megfeleltetni.

Amikor a tudósoknak még számítógépek nélkül kellett számolniuk, egy olyan folytonos matematikai függvényt kerestek, amely a végtelen tartományt összenyomja a $(0, 1)$ nyílt intervallumba. Ha a kimenet közelebb van a $0$-hoz, a gép a negatív osztályba sorolja az adatot; ha az $1$-hez, akkor a pozitívba.

Erre a tökéletes megoldás a **Standard logisztikus függvény** (vagy **Szigmoid függvény**):

```math
\large
f(x) = \frac{1}{1 + e^{-x}}
```

(Ahol $e$ az Euler-féle szám, a természetes logaritmus alapja, értéke kb. $2.718$. Programozásban gyakran `exp(x)`-ként hivatkoznak rá.)

Ha ebbe a függvénybe beágyazzuk a lineáris modellünket (az $x$ helyére beírjuk a $\mathbf{w} \cdot \mathbf{x} + b$-t), megkapjuk a logisztikus regresszió modelljét:

```math
\large
f_{\mathbf{w},b}(\mathbf{x}) = \frac{1}{1 + e^{-(\mathbf{w} \cdot \mathbf{x} + b)}}
```

A kimenetet úgy értelmezzük, mint annak a valószínűségét, hogy a vizsgált példa a pozitív ($1$-es) osztályba tartozik. Ha az eredmény $\ge 0.5$ (ez a küszöbérték, vagy threshold), akkor a gép döntése $1$. Ha $< 0.5$, akkor $0$. (Ez a küszöbérték a gyakorlatban a feladattól függően eltolható.)

### 🔢 1. Számpélda: Hogyan "nyomja össze" a Szigmoid a számokat?

Tegyük fel, a lineáris rész ($\mathbf{w} \cdot \mathbf{x} + b$) eredménye egy nagyon magas szám: $5.0$.

Számoljuk ki a valószínűséget:

```math
\large
f(5.0) = \frac{1}{1 + e^{-5.0}} \approx \frac{1}{1 + 0.0067} \approx \frac{1}{1.0067} \approx \mathbf{0.993}
```

A modell $99.3\%$-ig biztos benne, hogy ez a pozitív osztály.

Mi van, ha a lineáris rész egy nagy negatív szám: $-3.0$?

```math
\large
f(-3.0) = \frac{1}{1 + e^{-(-3.0)}} = \frac{1}{1 + e^{3.0}} \approx \frac{1}{1 + 20.08} = \frac{1}{21.08} \approx \mathbf{0.047}
```

A modell szerint csak $4.7\%$ az esély a pozitív osztályra, tehát egyértelműen a negatív ($0$) osztályba sorolja.

<img width="512" height="407" alt="3" src="https://github.com/user-attachments/assets/e3d353b0-23de-4a7b-a8b8-690eabc4aba5" />

## 3.2.2 A megoldás: Maximum Likelihood (Maximális Valószerűség)

A lineáris regressziónál a hiba négyzetét (MSE) minimalizáltuk. A logisztikus regressziónál ez a módszer nem működik jól (a Szigmoid függvény miatt a költségfüggvény "dimbes-dombos" lenne, tele lokális minimumokkal).

Ehelyett a **Maximális Valószerűség** (Maximum Likelihood, MLE) elvét használjuk: nem a hibát akarjuk minimalizálni, hanem a tanító adathalmazunk megfigyelésének valószínűségét akarjuk maximalizálni a modellünk szerint.

### A Likelihood (Valószerűség) függvény felírása

Tegyük fel, van egy adatunk $(\mathbf{x}_i, y_i)$. A modell ad egy $p$ becslést (ami a $0$ és $1$ közötti valószínűség, $p = f_{\mathbf{w},b}(\mathbf{x}_i)$).

- Ha a valódi osztály pozitív ($y_i = 1$), akkor a modell helyességének valószínűsége: $p$.
- Ha a valódi osztály negatív ($y_i = 0$), akkor a modell helyességének valószínűsége: $1 - p$.

Hogy ezt az algoritmusok számára egyetlen matematikai képletbe sűrítsük, használjuk azt a formulát, amit a könyv "ijesztőnek" (scary) nevez, pedig valójában csak egy elegáns matematikai if-else (ha-akkor) ág:

```math
\large
L_{\mathbf{w},b} = \prod_{i=1}^{N} \left( f_{\mathbf{w},b}(\mathbf{x}_i) \right)^{y_i} \cdot \left( 1 - f_{\mathbf{w},b}(\mathbf{x}_i) \right)^{(1 - y_i)}
```

### 🔢 2. Számpélda: Miért zseniális az "ijesztő" képlet?

Tudjuk, hogy bármely szám nulladik hatványon $1$-et ad ($x^0 = 1$). Nézzük meg, hogyan omlik össze a képlet a megfelelő részre a valódi címke ($y_i$) függvényében!

Tegyük fel, a modellünk predikciója $p = 0.8$.

**1. Eset: A valódi címke pozitív ($y_i = 1$).**

Helyettesítsünk be a képletbe:

```math
\large
0.8^{1} \cdot (1 - 0.8)^{(1 - 1)} = 0.8^1 \cdot 0.2^0 = 0.8 \cdot 1 = \mathbf{0.8}
```

(A negatív ág teljesen eltűnt, maradt a $p$.)

**2. Eset: A valódi címke negatív ($y_i = 0$).**

Helyettesítsünk be:

```math
\large
0.8^{0} \cdot (1 - 0.8)^{(1 - 0)} = 1 \cdot 0.2^1 = \mathbf{0.2}
```

(A pozitív ág eltűnt, maradt az $1-p$.)

### Log-Likelihood (A logaritmus trükk)

A fenti képletben egy óriási szorzás ($\prod$) van. Mivel $0$ és $1$ közötti számokat (valószínűségeket) szorzunk össze rengetegszer, az eredmény pillanatok alatt olyan közel kerülne a nullához, hogy a számítógép nem tudná kezelni (alulcsordulás).

Ezért vesszük az egész képlet természetes logaritmusát ($\ln$). A logaritmus varázslata, hogy a szorzásokat összeadássá alakítja, a kitevőket pedig szorzókká hozza előre:

```math
\large
\text{LogL}_{\mathbf{w},b} = \sum_{i=1}^{N} \left[ y_i \ln\left(f_{\mathbf{w},b}(\mathbf{x}_i)\right) + (1 - y_i) \ln\left(1 - f_{\mathbf{w},b}(\mathbf{x}_i)\right) \right]
```

Mivel a logaritmus szigorúan monoton nő, ha megtaláljuk azt a $\mathbf{w}$ és $b$ értéket, ami ezt az összeget maximalizálja, az egyben az eredeti (szorzatos) valószínűséget is maximalizálja.

**Fontos különbség:** A lineáris regresszióval ellentétben ennek a Log-Likelihood függvénynek a maximalizálására **nincs zárt matematikai formula** (nem lehet egyetlen lépésben kiszámolni). Ehelyett numerikus optimalizációra, például **Gradiens ereszkedésre** (Gradient Descent) van szükség, amit a következő fejezet tárgyal.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/03%20Fejezet/LogisticSigmoidFunction.html)

# 3.3 Döntési fák (Decision Tree Learning)

A döntési fa egy olyan irányított, körmentes gráf (acyclic graph), amelyben logikai döntések sorozatával jutunk el a végeredményhez.

- **Elágazások (Branching nodes):** A fa belső csomópontjaiban kiválasztunk egy $j$-edik jellemzőt, és megnézzük, hogy az értéke egy bizonyos $t$ küszöbérték (threshold) alatt van-e. Ha igen, balra megyünk, ha nem, jobbra.
- **Levelek (Leaf nodes):** Amikor eljutunk a fa végére egy levélbe, megszületik a döntés (a predikció), hogy az adott példa melyik osztályba tartozik.

A gépi tanulás célja itt az, hogy magát ezt a fát – a feltételrendszert és a küszöbértékeket – az adatokból tanulja meg.

## 3.3.1 A probléma meghatározása

Ugyanaz az alapfelállás, mint korábban: van egy $\{(\mathbf{x}_i, y_i)\}_{i=1}^{N}$ adathalmazunk, ahol a címkék binárisak ($y_i \in \{0, 1\}$). A feladat egy olyan döntési fa felépítése, ami egy bemeneti $\mathbf{x}$ vektor alapján megjósolja az osztályt.

## 3.3.2 A megoldás: Az ID3 algoritmus (és társai)

A döntési fák építésére sok algoritmus létezik, a könyv az ID3 egy változatát mutatja be (amely folytonos küszöbértékeket használ).

A logisztikus regressziónál egy matematikai egyenlet paramétereit ($\mathbf{w}, b$) finomítottuk. A döntési fa viszont egy **nem-parametrikus modell**: nem egy egyenletet old meg, hanem az adathalmazt darabolja egyre kisebb, "tisztább" részekre.

A modellünk ($f_{\text{ID3}}$) pusztán azt számolja ki, hogy egy adott ág végén lévő adatoknak hány százaléka pozitív:

```math
\large
f_{\text{ID3}}(\mathbf{x}) \overset{\text{def}}{=} \Pr(y=1 \mid \mathbf{x})
```

### Hogyan épül fel a fa lépésről lépésre?

#### 1. Kezdő lépés (A Gyökér)

A fa egyetlen "gyökér" csomóponttal indul, amely tartalmazza az összes ($N$ darab) tanító példát. Ezt a halmazt $S$-nek hívjuk.

Ezen a ponton a modell becslése egy konstans érték: egyszerűen a halmazban lévő pozitív ($y=1$) példák aránya (átlaga).

```math
\large
f_{\text{ID3}}^{S} = \frac{1}{|S|} \sum_{(\mathbf{x},y) \in S} y
```

### 🔢 1. Számpélda: A konstans modell (Gyökér)

Képzeljük el, hogy 10 páciensünk van ($|S| = 10$). Ebből 6 beteg ($y=1$) és 4 egészséges ($y=0$).

Ha a fa ezen a ponton megállna, és jönne egy új páciens, a modell predikciója egyszerűen:

```math
\large
f_{\text{ID3}}^{S} = \frac{6}{10} = \mathbf{0.6}
```

Azaz $60\%$ eséllyel beteg. Ez még nagyon pontatlan. Ezt a kiindulási állapotot mutatja be az alábbi ábra (a) része.

<img width="736" height="438" alt="4" src="https://github.com/user-attachments/assets/fe90761d-a3d0-4b7c-b4ab-ac65d1eb9bdb" />


#### 2. A Vágás (Splitting)

A gép végignézi az összes jellemzőt ($j$) és az összes lehetséges küszöbértéket ($t$), és kipróbálja, mi történne, ha ott kettévágná a halmazt:

- **Bal ág ($S^-$):** Ide kerülnek azok a példák, ahol $x^{(j)} < t$.
- **Jobb ág ($S^+$):** Ide kerülnek azok a példák, ahol $x^{(j)} \ge t$.

De hogyan dönti el az algoritmus, hogy melyik volt a "legjobb" vágás? Erre való az **Entrópia**!

### Az Entrópia ($H$) mint a bizonytalanság mértéke

Az entrópia egy halmaz "kevertségét" vagy bizonytalanságát méri.

- **Maximális** az entrópia, ha a halmaz teljesen kevert (pl. 50% pozitív, 50% negatív). Ilyenkor a legnehezebb döntést hozni.
- **Minimális** az entrópia, ha a halmaz teljesen "tiszta" (pl. 100% pozitív). Ilyenkor nincs bizonytalanság.

Egy $S$ halmaz entrópiájának képlete (ahol $p = f_{\text{ID3}}^{S}$, azaz a pozitívok aránya):

```math
\large
H(S) = -p \ln(p) - (1 - p) \ln(1 - p)
```

### 🔢 2. Számpélda: Entrópia számítása

Nézzük meg a korábbi 10 páciensünk gyökércsomópontját ($p = 0.6$).

```math
\large
H(S) = -0.6 \ln(0.6) - 0.4 \ln(0.4)
```

Kiszámolva a természetes logaritmusokat:

```math
\large
H(S) \approx -0.6 \cdot (-0.51) - 0.4 \cdot (-0.916) = 0.306 + 0.366 = \mathbf{0.672}
```

Ez egy elég magas entrópia, tehát a halmaz bizonytalan.

### Hogyan értékeljük a vágást?

Amikor kettévágjuk a halmazt $S^-$-ra és $S^+$-ra, kiszámoljuk az új csomópontok entrópiáinak súlyozott átlagát:

```math
\large
H(S^-, S^+) = \frac{|S^-|}{|S|} H(S^-) + \frac{|S^+|}{|S|} H(S^+)
```

A gép azt a $(j, t)$ vágást fogja véglegesíteni, amelyik ezt a súlyozott entrópiát a lehető legkisebbre csökkenti (ezt a csökkenést **Információnyereségnek**, vagy Information Gain-nek is hívják).

### 🔢 3. Számpélda: A tökéletes vágás

Tegyük fel, hogy a gépezet kipróbál egy vágást: a páciens láza > 38.0°C.

Ez 4 embert küld a bal ágra ($S^-$), és 6 embert a jobb ágra ($S^+$).

- **Bal ág ($S^-$):** 4 emberből 0 beteg, 4 egészséges. $p = 0/4 = 0$. Itt az entrópia $H(S^-) = 0$, mert a halmaz teljesen tiszta!
- **Jobb ág ($S^+$):** 6 emberből 6 beteg, 0 egészséges. $p = 6/6 = 1$. Itt is $H(S^+) = 0$, mert ez is teljesen tiszta!

A vágás utáni súlyozott entrópia:

```math
\large
H(S^-, S^+) = \frac{4}{10} \cdot 0 + \frac{6}{10} \cdot 0 = \mathbf{0}
```

Mivel az entrópia $0$-ra csökkent, a gép tökéletes vágást talált: megtanulta, hogy aki lázas, az beteg, aki nem, az egészséges.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/03%20Fejezet/BinaryEntropyViz.html)

---

## Megállási feltételek (Stopping criteria)

A fa építése (a halmazok rekurzív darabolása) egy adott ágon véget ér (levéllé válik), ha az alábbiak valamelyike teljesül:

1. A csomópontban lévő összes adat ugyanahhoz az osztályhoz tartozik (tiszta csomópont, $H=0$).
2. Nincs már olyan jellemző, ami alapján tovább lehetne vágni.
3. A legjobb vágás is csak nagyon kicsivel (egy $\epsilon$ értéknél kisebb mértékben) csökkentené az entrópiát.
4. A fa elért egy előre meghatározott maximális mélységet ($d$) – ez segít a túltanulás megakadályozásában.

**Egy fontos tulajdonság:** Az ID3 egy "mohó" (greedy) algoritmus. Minden lépésben az adott pillanatban legjobbnak tűnő vágást választja, és sosem néz előre, hogy egy most gyengébbnek tűnő vágás esetleg később zseniális elágazásokhoz vezetne-e. Emiatt nem garantált, hogy a globálisan legoptimálisabb fát találja meg (bár visszalépéses kereséssel – backtracking – javítható, az nagyon lassítaná a folyamatot).


# 3.4 Támogatóvektor-gép (Support Vector Machine – SVM) kiegészítés

Az első fejezetben már láttuk az SVM alapjait: egy olyan döntési határt (egyenest/hipersíkot) keresünk, ami tökéletesen elválasztja a két osztályt, méghozzá úgy, hogy a margó (az üres sáv) a lehető legszélesebb legyen.

Azonban a való világban az adatok sosem ilyen tökéletesek. Két kritikus probléma merül fel, amit ez a fejezet old meg:

1. **Zaj (Noise):** Mi van, ha egy-két hibás adatpont átlóg a rossz térfélre, és emiatt lehetetlen egy tökéletes egyenest húzni közéjük?
2. **Belső nem-linearitás (Inherent Non-Linearity):** Mi van, ha az adatok alakzata mondjuk egy kör, amit matematikailag képtelenség egyetlen egyenessel kettévágni?

<img width="737" height="322" alt="5" src="https://github.com/user-attachments/assets/e03af1cc-bdfc-4b64-a3c8-3128631a0dff" />

>  A bal oldali kép a zajt mutatja (ahol egy egyenes működne, de egy-egy pont bezavar), a jobb oldali kép pedig a nem-lineáris köralakzatot, ahol az egyenes teljesen csődöt mond.

## 3.4.1 Zajos adatok kezelése (Soft-Margin SVM)

Ha ragaszkodunk a tökéletes elválasztáshoz (ezt hívják Hard-margin SVM-nek), de van egy zajos (rossz helyen lévő) pontunk, a gép egyszerűen nem talál megoldást, és az algoritmus összeomlik.

Ezt úgy orvosoljuk, hogy megengedjük a gépnek a hibázást, de minden hibát büntetünk. Ezt hívják **Soft-margin SVM**-nek (Lágy margójú SVM).

Ehhez bevezetjük a **Csuklópánt veszteségfüggvényt** (Hinge Loss):

```math
\large
\max\left(0,\ 1 - y_i(\mathbf{w} \cdot \mathbf{x}_i - b)\right)
```

### 🔢 1. Számpélda: Hogyan működik a Hinge Loss?

Tegyük fel, a valódi címke pozitív ($y_i = 1$). A gépnek a $(\mathbf{w} \cdot \mathbf{x}_i - b)$ értékre legalább $1$-et kell adnia, hogy a pont a margón kívül, biztonságban legyen.

**Eset 1: Jó predikció (a pont jó helyen van).** A gép eredménye $2$.

Kiszámoljuk a büntetést:

```math
\large
\max(0,\ 1 - 1 \cdot 2) = \max(0,\ -1) = \mathbf{0}
```

A gép nem kap büntetést, mert helyesen döntött.

**Eset 2: Rossz predikció (a pont átlógott a határon).** A gép eredménye $-0.5$.

Kiszámoljuk a büntetést:

```math
\large
\max(0,\ 1 - 1 \cdot (-0.5)) = \max(0,\ 1.5) = \mathbf{1.5}
```

A gép $1.5$-ös büntetést kap. Minél mélyebbre lóg át a pont a rossz térfélre, a büntetés annál nagyobb.

### Az új költségfüggvény (Két erő harca)

Az SVM-nek most két dolgot kell egyszerre minimalizálnia:

```math
\large
C\, \|\mathbf{w}\|^2 + \frac{1}{N} \sum_{i=1}^{N} \max\left(0,\ 1 - y_i(\mathbf{w} \cdot \mathbf{x}_i - b)\right)
```

- **Bal oldal ($\|\mathbf{w}\|^2$):** A széles margóért felel (általánosítás).
- **Jobb oldal (A Szumma):** A hibák (a Hinge loss) összessége.

**A $C$ hiperparaméter:** Ez egy "potméter", amit te állítasz be! Ez szabályozza a csereszabályozást (trade-off).

> *(Fontos szakmai megjegyzés: Burkov képletében a $C$ a margó részhez van kötve. A legtöbb programkönyvtárban, pl. a Python Scikit-Learn-ben a $C$ a hiba részhez van kötve. A lényeg az elv: ha a $C$ beállításával azt mondod a gépnek, hogy "Ignoráld a hibákat!", akkor egy nagyon széles, általánosító margót fog húzni, amibe sok pont belóg. Ha azt mondod: "Egyetlen hibát sem tűrök!", akkor egy nagyon szűk margót fog húzni, és esélyes a túltanulásra (overfitting).)*

## 3.4.2 Belső nem-linearitás (A Kernel Trükk / Kernel Trick)

Mi van a jobb oldali ábrával, ahol a kék pontok egy kört formáznak, amit körülvesznek a narancs pontok? Ezt egy 2D síkon soha nem vágod ketté egy egyenessel.

**Az ötlet:** Emeljük fel az adatokat 3 Dimenzióba! Ha a pontokat egy megfelelő matematikai függvénnyel ($\phi$) átsugározzuk a 3D térbe, ott a köralakzat egy tálca formáját fogja felvenni, amit már simán el lehet vágni egy sík lappal (hipersíkkal).

<img width="787" height="430" alt="6" src="https://github.com/user-attachments/assets/91b67e98-c6b1-4b5b-a7ec-1caf1cf29118" />

> Megmutatja, ahogy a 2 dimenziós körből egy 3 dimenziós "kehely" lesz, amit már gyönyörűen el tud vágni egy sík.

### A probléma a dimenzióváltással (És a zseniális megoldás)

Tegyük fel, van egy sima, kétváltozós pontunk: $\mathbf{x} = [q, p]$.

Hogy 3D-be tegyük (amit a 6. ábra mutat), egy ilyen torzító függvényen kéne átküldeni:

```math
\large
\phi(\mathbf{x}) = \left[ q^2,\ \sqrt{2}\,qp,\ p^2 \right]
```

Ha ezt több tízezer dimenziónál kéne megcsinálni, a számítógép processzora azonnal leolvadna az irdatlan számítási igénytől.

### A Kernel Trükk (A varázslat)

Amikor a matematikusok megoldották az SVM egyenletrendszerét (a Lagrange-multiplikátorok segítségével), észrevettek valami furcsát. Az SVM matematikai motorjának soha nem kell ismernie a pontok pontos 3D-s koordinátáit! Az egyetlen dolog, amit az algoritmus használ, az a pontok közötti skaláris szorzat ($\mathbf{x}_i \cdot \mathbf{x}_k$).

### 🔢 2. Számpélda: Miért "trükk" a Kernel trükk?

Vegyünk két adatpontot: $\mathbf{x}_1 = [1, 2]$ és $\mathbf{x}_2 = [3, 4]$.

**A nehéz, lassú út (Számoljunk mindent 3D-ben):**

Felemeljük $\mathbf{x}_1$-et a fenti $\phi$ függvénnyel:

```math
\large
\left[1^2,\ \sqrt{2} \cdot 1 \cdot 2,\ 2^2\right] = \left[1,\ 2\sqrt{2},\ 4\right]
```

Felemeljük $\mathbf{x}_2$-t:

```math
\large
\left[3^2,\ \sqrt{2} \cdot 3 \cdot 4,\ 4^2\right] = \left[9,\ 12\sqrt{2},\ 16\right]
```

Összeszorozzuk a két 3D vektort (skaláris szorzat):

```math
\large
(1 \cdot 9) + (2\sqrt{2} \cdot 12\sqrt{2}) + (4 \cdot 16) = 9 + 48 + 64 = \mathbf{121}
```

**A Kernel Trükk (Az okos, gyors út):**

A matematikusok észrevették, hogy erre a konkrét 3D transzformációra létezik egy Kernel függvény ($K$):

```math
\large
K(\mathbf{x}_1, \mathbf{x}_2) = (\mathbf{x}_1 \cdot \mathbf{x}_2)^2
```

(Azaz szorozd össze őket az eredeti, kicsi 2D térben, és az eredményt emeld négyzetre).

Nézzük meg:

- 2D szorzat: $(1 \cdot 3) + (2 \cdot 4) = 3 + 8 = 11$.
- Emeljük négyzetre: $11^2 = \mathbf{121}$.

**Eredmény:** Másodpercek törtrésze alatt megkaptuk a **tökéletesen ugyanazt** az eredményt ($121$), mintha elvégeztük volna a brutális 3D-s transzformációkat! A gép sosem járt a 3D térben, mégis tudott benne számolni.

### Az RBF Kernel (Radial Basis Function)

A könyv megemlíti a világ legnépszerűbb kernelét, az RBF kernelt:

```math
\large
K(\mathbf{x}, \mathbf{x}') = \exp\left( -\frac{\|\mathbf{x} - \mathbf{x}'\|^2}{2\sigma^2} \right)
```

Ennek a kernelnek az igazi szuperképessége az, hogy a matematikai háttere miatt a pontokat nem 3D-be, hanem **végtelen dimenziós térbe** vetíti ki! Ezzel szinte bármilyen, hihetetlenül komplex alakzatot szét tud vágni anélkül, hogy a számítógép összeomlana a végtelen dimenziók számolásától. A $\sigma$ (szigma) hiperparaméterrel tudjuk szabályozni, hogy a meghúzott döntési határ mennyire legyen sima vagy éppen göröngyös.

### Nézd meg működés közben!

Ez a rövid vizualizációs galéria zseniálisan mutatja be mozgásban, miről is szól ez az egész. Kattints a videóra, és figyeld meg, ahogy a 2D síkon körbe zárt piros/kék pontok hirtelen a magasba emelkednek (a 3D térbe), ahol egy egyszerű sík egyetlen vágással kettéválasztja őket. Pontosan ez a Kernel Trükk geometriai lényege.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/03%20Fejezet/KernelTrickPolynomialFeatureMapping.html)

# 3.5 k-Legközelebbi Szomszéd (k-Nearest Neighbors – kNN)

A kNN egy **nem-parametrikus** (non-parametric) tanuló algoritmus. Míg a lineáris regressziónál vagy az SVM-nél a gép megtanulta a $\mathbf{w}$ és $b$ paramétereket, majd az eredeti adatokat kidobtuk a memóriából, a kNN pont az ellenkezőjét csinálja: nincs külön "betanulási" fázisa, a teljes adathalmazt megtartja a memóriában, mert maga az adathalmaz a modell. Ezt **példány-alapú** (instance-based) tanulásnak hívjuk.

## 3.5.1 Hogyan hoz döntést (Predikció)?

Amikor érkezik egy új, eddig sosem látott $\mathbf{x}$ adat, a kNN a következőképpen jár el:

1. Kiszámolja a távolságot az új $\mathbf{x}$ pont és a memóriájában lévő összes régi adatpont között.
2. Kiválasztja azt a $k$ darab régi pontot, amelyek a legközelebb vannak (ezek a szomszédok).
3. **Osztályozás (Classification) esetén:** Megnézi ezen a $k$ szomszéd osztályát, és többségi szavazással (majority vote) dönt. Amelyik osztályból több van a szomszédok között, az lesz a végeredmény.
4. **Regresszió (Regression) esetén:** Fogja a $k$ szomszéd célértékét (pl. lakásárakat), és egyszerűen veszi az átlagukat.

### 🔢 1. Számpélda: A többségi szavazás

Tegyük fel, $k=5$. Bejön egy új adatpont. A gép megkeresi az 5 legközelebbi szomszédot, akiknek a címkéi a következők: `[Beteg, Egészséges, Beteg, Beteg, Egészséges]`.

Mivel 3 szavazat érkezett a "Beteg" osztályra és csak 2 az "Egészséges"-re, a modell az új pontot a **Beteg** kategóriába sorolja.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/03%20Fejezet/k-NearestNeighbors(kNN)Classifier.html)

---

## 3.5.2 A Távolság (és a Koszinusz hasonlóság)

Honnan tudjuk, hogy két pont "közel" van egymáshoz? Erre egy **távolságfüggvényt** (distance function) használunk. A legismertebb az **Euklideszi távolság** (a klasszikus vonalzós távolság), de van egy másik nagyon népszerű is: a **Koszinusz hasonlóság** (Cosine Similarity).

A Koszinusz hasonlóság nem a pontok közötti fizikai távolságot méri, hanem a két vektor (irány) által bezárt szöget. Minél kisebb a szög, annál jobban hasonlít a két vektor.

A képlete:

```math
\large
s(\mathbf{x}_i, \mathbf{x}_k) \overset{\text{def}}{=} \cos(\theta) = \frac{\sum_{j=1}^{D} x_i^{(j)} x_k^{(j)}}{\sqrt{\sum_{j=1}^{D} \left(x_i^{(j)}\right)^2} \sqrt{\sum_{j=1}^{D} \left(x_k^{(j)}\right)^2}}
```

- **A Számláló:** A két vektor skaláris szorzata ($\mathbf{x}_i \cdot \mathbf{x}_k$).
- **A Nevező:** A két vektor euklideszi hosszainak (normáinak) szorzata ($\|\mathbf{x}_i\| \cdot \|\mathbf{x}_k\|$).

**Értelmezése:**

- Ha a két vektor pontosan ugyanabba az irányba mutat (a bezárt szög $0^\circ$), a koszinusz hasonlóság $1$.
- Ha a két vektor merőleges egymásra ($90^\circ$), a hasonlóság $0$.
- Ha pontosan ellentétes irányba mutatnak ($180^\circ$), a hasonlóság $-1$.

**Fontos:** Mivel a kNN "távolságot" keres (amiből a kicsi a jó), és a koszinusznál a nagy érték (az $1$) jelenti a hasonlóságot, ezért távolságként úgy használjuk, hogy megszorozzuk $-1$-gyel.

### 🔢 2. Számpélda: Koszinusz hasonlóság kiszámítása

Vegyünk két kétdimenziós vektort (pl. egy dokumentum szavainak gyakorisága):

- $\mathbf{a} = [3, 0]$ (Ez a vektor tisztán jobbra mutat a vízszintes tengelyen)
- $\mathbf{b} = [3, 3]$ (Ez a vektor $45^\circ$-os szögben felfelé mutat)

**Számláló (Skaláris szorzat):** $(3 \cdot 3) + (0 \cdot 3) = 9 + 0 = 9$

**Nevező (A vektorok hossza):**

- Az $\mathbf{a}$ hossza: $\sqrt{3^2 + 0^2} = \sqrt{9} = 3$
- A $\mathbf{b}$ hossza: $\sqrt{3^2 + 3^2} = \sqrt{18} \approx 4.24$

**Végeredmény:**

```math
\large
\cos(\theta) = \frac{9}{3 \cdot 4.24} = \frac{9}{12.72} \approx \mathbf{0.707}
```

*(Matekos érdekesség: A $0.707$ pontosan a $45^\circ$ koszinusza ($\sqrt{2}/2$), ami logikus, hiszen az $[3, 0]$ és a $[3, 3]$ vektorok pontosan $45^\circ$-ot zárnak be egymással!)*

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/03%20Fejezet/CosineSimilarity.html)

---

## 3.5.3 A kNN Költségfüggvénye (Haladó elmélet – Li & Yang 2003)

Bár a kNN egy végtelenül egyszerű algoritmus, az 1960-as évekbeli feltalálása óta szinte senki nem vizsgálta, hogy mi is az ő "költségfüggvénye" (hogy milyen matematikai egyenletet old meg). Burkov itt bemutat egy 2003-as kutatást, ami rávilágít a kNN belső logikájára.

Ha a feladat bináris osztályozás ($y \in \{0, 1\}$), a jellemzővektorok normalizálva vannak (minden vektor hossza $1$), és a koszinusz hasonlóságot használjuk, akkor a kNN valójában egy **lokális lineáris osztályozóként** viselkedik.

Ez azt jelenti, hogy a kNN az új $\mathbf{x}$ pont környékén "épít" egy lokális $\mathbf{w_x}$ súlyvektort, pusztán a szomszédokból:

```math
\large
\mathbf{w_x} = \sum_{(\mathbf{x}', y') \in R_k(\mathbf{x})} y' \mathbf{x}'
```

### 🔢 3. Számpélda: A lokális súlyvektor építése

Mit jelent ez a képlet a gyakorlatban? A szumma ($\Sigma$) alatt látható $R_k(\mathbf{x})$ egyszerűen a megkeresett $k$ darab szomszédot jelenti.

Tegyük fel, $k=3$. A három szomszéd adatai (ahol $\mathbf{x}'$ a vektor, $y'$ a címke):

- szomszéd: $\mathbf{x}'_1 = [0.8, 0.6]$, címkéje $y'_1 = \mathbf{1}$ (Pozitív)
- szomszéd: $\mathbf{x}'_2 = [0.9, -0.4]$, címkéje $y'_2 = \mathbf{0}$ (Negatív)
- szomszéd: $\mathbf{x}'_3 = [0.1, 0.9]$, címkéje $y'_3 = \mathbf{1}$ (Pozitív)

Építsük fel a lokális súlyvektort:

```math
\large
\mathbf{w_x} = (1 \cdot [0.8, 0.6]) + (0 \cdot [0.9, -0.4]) + (1 \cdot [0.1, 0.9])
```

Vedd észre, hogy mivel $y \in \{0, 1\}$, a negatív osztályú szomszéd egyszerűen kiesik (megszorzódik nullával)! Csak a pozitív szomszédok vektorait adjuk össze:

```math
\large
\mathbf{w_x} = [0.8, 0.6] + [0.1, 0.9] = \mathbf{[0.9,\ 1.5]}
```

A kutatók levezették, hogy a kNN valójában az alábbi költségfüggvényt ($L$) minimalizálja lokálisan:

```math
\large
L = -\sum_{(\mathbf{x}', y') \in R_k(\mathbf{x})} y' \mathbf{x}' \cdot \mathbf{w_x} + \frac{1}{2} \|\mathbf{w_x}\|^2
```

Ha ennek kiszámolod az első deriváltját és egyenlővé teszed nullával (ahogy a lineáris regressziónál is tettük), hajszálpontosan megkapod a fenti $\mathbf{w_x}$ építő képletet.

**Összegzés:** Bár a kNN egy "lusta" (lazy) algoritmus, mert betanulás nélkül, futásidőben csak távolságokat méreget, matematikai szempontból a háttérben valójában minden egyes új adatpontra felépít egy apró, lokális lineáris modellt a legközelebbi szomszédokból!
