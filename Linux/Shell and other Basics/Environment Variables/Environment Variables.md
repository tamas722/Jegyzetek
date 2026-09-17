# Környezeti Változók (Environment Variables) Beállítása Linuxban

A Linuxban a változókat a parancssoros eszközök konfigurálására, nyelvi beállításokra vagy programok elérési útjának (PATH) megadására használjuk.

## Elmélet: Shell változók vs. Környezeti változók

- **Shell változó:** Csak az aktuális shell (pl. Bash) munkamenetben létezik. A belőle indított gyermekfolyamatok (child processes) nem öröklik meg.

- **Környezeti változó:** Olyan változó, amely exportálva lett, így a jelenlegi folyamatból indított összes többi program és gyermekfolyamat is megörökli a másolatát.

**Szintaktikai szabályok:**

- Formátum: `KEY=value` (Több érték esetén: `KEY=value1:value2`).
- Nagybetűs konvenció: Az exportált környezeti változók neveit hagyományosan csupa nagybetűvel írjuk.
- Nincs szóköz az egyenlőségjel (`=`) körül! (A `KEY = value` szintaktikai hiba).
- Ha az érték szóközt tartalmaz, idézőjelbe kell tenni: `KEY="Some other value"`.

## 1. Változók listázása és lekérdezése

- Minden környezeti változó listázása:

  ```bash
  printenv
  # vagy
  env
  ```

- Egy (vagy több) konkrét környezeti változó értékének lekérdezése:

  ```bash
  printenv HOME
  printenv LANG PWD
  ```

- Shell változók kiíratása:

  A `printenv` csak az exportált változókat látja. Shell változó kiíratásához az `echo` vagy a `printf` használható:

  ```bash
  echo "$BASH_VERSION"
  ```

- Minden változó (shell és környezeti) és függvény listázása:

  ```bash
  set
  # Mivel a kimenet nagyon hosszú lehet, érdemes lapozóba csövezni:
  set | less
  ```

**Gyakori környezeti változók:**

- `USER`: Az aktuális felhasználó neve.
- `HOME`: A felhasználó saját (home) könyvtára.
- `EDITOR`: Az alapértelmezett parancssoros szövegszerkesztő.
- `SHELL`: A bejelentkezési shell útvonala.
- `LOGNAME`: A bejelentkezési név.
- `PATH`: A futtatható fájlok keresési útvonala (balról jobbra haladva).
- `LANG`: Az alapértelmezett nyelvi/területi (locale) beállítás.
- `TERM`: A terminál típusa.
- `MAIL`: A felhasználó levelezési fájljának helye.
- `XDG_SESSION_TYPE`: A munkamenet típusa (pl. tty, x11, wayland).

## 2. Változók beállítása (Bash)

**Shell változó létrehozása:**

```bash
MY_VAR="Linuxize"
```

(Ellenőrzés: A `printf '%s\n' "$MY_VAR"` kiírja, de a `printenv MY_VAR` nem ad vissza semmit, mert még nem környezeti változó).

**Változó exportálása (környezetivé alakítása):**

```bash
export MY_VAR
```

(Ezután a `printenv MY_VAR` már működik, és a gyermekfolyamatok is látják, pl.: `bash -c 'printenv MY_VAR'`)

**Létrehozás és exportálás egyetlen paranccsal:**

```bash
export MY_NEW_VAR="My New Var"
```

**Változó beállítása egyetlen parancs erejéig:**

Ha az értékadást közvetlenül a parancs elé teszed, az csak a parancs lefutásának idejére lesz érvényes:

```bash
DEBUG=1 printenv DEBUG
```

## 3. Változók törlése (Unset) és Ellenőrzése

- Változó eltávolítása (mind shell, mind környezeti):

  ```bash
  unset MY_VAR
  ```

- Változó létezésének ellenőrzése (Bash):

  A `-v` feltétellel megkülönböztethető a nem létező (unset) változó az üres stringet tartalmazó változótól:

  ```bash
  if [[ -v MY_VAR ]]; then printf 'MY_VAR is set\n'; else printf 'MY_VAR is unset\n'; fi
  ```

