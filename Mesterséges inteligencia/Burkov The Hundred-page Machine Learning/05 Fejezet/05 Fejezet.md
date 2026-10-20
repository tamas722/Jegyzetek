# 5. Alapvető Gyakorlat (Basic Practice)

Ahogy az előző fejezetből megtudtuk, a gépi tanulás mérnökei nem magukat az algoritmusokat írják meg. Mégsem abból áll a munkájuk, hogy egész nap a `model.fit(x,y)` parancsot gépelik. A munka **80%-a** a nyers adatok átalakítása a modell számára "fogyasztható" formára. Ezt az 5. fejezet részletezi.

## 5.1 Jellemzőmérnökség (Feature Engineering)

Amikor egy főnök odadobja neked egy cég 5 éves nyers szervernaplóját (log fájlokat), és kéri, hogy jósold meg, melyik ügyfél fog lemondani a szolgáltatásról, a gép még nem tud mit kezdeni a logokkal.

Neked kell egy adathalmazt (dataset) építened, méghozzá **Jellemzőmérnökség** (Feature Engineering) segítségével. Ez egy kreatív, sokszor iparági tudást igénylő folyamat, aminek során a nyers adatokból kinyersz minden olyan számszerűsíthető dolgot (jellemzőt / feature-t), ami segíthet a modellnek.

**Példa egy log fájlból kinyert jellemzővektorra:**
`[Előfizetés_ára, Napi_belépések_száma, Átlagos_munkamenet_hossza_mp]`

A "jó" jellemzőket úgy hívjuk, hogy magas prediktív erejük van (high predictive power). Például, ha valaki egy héten át napi 10 órát játszik egy játékkal, annak a "játékidő" jellemzője nagyon erős prediktív erővel bír arra, hogy meg fogja venni a kiegészítőt.

Ha a modellünk a kinyert jellemzők alapján jól tud prediktálni a tanítóadatokon (kevés hibát ejt), akkor azt mondjuk, hogy a modellnek alacsony a torzítása (low bias).

### 5.1.1 One-Hot Kódolás (Kategorikus adatok kezelése)

A matematikai algoritmusok (SVM, Lineáris Regresszió) csak számokat fogadnak el. Mit csinálunk, ha egy oszlopunk szöveges, pl. "Szín"?

**A rossz megoldás (Sorszámozás / Ordinal Encoding):**
Azt mondjuk: Piros = 1, Sárga = 2, Zöld = 3.

**Miért rossz?** Mert a számítógép úgy fogja értelmezni, hogy a Zöld (3) az "több", vagy "háromszor annyi", mint a Piros (1). Olyan matematikai összefüggést (sorrendiséget) keres, ami a színeknél nem létezik, és ezzel teljesen összezavarodik a modell.

> **Kivétel:** Ha a szavak között van logikai sorrend (pl. Rossz=1, Elfogadható=2, Jó=3, Kiváló=4), akkor a sorszámozás tökéletesen működik!

**A jó megoldás (One-Hot Encoding):**
Minden egyes kategóriából (színből) csinálunk egy külön, új oszlopot (dimenziót), ami csak 1-eseket és 0-kat tartalmaz (igaz/hamis).

- Piros = `[1, 0, 0]` (Piros? Igen. Sárga? Nem. Zöld? Nem.)
- Sárga = `[0, 1, 0]`
- Zöld = `[0, 0, 1]`

**Ára:** Megnöveli a dimenziók számát. Ha 1000 féle városnév volt az oszlopban, a One-Hot kódolás hirtelen 1000 új oszlopot ad az adathalmazunkhoz.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/05%20Fejezet/CategoricalEncodingVisualizer.html)

### 5.1.2 Dobozolás / Kategorizálás (Binning / Bucketing)

Ez pontosan a One-Hot kódolás fordítottja: egy folytonos, végtelen számértéket alakítunk át mesterséges kategóriákká.

**Példa:** Életkor (0-100+). Ahelyett, hogy meghagynánk egyetlen valós számként (pl. 23 év), létrehozunk "dobozokat" (bins):

- 0 - 18 év: `bin1`
- 19 - 35 év: `bin2`
- 36+ év: `bin3`

Ha valaki 23 éves, akkor a gépi bemenete `bin2` lesz (amit aztán a fenti One-Hot kódolással `[0, 1, 0]`-ra alakítunk).

**Mire jó ez?**
"Súgunk" az algoritmusnak, hogy bizonyos értékhatárokon belül a pontos szám teljesen lényegtelen. Egy lakáshitelnél mindegy, hogy valaki 42 vagy 43 éves, pontosan ugyanúgy fognak viselkedni. Ezzel megkönnyítjük a gép tanulását, kevesebb adatból is átlátja az összefüggést.

### 5.1.3 Normalizáció (Min-Max Scaling) és 5.1.4 Standardizáció (Z-score)

