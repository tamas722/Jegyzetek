# 7. Problémák és Megoldások (Problems and Solutions)

## 7.1 Kernel Regresszió (Kernel Regression)

**A probléma:**
Eddig a lineáris regresszióról beszéltünk, aminek a kimenete egy egyenes (vagy egy sík). De mi van, ha a való világ adatai kanyarognak? Próbálkozhatunk polinomiális regresszióval (pl. egy parabola: $y = w_1 x_i + w_2 x_i^2 + b$), de ha nagyon sok dimenziónk van (pl. 50 jellemző), lehetetlen megtippelni, hogy milyen fokú polinomra (a köbön? a negyediken?) lesz szükségünk a jó illeszkedéshez.

### A megoldás: Kernel Regresszió

Ahelyett, hogy merev paramétereket ($\mathbf{w}$) próbálnánk megtanulni, dobjuk el az egészet! A Kernel regresszió egy **nem-parametrikus** (non-parametric) módszer, akárcsak a kNN. Nincs tanulási fázisa, a modellt maga a nyers adathalmaz alkotja.

Az elképzelés zseniálisan egyszerű: Ha egy új $x$ pont értékét akarod megjósolni, fogd az összes ismert $y$ értéket az adatbázisodban, és vonj belőlük egy Súlyozott Átlagot.

Képletben (ahogy a könyv is írja, a belső eloszlás figyelembevételével):

```math
f(x) = \sum_{i=1}^{N} w_i y_i
```

**Hogyan számoljuk ki a $w_i$ súlyt?**
A súlyt a Kernel függvény ($k$) adja meg. Ez a függvény megméri a távolságot az új pontod ($x$) és a régi pont ($x_i$) között. Minél közelebb van a régi pont az újhoz, annál nagyobb súlyt (beleszólást) kap az átlagoláskor!

A leggyakoribb kernel a **Gauss-kernel** (ami egy haranggörbe):

```math
k(z) = \frac{1}{\sqrt{2\pi}} \exp\left(-\frac{z^2}{2}\right)
```

> *(Megjegyzés több dimenzió esetén: az egydimenziós különbséget $(x_i - x)$ egyszerűen kicseréljük a többdimenziós Euklideszi távolságra $\|x_i - x\|$.)*

### A Sávszélesség ($b$) hiperparaméter - A titkos fegyver

A súlyok képletében van egy kis "osztó", a $b$ (bandwidth / sávszélesség): $k\left(\frac{x_i - x}{b}\right)$.

Ez határozza meg, hogy mennyire "messzire" lát a kernel:

- Ha $b$ kicsi, csak a közvetlen szomszédok számítanak. A görbe minden pontra rá fog ugrani $\rightarrow$ **Túltanulás (Overfit)**.
- Ha $b$ nagy, minden pont (még a távoliak is) beleszól az átlagba. A görbe kisimul, de lehet, hogy túl merev lesz $\rightarrow$ **Alulillesztés (Underfit)**.

<img width="641" height="221" alt="1" src="https://github.com/user-attachments/assets/b48ede9a-ad0f-422b-a838-72277825858f" />


>  Ez a klasszikus 3 paneles kép, ami bemutatja a Kernel regressziót három különböző $b$ értékkel (Jó illeszkedés / Enyhe túltanulás / Erős túltanulás).

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/07%20Fejezet/KernelRegressionSmoothing.html)

### 🔢 Számpélda: Kernel Regresszió a gyakorlatban

Meg akarjuk jósolni a hőmérsékletet délben ($x = 12:00$). Két mérésünk van:

- **Mérés 1:** $x_1 = 11:00$, hőmérséklet $y_1 = 20^\circ C$
- **Mérés 2:** $x_2 = 18:00$, hőmérséklet $y_2 = 10^\circ C$

A Kernel függvény alapján az $x_1$ nagyon közel van (csak 1 óra különbség), így az ő súlya hatalmas lesz (mondjuk $90\%$). Az $x_2$ messze van (6 óra különbség), így ő csak $10\%$-os súlyt kap.

A predikciónk: $0.9 \cdot 20^\circ C + 0.1 \cdot 10^\circ C = 18 + 1 = \mathbf{19^\circ C}$.

Tökéletesen logikus, a közelebbi ponthoz hasonlít az eredmény!

---

## 7.2 Többosztályos Osztályozás (Multiclass Classification)

