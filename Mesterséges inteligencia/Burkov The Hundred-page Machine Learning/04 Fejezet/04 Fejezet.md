# 4. Egy tanuló algoritmus anatómiája (Anatomy of a Learning Algorithm)

Ahogy az előző fejezetben láthattad, szinte minden tanuló algoritmus három fő építőelemből áll össze:

1. **Veszteségfüggvény (Loss function):** Ami megmondja, mekkorát tévedett a modell egyetlen adatponton.
2. **Optimalizációs kritérium (Cost function / Költségfüggvény):** Ami az összes adatponton ejtett hiba átlagolásával megadja a modell globális jóságát.
3. **Optimalizációs eljárás (Optimization routine):** A "motor", ami lépésről lépésre megtalálja azokat a paramétereket, amelyek minimalizálják a költségfüggvényt.

Bizonyos esetekben (mint a lineáris regressziónál) a minimum egyetlen lépésben, egy pontos egyenlettel (zárt formulával) kiszámolható. A legtöbb modern modellnél azonban (mint a neurális hálózatok) ez nem lehetséges. Ilyenkor lép színre a gépi tanulás legfontosabb motorja: a **Gradiens ereszkedés** (Gradient Descent).

## 4.1 Gradiens ereszkedés (Gradient Descent)

A gradiens ereszkedés egy **iteratív** (lépésről lépésre haladó) optimalizációs algoritmus. Célja, hogy megtalálja egy függvény (a mi esetünkben a költségfüggvény) lokális vagy globális minimumát.

### Hogyan működik vizuálisan?

Képzeld el, hogy bekötött szemmel állsz egy hegyen, és le akarsz jutni a völgy legaljára (ahol a hiba a legkisebb).

1. Körülnézel a lábaddal, és megállapítod, melyik irányba lejt a legmeredekebben a talaj. Ez a meredekség a **Gradiens**.
2. Lépsz egyet ebbe a lejtős irányba. A lépésed nagyságát a **Tanulási ráta** (Learning rate, α) határozza meg.
3. Ezt ismétled újra és újra, amíg el nem éred a völgy alját, ahol a talaj már vízszintes (a gradiens nulla).

A legtöbb klasszikus algoritmusnál (mint a logisztikus regresszió vagy az SVM) a "hegyoldalunk" (a költségfüggvény) **konvex**. Ez azt jelenti, hogy olyan, mint egy tökéletes tálca: csak egyetlen völgy (egy globális minimum) van benne, tehát nem lehet "beragadni" egy rossz gödörbe.

## 4.2 Példa: Lineáris regresszió optimalizálása Pythonban

Nézzük meg egy konkrét, 1-dimenziós lineáris regressziós feladaton, hogyan programozható le ez a matematikai koncepció!

**A feladat:** Vállalatok rádiós reklámköltéseiből ($x$) próbáljuk megjósolni az eladott egységek számát ($y$). A modellünk egy egyszerű egyenes:

```math
f(x) = wx + b
```

A költségfüggvényünk (MSE) az előző fejezetből ismerős:

```math
L = \frac{1}{N} \sum_{i=1}^{N} (y_i - (wx_i + b))^2
```

### 1. A gradiensek (deriváltak) kiszámítása

Ahhoz, hogy tudjuk, merre van "lefelé", a láncszabállyal kiszámoljuk a költségfüggvény parciális deriváltjait a két paraméterre, $w$-re és $b$-re.

**A láncszabály lépései a $w$ deriváltjához:**

- A külső függvény az emelés négyzetre: $f(z) = z^2$. Deriváltja $2z$.
- A belső függvény az $(y_i - wx_i - b)$. Mivel most csak $w$ szerint deriválunk, $x_i$ a változó melletti szorzó, az $y_i$ és a $b$ pedig konstans. A belső derivált így $-x_i$.
- A kettőt összeszorozva kapjuk a végeredményt.

```math
\frac{\partial L}{\partial w} = \frac{1}{N} \sum_{i=1}^{N} -2x_i (y_i - (wx_i + b))
```

```math
\frac{\partial L}{\partial b} = \frac{1}{N} \sum_{i=1}^{N} -2(y_i - (wx_i + b))
```

### 2. A frissítési szabály

Ezután egy $0.0$-ról indított $w$ és $b$ értéket minden egyes lépésben frissítünk:

```math
w_{\text{új}} = w_{\text{régi}} - \alpha \cdot \frac{\partial L}{\partial w}
```

```math
b_{\text{új}} = b_{\text{régi}} - \alpha \cdot \frac{\partial L}{\partial b}
```

(Az $\alpha$ (alpha) a tanulási ráta, ami megszabja, mekkora lépéseket teszünk.)

