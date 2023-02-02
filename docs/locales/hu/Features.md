# Funkciók

A Klipper számos lenyűgöző tulajdonsággal rendelkezik:

* High precision stepper movement. Klipper utilizes an application processor (such as a low-cost Raspberry Pi) when calculating printer movements. The application processor determines when to step each stepper motor, it compresses those events, transmits them to the micro-controller, and then the micro-controller executes each event at the requested time. Each stepper event is scheduled with a precision of 25 micro-seconds or better. The software does not use kinematic estimations (such as the Bresenham algorithm) - instead it calculates precise step times based on the physics of acceleration and the physics of the machine kinematics. More precise stepper movement provides quieter and more stable printer operation.
* Kategóriájában legjobb teljesítmény. A Klipper képes magas léptetési sebességet elérni mind az új, mind a régi mikrokontrollereken. Még a régi 8 bites mikrovezérlők is képesek 175 000 lépés/másodperc feletti sebességet elérni. Az újabb mikrokontrollereken másodpercenként több millió lépés is lehetséges. A nagyobb léptetési sebesség nagyobb nyomtatási sebességet tesz lehetővé. A léptetések időzítése még nagy sebességnél is pontos marad, ami javítja az általános stabilitást.
* A Klipper támogatja a több mikrovezérlővel rendelkező nyomtatókat. Például egy mikrokontroller használható az extruder vezérlésére, míg egy másik a nyomtató fűtőberendezését, míg egy harmadik a nyomtató többi részét vezérli. A Klipper gazdaszoftver órajel-szinkronizációt valósít meg a mikrovezérlők közötti órajel-eltolódás figyelembevétele érdekében. A több mikrovezérlő engedélyezéséhez nincs szükség külön kódra, csak néhány extra sorra a konfigurációs fájlban.
* Konfiguráció egyszerű konfigurációs fájlon keresztül. Nincs szükség a mikrokontroller újrafrissítésére a beállítások megváltoztatásához. Az összes Klipper konfiguráció egy szabványos konfigurációs fájlban van tárolva, amely könnyen szerkeszthető. Ez megkönnyíti a hardver beállítását és karbantartását.
* A Klipper támogatja a "Smooth Pressure Advance" - egy olyan mechanizmust, amely figyelembe veszi a nyomást az extruderben. Ez csökkenti az extruder "szivárgását" és javítja a nyomtatási sarkok minőségét. A Klipper beavatkozása nem vezet be pillanatnyi extruder sebességváltozást, ami javítja az általános stabilitást és robusztusságot.
* A Klipper támogatja az "Input Shaping" funkciót a rezgések nyomtatási minőségre gyakorolt hatásának csökkentése érdekében. Ez csökkentheti vagy kiküszöbölheti a "gyűrődést" (más néven "szellemkép", "visszhang" vagy "hullámzás") a nyomatokon. Lehetővé teheti a gyorsabb nyomtatási sebesség elérését is, miközben a nyomtatás minősége továbbra is magas marad.
* A Klipper egy "interaktív megoldást" használ a pontos lépésidők kiszámításához egyszerű kinematikai egyenletekből. Ez megkönnyíti a Klipper átültetését új típusú robotokra, és az időzítést még összetett kinematika esetén is pontosan tartja (nincs szükség "vonalszegmentálásra").
* Klipper is hardware agnostic. One should get the same precise timing independent of the low-level electronics hardware. The Klipper micro-controller code is designed to faithfully follow the schedule provided by the Klipper host software (or prominently alert the user if it is unable to). This makes it easier to use available hardware, to upgrade to new hardware, and to have confidence in the hardware.
* Átvihető kód. A Klipper ARM, AVR és PRU alapú mikrovezérlőkön is működik. A meglévő "RepRap" stílusú nyomtatók hardveres módosítás nélkül futtathatják a Klippert. Csak egy Raspberry Pi-t kell hozzáadni. A Klipper belső kódelrendezése megkönnyíti más mikrokontroller-architektúrák támogatását is.
* Egyszerűbb kód. A Klipper egy nagyon magas szintű nyelvet (Python) használ a legtöbb kódhoz. A kinematikai algoritmusok, a G-kód elemzése, a fűtési és termisztor algoritmusok stb. mind Pythonban íródnak. Ez megkönnyíti az új funkciók fejlesztését is.
* Egyéni programozható makrók. Új G-kód parancsok definiálhatók a nyomtató konfigurációs fájljában (nincs szükség kódmódosításra). Ezek a parancsok programozhatók lehetővé téve, hogy a nyomtató állapotától függően különböző műveleteket hajtsanak végre.
* Beépített API-kiszolgáló. A Klipper a szabványos G-kódos interfész mellett egy gazdag JSON-alapú alkalmazási felületet is támogat. Ez lehetővé teszi a programozók számára, hogy külső alkalmazásokat készítsenek a nyomtató részletes vezérlésével.

## További funkciók

A Klipper számos szabványos 3D nyomtató funkciót támogat:

* Several web interfaces available. Works with Mainsail, Fluidd, OctoPrint and others. This allows the printer to be controlled using a regular web-browser. The same Raspberry Pi that runs Klipper can also run the web interface.
* Standard G-kód támogatás. A tipikus "szeletelők" (SuperSlicer, Cura, PrusaSlicer, stb.) által előállított általános G-kód parancsok támogatottak.
* Több extruder támogatása. A közös fűtőberendezéssel rendelkező extrudereket és a független kocsikon (IDEX) lévő extrudereket is támogatják.
* Support for cartesian, delta, corexy, corexz, hybrid-corexy, hybrid-corexz, deltesian, rotary delta, polar, and cable winch style printers.
* Tárgyasztal szintezésének automatikus támogatása. A Klipper konfigurálható alapszintű tárgyasztal dőlés-érzékelésre vagy a tárgyasztal teljes hálós szintezésére. Ha a tárgyasztal több Z steppert használ, akkor a Klipper a Z stepper-ek független manipulálásával is képes szintezni. A legtöbb Z magasságmérő szonda támogatott, beleértve a BL-Touch szondákat és a szervómotoros szondákat is.
* Automatikus delta kalibráció támogatása. A kalibráló eszköz alapvető magassági kalibrálást, valamint továbbfejlesztett X és Y dimenzió kalibrálást végezhet. A kalibrálás elvégezhető Z magasságmérővel vagy kézi szintezővel.
* Run-time "exclude object" support. When configured, this module may facilitate canceling of just one object in a multi-part print.
* Az általános hőmérséklet-érzékelők támogatása (pl. általános termisztorok, AD595, AD597, AD849x, PT100, PT1000, MAX6675, MAX31855, MAX31856, MAX31865, BME280, HTU21D, DS18B20 és LM75). Egyedi termisztorok és egyedi analóg hőmérséklet-érzékelők is konfigurálhatók. Lehet figyelni a mikrokontroller hőmérsékletét és a Raspberry Pi processzor hőmérsékletét.
* Alapértelmezés szerint a fűtésvédelem engedélyezett.
* Standard ventilátorok, fejhűtő ventilátorok és hőmérséklet-szabályozott ventilátorok támogatása. Nincs szükség arra, hogy a ventilátorok folyamatosan működjenek, amikor a nyomtató üresjáratban van. A fordulatszámmérővel ellátott ventilátoroknál a ventilátorok fordulatszáma ellenőrizhető.
* Support for run-time configuration of TMC2130, TMC2208/TMC2224, TMC2209, TMC2660, and TMC5160 stepper motor drivers. There is also support for current control of traditional stepper drivers via AD5206, DAC084S085, MCP4451, MCP4728, MCP4018, and PWM pins.
* Közvetlenül a nyomtatóhoz csatlakoztatott általános LCD-kijelzők támogatása. Egy alapértelmezett menü is rendelkezésre áll. A kijelző és a menü tartalma a konfigurációs fájlon keresztül teljesen testreszabható.
* Állandó gyorsulás és "look-ahead" támogatás. Minden mozgás fokozatosan gyorsul fel álló helyzetből utazósebességre, majd lassul vissza álló helyzetbe. A beérkező G-kódos mozgásparancsok sorba kerülnek és elemzi őket - a hasonló irányú mozgások közötti gyorsulás optimalizálva lesz a nyomtatási hibák csökkentése és a teljes nyomtatási idő javítása érdekében.
* A Klipper egy olyan "léptetőfázis végállás" algoritmust valósít meg, amely javíthatja a tipikus végálláskapcsolók pontosságát. Megfelelő beállítás esetén javíthatja a nyomtatás első réteg tárgyasztalhoz tapadását.
* Száljelenlét-, szálmozgás- és szálszélesség-érzékelők támogatása.
* Support for measuring and recording acceleration using an adxl345, mpu9250, and mpu6050 accelerometers.
* A nyomtató rezgésének és zajának csökkentése érdekében a rövid "cikcakk" mozgások csúcssebességének korlátozásának támogatása. További információkért lásd a [Kinematika](Kinematics.md) dokumentumot.
* Számos gyakori nyomtatóhoz rendelkezésre állnak minta konfigurációs fájlok. Listát a [config könyvtárban](../config/) találod.

A Klipper használata előtt olvasd el a [telepítési](Installation.md) útmutatót.

## Lépés Teljesítményérték

Az alábbiakban a léptető teljesítménytesztek eredményeit mutatjuk be. A feltüntetett számok a mikrokontroller másodpercenkénti összes lépésszámát jelentik.

| Mikrokontroller | 1 aktív léptető | 3 aktív léptető |
| --- | --- | --- |
| 16Mhz AVR | 157K | 99K |
| 20Mhz AVR | 196K | 123K |
| SAMD21 | 686K | 471K |
| STM32F042 | 814K | 578K |
| Beaglebone PRU | 866K | 708K |
| STM32G0B1 | 1103K | 790K |
| STM32F103 | 1180K | 818K |
| SAM3X8E | 1273K | 981K |
| SAM4S8C | 1690K | 1385K |
| LPC1768 | 1923K | 1351K |
| LPC1769 | 2353K | 1622K |
| RP2040 | 2400K | 1636K |
| SAM4E8E | 2500K | 1674K |
| SAMD51 | 3077K | 1885K |
| STM32F407 | 3652K | 2459K |
| STM32F446 | 3913K | 2634K |
| STM32H743 | 9091K | 6061K |

Ha nem tudod, hogy egy adott lapon milyen mikrokontroller van, keresd meg a megfelelő [config fájlt](../config/), és keresd meg a mikrokontroller nevét a fájl tetején lévő megjegyzésekben.

A referenciaértékekkel kapcsolatos további részletek a [Referenciaértékek dokumentumban](Benchmarks.md) találhatók.