**A probléma:**
A legtöbb algoritmus (mint a Logisztikus regresszió vagy az SVM) eleve bináris problémákra lett kitalálva. Tudják, mi az a $0$ és $1$, vagy a $+1$ és $-1$. De a való világban gyakran három vagy több osztályunk van, például egy gyümölcs lehet: $y \in \{ \text{Alma (1)}, \text{Körte (2)}, \text{Szilva (3)} \}$.

Néhány algoritmust gyerekjáték felokosítani:

- **Döntési Fa (ID3):** Sima ügy. A falevél egyszerűen megnézi, hogy a bejutott adatok közül melyik osztályból van a legtöbb.
- **Logisztikus regresszió:** A $0/1$ közötti Szigmoidot lecseréljük a (6. fejezetből ismerős) Softmax függvényre, ami százalékokat oszt szét az összes osztály között.
- **kNN:** Itt is egyszerű többségi szavazást tartunk (ha a 3 szomszédom Körte, Alma, Körte, akkor én Körte vagyok).

### Mi van az SVM-mel?

Az SVM egy matematikai vonal (döntési határ), ami szigorúan két térfelet választ el (Pozitív vs. Negatív). A matematikájába kódolva nem létezik a "harmadik térfél".

### A megoldás: Egy a Többi Ellen (One-Versus-Rest / OvR)

Ezt a stratégiát bármilyen bináris algoritmusra (pl. SVM-re) rá lehet húzni. A trükk az, hogy a feladatot szétszedjük annyi bináris problémára, ahány osztályunk van ($C$ darab).

Ha Alma (1), Körte (2) és Szilva (3) osztályaink vannak:

1. **Első modell (Az "Alma" detektor):** A tanítóadatokban az összes Körte és Szilva címkéjét átírjuk $0$-ra. Az SVM betanul: Alma (1) vs. Mindenki Más (0).
2. **Második modell (A "Körte" detektor):** Az Alma és a Szilva lesz a $0$. Az SVM betanul: Körte (1) vs. Mindenki Más (0).
3. **Harmadik modell (A "Szilva" detektor):** Az Alma és a Körte lesz a $0$. Szilva (1) vs. Mindenki Más (0).

### A Predikció (Döntés):

Bejön egy titokzatos új gyümölcs. Mind a 3 betanított modellt ráeresztjük. A kérdés már csak az: kinek higgyünk, ha netán az Alma-detektor és a Körte-detektor is riaszt?

A megoldás az, hogy megnézzük a **Bizonyosságot (Certainty / Confidence)**! Azt az osztályt választjuk, amelyiknek a detektora a leghangosabban (legbiztosabban) jelez.

- **Logisztikus regressziónál** ez könnyű, mert ad egy valószínűséget (pl. az Alma-detektor $0.81$-et adott, a Körte-detektor $0.55$-öt $\rightarrow$ Alma nyert!).
- **SVM-nél** a bizonyosságot a döntési határtól való távolság ($d$) adja meg. Minél messzebb van az új pont a határvonal "pozitív" oldalán, a gép annál biztosabb benne:

```math
d = \frac{\mathbf{w}^* \cdot \mathbf{x} + b^*}{\|\mathbf{w}\|}
```

A legmesszebb lévő távolság adja a győztes osztályt!

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/07%20Fejezet/One-vs-Rest(OvR)Strategy.html)

## 7.3 Egyosztályos Osztályozás (One-Class Classification)

A hagyományos gépi tanulásban a gép úgy tanul meg dönteni, hogy kap pozitív és negatív példákat is (pl. képeket kutyákról és képeket macskákról).

Az **Egyosztályos osztályozás** (vagy anomália detektálás) ezzel szemben sokkal nehezebb: kizárólag egyetlen osztályról (a "Normálisról") vannak adataink, és a gépnek meg kell tanulnia kiszűrni mindent, ami ettől eltér (az "Idegent").

> **Tipikus példa:** Egy biztonságos szerver hálózati forgalmának figyelése. Naponta millió adatcsomag jön be, ami mind normális. Talán soha nem volt még hackertámadásunk (nincs "rossz" adatunk), de ha egy napon történik egy, a gépnek riasztania kell, mert a forgalom hirtelen mássá válik, mint a megszokott.

**Alkalmazások:** Kiugró értékek (outlier) szűrése, anomália detektálás, újdonság detektálás (novelty detection).

A legnépszerűbb algoritmusok erre a feladatra az **Egyosztályos Gauss-modell**, az **Egyosztályos k-Means**, az **Egyosztályos kNN** és az **Egyosztályos SVM**.

### Az Egyosztályos Gauss-modell (One-Class Gaussian)

