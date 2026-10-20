# 2. Jelölések és definíciók (Notation and Definitions)

A gépi tanulás megértéséhez elengedhetetlen a matematikai alapjelölések pontos ismerete. Ez a fejezet azokat a koncepciókat veszi át, amelyek az algoritmusok formális leírásához szükségesek.

## 2.1.1 Skalárok, vektorok és halmazok

### Skalárok

A skalár egy egyszerű numerikus (szám) érték, például $\large 15$ vagy $\large -3.25$.

**Jelölése:** Dőlt (italic) betűkkel, például $\large x$ vagy $\large a$.

### Vektorok

A vektor skalár értékek (úgynevezett attribútumok) rendezett listája.

**Jelölése:** Félkövér (bold) betűkkel, például $\large \mathbf{x}$ vagy $\large \mathbf{w}$.

**Geometriai értelmezés:** A többdimenziós térben ábrázolhatók iránymutató nyílként vagy konkrét pontként.

<img width="1258" height="466" alt="1" src="https://github.com/user-attachments/assets/1f8a6e92-1da4-46a5-b8aa-f7595993ddf4" />


> *Figure 1: Vektorok vizuális ábrázolása a kétdimenziós térben. Forrás: Ensar Karabudak / Getty Images*

**Indexelés:** Egy vektor adott dimenzióját (pozícióját) felső indexben, zárójelben jelöljük. Például a $\large \mathbf{w}$ vektor $\large j$-edik eleme: $\large w^{(j)}$.

⚠️ **Fontos megkülönböztetés:** A $\large x^{(j)}$ **nem** hatványozást jelent! Ha egy vektor adott elemét a négyzetre akarjuk emelni, azt így írjuk: $\large \left(x^{(j)}\right)^2$.

**Többszörös indexelés:** Bonyolultabb struktúráknál (pl. neurális hálózatok) alsó és felső indexeket is használunk: $\large x_{l,u}^{(j)}$ jelentheti az $\large l$-edik réteg $\large u$-adik neuronjának $\large j$-edik bemeneti jellemzőjét.

### Halmazok (Sets)

A halmaz egyedi elemek rendezetlen gyűjteménye.

**Jelölése:** Kalligrafikus nagybetűvel, pl. $\large \mathcal{S}$.

**Véges halmazok:** Kapcsos zárójellel jelöljük: $\large \{1, 3, 18, 23\}$ vagy $\large \{x_1, x_2, \dots, x_n\}$.

**Intervallumok (végtelen halmazok):**

- Zárt intervallum (tartalmazza a határértékeket is): $\large [a, b]$
- Nyílt intervallum (nem tartalmazza a határokat): $\large (a, b)$

A valós számok halmazát a mínusz végtelentől a plusz végtelenig egy speciális szimbólum, a $\large \mathbb{R}$ jelöli.

**Halmazműveletek:**

- Eleme a halmaznak: $\large x \in S$
- Metszet (közös elemek): $\large S_3 = S_1 \cap S_2$
- Unió (összes elem egyesítése): $\large S_3 = S_1 \cup S_2$

## 2.1.2 Szumma jelölés (Nagy Szigma – $\large \Sigma$)

Egy gyűjtemény vagy egy vektor elemeinek összeadását jelöli.

A vektor elemeinek összege:

```math
\large
\sum_{j=1}^{m} x^{(j)} \overset{\text{def}}{=} x^{(1)} + x^{(2)} + \dots + x^{(m-1)} + x^{(m)}
```

(A $\large \overset{\text{def}}{=}$ jelentése: "definíció szerint egyenlő".)

## 2.1.3 Pi jelölés (Nagy Pi – $\large \Pi$)

Hasonló a szummához, de itt az elemek szorzását végezzük el egymással:

```math
\large
\prod_{i=1}^{n} x_i \overset{\text{def}}{=} x_1 \cdot x_2 \cdot \dots \cdot x_n
```

## 2.1.4 Műveletek halmazokon

**Származtatott halmaz (Set comprehension):** Kialakíthatunk új halmazokat feltételek alapján.

