# `docker image pull`

## Leírás

Image letöltése egy regisztrumból.

## Használat

```bash
docker image pull [OPTIONS] NAME[:TAG|@DIGEST]
```

## Álnév (Aliases)

```bash
docker pull
```

## Leírás

A legtöbb image-ed egy Docker Hub regisztrumból származó alapimage-re épül.

A Docker Hub sok előre elkészített image-et tartalmaz, amelyeket letölthetsz és kipróbálhatsz anélkül, hogy sajátot kellene definiálnod és konfigurálnod.

Egy adott image vagy image-készlet (azaz egy repository) letöltéséhez használd a `docker pull` parancsot.

## Proxy konfiguráció

Ha HTTP proxy szerver mögött vagy, például vállalati környezetben, lehet, hogy konfigurálnod kell a Docker démont a proxy szerver használatára olyan műveletekhez, mint az image-ek letöltése és feltöltése. A részletekért lásd a `dockerd` parancssori referenciát.

## Párhuzamos letöltések

Alapértelmezésben a Docker démon egyszerre három réteget tölt le egy image-ből. Ha alacsony sávszélességű kapcsolaton vagy, ez időtúllépési problémákat okozhat, és csökkentheted a `--max-concurrent-downloads` démon opcióval. A részletekért lásd a démon dokumentációját.

## Opciók

| Opció | Alapértelmezés | Leírás |
| --- | --- | --- |
| `-a`, `--all-tags` | | Az összes címkézett image letöltése a repository-ból |
| `--platform` | | API 1.32+ Platform beállítása, ha a szerver többplatformos |
| `-q`, `--quiet` | | A részletes kimenet elnyomása |

## Példák

### Image letöltése a Docker Hubról

Egy adott image vagy image-készlet (azaz egy repository) letöltéséhez használd a `docker image pull` parancsot (vagy a `docker pull` rövidítést). Ha nincs megadva címke, a Docker Engine alapértelmezésben a `:latest` címkét használja. Ez a példa a `debian:latest` image-et tölti le:

```bash
docker image pull debian
```

```plaintext
Using default tag: latest
latest: Pulling from library/debian
e756f3fdd6a3: Pull complete
Digest: sha256:3f1d6c17773a45c97bd8f158d665c9709d7b29ed7917ac934086ad96f92e4510
Status: Downloaded newer image for debian:latest
docker.io/library/debian:latest
```

A Docker image-ek több rétegből állhatnak. A fenti példában az image egyetlen rétegből áll: `e756f3fdd6a3`.

A rétegeket az image-ek újrahasznosíthatják. Például a `debian:bookworm` image megosztja a rétegét a `debian:latest`-szel. A `debian:bookworm` image letöltése ezért csak a metaadatait tölti le, de a rétegeit nem, mert a réteg már helyben van:

```bash
docker image pull debian:bookworm
```

```plaintext
bookworm: Pulling from library/debian
Digest: sha256:3f1d6c17773a45c97bd8f158d665c9709d7b29ed7917ac934086ad96f92e4510
Status: Downloaded newer image for debian:bookworm
docker.io/library/debian:bookworm
```

Annak megtekintéséhez, hogy mely image-ek vannak helyben, használd a `docker images` parancsot:

```bash
docker images
```

```plaintext
REPOSITORY   TAG        IMAGE ID       CREATED        SIZE
debian       bookworm   4eacea30377a   8 days ago     124MB
debian       latest     4eacea30377a   8 days ago     124MB
```

A Docker egy tartalomcímezhető image-tárolót használ, és az image ID egy SHA256 digest, amely lefedi az image konfigurációját és rétegeit. A fenti példában a `debian:bookworm` és a `debian:latest` ugyanazzal az image ID-vel rendelkezik, mert ugyanaz az image különböző nevekkel címkézve. Mivel ugyanaz az image, a rétegeik csak egyszer vannak tárolva, és nem fogyasztanak extra lemezterületet.

Az image-ekről, rétegekről és a tartalomcímezhető tárolóról további információért lásd: understand images, containers, and storage drivers.

### Image letöltése digest alapján (változhatatlan azonosító)

Eddig az image-eket a nevük (és „címkéjük”) alapján töltötted le. A nevek és címkék használata kényelmes módja az image-ekkel való munkának. Címkék használatakor újra `docker pull`-olhatsz egy image-et, hogy megbizonyosodj arról, hogy az adott image legfrissebb verziójával rendelkezel. Például a `docker pull ubuntu:24.04` letölti az Ubuntu 24.04 image legfrissebb verzióját.

Bizonyos esetekben nem szeretnéd, hogy az image-ek újabb verziókra frissüljenek, hanem egy rögzített verziót szeretnél használni. A Docker lehetővé teszi, hogy egy image-et a digestje alapján tölts le. Amikor egy image-et digest alapján töltesz le, pontosan megadod, melyik verziót töltsd le. Ezáltal „rögzítheted” az image-et ahhoz a verzióhoz, és garantálhatod, hogy az általad használt image mindig ugyanaz.

