# Caractéristiques

Klipper propose plusieurs caractéristiques intéressantes :

* Mouvements pas à pas de haute précision. Klipper utilise un processeur d'application (tel qu'un Raspberry Pi à bas prix) pour calculer les mouvements de l'imprimante. Ce processeur d'application détermine le moment où il faut faire marcher chaque moteur pas à pas, compresse ces événements, les transmet au microcontrôleur pour que celui-ci exécute chaque événement au moment demandé. Chaque événement du moteur pas à pas est programmé avec une précision de 25 microsecondes ou mieux. Le logiciel n'utilise pas d'estimations cinématiques (telles que l'algorithme de Bresenham), au lieu de cela, il calcule des durées de pas précises basées sur les physiques de l'accélération et de la cinématique de la machine. Un mouvement plus précis des pas permet un fonctionnement plus silencieux et plus stable de l'imprimante.
* Meilleures performances de sa catégorie. Klipper est capable d'atteindre des taux de pas élevés sur les micro-contrôleurs nouveaux et anciens. Même les anciens microcontrôleurs 8 bits peuvent obtenir des taux supérieurs à 175 000 pas par seconde. Sur les micro-contrôleurs plus récents, plusieurs millions de pas par seconde sont possibles. Des taux de pas plus élevés permettent des vitesses d'impression plus élevées. La synchronisation des événements du pas à pas reste précise même à des vitesses élevées, ce qui améliore la stabilité globale.
* Klipper prend en charge les imprimantes dotées de plusieurs microcontrôleurs. Par exemple, un microcontrôleur peut être utilisé pour contrôler l'extrudeur, tandis qu'un autre contrôle les pièces chauffantes de l'imprimante, et un troisième s'occupe du reste de l'imprimante. Le logiciel Klipper met en œuvre la synchronisation de l'horloge pour tenir compte de la dérive entre les microcontrôleurs. Il n'y a pas besoin de code particulier pour activer plusieurs microcontrôleurs - il suffit de quelques lignes supplémentaires dans le fichier de configuration.
* Configuration grâce à un fichier de configuration unique. Il n'est pas nécessaire de reflasher le microcontrôleur pour modifier un paramètre. Toute la configuration de Klipper est stockée dans un fichier de configuration standard qui peut être facilement modifié. Cela facilite la configuration et la maintenance du matériel.
* Klipper prend en charge la fonction "Smooth Pressure Advance", un mécanisme permettant de prendre en compte les effets de la pression dans un extrudeur. Cela réduit le "suintement" de l'extrudeur et améliore la qualité des d'impression des coins. L'implémentation de Klipper n'introduit pas de changements instantanés de la vitesse de l'extrudeur, ce qui améliore la stabilité et la robustesse générales.
* Klipper prend en charge la fonction "Input Shaping" pour réduire l'impact des vibrations sur la qualité d'impression. Cela peut réduire ou éliminer le "ringing" (également appelé "ghosting", "echoing" ou "rippling") des impressions. Cela peut également permettre d'obtenir des vitesses d'impression plus rapides tout en maintenant une qualité d'impression élevée.
* Klipper utilise un "solveur itératif" pour calculer des temps de pas précis à partir d'équations cinématiques simples. Cela facilite le portage de Klipper sur de nouveaux types de robots et permet de conserver un timing précis même avec une cinématique complexe (aucune "segmentation de ligne" n'est nécessaire).
* Klipper est agnostique vis-à-vis du matériel. On doit obtenir le même timing précis indépendamment du matériel électronique de bas niveau. Le code du microcontrôleur de Klipper est conçu pour suivre fidèlement l’ordonnancement fourni par le logiciel hôte de Klipper (ou pour alerter l'utilisateur de manière évidente s'il n'y parvient pas). Il est ainsi plus facile d'utiliser le matériel disponible, de mettre à niveau vers un nouveau matériel et d'être confiant dans ce matériel.
* Code portable. Klipper fonctionne sur les micro-contrôleurs basés sur ARM, AVR et PRU. Les imprimantes existantes de type "reprap" peuvent utiliser Klipper sans modification matérielle - il suffit d'ajouter un Raspberry Pi. La conception interne du code de Klipper facilite le support d'autres architectures de micro-contrôleurs.
* Un code plus simple. Klipper utilise un langage de très haut niveau (Python) pour la plupart du code. Les algorithmes cinématiques, l'analyse du code G, les algorithmes de chauffage et de thermistance, etc. sont tous écrits en Python. Il est donc plus facile de développer de nouvelles fonctionnalités.
* Macros programmables personnalisées. De nouvelles commandes G-Code peuvent être définies dans le fichier de configuration de l'imprimante (aucune modification du code n'est alors nécessaire). Ces commandes sont programmables - ce qui leur permet de produire différentes actions selon l'état de l'imprimante.
* Serveur d’API intégré. En plus de l’interface G-Code standard, Klipper prend en charge une interface d’application en JSON. Cela permet aux programmeurs de créer des applications externes avec un contrôle précis de l’imprimante.

## Caractéristiques supplémentaires

Klipper prend en charge de nombreuses fonctionnalités standard des imprimantes 3d :

