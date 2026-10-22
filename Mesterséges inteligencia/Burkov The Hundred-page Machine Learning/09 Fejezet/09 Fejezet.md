# 9. Felügyelet Nélküli Tanulás (Unsupervised Learning)

**A Probléma:**
Eddig minden modellünk úgy tanult, hogy megadtuk neki a bemenetet ($x$) és a "helyes választ" (a címkét, $y$). De a való életben a legtöbb adatnak nincs címkéje.

Gondolj egy e-kereskedelmi oldal látogatóinak kattintási adataira. Nincs odaírva, hogy a felhasználó "Vásárló" vagy "Nézelődő". A gépnek pusztán a nyers adatok ($x$) alapján kell felfedeznie valamilyen rejtett struktúrát, mintázatot, vagy csoportosulást.

Mivel nincsenek címkék (nincs "helyes megoldás"), nagyon nehéz mérni, hogy a modellünk jó munkát végzett-e. Ebben a fejezetben csak olyan módszereket nézünk meg, amelyek emberi szubjektivitás nélkül, pusztán matematikailag értékelhetők.

## 9.1 Sűrűségbecslés (Density Estimation)

A sűrűségbecslés célja megérteni, hogy az adataink milyen valószínűségi eloszlásból (Probability Density Function - PDF) származnak.

Ha tudjuk az eloszlást, azonnal látjuk, mi számít "normális" (gyakori) adatnak, és mi számít kiugró, gyanús adatnak (újdonság vagy anomália detektálás).

A 7. fejezetben már csináltunk ilyet: ott feltételeztük, hogy az adatok egy szép haranggörbét (Gauss-eloszlást) követnek. De mi van, ha a valódi adatok egyáltalán nem harang alakúak, hanem mondjuk két púpjuk van, vagy teljesen aszimmetrikusak? Ilyenkor a Gauss-feltételezés csúnyán elbukik.

### A Megoldás: Kernel Sűrűségbecslés (Kernel Density Estimation - KDE)

Ahelyett, hogy rákényszerítenénk egy merev formát az adatokra, hagyjuk, hogy maguk az adatok rajzolják ki a görbét! Ez egy **nem-parametrikus** módszer.

**Hogyan működik a KDE matematikai szinten?**

Képzelj el egy egydimenziós tengelyt, amin pontokként be vannak jelölve az adataid ($x_1, x_2, \dots, x_N$).
Minden egyes adatpontra ráültetünk egy pici, vékony haranggörbét (ezt hívják Kernelnek, $k$). Majd az összes kis haranggörbét egyszerűen összeadjuk (átlagoljuk)! Ahol sok adatpont van sűrűn egymás mellett, ott a kis harangok összeadódnak, és egy nagy hegyet (nagy sűrűséget) alkotnak.

A becsült eloszlás függvénye ($\hat{f}_b$):

```math
\hat{f}_b(x) = \frac{1}{N b} \sum_{i=1}^{N} k \left( \frac{x - x_i}{b} \right)
```

A Kernel ($k$) itt is általában a Gauss-függvény:

```math
k(z) = \frac{1}{\sqrt{2\pi}} \exp\left( -\frac{z^2}{2} \right)
```

### A Sávszélesség ($b$ - Bandwidth) szerepe

Akárcsak a Kernel Regressziónál (7. fejezet), a $b$ hiperparaméter a legfontosabb beállítás. Ez szabályozza, hogy az adatpontokra ültetett kis haranggörbék milyen "szélesek" legyenek. Ez az a pont, ahol bejön a Torzítás-Variancia (Bias-Variance) tradeoff!

- **Túl kicsi $b$ (Túltanulás / Overfitting):** A kis harangok nagyon vékonyak, tűszerűek lesznek. A végső görbe eszeveszetten fog ugrálni (tüskés lesz), minden egyes adatpontnál csinál egy csúcsot. A modell "bemagolja" az adatokat ahelyett, hogy az általános trendet mutatná.
- **Túl nagy $b$ (Alulillesztés / Underfitting):** A kis harangok hatalmasra és laposra híznak. Szinte teljesen összemossák az adatokat, a végső görbe egy unalmas, lapos domb lesz. Elveszítjük a finom, fontos mintázatokat.
- **A tökéletes $b$ (Jó illeszkedés):** A görbe szépen, simán hullámzik, kiemelve a sűrűbb régiókat.