Emlékezz vissza a Gradiens Ereszkedésre: a gép lépked lefelé a hegyoldalon, az alapján, hogy mekkora az adott jellemző (bemenet).

Ha van egy adathalmazod lakásokról:

- Jellemző: Szobák száma (értéke: 1 - 5)
- Jellemző: Ár (értéke: 10 000 000 - 50 000 000)

A gép összezavarodik: az Ár paraméter deriváltjai milliószor akkorák lesznek, mint a szobaszámé, és teljesen "elnyomják" a modellt (nem beszélve az óriási számok okozta számítógépes túlcsordulásról). Ezt oldja meg a **Jellemzők skálázása** (Feature Rescaling).

#### A Normalizáció (Min-Max scaling)

Minden értéket összepréselünk szigorúan a $[0, 1]$ vagy $[-1, 1]$ intervallumba.

Képlete:

```math
x_{\text{norm}} = \frac{x - \min(x)}{\max(x) - \min(x)}
```

#### A Standardizáció (Z-score)

Nem szorítja szigorú határok közé az adatokat, hanem úgy tolja és nyomja össze őket, hogy a jellemző átlaga $0$, a szórása pedig $1$ legyen. (Olyan lesz, mint egy klasszikus haranggörbe a $0$ körül).

Képlete:

```math
x_{\text{stand}} = \frac{x - \mu}{\sigma}
```

(Ahol $\mu$ az átlag, $\sigma$ a szórás).

#### Melyiket mikor használjuk? (Ökölszabályok):

- **Standardizációt** használj, ha az adataid eleve "haranggörbe" (normál eloszlás) szerint oszlanak el.
- **Standardizációt** használj, ha az adataidban vannak extrém kiugró értékek (outliers). (A normalizáció a $\max(x)$ miatt egyetlen gigantikus kiugró adat miatt a többi millió normál adatot annyira "összenyomná", hogy eltűnnének köztük a különbségek).
- **Normalizációt** használj minden egyéb, "normális" tartományú adatnál.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/05%20Fejezet/ScalingUnderOutliers.html)

### 5.1.5 és 5.1.6 Hiányzó adatok (Missing Features) kezelése és Imputáció

A valóságban az adathalmazok "lyukasak". Valaki elfelejtette beírni a lakás alapterületét, vagy megszakadt a szenzor kapcsolata. Mivel a matematikai képletek nem tudnak az "üres" hellyel (Null / NaN) számolni, kezdenünk kell velük valamit.

**1. Törlés (Drop):** Ha van 10 millió sorunk, és 5 sorban hiányzik valami, gondolkodás nélkül töröljük azt az 5 sort.

**2. Dedikált Algoritmus:** Egyes modernebb algoritmus-implementációk maguktól is tudják kezelni az üres értékeket.

**3. Adat-imputáció (Data Imputation):** Kitöltjük a lyukakat a meglévő adatok alapján! Ennek több szintje van:

- **Átlag (Mean):** Kiszámoljuk a meglévő lakások átlagos alapterületét, és minden lyukba beírjuk ezt.
- **Extrém érték (Out-of-range):** Beírunk egy lehetetlen értéket (pl. életkornál $-1$). A modell (ha elég okos, pl. egy döntési fa) hamar megtanulja, hogy a "$-1$" valójában azt jelenti: "Hiányzó adat", és ez alapján hoz döntést.
- **Középérték (Middle of range):** Ha az adat $[-1, 1]$ közötti, beírunk egy semleges $0$-t. (Ez nem zavarja meg a predikciót).
- **Regresszió (A legokosabb):** Fogjuk a lyukas oszlopot (pl. életkor), kinevezzük Célértéknek (y), a többi adatot bemenetnek (x), és betanítunk egy gépi tanulási modellt, ami megjósolja a páciens életkorát a többi (magasság, súly, betegségek) adata alapján! Ezután a modell becsléseivel töltjük be a lyukakat.

> **Trükk a profiktól:** Ha sok adatod van, az eredeti oszlop mellé hozz létre egy új, bináris (0/1) indikátor oszlopot: `hianyzott_az_eletkor: 1`. Az eredeti lyukat töltsd fel nullával. Így a modell pontosan tudni fogja, hogy ott eredetileg nem volt adat.

> **(Nagyon fontos szabály:** Bármilyen imputációs technikát (pl. átlagolás) használtál a tanítóadatokon, pontosan ugyanazt a technikát és ugyanazt az átlagértéket kell használnod éles környezetben, amikor bejön egy új, hiányos adat a predikció során!)
>
> ## 5.2 Tanuló algoritmus kiválasztása (Learning Algorithm Selection)

A gyakorlatban egy adatkutatónak sosincs ideje az összes létező algoritmust végigpróbálgatni. Ehelyett az alábbi kérdések megválaszolásával szűkíti le a kört néhány esélyes jelöltre:

**1. Magyarázhatóság (Explainability):**
Kinek készül a modell? Ha egy banki hitelbíráló rendszert írsz, a jogszabályok megkövetelik, hogy meg tudd magyarázni, miért utasították el az ügyfelet. Ilyenkor a fekete-dobozos (black box) modellek (Neurális hálózatok, Ensemble módszerek) kiesnek, bármilyen pontosak is. Ehelyett Lineáris regressziót, Döntési fát vagy kNN-t kell használnod, amiknek a döntési logikája emberi ésszel levezethető.

**2. Memóriaigény (In-memory vs. out-of-memory):**
Befér az adathalmazod a géped RAM-jába? Ha igen, szinte bármit használhatsz (pl. SVM, Döntési fa). Ha több száz gigabájtos adatod van, olyan iteratív (incremental) algoritmus kell, ami darabokban (batch) tudja betölteni és tanulni az adatokat (pl. `SGDClassifier`, Neurális hálók).

**3. Adatok és jellemzők száma:**
Néhány algoritmus (pl. Neurális hálók, Gradient Boosting) imádja a milliós nagyságrendű adatokat és a rengeteg dimenziót. Az alap SVM viszont nagyon lassúvá válhat ilyen méreteknél.

**4. Kategorikus vagy Numerikus adatok:**
Ha nem akarsz One-Hot kódolással bajlódni a kategóriáknál (pl. színek, városok), a Döntési fák alapból megeszik ezeket. (Bár ahogy korábban említettem, a Scikit-Learn implementációi mindent számmá konvertálva várnak).

**5. A probléma linearitása:**
Ha az adatok egy egyszerű egyenessel/síkkal szétválaszthatók, a Logisztikus regresszió vagy a Lineáris SVM villámgyors és tökéletes lesz. Ha nagyon bonyolult, nem-lineáris a kapcsolat, akkor mély tanulás (Deep Learning), Ensemble módszerek, vagy RBF (Kernel) SVM kell.

**6. Sebesség (Tanítás vs. Predikció):**

- **Tanítási sebesség:** A Neurális hálók betanítása napokig is tarthat, míg egy Logisztikus regresszió másodpercek alatt megvan. A Véletlen Erdő (Random Forest) algoritmusok ráadásul könnyen párhuzamosíthatók több CPU magra.
- **Predikciós (Éles) sebesség:** Ha a modellt egy önvezető autóba vagy egy magas frekvenciájú tőzsdei robotba teszed, milliszekundumok alatt kell döntenie. Az SVM vagy a Regressziók villámgyorsak, mert csak egy képletbe kell behelyettesíteni. A kNN viszont élesben nagyon lassú lehet, mert minden új adatnál a teljes adatbázissal távolságot kell mérnie.

> *(Ha Pythonban dolgozol, a Scikit-Learn csapata készített egy zseniális "csalópuskát" (Cheat Sheet) a választáshoz, amit a könyv is bemutat).*

<img width="581" height="733" alt="1" src="https://github.com/user-attachments/assets/fd15c8a1-86b5-4164-817e-60a6b6da4c78" />

>  Ez egy folyamatábra, amely Start pontból indulva ("Több mint 50 adatod van?") vezet el a megfelelő algoritmushoz (pl. "Classification $\rightarrow$ Linear SVC").


👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/05%20Fejezet/MLAlgorithmDecisionNavigator.html)

---

## 5.3 A Három Halmaz (The Three Sets)

Eddig lazán használtuk az "adathalmaz" (dataset) és "tanítóhalmaz" (training set) szavakat szinonimaként. A professzionális gépi tanulásban azonban ez a legnagyobb bűn, amit elkövethetsz.

Ha a teljes adatbázisodon tanítod be a modellt, majd ugyanezen az adatbázison teszteled a pontosságát, becsapod magad. Egy olyan buta algoritmus, ami egyszerűen "készít egy képernyőfotót" a memóriájába a tanító adatokról, 100%-os pontosságot érne el a teszten, de a való világban, egy teljesen új adaton azonnal elbukna (túltanulás / overfitting). Mi azt akarjuk, hogy a gép az összefüggést tanulja meg, ne a konkrét számokat.

Ezért a legelső lépés, mielőtt bármit csinálnál: **Megkevered a nyers adatokat, és három részre (halmazra) vágod őket!**

**1. Tanító halmaz (Training set):**
Ez a legnagyobb szelet. Az algoritmus kizárólag ezeket az adatokat láthatja a $\mathbf{w}$ és $b$ paraméterek (a matematika) beállításához.

**2. Validációs halmaz (Validation set):**
Ezt az algoritmus tanulás közben nem látja (Hold-out set). Arra használod te, a mérnök, hogy teszteld a különböző algoritmusokat (pl. kipróbálod rajta az SVM-et és a kNN-t is), és itt állítod be a hiperparamétereket (pl. próbálgatod az SVM $C$ értékét, hogy mikor adja a legjobb eredményt ezen a halmazon).