Példa:

```math
\large
S' = \left\{ x^2 \mid x \in S,\ x > 3 \right\}
```

(Jelentése: Az új $\large S'$ halmaz tartalmazza minden olyan $\large S$-beli $\large x$ elem négyzetét, ahol $\large x$ nagyobb, mint 3).

**Kardinalitás:** A halmaz elemeinek számát jelöli: $\large |S|$.

## 2.1.5 Műveletek vektorokon (Részletes matematikai magyarázat)

### 1. Összeadás és kivonás

Két azonos méretű vektor összeadása vagy kivonása komponensenként történik.

```math
\large
\mathbf{x} + \mathbf{z} = \begin{bmatrix} x^{(1)} + z^{(1)} \\ x^{(2)} + z^{(2)} \\ \vdots \\ x^{(m)} + z^{(m)} \end{bmatrix}
```

### 2. Skalárral való szorzás

Ha egy vektort megszorzunk egy számmal, a vektor minden egyes elemét meg kell szorozni az adott számmal. Az eredmény egy vektor.

```math
\large
c\,\mathbf{x} \overset{\text{def}}{=} \begin{bmatrix} c\,x^{(1)} \\ c\,x^{(2)} \\ \vdots \\ c\,x^{(m)} \end{bmatrix}
```

### 3. Skaláris szorzat (Dot-product)

Két azonos dimenziójú vektor skaláris szorzatának eredménye egyetlen szám (skalár). A megfelelő pozícióban lévő elemeket összeszorozzuk, majd ezeket a szorzatokat összeadjuk. Jelölése gyakran $\large \mathbf{w}\mathbf{x}$ vagy $\large \mathbf{w} \cdot \mathbf{x}$.

```math
\large
\mathbf{w} \cdot \mathbf{x} \overset{\text{def}}{=} \sum_{i=1}^{m} w^{(i)} x^{(i)}
```

Ha a két vektor dimenziója nem egyezik, a művelet értelmezhetetlen.

### 4. Mátrix-vektor szorzás ($\large W\mathbf{x}$)

Amikor egy mátrixszal ($\large W$) szorzunk egy vektort ($\large \mathbf{x}$), a vektort alapértelmezetten oszlopvektornak tekintjük.

**Mikor végezhető el?** Csak akkor, ha a vektor mérete (dimenziója) megegyezik a mátrix oszlopainak számával.

**Az eredmény:** Egy új vektor, amelynek annyi sora (dimenziója) lesz, ahány sora a mátrixnak van.

**Kidolgozott számpélda a könyvből:**

Legyen a mátrixunk egy $\large 2 \times 3$-as mátrix (2 sor, 3 oszlop):

```math
\large
W = \begin{bmatrix}
w_{1,1} & w_{1,2} & w_{1,3} \\
w_{2,1} & w_{2,2} & w_{2,3}
\end{bmatrix}
```

És egy 3-dimenziós bemeneti vektorunk:

```math
\large
\mathbf{x} = \begin{bmatrix} x^{(1)} \\ x^{(2)} \\ x^{(3)} \end{bmatrix}
```

A szorzás során a mátrix minden sorának és a vektornak vesszük a skaláris szorzatát:

```math
\large
W\mathbf{x} = \begin{bmatrix}
w_{1,1} x^{(1)} + w_{1,2} x^{(2)} + w_{1,3} x^{(3)} \\
w_{2,1} x^{(1)} + w_{2,2} x^{(2)} + w_{2,3} x^{(3)}
\end{bmatrix}
```

### 5. Transzponálás és szorzás balról ($\large \mathbf{x}^T W$)

Ha a vektort a mátrix bal oldalára tesszük, transzponálni kell, azaz az oszlopvektorból sorvektort kell csinálni. Ezt egy felső index $\large T$ vagy $\large \top$ jelöli: $\large \mathbf{x}^T$.

```math
\large
\mathbf{x} = \begin{bmatrix} x^{(1)} \\ x^{(2)} \end{bmatrix}
\quad \Rightarrow \quad
\mathbf{x}^T = \begin{bmatrix} x^{(1)} & x^{(2)} \end{bmatrix}
```

Ebben az esetben a vektor dimenziószámának a mátrix sorainak számával kell megegyeznie. Az eredmény egy sorvektor lesz, ahol a vektor az egyes mátrix-oszlopokkal lép skaláris szorzatba.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez](https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/02%20Fejezet/widget.html)


