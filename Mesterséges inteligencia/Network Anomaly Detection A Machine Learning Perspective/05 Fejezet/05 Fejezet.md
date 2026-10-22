# 5. Fejezet: Jellemzőkiválasztás (Feature Selection)

## Bevezetés (Introduction)

A jellemzőkiválasztás (ismert még mint változókiválasztás / variable selection, jellemzőcsökkentés / feature reduction vagy attribútumkiválasztás) a gépi tanulás és az adatbányászat elengedhetetlen része. Célja egy robusztus és költséghatékony tanulási modell felépítése a releváns jellemzők (features) egy részhalmazának kiválasztásával.

Különösen fontos a valós, magas dimenziójú (high-dimensional) adatok elemzésekor (pl. hálózati behatolások, biológiai adatok, bűnügyi nyomozások). A felügyelt osztályozásban (supervised classification) a feature selection a következőképp javítja a modellt:

- Enyhíti a "dimenziók átkát" (high dimensionality problem).
- Növeli a modell általánosító képességét (generalization) azáltal, hogy csökkenti a túltanulás (overfitting) esélyét.
- Jelentősen gyorsítja a tanulási (betanítási) folyamatot.
- Segíti az adatok megértését azáltal, hogy rávilágít, mely attribútumok a legfontosabbak és hogyan kapcsolódnak egymáshoz.

## 5.1 Jellemzőkiválasztás vs. Jellemzőkinyerés (Feature Selection vs. Feature Extraction)

Bár mindkét módszer célja az adathalmaz dimenziószámának (a változók számának) csökkentése, a két megközelítés gyökeresen eltér egymástól:

- **Jellemzőkiválasztás (Feature Selection):**
  - Az eredeti jellemzők egy részhalmazát (subset) tartja meg. Nem változtatja meg az adatokat, csak "kiszűri" a feleslegeseket.
  - Olyan metrikák alapján választ, mint az információs nyereség (information gain), korreláció vagy kölcsönös információ (mutual information).
  - **Példa:** Van $50$ hálózati feature. Kiválasztjuk a $10$ legjobbat (pl. Cél port, TCP flag, Bájtok száma), a többi $40$-et egyszerűen eldobjuk. A megmaradó adatok továbbra is ember által értelmezhetők.
- **Jellemzőkinyerés (Feature Extraction):**
  - Matematikai (lineáris vagy nemlineáris) transzformációt alkalmaz az adatokon, és egy teljesen új, alacsonyabb dimenziós teret hoz létre ($k < n$).
  - A legismertebb példák a Főkomponens-analízis (PCA - Principal Component Analysis) és a Szinguláris Érték Felbontás (SVD - Singular Value Decomposition).
  - **Példa:** Az $50$ eredeti feature-ből matematikai kombinációval csinál $10$ teljesen újat (pl. az "Új_Feature_1" = $0.4 \times \text{Port} + 0.6 \times \text{IP}$). Ennek hátránya, hogy elveszíti a fizikai/emberi értelmezhetőségét.

> 📍 **[IDE JÖN A KÉP: Figure 5.1: Feature selection vs. feature extraction]**
>
> Az ábra vizuálisan szemlélteti, ahogy a Selection egyszerűen kihagy elemeket (szűrés), míg az Extraction egy transzformációs mátrixszal új teret képez le.

## 5.2 A Jellemzők Relevanciája (Feature Relevance)

Egy jellemzőkiválasztó algoritmus a "releváns" jellemzőket keresi. De mit is jelent pontosan a relevancia? A szakirodalomban több különböző matematikai és logikai definíció létezik rá:

- **Kohavi és John (A leginkább elfogadott definíció):**
  - **Erősen releváns (Strongly relevant):** Ha ezt a feature-t eltávolítjuk, a gépi tanulási modell (classifier) teljesítménye egyértelműen romlik. (Hálózati ML Példa: Portscan felismerésénél a "Célport" attribútum erősen releváns).
  - **Gyengén releváns (Weakly relevant):** Önmagában nem erősen releváns, de ha egy olyan részhalmazt távolítunk el, amiben benne van, akkor a modell teljesítménye romlik. (Tehát bizonyos más feature-ökkel együtt hordoz hasznos információt).
  - **Irreleváns (Irrelevant):** Sem erősen, sem gyengén nem releváns. Eltávolítása semmilyen hatással nincs a modellre (vagy akár javítja is azt). (Hálózati ML Példa: A hálózati kártya MAC címének utolsó 2 bitje egy globális DDoS támadás felismerésénél).
- **Hall és Smith (Korrelációs alapú):**
  - Egy jellemzőhalmaz akkor jó, ha a benne lévő jellemzők erősen korrelálnak a célosztállyal (pl. Támadás/Normál), de egymással nem korrelálnak (nincs köztük redundancia).
- **Almuallim és Dietterich:**
  - Egy feature akkor releváns, ha része egy olyan optimális szabályrendszernek, amely képes hibátlanul megkülönböztetni az összes osztályt.
- **Sugumaran et al.:**
  - A feature releváns, ha az osztályozási folyamat során felépített Döntési Fa (Decision Tree) részét képezi (a fa beépített "kiválasztó" tulajdonsága miatt).
- **Dash és Liu (Inkonzisztencia ráta alapján):**
  - A releváns jellemzőhalmaz az, aminek a legalacsonyabb az inkonzisztencia rátája. (Inkonzisztens egy adat, ha ugyanaz a feature-minta egyszer "Normál", máskor "Támadás" címkét kap).

## 5.3 A Jellemzőkiválasztás Előnyei (Advantages)

Az ML modellbe illesztett Feature Selection lépés öt fő előnnyel jár:

1. **Dimenziócsökkentés:** Felgyorsítja az algoritmus működését, csökkenti a számítási (CPU/RAM) igényt.
2. **Zajszűrés:** Eltávolítja a redundáns, irreleváns vagy egyenesen zajos (félrevezető) adatokat.
3. **Adatminőség javítása:** A tisztább adatbemenet jobb felépítésű tanulási mechanizmust eredményez.
4. **Pontosság (Accuracy) növelése:** A felesleges adatok kidobásával a modell kevésbé fog "túltanulni" (overfit), így élesben jobban fog teljesíteni.
5. **Adatmegértés (Data understanding):** Segít a fejlesztőnek megérteni az adatokat generáló folyamatokat és vizualizálni a kritikus paramétereket.

## 5.4 A Jellemzőkiválasztás Alkalmazási Területei (Applications)

A modern rendszerekben az objektumokat gyakran több ezer attribútummal írják le. A Feature Selection kulcsfontosságú előfeldolgozási (preprocessing) lépés lett az alábbi területeken:

> 📍 **[IDE JÖN A KÉP: Figure 5.2: Feature selection in various applications]**
>
> Az ábra mutatja a feature selection kapcsolatát az osztályozással (classification), klaszterezéssel (clustering), asszociációs szabályokkal és mintázat-illesztéssel.

