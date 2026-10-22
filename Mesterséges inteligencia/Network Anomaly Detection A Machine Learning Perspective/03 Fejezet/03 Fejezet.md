# 3. Fejezet: Gépi tanulási módszerek áttekintése (An Overview of Machine Learning Methods)

## 3.1 Bevezetés (Introduction)

A gépi tanulás (Machine Learning - ML) olyan módszerek tanulmányozása, amelyekkel a számítógépeket tanulásra programozzuk. Bár a hagyományos, szabályalapú programozás sok feladatra jó, léteznek olyan komplex problémák, amelyeket nem lehet egyszerű "if-else" lépésekkel leírni.

Az ML négy fő problématípus esetén nyújt megoldást (hálózati példákkal):

1. **Nincsenek emberi szakértők:** Olyan összefüggéseket kell találni, amiket az emberi agy nem lát át.
   - *Hálózati példa:* Egy hatalmas adatközpont $10\,000$ szerverének szenzoradataiból (hőmérséklet, CPU terhelés, I/O várakozás) előre jelezni egy meghibásodást vagy egy rejtett APT (Advanced Persistent Threat) támadást.
2. **Vannak szakértők, de nem tudják megfogalmazni a szabályokat:** Intuitív (perceptuális) folyamatok, mint a beszédfelismerés vagy a kézírás olvasása. Az ember tudja a választ, de nem tudja algoritmizálni. Az ML példákból (bemenet-kimenet párokból) tanul.
3. **Gyorsan változó jelenségek:** A környezet és a szabályok folyamatosan módosulnak (pl. tőzsde árfolyamok). Egy hagyományos programot naponta kellene újraírni.
   - *Hálózati példa:* A hálózati forgalom (vagy a zero-day támadások) mintázata napról napra változik. Egy adaptív ML modell automatikusan frissíti a detekciós szabályait (pl. egy új Botnet viselkedése alapján).
4. **Egyedi testreszabást igénylő feladatok:** Alkalmazások, amiket minden felhasználóhoz külön kell igazítani (pl. egyedi spam szűrők).
   - *Hálózati példa:* Minden szervernek más a "normális" viselkedése. Egy webszerver forgalma nem hasonlítható egy belső adatbázis szerveréhez. Az ML képes minden egyes hosztra (host) egyedi baseline-t (alapvonalat) tanulni.

### ML vs. Más tudományágak:

- **Statisztika:** Az adatok mögötti jelenségek megértésére és hipotézisek tesztelésére fókuszál.
- **Adatbányászat (Data Mining):** Ember által értelmezhető mintázatok keresése az adatokban.
- **Pszichológia:** Az emberi tanulás mechanizmusait kutatja.
- **Gépi tanulás (ML):** A létrejövő számítógépes rendszer pontosságára és hatékonyságára (prediktív erejére) fókuszál.

### Tapasztalati (Empirical) vs. Analitikus (Analytical) tanulás:

- **Tapasztalati:** Külső tapasztalatokra (adatokra, bemenetekre) támaszkodik.
- **Analitikus:** Külső bemenet nélkül, pusztán a problématér elemzésével és önmagával játszva (pl. Tic-tac-toe iterációkkal) fejleszti magát a gép.

## 3.2 A gépi tanulási módszerek típusai

A hálózatbiztonságban a módszereket leggyakrabban felügyelt és felügyelet nélküli kategóriákba soroljuk.

- **Felügyelt tanulás (Supervised learning):**
  - A tanuló program (Inducer) címkézett adatokat (labeled examples) kap egy "tanártól".
  - Prediktív modellt épít a normál vs. anomália osztályokra.
- **Felügyelet nélküli tanulás (Unsupervised learning):**
  - Nincsenek címkék, emberi beavatkozás nélkül keres mintázatokat (pl. klaszterezés).
  - **Kritikus alapfeltevései (Assumptions):** (i) A normál esetek sokkal gyakoribbak, mint az anomáliák. (ii) Az anomáliák statisztikailag eltérnek a normál esetektől.
  - **Hálózati veszély:** Ha ezek a feltevések nem igazak (pl. a hálózat folyamatos DDoS alatt áll, így a "rossz" forgalom a többség), a modell a támadást hiszi normálisnak, ami magas téves riasztási arányt (False Alarm Rate) eredményez. Ezt gyakran mesterséges anomáliák (artificial anomalies) injektálásával próbálják javítani az adatkészletben.

> 📍 **[IDE JÖN A KÉP: Figure 3.1: Supervised learning vs. unsupervised learning]**
>
> Az ábra a felügyelt (szaggatott vonal) és felügyelet nélküli (folytonos vonal) keretrendszereket mutatja be, benne az Inducer és Class Predictor modulokkal.

## 3.3 Felügyelt tanulás: Népszerű módszerek (Supervised Learning)

A bemeneti adatokat $\langle X, Y \rangle$ párokként reprezentáljuk, ahol az $X$ elemei az adatok (pl. hálózati forgalom jellemzői), az $Y$ elemei pedig a valós címkék (pl. $0 = \text{Normál}$, $1 = \text{Támadás}$). Cél egy olyan prediktor építése, amely új, címkézetlen $X_{new}$ adatok esetén megjósolja az $Y_{new}$ címkéket.

### 3.3.1 Döntési és Regressziós Fák (Decision and Regression Trees)

A fák adatok osztályozására vagy értékek becslésére szolgáló adatstruktúrák. Egy fa csomópontokból (nodes), ágakból/élekből (edges) és levelekből (leaves) áll.

- **Csomópont (Node):** Egy vizsgált attribútum vagy jellemző (feature).
- **Él/Ág (Edge):** A csomópontban lévő teszt eredménye (egy lehetséges érték).
- **Levél (Leaf):** A végső döntési érték vagy kategória címkéje (Class label).

> 💡 **Hálózati ML Példa egy Döntési fára (IDS kontextus):**
> Gyökércsomópont: Milyen protokoll fut? $\rightarrow$ Ág 1: TCP $\rightarrow$ Belső csomópont: Cél port = 80? $\rightarrow$ Ág 2: Igen $\rightarrow$ Belső csomópont: TCP SYN flag-ek száma az elmúlt 1 mp-ben > 5000? $\rightarrow$ Ág 3: Igen $\rightarrow$ Levél: DoS Támadás.

> 📍 **[IDE JÖN A KÉP: Figure 3.2: An example decision tree]**
>
> Az ábra egy $m+2$ mélységű, $n$ objektumot tartalmazó példafát mutat be ovális levélcsomópontokkal.

**Működési elv és problémák:**

Az algoritmus az **Oszd meg és uralkodj (Divide and conquer)** elvet használja: addig bontja (partícionálja) az adatokat a tesztek alapján, amíg egy levélbe már csak azonos osztályba tartozó elemek kerülnek (vagy amíg tovább nem bontható).

- **Túltanulás (Overfitting):** Ha a fa tökéletesen megtanul minden apró, jelentéktelen zajt a kiképző adatokon (training data), élesben nagyon rosszul fog teljesíteni az új adatokon.
- **Védekezés:** Megállítási feltételek (stopping criteria) alkalmazása statisztikai szignifikancia alapján, vagy metszés (pruning) alkalmazása a már felépült fán.
- **Különbség:** A Döntési fák (Decision trees) célváltozója diszkrét / névleges (pl. Támadás vagy Normál), míg a Regressziós fák (Regression trees) célváltozója folytonos érték (pl. a hálózati késleltetés mértéke ezredmásodpercben).

#### 3.3.1.1 CART algoritmus (Classification and Regression Tree)

A CART egy robusztus, numerikus és kategorikus adatokat egyaránt kezelni képes döntési fa építő algoritmus.

**Hogyan építi fel a fát?**

"Igen/Nem" kérdések sorozatát generálja. (Pl. a könyv példája: Magasság $\ge 5$ ft?). Minden lépésnél megkeresi azt a változót és azt a küszöbértéket, amely a legjobban kettévágja (split) az adatokat úgy, hogy a kapott alcsoportokon belül a homogenitás a lehető legnagyobb legyen. Ezt minden új csomóponton megismétli.