## 2.1.6 Függvények (Functions)

A függvény egy olyan matematikai kapcsolat (hozzárendelés), amely egy $\large X$ halmaz (az értelmezési tartomány vagy bemenet) minden egyes eleméhez hozzárendeli egy $\large Y$ halmaz (a képtartomány vagy kimenet) pontosan egy elemét.

**Jelölése:** $\large y = f(x)$

- $\large x$: Az argumentum, a bemenet, vagy a függvény változója.
- $\large y$: A függvény értéke, a kimenet.

### Lokális és Globális Minimum

A gépi tanulásban (mint ahogy az 1. fejezetben az SVM-nél láttuk) folyamatosan minimalizálni akarunk valamit (pl. a hibát).

**Lokális minimum:** Azt mondjuk, hogy az $\large f(x)$ függvénynek az $\large x = c$ pontban lokális minimuma van, ha a $\large c$ közvetlen környezetében (egy nyílt intervallumon belül) nincs nála kisebb érték. Matematikailag: $\large f(x) \ge f(c)$.

**Globális minimum:** Az összes lokális minimum közül a legkisebb érték a teljes értelmezési tartományon. A gépi tanulás Szent Grálja ennek a megtalálása.

*(Nyílt intervallum: Olyan számtartomány, amely nem tartalmazza a végpontjait, jelölése gömbölyű zárójellel történik, pl. a $(0, 1)$ jelentése: $0$-nál nagyobb, de $1$-nél kisebb számok).*

> <img width="812" height="611" alt="2" src="https://github.com/user-attachments/assets/cc3cbd9b-8e94-4391-b640-3815549e94f9" />

 *Figure 2: A local and a global minima of a function.* című ábrát, ami egy hullámzó görbét mutat, bejelölve rajta a völgyek alját (lokális minimumok) és a legmélyebb völgyet (globális minimum).

**Vektorfüggvény:** Olyan függvény, amelynek kimenete nem egyetlen szám, hanem egy vektor. Jelölése: $\large \mathbf{y} = f(\mathbf{x})$. Bemenete lehet skalár vagy vektor is.

---

## 2.1.7 A Max és az Arg Max operátorok (Nagyon fontos különbség!)

Adott egy $\large A$ halmaz, például: $\large A = \{a_1, a_2, \dots, a_n\}$.

Amikor egy függvény legnagyobb értékét keressük, két teljesen különböző kérdést tehetünk fel.

**A maximum érték (Mi a legnagyobb kimenet?):**

```math
\large
\max_{a \in A} f(a)
```

Ez magát a legmagasabb függvényértéket (a kimenetet) adja vissza.

**Az argumentum maximuma (Melyik bemenet okozza a legnagyobb kimenetet?):**

```math
\large
\arg\max_{a \in A} f(a)
```

Ez azt a konkrét $\large a$ bemeneti elemet adja vissza a halmazból, amelynél a függvény a maximumát felveszi.

**Számpélda a megértéshez:**

Legyen a vizsgált halmazunk három szám: $\large A = \{-3, 0, 2\}$.

A függvényünk legyen a négyzetre emelés: $\large f(x) = x^2$.

Számoljuk ki a kimeneteket: $\large f(-3) = 9$, $\quad f(0) = 0$, $\quad f(2) = 4$.

Ha a kérdés a **max**:

```math
\large
\max_{x \in A} f(x) = 9
```

(mert a 9 a legnagyobb kimenet).

Ha a kérdés az **argmax**:

```math
\large
\arg\max_{x \in A} f(x) = -3
```

(mert a $\large -3$-as bemenet okozta a 9-et).

Ugyanez a logika érvényes a **min** és **argmin** operátorokra is.