### 5.4.1 Bioinformatika (Bioinformatics)

Biológiai adatok (pl. DNS szekvenciák, fehérje struktúrák) tárolása és elemzése. Különösen a génkifejeződési (gene expression) adatoknál kritikus, ahol gyakran több tízezer dimenzió (gén) van, de csak nagyon kevés minta (páciens). A feature selection segít megtalálni azokat a specifikus géneket (co-expressed patterns), amelyek egy betegségért felelősek.

### 5.4.2 Hálózatbiztonság (Network Security)

A Behatolásjelző Rendszerek (IDS) fejlesztésében óriási probléma a hálózati forgalomból származó adatok magas dimenziószáma és a feature-ök közötti komplex, rejtett kapcsolatok.

**Miért kritikus itt?** Egy IDS-nek valós időben (real-time) kell felismernie a támadásokat. Ha a modellnek másodpercenként százezer csomagot kell átvizsgálnia $50$ attribútum mentén, a számítási overhead miatt a rendszer lemarad (delay). A feature selection segítségével az $50$ attribútumból mondjuk $8$ relevánsat választanak ki, így a detektálás azonnali lesz, és a téves riasztások száma (False Alarms) is csökken.

### 5.4.3 Szövegkategorizálás (Text Categorization)

Bemeneti szöveges dokumentumok (cikkek, emailek) előre definiált kategóriákba sorolása (pl. Spam-szűrés vagy téma-felismerés).

**A probléma:** Egy átlagos dokumentumhalmazban az egyedi szavak vagy kifejezések (mint feature-ök) száma könnyen elérheti a több tízezret. Ez kezelhetetlen a legtöbb ML algoritmus számára. A jellemzőkiválasztás megkeresi azokat a kulcsszavakat, amelyek ténylegesen hordozzák az osztályozási információt (pl. a "nyeremény", "ingyen" szavak erősen relevánsak a Spam detekcióban, a kötőszavak irrelevánsak).

### 5.4.4 Biometria (Biometrics)

A felhasználók identitásának ellenőrzésére vagy hitelesítésére (authentication) olyan biometrikus jellemzőket használnak, mint az ujjlenyomat, a kézgeometria, az arckép és a hang.

**A Feature Selection szerepe:** Ahhoz, hogy egy embert a lehető leggyorsabban és legpontosabban lehessen megkülönböztetni a több millió másiktól, nem kell az ujjlenyomat vagy az arc összes mikroszkopikus képpontját vizsgálni. Egy optimális, releváns jellemzőkből álló részhalmaz (subset) kiválasztása jelentősen javítja a hitelesítési rendszerek teljesítményét és sebességét.

### 5.4.5 Tartalom-alapú Képkeresés (Content-Based Image Retrieval - CBIR)

A CBIR rendszerek képeket elemeznek a tartalmuk (szín, textúra, forma, térbeli elrendezés) és a hozzájuk tartozó szöveges leírások alapján.

**A Feature Selection szerepe:** Amikor egy felhasználó egy képre keres (pl. Google Image Search), az adatbázis óriási multimédiás tartalmakból áll. A magas memória- és számítási igény (főleg mobiltelefonokon vagy elosztott rendszereken) szűk keresztmetszetet (bottleneck) jelent. Az optimális jellemzőkiválasztás csökkenti az indexelési és lekérdezési folyamat komplexitását, miközben megőrzi (vagy akár javítja) a keresés pontosságát.

## 5.5 Korábbi felmérések (Surveys) a Jellemzőkiválasztásról

A statisztikusok már a 19. századtól kezdve kutatják az optimalizált jellemzőhalmazok kiválasztásának problémáját. Ez a könyvfejezet egy átfogó, 1970-es évektől napjainkig tartó felmérést (survey) nyújt a témáról.

Az algoritmusok kategorizálása a következő három tengely mentén történik:

1. **Alkalmazott megközelítés:** Szűrő (Filter), Csomagoló (Wrapper), Beágyazott (Embedded) vagy Hibrid.
2. **Alkalmazott módszer/matematika:** Statisztikai, Információelméleti, Klaszterező, Asszociációs, Lágy-számítási (Soft computing) vagy Együttes (Ensemble).
3. **Célterület (Osztályok száma alapján):** 2-osztályos (bináris), n-osztályos (multiclass), vagy többcímkés (multilabeled).

> 📍 **[IDE JÖN A KÉP: Figure 5.3: Statistics of the methods introduced during the years 1990 to 2012]**
>
> Az ábra egy kördiagram, amely bemutatja a különböző módszerek eloszlását. Ebből látszik, hogy a legtöbb megközelítés statisztikai szűrő - statistical filter - módszereken alapul.

### 5.5.1 Összehasonlítás a korábbi tanulmányokkal (A Comparison with Prior Surveys)

Sok átfogó tanulmány (survey) létezik a témában (pl. Kohavi és John, Liu és Motoda, stb.). Ezek általában az alábbiakat vizsgálták:

- Irrelevancia vs. Relevancia.
- Redundancia vs. Relevancia.
- Mintanagyság (Sample size).
- Az algoritmusok viselkedése mesterséges és valós adatbázisokon.

**Miben tér el (és miben jobb) ez a könyvfejezet a korábbi tanulmányoknál?**

A könyv egy 12 témát felölelő mátrixban (T1-T11) hasonlítja össze magát a korábbi publikációkkal. Fő eltérések:

- **Gyakorlatiasság (Practicality):** Egy 3-szintű, gyakorlatias taxonómiát (osztályozási rendszert) javasol a feature selection módszerekre.
- **Eszközök bemutatása:** Nemcsak elméletet ad, hanem listázza a jellemzőkiválasztást, osztályozást és kiértékelést támogató szoftveres eszközöket és rendszereket (Tools/Systems).
- **Irányelvek (Guidelines) és Ajánlások:** Konkrét tanácsokat ad a kutatóknak, hogy milyen adattípushoz milyen algoritmust érdemes választani.
- **Kihívások:** Mind elméleti, mind gyakorlati szempontból felvázolja a jövőbeli kutatási problémákat (Research issues and challenges).

## 5.6 A Probléma Megfogalmazása (Problem Formulation)

A Jellemzőkiválasztás (Feature Selection) problémája matematikailag a következőképpen definiálható:

Célunk egy optimális részhalmaz (subset), azaz $F'$ azonosítása az eredeti $F$ jellemzőhalmazból (feature set) egy adott $D$ adathalmazban, amely a lehető legjobb osztályozási pontosságot (classification accuracy) nyújtja.

**Formális definíció:**
Adott egy címkézett adathalmaz $D$, amelynek jellemzői $f_1, f_2, f_3 \dots f_n \in F$. A jellemzőkiválasztás az a folyamat, amely megkeresi az eredeti $F$ halmaz egy $F'$ részhalmazát (vagyis $F' \subseteq F$), ahol az alábbi két feltétel teljesül:

1. $|F'|$ (a kiválasztott jellemzők száma) optimális (azaz a lehető legkisebb, amivel még nem veszítünk információt).
2. A $D$ adathalmazon elvégzett osztályozás (classification) pontossága az $F'$ részhalmaz használatával a lehető legjobb.

> *(Röviden: A lehető legkevesebb adatból a lehető legpontosabb döntést meghozni).*

## 5.7 A Jellemzőkiválasztás Lépései (Steps in Feature Selection)

A jellemzőkiválasztás egy többlépcsős folyamat. Ahhoz, hogy egy nyers adathalmazból megkapjuk az optimális jellemző-részhalmazt (feature subset), négy alapvető lépésen kell végigmenni:

1. **Jelölt részhalmazok generálása (Generating candidate feature subsets)**
2. **Jelölt részhalmazok kiértékelése (Evaluating candidate subsets)**
3. **A részhalmazok megfelelőségének ellenőrzése (Checking goodness of candidate subsets / Megállási kritérium)**
4. **Az eredmény validálása (Validating the satisfied subset)**

> 📍 **[IDE JÖN A KÉP: Figure 5.4: Basic steps of feature selection]**
>
> Az ábra ezt a négy, körkörösen visszacsatolt folyamatot mutatja be: Generálás $\rightarrow$ Kiértékelés $\rightarrow$ Megállási Kritérium (ha nem, vissza a Generáláshoz) $\rightarrow$ Validálás.

### 5.7.1 Részhalmaz Generálása (Subset Generation)

Ha egy adathalmaz $n$ darab jellemzőt (feature) tartalmaz, akkor matematikailag $2^n$ darab lehetséges részhalmaz létezik. Magas dimenziószámú adatoknál (pl. hálózati behatolások, ahol az $n$ lehet akár $41$ vagy több ezer is) ez a szám csillagászati, így "brute-force" (nyers erő) módszerrel végigpróbálni az összes kombinációt lehetetlen.

A generálási stratégiákat 3 fő kategóriába soroljuk:

#### 5.7.1.1 Véletlenszerű Generálás (Random Subset Generation)

A részhalmazokat véletlenszerűen generálják, majd kiértékelik. A vizsgált halmazok száma drasztikusan kevesebb, mint $2^n$.

- **Hatékonysága:** Kizárólag a kiválasztást végző véletlenszám-generáló / valószínűségi funkció minőségétől függ.

#### 5.7.1.2 Heurisztikus Generálás (Heuristic Subset Generation)

Egy kezdeti állapotból indulva, a megoldási teret (solution space) heurisztikus (célirányos) lépésekkel járja be egy célérték felé. Három iránya van:

- **Előrehaladó (Forward):** Egy teljesen üres halmazzal kezd, majd heurisztikus kritériumok alapján (pl. melyik feature adja a legnagyobb pontosságot) egyesével adja hozzá a jellemzőket.
- **Hátrafelé haladó (Backward):** A teljes ($n$ elemű) halmazzal kezd, majd egyesével távolítja el a legkevésbé hasznos / legrosszabb jellemzőket.
- **Kétirányú (Bidirectional):** Egyszerre indul az üres és a teljes halmazból, és párhuzamosan ad hozzá és vesz el jellemzőket.

**Gyakorlati tapasztalat:** A Backward és a Bidirectional technikák a legtöbb esetben jobb eredményt adnak, mint a Forward szelekció.

#### 5.7.1.3 Teljes Generálás (Complete / Exhaustive Subset Generation)

Kivétel nélkül az összes lehetséges ($2^n$) részhalmazt legenerálja és megvizsgálja. Csak nagyon kis dimenziószámú adatoknál (kevés feature esetén) alkalmazható.

### 5.7.2 Részhalmaz Kiértékelése (Feature Subset Evaluation)

A legenerált részhalmazokat egy kiértékelő kritérium (Evaluation criteria) alapján kell pontozni. Két kategóriára bontható:

#### 5.7.2.1 Függő Kritérium (Dependent Criteria - Csomagoló/Wrapper módszerek alapja)

A részhalmazt aszerint értékeli ki, hogy egy adott gépi tanulási algoritmus (pl. döntési fa, SVM) milyen eredményt (pontosságot) ér el vele.

- **Belső (Internal):** Nincs előzetes tudás a címkékről, csak a klaszterezés homogenitását / szeparációját nézi.
- **Külső (External):** Előzetes tudás (címkék) alapján nézi a pontosságot.

**Jellemző:** Számításigényes (drága), de rendkívül hatékony az optimális részhalmaz megtalálásában, mivel közvetlenül a modell teljesítményére optimalizál.

#### 5.7.2.2 Független Kritérium (Independent Criteria - Szűrő/Filter módszerek alapja)

Nem futtatja le a gépi tanulási algoritmust az értékeléshez. Helyette a jellemzők belső, statisztikai tulajdonságait használja (pl. információs nyereség, korreláció, jellemzők közötti hasonlóság/különbözőség). Gyorsabb, de nem feltétlenül adja a legjobb eredményt a végső osztályozónál.

### 5.7.3 Megfelelőségi / Megállási Kritérium (Goodness Criteria)

Ez a kritérium dönti el, hogy mikor állhat le a generálási és kiértékelési ciklus (mikor vagyunk "elégedettek" a részhalmazzal).

A döntés azon múlik, hogy a **Sebesség (Speed)** vagy a **Pontosság (Accuracy)** a fontosabb a feladatban:

- **Pontosság fókusz:** Nagyon szigorú leállási feltétel.
- **Sebesség fókusz:** Lazább feltétel.

**Példa egy megállási kritériumra:** A ciklus akkor áll le, ha egy újabb feature hozzáadása (Forward szelekcióban) már nem javítja az algoritmus eredményét, tehát elértük a lokális maximumot.

### 5.7.4 Eredmény Validálása (Result Validation)

Az utolsó lépésben a kiválasztott "optimális" részhalmazt ellenőrizni (validálni) kell.

#### 5.7.4.1 Külső Validáció (External Validation)

- **Feltétel:** Ismernünk kell az "előre tudott" optimális eredményt (Ground Truth).
- **Működés:** Összehasonlítja a mi algoritmusunk által talált részhalmazt az elvárt optimális részhalmazzal. Főként mesterséges (szintetikus) adathalmazoknál alkalmazható, ahol a fejlesztők pontosan tudják, melyik feature a fontos.
- **Metrikák:** Precízió (Precision), Visszahívás (Recall), F-mérték, ROC görbe, Tévesztési mátrix (Confusion matrix), Tisztaság (Purity).

#### 5.7.4.2 Belső Validáció (Internal Validation)

