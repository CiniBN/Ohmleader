# DIY Ohmleader
DIY Ohmleader

Előzmények:
Rendelkezem hálózatra visszatápláló napelemes rendszerrel, amelynek teljesítménye 4 kWp és egy Fronius Symo 4.5-3-M inverter képezi az alapját.
Én még Magyarországon beleestem abba a körbe, akik 10 évig éves szaldó elszámolással kötöttek csatlakozási szerződést a villamosenergia szolgáltatóval. A legutóbbi elszámoláskor (május 12-e) kb. 800 kWh visszatáplálással zárult az egyenleg. Úgy döntöttem, hogy a visszatáplálásom minimálisra fogom szorítani, ez elsősorban életmódváltással és másodsorban egy olyan fogyasztó hálózatba kapcsolását jelenti, amely az aktuális visszatáplált villamos energiát fel tudja használni. Erre egy hőszivattyú tökéletes választás lehet, de sajnos a tesztelések alkalmával a visszatáplált energia változása nem lekövethető egy hőszivattyúval. Nem mondom, hogy nem lehetetlen, de én jobbnak láttam egy fűtőszál szabályozás megvalósítást. A családunk energiafelhasználását figyelemmel kísérve számomra egyébént megdöbbentő módon a HMV készítés energiaigénye vetekedett a fűtésre fordított energia mennyiségével. 

Eszközök:
- HomeWisard P1 Meter (Villamos fogyasztásmérő olvasására; export / import energia)
- Fronius Symo inverter, Datameneger kártyával (napelemekkel)
- Inepro PRO380-Mod 100A MID (fűtőszál fogyasztásmérésére)
- 1 db 3P 20A főkapcssoló
- 1 db 3P 10A B kismegszakító (Pl.: Eaton PL7-B10/3)
- 4P 25A 30mA áramvédő kapcsoló (Pl.: Eaton PF7-25/4/003-A, hibaáram védelemre)
- 3 db HOYMK SSR-25 DA szilárdtest relé (fontos, hogy nullaátmenet triggerrel rendelkezzen!)
- 1 db 4 csatornás optocsatoló izolációs kártya (Pl.: HL-OI-VT-4-N bemenet: 3,3V; kimenet: 24V)
- 1 db 230VAC / 24VDC tápegység
- KinCony KC868-A2v3 ESP32-S3 vezérlőkártya (ezt különálló elemekből is össze lehet rakni)
- 3 db AC-1-es üzemmódban 20A kapcsolni képes mágneskapcsoló (Pl.: Eaton DILMP20)
- 3kW-os 3x230V-os csillagba kapcsol fűtőpatron
- Home Assistant

Nézzük egyenként, mi mire kell:
1. HomeWizard P1 Meter: ez az eszköz szolgáltatja a PID szabályozó visszacsatoló jelét, ez tulajdonképpen bármilyen fogyasztásmérő lehet, nem muszály a szolgáltató mérőjét használni, az általa szolgáltatott pillanatnyi teljesítmény adatot le lehet cserélni az aktuális mérő adatára. Itt ezt a Home Assistanton keresztül kapjuk meg.
2. Inepro PRO380-Mod 100A MID: ez is egy fogyasztásmérő, ez fogja mérni a fűtőpatron által fogyasztott villamos energiát. Ez nem vesz részt a szabályozásban, ez csak tájékoztató adatot küld számunkra.
3. 3P-ú 20A-es főkapcsoló: Ezzel az eszközzel tudjuk feszültség mentesíteni a berendezésünket.
4. 3P-ú B védelmi karakterisztikájú 10A-es kismegszakító: Ez a készülék felel az érintésvédelemért, túláram- és zárlatvédelemért.
5. 30mA-es áramvédő-kapcsoló: ez a jelenleg érvényben lévő magyar szabványokban lévő kiegészítő védelem. Ez nem alakalmas önmagában a villamos védelemre, ez csak a túláram- és zárlatvédelmi készülék melleti kiegészítő hibaáram védelem.
6. HOYMK SSR-25 DA szilárdtest relé: Ez az eszköz fogja végezni a fűtőszál teljesítmény vezérlését. Nagyon fontos, hogy az eszköz nullaátmenet triggerrel rendelkezzen. A triak és a fázishasítás módszer sajnos az inverter H-hídját károsíthatja, így egyáltalán nem ajánlott, sőt kerülendő!
7. 4 csatornás optocsatoló izolációs kártya: Ez az ESP kimeneteit illeszti az SSR-ek részére, elméletileg az ESP-t közvetlenül is rá lehet kötni az SSR-re, de jobbnak láttam egy optikai leválasztást és egy szintillesztést közbeiktatni.
8. Tápegység: az elektronikák és SSR meghajtására ez bármilyen a célnak megfeleő tápegység lehet.
9. KC868-A2v3 vezérlőkártya: ezt különálló elemekből is össze lehet építeni, én kifejezetten olyan eszközt kerestem, ami sz alábbi funkciókkal rendelkezik:
    - ESP32-es kontroller
    - 2db relékimenet
    - 2db leválasztott bemenet
    - vezetékes ethernet port (WiFi a lemezszekrény miatt nem opció)
    - RS-485 kommunikációs felület
    - szabadon használható PWM csapok min. 3 db
