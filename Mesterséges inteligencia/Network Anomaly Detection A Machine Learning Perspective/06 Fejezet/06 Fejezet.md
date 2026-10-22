# 6. Fejezet: A Hálózati Anomália Detekció Megközelítései (Approaches to Network Anomaly Detection)

Ez a fejezet a hálózati anomáliák felismerésére szolgáló legfőbb gépi tanulási és statisztikai módszereket mutatja be, kitérve az erősségeikre, gyengeségeikre és a kutatási kihívásokra.

## 6.1 Hálózati Anomália Detekciós Módszerek

A legtöbb módszer úgy működik, hogy egy mögöttes "normál" forgalmi modelltől vett eltéréseket (deviations) keresi. A modelleket ideális esetben normál, támadásmentes forgalmi mintákon (traces) tanítják be egy hosszabb időszakon keresztül.

> 💡 **A gyakorlati probléma:** Szinte lehetetlen garantálni, hogy egy betanító adathalmaz $100\%$-ig támadásmentes (tiszta) legyen. Ha van benne rejtett támadás, a modell azt "normálisként" fogja megtanulni. Ezen felül a hálózat folyamatosan változik, így a betanítást (training) időről időre meg kell ismételni.

A detektálás általános 4-lépéses folyamata:

1. Hálózati forgalmi adatok gyűjtése (Data Collection).
2. A releváns paraméterek vagy jellemzők (features) kinyerése és kiválasztása.
3. A befogott és előfeldolgozott adat monitorozása, elemzése és osztályozása (Classification: Normál vagy Anomália).
4. A megközelítés (azaz az elért pontosság) validálása.

### 6.1.1 Követelmények (Requirements)

Egy anomália-alapú NIDS-nek (ANIDS) szigorú feltételeknek kell megfelelnie a valós idejű, éles hálózati működéshez:

- Képesnek kell lennie magas pontosságú detektálásra minimális vagy semmilyen előzetes tudás (prior knowledge) nélkül. Folyamatosan tanulnia kell a megfigyelésekből.
- Konzisztens teljesítményt kell nyújtania különböző hálózati forgatókönyvekben, minimális téves riasztással (false alarms).
- A bemeneti paraméterek (features) számának a lehető legalacsonyabbnak kell lennie a gyorsaság miatt.
- Nemcsak a hatalmas, "zajos" támadásokat (bursty attacks, pl. DoS), hanem a ritka, nagyon óvatosan indított (carefully launched), rejtőzködő támadásokat is észre kell vennie.
- Az ismert támadások mellett fel kell ismernie az eddig sosem látott (unknown / zero-day) anomáliákat is.

## 6.2 A Hálózati Anomália Detekciós Módszerek Típusai

A rendelkezésre álló előzetes tudás, a használt mechanizmus és a teljesítmény módja alapján a módszereket hat (6) fő kategóriába soroljuk:

1. **Felügyelt (Supervised) tanulás**
2. **Felügyelet nélküli (Unsupervised) tanulás**
3. **Valószínűségi (Probabilistic) tanulás**
4. **Lágy-számítási (Soft computing) módszerek**
5. **Tudásalapú (Knowledge-based) rendszerek**
6. **Hibrid (Hybrid) rendszerek**

> 📍 **[IDE JÖN A KÉP: Figure 6.1: Classification of anomaly-based intrusion detection methods]**

## 6.3 Anomália Detekció Felügyelt Tanulással (Supervised Learning)

A felügyelt tanulás feltételezi, hogy rendelkezünk címkézett tanító adathalmazzal (Normál vs. Ismert Támadás osztályokkal). A cél egy prediktív osztályozó modell (class predictive model) felépítése. Két fő problémája van:

- Az adatok erősen kiegyensúlyozatlanok (sokkal több a normál, mint a támadás).
- A pontos, megbízható címkék megszerzése (főleg a támadásokhoz) rendkívül nehéz. Ezt gyakran mesterséges anomáliák beinjektálásával próbálják orvosolni a tanító halmazban.

A felügyelt statisztikai módszerek két alapvető típusra bonthatók: **Parametrikus** és **Nem-parametrikus**.

### 6.3.1 Parametrikus Módszerek (Parametric Methods)

A parametrikus módszer feltételezi, hogy a normál adatok egy konkrét matematikai/statisztikai eloszlást (pl. Gauss/Normál eloszlás) követnek, aminek vannak előre definiált paraméterei ($\Theta$).

Egy $p$ hálózati esemény anomália pontszámát ezen eloszlás sűrűségfüggvényének (pdf) inverze adja meg. Alternatívaként a modell egy **Statisztikai Hipotézis Tesztet** hajt végre: a Null-hipotézis ($H_0$) az, hogy a $p$ adatpont "normális". Ha a teszt elveti a $H_0$-t, a csomag anomália.

**Gyakorlati Hálózati Alkalmazás (A $\chi^2$ / Chi-négyzet Teszt):**
Mivel a NIDS-nek nagysebességű hálózatokban óriási adatmennyiséget kell valós időben (minimális késleltetéssel) kezelnie, nagyon gyors többváltozós (multivariate) módszerekre van szükség.
A Chi-négyzet ($\chi^2$) teszt egy kiváló távolságmérő a megfigyelt ($X_i$) és a várható ($\bar{X}_i$) értékek között:

```math
\chi^2 = \sum_{i=1}^{n} \frac{(X_i - \bar{X}_i)^2}{\bar{X}_i}
```

**Hogyan detektál?** A statisztikában meghatározzák a 3-szigmás kontroll határokat ($X^2 + 3S_X^2$). Ha egy bejövő hálózati eseményből számolt $\chi^2$ érték meghaladja ezt a felső határt, az eseményt azonnal támadásnak (anomáliának) bélyegzi.

#### Táblázat 6.1: Statisztikai Parametrikus Anomália Detekciós Módszerek

| Módszer | Leírás |
| :--- | :--- |
| **Keverék modell (Mixture model)** | (i) Keverék modell az anomália detekcióhoz, "normál" adatokon való betanítás nélkül. (ii) Megbecsüli a valószínűségi eloszlást az adatokban, és statisztikai teszteket alkalmaz az anomáliák észlelésére. (iii) UNIX rendszerhívás-mintákon (system call traces) értékelték ki. |
| **Statisztikai modell (Statistical model)** | (i) Egy automatikus, adaptív, proaktív és hierarchikus többrétegű (multitier) többablakos (multiwindow) módszer. (ii) Vezetékes és vezeték nélküli ad-hoc hálózatokra is alkalmazható. (iii) A DARPA98 adathalmazon és valós időben is tesztelték. |
| **ScanAID technika** | (i) A rendszerhívások (system calls) szekvenciáit absztrahálja egy privilegizált folyamat normál viselkedésének modellezésére; N-gram vektoros anomália értékeket használ. (ii) Az MIT rendszerhívás audit nyomaival (audit trails) értékelték ki. |
| **ARMA modell** | (i) "Frekvenciatartományú" (frequency domain) forgalomelemzést és szűrést használ. (ii) A forgalmat egy alapvonalra (baseline), amely alacsony frekvenciájú/kitörésű (burstiness), és egy rövidtávú forgalomra (a legdinamikusabb rész) választja szét. (iii) OPNET szimulátorral tesztelték. |
| **CAD modell** | (i) Általános célú feltételes (conditional) anomália detekciós módszer. (ii) Három különböző Expectation-Maximization (EM) algoritmust mutat be. (iii) Több mint 13 UCI adatbázison tesztelték. |
| **Elosztott Rejtett Markov Modell (HMM)** | (i) Kontroll-elméleti HMM stratégia az elosztott csomópontokon végzett megfigyelésekhez. (ii) Egy elosztott HMM motort (engine) foglal magába, amely egy véletlenszerűen választott monitor csomóponton fut. |
| **Hálózati jelmodell (Network signal model)** | (i) A wavelet-approximációt és a rendszer-azonosítási elméletet (system identification theory) kombinálja. (ii) 15 jellemzőt használ bemeneti "jelként" (signal). (iii) A DARPA99 adathalmazon értékelték, magas találati rátát ért el. |
| **G-Hidden Markov modell** | (i) Egy mezőválasztási (field selection) technika Genetikus Algoritmussal (GA), amely csomag-alapú asszociációs szabályokat generál. (ii) Gauss megfigyelési HMM-et használ a rejtett kapcsolatok kihasználására. (iii) Az MIT Lincoln Lab adathalmazon tesztelték. |
| **$\alpha$-stabil modell** | (i) Statisztikai hipotézis tesztelésen és nem-korlátozott $\alpha$-stabil elsőrendű modellen alapul. (ii) Publikus és valós idejű forgalmi adatokkal is tesztelték. |

### 6.3.2 Nem-parametrikus Módszerek (Nonparametric Methods)

A nem-parametrikus statisztikai módszereknél nincs előre (a priori) definiált eloszlási modell (nem feltételezi, hogy az adat pl. Gauss-eloszlású). A modell magából a nyers adatból épül fel (tailored to the data), így sokkal kevesebb előfeltételezést tartalmaz.

- **Hisztogram:** A legegyszerűbb frekvencia- (vagy számlálás-) alapú módszer. Többváltozós adatoknál minden egyes attribútumra hisztogramot épít (pl. PHAD és ALAD rendszerek). Amikor jön egy új csomag, megnézi, hogy a csomag jellemzője (pl. hossza) milyen magas hisztogram-oszlopba (bin height) esik. Ha nagyon alacsonyba (mert ritka), akkor magas anomália pontszámot kap.
- **Adaptív viselkedés ($k$-NNG / 1-Nearest Neighbor Graph):** Az adatok $[0, 1]$ intervallumba képezhetők le. Az algoritmus összeköti az adatpontokat a legközelebbi $k$ szomszédjukkal. A tesztpont (a bejövő csomag) anomália, ha a gráfban elfoglalt viszonylagos "koncentrációja" egy megadott $\beta$ hibahatár alá esik (tehát nagyon távol van minden más ismert ponttól).

#### Táblázat 6.2: Statisztikai Nem-parametrikus Anomália Detekciós Módszerek

