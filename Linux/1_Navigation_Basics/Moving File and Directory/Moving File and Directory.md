
Az mv (move) parancs fájlok és könyvtárak áthelyezésére, valamint átnevezésére szolgál Linux rendszerekben.

## Elmélet

Míg a cp parancs másolást és beillesztést (copy and paste) végez, addig az mv egy kivágás és beillesztés (cut and paste) művelet. Ez azt jelenti, hogy az áthelyezés után a forrásfájl vagy könyvtár törlődik az eredeti helyéről.

## Parancsok és Példák

1. Fájl áthelyezése másik könyvtárba

Bash

    mv source_file target_directory

(A forrásfájlt áthelyezi a célkönyvtárba.)

2. Több fájl áthelyezése egyszerre

Bash

    mv file1.txt file.2.txt file3.txt target_directory

Minta (glob) használatával minden .txt fájl áthelyezése:

Bash

    mv *.txt target_directory

3. Fájl átnevezése

Bash

    mv source_file target_directory/target_file

(Ha a célkönyvtárban nem létezik a fájl, létrehozza. Ha már létezik, kérdés nélkül felülírja!)

4. Könyvtár áthelyezése

Bash

    mv source_directory target_directory

(Ha a célkönyvtár létezik, a forráskönyvtár annak alkönyvtára lesz.)

5. Könyvtár átnevezése

Bash

    mv source_directory path_to_non_existing_directory

(A célkönyvtár nem létezhet, így a forráskönyvtár új nevet kap.)

6. Fájlok felülírásának kezelése (Opciók)

    -n (no overwrite): Nincs felülírás, megvédi a meglévő fájlokat.

    Bash

        mv -n source_file target_directory

    -i (interactive): Megerősítés kérése felülírás előtt.

    Bash

        mv -i source_file target_directory
        mv: overwrite 'target_directory/source_file'?

    -b (backup): Felülírás, de előtte biztonsági másolatot készít. Alapértelmezésben a fájl neve ~ jellel végződik.

    Bash

        mv -b file.txt target_dir/file.txt
        ls target_dir
        file.txt file.txt~

    -S (suffix): A biztonsági másolat alapértelmezett ~ végződésének megváltoztatása (a -b kapcsolóval együtt használva):

    Bash

        mv -S .back -b file.txt target_dir/file.txt
        ls target_dir
        file.txt file.txt.back

    -u (update): Csak akkor írja felül a fájlt, ha a forrás újabb (frissebb), vagy ha a célfájl még nem létezik.

7. Erőszakos áthelyezés / felülírás

Írásvédett fájlok esetén a rendszer kérdést tesz fel (pl. mv: replace 'target/file1.txt', overriding mode 0444 (r--r--r--)? y). Ennek elkerülésére és az azonnali felülírás kikényszerítésére a -f (force) kapcsoló használható:

Bash