10. Mágneskapcsoló: olyan mágneskapcsolót válasszunk, amely AC-1 üzemmódban tudja kapcsolni a fűtőpatronokat. Tehát, ha azt látod, hogy AC-3 25A, az nem biztos, hogy megfelelő lesz!
A mágneskapcsolót vezéreljük, ill. kapcsoljuk le ha rendellenes üzemállapot van. Ez biztonsági kérdés. Szükség van olyan pl. kapilláriscsöves termosztátra, amyelyet a tartály hőmérőhüvelyébe helyezünk és a beállított hőmérséklet elérésekor a mágneskapcsoló által a fűtőpartonokat lekapcsolja a hálózatról.
11. Home Assistant: Ez lesz a megjelenítő felületünk, itt mindenki saját maga létrehozhatja az ESP által szolgáltatott adatokat.
    Második funkciója, hogy egy pár érzékelő értékét is szolgáltatja az ESP számára:
    - P1 mérő adatai
    - HMV tartály hőmérséklete
Ha a tartály hőmérséklet adatai nem álnak rendelkezésre, akkor azokat pl. DS18B20 hőmérővel lehet helyetesíteni, természetesen ebben az esetben az ESP programját módosítani kell.
    - fennmaradó villamos energia (ez a HA-ban kerül leképezésre, egy egyszerű kivonásról van szó. A hálózatba betáplált energiából kivonjuk a hálózatból vételezett energiát, ESP-ban is programozható)
   
Figyelmeztetés!
Jelen projekt 3x230/400V 50Hz TN-S hálózatra készült!
Minden esetben tartsa be az oszágában évrényes szabványokat, jogszabályokat a villamos berendezések tervezésére, kivitelezésére, felülvizsgálatára vonatkozóan!
A szerző semmilyen jogi következményt nem vállal a hibás és nem megfelelő méretezésből és kivitelezésből származó balesetek, tűzesetek miatt!
Minden nemű a villamos hálózatra kapcsolt saját gyártmányú nem minősített berendezés hálózatra kapcsolása az Ön felelősége!

Hardver konfiguráció
 ESP32-S3 és hálózat
 Board: ESP32-S3-DevKitC-1, ESP-IDF frameworkkel
 Hálózat: Ethernet W5500 chip (GPIO42-44, CS:41, interrupt:2, reset:1)
 Statikus IP: 192.168.1.22 - kiküszöböli a Wi-Fi problémákat
 Web szerver: 80-as porton fut

Perifériák
 UART (Modbus): GPIO7 (TX), GPIO15 (RX), 4800 baud
 PWM kimenetek (triak vezérlés?): GPIO5,38,6 (50Hz, invertált)
 Relék: GPIO40 (MK1), GPIO39 (MK2)
  bemenetek: GPIO16 (MK1 állapot), GPIO17 (Engedélyező kapcsoló)

Modbus kommunikáció
 Eszköz: Omero (address 0x0003)
 Olvasott értékek:
  - Pillanatnyi teljesítmény (kW) - 0x5012 (3 fázis + összes)
  - Hatásos villamos energia (kWh) - 0x600C

Szabályozási logika
1. Fűtési feltételek ellenőrzése (power script)
A fűtés csak akkor indul, ha:
 - HMV hőmérséklet < Célhőmérséklet
 - Max hőmérséklet > Aktuális hőmérséklet
 - MK1 bemenet aktív
 - Engedélyező kapcsoló aktív

2. Üzemmódok
Téli üzem (október 15 - március 15)
 - 100% fűtés (98% PWM) minden fázison
 - Napközben (napkelte után)
 - Fennmaradó energia > 50kWh (állítható)

Nyári üzem (március 15 - október 15)
 - PID szabályozás a fogyasztásmérő alapján
 - Cél: -100W (visszatáplálás minimalizálása)
 - 3 fázis azonos PWM jellel

Kézi mód
 - Állítható PWM (0-100%)

3. PID szabályozás részletei
   
        Setpoint: -100W (minimális hálózati betáplálás)
        Mérés: Fogyasztásmérő teljesítménye
        Hiba = -100 - mérés
Paraméterek:
 - Kp: 0-4 (alap: 2.5)
 - Ki: 0-2 (alap: 0.6)
 - Kd: 0-2 (alap: 0.6)

Speciális funkciók:
 - Derivatív szűrés (α=0.15) a zaj csökkentésére
 - Anti-windup: integrál korlátozás ±max_power
 - Rámpa funkció: 2%/sec PWM változás

4. Auto-tune funkció
Automata PID hangolás:
 - Alap teljesítmény mérés
 - 20% PWM lépcső adás
 - Állandósult állapot elérésének mérése
 - Ku (kritikus erősítés) és Tu (kritikus periódus) számítás
 - Ziegler-Nichols módszer: Kp=0.6Ku, Ki=1.2Ku/Tu, Kd=0.075Ku*Tu
 - 3 ciklus átlagolása

