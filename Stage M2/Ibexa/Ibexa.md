on lance le projet avec ```ddev start```, ensuite on peut connaître l'URL local sur lequel le serveur est lancé via la commande ```ddev describe``` 
cela permet de savoir ou tournent chacun des services (*par exemple la base de données dans un conteneur docker*)
ex : ``http://localhost:42086``


![[content_create.png]]

On accède au **backoffice** avec l'url ``http://localhost:42086/admin``
les identifiants de connexions sont user : ```admin``` et password : ```publish```

#### Content
L'orchestration du Framework consiste essentiellement en la disposition de **block** dit ``content``. 
On peut créer du contenu suivant plusieurs type de contenu standards prédéfini ou créer son propre contenu customisé avec les champs suivants :
```Field type Name Identifier Required Searchable Translatable```

###### Configuration de Webpack
Il faut également ajouter les imports dans le fichier ``webpack.config.js`` 
ex : on veut ajouter des fichier ``normalize.css`` et ``style.css``

```js
Encore.addStyleEntry('tutorial', [
        path.resolve(__dirname, './assets/css/normalize.css'),
        path.resolve(__dirname, './assets/css/style.css')
    ])
```

pour recompiler les assets on effectue la commande suivante : ``yarn encore dev``

#### styles

Les assets s'ajoutent de la même façon que dans un projet symfony : 
les dossiers ``css (ou scss), js`` et ``font`` seront dans le dossier ``assets/`` à la racine du projet 
tandis que les images seront ajoutées dans ``public/assets/``

<div style="width: 100%; text-align:center; margin: 20px auto;">
<img src="assets.png" style="width: 200px; border: solid 1px black ">
</div>

On peut établir des configurations selon si l'on souhaite mettre en place une certaine scalabilité des images par exemples par l'inscription de filtres dans un ficher YAML de configuration ``config/packages/image_variations.yaml``:

```yaml
ibexa:
    system:
        default:
            image_variations:
                ride_list:
                    reference: null
                    filters:
                        - {name: geometry/scaledownonly, params: [140, 100]}
```

On définit ici un Template d'ajustement de taille nommé ``ride_list`` que l'on peut intégrer dans un Template Twig :

```twig
{% if not ibexa_field_is_empty( content, 'photo' ) %}
    {{ ibexa_render_field(content, 'photo', {
        'parameters': {
            'alias': 'ride_list'
        }
    }) }}
{% endif %}
```



```
#### Templates 

de la même manière qu'en symfony, les templates étendent tous du fichier ``main.html.twig`` situé à la racine du dossier ``templates``  et leur contenu est disposé dans un block :
```twig
{% extends "main_layout.html.twig" %}

{% block content %}
    <div class="col-xs-10 col-xs-offset-1 text-justified">
        <h1>Hello World!</h1>
    </div>
{% endblock %}
```
#### Page d'accueil

Pour personnaliser la page d'accueil, on supprime la page d'accueil par défaut :
- Supprimer le fichier de configuration ``config/packages/ibexa_welcome_page.yaml``
- Puis la vue associée dans ``templates/themes/standard/full/welcome_page.html.twig`` 

```yaml
ibexa:
    system:
        site:
            content_view:
                full:
                    home_page:
                        controller: ibexa_query::pagingQueryAction
                        template: themes/standard/full/home_page.html.twig
                        match:
                            Id\Location: 2
                        params:
                            query:
                                query_type: Ride
                                limit: 4
                                assign_results_to: rides
                    ride:
                        template: themes/standard/full/ride.html.twig
                        controller: App\Controller\RideController::viewRide
                        match:
                            Identifier\ContentType: ride

                line:
                    landmark:
                        template: line/landmark.html.twig
                        match:
                            Identifier\ContentType: landmark
```

On peux définir plusieurs sorte d'affichage pour un contenu similaire en définissant ici les vues associées à chacune des dispositions. 

par exemple : ici on définie la vue ``home_page.html.twig`` dans le type de vue de contenu ``full``
*(Qui va être définie dans le dossier ``templates/themes/standard/full/`` par défaut ou  simplement ``templates/full/``)*

La convention préconise de créer un Template d'un type de vue de contenu dans un dossier de même nom placé à la racine du dossier ``templates``

Si on souhaite créer une vue ``full`` pour le content ``Ride`` :

On configure d'abords la vue dans le fichier ``ibexa_welcome_page.yaml`` :

```yaml
ibexa:
    system:
        site:
            content_view:
                full:
					#else content
                    ride:
                        template: themes/standard/full/ride.html.twig
                        controller: App\Controller\RideController::viewRide
                        match:
                            Identifier\ContentType: ride