* Plusieurs interfaces web disponibles. Fonctionne avec Mainsail, Fluidd, OctoPrint et d'autres. Cela permet de contrôler l'imprimante à l'aide d'un navigateur web ordinaire. Le même Raspberry Pi qui fait fonctionner Klipper peut également faire fonctionner l'interface web.
* Prise en charge du G-Code standard. Les commandes de g-code courantes produites par les "trancheurs" (slicers) typiques (SuperSlicer, Cura, PrusaSlicer, etc.) sont prises en charge.
* Prise en charge de l'extrusion multiple. Les extrudeurs avec réchauffeurs partagés et les extrudeurs sur chariots indépendants (IDEX) sont également prises en charge.
* Prise en charge des imprimantes cartésiennes, delta, corexy, corexz, hybrides-corexy, hybrides-corexz, deltesiennes, rotatives delta, polaires et à treuil.
* Support du nivellement automatique du bed. Klipper peut être configuré pour une détection de base de l'inclinaison du bed ou pour une mise à niveau complète de celui-ci. Si le bed utilise plusieurs steppers Z, Klipper peut également le mettre à niveau en manipulant indépendamment les steppers Z. La plupart des capteurs de hauteur Z sont prises en charge, y compris les sondes BL-Touch et les sondes activées par servomoteur.
* Prise en charge du calibrage delta automatique. L'outil d'étalonnage peut effectuer un étalonnage de base de la hauteur ainsi qu'un avancé des dimensions X et Y. L'étalonnage peut être effectué avec un palpeur de l'axe Z ou manuellement.
* Prise en charge de l'option "exclure un objet" lors de l'impression. Lorsqu'il est configuré, ce module peut faciliter l'annulation d'un seul objet dans une impression en plusieurs parties.
* Prise en charge des capteurs de température courants (par exemple, les thermistances courantes, AD595, AD597, AD849x, PT100, PT1000, MAX6675, MAX31855, MAX31856, MAX31865, BME280, HTU21D, DS18B20 et LM75). Des thermistances et des capteurs de température analogiques personnalisés peuvent également être configurés. On peut surveiller le capteur de température interne du microcontrôleur et le capteur de température interne d'un Raspberry Pi.
* Protection thermique basique de l'appareil activée par défaut.
* Prise en charge des ventilateurs standard, des ventilateurs de buses et des ventilateurs à température contrôlée. Il n'est pas nécessaire de faire tourner les ventilateurs lorsque l'imprimante est inactive. La vitesse du ventilateur peut être contrôlée sur les ventilateurs dotés d'un tachymètre.
* Prise en charge de la configuration en temps réel des pilotes de moteurs pas à pas TMC2130, TMC2208/TMC2224, TMC2209, TMC2660 et TMC5160. Il existe également un support pour le contrôle du courant des pilotes de moteurs pas à pas traditionnels via AD5206, DAC084S085, MCP4451, MCP4728, MCP4018, et les broches PWM.
* Prise en charge des écrans LCD courants fixés directement à l'imprimante. Un menu par défaut est également disponible. Le contenu de l'écran et du menu peut être entièrement personnalisé via le fichier de configuration.
* Accélération constante et prise en charge du "look-ahead". Tous les mouvements de l'imprimante s'accélèrent progressivement de l'arrêt à la vitesse de croisière, puis décélèrent pour revenir à l'arrêt. Le flux entrant de commandes de mouvement en G-Code est mis en file d'attente et analysé - l'accélération entre les mouvements dans une direction similaire sera optimisée pour réduire les blocages d'impression et améliorer le temps d'impression global.
* Klipper met en œuvre un algorithme de "fin de phase pas à pas" qui peut améliorer la précision des interrupteurs de butée. Lorsqu'il est correctement réglé, il peut améliorer l'adhérence de la première couche d'une impression.
* Prise en charge des capteurs de présence de filaments, des capteurs de mouvement de filaments et des capteurs de largeur de filaments.
* Prise en charge de la mesure et l'enregistrement de l'accélération à l'aide d'un accéléromètre adxl345, mpu9250 et mpu6050.
* Prise en charge de la limitation de la vitesse maximale des mouvements courts en "zigzag" pour réduire les vibrations et le bruit de l'imprimante. Voir le document [Cinématiques](Kinematics.md) pour plus d'informations.
* Des exemples de fichiers de configuration sont disponibles pour de nombreuses imprimantes courantes. Consultez le [répertoire des configurations](../config/) pour en obtenir la liste.

Pour commencer avec Klipper, lisez le guide d'[installation](Installation.md).

## Tests de performance du stepper

Ci-après les résultats des tests de performance du stepper. Les chiffres indiqués représentent le nombre total de pas par seconde sur le micro-contrôleur.

| Microcontrôleur | 1 stepper actif | 3 steppers actifs |
| --- | --- | --- |
| AVR 16Mhz | 157K | 99K |
| AVR 20Mhz | 196K | 123K |
| SAMD21 | 686K | 471K |
| STM32F042 | 814K | 578K |
| PRU Beaglebone | 866K | 708K |
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

Si vous n'êtes pas sûr du microcontrôleur d'une carte particulière, trouvez le [fichier de configuration](../config/) approprié , et cherchez le nom du microcontrôleur dans les commentaires en haut de ce fichier.

De plus amples détails sur les bancs d'essais sont disponibles dans le [document des bancs d'essais](Benchmarks.md).
