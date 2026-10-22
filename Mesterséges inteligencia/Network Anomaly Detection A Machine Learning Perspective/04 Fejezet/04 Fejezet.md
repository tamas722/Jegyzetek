# 4. Fejezet: Hálózati adatok anomáliáinak detektálása (Detecting Anomalies in Network Data)

Ez a fejezet a hálózati anomáliadetektálás (Intrusion Detection) különböző megközelítéseit, architektúráit és komponenseit tárgyalja. Emellett bemutatja az IDS-ek (Behatolásjelző Rendszerek) betanításához és teszteléséhez szükséges referenciakészleteket (benchmark datasets).

## 4.1 Hálózati anomáliák detektálása

Az intrusion detection (behatolásjelzés) mögötti alapelv az a feltételezés, hogy a támadó viselkedése észrevehetően eltér a legitim felhasználóétól, így a jogosulatlan tevékenységek kiszűrhetők. Az IDS-ek a megelőző mechanizmusok (pl. tűzfalak, jelszavas autentikáció) mellett a második védelmi vonalat (second line of defense) alkotják.

Telepítési (deployment) helyük alapján két fő típusuk van: HIDS és NIDS.

### 4.1.1 Hoszt-alapú IDS (Host-Based IDS - HIDS)

A HIDS nem a hálózati forgalmat, hanem egy adott számítógépes rendszer belső folyamatait és állapotát figyeli.

- **Mit figyel?** Rendszerhívásokat (system calls), memóriát (RAM), fájlrendszert, futó folyamatokat (processes), és a naplófájlokat (logokat).
- **Példa egy anomáliára:** Egy szövegszerkesztő program (pl. MS Word) hirtelen elkezdi módosítani a rendszer jelszó-adatbázisát (pl. a `/etc/shadow` fájlt Linuxon). Ez azonnal behatolásra utal (User-to-Root támadás / Rootkit).
- **Szerepe:** Ellenőrzi, hogy a belső vagy külső entitások nem kerülték-e meg az operációs rendszer által érvényesített biztonsági házirendet (security policy).

### 4.1.2 Hálózat-alapú IDS (Network-Based IDS - NIDS)

A NIDS a hálózaton utazó adatokat (csomagokat vagy folyamatokat) elemzi, leggyakrabban szekvenciális módon, gyanús (anomális) mintázatokat keresve.

- **Mit figyel?** Az összes bejövő és kimenő forgalmat az adott hálózati szegmensben (pl. egy router mögött).
- **Példa egy anomáliára:** Ha rövid időn belül rengeteg TCP kapcsolatfelépítési kérés (SYN) érkezik egy géptől rengeteg különböző portra, a NIDS ebből arra következtet, hogy valaki épp Port Scant (nmap) hajt végre a hálózaton.
- **Belső támadások:** A NIDS nemcsak a kintről jövő forgalmat, hanem a helyi hálózaton (LAN) belüli forgalmat is látja, így a belső támadókat (Insider threat) is képes elkapni.

> 💡 **ML Szempont - Granularitás:** A NIDS kétféle mélységben dolgozhat. Vagy a nyers csomagszinten (packet level / PCAP), ami nagyon pontos, de óriási számítási igényű; vagy összesített folyamatszinten (flow level / NetFlow), ami gyorsabb, de kevesebb adatot (csak metaadatokat: forrás, cél, protokoll, bájtok száma) tartalmaz. A bemeneti adat általában magas dimenziószámú, folytonos és kategorikus attribútumok keveréke.

A detektálás módja szerint a NIDS lehet **Misuse-based** (ismert szignatúrákat keres) vagy **Anomaly-based** (szokatlan viselkedést keres). A könyv a továbbiakban az Anomaly-based NIDS-re (ANIDS) fókuszál.

### 4.1.3 Anomália-alapú Hálózati Behatolásjelzés (Anomaly-Based Network Intrusion Detection)

Az anomália detektálása gépi tanulási szempontból egy osztályozási probléma (classification problem): a hálózati forgalmat "Normál" vagy "Támadás/Abnormális" kategóriákba kell sorolni. A szakirodalom alapján négy fő tanulási mód (mode) létezik:

1. **Felügyelt (Supervised) anomália detekció:**
   - Címkézett tanítóadatokat (labeled training data) igényel mind a normál, mind a támadó osztályról (pl. KDD Cup 99 dataset). Prediktív modellt épít.
   - **Kihívások:** (1) A klasszikus **imbalanced data** probléma: a támadások sokkal ritkábbak, mint a normál forgalom. (2) Nagyon nehéz a támadási adatok pontos, ember általi címkézése. Ennek orvoslására gyakran mesterséges anomáliákat injektálnak az adatbázisba.
2. **Félig felügyelt (Semi-supervised) anomália detekció:**
   - Csak a Normál osztályról vannak címkézett adatai. A modell megtanulja, mi a normális, és élesben minden, ami ettől eltér, azt anomáliának jelöli.
   - Ez sokkal elterjedtebb, mert könnyebb begyűjteni (egy garantáltan tiszta hálózaton).
3. **Felügyelet nélküli (Unsupervised) anomália detekció:**
   - Nincsenek címkézett adatok.
   - **Kritikus feltevés:** A tesztadatokban a normál forgalom sokkal gyakoribb, mint az anomália. Ha a támadás alatt álló hálózatból veszünk mintát (ahol a támadás a forgalom $90\%$-a), a modell "normálisnak" fogja tanulni a támadást, ami borzasztóan magas False Alarm rátát (téves riasztást) okoz.
4. **Hibrid (Hybrid) anomália detekció:**
   - A felügyelt (magas pontosság, ismert támadások felismerése) és a felügyelet nélküli (új/zero-day támadások felismerése) módszerek kombinációja a lehető legjobb találati arány elérése érdekében.

### 4.1.4 A Felügyelt Anomália Detekciós Megközelítés (Supervised Anomaly Detection)

A felügyelt rendszer betanulja a normalitás (normality) fogalmát egy formális modell segítségével, ami leírja a hálózat alapvető dinamikáit.

Ha a rendszer $S = (M, D)$, ahol $M$ a normalitási modell (a betanult profil), a $D$ pedig a távolság/hasonlóság mérőszám (proximity measure). Ha egy új esemény $D$ devianciája (eltérése) az $M$ modelltől egy adott küszöb felett van, akkor a rendszer anomáliaként osztályozza.

#### Egy általános ANIDS (Anomaly-Based NIDS) Architektúrája

Egy tipikus ANIDS 7 fő komponensből áll:

> 📍 **[IDE JÖN A KÉP: Figure 4.1: A generic architecture of supervised ANIDS]**
>
> Az ábra egy ANIDS blokkdiagramját mutatja a forgalombefogástól a riasztásig.

1. **Forgalom befogása (Traffic Capture):**
   - A nyers forgalmat gyűjti össze (packet vagy flow szinten). Ezt a részt előfeldolgozás (preprocessing) követi.
   - **Csomagszintű (Packet) eszközök:** Gulp, Wireshark, tcpdump (libpcap könyvtárat használva).
   - **Folyamatszintű (Flow) eszközök nagy sebességű hálózatokhoz:** Nfdump, NfSen, Cisco Netflow V.9.

