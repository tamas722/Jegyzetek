# CCNA Bővített Jegyzet – 32. Lecke: A Szállítási Réteg (Layer 4) – TCP és UDP

Ebben a leckében az OSI modell 4. rétegét (Transport Layer) vizsgáljuk meg, amely a végpontok (Hostok) közötti adatátvitelért felel. Megismerkedünk a réteg alapvető funkcióival, a Port számokkal, és részletesen összehasonlítjuk a két legfontosabb Layer 4 protokollt: a megbízható TCP-t és a gyors UDP-t.

## 1. A Layer 4 (Szállítási Réteg) Alapvető Funkciói

A Layer 4 feladata a végpontok (End Hosts) közötti "transzparens" adatátvitel biztosítása. A számítógép operációs rendszere (vagy a böngésző) egyszerűen átadja az adatot a 4. rétegnek, amely beburkolja azt (Layer 4 header), és a hálózat (Layer 1-3) segítségével eljuttatja a célhoz, anélkül, hogy a végpontoknak tudniuk kellene a hálózat fizikai felépítéséről.

A Layer 4 két legfőbb feladata:

- **Címzés (Layer 4 Addressing) / Port számok:** Megkülönbözteti, hogy az adat melyik alkalmazásnak vagy szolgáltatásnak szól.
- **Szolgáltatások biztosítása (vagy mellőzése):** A választott protokolltól (TCP vagy UDP) függően biztosíthat hibajavítást, sorrendezést és megbízhatóságot.

## 2. A Port Számok és a Session Multiplexing

Ahhoz, hogy egy számítógép egyszerre tudjon több dolgot csinálni az Interneten (pl. 3 böngészőfül nyitva, miközben fut a Skype és egy fájlletöltés), a Layer 4 a **Port számokat** használja. A port számok egy 16 bites értékek (0-tól 65535-ig terjednek).

- **Destination Port (Cél Port):** A szerver (vagy szolgáltatás) portja. Ebből tudja a túloldal, hogy az adat melyik alkalmazásnak szól (pl. ha a Cél Port 80, akkor a webszerver (HTTP) fogja feldolgozni a kérést).
- **Source Port (Forrás Port):** A kliens (a te géped) által véletlenszerűen generált szám. Amikor a szerver válaszol, ezt a számot használja majd Cél Portként.

**Session Multiplexing (Munkamenet Többszörözés):**
Ennek a két számnak (Source IP + Source Port ↔ Destination IP + Destination Port) az egyedi párosítása határoz meg egy "Sessiont" (Munkamenetet). A te böngésződ ebből tudja megkülönböztetni, hogy a szervertől beérkező válasz a Facebook-os vagy a YouTube-os füledhez tartozik-e.

### A Port Számok Tartományai (IANA kiosztás)

(A vizsgára ezeket a kategóriákat és határokat tudni kell!)

- **Well-known Ports (Jól ismert portok):** `0 - 1023`
  Ezeket a szerverek fő protokolljai használják (pl. HTTP, SSH). A használatuk szigorúan szabályozott.
- **Registered Ports (Regisztrált portok):** `1024 - 49151`
  Ezeket különböző szoftvercégek (pl. játékok, alkalmazások) regisztráltatják maguknak.
- **Ephemeral (Múló / Dinamikus / Privát) Ports:** `49152 - 65535`
  Ezt a tartományt használják a kliens gépek a Source Port véletlenszerű generálására, amikor elindítanak egy kommunikációt.

## 3. TCP (Transmission Control Protocol) – A Megbízható

A TCP a legszélesebb körben használt protokoll, ha az adatok pontossága és hiánytalansága a cél (pl. weblapok betöltése, fájlok letöltése, emailek küldése).

**Jellemzői:**

- **Connection-Oriented (Kapcsolatalapú):** Mielőtt egyetlen bájtnyi adatot is küldene, a két gép felépít egy kapcsolatot egymással.
- **Reliable (Megbízható adatátvitel):** A fogadó félnek minden egyes beérkezett csomagot (Szegmenst) nyugtáznia kell (Acknowledge).
- **Error Recovery (Hibajavítás):** Ha a küldő nem kap nyugtát egy adott szegmensről, bizonyos idő után automatikusan újraküldi (Retransmission) azt.
- **Sequencing (Sorrendezés):** Minden TCP szegmens kap egy Sorszámot (Sequence Number). Ha a csomagok összekeverednek az Interneten, a fogadó gép a sorszámok alapján sorba rakja őket.
- **Flow Control (Forgalomszabályozás):** A fogadó gép egy "Ablakméret" (Window Size) értékkel folyamatosan szabályozza, hogy a küldő milyen sebességgel (mennyi adatot egyszerre) küldhet neki, hogy ne terhelje túl.

### A TCP "Kézfogások" (Handshakes)

