#compsoer #symfony
# Composer 

Composer est un gestionnaire de paquets pour php. Il place toutes ces dépendances dans le dossier **vendor**. 

### composer.json

Ce fichier contient "ce que l'on souhaite" c'est à dire les paquets installés avec la commande ```composer require```.

### composer.lock

Ce fichier contient "ce que l'on a", c'est à dire ce qui est installé sur notre ordinateur.

## Symfony

-h => info sur les commandes symfony php bin/console

un paquet va generer des classes et le bundle va interagir avec le symfony

php bin/console doctrine:database <=> php bin/console d:d

### Fixture

la commande de **Fixture** permet de générer une classe ou l'on va pouvoir créer un jeu de données aléatoire rapidement dans une  classe dédiée est exécuter ensuite le script qui va, ou non, vider la base de données