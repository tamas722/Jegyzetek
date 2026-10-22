# 9. Fejezet: Nyitott Kérdések, Kihívások és Záró Gondolatok (Open Issues, Challenges and Concluding Remarks)

Bár az elmúlt évtizedben rengeteg módszert és rendszert fejlesztettek ki a hálózati behatolások kivédésére, a terület még mindig számos nyitott kutatási kérdéssel és kihívással küzd. A legfőbb probléma továbbra is a NIDS-ek kiértékelésére szolgáló teljesítménymutatók (performance metrics) alkalmassága. Egy NIDS kiértékelésekor négy alapvető minőséget kell mérni: **pontosság (accuracy)**, **teljesítmény (performance)**, **teljesség (completeness)** és **adatminőség (data quality)**.

**A "Kielégítő garancia" (Pretty Good Assurance) elve:** Egy behatolásjelző rendszer tökéletes, formális matematikai bizonyítása minden ismert sebezhetőségre extrém nehéz és drága. Ezért a gyakorlatban a "kielégítő garancia" elvét alkalmazzák, amely megengedi a fuzzy (nem egyértelmű) döntéseket, a kompromisszumokat (trade-offs) és a priorizálást. Ennek mérlegelnie kell, mennyi munkába kerül a támadónak egy sebezhetőség felfedezése és kihasználása.

**Adat-szűk keresztmetszet:** A megbízható, érvényes és minőségi adatok hiánya a modern NIDS-ek rossz teljesítményének és pontatlanságának egyik legnagyobb oka.

**Telepítési nehézségek:** A gyakorlati szakemberek számára továbbra is komoly kihívás eldönteni, hogy hova helyezzék el a NIDS-t a hálózatban, és hogyan konfigurálják azt optimálisan egy több szereplős környezetben.

Az alábbiakban pontokba szedve tekintjük át a legfontosabb megoldandó kihívásokat.

## 9.1 Futásidejű Korlátok (Runtime Limitations)

A valós idejű anomália detekció legfőbb korlátja az idő. Egy ideális valós idejű NIDS-nek csomagvesztés (packet loss) nélkül kellene befognia és megvizsgálnia minden egyes csomagot, alkalmazkodva az aktuális hálózati forgatókönyvekhez az azonnali és pontos észlelés érdekében. A gigabites hálózatokon ez óriási kihívás.

## 9.2 A Téves Riasztási Arány Csökkentése (Reducing the False Alarm Rate)

Egy NIDS-nek vagy detekciós módszernek el kell kerülnie a magas téves riasztási arányt (false positive rate). Bár az anomália-alapú behatolásjelzőknél lehetetlen teljesen elkerülni a téves riasztásokat, a fejlesztők célja, hogy megközelítsék a "zajmentes" (zero false alarm) működést bármilyen környezetben, mindezt úgy, hogy a rendszer futásidőben adaptív maradjon.

## 9.3 Dimenziócsökkentési Kérdések (Issues in Dimensionality Reduction)

A hálózati forgalomra jellemző attribútumok (feature-ök) gyakran:

- Nagyon alacsony varianciával (változatossággal) rendelkeznek.
- Erősen korrelálnak egymással, vagy funkcionálisan függenek egymástól.

Egy olyan módszer kifejlesztése, amely kiválasztja a jellemzők optimális halmazát (optimal set of features) az anomália detekcióhoz – anélkül, hogy a rendszer teljesítménye romlana – továbbra is nyitott probléma.

## 9.4 A Hálózatvédelmi Mechanizmusok Számítási Igényei (Computational Needs)

A modern, egyre kifinomultabb és többrétegű (layered) támadások elhárítása érdekében a védelmi mechanizmusokat gyakran meglévő rendszerek tetejére építik újabb és újabb számítási modulok formájában.

**A probléma:** A nagysebességű hálózati technológiák folyamatos fejlődése mellett ezek az inkrementális "rátétek" (bővítmények) kontraproduktívvá válhatnak, és szűk keresztmetszetet (bottleneck) okozhatnak a detekciós teljesítményben. Az IDS fejlesztőknek vigyázniuk kell, hogy a komplexitás növelése ne menjen a hatékonyság rovására.

## 9.5 Általános (Generic) Anomália Detekciós Rendszerek Tervezése

A legtöbb NIDS és behatolásjelző módszer erősen környezetfüggő (environment-dependent). Egy ideális rendszernek azonban függetlennek kellene lennie az alatta lévő architektúrától. Egy olyan NIDS kifejlesztése, amely konzisztens teljesítményt nyújt teljesen eltérő környezetekben is, kulcsfontosságú kihívás.

## 9.6 Kifinomult Anomáliák Kezelése (Handling Sophisticated Anomalies)

Az anomáliák természete az idő múlásával folyamatosan változik, mivel a behatolók folyamatosan adaptálják a támadásaikat, hogy kikerüljék (evade) a meglévő védelmi megoldásokat. Egy jó NIDS-nek képesnek kell lennie arra, hogy önmagát is dinamikusan frissítse a helyi hálózaton vagy az interneten felbukkanó legújabb anomáliák alapján.