```

 Ensuite, on créée la vue Twig associée au chemin suivant : ``templates/full/rideview.html.twig``

Pour récupérer les champs du content on utilise la fonction ``ibexa_render_field()`` comme ceci :

```twig
{{ ibexa_render_field( content, 'length') }} km</p>
```

Ou en ajoutant des paramètres :

```twig
{{ ibexa_render_field(content, 'ending_point', {'parameters': { 'width': '100%', height: '200px', 'showMap': true, 'showInfo': false }}) }}
```


Il est possible d'afficher des variables pour débuguer avec  :
```php
{{dump()}}
```

ou pour une variable en particulier

```php
{{dump(variable)}}
```

Si l'entité concernée est sous forme de **liste**, on itère dessus de façon classique :

```twig
{% for ride in rides.currentPageResults %}
	{{ render(controller('ibexa_content::viewAction',{'location': ride.valueObject,'viewType':'line'})) }}
{% endfor %}
```

Ce cas précis illustre une récupération **ordonnée** et **limitée** de contenu, dans ces cas-ci on crée un fichier spécial de requête qui va s'occuper de l'interaction avec la base de données :

```php
<?php

namespace App\QueryType;

use Ibexa\Contracts\Core\Repository\Values\Content\LocationQuery;
use Ibexa\Contracts\Core\Repository\Values\Content\Query\Criterion;
use Ibexa\Core\QueryType\QueryType;

class RideQueryType implements QueryType
{
    public static function getName()
    {
        return 'Ride';
    }

    public function getQuery(array $parameters = [])
    {
        return new LocationQuery([
            'filter' => new Criterion\LogicalAnd(
                [
                    new Criterion\Visibility(Criterion\Visibility::VISIBLE),
                    new Criterion\ContentTypeIdentifier(['ride']),
                ]
            )
        ]);
    }

    public function getSupportedParameters() {}
}
```

Ce fichier sera placé dans le dossier ``src/QueryType``. La bonne pratique recommande d'effectuer 1 fichier de requête par type de content *(type d'entité)*. 

Par exemple pour récupérer une liste de **Ride**, on crée le fichier **RideQueryType.php**

Pour compléter cette mécanique on ajoute les dernières configurations dans le fichier ``config/packages/views.yaml`` 
```yaml
site:
    content_view:
        full:
            home_page:
                controller: ibexa_query::pagingQueryAction #Controller par défaut
                template: ...
                match: ...
                #Paramètres à ajouter
                params:
                    query:
                        query_type: Ride
                        limit: 4
                        assign_results_to: rides
```

``query_type`` indique quel type de requêtes utiliser. Correspond ici au nom ``Ride`` retourné par la méthode ``getName()`` di fichier ``RideQueryType.php``

``pagingQueryAction`` est le controlleur par défaut d'Ibexa, il pagine automatiquement les résultats et on peux limiter le nombre de résultats souhaités avec le paramètre ``limit``

### Créer un template pour du contenu

Par exemple déclarer le template **line** pour le contenu **Ride**
On déclare donc d'abords la vue dans le fichier ``config/packages/views.yaml``
```yaml
system:
    site:
        content_view:
            line:
                ride:
                    template: line/rides.html.twig
                    match:
                        Identifier\ContentType: ride
```

Puis on crée le Template ``templates/line/rides.html.twig``

```twig
<tr>
<td>
	<a href="{{ path("ibexa.url.alias",{'locationId': content.contentInfo.mainLocationId}) }}"
	   target="_self">
		<strong>
			{{ content.name }}
		</strong>
		{% if not ibexa_field_is_empty( content, 'photo' ) %}
			{{ ibexa_render_field(content, 'photo') }}
		{% endif %}
	</a>
</td>
<td>
	{{ ibexa_render_field(content,'starting_point',{'parameters':{'width':'100%', 'height':'100px','showMap':true,'showInfo':true }}) }}
</td>
<td>
	{{ ibexa_render_field(content,'ending_point',{'parameters':{'width':'100%', 'height':'100px','showMap':true,'showInfo':true }}) }}
</td>
<td>
	<p>{{ ibexa_render_field( content, 'length' ) }} Km</p>
</td>
</tr>
```

Si jamais l'entité contient des **Media**, il faut ajouter les permissions de lecture de Media :
``Admin`` →  ``Roles`` → ``Anonymous``  

Se rendre dans le mode édition 
<div style="width: 100%; text-align:center; margin: 20px auto">
<img src="anonymous_content_read_edit.png" style="width: 600px">
</div>
Puis cocher ``Media`` dans l'input ``section``

<div style="width: 100%; text-align:center; margin: 20px auto">
<img src="edit_limitations_media.png" style="width: 450px">
</div>

### Authentification d'un utilisateur

Pour ajouter un formulaire de **Connexion/inscription**, on doit se rendre dans : 
``Admin`` →  ``Roles`` → ``Anonymous`` et ajouter la police ``User/Register``. Un formulaire sera alors disponible à l'URL ``/register`` 

Pour personnaliser son Template, on initialise la vue que l'on peut appeler ``user_registration``  
dans le fichier ``config/packages/views.yaml`` :

```yaml
ibexa:
    system:
        site:
            # existing content_view keys
            user_registration:
                templates:
                    form: user/registration_form.html.twig
```

On créée le fichier ``templates/user/registration_form.html.twig`` :

```twig
{% extends "main_layout.html.twig" %}

{% block page_head %}
    {% set title = 'Register user'|trans %}

    {{ parent() }}
{% endblock %}

