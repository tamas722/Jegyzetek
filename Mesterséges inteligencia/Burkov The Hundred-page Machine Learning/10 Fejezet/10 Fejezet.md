# 10. Egyéb Tanulási Formák (Other Forms of Learning)

## 10.1 Metrika Tanulás (Metric Learning)

Eddig minden algoritmusnál (pl. K-Means, kNN) a távolságot a klasszikus Euklideszi távolsággal (vagy Koszinusz hasonlósággal) mértük. De miért pont ezzel? Mi van, ha az adathalmazunkban bizonyos jellemzők (pl. a fizetés) ezerszer fontosabbak a távolság szempontjából, mint mások (pl. az életkor)?

Ahelyett, hogy vakon használnánk egy "gyári" matematikai képletet, megtaníthatjuk a gépnek, hogy mi a legjobb távolságmérési formula (metrika) az adott adathalmazra!

### A Matematikai Trükk

Az Euklideszi távolság négyzetének klasszikus képlete két vektor ($\mathbf{x}$ és $\mathbf{x}'$) között:

```math
d(\mathbf{x}, \mathbf{x}') = (\mathbf{x} - \mathbf{x}')^T (\mathbf{x} - \mathbf{x}')
```

Vigyünk ebbe bele egy betanítható $\mathbf{A}$ mátrixot a közepére:

```math
d_A(\mathbf{x}, \mathbf{x}') = \|\mathbf{x} - \mathbf{x}'\|_A = \sqrt{ (\mathbf{x} - \mathbf{x}')^T \mathbf{A} (\mathbf{x} - \mathbf{x}') }
```

**Mi ez az $\mathbf{A}$ mátrix?**

- Ha az $\mathbf{A}$ egy **Identitásmátrix** (csupa $1$-es az átlójában, mindenhol máshol $0$), akkor visszakapjuk a sima Euklideszi távolságot (minden jellemző ugyanannyit ér).
- Ha az $\mathbf{A}$ átlójában a második szám nem $1$, hanem mondjuk $8$, azzal a gép azt mondja: *"A 2. jellemző nyolcszor jobban büntet a távolságmérésnél, mint a többi!"*

### Hogyan tanítjuk be az $\mathbf{A}$ mátrixot?

Szükségünk van egy kis emberi (szubjektív) segítségre. Két listát kell készítenünk:

- **$S$ (Similar / Hasonló) halmaz:** Olyan adatpárok, amikről mi tudjuk, hogy hasonlók (pl. két kép ugyanarról az emberről).
- **$D$ (Dissimilar / Eltérő) halmaz:** Olyan adatpárok, amik egyértelműen különböznek.

A gép feladata (az optimalizációs probléma), hogy találjon egy olyan $\mathbf{A}$ mátrixot, amely:

1. **Minimalizálja** a távolságot az $S$ halmaz elemei között (húzza őket össze).
2. Ugyanakkor biztosítja, hogy a $D$ halmaz elemei között a távolság legalább egy $c$ konstansnál (margónál) nagyobb legyen (tolja őket szét).

> *(Ez kísértetiesen hasonlít az előző fejezetből megismert Sziámi Hálózatok / Triplet Loss logikájára, csak itt magát a távolságmérés képletét torzítjuk el!)*

---

## 10.2 Rangsorolás Tanulása (Learning to Rank)

Képzeld el, hogy a Google keresőjét programozod. Amikor valaki beírja, hogy "Gépi tanulás", kapsz $10\,000$ weboldalt, amiben szerepel ez a szó. Milyen sorrendben (rangsorban) mutasd meg őket a felhasználónak?

Ez egy klasszikus **Felügyelt Tanulási (Supervised Learning)** feladat.

- **Bemenet:** Weboldalak jellemzővektorai (pl. a hossza, mikor frissült, hányszor szerepel benne a kulcsszó).
- **Kimenet:** A "Rang" (1., 2., 3. hely) vagy egy "Relevancia pontszám".

Három fő stratégiánk van ennek a megoldására:

### 1. Pontonkénti (Pointwise) megközelítés

Ez a leglustább módszer. A rangsorolási feladatot visszabutítjuk egy sima Regresszióvá vagy Osztályozássá. A gép megnéz egyetlen weboldalt önmagában, és megpróbál neki adni egy pontszámot (pl. $8.5/10$).

**Miért rossz?** Mert a weboldalakat izoláltan vizsgálja. Ha egy Wikipédia cikk kapott egy magas pontszámot, a gép nem tudja, hogy a listában szerepel egy másik, szinte azonos Wikipédia cikk is, amit emiatt lejjebb kéne rangsorolni (hogy változatos legyen a találati lista).

### 2. Páronkénti (Pairwise) megközelítés

Itt már okosabbak vagyunk. A gépet nem arra tanítjuk, hogy adjon egy abszolút pontszámot az A weboldalra. Arra tanítjuk, hogy ha kap két weboldalt (A és B), mondja meg, hogy melyik a jobb (melyiknek kell feljebb lennie).

- **Kimenet:** $1$, ha A > B. $0$, ha B > A.

Ha a gép ezt megtanulja az összes lehetséges párra, a végén egy hagyományos "rendező" (Sorting) algoritmussal sorba tudjuk rakni a weboldalakat.

### 3. Listánkénti (Listwise) megközelítés - A csúcsragadozó (LambdaMART)

Ez a modern keresőmotorok (State of the Art) technológiája. Ahelyett, hogy a gépet a "Költségfüggvény" (pl. MSE vagy Keresztentrópia) csökkentésére tanítanánk, közvetlenül arra a metrikára tanítjuk, amit a főnök látni akar!

Keresők esetén a főnökök (és a Google értékelői) a **MAP (Mean Average Precision - Átlagos Precízió Középértéke)** metrikát nézik.

- **Precízió (Precision):** Hány releváns (hasznos) dokumentum van a visszakapott listában a kéretlen dokumentumokhoz képest.
- **MAP:** Azt is figyelembe veszi, hogy a hasznos dokumentumok mennyire vannak elöl a listán (egy 1. helyen lévő hasznos cikk többet ér, mint egy 10. helyen lévő).

A **LambdaMART** algoritmus Gradiens Boostingot (fák sorozatát, lásd 7. fejezet) használ a Páronkénti (Pairwise) logikával ötvözve.

**A zseniális trükk:** A Gradiens Boostingnál a hibát (a fák módosítási irányát, a gradienst) módosítják egy speciális szorzóval. Ha két weboldal sorrendjének felcserélése drasztikusan rontana a MAP metrikán, akkor az algoritmus "hatalmas gradienst" (büntetést) számol, így a gép sokkal jobban fog fókuszálni azokra a cserékre, amik a lista legtetején vannak!

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/10%20Fejezet/LearningtoRank(LTR)Comparison.html)