- **Feltétel:** Nem használunk előzetes tudást vagy külső referenciát az optimális halmazról.
- **Működés:** A kapott megoldás elvárt belső tulajdonságait vizsgálja. Például a klaszterezésnél használt belső érvényességi indexek (homogenitás, négyzetes hibaösszeg) itt is használhatók.

**Alapelv a Feature Selectionben:** Ha minden egyes feature-t egy "objektumnak" tekintünk, a jó részhalmazban a feature-ök közötti különbözőségnek (dissimilarity) nagynak kell lennie (azaz ne legyenek redundánsak, ne korreláljanak egymással).

> 📍 **[IDE JÖN A KÉP: Figure 5.5: A taxonomy of feature selection techniques]**
>
> Ez az ábra a következő alfejezetekben részletezett taxonómiát vizualizálja: Filter, Wrapper, Embedded, Hybrid módszerek.

## 5.8 Jellemzőkiválasztási Módszerek: Egy Taxonómia (A Taxonomy)

A jellemzőkiválasztó technikákat egy átfogó, háromszintű taxonómia (osztályozási rendszer) alapján csoportosíthatjuk. A szintek a feladat jellegét, a működés elvét és a matematikai modellt írják le.

### 1. Szint: Osztályinformáció jellege (A hálózati adatok kimenete alapján)

- **Kétosztályos (Two-class / Bináris):** Az adatok pontosan két kategóriába sorolhatók (pl. Normál vs. Támadás).
- **Többosztályos (Multiclass):** Az adatok több mint két kategóriába tartozhatnak (pl. Normál, DoS, Probe, U2R, R2L). Egy objektum pontosan egy osztályba tartozik.
- **Többcímkés (Multilabeled):** Egy objektum egyszerre több osztályba is tartozhat (pl. egy csomag egyszerre lehet port scan és sebezhetőség-keresés).

### 2. Szint: Működési elv (Architektúra)

- **Szűrő (Filter) módszerek:** A jellemzőket valamilyen relevancia-mérőszám alapján rangsorolják. Sorban hozzáadják a magas pontszámúakat, vagy eltávolítják az alacsonyakat, amíg el nem érnek egy megállási (jósági) kritériumot. Függetlenek a gépi tanulási algoritmustól.
- **Csomagoló (Wrapper) módszerek:** Hasonlóan generálnak részhalmazokat, de a kiértékelést és a jósági kritériumot egy konkrét osztályozó (classifier) kimenetén (pl. annak pontosságán) mérik le.
- **Beágyazott (Embedded) módszerek:** A kiválasztás maga az osztályozó felépítésének (tanulásának) szerves része. A tanuló modul (inducer) profilépítés közben egyszerűen csak a releváns attribútumokat veszi figyelembe (tipikus példa erre a Döntési Fa).
- **Hibrid (Hybrid) módszerek:** A Szűrő és a Csomagoló módszerek előnyeit ötvözi. Egy független mérőszámot ÉS egy osztályozót is használ a jósági kritérium megállapításához (elkerülve az előre kőbe vésett megállási szabályokat).

### 3. Szint: Alkalmazott Modell

Ezek alapján lehetnek: Statisztikai (Statistical), Információelméleti (Information theoretic), Lágy-számítási (Soft computing), Klaszterező / Asszociációs (Clustering / Association rule mining) és Együttes (Ensemble) technikák.

### Táblázat 5.2: Jellemzőkiválasztó Módszerek Összehasonlítása

| Módszer / Szerző | 1. Szintű Kategória | 2. Szintű Kategória | 3. Szintű Kategória | Használt Mérőszám |
| :--- | :--- | :--- | :--- | :--- |
| Dash és Liu [90] | Többosztályos | Szűrő | Statisztikai | Inkonzisztencia ráta |
| Kira és Rendell [201] | Kétosztályos | Szűrő | Statisztikai | Relevancia |
| Liu et al. [233] | Többosztályos | Szűrő | Statisztikai | Inkonzisztencia ráta |
| Kohavi és John [204] | Többosztályos | Csomagoló | Statisztikai | N/A |
| Liu és Setiono [232] | Többosztályos | Beágyazott | Statisztikai | $\chi^2$ pontszám |
| Das [87] | Többosztályos | Hibrid | Információelméleti | Információs nyereség |
| Yu és Liu [411] | Többosztályos | Szűrő | Információelméleti | Szimmetrikus bizonytalanság |
| Hall és Smith [155] | Többosztályos | Szűrő | Információelméleti | Heurisztikus jóság |
| Peng et al. [290] | Többosztályos | Csomagoló | Információelméleti | mRMR kritérium |
| Fleuret [119] | Többosztályos | Szűrő | Információelméleti | CMIM kritérium |
| Zhong et al. [428] | Többosztályos | Szűrő | Lágy-számítás | N/A |
| Hu et al. [178] | Többosztályos | Szűrő | Lágy-számítás | Függőség |
| Lee et al. [218] | Többosztályos | Csomagoló | Lágy-számítás | Fuzzy entrópia |
| Setiono és Liu [332] | Többosztályos | Csomagoló | Lágy-számítás | N/A |
| Yang és Honavar [398] | Többosztályos | Csomagoló | Lágy-számítás | N/A |
| Zhang et al. [423] | Többcímkés | Csomagoló | Lágy-számítás | Hamming veszteség, rangsorolási veszteség |
| Almuallim és Dietterich [13] | Bináris (Kétosztályos) | Beágyazott | Klaszterezés / ARM | N/A |
| Mitra et al. [257] | Többcímkés | Beágyazott | Klaszterezés / ARM | N/A |
| Sugumaran et al. [359] | Többcímkés | Beágyazott | Klaszterezés / ARM | N/A |

*(Megjegyzés: Az 1., 2. és 3. szintű kategóriák az 5.8-as szakasz taxonómiájára utalnak.)*

## 5.9 Meglévő Jellemzőkiválasztó Módszerek (Existing Methods)

Ebben a részben a leggyakrabban hivatkozott (és a fenti táblázatban is szereplő) algoritmusok részletes logikáját tekintjük át.

### 5.9.1 Statisztikai Jellemzőkiválasztás (Statistical Feature Selection)

A statisztikai módszerek matematikai egyenletekben (kritériumokban) formalizálják a jellemzők és az osztálycímkék közötti kapcsolatot.

#### 1. Dash és Liu - Inkonzisztencia ráta (Inconsistency rate)

- **Alapelv:** Egy jellemző-részhalmaz "jóságát" aszerint méri, hogy az adatok mennyire egyediek és egyértelműek a címkézés szempontjából.
- **Számítása:** Egy adott mintázat (pattern) inkonzisztenciája egyenlő a mintázat összes előfordulásának számából kivonva annak az osztálynak a darabszámát, ahova a legtöbbször tartozott. (Vagyis: hányszor "tévedt" a minta).
- **Kiértékelés:** Az inkonzisztencia rátát (összes inkonzisztencia osztva az összes mintával) veti össze egy felhasználó által megadott küszöbértékkel. Keresésre teljes (exhaustive), heurisztikus és hibrid módokat is használnak.