> 📍 **[IDE JÖN A KÉP: Figure 3.3: Classification with CART]**

**A CART speciális tulajdonságai és metszése (Pruning):**

- **Regressziós fák hasítása:** Numerikus célváltozó esetén Gini hasítási szabályt (Gini splitting rule), Twoing szabályt, vagy az SRM-et (Squared Residual Minimization - négyzetes maradék minimalizálása) használja az optimális bontáshoz.
- **A maximális fa problémája:** $100+$ változó (feature) esetén a fa óriási lesz, elveszíti a költséghatékonyságát teszteléskor, és túltanul (overfitting).
- **A fa metszése (Pruning):**
  - **Minimális node méret:** Az $\alpha_{min}$ küszöbérték beállítása (általában $8\%$ és $15\%$ között). Ez kontrollálja a komplexitás és a tisztaság (impurity) közötti kompromisszumot.
  - **Kereszt-validáció (Cross validation):** A költség-komplexitás függvény (cost complexity function) használata:

```math
ME_p(T) = ME(T) + \beta(\hat{T})
```

ahol az $ME(T)$ a fa tévesztési aránya (misclassification error), a $\beta(\hat{T})$ pedig a komplexitást büntető tag. Ezzel vágják le a statisztikailag jelentéktelen ágakat.

**A CART előnyei:**
- Nem-parametrikus: nem igényli matematikai egyenletek (funkcionális formák) előzetes megadását.
- Kiválasztja a legfontosabb változókat (beépített feature selection).
- Kezeli a numerikus és kategorikus adatokat, valamint rendkívül jól bírja a kiugró értékeket (outliers).
- **Invariáns a monoton transzformációkra:** Ha a bemeneti hálózati forgalom $x$ bájtszámát lecseréljük $\log(x)$-re vagy $\sqrt{x}$-re, a fa struktúrája ugyanaz marad!
- Költséghatékony és könnyen adaptálható.

**A CART hátrányai:**
- Nagy dimenziószámú (sok feature-t tartalmazó) adatbázisoknál megfelelő metszés (pruning) nélkül nem költséghatékony.
- A túlzott metszés során értékes (érdekes) információk veszhetnek el.

#### Parametrikus vs. Nem-parametrikus Regresszió

A gépi tanulásban a regressziós modellek (így a fák is) lehetnek parametrikusak vagy nem-parametrikusak.

- **Parametrikus regresszió:** Feltételezzük a függő és független változók közötti matematikai kapcsolat formáját. Például egy lineáris egyenletet: $y = f(x) = mx + c$. A feladat csupán az $m$ és $c$ konstansok (paraméterek) becslése, hogy az egyenes a legjobban illeszkedjen a pontokra.
  - **Példa:** 4 adatpont $(1, 2), (2, 2.5), (3, 3)$ és $(4, 3.5)$. Ez tökéletesen felírható $m=0.5$ és $c=1.5$ paraméterekkel.

> 📍 **[IDE JÖN A KÉP: Figure 3.4: A scenario of parametric regression]**

- **Nem-parametrikus regresszió (Nonparametric regression):** A valós (hálózati) problémáknál a változók kapcsolata túl bonyolult, és nem írható le egyetlen globális egyenlettel. Itt a modell nem feltételez előzetes egyenletet, és nem globális paramétereket keres. Ehelyett helyi szomszédságok (local neighbors) alapján modellezi a pontokat.
  - **Módszerek:** Kernel regresszió (pl. Nadaraya–Watson kernel becslő, Priestley–Chao, Gasser–Muller). Ezek a függvények a helyi (szomszédos) adatpontok információit használják fel a becsléshez.

> 📍 **[IDE JÖN A KÉP: Figure 3.5: A scenario of nonparametric regression]**


### 3.3.2 Támogató Vektor Gépek (Support Vector Machines – SVM)

Az SVM egy rendkívül hatékony, korszerű felügyelt tanulási algoritmus, amely lineáris és nemlineáris regresszión, illetve osztályozáson alapul. Fő célja egy olyan optimális, $N$-dimenziós hipersík (hyperplane) létrehozása, amely a legszélesebb margóval (margin) választja el egymástól a különböző kategóriákat (osztályokat).

#### Matematikai háttér és Működés

Adott egy betanító adathalmaz $\langle x_i, y_i \rangle$ párokkal, ahol $x_i \in \mathbb{R}^d$ ($d$-dimenziós térben) és $y_i \in \{+1, -1\}$ (a bináris osztálycímke, pl. Támadás vagy Normál). A hipersíkot a $w$ (súlyvektor) és a $b$ (eltolás/bias) definiálja. Egy új $x$ objektum osztályozása a következő egyenlettel történik:

```math
f(x) = \text{sign}(w \cdot x + b) = \text{sign}\left(\sum_{i=1}^{n} \alpha_i y_i (x_i \cdot x) + b\right)
```

- **Támogató vektorok (Support Vectors):** Az egyenletben szereplő $\alpha_i$ a Lagrange-féle szorzó, amely az adott adatpont fontosságát mutatja. Az optimális hipersík megtalálásakor csak a hipersíkhoz legközelebb eső pontok esetén lesz $\alpha_i > 0$. Ezeket a pontokat hívjuk támogató vektoroknak, mivel kizárólag ezek határozzák meg a döntési határvonalat és a margót.
- **Nem-támogató vektorok (Nonsupport Vectors):** Az összes többi, távolabb eső pont esetén $\alpha_i = 0$. Ha ezeket a pontokat eltávolítjuk vagy újakat adunk hozzájuk, a modell és a margó egyáltalán nem változik.

> 📍 **[IDE JÖN A KÉP: Figure 3.6: $\alpha_i$ for support vectors and nonsupport vectors]**
>
> Az ábra a támogató vektorokat teli alakzatokkal, a nem-támogató vektorokat üres alakzatokkal mutatja.

#### Nemlineáris szeparálhatóság és Magfüggvények (Kernel Trick)

Ha az adatok az eredeti térben nem választhatók el egy egyenessel (lineárisan nem szeparálhatók), az SVM egy matematikai transzformációval (kernel) magasabb dimenzióba vetíti (mapping) őket. Egy kellően magas dimenzióban a mintázatok mindig lineárisan elválaszthatóvá válnak.

> 📍 **[IDE JÖN A KÉP: Figure 3.7: Optimal hyperplane and margins of a SVM]**
>
> Az ábra bemutatja az optimális hipersíkot, mint a támogató vektorok közötti merőleges felezőt, és a $P1$, $P2$ margókat.

> 💡 **Hálózati ML Példa:** Egy IDS bináris SVM-et használ. A $+1$ osztály a "Normál forgalom", a $-1$ osztály a "Port Scan". Bár a normál és a szkennelt forgalom 2 dimenzióban (pl. csomagszám és bájtméret) összefolyhat, az SVM egy magasabb (pl. 20) dimenziós térbe emeli az adatokat a TCP flag-ek, inter-arrival time és hibás kapcsolatok aránya alapján, ahol egyértelmű hipersíkkal el tudja vágni a kettőt.

## 3.4 Felügyelet nélküli tanulás (Unsupervised Learning)

Ebben a paradigmában nincsenek előzetesen felcímkézett adatok (nincs "tanár"). Az Inducer modul a bemeneti példányok belső, rejtett tulajdonságai és hasonlóságai alapján hoz létre csoportokat/profilokat.

A felügyelet nélküli adatbányászati (mining) feladatok két fő logikai típusba sorolhatók:

- **Prediktív (Predictive):** Következtetéseket von le a meglévő adatokból predikciók (jóslatok) készítéséhez.
- **Leíró (Descriptive):** Az adatok általános, alapvető tulajdonságait jellemzi.

**A három fő felügyelet nélküli feladat (Han és Kamber felosztása alapján):**

1. **Klaszteranalízis (Cluster analysis):**
   - Az adatok csoportosítása (klaszterezése) oly módon, hogy egy csoporton belül az objektumok minél inkább hasonlítsanak egymásra (magas belső hasonlóság), míg a különböző csoportokba tartozók minél jobban eltérjenek (magas csoportok közötti különbség).