| Módszer | Leírás |
| :--- | :--- |
| **IntruShield rendszer** | (i) Adaptív statisztikai detekciós rendszer. (ii) Statisztikai használati profilalkotást (profiling) alkalmaz, folyamatosan módosítva a "normál" alapvonalat (baseline). (iii) Két adatsort tart karban: egy hosszú-távú és egy rövid-távú használati profilt. (iv) A kettő összehasonlításával jelenti a jelentős devianciákat (potenciális támadásokat). |
| **Multichart CUSUM** | (i) Multichart (Többdiagramos) CUSUM algoritmus a DoS támadások észlelésére. (ii) Minimális információt használ a változás előtti és utáni forgalmi modellekről. |
| **Parzen-Window & KNN** | (i) Hibrid jellemzőkiválasztó módszer optimalizált k-means klaszterezéssel. (ii) Parzen ablak (Parzen window) és k-legközelebbi szomszéd (KNN) módszerekkel értékelték ki a teljesítményét. |
| **Elosztott SAD** | (i) Egy elosztott anomália detekciós módszer parametrikus és nem-parametrikus variánsokkal is. (ii) Az egyéni észlelés részletes vizsgálata nem volt célja a kutatásnak. |
| **Markov lánc modell** | (i) Egy szenzor-hálózat normál viselkedését jellemzi Markov modellekkel. (ii) A múltbeli adatokból (traces) egy "anomáliamentes valószínűségi törvényt" állít fel; ha ettől eltérést talál, anomáliaként jelenti. |
| **Adaptív CUSUM** | (i) Egy adaptív CUSUM módszer hálózati behatolások detektálására. (ii) Forrás-végpont (source-end) alapú detektálást jelent a SYN flood támadások ellen. |

### A Parametrikus és Nem-parametrikus módszerek Előnyei és Hátrányai

**(i) Előnyök:**
- Képesek megtanulni a rendszer elvárt viselkedését puszta megfigyelésből.
- Pontos értesítést adnak a hosszú távon (long periods) történő rosszindulatú eseményekről (megfelelő küszöbérték beállítása esetén).
- Az "hirtelen változások elméletére" (abrupt changes) építenek: hosszan figyelik a hálózatot, és azonnal riasztanak, ha valami (pl. csomagszám) hirtelen kiugrik.

**(ii) Hátrányok:**
- **Sebezhetőség (Poisoning):** Egy okos támadó lassan, fokozatosan betaníthatja a modellt arra, hogy a támadó forgalma "normálisnak" minősüljön.
- **Küszöbök beállítása:** A paraméterek hangolása rendkívül nehéz (mindig egyensúlyozni kell a False Positive és a False Negative között). Ráadásul a parametrikus módszereknél egy kőbe vésett statisztikai eloszlást feltételeznek, pedig a hálózati forgalom sokszor nem követi azt (nem kvázi-stacionárius).
- **Lassú indítás:** Sok időbe telik, amíg az IDS életében először egy megbízható modellt felépít.
- **Komplexitás (Dimenziók átka):** Magas dimenziójú adatokra statisztikai hipotézistesztet alkotni borzasztó nehéz.
- **Hisztogramok hiányossága:** Bár pofonegyszerűek, a többváltozós adatoknál a hisztogramok képtelenek felismerni az attribútumok közötti interakciókat (pl. ha a portszám önmagában normális, és a csomagméret is normális, de együtt a kettő a támadás jele).

- ## 6.4 Anomália Detekció Felügyelet Nélküli Tanulással (Unsupervised Learning)

Tökéletesen címkézett adathalmazt előállítani – amely a hálózati viselkedés minden típusát lefedi – rendkívül drága és lassú folyamat, mivel magasan képzett biztonsági szakemberek manuális munkáját igényli. Míg a "normál" forgalmat viszonylag könnyű rögzíteni, a támadások folyamatosan változnak (dinamikusak), így az új (zero-day) anomáliákhoz lehetetlen előre címkézett tanítóadatot szerezni.

A felügyelet nélküli (unsupervised) anomália detekció nem igényel tanítóadatot. Bemenetként nyers, címkézetlen forgalmat kap, és megpróbálja megkeresni a benne megbúvó (lurking) behatolási kísérleteket. A felügyelet nélküli módszerek által megtalált új támadásokat később fel lehet használni a felügyelt (szabályalapú) rendszerek betanítására.

### 6.4.1 Klaszterezés-alapú Anomália Detekciós Módszerek

A klaszterező algoritmusok a hálózati adatokat hasonlósági vagy távolsági metrikák alapján csoportosítják. A folyamat általában úgy indul, hogy kijelölnek egy reprezentatív pontot minden klaszterhez, majd az új adatpontokat ahhoz a klaszterhez sorolják, amelynek a középpontjához a legközelebb esnek. Hatalmas előnye, hogy a rendszergazdáknak nem kell előre definiálniuk a támadási osztályokat.

#### Yang et al. megközelítése (Klaszterezés + Osztályozás)

> 📍 **[IDE JÖN A KÉP: Figure 6.2: Clustering-based anomaly detection]**

A modell a tanító adatokon klaszterezést hajt végre. Bizonyos klasztereket "ismert támadásként", másokat "normálként" profiloz előre megadott kritériumok alapján. Azokat a pontokat, amelyek nem kerültek be ezekbe a profilokba, egy új osztályozó (classifier) betanítására használja. Éles teszteléskor egy befolyás-alapú (influence-based) algoritmus dönti el a hálózati forgalomról, hogy normál vagy anomália.

#### UNADA (Unsupervised Network Anomaly Detection Algorithm) - Casas et al.

A UNADA egy tudásfüggetlen (knowledge-independent) megközelítés, amely altér-sűrűséget (subspace density) használ a klaszterek és a kiugró értékek (outliers) megtalálására. Több, az eredetinél alacsonyabb dimenziójú klasztert hoz létre, majd ezeket kombinálva rangsorolja a hálózati forgalmat.

> 📍 **[IDE JÖN A KÉP: Figure 6.3: Conceptual framework of UNADA]**

1. **Aggregáció és Időablakok:** A hálózati forgalmat fix hosszúságú időablakokban (time slots) rögzíti, és IP folyamokká (flows) aggregálja különböző kulcsok alapján (durva és finom felbontásban). Idősorokat (time series) épít a metrikákból (bájtok, csomagok, folyamok száma).
2. **Változásérzékelés (Change Detection):** Minden új időablak érkezésekor megvizsgálja az idősor-grafikonokat. Ha szignifikáns eltérést (deviációt) talál, azt az időablakot megjelöli (flag).
3. **Klaszterezés a megjelölt ablakokban:** A megjelölt időablakokban lévő IP folyamokat felügyelet nélküli vizsgálatnak veti alá. Kétféle anomáliát keres:
   - **1-az-N-hez (1-to-N):** Egy forrás IP rengeteg különböző cél IP-vel kommunikál. (Hálózati példa: Portscan vagy hálózat-felderítés). Ezt a Forrás IP kulcs emeli ki.
   - **N-az-1-hez (N-to-1):** Rengeteg különböző forrás IP támad egyetlen cél IP-t. (Hálózati példa: DDoS támadás). Ezt a Cél IP kulcs emeli ki.
4. **Outlier rangsorolás:** A rendszer homogén klasztereket képez a folyamokból. Amelyik folyam nem illik egyik klaszterbe sem, az kiugró érték (outlier). Ezeket a deviancia mértéke szerint rangsorolja, és a lista élén lévőket anomáliaként jelöli meg egy küszöbérték alapján.

### 6.4.2 Anomália Detekció Kiugró Érték Bányászattal (Outlier Mining)

A kiugró értékek (outliers) olyan objektumok, amelyek nem felelnek meg a többségi adat elvárásainak és szabályainak. Klaszterezési szempontból az outlier az a pont, amely a klasztereken kívül esik. Az IDS rendszerekben ezek az outlierek maguk a támadások.

A kiugró értékeket kereső módszereket három típusra bontjuk:

- **Távolság-alapú (Distance-based):** Geometriai mértékeket használ (pl. a LOADED algoritmus vegyes attribútumokra).
- **Sűrűség-alapú (Density-based):** A szomszédos objektumok sűrűségét méri. Ha egy pont alacsony sűrűségű területen van, az outlier. (pl. LOF - Local Outlier Factor algoritmus).
- **Lágy-számítási (Soft computing):** Kategorikus és folytonos adatokat, valamint a köztük lévő térképezést vizsgálja (pl. az ODMAD algoritmus a koszinusz hasonlóság segítségével).

#### A MINDS (Minnesota Intrusion Detection System) Architektúrája

A MINDS a kiugró értékek bányászatát használja az ismeretlen (zero-day) támadások azonosítására, egy visszacsatolásos, hibrid rendszerben.

> 📍 **[IDE JÖN A KÉP: Figure 6.4: Architecture of the MINDS system]**

**A MINDS működési folyamata:**

1. **Jellemzőkinyerés és Összegzés:** Kinyeri a releváns hálózati jellemzőket, majd időablakok alapján statisztikai összegzést (summary) készít.
2. **Ismert támadások szűrése:** A forgalom áthalad egy Szignatúra/Ismert támadás detektoron. Ha egy kapcsolat megfelel egy meglévő szabálynak (pl. ismert féreg vagy SQL injection), a rendszer riaszt, és eltávolítja a csomagot a további elemzésből.
3. **Anomália detekciós modul (Outlier Mining):** A maradék, eddig ismeretlen forgalmat a LOF (Local Outlier Factor) algoritmussal vizsgálja meg. Minden kapcsolat kap egy sűrűség-alapú anomália pontszámot. A magas LOF értékű kapcsolatok outlierek.
4. **Emberi visszacsatolás (Analyst Feedback):** Egy biztonsági elemző (human analyst) megvizsgálja a legmagasabb pontszámot kapott kapcsolatokat. Eldönti, hogy ezek valódi támadások-e, vagy csak szokatlan, de legitim viselkedések.
5. **Szabálygenerálás:** Ha a kapcsolat valódi támadás volt, az elemző visszacsatolása alapján új szignatúra készül belőle, amely bekerül az "Ismert támadások szűrése" modulba, így a jövőben a rendszer ezt már azonnal blokkolni tudja.

6. ### 6.4.3 Anomália Detekció Asszociációs Bányászattal (Association Rule Mining - ARM)

Az asszociációs szabálybányászat egy olyan adatbányászati technika, amely együtt előforduló eseményeket (mintázatokat) keres.

- **Formája:** $A \rightarrow B$, ahol $A$ és $B$ a tranzakciós adatbázis attribútumai. $A$ az előzmény (antecedent), $B$ a következmény (consequent).
- **Paraméterek:** Két küszöbérték határozza meg a szabályt: a Támogatottság (Support - $m\%$) és a Megbízhatóság (Confidence - $t\%$). (Lásd korábban a 3. fejezetnél).
- **Szerepe az Anomália Detekcióban:** Az ARM segítségével felépíthetők a normál forgalom profiljai, vagy összegzéseket készíthetünk az észlelt anomáliás kapcsolatokról.

#### Az ADAM (Audit Data Analysis and Mining) Rendszer (Daniel et al.)

Az ADAM ötvözi az ARM-et és a klasszifikációt a TCPdump audit adatok elemzéséhez.