Ennek a módszernek a lényege, hogy azt feltételezzük: a normális adataink egy statisztikai **Többváltozós Normál Eloszlásból** (Multivariate Normal Distribution - MND) származnak. Ennek képe egy többdimenziós haranggörbe.

A valószínűségsűrűség-függvény (PDF) képlete, amely kiszámolja, hogy egy $\mathbf{x}$ adat mennyire valószínű (mennyire "normális"):

```math
f_{\mu, \Sigma}(\mathbf{x}) = \frac{\exp\left( -\frac{1}{2} (\mathbf{x} - \mu)^T \Sigma^{-1} (\mathbf{x} - \mu) \right)}{\sqrt{(2\pi)^D |\Sigma|}}
```

Ez a képlet elsőre ijesztőnek tűnik, de bontsuk darabjaira!

Csak két dolgot (két paramétert) kell a gépnek megtanulnia (a Maximum Likelihood elvvel, ahogy a logisztikus regressziónál láttuk):

- **$\mu$ (Mű - Átlagvektor):** Ez adja meg a haranggörbe középpontját. (Hol van a "legnormálisabb" adat).
- **$\Sigma$ (Szigma - Kovarianciamátrix):** Ez adja meg a haranggörbe alakját és kiterjedését. (Mennyire széles, szűk, vagy épp ovális a harang).

**A "félelmetes" mátrixműveletek emberi nyelven:**

- **Transzponált ($T$):** Csak megfordítjuk a vektort (oszlopból sor lesz), hogy el lehessen végezni a mátrixszorzást.
- **Inverz ($\Sigma^{-1}$):** A mátrixok világában nem lehet simán "osztani". Az inverz mátrixszal való szorzás a mátrixos megfelelője a sima számokkal való osztásnak. (Itt gyakorlatilag elosztjuk a távolságot az adatok szórásával).
- **Determináns ($|\Sigma|$):** Egyetlen szám, ami megadja a mátrix által bezárt "térfogatot" vagy kiterjedést.

### 🔢 Számpélda: Hogyan riaszt az Egyosztályos Gauss?

Egyszerűsítsük le a képletet 1 Dimenzióra (csak egy jellemzőnk van: a szerver CPU hőmérséklete).

A tanító adatok alapján a gép megtanulja, hogy a normális hőmérséklet átlaga $\mu = 40^\circ C$, a szórása $\sigma = 5^\circ C$.

1. **Eset: Bejön egy $42^\circ C$-os mérés.**
   A képlet kiszámolja a távolságot az átlagtól: $42 - 40 = 2$. Ez nagyon kicsi különbség a szóráshoz képest. A gép kiköp egy valószínűséget: $p = 0.85$ ($85\%$). Mivel ez magas, a gép azt mondja: *"Ez az adat normális!"*

2. **Eset: Bejön egy $95^\circ C$-os mérés.**
   A távolság $95 - 40 = 55$. Ezt négyzetre emelve és a szórásalapú képletbe (a fenti $\exp$ részbe) betéve egy irdatlanul kicsi számot kapunk. A valószínűség: $p = 0.0000001$. A gép azonnal látja, hogy ez szinte lehetetlen a normál ügymenetben, és riaszt: *"Ez egy Anomália!"*

**A Küszöbérték (Threshold):**
Neked, a mérnöknek kell beállítanod egy határt (threshold). Például: *"Ha a kiszámolt $f_{\mu, \Sigma}(\mathbf{x})$ valószínűség kisebb, mint $0.01$ (azaz 1%), akkor dobd ki anomáliaként!"* Ez általában kísérletezés kérdése.

<img width="578" height="285" alt="2" src="https://github.com/user-attachments/assets/d451c80d-d914-4aab-a08f-20962b1cb4ab" />


>  A bal oldalán 2D-s kék pontok láthatók (ezek a normál adatok), a jobb oldalán pedig egy ezekre ráillesztett 3D-s haranggörbe (a Többváltozós Normál Eloszlás).

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/07%20Fejezet/3DGaussianAnomalyDetector.html)

---

### Bonyolultabb alakzatok: Gauss-keverék (Mixture of Gaussians)

Mi van akkor, ha a normális adataid nem egyetlen kerek kupacban vannak, hanem mondjuk két teljesen különálló csoportban (pl. a szerver terhelése hétköznap magas, hétvégén nagyon alacsony)? Ilyenkor egyetlen haranggörbe nem elég.

Erre való a **Gauss-keverék modell**, ami több haranggörbét mos össze egyetlen komplex formává (ezt a 9. fejezetben tárgyalja a könyv).