#### A Kapcsolat Felépítése: A 3-Way Handshake (3 Lépéses Kézfogás)

(Ezt a három lépést kívülről kell tudni! Az üzenetek a TCP Header "Flags" - Zászlók mezőit használják).

1. A Kliens küld egy **SYN (Synchronize)** üzenetet a szervernek (Ezzel egy véletlenszerű sorszámot ajánl a kezdéshez, pl. Seq: 10).
2. A Szerver válaszol egy **SYN-ACK** üzenettel. (Elfogadja a kliens sorszámát egy Ack értékkel, és küld egy saját véletlenszerű sorszámot a SYN mezőben).
3. A Kliens küld egy **ACK** üzenetet. A kapcsolat felépült, kezdődhet az adatcsere.

#### A Kapcsolat Bontása: A 4-Way Handshake (4 Lépéses Bontás)

1. A Kliens küld egy **FIN (Finish)** üzenetet.
2. A Szerver nyugtázza ezt egy **ACK**-kal.
3. A Szerver lezárja a saját részét is, és küld egy **FIN** üzenetet.
4. A Kliens nyugtázza ezt egy **ACK**-kal. A kapcsolat lezárult.

### Forward Acknowledgment (Előretekintő Nyugtázás)

(Kvíz csapda!) Amikor a fogadó fél nyugtáz egy TCP csomagot, **NEM** a megkapott sorszámot (Sequence number) küldi vissza! Az "Acknowledge" mezőbe mindig azt a sorszámot írja bele, amit a **KÖVETKEZŐ LÉPÉSBEN vár!**

Példa: Ha a Kliens küld egy csomagot Seq: 20 azonosítóval, a Szerver nyugtája Ack: 21 lesz.

## 4. UDP (User Datagram Protocol) – A Gyors

Az UDP egy végtelenül egyszerű, "buta" protokoll. Csak egy nagyon pici fejléce van (Source Port, Destination Port, Length, Checksum).

**Jellemzői:**

- **Connectionless (Kapcsolatmentes):** Nem épít fel kapcsolatot, nem fog kezet. Egyszerűen csak ráöntözi az adatot a hálózatra.
- **Best-Effort (Nincs garancia):** Nem kér nyugtát a megérkezésről (Nincs Acknowledgment).
- **Nincs Hibajavítás:** Ha egy csomag elvész az Interneten, az UDP nem küldi újra! A csomag végleg elveszett.
- **Nincs Sorrendezés és Flow Control:** Nem számozza meg a csomagokat, és nem érdekli, ha a fogadó gép nem bírja a tempót.

**Mikor használjuk?**
Kifejezetten olyan alkalmazásoknál, ahol a **Sebesség és a Késleltetés hiánya** a legfontosabb, és egy-két kieső csomag nem okoz gondot. (Például VoIP telefonhívások, Videókonferencia, Élő közvetítések - ha megakad a kép, nem akarjuk másodpercekkel később megkapni az "elveszett" kockákat, inkább megyünk tovább a jelenbe). Használják még apró, gyors kérésekre is (pl. DNS lekérdezések).

## 5. A Legfontosabb Port Számok Listája (Memorizálandó!)

A CCNA vizsgán a portszámok kérdései alapvetőek. Ezt a táblázatot fejből kell tudni!

| Protokoll | TCP vagy UDP? | Port Szám | Mire való? |
|---|---|---|---|
| **FTP** | TCP | 20 és 21 | Fájlátvitel (File Transfer Protocol). |
| **SSH** | TCP | 22 | Biztonságos távoli parancssoros bejelentkezés (Secure Shell). |
| **Telnet** | TCP | 23 | Titkosítatlan, elavult távoli bejelentkezés. |
| **SMTP** | TCP | 25 | Email küldés (Simple Mail Transfer Protocol). |
| **DNS** | TCP és UDP | 53 | Domain nevek IP-címre fordítása. (A Boson kvíz ezt kérdezi!) |
| **DHCP** | UDP | 67 és 68 | Automatikus IP-cím kiosztás (Dynamic Host Config Protocol). |
| **TFTP** | UDP | 69 | Egyszerűsített, gyors fájlátvitel (Trivial FTP). |
| **HTTP** | TCP | 80 | Titkosítatlan weboldalak elérése. |
| **POP3** | TCP | 110 | Email letöltés / fogadás. |
| **SNMP** | UDP | 161 és 162 | Hálózati eszközök menedzselése és monitorozása. |
| **HTTPS** | TCP | 443 | Titkosított weboldalak elérése. |
| **Syslog** | UDP | 514 | Eszközök naplóüzeneteinek (logok) szerverre küldése. |

## Kvíz a 32. Leckéhez (Boson ExSim Bónusz Kérdéssel!)

Ellenőrizd a tudásod a TCP és UDP alapjairól!

### Kérdés 1