## Kézzel fogható számpélda: A Gradiens Ereszkedés első lépései

Hogy megértsük a matematikai motort, hajtsunk végre manuálisan egy teljes iterációt (Epoch 1) egy végtelenül leegyszerűsített adathalmazon!

### 1. A kiindulási adatok és paraméterek

Legyen egy "játék" adathalmazunk, ahol csak $N=2$ vállalatunk van.

- **1. Vállalat:** Költés: $x_1 = 1$, Eladás: $y_1 = 2$
- **2. Vállalat:** Költés: $x_2 = 2$, Eladás: $y_2 = 4$

*(Emberi szemmel azonnal látjuk, hogy a tökéletes összefüggés $y = 2x$, azaz az optimális modell $w=2$ és $b=0$ lenne. Lássuk, rájön-e a gép!)*

**Kezdeti beállítások a gépben:**

- Kezdő súly: $w = 0$
- Kezdő eltolás (bias): $b = 0$
- Tanulási ráta: $\alpha = 0.1$ (Mekkora lépést tegyen a gép)

### 2. A hibák (Errors) kiszámítása a 0. pillanatban

A modellünk jelenleg: $f(x) = 0 \cdot x + 0 = 0$. Vagyis mindenre nullát tippel.

- 1. vállalatnál a tévedés: $(y_1 - f(x_1)) = 2 - 0 = \mathbf{2}$
- 2. vállalatnál a tévedés: $(y_2 - f(x_2)) = 4 - 0 = \mathbf{4}$

### 3. A Gradiensek (Deriváltak) kiszámítása

A képletek alapján kiszámoljuk, merre lejt a hegyoldal (vagyis mennyit változzon a $w$ és a $b$). A derivált képlete a tévedéseket használja fel:

**A gradiens $w$ szerint (Meredekség módosítása):**

```math
\frac{\partial L}{\partial w} = \frac{1}{N} \left( -2 x_1 \cdot \text{tévedés}_1 - 2 x_2 \cdot \text{tévedés}_2 \right)
```

Behelyettesítve:

```math
\frac{\partial L}{\partial w} = \frac{1}{2} \left( -2(1)(2) + -2(2)(4) \right) = \frac{1}{2} \left( -4 - 16 \right) = \frac{-20}{2} = \mathbf{-10}
```

**A gradiens $b$ szerint (Eltolás módosítása):**

```math
\frac{\partial L}{\partial b} = \frac{1}{N} \left( -2 \cdot \text{tévedés}_1 - 2 \cdot \text{tévedés}_2 \right)
```

Behelyettesítve:

```math
\frac{\partial L}{\partial b} = \frac{1}{2} \left( -2(2) + -2(4) \right) = \frac{1}{2} \left( -4 - 8 \right) = \frac{-12}{2} = \mathbf{-6}
```

### 4. A paraméterek frissítése (A "Lépés")

Most, hogy megvannak az irányok ($-10$ és $-6$), a gép tesz egy lépést lefelé, a tanulási ráta ($\alpha = 0.1$) nagyságával szorozva:

```math
w_{\text{új}} = w_{\text{régi}} - \alpha \cdot \frac{\partial L}{\partial w} = 0 - 0.1 \cdot (-10) = 0 + 1 = \mathbf{1}
```

```math
b_{\text{új}} = b_{\text{régi}} - \alpha \cdot \frac{\partial L}{\partial b} = 0 - 0.1 \cdot (-6) = 0 + 0.6 = \mathbf{0.6}
```

### 🎉 Az első Epoch (iteráció) véget ért!

A gép új modellje mostantól: $f(x) = 1x + 0.6$

**Bizonyítás: Valóban okosodott a gép?**

Nézzük meg az új tévedéseket az 1. lépés után!

- 1. vállalat becslése: $1(1) + 0.6 = 1.6 \quad \rightarrow$ Új hiba: $2 - 1.6 = \mathbf{0.4}$ (A korábbi 2.0 helyett!)
- 2. vállalat becslése: $1(2) + 0.6 = 2.6 \quad \rightarrow$ Új hiba: $4 - 2.6 = \mathbf{1.4}$ (A korábbi 4.0 helyett!)

### Konklúzió

Egyetlen iteráció alatt a gép radikálisan csökkentette a hibát, közeledve a tökéletes ($w=2, b=0$) állapothoz! Ezt ismétli meg a Python ciklus több ezerszer (Epoch), amíg a hiba el nem éri a nullát.

### A Python kód magyarázata

Lássuk, hogyan néz ki ez a fenti elmélet tiszta Python kódban!