<img width="627" height="532" alt="1" src="https://github.com/user-attachments/assets/f8348ad4-0b5a-4e61-b26a-f87dce6f4f88" />

>  Ez 4 kis panelt mutat:
> - **(a)** A tökéletes eloszlásgörbe.
> - **(b)** A tüskés, túltanult görbe (túl kicsi $b$).
> - **(c)** A lapos, alulillesztett görbe (túl nagy $b$).
> - **(d)** A Grid Search "U-alakú" hibagörbéje, ami megmutatja, hol a legkisebb a hiba a $b$ változtatásakor.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/09%20Fejezet/KernelDensityEstimation(KDE).html)

### Hogyan találjuk meg a legjobb $b$-t matematikailag? (MISE és Keresztvalidáció)

Mivel nincsenek "helyes" címkéink, nem tudjuk egyszerűen kiszámolni a megszokott MSE-t (Átlagos Négyzetes Hiba). Ehelyett az integrálszámításhoz kell nyúlnunk!

Keressük azt a $b$ értéket, amelyik minimalizálja az integrált négyzetes hibát a (számunkra láthatatlan) igazi eloszlás ($f$) és a mi modellünk ($\hat{f}_b$) között. Ezt hívják **MISE**-nek (Mean Integrated Squared Error):

```math
\text{MISE}(b) = \mathbb{E} \left[ \int_{\mathbb{R}} (\hat{f}_b(x) - f(x))^2 \,dx \right]
```

**Várj, de hát nem is ismerjük az igazi $f(x)$-et! Hogyan számoljunk vele?**

Itt jön a statisztikai varázslat. A fenti integrált fel lehet bontani három tagra. Ebből a harmadik tag független a $b$-től (tehát kidobható az optimalizálásnál).

A legtrükkösebb a középső tag, ami valójában a $\hat{f}_b$ függvény várható értéke. Ezt a tagot a statisztikusok egy ügyes **Hagyj-ki-egyet Keresztvalidációval** (Leave-One-Out Cross Validation - LOOCV) közelítik meg:

```math
- \frac{2}{N} \sum_{i=1}^{N} \hat{f}_{b}^{(i)}(x_i)
```

Ahol az $\hat{f}_{b}^{(i)}$ azt jelenti, hogy a sűrűségbecslést kiszámoljuk az $i$-edik ponton úgy, hogy azt az egy konkrét adatpontot ideiglenesen kivettük az adathalmazból.

Összerakva a számolható részeket, a végleges, minimalizálandó költségfüggvényünk:

```math
\int_{\mathbb{R}} \hat{f}_b^2(x) \,dx - \frac{2}{N} \sum_{i=1}^{N} \hat{f}_{b}^{(i)}(x_i)
```

Erre a formulára már nyugodtan rá lehet engedni egy számítógépes **Rácskeresést (Grid Search)**. Kipróbálunk rengeteg $b$ értéket, és amelyik a legkisebb értéket adja erre az egyenletre (ez látszik az 1/d ábra U-alakjának az alján), az lesz a mi tökéletes sávszélességünk!

## 9.2 Klaszterezés (Clustering)

A klaszterezés célja, hogy a címkézetlen adatokat csoportokba (klaszterekbe) rendezze. Mivel nincsenek címkéink, a gép csak arra hagyatkozhat, hogy az adatok mennyire vannak "közel" egymáshoz a térben. Ha két adatpont közel van egymáshoz, valószínűleg hasonlók (egy csoportba tartoznak).

A legnagyobb kihívás itt az, hogy **nincs tökéletes algoritmus**. Mivel nem tudjuk, milyen a valódi eloszlás, nem tudjuk egyértelműen megmondani, hogy egy klaszterezés "jó" vagy "rossz". Szubjektív döntéseket kell hoznunk.

### 9.2.1 K-Közép (K-Means)

Ez a világ leghíresebb és leggyakrabban használt klaszterező algoritmusa. Gyors, egyszerű, de megvannak a maga korlátai.

**Hogyan működik lépésről lépésre?**

