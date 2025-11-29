# DIY Ohmleader
DIY Ohmleader

Előzmények:
Rendelkezem hálózatra visszatápláló napelemes rendszerrel, amelynek teljesítménye 4 kWp és egy Fronius Symo 4.5-3-M inverter képezi az alapját.
Én még Magyarországon beleestem abba a körbe, akik 10 évig éves szaldó elszámolással kötöttek csatlakozási szerződést a villamosenergia szolgáltatóval. A legutóbbi elszámoláskor (május 12-e) kb. 800 kWh visszatáplálással zárult az egyenleg és mint tudjuk kishazánkban 5 Ft/kWh áron vásárolta vissza a szolgáltató ezt a megtermelt energiát. (A szomszédnak, meg el tudja adni extrém esetben 70 Ft/kWh áron...) Úgy döntöttem, hogy a visszatáplálásom minimálisra fogom szorítani, ez elsősorban életmódváltással és másodsorban egy olyan fogyasztó hálózatba kapcsolását jelenti, amely az aktuális visszatáplált villamos energiát fel tudja használni. Erre egy hőszivattyú tökéletes választás lehet, de sajnos a tesztelések alkalmával a visszatáplált energia változása nem lekövethető egy hőszivattyúval. Nem mondom, hogy nem lehetetlen, de én jobbnak láttam egy fűtőszál szabályozás megvalósítást. A családunk energiafelhasználását figyelemmel kísérve számomra egyébént megdöbbentő módon a HMV készítés energiaigénye vetekedett a fűtésre fordított energia mennyiségével. 

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
10. Mágneskapcsolók: olyan mágneskapcsolót válasszunk, amely AC-1 üzemmódban tudja kapcsolni a fűtőpatronokat. Tehát, ha azt látod, hogy AC-3 25A, az nem biztos, hogy megfelelő lesz!
Az első mágneskapcsoló a fő mágneskapcsoló, ezt vezéreljük, ill. kapcsoljuk le ha rendellenes üzemállapot van. Ez biztonsági kérdés. Szükség van olyan pl. kapilláriscsöves termosztátra, amyelyet a tartály hőmérőhüvelyébe helyezünk és a beállított hőmérséklet elérésekor a mágneskapcsoló által a fűtőpartonokat lekapcsolja a hálózatról.
A másik két mágneskapcsoló a HMV és puffertartályban lévő patronokat kapcsolja az SSR-k után. Ezek felelnek a patronok kiválasztásáért.
11. Home Assistant: Ez lesz a megjelenítő felületünk, itt mindenki saját maga létrehozhatja az ESP által szolgáltatott adatokat.
    Második funkciója, hogy egy pár érzékelő értékét is szolgáltatja az ESP számára:
    - P1 mérő adatai
    - HMV tartály hőmérséklete
    - Puffer tartály hőmérséklete
Ha a tartály hőmérséklet adatai nem álnak rendelkezésre, akkor azokat pl. DS18B20 hőmérővel lehet helyetesíteni, természetesen ebben az esetben az ESP programját módosítani kell.
    - fennmaradó villamos energia (ez a HA-ban kerül leképezésre, egy egyszerű kivonásról van szó. A hálózatba betáplált energiából kivonjuk a hálózatból vételezett energiát, ESP-ban is programozható)
   
Figyelmeztetés!
Jelen projekt 3x230/400V 50Hz TN-S hálózatra készült!
Minden esetben tartsa be az oszágában évrényes szabványokat, jogszabályokat a villamos berendezések tervezésére, kivitelezésére, felülvizsgálatára vonatkozóan!
A szerző semmilyen jogi következményt nem vállal a hibás és nem megfelelő méretezésből és kivitelezésből származó balesetek, tűzesetek miatt!
Minden nemű a villamos hálózatra kapcsolt saját gyártmányú nem minősített berendezés hálózatra kapcsolása az Ön felelősége!

🔧 Főbb funkciók és működési módok
1. Kapcsolódás és alap infrastruktúra

ESP32-S3 vezérlő, Ethernet (W5500) kommunikációval.

Modbus RTU: egy „Omero” nevű eszközről energiamérés.