Ahhoz, hogy megtudd egy image digestjét, először töltsd le az image-et. Töltsük le a legfrissebb `ubuntu:24.04` image-et a Docker Hubról:

```bash
docker pull ubuntu:24.04
```

```plaintext
24.04: Pulling from library/ubuntu
125a6e411906: Pull complete
Digest: sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30
Status: Downloaded newer image for ubuntu:24.04
docker.io/library/ubuntu:24.04
```

A Docker kiírja az image digestjét a letöltés befejezése után. A fenti példában az image digestje:

```plaintext
sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30
```

A Docker akkor is kiírja egy image digestjét, amikor egy regisztrumba pushol. Ez hasznos lehet, ha ahhoz a verzióhoz szeretnél rögzíteni, amelyet éppen pusholtál.

A digest a címke helyét veszi át egy image letöltésekor. Például a fenti image digest alapján való letöltéséhez futtasd a következő parancsot:

```bash
docker pull ubuntu@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30
```

```plaintext
docker.io/library/ubuntu@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30: Pulling from library/ubuntu
Digest: sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30
Status: Image is up to date for ubuntu@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30
docker.io/library/ubuntu@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30
```

A digest a Dockerfile `FROM` utasításában is használható, például:

```dockerfile
FROM ubuntu@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30
LABEL org.opencontainers.image.authors="some maintainer <maintainer@example.com>"
```

> **Megjegyzés**
>
> Ennek a funkciónak a használata „rögzít” egy image-et egy adott időponthoz tartozó verzióhoz. A Docker ezért nem tölti le az image frissített verzióit, amelyek biztonsági frissítéseket tartalmazhatnak. Ha frissített image-et szeretnél letölteni, ennek megfelelően módosítanod kell a digestet.

### Letöltés egy másik regisztrumból

Alapértelmezésben a `docker pull` a Docker Hubról tölti le az image-eket. Lehetőség van manuálisan megadni a regisztrum elérési útját, ahonnan letölteni szeretnél. Például, ha beállítottál egy helyi regisztrumot, megadhatod annak elérési útját, hogy onnan tölts le. A regisztrum elérési útja hasonló egy URL-hez, de nem tartalmaz protokoll-specifikátort (`https://`).

A következő parancs a `testing/test-image` image-et tölti le egy helyi regisztrumból, amely az 5000-es porton figyel (`myregistry.local:5000`):

```bash
docker image pull myregistry.local:5000/testing/test-image
```

A regisztrum hitelesítő adatait a `docker login` kezeli.

A Docker a `https://` protokollt használja a regisztrummal való kommunikációhoz, kivéve, ha a regisztrumot nem biztonságos kapcsolaton keresztül engedélyezik. További információért lásd az insecure registries szakaszt.

### Repository letöltése több image-el (`-a`, `--all-tags`)

Alapértelmezésben a `docker pull` egyetlen image-et tölt le a regisztrumból. Egy repository több image-et is tartalmazhat. Egy repository összes image-ének letöltéséhez add meg a `-a` (vagy `--all-tags`) opciót a `docker pull` használatakor.

Ez a parancs letölti az összes image-et az `ubuntu` repository-ból:

```bash
docker image pull --all-tags ubuntu
```

```plaintext
Pulling repository ubuntu
ad57ef8d78d7: Download complete
105182bb5e8b: Download complete
511136ea3c5a: Download complete
73bd853d2ea5: Download complete
....

Status: Downloaded newer image for ubuntu
```

A letöltés befejezése után használd a `docker image ls` parancsot (vagy a `docker images` rövidítést) a letöltött image-ek megtekintéséhez. Az alábbi példa az összes helyben lévő `ubuntu` image-et mutatja:

```bash
docker image ls --filter reference=ubuntu
```

```plaintext
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
ubuntu       22.04     8a3cdc4d1ad3   3 weeks ago    77.9MB
ubuntu       jammy     8a3cdc4d1ad3   3 weeks ago    77.9MB
ubuntu       24.04     35a88802559d   6 weeks ago    78.1MB
ubuntu       latest    35a88802559d   6 weeks ago    78.1MB
ubuntu       noble     35a88802559d   6 weeks ago    78.1MB
```

### Letöltés megszakítása

A `docker pull` folyamat megölése, például a `CTRL-c` megnyomásával, miközben egy terminálban fut, megszakítja a letöltési műveletet.

```bash
docker pull ubuntu
```

```plaintext
Using default tag: latest
latest: Pulling from library/ubuntu
a3ed95caeb02: Pulling fs layer
236608c7b546: Pulling fs layer
^C
```

Az Engine megszakítja a letöltési műveletet, amikor a démon és a kliens (amely kezdeményezi a letöltést) közötti kapcsolat bármilyen okból megszakad vagy elveszik, vagy a parancsot manuálisan megszakítják.