1. **A $k$ hiperparaméter:** Neked kell kitalálnod és megadnod a gépnek, hogy hány csoportot (klasztert) keressen. Legyen mondjuk $k=3$.
2. **Inicializálás:** A gép teljesen véletlenszerűen letesz $3$ darab "középpontot" (Centroidot) a térbe.
3. **Hozzárendelés (Assignment):** A gép megméri minden egyes adatpont távolságát (pl. Euklideszi távolság) ettől a $3$ Centroidtól. Minden adatpontot besorol ahhoz a Centroidhoz, amelyikhez a legközelebb van. (Ezzel létrejön $3$ kezdetleges csoport).
4. **Középpont frissítése (Update):** Most, hogy megvannak a csoportok, a gép kiszámolja az adott csoporthoz tartozó pontok átlagát (súlypontját). A Centroidokat áthelyezi ebbe az új súlypontba.
5. **Ismétlés:** Mivel a Centroidok elmozdultak, a távolságok megváltoztak. Újra besoroljuk a pontokat a hozzájuk legközelebbi (új) Centroidhoz. Ezt addig ismételjük, amíg a Centroidok már egyáltalán nem mozognak (konvergálnak).

<img width="638" height="543" alt="2" src="https://github.com/user-attachments/assets/b325fdbe-f209-495b-b708-4125a0e0bbcd" />


> Ez 4 kis panelt mutat:
> - **(a)** Eredeti, szürke adatok.
> - **(b)** 1. iteráció: Letettünk 3 véletlen négyzetet (Centroidot), a tér 3 színre bomlik.
> - **(c)** 3. iteráció: A négyzetek elkezdenek vándorolni az adatcsoportok közepe felé.
> - **(d)** 5. iteráció: A Centroidok beálltak a sűrű csomópontok közepére, kialakult a 3 tökéletes klaszter.

**A K-Means két nagy gyengesége:**

- **Neked kell tudnod a $k$-t:** Ha a valóságban 4 csoport van, de te $k=3$-at adsz meg, a gép kénytelen lesz két csoportot egybemosni. Ezt nagyon nehéz előre megtippelni sokdimenziós adatoknál.
- **Gömb alakú klaszterek:** A távolságmérés miatt a K-Means mindig "gömb" (vagy kör) alakú csoportokat keres. Ha az adataid mondjuk kifli formájúak, vagy egymásba fonódó gyűrűk, a K-Means borzalmas munkát fog végezni, és egyszerűen kettévágja őket középen.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/09%20Fejezet/K-MeansAlgorithmStep-by-Step.html)

---

### 9.2.2 DBSCAN és HDBSCAN (Sűrűség alapú klaszterezés)

Mit tegyünk, ha nem tudjuk a $k$-t, és az adataink nem gömb alakúak? Használjunk **Sűrűség-alapú (Density-based)** algoritmusokat!

A DBSCAN nem a középpontokból indul ki, hanem úgy működik, mint egy tintafolt, ami szétterjed a papíron: ott folyik tovább, ahol sűrűn vannak a pontok.

**Hogyan működik a DBSCAN?**

Nem $k$-t kér, hanem két másik hiperparamétert:

- **$\epsilon$ (Epszilon):** Mekkora sugarú körben keressünk szomszédokat?
- **$n$ (Min_samples):** Hány adatpontnak kell lennie ebben az $\epsilon$ sugarú körben ahhoz, hogy a pontot "sűrűnek" (magpontnak / core pointnak) nevezzük?

**A folyamat:**

1. Rábökünk egy véletlen adatra. Megnézzük az $\epsilon$ sugarú környezetét.
2. Ha van benne legalább $n$ darab szomszéd, kinevezzük az 1-es Klaszter kezdőpontjának.
3. Ezután megvizsgáljuk a szomszédokat is. Ha az ő $\epsilon$ sugarukban is van legalább $n$ pont, akkor ők is hozzákapcsolják a szomszédaikat az 1-es Klaszterhez. A klaszter így "terjeszkedik", amíg el nem ér egy ritka, üres területhez.
4. Ha egy pontnak nincsenek szomszédai, a gép **Kivételnek (Outlier / Zajnak)** bélyegzi. (A K-Means erre képtelen, ő mindent beterel valahova).