> 📍 **[IDE JÖN A KÉP: Figure 4.2: Network traffic capturing components]**

2. **Anomália Detekciós Motor (Anomaly Detection Engine):**
   - A rendszer "szíve". Összeveti az előfeldolgozott bejövő forgalmat a betanult profilokkal.
   - **Követelmények:** A párosításnak (matching) rettentő gyorsnak kell lennie (valós idejű védelem); a párosítás lehet "pontatlan" (inexact) is (tehát nem egzakt szignatúrát keres, hanem távolságot mér egy profilhoz képest a megfelelő küszöbérték - proximity threshold - figyelembevételével).
3. **Referencia adatok (Reference Data):**
   - A normál viselkedés, a hosztok, vagy az ismert támadások eltárolt profiljai (profiles) vagy szignatúrái. Egy felügyelt ANIDS esetén ezek leginkább a betanult profilok. Ezeket az új ismeretek fényében batch (kötegelt) üzemmódban frissítik.

> 📍 **[IDE JÖN A KÉP: Figure 4.3: Types of references used in supervised NIDS]**

4. **Konfigurációs adatok (Configuration Data):**
   - A rendszer átmeneti vagy belső beállításait (pl. részlegesen létrehozott behatolási szignatúrákat) tárolja, amelyek a működést paraméterezik.

> 📍 **[IDE JÖN A KÉP: Figure 4.4: Steps for updating configuration data in ANIDS]**

5. **Riasztás (Alarm):**
   - Ha a Detekciós Motor anomáliát talál, vizuális vagy log formátumú riasztást generál.
   - **Tartalmaznia kell:** A riasztás okát, a forrás és cél IP-t/portot, valamint háttér-információt (bizonyítékot), hogy miért gyanús a forgalom.
6. **Utófeldolgozás (Post-Processing):**
   - Feldolgozza a riasztásokat, hogy kiszűrje a statisztikai hibákat. A jó utófeldolgozás drasztikusan képes csökkenteni a False Positive (téves riasztási) arányt. (Pl. Ha a modell másodpercenként $1$ port scan gyanút dob, az utófeldolgozó megvárja, amíg összegyűlik $50$, és csak akkor szól a biztonsági őrnek, megelőzve az "alert fatigue"-et, a riasztási fásultságot).
7. **Biztonsági Menedzser (Security Manager - SM):**
   - Emberi vagy magasan automatizált komponens. Szerepe a riasztások végleges elemzése, az új/ismeretlen támadások (novel intrusions) nyugtázása, és a referencia adatbázis / profilok (Signature base) frissítése a jövőbeli gyorsabb detektálás érdekében.
  
   - ### 4.1.5 A felügyelt módszer problémái (Issues in Supervised Detection)

Bár a felügyelt ANIDS (Anomaly-Based NIDS) nagyon pontos az ismert anomáliák felismerésében, a gyakorlatban komoly akadályokba ütközik:

1. **A "tiszta" adat hiánya:** A modell betanításához tökéletesen címkézett (labeled) vagy garantáltan "tiszta", támadásmentes normál adathalmazra van szükség. Ilyet a valós hálózatokon szinte lehetetlen szerezni.
2. **Emberi erőforrás:** A hálózati forgalom manuális címkézése rendkívül időigényes, drága és magasan képzett biztonsági szakembereket igényel.
3. **Emberi hiba:** A manuális címkézés hajlamos a hibákra (error-prone), ami "megmérgezi" a betanított ML modellt.

### 4.1.6 A Felügyelet Nélküli Anomália Detekciós Megközelítés (Unsupervised Anomaly Detection)

Mivel nap mint nap jelennek meg új fenyegetések (zero-day attacks), és a címkézett adat drága, a felügyelet nélküli módszerek (leginkább a Klaszterezés / Clustering) jelentik a legkézenfekvőbb megoldást. Ez a módszer semmilyen előzetes tudást nem igényel: egyszerűen a hasonlóságok alapján csoportosítja (klaszterezi) az adatokat.

#### Az Unsupervised ANIDS Architektúrája

Nagyon hasonlít a felügyelt architektúrához, de a Detekciós Motor (Detection Engine) két teljesen más belső modulra van bontva:

> 📍 **[IDE JÖN A KÉP: Figure 4.5: A generic architecture of unsupervised ANIDS]**

- **Detekciós modul (Detection module):** A nyers, címkézetlen adatokat klaszterekbe (csoportokba) rendezi a hasonlóságuk alapján, vagy azonosítja a teljesen elszigetelt, kivételes eseteket (outliers).
- **Címkéző modul (Label module) és Stratégia:** Maga a klaszterező algoritmus nem tudja, hogy egy adott csoport "jó" vagy "rossz", csak azt, hogy a benne lévő csomagok hasonlítanak. A címkéző modul feladata, hogy eldöntse: a kapott klaszter Normál vagy Anomália? Ezt három alapfeltevés (assumption) alapján teszi:

1. **Feltevés:** A normál esetek száma (a klaszter mérete) hatalmas a támadásokhoz képest.
2. **Feltevés:** Az anomáliák minőségileg teljesen eltérnek a normál esetektől.
3. **Feltevés:** Egy anomáliás klaszteren belül a hasonlóság (kompaktság) sokkal magasabb, mint egy normál forgalmi klaszteren belül (a támadó kódja/szkriptje ismétlődő, gépi mintát követ).

> 💡 **Biztonsági / ML buktató (A 4.1.7-es rész alapján):** Az 1. feltevés (méret alapján történő címkézés) kudarcot vall egy DDoS támadásnál! Egy DDoS során a támadó forgalom sokkal nagyobb lesz (óriási klasztert alkot), mint a normál forgalom. Ha a modell csak a méretre hagyatkozik, a normál forgalmat fogja anomáliának, a DDoS-t pedig normálisnak hinni. Ezért a modern címkéző modulok a klaszterek kompaktságát (sűrűségét) is mérik (pl. Dunn-index, C-index, Davies-Bouldin index, Silhouette index segítségével). A DDoS klaszter hiába óriási, rendkívül homogén (tömör) lesz, amiből a rendszer rájöhet, hogy az egy támadás.

### 4.1.8 Hibrid Detekciós Megközelítés (Hybrid Detection Approach)

A Hibrid ANIDS célja, hogy a két világ (Supervised és Unsupervised) legjobb tulajdonságait ötvözze.

- **Működési elv:** A beérkező forgalmat először a Felügyelt modell (Supervised) vizsgálja meg. Ha a forgalom egyezik egy már ismert támadással vagy a megszokott normál profillal, azonnal döntést hoz (ez nagyon gyors és pontos).
- **A "maradék" kezelése:** Azokat a hálózati csomagokat, amelyeket az első modell nem tudott besorolni se a normál, se az ismert támadási osztályba, továbbítja a Felügyelet nélküli (Unsupervised) modellnek.
- **Visszacsatolás (Feedback loop):** Ha az unsupervised modul azonosít egy újfajta betörést (novel intrusion), arról egy szabályt vagy szignatúrát generál, amit visszatáplál a Felügyelt modell szabálybázisába (Rulebase). Így a rendszer folyamatosan tanul, és a következő alkalommal ezt a támadást már a gyorsabb, felügyelt modul fogja megfogni.