2. **Kiugró értékek (anomáliák) bányászata (Outlier Mining):**
   - Olyan kis elemszámú adatobjektumok azonosítása, amelyek szignifikánsan eltérnek az adatbázis többi részétől.
   - **Alkalmazás:** Hálózati anomália detekció, pénzügyi csalások felismerése (fraud detection), ritka gének azonosítása.
3. **Gyakori mintázatok, asszociációk és korrelációk bányászata:**
   - Az adatokban gyakran együtt előforduló események (asszociációs szabályok) megtalálása.
   - **Klasszikus példa:** Piaci kosárelemzés (market-basket analysis) – ha valaki vesz termék A-t, nagy magabiztossággal (confidence) megveszi termék B-t is.

> 💡 **Hálózati ML Példa:** "Ha egy hoszt 10 másodpercen belül 50 különböző IP címhez próbál kapcsolódni a 445-ös porton (A esemény), akkor $95\%$-os valószínűséggel ezt egy 6667-es (IRC) porton történő adatátvitel fogja követni (B esemény) – ez egy Botnet féreg terjedési mintázata."

### 3.4.1 Klaszteranalízis (Cluster Analysis)

A klaszteranalízis első ismert alkalmazása John Snow 1854-es londoni kolera-térképe volt, ahol a megbetegedések sűrűségének (klaszterének) térbeli ábrázolásával rájött, hogy a fertőzés forrása egyetlen központi kút.

A hálózatbiztonságban a klaszterezés (például forgalmi adatok, DNS-kérések vagy hálózati logok csoportosítása) alapvető fontosságú. A siker nagymértékben függ az elemzett adatok típusától.

#### 3.4.1.1 Változók és adattípusok (Various Types of Data)

A valós objektumokat (és hálózati csomagokat) leíró attribútumok az alábbi típusokba sorolhatók:

- **Intervallum-skálázott (Interval-Scaled):** Lineáris skálán értelmezhető változók, ahol az intervallumoknak/különbségeknek jelentése van (pl. hőmérséklet, szélességi/hosszúsági fok).
- **Névleges / Nominális (Nominal):** Kategóriák, amelyeknek nincs sorrendisége, csupán megkülönböztetnek.
  - *Hálózati példa:* IP címek, MAC címek, dolgozói azonosítók.
- **Bináris (Binary):** Csak két állapota lehet ($0$ vagy $1$ / Jelen van vagy Nincs jelen).
  - *Hálózati példa:* TCP SYN flag be van állítva ($1$) vagy nincs ($0$).
- **Kategorikus (Categorical):** Több mint két állapotú, de sorrend nélküli változók.
  - *Hálózati példa:* Protokoll típusa (TCP, UDP, ICMP), térképszínek.
- **Sorrendi / Ordinális (Ordinal):** Kategorikus, de az állapotok értelmes sorrendbe (hierarchiába) rendezhetők.
  - *Hálózati példa:* IDS riasztási szintek (Kritikus, Magas, Közepes, Alacsony).
- **Arány-skálázott (Ratio-Scaled):** Pozitív mérések (gyakran nemlineáris/exponenciális skálán), ahol a nulla pont valós, így a különbségek és az arányok (hányszorosa) is értelmesek.
  - *Hálózati példa:* Csomagok száma (counts), bájt/másodperc, időtartam.
- **Vegyes (Mixed-type):** A fenti változótípusok keveréke egy adatrekordon belül.

**A klaszterező algoritmusok teljesítményét befolyásoló 3 fő tényező:**

1. **Dimenziószám (Dimensionality):** Az attribútumok (feature-ök) száma.
2. **Adateloszlás (Data distribution):** Lehet egyenletes (uniform), nem egyenletes vagy ferde/torz (skewed).
3. **Felbontás (Resolution):** A mintázatok láthatósága függ a skálától. Egy finom felbontáson (másodperc alapú forgalom) látható DDoS anomália teljesen eltűnhet egy durva (napi szintű) felbontás esetén.

   ### 3.4.1.2 Közelségi (Proximity) és Hasonlósági Mértékek

A klaszterezés alapja, hogy meg tudjuk mondani, mennyire hasonlít (vagy tér el) két adatobjektum.

- **Távolság (Distance / Dissimilarity - $d$):** Kvantitatív mérték, amely megmutatja, milyen messze van két objektum egymástól. Ha két objektum pontosan ugyanolyan, a távolság $0$.
- **Hasonlóság (Similarity - $S$):** Numerikus érték, amely a kapcsolat erősségét mutatja (általában $-1$ és $+1$, vagy normalizálva $0$ és $1$ között).

A normalizált hasonlóság és a normalizált távolság közötti matematikai kapcsolat egy adott $(O_i, O_j)$ objektumpár esetén:

```math
S(O_i, O_j) = 1 - d(O_i, O_j)
```

> 💡 **Hálózati ML Példa:** Két hálózati folyam (flow) távolságát mérhetjük az Euklideszi távolsággal a "csomag/másodperc" és "bájt/másodperc" dimenziókban. Minél közelebb van a két érték egymáshoz (minél kisebb a távolság), a fenti képlet alapján annál közelebb lesz az $S$ (hasonlóság) értéke az $1$-hez.

### 3.4.1.3 Klaszterezési Módszerek (Clustering Methods)

A klaszterező algoritmusok kiválasztása függ az adattípustól, a dimenziószámtól és a feladat céljától. A szakirodalom a klaszterezést 10 fő osztályba sorolja:

#### 1. Particionáló (Partitional) Algoritmusok

Ezek az algoritmusok az adatbázist pontosan $k$ (a felhasználó által előre megadott) nem átfedő (diszjunkt) klaszterre bontják. Minden objektum pontosan egy klaszterbe tartozik.

- **Centroid-alapú (Centroid based):** A klasztert az elemek súlypontjával / átlagával (mean) reprezentálja.
  - **K-means (K-közép):** A leghíresebb algoritmus. Kiválaszt random $k$ középpontot, majd minden pontot a hozzá legközelebbi középponthoz rendel. Kiszámolja az új átlagokat, amik lesznek az új középpontok. Ezt ismétli a konvergenciáig. Jól skálázódik (gyors nagy adatokon is), de érzékeny a zajra (outlierek elhúzzák az átlagot), és csak gömb alakú klasztereket talál jól. Hálózati anomália detekcióban (ahol az anomáliák extrém outlierek) gyakran elbukik emiatt.
- **Medoid-alapú (Medoid based):** A klasztert a középponthoz legközelebb eső valós adatobjektummal (medoid) reprezentálja (nem egy kitalált átlaggal).
  - **K-medoids / PAM (Partitioning Around Medoids):** Sokkal robusztusabb a zajjal és a kiugró értékekkel (outliers) szemben, mint a k-means. A konvergenciát objektumok cserélgetésével (swapping) éri el, minimalizálva az összköltséget ($S_{cost}$). Hátránya: rosszul skálázódik nagy hálózati adatbázisokon (komplexitása $O(n^2)$).
- **K-modes / K-prototypes:** Kategorikus adatok kezelésére, illetve a kettő keverékére (mixed attributes) lettek kitalálva.

#### 2. Hierarchikus (Hierarchical) Algoritmusok

Ezek a módszerek egy egymásba ágyazott, fához hasonló struktúrát (dendrogram) építenek fel. Nem kell előre megadni a $k$ klaszterszámot! Két megközelítése van:

- **Agglomeratív (Bottom-up):** Minden adatpont egy külön klaszterként indul, és a legközelebbieket lépésről lépésre összevonja (merge), amíg minden $1$ nagy klaszterbe nem kerül.
- **Divizív (Top-down):** Minden pont egy nagy klaszterben indul, és ezt bontja (split) egyre kisebbekre.

**Távolságmérés klaszterek között (Linkage metrics):**
- **Single-link:** A két klaszter legközelebbi két pontjának távolsága.
- **Complete-link:** A két klaszter legtávolabbi két pontjának távolsága (kevésbé érzékeny a zajra).
- **Average-link:** A kompromisszumos megoldás (az összes pontpár távolságának átlaga).

