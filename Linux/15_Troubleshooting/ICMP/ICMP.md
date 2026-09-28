# Az ICMP-csomagok megértése – Példákkal

**Szerző:** David Cao
**Dátum:** 2025. október 9.
**Kategória:** Linux

Az Internet Control Message Protocol (ICMP), amelyet a RFC 792 ír le, és a TCP/IP protokollverem része, egy hibajelentő és vezérlésalapú protokoll, amelyet hálózati eszközök között használnak.

Az ICMP-üzenetek IP-datagramokba vannak beágyazva, amelyek aztán Ethernet-keretbe vannak csomagolva.

## Tartalomjegyzék

- Mi az ICMP?
- ICMP-csomag szerkezete – ICMP fejléc
- ICMP-típusok és -kódok
- ICMP-csomagok rögzítése a `tcpdump` paranccsal

## Mi az ICMP?

Az Internet Control Message Protocol (ICMP) a TCP/IP segédprotokollja, amely felelős az eszközök, szolgáltatások vagy útvonalak elérhetőségével kapcsolatos információk biztosításáért egy TCP/IP-hálózaton. A legtöbb hálózati hibaelhárítási technika és eszköz a gyakori ICMP-üzenettípusok köré összpontosul.

Az ICMP ping elsődleges célja a kommunikáció tesztelése az eszközök között. Az adatokat az egyik hoszt küldi a másiknak kérésként, és a fogadó hosztnak vissza kell küldenie ezeket az adatokat válaszként.

## ICMP-csomag szerkezete – ICMP fejléc

Az ICMP az IP része, és az IP-re támaszkodik üzenetei továbbításához. Az ICMP viszonylag kis fejléccel rendelkezik, amely a céltól függően változik. Az ICMP-fejléc a következő mezőket tartalmazza:

> **Lásd még:** A Linux parancssor elsajátítása – Teljes ingyenes képzési útmutató

- **Type (Típus)** – Az ICMP-üzenet típusa vagy osztályozása, az RFC-specifikáció alapján
- **Code (Kód)** – Az ICMP-üzenet alkategóriája, az RFC-specifikáció alapján
- **Checksum (Ellenőrzőösszeg)** – Az ICMP-fejléc és az adatok tartalmának sértetlenségének biztosítására szolgál a megérkezéskor
- **Variable (Változó)** – Egy rész, amely a Type és Code mezőktől függően változik

## ICMP-típusok és -kódok

Az ICMP-protokollnak van egy `type` nevű mezője, amely jelzi, hogy milyen típusú az ICMP-csomag. Ha a type mező értéke 8, akkor a csomag egy ICMP echo (ping) kérés, míg ha a type mező értéke 0, akkor a csomag egy ICMP echo (ping) válasz.

Ez a type mező egy egybájtos mező az ICMP-protokoll fejlécének legelején.

Az ICMP Type mezőt tekinthetjük a csomag osztályozásának, a Code mezőt pedig az alkategóriájának. Például egy 3-as Type mezőérték a „destination unreachable” (cél elérhetetlen) állapotot jelzi.

Bár ez az információ önmagában talán nem elég egy probléma elhárításához, ha az a csomag egy 3-as Code mezőértéket is megadna, ami a „port unreachable” (port elérhetetlen) állapotot jelzi, arra következtethetnénk, hogy probléma van azzal a porttal, amellyel kommunikálni próbálunk.

## ICMP-csomagok rögzítése a `tcpdump` paranccsal

IPv4-ben ezzel a `tcpdump` paranccsal szűrhetjük az összes ICMP-csomagot. Minden példánkban az `eth0` hálózati interfészt használjuk. Kérjük, változtasd meg a környezetnek megfelelően.

```bash
# tcpdump -i eth0 icmp
```

Az ICMP echo-kérések szűréséhez ezt a `tcpdump` parancsot használhatjuk.

```bash
# tcpdump -i eth0 "icmp[0] == 8"
```

Ezek azok a csomagok, amelyeket a `tcpdump` paranccsal rögzítünk.

```plaintext
14:37:14.555295 IP 10.79.101.23 > 108.177.125.101: ICMP echo request, id 61205, seq 0, length 64
14:37:15.557948 IP 10.79.101.23 > 108.177.125.101: ICMP echo request, id 61205, seq 1, length 64
14:37:16.562905 IP 10.79.101.23 > 108.177.125.101: ICMP echo request, id 61205, seq 2, length 64
```
## Forrás
-[https://www.howtouselinux.com/post/icmp-packets]
-[https://www.man7.org/linux/man-pages/man7/icmp.7.html]