> 📍 **[IDE JÖN A KÉP: Figure 4.6: A generic architecture of a hybrid ANIDS]**

### 4.1.9 A Hibrid módszer problémái (Issues)

Egyetlen osztályozó (classifier) sem képes minden támadástípus (DoS, U2R, R2L, Probe) esetén egyformán jól teljesíteni. A Hibrid architektúrák legnagyobb kihívása a megfelelő algoritmusok kiválasztása és kombinálása.

Ha rossz algoritmusokat (vagy rossz kombinációs függvényt) választanak, a két modul egymás ellen dolgozhat, vagy a feldolgozási idő (computational cost) annyira megnőhet, hogy a NIDS képtelen lesz valós időben (real-time) elemezni a gigabites hálózati forgalmat.

## 4.2 A Hálózati Anomália Detekció Aspektusai (Aspects of Network Anomaly Detection)

Egy anomália-alapú NIDS (ANIDS) sikere három kritikus pilléren nyugszik. Ha ezeket megfelelően választjuk ki, a rendszer teljesítménye, sebessége és pontossága drasztikusan javul:

1. **Közelségi/Távolsági mértékek (Proximity measures)**
2. **Releváns jellemzők kiválasztása (Feature selection)**
3. **Anomália pontszámítás (Anomaly scores)**

### 4.2.1 Közelségi Mértékek és Adattípusok (Proximity Measure and Types of Data)

Bármilyen gépi tanulási feladatban (osztályozás, klaszterezés) tudnunk kell, mennyire "közel" van egymáshoz két objektum (pl. két hálózati folyam). Ezt a **Közelségi mérték (Proximity measure)** határozza meg, amely lehet:

- **Hasonlóság (Similarity):** Minél inkább egyforma a két objektum, annál nagyobb az érték.
- **Különbözőség / Távolság (Dissimilarity / Distance):** Minél távolabb vannak, annál nagyobb az érték. (Megjegyzés: Ha egy távolságmérték nem teljesíti a metrikus axiómákat – pl. a háromszög-egyenlőtlenséget –, akkor **divergenciának** hívjuk).

A mérték kiválasztása erősen függ a hálózati adatok típusától:

- **Numerikus adatok:** (pl. csomagméret, kapcsolat időtartama). Itt a klasszikus geometriai távolságok (pl. Euklideszi, Manhattan) jól működnek.
- **Kategorikus adatok:** (pl. Protokoll = TCP, Forrás IP = 192.168.1.10). Mivel itt nincsenek matematikai távolságok, vagy adat-vezérelt kategorikus mértékeket használunk, vagy bináris adattá kódoljuk őket (pl. One-Hot Encoding).
- **Vegyes (Mixed-type) adatok:** A hálózati forgalom szinte mindig vegyes (tartalmaz IP címeket és bájtszámokat is).
  - **Megoldások:** (a) A kategorikust numerikussá alakítjuk; (b) A numerikust kategorikussá (sávokra osztjuk); (c) Közvetlen vegyes mértékeket használunk, ahol a kategorikus egyezés = $0$ távolság, az eltérés = $1$ távolság.

### 4.2.2 Releváns Jellemzők Kiválasztása (Relevant Feature Identification)

A nyers hálózati forgalomban (pl. egy TCP kapcsolat során) akár $40-50$ különböző jellemzőt (feature-t) is kinyerhetünk. Ezeknek azonban csak egy kis része releváns egy adott támadás felismeréséhez.

**A Jellemzőkiválasztás (Feature Selection) céljai:**

- Számítási komplexitás (és ezáltal a detekciós idő) csökkentése.
- Információs redundancia megszüntetése (ha két feature ugyanazt mutatja, az egyik felesleges).
- Az algoritmus pontosságának és általánosító képességének (generalization) növelése.

**A Jellemzőkiválasztó módszerek 4 fő típusa:**

1. **Szűrő (Filter):** Az ML algoritmustól függetlenül, statisztikai jellemzők (pl. korreláció) alapján szűri az adatokat.
2. **Csomagoló (Wrapper):** Magát az ML algoritmust futtatja le a különböző feature-részhalmazokon, és az eredmény (pontosság) alapján választ. (Nagyon lassú, de pontos).
3. **Beágyazott (Embedded):** Az ML algoritmus (pl. Döntési Fa) tanulás közben maga választja ki a jó feature-öket.
4. **Hibrid:** A fentiek ötvözete.

> 💡 **Hálózati ML Példa:** Egy DoS támadás felismeréséhez a Forrás IP és a Másodpercenkénti csomagszám releváns feature. Ugyanakkor a csomag payload (adat) tartalma irreleváns és feleslegesen lassítaná az IDS-t. A Feature Selection ezt a payload mezőt automatikusan eldobja.

### 4.2.3 Anomália Pontszám (Anomaly Score)

A gépi tanulási modellek kimenete ritkán egy egyszerű "Igen/Nem". Legtöbbször egy **Anomália pontszámot** rendelnek minden egyes hálózati eseményhez. Ez a pontszám mutatja meg, mennyire tartja abnormálisnak a modell az adott csomagot.
A biztonsági elemzők ez alapján felállíthatnak egy rangsort (ranked list), vagy meghúzhatnak egy küszöbértéket (cut-off threshold), ami felett a rendszer riaszt.

A pontszámítási technikák három nagy kategóriába sorolhatók:

#### A. Távolság-alapú pontszámítás (Distance-based anomaly score)

Gyakori elemhalmaz-bányászati (frequent itemset) elvekre épít. Ritka kombinációkat keres.

- **A.1 LOADED (Link-Based Outlier and Anomaly Detection in Evolving Datasets):**
  - Vegyes (mixed) adatokra tervezték. A pontok közötti "kapcsolati erőt" a közös attribútum-érték párok száma adja. A pontszámképzés azon alapul, hogy minél több ritka (alacsony támogatottságú, $supp(d) \le s$) elemhalmazt tartalmaz egy adatpont, annál nagyobb az anomália pontszáma.
  - Kategorikus adatokra a képlet:

```math
Score_1(p_i) = \sum_{d \subseteq p_i} \left( \frac{1}{|d|} \Big| \sup(d) \le s \right)
```

  - Vegyes adatoknál ($Score_2$) a modell egy korrelációs mátrixot is megvizsgál ($C_2$ feltétel), hogy a folytonos változók (pl. bájtok száma) eltérnek-e az elvárt eloszlástól.
- **A.2 RELOADED (REduced memory LOADED):**
  - Memóriahatékonyabb verzió. Az anomáliát aszerint pontozza, hogy az attribútumok hány helytelen predikciót (eltérést) generáltak ($W_j$) az előző adatpontokhoz képest, egy halmozott sértési pontszámmal ($V_\tau$) kombinálva.

#### B. Sűrűség-alapú pontszámítás (Density-based anomaly score)

Azt vizsgálja, hogy az adatpont egy sűrű (normál) vagy egy ritka (anomália) régióba esik-e.

