# 1. Fejezet: Bevezetés (Introduction)

## Alapfogalmak és az Internet fejlődése

A hálózatba kötött számítógépes rendszerek a modern társadalom mozgatórugói (pénzügyi tranzakciók, kormányzati szolgáltatások, közösségi média). Az Internet ezen tevékenységek alapja, amely több ezer különálló hálózat összekapcsolásából áll.

- **Infrastrukturális fejlődés:** Az internethasználók számának robbanásszerű növekedésével a hálózatok kapacitása is nő.
- **Példa a könyvből:** A Ciena, Cisco, Juniper és mások által fejlesztett 4. generációs technológia (4th Gen Technology) $8.8$ terabit/másodperc kapacitást és $100$ gigabites Ethernet technológiát használ.

### Biztonsági alapelvek (CIA triád)

Egy számítógépes rendszer vagy hálózat biztonsága akkor sérül, ha az alábbi három pillér valamelyike kompromittálódik:

1. **Titkosítás/Bizalmasság (Confidentiality):** Csak az arra jogosultak férjenek hozzá az adathoz.
2. **Rendelkezésre állás (Availability):** A rendszer mindig elérhető legyen a jogosult felhasználóknak (egy DoS támadás ezt teszi tönkre).
3. **Integritás (Integrity):** Az adatok ne sérüljenek vagy módosuljanak illetéktelenül.

<img width="640" height="547" alt="1" src="https://github.com/user-attachments/assets/c2033290-a416-42a1-9b72-693e7fa4c3de" />


> Kontinensek szerinti internethasználói statisztikát mutató ábrát, ami jól szemlélteti a hálózatok globális kiterjedését.

<img width="546" height="572" alt="2" src="https://github.com/user-attachments/assets/8d0a24fa-f20d-43e2-af57-5328bd7d2b77" />


>Globális, fejlett és fejlődő világ internetfelhasználóinak lineáris növekedését mutató grafikont.

## 1.1 Az Internet és a modern hálózatok

A hálózatok megértéséhez rétegzett architektúrákat (layered architectures) használunk, mint az OSI modell vagy a TCP/IP modell. Minden réteg egy adott funkcióért felel. A komplexitás növekedése azonban növeli a meghibásodások és a támadási felületek számát.

### Behatolásjelző rendszerek (Intrusion Detection System – IDS)

Az IDS a számítógépes rendszerekben és hálózatokban zajló eseményeket figyeli, és jogosulatlan hozzáférésre utaló jeleket keres. A Hálózati IDS (NIDS) a hálózat különböző pontjain (pl. szenzorokon keresztül) figyeli a forgalmat. A biztonságban az IDS a "második védelmi vonal" a tűzfalak és az autentikáció után.

<img width="702" height="618" alt="3" src="https://github.com/user-attachments/assets/7e3259ff-97f0-4664-afa2-0f9a820e50fc" />

> DMZ (Demilitarizált Zónát), a belső LAN-t és az IDS elhelyezkedését bemutató hálózati topológia ábrát.

### Hálózati anomália detekció 2 fő megközelítése

| Jellemző | Szabályalapú (Rule-based / Misuse-based) | Anomália-alapú (Anomaly-based) |
| :--- | :--- | :--- |
| **Működési elv** | Előre definiált mintákat, ún. szignatúrákat keres az adatokban. | Eltéréseket keres a hálózat "normális" vagy megszokott viselkedéséhez képest. |
| **Előny** | Nagyon alacsony a téves riasztások száma (false alarm), pontos a már ismert támadásoknál. | Képes felismerni az új, még sosem látott támadásokat (Zero-day attack). |
| **Hátrány** | Teljesen vak az új/ismeretlen támadásokkal szemben, amiknek még nincs szignatúrájuk. | Magasabb lehet a téves riasztási arány, ha a "normál" hálózati forgalom hirtelen, de legálisan megváltozik. |
| **Gyakorlati példa** | A Snort IDS riasztást küld, ha a csomagban pontosan megtalálja az "X" féreg hexadecimális kódját. | A gépi tanulási modell riasztást küld, mert a szerver hajnali 3-kor hirtelen 50 GB adatot kezdett el kifelé küldeni (ami eltér a betanult mintától). |

