# Docker Desktop telepítése Linuxra

> **Docker Desktop feltételek**
>
> A Docker Desktop nagyobb vállalatoknál (több mint 250 alkalmazott vagy több mint 10 millió USD éves árbevétel) történő kereskedelmi használata fizetős előfizetést igényel.

Ez az oldal általános rendszerkövetelményeket, támogatott platformokat és a Docker Desktop Linuxra való telepítésének utasításait tartalmazza.

> **Fontos**
>
> A Docker Desktop Linuxon egy virtuális gépet (VM) futtat, amely létrehoz és használ egy egyedi docker kontextust, a `desktop-linux`-ot indításkor.
>
> Ez azt jelenti, hogy a Linux Docker Engine-en (a telepítés előtt) telepített képek és konténerek nem érhetők el a Docker Desktop for Linuxban.
>
> **Docker Desktop vs Docker Engine: Mi a különbség?**

## Támogatott platformok

A Docker `.deb` és `.rpm` csomagokat biztosít a következő Linux disztribúciókhoz és architektúrákhoz:

| Platform | x86_64 / amd64 |
| --- | --- |
| Ubuntu | ✅ |
| Debian | ✅ |
| Red Hat Enterprise Linux (RHEL) | ✅ |
| Fedora | ✅ |

Egy kísérleti csomag elérhető Arch-alapú disztribúciókhoz. A Docker nem tesztelte vagy ellenőrizte a telepítést.

A Docker a fent említett disztribúciók aktuális és előző LTS-kiadásain, valamint a legutóbbi verzión támogatja a Docker Desktopot.

## Általános rendszerkövetelmények

A Docker Desktop sikeres telepítéséhez a Linux hosztodnak meg kell felelnie a következő általános követelményeknek:

- 64 bites kernel és CPU virtualizációs támogatás.
- KVM virtualizációs támogatás. Kövesd a KVM virtualization support utasításokat annak ellenőrzéséhez, hogy a KVM kernelmodulok engedélyezve vannak-e, és hogyan biztosítható a hozzáférés a KVM eszközhöz.
- A QEMU-nak 5.2-es vagy újabb verziónak kell lennie. Javasoljuk a legújabb verzióra való frissítést.
- systemd init rendszer.
- A GNOME, KDE vagy MATE asztali környezetek támogatottak, de mások is működhetnek.
  - Sok Linux disztribúción a GNOME környezet nem támogatja a tálcaikonokat. A tálcaikonok támogatásához telepítened kell egy GNOME kiterjesztést. Például: AppIndicator.
- Legalább 4 GB RAM.
- Engedélyezd az ID-mapping konfigurálását a felhasználói névterekben, lásd: File sharing. Ne feledd, hogy a Docker Desktop 4.35-ös és újabb verzióihoz erre már nincs szükség.
- Javasolt: Initialize pass a hitelesítő adatok kezeléséhez.

A Docker Desktop for Linux egy virtuális gépet (VM) futtat. További információért arról, hogy miért, lásd: Why Docker Desktop for Linux runs a VM.

> **Megjegyzés**
>
> A Docker nem nyújt támogatást a Docker Desktop for Linux beágyazott virtualizációs forgatókönyvekben való futtatásához. Javasoljuk, hogy a Docker Desktop for Linuxot natívan futtasd a támogatott disztribúciókon.

## KVM virtualizációs támogatás

A Docker Desktop egy VM-et futtat, amely KVM-támogatást igényel.

A `kvm` modulnak automatikusan be kell töltődnie, ha a hoszt rendelkezik virtualizációs támogatással. A modul manuális betöltéséhez futtasd:

```bash
modprobe kvm
```

A hosztgép processzorától függően a megfelelő modult kell betölteni:

```bash
modprobe kvm_intel  # Intel processzorok

modprobe kvm_amd    # AMD processzorok
```

Ha a fenti parancsok meghiúsulnak, a diagnosztikát a következő futtatásával tekintheted meg:

```bash
kvm-ok
```

Annak ellenőrzéséhez, hogy a KVM modulok engedélyezve vannak-e, futtasd:

```bash
lsmod | grep kvm
kvm_amd               167936  0
ccp                   126976  1 kvm_amd
kvm                  1089536  1 kvm_amd
irqbypass              16384  1 kvm
```

### KVM eszköz felhasználói jogosultságok beállítása

A `/dev/kvm` tulajdonjogának ellenőrzéséhez futtasd:

```bash
ls -al /dev/kvm
```

Add hozzá a felhasználódat a `kvm` csoporthoz a kvm eszközhöz való hozzáférés érdekében:

```bash
sudo usermod -aG kvm $USER
```

Jelentkezz ki és vissza, hogy a csoporttagságod újraértékelődjön.

## Docker SDK-k használata a Docker Desktoppal

A Docker Desktop for Linux egy felhasználónkénti socketet használ a rendszerszintű `/var/run/docker.sock` helyett. A Docker SDK-knak és a Docker démonhoz közvetlenül kapcsolódó eszközöknek a `DOCKER_HOST` környezeti változót kell beállítaniuk a Docker Desktophoz való kapcsolódáshoz. A konfigurációs részletekért lásd: How do I use Docker SDKs with Docker Desktop for Linux?.

## Forrás
-[https://docs.docker.com/desktop/setup/install/linux/]
