## Zárásként: 7 Haladó Témakör a Továbbfejlődéshez

A könyv röviden felsorol 7 olyan haladó témakört, amelyek kimaradtak ebből a 100 oldalból, de ha tovább akarsz fejlődni az adatkutatói (Data Scientist) pályán, érdemes lesz utánuk olvasnod:

### 1. Témamodellezés (Topic Modeling)

A szövegelemzés (NLP) egyik legnépszerűbb felügyelet nélküli tanulási módszere. Ha van $10\,000$ címkézetlen cikked, a gép képes magától felfedezni bennük a rejtett "témákat" (pl. politika, sport, technológia), és megmondani, hogy egy adott cikk hány százalékban szól az egyikről vagy a másikról. A legismertebb algoritmusa a **Latent Dirichlet Allocation (LDA)**.

### 2. Gauss-folyamatok (Gaussian Processes - GP)

A Kernel Regresszió (7.1) egy felügyelt, fejlettebb alternatívája. Legnagyobb előnye, hogy nem csak egyetlen "tippet" (regressziós vonalat) ad vissza, hanem **Konfidencia-intervallumokat** is. Azaz megmondja: *"A jóslatom 45, de $95\%$ eséllyel valahol 40 és 50 között van a valós érték."* Matematikailag nehéz elmagyarázni, de rendkívül hasznos eszköz.

### 3. Általánosított Lineáris Modellek (Generalized Linear Models - GLM)

A klasszikus Lineáris Regresszió továbbfejlesztései, amelyek bonyolultabb (nem egyenes vonalú) összefüggéseket is képesek leírni a bemenet és a kimenet között. Maga a jól ismert Logisztikus Regresszió is egy GLM. Ha jól magyarázható, egyszerű modelleket keresel, ez a te utad.

### 4. Valószínűségi Gráfmodellek (Probabilistic Graphical Models - PGM)

Ezek olyan hálózatok (Bayes-hálózatok), ahol a csomópontok változók (pl. "Esik az eső"), az őket összekötő vonalak pedig a feltételes valószínűséget jelzik (pl. "Vizes a járda"). Képesek ok-okozati (kauzális) összefüggések feltárására is, amit a sima neurális hálók nem tudnak.

- **Hátrányuk:** Felépítésük komoly matematikai és domain-tudást igényel.
- Ide tartozik a könyvben korábban említett CRF és a Rejtett Markov-modellek (HMM) is, bár a kép- és szövegfeldolgozásban a Mélytanulás (Deep Learning) mára szinte teljesen leváltotta őket.

### 5. Markov-lánc Monte Carlo (MCMC)

Ha gráfmodellekkel dolgozol, gyakran előfordul, hogy egy olyan őrülten bonyolult valószínűségi eloszlásból kellene adatot kinyerned (mintavételezned), aminek a matematikája megoldhatatlan. Az MCMC egy olyan algoritmus-család, ami közelítő lépésekkel képes még a legbonyolultabb eloszlásokból is precízen mintát venni.

### 6. Genetikus Algoritmusok (Genetic Algorithms - GA)

Mi a teendő, ha a Költségfüggvényed nem deriválható, így a Gradiens Ereszkedés (a lejtőn való lépdelés) nem működik? Jöhet a biológia!

A gép legenerál ezer teljesen véletlenszerű megoldást (ez a **populáció**). Leteszteli őket, a legrosszabbakat megöli (**Kiválasztás**), a legjobbakat pedig "keresztezi" egymással és mutációkat rak beléjük, hogy létrehozzon egy új, okosabb generációt. Lassabb, mint a gradiens alapú módszerek, de hiperparaméterek hangolására vagy extrém furcsa függvények optimalizálására kiváló.

### 7. Megerősítéses Tanulás (Reinforcement Learning - RL)

A gépi tanulás harmadik nagy ága (a Felügyelt és a Felügyelet Nélküli mellett). Itt nincs statikus adathalmaz. Egy **Ágens** mozog egy **Környezetben**, **Akciókat** hajt végre, és ezekért **Jutalmat** vagy büntetést kap. A cél a hosszú távú jutalom maximalizálása.

> *(Ezt használják a robotikában, az önvezető autóknál, a tőzsdei botoknál, és ez az a technológia, amivel a mesterséges intelligencia legyőzte az embert Sakkban, Go-ban és videójátékokban).*