#### 2. Kira és Rendell - A "Relief" algoritmus

- **Alapelv:** Kétosztályos szűrő (Filter) technika. Minden jellemzőhöz egy nulla értékkel inicializált relevancia súlyvektort rendel.
- **Működése (Iteratív):**
  1. Véletlenszerűen kiválaszt egy adatpontot.
  2. Megkeresi hozzá a legközelebbi azonos osztályba tartozó pontot (nearhit).
  3. Megkeresi a legközelebbi eltérő osztályba tartozó pontot (nearmiss).
  4. A jellemző súlyát (vektorát) növeli az eltérő osztálytól vett négyzetes távolsággal, és csökkenti az azonos osztálytól vett négyzetes távolsággal.
- **Eredmény:** Azok a jellemzők lesznek kiválasztva, amelyeknek a súlya egy adott küszöb fölé nő.

#### 3. Liu et al. - Las Vegas Algoritmus

- **Alapelv:** Valószínűségi (Probabilistic) megközelítés.
- **Működése:** A Las Vegas algoritmus minden iterációban egy véletlenszerű (random) jellemző-részhalmazt generál. Ha ez az új részhalmaz kevesebb jellemzőt tartalmaz, mint az eddig talált legjobb (best feature subset), és a jósága (amit Dash és Liu inkonzisztencia rátájával mérnek) is megfelelő, akkor ez lesz az új legjobb halmaz.

#### 4. Kohavi és John - Csomagoló (Wrapper) Módszer

- **Alapelv:** Keresést hajt végre a részhalmazok terében egy tanuló algoritmus (induction algorithm) bevonásával.
- **Működése:** Előrehaladó (Forward) szelekcióval indul (hegymászó / hill climbing és best-first stratégiával). Hogy a hátrafelé (Backward) haladó kizárás számítási idejét csökkentsék, bevezetik a kombinált operátort (compound operator), ami a legjobb $i+1$ lépéseket vonja össze.
- **Értékelés:** 5-szörös kereszt-validációt (5-fold cross validation) használnak ismétlődően a stabilitás eléréséig.

#### 5. Liu és Setiono - $\chi^2$ (Chi-négyzet) alapú beágyazott módszer

- **Alapelv:** Diszkretizálja (felosztja) a jellemzők értékeit $\chi^2$ statisztika segítségével.
- **Működése:**
  1. A jellemző értékeit $n$ darab intervallumra osztja.
  2. Kiszámolja a $\chi^2$ értéket minden szomszédos intervallumpárra.
  3. A legalacsonyabb $\chi^2$ értékű (tehát a leginkább hasonló) szomszédos intervallumokat összevonja. Ezt addig folytatja, amíg minden pár $\chi^2$ értéke meg nem halad egy küszöböt.
- **Kiválasztás:** A küszöböt lépésről lépésre csökkentik, egészen addig, amíg az inkonzisztencia ráta nőni nem kezd. Ha egy jellemző minden értékét egyetlen intervallumba vonja össze az algoritmus, az a jellemző irreleváns, így eldobják.


### 5.9.2 Információelméleti Jellemzőkiválasztás (Information Theoretic Feature Selection)

Az információelméleti technikák a valószínűségre és a Shannon-féle információelméletre támaszkodnak. Olyan mérőszámokat használnak, mint az entrópia (entropy), a kölcsönös információ (mutual information) és az információs nyereség (information gain), hogy felmérjék a jellemzők és az osztálycímkék (pl. Támadás/Normál), vagy két jellemző közötti korrelációt.

#### 1. Das - Hibrid módszer információs nyereséggel

- **Működés:** Egy iteratív folyamat során mindig azt a még ki nem választott jellemzőt adja a halmazhoz, amelyik a legmagasabb információs nyereséget (information gain) adja.
- **Súlyozás:** Azokat a tanító példákat, amiket a modell korábban elrontott (misclassified), magasabb súllyal veszi figyelembe a következő lépésben (hasonló a Boosting elvéhez).
- **Megállási kritérium:** Ha a tanítási pontosság (training accuracy) már nem javul az új jellemző hozzáadásától, a folyamat leáll.

#### 2. Yu és Liu - Szimmetrikus Bizonytalanság (Symmetric Uncertainty)

- **Alapelv:** Szűrő (Filter) módszer. A szimmetrikus bizonytalanság a két jellemző közötti információs nyereség kétszerese osztva a saját entrópiáik összegével.
- **Kétlépéses szűrés:**
  1. Kiválasztja azokat a jellemzőket, amelyek erősen korrelálnak az Osztály-címkével (magas szimmetrikus bizonytalanság a céllal).
  2. Megvizsgálja a kiválasztott jellemzőket egymáshoz képest. Ha két jellemző között magas a szimmetrikus bizonytalanság (tehát erősen korrelálnak egymással), az egyiket eldobja, mert redundáns.

#### 3. Hall és Smith - Heurisztikus Szűrő (Filter)

- **Működés:** Mohó hegymászó (greedy hill climbing) keresést használ egy "érdem" (merit) függvény alapján.
- **Feltétel:** Az a jó jellemzőhalmaz, ahol a jellemzők erősen korrelálnak a célosztállyal, de egymással nem korrelálnak (feltételes entrópia alapján mérve).

#### 4. Peng et al. - mRMR (Max-Relevance and Min-Redundancy)

- **Cél:** Kölcsönös információ (mutual information) alapján maximalizálni a relevanciát az osztályhoz, és minimalizálni a redundanciát a jellemzők között.
- **Működés (2 fázis):**
  1. Keres egy jelölt részhalmazt, ahol stabilan alacsony az osztályozási hiba.
  2. Ezt a jelölt halmazt tovább tömöríti egy előrehaladó (Forward) és hátrafelé haladó (Backward) Csomagoló (Wrapper) módszerrel.

#### 5. Fleuret - CMIM (Conditional Mutual Information Maximization)

- **Alapelv:** Biztosítja, hogy egy olyan jellemző, ami önmagában nagyon erős előrejelzője az osztálynak, de túlságosan hasonlít egy már korábban kiválasztott jellemzőhöz, NE kerüljön be az optimális halmazba (mert nem ad új információt).

### 5.9.3 Lágy-számítási Módszerek (Soft Computing Methods)

A lágy-számítási modellek gyakran hajlandóak feláldozni a $100\%$-os pontosság egy részét cserébe a sokkal gyorsabb megoldásért és a bizonytalanság (uncertainty) jobb kezeléséért. Ide tartoznak a Fuzzy halmazok, Neurális hálók, Durva halmazok (Rough sets) és Genetikus algoritmusok.