**Fontos hierarchikus algoritmusok:**
- **BIRCH:** Memóriahatékony. "CF-fát" (Cluster Feature tree) épít, ami tömörített statisztikákat ($0.$, $1.$, $2.$ momentum) tárol a klaszterekről. Lineáris idejű, de csak gömb alakú klasztereknél jó.
- **CURE:** A klasztert nem egyetlen középponttal, hanem több, egymástól távol eső reprezentatív ponttal írja le. Képes tetszőleges alakú klasztereket megtalálni és kiszűri a zajt, de nagy adatokon (csak mintavételezéssel működik) lassú lehet.
- **CHAMELEON:** Kétfázisú gráfalapú algoritmus, amely a klaszterek összevonásánál a belső összeköttetéseket (interconnectivity) és a közelséget is figyelembe veszi. Jobb, mint a CURE vagy a ROCK.

#### 3. Sűrűségalapú (Density-Based) Algoritmusok

Ezek az algoritmusok a klasztert a sűrű pontok halmazaként definiálják, amelyeket ritka (üres) térrészek választanak el.
Két bemeneti paraméterük van: a sugár ($\epsilon$) és a minimális pontszám a sugáron belül ($MinPts$).

- **DBSCAN:** Az egyik legismertebb ML algoritmus. Háromféle pontot azonosít: Magpont (Core object), ha van elég szomszédja; Határpont (Border); és Zaj (Noise / Outlier). Zseniális anomália detekcióra, mert minden tetszőleges alakú klasztert megtalál, és ami egyikbe sem esik bele, azt automatikusan zajnak (anomáliának) tekinti.
- **OPTICS:** A DBSCAN továbbfejlesztése, amely képes kezelni a változó sűrűségű adatokat is egy hierarchikus sűrűségi struktúra felépítésével (kiegészül a core-distance és reachability-distance fogalmakkal).
- **DENCLUE:** Matematikai befolyás-függvényekkel (influence functions) modellezi a sűrűséget. Gyors és zajtűrő.

> 📍 **[IDE JÖN A KÉP: Figure 3.8: An example of core, border and noise objects]**

#### 4. Rács-alapú (Grid-Based) Algoritmusok

Az adatteret véges számú cellára (hyper-rectangles) kvantálják, és magukat a cellákat klaszterezik (a cellán belüli pontok száma alapján), nem pedig a pontokat. Ez drasztikusan felgyorsítja a folyamatot.

- **STING:** Statisztikai információkat tárol egy hierarchikus rácsban (mean, min, max, std). Nagyon gyors, de az eredmény csak függőlegesen vagy vízszintesen határolt (diagonálisan sosem).
- **WaveCluster:** Hullám-transzformációt (wavelet transform) használ a rácson. Remekül kiszűri az outliereket, de a dimenziószám növekedésével a komplexitása exponenciálisan nő.
- **CLIQUE:** Egy-dimenziós sűrű rácsokból épít fel többdimenziós hálókat (apriori elv alapján).

#### 5. Modell-alapú (Model-Based) Algoritmusok

Feltételezik, hogy az adatok egy bizonyos matematikai modell (pl. neurális háló vagy statisztikai eloszlás) szerint generálódtak.

- **SOM (Self-Organizing Map / Kohonen-háló):** Egy egyrétegű neurális háló, amely a többdimenziós adatokat egy 2D-s rácsra (térképre) képezi le (mapping). Jól tűri a zajt, de tudni kell előre a rács méretét.

#### 6. Gráf-alapú (Graph-Based) Algoritmusok

Az objektumokat gráf csomópontjaiként, a hasonlóságokat a köztük lévő élekként (links) modellezi. A klaszterek az erősen összekötött komponensek (klikkek - cliques).

- **Példák:** CAST (Klikk gráf modell), AUTOCLUST (Delaunay/Voronoi diagramokat használ), CLICK. Erősségük az átfedő (intersecting) klaszterek kezelése.

#### 7. Klaszter Együttesek (Ensemble of Clustering)

Több különböző klaszterező algoritmus (vagy ugyanazon algoritmus több futtatása különböző kezdőértékekkel) eredményeit vonja össze (pl. szavazásos módszerrel), hogy egy sokkal pontosabb, konszenzusos klaszterezést hozzon létre.

#### 8. Elosztott Klaszterezés (Distributed Clustering)

Nagy adathalmazok esetén (pl. Big Data logelemzés 50 különböző tűzfalról) az adatokat nem küldik be egy központi szerverre, hanem lokálisan klaszterezik, és csak az eredményt (modellt) küldik be a központba összegzésre (FDC, ODC, FODC architektúrák).

- **Példa:** Parallel k-means, PDBSCAN.

#### 9. Lágy-számítási (Soft Computing-Based) Algoritmusok

A hagyományos, "kemény" (Hard) módszereknél egy adatpont pontosan egy klaszterbe tartozik. A "lágy" módszereknél tagsági értékeket (membership values) használnak (pl. a pont $70\%$-ban az A klaszterhez, $30\%$-ban a B klaszterhez tartozik).

- **Fuzzy C-Means (FCM):** A k-means elmosódott (fuzzy) logikájú változata. Jól kezeli az átfedő (overlapping) klasztereket.
- **Genetikus Algoritmusok (GA):** Evolúciós elvek (mutáció, keresztezés) alapján keresik az optimális klasztereket, de nagy a számítási költségük.

#### 10. Alttér Klaszterezés (Subspace Clustering)

Magas dimenziószámú (high-dimensional) adatoknál a távolságfogalom értelmét veszti ("Dimenziók átka"). Ezek az algoritmusok megpróbálják megtalálni azokat az altéteket (a dimenziók egy részhalmazát), ahol a klaszterek érvényesek.

- **Bottom-up (Alulról felfelé):** $1$-dimenziós rekeszekből (bins) építkezik. (pl. CLIQUE, MAFIA).
- **Top-down (Felülről lefelé):** A teljes térből indul, és a dimenziókhoz súlyokat rendel. (pl. PROCLUS, ORCLUS, COSA).

#### 3.4.1.4 Összegzés a klaszterezési módszerekről (Discussion)

A különböző felügyelet nélküli (unsupervised) algoritmusok tulajdonságainak és korlátainak megértése kritikus az IDS (Intrusion Detection System) tervezésekor:

- **Particionáló (Partitioning) algoritmusok (pl. K-means):** Konvex (gömb) alakú, hasonló méretű és sűrűségű klaszterek megtalálására jók. Fő hátrányuk: előre tudni kell a klaszterek számát ($k$), ami egy folyamatosan változó hálózati forgalomban szinte lehetetlen.
- **Hierarchikus (Hierarchical) algoritmusok:** Faszkebezetet (dendrogram) építenek, de nagy hálózati adathalmazok esetén az építés költsége (számítási kapacitás) rendkívül magas.
- **Sűrűségalapú (Density-based) algoritmusok (pl. DBSCAN):** Tetszőleges alakú klasztereket találnak, kiválóan szűrik a zajt/anomáliát. Fő hátrányuk: Teljesítményük drasztikusan függ a bemeneti paraméterektől ($MinPts$ és sugár $\epsilon$).
- **Rács-alapú (Grid-based) algoritmusok:** Sebesek alacsony dimenzióban, de ahogy nő a dimenziószám (hálózati forgalom feature-ök száma), exponenciálisan nő a komplexitásuk.
- **Altér (Subspace) klaszterezés:** Kifejezetten a magas dimenziójú (high-dimensional) adatoknál ad nagyon jó eredményeket, főleg mintavételezéssel kombinálva.
- **Klaszter együttesek (Ensemble):** Jobb eredményt adhatnak, ha ügyesen választják meg az alap-algoritmusokat és a kombinációs (szavazó) funkciót.

### 3.4.2 Kiugró értékek bányászata (Outlier Mining)

Az outlier mining célja olyan adatmintázatok azonosítása, amelyek nem felelnek meg a többi adat viselkedésének. Különböző területeken hívják ezeket zajnak, anomáliának, hibának vagy kivételnek.

> 💡 **Hálózati ML Példa:** A rendszerlogok elemzésekor talált szokatlan hálózati csomagáramlás (unusual flow of network packets) outliernek számít, és gyakran vírustámadást vagy behatolási kísérletet (intrusion) jelez.