- **B.1 ODMAD (Outlier Detection for Mixed Attribute Datasets):**
  - A kategorikus értékek rendellenességét és a folytonos értékek közötti kapcsolatot vizsgálja. Egy esemény akkor kap magas pontszámot ($Score_1$), ha olyan kategorikus értékeket (vagy azok kombinációit) tartalmazza, amelyek a felhasználó által megadott $\delta$ küszöbnél kevesebbszer fordultak elő (infrequent).
  - Vegyes adatoknál ($Score_2$) a folytonos rész ($X_i^q$) és a vizsgált kategória átlaga ($\mu_a$) közötti koszinusz hasonlóságot (cosine similarity) használja. Ha ez a hasonlóság közelít a $0$-hoz, a pontszám megnő (outlier).

#### C. Lágy-számítás és ML-alapú pontszámítás (Soft computing and machine learning)

Neurális hálókat és statisztikai modelleket használ.

- **C.1 RNN (Replicator Neural Network):**
  - Egy olyan neurális hálózat, amelyet arra tanítanak, hogy a bemenetét ($x_{ij}$) lemásolja a kimenetére ($o_{ij}$).
  - Az Anomália pontszám a **Helyreállítási Hiba (Reconstruction Error)**:

```math
\delta_i = \frac{1}{n} \sum_{j=1}^{n} (x_{ij} - o_{ij})^2
```

> 💡 **Működési elv:** Ha a neurális hálót normál hálózati forgalmon tanítottuk be, tökéletesen le tudja másolni (rekonstruálni) a normál csomagokat (a hiba közel $0$). Ha beérkezik egy DDoS támadás csomagja, a hálózat nem ismeri ezt a mintát, nem tudja jól rekonstruálni, a hiba (és így az anomália pontszám) az egekbe szökik.

- **C.2 GMM (Gaussian Mixture Model CAD):**
  - A hálózati adatot környezeti/kontextuális ($U$) és viselkedési ($V$) részekre bontja. Kiszámolja, mi a valószínűsége annak, hogy a látott viselkedést az adott kontextus generálta.
- **C.3 Markov Lánc Modell (Markov Chain Model):**
  - Az időbeli (temporális) profilokat, szekvenciákat figyeli (pl. egy protokoll állapotmeneteit). Kiszámolja a megfigyelt szekvencia ($S$) valószínűségét ($P(S)$) a tanulóhalmaz átmeneti valószínűségei ($p_{S_{t-1}S_t}$) alapján.

```math
P(S) = q_{S_1} \prod_{t=2}^{|S|} p_{S_{t-1}S_t}
```

Az anomália pontszám a valószínűség inverze ($1 / P(S)$). Minél valószínűtlenebb a szekvencia (pl. egy TCP kapcsolat felépítése RST, majd azonnal SYN-ACK csomaggal), annál nagyobb lesz a kapott anomália pontszám.

#### Összefoglaló Táblázat

Az alábbi táblázat az Anomália Pontszámítási technikák főbb jellemzőit foglalja össze:

| Szerző & Év | Pontszámítási Megközelítés / Modell | Használt Adattípus |
| :--- | :--- | :--- |
| Ye, 2000 | Lágy-számítás alapú (HMM / Markov Lánc inverz valószínűség) | Vegyes (Mixed) |
| Hawkins, 2002 | Lágy-számítás alapú (RNN - Helyreállítási hiba / $\delta_i$) | Kategorikus (Categorical) |
| Ghoting, 2004 | Távolság-alapú (LOADED - Ritka elemhalmazok $Score_1$) | Kategorikus (Categorical) |
| Ghoting, 2004 | Távolság-alapú (LOADED - Vegyes, korrelációs szabályok $Score_2$) | Vegyes (Mixed) |
| Otey, 2005 | Távolság-alapú (RELOADED - Hibás predikciók $W_j$ és sértések) | Vegyes (Mixed) |
| Song, 2007 | Lágy-számítás alapú (GMMCAD - Gauss keverék modellek) | Vegyes (Mixed) |
| Koufakou, 2010 | Sűrűség-alapú (ODMAD $Score_1$ - Ritka kategorikus halmazok) | Kategorikus (Categorical) |
| Koufakou, 2010 | Sűrűség-alapú (ODMAD $Score_2$ - Koszinusz hasonlóság az átlaghoz) | Vegyes (Mixed) |


## 4.3 Adathalmazok (Datasets)

Egy anomália-alapú behatolásjelző rendszer (ANIDS) teljesítményének, pontosságának és hibarátájának értékeléséhez elengedhetetlenek a megfelelő adathalmazok. A kutatócsoportok által létrehozott adathalmazokat három fő kategóriába sorolhatjuk:

1. **Publikus adathalmazok (Public datasets):** Bárki számára ingyenesen elérhető (pl. egyetemek, kutatóintézetek által publikált) adatok.
2. **Privát adathalmazok (Private datasets):** Cégek, telekommunikációs szolgáltatók belső hálózatáról származó adatok (ezek általában titkosítottak, mert érzékeny felhasználói információkat tartalmaznak).
3. **Hálózati szimulációs adathalmazok (Network simulation datasets):** Ezek nem valódi betörésekből származnak, hanem egy elszigetelt, mesterséges teszt-hálózaton (testbed) szimulálják a normál forgalmat és a támadási szcenáriókat szoftverek segítségével. Valójában a publikus adathalmazok (mint a KDD) is így készültek.

> 💡 **Hogyan készülnek ezek az adathalmazok?** A generáláshoz használt eszközök négy fázist fednek le:
> 1. Az áldozatok azonosítása.
> 2. A forgalom rögzítése és előfeldolgozása (sniffing).
> 3. A különböző támadások elindítása (exploit keretrendszerekkel).
> 4. A forgalmi mintázatok monitorozása.

### 4.3.1 Publikus Adathalmazok (Public Datasets)

Bár a publikus adathalmazokat rengeteg kutatás használja, a könyv megjegyez egy kritikus problémát:
A támadók módszerei annyit fejlődtek, és ezekben az adathalmazokban annyira kiegyensúlyozatlan (imbalanced) a támadások és a normál forgalom eloszlása, hogy ma már **elavultnak és torzítottnak (old and biased)** számítanak.

Akkor miért tanuljuk őket? Mert felépítésük (milyen attribútumokat, metrikákat tartalmaznak) megértése elengedhetetlen ahhoz, hogy egy biztonsági mérnök képes legyen felépíteni a saját teszthálózatát, kinyerni a megfelelő hálózati feature-öket, és létrehozni egy modern, torzításmentes adathalmazt.

#### 4.3.1.1 KDD Cup 1999 Dataset (A klasszikus benchmark)

A KDD Cup '99 a történelem legismertebb és legtöbbet hivatkozott behatolásjelzési adathalmaza. A DARPA (az USA védelmi kutatási ügynöksége) megbízásából készítették egy katonai hálózat szimulációjával.

**Az adathalmaz jellemzői (Dataset Characteristics):**

- **Adatszerkezet:** A hálózati forgalmat hálózati kapcsolatok (connections) formájában tárolja (pl. egy teljes TCP munkamenet).
- **Attribútumok (Features):** Minden rekordot $41$ attribútum ír le. Ebből $38$ numerikus (folytonos vagy diszkrét, pl. küldött bájtok száma, hibás bejelentkezések száma) és $3$ kategorikus (pl. protokoll típusa: TCP/UDP/ICMP).
- **Címkék (Labels):** Minden rekord fel van címkézve "Normál" vagy egy specifikus támadás neveként.