## 4. A PATH változó kezelése

A `PATH` könyvtárakat tartalmaz kettősponttal (`:`) elválasztva.

**Jelenlegi PATH kiíratása:**

```bash
printf '%s\n' "$PATH"
```

- Könyvtár hozzáadása a PATH elejéhez (így ez kap prioritást):

  ```bash
  export PATH="$HOME/bin:$PATH"
  ```

- Könyvtár hozzáadása a PATH végéhez:

  ```bash
  export PATH="$PATH:/opt/myapp/bin"
  ```

## 5. Változók véglegesítése (Perzisztencia)

A terminálban kiadott `export` parancs csak az aktuális munkamenetben érvényes. A véglegesítéshez konfigurációs fájlokat kell szerkeszteni.

### A) Felhasználói szintű beállítások (Bash és Zsh)

Az interaktív terminálok számára a `~/.bashrc` (Zsh esetén `~/.zshrc`) fájlba kell tenni az értékadást.

Példa a `~/.bashrc` tartalmára:

```bash
export EDITOR="nano"
export PATH="$HOME/bin:$PATH"
```

Módosítás után az aktuális shell frissítése:

```bash
source ~/.bashrc
```

> **Megjegyzés:** A login shellek a `~/.bash_profile`, `~/.bash_login`, vagy `~/.profile` fájlokat olvassák be, Zsh esetén a `~/.zprofile`-t.

### B) Rendszerszintű beállítások (Minden felhasználónak)

A PAM (Pluggable Authentication Modules) általában az `/etc/environment` fájlt olvassa be. Ide NEM szabad `export` parancsot írni, csak egyszerű értékadást:

Példa az `/etc/environment` tartalmára:

```plaintext
JAVA_HOME=/usr/lib/jvm/default-java
```

Ha shell szintaxist kell használni, akkor az `/etc/profile.d/` könyvtárba érdemes egy `.sh` scriptet tenni (pl. `/etc/profile.d/java.sh`), amibe már mehet az `export`:

```bash
export JAVA_HOME="/usr/lib/jvm/default-java"
export PATH="$JAVA_HOME/bin:$PATH"
```

### C) Systemd felhasználói szolgáltatások (User Services)

A systemd user manager által indított szolgáltatásokhoz a `~/.config/environment.d/` mappában kell `.conf` kiterjesztésű fájlt létrehozni (szintén `export` szó nélkül!).

Példa a `~/.config/environment.d/60-myapp.conf` tartalmára:

```plaintext
MYAPP_MODE=production
```

Módosítás után a systemd konfiguráció újratöltése:

```bash
systemctl --user daemon-reload
```

## Gyakori kérdések és Hibaelhárítás (FAQ / Troubleshooting)

- **A program nem látja a változót:** Valószínűleg csak shell változót hoztál létre (`VAR=value`). Használd az `export VAR=value` formátumot, mielőtt elindítod a programot.

- **A GUI alkalmazás vagy systemd service nem látja:** A `~/.bashrc` csak a shell által indított folyamatokra hat. Systemd esetén használd az `Environment=` opciót a unit fájlban, vagy az `environment.d` könyvtárat.

