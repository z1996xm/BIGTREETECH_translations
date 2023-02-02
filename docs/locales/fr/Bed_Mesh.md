# Maillage du Bed

Le module Maillage du lit peut être utilisé pour compenser les irrégularités de la surface du lit afin d'obtenir une meilleure première couche sur l'ensemble du lit. Il convient de noter que la correction logicielle ne permet pas d'obtenir de résultats parfaits, elle ne peut qu'approximer la forme du lit. Le maillage du lit ne peut pas non plus compenser les problèmes mécaniques et électriques. Si un axe est de travers ou si un palpeur n'est pas précis, le module bed_mesh n'obtiendra pas de résultats précis lors du processus de palpage.

Avant de procéder à l'étalonnage du maillage, vous devez vous assurer que l'offset Z de votre sonde est réglé. Si vous utilisez une butée de fin de course pour la mise à l'origine en Z, elle doit également être réglée. Voir [Calibration de la sonde](Probe_Calibrate.md) et Z_ENDSTOP_CALIBRATE dans [Nivelage manuel](Manual_Level.md) pour plus d'informations.

## Configuration de base

### Lits rectangulaires

Cet exemple suppose une imprimante avec un lit rectangulaire de 250 mm x 220 mm et une sonde avec un décalage x de 24 mm et un décalage y de 5 mm.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
```

- `speed : 120` *Valeur par défaut : 50* La vitesse à laquelle l'outil se déplace entre les points palpés.
- `horizontal_move_z : 5` *Valeur par défaut : 5* La coordonnée Z à laquelle la sonde s'élève avant de se déplacer entre les points.
- `mesh_min : 35, 6` *Requis* La première coordonnée palpée, la plus proche de l'origine. Cette coordonnée est relative à l'emplacement de la sonde.
- `mesh_max : 240, 198` *Requis* La coordonnée palpée la plus éloignée de l'origine. Ce n'est pas nécessairement le dernier point palpé, car le processus de palpage se déroule en zig-zag. Comme pour `mesh_min`, cette coordonnée est relative à l'emplacement de la sonde.
- `probe_count : 5, 3` *Valeur par défaut : 3, 3* Le nombre de points à palper sur chaque axe, spécifié sous forme de valeurs entières X, Y. Dans cet exemple, 5 points seront palpés le long de l'axe X, avec 3 points le long de l'axe Y, pour un total de 15 points palpés. Notez que si vous voulez une grille carrée, par exemple 3x3, il est possible de n'utiliser qu'une seule valeur entière pour les deux axes, par exemple `probe_count : 3`. Notez qu'un maillage nécessite un nombre minimum de 3 points de sondage sur chaque axe.

L'illustration ci-dessous montre comment les options `mesh_min`, `mesh_max`, et `probe_count` sont utilisées pour générer des points de palpage. Les flèches indiquent la direction de la procédure de palpage, commençant en `mesh_min`. Pour référence, lorsque la sonde est à `mesh_min`, la buse sera à (11, 1), et lorsque la sonde est à `mesh_max`, la buse sera à (206, 193).

![bedmesh_rect_basic](img/bedmesh_rect_basic.svg)

### Lits circulaires

Cet exemple suppose une imprimante équipée d'un lit circulaire de 100 mm de rayon. Nous utiliserons les mêmes décalages de sonde que dans l'exemple rectangulaire, 24 mm sur X et 5 mm sur Y.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_radius: 75
mesh_origin: 0, 0
round_probe_count: 5
```

- `mesh_radius : 75` *Requis* Le rayon du maillage palpé en mm, par rapport à `mesh_origin`. Notez que les décalages de la sonde limitent la taille du rayon du maillage. Dans cet exemple, un rayon supérieur à 76 déplacerait la tête de l'outil en dehors des limites physiques de l'imprimante.
- `mesh_origin : 0, 0` *Valeur par défaut : 0, 0* Le point central du maillage. Cette coordonnée est relative à l'emplacement de la sonde. Bien que la valeur par défaut soit 0, 0, il peut être utile d'ajuster l'origine dans le but de sonder une plus grande partie du lit. Voir l'illustration ci-dessous.
- `round_probe_count : 5` *Valeur par défaut : 5* C'est une valeur entière définissant le nombre maximum de points palpés le long des axes X et Y. Par "maximum", nous entendons le nombre de points palpés le long de l'origine du maillage. Cette valeur doit être un nombre impair, car il est nécessaire que le centre du maillage soit palpé.