> *(A "Learning to Rank" a mai modern e-kereskedelem (Amazon), streaming szolgáltatások (Netflix) és keresőmotorok (Google) egyik legfontosabb gépi tanulási eszköze. A LambdaMART algoritmust az e-kereskedelemben a mai napig nagyon széles körben alkalmazzák).*
>
> ## 10.3 Ajánlórendszerek (Learning to Recommend)

Amikor meg akarod jósolni, hogy egy felhasználónak tetszene-e egy film, hagyományosan két úton indulhatsz el:

### 1. Tartalom-alapú szűrés (Content-based filtering)

Azt elemzed, hogy a felhasználó mit fogyaszt. Ha sok sci-fi cikket olvas, sci-fi cikkeket fogsz neki ajánlani.

- **Előnye:** Nincs szükség más felhasználók adataira.
- **Hátránya a "Szűrőbuborék" (Filter bubble):** A felhasználó bezárkózik. Sosem fog kapni egy jó vígjátékot, mert eddig csak sci-fit nézett. Csak olyan dolgokat kap, amiket amúgy is ismer.

### 2. Együttműködő szűrés (Collaborative filtering)

Azt elemzed, hogy mások mit fogyasztanak. Ha Te és János ugyanarra a 10 filmre adtatok 5 csillagot, a gép feltételezi, hogy a ti ízlésetek azonos. Ha János megnéz egy 11. filmet, amit imád, a gép azonnal ajánlani fogja neked, anélkül, hogy tudná, miről szól a film!