1. **Működése (Training fázis):** Támadásmentes időszakokban felépíti a "normál" gyakori elemhalmazok tárházát (repository). Elkészül a felhasználók és a rendszer normál viselkedésének profilja, ami alapján betanítják az osztályozó motort (Classification Engine).
2. **Működése (Detection fázis):** Egy csúszó-ablakos (sliding-window) algoritmus keresi a gyakori elemhalmazokat a hálózaton. Ha olyan "váratlan" szabályokat (unexpected association rules) bányász ki, amelyek eltérnek a normál profiltól, azokat átadja az Osztályozó Motornak. A motor eldönti róluk, hogy Normál, Ismert Támadás, Ismeretlen Támadás vagy Téves Riasztás (False Alarm) kategóriába esnek-e.

> 📍 **[IDE JÖN A KÉP: Figure 6.5: Training phase of ADAM] és [Figure 6.6: Detecting mode of ADAM]**

#### Táblázat 6.3: Felügyelet Nélküli Anomália Detekciós Módszerek

| Módszer / Szerző | Leírás |
| :--- | :--- |
| **[331]** | (i) Felhasználói profiltól függő anomália detekciós módszer. (ii) Időbeli szekvencia (temporal sequence) klaszterezési technikát használ a valós idejű észleléshez, hoszt-alapú adatgyűjtéssel és feldolgozással. (iii) A kiképző fázisban felhasználói profilokat hoz létre, a teszt fázisban pedig a felhasználói parancsfolyam-adatokat ellenőrzi a profilokhoz képest. |
| **[242]** | (i) Szabályokat tanul ritka események (rare events) keresésére hosszú távú függőségekkel rendelkező névleges idősor-adatokban (time-series). (ii) Anomáliákat talál a hálózati csomagokban és az új típusú (novel) behatoló TCP munkamenetekben. |
| **[378]** | (i) Payload-alapú (adat-tartalom alapú) anomália detektort mutat be. (ii) A hálózati forgalom normál alkalmazás-payloadját teljesen automatikusan, felügyelet nélkül és nagyon hatékonyan modellezi. (iii) Kiszámítja egyetlen hoszthoz/porthoz áramló alkalmazás-payload bájt-frekvencia eloszlásának profilját és szórását (standard deviation) a betanítási fázis alatt. |
| **[424]** | (i) Klaszterezés-alapú behatolásjelző technikát mutat be, amely nem támaszkodik címkézett tanítóadatokra. (ii) A támadások hatékony azonosítása érdekében a klaszterezést kétszer alkalmazza: először az Ágens IDS-nél választja ki a jelölt anomáliákat, másodszor pedig a Központi IDS-nél azonosítja a valódi támadásokat. |
| **[274]** | (i) Egy távolság-mérőszámot (distance measure) vezet be vegyes típusú (mixed-type) behatolási adatokhoz. (ii) Két példány (instance) közötti kapcsolatokat (links) úgy definiálja, hogy a kategorikus és folytonos attribútumok távolságát külön-külön adja össze. |
| **[226]** | (i) A módszer a K-legközelebbi szomszédhoz (KNN) alkalmazott Transzduktív Konfidencia Gépeken (Transductive Confidence Machines - TCM) alapul. (ii) Genetikus Algoritmus-alapú példánykiválasztó módszert használ a tanítóhalmaz méretének korlátozására. (iii) Szűrő-alapú (filter-based) jellemzőkiválasztást használ a legrelevánsabb jellemzők kinyeréséhez a TCM-KNN betanításához. |
| **[362]** | (i) Keretrendszert mutat be a hálózati behatolásjelzésre adatbányászati technikákkal. (ii) Fuzzy asszociációs szabályokat használ az osztályozók (classifiers) felépítéséhez. (iii) A szabálykészleteket leíró modellként (descriptive models) használja. (iv) Felgyorsítja a szabály-indukciós algoritmust azáltal, hogy csökkenti az elemek számát a kinyert szabályokban. |
| **[419]** | (i) Vegyes típusú adatokhoz tervezett IDS modellt mutat be. (ii) A k-means klaszterezés továbbfejlesztett verzióját használja. |
| **[250]** | (i) Klaszterezés-alapú osztályozót javasol. (ii) Több klasztert választ ki egy tesztmintához. (iii) Kiszámítja minden osztályozó átlagos teljesítményét a kiválasztott klasztereken, és a legjobb átlagot nyújtó osztályozót választja a tesztminta besorolásához. |
| **[263]** | (i) K-means klaszterező és Naiv Bayes (KMNB) osztályozó modellt mutat be. (ii) A klaszterezést használja a minták rosszindulatú (malicious) és nem-rosszindulatú osztályokba rendezéséhez. (iii) A végső stádiumban a Naiv Bayes osztályozó végzi a pontos kategóriákba sorolást. |

#### A Felügyelet Nélküli Anomália Detekciós Módszerek Előnyei és Hátrányai

**(i) Előnyök:**

- Particionáló klaszterezésnél, ha a klaszterek száma ($k$) pontosan megbecsülhető, a feladat egyszerű.
- A sűrűség-alapú és outlier (kiugró érték) bányászat természetesen és hatékonyan illeszkedik a behatolásjelzés logikájához (hiszen a támadás egy ritka deviancia).
- A hatalmas hálózati adatbázisok néhány klaszterbe tömörítése drasztikusan csökkenti a számítási komplexitást és gyorsítja a detektálást.
- Miután az ARM (Asszociációs bányászat) felépítette a profilokat, az osztályozó motor betanítása rendkívül egyszerű.

**(ii) Hátrányok:**

- Sok technika (pl. a klasszikus k-means) csak folytonos (numerikus) adatokat tud kezelni, a hálózati forgalom viszont vegyes (IP cím, protokoll).
- **A "Klaszter méret" feltevés hibája:** A módszer feltételezi, hogy a nagy klaszter a normál forgalom, a kis klaszter a támadás. Ezen feltevés nélkül a technika összeomlik (lásd a DDoS problémát a 4. fejezetnél).
- A profilok dinamikus, valós idejű frissítése rendkívül lassú és időigényes.
- A szimpla Support-Confidence alapú ARM módszerek könnyen megtéveszthetők (misleading), amikor ritka (rare class) támadásokat kell felismerni.

## 6.5 Anomália Detekció Valószínűségi Tanulással (Probabilistic Learning)

A valószínűségi tanulás (pl. Bayes-tétel vagy a Dempster-Shafer hiedelem-elmélet alapján) képes értékelni azokat a rendszereket, amelyek véletlenszerűséggel vagy bizonytalansággal (uncertainty) küzdenek. Fő ereje, hogy képes az előzetes becsléseit folyamatosan frissíteni (update) az újonnan beérkező bizonyítékok / hálózati adatok alapján.

### 6.5.1 Rejtett Markov Modell (Hidden Markov Model - HMM) alapú módszerek

A HMM olyan modellezés, ahol a rendszer "Valós/Rejtett állapota" (pl. az, hogy a hálózat épp támadás alatt áll-e) láthatatlan, és csak a megfigyelhető kibocsátásokból (pl. lelassult portok, megnövekedett csomagszám) tudunk következtetni rá. Képes nem-stacionárius szekvenciákat leírni az időben változó állapotátmenetekkel.

> 📍 **[IDE JÖN A KÉP: Figure 6.7: Example of a hidden Markov model]**

A hálózatbiztonságban a HMM használatához 3 kulcsproblémát kell megoldani (az Expectation-Maximization algoritmussal):

1. **Kiértékelés (Evaluation):** Mennyi az esélye, hogy ezt a megfigyelt forgalmat a normál modell generálta? (Ha alacsony, akkor anomália).
2. **Tanulás (Learning):** Hogyan építsük fel a modellt a hálózati audit logokból?
3. **Dekódolás (Decoding):** Mi az a rejtett (támadási) lépéssorozat, ami a látott hálózati eseményeket kiváltotta?

- **Ye HMM modellje:** A hálózat normál temporális (időbeli) profilját építi fel. Egy új eseménysorozatot betáplálva kiszámolja annak valószínűségét. Alacsony valószínűség (low probability of support) = Riasztás.
- **Yeung HMM modellje:** Rendszerhívások és Shell parancsok szekvenciáira épít modellt. Forward/Backward logikát használ a likelihood (valószínűség) számításához. Hátránya: Nehezen általánosít, és nem tud mit kezdeni az egyedi azonosítóval nem rendelkező felhasználókkal.

### 6.5.2 Bayes Hálózatok (Bayesian Networks - BN) alapú módszerek

A Bayes hálózat kódolja az attribútumok (hálózati változók) közötti okozati (causal) és valószínűségi kapcsolatokat.

**Előnyei:**
- Kiválóan kezeli a hiányzó adatokat (missing data), mert ismeri a többi változó egymásrautaltságát.
- Az okozati kapcsolatok miatt képes előre jelezni egy akció következményeit (pl. egy letapogatás után jönni fog egy jelszó-törés).
- Az előzetes emberi/szakértői tudás (prior knowledge) könnyen beépíthető a modellbe.

**Kruegel et al. Szenzorfúziója (Multisensor fusion):**
Kruegel arra a megállapításra jutott, hogy egyetlen anomália detektor sem képes megfelelő bizonyossággal dönteni. Ezért egy Bayes Hálózatot használt több különböző behatolásjelző szenzor kimenetének összefésülésére (aggregálására), ezáltal drasztikusan elnyomva a téves riasztásokat (suppression of false alarms).

**Kockázat (Hátrány):** A módszer pontossága teljes mértékben az előzetes emberi feltételezésektől (assumptions) függ. Ha a támadó viselkedése jelentősen eltér az elvárttól, a Bayes-hálózat nem fog riasztani.

### 6.5.3 Naiv Bayes Módszerek (Naive Bayes Methods)

A Naiv Bayes (NB) modell egy leegyszerűsített Bayes-i valószínűségi modell. Azt a valószínűséget számítja ki, hogy egy esemény egy adott végeredményhez (osztályhoz) tartozik, bizonyos bizonyíték-változók (jellemzők) ismeretében.

- **"Naiv" feltevés:** Feltételezi, hogy a bizonyíték-változók valószínűségei teljesen függetlenek egymástól az adott végeredmény bekövetkezte esetén. (Pl. a csomagméret és a protokoll függetlenek).
- **Működése:** A betanítási (training) fázisban az algoritmus minden egyes attribútumra kiszámolja az adott eredmény (osztály) valószínűségét, majd eltárolja azt. A tesztelési fázisban az időigény egyenesen arányos ($O(n)$) a jellemzők számával.

**Gyakorlati példák az NB használatára:**

- **Ahirwar et al.:** A Naiv Bayes osztályozót kombinálták egy súlyozott RBF (Radiális Bázisfüggvény) Neurális Hálózattal, amely a Gauss-bázisfüggvények keverékével becsülte meg az osztálytagság posterior (utólagos) valószínűségét.
- **Valdes és Skinner (EMERALD rendszer):** A Naiv Bayes módszert forgalmi kitörések (traffic bursts) behatolásjelzésére használták. Képes elosztott támadások detektálására, ahol egyetlen támadási szekció önmagában nem lenne elég gyanús riasztás generálásához.