### Egyéb Egyosztályos módszerek (k-Means és kNN)

Ezek nagyon hasonló logikára épülnek, de nem használnak haranggörbét és valószínűségeket:

- **Egyosztályos k-Means:** Fogja a normál adatokat, és klaszterekbe (csoportokba) rendezi őket. Kiszámolja a csoportok középpontjait. Amikor jön egy új adat ($\mathbf{x}$), megnézi annak távolságát ($d(\mathbf{x})$) a legközelebbi középponttól. Ha ez a távolság nagyobb egy megadott sugárnál (küszöbértéknél), akkor az adat anomália.
- **Egyosztályos kNN:** Megnézi, milyen messze van az új adat a meglévő "normál" adatok halmazától. Ha egy új pont bedobva a térbe nagyon messze "esik" minden ismert normál adathoz képest (nincsenek közeli szomszédai), azonnal gyanússá válik és kiszűrődik.

- ## 7.4 Többcímkés Osztályozás (Multi-Label Classification)

Nagyon fontos, hogy ne keverjük össze a Többosztályos (Multiclass) és a Többcímkés (Multilabel) osztályozást!

- **Multiclass (Többosztályos):** A képen vagy alma, vagy körte, vagy szilva van. Egy adatnak csak egy címkéje lehet.
- **Multilabel (Többcímkés):** A képen lehet "Ember", "Természet" és "Koncert" is, egyszerre. Egy adatnak bármennyi címkéje lehet.

### Hogyan oldjuk meg a Többcímkés problémát?

**1. A "Szétvágós" módszer (Független bináris osztályozók):**
A legegyszerűbb, ha az Egy-a-Többi-Ellen (OvR) stratégiát alkalmazzuk. Ha van 10 lehetséges címkéd (címkénként Igen/Nem), betanítasz 10 különálló bináris modellt.

A különbség a Multiclass-hoz képest az, hogy a végén nem csak a legmagasabb pontszámút választod ki, hanem beállítasz egy küszöbértéket (threshold). Ha a "Koncert-detektor" $0.8$-at ad, és az "Ember-detektor" $0.9$-et, a küszöböd pedig $0.5$, akkor a gép mindkét címkét ráteszi a képre.

**Neurális hálózatoknál:** A kimeneti réteg minden neuronja (minden címke) egy Szigmoid függvényt kap, és a hibát **Bináris Keresztentrópiával** (Binary Cross-Entropy) számoljuk:

```math
-\left[ y_{i,l} \ln(\hat{y}_{i,l}) + (1 - y_{i,l})\ln(1 - \hat{y}_{i,l}) \right]
```

**2. A "Hamis Osztály" (Kombinációs) módszer:**
Ha a címkék szorosan összefüggnek (korrelálnak), és nincs belőlük túl sok, kombinálhatod őket.

Ha van egy "Fotó / Festmény" és egy "Portré / Táj / Egyéb" címkéd, létrehozol $2 \times 3 = 6$ új osztályt (pl. "Fotó-Portré", "Festmény-Táj"). Ezzel a Multilabel problémát visszabutítottad egy sima Multiclass problémára.

- **Előnye:** Megtanulja, hogy a címkék hogyan viszonyulnak egymáshoz (pl. a "Spam" ritkán "Kiemelt fontosságú").
- **Hátránya:** Ha sok a címke, az osztályok száma felrobban (kombinatorikus robbanás), és hihetetlen mennyiségű tanító adatra lesz szükséged.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/07%20Fejezet/ActivationLogicVisualizer.html)

---

## 7.5 Együttes Tanulás (Ensemble Learning)

A gépi tanulás egyik legzseniálisabb felismerése: **Sok "buta" (gyenge) modell együttesen okosabb tud lenni, mint egyetlen zseniális szupermodell.**
Ez a **Demokrácia elve** a matematikában.

- **A Gyenge Tanuló (Weak Learner):** Egy nagyon egyszerű modell, például egy Döntési fa, ami mindössze 1 vagy 2 mélységű. Nagyon pontatlan, de épphogy jobb, mint a véletlen pénzfeldobás (pl. 51%-os pontosságú).
- **Az Együttes (Ensemble):** Ha veszel 1000 ilyen buta fát, és mindegyik kicsit más, a szavazatuk átlaga drasztikusan lecsökkenti a hibát. Két fő típusa van: a **Bagging** és a **Boosting**.

### 7.5.1 Véletlen Erdő (Random Forest) - A "Bagging" elve