L'illustration ci-dessous montre comment les points palpés sont générés. Comme vous pouvez le voir, le réglage de `mesh_origin` à (-10, 0) nous permet de spécifier un rayon de maillage plus grand de 85.

![bedmesh_round_basic](img/bedmesh_round_basic.svg)

## Configuration avancée

Les options de configuration plus avancées sont expliquées en détail ci-dessous. Chaque exemple s'appuie sur la configuration de base du lit rectangulaire présentée ci-dessus. Chacune des options avancées s'applique de la même manière aux lits circulaires.

### Interpolation du maillage

Bien qu'il soit possible d'échantillonner directement la matrice palpée en utilisant une simple interpolation bilinéaire afin de déterminer les valeurs Z entre les points palpés, il est souvent utile d'interpoler des points supplémentaires en utilisant des algorithmes d'interpolation plus avancés pour augmenter la densité du maillage. Ces algorithmes ajoutent une courbure au maillage, en essayant de simuler les propriétés matérielles du lit. Le maillage du lit offre l'interpolation de lagrange et bicubique pour accomplir ceci.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
mesh_pps: 2, 3
algorithm: bicubic
bicubic_tension: 0.2
```

- `mesh_pps : 2, 3` *Valeur par défaut : 2, 2* L'option `mesh_pps` est l'abréviation de Mesh Points Per Segment. Cette option indique le nombre de points à interpoler pour chaque segment le long des axes X et Y. Un 'segment' est l'espace entre chaque point palpé. Comme pour `probe_count`, `mesh_pps` est spécifié en tant que paire de nombres entiers X, Y mais peut aussi être spécifié comme un seul nombre entier en ce cas appliqué aux deux axes. Dans cet exemple, il y a 4 segments le long de l'axe X et 2 segments le long de l'axe Y. Cela résulte en 8 points interpolés le long de l'axe X et 6 points interpolés le long de l'axe Y, ce qui donne un maillage de 13x8. Notez que si mesh_pps est défini à 0, l'interpolation de maillage est désactivée et la matrice sondée sera échantillonnée directement.
- `algorithm : lagrange` *Valeur par défaut : lagrange* L'algorithme utilisé pour interpoler le maillage. Peut être `lagrange` ou `bicubique`. L'interpolation de Lagrange est plafonnée à 6 points palpés car une oscillation tend à se produire avec un plus grand nombre d'échantillons. L'interpolation bicubique requiert un minimum de 4 points le long de chaque axe, si moins de 4 points sont spécifiés, l'échantillonnage de Lagrange est forcé. Si `mesh_pps` est défini à 0 alors cette valeur est ignorée car aucune interpolation de maille n'est faite.
- `bicubic_tension : 0.2` *Valeur par défaut : 0.2* Si l'option `algorithm` est définie sur bicubique, il est possible d'indiquer une valeur de tension. Plus la tension est élevée, plus la pente est interpolée. Soyez prudent lorsque vous ajustez cette valeur, car des valeurs plus élevées créent également plus de dépassement, ce qui entraînera des valeurs interpolées plus élevées ou plus basses que vos points palpés.

L'illustration ci-dessous montre comment les options ci-dessus sont utilisées pour générer un maillage interpolé.

![bedmesh_interpolated](img/bedmesh_interpolated.svg)

### Fractionnement des déplacements

Le maillage du lit fonctionne en interceptant les commandes de déplacement du gcode et en appliquant une transformation à leur coordonnée Z. Les longs déplacements doivent être divisés en déplacements plus petits pour suivre correctement la forme du lit. Les options ci-dessous contrôlent le comportement du fractionnement.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
move_check_distance: 5
split_delta_z: .025
```

