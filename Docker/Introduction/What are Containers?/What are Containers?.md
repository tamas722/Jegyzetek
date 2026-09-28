# Konténerek használata alkalmazások építéséhez, megosztásához és futtatásához

## Szoftver csomagolása szabványos egységekbe fejlesztéshez, szállításhoz és telepítéshez

A konténer egy szabványos szoftveregység, amely a kódot és annak összes függőségét csomagolja, hogy az alkalmazás gyorsan és megbízhatóan fusson egyik számítógépes környezetből a másikba. A Docker konténerkép egy könnyűsúlyú, önálló, futtatható szoftvercsomag, amely mindent tartalmaz, ami az alkalmazás futtatásához szükséges: kódot, futtatókörnyezetet, rendszereszközöket, rendszerkönyvtárakat és beállításokat.

A konténerképek futásidőben konténerekké válnak, és a Docker konténerek esetében – a képek akkor válnak konténerekké, amikor a Docker Engine-en futnak. Linux- és Windows-alapú alkalmazásokhoz egyaránt elérhető, a konténerizált szoftver mindig ugyanúgy fog futni, függetlenül az infrastruktúrától. A konténerek elkülönítik a szoftvert a környezetétől, és biztosítják, hogy egységesen működjön a fejlesztés és a staging közötti különbségek ellenére is.

**A Docker Engine-en futó Docker konténerek:**

- **Szabványos:** A Docker teremtette meg a konténerek iparági szabványát, így azok bárhová hordozhatók.
- **Könnyűsúlyú:** A konténerek megosztják a gép OS rendszermagját, ezért nem igényelnek külön OS-t alkalmazásonként, ami nagyobb szerverhatékonyságot eredményez, és csökkenti a szerver- és licencköltségeket.
- **Biztonságos:** Az alkalmazások biztonságosabbak a konténerekben, és a Docker biztosítja az iparág legerősebb alapértelmezett izolációs képességeit.

## A Docker konténerek mindenhol jelen vannak: Linux, Windows, adatközpont, felhő, szerver nélküli (serverless) stb.

A Docker konténertechnológia 2013-ban indult nyílt forráskódú Docker Engine-ként.

A meglévő számítástechnikai konténerkoncepciókat használta fel, különösen a Linux világában, az úgynevezett cgroupok és namespaces primitíveket. A Docker technológiája azért egyedi, mert a fejlesztők és rendszerüzemeltetők igényeire összpontosít, hogy elválassza az alkalmazásfüggőségeket az infrastruktúrától.

A Linux világban elért siker a Microsofttal való partnerséghez vezetett, amely a Docker konténereket és azok funkcionalitását a Windows Serverre hozta.

A Docker és nyílt forráskódú projektje, a Moby által elérhető technológiát minden jelentős adatközponti szállító és felhőszolgáltató hasznosította. Ezen szolgáltatók közül sokan a Docker-t használják konténer-natív IaaS kínálatukhoz. Ezenkívül a vezető nyílt forráskódú szerver nélküli (serverless) keretrendszerek is hasznosítják a Docker konténertechnológiát.

## Konténerek és virtuális gépek összehasonlítása

A konténerek és a virtuális gépek hasonló erőforrás-izolációs és -kiosztási előnyökkel rendelkeznek, de másképp működnek, mert a konténerek az operációs rendszert virtualizálják a hardver helyett. A konténerek hordozhatóbbak és hatékonyabbak.

### KONTÉNEREK

A konténerek az alkalmazási réteg szintjén lévő absztrakció, amely a kódot és a függőségeket együtt csomagolja. Több konténer futhat ugyanazon a gépen, és megoszthatja az OS rendszermagot más konténerekkel, mindegyik izolált folyamatként futva a felhasználói térben. A konténerek kevesebb helyet foglalnak, mint a VM-ek (a konténerképek jellemzően tíz-tíz MB méretűek), több alkalmazást tudnak kezelni, és kevesebb VM-et és operációs rendszert igényelnek.

### VIRTUÁLIS GÉPEK

A virtuális gépek (VM-ek) a fizikai hardver absztrakciója, amely egy szervert több szerverré alakít. A hipervizor lehetővé teszi, hogy több VM fusson egyetlen gépen. Minden VM tartalmaz egy teljes operációs rendszer-másolatot, az alkalmazást, a szükséges binárisokat és könyvtárakat – tíz-tíz GB-ot foglalva. A VM-ek indítása is lassú lehet.

### Konténerek és virtuális gépek együtt

A konténerek és a VM-ek együtt használva nagyfokú rugalmasságot biztosítanak az alkalmazások telepítésében és kezelésében.

## Konténer szabványok és iparági vezető szerep

A Docker 2013-as indulása forradalmat indított el az alkalmazásfejlesztésben – a szoftverkonténerek demokratizálásával. A Docker kifejlesztett egy Linux konténertechnológiát – amely hordozható, rugalmas és könnyen telepíthető. A Docker nyílt forráskódúvá tette a libcontainer-t, és partnerséget kötött a közreműködők világméretű közösségével annak továbbfejlesztéséhez. 2015 júniusában a Docker adományozta a konténerkép-specifikációt és a runc néven ismert futtatókörnyezeti kódot az Open Container Initiative (OCI) szervezetnek, hogy segítse a szabványosítás megteremtését, ahogy a konténerökoszisztéma növekszik és érik.

Ezt a fejlődést követve a Docker továbbra is visszaad a containerd projekttel, amelyet a Docker 2017-ben adományozott a Cloud Native Computing Foundation (CNCF) szervezetnek. A containerd egy iparági szabványú konténer futtatókörnyezet, amely a runc-t hasznosítja, és az egyszerűségre, robusztusságra és hordozhatóságra helyezve a hangsúlyt hozták létre. A containerd a Docker Engine alapvető konténer futtatókörnyezete.

## Forrás
-[https://www.docker.com/resources/what-container/]