**Támadási eloszlás:**
Két verziója terjedt el: a Corrected KDD és a 10-percent KDD (amely csak egy tizede az eredetinek, hogy beférjen az akkori gépek memóriájába).

**A probléma:** Ha megnézzük a számokat, a $10\%$-os verzióban van $391\,458$ db DoS támadás, de mindössze $52$ db U2R (User to Root) és $1126$ db R2L (Remote to Local) támadás. Ez drasztikus osztály-kiegyensúlyozatlanságot jelent, amivel a legtöbb ML algoritmus nem tud mit kezdeni (egyszerűen zajnak nézi az U2R támadásokat, ahogy azt az előző fejezetben láttuk).

**A KDD Cup Támadástípusainak Részletes Bontása:**

A támadásokat 4 nagy kategóriába (osztályba) sorolták:

**1. Szolgáltatásmegtagadás (DoS - Denial of Service)**
- **Célja:** Leállítani a rendszert erőforrás-kimerítéssel.
- **Smurf:** Hamisított (spoofed) feladó IP-vel küld ping kéréseket broadcast címekre, így a hálózat összes gépe a feladónak (az áldozatnak) küldi vissza a válaszokat, lefolytva azt.
- **Neptune:** (más néven SYN Flood) Hamis IP-címekről indít TCP kapcsolatfelépítéseket, amelyeket sosem fejez be. Az áldozat memóriája megtelik a félig nyitott kapcsolatokkal.
- **Back:** Az Apache webszervernek rengeteg perjelet (backslash) tartalmazó URL-t küld, amitől a szerver lelassul/összeomlik.
- **Teardrop:** Szándékosan hibásan darabolt (misfragmented) UDP csomagokat küld, amiknek az összeillesztésekor a célgép operációs rendszere lefagy.
- **Ping-of-Death:** Az engedélyezettnél ($65535$ bájt) nagyobbra összerakható ping csomagokat küld, ami puffer-túlcsordulást okoz az OS-ben.
- **Land:** A csomag feladójának és céljának is a célgép IP-címét és portját állítja be, amitől a gép végtelen hurokba (loopba) kerül.

**2. Távoliról Helyi (R2L - Remote to Local)**
- **Célja:** Egy távoli gépről próbál jogosulatlan helyi (felhasználói) hozzáférést szerezni.
- **FTP-write:** Kihasználja az FTP szerver rossz konfigurációját (pl. az anonymous felhasználó írhat a gyökérkönyvtárba).
- **Guess-password:** Brute-force vagy szótáralapú jelszótalálgatás (Telneten vagy vendégfiókon).
- **Imap:** Távoli puffer-túlcsordulás az IMAP porton, ami root shellt ad.
- **Multihop, Spy:** Többnapos, lopakodó támadások, több gépen keresztül ugrálva információszerzés céljából.
- **Phf:** Webes CGI szkriptek sebezhetőségének kihasználása (Remote Code Execution).
- **Warezclient / Warezmaster:** Szerzői jogvédett, illegális szoftverek (warez) feltöltése és letöltése kompromittált, rosszul beállított anonymous FTP szervereken keresztül.

**3. Felhasználóból Rendszergazda (U2R - User to Root)**
- **Célja:** A támadónak már van normál fiókja, root (superuser) jogot akar szerezni.
- **Bufferoverflow:** A kliens a vártnál több adatot küld, ami túlcsordul a veremben (stack), és lehetővé teszi a támadó kódjának lefutását.
- **Loadmodule, Perl:** Különféle szkript-környezetek és környezeti változók (pl. IFS) módosítása egy root shell (rendszergazdai parancssor) létrehozásához.
- **Rootkit:** Trójai programcsomag telepítése (gyakran napokon át), amely elrejti a behatoló jelenlétét a rendszerben, miközben folyamatos root hozzáférést biztosít.

**4. Feltérképezés (Probe)**
- **Célja:** Információgyűjtés a későbbi támadásokhoz.
- **Ipsweep:** Pingeléssel végignézi a hálózatot, hogy mely IP-címeken vannak élő gépek.
- **Portsweep:** Egy adott gépen végignézi az összes TCP/UDP portot, hogy kiderítse, milyen szolgáltatások (services) futnak rajta.
- **Nmap:** A világ legismertebb hálózat-feltérképező eszköze.
- **SATAN:** Egy régi szoftvereszköz, amely kifejezetten a hálózat ismert biztonsági sebezhetőségeit kutatta fel (Security Administrator Tool for Analyzing Networks).

#### 4.3.1.2 NSL-KDD Dataset (A javított verzió)

A KDD Cup 1999 adatbázissal az volt a legnagyobb probléma, hogy a rekordok hatalmas része (néhol $78\%$-a) teljesen megegyező, redundáns adat (duplikátum) volt.

> 💡 **ML Szempont - A torzítás oka:** Mivel egy gépi tanulási algoritmus statisztikai alapon működik, ha lát $1$ millió teljesen egyforma DoS kapcsolatot és $50$ darab U2R kapcsolatot, a modell túltanul (overfit) a gyakori rekordokra. Ennek eredményeképp azokat a ML modelleket, amik a KDD '99-en $99\%$-os pontosságot értek el, a való életben szinte használhatatlanok voltak.

A kutatók ezen probléma orvoslására hozták létre az NSL-KDD adathalmazt.

**Fő módosítás:** A redundáns adatok eltávolítása. A dataset csak a teljesen egyedi (duplicate-free) rekordokat tartalmazza. Emiatt a tanuló algoritmusokat már nem torzítja el a leggyakoribb támadások (főleg a DoS) puszta mennyisége.

**Felépítése:** Az adathalmazt két részre bontották:

- **KDDTrain+ (Betanító halmaz):** $125\,973$ egyedi rekord. Bár a DoS ($45\,927$) és a Normál ($67\,343$) még mindig domináns, az algoritmusok sokkal jobban képesek általánosítani.
- **KDDTest+ (Tesztelő halmaz):** $22\,544$ rekord. Ezen mérik le a modell végső teljesítményét. (Fontos, hogy olyan új támadástípusokat is tartalmaz, amik a Train halmazban egyáltalán nem voltak benne, így tesztelve az ML modell anomália detektáló, azaz általánosító képességét).

- ### 4.3.2 Privát Adathalmazok (Private Datasets)

A publikus adathalmazok (pl. KDD) elavulása és torzítása miatt a kutatók, egyetemek és vállalatok gyakran maguk generálnak adathalmazokat egy kontrollált környezetben. A könyv szerzői is ezt tették a Tezpur Egyetemen: így született meg a **TUIDS (Tezpur University Intrusion Dataset)**.

#### 4.3.2.1 TUIDS Behatolási Adathalmaz (TUIDS Intrusion Dataset)

A szerzők egy elszigetelt, laboratóriumi hálózatot (testbed) építettek fel. Különböző nyílt forráskódú hackereszközöket használtak a szerverek és kliensek támadására, miközben minden forgalmat rögzítettek.

A TUIDS három különböző adathalmaz-kategóriába van bontva az eltérő attribútumok alapján:

1. **Csomagszintű forgalmi adatok (Packet traffic feature dataset)**
2. **Folyamatszintű forgalmi adatok (Network flow traffic feature dataset)**
3. **Portscan (Portszkenneléses) adathalmaz**

> 📍 **[IDE JÖN A KÉP: Figure 4.7: Testbed for generation of the TUIDS intrusion dataset]**
>
> Az ábra a felépített hálózatot mutatja: L2/L3 switchek, routerek, szerverek és a forgalmat rögzítő "Mirror Port"-ra kötött gép.

**A Teszthálózat (Testbed) felépítése:**

- A hálózat 2 darab L2 switch-ből, 1 darab L3 switch-ből, routerekből, egy szerverből és 40 kliens gépből (node) áll.
- Az izoláció érdekében 6 VLAN-t (Virtuális LAN) hoztak létre.
- **Adatgyűjtés:** A forgalom rögzítése a szerver és az L3 switch közé bekötött **Tükrözött Porton (Mirror Port / SPAN port)** keresztül történik. A switch ide másol minden átmenő forgalmat, anélkül, hogy megzavarná a hálózat működését.
- **Támadási szcenáriók:** A támadásokat indították (a) ugyanazon a VLAN-on belülről, (b) egy másik VLAN-ból, és (c) az Internet felől egy külső routeren keresztül. Normál forgalom rögzítésekor a hálózatot teljesen leválasztották az internetről.

**A TUIDS Adathalmazban szereplő támadások:**

> *(Az eszközök főként a `targa2.c` támadócsomagot, az `Nmap`-et és C szkripteket használták).*

### Table 4.5: A TUIDS Adathalmazban szereplő támadások leírása

Ez a táblázat részletezi, hogy az egyes támadások technikailag hogyan működnek a hálózaton.

| Támadás neve | Leírás (Hogyan működik?) |
| :--- | :--- |
| **Bonk** | Manipulálja a "fragment offset" (töredék eltolás) mezőt a TCP/IP csomagokban. Amikor az áldozat gépe megpróbálja összerakni a csomagokat, azok túl nagyok lesznek, ami fagyást okoz. |
| **Jolt** | Nagyon nagy, töredezett ICMP (ping) csomagokat küld. Amikor az áldozat megpróbálja ezeket újraépíteni, inkompatibilitás lép fel. |
| **Land** | Egy hamisított (spoofed) TCP SYN csomagot küld, ahol a feladó és a cél IP-címe, valamint a forrás és cél portja is megegyezik (az áldozaté). A gép végtelen hurokban önmagának kezd válaszolgatni. |
| **Nestea** | IP töredékeket (fragments) küld egy Linux gépnek, és az ún. "off-by-one" (eggyel elcsúszott) IP fejléc hibát használja ki az újra-töredezési (refragmentation) kódban. |
| **Newtear** | A UDP fejléc információkat hamisítja meg: megváltoztatja a kitöltés (padding) hosszát, és a UDP fejléc hosszát a csomagméret duplájára állítja. |
| **Syndrop** | Túl kicsire állítja a csomagméretet, ami Kernel-problémát okoz, és a rendszer összeomlásához vezethet. |
| **Teardrop** | Egy IP töredék-átfedési (overlapping fragment) hibát használ ki, ami miatt a TCP/IP újra-összeszerelő (reassembly) kód képtelen kezelni az egymást átfedő töredékeket. |
| **Winnuke** | OOB (Out-of-Band) adatokat küld egy Windows gépnek a 139-es porton keresztül. A gép nem tudja lekezelni, ami furcsa rendszer-viselkedést (gyakran Kék Halált) okoz. |
| **1234** | Túlméretezett ping csomagot küld, amit a hálózati szoftver nem bír kezelni, így az áldozat gépe lelassul, majd lefagy. |
| **Saihyousen** | UDP csomagok folyamát (stream) küldi, amitől a tűzfalak összeomlanak. Erőforrásokat emészt fel, és ha a gép fagyása után még $10-30$ másodpercig tart, csúnya újraindulást (messy reboot) okoz. |
| **Oshare** | Újszerű csomagszerkezetet küld Windows 98 és NT gépeknek, ami a CPU terhelésének drasztikus növekedését vagy pillanatnyi fagyásokat okoz. |
| **Smurf** | Hatalmas mennyiségű ICMP echo request (ping) csomagot küld IP broadcast (szórási) címekre, úgy, hogy a feladó IP-címét az áldozatéra hamisítja. A teljes hálózat az áldozatnak fog válaszolni. |
| **Fraggle** | Ugyanaz, mint a Smurf, csak ICMP helyett UDP echo csomagokat használ. |
| **Syn Scan** | Az alapértelmezett portszkennelés. Ha a célport nyitva van, egy SYN csomagra SYN/ACK-kal válaszol. Ha zárva van, egy RST (Reset) csomaggal. Nincs teljes kapcsolatfelépítés. |
| **Xmass Tree Scan** | A TCP fejlécben beállítja a FIN, PSH és URG flag-eket. Ha a port zárva van, RST-t küld vissza. Ha nyitva, nem küld semmit. Ha ICMP "unreachable" jön, a port szűrve van (filtered). |
| **Window Scan** | A visszatérő RST csomagok TCP Ablakméret (Window) mezőjét vizsgálja. Bizonyos rendszereken a zárt portok ablakmérete nulla, a nyitottaké nem nulla, így pontosan felmérhető a rendszer állapota. |


### Table 4.6: A TUIDS Adathalmazban használt támadások és eszközök

Ez a táblázat megmutatja, hogy a fenti támadásokat milyen konkrét szoftveres/kód szintű eszközzel generálták a laborban.

| Támadás | Generáló Eszköz | Támadás | Generáló Eszköz |
| :--- | :--- | :--- | :--- |
| **bonk** | `targa2.c` | **1234** | `targa2.c` |
| **jolt** | `targa2.c` | **saihyousen** | `targa2.c` |
| **land** | `targa2.c` | **oshare** | `targa2.c` |
| **nestea** | `targa2.c` | **window** | `targa2.c` |
| **newtear** | `targa2.c` | **syn** | `Nmap` |
| **syndrop** | `targa2.c` | **xmas** | `Nmap` |
| **teardrop** | `targa2.c` | **fraggle** | `fraggle.c` |
| **winnuke** | `targa2.c` | **smurf** | `smurf4.c` |

**(ii) A Csomagszintű (Packet Traffic) Adathalmaz Jellemzői**

- **Adatgyűjtés eszközei:** `gulp` (szupergyors lemezre írás a hálózatról) és `Wireshark` (elemzés).
- **Feature extrakció:** `tcptrace` és Perl/C szkriptek segítségével a nyers pcap fájlokból 50 darab feature-t nyertek ki.

> 💡 **ML Szempont - Jellemzők (Features) kategorizálása:** Az IDS-ek betanításához a kinyert jellemzőket négy csoportba osztjuk (C = Folytonos/Numerikus, D = Diszkrét/Kategorikus):

### Table 4.7: Alapvető csomagjellemzők (Basic Packet Features) - 25 db

A nyers csomagfejlécekből kinyert alapinformációk. (A típusnál **C** = Folytonos / Numerikus, **D** = Diszkrét / Kategorikus.)

