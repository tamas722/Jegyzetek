# SUPER + Escape
Grafikus kikapcsoló, újraindító és altató menü (wlogout) megnyitása.

# Print Screen (PrtScn)
Teljes képernyő azonnali lefotózása, a fájl mentése a Képek mappába dátumozva (grim).

# SUPER + SHIFT + S
Egyedi terület kijelölése és lefotózása, ami egyenesen a vágólapra kerül, így azonnal beilleszthető bárhová (grim + slurp).

# Fn + Fényerő / Hangerő
Hardveres gombok a laptopon a háttérvilágítás és a hangvezérlés módosítására (a rendszer alapból kezeli).

## Alapvető alkalmazások és ablakkezelés

- **SUPER + Q**: Alapértelmezett terminál (kitty) megnyitása.
- **SUPER + C**: Az aktív ablak azonnali bezárása (kill).
- **SUPER + R**: Programindító menü (wofi) megnyitása a képernyő közepén.
- **SUPER + M**: Azonnali kilépés a Hyprlandből (kijelentkezés).
- **SUPER + V**: Az aktuális ablak lebegő (floating) módba tétele, vagy visszarakása a rácsba (tiling).

## Munkaterületek (Workspaces) kezelése

- **SUPER + 1, 2, 3 ... 0**: Azonnali átváltás a megadott számú munkaterületre. (Ha még nem létezik, létrehozza).
- **SUPER + SHIFT + 1, 2, 3 ... 0**: Az aktív (épp használt) ablak átdobása a megadott számú munkaterületre.
- **SUPER + Egér görgő**: Gyors lapozás a nyitott munkaterületek között (ugyanez működik, ha az egeret a felső sáv fölé húzod és ott görgetsz).

## Navigáció és ablakok fizikai mozgatása

- **SUPER + Nyilak (Balra, Jobbra, Fel, Le)**: A fókusz (aktív állapot) átadása a mellette lévő ablaknak.
- **SUPER + Bal egérgomb (nyomva tartva)**: Az ablak "megfogása" és mozgatása a képernyőn, vagy a rács átrendezése.
- **SUPER + Jobb egérgomb (nyomva tartva)**: Az ablak méretének megváltoztatása az egér húzásával.

## Böngésző (Firefox) biztonsági gombok

- **Ctrl + Q**: A böngésző szabályos és biztonságos leállítása (garantálja, hogy a lapok megmaradjanak).
- **Ctrl + Shift + T**: A legutóbb (akár véletlenül) bezárt fül vagy teljes ablak visszahozása.

  # SUPER + F
Igazi teljes képernyő (True Fullscreen). Az ablak a képernyő legutolsó pixelét is elfoglalja, még a fenti Waybar is eltűnik (filmnézéshez vagy kódoláshoz ideális).
- Beállítása a configban: `bind = SUPER, F, fullscreen, 0`

# SUPER + M
Maximalizálás. Az ablak kitölti a rendelkezésre álló helyet, de a fenti Waybar látható és használható marad.
- Beállítása a configban: `bind = SUPER, M, fullscreen, 1`

# SUPER + V
Lebegő (floating) mód be/kikapcsolása. Ha egy ablakot teljes képernyőre raktál, ezzel (vagy a SUPER + F újra-megnyomásával) tudod visszadobni az eredeti méretére.

## Firefox - Lapok (Tabok) és Ablakok kezelése

- **Ctrl + T**: Új lap megnyitása az aktuális ablakban.
- **Ctrl + W**: Csak az éppen nézett lap bezárása (a böngésző maga nyitva marad, ha van még mellette másik lap).
- **Ctrl + N**: Egy teljesen új, különálló Firefox ablak megnyitása.
- **Ctrl + Shift + T**: A véletlenül bezárt lap (vagy teljes ablak) visszahozása.
- **Ctrl + Q**: A teljes böngésző biztonságos bezárása az összes lappal és ablakkal együtt.

## Firefox - Gyors navigáció egér nélkül

- **Ctrl + Tab**: Lépkedés a nyitott lapok között előre (jobbra).
- **Ctrl + Shift + Tab**: Lépkedés a nyitott lapok között visszafelé (balra).
- **Alt + 1, 2, 3 ... 8**: Azonnali ugrás a fenti sávban lévő sorszámnak megfelelő lapra (pl. Alt + 2 a második lapra ugrik).
- **Alt + 9**: Ugrás a legutolsó (legjobb oldali) lapra, függetlenül attól, hány darab van nyitva.
- **Ctrl + L**: A fenti URL (kereső) sáv azonnali kijelölése. Ha új weboldalt akarsz megnyitni vagy keresni valamit a Google-ben, nem kell az egérrel odavinned a kurzort, csak megnyomod ezt, és egyből gépelhetsz.
- **F5 vagy Ctrl + R**: Az aktuális oldal frissítése.