- **Linux betölti automatikusan a `.env` fájlokat?** Nem. A `.env` fájlokat az alkalmazások (frameworkök, betöltő scriptek) olvassák be, ez nem Linux szintű szabvány.
##Forrás
-[https://linuxize.com/post/how-to-set-and-list-environment-variables-in-linux/]

# Linux Környezeti Változók: Listázás, Beállítás, Létrehozás és Törlés

A környezeti változók bővítik és egységesítik a shell környezetet a Linux rendszereken.

- **Standard környezeti változók:** A rendszer által beállított alapértelmezett változók (általában csupa nagybetűvel írva).
- **Egyedi (Custom) környezeti változók:** Amelyeket mi hozunk létre a saját, specifikus igényeinkhez.

## 1. Környezeti változók értékének megtekintése

Az `echo` parancs megjeleníti a megadott szöveget. A `$` jel mondja meg a shellnek, hogy a szöveget változóként értelmezze. Ha ezt elhagyjuk, egyszerű szövegként írja ki.

Példa a `SHLVL` (Shell Level) változóval:

`$` jel nélkül (sima szövegként értelmezi):

```bash
$ echo SHLVL
SHLVL
```

`$` jellel (változó értékének lekérdezése):

```bash
$ echo $SHLVL
1
```

A `SHLVL` a megnyitott al-shelleket (subshells) számolja. Az alapértelmezett érték 1. Ha megnyitunk 3 új shellt a `bash` paranccsal, az érték 4 lesz:

```bash
$ bash
$ bash
$ bash
$ echo $SHLVL
4
```

**Összes környezeti változó listázása:**

Ha nem tudod egy változó pontos nevét, az összes környezeti változó listázható az `env` vagy a `printenv` paranccsal. A bal oldalon a változó neve, a jobb oldalon az értéke jelenik meg.

```bash
$ env
```

vagy

```bash
$ printenv
```

## 2. Környezeti változók létrehozása / frissítése (Ideiglenesen)

Az aktuális munkamenetben (session) létrehozhatsz új változókat, egyszerűen a nevük és értékük megadásával:

```bash
$ XYZ=test
$ echo $XYZ
test
```

Ugyanezzel a módszerrel frissíthetsz vagy fűzhetsz hozzá meglévő változókhoz. Például a `PATH` változó végéhez az `/opt/bin` mappa hozzáadása (`:` a kód elválasztó):

```bash
$ echo $PATH
$ PATH=$PATH:/opt/bin
```

> **Figyelem:** Ez a változás ideiglenes, kijelentkezéskor (logout) a shell törli.

## 3. Környezeti változók létrehozása / frissítése (Véglegesen, egy felhasználónak)

A felhasználó saját (home) könyvtárában található a `~/.bash_profile` szkriptfájl. A shell minden bejelentkezéskor lefuttatja ezt. A saját változókat a fájl végéhez kell hozzáadni, és az `export` paranccsal exportálni kell őket.

Szerkeszd a `~/.bash_profile` fájlt, és add hozzá az alábbiakat a végéhez:

```bash
HISTSIZE=50
PATH=$PATH:/opt/bin
export HISTSIZE PATH
```

Mentés és kijelentkezés után a következő bejelentkezéskor ezek a változók már élni fognak.

## 4. Környezeti változók létrehozása (Minden felhasználónak / Rendszerszinten)

A Linux alapértelmezett globális profilfájlja az `/etc/profile`. Azonban ezt **nem szabad szerkeszteni**, mert egy rendszerfrissítés felülírhatja!

Helyette a rendszergazdák az `/etc/profile.d/` könyvtárban hoznak létre egyedi szkripteket (pl. `custom.sh`). A shell az itt beállított változókat részesíti előnyben az `/etc/profile` alapértelmezéseivel szemben, és a frissítések sem írják felül őket.

**Lépések (root felhasználóként):**

1. Hozz létre egy fájlt, pl. `custom.sh` néven az `/etc/profile.d/` könyvtárban.
2. Add hozzá a kívánt változót és exportáld.
3. Kijelentkezés után minden felhasználó megörökli az új beállítást.

## 5. Alapértelmezések megváltoztatása az ÚJ felhasználói fiókokhoz

Az `/etc/skel` könyvtár tartalmazza azokat az alapértelmezett profilfájlokat, amiket a rendszer minden új felhasználó létrehozásakor átmásol annak saját mappájába.

Ha módosítod az `/etc/skel/.bash_profile` fájlt (pl. rootként), az az újonnan létrehozott fiókokra fog vonatkozni, de a már meglévő fiókok beállításait nem érinti.

## Standard Alapértelmezett Környezeti Változók Listája

| Név | Leírás / Alapértelmezett Érték |
| --- | --- |
| `BASH` | A BASH parancs teljes útvonala (pathname) |
| `BASH_VERSION` | Aktuális BASH verziószám |
| `DISPLAY` | Az X Window rendszer által használt kijelzőszerver azonosítója |
| `DOMAIN` | Tartománynév (Domain name) |
| `EDITOR` | Alapértelmezett parancssori szövegszerkesztő |
| `EUID` | Effektív felhasználói azonosító (numerikus) |
| `GROUPS` | A felhasználó csoportjai |
| `HISTCMD` | Az aktuális parancs sorszáma az előzményekben (history) |
| `HISTFILE` | Az előzményfájl (history file) útvonala |
| `HISTFILESIZE` | Az előzményfájl maximális mérete sorokban |
| `HISTSIZE` | Az előzményekben (memóriában) tárolható parancsok száma |
| `HOME` | A felhasználó saját (home) könyvtárának útvonala |
| `HOSTFILE` | A hosts fájl neve (ha eltér az `/etc/hosts`-tól) |
| `HOSTNAME` | Gépnév (Hostname) |
| `HOSTTYPE` | Linux platform architektúrája (pl. i686, x86_64) |
| `INPUTRC` | A beviteli eszközök (pl. billentyűzet) definíciós fájljának helye |
| `LANG` | Preferált nyelv és lokalizáció |
| `LD_LIBRARY_PATH` | Megosztott könyvtárak (libraries) keresési útvonalai |
| `LOGNAME` | Bejelentkezési név |
| `MAIL` | A specifikus levélfájl neve (ha a `MAILPATH` nincs beállítva) |
| `MAILCHECK` | A beérkező levelek ellenőrzési intervalluma |
| `MAILPATH` | A bejövő üzenetek miatt ellenőrizendő levélfájlok listája |
| `MANPATH` | A kézikönyvek (man oldalak) keresési útvonalai |
| `OLDPWD` | Előző (korábbi) munkakönyvtár |
| `OS` | Az operációs rendszert leíró karakterlánc |
| `OSTYPE` | További információ az operációs rendszer verziójáról |
| `PAGER` | Akció (lapozó program), ha a kimenet hosszabb egy terminálablaknál |
| `PATH` | A parancsok futtatható fájljait tartalmazó könyvtárak keresési listája |
| `PPID` | A szülő shell folyamatazonosítója (Parent Process ID) |
| `PROMPT_COMMAND` | Parancs, amely minden prompt előtt lefut |
| `PS1` | Elsődleges shell prompt (megjelenés) |
| `PS2` | Másodlagos shell prompt (megjelenés, több soros parancsokhoz) |
| `PWD` | Jelenlegi munkakönyvtár (Present Working Directory) |
| `RANDOM` | Egy véletlenszerű szám generálása |
| `SHELL` | A jelenleg használt shell (parancsértelmező) neve és útvonala |
| `SHLVL` | Aktuális shell szint (az al-shelleket számolja) |
| `TERM` | Terminál típusa |
| `TMOUT` | Az időkorlát, ameddig a shell aktív marad bemenetre várva |
| `UID` | Felhasználói azonosító (User ID) |
| `USER` | Felhasználónév |
| `VISUAL` | Alapértelmezett teljes képernyős szerkesztő |
| `XFILESEARCHPATH` | Grafikus könyvtárak keresési útvonalai |

## Összegzés

- Az `/etc/profile` határozza meg a globális alapértelmezéseket (**ne szerkeszd**).
- Az `/etc/profile.d/*` fájlok felülírják a globális alapértelmezéseket (rendszerszintű testreszabás itt történik).
- A `~/.bash_profile` pedig csak egy adott felhasználó alapértelmezéseit írja felül (egyéni testreszabás).
##Forrás
-[https://www.computernetworkingnotes.com/linux-tutorials/linux-environment-variables-list-set-create-remove.html]