A Bagging (Bootstrap Aggregating) lényege, hogy párhuzamosan építünk rengeteg gyenge modellt, és a végén átlagoljuk/szavaztatjuk őket.

**Hogyan épül fel a Véletlen Erdő?**

1. **Adat-mintavétel (Visszatevéssel):** Van 1000 adatod. A gép 1000 alkalommal nyúl be a kalapba egy adatért, de miután kihúzta, visszadobja. Így egy új adathalmazt kap, amiben lesznek duplikációk, és lesznek hiányzó adatok. Ezt megcsinálja 100-szor, így kap 100 enyhén különböző adathalmazt.
2. **Fa építése (Jellemző-véletlenszerűséggel):** Mind a 100 adathalmazon betanít egy Döntési fát. A Véletlen Erdő trükkje (ami megkülönbözteti a sima Baggingtől), hogy a fák nem használhatják az összes jellemzőt a vágáshoz. Minden elágazásnál csak egy véletlenszerű alhalmazt (pl. a 10-ből csak 3 jellemzőt) kapnak meg.

**Miért zseniális ez? (A Korreláció elkerülése)**
Ha van egy iszonyatosan erős jellemződ (pl. a lakás alapterülete), akkor a sima fáknál az összes fa legelső vágása az alapterület lenne. Minden fa egyforma lenne (korrelálnának). Ha egyforma fák szavaznak, az nem csökkenti a hibát. A jellemzők korlátozásával a fák kénytelenek más, rejtettebb összefüggéseket is megtanulni. Ha 100 egymástól független (nem korreláló) modell szavaz, az átlaguk drasztikusan csökkenti a varianciát (a túltanulást).

### 7.5.2 Gradiens Turbózás (Gradient Boosting)

A Boosting teljesen más filozófia. Itt a fák nem párhuzamosan épülnek, hanem sorban egymás után, és minden új fa a korábbi fák hibáiból tanul.

**Hogyan működik (Regresszió esetén)?**

1. **Kezdő modell ($f_0$):** A legbutább modell, egyszerűen megtippeli a tanító adatok átlagát.
2. **A Maradék (Residual) kiszámítása:** Megnézzük, mennyit tévedett a modell minden egyes adatnál: $\hat{y}_i = y_i - f_0(x_i)$. Ezek lesznek az új célpontok (címkék)!
3. **Az Új fa ($f_1$):** Betanítunk egy új, buta fát, aminek az a feladata, hogy kizárólag ezeket a hibákat (a maradékot) jósolja meg.
4. **A Modell frissítése:** Hozzáadjuk az új fát a régi modellhez, megszorozva egy $\alpha$ (tanulási ráta) súllyal: $f_{\text{új}} = f_0 + \alpha f_1$.
5. A folyamat kezdődik elölről, újra és újra.

**Miért hívják Gradiens Boostingnak?**
Gondolj a Gradiens Ereszkedésre a 4. fejezetből! Ott a gradienseket (deriváltakat) használtuk arra, hogy megmondják, merre módosítsuk a paramétereket a hiba csökkentéséhez.

Itt a **Maradékok (Residuals)** pontosan a **Gradiens** szerepét töltik be! Megmutatják, milyen irányba (milyen fával) kell kiegészíteni a modellt, hogy a végső Költségfüggvény (MSE) közelebb kerüljön a minimumhoz. Míg a Bagging a **Varianciát** (túltanulást) csökkenti, a Boosting a **Torzítást** (Bias / alulillesztést) támadja meg és csökkenti. (Bár túl sok fával a Boosting túl tud tanulni).

> *(A könyv röviden kitér a Klasszifikációs Boostingra is, ahol a folyamat hasonló, de ott MSE helyett a Maximális Valószerűséget / Log-Likelihood-ot optimalizáljuk, a fákat pedig a Szigmoid függvény deriváltjaiból építjük).*

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/07%20Fejezet/Baggingvs.BoostingArchitecture.html)

## 7.6 Szekvenciák címkézése (Learning to Label Sequences)

A világunk tele van szekvenciákkal. A kommunikáció (szöveg), a DNS (génszekvenciák) vagy egy videó mind egymás után következő elemek sorozata.

**A probléma:**
Van egy $X$ listánk (a bemenet) és egy pontosan ugyanilyen hosszú $Y$ listánk (a kimenet/címkék).

**Példa:**
$X = [\text{"Nagy"}, \text{"szép"}, \text{"autó"}]$. Szeretnénk, ha a gép minden szónál megmondaná a szófajt. A kimenetünk így nézne ki: $Y = [\text{"Melléknév"}, \text{"Melléknév"}, \text{"Főnév"}]$.

