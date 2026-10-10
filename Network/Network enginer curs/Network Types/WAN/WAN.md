# Mi az a wide area network (WAN)?

A **wide area network (WAN)** egy nagy kiterjedésű számítógépes hálózat, amely számítógépcsoportokat köt össze nagy távolságokon keresztül. A WAN-okat gyakran használják nagyvállalatok az irodai hálózataik összekapcsolására; minden irodának általában saját helyi hálózata (LAN) van, és ezek a LAN-ok WAN-on keresztül kapcsolódnak össze. Ezek a hosszú kapcsolatok többféle módon jöhetnek létre, beleértve a bérelt vonalakat, VPN-eket vagy IP-alagutakat (lásd alább).

Annak meghatározása, hogy mi minősül WAN-nak, meglehetősen tág. Technikailag minden olyan nagy hálózat, amely széles földrajzi területen terjed ki, WAN. Maga az internet is WAN-nak számít.

![Wide area network WAN - Több LAN összekapcsolva]

## Mi az a LAN?

A **local area network (LAN)** egy kis, lokalizált területre korlátozódó hálózat. Az otthoni WiFi hálózatok és a kisvállalkozási hálózatok a LAN-ok gyakori példái. Általában aki a LAN-t kezeli, az kezeli az általa használt hálózati berendezéseket is. Egy kisvállalkozás például kezeli a LAN beállításában részt vevő routereket és switcheket.

## WAN vs. LAN

A LAN-ok általában egy zárt területen léteznek, és általában egyetlen központi internetkapcsolódási pontot osztanak meg. A WAN-okat hosszú távolságokon keresztüli hálózati kapcsolat biztosítására tervezték. Általában több összekapcsolt LAN-ból állnak. Egy szervezet, amely saját WAN-t hoz létre, szinte mindig az ellenőrzésén kívül eső hálózati infrastruktúrára támaszkodik: például egy Párizsban és New Yorkban irodával rendelkező vállalatnak az adatokat ezen irodák között az Atlanti-óceánt átszelő tenger alatti kábeleken keresztül kell küldenie.

Általában egy WAN több routert és switch-et tartalmaz. Egy LAN-nak csak egy routerre van szüksége az internethez vagy más LAN-okhoz való csatlakozáshoz, bár switcheket is használhat.

## Mi az a bérelt vonal?

Az egyik módja annak, hogy a szervezetek összekapcsolják LAN-jaikat WAN kialakításához, az úgynevezett **bérelt vonal** (leased line) használata. A bérelt vonal egy közvetlen hálózati kapcsolat, amelyet egy nagy hálózati szolgáltatótól, például egy ISP-től bérelnek. Saját fizikai hálózati infrastruktúra építése – beleértve a kábeleket, routereket és internetes cserélőpontokat több száz vagy ezer mérföldön keresztül – szinte lehetetlen feladat lenne a legtöbb szervezet számára. Ezért inkább egy közvetlen, dedikált kapcsolatot bérelnek egy olyan vállalattól, amely már rendelkezik ezzel az infrastruktúrával.

## Mi az alagutazás? Mi a VPN?

Ha egy vállalat nem akar fizetni bérelt vonalért, összekapcsolhatja LAN-jait **alagutazással** (tunneling). A hálózatkezelésben az alagutazás egy módszer arra, hogy adatcsomagokat* más adatcsomagokba ágyazzunk, hogy olyan helyre jussanak, ahová egyébként nem jutnának el. Képzeld el, hogy egy borítékot egy másik borítékba postázol, mindkét borítéknak más címe van, így a belső boríték a külső boríték célcíméről kerül postázásra. Ez az alagutazás általános ötlete, azzal a különbséggel, hogy az adatok borítékok helyett csomagokban vannak.

Egyes hálózati alagutak titkosítottak, hogy megvédjék a csomagok tartalmát mindenkitől, aki elfoghatja azokat útközben. A titkosított alagutakat **VPN-eknek** vagy virtuális magánhálózatoknak nevezik. A WAN-ok közötti VPN-kapcsolatok biztonságosabbak, mint a titkosítatlan alagútkapcsolatok. Az IPsec egy gyakori VPN-titkosítási protokoll.

Az alagutazás LAN-ok összekapcsolására való használatának fő hátránya, hogy az alagutazás növeli az overheadet; több számítási teljesítményt, és így több időt igényel az adatok ilyen módon történő küldése. Az egyes csomagok beágyazása és titkosítása lelassítja a kommunikációt, ahogy a boríték kétszeri megtöltése egyszeri helyett lelassítja annak postázását. Emellett a beágyazott csomagok nagyobbak lehetnek, mint amit a hálózat egyes routerei kezelni tudnak, ami fragmentációt és további késleltetéseket okoz.

*\*Minden hálózaton keresztül küldött adat csomagokra bomlik, amelyek kisebb adatdarabok. Minden csomag tartalmaz információt a csomag eredetéről, célállomásáról és a csomagsorozatban elfoglalt helyéről.*

## Mi az a software-defined WAN (SD-WAN)?

A **software-defined WAN**, vagy **SD-WAN**, egy rugalmasabb WAN-architektúra, amely több hardverplatform és kapcsolódási lehetőség előnyeit tudja kihasználni. A vezérlő szoftver bármilyen hálózati hardverrel működik.

Az SD-WAN-ok a **software-defined networking (SDN)** egyik formája, amely olyan technológiák kategóriája, amelyek lehetővé teszik a hálózatok szoftverrel való kezelését. Ezek a **secure access service edge (SASE)** megoldások kulcsfontosságú összetevői is, amelyek a hálózatkezelési és hálózati biztonsági funkciókat egyetlen, felhőalapú szolgáltatásba kombinálják.

## Mi a WAN-as-a-service?

A **WAN-as-a-service** egy felhőalapú WAN-modell. A WAN-as-a-service kínálatokat úgy tervezték, hogy lecseréljék a hardverre támaszkodó, olyan kapcsolódási protokollokat használó, mint a multiprotocol label switching (MPLS), és nehezen skálázható örökölt WAN-konfigurációkat. Mivel a WAN-as-a-service a felhőn keresztül érhető el, az ügyfeleknek csak internetkapcsolatra van szükségük, és szoftverrel konfigurálhatják WAN-jukat hardveres berendezések helyett. Tudj meg többet a Cloudflare WAN-ról; lásd még: Mi az a network-as-a-service (NaaS)?
