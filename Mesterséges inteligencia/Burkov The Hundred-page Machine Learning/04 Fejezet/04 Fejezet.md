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

> 📍 **[KÉP HELYE: IDE JÖN A 2. ÁBRA]**
>
> Itt kell szerepelnie a könyv *Figure 2* ábrájának, amely egy 6 panelből álló képsorozat. Azt mutatja be vizuálisan, ahogy a kiindulási, vízszintes 0-ás vonal (Epoch 0) az iterációk során lassan ráfekszik a pontokra, amíg el nem éri a tökéletes dőlésszöget (Epoch 3000).

## További fejlesztések a Gradiens Ereszkedésben

Bár az alap algoritmus gyönyörűen működik, a gyakorlatban két nagy problémája van: nagyon érzékeny az $\alpha$ (tanulási ráta) beállítására, és hatalmas adathalmazokon iszonyatosan lassú (mert minden egyes lépéshez – `for` ciklushoz – végig kell néznie a teljes adathalmazt).

Erre találták ki a következő kiegészítőket:

- **Stochastic Gradient Descent (SGD):** Nem nézi végig a teljes adathalmazt egy lépéshez, csak egy apró véletlenszerű mintát (batch). Kicsit cikk-cakkosan halad a völgy felé, de nagyságrendekkel gyorsabb.
- **Adagrad / RMSprop:** Ezek az algoritmusok automatikusan állítgatják a lépésközt ($\alpha$) a futás során.
- **Momentum:** Eltárolja a korábbi mozgás "lendületét". A legnépszerűbb modern ötvözet minderre az **Adam** optimalizáló.