**A probléma a Ritkaság (Sparsity):** Ahhoz, hogy ez működjön, csinálni kell egy óriási Mátrixot: Sorok = Felhasználók (pl. 100 millió), Oszlopok = Filmek (pl. 100 ezer). Ez a mátrix szinte teljesen üres (sparse), mert te életed során jó, ha 500 filmet értékeltél a 100 ezerből. Nagyon nehéz a gépnek összefüggéseket találni egy $99.9\%$-ban nulla értékeket tartalmazó hatalmas táblázatban.

A valóságban (pl. Netflix) a modern rendszerek **Hibrid** megoldást használnak, ami a fenti kettő keveréke.

### A Ritka Mátrix problémájának megoldása

Ha ezt a hatalmas, üres mátrixot bedobod egy sima Logisztikus Regresszióba vagy egy Döntési Fába, a gép csúnyán el fog bukni. Két modern fegyverünk van az ilyen szuper-ritka adatok kezelésére:

#### 10.3.1 Faktorizációs Gépek (Factorization Machines - FM)

Képzeld el az adatokat egyetlen óriási vektorként (ahogy az 1. Ábra mutatja).

- Az első $N$ oszlop a Felhasználó (One-Hot kódolva, tehát csak egyetlen $1$-es van benne, a többi $0$).
- A következő $M$ oszlop a Film (szintén One-Hot kódolva).
- A végén lehetnek extra jellemzők (pl. Oscart nyert-e a film).

Ha klasszikus lineáris regressziót írunk, az így néz ki: $f(x) = b + \sum w_i x_i$. De ez nem veszi figyelembe a felhasználó és a film kölcsönhatását (interaction). Ha hozzáadnánk a kölcsönhatásokat ($w_{i,j} x_i x_j$), az egy $100\,000 \times 100\,000$-es új súlymátrixot jelentene, amit lehetetlen betanítani (mert szinte minden $x$ nulla).

**A Faktorizációs Gép Zsenialitása:**

Ahelyett, hogy minden egyes $(i, j)$ kölcsönhatásra egy egyedi $w_{i,j}$ súlyt tanulna, a gép **Faktorizálja** a problémát!

Minden egyes $i$ jellemzőhöz (legyen az János vagy a Titanic) hozzárendel egy nagyon pici, $k$ dimenziós **"Látens Vektort"** ($\mathbf{v}_i$).

Az új képlet:

```math
f(x) = b + \sum_{i=1}^{D} w_i x_i + \sum_{i=1}^{D} \sum_{j=i+1}^{D} (\mathbf{v}_i \cdot \mathbf{v}_j) x_i x_j
```

**Mi történt itt?** A gigantikus, betaníthatatlan $w_{i,j}$ súlymátrixot helyettesítettük a két pici látens vektor skaláris szorzatával ($\mathbf{v}_i \cdot \mathbf{v}_j$). A paraméterek száma $D(D-1)$ helyett mindössze $D \times k$ lett ($D \times k \ll D^2$).

A gép így villámgyorsan megtanulja az összefüggéseket (az üres nullákat ignorálva), és kiválóan meg tudja jósolni, hány csillagot adnál egy filmre!

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/10%20Fejezet/FactorizationMachines(FM).html)

#### 10.3.2 Zajszűrő Autoenkóderek (Denoising Autoencoders - DAE)

Az ajánlórendszerek problémáját rá lehet húzni a korábban megismert Autoenkóder neurális hálókra is! (Lásd: 7.9 és 9.4).