**A DBSCAN előnye:**
Bármilyen amorf, furcsa alakzatot (pl. S-alak, egymást körülölelő félholdak) képes egyetlen klaszterként azonosítani, amíg a pontok sűrűsége folyamatos. Valamint magától megtalálja, hány klaszter van, és kiszűri a zajt.

**A DBSCAN hátránya:**
Pokolian nehéz jól beállítani az $\epsilon$-t. Ha az adathalmazban vannak nagyon sűrű és kevésbé sűrű klaszterek is (varying density), egyetlen fix $\epsilon$ értékkel lehetetlen mindet megtalálni.

### A modern megváltó: HDBSCAN

A **HDBSCAN** a DBSCAN továbbfejlesztett, modern verziója. Zsenialitása abban rejlik, hogy kidobja az $\epsilon$ hiperparamétert!

A gép kiszámolja az összes lehetséges sűrűséget és távolságot, és magától felépít egy hierarchiát.

Neked mindössze egyetlen, nagyon intuitív paramétert kell megadnod: **$n$ (Min_cluster_size)** - azaz *"Szerinted minimum hány pontból álljon egy csoport, hogy azt érdemes legyen klaszternek hívni?"* (Pl. 5 vagy 10).

Képes kezelni a változó sűrűségű csoportokat is, és rendkívül gyors millió soros adatokon is.

> *(A könyv szerzőjének jótanácsa: A való életben mindig a HDBSCAN-nel kezdd a klaszterezést. Ha az nem működik, csak akkor nyúlj a K-Means-hez!)*

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/09%20Fejezet/ClusteringGeometryK-MeansvsDBSCAN.html)

 ### 9.2.3 A klaszterek számának meghatározása (Hány $k$ legyen?)

Ha a HDBSCAN helyett K-Means-t (vagy GMM-et) használsz, a legnagyobb fejfájásod a $k$ (a csoportok számának) kitalálása lesz. Dimenziók hiányában (pl. 2D vagy 3D) egyszerűen ránézel a pontfelhőre. De mi van, ha 50 dimenziós adataid vannak? Az emberi agy itt feladja.

Erre a problémára a könyv a **Prediktív Erő (Prediction Strength)** módszerét javasolja. Ez tulajdonképpen a felügyelt tanulásból ismert Keresztvalidáció logikájának átemelése a felügyelet nélküli tanulásba.

**A Prediktív Erő módszer logikája:**

1. **Vágd ketté az adatokat:** Oszd fel az adataidat egy Tanító (Training) és egy Teszt (Test) halmazra.
2. **Tippelj egy $k$-t:** Mondjuk legyen $k=4$.
3. **Két külön klaszterezés:** Futtasd le a K-Means-t a Tanító adatokon (megkapod a Tanító Centroidokat), és futtasd le teljesen külön a Teszt adatokon is (megkapod a Teszt Centroidokat).
4. **A Teszt (A Kereszt-ellenőrzés):** Most jön a trükk! Fogd a Teszt adataidat, és vizsgáld meg őket a Tanító Centroidok "szemüvegén" keresztül. Nézd meg:
   - Ha két Teszt adatpont a saját (Teszt) klaszterezése szerint ugyanabba a csoportba került...
   - ...akkor a Tanító klaszterezés (a Tanító Centroidok vonalai) szerint is ugyanabba a csoportba esnek?

**Mit jelent ez?**
Ha a $k=4$ a "valódi", természetes csoportok száma, akkor a Tanító adatokból rajzolt 4 régió szinte hajszálpontosan meg fog egyezni a Teszt adatokból rajzolt 4 régióval. A prediktív erő közel lesz az $1.0$-hoz ($100\%$).

Ha viszont hülyeséget tippeltél (pl. valójában 2 csoport van, de te $k=4$-et kértél), az algoritmus a két nagy csoportot véletlenszerűen, mesterségesen fogja kettévágni. A Tanító adatokon a vágás máshová esik, mint a Teszt adatokon. Amikor egymásra vetíted őket, teljes lesz a káosz, a prediktív erő drasztikusan lecsökken.

<img width="640" height="416" alt="3" src="https://github.com/user-attachments/assets/8591f370-a353-48aa-a603-b95407574f87" />