Home Assistant integráció: több külső szenzort figyel.

Webszerver + API + OTA: távoli menedzsment.

Sun + SNTP: idő és napszak meghatározása.

🌡️ Szabályozás — Áttekintés

A rendszer akkor kezd fűteni, ha mindhárom feltétel teljesül:

A HMV tartály hőmérséklete < célhőmérséklet

A HMV hőmérséklete < maximális hőmérséklet

MK1 bemenet aktív (valamilyen külső engedély vagy kontaktor visszajelzés)

Ha nem teljesülnek → minden PWM 0, integrátorok lenullázva.

🕹️ Üzemmód választó
Működés mód (mukodes)

Auto

Kézi

Fázis mód (mukodes_fazis)

Egy

Három

Ennek megfelelően választ:

1 fázis → három PWM kimenet külön PID-del

3 fázis → három kimenet egyszerre, 3-fázisú PID-del

🍂 / ☀️ Szezonfüggő engedélyezés
Őszi–tavaszi üzem (futas_engedelyezett_ev)

A fűtés akkor engedélyezett, ha:

szeptember 15. – december 31. vagy

január 1. – május 15.

fennmaradó villamos energia: > 0

nappal van (nap felett a horizonton)

Ez a TÉLI üzemhez használatos.

Nyári üzem (futas_engedelyezett_nyar)

május 15. – szeptember 15.

fennmaradó villamos energia: > 0

Ez a PID-es NYÁRI üzemhez használatos.

❄️ TÉLI üzem (Auto mód)

Ha őszi–tavaszi időszak van és Auto mód:

1 fázis

PWM mindhárom fázison 95%.

3 fázis

Ugyanúgy: minden fázison 95%.

👉 Tehát a téli üzem nem szabályoz, hanem fix intenzitással fűt, amíg engedélyezve van.

☀️ NYÁRI PID-szabályozás (Auto mód)

Csak ha:

nyári időszak

Auto mód

van fennmaradó energia

A cél: ne legyen pozitív fogyasztás, azaz a ház vagy nulla energiát vesz fel, vagy termel.

Egyfázisú NYÁRI PID

Mindhárom fázist külön PID szabályozza:

setpoint = −100 W

ház aktuális fogyasztása (L1, L2, L3) → PID

a PID kimenet → PWM érték

A három PID teljesen külön dolgozik.

Háromfázisú NYÁRI PID

setpoint = −100 W

a teljes háromfázisú fogyasztás (fmw) alapján egyetlen PID számol PWM-et

ugyanaz a PWM megy mindhárom kimenetre

✋ KÉZI üzem

Ha nem „Auto”:

kcel százalékos értéke → PWM (0–100%)

mindhárom fázis ugyanazt a PWM-et kapja

a szezon, fogyasztás, napszak nem számít, csak a hőmérséklet és MK1 bemenet

🔋 Döntési logika összefoglaló
Feltételek	Mit csinál
Nincs szükség fűtésre	PWM = 0, integrátorok reset
Auto + téli szezon	1F: 95% PWM, 3F: 95% PWM
Auto + nyári szezon	PID szabályozás (1F külön PID, 3F egy PID)
Kézi mód	PWM = kcel (%) minden fázison
Biztonsági fallback	PWM=0, ha nem illik egyik feltételre sem
⚡ Kimenetek
PWM:

pwm_output1 → GPIO5

pwm_output2 → GPIO38

pwm_output3 → GPIO6
50 Hz, 98% max, invertált.

Relék:

rele1 → tartályhőmérséklet és státusz szenzor alapján

rele2 szabadon

📊 Szenzorok
Modbus:

teljesítmény (össz + L1/L2/L3)

összes energia

Home Assistant szenzorok:

3 fázis fogyasztás

HMV hőmérséklet

fennmaradó energia

🧠 PID struktúra

A program több külön PID integrátort tart fenn:

L1, L2, L3 (egyfázisú üzemhez)

3f (összfázisú PID)

Mindegyik rendelkezik:

error

integral

prev_error

Integrátor limitált (±1000, ±3000), nehogy elszálljon.