5. Adaptív P szabályozás
Dinamikusan módosítja Kp-t:
 - Növeli (×1.05, max 4.0): ha nagy hiba (>200W) és nem csökken
 - Csökkenti (×0.9, min 0.1): ha túllövés van (>50W hiba)

Időzített feladatok
1 másodpercenként:
 - power script végrehajtása (fűtés szabályozás)
5 másodpercenként:
 - MK1 relé vezérlése (biztonsági feltételek)
1 másodpercenként:
 - MK2 relé vezérlése (PWM aktív állapot jelzése)

Biztonsági funkciók
 - Túlmelegedés védelem: Maxt > HMV hőmérséklet ellenőrzés
 - Energia limit: Fennmaradó energia küszöb (ehtr)
 - Időszakos engedélyezés: Dátum- és időablakok
 - Hardver engedélyezés: MK1 és engedélyező kapcsoló
 - PID integrál újraindítás fűtési feltételek megszűnésekor

Adatgyűjtés és monitorozás
Szenzorok:
 - Fogyasztásmérő (3 fázis + összes) - Home Assistantból
 - HMV hőmérséklet - Home Assistantból
 - Modbus teljesítmény adatok
 - Napi energiafogyasztás

Felhasználói vezérlők:
 - Célhőmérséklet (30-65°C): A tartályban elérni kívént hőmérséklet
 - Maximális hőmérséklet (30-75°C): A tartály maximális hőmérséklete
 - PID paraméterek (Kp, Ki, Kd): Nyári fűtési szezonban a PID szabályzó paraméterei
 - Energiahatár (-100 - +100 kWh): Visszatáplált energia minimális értéke. Ezt az értéket minden esetben megtartja a szabályozó. Egész éves visszatáplálás alapján ez az érték fog megmaradni napsötés nélküli napok esetén. Érthetően: ha értéke 50 kWh, akkor ez lesz az az energia mennyiség, ami télen borús idő esetén a ház fogyasztását fedezi.
 - Fűtési időszak dátumai: A fűtési és szabályozási metodika felosztásra került téli és nyári időszakokra. A téli időszakban a szabályzó nem végez szabályozást. A fennmaradó villamos energiából dolgozik az Energiahatár eléréséig 98%-os teljesítménnyel. Nyári időszakban működik a PID szabályozás. (Az entitásnál az "év" mezőt nem figyeljük, csak a "hónap" és "nap" mezők számítanak.)
 - Fűtési idő kezdete: Szintén csak téli fűtési időszakban használatos, adott napon belül a fűtés kezdeti idejét tudjuk megadni.
 - Kézi alapjel: Kézi üzemmódban alapjel beállítása.


Kp, Ki, Kd paraméterek automatkus hangolása (kizárólag nyári üzemmódban):
1. Auto-tune engedélyezése számot állítsuk 1-re.
2. Nyomjuk meg a PID Auto-tune indítása gomgot.
3. A hangolás elindul 3 ciklus erejéig, ekkor a PID szabályozás nem aktív.
4. A hangolás elvégzéséről a Auto-tune aktív bináris szenzor ad visszajelzést. (kb. 30-45 másodperc)
5. A hangolás végeztével a Kp, Ki, Kd paraméterek beíródnak és tárolódnak.
6. Auto-tune engedélyezése számot állítsuk 0-ra. Ekkor elindul a PID szabályozás az új paraméterekkel.

Finomhangolás:
1. A Kp-t automatikusan állítja.
2. Ki beállítása:
 - Növeld lassan az I értékét
Mit csinál?
Megszünteti az állandósult hibát (offset)
Figyelj:
 - Túl nagy I → lengés, instabilitás
👉 Addig növeld, amíg:
 - A hiba eltűnik
 - De a rendszer még stabil marad
3. Kd beállítása:
 - Adj hozzá kis D értéket
Mit csinál?
 - Csökkenti a túllövést
 - Stabilizálja a gyors változásokat
👉 Ha:
Zajos a jel → ne növeld túl a D-t (érzékeny a zajra)


FONTOS!

A Homewisard P1 mérők frissítési időköze 5s. Ez kevés a fenti szabályozáshoz. Egy egyszerű REST hívássall viszont 1s-ként ezt elvégezhetjük.
Ha van sensor.yaml fájlunk a HA-ban, akkor a végére illesszük be az alábbi szenzort:

    # Ohmleaderhez kell 1 s-kénti mérés!
    - platform: rest
      name: "P1 Aktív Teljesítmény"
      resource: "http://<HOMEWISARD_P1_IP_ADDRESS>/api/v1/data"
      method: GET
      value_template: "{{ value_json.active_power_w }}"
      unit_of_measurement: "W"
      scan_interval: 1

A <HOMEWISARD_P1_IP_ADDRESS> helyére a P1 olvasó IP címét kell beírni.