### Hogyan oldjuk meg?

Emlékszel az RNN-ekre (Rekurrens Neurális Hálózatok) a 6. fejezetből? Ott a hálózat lépésről lépésre (időpillanatról időpillanatra) olvassa be az adatot, és mivel van memóriája, minden egyes időlépésben ki tud köpni egy $y^{(t)}$ címkét. Tehát simán rátehetjük a "Melléknév" címkét a második szóra, tudva, hogy az első is az volt.

**A régi módszer (CRF):**
A neurális hálók kora előtt a **Feltételes Véletlen Mezők** (Conditional Random Fields - CRF) algoritmust használták erre. Ez zseniális volt (a Logisztikus Regresszió egyfajta kiterjesztése szekvenciákra), de hatalmas mérnöki munkát igényelt.

Neked, az adatkutatónak kellett "kézzel" olyan jellemzőket (features) kódolnod a szavakhoz, mint: *"Kezdődik-e nagybetűvel?"* vagy *"Szerepel-e a városok listájában?"*.

Ma már a CRF-et teljesen kiszorították a **Kétirányú (Bidirectional) Mély RNN-ek és LSTM-ek**, amelyek emberi beavatkozás (kézi jellemzőgyártás) nélkül maguktól tanulják meg ezeket az összefüggéseket a nyers szövegből.

---

## 7.7 Szekvenciából Szekvenciába tanulás (Sequence-to-Sequence / Seq2Seq)

A Szekvencia-címkézés (7.6) korlátja az, hogy a kimenetnek pontosan ugyanolyan hosszúnak kell lennie, mint a bemenetnek (minden szóra jut egy címke).

**De mi a helyzet a Gépi Fordítással (Machine Translation)?**
Az angol *"The weather is fine"* mondat 4 szóból áll. A francia megfelelője, az *"Il fait beau"* csak 3 szóból. A chatbotoknál még durvább a helyzet: a bemenet lehet egy 5 szavas kérdés, a kimenet (a válasz) meg egy 50 szavas bekezdés.

Itt a bemenet ($X$) és a kimenet ($Y$) hossza eltérő. Erre találták ki a **Seq2Seq (Encoder-Decoder)** architektúrákat!

### Az Encoder-Decoder (Kódoló-Dekódoló) Architektúra

Ezek a modellek valójában két különálló neurális hálózatból állnak, amelyeket összekapcsolnak.

**1. Az Encoder (Kódoló):**
Az Encoder (általában egy RNN vagy LSTM) beolvassa a bemeneti mondatot (pl. az angolt) szóról szóra. Számol, emlékszik, és a legutolsó szó beolvasása után az egész mondat "értelmét" (szemantikáját) összesűríti egyetlen, tömör számsorba.

Ezt a tömörített esszenciát a gépi tanulás zsargonjában **Beágyazásnak (Embedding)** vagy **Kontextus vektornak (Context Vector)** hívjuk.

**2. A Decoder (Dekódoló):**
A Decoder egy másik hálózat, ami kizárólag ezt a tömörített Embedding vektort kapja meg. Betölti a memóriájába, és elkezd "beszélni".

Kap egy speciális `<start>` jelet, és elkezdi generálni az első francia szót. Ezt az első szót visszatáplálja saját magába, generálja a másodikat, és így tovább, amíg ki nem köp egy `<end>` (vége) jelet.

<img width="597" height="290" alt="4" src="https://github.com/user-attachments/assets/a6a9f2b8-e68a-4c94-a397-89b61a9788b5" />


>  Ez azt mutatja be, ahogy a 4 angol szó bemegy a zöld dobozokba (Encoder), amik a végén átadnak egyetlen vastag nyilat a piros dobozoknak (Decoder), hogy abból 3 francia szó szülessen.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/07%20Fejezet/Encoder-DecoderRNNSequenceFlow.html)

### Az Információs Szűkület (Bottleneck) Problémája

Az alap Seq2Seq architektúrával van egy nagy gond. Az Encodernek egyetlen, véges méretű vektorba (az Embeddingbe) kell besűrítenie egy egész mondat jelentését.

Ha a mondat rövid (4 szó), ez szuper. De ha bejön egy 50 szavas, többszörösen összetett mondat, a vektor "megtelik", a hálózat pedig elfelejti a mondat elejét, mire a végére ér. Amikor a Decoder próbál fordítani, egy zavaros, információvesztett vektorral kell dolgoznia.

### A Megoldás: A Figyelem Mechanizmus (Attention Mechanism)