- `move_check_distance : 5` *Valeur par défaut : 5* La distance minimale de vérification de changement de Z souhaité avant d'effectuer un fractionnemeny. Dans cet exemple, un mouvement de plus de 5mm sera traversé par l'algorithme. Tous les 5 mm, une recherche de maille Z sera effectuée, en la comparant à la valeur Z du mouvement précédent. Si le delta atteint le seuil fixé par `split_delta_z`, le mouvement sera divisé et la traversée continuera. Ce processus se répète jusqu'à ce que la fin du déplacement soit atteinte, où un ajustement final sera appliqué. Les déplacements plus courts que la `move_check_distance` ont l'ajustement Z correct appliqué directement au déplacement sans traversée ou division.
- `split_delta_z : .025` *Valeur par défaut : .025* Comme mentionné ci-dessus, il s'agit de l'écart minimum requis pour déclencher un fractionnement du mouvement. Dans cet exemple, toute valeur Z avec un écart de +/- 0,025 mm déclenchera un fractionnement.

Généralement les valeurs par défaut de ces options sont suffisantes, en fait la valeur par défaut de 5mm pour la `move_check_distance` est probablement exagérée. Cependant, un utilisateur avancé peut souhaiter expérimenter avec ces options dans le but d'obtenir une première couche optimale.

### Atténuation du maillage

Lorsque l'option "fondu" est activée, l'ajustement du Z est réduit progressivement sur une distance définie par la configuration. Ceci est réalisé en appliquant de petits ajustements à la hauteur de la couche, en augmentant ou en diminuant selon la forme du lit. Lorsque le fondu est terminé, l'ajustement Z n'est plus appliqué, ce qui permet au sommet de l'impression d'être plat plutôt que de refléter la forme du lit. Le fondu peut également présenter quelques caractéristiques indésirables, si le fondu est effectué trop rapidement, il peut entraîner des artefacts visibles sur l'impression. De plus, si votre lit est sensiblement déformé, le fondu peut rétrécir ou étirer la hauteur Z de l'impression. C'est pourquoi le fondu est désactivé par défaut.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
fade_start: 1
fade_end: 10
fade_target: 0
```

- `fade_start: 1` *Valeur par défaut : 1* La hauteur Z à laquelle il faut commencer l'atténuation progressive de l'ajustement. C'est une bonne idée d'avoir quelques couches déjà déposées avant de commencer le processus de fondu.
- `fade_end : 10` *Valeur par défaut : 0* La hauteur Z à laquelle le fondu doit s'arrêter. Si cette valeur est inférieure à `fade_start`, le fondu est désactivé. Cette valeur peut être ajustée en fonction de la déformation de la surface d'impression. Une surface fortement déformée devrait s'estomper sur une plus grande distance. Une surface presque plate peut être capable de réduire cette valeur pour s'estomper plus rapidement. 10mm est une valeur raisonnable pour commencer si vous utilisez la valeur par défaut de 1 pour `fade_start`.
- `fade_target : 0` *Valeur par défaut : La valeur Z moyenne du maillage* Le `fade_target` peut être considéré comme un décalage Z supplémentaire appliqué à l'ensemble du lit après la fin du fondu. En général, nous aimerions que cette valeur soit égale à 0, mais il y a des circonstances où elle ne devrait pas l'être. Par exemple, supposons que votre position d'origine sur le lit est aberrante, elle est inférieure de 0,2 mm à la hauteur moyenne palpée du lit. Si le `fade_target` est 0, le fondu va rétrécir l'impression d'une moyenne de 0,2 mm à travers le lit. En réglant la `fade_target` sur .2, la zone d'origine sera agrandie de .2 mm, mais le reste du lit aura une taille précise. En général, c'est une bonne idée de laisser `fade_target` en dehors de la configuration afin que la hauteur moyenne du maillage soit utilisée, cependant il peut être souhaitable d'ajuster manuellement la cible du fondu si l'on ne veut imprimer que sur une partie spécifique du lit.

### L'indice de référence relatif

La plupart des sondes sont sensibles à une dérive, c'est-à-dire à des inexactitudes dans le palpage introduites par la chaleur ou des interférences. Cela peut rendre difficile le calcul du décalage en Z de la sonde, en particulier à différentes températures du lit. C'est pourquoi certaines imprimantes utilisent à la fois une butée pour la mise à l'origine de l'axe Z et une sonde pour réaliser le maillage. Ces imprimantes peuvent bénéficier de la configuration de l'index de référence relatif.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
relative_reference_index: 7
```

