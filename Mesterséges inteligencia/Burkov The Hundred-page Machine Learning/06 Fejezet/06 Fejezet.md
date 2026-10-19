# 6. Neurális Hálózatok és Mély Tanulás (Neural Networks and Deep Learning)

## 6.1 Neurális Hálózatok Alapjai

Ha érted a lineáris regressziót, a logisztikus regressziót és a gradiens ereszkedést, a neurális hálózatok megértése gyerekjáték lesz.

A neurális hálózat matematikai szempontból pusztán egy hatalmas, egymásba ágyazott függvény (nested function):

```math
y = f_{NN}(\mathbf{x})
```

Ha egy 3 rétegű (3-layer) hálózatunk van, ami egyetlen számot ad vissza kimenetként, akkor a függvény így néz ki:

```math
y = f_{NN}(\mathbf{x}) = f_3(f_2(f_1(\mathbf{x})))
```

### Mit csinál egyetlen réteg (layer)?

Minden $l$-edik réteg egy $f_l(\mathbf{z})$ vektorfüggvényt hajt végre:

```math
f_l(\mathbf{z}) \overset{\text{def}}{=} g_l(\mathbf{W}_l \mathbf{z} + \mathbf{b}_l)
```

**A képlet elemei (Ezek már ismerősek!):**

- **$\mathbf{z}$:** A réteg bemenete (ami az előző réteg kimenete).
- **$\mathbf{W}_l$ (Súlymátrix):** Ez felel meg a régi $\mathbf{w}$ paraméterünknek. Azért mátrix, és nem egyetlen vektor, mert egy rétegben sok neuron (unit) van, és mindegyiknek megvan a maga saját kis $\mathbf{w}$ vektora. Ezek a vektorok alkotják a mátrix sorait.
- **$\mathbf{b}_l$ (Eltolás/Bias vektor):** Minden neuronnak van egy eltolása.
- **$g_l$ (Aktivációs függvény):** Ez egy rögzített, nem-lineáris matematikai függvény (mint a Szigmoid), amit a fejlesztő választ ki.

### 🔢 1. Számpélda: Miért Mátrix a Súly ($\mathbf{W}_l$)?

Képzeljük el az 1. réteget. Bejön egy $\mathbf{x}$ adat, aminek 2 jellemzője van (pl. életkor, súly). Tehát a bemenetünk egy 2D vektor: $\mathbf{x} = [25, 80]$.

Tegyük fel, hogy ebben a rétegben 3 darab neuron dolgozik. Minden neuronnak szüksége van saját súlyokra mindkét bemenethez.

- 1. neuron súlyai: $\mathbf{w}_{1,1} = [0.1, 0.5]$
- 2. neuron súlyai: $\mathbf{w}_{1,2} = [-0.2, 0.3]$
- 3. neuron súlyai: $\mathbf{w}_{1,3} = [0.4, -0.1]$

Ha ezeket egymás alá tesszük, egy $3 \times 2$-es Mátrixot ($\mathbf{W}_1$) kapunk!

Amikor a hálózat számol, a Mátrixot megszorozza a bemenettel ($\mathbf{W}_1 \mathbf{x}$), hozzáadja a $3$ bias-t, és megkap egy 3 elemű vektort. Ezt a 3 elemű vektort küldi tovább a 2. rétegnek. Így épülnek egymásra a rétegek.

### 6.1.1 Többrétegű Perceptron (Multilayer Perceptron - MLP) példa

A legklasszikusabb, "vanília" neurális hálózat az Előrecsatolt (Feed-Forward) Neurális Hálózat (FFNN), aminek a leghíresebb architektúrája a Többrétegű Perceptron (MLP).

- **A "Neuron" (Unit):** Egy doboz, ami megkapja a bemeneteket, csinál egy lineáris regressziót ($\mathbf{w} \cdot \mathbf{x} + b$), majd az eredményt áttolja az aktivációs függvényen ($g$), ami kiköp egy számot. Ezt a számot küldi tovább a következő rétegbe.
- **Teljesen összekötött (Fully-connected / Dense) rétegek:** Ez azt jelenti, hogy az 1. réteg összes neuronjának kimenete bemegy a 2. réteg összes neuronjának bemenetébe. Mindenki beszél mindenkivel.

