# Commandes MCU

Ce document fournit des informations sur les commandes de bas niveau du microcontrôleur qui sont envoyées depuis le logiciel "hôte" Klipper et traitées par le firmware Klipper du microcontrôleur. Ce document n'est pas une référence faisant autorité pour ces commandes, ni une liste exclusive de toutes les commandes disponibles.

Ce document peut être utile aux développeurs souhaitant comprendre les commandes de bas niveau du microcontrôleur.

Voir le document [protocole](Protocol.md) pour plus d'informations sur le format des commandes et leur transmission. Les commandes ici sont décrites en utilisant une syntaxe de style "printf" - pour ceux qui ne connaissent pas ce format, notez simplement que lorsqu'une séquence '%...' est vue, elle doit être remplacée par un entier réel. Par exemple, une description avec "count=%c" pourrait être remplacée par le texte "count=10". Notez que les paramètres qui sont considérés comme des "énumérations" (voir le document de protocole ci-dessus) prennent une valeur de chaîne qui est automatiquement convertie en une valeur entière pour le microcontrôleur. Ceci est courant avec les paramètres nommés "pin" (ou qui ont le suffixe "_pin").

## Commandes de démarrage

Il peut être nécessaire d'effectuer des actions ponctuelles pour configurer le microcontrôleur et ses périphériques. Cette section répertorie les commandes courantes disponibles à cette effet. Contrairement à la plupart des commandes de microcontrôleur, ces commandes s'exécutent dès leur réception et ne nécessitent aucune configuration particulière.

Commandes de démarrage courantes :

* `set_digital_out pin=%u value=%c` : Cette commande configure immédiatement la broche donnée en tant que sortie numérique GPIO et la définit soit à un niveau bas (valeur=0) soit à un niveau haut (valeur=1 ). Cette commande peut être utile pour configurer la valeur initiale des LED et pour configurer la valeur initiale des broches de micro-pas du pilote de moteur pas à pas.
* `set_pwm_out pin=%u cycle_ticks=%u value=%hu` : Cette commande configurera la broche donnée pour utiliser la modulation de largeur d'impulsion (PWM) basée sur le matériel avec le nombre donné de cycle_ticks. Le "cycle_ticks" est le nombre de ticks d'horloge du MCU que chaque cycle de mise sous tension et hors tension doit durer. Une valeur cycle_ticks de 1 peut être utilisée pour demander le temps de cycle le plus rapide possible. Le paramètre "valeur" est compris entre 0 et 255, 0 indiquant un état entièrement désactivé et 255 indiquant un état entièrement activé. Cette commande peut être utile pour activer les ventilateurs de refroidissement du processeur et des buses.

## Configuration bas niveau du microcontrôleur

La plupart des commandes du microcontrôleur nécessitent une configuration initiale avant de pouvoir être appelées. Cette section fournit une vue d'ensemble du processus de configuration. Cette section et les sections suivantes ne sont susceptibles d'intéresser que les développeurs intéressés par les détails internes de Klipper.

Lorsque l'hôte se connecte pour la première fois au microcontrôleur, il commence toujours par obtenir un dictionnaire de données (voir [protocole](Protocol.md) pour plus d'informations). Une fois le dictionnaire de données obtenu, l'hôte vérifie si le microcontrôleur est dans un état "configuré" et le configure si ce n'est pas le cas. La configuration comprend les phases suivantes :