- **Zhong et al. (Durva halmazok / Rough Sets):**
  - Szűrő (Filter) módszer. Először megkeresi a "mag" (core) jellemzőket, majd egyesével ad hozzá új jellemzőket addig, amíg el nem éri az optimális részhalmazt (reduct).
- **Hu et al. (NFARNRS):**
  - Naiv Előrehaladó (Forward) Attribútum Csökkentés. Üres halmazzal kezd, és mindig azt a jellemzőt adja hozzá, ami a legnagyobb mértékben növeli a függőséget (dependency). Ha a növekmény egy küszöb alá esik, leáll. Képes heterogén adatokat kezelni.
- **Lee et al. (Fuzzy Entrópia):**
  - K-means klaszterezéssel és Fuzzy entrópiával dolgozik. Minél kisebb a fuzzy entrópia, annál relevánsabb a jellemző. Hátrafelé haladó (Backward) kizárással dobálja ki a legmagasabb entrópiájú jellemzőket, amíg a pontosság el nem kezd romlani.
- **Setiono és Liu (Neurális Hálózatok - ANN):**
  - Háromrétegű, előrecsatolt (feedforward) neurális hálót használ Hátrafelé (Backward) kizárással. A teljes hálót betanítja az összes jellemzővel, majd egyesével kiveszi őket. Amelyiknek az eltávolítása a legjobb (vagy legkevésbé rossz) eredményt adja, azt véglegesen kiveszi, amíg a pontosság esése meg nem halad egy küszöböt.
- **Yang és Honavar (Genetikus Algoritmus - GA):**
  - Evolúciós folyamat (mutáció és keresztezés). A populáció tagjai bináris vektorok ($1$ = a jellemző kiválasztva, $0$ = nincs). A "rátermettséget" (fitness) egy neurális hálós osztályozó pontossága adja meg.
- **Zhang et al. (Többcímkés / Multilabel GA):**
  - Naiv Bayes osztályozót és Genetikus Algoritmust használ többcímkés adatokhoz. A jósági (fitness) funkció a Hamming-veszteség és a Rangsorolási veszteség minimalizálásán alapul.

### 5.9.4 Klaszterezési és Asszociációs Bányászati Megközelítés (Clustering and ARM)

Ebben a megközelítésben a jellemzőkiválasztás maga az adatbányászati (szabályképző vagy csoportosító) folyamat szerves, beágyazott (Embedded) része.

- **Almuallim és Dietterich (Szabályalapú):**
  - Bináris jellemzőkre. A modell szabályokat generál a célosztályok megkülönböztetésére. Amelyik jellemző egyetlen szabályban sem szerepel, azt egyszerűen irrelevánsnak minősíti és eldobja. A cél, hogy a szabályok a lehető legkevesebb jellemzőből álljanak.
- **Mitra et al. (Klaszterezés alapú szűrés):**
  - Kétlépéses felügyelet nélküli (unsupervised) módszer. Először a jellemzőket (nem az adatpontokat!) klaszterezi KNN (K-Nearest Neighbors) algoritmussal. Mivel az egy klaszterbe eső jellemzők nagyon hasonlítanak egymásra (redundánsak), a második lépésben minden klaszterből kiválasztanak egyetlen reprezentatív jellemzőt, a többit eldobják.
- **Sugumaran et al. (Döntési Fák - Decision Trees):**
  - A döntési fák felépítése ösztönösen beágyazott (embedded) jellemzőkiválasztást végez. A fa minden csomópontja egy jellemző. A gyökérhez közelebbi jellemzők a legfontosabbak. Ami nem kerül be a fába, az irreleváns. A kevésbé diszkrimináló (levél szintű) jellemzők egy küszöbbel szintén levághatók.

### 5.9.5 Együttes Megközelítés (Ensemble Approach)

Az együttes (ensemble) módszerek alapgondolata, hogy több különböző egyéni prediktor (osztályozó) kimenetének kombinálása jobb és stabilabb eredményt ad, mint bármelyik prediktor önmagában.

- **Feltétel:** A siker kulcsa, hogy az egyéni prediktorok diverzek (különbözőek) legyenek, így a statisztikai torzításaik (biases) kioltják egymást.
- **Módszerek:** Bagging, Boosting, Stack generalization.

**Ismert Ensemble Jellemzőkiválasztó Algoritmusok:**

- **Breiman - Random Forests (Véletlen Erdők):** Több döntési fa kombinációja. Amellett, hogy remek osztályozó, a hatalmas adathalmazokban kiválóan képes rangsorolni a jellemzők fontosságát (feature ranking).
- **Friedman - Gradient Boosting Trees:** Fokozatosan javuló fák algoritmusa prediktív modellezéshez.
- **Tuv et al.:** Fák párhuzamos és soros kombinációjával próbálják felfedni a "maszkolást" (változó maszkolás), hogy biztosan megtalálják az irreleváns jellemzőket.
- **ANN + RFE kombináció:** Rekurzív Jellemzőkizárás (Recursive Feature Elimination - RFE) és Neurális Hálózatok együttese. Különösen jól teljesít extrém magas dimenziószámú adatoknál (pl. mikrobiológiai / génkifejeződési adatok).
- **Puuronen et al.:** Különböző heurisztikákon (alap, korreláció-alapú, kontextuális érdem-alapú) nyugvó sokszínű együtteseket hoztak létre a jellemzők szűrésére.


  ## 5.10 Részhalmaz-kiértékelési Mérőszámok (Subset Evaluation Measures)

A különböző jellemzőkiválasztó (feature selection) technikák eltérő matematikai mérőszámokat használnak annak eldöntésére, hogy egy jellemző mennyire releváns. Ezek a mérőszámok alkotják a szűrő (Filter) algoritmusok magját.

### 5.10.1 Inkonzisztencia Ráta (Inconsistency Rate)

Az inkonzisztencia ráta azt méri, hogy egy adott jellemző-kombináció (mintázat) mennyire egyértelműen határozza meg az osztálycímkét.

**Képletek:**

Egy adott $pat$ mintázat inkonzisztencia száma (Inconsistency count) a vizsgált $F$ jellemzőhalmazon:

```math
Inconsistency\_count_F(pat) = n_{pat} - \max(n_{pat}^A, n_{pat}^B, n_{pat}^C, n_{pat}^D)
```

*(Ahol $n_{pat}$ a mintázat összes előfordulása, a $n_{pat}^A \dots n_{pat}^D$ pedig az egyes osztálycímkékhez tartozó előfordulások száma. Vagyis levonjuk az összesből a leggyakoribb osztály darabszámát).*

A teljes $D$ adathalmaz inkonzisztencia rátája az $F$ részhalmazon:

```math
Inconsistency\_rate_F(D) = \frac{\sum_{pat} Inconsistency\_count_F(pat)}{n}
```

### 5.10.2 Relevancia (Relevance)