> 📍 **[KÉP HELYE: IDE JÖN AZ 1. ÁBRA (6. FEJEZET)]**
>
> Itt kell szerepelnie a könyv *Figure 1* ábrájának, ami egy 3 rétegű MLP-t ábrázol. Bal oldalon a 2 db kör (bemenet), középen két rejtett réteg (egyenként 4 téglalappal/neuronnal), a jobb szélén pedig az 1 db kimeneti neuron.

### 6.1.2 Előrecsatolt (Feed-Forward) Architektúra felépítése

Egy neurális háló végén lévő utolsó réteg (output layer) határozza meg, hogy a hálózat mit csinál:

- Ha az utolsó neuron **Lineáris aktivációs függvényt** használ (tehát nem csinál semmit a számmal, csak kiadja), akkor a modell **Regressziót** végez (pl. lakásárat becsül).
- Ha az utolsó neuron **Szigmoid (Logisztikus) függvényt** használ, a kimenet $0$ és $1$ közötti lesz, azaz a modell **Bináris Osztályozást** végez.

### A Nem-linearitás fontossága (Miért kellenek az Aktivációs függvények?)

Felmerül a kérdés: miért nyomjuk át a számokat minden rétegben ezeken a furcsa $g$ függvényeken? Miért nem hagyjuk csak simán $\mathbf{W}\mathbf{x} + \mathbf{b}$ formában?

A válasz a matematika kegyetlen szabályaiban rejlik: **Egy lineáris függvény lineáris függvénye továbbra is csak egy sima lineáris függvény.**

Ha nem lennének nem-lineáris aktivációs függvények ($g$), akkor egy 1000 rétegű, milliárd paraméteres, mély neurális hálózat pontosan ugyanazt a buta, merev egyenest tudná csak felrajzolni, mint a legelső fejezetben megismert mezei lineáris regresszió. A nem-linearitás ($g$) engedi meg a hálózatnak, hogy meghajlítsa a teret, és komplex, agytekervény-szerű döntési határokat rajzoljon.

### A legnépszerűbb Aktivációs Függvények:

**1. Szigmoid (Logistic):**
(Ezt már ismerjük, $0$ és $1$ közé szorít).

**2. TanH (Hiperbolikus Tangens):**

```math
\tanh(z) = \frac{e^z - e^{-z}}{e^z + e^{-z}}
```

Nagyon hasonlít a Szigmoidra, de ez $-1$ és $+1$ közé szorítja a számokat. Sokkal jobban szeretik a belső (rejtett) rétegekben, mert a kimenete a $0$ körül van központosítva, ami segíti a gradiens ereszkedést.

**3. ReLU (Rectified Linear Unit):**

```math
\text{relu}(z) = \begin{cases} 0 & \text{ha } z < 0 \\ z & \text{egyébként} \end{cases}
```

A modern mélytanulás (Deep Learning) királya. Végtelenül egyszerű: ha a bemenet negatív, kinullázza. Ha pozitív, békén hagyja. Bár nagyon butának tűnik, pont ez a törtvonal (a nullánál) adja meg azt a nem-linearitást, ami a neurális hálózatok intelligenciájához kell, miközben nem terheli túl a processzort felesleges exponenciális ($e^x$) számításokkal.

### 🔢 2. Számpélda: Hogyan viselkedik a ReLU és a TanH?

Tegyük fel, egy neuron lineáris része ($\mathbf{w}\cdot\mathbf{x} + b$) kiszámolt egy értéket. Nézzük meg, mit adnak tovább a következő rétegnek az aktivációs függvények!

**Eset 1: Az eredmény $z = 2.0$**