* `get_config` : L'hôte commence par vérifier si le micro-contrôleur est déjà configuré. Le microcontrôleur répond à cette commande par un message de réponse « config ». Le logiciel du microcontrôleur démarre toujours dans un état non configuré à la mise sous tension. Il reste dans cet état jusqu'à ce que l'hôte termine les processus de configuration (en émettant une commande finalize_config). Si le microcontrôleur est déjà configuré à partir d'une session précédente (et est configuré avec les paramètres souhaités), aucune autre action n'est requise de la part de l'hôte et le processus de configuration se termine avec succès.
* `allocate_oids count=%c` : cette commande est envoyée pour informer le microcontrôleur du nombre maximal d'identifiants d'objet (oid) requis par l'hôte. Il n'e faut envoyer cette commande qu'une seule fois. Un oid est un identifiant entier alloué à chaque stepper, chaque fin de course et chaque broche gpio programmable. L'hôte détermine à l'avance le nombre d'oids dont il aura besoin pour faire fonctionner le matériel et le transmet au microcontrôleur afin qu'il puisse allouer suffisamment de mémoire pour stocker un mappage de l'oid à l'objet interne.
* `config_XXX oid=%c ...` : Par convention toute commande commençant par le préfixe "config_" crée un nouvel objet microcontrôleur et lui affecte l'oid donné. Par exemple, la commande config_digital_out configurera la broche spécifiée en tant que GPIO de sortie numérique et créera un objet interne que l'hôte peut utiliser pour planifier des modifications du GPIO donné. Le paramètre oid passé dans la commande config est sélectionné par l'hôte et doit être compris entre zéro et le nombre maximal fourni dans la commande allow_oids. Les commandes de configuration ne peuvent être exécutées que lorsque le microcontrôleur n'est pas dans un état configuré (c'est-à-dire avant que l'hôte n'envoie finalize_config) et après que la commande allow_oids a été envoyée.
* `finalize_config crc=%u` : La commande finalize_config fait passer le microcontrôleur d'un état non configuré à un état configuré. Le paramètre crc transmis au microcontrôleur est stocké et renvoyé à l'hôte dans les messages de réponse "config". Par convention, l'hôte prend un CRC 32 bits de la configuration qu'il demandera et au début des sessions de communication suivantes, il vérifie que le CRC stocké dans le microcontrôleur correspond exactement à son CRC souhaité. Si le CRC ne correspond pas, l'hôte saura que le microcontrôleur n'a pas été configuré correctement.

### Objets de base du microcontrôleur

Cette section répertorie quelques commandes de configuration couramment utilisées.

* `config_digital_out oid=%c pin=%u value=%c default_value=%c max_duration=%u` : cette commande crée un objet de microcontrôleur interne pour la "broche" GPIO donnée. La broche sera configurée en mode de sortie numérique et définie sur une valeur initiale telle que spécifiée par 'value' (0 pour bas, 1 pour haut). La création d'un objet digital_out permet à l'hôte de programmer des mises à jour GPIO pour la broche donnée à des heures spécifiées (voir la commande queue_digital_out décrite ci-dessous). Si le logiciel du microcontrôleur passe en mode d'arrêt, tous les objets digital_out configurés seront définis sur 'default_value'. Le paramètre 'max_duration' est utilisé pour implémenter un contrôle de sécurité - s'il est différent de zéro, il s'agit du nombre maximal de ticks d'horloge pendannt lesquels l'hôte peut définir le GPIO donné sur une valeur autre que celle par défaut sans autres mises à jour. Par exemple, si la default_value est zéro et la max_duration est de 16000, alors si l'hôte définit le gpio sur une valeur de un, il doit planifier une autre mise à jour de la broche gpio (à zéro ou à un) dans les 16000 tics d'horloge. Cette fonction de sécurité peut être utilisée avec des broches de chauffage pour s'assurer que l'hôte n'active pas le chauffage puis se déconnecte en le laissant activé.
* `config_pwm_out oid=%c pin=%u cycle_ticks=%u value=%hu default_value=%hu max_duration=%u` : cette commande crée un objet interne pour les broches PWM matérielles pour lesquelles l'hôte peut planifier des mises à jour . Son utilisation est analogue à config_digital_out - voir la description des commandes 'set_pwm_out' et 'config_digital_out' pour la description des paramètres.
* `config_analog_in oid=%c pin=%u` : Cette commande permet de configurer une broche en mode d'échantillonnage d'entrée analogique. Une fois configurée, la broche peut être échantillonnée à intervalle régulier à l'aide de la commande query_analog_in (voir ci-dessous).
* `config_stepper oid=%c step_pin=%c dir_pin=%c invert_step=%c step_pulse_ticks=%u` : Cette commande crée un objet "moteur pas à pas" interne. Les paramètres 'step_pin' et 'dir_pin' spécifient respectivement les broches de pas et de direction ; cette commande les configurera en mode de sortie numérique. Le paramètre 'invert_step' spécifie si une étape se produit sur un front montant (invert_step=0) ou descendant (invert_step=1). Le paramètre 'step_pulse_ticks' spécifie la durée minimale de l'impulsion de pas. Si le mcu exporte la constante 'STEPPER_BOTH_EDGE=1', le réglage step_pulse_ticks=0 et invert_step=-1 sera configuré pour marcher sur les fronts montant et descendant de la broche de pas.
* `config_endstop oid=%c pin=%c pull_up=%c stepper_count=%c` : Cette commande crée un objet "endstop" interne. Il est utilisé pour spécifier les broches d'arrêt et pour activer les opérations de "homing" (voir la commande endstop_home ci-dessous). La commande configurera la broche spécifiée en mode d'entrée numérique. Le paramètre 'pull_up' détermine si les résistances pullup fournies par le matériel pour la broche (si disponibles) seront activées. Le paramètre 'stepper_count' spécifie le nombre maximum de pas dont cette butée peut avoir besoin pendant une opération de prise d'origine (voir endstop_home ci-dessous).
* `config_spi oid=%c bus=%u pin=%u mode=%u rate=%u shutdown_msg=%*s` : Cette commande crée un objet SPI interne. Il est utilisé avec les commandes spi_transfer et spi_send (voir ci-dessous). Le paramètre "bus" identifie le bus SPI à utiliser (si le microcontrôleur a plus d'un bus SPI disponible). Le paramètre "broche" spécifie la broche de sélection de puce (CS) pour le périphérique. Le paramètre "mode" est le mode SPI (doit être compris entre 0 et 3). Le paramètre "rate" spécifie le débit du bus SPI (en cycles par seconde). Enfin, le paramètre "shutdown_msg" est une commande SPI à envoyer à l'appareil donné si le microcontrôleur passe en état d'arrêt.
* `config_spi_without_cs oid=%c bus=%u mode=%u rate=%u shutdown_msg=%*s` : Cette commande est similaire à config_spi, mais sans définition de la broche CS. Il est utile pour les appareils SPI qui n'ont pas de ligne de sélection de puce.

## Commandes courantes

Cette section répertorie les commandes les plus couramment utilisées. Cela n'intéresse probablement que les développeurs qui cherchent à mieux comprendre Klipper.

* `set_digital_out_pwm_cycle oid=%c cycle_ticks=%u` : cette commande configure une broche de sortie numérique (telle que créée par config_digital_out) pour utiliser le "PWM logciel". Le 'cycle_ticks' est le nombre de tics d'horloge pour le cycle PWM. Étant donné que la commutation de sortie est implémentée dans le logiciel du microcontrôleur, il est recommandé que 'cycle_ticks' corresponde à un temps de 10 ms ou plus.
* `queue_digital_out oid=%c clock=%u on_ticks=%u` : Cette commande permet de planifier une changement d'état d'une broche GPIO de sortie numérique au tick d'horloge donnée. Pour utiliser cette commande, une commande 'config_digital_out' avec le même paramètre 'oid' doit avoir été émise lors de la configuration du microcontrôleur. Si 'set_digital_out_pwm_cycle' a été appelé, alors 'on_ticks' est la durée d'activation (en ticks d'horloge) pour le cycle pwm. Sinon, 'on_ticks' doit être soit 0 (pour basse tension) soit 1 (pour haute tension).
* `queue_pwm_out oid=%c clock=%u value=%hu` : planifie un changement sur une broche de sortie PWM matérielle. Voir les commandes 'queue_digital_out' et 'config_pwm_out' pour plus d'informations.
* `query_analog_in oid=%c clock=%u sample_ticks=%u sample_count=%c rest_ticks=%u min_value=%hu max_value=%hu` : Cette commande définit une récupération récurrente d'échantillons d'entrée analogique. Pour utiliser cette commande, une commande 'config_analog_in' avec le même paramètre 'oid' doit avoir été émise lors de la configuration du microcontrôleur. Les échantillons commenceront à partir de l'heure 'clock', et ils rapporteront la valeur obtenue à chaque tic d'horloge 'rest_ticks', il suréchantillonnera avec un nombre 'sample_count' d'échantillons et il se mettra en pause pendant le nombre 'sample_ticks' de tic d'horloge entre chaque échantillonnage. Les paramètres 'min_value' et 'max_value' implémentent une fonction de sécurité - le logiciel du microcontrôleur vérifiera que la valeur échantillonnée se situe toujours dans la plage fournie. Ceci est destiné à être utilisé avec des broches rattachées aux thermistances contrôlant les radiateurs - il peut être utilisé pour vérifier qu'un radiateur se trouve dans une plage de température correcte.
* `get_clock` : Cette commande amène le microcontrôleur à générer un message de réponse "clock". L'hôte envoie cette commande une fois par seconde pour obtenir la valeur de l'horloge du microcontrôleur et pour estimer la dérive entre les horloges de l'hôte et du microcontrôleur. Il permet à l'hôte d'estimer avec précision l'horloge du microcontrôleur.

### Commandes de moteurs pas à pas

* `queue_step oid=%c interval=%u count=%hu add=%hi` : cette commande planifie le nombre de pas 'count' pour le stepper donné, avec un nombre 'd'intervalle' de ticks d'horloge entre chaque pas. La première étape sera le nombre "d'intervalle" de ticks d'horloge depuis la dernière étape planifiée pour le stepper donné. Si 'add' n'est pas nul, l'intervalle sera ajusté du montant 'add' après chaque pas. Cette commande ajoute la séquence d'intervalle/compte/ajout donnée à une file d'attente des pas. Il peut y avoir des centaines de ces séquences en file d'attente pendant le fonctionnement normal. Une nouvelle séquence est ajoutée à la fin de la file d'attente et lorsque chaque séquence termine son nombre de pas, elle est retirée du début de la file d'attente. Ce système permet au microcontrôleur de mettre en file d'attente potentiellement des centaines de milliers d'étapes, le tout avec des durées fiables et prévisibles.
* `set_next_step_dir oid=%c dir=%c` : Cette commande spécifie la valeur du dir_pin que la prochaine commande queue_step utilisera.
* `reset_step_clock oid=%c clock=%u` : Normalement, la synchronisation des pas est relative au dernier pas pour un stepper donné. Cette commande réinitialise l'horloge de sorte que l'étape suivante soit relative à l'heure "horloge" fournie. L'hôte n'envoie généralement cette commande qu'au début d'une impression.
* `stepper_get_position oid=%c` : Cette commande amène le microcontrôleur à générer un message de réponse "stepper_position" avec la position actuelle du stepper. La position est le nombre total d'étapes générées avec dir=1 moins le nombre total d'étapes générées avec dir=0.
* `endstop_home oid=%c clock=%u sample_ticks=%u sample_count=%c rest_ticks=%u pin_value=%c` : Cette commande est utilisée lors des opérations de "homing" des moteurs pas à pas. Pour utiliser cette commande, une commande 'config_endstop' avec le même paramètre 'oid' doit avoir été émise lors de la configuration du microcontrôleur. Lorsque cette commande est invoquée, le microcontrôleur échantillonnera la broche endstop à chaque tic d'horloge 'rest_ticks' et vérifiera si elle a une valeur égale à 'pin_value'. Si la valeur correspond (et continue de correspondre pour 'sample_count' échantillons supplémentaires répartis 'sample_ticks'), alors la file d'attente de mouvement pour le stepper associé sera effacée et le stepper s'arrêtera immédiatement. L'hôte utilise cette commande pour implémenter la prise d'origine - l'hôte demande à la butée d'échantillonner pour le déclencheur de butée, puis il émet une série de commandes queue_step pour déplacer un stepper vers la butée. Une fois que le stepper atteint la butée, le déclencheur sera détecté, le mouvement arrêté et l'hôte notifié.

### Déplacer la file d'attente

Chaque commande queue_step utilise une entrée dans la "file d'attente de déplacement" du microcontrôleur. Cette file d'attente est allouée lorsqu'elle reçoit la commande "finalize_config", et elle signale le nombre d'entrées de file d'attente disponibles dans les messages de réponse "config".

Il est de la responsabilité de l'hôte de s'assurer qu'il y a de l'espace disponible dans la file d'attente avant d'envoyer une commande queue_step. Pour ce faire, l'hôte calcule le moment où chaque commande queue_step se termine et planifie de nouvelles commandes queue_step en conséquence.

### Commandes SPI

* `spi_transfer oid=%c data=%*s` : Cette commande amène le microcontrôleur à envoyer des "données" au périphérique spi spécifié par "oid" et génère un message de réponse "spi_transfer_response" avec les données retournées par le périphérique.
* `spi_send oid=%c data=%*s` : Cette commande est similaire à "spi_transfer", mais elle ne génère pas de message "spi_transfer_response".