> Ami ezt a folyamatot mutatja:
> - **(a)** A Tanító adatokon felépült 4 klaszter régió.
> - **(b)** A Teszt adatok saját 4 klaszter régiója.
> - **(c)** Amikor a (b)-t rávetítik az (a)-ra, látszik, hogy a pontok teljesen rossz helyre (más színű régiókba) esnek. Ez bizonyítja, hogy a $k=4$ egy rossz választás volt.

A kutatások szerint azt a legmagasabb $k$ értéket érdemes választani, ahol a Prediktív Erő $ps(k)$ még $0.8$ ($80\%$) felett marad.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/09%20Fejezet/PredictionStrengthMetric.html)

---

### 9.2.4 Egyéb Klaszterező Algoritmusok: A GMM és az EM algoritmus

A K-Means úgynevezett "Kemény" (Hard) klaszterezést végez. Egy adatpont vagy az 1-es csoportba tartozik, vagy a 2-esbe. Nincs átmenet. Ráadásul csak gömb alakú csoportokat tud keresni.

Ennek a modern, "Puha" (Soft) alternatívája a **Gauss-keverék Modell (Gaussian Mixture Model - GMM)**.

A GMM nem egyszerűen középpontokat keres. Azt feltételezi, hogy az adataidat több különböző (mondjuk 2 darab) haranggörbe (Gauss-eloszlás) hozta létre. A célja, hogy megtalálja ennek a két haranggörbének a:

- **Középpontját ($\mu$)**
- **Kiterjedését / Alakját ($\sigma^2$ vagy $\Sigma$)** - Így a csoportok lehetnek hosszúkás elipszisek is, nem csak gömbök!
- **Súlyát ($\pi$)** - Melyik haranggörbe "erősebb", azaz melyikből származik több adat.

### Az Elvárás-Maximalizálás (EM - Expectation-Maximization) Algoritmus

Hogyan tanulja meg a gép ezeket a haranggörbéket, ha nincsenek címkék? Egy zseniális, iteratív ping-pong játékkal, amit **EM algoritmusnak** hívnak.
(Ez a K-Means folyamatának a matematikai, valószínűségi verziója).

Tegyük fel, hogy $k=2$ csoportot keresünk egydimenziós (vonalon lévő) adatokon.
A gép először teljesen véletlenszerűen letesz két haranggörbét ($\mu_1, \mu_2$), és azt mondja, mindkettőnek $50-50\%$ az esélye ($\pi_1 = \pi_2 = 0.5$).

Ezután indul az iteráció:

**1. E-Lépés (Elvárás / Expectation) - A "Puha" Hozzárendelés:**
A gép végigmegy minden egyes $x_i$ adatponton, és a Bayes-tétel segítségével kiszámolja egy $b_i^{(j)}$ valószínűséget: *"Mekkora az esélye, hogy ez az adat az 1-es haranggörbéből jött, és mekkora, hogy a 2-esből?"*

**Eredmény:** Egy adat nem szigorúan az 1-esé lesz, hanem mondjuk $80\%$-ban az 1-esé, és $20\%$-ban a 2-esé.

**2. M-Lépés (Maximalizálás / Maximization) - A Harangok Frissítése:**
Most, hogy minden adatról tudjuk a százalékokat, "áthelyezzük" és "átformáljuk" a haranggörbéket.
A K-Means-ben sima átlagot számoltunk. Itt **Súlyozott Átlagot** számolunk!

Amikor az 1-es haranggörbe új középpontját ($\mu_1$) keressük, az a pont, ami $80\%$-ban az övé, nagy erővel húzza magához a harangot, a $20\%$-os pont alig szól bele.

A képlet az új középpontra ($\mu_j$):

```math
\mu_j = \frac{\sum_{i=1}^{N} b_i^{(j)} x_i}{\sum_{i=1}^{N} b_i^{(j)}}
```

**A ping-pong folytatódik:**
A haranggörbék elmozdultak. Visszaugrunk az E-Lépésre, újra kiszámoljuk a százalékokat az új pozíciók alapján, majd újra az M-Lépésre... Addig ismételjük a lépéseket, amíg a haranggörbék a helyükre nem csúsznak és meg nem állnak.

<img width="651" height="552" alt="5" src="https://github.com/user-attachments/assets/dc52d31c-19c3-485e-b814-1361dc956e89" />