**3. Teszt halmaz (Test set):**
A legszentebb halmaz. Ezt soha, semmilyen körülmények között nem használhatod se tanulásra, se paraméter-állítgatásra. Amikor a Validációs halmazon kiválasztottad a végső, nyertes modelledet a legjobb paraméterekkel, egyetlen egyszer lefuttatod ezen a Teszt halmazon. Ez az az eredmény (pontosság), amit beírhatsz a prezentációba a főnöknek vagy az ügyfélnek, mert ez mutatja meg a modell valódi, élesbeli teljesítményét egy teljesen ismeretlen adaton.

### Arányok (Proportions):

A klasszikus ökölszabály: **70% Tanító / 15% Validációs / 15% Teszt.**

**A Big Data korszaka:** Ha több millió adatod van, felesleges százezreket pazarolni a tesztelésre. Ilyenkor a modern arány gyakran: **95% Tanító / 2.5% Validációs / 2.5% Teszt.**

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/05%20Fejezet/DatasetVolumeSplitter.html)

## 5.4 Alulillesztés (Underfitting) és Túltanulás (Overfitting)

A modellek teljesítményét két fő statisztikai fogalommal, a torzítással (bias) és a varianciával (variance) jellemezzük.

### 1. Alulillesztés (High Bias / Magas torzítás)

Ha a modell a tanító adatokon is sok hibát vét, azt mondjuk, hogy alulilleszt (underfit) vagy magas a torzítása. Egyszerűen képtelen megtanulni az összefüggéseket.

**Okai:**
- A modell túl egyszerű (pl. egy görbült, exponenciális adathalmazra egy merev egyenest akarunk ráerőltetni).
- A jellemzők (features) nem hordoznak elég információt. (Hiába akarjuk megjósolni valakiről, hogy rákos-e, ha csak a magasságát és a pulzusát ismerjük).

**Megoldás:** Bonyolultabb modell választása (pl. lineáris helyett polinomiális), vagy jobb jellemzőmérnökség (erősebb prediktív erejű adatok kinyerése).

<img width="932" height="333" alt="2" src="https://github.com/user-attachments/assets/4d2809d8-6415-412f-94eb-2d474f1cadec" />

>  Ez a klasszikus 3 paneles kép, ami bemutatja az alulillesztést (merev egyenes), a jó illeszkedést (U-alakú görbe) és a túltanulást (vadul oszcilláló 15-ödfokú polinom).

### 2. Túltanulás (High Variance / Magas variancia)

Ha a modell a tanító adatokon tökéletesen (hiba nélkül) teljesít, de a teszt/validációs adatokon hatalmasat téved, akkor túltanult (overfit).

**Mi az a Variancia?** A statisztikában ez azt jelenti, hogy a modell túlságosan érzékeny a tanítóhalmaz apró zajaira (fluktuációira). Ha csak egy picit is másmilyen adathalmazt adnál neki, egy teljesen más modellt (pl. máshol kanyargó görbét) építene fel.

**Okai:**
- A modell túl komplex (pl. egy túl mély döntési fa, vagy egy óriási neurális háló).
- Túl sok a jellemző (dimenzió), de túl kevés hozzá a tanító példa.

**Megoldások:**
- Egyszerűbb modell választása.
- Dimenziócsökkentés (kevesebb jellemző használata).
- Több adat gyűjtése.
- Regularizáció (a legfontosabb és leggyakoribb technika, lásd alább).

---

## 5.5 Regularizáció (A Bias-Variance Tradeoff kezelése)

Még a legegyszerűbb, lineáris modell is képes túltanulni, ha nagyon sok dimenzió (jellemző) van, de kevés az adat. Ilyenkor a gép megpróbál minden egyes paraméternek ($\mathbf{w}$) valamilyen nem nulla értéket adni, hogy extrém bonyolult összefüggéseket kreáljon, és tökéletesen bemagolja a tanítóadatok zaját.

A **Regularizáció** egy gyűjtőfogalom azokra a módszerekre, amelyek kényszerítik az algoritmust, hogy egyszerűbb modellt építsen.

**A Tradeoff (Kompromisszum):** Ezzel tudatosan beáldozunk egy picit a pontosságból (megnöveljük a bias-t), cserébe drasztikusan csökkentjük a varianciát, így a modell a való világban (új adatokon) sokkal megbízhatóbb lesz.

**A módszer lényege:** Fogjuk az eredeti optimalizációs (költség) függvényt, és kiegészítjük egy büntető taggal. Minél bonyolultabb (minél nagyobb paraméterekkel dolgozik) a modell, ez a büntető tag annál nagyobbat "üt" az eredményen.

### L1 Regularizáció (Lasso)

Az L1 módszernél a súlyvektorok ($\mathbf{w}$) abszolút értékeinek összegét adjuk hozzá a hibához.