Kira és Rendell Relief algoritmusa használja. Nem determinisztikus mérőszám, ami azt mutatja, mennyire felelnek meg a jellemző értékei az osztályinformációnak.

**Működése:** Random kiválaszt egy $P$ adatpontot. Keres hozzá egy azonos osztályba tartozó legközelebbi szomszédot (nearest hit - H) és egy eltérő osztályba tartozó legközelebbi szomszédot (nearest miss - M).
A jellemző $f_j$ relevanciáját iteratívan frissíti: növeli az eltérő osztálytól vett távolsággal, és csökkenti az azonos osztálytól vett távolsággal.

### 5.10.3 Szimmetrikus Bizonytalanság (Symmetric Uncertainty)

Információelméleti mérőszám, ami azt vizsgálja, mennyire képes egy minta megjósolni egy másik minta értékeit (Yu és Liu).

```math
Symmetric\_uncertainty(P,Q) = 2 \left[ \frac{IG(P \mid Q)}{H(P) + H(Q)} \right]
```

*(Ahol $H$ az entrópia, $IG$ pedig az információs nyereség. Értéke $0$ és $1$ között mozog).*

### 5.10.4 Függőség (Dependency)

Azt méri, hogy az osztálycímkék ($C$) mennyire függnek egy $S$ jellemző-részhalmaztól (Durva halmazok / Rough Sets elmélet alapján).

```math
Dependency_S(C) = \frac{POS_S(C)}{|U|}
```

*(Ahol $POS_S(C)$ a pozitív régióba tartozó objektumok száma, amelyeket bizonyossággal be lehet sorolni egy döntési osztályba az $S$ részhalmaz alapján, $|U|$ pedig az összes objektum száma).*

### 5.10.5 Fuzzy Entrópia (Fuzzy Entropy)

A mintaeloszlási információk kiértékelésére szolgál. Egy $c_i$ osztály fuzzy entrópiája az $A$ fuzzy halmazban:

```math
Fuzzy\_Entropy_{c_i}(A) = -M_{c_j} \log_2 M_{c_j}
```

*(Ahol $M_{c_j}$ az egyezési fok - match degree -, amely megmutatja a fuzzy tagság és az osztálytagság közötti egyezés mértékét).*

### 5.10.6 Hamming Veszteség (Hamming Loss)

Többcímkés (multilabeled) adathalmazokon alkalmazzák annak mérésére, hogy egy többcímkés osztályozó mennyire kategorizál félre (misclassify) objektumokat.

```math
Hamming\_loss(h, D) = \frac{1}{n} \sum_{i=1}^{n} \frac{1}{n_c} |h(o_i) \Delta l_i|
```

*(Megmutatja a modell által jósolt címkék $h(o_i)$ és a valós címkék $l_i$ közötti szimmetrikus különbséget, normalizálva az osztályok számával).*

### 5.10.7 Rangsorolási Veszteség (Ranking Loss)

A Hamming veszteséghez hasonlóan ez is többcímkés esetekben méri, hogy a kimeneti címkékből hány pár van rosszul sorba rendezve (misordered).

---

## 5.11 Rendszerek és Eszközök a Jellemzőkiválasztáshoz (Systems and Tools for Feature Selection)

Számos szoftveres könyvtár (library) tartalmaz beépített jellemzőkiválasztó algoritmusokat. Ezek ismerete elengedhetetlen a gépi tanulással foglalkozó kutatók számára.

**(i) SciLab**
Numerikus számításokra használt szoftverkönyvtár, amely felhasználóbarát környezetet és könnyű szkriptelést biztosít. A `mutual information toolbox` csomagja implementálja az mRMR algoritmust (Peng et al.) és képességeket biztosít az entrópia, együttes és feltételes valószínűségek számítására.

**(ii) Python**
A világ egyik legelterjedtebb programozási nyelve a gépi tanulásban.
- **`PyML` csomag:** Implementál egy SVM-alapú Rekurzív Jellemzőkizáró (RFE) technikát, valamint egy Filter technikát.
- **A Filter technika "golub" pontszáma:** A jellemző-osztály korrelációt úgy értékeli ki, hogy kiszámolja az osztályok átlagai közötti különbséget a jellemzőn keresztül, súlyozva a szórással (standard deviation).

**(iii) R nyelv**
Erőteljes nyelv és eszköz statisztikai és gépi számításokhoz.
- **`Boruta` csomag:** Random Forest osztályozó iteratív tanulásával találja meg a fontos attribútumokat.
- **`Caret` csomag:** Rugalmas csomag az RFE (Recursive Feature Elimination) variánsaihoz.
- **`FSelector` csomag:** Kiemelkedően rugalmas csomag, számámos modullal.

### Táblázat 5.3: Néhány Jellemzőkiválasztó Funkció az FSelector csomagban (R nyelv)

| Funkció (Function) | Leírás (Description) |
| :--- | :--- |
| `best.first.search` | Jellemző-részhalmazokat keres a "legjobb az első" (best first) kereséssel |
| `exhaustive.search` | Jellemző-részhalmazokat keres teljes (exhaustive) kereséssel |
| `greedy.search` | Jellemző-részhalmazokat keres mohó (greedy) kereséssel |
| `hill.climbing.search` | Jellemző-részhalmazokat keres hegymászó (hill climbing) kereséssel |
| `cfs` | Jellemző-részhalmazt választ korreláció-alapú jellemzőkiválasztással |
| `chi.squared` | Súlyokat rendel a diszkrét attribútumokhoz a chi-négyzet ($\chi^2$) teszt alapján |
| `consistency` | Jellemző-részhalmazt választ folytonos és diszkrét adatokhoz egy inkonzisztencia mérőszám segítségével |
| `correlation` | Súlyokat rendel a folytonos jellemzőkhöz a folytonos osztály-attribútumokkal való korrelációjuk alapján |
| `cutoff` | Kiválaszt egy részhalmazt a rangsorolt jellemzők közül |
| `entropy.based` | Súlyokat rendel a diszkrét jellemzőkhöz a folytonos osztály-attribútumokkal való korrelációjuk alapján |
| `oneR` | Súlyokat rendel a diszkrét jellemzőkhöz az egyes jellemzőkhöz generált asszociációs szabályok alapján |
| `random.forest.importance` | Súlyokat rendel a jellemzőkhöz a Random Forest algoritmus segítségével |
| `relief` | Súlyt rendel a folytonos és diszkrét jellemzőkhöz a véletlenszerűen mintavételezett esetek közötti távolságok alapján |

**(iv) MATLAB**
Széles körben használt számítási eszköz és programozási nyelv.
- **`Machine Learning Tool Box`:** Támogatja a teljes (exhaustive) és szekvenciális (előrehaladó és hátrafelé) keresést, 10-szeres keresztvalidációval és KNN osztályozóval.
- **`MIToolbox`:** Kölcsönös információn (mutual information) alapuló kiválasztást támogat (implementálja a CMIM, mRMR módszereket).
- **`FEAST`:** (Feature Selection Toolbox for C and MATLAB) Tartalmazza a RELIEF, CMIM, mRMR technikákat.