**A Naiv Bayes módszer korlátai (Hátrányok):**

- Osztályozási képessége lényegében megegyezik egy egyszerű küszöbérték-alapú (threshold-based) rendszerrel, ami csupán összegzi a gyermek csomópontok kimeneteit.
- Mivel a gyermek csomópontok (jellemzők) nem lépnek interakcióba egymással, nagyon nehéz további, a változók közötti összefüggéseken alapuló információt beépíteni a modellbe.

### 6.5.4 Gauss Keverék Modell (Gaussian Mixture Model - GMM)

A GMM egy sűrűségmodell (density model), amely több – általában Gauss-féle (normál) – komponensfüggvény súlyozott összegéből áll.

A keverék sűrűségképlete:

```math
p(\bar{x} \mid \lambda) = \sum_{i=1}^{M} p_i b_i(\bar{x})
```

*(Ahol $\bar{x}$ a D-dimenziós vektor, $b_i(\bar{x})$ a Gauss komponensek sűrűségfüggvénye - ami a vektor, az átlag $\mu$ és a kovariancia $\Sigma$ alapján számolódik -, és $p_i$ a súly, ahol a súlyok összege $1$).*

**A GMM használata hálózati anomália detektálásra:**
Minden anomália-kategóriát (pl. DoS, Probe) egy külön GMM modell ($\lambda$) reprezentál.

**Bahrololum és Khaleghi GMM módszere:**

1. **Betanítási fázis:** Referencia sablonokat (templates) generál a normál forgalomhoz és az egyes támadástípusokhoz (DoS, U2R, R2L, Probe) a kinyert statisztikai jellemzők alapján.
2. **Detektálási fázis:** Amikor beérkezik egy tesztpéldány, a rendszer megvizsgálja a statisztikai eltérést (deviation) az eltárolt referencia modellekhez képest, és az alapján dönti el, hogy melyik osztályba tartozik.

> 📍 **[IDE JÖN A KÉP: Figure 6.8: Training phase of the GMM model]**
>
> 📍 **[IDE JÖN A KÉP: Figure 6.9: Detection phase of the GMM model]**

### 6.5.5 EM (Expectation-Maximization) Algoritmust Használó Módszerek

Az EM algoritmus egy általános módszer a Maximális Valószínűség Becslés (MLE) megtalálására, különösen akkor, ha az adathalmaz hiányos (missing values) vagy rejtett (hidden) paramétereket tartalmaz.

**Működése (Iteratív):**
Nem a globális optimumot számolja ki rögtön, hanem egy kezdeti találgatásból indul, majd ismétel:

1. **E-lépés (Expectation):** Kiszámolja a megfigyelt adatok várható értékét a jelenlegi paraméter-becslések alapján.
2. **M-lépés (Maximization):** Újrakalkulálja (maximalizálja) a paramétereket az E-lépés eredményei alapján.

Ezt addig ismétli, amíg az adatok log-likelihood (logaritmus-valószínűségi) növekménye egy adott pontossági küszöb alá nem esik.

**Példa (Patcha és Park):** Az EM algoritmust használták a beérkező audit adatok klaszterezésére és a hiányzó értékek kiszámítására. Az algoritmus többször lefut különböző kezdeti értékekkel, majd az adatpontot abba a klaszterbe sorolja, amelyiknél a legmagasabb a log-likelihood pontszám.

#### Táblázat 6.4: Anomália Detekció Valószínűségi Tanulással

| Módszer | Leírás |
| :--- | :--- |
| **kmeans+ID3 [126]** | (i) Egy módszert mutat be az anomális tevékenységek osztályozására, a k-means és az ID3 (döntési fa) algoritmusok kombinálásával. (ii) A végső döntéshez a k-means és az ID3 döntéseit két szabállyal kombinálja: (a) legközelebbi-szomszéd (nearest-neighbor) szabály és (b) legközelebbi-konszenzus (nearest-consensus) szabály. (iii) A k-means+ID3 módszer detekciós pontossága magas. |
| **APD algoritmus [86]** | (i) Módszer anomáliák detektálására kategorikus adathalmazokban. (ii) A módszer két lépésből áll: (a) Egy lokális anomália detektor azonosítja azokat az egyéni rekordokat, amelyeknek anomális attribútum-értékeik vannak, majd (b) Olyan mintázatokat (patterns) keres, ahol az anomális rekordok száma magasabb a vártnál. (iii) Valós tengeri konténerszállítási és hálózati behatolási adatokon észlelt anomális mintázatokat. |
| **DGSOT + SVM algoritmus [196]** | (i) SVM-et (Support Vector Machine) és hierarchikus klaszterezést használ az anomáliák osztályozására. (ii) Megkeresi a két osztály közötti határpontokat (boundary points) az SVM-ek betanításához, ezzel csökkentve a tanítási időt. (iii) Kombinálja a dinamikusan növekvő önszervező fát (DGSOT) az SVM-ekkel a releváns határpontok megtalálásához a 2-osztályos SVM betanítására. |
| **CUSUM-EM algoritmus [237]** | (i) Egy hibrid keretrendszert használ a hálózati anomáliák optimális felismerésére. (ii) A SNORT-ot alkalmazza szignatúra-alapú detektorként, kiegészítve két másik anomália detekciós módszerrel: a nem-parametrikus CUSUM-mal és az EM-alapú klaszterezéssel. |
| **LERAD modell [62]** | (i) Két módszert mutat be az anomália detekciós modellek múltbeli viselkedésből történő felépítésére. (ii) Az első módszer egy szabályalapú (rule-based) tanuló algoritmus, amely a normál viselkedést jellemzi címkézett támadási adatok hiányában. (iii) A második módszer egy klaszterező algoritmus a kiugró értékek (outliers) azonosítására. |
| **KPCC modell [369]** | (i) Egy Kernel Főkomponens Osztályozó (Kernel Principal Component Classifier - KPCC) sémát mutat be hálózati anomália detekcióra. (ii) Kétféle anomális forgalmat fog meg: (a) extrém nagy értékeket néhány eredeti jellemzőn és (b) olyan forgalmi mintázatokat, amelyek nem felelnek meg a normál forgalom korrelációs struktúrájának. |
| **0-day (Oday) anomália detekció [276]** | (i) Egy kognitív algoritmust és mechanizmust mutat be a 0-day (nulladik napi / eddig ismeretlen) támadások felismerésére. (ii) Képes azonnal új védelmi stratégiákat (defense strategies) telepíteni, amikor egy új 0-day támadással találkozik. |
| **Profil-alapú megközelítés [345]** | (i) A hálózat aktivitását monitorozza anomáliák után kutatva. (ii) Ha bármilyen eltérést tapasztal a jóindulatú (benign) forgalomból korábban betanult normál profiloktól, azt anomáliaként jelenti. |
| **Kernel OCSVM [381]** | (i) SVM-eket használ nagy mennyiségű netflow rekord elemzésére. (ii) Speciális kernel-függvényt alkalmaz, és mind kontextuális, mind kvalitatív információkkal dolgozik, amiket a hálózati folyamatokból (network flows) nyer ki az anomáliák jelentésére. |

#### A Valószínűségi Tanulási Módszerek Előnyei és Hátrányai

**(i) Előnyök:**

- A betanításuk (training) és tesztelésük nagyon rugalmas (flexible), a végrehajtási stratégiák könnyen frissíthetők.
- Ha a modellt megfelelően betanítják az adott támadásra, rendkívül pontosan és időben (on time) képesek észlelni az ismert támadásokat.

**(ii) Hátrányok:**

- Erősen függnek a rendszerbe táplált előzetes feltételezésektől (assumptions).
- Érzékenyek a bemeneti paraméterekre: a paraméterek legkisebb változtatása is drasztikusan lerombolhatja a detekciós teljesítményt.
- Számítási igényük nagyon magas: sokkal több erőforrást (RAM, CPU) emésztenek fel, mint a legtöbb más módszer.
- Vakok az új támadásokra: Nem képesek ismeretlen (zero-day) eseményeket megjósolni vagy felismerni, amíg ahhoz releváns tanító (training) információt nem táplálnak beléjük.

- ## 6.6 Anomália Detekció Lágy Számítással (Soft Computing)

A lágy-számítási (soft computing) módszereket azért hívják "lágynak", hogy megkülönböztessék őket a hagyományos "kemény" (hard) számítástól, ami egzakt kommunikációt és $100\%$-os pontosságú válaszokat követel meg.

A hálózati anomáliák detektálásában ritkán létezik egzakt (tökéletes) megoldás, ezért a lágy számítás kiválóan alkalmazható. Két fő jellemzője a valószínűség (pl. Bayes-i véletlenszerűség) és a pontatlanság / homályosság (inexactness, pl. fuzzy halmazok).

Ide tartoznak a Genetikus algoritmusok, Mesterséges Neurális Hálózatok, Fuzzy halmazok, Durva halmazok (Rough sets), Hangyakolónia algoritmusok és a Mesterséges Immunrendszerek.

### 6.6.1 Genetikus Algoritmus (GA) Megközelítések

A Genetikus Algoritmusok az evolúció és a természetes kiválasztódás elvén alapuló számítási modellek. A problémát egy kromoszóma-szerű adatszerkezetbe kódolják, amelyet generációkon át fejlesztenek (kiválasztás/szelekció, keresztezés/rekombináció, mutáció). A hálózatbiztonságban főleg optimalizációs problémák megoldására használják.

**Példa egy "hálózati" kromoszómára:** Egy egyén (csomag vagy folyamat) kromoszómája olyan "géneket" (attribútumokat) tartalmazhat, mint a szolgáltatás típusa, TCP flag-ek, bejelentkezett-e a felhasználó, vagy hány sikertelen root próbálkozás történt.

**Kutatási eredmények:**
- **Khan** megfigyelte, hogy a GA-val a gyakori támadásokat sokkal pontosabban fel lehet ismerni, mint a ritkákat.
- **Balajinath és Raghavan** egy GA-alapú behatolásjelzőt (GBID) épített, amely a felhasználói viselkedést egy 3-elemű (3-tuple) struktúrában írja le, és az eltérések alapján detektálja a támadást.

### 6.6.2 Mesterséges Neurális Hálózat (ANN) Megközelítések

Az ANN az emberi agy működését próbálja utánozni. Kiválóan alkalmas adatok klaszterezésére, jellemzőkiválasztásra (feature extraction) és anomális mintázatok azonosítására.

