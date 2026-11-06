# 4. Fejezet: A Grafikus Felhasználói Felület (GUI)

Bár parancssorból is mindent elintézhetsz, a Proxmox VE beépített (**ExtJS 7.x** alapú) HTML5 webes felülete hiánytalan menedzsmentet ad az egész cluster felett. Nincs szükség külön "vCenter-szerű" menedzsment szerverre: bármelyik node IP-címén belépsz, a teljes clustert látod és irányítod.

## A GUI felépítése és beállításai

- **My Settings (Saját beállítások):** A jobb felső sarokban a felhasználónevedre kattintva érhető el. Itt szűrheted, hogy a Datacenter nézetben mely tárolók kapacitása adódjon össze (**Dashboard Storages**), illetve testreszabhatod a beépített **xterm.js** terminál betűtípusát és sorközét.
- **Nézetek (Views):** A bal oldali fastruktúrában 4 féle nézet érhető el:
  - **Server View** (node-ok szerint),
  - **Folder View** (objektumtípus szerint),
  - **Pool View** (erőforrás-készletek szerint),
  - **Tag View** (címkék szerint).
- **Konzolok:** A VM-ekhez és a hoszthoz háromféle konzol nyitható:
  - **noVNC** (HTML5),
  - **SPICE** (külön kliens kell hozzá, de jobb az élmény),
  - **xterm.js** (tisztán szöveges, soros konzol).
- **Bulk Actions (Tömeges műveletek):** A Node nézetben a fejlécen található. Több VM/konténer egyszerre történő indítására, leállítására vagy migrációjára szolgál.

## Tag-ek (Címkék) és Consent Banner

**Címkék:** A VM-ek rendszerezésére szolgálnak. A színek alapértelmezetten a szöveg hash-értékéből generálódnak, de a **Datacenter $\rightarrow$ Options $\rightarrow$ Tag Style Override** alatt (vagy CLI-ből:

```bash
pvesh set /cluster/options --tag-style color-map=example:000000:FFFFFF
```

) teljesen testreszabhatók.

**Jogosultságok:** A címkék módosításához alapból `VM.Config.Options` jog kell. A **User Tag Access** opcióval korlátozható, hogy a felhasználók szabadon beírhassanak-e újakat (`free`), csak meglévőket használhatnak-e (`existing`), vagy egy előre definiált listából választhatnak-e (`list`).

**Consent Banner:** Ha a belépés előtt egy kötelezően elfogadandó szabályzatot (Jogi nyilatkozatot) szeretnél megjeleníteni, a **Datacenter $\rightarrow$ Options $\rightarrow$ Consent Text** alatt tudod megadni. Ez Base64 formátumban mentődik el a `datacenter.cfg`-ben.

## GUI technológiai alapjai és funkciói

A felület az **ExtJS 7.x** JavaScript keretrendszerre épül. Támogatja a **Kétlépcsős azonosítást** (Two-Factor Authentication: OATH, Yubikey) és különböző hitelesítési forrásokat (helyi, MS ADS, LDAP). A keresőmező több száz vagy ezer VM kezelésére is optimalizált.

## Bejelentkezés

A bejelentkezési ablakban bepipálható a **"Save User name"** (Felhasználónév mentése) opció, így a következő belépésnél nem kell újra beírni a nevet.

## A Fejléc (Header) gombjai

- **Documentation:** Közvetlenül megnyitja a referenciadokumentációt.
- **Create VM / Create CT:** Varázslók virtuális gépek és konténerek létrehozásához.
- **User Menu (Felhasználói menü):** Itt nemcsak a **My Settings** található, hanem a jelszó-önkiszolgáló (**Password self-service**), a nyelv (**Language**), a színtéma (**Color Theme**) módosítása, és a Kétlépcsős azonosítás (**TFA**) gyorsgombja is.

## My Settings további beállításai

A terminál (**xterm.js**) beállításain túl itt lehet módosítani a noVNC méretezését (**Scaling mode**: Auto, Local Scaling, Off), nullázni a felület elrendezését (**Layout Reset**), és beállítani a vendég gépekhez tartozó jegyzetek (**Guest Notes**) alapértelmezett megjelenítését.

## Log Panel finomságai

A futó feladatok (**tasks**) naplóbejegyzéseire duplán kattintva részletes kimenetet kapsz, és itt tudod megszakítani (**abort**) is a folyamatot. A **"Cluster log"** fülön a rövidebb, minden node-ra kiküldött üzenetek láthatók.

## A Content Panels (Tartalmi panelek) pontos menüstruktúrája

- **Datacenter szint:** A **Storage** itt globálisan kezelhető, a **Backup** cluster-szinten ütemezhető (mindegy, melyik gépen van a VM), továbbá itt van a **Metric Server** és a **Notifications** beállítása is.
- **Node szint:** A **System** fül alatt érhető el a hálózat, DNS, idő és a syslog. A **Disks** fül adja a lokális lemezek áttekintését.
- **Guest szint:** Különbség van a KVM és az LXC között. KVM-nél **Hardware** és **Monitor** fülek vannak, LXC-nél **Resources**, **Network** és **DNS**. Mindkettőnél elérhető a **Snapshots**, **Backup** és **Replication**.
- **Storage szint:** Két részre osztott nézet. A **Content** fül alatt láthatók a mentések, ISO fájlok vagy sablonok.
- **Pools szint:** Csak **Summary**, **Members** és **Permissions** fülekből áll.

## Címkék (Tags) parancssorból

A GUI (ceruza ikon, `+`, `-` gombok) mellett a CLI-ből pontosan így adhatsz több címkét egy géphez (pontosvesszővel elválasztva):

```bash
qm set ID --tags 'myfirsttag;mysecondtag'
```