**Az outlier detektálás alapvető feltevései és kihívásai:**

1. A vizsgált objektumok túlnyomó többsége nem outlier (a normál forgalom sokkal gyakoribb, mint a ritka támadás).
2. Gyakran magas dimenziójú (high dimensional) adatokon kell dolgozni, míg sok régi algoritmus csak alacsony dimenziót feltételez.
3. Nehéz előre (a priori) meghatározni, hogy mi számít outliernek; ezt erősen befolyásolják az alkalmazott algoritmusok rejtett feltevései.

**Módszerek típusai a címkék alapján:**

- **Felügyelt (Supervised):** Külön modellt épít a normál és az outlier osztályra a betanító adatokból (ez ritka és nehéz, mert nehéz minden támadástípusra címkézett adatot szerezni).
- **Felügyelet nélküli (Unsupervised):** Nincs címkézett adat. Csak azt feltételezi, hogy a gyakori minta a "normál", a ritka minta pedig az "outlier".

#### Az Outlier detektáló algoritmusok 3 fő kategóriája

##### 1. Távolság-alapú Outlier Detektálás (Distance-Based Outlier Detection)

Ez a megközelítés az objektumok közötti geometriai távolságot számolja ki. Létrehoz egy $F(x)$ outlier faktort (vagy küszöböt), amely számszerűsíti az anomáliát.

- **Hawkins-féle definíció:** Az outlier egy olyan megfigyelés, amely annyira eltér a többitől, hogy felmerül a gyanú: egy teljesen más mechanizmus generálta (pl. hacker vs. normál user).
- **A Knorr és Ng féle $DB(pct, d_{min})$ modell:** Egy $p$ objektum akkor outlier egy $D$ adathalmazban, ha a $D$-ben lévő objektumok legalább $pct$ (százalék) része nagyobb $d_{min}$ távolságra van a $p$-től. (Tehát nagyon kevés szomszédja van a $d_{min}$ sugarú körön belül).
- **A 2D adathalmaz példája (Figure 3.9):** Ha van egy sűrű klaszterünk ($C1$) és egy ritkább klaszterünk ($C2$), és egy magányos pontunk ($O2$) a $C2$ mellett. Ha a $d_{min}$-t úgy állítjuk be, hogy $O2$-t outliernek lássa, előfordulhat, hogy a teljes sűrű $C1$ klasztert is outliernek fogja jelölni, mert a globális távolság paraméterek nem tudják kezelni a különböző sűrűségű területeket.
- **K-adik legközelebbi szomszéd ($D_k^n$) modell:** A pontokat rangsorolják a k-adik legközelebbi szomszédjuktól mért távolságuk alapján ($D_k(p)$). Akinek a legnagyobb ez a távolsága, az a legnagyobb outlier (hiszen senki sincs a közelében).

**Értékelés (Előnyök és Hátrányok):**
- **Előny:** Magas pontosság (megfelelő távolságmérték használatával).
- **Hátrány:** Magas dimenziószám esetén (a dimenziók átka miatt) az adatok megritkulnak (sparsity), és a pontok közötti távolságok nagyon hasonlóvá válnak, így a módszer elbukik. Továbbá óriási számítási időt igényel (bár adaptív küszöbértékekkel javítható).

> 📍 **[IDE JÖN A KÉP: Figure 3.9: A 2-D dataset]**
>
> Az ábra a $C1$, $C2$ klasztereket és az $O1$, $O2$ outliereket mutatja be.

##### 2. Sűrűség-alapú Outlier Detektálás (Density-Based Outlier Detection)

Ez a megközelítés orvosolja a távolság-alapú algoritmusok hibáját (a globális paraméterek problémáját). Megbecsüli a bemeneti tér helyi sűrűségét, és az alacsony sűrűségű régiókban lévő pontokat outliernek tekinti.

- **LOF (Local Outlier Factor) algoritmus:** Nem globálisan vizsgálja az adatbázist, hanem számszerűsíti az objektum helyi (local) outlier fokát a szomszédjaihoz képest. Csak egy paramétert használ: a szomszédok számát ($MinPts$).
- **Működés:** Kiszámolja a vizsgált $p$ pont Helyi Elérhetőségi Sűrűségét (Local Reachability Density - $lrd$), és elosztja a szomszédai $lrd$-jével.
  - Ha a $p$ pont sűrűsége ($lrd$) alacsony, de a szomszédaié magas, akkor a LOF érték magas lesz $\rightarrow$ a pont egy Outlier.
- **Elérhetőségi távolság (Reachability distance):** A zajok/statisztikai fluktuációk elkerülése érdekében (smoothing effect) bevezeti ezt a fogalmat. Ha egy $p$ pont nagyon közel van egy $o$ ponthoz, a valós távolságuk helyett egy fix értéket (az $o$ pont k-távolságát) fogja használni.

**Értékelés:**
A sűrűség-alapú módszerek paraméter-függőek. Ha jól választjuk meg a paramétereket, hálózati anomália detekcióra kiválóak, mert a hálózati forgalom (bizonyos attribútumok mentén) alacsony sűrűségű területein megjelenő adatok nagy eséllyel támadásokat jelölnek.

> 📍 **[IDE JÖN A KÉP: Figure 3.10: Reachability distance]**
>
> Az ábra azt mutatja, hogyan veszik fel az $o$ ponthoz nagyon közeli $p_1$ pontok az $o$ k-távolságát elérhetőségi távolságként, míg a távolabbi $p_2$ a valós távolságát tartja meg.

##### 3. Lágy-számítás alapú (Soft Computing) Outlier Detektálás

A matematikai bizonytalanság, a homályosság és az approximáció (becslés) kezelésére lettek kifejlesztve, amit a hagyományos módszerek (ahol valami vagy "Igen", vagy "Nem") nem tudnak.

- **Durva Halmazok (Rough Sets):** Olyan osztályozó rendszerek, ahol nincs teljes tudás a rendszerről. Egy osztályba olyan tagok kerülnek, amelyek megkülönböztethetetlenek (indiscernible) egymástól. A könyv bemutatja a ROF (Rough Outlier Factor) képletét, ami a megkülönböztethetetlenségi osztályok és részhalmazok alapján számol egy outlier értéket.
- **Fuzzy Halmazok (Fuzzy Logic):** Zadeh vezette be. Kezeli az imprecizitást és az approximatív érvelést. Egy elem nemcsak $0$ vagy $1$ értékkel tartozhat egy halmazhoz, hanem egy "tagsági értékkel" (membership function, $\mu$) $0$ és $1$ között (pl. $70\%$-ig Normál, $30\%$-ig Támadás). Ez emberközeli, magas szintű szabályok leírását teszi lehetővé.

**A kettő összehasonlítása és kombinálása:**
- A **Rough sets** az egymást nem átfedő, "durva" fogalmakkal, az adatok megkülönböztethetetlenségéből (indiscernibility) fakadó bizonytalansággal foglalkozik.
- A **Fuzzy sets** az átfedő halmazok közötti homályossággal (vagueness) foglalkozik.

Mivel a hálózatokban mindkét típusú bizonytalanság jelen van, a kutatók gyakran alkalmazzák a hibrid **Fuzzy-Rough Sets** modellt.

### 3.4.3 Asszociációs Szabályok Tanulása (Association Rule Learning)

Az asszociációs bányászat célja érdekes, gyakran együtt előforduló mintázatok (szabályok) megtalálása hatalmas adatbázisokban. Eredetileg a szupermarketek tranzakciós adatainak (piaci kosárelemzés) vizsgálatára hozták létre, hogy megértsék a vásárlói viselkedést.

> 💡 **Hálózati ML / Biztonsági Példa:** A hálózatbiztonságban a "vásárolt termékek" megfelelnek a hálózati csomagok jellemzőinek (pl. Forrás port = 21, Cél port = 21, TCP flag = SYN). Ha ezek az elemek gyakran fordulnak elő együtt, a modell asszociációs szabályt generál belőlük. Ezzel automatikusan feltérképezhetők a normál hálózati profilok, vagy épp egy port scan mintázata.

#### 3.4.3.1 Alapfogalmak és Metrikák