- **Cannady:** Adaptív NN-t fejlesztett, amely módosított megerősítéses tanulással (reinforcement learning) nagyon gyorsan, autonóm módon tanul meg új támadásokat, visszacsatolás (feedback) használatával a szignatúrák frissítésére.
- **Liu et al.:** Főkomponens Elemzésre (PCA) épülő, hierarchikus felügyelet nélküli neurális hálót alkotott a valós idejű észleléshez, kiküszöbölve az egyrétegű hálózatok hibáit.
- **Yong és Feng:** Visszacsatolt többrétegű perceptronokat (RMLP) használtak. Mivel ez a hálózat képes visszamenőleg emlékezni, meg tudja tanulni az időbeli (temporális) információkat is (pl. egy lassú portscan időbeliségét).

### 6.6.3 Fuzzy Halmazelméleti Megközelítés (Fuzzy Set Theoretic)

A Fuzzy IDS-ek fuzzy szabályokat használnak bizonyos támadások bekövetkezési valószínűségének meghatározására.

- **Tajbakhsh et al.:** Fuzzy asszociációs szabályokat generáltak a normál és anomáliás osztályok leírására. Ha egy beérkező csomag összeegyeztethetősége (compatibility) a szabálykészlettel egy bizonyos küszöb felett van, akkor normális, különben anomália.
- **Mabu et al.:** Genetikus Hálózati Programozást (GNP) használtak. A GNP irányított gráf (directed graph) struktúrákat használ a standard GA-ban megszokott szöveges sztringek helyett, ami sokkal kompaktabb és hatékonyabb reprezentációt tesz lehetővé a csomópontok újrahasználhatósága miatt.

### 6.6.4 Durva Halmaz (Rough Set) Megközelítések

A Durva halmaz egy éles (crisp) halmaz alsó és felső közelítését (approximation) adja meg. Segít a kapcsolatok minimális számú szabállyal történő modellezésében, kifejezetten hasznos bizonytalanság esetén.

**Két fő előnye:**
1. Képes nagyon kicsi tanító adathalmazokból is tanulni.
2. Rendkívül egyszerű.

**Példa:** Chen et al. egy kétlépcsős rendszert épített. A durva halmazokat csak arra használták, hogy a hálózati adatokból kinyerjék és lecsökkentsék a jellemzőket (feature reduction), amit aztán átadtak egy SVM (Support Vector Machine) osztályozónak a végső detektáláshoz.

### 6.6.5 Hangyakolónia és Mesterséges Immunrendszer (AIS) Megközelítések

- **Hangyakolónia (Ant Colony):** Problémákat próbál megoldani gráfok optimális útvonalainak bejárásával (ahogy a hangyák keresik az élelmet, feromont hagyva). **Gao et al.** jellemzőkiválasztásra használta: a gráf csomópontjai a feature-ök voltak. A hangyák addig járták a gráfot és gyűjtötték a feature-öket, amíg el nem értek egy megállási kritériumot, az eredményt pedig egy SVM-nek adták.
- **Mesterséges Immunrendszer (AIS):** Az emberi immunrendszer működését utánozza. Az idegen anyagokat (antigéneket / hálózati támadásokat) felismeri és ellenanyagot (antitestet / szignatúrát) termel. Ez a **Negatív Kiválasztás (Negative Selection)** folyamata: a rendszer megtanulja, hogy mi a "Saját" (Self / Normál forgalom), és minden, ami "Nem-Saját" (Nonself), az betolakodó.
- **Stibor et al.** megállapította, hogy bár az AIS elméletben kiváló, a valóságban a hatalmas dimenziószámú hálózati adatoknál túl sok időbe telik elegendő számú "detektor sejtet" legenerálni, így a gyakorlatban (egyelőre) kevésbé használható.

### Táblázat 6.5: Lágy Számítás Alapú Anomália Detekciós Módszerek (1/2)

| Módszer | Leírás |
| :--- | :--- |
| **Genetikus Algoritmus (GA) alapú megközelítés** | |
| Viselkedési modell [30] | (i) Egy GA-alapú behatolásjelző (GBID), ami az egyéni felhasználói viselkedést tanulja meg. (ii) A felhasználói viselkedést 3-elemű (3-tuple) struktúraként írja le és genetikus algoritmussal tanulja meg. (iii) A múltbeli viselkedés alapján detektálja a behatolást. |
| **ANN (Neurális Háló) alapú megközelítés** | |
| NSOM modell [215] | (i) Mintavételezési módszerrel előfeldolgozza az adatokat az attribútumok kiválasztására. (ii) Az EM algoritmust alkalmazza a behatolásjelzéshez. |
| Adaptív NN [59] | (i) Adaptív neurális hálót alkalmaz az anomália detekcióhoz, amely módosított megerősítéses tanulással (reinforcement learning) nagyon gyorsan és önállóan tanul meg új támadásokat. (ii) Visszacsatolást használ a szignatúrák frissítésére. |
| Hierarchikus NN [220] | (i) Neurális hálózatok hierarchiájából áll, kiváló anomália detektorként működik. (ii) A neurális hálókat a teljes "normál" teret lefedő adatokkal tanítják be, így képesek felismerni az ismeretlen támadásokat. |
| HNNIDS Modell [418] | (i) Két hierarchikus IDS keretrendszer RBF (Radiális Bázisfüggvény) használatával. (ii) Először egy soros hierarchikus IDS (SHIDS) azonosítja a visszaélésszerű (misuse) támadásokat pontosan, és az anomália támadásokat adaptívan. (iii) Ezután egy párhuzamos hierarchikus IDS (PHIDS) javítja a SHIDS funkcionalitását és teljesítményét. |
| RT-UNNID rendszer [14] | (i) Intelligensen, valós időben detektál felügyelet nélküli (unsupervised) neurális hálóval. (ii) Újratanítás (retraining) nélkül is hatékonyan kezeli az új eseteket. (iii) Adaptív Rezonancia Elméletet (ART) és Önszervező Térképes (SOM) Neurális Hálózatot használ offline adatokkal. |
| Unsupervised NN [231] | (i) Valós időben detektál ismert és ismeretlen támadásokat. (ii) Egy hierarchikus behatolásjelző modellt használ PCA neurális hálókkal az egyrétegű struktúrák hibáinak kiküszöbölésére. |
| WNN modell [360] | (i) Egy Wavelet NN (WNN)-alapú IDS. (ii) Csökkenti a wavelet alapfüggvények számát a mintaadatok ritkasági (sparseness) tulajdonságainak elemzésével, ami nagymértékben optimalizálja a wavelet hálózatot. A tanuló algoritmus gradiens ereszkedéssel (gradient descent) tanítja a hálózatot. (iii) A teljesítménye kielégítő. |
| NN-alapú IDS [210] | (i) Egy experimentális elemzést mutat be a támadások felismerésére NN-alapú IDS használatával. (ii) Az adaptabilitás egyszerűvé teszi a neurális osztályozó tesztelését és újratanítását. |

### Táblázat 6.6: Lágy Számítás Alapú Anomália Detekciós Módszerek (2/2)

| Módszer | Leírás |
| :--- | :--- |
| **Fuzzy Halmazelméleti megközelítés** | |
| Unsupervised fuzzy clustering [388] | (i) Újszerű, felügyelet nélküli fuzzy klaszterező módszer klonális szelekció (clonal selection) alapján anomália detekcióhoz. (ii) A konkurens algoritmusoknál gyorsabban képes globális optimális klasztereket találni. |
| Fuzzy rule mining [240] | (i) Genetikus Hálózati Programozáson (GNP) alapul a hálózati behatolások észleléséhez. (ii) Irányított gráf-struktúrákat használ a GA-ban lévő sztringek helyett, ami javítja a reprezentációs képességet, mivel a gráfstruktúrából eredő csomópont-újrafelhasználhatóság (node reusability) tömörebb programokat eredményez. |
| **Durva Halmaz (Rough set) alapú megközelítés** | |
| Fuzzy rough C-means [75] | Egy fuzzy durva C-means klaszterezési technika, amely a fuzzy halmazelméletet és a durva halmazelméletet integrálja a behatolásjelzés magas találati arányának eléréséhez. |
| RST-SVM módszer [67] | Durva halmazelméletet (RST) és SVM-et használ anomáliák detektálására. Az RST segít a jellemzők csökkentésében (feature reduction), majd az adatokat az SVM osztályozónak küldi végső besorolásra. |
| **Hangyakolónia és AIS alapú megközelítés** | |
| Hangyakolónia és SVM [129] | (i) Hangyakolónia optimalizálást használ jellemzőkiválasztásra, és SVM-et hálózati behatolásjelzésre. (ii) A jellemzőket gráf-csomópontokként ábrázolja, ahol a köztük lévő élek a következő jellemző hozzáadását jelentik. A hangyák addig járják a gráfot új csomópontokat hozzáadva, amíg el nem érik a megállási kritériumot. |
| AIS rendszer [376] | (i) Egy teljesítmény-alapú AIS az egyedi anomális viselkedés azonosítására. (ii) Monitorozza a rendszert a paraméterhalmaz elemzésével, hogy általános információt nyújtson annak állapotáról. (iii) Egy 2-es típusú intervallum (interval type-2) fuzzy halmaz paradigmát használ a rendszerállapot dinamikus aktiválására. |

### A Lágy Számítási Módszerek Előnyei és Hátrányai

**(i) Előnyök:**

- **Nem törékenyek (Not brittle):** Olyan problémákat is képesek megoldani, amikre a matekban nincs egzakt, tökéletes válasz.
- A természet ihlette módszerek (genetika, immunrendszer, hangyák) bizonyítottan kiválóak a legkomplexebb optimalizációs problémák megoldásában.
- A rendszer környezeti visszacsatolás nélkül is képes észlelni a tartós (persistent) mintázatokat.
- A Neurális Hálózatok (ANN) adaptívak, lehetővé téve a példányok inkrementális (apró lépésekben történő) tanítását és tesztelését.
- A hierarchikus (többszintű) neurális hálózatok sokkal hatékonyabbak, mint az egyrétegűek.

**(ii) Hátrányok:**

- **Túltanulás (Over-fitting):** A Neurális Hálózatok betanításának egyik legnagyobb veszélye (a modell "betanulja" a zajt is).
- Megfelelő mennyiségű tiszta normál forgalom nélkül a technika betanítása nagyon nehéz.
- **Skálázhatósági problémák (Scalability):** A legtöbb módszer nem bírja a hatalmas (Gigabites) adatterhelést.
- A Durva Halmaz (Rough Set) hátránya, hogy nem garantálja a teljességet (lack of completeness), vagyis nem biztos, hogy az összes lehetséges szabályt legenerálja.
- A Fuzzy asszociációs szabályoknál a meglévő szabályok dinamikus frissítése, valós idejű újrakalkulálása rendkívül nehéz feladat.

- ## 6.7 Tudásalapú Anomália Detekció (Knowledge in Anomaly Detection)