## 9.7 Alkalmazkodás az Ismeretlen Támadásokhoz (Adaptability to Unknown Attacks)

Szinte minden nap új sebezhetőségek (vulnerabilities) és exploitok (kódkivitelezések) jelennek meg. A profilok vagy a szignatúra-bázisok dinamikus frissítése a valós idejű teljesítmény romlása és a meglévő szignatúrákkal való ütközés (conflict) nélkül az egyik legnehezebb feladat.

## 9.8 Nagyskálás Támadások Detektálása és Kezelése (Detecting and Handling Large-Scale Attacks)

A mai elosztott (DDoS) támadások pillanatok alatt több ezer gépet kompromittálhatnak, és azonnal megbéníthatják a hálózatot. A NIDS-nek nemcsak korai szakaszban kell felismernie a támadást, hanem azonnal kontrollálnia is kell a támadás sebességét (attack rate) úgy, hogy közben a legitim (valós) felhasználók kiszolgálása ne sérüljön.

## 9.9 Infrastruktúra Elleni Támadások (Infrastructure Attacks)

A jelenlegi elosztott támadások gyakran célozzák a hálózati végpontokat és szervereket. Azonban az elárasztás (flooding) mellett a legkritikusabb támadások a hálózat lelkét jelentő infrastruktúra ellen irányulnak, hogy minél több felhasználó számára tegyék elérhetetlenné a szolgáltatásokat. Példák ilyen kiemelt célpontokra:

- Gyökér DNS szerverek (Root DNS servers)
- Tanúsítvány szerverek (Certificate servers)
- LDAP szerverek
- Kulcselosztó szerverek (Key distribution servers)

## 9.10 Nagy Intenzitású Támadások (High Intensity Attacks)

Egy mai DDoS támadás forgatókönyve több tízezer zombigépet (compromised hosts) is magában foglalhat. Ezek a támadások másodpercek alatt terjednek el, és képesek az egész hálózatot azonnal térdre kényszeríteni. Az IDS tervezőknek nem csupán az észlelésre kell fókuszálniuk, hanem arra is, hogy hogyan lehet az ilyen gigantikus intenzitású forgalmat azonnal megfékezni.

## 9.11 Újszerű, Találékony Támadások (More Inventive Attacks)

Régebben a legtöbb támadást csak azért hajtották végre a hackerek, hogy bizonyítsák a képességeiket (dicsekvés). Ma már a kifinomult támadások mögött társadalmi, politikai vagy komoly bűnügyi (criminal) motivációk állnak.

- **Új célpontok:** Mobilszolgáltatások, SMS-ek és az internetre csatlakozó vezeték nélküli okoseszközök.
- **Kritikus rendszerek:** Az internet a fejlett államok jogrendszerének és rendfenntartásának szerves részévé vált. Sőt, ma már az Elektronikus Szavazógépeket (Electronic Voting Machines - EVM) is gyakran csatlakoztatják a hálózatra, ami extrém veszélyeket rejt. Egy olyan robusztus NIDS fejlesztése, amely képes ezeket a társadalomra veszélyes, újszerű támadásokat is nagy pontossággal elhárítani, kritikus kutatási terület.

## 9.12 Záró Gondolatok (Concluding Remarks)

Ebben a könyvben a szerzők a hálózati anomáliák (rosszindulatú kísérletek) jellemzésére és azok gépi tanulási (Machine Learning) technikákkal történő észlelésére fókuszáltak.

**Mit tanultunk meg ebből az anyagból?**

- Megismertük a protokollok gyengeségeiből fakadó, különböző hálózati rétegeket érintő sebezhetőségeket.
- Áttekintettük a réteg-specifikus (layer-specific) behatolások típusait és működési módjait.
- Bevezetésre kerültek a behatolásjelző gépi tanulási módszerek: felügyelt (supervised), felügyelet nélküli (unsupervised), valószínűségi (probabilistic), lágy-számítási (soft computing) és kombinált (combination) tanulók. Részletesen megvizsgáltuk ezek előnyeit és hátrányait.
- A szakirodalomban tapasztalható káosz elkerülése végett a könyv egy egységes támadási taxonómiát (osztályozási rendszert) adott.
- Bemutatta a támadók által használt offenzív, és a hálózatvédők által alkalmazható defenzív eszközöket és rendszereket.

**A végső cél:** A könyvben felvázolt algoritmusok és kihívások (amelyeket ebben a jegyzetben az elejétől a végéig kidolgoztunk) átfogó képet adnak a mérnökök, kutatók és rendszergazdák számára. Ennek az elméleti és gyakorlati tudásnak a birtokában a jövő szakemberei (mint Te is) képesek lesznek költséghatékony, robusztus és intelligens stratégiákat építeni a jövőbeli, egyre komplexebb és kifinomultabb kibertámadásokkal szemben.
