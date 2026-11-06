# 5. Fejezet: Cluster Manager (A valódi mérnöki munka)

Ez a Proxmox VE legfontosabb architekturális eleme. A **`pvecm` (Proxmox VE Cluster Manager)** és a **Corosync** felel azért, hogy a fizikai gépek egyetlen logikai egységként működjenek. A Corosync osztja meg a `/etc/pve` (**`pmxcfs`**) fájlrendszert a node-ok között, és biztosítja a **quorumot**.

## 5.1 és 5.8 Hálózati és rendszerkövetelmények

A cluster kommunikációja rendkívül érzékeny a késleltetésre.

- **Portok és kapcsolat:** UDP `5405-5412` portok (Corosync/Kronosnet) és TCP `22` (SSH) szükségesek.
- **Dedikált hálózat:** Kötelező (vagy nagyon erősen ajánlott) egy dedikált fizikai hálózati kártya a Corosync forgalomnak. A késleltetésnek **5 milliszekundum** alatt kell lennie. Bár a sávszélességigény kicsi, a **"jitter"** (késleltetés-ingadozás) szétesést okoz. **Sose tedd a tárolóforgalommal (pl. Ceph, iSCSI) közös hálózatra!**
- **Multicast vs. Unicast:** A Proxmox VE 6.0 óta a **Kronosnet (Knet)** a standard, ami hagyományos UDP unicastot használ. A régi Multicast mód visszaállítható, de ezzel elveszíted a hálózati titkosítást és a redundanciát.

## 5.8.2 Corosync Bondokon keresztül (Kritikus figyelmeztetés!)

Ha **LACP (802.3ad)** alapú Link Aggregation-t használsz a Corosync hálózaton, **KÖTELEZŐ** a `bond-lacp-rate fast` paraméter beállítása mind a Proxmox-on, mind a fizikai switch-en.

**Miért?** Alapértelmezett (`slow`) beállítással az LACP hibaérzékelése 90 másodperc. A Proxmox HA watchdogja viszont nagyjából 60 másodperc után újraindítja (**fencing**) a gépet, ha nincs quorum. Ha egy kábel meghal, a gépek hamarabb lelövik magukat, mint ahogy a bond átállna a másik portra. A `fast` beállítással ez 3 másodpercre csökken. A `balance-rr`, `balance-xor` és `balance-alb` bond módok használata a Corosync-hez szintén kerülendő, mert aszimmetrikus kapcsolatot eredményezhetnek.

## 5.3 és 5.4 Cluster építése és bővítése

A cluster létrehozása során egyedi nevet kell választanod, ami később **nem változtatható**. Több cluster is lehet ugyanazon a hálózaton, de a neveiknek különbözniük kell.

- **Létrehozás:**

```bash
pvecm create CLUSTERNAME --link0 10.10.10.1
```

(Kifejezetten IP-cím megadása ajánlott a hosztnév helyett).

- **Node hozzáadása (Join):** A csatlakozandó gépen futtasd:

```bash
pvecm add <MASTER_IP>
```

> **VIGYÁZAT:** A csatlakozó gépen a `/etc/pve` **teljes tartalma felülíródik!** Egy olyan gépet, amin már vannak virtuális gépek (VMID-k), **tilos becsatlakoztatni**, mert névütközést és adatvesztést okoz. A VM-eket előbb le kell menteni (`vzdump`), a gépet be kell léptetni, majd más ID-val visszaállítani.

## 5.5 és 5.6 Node eltávolítása a clusterből (A helyes sorrend)

Ha egy szervert végleg ki akarsz venni a clusterből (és a Ceph-ből):

1. **Ceph takarítás (ha van):** Először az eltávolítandó gépen semmisítsd meg az OSD-ket, majd az MDS, MON és MGR szolgáltatásokat. Végül vedd ki a node-ot a CRUSH térképből (`ceph osd crush remove <node>`).
2. **Áramtalanítás:** Állítsd le és **Fizikailag / Hálózatilag kapcsold le** a gépet. **SOHA ne futtasd a törlő parancsot úgy, hogy a gép még be van kapcsolva a hálózaton, mert szétveri a clustert!**
3. **Eltávolítás a túlélő gépről:** Egy egészséges node-on futtasd:

```bash
pvecm delnode <halott_gep_neve>
```

4. **Ha a gépet újra akarod használni (Separate without reinstall):** Állítsd le a `pve-cluster` és `corosync` szolgáltatásokat, indítsd el a fájlrendszert lokális módban (`pmxcfs -l`), töröld a `/etc/pve/corosync.conf` és a `/etc/corosync/*` fájlokat, majd indítsd újra a `pve-cluster`-t.