```math
\min_{\mathbf{w},b} C \sum_{j=1}^{D} |w^{(j)}| + \frac{1}{N} \sum_{i=1}^{N} (f_{\mathbf{w},b}(\mathbf{x}_i) - y_i)^2
```

**A $C$ hiperparaméter:** Ha $C=0$, nincs regularizáció (sima regresszió). Ha a $C$ nagyon nagy, a gép pánikba esik a büntetéstől, és elkezdi a kevésbé fontos jellemzők súlyát pontosan nullára redukálni.

**Hatása (Ritka modell / Sparse model):** Beépített jellemzőválasztást (feature selection) csinál. Ha volt 100 jellemződ, az L1 lehet, hogy 80-at lenulláz, és csak a 20 legfontosabbat tartja meg. Ez nagyon jó, ha ember számára megmagyarázható (explainable) modellt akarsz.

### L2 Regularizáció (Ridge)

Az L2 módszernél a súlyvektorok négyzetösszegét használjuk büntetésként.

```math
\min_{\mathbf{w},b} C \sum_{j=1}^{D} (w^{(j)})^2 + \frac{1}{N} \sum_{i=1}^{N} (f_{\mathbf{w},b}(\mathbf{x}_i) - y_i)^2
```

**Hatása:** Nem nullázza le a paramétereket, csak egyenletesen nagyon kicsire (nulla közelébe) nyomja össze őket.

**Előnye:** Mivel a négyzetes függvény ($x^2$) folytonosan deriválható, a gradiens ereszkedés nagyon könnyen és gyorsan tud vele számolni. Ha pusztán a modell teljesítménye (pontossága) a cél a teszt adatokon, a gyakorlatban szinte mindig az L2 adja a jobb eredményt.

> *(Megjegyzés: A kettő kombinációját Elastic Net-nek hívják).*

### 🔢 Számpélda: Hogyan büntet az L1 és az L2?

Képzeljük el, hogy a gép betanult egy modellt három jellemzőre, a következő súlyokkal: $\mathbf{w} = [0.8, -1.5, 0.1]$. A büntető szorzó legyen $C = 2$.

**Kiszámoljuk az L1 (Lasso) büntetést:**
- Vesszük az abszolút értékeket: $|0.8| + |-1.5| + |0.1| = 0.8 + 1.5 + 0.1 = \mathbf{2.4}$
- Szorozzuk a $C$-vel: $2 \cdot 2.4 = \mathbf{4.8}$
- (Az eredeti hibához hozzáadódik egy $4.8$-as büntetés).

**Kiszámoljuk az L2 (Ridge) büntetést:**
- Vesszük a négyzetösszeget: $0.8^2 + (-1.5)^2 + 0.1^2 = 0.64 + 2.25 + 0.01 = \mathbf{2.90}$
- Szorozzuk a $C$-vel: $2 \cdot 2.90 = \mathbf{5.8}$
- (Az eredeti hibához hozzáadódik egy $5.8$-as büntetés).

**Mi történik a legkisebb súllyal ($0.1$)?**
- Az **L1** esetében a $0.1$ abszolút értéke $0.1$. A gépnek megéri ezt teljesen levinni nullára, mert lineárisan csökkenti a büntetést. Így az L1 "kidobja" ezt a jellemzőt.
- Az **L2** esetében a $0.1$ négyzete már csak $0.01$. Ez olyan pici büntetés, hogy a gép nem fog izzadni azon, hogy ezt teljesen lenullázza, csak meghagyja apró számként. Ezért nem ad az L2 "ritka" (sparse) modellt.

- ## 5.6 A modell teljesítményének értékelése (Model Performance Assessment)

### Regressziós modellek értékelése

A regressziónál (ahol egy folytonos számot, pl. lakásárat jósolunk) az értékelés pofonegyszerű. Emlékszel az MSE-re (Mean Squared Error / Átlagos négyzetes hiba)?

Kiszámoljuk az MSE-t a **Tanító halmazon**, és kiszámoljuk a **Teszt halmazon** is.

Ha a Teszt halmazon a hiba drasztikusan nagyobb, mint a Tanító halmazon, akkor a modellünk túltanult (overfitting). Ezen regularizációval vagy paraméter-hangolással lehet segíteni.

**Mi számít jó modellnek?** Létezik egy "Átlag-modell" (Mean model), ami a világ legbutább modellje: gondolkodás nélkül mindenre a tanító adatok átlagát tippeli (pl. minden lakás $40$ millió). Ha a te szuper okos gépi tanulási modelled nem tud kisebb hibát (MSE) produkálni, mint ez a buta átlag-modell, akkor a modelled rossz.

---

### Osztályozó modellek (Classification) értékelése

Itt a dolgok sokkal trükkösebbek. Ha azt jósolod, hogy egy email spam-e vagy sem, egy sima hiba-átlag nem mond el mindent. Erre találták ki a következő eszközöket.

