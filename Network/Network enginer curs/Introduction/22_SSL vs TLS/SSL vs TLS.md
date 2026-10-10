# Mi a különbség az SSL és a TLS között?

A **Secure Sockets Layer (SSL)** egy kommunikációs protokoll, azaz szabályrendszer, amely biztonságos kapcsolatot hoz létre két eszköz vagy alkalmazás között egy hálózaton. Fontos, hogy bizalmat építsünk ki és hitelesítsük a másik felet, mielőtt hitelesítő adatokat vagy adatokat osztanánk meg az interneten. Az SSL olyan technológia, amelyet az alkalmazásaid vagy böngészőid használhattak biztonságos, titkosított kommunikációs csatorna létrehozására bármely hálózaton. Az SSL azonban egy régebbi technológia, amely bizonyos biztonsági hiányosságokat tartalmaz. A **Transport Layer Security (TLS)** az SSL továbbfejlesztett verziója, amely kijavítja a meglévő SSL sebezhetőségeket. A TLS hatékonyabban hitelesít, és továbbra is támogatja a titkosított kommunikációs csatornákat.

*Olvasson az SSL/TLS tanúsítványokról »*

## Mik a hasonlóságok az SSL és a TLS között?

Mind az SSL, mind a TLS olyan kommunikációs protokollok, amelyek titkosítják az adatokat szerverek, alkalmazások, felhasználók és rendszerek között. Hitelesítik a hálózaton keresztül kapcsolódó két felet, hogy biztonságosan cserélhessenek adatokat.

Taher Elgamal vezette az SSL fejlesztését, és 1995-ben nyilvánosan kiadta az SSL 2.0-t. Az SSL célja az volt, hogy biztonságos kommunikációt tartson fenn a World Wide Weben. Miután az SSL több iteráción ment keresztül, Tim Dierks és Christopher Allen 1999-ben létrehozta a TLS 1.0-t az SSL 3.0 utódjaként.

### Terminológia

A TLS az SSL közvetlen utódja, és az SSL minden verziója mára elavult. Azonban gyakori, hogy az „SSL" kifejezést egy TLS kapcsolat leírására használják. A legtöbb esetben az SSL és az SSL/TLS kifejezések egyaránt a TLS protokollra és a TLS tanúsítványokra utalnak.

### Cél

A TLS egy biztonságos kommunikációs protokoll, amely lehetővé teszi a titkosítást és a hitelesítést, és ez igaz volt az SSL-re is, mielőtt elavulttá vált. A TLS és az SSL egyaránt digitális tanúsítványokat használnak, amelyek megkönnyítik a kézfogási folyamatot és titkosított kommunikációt hoznak létre egy böngésző és egy webszerver között.

### Használat a HTTPS-ben

A HTTP egy protokoll vagy kommunikációs szabályrendszer a kliens-szerver kommunikációhoz bármely hálózaton. A HTTPS egy biztonságos SSL/TLS protokoll létrehozásának gyakorlata egy nem biztonságos HTTP kapcsolaton.

Mielőtt csatlakozna egy weboldalhoz, a böngésződ TLS-t használ a weboldal TLS vagy SSL tanúsítványának ellenőrzéséhez. A TLS és SSL tanúsítványok azt mutatják, hogy egy szerver betartja a jelenlegi biztonsági szabványokat. A tanúsítványra vonatkozó bizonyítékot a böngésző címsorában találhatod. Egy hiteles és titkosított kapcsolat `https://`-t jelenít meg `http://` helyett. A további „s" a secure (biztonságos) szót jelenti.

*Olvasson a HTTP-ről és a HTTPS-ről »*

## Fő különbségek: SSL vs. TLS

Bár az SSL és a TLS céljai nagyon hasonlóak, ezek a kommunikációs protokollok abban különböznek, hogyan működnek. Ezek a változások az idő múlásával fejlődtek ki, ahogy az SSL különböző verziókon ment keresztül, mielőtt a TLS követte.

### SSL/TLS kézfogások

A kézfogás egy olyan folyamat, amelyben a böngésző hitelesíti a szerver SSL vagy TLS tanúsítványát. Ez a folyamat mindkét felet hitelesíti, majd kriptográfiai kulcsokat cserél.

Az SSL kézfogás egy explicit kapcsolat volt, míg a TLS kézfogás egy implicit. Az SSL kézfogási folyamat több lépést tartalmazott, mint a TLS folyamat. A további lépések eltávolításával és a titkosítási csomagok teljes számának csökkentésével a TLS felgyorsította a folyamatot.

### Riasztási üzenetek