## 5.11 Külső szavazó (QDevice / Corosync External Vote Support)

A Proxmox szavazatalapú rendszert (**Quorum**) használ. Két node-os cluster esetén, ha az egyik gép kiesik, a maradék 1 szavazattal nem lesz meg a többség (Quorum), a fájlrendszer írásvédetté válik, a VM-ek nem indulnak el.
Ennek megoldása a **QDevice (Külső szavazó)**.

- **Működés:** Egy teljesen független, harmadik gépre (pl. egy málnaPC-re vagy egy apró felhős VPS-re) feltelepíted a `corosync-qnetd` csomagot (bármilyen Debian/Ubuntu linux jó).
- **Bekötés a Proxmoxokon:**

```bash
apt install corosync-qdevice
pvecm qdevice setup <KULSO_GEP_IP>
```

A QDevice TCP-n kommunikál, így lehet magasabb a hálózati késleltetése.

> **Páratlan gépszámnál kerülendő:** Ha 3 géped van, ne használj QDevice-t, mert ha a külső szavazó offline lesz, a cluster elveszti a hibatűrését.

## 5.12 Corosync haladó beállításai

A `/etc/pve/corosync.conf` fájl kézi szerkesztésekor **mindig növelni kell a `config_version` értékét**.

**Token Coefficient (Failover gyorsítás):** A Proxmox 9.2-től kezdve az új clusterek **125 ms**-os "token coefficient" értékkel jönnek létre (korábban 650 ms volt). A token timeout és a consensus timeout összege adja meg, mennyi idő alatt áll fel újra a cluster egy gép kiesése után. Ennek az összegnek feltétlenül **45 másodperc alatt** kell maradnia, máskülönben a HA Watchdog hamarabb indul be, és feleslegesen lelövi a túlélő gépeket. A 125 ms-os érték beállítása a `totem` szekcióban ajánlott lépés, ha a hálózatod elég stabil hozzá.

**Vészhelyzeti Quorum:** Ha nincs quorum, de módosítanod kell egy fájlt, a

```bash
pvecm expected 1
```

paranccsal átmenetileg becsaphatod a rendszert, hogy elfogadja az 1 db szavazatot (**csak ha pontosan tudod, mit csinálsz**).

## 5.15 Dedikált migrációs hálózat

Az élő migráció (**vMotion**) alapértelmezetten a cluster kommunikációs (Corosync) hálózatát terheli, ami széteséshez vezethet.
A `datacenter.cfg`-ben (vagy a GUI **Datacenter $\rightarrow$ Options** alatt) dedikálhatsz egy külön, **10Gbps+** sebességű hálózatot csak a migrációnak. Példa a konfigurációra: `migration: secure,network=10.1.2.0/24`.
A `secure` (titkosított, SSH-alapú) mód leterheli a CPU-t, gyors hálózaton (10Gbps felett) a teljesítmény növelése érdekében átállítható `insecure` módra, de ekkor a VM-ek RAM tartalma **titkosítatlanul** utazik a hálózaton.

## 5.10 SSH és a `.bashrc` csapdája

A Proxmox az SSH-t használja a konzol-proxykhoz, a biztonságos memóriamigrációhoz és a storage replikációhoz. Az SSH hitelesítőkulcsok szinkronban vannak a node-ok között.

**Kritikus probléma:** Ha a `/root/.bashrc` fájlba olyan parancsokat írtál (pl. `neofetch`, `tmux` auto-start, vagy egyedi prompt szkriptek), amik minden bejelentkezéskor lefutnak, azok **tönkretehetik a cluster működését**, mert a háttérben futó non-interaktív SSH alagutak is lefuttatják őket. Ennek elkerülésére a `.bashrc` elejére be kell szúrni egy ellenőrzést, hogy csak interaktív session esetén fusson le a kódod.


## További Cluster követelmények

Az élő migráció csak akkor támogatott, ha a node-ok processzorai azonos gyártótól származnak (**ugyanaz a CPU vendor**). A megbízható quorumhoz minimálisan **3 node** kell (a 2-node-oshoz a **QDevice** az alternatíva).

## GUI Cluster Csatlakozás – Assist mód

A csatlakozáshoz a **Datacenter $\rightarrow$ Cluster** menüben a **Join Information** gombbal egy kódolt stringet kell kimásolni, majd a csatlakozó gépen a **Join Cluster** ablakba beilleszteni. Az **Assisted Join** pipa automatikusan kitölti a Fingerprint és Peer Address mezőket, csak a jelszót kell beírni.