> Az EM algoritmus folyamatát mutatja 4 panelen (Iteration 1, 2, 10, 40). A képeken látszik, ahogy a kezdetben rossz helyen lévő, hatalmas zöld és kék haranggörbék (ellipszisek) lépésről lépésre összezsugorodnak és ráhúzódnak a két sűrű adatcsomópontra.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/09%20Fejezet/GaussianMixtureModel(EM%20Algorithm).html)

 ## 9.3 Dimenziócsökkentés (Dimensionality Reduction)

A modern mélytanulási algoritmusok (pl. neurális hálózatok) röhögve megeszik a több millió dimenziós (több millió jellemzőből álló) adatokat is. Akkor miért akarjuk csökkenteni a dimenziókat?

**Két fő oka van:**

1. **Értelmezhetőség (Interpretability):** Ha kigyomláljuk a felesleges, egymással korreláló vagy zajos adatokat, egyszerűbb algoritmusokat (pl. Döntési fákat) is használhatunk, amelyeknek a logikáját el tudjuk magyarázni a főnöknek.
2. **Adatvizualizáció (Data Visualization):** Az emberi agy és a monitorunk maximum 3 dimenziót (3D teret) képes befogadni. Ha a betegeidnek 50 paramétere van (50 dimenzió), lehetetlen őket egy grafikonra rajzolni, hogy megnézzük, hol vannak a beteg és az egészséges "csomópontok".

A három leggyakrabban használt technika: a **PCA**, a **UMAP** és az **Autoenkóderek** (amikről a 7. fejezetben már beszéltünk: a hálózat középső, vékony "bottleneck" rétege maga a csökkentett dimenziójú reprezentáció).

### 9.3.1 Főkomponens-elemzés (PCA - Principal Component Analysis)

Ez a legrégebbi és legklasszikusabb módszer. A lényege: keressük meg az adatokban azt az irányt, amiben a legnagyobb a szórás (variancia), mert a szórás jelenti az információt!
(Ha egy tulajdonság mindenkinél ugyanaz – nulla a szórása –, az nem hordoz információt).

**A PCA logikája (Emberi nyelven):**
Képzeld el, hogy a levegőben lebeg egy csapatnyi méhecske, amelyek egy hosszúkás szivar alakzatot (ellipszoidot) vesznek fel a 3D térben.

1. A PCA megnézi ezt a szivart, és húz egy egyenest (tengelyt) pontosan a szivar hossztengelye mentén (ebben az irányban vannak a legmesszebb egymástól a méhek, itt a legnagyobb a szórás). Ez az **1. Főkomponens (PC1)**.
2. Ezután húz egy második tengelyt, ami szigorúan derékszöget zár be az elsővel (ortogonális), és a szivar "vastagságát" méri. Ez a **2. Főkomponens (PC2)**.
3. Végül "rálapítja" (kivetíti / projektálja) az összes méhet erre az új 2D-s síkra, amit a PC1 és PC2 alkot. A 3D-s szivarból egy 2D-s palacsinta lett, de a formája és a méhek egymáshoz viszonyított távolsága a lehető legjobban megmaradt!

<img width="627" height="237" alt="6" src="https://github.com/user-attachments/assets/6ea61a4b-c526-4b55-8de2-8b8b4e73d143" />


> Az ábrán látható:
> - **(a)** Az eredeti 2D-s, átlósan elnyúló adatfelhő.
> - **(b)** A két egymásra merőleges Főkomponens-vektor (nyíl) berajzolva a felhőbe. A hosszabb nyíl mutat az elnyúlás irányába.
> - **(c)** Az adatok "rálapítva" (leprojektálva) az 1. Főkomponensre, így 2 dimenzióból 1 dimenzió (egy egyenes) lett.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/09%20Fejezet/PCADimensionalityReduction.html)

### 9.3.2 UMAP (Uniform Manifold Approximation and Projection)

A PCA gyönyörű matematikai trükk, de van egy óriási hibája: csak lineáris összefüggéseket (egyenes tengelyeket) tud keresni. Ha az adataid egy összetekeredett svájci roládra hasonlítanak, a PCA egyszerűen rálapítja a rétegeket egymásra, és mindent összekuszál.