A tudásalapú (Knowledge-based) módszereknél a hálózati vagy hoszt (gép) eseményeket előre meghatározott szabályokhoz, vagy támadási mintázatokhoz (patterns) viszonyítják. Ezek a módszerek az úgynevezett **Visszaélés-felismerő (Misuse detection)** kategóriába tartoznak. Lényegében "bűnösségi fokozatokat" (degrees of attack guilt) állítanak fel, és az ismert támadási leírások (representations) alapján keresnek.

**Hátrányuk a gyakorlatban:** Egy teljesen új támadást, vagy egy ismert támadás enyhén módosított változatát szinte biztosan elvétik (missed).

### 6.7.1 Szakértői Rendszerek és Szabályalapú Megközelítések (Expert System and Rule-Based)

A Szakértői Rendszer általában egy szabályalapú (rule-based) rendszer, amelyhez tartozik egy tudásbázis és egy szabály-motor (rule engine). Ez a motor veti össze az aktuális hálózati állapotot a szabályokkal, és ha egyezést talál, "elsüti" (fires) a riasztást.

#### A Snort IDS

A Snort a világ legnépszerűbb, nyílt forráskódú szabályalapú IDS-e.

- **Működése:** Minden bejövő IP csomagot összevet egy szabálykészlettel. A Snort szabály egy logikai (boolean) formula, amely konkrét értékeket keres az IP fejlécben, a TCP/UDP portokban, ICMP kódokban, vagy épp specifikus sztringeket (karaktersorozatokat) a csomag adat-tartalmában (payload).
- **Prioritások:** A Snort a szabályokat aszerint rangsorolja, hogy mekkora a támadás potenciális hatása. Jelenleg több mint $20\,000$ szabályt tartalmaz, melyeket a felhasználói közösség folyamatosan frissít.
- **A skálázódás (Scaling) problémája a Snortnál:** Egy hatalmas, több gigabites hálózaton az összes csomag mélyvizsgálata (deep packet inspection) iszonyatos számítási költséggel jár, a százszámra futó szabályok miatt a méretezés szinte lehetetlen.
- **Megoldás (Duffield et al.):** Az Adaboost gépi tanulási algoritmus segítségével "lefordították" a Snort csomagszintű (packet level) szignatúráit folyamatszintű (flow level) statisztikákká. A Flow adatokból felépített gépi tanulási modellel a Snort-hoz mérhető pontosságot értek el, a gigászi számítási overhead nélkül.
- **Egyéb kísérletek (Prayote):** Egy olyan szakértői rendszert épített, ahol a szabályok dinamikusan szűkülnek vagy tágulnak a beérkező adatok statisztikai (outlier) értékei alapján.
- **Polimorfikus férgek (Scheirer és Chuah):** A Rabin-ujjlenyomatok és szemantikai sablonok (templates) segítségével próbálták megfogni a kódjukat folyamatosan változtató (polimorfikus) férgeket.

### 6.7.2 Ontológia- és Logika-alapú Megközelítések (Ontology- and Logic-Based)

Lehetőség van a támadási szignatúrák modellezésére rendkívül kifejező (expressive) logikai struktúrákkal is.

- **Időbeli Logika (Temporal Logic):** Naldurg et al. egy EAGLE nevű logikai nyelvet használtak, amely paraméterezett rekurzív egyenleteket támogat. A rendszer figyeli a végrehajtási folyamatokat, és ha megsértik a specifikációt (szabályt), azonnal riaszt.
- **KBTA (Mobil biztonság):** Shabtai et al. a Knowledge-Based Temporal Abstraction módszert használták okostelefonokon (Android) lévő, eddig ismeretlen kártevők (malware) felismerésére.
- **Ontológiák:** Hung és Liu az ontológiákat (tudás-reprezentációs hálókat) arra használták, hogy egy nem-szakértő (átlagos) felhasználó is könnyen tudjon behatolásjelző modelleket definiálni emberközeli fogalmakkal.

#### Táblázat 6.7: Tudásalapú Anomália Detekciós Módszerek

| Módszer | Leírás |
| :--- | :--- |
| **Szakértői és Szabályalapú megközelítés** | |
| Szabályalapú keretrendszer | Egy szakértői rendszer, amely formalizálja a hálózati támadási szabályokat az IDS-hez az Open System Interconnection (OSI) hálózati keretrendszerben meghatározott 7 réteg referenciájára építve. Kifejezett sebezhetőség-keresési fázisokat azonosít az OSI modell szemszögéből. |
| Elosztott több-ágenses (multi-agent) rendszer | Egy IDS modell, amely a rendszeren belül együttműködő ágensekből áll egy elosztott és skálázható megközelítés támogatására. Javítja az észlelt betörések kezelését. Minden ágens egy specifikus feladatért felelős. |
| Polimorfikus féreg észlelés (Worm detection) | Végrehajtható (executable) hálózati forgalomra fókuszál. Egy hálózati IDS-ben az elemzést (parsing), az attribútumok kiértékelését (attribute evaluation) és az attribútumok elemzését (attribute analysis) integrálja a gyanús folyamatok észleléséhez. Részletes leírást biztosít a támadási forgatókönyvekről. |
| **Ontológia és Logika-alapú megközelítés** | |
| Időbeli absztrakció (Temporal abstraction) | Egy tudásalapú időbeli absztrakciós megközelítés. Többszörös absztrakciós modellt alkalmaz időbélyeggel ellátott biztonsági adatok (több forrásból, hálózatról és hosztról) leírására a megelőző folyamatok integrálásával a predikciós képességek biztosítására. |
| Időbeli logika specifikáció (Temporal logic specification) | Az Eagles használata a hálózati behatolások formális leírására (formal representation) állapotátmenetekkel (state transitions) kiegészítve. Új szabályok formális ellenőrzését (formal verification) és konzisztencia-ellenőrzését (consistency check) is biztosítja a riasztási mechanizmus fokozására. |

**(i) Előnyök:**
Nagyon robusztusak és rugalmasak. Ha az emberi tudás pontos (jó a szignatúra), a detekciós ráta (Detection rate) majdnem $100\%$-os.

**(ii) Hátrányok:**
A tudásbázis felépítése és karbantartása emberi munkát igényel, nagyon lassú és drága. Vakok (incapable) az ismeretlen támadásokkal szemben.

## 6.8 Anomália Detekció Kombinált Tanulókkal (Combination Learners)

Ezek a módszerek több különböző gépi tanulási technikát ötvöznek azzal a céllal, hogy növeljék a pontosságot (accuracy) és csökkentsék a téves riasztásokat (false alarms). Három fő kategóriájuk van: **Együttes (Ensemble)**, **Fúziós (Fusion)** és **Hibrid (Hybrid)**.

### 6.8.1 Együttes (Ensemble) Módszerek

Ahogy az ember is több véleményt kérdez meg egy fontos döntés előtt, az Ensemble módszerek több különböző osztályozó "szavazatát" súlyozzák és kombinálják. Gyakran a leggyengébb egyéni osztályozók sokasága jobb eredményt ad, mint egyetlen erős algoritmus.

A három fő technika: (i) Bagging, (ii) Boosting, és (iii) Stack generalization.

#### Táblázat 6.8: Együttes (Ensemble) Hálózati Anomália Detekciós Módszerek

| Módszer | Leírás |
| :--- | :--- |
| **Együttes k-means (Ensemble k-means)** | Célja a teljesítmény javítása többszörös egyszerű modellek kimenetének kombinálásával. Csökkenti a véletlenszerű kezdőértékek (random initial conditions) befolyását, kiküszöbölve a k-means algoritmus hiányosságait a KDD 1999 adatbázis használatával. |
| **SVM Együttesek (SVM Ensembles)** | (i) Minimalizálja a képzési adatokkal kapcsolatos (data-related) zajok vagy kiugró értékek (outliers) hatását az SVM betanítása során több kisebb adathalmaz betanításával. (ii) Jelentősen lecsökkenti a Support Vector Machine-ok (SVM-ek) betanítási idejét, különösen nagy hálózati behatolási adatokon (a KDD 99 alapján értékelve). |
| **K-legközelebbi szomszéd meta-osztályozó** | (i) Egy új, osztályozók együttesén (ensemble of classifiers) alapuló megközelítés hálózati alapú IDS-hez, meta-tanulás (meta-learning) koncepcióval kiegészítve, csökkentett attribútum-készlet használatával. |
| **Neuro-fa (Neuro-tree)** | (i) Ötvözi a neurális hálózatokat a döntési fákkal (decision trees), így a két technika előnyeit kombinálja. (ii) Az eredmények magas észlelési arányt (detection rate) mutatnak viszonylag alacsony téves riasztási arány (false positive rate) mellett a KDD-99 adathalmazon mérve. |
| **RS-kNN** | A k-Nearest-Neighbor (kNN) technikát Random Selection (RS - Véletlenszerű kiválasztás) algoritmussal egészíti ki az észlelési arány javítása érdekében U2R és R2L támadásoknál. A téves riasztási arányokat rendkívül alacsonyan tartja. |

**(i) Előnyök:**
Gyenge osztályozókból erős eredményt adnak; hatalmas adathalmazokra is skálázhatók; paramétereik könnyen finomhangolhatók. A Stack generalization remekül kiaknázza az egyéni predikciók diverzitását.

**(ii) Hátrányok:**
A megfelelő algoritmus-kombináció kiválasztása nagyon nehéz; a mohó (greedy) kiválasztási folyamat nagyon lassú nagy adatbázisokon; több modell párhuzamos betanítása (és újratanítása) időrabló; valós idejű (real-time) használatuk a többszörös számítások miatt nehézkes.

### 6.8.2 Fúziós Módszerek (Fusion Methods)

A fúziós módszerek az anomália detekció különböző szintjein vonják össze az információkat: (i) adatszinten (data level), (ii) jellemzőszinten (feature level) vagy (iii) döntési szinten (decision level). Olyan platformokat próbálnak alkotni, ahol számtalan módszer dolgozik együtt a "hálózat állapotának minél realisztikusabb megalkotása" érdekében.

#### Táblázat 6.9: Fúziós Anomália Detekciós Módszerek

| Módszer | Leírás |
| :--- | :--- |
| **Szenzorfúzió (Multisensor fusion)** | Bővített szignatúra-szűrést (advanced signature filtering) alkalmaz. Ötvözi az ismert TCP protokoll jellemzőket egy véletlenszerű (randomized) küszöbérték/frekvencia technikával a szignatúrák felismeréséhez. Több NIDS riasztást fuzionál, hogy összefüggéseket (correlations) találjon a korábban függetlennek tűnő betörések között, csökkentve a téves riasztások és a redundáns logok számát. |
| **Együttes Klasszifikátor (Classifier Ensemble)** | Többszörös gépi tanulási modellek döntéseit fuzionálja, hogy egyedülálló, magasan integrált döntést hozzon, amely robusztusabb, mint az egyedi rész-osztályozók. |

