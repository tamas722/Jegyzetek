# Mik az e-mail protokollok (POP3, SMTP és IMAP) és az alapértelmezett portjaik?

**Utolsó frissítés: 2025. szeptember 3.**
• 3 perc olvasás

Az e-mail a vállalati és személyes online kommunikáció elengedhetetlen része. Az e-mail protokollok határozzák meg a szerverek és kliensek közötti e-mail-csere mechanizmusát. Így teszik lehetővé, hogy helyesen küldjünk és fogadjunk üzeneteket a hálózaton.

Ebben a cikkben megtalálod a leggyakoribb e-mail protokollok magyarázatát – POP3, IMAP és SMTP. Mindegyiküknek sajátos funkciója és mechanizmusa van. Olvass tovább, hogy megtudd, melyik konfiguráció szolgálja legjobban a konkrét e-mail igényeidet.

## Mik az e-mail protokollok?

Az e-mail protokollok olyan szabályrendszerek, amelyek meghatározzák, hogyan küldik, fogadják és tárolják az e-maileket. A fő protokollok az **SMTP** az e-mailek küldéséhez, valamint a **POP3** és az **IMAP** az e-mailek fogadásához. Az SMTP kezeli az e-mail továbbítását a szerverek között, míg a POP3 letölti az e-maileket egy eszközre, és az IMAP szinkronizálja az e-maileket több eszköz között.

### E-mail protokollok listája

A szabványos e-mail protokollok listája a következőket tartalmazza:

- SMTP
- POP3
- IMAP

Mindegyikük másképp működik, és különböző szolgáltatást nyújt az e-mail fiókod kezeléséhez.

## Mi az SMTP?

Az **SMTP** a **Simple Mail Transfer Protocol** rövidítése, és felelős az e-mail üzenetek küldéséért. Ezt a protokollt e-mail kliensek és levelezőszerverek használják e-mailek cseréjére a számítógépek között.

### Hogyan működik az SMTP e-mail protokoll?

Egy levelezőkliens és az SMTP szerver egy adott e-mail porton keresztül létesített kapcsolaton kommunikál egymással. Mindkét entitás SMTP parancsokat és válaszokat használ a kimenő e-mailek feldolgozásához. A Simple Mail Transfer Protocolnak köszönhetően az üzenetek ugyanarról a fiókról különböző e-mail alkalmazásokban küldhetők.

## Mi a POP3?

### Hogyan működik a POP3 e-mail protokoll?

A **POP3** rövidítés a **Post Office Protocol version 3**-at jelenti, amely hozzáférést biztosít egy e-mail szerveren tárolt beérkező postafiókhoz. Végrehajtja az üzenetek letöltési és törlési műveleteit. Így amikor egy POP3 kliens csatlakozik a levelezőszerverhez, letölti az összes üzenetet a postafiókból. Ezután a helyi számítógépeden tárolja őket, és törli azokat a távoli szerverről.

Ennek a protokollnak köszönhetően offline módban is hozzáférhetsz az üzenetekhez helyben.

A modern POP3 kliensek lehetővé teszik, hogy megőrizd az üzeneteid másolatát a szerveren, ha kifejezetten ezt a beállítást választod.

## Mi az IMAP?

### Hogyan működik az IMAP e-mail protokoll?

Az **Internet Message Access Protocol (IMAP)** lehetővé teszi az e-mail üzeneteid elérését és kezelését az e-mail szerveren. Ez a protokoll lehetővé teszi mappák manipulálását, végleges törlést és hatékony keresést az üzenetek között. Lehetőséget ad e-mail jelzők beállítására vagy eltávolítására, vagy az e-mail attribútumok szelektív lekérésére. Alapértelmezés szerint minden üzenet a szerveren marad, amíg a felhasználó kifejezetten nem törli azokat.

Az IMAP támogatja több felhasználó csatlakozását egyetlen levelezőszerverhez.