#### 5.6.1 Tévesztési Mátrix (Confusion Matrix)

Ez egy zseniális táblázat, ami darabokra szedi, hogy pontosan miben tévedett a modell. Egy bináris (kétosztályos) probléma esetén (pl. Spam / Nem Spam) a táblázat $2 \times 2$-es.

| | Spam (A Gép szerint) | Nem Spam (A Gép szerint) |
| :--- | :---: | :---: |
| **Spam (Valóságban)** | $23$ (TP) | $1$ (FN) |
| **Nem Spam (Valóságban)** | $12$ (FP) | $556$ (TN) |

**A 4 varázsszó:**

- **TP (True Positive - Valós Pozitív):** A levél Spam volt, és a gép is Spamnek mondta. ($23$ db) $\rightarrow$ Tökéletes!
- **TN (True Negative - Valós Negatív):** A levél normál volt, és a gép is normálnak mondta. ($556$ db) $\rightarrow$ Tökéletes!
- **FP (False Positive - Téves Pozitív / 1-es típusú hiba):** A levél normál volt, de a gép Spamnek nézte (pozitívan riasztott, de tévesen). ($12$ db) $\rightarrow$ Ez nagyon idegesítő, mert a főnököd fontos levele a Kukában landolt.
- **FN (False Negative - Téves Negatív / 2-es típusú hiba):** A levél Spam volt, de a gép normálnak hitte (nem riasztott). ($1$ db) $\rightarrow$ Kevésbé fájdalmas, legfeljebb te törlöd ki kézzel a beérkezettek közül.

> *(Többosztályos esetben - pl. Kutya, Macska, Egér - a mátrix $3 \times 3$-as lesz. Ott pontosan látod, ha a gép a macskát mindig kutyával keveri, de az egérrel sosem).*

#### 5.6.2 Precízió és Visszahívás (Precision & Recall)

A fenti betűszavakból két nagyon fontos metrikát számolunk.

**1. Precízió (Precision): Mennyire hihetek a gépnek, ha riaszt?**

```math
\text{Precision} = \frac{TP}{TP + FP}
```

Példa a fenti mátrixból: $\frac{23}{23 + 12} = \frac{23}{35} \approx \mathbf{0.65}$ ($65\%$).

**Jelentése:** Ha a gépem azt mondja egy levélre, hogy Spam, akkor csak $65\%$ az esélye, hogy tényleg az (és $35\%$, hogy egy tévesen kidobott normál levél).

**2. Visszahívás (Recall): Az összes meglévő rosszfiúból mennyit talált meg a gép?**

```math
\text{Recall} = \frac{TP}{TP + FN}
```

Példa a fenti mátrixból: $\frac{23}{23 + 1} = \frac{23}{24} \approx \mathbf{0.95}$ ($95\%$).

**Jelentése:** A világon létező Spamek $95\%$-át sikeresen elfogta a rendszerünk. (De ahogy a Precíziónál láttuk, ennek ára volt: sok ártatlan levelet is kidobott).

**A Precízió-Visszahívás Dilemma (Trade-off):**

A gyakorlatban lehetetlen mindkettőt 100%-on tartani.

- **Spamszűrőnél:** Magas Precíziót akarsz! Nem baj, ha átcsúszik pár spam (alacsony Recall), de egyetlen fontos levelet (FP) se dobjon ki a gép. Ezt úgy éred el, hogy a gép csak akkor riaszt, ha legalább $0.9$ ($90\%$) a valószínűség.
- **Rákszűrésnél:** Magas Recall-t akarsz! Nem baj, ha sok embert tévesen hívsz vissza további drága tesztekre (FP), de egyetlen rákos beteget se küldj haza úgy, hogy makkegészséges (FN)! Itt már $0.1$ ($10\%$)-os rákesélynél is riasztania kell a gépnek.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/05%20Fejezet/ClassificationThresholdSimulator.html)

#### 5.6.3 Pontosság (Accuracy)

A legnépszerűbb, de a legveszélyesebb metrika.

```math
\text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN}
```

Egyszerűen a helyes tippek (TP+TN) aránya az összes tippen belül.

- **Mikor jó?** Ha a tévesztések (FP és FN) ugyanolyan súlyúak, és az osztályok egyensúlyban vannak (50% kutyás, 50% macskás kép).
- **Mikor tragikus?** Kiegyensúlyozatlan (imbalanced) adatoknál. Ha 1000 emailből 1 Spam, a gép mondhatja vakon mindenre, hogy "Nem Spam". A pontossága $\frac{999}{1000} = 99.9\%$ lesz, a modell mégis használhatatlan.

#### 5.6.4 Költség-súlyozott Pontosság (Cost-Sensitive Accuracy)