Erre a problémára találták ki a modern, vizualizációra optimalizált algoritmusokat, mint a t-SNE és az új király, az **UMAP**. A UMAP nem egyenes tengelyeket keres, hanem a lokális "szomszédságot" próbálja megőrizni egy topológiai gráf (hálózat) segítségével.

**A UMAP mély matematikája (Hogyan épül fel a gráf?)**

Az algoritmus lelke egy Hasonlósági metrika ($w$), amit kiszámol minden pontpár ($x_i$ és $x_j$) között.
A hasonlóság súlya egy exponenciális függvény:

```math
w_i(x_i, x_j) = \exp \left( - \frac{d(x_i, x_j) - \rho_i}{\sigma_i} \right)
```

**Bontsuk fel ezt az egyenletet emberi nyelvre!**

- **$d(x_i, x_j)$:** Ez a sima Euklideszi távolság a két adatpont között.
- **$\rho_i$ (A legközelebbi szomszéd):** A UMAP levonja a legközelebbi szomszéd távolságát! Ez egy zseniális trükk: garantálja, hogy a tér legritkább pontjának is lesz legalább egy $100\%$-os "barátja" (kapcsolata). Ez biztosítja a gráf folytonosságát (local connectivity).
- **$\sigma_i$ (Sűrűség-alapú skálázás):** A nevezőben lévő szigma a pont környezetének sűrűségétől függ (hogy milyen messze van a $k$-adik szomszéd). Ha egy pont nagyon sűrű "városban" van, a távolságok rohamosan nőnek (gyorsan elvágja a kapcsolatokat a távolabbiakkal). Ha "vidéken" van, a kapcsolatok messzebbre nyúlnak. Ezzel a UMAP kisimítja a sűrűségkülönbségeket.

Miután ezt kiszámolta mindkét irányba, szimmetrikussá teszi a kapcsolatot:

```math
w(x_i, x_j) = w_i + w_j - w_i \cdot w_j
```

**A Keresztentrópia-optimalizálás (Az igazi varázslat)**