| Sorszám | Jellemző neve | Típus | Leírás |
| :---: | :--- | :---: | :--- |
| 1. | `Duration` | C | Az első képkocka (frame) érkezése óta eltelt idő |
| 2. | `Protocol` | D | 3-as réteg protokollja: IP, TCP, UDP |
| 3. | `Src IP` | C | Feladó IP címe |
| 4. | `Dst IP` | C | Cél IP címe |
| 5. | `Src port` | C | Feladó gép portja |
| 6. | `Dst port` | C | Cél gép portja |
| 7. | `Service` | D | Hálózati szolgáltatás a célgépen (pl. http, telnet) |
| 8. | `num-bytes-src-dst` | C | Adatbájtok száma a feladótól a cél felé |
| 9. | `num-bytes-dst-src` | C | Adatbájtok száma a céltól a feladó felé |
| 10. | `Fr-no.` | C | Képkocka (Frame) sorszáma |
| 11. | `Fr-length` | C | A képkocka hossza |
| 12. | `Cap-length` | C | Rögzített (captured) képkocka hossza |
| 13. | `Head-len` | C | A csomag fejlécének hossza |
| 14. | `Frag-offset` | D | A töredék eltolásának (Fragment offset) értéke |
| 15. | `TTL` | C | Time To Live (Élettartam) |
| 16. | `Seq-no.` | C | Szekvenciaszám |
| 17. | `CWR` | D | Congestion Window Record (Torlódási ablak) flag |
| 18. | `ECN` | D | Explicit Congestion Notification (Kifejezett torlódásjelzés) flag |
| 19. | `URG` | D | Urgent TCP flag |
| 20. | `ACK` | D | Acknowledgment flag |
| 21. | `PSH` | D | Push TCP flag |
| 22. | `RST` | D | Reset flag |
| 23. | `SYN` | D | Synchronize flag |
| 24. | `FIN` | D | Finish flag |
| 25. | `Land` | D | $1$, ha a kapcsolat a forrás és cél gép/port között azonos; amúgy $0$ |

### Table 4.8: Tartalom-alapú csomagjellemzők (Content-Based) - 5 db

Mélyebb beállítások és kapcsolati státusz.

| Sorszám | Jellemző neve | Típus | Leírás |
| :---: | :--- | :---: | :--- |
| 1. | `Mss-src-dst-requested` | C | A feladó által a cél felé kért maximális szegmensméret (MSS) |
| 2. | `Mss-dst-src-requested` | C | A cél által a feladó felé kért maximális szegmensméret |
| 3. | `Ttl-len-src-dst` | C | TTL (Time to live) hossza feladótól cél felé |
| 4. | `Ttl-len-dst-src` | C | TTL hossza céltól feladó felé |
| 5. | `Conn-status` | C | A kapcsolat státusza ($1$ - befejezett, $0$ - resetelve) |

---

### Table 4.9: Idő-alapú csomagjellemzők (Time-Based) - 10 db

Ezek a metrikák egy meghatározott $T$ másodperces időablakot vizsgálnak.

| Sorszám | Jellemző neve | Típus | Leírás |
| :---: | :--- | :---: | :--- |
| 1. | `count-fr-dst` | C | Ugyanattól a feladótól kapott képkockák száma az egyedi célgéphez az elmúlt $T$ mp-ben |
| 2. | `count-fr-src` | C | Ugyanarra a célra indított képkockák száma az egyedi feladótól az elmúlt $T$ mp-ben |
| 3. | `count-serv-src` | C | Képkockák száma a feladótól ugyanarra a célportra az elmúlt $T$ mp-ben |
| 4. | `count-serv-dst` | C | Képkockák száma a céltól ugyanarra a forrásportra az elmúlt $T$ mp-ben |
| 5. | `num-pushed-src-dst` | C | PUSH csomagok száma feladótól cél felé |
| 6. | `num-pushed-dst-src` | C | PUSH csomagok száma céltól feladó felé |
| 7. | `num-SYN-FIN-src-dst` | C | SYN/FIN csomagok száma feladótól cél felé |
| 8. | `num-SYN-FIN-dst-src` | C | SYN/FIN csomagok száma céltól feladó felé |
| 9. | `num-FIN-src-dst` | C | FIN csomagok száma feladótól cél felé |
| 10. | `num-FIN-dst-src` | C | FIN csomagok száma céltól feladó felé |

---

### Table 4.10: Kapcsolat-alapú csomagjellemzők (Connection-Based) - 10 db

Ezek a metrikák nem az időt, hanem az utolsó $N$ darab csomagot (eseményablakot) veszik alapul.

| Sorszám | Jellemző neve | Típus | Leírás |
| :---: | :--- | :---: | :--- |
| 1. | `count-dst-conn` | C | Képkockák száma az egyedi célgéphez az utolsó $N$ csomagból ugyanattól a feladótól |
| 2. | `count-src-conn` | C | Képkockák száma az egyedi feladótól az utolsó $N$ csomagból ugyanarra a célgépre |
| 3. | `count-serv-src-conn` | C | Képkockák száma a feladótól ugyanarra a célportra az utolsó $N$ csomagban |
| 4. | `count-serv-dst-conn` | C | Képkockák száma a céltól ugyanarra a forrásportra az utolsó $N$ csomagban |
| 5. | `num-packets-src-dst` | C | Csomagok száma feladótól cél felé |
| 6. | `num-packets-dst-src` | C | Csomagok száma céltól feladó felé |
| 7. | `num-acks-src-dst` | C | ACK (Nyugtázó) csomagok száma feladótól cél felé |
| 8. | `num-acks-dst-src` | C | ACK csomagok száma céltól feladó felé |
| 9. | `num-retransmit-srcdst` | C | Újraküldött (retransmitted) csomagok száma feladótól cél felé |
| 10. | `num-retransmit-dstsrc` | C | Újraküldött csomagok száma céltól feladó felé |

**(iii) A Folyamatszintű (Network Flow) Adathalmaz Jellemzői**

Amikor a hálózat gigabites sebességű (High Speed Networks), minden egyes csomag fejlécének vizsgálata leterhelné a CPU-t. Ilyenkor a routerek/switchek csak az ún. **Folyamatokat (Flows)** rögzítik.

- **Mi az a Flow?** Egy egyirányú (unidirectional) csomagsorozat két gép között egy adott időintervallumban. Egy folymatot 5 dolog azonosít (5-tuple): Forrás IP, Cél IP, Forrás Port, Cél Port, Protokoll (+ ToS, azaz Type of Service).
- **Hogyan készül?** A Cisco NetFlow V.9 (vagy az IPFIX szabvány) összesíti az adatokat, és kiküldi egy kollektornak (pl. `nfdump`).
- **Mikor zár le a router egy flow-t?**
  1. Ha $X$ másodperce ($10-600$s) nincs új csomag az adott kapcsolatban (idle).
  2. Ha a folyamat túl sokáig él (pl. letelt a 30 perc maximum).
  3. Ha TCP FIN vagy RST flag-et lát.
