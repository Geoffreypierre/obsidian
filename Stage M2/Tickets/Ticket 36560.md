# Site Codéin

Ce ticket concerne exclusivement de la refonte graphique 

### Tâches à réaliser

##### Menu desktop :

- ajouter background blanc derrière menu de deuxième niveau
- finir/corriger le dropdown du menu de troisième niveau (style et fonctionnalié)
- ajouter le push à droite (les infos viennent du content type ibexa nommé **config**)
- vérification et/ou ajout aria
##### Menu mobile :

- gérer l'affichage du menu pour les second et troisième niveaux
- vérification et/ou ajout aria

#### Lancement 

Suite à des soucis d'installation, il faut procéder ainsi :
- Se placer dans la branche ``v46/develop`` (branche sur laquelle a été faite l'installation)
- Lancer tous les services avec : ``./cmd up``
- Permuter avec la branche de mr : ``git checkout feature/36560-menu-principal-refonte-ux-ui``
- lancer le compilateur automatique de scss : ``./cmd yarn-watch``

### Réalisations

##### finir/corriger le dropdown du menu de troisième niveau (style et fonctionnalié) :
- modification de la position en `static` lors de l'activation
- ajout d'une ligne et d'une flèche dans le html
- mise à jour des couleurs quand la balise `li` est active grâce au **jquery**, en particulier un **MutationObserver** qui  **toggle** l'ajout d'une classe auprès de ces 2 éléments pour la mise à jour 
	Ce schéma était nécessaire, on ne pouvait pas utiliser un élément `::after` car le `::before` est celui de la div parent de la balise qui va porter la classe activation donc impossible avec du scss uniquement de jouer sur les couleurs lors de l'activation

##### gérer l'affichage du menu pour les second et troisième niveaux
-  Ajout d'une image dans les *assets* symbolisant la flèche présente sur la maquette
- Modification du scss pour n'afficher que le sous-menu lors du click sur un lien qui possède un sous-menu.  Le bouton **Retour** permet de revenir au 1er niveau de menu en cloturant le ou les sous-menu ouverts grâce à un **listener** en **jquery**.

### Notes

### Ce qui avait été fait
- ajout de nouveaux field_groups ibexa
- migration kaliop pour ajouter des fields et des field_group au content-type config
- ajout du bouton "Blog" dans le header
- update du bouton "Contactez-nous" du header
- modification du css des éléments du menu
- retrait des liens des entrées du menu ayant des enfants
- ajout des liens des entrées du menu ayant des enfants dans le niveau d'en dessous
- ajout de traductions
- ajout d'un background au mégamenu
- update du fil d'Arianne
- début du menu mobile / dropdown
- retrait de l'affichage des réseaux sociaux au format mobile
- menu au clic
- style des items du deuxième et troisième niveaux
- ajout boutons additionnels et update css
### Derniers ajouts

Menu desktop :
- ajouter background blanc derrière menu de deuxième niveau
- finir/corriger le dropdown du menu de troisième niveau (style et fonctionnalié)
- ajouter le push à droite (les infos viennent du content type ibexa nommé **config**)
- vérification et/ou ajout aria

Menu mobile :
- gérer l'affichage du menu pour les second et troisième niveaux
- vérification et/ou ajout aria

Général :
- Modification du **content type** `config` (voir maquette)
### Pour tester

Un souci avec l'image MariaDB `Bitnami` à provoqué une erreur lors de la mise en route du projet. A l'heure actuelle, il faut lancer les services depuis la branche `develop` puis changer de branche sur la branche de travail (Pour plus d'informations, contacter Marc Teinturier):

- Se placer sur la branche `develop`
- Lancer les scripts de migration Kaliop 
- Lancer la commande ``./cmd yarn-dev`` pour compiler les assets
- Lancer les services Docker avec : `./cmd up`
- Basculer sur la branche de travail

On peut ensuite vérifier l'affichage et la fonctionnalité du menu sur plusieurs pages
### Divers

- Lien vers la merge request : [https://gitlab.codein.fr/beclood/codein/-/merge_requests/319](https://gitlab.codein.fr/beclood/codein/-/merge_requests/319)
- Présence d'un script de migration Doctrine : Non
- Présence d'un script de migration Kaliop : **Oui**