Az alábbiak közül melyik számít "Well-known" (Jól ismert) port számnak az IANA kiosztása alapján?

- A) 1010
- B) 2001
- C) 4023
- D) 65000

**Helyes Válasz: A**

**Magyarázat:** A Well-known portok tartománya 0-tól 1023-ig terjed. Bár a 1010-hez nincs konkrét híres protokoll kötve a vizsgán, számszakilag egyedül ez esik ebbe a tartományba. A 2001 és a 4023 a Registered, a 65000 az Ephemeral (Dinamikus) tartományba tartozik.

### Kérdés 2

Melyik port tartományból kell a kliens gépeknek (Hostoknak) véletlenszerűen választaniuk a Forrás Port (Source Port) számát egy kommunikáció megkezdésekor az IANA specifikációk szerint?

- A) Well-known
- B) Registered
- C) Ephemeral
- D) Reserved

**Helyes Válasz: C**

**Magyarázat:** Az Ephemeral (Múló / Dinamikus) portokat (49152 - 65535) tartják fenn erre a célra. A Reserved (Foglalt) nevű tartomány egyáltalán nem létezik a kategóriák között.

### Kérdés 3

Az alábbi funkciók közül melyik **HÁROM** az, amit **KIZÁRÓLAG a TCP** biztosít, az UDP viszont NEM? (Válassz HÁRMAT!)

- A) Layer 4 Addressing (Címzés Port számokkal)
- B) Error Recovery (Hibajavítás / Újraküldés)
- C) Session Multiplexing (Munkamenet többszörözés)
- D) Flow Control (Forgalomszabályozás)
- E) Sequencing (Sorrendezés)

**Helyes Válasz: B, D és E**

**Magyarázat:** A TCP a robusztus protokoll, amely garantálja a megérkezést (Error Recovery), a helyes sorrendet (Sequencing) és védi a fogadót a túlterheléstől (Flow control a Window Size segítségével). A Layer 4 portcímzést és a Session Multiplexinget az UDP is pontosan ugyanúgy tudja biztosítani.

### Kérdés 4

Az alábbi Alkalmazási Réteg (Application Layer) protokollok közül melyik **HÁROM** használ TCP-t a megbízható kommunikáció érdekében? (Válassz HÁRMAT!)

- A) SMTP
- B) SNMP
- C) HTTPS
- D) DHCP
- E) Syslog
- F) SSH

**Helyes Válasz: A, C és F**

**Magyarázat:** Az SMTP (Email), a HTTPS (Web) és az SSH (Titkosított távoli elérés) garantált, hibamentes adatátvitelt igényel, ezért TCP-t használnak. Az SNMP, a DHCP és a Syslog gyors, "lődd el és felejtsd el" jellegű apró üzenetekkel operál, ezért az UDP-t preferálják.

### Kérdés 5

A PC1 és a SRV1 között egy aktív TCP kapcsolat él. A SRV1 kap egy TCP Szegmenst a PC1-től, amiben a Sequence number (Sorszám) 27. Amikor a SRV1 nyugtázza a szegmens megérkezését, mi lesz a TCP Header Acknowledgment mezőjének értéke? (Feltételezve, hogy a TCP Window Size pontosan 1).

- A) 26
- B) 27
- C) 28

**Helyes Válasz: C**

**Magyarázat:** A TCP "Forward Acknowledgment" (Előretekintő nyugtázás) logikát használ. Soha nem azt a számot küldi vissza, amit megkapott, hanem azt, amit **KÖVETKEZŐKÉNT VÁR!** Mivel megkapta a 27-est, a következő, amit vár, a 28-as lesz. (Ha 27-et küldene vissza, a PC1 azt hinné, hogy a 27-es elveszett, és újra elküldené azt).

### Bónusz Kérdés (Boson ExSim vizsgakérdés - Drag-and-Drop Feladat)

(Húzd a megfelelő Application Layer protokollt a megfelelő Transport Layer protokollhoz!)

A megfejtés a táblázat alapján:

**Csak TCP-t használ (Megbízható, lassabb):**

- FTP (Fájlátvitel, Port 20/21)
- HTTP (Web, Port 80)
- SMTP (Email, Port 25)

**Csak UDP-t használ (Gyors, nincs garancia):**

- DHCP (IP kiosztás, Port 67/68)
- SNMP (Eszköz menedzsment, Port 161/162)
- TFTP (Trivial FTP, gyors fájlátvitel, Port 69)

**Mindkettőt (TCP ÉS UDP) is használja a helyzettől függően:**

- DNS (Domain név fordítás, Port 53). Alapesetben a gyors UDP-t használja a névlekérdezésekhez, de ha nagy adatmennyiséget kell mozgatni (pl. Zone Transfer szerverek között), akkor átvált TCP-re. Ezt nagyon szeretik kérdezni a vizsgán!