Az **Attention mechanizmus** (ami a modern AI, például a ChatGPT lelke is) egy zseniális trükk: Engedjük meg a Decodernek, hogy fordítás közben "visszanézzen" az eredeti mondat bizonyos szavaira!

Ahelyett, hogy az Encoder csak a legutolsó állapotot adná át, átadja az **összes szó beolvasásakor keletkezett memória-állapotot**.
Amikor a Decoder próbálja kitalálni a megfelelő francia szót (pl. a 3. időlépésben), a Figyelem mechanizmus csinál egy gyors matematikai számítást (egy pontozást/skaláris szorzatot). Kiszámolja: *"Ahhoz a francia szóhoz, amit épp mondani akarok, az eredeti angol mondat melyik szavára kell a legjobban fókuszálnom?"*

Amikor a *"beau"* (szép) szót kell generálnia, a mechanizmus a *"fine"* szóra fog a legmagasabb "figyelem-súllyal" (attention weight) fókuszálni, a *"The"*-t pedig szinte teljesen ignorálja.

<img width="571" height="307" alt="5" src="https://github.com/user-attachments/assets/c9a4dd25-64f7-4179-9554-ad5c5cbc8eb4" />

>Ezen a rajzon már nem csak egyetlen összekötő nyíl van, hanem a Decoder minden egyes lépésénél nyilak (kapcsolatok) mutatnak vissza az Encoder összes korábbi állapotához.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/07%20Fejezet/AttentionMechanismAlignment.html)

 ## 7.8 Aktív Tanulás (Active Learning)

A felügyelt tanulás legnagyobb rákfenéje, hogy az adatok "címkézése" nagyon drága. Ha millió orvosi röntgenfelvételed van, nem ültethetsz le elé egy főorvost hetekre, hogy mindegyikre ráírja: *"Rákos"* vagy *"Egészséges"*.

Az **Aktív Tanulás** lényege: a gép csak egy maroknyi adatból (pl. 100 képből) tanul meg egy alap (buta) modellt. Ezután ráeresztjük a maradék (999 900 db) felcímkézetlen képre.

Ahelyett, hogy véletlenszerű képeket adnánk a főorvosnak, maga a gép választja ki azokat a képeket, amikben a legkevésbé biztos, és csak ezekhez kér emberi segítséget! Ezt a bizonytalanságot az **Entrópia** (lásd 3. fejezet) számításával végzi.

- Ha a gép egy képre $0.99$-es eséllyel mondja, hogy rákos, azt békén hagyja.
- Ha egy képre pontosan $0.5$ ($50\%$) eséllyel tippel (maximális bizonytalanság / entrópia), akkor azt a képet "feladja" az orvosnak (az expertnek) címkézésre. Ezt a képet utána visszatáplálja magába, újratanul, és hirtelen sokkal okosabb lesz a "szürke zónás" esetekben.

---

## 7.9 Fél-felügyelt Tanulás (Semi-Supervised Learning)

Itt is kevés a címkézett adatunk, de itt emberi segítség (orvos) nélkül akarjuk felhasználni a millió felcímkézetlen adatot.

A legzseniálisabb modern módszer erre az **Autoenkóder** (Autoencoder) és a **Létra Hálózat** (Ladder Network).

### Mi az az Autoenkóder?

Ez egy olyan neurális hálózat, ami homokóra alakú.

- **Bemenet:** egy kép (pl. egy cica, $10\,000$ pixel).
- **A hálózat középső rétege (bottleneck):** nagyon pici, mondjuk csak $10$ neuronból áll.
- **Kimenet:** A hálózatnak meg kell próbálnia újraalkotni a saját bemenetét (a cicát) pusztán a középső 10 számból!

A gépnek nincs szüksége "Kutya/Macska" címkére. Egyszerűen ráereszted 1 millió képre, ő pedig megtanulja az $x$ bemenetet $x$-ként visszaadni. Mivel a középső réteg szűkös, a gép kénytelen megérteni a kép lényegét (pl. a fülformákat, szőrt), és azt egy tömör **Beágyazásba (Embedding)** menteni.

<img width="582" height="332" alt="6" src="https://github.com/user-attachments/assets/4ca10973-06a5-4e2b-8cef-d369c39cf26c" />


>  Ez egy olyan neurális hálót ábrázol, ami középen nagyon elvékonyodik (bottleneck / embedding layer), majd újra kiszélesedik.

### A Létra Hálózat (Ladder Network)