Ha a hibák nem egyenrangúak (pl. egy FN rákdiagnózis emberéletbe kerül, míg egy FP csak egy felesleges vérvételbe), büntetőszorzókat (Költséget) rendelünk hozzájuk.

- $FN$ költség = $1000$
- $FP$ költség = $10$

A számolásnál a sima FP és FN darabszámokat megszorozzuk ezekkel a súlyokkal, mielőtt osztanánk. Így az a modell fog nyerni, amelyik minimalizálja az életveszélyes tévedéseket.

#### 5.6.5 ROC Görbe és az AUC (Area Under the Curve)

Ez a mérnökök kedvenc vizuális eszköze. Arra szolgál, hogy egyetlen görbével megmutassa, hogyan teljesít a modell az összes lehetséges küszöbérték (threshold) mellett.

> *(Figyelem: Ezt csak olyan modelleken lehet használni, amik valószínűséget adnak vissza – pl. Logisztikus regresszió, Döntési fák. A sima SVM-en nem működik, mert az csak "igen/nem"-et mond).*

**A két tengely:**

- **Y tengely (TPR - True Positive Rate):** Ez hajszálpontosan ugyanaz, mint a Visszahívás (Recall). $\frac{TP}{TP+FN}$.
- **X tengely (FPR - False Positive Rate):** A téves riasztások aránya a valós negatívok között. $\frac{FP}{FP+TN}$.

**Hogyan rajzolódik ki a görbe?**

A gép végigzongorázza a küszöbértéket $0.0$-tól $1.0$-ig (pl. $[0.1, 0.2 \dots 0.9]$). Minden egyes küszöbnél kiszámolja az $X$ és $Y$ koordinátát, és letesz egy pontot a grafikonra.

- Ha a küszöb $0.0$: A gép mindenre riaszt. $100\%$ lesz a TPR (megtalált minden spamet), de $100\%$ lesz az FPR is (minden normál levelet is kidobott). Ez a grafikon jobb felső sarka.
- Ha a küszöb $1.0$: A gép sosem riaszt. $0\%$ a TPR, és $0\%$ az FPR. Ez a bal alsó sarok.

<img width="762" height="777" alt="3" src="https://github.com/user-attachments/assets/52fd1500-9e26-4340-8a55-d36d81949307" />

> Ez 4 db kis panelt mutat egymás mellett, ahol látszik, ahogy a görbe egyre jobban "kidudorodik" a bal felső sarok felé, és az alatta lévő terület (AUC) egyre nő.

**Az AUC (Area Under the Curve - Görbe alatti terület):**

Ez egyetlen szám, ami megmondja, milyen jó a modell.

- **$\text{AUC} = 1.0$:** Tökéletes, hibátlan modell. (Ilyen a valóságban nincs).
- **$\text{AUC} = 0.85$:** Nagyon jó modell (Erősen kidudorodik a bal felső sarok felé).
- **$\text{AUC} = 0.5$:** A teljesen véletlenszerű, pénzfeldobós modell. (Egy egyenes átló a grafikonon). Ha ilyet kapsz, dobhatod ki a modellt.
- **$\text{AUC} < 0.5$:** A modell rosszabb, mint a vak tippelés. Valamit nagyon elrontottál a programozás során (vagy fordítva tanítottad be a címkéket).

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/05%20Fejezet/ROCCurve%20ClassSeparability.html)

- ## 5.7 Hiperparaméter-hangolás (Hyperparameter Tuning)

Korábban már beszéltünk arról, hogy az algoritmusoknak vannak olyan beállításai (hiperparaméterei), amiket a gép nem tud magától megtanulni. Ezeket a mérnöknek kell beállítania a tanítás előtt. Ilyen például a döntési fáknál a fa maximális mélysége ($d$), az SVM-nél a büntetőszorzó ($C$), vagy a gradiens ereszkedésnél a tanulási ráta ($\alpha$).

De honnan tudjuk, hogy mi a legjobb érték? A válasz: **Kísérletezéssel.** Erre szolgálnak az alábbi stratégiák.

> *(Megjegyzés: A könyv ehhez a szakaszhoz nem rendelt külön ábrákat, így ide nem kell könyvbeli képet beillesztened).*

### 1. Rácskeresés (Grid Search)

A legnépszerűbb, de "legbutább" (brute-force) módszer. Előre megadunk egy listát a kipróbálandó értékekről, és a gép az összes lehetséges kombinációt lefuttatja.

**A Logaritmikus trükk:** Ha nem tudod, hol keresd a $C$ paramétert, ne lineárisan lépkedj (1, 2, 3, 4...), hanem logaritmikusan (nagyságrendenként ugorva).

**Számpélda a könyvből:**
Tegyük fel, egy SVM-et tanítasz, és két hiperparamétert keresel:

- A büntetőszorzót ($C$): `[0.001, 0.01, 0.1, 1.0, 10, 100, 1000]` (7 darab érték)
- A Kernelt (a dimenzióváltó matematikát): `['linear', 'rbf']` (2 darab érték)