## CLI csatlakozás jelszó nélküli fall-back

A `pvecm add IP` parancs ma már API-n keresztül, ujjlenyomat-ellenőrzéssel hitelesít, így a root SSH jelszavas belépés tiltva lehet (`PasswordAuthentication no`). A régi, SSH-alapú csatlakozás kényszeríthető a `--use_ssh` paraméterrel (ami az `ssh-copy-id`-t használja).

## Nodes listázása

Csak a gépek listájának lekéréséhez a `pvecm status` helyett használható a rövidebb parancs:

```bash
pvecm nodes
```

## Node törlésének kritikus szabályai

- Ha egy replikációs feladatot (**Replication job**) nem törölsz a gép eltávolítása előtt, az **törölhetetlenné válik**. Fontos, hogy a migráció automatikusan megfordítja a replikáció irányát, így migráció után is maradhatnak feladatok az eltávolítandó gépen!
- Amikor lefuttatod a `pvecm delnode` parancsot, megjelenhet egy `could not kill node (error = cs_err_not_exist)` hibaüzenet. Ez **teljesen normális** (mert az offline gépet nem tudja a Corosync kilőni), nyugodtan figyelmen kívül hagyható.
- Ha ugyanazt a gépet újra be akarod léptetni a clusterbe, **kötelező a Proxmox VE nulláról történő újratelepítése** rajta.

## Corosync IP és Hosztnév

A hosztnevek feloldása `getaddrinfo` segítségével történik (az IPv6 élvez prioritást). Mivel a hosztnevek IP-je a Corosync tudta nélkül is megváltozhat (pl. DNS átírás), a dokumentáció kifejezetten a **statikus IP-címek közvetlen használatát** javasolja.

## Corosync linkek prioritása

Ha több linket adsz meg (redundancia), azok prioritás alapján működnek. A `pvecm create` paranccsal így adhatók meg: `priority=15`. Minél **kisebb** a szám, annál **magasabb** a prioritás (ha nincs megadva, a Link 0 a legmagasabb). A különböző prioritású linkek nem tudnak egymással kommunikálni, a rendszer mindig csak az elérhető legmagasabb prioritásút használja.

## SSH kulcsok takarítása

Régebbi rendszereken a `/etc/ssh/ssh_known_hosts` egy symlink volt a `/etc/pve/priv/known_hosts` fájlra. Ez ma már **host-key pinning**-re cserélődött. A régi symlink eltávolítható a következő paranccsal:

```bash
pvecm updatecerts --unmerge-known-hosts
```

## QDevice finomságok

- **Döntés (Tie breaking):** Ha két azonos méretű részre szakad a cluster, a QDevice alapértelmezetten a kisebb Node ID-val rendelkező partíciónak adja a szavazatát. Ez a `tie_breaker` opcióval módosítható a konfigurációban.
- **Státusz jelek:** A `pvecm status` kimenetén a QDevice oszlopai:
  - **A/NA** (Alive / Not Alive – eléri-e a daemont),
  - **V/NV** (Vote / No Vote),
  - **MW/NMW** (Master Wins / Not Master Wins),
  - **NR** (Not Registered).
  Ha a státusz **NA**, ellenőrizni kell a QDevice TCP **5403**-as portját.
- **Eltávolítás parancssorból:**

```bash
pvecm qdevice remove
```

## Token Timeout ellenőrzése és számítása

Hogy lásd a pontos időtúllépési értékeket (miliszekundumban), ezt a parancsot kell futtatni:

```bash
corosync-cmapctl | grep -Ew 'runtime.config.totem.token|runtime.config.totem.consensus'
```

A háttérben futó matematikai képlet:

- $token = 3000 + (node\text{-ok\_száma} - 2) \times token\_coefficient$
- $consensus = 1.2 \times token$

## Cluster Hidegindítás (Cold Start)

Áramszünet után a gépek nem egyszerre indulnak el. A `pve-guests` szolgáltatás vár, amíg a cluster el nem éri a Quorumot, és a beállított `onboot` flaggel rendelkező VM-ek csak ezután kezdenek el indulni.

## Vendég VMID Auto-kiválasztás

Amikor új VM-et hozol létre, a rendszer alapértelmezetten a **100 és 1.000.000** közötti tartományban keres szabad ID-t. Ezt a tartományt a **Datacenter $\rightarrow$ Options** menüben (vagy a `datacenter.cfg` fájlban) felülbírálhatod, hogy mondjuk csak 100.000 feletti ID-kat osszon ki (ez **nem hard limit**, csak a `next-id` API hívást befolyásolja).