A riasztási üzenetek az SSL és a TLS protokollok hibák és figyelmeztetések kommunikálásának módja. Az SSL-ben csak két riasztási üzenettípus létezik: figyelmeztetés és végzetes. A figyelmeztető riasztás azt jelzi, hogy hiba történt, de a kapcsolat folytatódhat. A végzetes riasztás azt jelzi, hogy a kapcsolatot azonnal meg kell szüntetni. Emellett az SSL riasztási üzenetek titkosítatlanok.

A TLS-nek van egy további riasztási üzenettípusa, amelyet close notify-nak (bezárási értesítés) neveznek. A close notify riasztás a munkamenet végét jelzi. A TLS riasztások a további biztonság érdekében titkosítottak is.

### Üzenethitelesítés

Mind az SSL, mind a TLS üzenethitelesítési kódokat (MAC) használ, amely egy kriptográfiai technika az üzenetek hitelességének és integritásának ellenőrzésére. Egy titkos kulcs használatával a rekordprotokoll egy rögzített hosszúságú kódként állítja elő a MAC-ot, és csatolja azt az eredeti üzenethez.

Az SSL protokoll az MD5 algoritmust használja – amely mára elavult – a MAC generálásához. A TLS a Hash-Based Message Authentication Code (HMAC) technológiát használja a bonyolultabb kriptográfiához és biztonsághoz.

### Titkosítási csomagok

A titkosítási csomag (cipher suite) olyan algoritmusok gyűjteménye, amelyek kulcsokat hoznak létre az információk titkosításához egy böngésző és egy szerver között. Általában egy titkosítási csomag tartalmaz egy kulcscsere-algoritmust, egy érvényesítési algoritmust, egy tömeges titkosítási algoritmust és egy MAC algoritmust. A TLS-ben számos algoritmust fejlesztettek tovább az SSL-hez képest biztonsági aggályok miatt.

## Mi a különbség az SSL tanúsítványok és a TLS tanúsítványok között?

Jelenleg az összes SSL tanúsítvány már nincs használatban. A TLS tanúsítványok az iparági szabvány. Az iparág azonban továbbra is az „SSL" kifejezést használja a TLS tanúsítványokra való hivatkozáshoz.

A TLS tanúsítványok az SSL tanúsítványokra épültek, és az idő múlásával javították azokat. Az SSL tanúsítványok és a TLS tanúsítványok végső funkciója nem változott.

## Lecseréljük az SSL tanúsítványokat TLS tanúsítványokra?

A lassú kulturális változás miatt a legtöbb TLS tanúsítvány helytelenül SSL tanúsítványként van elnevezve. Még ha a tanúsítványod SSL tanúsítványként is hirdeti magát, már mind az SSL, mind a TLS protokollokat támogatja.

Fontos azonban megjegyezni, hogy a TLS 1.0 és a TLS 1.1 2021-ben szintén hivatalosan elavulttá vált. 2023 júniusáig minden Amazon Web Services ügyfélnek támogatnia kell a TLS 1.2-t vagy újabbat. Ne feledje, hogy a tanúsítványok nem ugyanazok, mint a protokoll. Győződjön meg arról, hogy a szerverkonfigurációja támogatja a TLS protokollokat.

## A különbségek összefoglalása: SSL vs. TLS

| | SSL | TLS |
|---|---|---|
| **Mit jelent?** | SSL = Secure Sockets Layer | TLS = Transport Layer Security |
| **Verziótörténet** | Az SSL-t mára a TLS váltotta fel. Az SSL az 1.0, 2.0 és 3.0 verziókon ment keresztül. | A TLS az SSL továbbfejlesztett verziója. A TLS az 1.0, 1.1, 1.2 és 1.3 verziókon ment keresztül. |
| **Aktivitás** | Minden SSL verzió mára elavult. | A TLS 1.2 és 1.3 verziók aktívan használatban vannak. |
| **Riasztási üzenetek** | Az SSL-nek csak két típusú riasztási üzenete van. A riasztási üzenetek titkosítatlanok. | A TLS riasztási üzenetek titkosítottak és változatosabbak. |
| **Üzenethitelesítés** | Az SSL MAC-okat használ. | A TLS HMAC-okat használ. |
| **Titkosítási csomagok** | Az SSL régebbi, ismert biztonsági sebezhetőségekkel rendelkező algoritmusokat támogat. | A TLS fejlett titkosítási algoritmusokat használ. |
| **Kézfogás** | Az SSL kézfogás összetett és lassú. | A TLS kézfogás kevesebb lépést és gyorsabb kapcsolatot tartalmaz. |