##Forrás
-[https://linuxhandbook.com/mv-command/]

# mv Cheat Sheet

A Linuxban az `mv` parancs fájlok és könyvtárak áthelyezésére vagy átnevezésére szolgál. Áthelyezhet fájlokat és könyvtárakat egyik helyről a másikra, vagy átnevezheti azokat anélkül, hogy a helyük megváltozna. A `cp` paranccsal ellentétben, amely fájlokat másol, az `mv` átviszi azokat, ami azt jelenti, hogy az áthelyezés után az eredeti fájl vagy könyvtár törlődik a forráshelyéről.

## Gyakran használt paraméterek és jelentésük

- `mv file1.txt /path/to/destination/`
  - **Leírás:** A `file1.txt` fájlt a megadott célkönyvtárba helyezi át.
  - **Használat:** Egyszerű fájláthelyezés.

- `mv file1.txt file2.txt /path/to/destination/`
  - **Leírás:** Több fájlt (ebben az esetben a `file1.txt` és `file2.txt` fájlt) helyez át a megadott könyvtárba.
  - **Használat:** Több fájl áthelyezése egyszerre.

- `mv file1.txt newfile.txt`
  - **Leírás:** A `file1.txt` fájlt `newfile.txt`-re nevezi át ugyanabban a könyvtárban.
  - **Használat:** Fájl átnevezése anélkül, hogy áthelyeznénk.

- `mv -i file1.txt /path/to/destination/`
  - **Leírás:** Megerősítést kér, mielőtt felülírna egy fájlt a célkönyvtárban, ha már létezik azonos nevű fájl.
  - **Használat:** Biztosítja a megerősítést a meglévő fájlok felülírása előtt.

- `mv -u file1.txt /path/to/destination/`
  - **Leírás:** Csak akkor helyezi át a fájlt, ha az újabb, mint a célkönyvtárban lévő meglévő fájl, vagy ha a fájl nem létezik a célkönyvtárban.
  - **Használat:** Fájlok frissítése újabb verziókkal.

- `mv -v file1.txt /path/to/destination/`
  - **Leírás:** Részletes kimenetet jelenít meg, bemutatva az áthelyezési művelet részleteit.
  - **Használat:** Hasznos annak megtekintéséhez, hogy mely fájlok kerülnek áthelyezésre és hová.

- `mv -f file1.txt /path/to/destination/`
  - **Leírás:** Kikényszeríti az áthelyezési műveletet, felülírva a célkönyvtárban lévő fájlokat anélkül, hogy megerősítést kérne.
  - **Használat:** Meglévő fájlok felülírása megerősítés nélkül.

- `mv /source/directory/ /path/to/destination/`
  - **Leírás:** Egy teljes könyvtárat és annak tartalmát (fájlok, alkönyvtárak) helyezi át egy új helyre.
  - **Használat:** Könyvtárak áthelyezése.

- `mv --backup file1.txt /path/to/destination/`
  - **Leírás:** Biztonsági másolatot készít a célkönyvtárban lévő meglévő fájlról, ha már létezik azonos fájlnév.
  - **Használat:** Védelmet nyújt a fontos fájlok felülírása ellen.

- `mv --no-clobber file1.txt /path/to/destination/`
  - **Leírás:** Megakadályozza a célkönyvtárban lévő fájlok felülírását, ha azok már léteznek.
  - **Használat:** Biztosítja, hogy a fájlok véletlenül se legyenek felülírva.

## mv parancs Cheat Sheet

| Opció | Leírás | Példa |
|-------|--------|-------|
| `mv` | Alapvető áthelyezés vagy átnevezés | `mv file1.txt /home/user/docs/` |
| `-i` | Megerősítést kér a meglévő fájlok felülírása előtt | `mv -i file1.txt /path/` |
| `-u` | Csak akkor helyezi át, ha a forrásfájl újabb | `mv -u file1.txt /path/` |
| `-v` | Részletes mód (az áthelyezési művelet részleteinek megjelenítése) | `mv -v file1.txt /path/` |
| `-f` | Felülírás kikényszerítése megerősítés nélkül | `mv -f file1.txt /path/` |
| `--backup` | Biztonsági másolat készítése a célfájlról, ha létezik | `mv --backup file1.txt /path/` |
| `--no-clobber` | Semmilyen meglévő fájl felülírása nem történik meg | `mv --no-clobber file1.txt /path/` |
| `mv /dir/ /path/` | Egy könyvtár és annak összes tartalmának áthelyezése egy új helyre | `mv /source/dir /home/user/newlocation/` |
| `mv file1 newfile` | Fájl átnevezése | `mv file1.txt newfile.txt` |

## Összefoglalás

Az `mv` parancs egy egyszerű, mégis erőteljes eszköz a Linuxban, amelyet általában fájlok és könyvtárak áthelyezésére vagy átnevezésére használnak. Olyan opciókkal, mint a felülírás előtti megerősítés kérése, az áthelyezés kikényszerítése vagy a biztonsági másolatok megőrzése, az `mv` nagy rugalmasságot és irányítást biztosít a fájlkezelésben. A parancs hatékony használatának megértése zökkenőmentessé teheti a fájlok rendszerezését és áthelyezését Linux környezetekben.

##Forrás
-[https://www.commandinline.com/cheat-sheet/mv/