Oké, felépítettünk egy 50 dimenziós, szuper-bonyolult barátság-hálózatot (gráfot), ahol a vonalak erőssége $w$.
Most szeretnénk ezt levinni 2 Dimenzióba (a képernyőnkre).
A UMAP ledobálja a pontokat ($x'_i$) egy 2D-s síkra teljesen véletlenszerűen.
Kiszámolja a barátság-hálózatot ($w'$) ezen az új, 2D-s síkon is.
Kiszámolja a **Keresztentrópiát** (Cross-Entropy, $C$) az 50 dimenziós $w$ és a 2 dimenziós $w'$ valószínűségi gráfok között:

```math
C(w, w') = \sum \sum \left[ w \ln\left(\frac{w}{w'}\right) + (1 - w) \ln\left(\frac{1 - w}{1 - w'}\right) \right]
```

**Mi a cél?** Hogy a két gráf pontosan ugyanúgy nézzen ki! A Keresztentrópia akkor minimális, ha a 2D-s hasonlóságok ($w'$) tökéletesen megegyeznek az eredeti 50D-s hasonlóságokkal ($w$).

A UMAP ráereszti a **Gradiens Ereszkedést** a 2D-s pontokra ($x'$), és elkezdi tologatni őket a síkon. Addig tologatja a pontokat, amíg az egyenlet a minimumra nem csökken.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/09%20Fejezet/UMAPOptimizationCross-EntropyMinimization.html)

**MNIST Összehasonlítás**

A 7. ábra (Figure 7) tökéletesen mutatja a különbséget a három módszer között az MNIST (kézzel írt számjegyek $28 \times 28 = 784$ dimenzióban) adathalmazon:

- **PCA:** A számjegyek egy nagy, összefolyó pacát alkotnak a központban. A lineáris tengelyek nem tudtak mit kezdeni a bonyolult pixelekkel.
- **Autoenkóder:** Jobb, de még mindig nagyon elmosódottak a határok.
- **UMAP:** A 784 dimenzióból gyönyörű, egymástól teljesen elkülönülő "szigeteket" hozott létre a 2D síkon, amik tökéletesen megfelelnek a 0-9 számjegyeknek (anélkül, hogy a gép ismerte volna a címkéket!).

### Zárószó az Algoritmikus Hatékonyságról (Kódrészletek)

A könyv egy fantasztikus gyakorlati tanáccsal zárul a Python kódoláshoz: **Soha ne írj dupla for ciklust, ha nem muszáj!**

Míg egy dupla ciklus $O(N^2)$ időbe telik (1 millió adatnál 1 billió lépés), a Numpy könyvtár C-ben írt vektorizált műveletei töredéktöredék idő alatt lefutnak.

**Ehelyett (Lassú):**

```python
wx = 0
for i in range(N):
    wx += w[i] * x[i]
```

**Mindig használd ezt (Gyors):**

```python
import numpy
wx = numpy.dot(w, x)
```

És ha halmazokból (Set) keresel elemeket, az időd $O(N)$ helyett $O(1)$ lesz, ami hatalmas (Big Data) adatbázisoknál a hetekig tartó fagyást másodpercekre csökkenti.

## 9.4 Kiugró értékek detektálása (Outlier Detection)

A kiugró értékek (anomáliák) detektálása az a feladat, amikor meg kell találnunk azokat az adatpontokat egy adathalmazban, amelyek drasztikusan eltérnek a "tipikus" (normális) példáktól.

Például egy gyárban a selejtes alkatrészek, vagy egy bankban a lopott bankkártyás tranzakciók megtalálása.

Ahogy a könyv is írja, ezt a problémát már láttuk a 7.3-as fejezetben (**Egyosztályos Osztályozás - One-Class Classification**), ahol a gép egy határvonalat / küszöbértéket húzott a normális adatok köré (pl. a Gauss-haranggörbe segítségével), és ami ezen kívül esett, azt kivételként kezelte.

Azonban a **Felügyelet Nélküli Tanulás** (Unsupervised Learning) eszköztárában van egy sokkal elegánsabb, modern neurális hálózatos megoldás is erre a problémára!

### Az Autoenkóder (Autoencoder) trükk

Emlékezz vissza az Autoenkóderekre (a 7.9-es és 9.3-as fejezetből)!

Az Autoenkóder egy olyan **homokóra alakú** neurális hálózat, ami arra van betanítva, hogy a bemenetét (pl. egy képet) átpréselje egy nagyon szűkös, kevés neuronból álló középső rétegen (a **bottleneck / szűkület**), majd a túloldalon megpróbálja pontosan újraalkotni (rekonstruálni) az eredeti bemenetet.

**Hogyan használjuk ezt Anomália Detektálásra?**

1. **A Tanítás:** Fogjuk a hatalmas adathalmazunkat, ami tele van normális adatokkal (és talán néhány észrevétlen anomáliával), és betanítjuk rajta az Autoenkódert. Mivel az adatok $99.9\%$-a normális, az Autoenkóder "szűkülete" arra fog optimalizálni, hogy a normális dolgok esszenciáját tanulja meg és tömörítse be.
2. **A Tesztelés (Normál adat):** Bedobunk egy új, normális adatot. A gép simán átpréseli a szűkületen, és a túloldalon szépen újraalkotja. Ha megmérjük a különbséget az eredeti bemenet és a gép által újraalkotott kimenet között (ezt hívják **Rekonstrukciós Hibának / Reconstruction Error**), ez a hiba nagyon kicsi lesz.
3. **A Tesztelés (Anomália):** Bedobunk egy selejtes adatot (pl. egy olyan képet, amin a termék karcos). Mivel az Autoenkóder szűkülete életében nem látott még karcot, nem tanulta meg, hogyan kell azt "betömöríteni", majd "kicsomagolni". A kimeneten a gép valószínűleg egy karc nélküli, normális terméket fog rajzolni (mert csak ehhez ért). Ha most megmérjük a különbséget (a karcos bemenet és a tiszta kimenet között), a **Rekonstrukciós Hiba hatalmasra ugrik!**

**A Döntés:**
A mérnöknek mindössze annyit kell tennie, hogy húz egy küszöbértéket a Rekonstrukciós Hibára. Ha a hiba efelett van, a gép automatikusan riaszt:

> *"Ezt az adatot képtelen voltam újraalkotni, tehát ez valami olyasmi, ami nem illik a normál ügymenetbe. Ez egy anomália!"*


👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/09%20Fejezet/AutoencoderReconstructionError.html)