- $\text{TanH}(2.0) \approx \mathbf{0.964}$ (Jól felvitte a $+1$ közelébe)
- $\text{ReLU}(2.0) = \mathbf{2.0}$ (Mivel pozitív, érintetlenül hagyja)

**Eset 2: Az eredmény $z = -3.0$**

- $\text{TanH}(-3.0) \approx \mathbf{-0.995}$ (Lenyomta a $-1$ közelébe)
- $\text{ReLU}(-3.0) = \mathbf{0.0}$ (Mivel negatív, a ReLU "kikapcsolja" ezt a neuront, nulla megy tovább)

## 6.2 Mély Tanulás (Deep Learning)

A Mély Tanulás (Deep Learning) hivatalosan azt jelenti, hogy kettőnél több rejtett réteget használunk a neurális hálózatban. Sokáig az volt a probléma, hogy minél több réteget tettek egy hálózatba, az annál butább lett, vagy egyszerűen képtelen volt tanulni. Ennek a hátterében egy nagyon komoly matematikai probléma állt.

### Az Eltűnő Gradiens Probléma (Vanishing Gradient)

Amikor a hálózat kiszámolja a hibát a legutolsó rétegnél, ezt a hibát "visszafelé" kell küldenie az első rétegekig, hogy minden neuron tudja, mennyit kell változtatnia a saját súlyain. Ezt az algoritmust hívják **Hiba-visszaterjesztésnek (Backpropagation)**, ami nem más, mint a 4. fejezetből megismert **Láncszabály (Chain Rule)** tömeges alkalmazása.

**Mi a baj a klasszikus aktivációs függvényekkel (pl. TanH vagy Szigmoid)?**

Ezeknek a függvényeknek a deriváltja (meredeksége) mindig egy $0$ és $1$ közötti nagyon kis szám (pl. $0.2$).

A Láncszabály szerint a visszaterjesztés során ezeket a kis számokat rétegről rétegre össze kell szorozni.

- Ha 1 rétegen mész vissza: $0.2$.
- Ha 2 rétegen mész vissza: $0.2 \cdot 0.2 = 0.04$.
- Ha 10 rétegen mész vissza: $0.2^{10} = \mathbf{0.0000001024}$.

Mire a hiba (a frissítési parancs) elérne a legelső (bemeneti) rétegekhez, az értéke szinte pontosan nullára csökken (eltűnik). Ennek eredményeként a hálózat első rétegei egyáltalán nem tanulnak semmit, a modell "lefagy".

**A Megoldás:**
A 2010-es évek deep learning forradalmát többek között az indította be, hogy rájöttek: ha a hagyományos függvények helyett **ReLU-t** használnak (aminek a deriváltja pozitív számoknál stabilan pontosan $1$), a gradiens sosem tűnik el, így hirtelen lehetővé vált akár 100 rétegű hálózatok betanítása is!

---

## 6.2.1 Konvolúciós Neurális Hálózatok (Convolutional Neural Network - CNN)

*(A könyv itt átlép a képfeldolgozás területére, amiben a CNN-ek a mai napig verhetetlenek).*

### Mi a probléma az eddigi (Teljesen Összekötött / MLP) hálózatokkal a képeknél?

A paraméterek száma felrobban! Ha egy aprócska, $100 \times 100$ pixeles képet (az $10\,000$ pixel) be akarsz küldeni egy $1000$ neuronos rétegbe, akkor az $10\,000 \times 1000 = \mathbf{10 \text{ millió}}$ paramétert ($\mathbf{w}$) jelent egyetlen rétegben! Ezt egyetlen gép sem tudja értelmes idő alatt betanítani.

Ráadásul a képeknél az információ lokális. Hogy megismerj egy "szemet", nem kell tudnod, mi van a kép másik sarkában. A szemet az egymás melletti pixelek (színek, élek, formák) alkotják.

### A Megoldás: A Konvolúció (Mozgó Ablak)

