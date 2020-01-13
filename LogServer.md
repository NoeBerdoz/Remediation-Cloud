# LogServer

L'objectif est de créer une machine permettant l'agrégation de `logs` de différentes sources.
Cette machine tournera sous Linux, l'installation et la configuration se fait "from scratch"
et sans interface graphique.

## Logiciels & services:

  1. Linux de base (vous avez le choix de la distribution)
  2. Applications générant des logs:
    1. Serveur Web Nginx
    2. PHP-FPM
  3. [fluentd](https://www.fluentd.org/)
  5. MongoDB

## Fonctionnement

Ce serveur fonctionne avec `fluentd` comme élément principal. `nginx` et `php-fpm` sont installés juste pour générer des logs.
Une simple configuration de base (mais utilisable) sera réalisée.

`MongoDB` sera le système de stockage des logs qui sortent de `fluentd`.

Voici un schéma:

    
    nginx >-----+
                |   +-----------+
                +-->|           |
                    |  fluentd  |----> mongoDB
                +-->|           |
                |   +-----------+
    php-fpm >---+
    

## Rendu

Le groupe réalisera un rapport d'installation expliquant chaque étape avec les commandes réalisées. Ceci afin de pouvoir suivre
le rapport pour re-créer la machine complète.

Il contiendra également:

Ce rapport sera écrit en markdown comme cette donnée!