```python
def update_w_and_b(spendings, sales, w, b, alpha):
    dl_dw = 0.0
    dl_db = 0.0
    N = len(spendings)

    for i in range(N):
        dl_dw += -2 * spendings[i] * (sales[i] - (w * spendings[i] + b))
        dl_db += -2 * (sales[i] - (w * spendings[i] + b))

    w = w - (1 / float(N)) * dl_dw * alpha
    b = b - (1 / float(N)) * dl_db * alpha

    return w, b
```

Hogy eljussunk a völgy aljára, ezt a függvényt több ezerszer kell lefuttatnunk (egy teljes kört az adatokon egy **Epoch**-nak hívunk).

```python
def train(spendings, sales, w, b, alpha, epochs):
    for e in range(epochs):
        w, b = update_w_and_b(spendings, sales, w, b, alpha)

        if e % 400 == 0:
            print("epoch:", e, "loss: ", avg_loss(spendings, sales, w, b))

    return w, b
```

### Az eredmény

Ha elindítjuk a kódunkat $\alpha = 0.001$-es lépésközzel:

```
epoch: 0     loss: 92.32
epoch: 400   loss: 33.79
epoch: 1200  loss: 24.33
epoch: 2800  loss: 19.07
```

<img width="696" height="422" alt="1" src="https://github.com/user-attachments/assets/996a7745-c13b-4bce-b4a9-641892079feb" />

>  Azt mutatja be vizuálisan, ahogy a kiindulási, vízszintes 0-ás vonal (Epoch 0) az iterációk során lassan ráfekszik a pontokra, amíg el nem éri a tökéletes dőlésszöget (Epoch 3000).

## További fejlesztések a Gradiens Ereszkedésben

Bár az alap algoritmus gyönyörűen működik, a gyakorlatban két nagy problémája van: nagyon érzékeny az $\alpha$ (tanulási ráta) beállítására, és hatalmas adathalmazokon iszonyatosan lassú (mert minden egyes lépéshez – `for` ciklushoz – végig kell néznie a teljes adathalmazt).

Erre találták ki a következő kiegészítőket:

- **Stochastic Gradient Descent (SGD):** Nem nézi végig a teljes adathalmazt egy lépéshez, csak egy apró véletlenszerű mintát (batch). Kicsit cikk-cakkosan halad a völgy felé, de nagyságrendekkel gyorsabb.
- **Adagrad / RMSprop:** Ezek az algoritmusok automatikusan állítgatják a lépésközt ($\alpha$) a futás során.
- **Momentum:** Eltárolja a korábbi mozgás "lendületét". A legnépszerűbb modern ötvözet minderre az **Adam** optimalizáló.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/04%20Fejezet/3DGradientDescent.html)

## 4.3 Hogyan dolgoznak a gépi tanulás mérnökei a gyakorlatban?

Hacsak nem vagy egy egyetemi kutató, vagy nem dolgozol egy gigászi tech-cég (Google, Meta) K+F (R&D) részlegénél óriási büdzsével, soha nem fogsz saját magad implementálni gradiens ereszkedést vagy SVM egyenletrendszert.

A gépi tanulás mérnökei (Machine Learning Engineers) **szoftverkönyvtárakat** (libraries) használnak. Ezek nyílt forráskódú, professzionális szoftvercsomagok, amelyeket matematikusok és C/C++ programozók hada optimalizált a maximális stabilitásra és sebességre.

A legnépszerűbb, ipari sztenderd könyvtár a klasszikus gépi tanulásra a **Scikit-Learn** (Pythonban és C-ben írva).

### A Scikit-Learn csodája (Python kód)

Emlékszel, az előző részben írtunk egy saját `train` függvényt gradiens ereszkedéssel, amihez több sornyi bonyolult matekot kellett leprogramozni. Nézd meg, hogyan néz ki ugyanez a feladat a Scikit-Learn használatával:

```python
# Importáljuk a beépített Lineáris Regresszió algoritmust
from sklearn.linear_model import LinearRegression

def train(x, y):
    # Létrehozzuk a modellt, és a .fit() függvénnyel azonnal betanítjuk az adatokon
    model = LinearRegression().fit(x, y)
    return model

# 1. Betanítjuk a modellt a meglévő adatokon
model = train(x, y)

# 2. Bejön egy új adat (egy vállalat, ami 23.0 millió dollárt költött reklámra)
x_new = [[23.0]]  # (A scikit-learn mátrixokat vár, ezért a dupla zárójel)

# 3. Kérünk egy predikciót a betanított modelltől
y_new = model.predict(x_new)
print(y_new)
```

**Az eredmény:** A kód pontosan `13.97`-et fog kiköpni, hajszálpontosan ugyanúgy, mintha te írtad volna meg a gradiens ereszkedést.