Ahelyett, hogy minden pixelt összekötnénk minden neuronnal, a CNN egy apró, például $3 \times 3$ pixeles "szűrőt" (Filter / Kernel) használ, és ezt az ablakot végigcsúsztatja (convolves) az egész képen, balról jobbra, fentről lefelé.

### 🔢 Számpélda: Hogyan számol a Konvolúciós Szűrő?

Tegyük fel, hogy a hálózat keres egy "Kereszt" alakzatot az adott képkockán.

A kép egy apró, $3 \times 3$-as darabja ($P$ - Patch) így néz ki (ahol az $1$ a fekete, a $0$ a fehér pixel):

```math
P = \begin{bmatrix} 0 & 1 & 0 \\ 1 & 1 & 1 \\ 0 & 1 & 0 \end{bmatrix}
```

A gépnek van egy betanult Szűrője ($F$ - Filter), aminek a súlyai a következőképpen néznek ki:

```math
F = \begin{bmatrix} 0 & 2 & 3 \\ 2 & 4 & 1 \\ 0 & 3 & 0 \end{bmatrix}
```

**A Konvolúciós Lépés (Dot-product):**
Egyszerűen rátesszük az $F$ szűrőt a $P$ képrészletre, a rajtuk lévő számokat páronként összeszorozzuk, majd az összes eredményt összeadjuk:

- 1. sor: $(0 \cdot 0) + (1 \cdot 2) + (0 \cdot 3) = 0 + 2 + 0 = 2$
- 2. sor: $(1 \cdot 2) + (1 \cdot 4) + (1 \cdot 1) = 2 + 4 + 1 = 7$
- 3. sor: $(0 \cdot 0) + (1 \cdot 3) + (0 \cdot 0) = 0 + 3 + 0 = 3$

**Összegzés:** $2 + 7 + 3 = \mathbf{12}$.

Ez a magas ($12$-es) szám azt jelzi a neurális hálónak, hogy *"Hé, ezen a képkockán valószínűleg találtam egy Kereszt alakzatot!"* Ezt az egyetlen számot menti el a hálózat, majd a szűrő csúszik egy pixellel arrébb, és kezdi elölről.

> 📍 **[KÉP HELYE: IDE JÖN A 2. ÁBRA (6. FEJEZET)]**
>
> Itt kell szerepelnie a könyv *Figure 2* ábrájának, amely lépésről lépésre mutatja be, ahogy egy $3 \times 3$-as kék szűrő végigcsúszik egy nagy zöld mátrixon, és minden lépésnél kiad egyetlen számot a piros kimeneti mátrixba.

### Miért zseniális a CNN? (Súlymegosztás)

Ugyanez a $3 \times 3$-as szűrő (összesen $9$ darab súly!) csúszik végig az egész képen. Tehát egy MLP $10$ millió paramétere helyett a CNN mindössze $9$ darab paraméterrel képes megkeresni egy mintázatot (pl. egy vízszintes élet) a kép bármelyik pontján!

Természetesen egy rétegben nem $1$, hanem mondjuk $64$ darab ilyen apró szűrő van (az egyik szemet keres, a másik orrot, a harmadik éleket), de ez paraméterszámban még mindig nevetségesen kicsi a klasszikus hálózatokhoz képest.

### Mi az a Térfogat (Volume)?

A valóságban egy kép nem fekete-fehér (1 rétegű), hanem színes (RGB), azaz 3 rétegű (Piros, Zöld, Kék csatornák). Így a bemenet nem egy sík, hanem egy térfogat ($x$ pixel, $y$ pixel, $3$ csatorna mély).

Emiatt a szűrő is térfogati (pl. $3 \times 3 \times 3$-as), és egyszerre számol mind a három színcsatornán.

> 📍 **[KÉP HELYE: IDE JÖN A 3. ÁBRA (6. FEJEZET)]**
>
> Itt kell szerepelnie a könyv *Figure 3* ábrájának. Ez egy olyan térbeli mátrixszorzást mutat, ahol a szűrő 3 réteg (színcsatorna) mély, és az ezekből származó szorzatokat egyetlen végső számmá (itt $-3$) vonja össze.
>
> ## 6.2.2 Rekurrens Neurális Hálózatok (Recurrent Neural Network - RNN)

