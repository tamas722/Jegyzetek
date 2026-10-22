# 7. Fejezet: Kiértékelési Módszerek (Evaluation Methods)

Köztudott, hogy semmi sem tökéletesen és örökké biztonságos. Egy rendszer (pl. egy NIDS) minőségének vagy pontosságának kiértékelése mindig csak egy pillanatkép az időben. Ahogy telik az idő, a környezet megváltozik, új sebezhetőségek (vulnerabilities) jelennek meg, így a kiértékelést és a paraméterek hangolását újra meg kell ismételni.

A könyv ebben a fejezetben 16 különböző kiértékelési mérőszámot tárgyal három átfogó kategóriára bontva:

1. **Helyesség (Correctness / Accuracy)**
2. **Adat (Data)**
3. **Hatékonyság (Efficiency / Performance)**

> 📍 **[IDE JÖN A KÉP: Figure 7.1: 3-D perspective of evaluation measures]**
>
> Az ábra egy 3D-s tengelyrendszerben ábrázolja ezt a három kategóriát.

## 7.1 Pontosság (Accuracy / Helyesség)

A pontosság egy NIDS teljesítményét méri a helyesség szempontjából: megmutatja a detekciós és a hibaarányt, valamint a rendszer által generált téves riasztások (false alarms) számát.

Ha egy NIDS pontossága $95\%$, az azt jelenti, hogy $100$ esetből $95$-öt a valós osztályába sorolt (akár normál, akár támadás).

> **A gyakorlati probléma:** A hálózatokban a támadó csomagok száma töredéke a normál csomagokénak (erősen kiegyensúlyozatlan adatok), a támadások módja pedig hihetetlenül változatos. Emiatt a modern NIDS rendszerek még mindig rengeteg téves riasztást generálnak.

A NIDS pontosságát az alábbi öt (5) mérőszámmal értékeljük:

### 7.1.1 Érzékenység és Specificitás (Sensitivity and Specificity)

Ezek a mérőszámok a hálózati forgalmat egy 2-osztályos (Normál vs. Anomália) problémaként kezelik.

- **Pozitív címke (Positive):** Támadás / Anomália.
- **Negatív címke (Negative):** Normál forgalom.

Amikor a NIDS döntést hoz, 4 lehetséges kimenetel létezik (Ezt hívjuk **Tévesztési Mátrixnak / Confusion Matrix**):

- **True Positive (TP) - Valós Pozitív:** A NIDS helyesen támadásnak minősített egy valódi támadást. (Találat).
- **False Positive (FP) - Téves Riasztás:** A NIDS támadásnak minősített egy valójában normál, legitim csomagot. (Fals riasztás).
- **True Negative (TN) - Valós Negatív:** A NIDS helyesen normálnak minősített egy valódi normál csomagot.
- **False Negative (FN) - Téves Negatív / Elvétés:** A NIDS normálnak minősített egy valójában támadó csomagot. (Átengedte a behatolót).

> 📍 **[IDE JÖN A KÉP: Figure 7.2: Decision possibilities for 2-class problem] és [Figure 7.3: An illustration of TP, TN, FP and FN]**

- **Érzékenység (Sensitivity / Hit rate / Detekciós ráta):** $TP / (TP + FN)$. Azt méri, hogy az összes valós támadásból mennyit sikerült elkapni. Akkor élvez prioritást, ha a rendszert minden áron védeni kell (pl. bank).
- **Specificitás (Specificity):** $TN / (FP + TN)$. Azt méri, hogy az összes valós normál forgalomból mennyit ismert fel helyesen. Akkor élvez prioritást, ha a hatékonyság a lényeg, és el akarjuk kerülni a dolgozók munkájának (normál forgalom) blokkolását.

### 7.1.2 Tévesztési Ráta (Misclassification Rate)

Azt méri, hogy a NIDS hányszor "téved" a jóslataiban az összes esethez képest.

- **Képlete:** Helytelen predikciók $(FN + FP)$ osztva az összes predikcióval $(TP + FP + FN + TN)$.

### 7.1.3 Tévesztési Mátrix (Confusion Matrix)

N-osztályos (többkategóriás) problémáknál használatos, amikor nem csak azt kell eldönteni, hogy valami támadás-e, hanem azt is, hogy milyen támadás (pl. DoS, Probe, U2R).

**Példa:** Van $100$ eset ($45$ Normál, $35$ DoS, $15$ Probe, $5$ U2R). A mátrixban a sorok a Valós (Actual) osztályokat, az oszlopok a Jósolt (Predicted) osztályokat jelentik.

> 📍 **[IDE JÖN A KÉP: Figure 7.4: Confusion matrix for a 4-class problem]**

**Hogyan kell olvasni?** A bal felső saroktól a jobb alsó sarokig tartó átló (diagonal) tartalmazza a helyes predikciókat. Minden elem, ami az átlón kívül (nondiagonal) esik, hibás osztályozást jelent (pl. amikor egy DoS támadást a rendszer Normálnak hitt).

### 7.1.4 Precízió, Visszahívás és F-mérték (Precision, Recall, F-measure)