**(v) Octave (GNU Octave)**
Nagyon hasonló a MATLAB-hoz, a szkriptek könnyen portolhatók a kettő között. A `NAN` csomag implementálja az FSS (Feature Subset Selection) metódust, ami az mRMR (max-relevance-min-redundancy) technikára épül.

**(vi) Weka**
A Weka egy grafikus felhasználói felülettel (GUI) rendelkező, rendkívül felhasználóbarát adatbányászati eszköz. A kutatói közösség körében nagyon népszerű, mert programozási tudás nélkül használható.
Kiemelkedően rugalmas felületet biztosít a feature selection-höz: a felhasználó szabadon választhat a keresési stratégiák (search scheme) és a kiértékelők (evaluators) közül, továbbá beépített eszközökkel rendelkezik a kicsinyített adathalmaz (reduced data) egyszerű vizualizálására.

<Ide kép kell ????????????????????????????????????????????????????????????????????????????????????????????????????????????????????

### 5.11 Rendszerek és Eszközök a Jellemzőkiválasztáshoz (Folytatás)

**(vii) Orange**
Az Orange egy adatbányászati és gépi tanulási szoftvercsomag, amely barátságos grafikus felhasználói felülettel (GUI), valamint Python kötésekkel és szkriptkönyvtárakkal rendelkezik. A jellemzőkiválasztást az `OrngFSS` modul támogatja.

#### Táblázat 5.4: Néhány Funkció az orngFSS modulban

| Funkció (Function) | Leírás (Description) |
| :--- | :--- |
| `attMeasure` | Pontszámokat rendel a diszkrét jellemzőkhöz egy adott pontozó (scoring) függvény segítségével |
| `bestNAtts` | Visszaad $n$ darab magasan rangsorolt jellemzőt a pontszámlistából |
| `attsAbove` | Visszaadja azokat a jellemzőket, amelyek pontszáma meghalad egy adott küszöbértéket |
| `selectBestNAtts` | Visszaad egy adathalmazt a felhasználó által meghatározott számú legjobb jellemzővel |
| `selectAttsAboveThresh` | Visszaad egy adathalmazt azokkal a jellemzőkkel, amelyek pontszáma meghalad egy felhasználó által meghatározott küszöböt |
| `filterRelieff` | Jellemzőket választ ki egy adathalmazból a hátrafelé haladó kizárás (backward elimination) módszerével, amíg minden megmaradt jellemző pontszáma egy felhasználó által megadott küszöb felett nem lesz |

**(viii) RapidMiner**
A RapidMiner az egyik legszélesebb körben használt adatbányászati szoftvercsomag (GUI-val és hatalmas Java könyvtárral a fejlesztőknek). A `com.rapidminer.operator.features.selection` csomag külön operátor osztályokat biztosít a feature selection-höz.

#### Táblázat 5.5: Néhány Operátor Osztály a RapidMiner-ben

| Operátor Osztály (Function) | Leírás (Description) |
| :--- | :--- |
| `AttributeWeightSelection` | Kiválaszt egy jellemző-részhalmazt, amelynek súlyai megfelelnek egy megadott feltételnek |
| `BackwardElimination` | Jellemző-részhalmazt választ a hátrafelé haladó kizárás (backward elimination) módszerével |
| `BestSelection` | Kiválasztja a legjobb jellemző-részhalmazt |
| `BruteForceSelection` | Kiválasztja a legjobb jellemzőhalmazt az összes lehetséges kombináció végigpróbálásával |
| `ForwardSelection` | Jellemző-részhalmazt választ egy előrehaladó (forward selection) technika segítségével |
| `GeneticAlgorithm` | Genetikus algoritmust használ a jellemzőkiválasztáshoz |
| `IterativeFeatureAdding` | Iteratívan ad hozzá jellemzőket sorrendben egy megadott jellemzőnév-tömbből |

**(ix) TOOLDIAG**
A TOOLDIAG egy C nyelven írt statisztikai mintafelismerő (pattern recognition) könyvtár. Rugalmas keresési stratégiákat kínál (pl. a legjobb jellemzők, szekvenciális előrehaladó szelekció, szekvenciális hátrafelé haladó szelekció). A felhasználók olyan kiértékelési kritériumok közül választhatnak, mint az osztályok közötti távolság (interclass distance) vagy a valószínűségi távolság.

---

## 5.12 Összegzés (Discussion)

A jellemzőkiválasztó (Feature Selection) módszerek áttekintése során a kutatók (és a könyv szerzői) az alábbi kulcsfontosságú tanulságokat (lessons learned) vonták le arról, hogy melyik módszer milyen körülmények között működik jól:

- **Adattípusok kezelése:** Egy igazán jó jellemzőkiválasztó módszernek képesnek kell lennie mind numerikus, mind kategorikus (és a kettő keverékéből álló, ún. mixed) adatok kezelésére.
- **Szűrő (Filter) módszerek:** Számítási kapacitásban korlátozott helyzetekben (pl. valós idejű hálózati behatolásjelzésnél / NIDS) a Szűrő módszer a preferált, mert nagyon gyors. Ugyanakkor egy jó statisztikai szűrőnek egyaránt kezelnie kell a jellemző-jellemző (feature-feature) (redundancia) és a jellemző-osztály (feature-class) (relevancia) korrelációt.
- **Csomagoló (Wrapper) módszerek:** Ha a számítási költség/idő nem számít, a Csomagoló módszer sokkal jobb eredményt (pontosságot) ad, mint a Szűrő. Hátránya: Lassú, és hajlamos az "osztályozói torzításra" (classifier bias), mivel túlságosan rátanul egy specifikus ML algoritmusra.
- **Többcímkés (Multilabel) hiányosság:** Jelenleg nagyon kevés olyan feature selection módszer létezik, amely jól kezeli azokat az adatbázisokat, ahol egy objektumnak (pl. egy hálózati csomagnak) egyszerre több osztálycímkéje is lehet. Ez egy nyitott kutatási terület.

**A Jövő Iránya:**
Bár rengeteg módszer létezik, nincs egyetlen "Szent Grál" (univerzális megoldás), ami minden problémára jó lenne. A különböző technikák más-más igényeket elégítenek ki.

A legnagyobb kihívás jelenleg a Csomagoló (Wrapper) technikák felgyorsítása, hogy időkritikus (time-constrained) alkalmazásokban is használhatók legyenek. A kutatók szerint a megoldás az **inkrementális osztályozók (incremental classifiers)** fejlesztése lehet: ezek olyan modellek, amelyeknél a betanító (training) adatok apró megváltozásakor (egy-egy feature eltávolításakor) nem kell a nulláról, a teljes adathalmazon újra betanítani a modellt.