## 1.2 Hálózati sebezhetőségek (Network Vulnerabilities)

**Miért sebezhetőek a hálózatok?** A sérülékenységek a tervezésből, konfigurációból vagy implementációból fakadó gyengeségek.

1. **Hardver/Szoftver tervezési hibák:** Bugok a kódokban.
2. **Rossz konfiguráció:** Tapasztalatlanság vagy hanyagság miatt (pl. alapértelmezett jelszavak meghagyása egy routeren).
3. **Menedzsment hiányosságok:** Gyenge házirendek (policy-k), jogosultságkezelés (ki mihez férhet hozzá) és a forgalom ritka ellenőrzése.

## 1.3 Anomáliák a hálózatokban

Az anomáliák (anomalies / outliers) olyan hálózati események, amelyek eltérnek a normális, várt viselkedéstől. Két fő okból jöhetnek létre:

- **Teljesítmény alapú (Performance-related):** Eszközök meghibásodása, pl. elrontott router konfiguráció okozta csomagvesztés.
- **Biztonság alapú (Security-related):** Rosszindulatú tevékenységek (támadások).

### A biztonsági anomáliák 6 fő kategóriája (Példákkal)

1. **Fertőzés (Infection):** A célrendszer megfertőzése kártékony fájlokkal.
   - **Példa:** Vírusok, hálózati férgek (Worms), amelyek önmagukat másolva terjednek a hálózatban.
2. **Kimerítés / "Robbantás" (Exploding):** A célrendszer túlcsordulásának előidézése szoftveres hibák kihasználásával.
   - **Példa:** Puffer-túlcsordulás (Buffer overflow) – a támadó túl sok adatot küld egy bemeneti mezőbe, hogy lefagyassza a programot vagy saját kódot futtasson.
3. **Feltérképezés (Probe):** Információgyűjtés a rendszer sérülékenységeiről (1:1, 1:m, m:1, m:n módokon).
   - **Példa:** Port szkennelés az nmap szoftverrel. A támadó végigpróbálja, hogy a szerveren nyitva van-e pl. a 22-es (SSH) vagy 80-as (HTTP) port.
4. **Csalás / Hamisítás (Cheat):** Hamis, abnormális hívófelek használata az identitás elrejtésére.
   - **Példa:** IP Spoofing (a támadó meghamisítja a feladó IP-címét a csomagban) vagy MAC Spoofing.
5. **Bejárás / Törés (Traverse):** A célrendszer kompromittálása az összes lehetséges kulcs kipróbálásával.
   - **Példa:** Brute force (nyers erő) és szótár alapú (dictionary) jelszótörő támadások (pl. FTP vagy SSH belépési próbálkozások másodpercenként 100-szor).
6. **Párhuzamosság (Concurrency):** A rendszer kapacitásának kimerítése óriási mennyiségű kéréssel (flood).
   - **Példa:** DDoS (Distributed Denial of Service) támadás. Több ezer zombi gép (Botnet) egyszerre próbálja megnyitni a weboldalt, amíg a szerver össze nem omlik.

## 1.4 Gépi tanulás (Machine Learning) a hálózatbiztonságban

A modern hálózatokban keletkező adat (naplófájlok, csomagforgalom) emberi ésszel és manuális szabályokkal már átláthatatlan. A gépi tanulás célja, hogy rejtett, hasznos mintázatokat (patterns) találjon ezekben a hatalmas adathalmazokban.

**A legfőbb kihívás: Az általánosítás (Generalization)**
A modellnek nem szabad egyszerűen "betanulnia" a korábbi adatokat (ez az ún. túltanulás / overfitting), hanem képesnek kell lennie arra, hogy korábban sosem látott forgalomról is el tudja dönteni: ez normális vagy támadás?