{% block content %}
    {% import 'user/registration_content_form.html.twig' as registrationForm %}

    <div class="container">
        <section class="user-form col-md-6 col-md-offset-3">
            <h2>{{ 'Member Registration'|trans }}</h2>
            <div class="legend">* {{ 'All fields are required'|trans }}</div>

            {{ registrationForm.display_form(form) }}
        </section>
    </div>
{% endblock %}
```

Le Template contenant les champs du formulaire sont contenu dans le fichier ``user/registration_content_form.html.twig``,  Donc on créée ce fichier

```twig
{% macro display_form(form) %}
    {{ form_start(form) }}

    {% for fieldForm in form.fieldsData %}
        {% set fieldIdentifier = fieldForm.vars.data.fieldDefinition.identifier %}

        {% if fieldIdentifier == 'first_name' or fieldIdentifier == 'last_name' %}
            {% if fieldIdentifier == 'first_name' %}
                <div class="row">
            {% endif %}
            <div class="col-md-6">
                <div class="field-name">
                    <label class="required">{{ fieldForm.children.value.vars.label }}:</label>
                </div>
                {{ form_errors(fieldForm.value) }}
                {{ form_widget(fieldForm.value, {
                    'contentData': form.vars.data
                }) }}
            </div>
            {% if fieldIdentifier == 'last_name' %}
                </div>
            {% endif %}
        {% endif %}

        {% if fieldIdentifier == 'user_account' %}
            <div class="row">
                <div class="col-md-6">
                    {{ form_widget(fieldForm.value, {
                        'contentData': form.vars.data
                    }) }}
                </div>
            </div>
        {% endif %}

        {%- do fieldForm.setRendered() -%}
    {% endfor %}

    <div class="row">
        <div class="col-md-4 col-md-offset-4">
            {{ form_widget(form.register, {'attr': {
                'class': 'btn btn-block btn-primary'
            }}) }}
        </div>
    </div>

    {{ form_end(form) }}
{% endmacro %}
```

On peut ajouter une ultime confirmation du formulaire par le biais d'une seconde vue *(3ème ici)*, configurable depuis le fichier ``views.yaml`` :

```yaml
user_registration:
    templates:
        form: user/registration_form.html.twig
	    # On associe la confirmation à un template précis
        confirmation: user/registration_confirmation.html.twig 
```

Et enfin on ajoute le Template de la confirmation : 
```twig
{% extends "main_layout.html.twig"  %}

{% block page_head %}
  {% set title = 'Registration complete'|trans %}

  {{ parent() }}
{% endblock %}

{% block content %}
    <div class="container">
        <section class="user-form-confirmation col-md-4 col-md-offset-4">
            <h2>{{ 'Registration completed'|trans }}</h2>

            <div class="row confirmation-label">
                {{ 'You\'re all set up and ready to go'|trans }}
            </div>

            <div class="row">
                <div class="col-md-4 col-md-offset-4">
                    <button type="button" class="btn btn-block btn-primary" onclick="window.location='{{ path('login') }}';">{{ 'Log in'|trans }}</button>
                </div>
            </div>
        </section>
    </div>
{% endblock %}
```

### Permissions

#### Créer un **user group**

Pour ajouter des permissions à un groupe (**user group**) :

- ``Admin`` →  ``Users`` → ``+ Create content (Create Users)`` 
- Créer un user group (*ex: Go Bike Members*)

<div style="width: 100%; text-align:center; margin: 10px auto">
<img src="create_user_group.png" style="width: 650px">
</div>

#### Créer un **dossier de contribution**

On va créer un dossier qui sera le seul dans lequel les titulaires du user group **Go Bike Members** pourront créer et ajouter du contenu

``Content`` →  ``Content structure`` → ``+ Create content`` → ``Folder`` (ex: nom = **Member Rides**)
#### Attribuer les **permissions**

``Admin`` →  ``Roles``→  ``+ Create`` puis nommer le **role** (*ex: Contributeur*)

Assigner des droits (*les suivants sont génériques*) :

- `User / Login`  
- `User / Password` 
- `Content / Read`  
- `Content / VersionRead`  
-  `Content / Create`  
  ↳ Limité aux types : `Ride`, `Landmark`  *(exemple de content que le rôle peut **créer**)*
  ↳ Sous-arbre : `Member Rides`
- `Content / Publish`  
  ↳ Limité aux types : `Ride`, `Landmark`  *(exemple de content que le rôle peut **publier**)*
  ↳  Sous-arbre : `Member Rides`
- `Content / Edit`  
  ↳  Limité au contenu : `Owner = Self`
- `Section / View`
- `Content / ReverseRelatedList`

Enfin, pour assigner le **rôle** aux **permissions**, on se rend dans l'onglet `Assigments` du rôle créé puis on sélectionne `Assign to Users/Groups` puis on clique sur `Select User Groups`

On déroule l'onglet **Users** pour choisir le rôle souhaité :

<div style="width: 100%; text-align:center; margin: 10px auto">
<img src="select_user_role.png" style="width: 500px">
</div>