**Mi ebben a zseniális?** A Scikit-Learn API-ja (felülete) annyira egységes, hogy ha hirtelen rájössz, hogy a Lineáris Regresszió helyett egy kNN-t vagy egy Döntési fát akarsz használni, egyszerűen átírod a `LinearRegression` szót `DecisionTreeRegressor`-ra, és a kód többi része változatlan marad. Az algoritmusok csereszabatosak, mint a legókockák!

---

## 4.4 Az algoritmusok sajátosságai (Gyakorlati szempontok)

Bár az algoritmusok (SVM, kNN, Döntési fa, Logisztikus regresszió) elméleti szinten mind a hiba minimalizálására törekszenek, a gyakorlatban nagyon másképp viselkednek. Egy adattudós (Data Analyst) feladata kiválasztani a feladathoz legjobban illőt. Mik a fő különbségek?

### 1. Hiperparaméterek

Ezeket neked kell beállítanod a betanítás előtt (a gép ezeket nem tudja megtanulni).

- **SVM:** $C$ (a margó és a büntetés aránya).
- **Döntési fa (ID3):** Maximális mélység ($d$) vagy az entrópia-csökkenés minimuma ($\epsilon$).
- **Gradiens ereszkedés:** Tanulási ráta ($\alpha$).

### 2. Bemeneti adatok típusa (Számok vs. Kategóriák)

A legtöbb matematikai algoritmus (SVM, kNN, Logisztikus regresszió) szigorúan csak **számokat** fogad el (pl. vektorokat). A gépek nem tudnak a "Piros" vagy "Zöld" szavakkal osztani és szorozni.

Azonban a **Döntési fák** zokszó nélkül megeszik a szöveges (kategorikus) adatokat is, hiszen egy faágra simán fel lehet írni, hogy: *"HA a szín == Piros, menj balra"*.

> *(Megjegyzés: A gyakorlatban a Scikit-Learn mindenhez számokat kér, így a szövegeket mindig át kell majd alakítanunk számokká, de erről a könyv a következő fejezetben szól).*

### 3. Osztályok súlyozása (Osztályegyensúly felborulása)

Képzeld el, hogy rákot próbálsz szűrni: van $10\,000$ egészséges és mindössze $10$ beteg páciensed. A gépek imádnak lusták lenni: ha a gép mindenkit egészségesnek mond, máris $99.9\%$-os a pontossága, mégsem ér semmit.

Egyes algoritmusoknál (pl. SVM) be lehet állítani egy súlyozást. Megmondhatod a gépnek: *"Ha egy rákos (kisebbségi) pácienst elnézel, azért 1000-szer nagyobb büntetést kapsz, mintha egy egészségeset tévesztenél el."* Így kényszeríted az algoritmust, hogy figyeljen a ritka, de fontos osztályra.

### 4. A kimenet típusa (Osztály vs. Valószínűség)

Ha egy adatot beküldesz, a modell válasza többféle lehet:

- **SVM / kNN:** Ezek "kemény" (hard) osztályozók. Csak a végső kategóriát adják vissza (pl. *"Ez egy Macska"*). Nem tudják megmondani, mennyire biztosak benne.
- **Logisztikus Regresszió / Döntési Fa:** Ezek "lágy" (soft) osztályozók. Egy $0$ és $1$ közötti pontszámot (valószínűséget) adnak vissza. (Pl. *"0.87 az esélye, hogy ez egy Macska"*). Ez a gyakorlatban sokkal hasznosabb, mert a mérnök dönthet úgy, hogy csak $0.95$ felett hisz a gépnek.

### 5. Tanulási stílus (Egyszeri vs. Iteratív / Batch)

A Döntési fák, SVM, kNN csak úgy tudnak tanulni, ha egyszerre látják a memóriában a teljes, $100\%$-os adathalmazt. Ha holnap kapsz 10 új adatpontot, az egész modellt a nulláról, elölről kell újra tanítani az összes régi és az új adat együttesével.

Ezzel szemben bizonyos algoritmusok (pl. Naiv Bayes, Neurális hálózatok, SGD algoritmusok) képesek **iteratívan** tanulni. Úgy is hívják ezt, hogy "Online learning". Ha jön egy adag (batch) új adat, a modellt nem kell kidobni; egyszerűen "rá lehet tanítani" a meglévő "agyára" az új információkat.

### 6. Feladat típusa

Néhány algoritmus (Döntési fák, kNN, SVM – bizonyos kiegészítésekkel) igazi svájci bicska: **Osztályozásra (Classification) és Regresszióra (Regression) is használhatók**. Mások (mint a Logisztikus regresszió) szigorúan csak osztályozásra alkalmasak.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/04%20Fejezet/ClassificationDecisionBoundaries.html)