- `relative_reference_index : 7` *Valeur par défaut : None (désactivé)* Lorsque les points palpés sont générés, un index leur est attribué. Vous pouvez consulter cet index dans klippy.log ou en utilisant BED_MESH_OUTPUT (voir la section sur les GCodes de maillage de lit ci-dessous pour plus d'informations). Si vous assignez un index à l'option `relative_reference_index`, la valeur palpée à cette coordonnée remplacera le z_offset de la sonde. Ce qui fait de cette coordonnée la référence "zéro" pour le maillage.

Lorsque vous utilisez l'indice de référence relatif, vous devez choisir l'indice le plus proche de l'endroit du lit où le calibrage de la butée Z a été effectué. Notez que lorsque vous recherchez l'indice en utilisant le journal ou BED_MESH_OUTPUT, vous devez utiliser les coordonnées listées sous l'en-tête "Probe" pour trouver l'indice correct.

### Régions défectueuses

Il est possible que certaines zones d'un lit donnent des résultats inexacts lors du palpage en raison d'un "défaut" à des endroits particuliers. Le meilleur exemple de ce phénomène est celui des lits comportant une série d'aimants intégrés utilisés pour retenir les tôles d'acier amovibles. Le champ magnétique au niveau et autour de ces aimants peut faire qu'une sonde inductive se déclenche à une distance plus élevée ou plus basse qu'elle ne le ferait autrement, ce qui donne un maillage ne représentant pas précisément la surface à ces endroits. **Remarque : Il ne faut pas confondre ce phénomène avec le biais de l'emplacement de la sonde, qui produit des résultats inexacts sur l'ensemble du lit.**

Les options `faulty_region` peuvent être configurées pour compenser cet effet. Si un point généré se trouve dans une région défectueuse, le maillage du lit tentera de palper jusqu'à 4 points aux limites de cette région. Ces valeurs palpées seront moyennées et insérées dans le maillage comme valeur Z à la coordonnée (X, Y) générée.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
faulty_region_1_min: 130.0, 0.0
faulty_region_1_max: 145.0, 40.0
faulty_region_2_min: 225.0, 0.0
faulty_region_2_max: 250.0, 25.0
faulty_region_3_min: 165.0, 95.0
faulty_region_3_max: 205.0, 110.0
faulty_region_4_min: 30.0, 170.0
faulty_region_4_max: 45.0, 210.0
```

- `faulty_region_{1...99}_min` `faulty_region_{1..99}_max` *Valeur par défaut : None (désactivé)* Les régions défectueuses sont définies d'une manière similaire à celle du maillage lui-même, où les coordonnées minimum et maximum (X, Y) doivent être infiquées pour chaque région. Une région défectueuse peut s'étendre à l'extérieur d'un maillage, mais les points alternatifs générés seront toujours à l'intérieur des limites du maillage. Deux régions ne peuvent pas se chevaucher.

L'image ci-dessous illustre comment les points de remplacement sont générés lorsqu'un point généré se trouve dans une région défectueuse. Les régions représentées correspondent à celles de l'exemple de configuration ci-dessus. Les points de remplacement et leurs coordonnées sont identifiés en vert.

![bedmesh_interpolated](img/bedmesh_faulty_regions.svg)

## Gcodes de maillage du lit

### Calibration

`BED_MESH_CALIBRATE PROFILE=<name> METHOD=[manual | automatic] [<probe_parameter>=<value>] [<mesh_parameter>=<value>]` *Profil par défaut : default* *Méthode par défaut : automatique si une sonde est détectée, sinon manuelle*

Lance la procédure de palpage de l'étalonnage du maillage du lit.

Le maillage sera sauvegardé dans un profil indiqué par le paramètre `PROFILE`, ou `default` si non précisé. Si `METHOD=manual` est sélectionné, un palpage manuel sera effectué. Lorsque vous passez du mode automatique au mode manuel, les points de maillage générés seront automatiquement ajustés.

Il est possible de spécifier des paramètres de maillage pour modifier la zone palpée. Les paramètres suivants sont disponibles :

- Lits rectangulaires (cartésiens) :
   - `MESH_MIN`
   - `MESH_MAX`
   - `PROBE_COUNT`
- Lits circulaires (delta) :
   - `MESH_RADIUS`
   - `MESH_ORIGIN`
   - `ROUND_PROBE_COUNT`
- Tous les lits :
   - `RELATIVE_REFERNCE_INDEX`
   - `ALGORITHM`

Consultez la documentation de configuration ci-dessus pour plus de détails sur la façon dont chaque paramètre s'applique au maillage.

### Profils

`BED_MESH_PROFILE SAVE=<name> LOAD=<name> REMOVE=<name>`

Après avoir réalisé un BED_MESH_CALIBRATE, il est possible de sauvegarder l'état actuel du maillage dans un profil nommé. Cela permet de charger un maillage sans re-palper le lit. Après qu'un profil ait été enregistré en utilisant `BED_MESH_PROFILE SAVE=<name>`, le gcode `SAVE_CONFIG` peut être exécuté pour écrire le profil dans printer.cfg.

Les profils peuvent être chargés en exécutant `BED_MESH_PROFILE LOAD=<name>`.

Il convient de noter que chaque fois qu'un BED_MESH_CALIBRATE est produit, l'état actuel est automatiquement enregistré dans le profil *par défaut*. Si ce profil existe, il est automatiquement chargé au démarrage de Klipper. Si ce comportement n'est pas souhaitable, le profil *par défaut* peut être supprimé comme suit :

`BED_MESH_PROFILE REMOVE=default`

Tout autre profil enregistré peut être supprimé de la même manière, en remplaçant *default* par le nom du profil que vous souhaitez supprimer.

### Sortie

`BED_MESH_OUTPUT PGP=[0 | 1]`

Affiche l'état actuel du maillage dans le terminal. Notez que le maillage lui-même est affiché

Le paramètre PGP est l'abréviation de "Print Generated Points". Si `PGP=1` est défini, les points palpés générés seront affichés sur le terminal :

```
// bed_mesh: generated points
// Index | Tool Adjusted | Probe
// 0 | (11.0, 1.0) | (35.0, 6.0)
// 1 | (62.2, 1.0) | (86.2, 6.0)
// 2 | (113.5, 1.0) | (137.5, 6.0)
// 3 | (164.8, 1.0) | (188.8, 6.0)
// 4 | (216.0, 1.0) | (240.0, 6.0)
// 5 | (216.0, 97.0) | (240.0, 102.0)
// 6 | (164.8, 97.0) | (188.8, 102.0)
// 7 | (113.5, 97.0) | (137.5, 102.0)
// 8 | (62.2, 97.0) | (86.2, 102.0)
// 9 | (11.0, 97.0) | (35.0, 102.0)
// 10 | (11.0, 193.0) | (35.0, 198.0)
// 11 | (62.2, 193.0) | (86.2, 198.0)
// 12 | (113.5, 193.0) | (137.5, 198.0)
// 13 | (164.8, 193.0) | (188.8, 198.0)
// 14 | (216.0, 193.0) | (240.0, 198.0)
```

Les points de la colonne "Tool Adjusted" font référence à l'emplacement de la buse, et les points de la colonne "Probe" font référence à l'emplacement du palpeur. Notez que lors d'un palpage manuel, les points "Probe" se réfèrent à la fois à l'emplacement de l'outil et de la buse.

### Effacer l'état du maillage

`BED_MESH_CLEAR`

Ce G-Code peut être utilisé pour effacer l'état interne du maillage.

### Appliquer les décalages X/Y

`BED_MESH_OFFSET [X=<value>] [Y=<value>]`

Ceci est utile pour les imprimantes avec plusieurs extrudeuses indépendantes, car un décalage est nécessaire pour produire un ajustement Z correct après un changement d'outil. Les décalages doivent être indiqués par rapport à l'extrudeuse primaire. C'est-à-dire qu'un décalage X positif doit être indiqué si l'extrudeuse secondaire est montée à droite de l'extrudeuse primaire, et un décalage Y positif doit être indiqué si l'extrudeuse secondaire est montée "derrière" l'extrudeuse primaire.