## Alapértelmezett e-mail portok

Az e-mail portok olyan kommunikációs végpontok, amelyek meghatározzák, hogyan kell egy üzenetet továbbítani. Ez magában foglalja, hogy az üzenetet titkosítani kell-e és biztonságosan kell-e cserélni.

Az e-mail kliensed és a levelezőszervered közötti kapcsolat létrehozásához szükséged van az utóbbi IP-címére és egy portszámra. Ezeket az attribútumokat az IANA (Internet Assigned Numbers Authority) osztja ki.

Minden protokollnak megvannak a saját portszámai a csatlakozáshoz, és minden port más típusú titkosítást támogat.

### SMTP portok

A rendelkezésre álló SMTP portok száma négy, és mindegyikük más típusú titkosítást biztosít az e-mail küldéshez.

- **25** – Ez a port az üzenetek nyílt szövegben történő küldését szolgálja, bár ha a levelezőszerver támogatja, TLS-sel titkosítható. Ezért sok internetszolgáltató blokkolja, mivel biztonsági kockázatot jelent.
- **2525** – Ez a 25-ös SMTP port alternatívája, és TLS-en keresztül titkosítható.
- **587** – Ez az a port, amelyet az IANA a biztonságos SMTP portként regisztrált, és explicit TLS kapcsolatot igényel. Ha azonban az e-mail szerver nem támogatja a TLS-t, az üzenet nyílt szövegben kerül elküldésre.
- **465** – Ez a port implicit SSL kapcsolaton keresztül működik, és ha a szerver nem támogatja, a művelet megszakad.

### POP3 portok

- A **110-es port** az alapértelmezett POP3 port, és nem titkosított.
- A POP3 titkosított portja a **995-ös**, amely TLS/SSL felett működik.

### IMAP portok

Alapértelmezés szerint az IMAP két porton működik, akárcsak a POP3:

- **143** – ez az alapértelmezett port, amely nem biztosít titkosítást.
- A **993-as port** az IMAP biztonságos portja, amely TLS/SSL titkosításon keresztül működik.

## Mi a különbség az SMTP, a POP3 és az IMAP között?

### Bejövő vs. kimenő protokollok

A POP3 és az IMAP a bejövő e-maileket kezeli, és különböző módon működnek az e-mail üzeneteid lekéréséhez vagy eléréséhez. Így ezeket levelezés-elérési protokolloknak tekintik.

Másrészt a Simple Mail Transfer Protocol áll az üzenetek szerverről szerverre, vagy levelezőkliensről szerverre történő továbbítása mögött. Mivel ez a protokoll kezeli az e-mail küldését egy e-mail fiókból, kimenő protokollnak van címkézve.

Röviden, az IMAP-nak és a POP3-nak köszönhetően képes vagy fogadni e-maileket, az SMTP pedig lehetővé teszi az üzenetek küldését.

### IMAP vs. POP3

Ahogy már említettük, mindkét protokoll az e-mail lekéréséhez kapcsolódik. Minden modern szerver támogatja mindkét protokollt, bár különböző módon működnek.

Míg a POP3 protokoll feltételezi, hogy az e-mailjeidet csak egy alkalmazásból éred el, az IMAP lehetővé teszi több kliens egyidejű hozzáférését. Ezért az IMAP alkalmasabb számodra, ha különböző helyekről szeretnéd elérni az e-mailjeidet, vagy ha több felhasználó kezeli az üzeneteidet.

Másrészt a POP3 letölti az e-mailjeidet a helyi számítógépedre, törölve azokat a szerverről. Így csökkenti az e-mail fiókod által a webszervereden használt helyet.

Összefoglalva, három e-mail protokoll létezik – SMTP, POP3 és IMAP. Mindegyikük meghatározott portszámokon működik, és különböző módon működik. Ha problémád van egy bejövő vagy kimenő szerverhez való csatlakozással, próbálj meg alternatív portszámot használni.
