# A `traceroute` parancs példái Linuxon

A `traceroute` az egyik leggyakoribb Linux hálózati parancs. Arra szolgál, hogy nyomon kövesse az útvonalat a végső szerverig. Íme, hogyan kell használni a `traceroute` parancsot.

**2022. november 28. · Szerző: Sagar Sharma · 4 perc olvasás**

A `traceroute` hasonló a `ping` parancshoz, de részletesebb kimenetet kínál. Nyomon követi az útvonalat az eredettől a célrendszerig.

Ha nem tudnád, amikor egy távoli rendszerhez csatlakozol, az adat több köztes csomóponton (a routereden, az internetszolgáltatód routerén stb.) halad keresztül.

A `traceroute`-ot rendszergazdák és hálózati szakemberek használják a hálózat hibaelhárítására.

## A `traceroute` telepítése Linuxon

A `traceroute` segédprogram a legtöbb modern Linux disztribúción nincs előre telepítve.

Ha Ubuntu-alapú disztribúción vagy, a következő parancs elvégzi a feladatot:

```bash
sudo apt install traceroute
```

Hasonlóképpen, ha Fedorán vagy, a következő parancs segíthet:

```bash
sudo yum install traceroute
```

Most pedig térjünk rá a példákra.

## A `traceroute` parancs használata Linuxon

Kezdem a legegyszerűbbel, ahol a hosztnevet fűzöm a `traceroute` parancshoz, és megmutatja azoknak a routereknek a listáját (IP-vel), amelyeken keresztülhaladt a cél eléréséhez:

```bash
traceroute google.com
```

7 routerre volt szükség a cél eléréséhez, és alapértelmezésben 3 csomagot küldött mindegyikhez, és visszaadta a válaszadáshoz szükséges időt.

Többféleképpen használható a `traceroute` parancs Linuxon. Nézzük a példákat.

### 1. Várakozási idő beállítása

A `traceroute` alapértelmezett várakozási ideje 3 másodperc, ami azt jelenti, hogy 5 másodpercet vár, mielőtt a következőre ugrik.

Ha módosítani szeretnéd az alapértelmezést, fűzd hozzá a másodpercek számát a `-w` opcióval:

Például a várakozási időt 1 másodpercre változtattam a `google.com` esetében:

```bash
traceroute -w 1 google.com
```

### 2. A küldött/fogadott csomagok számának megváltoztatása

Alapértelmezésben a `traceroute` 3 csomagot küld egy ugráshoz, és ha ezt a viselkedést meg akarod változtatni, a `-q` opciót kell használnod.

Referenciaként 4 csomagot küldök ugrásonként a Google domainen keresztül:

```bash
traceroute -q 4 google.com
```

### 3. Az ugrások maximális számának megadása

Egyetlen lekérdezés alapértelmezett maximális ugrásszáma 30 a `traceroute`-ban, ami 255-ig bővíthető.

Ehhez fűzd hozzá az ugrások maximális számát a `-m` opcióval:

```bash
traceroute -m 4 google.com
```

### 4. Annak megadása, hogy milyen TTL-lel kezdjen

Alapértelmezésben a `traceroute` az első TTL-lel kezd, de ezt a viselkedést a `-f` opcióval változtathatod meg.

Referenciaként a Google domainjének traceroute-olását az 5. TTL-től szerettem volna kezdeni, ezért a következő parancsot kellett használnom:

```bash
traceroute -f 5 google.com
```

### 5. Alternatív forrás IP-cím hozzáadása a traceroute-hoz

Ha egy alternatív forrást szeretnél hozzáadni ahhoz, amit általában a traceroute-hoz adsz, hozzá kell fűznöd az alternatív IP-t a `-s` opcióval.

Például hozzáadtam egy alternatív forrás IP-t, a `192.168.1.7`-et a `google.com` mellé:

```bash
traceroute -s 192.168.1.7 google.com
```

### 6. A hosztnév IP-címre való leképezésének letiltása traceroute közben

Ha valamilyen okból le szeretnéd tiltani a hosztnév-leképezést traceroute közben, használhatod a `-n` opciót:

```bash
traceroute -n google.com
```

### 7. Csomagok átirányítása az átjárón keresztül a `traceroute` segítségével

Megadhatod az átjáró címét a `-g` opcióval a domain mellett, amelyet traceroute-olni szeretnél.

Például hozzáadtam a `192.168.1.7`-et átjárócímként:

```bash
traceroute -g 192.168.1.7 google.com
```

### 8. A célport megváltoztatása a `traceroute`-ban

Ha meg szeretnéd változtatni a forrásdomained célportját traceroute közben, a `-p` opcióval adhatod meg a portot.

Például a célportomat `29879`-re változtattam:

```bash
traceroute -p 29879 google.com
```

### 9. A maximális csomaghossz beállítása

Alapértelmezésben a `traceroute` 60 bájtos csomaghosszt használ, és ezt a viselkedést úgy változtathatod meg, hogy egyszerűen hozzáfűzöd a csomagméretet a forrás IP-hez.

Például 100 bájtos maximális csomaghosszra változtattam traceroute közben:

```bash
traceroute google.com 100
```

### 10. A próbacsomag-töredezés letiltása

Ha nem szeretnéd engedélyezni a próbacsomag-töredezést, van egy egyszerű mód a leállítására.

Csak a `-F` opciót kell használnod, és nem lesz próbacsomag-töredezés:

```bash
traceroute -F google.com
```

## Összegzés

Ha kíváncsi vagy arra, hogyan működik a `traceroute`, olvasd el a másik útmutatót, ahol elmagyaráztam a `traceroute` működését.

Remélem, hasznosnak találod, és ha bármilyen kérdésed van, jelezd a megjegyzésekben.

## Forrás
-[https://linuxhandbook.com/traceroute/]
-[https://www.howtogeek.com/657780/how-to-use-the-traceroute-command-on-linux/]
