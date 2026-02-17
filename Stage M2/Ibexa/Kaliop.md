
# Kaliop


### Générer la migration

`./cmd kaliop-migration-generate --format=yml --type=content --mode=update --match-type=content_id --match-value=70`

-  ``--type=content`` si la cible est un contenu, si on cible un **content type** alors ``--type=contenttype``
-  `--mode=update` s'il s'agit d'une mise à jour 
- `--match-type=content_id` si on veut récupérer par l'identifiant du contenu (dans l'url)
-  Pour voir les spécificités de chacun des arguments :  `./cmd kaliop-migration-generate --list-types`

`--match-value=70` précise l'identifiant sur lequel effectuer la liaison parmi les content du BO

### Pour exécuter la migration

`./cmd kaliop-migration-migrate --path=src/MigrationsDefinitions/20260217113839_update_config_fields_Menu.yml`

On écrit simplement le chemin du fichier dont on souhaite exécuter la migration

On peut ajouter le suffixe `-f` pour forcer la ré-exécution de la migration (pour tester par exemple)
 