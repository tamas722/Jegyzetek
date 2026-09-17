# A Linux `cut` Parancs Használata

A `cut` egy parancssoros segédprogram (utility), amellyel egy fájl (vagy egy adatfolyam) soraiból kivághatók bizonyos részek egy adott elválasztó (delimiter), bájtok vagy karakterek alapján, majd a kimenetet a szabványos kimenetre (stdout) írja.

## 1. A `cut` Parancs Szintaxisa és Alapvető Kapcsolói (Flags)

**Alapszintaxis:**

```bash
cut OPTION... [FILE]...
```

Ha nem adunk meg fájlt, vagy a `FILE` helyén `-` szerepel, a `cut` a szabványos bemenetről (stdin) olvas.

A `cut` működéséhez az alábbi három kapcsoló közül **pontosan egyet kötelező** megadni:

- `-f` (`--fields=LIST`): Kivágás mező(k) vagy mezőtartományok alapján. Ez a leggyakrabban használt opció. (Az alapértelmezett elválasztó a TAB).
- `-b` (`--bytes=LIST`): Kivágás bájt(ok) vagy bájttartományok alapján.
- `-c` (`--characters=LIST`): Kivágás karakter(ek) vagy karaktertartományok alapján.

A `LIST` argumentum lehet egyetlen szám (pl. `2`), számok vesszővel elválasztva (`1,3`), vagy egy tartomány (`1-5`).

**Tartományok típusai:**

- `N`: Csak az N-edik elem (1-től indulva).
- `N-`: Az N-edik elemtől a sor végéig.
- `N-M`: Az N-edik elemtől az M-edik elemig.
- `-M`: A sor elejétől az M-edik elemig.

## 2. További Fontos Kapcsolók

- `-d` (`--delimiter`): Egyedi elválasztó karakter megadása a default TAB helyett. Csak egyetlen karakter lehet.
- `--complement`: A kijelölés megfordítása (inverz). Azt írja ki, ami nincs kiválasztva.
- `-s` (`--only-delimited`): Alapértelmezésben a `cut` kiírja azokat a sorokat is, amelyekben egyáltalán nincs elválasztó karakter. Az `-s` kapcsolóval ezeket a sorokat kihagyja.
- `--output-delimiter`: Megváltoztatja a kimeneti elválasztót (alapból megegyezik a bemenetivel).

## 3. Példák és Működés

### Kivágás mezők (Fields) alapján

Tételezzük fel, hogy van egy `test.txt` fájlunk, amiben TAB-bal vannak elválasztva az adatok.

Az 1-es és 3-as mező kivágása:

```bash
cut -f 1,3 test.txt
```

Az 1-estől a 4-es mezőig történő kivágás:

```bash
cut -f -4 test.txt
```

### Egyedi elválasztó (Delimiter) megadása

Ha az adatokat kettőspont (`:`) választja el (pl. az `/etc/passwd` fájlban):

```bash
cut -d ':' -f 1,3 test.txt
```

Szóköz (Space) használata elválasztóként:

```bash
echo "Lorem ipsum dolor sit amet" | cut -d ' ' -f 2
# Kimenet: ipsum
```

### Kijelölés megfordítása (Complement)

Az 1-es és 3-as mező kivételével minden más mező kiírása:

```bash
cut test.txt -f 1,3 --complement
```

### Kimeneti elválasztó (Output Delimiter) megadása

Ha az 1-es és 3-as mezőt akarjuk kiírni, de a kimenetben aláhúzással (`_`) akarjuk őket elválasztani:

```bash
cut -f 1,3 --output-delimiter='_' test.txt
```

### Kivágás Bájtok (Bytes) és Karakterek (Characters) alapján

A UTF-8 karakterkódolás miatt egy karakter több bájtból is állhat (pl. az `ü` betű 2 bájtot foglal).

A `-b` opció bájtokat vág ki (ami eltörheti a több-bájtos karaktereket), míg a `-c` opció kifejezetten karaktereket számol (lokalizációtól függően).

**Példa bájtok kivágására** (`drüberspringen` szóból, ahol az `ü` a 3. és 4. bájt):

```bash
echo 'drüberspringen' | cut -b 5
# Kimenet: b

echo 'drüberspringen' | cut -b 5,9,13
# Kimenet: bpg

echo 'drüberspringen' | cut -b 1-5
# Kimenet: drüb
```

## 4. Valós (Gyakorlati) Példák csővezetékkel (Piping)

**Minden felhasználó listázása a rendszerről:**

A `getent passwd` kimenetéből kettőspont elválasztóval kivágjuk az 1-es mezőt (a felhasználónevet):

```bash
getent passwd | cut -d ':' -f1
```

**Adat kinyerése vesszővel elválasztott (CSV szerű) szövegből:**

```bash
echo "name,role,shell" | cut -d ',' -f2
# Kimenet: role
```

> **Megjegyzés:** Ha idézőjelekbe tett vesszők is vannak az adaton belül, a `cut` nem elég, arra speciális CSV eszközt kell használni.

**Ismétlődő szóközök kezelése:**

Mivel a `cut` a `-d` kapcsolóval csak egyetlen karaktert tud értelmezni, a több egymás melletti szóközt külön üres mezőknek tekinti. Ilyenkor a `tr` paranccsal (szelepelve) előbb le kell rövidíteni az ismétlődő szóközöket egyetlen szóközre:

```bash
echo "one   two   three" | tr -s ' ' | cut -d ' ' -f2
# Kimenet: two
```

**Az aktuális felhasználó alapértelmezett shelljének kiíratása:**

Kikeressük a felhasználó sorát a `/etc/passwd` fájlból, majd kivágjuk a 7-es mezőt:

```bash
grep "^$USER:" /etc/passwd | cut -d ':' -f7
# Kimenet: /bin/bash (vagy más shell)
```

##Források:
-[https://linuxize.com/post/linux-cut-command/]
-[https://labex.io/tutorials/linux-linux-cut-command-text-cutting-219187]