- **$I = \{I_1, I_2, \dots, I_m\}$:** Különböző attribútumok (elemek) halmaza.
- **$T$:** Egy tranzakció, amely az $I$ egy részhalmazát tartalmazza.
- **$D$:** A tranzakciós adatbázis.
- **Asszociációs szabály:** Egy $X \rightarrow Y$ formájú implikáció (ahol $X$ az előzmény/feltétel, $Y$ a következmény). Jelentése: Ha $X$ jelen van, akkor $Y$ is jelen van.

**Két kritikus mérőszám (amelyekhez a felhasználó küszöbértékeket állít be):**

**1. Támogatottság (Support - $s$):**
Megmutatja, hogy a teljes adatbázis hány százaléka tartalmazza egyszerre az $X$ és $Y$ elemeket ($X \cup Y$).

```math
s(XY) = \frac{\text{XY-t tartalmazó tranzakciók száma}}{\text{Összes tranzakció a } D \text{-ben}}
```

- **Küszöb:** minsup (Minimum Support). Csak azokat a mintákat vizsgáljuk, amik elég sokszor fordultak elő.

**2. Megbízhatóság (Confidence - $c$):**
A szabály "erejét" mutatja. Megadja, hogy azon tranzakciók közül, amelyek tartalmazzák $X$-et, hány százalék tartalmazza $Y$-t is. (Pl. $80\%$ confidence: ha $X$ ott van, $80\%$ eséllyel $Y$ is ott lesz).

```math
\text{Confidence}(X \mid Y) = \frac{\text{Support}(XY)}{\text{Support}(X)}
```

- **Küszöb:** minconf (Minimum Confidence).

**A Szabálygenerálás 2 alapvető lépése:**
1. Megkeresni az összes gyakori elemhalmazt (frequent itemsets), amelyek elérik a minsup küszöböt. (Ez a számításigényesebb feladat).
2. A gyakori elemhalmazokból szabályokat generálni, amelyek elérik a minconf küszöböt.

**Az Asszociációs Szabályok típusai:**
- **Kezelt értékek szerint:** Logikai (Boolean) (az elem jelen van=$1$, vagy nincs=$0$) vs. Kvantitatív (numerikus értékek, pl. kor(30..39) $\rightarrow$ vesz(TV)).
- **Dimenziók száma:** Egydimenziós vs. Többdimenziós.
- **Absztrakciós szintek:** Egyszintű vs. Többszintű (pl. Laptop vásárlás $\rightarrow$ tágabb értelemben Számítógép vásárlás).
- **Kiterjeszthetőség:** Maximális gyakori mintázatok (Maxpatterns) és Zárt elemhalmazok (Closed itemsets). Ezek a technikák drasztikusan csökkentik a vizsgálandó kombinációk számát.

### 3.4.4 Gyakori Elemhalmazt Kereső Algoritmusok (Frequent Itemset Mining)

Mivel a kombinációk száma óriási lehet, az algoritmusok közötti fő különbség a hatékonyságban (memória- és CPU-használat, adatbázis-olvasások száma) rejlik.

- **AIS Algoritmus:** Csak egyetlen elemet enged a szabály következmény részében (pl. $X \cap Y \rightarrow Z$). Hátránya: Rengetegszer kell végigolvasni (scan) az adatbázist. Túl sok felesleges jelölt (candidate) halmazt generál, ami rengeteg memóriát és időt pazarol.
- **Apriori Algoritmus (Az egyik leghíresebb):** Megoldja az AIS problémáját egy okos metszési (pruning) technikával, amely a lefelé zártság elvén (downward closure property) alapul: Ha egy elemhalmaz nem gyakori, akkor annak egyetlen szuperhalmaza (bővítése) sem lehet az. Működés: Először megkeresi az 1-elemű gyakori halmazokat. Utána ezekből 2-eleműeket épít (join), de ha egy 2-elemű részhalmaza nem volt gyakori az előző körben, azonnal eldobja. Hátránya: Még mindig sok adatbázis-olvasást (pass) igényel, és bár kevesebb, de még mindig nagyon sok jelöltet generál.
- **SETM:** SQL adatbázisokra optimalizált (TID - Tranzakció ID azonosítókkal). Több olvasást igényel, de a TIDs miatt borzasztóan nagy a helyigénye (space).
- **DIC (Dynamic Itemset Counting):** Nem olvassa végig az egész adatbázist egyben, hanem "megállókban" (stops) számol, szaggatott/folytonos dobozokkal követve a halmazok állapotát. Csökkenti a scan-ek számát, de teljesítménye adatfüggő.
- **Particionáló megközelítés (Partitioning):** Két adatbázis-olvasásra (scan) csökkenti a folyamatot. A memóriában elférő blokkokra osztja az adatokat. Ami globálisan gyakori, annak legalább egy helyi partícióban is gyakorinak kell lennie.
- **FP-Growth (Frequent Pattern Tree):** Forradalmi módszer: **NINCS jelölt-generálás** (no candidate generation)! Mindössze kétszer olvassa az adatbázist. Egy memóriában lévő, tömörített fába (FP-Tree) kódolja a gyakori elemeket. Oszd-meg-és-uralkodj elvet használ. Hátránya: Hatalmas tranzakció-szám esetén a fa nem fér be a RAM-ba. Interaktív bányászatnál (ha a user megváltoztatja a minsup küszöböt), a fát a nulláról újra kell építeni.
- **Inverted Matrix:** Lemez-alapú (disk-based) módszer, amely engedi az interaktív lekérdezéseket. Egy COFI-Tree nevű struktúrát használ, minimális memóriaigénnyel és rekúrzió nélkül. Gyorsabb, mint a klasszikus FP-Tree vagy az Apriori.

### 3.4.5 Szabálygeneráló Algoritmusok (Rule Generation Algorithms)

Ha megvannak a gyakori elemhalmazok, a következő lépés a szabályok felállítása a minconf küszöb alapján.

- **Agrawal algoritmusa:** Nagyon egyszerű: egy n-elemű halmazból csak olyan szabályokat csinál, ahol a következmény pontosan 1 elem (pl. $A, B \rightarrow C$). Hátrány: Maximum $n$ szabályt ellenőriz, pedig valójában $2^n - 2$ lehetséges szabály létezik, így nagyon sokat kihagy.
- **Srikant egyszerű algoritmusa:** Minden lehetséges nem-üres részhalmazt generál, és felállítja belőle a szabályt (pl. $A \rightarrow (L-A)$). Hátrány: Bár mindent megtalál, rengeteg a redundáns számítás (nagyon lassú).
- **Srikant gyorsabb algoritmusa:** Redundancia-szűrést alkalmaz: Itt is érvényesül egyfajta "lefelé zártság". Elv: Ha egy $A \rightarrow (L-A)$ szabály igaz, akkor minden olyan szabály is biztosan igaz, ahol az előzmény bővebb (tehát a következmény egy részhalmazára mutat). Példa: Ha tudjuk, hogy $A, B \rightarrow C, D$ érvényes szabály, akkor biztosan érvényes lesz az $A, B, C \rightarrow D$ szabály is. Hátrány: Bár gyorsabb, a jelöltek memóriában tartása és az ismétlődő generálások (amikor $X$ részhalmaza $Y$-nak) még mindig pazarolják az erőforrásokat.

### 3.4.6 Összegzés és Hálózati Biztonsági Relevancia (Discussion)

A fejezet záró gondolatai rámutatnak a jelenlegi gépi tanulási és adatbányászati technológiák korlátaira:

- Az FP-Tree megreformálta a területet a jelölt-generálás elhagyásával, de a nagy memóriafogyasztás továbbra is probléma Big Data környezetben.
- Az interaktív (valós idejű) bányászatot kevés algoritmus támogatja zökkenőmentesen.