### A Gépi Tanulási Algoritmusok Típusai

**1. Felügyelt tanulás (Supervised Learning):**
- **Működése:** A betanító adathalmaz (training dataset) címkézve van (labeled). A címkézést általában szakemberek végzik, ami drága és időigényes.
- **Típusa:** Általában Osztályozás (Classification). Az algoritmus (Classifier) eldönti, melyik kategóriába tartozik az új adat.
- **Hálózati ML Példa:** A modell kap egymillió hálózati folyamot (flow), ami fel van címkézve "Normál" vagy "DoS_támadás" néven. Megtanulja, hogy a "DoS" esetén a másodpercenkénti csomagszám nagyon magas, a csomagméret kicsi. Élesben ezt a mintát fogja keresni.

**2. Felügyelet nélküli tanulás (Unsupervised Learning):**
- **Működése:** Nincsenek előre megadott címkék. Az algoritmus a belső tulajdonságok, hasonlóságok és távolságok (matrikák) alapján maga próbálja csoportosítani az adatokat.
- **Típusa:** Klaszterezés (Clustering). Az azonos csoportba (klaszterbe) kerülő adatok hasonlítanak egymásra.
- **Hálózati ML Példa:** A modell megkapja a hálózati forgalmat, és rájön, hogy a forgalom $99\%$-a egy nagy klasztert alkot (ez lesz a normál forgalom). A maradék $1\%$, ami egy apró, különálló klaszterbe kerül (vagy teljesen elszigetelt), nagy eséllyel anomália (pl. rejtett adatlopás).

**3. Félig felügyelt tanulás (Semi-supervised Learning):**
Kevés címkézett és rengeteg címkézetlen adatot használ. Esetleg csak a "normál" osztályból vannak címkézett adatai, és minden mást, ami ettől eltér, anomáliának tekint.

**További módszerek:**
- **Valószínűségi (Probabilistic):** Statisztikai és valószínűségszámítási modellek.
- **Soft Computing:** Képes kezelni a bizonytalanságot (pl. Genetikus algoritmusok, Neurális hálózatok, Fuzzy logika).
- **Tudásbázisú (Knowledge-based):** Szakértői rendszerek, szabályalapú rendszerek.
- **Hibrid (Hybrid):** Több különböző algoritmus kombinációja a jobb felismerési arány érdekében.

## 1.5 - 1.7 A könyv céljai, hozzájárulása és felépítése (Összefoglaló)

A könyv 3 fő részre, összesen 9 fejezetre van bontva, és az alábbi kulcsterületeket fedi le, amelyek megkülönböztetik más biztonsági könyvektől:

- **Hálózatok és anomáliák áttekintése:** Sebezhetőségek és protokoll-specifikus támadások rétegek szerint.
- **Gépi tanulás fókusz:** Felügyelt, felügyelet nélküli, valószínűségi és hibrid ML módszerek technikai alkalmazása IDS rendszerekben.
- **Támadási taxonómia (Attack taxonomy):** A támadások konzisztens kategorizálása viselkedés alapján.
- **Jellemzőkiválasztás (Feature selection):** Az óriási adathalmazokból a redundáns adatok kiszűrése (pl. melyik hálózati paraméter számít igazán: a TCP flag-ek, vagy az IP TTL értéke? Ezzel gyorsítva a valós idejű észlelést).
- **IDS Értékelés (Evaluation):** Hogyan mérjük a teljesítményt? (Szenzitivitás, False Positive/Negative arányok, ROC görbék).
- **Gyakorlati tudás (Hands-on):** Támadások indítása, forgalom elfogása (packet/flow), adatok előfeldolgozása, és az IDS rendszerek tesztelése (pl. Snort).
- **Jövőbeli kihívások:** Hogyan fognak kinézni a jövő komplex támadásai, és hogyan kell felkészülnie rájuk a védelmi rendszereknek.