A Rácskeresés veszi a $7 \times 2 = \mathbf{14}$ kombinációt:

1. Modell: $C=0.001$, Kernel='linear'
2. Modell: $C=0.01$, Kernel='linear'
3. ...és így tovább, egészen a 14. Modellig ($C=1000$, Kernel='rbf').

A géped betanít 14 különálló modellt a Tanító halmazon, majd mind a 14-et leteszteli a Validációs halmazon. Amelyik a legmagasabb pontszámot éri el, az lesz a győztes párosítás.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/05%20Fejezet/GridSearchOptimization.html)

### 2. Véletlenszerű keresés (Random Search)

Ha mondjuk 5 hiperparamétered van, a Rácskeresés kombinációinak száma hamar több ezerre ugrik (ezt hívják a Dimenziók Átkának).

A Random Search-nél nem konkrét listát adsz meg, hanem egy statisztikai eloszlást (tartományt), és megmondod a gépnek: *"Válassz ki teljesen véletlenszerűen 50 kombinációt, és csak azt az 50-et próbáld ki!"* A gyakorlatban ez sokszor ugyanolyan jó eredményt ad, mint a Rácskeresés, de töredéknyi idő alatt.

### 3. Bayes-i optimalizáció (Bayesian Optimization)

Ez az intelligens keresés. Nem vakon próbálkozik az összes kombinációval, hanem "tanul" a korábbi próbálkozásaiból. Ha látja, hogy a $C=10$ és $C=100$ jó eredményt adott, a következő próbálkozását okosan ebbe a zónába irányítja, ignorálva a nyilvánvalóan rossz tartományokat.

---

## 5.7.1 Keresztvalidáció (Cross-Validation)

Emlékszel az előző fejezetből a 3-as osztásra (Tanító, Validációs, Teszt)?

Van egy nagy probléma: mi van akkor, ha nagyon kevés adatod van (pl. csak 100 db orvosi lelet)? Ha ebből levágsz 15-öt validációra és 15-öt tesztre, akkor mindössze 70 adatod marad tanítani, ami édeskevés egy jó modellhez.

Erre a problémára találták ki a **Keresztvalidációt**. Ilyenkor az adathalmazt csak 2 részre vágjuk: Tanítóra és Tesztre. A Validációs halmazt a Tanító halmazból "szimuláljuk".

### A K-Szeres Keresztvalidáció (K-Fold Cross-Validation) lépései:

A leggyakoribb a $K=5$ (5-szörös) keresztvalidáció. Lássuk egy konkrét, 100 elemű tanító halmazon keresztül:

1. **Darabolás (Folds):** A 100 elemet véletlenszerűen 5 egyenlő, 20 elemű darabra vágjuk ($F_1, F_2, F_3, F_4, F_5$).
2. **Iteratív tanítás (A trükk):** Felépítünk 5 teljesen különálló modellt (ugyanazokkal a hiperparaméterekkel), és forgó színpad-szerűen cserélgetjük a szerepeket:
   - **1. Modell ($f_1$):** Tanítunk az $F_2, F_3, F_4, F_5$-ön (80 adat), és validálunk (tesztelünk) az $F_1$-en (20 adat). Megkapjuk az 1. pontszámot.
   - **2. Modell ($f_2$):** Tanítunk az $F_1, F_3, F_4, F_5$-ön, és validálunk az $F_2$-n. Megkapjuk a 2. pontszámot.
   - **3. Modell ($f_3$):** Validálunk az $F_3$-on... és így tovább az 5. modellig.
3. **Átlagolás:** Vesszük a kapott 5 pontszámot, és egyszerűen átlagoljuk őket. Ez a végső Keresztvalidációs Pontszám.

**Miért zseniális ez?**
Mert így minden egyes adatpont pontosan egyszer szerepelt validációs adatként, és négyszer tanító adatként! Egyetlen adatot sem pazaroltunk el.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/05%20Fejezet/5-FoldCross-Validation.html)

---

### A Teljes, Professzionális Folyamat

A gyakorlatban a fenti két módszert kombináljuk (ezt hívják `GridSearchCV`-nek a Scikit-Learnben).

1. Kiválasztod a 14 hiperparaméter-kombináció egyikét.
2. Lefuttatod rajta az 5-szörös keresztvalidációt, felírod az átlagpontszámot.
3. Ezt megcsinálod mind a 14 kombinációra.
4. Kiválasztod a legmagasabb pontszámot elért hiperparaméter-párost.
5. **(A legfontosabb lépés!)** Fogod az eredeti, egyben lévő 100 elemű tanítóhalmazt, és a nyertes hiperparaméterekkel most már az egész adathalmazon betanítasz egy végső modellt.
6. Ezt a végső modellt engeded rá arra a Teszt halmazra, amit a legelső lépésnél félretettél és még sosem láttál.