A fentit úgy teszi még okosabbá (**Zajszűrő / Denoising Autoencoder**), hogy a bemeneti képet "összepiszkolja" véletlen zajjal (Gaussian noise), de elvárja a géptől, hogy a kimeneten a tiszta képet rajzolja újra.

Miután a gép ezen a millió felcímkézetlen adaton megtanulta, "milyen a világ" (hogy néznek ki a vonalak, élek, formák), elég neki az a maroknyi (pl. 100) címkézett adat, hogy azokat összekösse a már megtanult formákkal. Így lehetséges szinte tökéletes képfelismerő AI-t csinálni alig pár tucat címkézett képpel!

---

## 7.10 Egy-Lövéses Tanulás (One-Shot Learning)

Hagyományosan, ha be akarsz tanítani egy arcfelismerőt (pl. a telefonod feloldásához), 10 ezer képet kéne töltened az arcodról a telefonra, majd betanítani az egész hálózatot nulláról, hogy felismerje a "Te" osztályodat. Ez képtelenség.

Az **Egy-lövéses tanulás** (One-Shot Learning) lényege, hogy a gépet nem a konkrét arcokra tanítjuk be, hanem a különbségtételre. Ezt hívják **Sziámi Neurális Hálózatnak** (Siamese Neural Network - SNN).

### A Triplet Loss (Hármas Veszteség)

A gépet 3 képpel (egy Triplettel) bombázzuk egyszerre:

- **A (Anchor - Horgony):** Egy kép egy emberről (pl. Brad Pitt).
- **P (Positive - Pozitív):** Még egy kép ugyanarról az emberről (Brad Pitt, csak másik szögből).
- **N (Negative - Negatív):** Egy kép egy másik emberről (pl. Tom Cruise).

Mind a három képet átengedjük a hálózaton, és kapunk róluk egy-egy vektort (számsort).

**A gép feladata (a Költségfüggvény):**

1. Húzd a Horgony ($A$) és a Pozitív ($P$) vektorokat minél közelebb egymáshoz a térben.
2. Told a Negatív ($N$) vektort a Horgonytól minél messzebb.

```math
\max\left( \|f(A) - f(P)\|^2 - \|f(A) - f(N)\|^2 + \alpha, 0 \right)
```

> *(Az $\alpha$ egy kötelező margó: a gépnek nem elég csak kicsit távolabb tolnia a rossz arcot, legalább $\alpha$ távolságra el kell löknie magától).*

Amikor megveszed a telefont, az arc-hálózat már kész. Bepillantasz a kamerába (ez az 1 db kép, azaz az 1 lövés / One-Shot), a telefon csinál rólad egy vektort, és elmenti. Holnap bepillantasz megint, csinál egy új vektort, megnézi a távolságot a tegnapi vektortól, és mivel ez az érték kicsi, beenged. Ha én próbálom feloldani, az én arcom vektora hatalmas távolságra lesz a tiédtől, és kizár.

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/07%20Fejezet/TripletLossEmbeddingSpace.html)

---

## 7.11 Nulla-Lövéses Tanulás (Zero-Shot Learning)

Hogyan ismerhet fel egy gép egy állatot egy képen, ha életében nem látott még arról az állatról (pl. Tigrisről) képet a tanító adataiban? Még "Egy lövése" sem volt.

A trükk a **Szóbeágyazások (Word Embeddings)** használata!
A gépet rászabadítják a Wikipédiára, és megtanítják neki, hogyan "gondoljon" a szavakra dimenziókként (lásd 7.7. fejezet beágyazásait).

- **Zebra vektor:** `[Emlős: Igen, Csíkos: Igen, Színe: Fehér/Fekete]`
- **Bohóchal vektor:** `[Emlős: Nem, Csíkos: Igen, Színe: Narancs]`
- **Tigris vektor:** `[Emlős: Igen, Csíkos: Igen, Színe: Narancs]`

A képfelismerő CNN-t betanítják több ezer képen Zebrákra és Bohóchalakra. A gép megtanulja azonosítani a képen a "Csíkos", az "Emlős" és a "Narancssárga" vizuális jegyeit.

Amikor mutatsz neki egy képet egy Tigrisről (amit sosem látott), a gép "letapogatja" a képet, és a vizuális jegyekből összerak egy vektort: `[Igen, Igen, Narancs]`. Ezt a vektort beviszi a "szótárába" (Word Embedding űr), megnézi, melyik szó van a legközelebb ehhez a mintázathoz (Koszinusz hasonlósággal), és habozás nélkül rávágja, hogy: *"Ez egy Tigris!"*.
