# Mi az a kliens?

A házi és vállalati hálózatokban a **kliens** bármely olyan számítógépes hardver vagy szoftver eszköz, amely egy szerver által nyújtott szolgáltatáshoz kér hozzáférést. A klienseket általában a kérést küldő programként vagy felhasználóként tartják számon a kliens-szerver architektúrában. A kliens végfelhasználói eszközök jellemzően asztali számítógépeket, laptopokat és okostelefonokat foglalnak magukba.

Egy számítógépes hálózatban a kliens a kliens-szerver modellben az, amely szolgáltatást vagy erőforrást kér a szervertől. A szerver lehet a helyszínen vagy azon kívül.

## Hogyan kommunikál a kliens a szerverrel?

A kliens-szerver architektúrában a kliensek úgy kommunikálnak a szerverekkel, hogy adatokra vagy olyan erőforrásokra vonatkozó kéréseket küldenek, amelyeket a kliens nem képes biztosítani. A kliensek és a szerverek különböző helyeken lehetnek, és hálózaton keresztül kapcsolódhatnak egymáshoz. Ugyanazon a gépen is elhelyezkedhetnek, és folyamatok közötti kommunikációval kapcsolódhatnak.

Az elosztott alkalmazás-keretrendszer a feladatokat a szerverek és a kliensek között osztja fel. A kliens által elvégezhető feladatokat általában **kliensoldalinak** nevezik, míg a szerver által elvégzett feladatokat **szerveroldalinak**.

A szerverek jellemzően olyan erőforrásokat biztosítanak, mint fájlok, adatok, internet-hozzáférés, külső tároló vagy feldolgozási teljesítmény. A szerver egy vagy több programot bíz meg azzal, hogy ezeket a szolgáltatásokat elossza a kérést küldő kliensek számára.

A kliensek és a szerverek egy kérés-válasz üzenetküldési mintában kommunikálnak, amelyet egy kommunikációs protokoll határoz meg, például a Transmission Control Protocol/Internet Protocol (TCP/IP). A protokoll határozza meg a használt nyelvet és párbeszédmintákat. A TCP például fenntartja a kapcsolatot a kliens és a szerver között az üzenetváltás végéig. A TCP azt is meghatározza, hogyan ossza el az alkalmazásadatokat, továbbítja és fogadja a csomagokat, valamint kezeli az esetlegesen elveszett csomagokat. Ha a szerver egyszerre nagy számú kérést kap, a kéréseket egy ütemezési rendszerben prioritás alapján rendezik. A kliens hozzáférés-vezérlő eszközökkel a felhasználó kezelheti a szerverekhez való hozzáférést.

### Kérés és válasz a kliens és a szerver között

Ez a kép egy egyszerűsített kérés- és válaszfolyamatot mutat be a kliens és a szerver között.

## Kliensoldali vs. szerveroldali

A **kliensoldali** egy olyan művelet vagy tevékenység, amely a felhasználó oldalán történik, míg a **szerveroldali** olyan műveletekre vagy tevékenységekre utal, amelyek egy szerveren történnek.

Például a JavaScript szkriptek kliensoldaliak, mivel egy böngésző hajtja végre őket, amely a kliens. A Common Gateway Interface szkriptek szerveroldaliak, mivel egy szerveren futnak.

Egy szerver lehet például egy alkalmazásszerver, egy különálló számítógép, egy adatbázis-szerver vagy egy webszerver. Amikor egy programnak vagy eszköznek nincsenek meg az erőforrásai egy művelet elvégzéséhez, kliensként viselkedik, kérést küld a szervernek, és a művelet szerveroldalon történik meg.

## A kliensek típusai

Az alábbiakban a különböző kliens típusok meghatározásának négy módját vizsgáljuk meg:

- **Vastag kliens (Thick client).** A vastag kliensek biztosítják az erőforrások és az adatfeldolgozás többségét. Ezek a kliensek csak kis mértékben támaszkodnak a szerverre; ehelyett a legtöbb műveletet kliensoldalon végzik.
- **Vékony kliens (Thin client).** Ezek a könnyűsúlyú kliensek nagymértékben támaszkodnak a szerverre az erőforrások és az adatfeldolgozás nagy részének elvégzéséhez.
- **Hibrid kliens (Hybrid client).** A hibrid kliensek a vékony és vastag kliensek kombinációi. Ez a kliens típus helyben dolgozza fel az adatokat, de a tartós adatok tárolásához szerverre támaszkodik.
- **Nulla kliens (Zero client).** A nulla kliensek karcsúbbak és költséghatékonyabbak, mint a vékony kliensek. Ezek a kliens eszközök nem igényelnek konfigurációt, és semmi sem tárolódik rajtuk. Kevesebb energiát is fogyasztanak, mint a vastag vagy vékony kliensek.

### Hogyan különböznek a vékony, vastag és nulla kliensek?

Ez a kép kiemeli a vékony, vastag és nulla kliensek közötti különbségeket.

## Kliens-szerver példák

A kliens-szerver kapcsolatok példái közé tartoznak a web-, fájl- és levelezőszerverek. A webböngésző lényegében egy kliens, amely oldalakat kér a weben található szerverektől. A böngésző az a kliens, amely kéréseket küld, míg egy másik helyen lévő szerver visszaküldi a Hypertext Markup Language (HTML) fájlt. Az a számítógép, amely kezeli a kérést és visszaküldi a HTML fájlt, egy szerver.

A fájlszerverek olyan számítógépek, amelyek felelősek az adatfájlok tárolásáért és kezeléséért, lehetővé téve a felhasználók számára, hogy információt osszanak meg egy hálózaton keresztül anélkül, hogy a fájlokat fizikailag át kellene vinniük. A fájlszerverek példái közé tartozik egy hálózatra csatlakoztatott tárolóeszköz vagy egy különálló számítógép, amely megosztható fájlokat tárol.

A File Transfer Protocol egy kliens-szerver hálózati protokoll, amelyet fájlok továbbítására használnak a számítógépek között TCP/IP kapcsolatokon keresztül. A kliens-szerver protokoll két kommunikációs csatornára támaszkodik a kliens és a szerver között – egy parancscsatornára a beszélgetés vezérléséhez és egy adatcsatornára a fájltartalom továbbításához.