Eddig olyan hálózatokat (FFNN, CNN) néztünk, amelyek megkapnak egy bemenetet (pl. egy lakás adatait vagy egy képet), áttolják az adatot balról jobbra a rétegeken, és a végén kiköpnek egy eredményt. Nincs memóriájuk. Ha beküldesz egy képet, majd egy másikat, a hálózat nem emlékszik az elsőre.

**De mi van akkor, ha az adatunk egy szekvencia (sorozat)?**
Például egy mondat: *"A kutya megette a ..."* Ha a gépnek meg kell jósolnia a következő szót, emlékeznie kell arra, hogy a mondat elején egy "kutya" volt, nem pedig egy "ember". Az ilyen szekvenciális adatokra (szöveg, beszéd, idősoros tőzsdeadatok) találták ki a Rekurrens Neurális Hálózatokat (RNN).

Az RNN legfőbb jellemzője, hogy **hurok (loop)** van benne. Nemcsak az előző réteg kimenetét kapja meg bemenetként, hanem saját magának az előző időpillanatban (timestep) kiszámolt állapotát is! Ezt a belső állapotot ($\mathbf{h}$) hívjuk a hálózat memóriájának.

### Hogyan működik lépésről lépésre? (Az "Időben Kiterjesztett" hálózat)

Képzeljük el, hogy a bemenetünk egy 3 szavas mondat: $X = [\mathbf{x}_1, \mathbf{x}_2, \mathbf{x}_3]$, ahol a $t$-edik időpillanatban az $\mathbf{x}_t$ az adott szó jellemzővektora.

Az RNN a mondatot szavanként, időlépésenként ($t$) olvassa be.

Egy adott $l$-edik réteg $u$-adik neuronjában a belső állapot/memória ($\mathbf{h}_t$) frissítése így néz ki:

```math
\mathbf{h}_t^{(l,u)} = g_1 \left( \mathbf{w} \cdot \mathbf{x}_t + \mathbf{u} \cdot \mathbf{h}_{t-1}^{(l,u)} + b \right)
```

Nézzük meg ezt a képletet! Két bemenetet adunk össze (lineáris kombináció):

1. **A jelenlegi adat:** $\mathbf{w} \cdot \mathbf{x}_t$ (Mennyire fontos a most olvasott szó?)
2. **A múltbeli memória:** $\mathbf{u} \cdot \mathbf{h}_{t-1}^{(l,u)}$ (Mennyire fontos az, amire az előző szóból emlékszünk?)

Ezt az összeget átküldjük egy $g_1$ aktivációs függvényen (általában TanH), és bumm, megkaptuk a frissített memóriánkat a $t$ időpillanatban.

Amikor a hálózat a legutolsó szót is beolvasta, a legfelső réteg egy kimeneti függvényen (pl. **Softmax**) keresztül kiköpi a végső tippet (pl. a következő szót, vagy azt, hogy a mondat pozitív/negatív hangvételű-e).

> *(A Softmax a Szigmoid többdimenziós nagytesója: olyan százalékokat ad vissza minden lehetséges osztályra, amiknek az összege pontosan $100\%$ / $1.0$ lesz).*

> 📍 **[KÉP HELYE: IDE JÖN A 4. ÁBRA (6. FEJEZET)]**
>
> Itt kell szerepelnie a könyv *Figure 4: The first two layers of an RNN* ábrájának. Ez az ábra azt mutatja be, ahogy a hálózatot "kigöngyöljük" (unfold) az időben. Látszik rajta az $x_t$ bemenet, és ahogy a rejtett állapot ($h_{t-1}$) oldalról érkezik be a dobozba a jelenlegi számításhoz ($h_t$).

### Az RNN Két Végzetes Hibája

Bármilyen zseniális az elmélet, a sima RNN-ek a gyakorlatban hamar elvéreznek:

1. **Eltűnő Gradiens az időben (Vanishing Gradient through time):** Amikor a hibát visszaterjesztjük (Backpropagation Through Time), nem csak a rétegeken, hanem a múltbeli időlépéseken is vissza kell mennünk. Egy 50 szavas mondatnál ez olyan, mintha egy 50 rétegű hálózaton kéne visszaküldeni a jelet a TanH kis deriváltjain keresztül. A gradiens eltűnik, a gép nem tudja megtanulni az elején lévő szavakat.
2. **Rövidtávú Memória (Long-term dependencies):** Mivel minden új szó felülírja a belső $\mathbf{h}$ állapotot, az első szó emléke ("A nagybátyám, aki Párizsban él...") a 20. szóra teljesen elhalványul, és a gép nem fogja tudni, hogy a végén azt kell mondania: "...franciául beszél".

### A megoldás? Adjunk a hálózatnak kapukat (Gates)!

### A Szuper-RNN: LSTM és GRU (A Kapuzott Memóriacellák)

A mérnökök kitalálták, hogy ne csak vakon összemixeljük az új szót a régiekkel, hanem adjunk a neuronnak egy kis "agyat", ami eldönti, mit tartson meg és mit felejtsen el! Ezek a **Gated RNN**-ek, amiknek a két leghíresebb változata az **LSTM** (Long Short-Term Memory) és a **GRU** (Gated Recurrent Unit).

A kapuk (Gates) olyan Szigmoid ($g_2$) függvények, amik $0$ és $1$ közötti értéket adnak vissza.

- Ha a kapu $0$-t ad, a gép törli/elfelejti az információt.
- Ha a kapu $1$-et ad, a gép továbbengedi/megjegyzi az információt.

**Példa a való életből:**
A gép olvassa: *"A király besétált a terembe. A királyné utánament. A férfi leült, míg a ___ állva maradt."*
Amikor a gép elolvassa a "királyné" szót, az Írás Kapu elmenti a női nemet a memóriába, a Felejtés Kapu pedig törli a király (férfi) memóriáját. Így a mondat végén a gép emlékezni fog rá, hogy a hiányzó szó a "nő".

### A GRU Matematikája (Ahogy a könyv bemutatja)

A GRU cella zsenialitása, hogy a memóriát egy **Felejtés Kapu** (Forget Gate / Update Gate), jelölése $\Gamma$ (Gamma), szabályozza.

A frissítés matematikája egy gyönyörű lineáris egyenlet (ahol $\tilde{\mathbf{h}}$ a jelölt, új információ, $\mathbf{h}_{t-1}$ a régi memória):

```math
\mathbf{h}_t = \Gamma \cdot \tilde{\mathbf{h}} + (1 - \Gamma) \cdot \mathbf{h}_{t-1}
```

**Elemezzük ki!**

Tegyük fel, az agy (a kapu) úgy dönt, a régi infó már nem kell, $\Gamma = 1$.
A képlet: $1 \cdot \text{Új} + 0 \cdot \text{Régi} \rightarrow$ A memória teljesen felülíródott az Újjal!

Tegyük fel, az agy úgy dönt, az új infó hülyeség (pl. egy "és" kötőszó), a régit meg kell tartani: $\Gamma = 0$.
A képlet: $0 \cdot \text{Új} + 1 \cdot \text{Régi} \rightarrow$ A memória érintetlen maradt a Múltból!

### Miért oldja meg ez az Eltűnő Gradienst?

Mivel a régi memória ($\mathbf{h}_{t-1}$) egy egyszerű összeadással ($+$) van bekötve, és meg is tartható érintetlenül ($1 \cdot$ Régi), a funkció egy szimpla identitás-függvény ($f(x) = x$). Az identitás függvény deriváltja fixen $1$. Így a gradiens (a hibaüzenet) veszteség nélkül, akadálytalanul tud visszaszaladni az időben, egészen a mondat legelső szaváig!