**(i) Előnyök:**
Általában kevesebb betanítási (training) időt igényelnek, költséghatékonyabbak, drasztikusan csökkentik a téves riasztásokat (False Alarm). A döntési szintű fúzió rendkívül magas detekciós rátát hoz.

**(ii) Hátrányok:**
A szigorú adatmintás betanítás nagyon magas számítási (computational) költséggel jár. A jellemzőszintű fúzió (feature level) kifejezetten időrabló feladat, és több különböző osztályozó hipotézisének szinkronba hozása nagyon bonyolult.

### 6.8.3 Hibrid Módszerek (Hybrid Methods)

A jelenlegi IDS-ek általában Visszaélés (Misuse) vagy Anomália detekciót használnak (egyiket vagy másikat). A Hibrid módszerek mindkettőt egyszerre alkalmazzák, hogy az ismert ÉS az ismeretlen (zero-day) támadásokat is magas pontossággal fogják meg.

- **Multi-Agent Rendszerek (MAS):** ANN (Neurális Háló) és CBR (Eset-alapú érvelés / Case-based reasoning) kombinálása dinamikus, valós idejű védelemre több ágens bevonásával.
- **FLIP modell:** Megakadályozza a bináris kód-injekciós támadásokat. Egy anomália osztályozóból, egy szignatúra-szűrőből és egy felügyeleti keretrendszerből áll. Képes befogni a bejuttatott kódot, és abból azonnal szignatúrát generálni (zero-day exploit prevention).
- **DT + SVM:** Döntési Fák és Support Vector Machine-ok hibrid hierarchiája a számítási komplexitás minimalizálásáért.
- **Random Forests Hibridizáció:** A véletlen erdők kiterjesztése egyszerre misuse és anomaly módra.
- **Selim et al. (Többszintű intelligencia):** Többszintű hibrid neurális hálózatokból és döntési fákból álló rendszer, amely a jelenleg elérhető egyik legjobb teljesítményt nyújtja mind az ismert, mind az ismeretlen támadások ellen.

#### Táblázat 6.10: Hibrid Anomália Detekciós Módszerek

| Módszer | Leírás |
| :--- | :--- |
| **K-Means és Döntési fák** | Újszerű hálózati behatolásjelző rendszer a K-Means algoritmus és Döntési Fa alapú módszerek integrálásával (vegyítve a statisztikai és gépi tanulási modellek erejét) a DARPA98-ra alapozva. |
| **Bayes Hálózatok (Bayesian Networks - BN) és Döntési Fák** | Elosztott és többágenses (multi-agent) behatolásjelző modell. Döntési fákat használ az NIDS beépített tudásaként, kiegészítve BN-nel a megerősített detektálási döntésekhez és alacsonyabb téves riasztási (false positive) arányokhoz. |
| **GMM és RBFNeurális hálózat** | Egy adaptív Bayes hálózati technika a Gauss Keverék Modellek (GMM) és az RBF neurális hálózatok hibrid kombinációjával megerősítve, a hálózati paraméter-kiértékelés, valamint az adatbiztonság és hitelesítés folyamatának javítására. |
| **Fuzzy Logika és NN** | (i) Egy neuro-fuzzy osztályozó kiegészítve mesterséges neurális hálózattal (ANN). (ii) Elemzi a "normál" adat-előfordulásokat (data occurrences). |

**(i) Előnyök:**
Általánosságban felülmúlják az egyéni szignatúra- vagy anomália-alapú IDS-ek teljesítményét mind az ismert, mind az ismeretlen támadások (zero-day) felismerésében.

**(ii) Hátrányok:**
A "rossz" vagy inkompatibilis algoritmusok hibridizációja gigantikus erőforrásigényt és borzasztó lassú futást eredményezhet (low execution performance). Továbbá az új szabályok / szignatúrák ütközésmentes (without conflict), emberi beavatkozás nélküli dinamikus beépítése a szabálybázisba a mai napig megoldatlan kutatási probléma.

## 6.9 Összegzés (Discussion)

Ebben a fejezetben a hálózati anomália detekció számtalan módszerét vitattuk meg, különböző kategóriákba sorolva őket. A kutatók és biztonsági szakemberek számára a legfontosabb tanulságok a következők:

- **Nincs "Tökéletes" Módszer:** Az összes bemutatott behatolásjelző technikának és rendszernek megvannak a maga egyedi erősségei és gyengeségei (unique strengths and weaknesses). Hogy melyik anomália detekciós technikát érdemes választani, az teljes mértékben a megoldandó probléma természetétől függ.
- **A Magas Dimenziószám (High Dimensionality) Kihívása:** A Legközelebbi Szomszéd (Nearest Neighbor) és a klaszterező technikák nagyon rosszul teljesítenek, ha a dimenziók száma magas. Miért? Mert a távolságmérő képletek (distance measures) több ezer dimenzióban elveszítik a képességüket, hogy megkülönböztessék a normál és az anomáliás példányokat.
  - **Megoldás:** Az Altér (Subspace) klaszterezés segít, de időigényes. Az Osztályozó (Classification) technikák sokkal jobbak lennének erre, de azokhoz meg címkézett (labeled) adat kell, amit nehéz szerezni. A statisztikai módszerek sikere pedig tisztán azon múlik, hogy a matematikai feltevéseik mennyire igazak a valós hálózatra.
- **Távolság vs. Sebesség (Proximity vs. Speed):** Ha egy hálózatban nehéz jó távolságmérőt (distance measure) találni, akkor statisztikai vagy osztályozó technikákhoz kell nyúlni. Fontos megjegyezni, hogy bár ezek betanítása (training) nagyon drága (számítás-igényes), ez a valós idejű védelemnél elfogadható, mivel a betanítás offline történik, a tesztelés (az éles detektálás) viszont villámgyors.
- **A "Ritka Anomália" Feltevés Buktatója:** A felügyelet nélküli (unsupervised) technikák arra építenek, hogy az anomália ritka. Ez általában igaz, de ha a támadás tömeges (bulk anomalies - pl. egy gigantikus DDoS hullám), ezek a rendszerek elbuknak és magas téves riasztási rátát (false alarm) produkálnak. Tömeges támadások ellen a felügyelt (supervised) vagy félig-felügyelt módok teljesítenek a legjobban.

### Táblázat 6.7: Tudásalapú Anomália Detekciós Módszerek

| Módszer | Leírás |
| :--- | :--- |
| **Szabályalapú (Rule-based) megközelítés** | |
| Adaptív behatolásjelzés [223] | (i) Aktivitási mintázatokat számít ki a rendszer audit adataiból, és prediktív jellemzőket nyer ki a mintázatokból. (ii) Szabályokat generál a behatolások észlelésére egy gépi tanulási algoritmus audit rekordokra történő alkalmazásával, a jellemződefinícióknak megfelelően. |
| Rule-GA [197] | (i) Szabályalapú módszer, amely Genetikus Algoritmust (GA) használ. (ii) DoS és Probe támadások alapján értékelték ki. |
| IP folyam-alapú anomália detekció [106] | (i) Hálózati szintű rendszerarchitektúrát mutat be folyam-alapú (flow-based) anomália detekcióhoz. (ii) Gépi tanulási technikákkal értékelték ki. |
| **Szakértői rendszer (Expert system) megközelítés** | |
| DCEIPCA algoritmus [408] | (i) Egy inkrementális kernel PCA algoritmust vezet be nagy adathalmazok kezelésére. (ii) Szakértői rendszert használ a hálózati anomáliák azonosítására. (iii) A KDDcup99 behatolási adathalmazzal értékelték ki. |
| Szintaxis vs. Szemantika alapú megközelítés [325] | (i) Szintaxis-alapú sémát mutat be, amely változó hosszúságú particionálást használ többszörös töréspontokkal (break marks) a polimorf férgek észlelésére. (ii) Szemantika-tudatos (semantics-aware) képességet biztosít. (iii) Képes elkapni a polimorf shell kódokat kiegészítő verem-szekvenciákkal (stack sequences) és matematikai műveletekkel. |
| **Ontológia- és logika-alapú megközelítés** | |
| Markov lánc modell [116] | (i) Egy véges állapotgép (finite state machine) módszertant ír le, ahol az állapotok sorozata és a köztük lévő átmenetek alkalmasnak tűnnek a hálózati protokollok modellezésére. (ii) Hatékonyan képes detektálni az illegitim viselkedési mintázatokat. |
| EAGLE logikai modell [266] | (i) Egy időbeli logika specifikáción (temporal logic specification) alapuló keretrendszert mutat be behatolások észlelésére. (ii) A behatolási mintázatokat formulákként specifikálja egy EAGLE nevű, kifejezésekben gazdag és hatékonyan monitorozható logikában. (iii) DARPA log fájlokkal értékelték ki. |
| Ontológia-alapú algoritmus [181] | (i) Ontológiákat használ egy tartomány (domain) tudásának leírására. (ii) A behatolásjelző mechanizmust a végfelhasználó fogalmaival (domain) fejezi ki. (iii) Az ontológiákat fogalmi modellezési eszközként használja, lehetővé téve a nem-szakértők számára a behatolásjelző alkalmazás sokkal intuitívabb modellezését, kizárólag behatolásjelzési koncepciók használatával. |
| Logika-alapú modell [262] | (i) Egy föderált adatmodellt (federated data model) vezet be biztonsági rendszerek számára a biztonsági incidensekről és azok kontextusáról szóló tudás lekérdezésére és megerősítésére (assert). (ii) Következetes és formális alapot biztosít a kiegészítő bizonyítékok (complementary evidence) mérlegeléséhez (reason about) szükséges információk megjelenítéséhez, a behatolásjelző rendszerek által generált riasztások megerősítése vagy érvénytelenítése (invalidate) érdekében. |
| KBTA modell [333] | (i) Ismeretlen, mobileszközöket célzó kártevőket (malware) detektál. (ii) Folyamatosan figyeli az időbélyeggel ellátott biztonsági adatokat a célzott mobileszközökön (okostelefonok, PDA-k). (iii) Tudásalapú időbeli absztrakciós (KBTA) módszertannal dolgozza fel az adatokat. (iv) Könnyűsúlyú (lightweight) HIDS-szel és központi menedzsment képességekkel kombinálva értékelték ki Android-alapú mobiltelefonokon. |

### Táblázat 6.8: Együttes (Ensemble) Módszerek Anomália Detekcióra