> **A LEGFONTOSABB IDS / BIZTONSÁGI KÖVETKEZTETÉS:**
> A hagyományos **Támogatottság-Megbízhatóság (Support-Confidence)** keretrendszer alapján működő asszociációs szabályok csődöt mondanak a "ritka elemek" (rare itemsets) megtalálásában.
>
> **Miért probléma ez a hálózatbiztonságban?**
> A legveszélyesebb támadások – mint a User-to-Root (U2R) vagy a Remote-to-Local (R2L) – rendkívül ritkák a teljes hálózati forgalomhoz (DDoS-hez vagy normál böngészéshez) képest. Mivel ritkák, nem érik el a minsup (Minimum Support) küszöböt, így a klasszikus algoritmusok (pl. Apriori, FP-Growth) egyszerűen "zajként" eldobják őket, mielőtt egyáltalán szabályt generálhatnának a detektálásukra!
>
> ## 3.5 Valószínűségi Tanulás (Probabilistic Learning)

A valószínűségi osztályozók (classifiers) olyan modellek, amelyek a hálózati attribútumok közötti összefüggéseket és a bizonytalanságot matematikai valószínűségekkel kezelik.

### 3.5.1 Bayes-hálózatok (Bayesian Networks - BN)

A Bayes-hálózat egy olyan modell, amely a vizsgált változók közötti valószínűségi kapcsolatokat kódolja.

- **Struktúra:** Egy Irányított Körmentes Gráf (Directed Acyclic Graph - DAG), $B = (N, A, \Theta)$.
- **$N$ (Csomópontok):** A domain változóit (hálózati feature-öket) reprezentálják.
- **$A$ (Élek/Ágak):** A változók közötti probabilisztikus függőséget mutatják.
- **$\Theta$:** Feltételes valószínűségi eloszlás minden csomóponthoz.

**Célja:** Kiszámítani egy adott csomópont feltételes valószínűségét (pl. Mi az esélye annak, hogy ez egy támadás?), ismerve a többi csomópont értékét (pl. Tudjuk a portot, a csomagméretet és az IP-t).

**Előnyei a Hálózati Anomália Detekcióban (IDS):**

- **Hiányos adatok kezelése:** Mivel ismeri a változók egymásrautaltságát, ha egy hálózati csomagból hiányzik egy mező, a modell akkor is tud dönteni.
- **Oksági (Causal) kapcsolatok:** Képes megjósolni egy esemény következményeit.
- **Előzetes tudás (Prior knowledge) beépítése:** Szakértői emberi tudás könnyen integrálható a modellbe.
- **Szenzorfúzió (Multisensor fusion):** Kruegel et al. megközelítése szerint egyetlen IDS szenzor ritkán biztos a dolgában. A Bayes-hálózat kiválóan alkalmas több különböző IDS szenzor riasztásainak összevonására, drasztikusan csökkentve a téves riasztásokat (false alarms).

> 💡 **Biztonsági kockázat:** A modell pontossága nagymértékben függ az ember által felállított kezdeti (viselkedési) feltevésektől. Ha a támadó viselkedése teljesen eltér a feltevéstől, a Bayes-hálózat vak marad.

### 3.5.2 Naiv Bayes (Naive Bayes - NB)

A Naiv Bayes egy rendkívül leegyszerűsített Bayes-hálózat.

- **Struktúra:** Csak egyetlen gyökércsomópontja van (a Szülő, azaz a keresett Osztály, pl. "Normál" vagy "Támadás"), és az összes többi változó (Gyermek csomópontok, azaz a feature-ök) közvetlenül ebből ágazik le.
- **Az "Erős" (Naiv) Feltevés:** Feltételezi, hogy a gyermek csomópontok (a hálózati jellemzők) teljesen függetlenek egymástól a szülő osztály ismeretében. (Pl. feltételezi, hogy a TCP flag és a csomagméret között nincs semmi összefüggés - ami a valóságban ritkán igaz, ezért "naiv").
- **Matematikai működés:** A Bayes-tétel alapján kiszámítja a valószínűségeket az egyes attribútumokra, majd összeszorozza őket. A bejövő (ismeretlen) forgalmat abba az osztályba sorolja, amelyiknek a legmagasabb a végső szorzata (valószínűsége).
- **Előny:** Nagyon gyors betanítani és tesztelni (számítási ideje lineáris: $O(n)$).
- **Hátrány:** Mivel a változók közötti interakciót figyelmen kívül hagyja, a bonyolult, több hálózati jellemzőt okosan kombináló támadásokat (ahol $A$ és $B$ együtt gyanús, de külön-külön nem) nehezen veszi észre.

### 3.5.3 Rejtett Markov Modellek (Hidden Markov Models - HMM)

A HMM olyan rendszerek modellezésére szolgál, amelyek időben változnak (nem-stacionárius szekvenciák), de a rendszer valós (belső) állapota rejtve van a megfigyelő elől.

> 💡 **Hálózati ML Példa:** Képzeljünk el egy többlépcsős hálózati behatolást (pl. APT támadás). Mi (az IDS) csak az "Megfigyelhető állapotokat" ($O$) látjuk a hálózaton: port scan, majd egy jelszóhiba, majd egy fájlletöltés. A rendszer valós "Rejtett állapota" ($S$) maga a támadási fázis (Előkészítés $\rightarrow$ Betörés $\rightarrow$ Takarítás). A HMM a látható jelekből ($O$) próbálja kitalálni a rejtett állapotot ($S$).

**A HMM 3 alapproblémája (amit meg kell oldani az IDS-ben):**

1. **Kiértékelés (Evaluation):** Milyen valószínűséggel generálta a modell a látott hálózati eseménysort? (Ebből lesz az anomália küszöb).
2. **Tanulás (Learning):** Hogyan építsük fel a modellt (profilokat) a logfájlokból? Erre használják az Expectation Maximization (EM) algoritmust.
3. **Dekódolás (Decoding):** Mi az a legvalószínűbb rejtett-állapot sorozat (támadási lépések), ami a látott hálózati eseményeket okozta?

### 3.5.4 Expectation Maximization (EM) Algoritmus

Az EM egy iteratív módszer a Maximális Valószínűség Becslés (MLE) problémák megoldására, különösen akkor, ha az adatok hiányosak (missing data), vagy ha rejtett (hidden) változók vannak a rendszerben (mint a HMM-nél vagy felügyelet nélküli klaszterezésnél).

**Működése (Két lépéses ismétlődés):**

1. **E-lépés (Expectation):** Kiszámolja a rejtett paraméterek várható értékét a jelenlegi "legjobb becslés" (aktuális paraméterek) alapján.
2. **M-lépés (Maximization):** Újra-kalkulálja a modell paramétereit úgy, hogy az előző lépésben várt értékek valószínűségét maximalizálja. Az új paraméterek lesznek az új "legjobb becslés".

Ezt addig ismétli, amíg a javulás meg nem áll (konvergencia egy lokális optimumba).

---

## 3.6 Lágy-Számítás (Soft Computing)

A hagyományos ("Kemény" / Hard) számítástechnika pontos ($0$ vagy $1$, Igen vagy Nem) értékekkel dolgozik. A **Lágy-Számítás** tolerálja az imprecizitást, a bizonytalanságot (uncertainty) és a részleges igazságokat (partial truth). Nem egyetlen algoritmus, hanem különböző, egymást kiegészítő technológiák gyűjtőfogalma.

### 3.6.1 Mesterséges Neurális Hálózatok (Artificial Neural Networks - ANN)

- **Működés:** Az emberi agy felépítését imitálja (neuronok és szinapszisok/súlyok). A tanulás során (pl. backpropagation algoritmus) a hálózat folyamatosan finomhangolja a súlyokat, amíg meg nem tanulja a bemenet és kimenet közötti kapcsolatot.
- **Hálózati alkalmazás:** Kiváló a már ismert támadások felismerésében (felügyelt tanulásként), valamint jellemzőkinyerésre (feature extraction) és klaszterezésre.

### 3.6.2 Durva Halmazok (Rough Sets - RST)

- **Működés:** A klasszikus halmazelmélet kiterjesztése. Akkor használják, ha nincs teljes tudásunk a rendszerről (vagueness). Az egymástól "megkülönböztethetetlen" (indistinguishable) hálózati objektumokat használja építőelemként (alsó és felső approximációs határok).
- **Hálózati alkalmazás:** Ideális szabálykivonásra (rule extraction) a rendszerhívások (system calls) szekvenciáiból. Nagyon gyors (alacsony overhead), és hatékonyan detektálja az anomáliákat a már meglévő (ismert) adatokon.