- **A kinyert Flow Feature-ök:** Ezeknél nincs Payload (tartalom) vagy TCP Flag részletezés (mint a packet level-nél). Csak aggregált adatok vannak: Flow kezdete, Időtartam (Duration), Protokoll, IP-k, Portok, Összes Bájt (bytes), Csomag/másodperc (pps), Bit/másodperc (bps), Bájt/csomag (bpp).
- *(Ebből az adatból a könyv szerzői 4 héten keresztül gyűjtöttek normál és támadó forgalmat).*

**(iv) A Portscan Adathalmaz Jellemzői**

Ezt az adathalmazt kifejezetten a portszkennelési technikák (amelyek a legtöbb betörés előfutárai - Probe attack) felderítésére hozták létre.

Az `Nmap` eszközzel koordinált támadásokat indítottak a labor szervere ellen. 3 héten át gyűjtötték a Port Mirror (Tükrözött port) segítségével, majd a zaj (felesleges csomagok) kiszűrése után kinyerték belőle az ehhez specifikus attribútumokat.

### 4.3.3 Hálózati Szimuláció (Network Simulation)

Mivel élő, valós hálózatokból elfogulatlan (unbiased), tökéletesen felcímkézett támadási adatokat kinyerni szinte lehetetlen és rendkívül drága, a kutatók gyakran fordulnak a szimulátorokhoz.

**NS-3 Szimulátor:** Egy nyílt forráskódú, diszkrét eseményvezérelt (discrete event) hálózati szimulátor.

- **Technikai jellemzők:** Teljes egészében C++ nyelven íródott, de Python kötésekkel (bindings) is programozható. Az IP architektúra vermét (IP stack) szimulálja.
- **Miért ideális ML/IDS kutatáshoz?**
  - Támogatja a pcap (Packet Capture) könyvtárat, így a szimulált csomópontokon (nodes) áthaladó forgalom pontosan úgy menthető el tcpdump formátumban, mintha egy valós hálózatot hallgatnánk le.
  - Automatikus memóriakezeléssel, beágyazott tanítási lehetőséggel és rugalmas IP-címzéssel rendelkezik.
  - Tetszőleges hálózati topológiák hozhatók létre benne (pl. a kutatók gyakran használják a "súlyzó" / dumbbell topológiát DDoS támadások szimulálására és az ML modellek tesztelésére).

> 💡 **ML/IDS Szempont:** A szimulátorok legnagyobb előnye az ML modellek tanításánál a **Ground Truth (Alapigazság)** megléte. Mivel mi írjuk a szimulációs szkriptet, másodpercre pontosan tudjuk, hogy mikor, melyik csomópont indított támadást. Így a kinyert pcap fájl címkézése (labeling) $100\%$-ig hibátlan lesz, ami elengedhetetlen a felügyelt (Supervised) tanuláshoz.

### A TUIDS Adathalmazok Részletes Felépítése

A korábban említett TUIDS adathalmaz részletes feature-listája rávilágít, hogy mi a különbség a folyamatszintű (Flow) elemzés és a specifikus (Portscan) elemzés között.

#### 1. Folyamatszintű Jellemzők (Table 4.11: Flow Features)

Míg a csomagszintű (Packet) adatoknál $50$ jellemzőt nyertek ki, a NetFlow alapú elemzésnél mindössze $24$ aggregált attribútumot használnak. Ezek három csoportba oszlanak:

- **Alapvető (Basic):** Időtartam, Protokoll, IP-k, Portok, TCP Flag-ek megléte (URG, ACK, PSH, RST, SYN, FIN), átvitt bájtok száma, és a Land támadást jelző bináris érték.
- **Időablak-alapú (Time-window):** A $T$ időintervallumon belüli folyamatok (flows) száma (pl. hány különböző kapcsolat ment ugyanarra a cél IP-re).
- **Kapcsolat-alapú (Connection-based):** Az utolsó $N$ darab folyamat (flow) statisztikái.

#### 2. Az Adathalmazok Statisztikái

- **Packet vs. Flow eloszlás:** Jól látható, hogy a kutatók igyekeztek elkerülni a KDD '99 hibáját. Itt a Normál forgalom ($\sim 55-58\%$) és a DoS támadások ($\sim 35-40\%$) aránya sokkal kiegyensúlyozottabb, így az ML modell nem fog túltanulni (overfit) a normál adatokon.
- **Portscan Típusok:** A portszkenneléses adathalmazban a klasszikus szkennelések (SYN, ACK, FIN) mellett szerepelnek az egzotikusabbak is, mint az Xmas (karácsonyfa: URG+PSH+FIN flag beállítva), a Null (nincs flag beállítva) és a Maimon szkennelés.

#### 3. A Portscan Adathalmaz Jellemzői (Table 4.14: Portscan Features)

A portszkennelések detektálására a TUIDS egy specifikus, $24$ attribútumból álló csomagszintű vektort használ.

> 💡 **Biztonsági fókusz:** Figyeld meg, hogy itt kiemelten szerepelnek a TCP Flagek (CWR, ECN, URG, ACK, PSH, RST, SYN, FIN) és a Window size (Ablakméret). Ennek oka, hogy a modern portszkennerek (mint az Nmap) a TCP fejléc ezeket a mezőit manipulálják (pl. szándékosan hibás flag-kombinációkat küldenek), hogy a válaszból kikövetkeztessék a célgép operációs rendszerét (OS Fingerprinting) anélkül, hogy teljes kapcsolatot építenének fel.

---

## 4.4 Összegzés (Discussion)

A 4. fejezet legfontosabb megállapításai a hálózati anomália detekció kapcsán:

- **Az IDS szerepe:** A behatolásjelző rendszerek (legyen az Hoszt-alapú / HIDS vagy Hálózat-alapú / NIDS) a megelőző védelem (tűzfalak, jelszavak) mellett a második védelmi vonalat jelentik.
- **Miért Anomália-alapú?** A szabályalapú (misuse-based) rendszerek kizárólag a már ismert támadások felismerésére jók. Mivel a támadási technikák folyamatosan fejlődnek, a kutatások fókusza áttolódott a gépi tanulással támogatott anomália-alapú (ANIDS) megoldásokra.

**A működési módok kihívásai:**

- **Felügyelt (Supervised):** Nagy minőségű, tökéletesen tiszta "normál" forgalmat és címkézett támadásokat tartalmazó adathalmazt szerezni drága és időigényes.
- **Felügyelet nélküli (Unsupervised):** A klaszterek emberi vagy algoritmus általi "címkézése" (labeling) bizonyos alapfeltevéseken nyugszik (pl. az anomália mindig a kisebb klaszter). Ez a valóságban sokszor megdől (pl. DDoS esetén).
- **Hibrid (Hybrid):** Bár megpróbálja egyesíteni a fenti kettő előnyeit, rendkívül komplex feladat megoldani, hogy egy újonnan felfedezett behatolás szignatúráját ütközésmentesen (without conflict) emeljék át a szabálybázisba.

**Az Adathalmazok jelentősége:** Egy ANIDS teljesítményének hiteles értékeléséhez elengedhetetlen egy elfogulatlan (unbiased) referencia adathalmaz (benchmark dataset). Ennek létrehozása egy élő hálózatban, ahol a támadások teljes palettáját rögzíteni kell anélkül, hogy a normál felhasználókat megzavarnák, az egyik legnehezebb feladat a hálózatbiztonsági kutatásokban.