- **Precízió (Precision):** $TP / (TP + FP)$. Azt méri, hogy a rendszer által "támadásnak" riasztott esetek közül mennyi volt tényleg támadás. Ha alacsony, a rendszer túl sok fals riasztást ad.
- **Visszahívás (Recall):** Megegyezik az Érzékenységgel ($TP / (TP + FN)$). Elméletileg $100\%$-os Recall elérhető úgy, ha a rendszer mindent támadásnak minősít – de ez tönkretenné a Precíziót. A kettő között kompromisszum (trade-off) van.
- **F-mérték (F-measure / F1-score):** A Precízió és a Visszahívás harmonikus átlaga. Képlete:

```math
2 \times \frac{Precision \times Recall}{Precision + Recall}
```

Többosztályos hálózatbiztonsági problémáknál ez a leginkább preferált metrika. Akkor $1$ (vagy $100\%$), ha nincs téves riasztás ($FP=0$) és minden támadást észlel ($FN=0$).

### 7.1.5 ROC Görbék (Receiver Operating Characteristics)

A ROC görbét a Valós Pozitív Arány (TP ráta, Y-tengely) és a Téves Riasztási Arány (FP ráta, X-tengely) közötti vizuális kapcsolat bemutatására használják.

> 📍 **[IDE JÖN A KÉP: Figure 7.5: Receiver operating characteristics]**

**Hogyan kell olvasni a ROC görbét a NIDS-eknél?**

- **$(0,0)$ Bal alsó sarok:** $100\%$-os pontosság a normál adatokra ($0$ riasztás), de $0\%$ a támadásokra. Az IDS nem detektál semmit.
- **$(1,1)$ Jobb felső sarok:** Az IDS riasztást generál minden bejövő csomagra. $100\%$ detekció, de $100\%$ fals riasztás is.
- **$(0,0)$ és $(1,1)$ közötti egyenes átló:** A véletlenszerű (random) tippelés vonala. Egy valós IDS görbéjének mindig ez felett kell lennie.
- **$(0,1)$ Bal felső sarok:** Az Ideális IDS. $100\%$ detekciós ráta, $0\%$ téves riasztás. Minél jobban "felpúposodik" a ROC görbe a bal felső sarok felé, annál jobb (pontosabb) az IDS.

## 7.2 Teljesítmény (Performance)

A NIDS teljesítményének kiértékelése túlmegy a puszta "jól talált-e" kérdésen. Magában foglalja a hardverplatformot, az operációs rendszert és a telepítés módját is.

A legfontosabb teljesítménymutató: **Képes-e a rendszer egy nagysebességű hálózaton minimális csomagvesztéssel (packet loss), valós időben feldolgozni a forgalmat?**

Mivel a hálózatban a csomagok mérete (payload) változó, az IDS CPU- és memória-kihasználtsága (memory usage) ad indirekt képet a rendszer idő- és térkomplexitásáról. A teljesítmény nagyban függ attól, hogy a hálózat melyik pontjára (pl. Edge router vagy belső switch) helyezik el a NIDS-et.

## 7.3 Teljesség (Completeness)

Ez a mérőszám azt értékeli, hogy egy NIDS képes-e megvédeni a rendszert az összes ismert támadás (vulnerabilities and attacks) ellen.

Elméletben kiváló metrika, a gyakorlatban azonban szinte lehetetlen alkalmazni, mert ehhez a biztonsági szakembereknek birtokában kéne lennie az összes valaha létező támadási vektornak (complete knowledge), ami a folyamatosan fejlődő fenyegetések (zero-day attacks) korában kivitelezhetetlen. Egy valóban teljes megoldásnak muszáj valamilyen anomália-alapú mechanizmust is beépítenie.

## 7.4 Időszerűség / Gyorsaság (Timeliness)

Egy hálózati anomália detekciós rendszernek a lehető leggyorsabban kell elvégeznie az elemzést és a riasztást, hogy a rendszergazda még azelőtt beavatkozhasson, mielőtt a támadó komoly kárt okozna, vagy kompromittálná magát az IDS rendszert.

**A Késleltetés ($\delta$) kiszámítása:** A rendszer kiértékelésének egyik legkritikusabb mérőszáma az időbeli késleltetés (delay). Ha a támadás bekövetkezésének ideje $T_A$ (Time of Attack), és a rendszer válaszának (riasztásának) ideje $T_R$ (Time of Response), akkor a késleltetés:

```math
\delta = T_R - T_A
```

Minél kisebb a $\delta$ értéke, annál jobb a NIDS időszerűségi (timeliness) teljesítménye. Hiába $100\%$-os a pontosság, ha a detektálási idő órákban vagy napokban mérhető!

## 7.5 Stabilitás (Stability)

Egy robusztus NIDS-től elvárás, hogy konzisztensen viselkedjen különböző hálózati forgatókönyvek (scenarios) esetén is.