| Módszer | Leírás |
| :--- | :--- |
| Osztályspecifikus együttes IDS modell [65] | (i) Együttes megközelítést mutat be két osztályozó, a Bayes-hálózatok (BN) és a Klasszifikációs és Regressziós Fák (CART) kombinálásával. (ii) Egy hibrid architektúrát épít be különböző jellemzőkiválasztó algoritmusok kombinálására a jobb eredmények eléréséhez valós behatolásjelzés esetén. |
| Klaszter együttes [264] | (i) Egy klaszter-együttes (cluster ensemble) megközelítés a sokkal "természetesebb" klaszterezések megtalálására többszörös klaszterezések kombinálásával, feltételezve, hogy az adatok több nézőpontja (multiple views) javítja a támadások észlelését. (ii) Minden klaszterezés értékeli, hogy egy pont mennyire anomális. Az értékeléseket átlagolással, vagy a minimum, maximum, illetve medián pontszámok figyelembevételével kombinálják. |
| Egy-osztályos SVM osztályozó modell [293] | Egy megközelítés nagysebességű payload-alapú anomália IDS építésére egy-osztályos (one-class) SVM osztályozók együttesének használatával, azzal a céllal, hogy pontos legyen, és nehéz legyen kijátszani (evade). |
| Heterogén osztályozó együttes [49] | Négy alaposztályozó (ANN, SVM, kNN és döntési fák) kimeneteit fuzionálja három kombinációs stratégiával: (i) többségi szavazás (majority voting), (ii) Bayes-féle átlagolás (Bayesian averaging) és (iii) egy hiedelem-mérőszám (belief measure) alapján. |
| McPAD modell [292] | (i) A McPAD (Multiple classifier Payload-based Anomaly Detector) egy payload-alapú anomália detekciós rendszer, amely egy-osztályos osztályozók együtteséből áll. (ii) Az anomália detektor nagyon pontos az olyan hálózati támadások észlelésében, amelyek valamilyen formájú shell kódot hordoznak a rosszindulatú payloadban. |
| GEdIDS modell [120] | (i) Egy elosztott adatbányászati algoritmus a detekciós pontosság javítására a rosszindulatú vagy jogosulatlan hálózati tevékenységek osztályozásakor. (ii) Az együttes (ensemble) paradigmával kiterjesztett genetikus programozás (GP) felhasználásával fejlesztették ki. (iii) Az adatok több autonóm helyszínen (sites) oszlanak el, és a tanuló komponens kooperatív módon gyűjt hasznos tudást az adatokból. (iv) A hálózati profilt használja az abnormális viselkedés előrejelzésére. |
| FRaC modell [272] | (i) A félig-felügyelt (semi-supervised) anomália detekció egy megközelítése, amely egyetlen osztályos (single-class) példákból álló tanítóhalmazra épülő prediktorok együttesére épül. (ii) Egy újszerű, információelméleti anomália-mérőszám, amely a kísérleti eredmények alapján jól működik zajos és irreleváns jellemzők jelenlétében is. |
| Klaszter-alapú együttes [268] | (i) Az egyéni osztályozók mind a bemeneti jellemzőteret (input feature space), mind egy k-means klaszterezés által adott kiegészítő jellemző-részhalmazt felhasználva épülnek fel. (ii) Az együttes kombinációját az osztályozók különböző lokális adatszegmenseken (amelyeket a k-means adott) elért osztályozási képessége alapján számítják ki. |

### Táblázat 6.9: Anomália Detekció Fúziós Módszerekkel

| Módszer | Leírás |
| :--- | :--- |
| MCS modell [136] | (i) Egy hálózati behatolásjelző módszert vezet be többszörös osztályozók fúziója (fusion) alapján. (ii) Öt döntési-fúziós módszert értékel ki és hasonlítja össze a teljesítményüket. |
| SVO modell [118] | (i) Egy eseményvezérelt (event-driven) behatolásjelző architektúrát vezet be, amely Alany-Ige-Tárgy (Subject-Verb-Object - SVO) többpontos monitorokat és egy hatás-elemző (impact analysis) motort integrál. (ii) Riasztás-fúziós (alert fusion) és ellenőrzési (verification) modelleket implementál, hogy sokkal észszerűbb behatolási információt szolgáltasson az SVO monitorok által begyűjtött hiányos, inkonzisztens vagy pontatlan riasztásokból. |
| D-S és PCA algoritmus [64] | (i) Heterogén adatszintű (data level) fúziót vezet be hálózati anomália detekcióhoz. (ii) A technika kidolgozásához a Dempster-Shafer bizonyíték-elméletet és a PCA-t (Főkomponens Elemzést) használja. (iii) Emulációs és szimulációs környezetekben is értékelték ki. |
| dLEARNIN rendszer [278] | (i) Egy együttes (ensemble) megközelítést mutat be a többszörös forrásból származó információk intelligens kombinálására. (ii) Kifejezetten a hibák költségének (cost of errors) minimalizálására van hangolva, szemben magával a hibaaránnyal (error rate). (iii) Jobb teljesítményt ér el. |
| NAS modell [132] | (i) Két módszert egyesít (merges) többszörös-forrású (multisource) információfúziós technológiával. (ii) Pontosabb döntést biztosít a gyanús viselkedéseknél azzal, hogy több információt gyűjt be a rendszer különböző szintjeiről. (iii) Automatikusan képes frissíteni a profilokat. |
| IDEA modell [147] | (i) Egy ANN-alapú (Neurális Háló) módszert mutat be adatfúzió és behatolás-elemzés végrehajtására, valamint a nemkívánatos információk kimetszésére (prune) a multiszenzorokból. (ii) Magas észlelési pontosságot ér el. |
| HMMPayl modell [22] | (i) A payload-ot (hasznos adat) bájtok sorozataként (sequence of bytes) ábrázolja. (ii) HMM (Rejtett Markov Modell) használatával végez elemzést. (iii) Kinyeri a jellemzőket és HMM-et használ, hogy garantálja ugyanazt a kifejezőerőt, mint az n-gram elemzés, miközben csökkenti a számítási komplexitást. (iv) A Multiple Classifiers System (Többszörös Osztályozó Rendszer) paradigmát követi, hogy jobb osztályozási pontosságot biztosítson, növelve az IDS kijátszásának (evading) nehézségét. (v) Enyhíti a nem-optimális HMM paraméterválasztásból eredő gyengeségeket. |

### Táblázat 6.10: Hibrid Anomália Detekció

| Módszer | Leírás |
| :--- | :--- |
| DHABIDERS modell [193] | (i) Egy módszert mutat be a DoS támadásokat okozó aktív támadások, valamint a hozzáférési pont (access point) felderítő eszközök használatának észlelésére vezeték nélküli hálózatokban. (ii) Valós időben működik, riasztásokat küldve a hálózati adminisztrátornak. |
| FLIPS modell [235] | (i) Módszert mutat be a bináris kód-injekciós támadások megelőzésére a FLIPS (Feedback Learning Intrusion Prevention System) használatával. (ii) Három fő komponenst tartalmaz: egy anomália-alapú osztályozót, egy szignatúra-alapú szűrési sémát és egy felügyeleti keretrendszert, amely Utasításkészlet-Randomizációt (Instruction Set Randomization - ISR) alkalmaz. (iii) Befogja az injektált kódot, lehetővé téve a FLIPS számára, hogy szignatúrákat készítsen a zero-day exploitokhoz. |
| Véletlen erdők (Random forests) hibrid módszer [418] | (i) Egy olyan módszerről számol be, amely a visszaélés-felismerést (misuse detection) és az anomália-alapú behatolásjelzést ötvözi a véletlen erdők (random forests) algoritmus használatával. (ii) Jobb detekciós rátát ér el. |
| DT-SVM hibrid modell [289] | (i) Egy módszert mutat be a döntési fák (DT) és a support vector machine-ok (SVM) kombinálására, mint egy hierarchikus hibrid intelligens rendszermodell a behatolásjelzéshez. (ii) Maximalizálja a detekciós pontosságot és minimalizálja a számítási komplexitást. |
| E-FSM modell [36] | (i) Egy HIDS (Hoszt-alapú IDS) architektúrát mutat be a konvergált VoIP (Voice over IP) alkalmazásokhoz. (ii) A kommunikáló kiterjesztett véges állapotgépek (communicating extended finite state machines) formális modelljét használja, hogy állapottartó (stateful) és protokollok közötti (cross-protocol) észlelést biztosítson. (iii) Kombinálja a szignatúra-alapú és specifikáció-alapú észlelési technikákat a protokoll-szintaxis és -szemantika kombinálásával. |
| RFIDS modell [421] | (i) Egy szisztematikus keretrendszert mutat be, amely a véletlen erdők algoritmust alkalmazza visszaélés-, anomália- és hibrid-hálózat alapú IDS-ekben. (ii) Jobb teljesítményt mutat több adathalmazon. |
| Hibrid ANN-alapú modell [44] | (i) Egy hibrid módszert mutat be a felderítetlen támadások hatásainak felmérésére Neurális Hálózatok (ANN) segítségével. (ii) GA-alapú (Genetikus algoritmus) megközelítést is használ. (iii) Implementál egy sztochasztikus modellt, amely lehetővé teszi a rendszerbiztonság költség/haszon elemzését (cost/benefit analysis), és megalapozott döntések meghozatalát a behatolásjelzéssel és rendszervédelemmel kapcsolatban. |
| Hibrid IDS modell [277] | (i) Automatizált rendszert vezet be DoS támadások detektálására. (ii) Kétfázisú architektúraként tervezték, amely a korai támadás-azonosításhoz "változási-pont" (change-point) detekciós módszertant, a támadás jelenlétének megerősítéséhez pedig további spektrális profilalkotást (spectral profiling) foglal magába. |
| Hibrid RBF/Elman neurális hálózat modell [370] | (i) Egy hibrid RBF/Elman neurális hálózat modellt alkalmaz mind anomália detekcióhoz, mind visszaélés-felismeréshez (misuse detection). (ii) Hatékonyan detektálja az időben szétszórt (temporally dispersed) és kollaboratív támadásokat a múltbeli események memóriájának köszönhetően. |
| Hibrid IDS [146] | Egy anomália előfeldolgozót használ, amely kiterjeszti a SNORT IDS funkcionalitását. |
| Hibrid osztályozó (Hybrid classifier) [76] | (i) Egy jellemzőkiválasztó osztályozó (feature-selecting classifier) és egy adatbányászati osztályozó együttesét vezeti be hálózati anomáliák azonosítására. (ii) A két osztályozót fuzionálja a végső döntés meghozatalához. |
| Intelligens hibrid IDS [413] | (i) Egy ANN-alapú intelligens hibrid IDS modellt mutat be. (ii) A modell rugalmas, és kiterjeszthető különböző hálózati környezetekhez, javítva a detekciós teljesítményt és pontosságot. |
| Hibrid többszintű (multilevel) IDS modell [330] | (i) Egy hibrid intelligens IDS-t mutat be az ismert és ismeretlen támadások észlelési rátájának (detection rate) javítására. (ii) Több szintből áll: hibrid neurális hálózatokból és döntési fákból. |