**Az Alapötlet:** Képzeljük el úgy a feladatot, mintha a felhasználó valójában az összes filmet értékelte volna, amit szeretni fog az élete során, csak egy "Zaj" (a valóság) letörölte a jövőbeli filmek értékelését, és csak azokat hagyta meg, amiket eddig látott.

**Hogyan tanítjuk be?**

1. Fogjuk egy felhasználó adatlapját (pl. János megnézett 50 filmet, és adott rájuk értékeléseket).
2. **Mesterséges Zaj (Denoising):** Két tucat értékelését szándékosan "kifehérítjük" (kinullázzuk), mintha sosem látta volna őket.
3. Beküldjük ezt a hiányos listát az Autoenkóderbe, és arra kötelezzük a hálózatot, hogy a kimeneten alkossa újra a teljes, 50 filmes listát!

Mivel a gép több millió felhasználónál megcsinálja ezt a "kitalálósdit", a hálózat középső szűkülete (bottleneck) kénytelen lesz megtanulni az emberi ízlések globális logikáját.

**Hogyan Ajánlunk vele (Predikció)?**

Ha betanult a modell, egyszerűen fogjuk János valódi, jelenlegi listáját (a megtekintett 50 filmmel). Beküldjük az Autoenkóderbe. A gép nem csak az 50 filmet fogja visszaadni a túloldalon, hanem a tanult "ízlés-logika" alapján az üres (nulla) helyekre is tesz magas pontszámokat!
Ezek az új, gép által generált magas pontszámok lesznek azok a filmek, amiket másnap ajánlunk Jánosnak.

> *(Bónusz Neurális Hálózat: A könyv utolsó bekezdése egy egyszerű, két bemenetes FFNN-t is említ, ahol az egyik ágon bemegy a Felhasználó One-Hot vektora, a másikon a Film One-Hot vektora, a végén pedig a neurális hálózat egy ReLU függvényen keresztül kiköp egy 1-től 5-ig terjedő csillagos értékelést. Ez a Netflix-féle megoldások egyik alapköve).*
>
> ## 10.4 Önszabályozó Tanulás (Self-Supervised Learning): Szóbeágyazások

A 7. fejezetben (ZSL) már láttuk a varázslatot: a gép úgy kezeli a szavakat, mintha egy többdimenziós tér koordinátái lennének. Ha a "Kutya" és a "Farkas" vektorai egymás mellett vannak a térben, a gép "tudja", hogy a két szó jelentése hasonló.

De honnan veszi a gép ezeket a vektorokat? **Megtanulja őket a nyers, címkézetlen adatokból (Önszabályozó Tanulás)!**

Ennek a leghíresebb algoritmusa a Google által fejlesztett **Word2Vec** (azon belül is a Skip-gram nevű verzió).

### A Skip-gram modell logikája (Emberi nyelven)

A hagyományos gépi tanuláshoz kell egy adathalmaz és kellenek drága, ember által írt címkék ($y$). Az Önszabályozó Tanulás zsenialitása, hogy a gép **saját magának gyárt címkéket** az adatokból!

Nézzük a következő mondatot:
*"I almost finished reading the book on machine learning."* (Majdnem befejeztem a könyv olvasását a gépi tanulásról).

1. **A Cenzúra:** A gép fogja a mondatot, és kivesz belőle egy szót (a központi szót), mondjuk a `book` (könyv) szót. Helyettesíti egy ponttal (`·`).
2. **Az Ablak (Window):** A gép kijelöl egy ablakot a lyuk körül (pl. mérete = 7, azaz 3 szó előtte, 1 lyuk, 3 szó utána).
3. **A kapott "Skip-gram":** `[finished, reading, the, ·, on, machine, learning]`
4. **A Feladat:** A gép feladata kitalálni, hogy milyen szavak vették körül a lyukat!

> **Megjegyzés:** A könyv ezt a verziót mutatja be, de a Word2Vec Skip-gram hivatalosan úgy működik, hogy a központi szóból (book) kell megjósolni a környező szavakat (finished, reading...). A fordítottja, amikor a környező szavakból jósoljuk meg a lyukat, a **CBOW** (Continuous Bag of Words) algoritmus. A könyv kissé elmossa a határt, de a lényeg ugyanaz: a kontextus és a szó egymást jósolják.