---

## 2.1.8 Értékadás operátor (Assignment Operator)

Amikor programozunk vagy algoritmust írunk fel, a változóknak értéket adunk. A könyv erre a balra mutató nyilat használja (az OCR-ed ezt Ω-nak nézte).

```math
\large
a \leftarrow f(x)
```

Jelentése: Az $\large a$ változó vegye fel (kapja meg) az $\large f(x)$ eredményét.

Ugyanígy vektoroknál: $\large \mathbf{a} \leftarrow [a_1, a_2]$ azt jelenti, hogy az $\large \mathbf{a}$ vektor megkapja ezt a kétdimenziós értéket.

---

## 2.1.9 Derivált és Gradiens (A tanulási algoritmusok alapja)

### A derivált (Derivative)

A derivált ($\large f'$) egy függvény, vagy egy konkrét érték, ami azt mutatja meg, hogy milyen gyorsan nő vagy csökken az eredeti $\large f$ függvény egy adott pontban (ez a függvény meredeksége).

- Ha $\large f'(x)$ pozitív: a függvény ott éppen növekszik (felfelé tart).
- Ha $\large f'(x)$ negatív: a függvény ott éppen csökken (lefelé tart).
- Ha $\large f'(x) = 0$: a függvény meredeksége ott vízszintes. (Ez egy kritikus pont, itt lehet a függvénynek lokális minimuma vagy maximuma!).

**Alapderiváltak példái:**

- Ha $\large f(x) = x^2$, akkor a deriváltja $\large f'(x) = 2x$.
- Ha $\large f(x) = 2x$, akkor a deriváltja $\large f'(x) = 2$.
- Ha $\large f(x) = c$ (ahol $\large c$ egy konstans, pl. $\large 5$), a deriváltja mindig $\large 0$.

### A Láncszabály (Chain rule)

Ha összetett függvényünk van (egyik függvény a másikba van ágyazva), akkor a deriválást "kívülről befelé" haladva végezzük.

**Képlete:** Ha $\large F(x) = f(g(x))$, akkor

```math
\large
F'(x) = f'(g(x)) \cdot g'(x)
```

**Számpélda a könyvből:**

Legyen $\large F(x) = (5x + 1)^2$. Ezt bontsuk két részre:

- Belső rész: $\large g(x) = 5x + 1$. Ennek deriváltja: $\large g'(x) = 5$.
- Külső rész: $\large f(z) = z^2$ (ahol a $\large z$ most az egész belső zárójel). Ennek deriváltja: $\large 2z$.

Alkalmazzuk a láncszabályt:

```math
\large
F'(x) = 2 \cdot (5x + 1) \cdot g'(x) = 2 \cdot (5x + 1) \cdot 5 = 50x + 10
```

### A Gradiens (A derivált több dimenzióban)

A gépi tanulásban a modelljeink (mint láttuk a $\large 20\,000$ dimenziós vektornál) nem egy, hanem rengeteg bemenettel dolgoznak. Ilyenkor a sima derivált már nem elég, a gradiensre (jelölése: $\large \nabla f$, ejtsd: nabla f) van szükség.

A gradiens nem más, mint a parciális deriváltakból álló vektor.

### Mi az a parciális derivált? ($\large \frac{\partial f}{\partial x^{(i)}}$)

Amikor többváltozós függvényt deriválunk, kiválasztunk egyetlen bemenetet (pl. $\large x^{(1)}$), és azt mondjuk: "Csak ezt tekintem változónak, az összes többi bemenetet úgy kezelem, mintha sima konstans számok lennének."

**Számpélda a könyvből:**

Legyen egy kétváltozós függvényünk:

```math
\large
f(x^{(1)}, x^{(2)}) = a x^{(1)} + b x^{(2)} + c
```

(ahol $\large a, b, c$ konstans számok).

**Parciális derivált $\large x^{(1)}$ szerint:**

Mindent konstansnak tekintünk, kivéve az $\large x^{(1)}$-et.

```math
\large
\frac{\partial f}{\partial x^{(1)}} = a + 0 + 0 = a
```

(Az $\large ax^{(1)}$ deriváltja $\large a$. A $\large bx^{(2)}$ és a $\large c$ konstansnak számít, így a deriváltjuk $\large 0$.)

**Parciális derivált $\large x^{(2)}$ szerint:**

Most mindent konstansnak tekintünk, kivéve az $\large x^{(2)}$-t.

```math
\large
\frac{\partial f}{\partial x^{(2)}} = 0 + b + 0 = b
```

A függvény gradiense ($\large \nabla f$) ezen parciális deriváltak vektora lesz:

```math
\large
\nabla f = \begin{bmatrix} \dfrac{\partial f}{\partial x^{(1)}} \\[10pt] \dfrac{\partial f}{\partial x^{(2)}} \end{bmatrix} = \begin{bmatrix} a \\ b \end{bmatrix}
```



<img width="1265" height="558" alt="3" src="https://github.com/user-attachments/assets/21262f34-1b32-4837-b670-3dd1e915fbbd" />

 *Figure 3: A probability mass function and a probability density function.* ábrára, a hozzá tartozó valószínűségszámítási szövegrészt még nem küldted el. Amikor a könyvben ehhez az ábrához érsz, ide illeszd be!
>
> # 2.2 Valószínűségi változó (Random Variable)

A valószínűségi változó (általában dőlt nagybetűvel jelölve, mint $\large X$) egy olyan változó, amelynek lehetséges értékei egy véletlenszerű jelenség kimeneteitől függenek. Két fő típusát különböztetjük meg:

## 1. Diszkrét valószínűségi változó

Olyan változó, amely csak megszámlálhatóan sok, elkülönülő értéket vehet fel (pl. piros, sárga, kék, vagy 1, 2, 3...).

**Valószínűség-tömegfüggvény (Probability Mass Function – PMF):** Ez a függvény sorolja fel, hogy az egyes értékeket milyen valószínűséggel veszi fel a változó.

**Példa:** $\large \Pr(X = \text{piros}) = 0.3$, $\large \Pr(X = \text{sárga}) = 0.45$, $\large \Pr(X = \text{kék}) = 0.25$.

**Alapszabály:** Minden valószínűség $\large \ge 0$, és a valószínűségek összege pontosan $\large 1$.


><img width="585" height="551" alt="3a" src="https://github.com/user-attachments/assets/2500c47e-0af3-4daa-8599-b211c57f8ce5" />

 *Figure 3a* ábra, amely a diszkrét valószínűség-tömegfüggvényt (PMF) mutatja be oszlopdiagram formájában.

## 2. Folytonos valószínűségi változó

Olyan változó, amely egy adott intervallumon belül végtelen sok lehetséges értéket vehet fel (pl. testmagasság, súly, idő).

Mivel végtelen sok lehetséges érték van, annak a valószínűsége, hogy a változó hajszálpontosan egy konkrét értéket vesz fel, nulla: $\large \Pr(X = c) = 0$.

**Valószínűségsűrűség-függvény (Probability Density Function – PDF):** Mivel a pontszerű valószínűség $\large 0$, folytonos esetben sűrűségfüggvénnyel (jelölése: $\large f_X$) írjuk le az eloszlást. Ez egy folytonos görbe.

**Alapszabály:** A függvény értéke nem lehet negatív, és a görbe alatti teljes terület pontosan $\large 1$.

<img width="617" height="530" alt="3b" src="https://github.com/user-attachments/assets/4544c6c1-093a-4017-88f9-893bff114b90" />

>  *Figure 3b* ábra, amely a folytonos valószínűségsűrűség-függvényt (PDF) mutatja be egy folytonos haranggörbeként (pl. normál eloszlás).

---

## A Várható érték (Expectation / Mean)

A várható érték (jelölése $\large E[X]$ vagy $\large \mu$, ejtsd: mű) azt mutatja meg, hogy hosszú távon, rengeteg kísérlet után átlagosan milyen értékre számíthatunk.

**Diszkrét esetben (Képlet 1):**

```math
\large
E[X] \overset{\text{def}}{=} \sum_{i=1}^{k} x_i \Pr(X = x_i) = x_1 \Pr(X = x_1) + x_2 \Pr(X = x_2) + \dots + x_k \Pr(X = x_k)
```

**Magyarázat:** Vesszük az összes lehetséges értéket ($\large x_i$), megszorozzuk azzal a valószínűséggel, amilyen gyakran előfordul ($\large \Pr(X = x_i)$), majd ezeket a szorzatokat egyszerűen összeadjuk.

**Folytonos esetben (Képlet 2):**

```math
\large
E[X] \overset{\text{def}}{=} \int_{\mathbb{R}} x f_X(x) \, dx
```

**Magyarázat:** A szumma ($\large \Sigma$) folytonos megfelelője az integrál ($\large \int$). Az integrál a görbe alatti területet számolja ki a teljes valós számhalmazon ($\large \mathbb{R}$). Ugyanaz a logika, mint diszkrétnél: az adott $\large x$ értéket "megszorozzuk" a sűrűségfüggvény ottani értékével, és "összegezzük" (integráljuk). (A görbe alatti terület szabálya miatt $\large \int_{\mathbb{R}} f_X(x) \, dx = 1$).

---

## A Szórás (Standard Deviation)

A szórás (jelölése $\large \sigma$, ejtsd: szigma) azt méri, hogy az értékek átlagosan mennyire térnek el a várható értéktől ($\large \mu$). Diszkrét esetben a képlete:

```math
\large
\sigma \overset{\text{def}}{=} \sqrt{E\left[(X - \mu)^2\right]} = \sqrt{\Pr(X = x_1)(x_1 - \mu)^2 + \dots + \Pr(X = x_k)(x_k - \mu)^2}
```

**Magyarázat lépésről lépésre:**

1. Minden értéket kivonunk az átlagból ($\large x_i - \mu$). Ez az eltérés.
2. Ezt négyzetre emeljük, hogy a negatív eltérések (átlag alattiak) ne ejtsék ki a pozitívokat.
3. Ezeket a négyzetes eltéréseket súlyozzuk a valószínűségükkel és összeadjuk (ez a várható érték, más néven **Variancia**).
4. Végül gyököt vonunk az egészből ($\large \sqrt{\ }$), hogy visszakapjuk az eredeti mértékegységet.

---

# 2.3 Torzítatlan becslések (Unbiased Estimators)

A valóságban és a gépi tanulásban sosem ismerjük a valódi valószínűség-eloszlást ($\large f_X$). Csak egy adathalmazunk van, azaz megfigyeltünk néhány konkrét értéket. Ezt nevezzük **mintának** vagy **adathalmaznak** (Sample / Dataset), jelölése: $\large S_X = \{x_i\}_{i=1}^{N}$.

Mivel a valódi "isteni" paramétereket (pl. a valódi átlagot, $\large \theta$) nem ismerjük, a mintánkból kell becslést adnunk rá. A mintából számolt becslést $\large \hat{\theta}(S_X)$-nek jelöljük (a kis "kalap" $\large \hat{}$ mindig azt jelzi a statisztikában, hogy ez egy becsült érték).

**Mikor mondjuk, hogy egy becslés "torzítatlan" (unbiased)?**

Ha a becslésünk várható értéke hajszálpontosan megegyezik a valódi, ismeretlen paraméterrel:

```math
\large
E\left[\hat{\theta}(S_X)\right] = \theta
```

**Intuitív magyarázat:** Képzeld el, hogy végtelen sokszor gyűjtesz 100 elemű adathalmazt, és mindegyikből kiszámolsz egy becsült átlagot ($\large \hat{\mu}$). Ha ezeket a becsült átlagokat átlagolod, és az eredmény hajszálpontosan megegyezik a valódi sokaság átlagával, akkor a képleted torzítatlan.

**A Mintaátlag (Sample Mean):**

A leggyakoribb torzítatlan becslés a várható értékre ($\large E[X]$) a klasszikus mintaátlag, amit úgy kapunk meg, hogy összeadjuk az adatpontokat és elosztjuk a számukkal ($\large N$):

```math
\large
\hat{\mu} = \frac{1}{N} \sum_{i=1}^{N} x_i
```

---

# 2.4 Bayes-tétel (Bayes' Rule)

A Bayes-tétel a valószínűségszámítás egyik legfontosabb képlete, amely megmondja, hogyan frissítsük a tudásunkat (valószínűségeinket) új bizonyítékok fényében.

**Feltételes valószínűség (Conditional probability):**

A $\large \Pr(X = x \mid Y = y)$ azt jelenti: "Mekkora a valószínűsége annak, hogy $\large X$ értéke $\large x$, feltéve, hogy tudjuk, hogy az $\large Y$ értéke $\large y$?" (A függőleges vonal $\large |$ jelenti a "feltéve hogy" kapcsolatot).

**A Bayes-tétel képlete:**

```math
\large
\Pr(X = x \mid Y = y) = \frac{\Pr(Y = y \mid X = x) \cdot \Pr(X = x)}{\Pr(Y = y)}
```

**Magyarázat az ML kontextusában (pl. Spamszűrés):**

Tegyük fel, az $\large X$ az, hogy a levél spam-e, az $\large Y$ pedig az, hogy szerepel-e benne az "ingyen" szó.

- Téged az érdekel: Mekkora az esélye, hogy Spam, ha benne van az "ingyen" szó? $\large \rightarrow \Pr(\text{Spam} \mid \text{"ingyen"})$
- Azt viszont a múltbéli adatbázisodból tudod, hogy a spamek hány százaléka tartalmazza az "ingyen" szót $\large \rightarrow \Pr(\text{"ingyen"} \mid \text{Spam})$.

A Bayes-tétel lehetővé teszi, hogy "megfordítsd" az ok-okozati irányt, és a történelmi adatokból kikövetkeztesd a jövőbeli levél besorolását!

# 2.5 Paraméterbecslés (Parameter Estimation)

Amikor van egy statisztikai modellünk, amely leírja a valószínűségi változónk ($\large X$) eloszlását, ez a modell általában paraméterektől függ. Ezeket a paramétereket egy $\large \theta$ (théta) vektorba gyűjtjük.

## Példa: A Gauss-eloszlás (Normál eloszlás)

A legismertebb eloszlás a haranggörbe, amelynek pontosan két paramétere van: a várható érték ($\large \mu$) és a szórás ($\large \sigma$). Így a paramétervektorunk: $\large \theta = [\mu, \sigma]$.

A sűrűségfüggvény képlete:

```math
\large
f_\theta(x) = \frac{1}{\sqrt{2\pi\sigma^2}} \, e^{-\frac{(x-\mu)^2}{2\sigma^2}}
```

(Mivel ez egy érvényes sűrűségfüggvény, használhatjuk az $\large X$ ismeretlen eloszlásának modellezésére.)

## A paraméterek frissítése a Bayes-tétellel

Ha érkezik egy új adat ($\large x$), a Bayes-tétel segítségével frissíteni tudjuk a modellünk $\large \theta$ paramétereivel kapcsolatos tudásunkat (hiedelmünket):

```math
\large
\Pr(\theta = \hat{\theta} \mid X = x) \leftarrow \frac{\Pr(X = x \mid \theta = \hat{\theta}) \, \Pr(\theta = \hat{\theta})}{\sum_{\tilde{\theta}} \Pr(X = x \mid \theta = \tilde{\theta})}
```

**Prior (Előzetes hiedelem – $\large \Pr(\theta = \hat{\theta})$):** Mielőtt látnánk az adatot, mennyire tartjuk valószínűnek ezt a paramétertéket. Kezdetben ezt megtippelhetjük.

**Iteratív frissítés:** Ha van egy $\large S$ adathalmazunk, az adatpontokon (példákon) egyesével végigmegyünk. Minden lépésben kiszámoljuk az új valószínűséget, és a következő lépésnél már ezt az új becslést használjuk priorként.

## Maximum Likelihood Becslés (MLE – Maximum Likelihood Estimation)

Mi a "legjobb" $\large \theta^{*}$ paraméterérték az adatok alapján? Az, amelyik maximalizálja az adatok megfigyelésének valószínűségét:

```math
\large
\theta^{*} = \arg\max_{\theta} \prod_{i=1}^{N} \Pr(\theta = \hat{\theta} \mid X = x_i)
```

**A programozói valóság (Log-likelihood trükk):** A fenti képletben a $\large \prod$ azt jelenti, hogy rengeteg nagyon kicsi valószínűséget (pl. $\large 0.001$) kell összeszoroznunk. Egy számítógép memóriájában ez gyorsan "alulcsordulást" (numerical overflow/underflow) okoz, és kerekítési hibából $\large 0$ lesz. Emiatt a gyakorlatban mindig a kifejezés természetes logaritmusát ($\large \ln$) optimalizáljuk. A logaritmus azonossága miatt a szorzásból összeadás lesz, amivel a gépek már tökéletesen tudnak dolgozni!

👉 [Kattints ide az interaktív HTML widget megtekintéséhez](https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/02%20Fejezet/gaus.html<img width="783" height="391" alt="images" src="https://github.com/user-attachments/assets/b70db4cc-b29e-4125-b86c-c9b63bd907a8" />
)
---

# 2.6 Osztályozás vs. Regresszió (Classification vs. Regression)

A felügyelt tanulás (Supervised Learning) két nagy fő kategóriára bomlik, attól függően, hogy milyen kimenetet várunk.

## Osztályozás (Classification)

- A cél egy **kategória (címke)** hozzárendelése a bemenethez (pl. spam szűrés).
- A kimenet egy **véges halmazból** kerül ki.
- **Bináris osztályozás:** Két lehetséges kimenet van ("beteg" / "egészséges", "spam" / "nem_spam").
- **Többosztályos (Multiclass) osztályozás:** Három vagy több lehetséges kategória.

## Regresszió (Regression)

- A cél egy **folytonos valós szám** (célérték / target) megjóslása.
- **Példa:** Ház árának becslése az alapterület, szobák száma és az elhelyezkedés alapján. A kimenet nem egy kategória, hanem mondjuk $\large 45.5$ millió Forint.

---

# 2.7 Modell-alapú vs. Példány-alapú tanulás

## Modell-alapú tanulás (Model-Based Learning)

Az algoritmus a tanító adatok alapján matematikai paramétereket tanul meg (mint a $\large w^{*}$ és $\large b^{*}$ az SVM-nél). Miután a modellt betanították és a paraméterek beálltak, az eredeti tanító adathalmazt el lehet dobni. A legtöbb algoritmus ilyen.

## Példány-alapú tanulás (Instance-Based Learning)

Itt maga az egész adathalmaz a modell. Az algoritmus "megjegyzi" az összes adatot.

**Példa – kNN (k-Nearest Neighbors):** Új adat osztályozásakor az algoritmus megnézi az új pont környezetét a térben, és a $\large k$ darab legközelebbi szomszédja alapján hoz döntést (amelyik osztályból több van a szomszédok között, az nyer).

<img width="783" height="391" alt="images" src="https://github.com/user-attachments/assets/cac2dcd8-a8a3-4466-a5f0-b0108a62be62" />


---

# 2.8 Sekély (Shallow) vs. Mély (Deep) tanulás

## Sekély tanulás (Shallow Learning)

Az algoritmus a modell paramétereit közvetlenül a bemeneti jellemzőkből (raw features) tanulja meg. Az eddig említett összes algoritmus (SVM, kNN) ide tartozik.

## Mély tanulás (Deep Learning)

A több rejtett réteggel rendelkező (Deep) neurális hálózatok tartoznak ide. Itt a modell a paramétereit nem közvetlenül a nyers adatokból, hanem a hálózat korábbi rétegeinek kimenetéből tanulja meg, így a gép képes egyre magasabb szintű, absztrakt reprezentációkat építeni az adatokból.
