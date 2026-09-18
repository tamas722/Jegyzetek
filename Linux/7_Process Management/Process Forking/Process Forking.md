# A Linux `fork()` Rendszerhívás (System Call) Megértése

A `fork()` rendszerhívás a Linux és más Unix-szerű operációs rendszerek egyik legfontosabb és leggyakrabban használt alapköve új folyamatok (processzek) létrehozására.

## 1. Mit csinál a `fork()`?

A `fork()` úgy hoz létre egy új folyamatot, hogy lemásolja (duplikálja) a hívó folyamatot. A sikeres hívás után két szinte teljesen azonos folyamat fog futni ugyanazzal a kóddal: az eredeti "szülő" (parent) folyamat és az újonnan létrehozott "gyermek" (child) folyamat.

A két folyamat memóriaterülete innentől kezdve elkülönül, de ugyanazt a futtatható kódot és változókat tartalmazzák. Ezután a két folyamat szétválhat, és a feladataiknak megfelelően eltérően viselkedhetnek.

(Mivel a gyermek a szülőtől örökli az erőforrásokat, a `fork()` rendkívül hatékony ahhoz képest, mintha a nulláról kellene felépíteni egy új folyamatot).

## 2. Hogyan működik a `fork()`?

Amikor a `fork()` lefut, az alábbiak történnek:

- Duplikálja a folyamat memóriáját (mind a szülő, mind a gyermek kap egy másolatot).
- Duplikálja a folyamat erőforrásait (pl. nyitott fájlokat, socketeket).
- Mindkét folyamatot független futásra állítja be.

A két folyamat pontosan ugyanattól a ponttól (a `fork()` hívása utáni sortól) folytatja a futását. Az egyetlen különbség a `fork()` visszatérési értéke, amely alapján a kód el tudja dönteni, hogy éppen melyik folyamatban fut:

- A **szülő** folyamat visszatérési értékként megkapja a **gyermek PID-jét** (Folyamatazonosítóját).
- A **gyermek** folyamat visszatérési értéke **0**.

## 3. Szintaxis és Gyakorlati Példa (C nyelv)

A `fork()` a `<unistd.h>` fejlécfájlban található.

**Szintaxis:** `pid_t fork(void);`

**Példa kód:**

```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
  pid_t pid = fork();

  if (pid == -1) {
    // A fork sikertelen volt
    perror("fork failed");
    exit(1);
  }

  if (pid == 0) {
    // Ez a kód a GYERMEK folyamatban fut
    printf("Helló a gyermek folyamatból!\n"); 
  } else {
    // Ez a kód a SZÜLŐ folyamatban fut  
    printf("Helló a szülő folyamatból! A gyermek PID-je: %d\n", pid);
  }

  return 0;
}
```

## 4. Memóriamegosztás: A "Copy-on-Write" (Másolás íráskor)

Bár úgy tűnik, hogy a `fork()` a teljes memóriát azonnal lemásolja, ez valójában egy optimalizált folyamat, amit **Copy-on-Write (CoW)**-nak hívnak.

A `fork()` után a szülő és a gyermek ugyanazokat a fizikai memórialapokat használja, amíg valamelyikük meg nem próbál módosítani egy adatot (írni a memóriába). A kernel csak ekkor készít tényleges, privát másolatot az adott memóriarészről. Ez a módszer rengeteg felesleges másolást spórol meg, így a `fork()` nagyon gyors.

## 5. Erőforrások Öröklése

A gyermek folyamat az alábbiakat örökli a szülőtől:

- **Nyitott fájlleírók (file descriptors):** fájlok, pipe-ok, socketek. (A fájlpozíció/offset megosztott a szülő és a gyermek között).
- **Szignálkezelők (Signal handlers)** és szignálmaszkok.
- **Aktuális munkakönyvtár (Current working directory).**

**Kivétel:** A szülő által birtokolt zárolások (locks) nem öröklődnek.

## 6. Több folyamat létrehozása

A `fork()` többször is meghívható hierarchikus folyamatfák (process trees) létrehozására:

```c
int main() {
  pid_t pid1 = fork();

  if (pid1 == 0) { 
    // Első gyermek

    pid_t pid2 = fork();

    if (pid2 == 0) {
      // Unoka (Grandchild) 
    } 
  }
}
```

> **Figyelem:** A kontrollálatlan `fork()` hívások (ún. fork bombák) teljesen leterhelhetik és lefagyaszthatják a rendszert.

## 7. Alternatívák a `fork()`-ra

- **`vfork()`:** Nem másolja le a memóriát, hanem teljesen megosztja a szülővel. Nagyon gyors, de veszélyes: a gyermeknek azonnal meg kell hívnia egy `exec()`-et vagy `exit()`-et, különben korrumpálhatja a szülő memóriáját.

- **`clone()`:** Alacsony szintű rendszerhívás. Finomhangolható vele, hogy pontosan mik (memória, fájlok, hálózat) legyenek megosztva a szülő és a gyermek között. A szálak (threads) létrehozására használják.

- **`posix_spawn()`:** Kombinálja a `fork()`-ot és az `exec()`-et egyetlen lépésben. (Létrehoz egy folyamatot, és azonnal elindít benne egy új programot).

## 8. Gyakori Buktatók (Pitfalls)

- **Memória kétszeres felszabadítása (Double freeing):** Mivel a memória megosztott (majd lemásolódik), ha egy dinamikusan lefoglalt memóriát mind a gyermekben, mind a szülőben felszabadítasz (`free`), az hibát okozhat.

- **Végrehajtási sorrend feltételezése:** A kernel osztja ki a processzoridőt. Soha ne feltételezd, hogy a `fork()` után a szülő vagy a gyermek fog előbb lefutni.

- **Megosztott állapot (pl. nyitott fájlok) módosítása:** Ha mindkét folyamat ugyanabba a nyitott fájlba ír szinkronizáció nélkül, az adatok összekeveredhetnek.

(A fenti problémák többsége elkerülhető, ha a gyermek folyamat a `fork()` után röviddel meghívja az `exec()` függvénycsaládot egy teljesen új program betöltésére).

##Forrás
-[https://thelinuxcode.com/fork-system-call-linux/]
-[https://www.man7.org/linux/man-pages/man2/fork.2.html]
-[https://medium.com/@joshuaudayagiri/linux-process-calls-creating-process-using-fork-52a1eac7de8b]