### Miért tanulja meg ebből a jelentést?

Képzeld el, hogy a gép ezerszer látja a Wikipédián ezeket a környezeteket:

- *"Befejeztem a ___ olvasását."* $\rightarrow$ A lyukba kerülhet: **könyv, cikk, folyóirat**.

Mivel ez a három szó (könyv, cikk, folyóirat) folyamatosan ugyanolyan matematikai kontextusban jelenik meg (a környező szavaik egyformák), a hálózat kénytelen lesz ezeknek a szavaknak nagyon hasonló belső reprezentációt (súlyokat) adni a szűkületben, hogy a kimenet jól működjön. Így kerülnek egymás mellé a 300 dimenziós térben!

### A Hálózat Architektúrája (Word2Vec)

A modell valójában egy nagyon egyszerű, egyetlen rejtett réteggel rendelkező Neurális Hálózat (Egy sekély Autoenkóder):

1. **Bemeneti réteg:** Egy $10\,000$ méretű One-Hot vektor. (A szótár $10\,000$ szóból áll. Mindenhol $0$ van, kivéve annál az egy szónál, amit vizsgálunk, ott $1$-es van).
2. **Beágyazó réteg (Embedding Layer):** Ez a szűkület (bottleneck). Csak $300$ neuronból áll. Nincs aktivációs függvény (csak sima mátrixszorzás). Amikor a bemeneti $1$-es aktiválja ezt a réteget, kiválaszt egy $300$ hosszú számsort. Ez a $300$ hosszú számsor lesz maga a megkeresett **Word Embedding (a Szóvektor)**!
3. **Kimeneti réteg:** Újra $10\,000$ méretű réteg. A végén egy Softmax függvény van. Ez százalékos valószínűségeket ad arra, hogy a szótár $10\,000$ szava közül melyik szerepelhet a bemeneti szó környezetében (pl. $12\%$ eséllyel a "reading", $0.001\%$ eséllyel a "krumpli").

A gép hetekig olvassa a Wikipédiát, állítgatja a 300 neuront (a súlyokat) a **Negatív Log-Likelihood** költségfüggvény alapján, míg végül a szűkületben kialakul egy tökéletes, matematikailag értelmezhető szótár. Ezt a 300 dimenziós szótárat utána bárki letöltheti, és használhatja a saját NLP (Natural Language Processing) projektjeihez.

<img width="587" height="567" alt="2" src="https://github.com/user-attachments/assets/50782b63-992e-43d3-9f12-b49718808558" />


>  Ez mutatja bal oldalon a központi szó $10\,000$ méretű One-Hot vektorát (Bemenet), középen a szűk $300$ méretű Beágyazó réteget, jobb oldalon pedig a 4 darab $10\,000$ méretű kimeneti vektort (a kontextus szavakat, amiket a gép megpróbál eltalálni).

👉 [Kattints ide az interaktív HTML widget megtekintéséhez]( https://rawcdn.githack.com/tamas722/Jegyzetek/refs/heads/main/Mesters%C3%A9ges%20inteligencia/Burkov%20The%20Hundred-page%20Machine%20Learning/10%20Fejezet/Word2VecSkip-GramArchitecture.html)

## Végül a sebesség (Hierarchikus Softmax)

A könyv utolsó bekezdése megjegyzi: egy 10 000 (vagy valójában inkább 1 millió) méretű szótáron futtatni a Softmax függvényt minden egyes szónál pokolian lassú. A Google mérnökei zseniális trükköket találtak ki ennek gyorsítására: a Negatív Mintavételezést (Negative Sampling) (ahol nem az összes szót büntetik, csak kiválasztanak 5 random "rossz" szót) és a Hierarchikus Softmaxot (ami az 1 millió szót egy bináris fába rendezi, így 1 millió számolás helyett csak 20 lépés kell az elágazásokon).