### 3.6.3 Fuzzy Logika (Fuzzy Logic)

- **Működés:** A halmazba tartozás mértéke nem csak $0\%$ vagy $100\%$ lehet, hanem bármi a kettő között (Membership function - $\mu$).
- **Hálózati alkalmazás:** Robusztus, interpolatív érvelést tesz lehetővé. A hálózati forgalom viselkedését "magas szintű", emberi nyelven leírt szabályokkal (pl. Ha a forgalom NAGYON MAGAS, és az ICMP csomagok aránya KÖZEPES...) tudja reprezentálni. Kiválóan integrálható asszociációs és klaszterező algoritmusokba.

### 3.6.4 Evolúciós Számítás / Genetikus Algoritmusok (Evolutionary / Genetic Algorithms - GA)

- **Működés:** A természetes kiválasztódás és evolúció elveit (mutáció, keresztezés / crossover, szelekció) alkalmazza. A problémát egy kromoszóma-szerű adatszerkezetbe kódolja, és véletlenszerű globális kereséssel optimalizál. A legéletképesebb (fittest) megoldások maradnak fent.
- **Hálózati alkalmazás:** A kiberbiztonságban nagyrészt optimalizációs feladatokra (pl. a legrelevánsabb hálózati feature-ök kiválasztására, vagy klaszterezés kezdőpontjainak keresésére) használják, mert számítási igénye nagyon magas.

### 3.6.5 Hangyakolónia Optimalizálás (Ant Colony Optimization - ACO)

- **Működés:** A hangyák szociális táplálékkereső viselkedésén alapul. A sikeres útvonalakon a "digitális hangyák" feromont (pheromone) hagynak, amit a többiek követnek.
- **Hálózati alkalmazás:** A 90-es évektől létező keretrendszer. Hálózatbiztonságban kiválóan használható Jellemzőkiválasztásra (Feature selection), anomália detekcióra és multi-szenzor optimalizációra.

- ## 3.7 Megerősítéses Tanulás (Reinforcement Learning – RL)

A megerősítéses tanulás célja a lehető legjobb akciósorozat (sequence of actions) megtalálása egy adott cél elérése érdekében, korábbi tapasztalatok alapján.

### Alapelvek és Működés

Az RL rendszerben egy **Aktív Ágens (Active Agent)** interakcióba lép egy bizonytalan **környezettel (Environment)**. Az ágens bizonyos állapotokban (states) akciókat (actions) hajt végre, amiért a környezettől pozitív vagy negatív **jutalmat (reward)** kap. A cél a halmozott numerikus jutalom maximalizálása.

**Kritikus tényezők:**

- **Késleltetett jutalom (Delayed reward):** Egy rossz döntés nemcsak az azonnali jutalmat rontja el, hanem a következő állapotokat is, láncreakciót indítva el. A modellnek meg kell tanulnia hosszú távon gondolkodni.
- **Próba-szerencse (Trial-and-error):** Az ágens a környezet folyamatos próbálgatásával tanul.

### A legnagyobb kihívás: Felfedezés vs. Kiaknázás (Exploration vs. Exploitation)

Az ágensnek egyensúlyoznia kell aközött, hogy a már bevált, biztos jutalmat hozó lépéseket ismétli (**exploitation**), vagy új, eddig ismeretlen szituációkat próbál ki (**exploration**), amik esetleg még nagyobb jutalommal kecsegtetnek a jövőben.

> 💡 **Hálózati ML / Biztonsági Példa:** Bár a könyv egy takarító robotot hoz fel példaként, hálózatbiztonságban az RL-t az Intrusion Prevention Systems (IPS) vagy autonóm reagáló rendszerek esetén használják.
> - **Az ágens:** A hálózati védelmi rendszer.
> - **Akciók:** Egy IP-cím tiltása, forgalom korlátozása (rate-limiting), vagy a csomagok átengedése.
> - **Jutalom:** Pozitív, ha a legitim felhasználók hálózati sebessége jó. Negatív, ha a szerver egy DoS támadás miatt leáll (túl kevés tiltás), VAGY ha a legitim felhasználókat is kizárja a rendszer (túl agresszív tiltás). Az RL ágens így megtanulja az optimális hálózati beavatkozást.

---

## 3.8 Hibrid Tanulási Módszerek (Hybrid Learning Methods)

A Hálózati Behatolásjelző Rendszerek (NIDS) fejlesztésében mind a felügyelt, mind a felügyelet nélküli módszereknek megvannak a maguk korlátai:

- **Felügyelt NIDS (Szabályalapú):** Emberi beavatkozást igényel a szignatúrák létrehozásához. Ez órákat vagy napokat is igénybe vehet, ami túl lassú a gyorsan terjedő fenyegetések (pl. hálózati férgek) megállításához.
- **Felügyelet nélküli NIDS (Anomália-alapú):** Képes felismerni az új támadásokat, de hatalmas problémája a magas téves riasztási arány (High False Alarms).

Ezeknek a problémáknak a kiküszöbölésére hozták létre a hibrid rendszereket, amelyek több módszer előnyeit ötvözik a jobb detekciós pontosság érdekében.

**A Hibridizáció 3 fő módja a NIDS fejlesztésben:**

1. **Többrétegű (Multi-layer) IDS:** Különböző ML osztályozókat raknak egymás mögé. Például az első réteg egy gyors algoritmus, ami kiszűri az ismert (specifikus) támadásokat, a második réteg pedig egy komplexebb algoritmus, ami az ismeretlen (zero-day) anomáliákat keresi a fennmaradó forgalomban.
2. **Több-ágenses (Multi-agent) keretrendszer:** Elosztott architektúra a dinamikus hálózatok valós idejű (real-time) megfigyelésére és feldolgozására.
3. **Osztályozók Együttese (Ensembles of Classifiers):** Több különböző algoritmus (pl. SVM, Random Forest, Döntési fák, Neurális Hálózatok) párhuzamos futtatása ugyanazon az adaton, majd a döntéseik összevonása egy matematikai funkcióval (kombinációs függvény) a végleges döntéshez.

---

## 3.9 Összegzés (Discussion) - A 3. fejezet kulcsfontosságú megállapításai

Ez a szekció összefoglalja a gépi tanulási módszerek hálózati anomália detekcióban betöltött szerepét:

- **Felügyelt tanulás:** Prediktív modelleket épít a "Normál vs. Támadás" osztályokra, meglévő tudás alapján. **Korlát:** A tökéletesen címkézett, "tiszta" kiképző adat (training data) a valóságban ritkán áll rendelkezésre.
- **Felügyelet nélküli tanulás:** Rendkívül hasznos, ha nincs címkézett adatunk, vagy ha óriási adatmennyiséget kellene manuálisan osztályozni (ami túl drága és lassú lenne).
- **Klaszterezés (Clustering):** Teljesítményét az adatok dimenziószáma, eloszlása és felbontása befolyásolja. A 10 különböző klaszterezési osztály közül egyik sem képes minden formájú (alakzatú) klasztert megtalálni, és sokuk elbukik a hálózati adatok magas dimenziószáma vagy a tranzakciók óriási száma miatt (nem skálázódnak).
- **Kiugró értékek bányászata (Outlier Mining):** Hasznos a rendszer zajának vagy abnormalitásainak azonosítására. Az előzetes tudás (prior knowledge) beépítése sokat javíthat a teljesítményén.
- **Asszociációs szabályok:** A Támogatottság-Megbízhatóság (Support-Confidence) keretrendszer alapján (minsup, minconf) működő algoritmusok kudarcot vallanak a ritka támadások felismerésében. Egy támadás lehet nagyon ritka (így nem éri el a minsup küszöböt), mégis végzetes.
- **Együttesek (Ensemble):** A különböző osztályozók legjobb tulajdonságainak ötvözésével javítják az IDS pontosságát (accuracy) és precizitását (precision).
- **Hibrid tanulás:** Bár a felügyelt és felügyelet nélküli módszerek kombinálása magasabb detekciós rátát eredményezhet, a nem megfelelő kombináció (inappropriate combination) óriási számítási költségekhez (késleltetéshez) vezethet, ami a valós idejű hálózati védelemnél megengedhetetlen.