**Feltétel:** Teljesen azonos hálózati eseményekre mindig pontosan ugyanolyan riasztási üzeneteket kell generálnia. Ha a felhasználóknak megengedik, hogy túlságosan testreszabják a riasztásokat a különböző környezetekben, az a rendszer instabil állapotához és kétértelműséghez (ambiguity) vezethet.

## 7.6 Interoperabilitás / Együttműködési képesség (Interoperability)

Egy nagyvállalati környezetben ritkán fut egyetlen védelmi vonal. Gyakran több különböző IDS (anomália-alapú NIDS, szignatúra-alapú NIDS, hoszt-alapú HIDS) dolgozik párhuzamosan, különböző gyártóktól (vendors).

**A lényeg:** Egy hatékony behatolásjelző mechanizmusnak képesnek kell lennie arra, hogy összekapcsolja (correlate) ezekből a különböző forrásokból, valamint a tűzfalakból és rendszernaplókból (system logs) származó információkat.

## 7.7 Adatminőség, Érvényesség és Megbízhatóság (Data Quality, Validity and Reliability)

A betanításhoz használt (training) adatok hasznossága három fő paraméteren múlik:

### 1. Minőség (Quality)

A minőséget a következő tényezők befolyásolják:

- **Forrás:** Megbízhatónak és megfelelőnek kell lennie.
- **Mintavétel:** Elfogulatlannak (unbiased) kell lennie.
- **Mintaméret:** El kell kerülni a túl- és alul-mintavételezést (over/under-sampling).
- **Időbeliség:** Friss adatokra van szükség.
- **Komplexitás:** Az adatnak elég egyszerűnek kell lennie a könnyű kezelhetőséghez.

### 2. Érvényesség (Validity)

Azt vizsgálja, hogy az adat valóban azt reprezentálja-e, amit elemezni akarunk.

- **Belső érvényesség (Internal validity):** Azt méri, mennyire vagyunk biztosak abban, hogy a detektált hatást tényleg az a dolog (ok) váltotta ki, amit vizsgálunk, és nem egy külső, zavaró tényező (confounding variables).
- **Külső érvényesség (External validity):** A módszer valós világban való alkalmazhatóságát és általánosíthatóságát méri.
- **Kompromisszum:** Ha nagyon szigorítjuk a belső érvényességet laboratóriumi körülmények között, azzal gyakran rontjuk a külső érvényességet (a való életbeli használhatóságot). A kiugró értékek (outliers) és a hiányzó adatok statisztikai szűrése kritikus a validitás növeléséhez.

### 3. Megbízhatóság (Reliability)

Az adathalmaznak teljesnek, következetesnek (egyértelműnek) és pontosnak kell lennie.

- **Kappa-analízis (Kappa-analysis):** Egy gyakori statisztikai megbízhatósági teszt, amely két független megfigyelő (observer) közötti egyezési pontszámot (match score) számszerűsíti. A skála $0$ és $1$ között mozog. Az $1$ a tökéletes egyezést, a $0$ pedig azt jelenti, hogy az egyezés pusztán a véletlen (chance) műve.

## 7.8 Riasztási Információ (Alert Information)

A NIDS által generált riasztásoknak kellően értelmesnek (meaningful) kell lenniük. Egy jó riasztás azonnal megválaszolja az alábbi három kérdést az elemző (analyst) számára:

1. Mi volt az ok, ami kiváltotta az eseményt?
2. Miért érdekes/veszélyes ez az esemény?
3. Ki a támadás forrása (Source) és mi a célpontja (Target)?

## 7.9 Ismeretlen Támadások Észlelése (Unknown Attacks Detection)

A kiberbiztonságban szinte minden nap születnek új sebezhetőségek (exploits). Ami nincs benne a szignatúra-adatbázisban, az "ismeretlen" támadás.

Egy ideális NIDS-nek felügyelt (supervised) mechanizmusokkal kell fognia az ismert támadásokat, és felügyelet nélküli (unsupervised) mechanizmusokkal kell rendelkeznie a még sosem látott, módosított vagy zero-day behatolások felismerésére.

## 7.10 Referenciák Frissítése (Updating References)

Ha a rendszer felfedezett egy új sebezhetőséget, képesnek kell lennie a profilok és a referenciák (szabályok) frissítésére, hogy a jövőben ezt már "ismert" támadásként blokkolhassa.

**A modern kihívás:** Egy nagysebességű hálózatban új szabályokat írni vagy módosítani valós időben (real time) úgy, hogy azok ne okozzanak logikai ütközést (conflict) a már meglévő szabályokkal, az egyik legnehezebb feladat.

## 7.11 Összegzés (Discussion)

A 7. fejezet bemutatta, hogyan lehet a NIDS rendszereket helyesség (accuracy), adatminőség (data) és hatékonyság (efficiency) szempontjából kiértékelni.

**A legfőbb tanulság:** Egyetlen rendszer kiértékelése sem örökérvényű. Egy elvégzett teszt pusztán egy "pillanatkép az időben" (snapshot in time). Ahogy a környezet változik, a hálózat bővül és új támadási vektorok jelennek meg, a korábbi értékelések érvényüket vesztik, és a rendszert új paraméterekkel újra kell értékelni.
