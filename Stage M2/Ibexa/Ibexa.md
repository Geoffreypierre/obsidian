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

### Redéfinir le Template d'un formulaire

Lorsqu'on a créé un formulaire depuis le back-office, on peut être amené à modifier son template ou bien ajouter des règles pour la validation telles que les messages d'erreur retourné par le CAPCHA. POur cela on se rend dans : 

``Content`` →  ``Forms``→ ``+ Create content (Form)`` puis ajouter les champs souhaités
<div style="width: 100%; text-align:center; margin: 10px auto">
<img src="form_create.png" style="width: 500px">
</div>

On comment par définir le fichier `newsletter.html.twig` dans le dossier `templates/blocks/form/` 

```twig
<div class="row">
    <div class="block-form {{ block_class }}">
        {{ ibexa_http_cache_tag_relation_location_ids(locationId) }}
        {{ render(controller('ibexa_content:viewAction', {
            'contentId': contentId,
            'locationId': locationId,
            'viewType': 'embed'
        })) }}
        <style type="text/css">{{ block_style|raw }}</style>
    </div>
</div>
```

Pour que ce block soit visible depuis le mode édition du back-office, on ajoute la configuration dans le fichier `config/packages/ibexa_fieldtype_page.yaml` juste au dessous de `block:` :

```yaml
form:
	views:
		default:
			template: blocks/form/newsletter.html.twig
			name: Newsletter Form View
```

Enfin, se rendre dans l'onglet `Design` du formulaire placé et sélectionner `Newsletter Form View`

<div style="width: 100%; text-align:center; margin: 10px auto">
<img src="newsletter-bo.png" style="width: 500px">
</div>

Il faut également changer les champs du formulaire :

Créer le fichier `form_field.html.twig` dans `templates/fields/` :

```twig
{% block ezform_field %}
    {% set formValue = field.value.getForm() %}
    {% if formValue %}
        {% set form = formValue.createView() %}
        {% form_theme form 'bootstrap_4_layout.html.twig' %}
        {% apply spaceless %}
            {% if not ibexa_field_is_empty(content, field) %}
                {{ form(form) }}
            {% endif %}
        {% endapply %}
    {% endif %}
{% endblock %}
```

Dans `config/packages/views.yaml`, au même niveau que `pagelayout` ajouter :
```yaml
field_templates:
    - { template: fields/form_field.html.twig, priority: 30 }
```

Pour observer le rendu, effectuer un `bin/console cache:clear`. Afin d'ajouter les ultimes configuration du CAPCHA, créer le fichier `config/packages/gregwar_captcha.yaml` et ajouter  :

```yaml
gregwar_captcha:
    width: 150
    invalid_message: Please, enter again.
    length: 4
```

SI le css a été modifié, relancer les assets et videz le cache :

*(optionnel)*
```bash
yarn encore dev
```
puis
```bash
php bin/console c:c
```

### Créer un `type de champ` personnalisé

Le type de champ générique est un point d'extension puissant qui vous permet de concevoir des solutions complexes à partir d'un modèle de champ prêt à l'emploi.

Les types de champs sont responsables de :

- Le stockage des données, soit à l'aide des mécanismes natifs du moteur de stockage, soit par des moyens spécifiques
- Validation des données d'entrée
- Rendre les données consultables (le cas échéant)
- Affichage des champs

#### Créer un type de champ **Point 2D**

On créée le **modèle** *(par exemple dans `src/FieldType/Point2D/Value.php`)*correspondant au type de champ comprenant attribut *(x et y)*, getters et setters :

```php
<?php
declare(strict_types=1);

namespace App\FieldType\Point2D;

use Ibexa\Contracts\Core\FieldType\Value as ValueInterface;

final class Value implements ValueInterface
{
    /** @var float|null */
    private $x;

    /** @var float|null */
    private $y;

    public function __construct(?float $x = null, ?float $y = null)
    {
        $this->x = $x;
        $this->y = $y;
    }

    public function getX(): ...

    public function setX(?float $x): ...

    public function getY(): ...

    public function setY(?float $y): ...

    public function __toString()
    {
        return "({$this->x}, {$this->y})";
    }
}
```

#### Définir le type de champ

On créée ensuite un **Identificateur de type de champ** *(par exemple dans `src/FieldType/Point2D/Type.php`)*. 

```php
<?php
declare(strict_types=1);

namespace App\FieldType\Point2D;

use Ibexa\Contracts\Core\FieldType\Generic\Type as GenericType;

final class Type extends GenericType
{
    public function getFieldTypeIdentifier(): string
    {
        return 'point2d';
    }
}
``` 
Cette méthode va identifier de manière unique le type de champ.
Il faut ensuite ajouter cette nouvelle définition dans le fichier de configuration `config/services.yaml` :
```yaml
services:
    App\FieldType\Point2D\Type:
        tags:
            - { name: ibexa.field_type, alias: point2d }
```

#### Définir le formulaire de modification du type de champ

On créée ensuite un **Formulaire** pour **modifier** le type de champ. On ajoute ainsi le fichier `src/Form/Type/Point2DType.php`. Cette classe étend de AbstractType *(comme en symfony standard)* et de la méthode **buildForm()** :

```php
<?php
declare(strict_types=1);

namespace App\Form\Type;

use App\FieldType\Point2D\Value;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\NumberType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

final class Point2DType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder->add('x', NumberType::class);
        $builder->add('y', NumberType::class);
    }
}
```

On doit ensuite modifier la classe **Type.php** afin d'ajouter une interface de mappage de formulaire. FormMapper ajoute les définitions de champs aux formulaires Symfony à l'aide de la `add()`méthode appropriée. `FieldValueFormMapperInterface`Un formulaire d'édition est ensuite disponible pour chaque type de champ dans l'interface d'administration.

```php
<?php
declare(strict_types=1);

namespace App\FieldType\Point2D;

use App\Form\Type\Point2DType;
use Ibexa\Contracts\ContentForms\Data\Content\FieldData;
use Ibexa\Contracts\ContentForms\FieldType\FieldValueFormMapperInterface;
use Ibexa\Contracts\Core\FieldType\Generic\Type as GenericType;
use Symfony\Component\Form\FormInterface;

final class Type extends GenericType implements FieldValueFormMapperInterface
{
    public function getFieldTypeIdentifier(): string
    {
        return 'point2d';
    }

    public function mapFieldValueForm(FormInterface $fieldForm, FieldData $data): void
    {
        $definition = $data->fieldDefinition;
        $fieldForm->add('value', Point2DType::class, [
            'required' => $definition->isRequired,
            'label' => $definition->getName(),
        ]);
    }
}
```

Enfin, ajoutez une `configureOptions`méthode et définissez la valeur par défaut de `data_class`to `Value::class`dans `src/Form/Type/Point2DType.php`. Cela permet à votre formulaire de fonctionner sur cet objet :

```php
public function configureOptions(OptionsResolver $resolver): void { $resolver->setDefaults([ 'data_class' => Value::class, ]); }
``` 

Ajouter cette étiquette dans le fichier de configuration services : `config/services.yaml` :
```yaml
- { name: ibexa.admin_ui.field_type.form.mapper.value, fieldType: point2d }
```

#### Introduire le modèle

Pour afficher les données d'un type de champ, vous devez créer et enregistrer un modèle. Chaque modèle de type de champ reçoit un ensemble de variables permettant d'atteindre l'objectif souhaité. Dans ce cas, la variable la plus importante est l' `field`instance de `FieldType` `Ibexa\Contracts\Core\Repository\Values\Content\Field`. Outre ses propres métadonnées (par exemple, `value` `id`ou `fieldDefIdentifier`value), elle expose la valeur du champ via la `value`propriété `value`.

On commence par créer un fichier `templates/point2d_field.html.twig`:
```twig
{% block point2d_field %}
    ({{ field.value.getX() }}, {{ field.value.getY() }})
{% endblock %}
```

Ensuite, indiquez le mappage du modèle dans `config/packages/ibexa.yaml`:

```yaml
ibexa:
    system:
        default:
            field_templates:
                - { template: 'point2d_field.html.twig', priority: 0 }
```

#### Ajouter un nouveau champ Point 2D

``Content`` → ``Content types`` → ``+ Create``  

On crée un **point2d.name**

<div style="width: 100%; text-align:center; margin: 10px auto">
<img src="point_2d.png" style="width: 500px">
</div>
Une fois les détails réglés, on se rend dans : 

``Content`` → ``Content structure`` → ``+ Create``  
On peut ainsi voir apparaître `Point 2D`

<div style="width: 100%; text-align:center; margin: 10px auto">
<img src="point_2d_content.png" style="width: 500px">
</div>

La configuration vous permet de définir le format d'affichage du champ sur la page. Pour ce faire, créez le `format`champ dans lequel vous pourrez modifier l'affichage des coordonnées du Point 2D.
##### Définir le format du type de champ

Dans cette étape , vous créez le `format`champ pour les coordonnées 2D du point. Pour ce faire, vous devez définir une `SettingsSchema`définition. Vous spécifiez également les coordonnées comme valeurs d'espace réservé `%x%`.`%y%`

Ouvrez `src/FieldType/Point2D/Type.php`et ajoutez une `getSettingsSchema`méthode en suivant le bloc de code suivant :

```php
public function getSettingsSchema(): array { 
	return [
		'format' => [
			  'type' => 'string',
			  'default' => '(%x%, %y%)', 
		], 
	]; 
}
``` 

##### Ajouter un champ de format

Il nous faut le formulaire d'édition pour votre type de champ
Créer le fichier `src/Form/Type/Point2DSettingsType.php` pour ajouter un `format` champ :
```php
<?php
declare(strict_types=1);

namespace App\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;

final class Point2DSettingsType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder->add('format', TextType::class);
    }
}
```

Pour autoriser maintenant l'utilisateur à ajouter les coordonnées validées, on modifie `src/FieldType/Point2D/Type.php` pour :
- Implémenter l'interface `FieldDefinitionFormMapperInterface`
- Ajoutez une `mapFieldDefinitionForm`méthode à la fin qui définit les paramètres du champ

```php
<?php
declare(strict_types=1);

namespace App\FieldType\Point2D;

use App\Form\Type\Point2DSettingsType;
use App\Form\Type\Point2DType;
use Ibexa\AdminUi\FieldType\FieldDefinitionFormMapperInterface;
use Ibexa\AdminUi\Form\Data\FieldDefinitionData;
use Ibexa\Contracts\ContentForms\Data\Content\FieldData;
use Ibexa\Contracts\ContentForms\FieldType\FieldValueFormMapperInterface;
use Ibexa\Contracts\Core\FieldType\Generic\Type as GenericType;
use Symfony\Component\Form\FormInterface;

final class Type extends GenericType implements FieldValueFormMapperInterface, FieldDefinitionFormMapperInterface
{
    public function getFieldTypeIdentifier(): string
    {
        return 'point2d';
    }

    public function getSettingsSchema(): array
    {
        return [
            'format' => [
                'type' => 'string',
                'default' => '(%x%, %y%)',
            ],
        ];
    }

    public function mapFieldValueForm(FormInterface $fieldForm, FieldData $data): void
    {
        $definition = $data->fieldDefinition;
        $fieldForm->add('value', Point2DType::class, [
            'required' => $definition->isRequired,
            'label' => $definition->getName(),
        ]);
    }

    public function mapFieldDefinitionForm(FormInterface $fieldDefinitionForm, FieldDefinitionData $data): void
    {
        $fieldDefinitionForm->add('fieldSettings', Point2DSettingsType::class, [
            'label' => false,
        ]);
    }
}
```

##### Ajouter une nouvelle étiquette

Ajouter dans `config/services.yaml` :

```yaml
- { name: ibexa.admin_ui.field_type.form.mapper.definition, fieldType: point2d }
```

##### Définition du type de champ

Pour pouvoir afficher le nouveau `format`champ, vous devez ajouter un modèle. Créer `templates/point2d_field_type_definition.html.twig`

```twig
{% block point2d_field_definition_edit %}
    <div class="{% if group_class is not empty %}{{ group_class }}{% endif %}">
        {{- form_label(form.fieldSettings.format) -}}
        {{- form_errors(form.fieldSettings.format) -}}
        {{- form_widget(form.fieldSettings.format) -}}
    </div>
{% endblock %}
```

##### Ajouter une configuration pour le champ de format

Ajouter dans `config/packages/ibexa.yaml`

```yaml
fielddefinition_edit_templates: - { template: 'point2d_field_type_definition.html.twig', priority: 0 }
```

##### Redéfinir le modèle

Enfin, redéfinissez le modèle Point 2D afin qu'il prenne en compte le nouveau `format`champ.
Remplacez `templates/point2d_field.html.twig`le contenu par :

```twig
{% block point2d_field %}
    {{ fieldSettings.format|replace({
        '%x%': field.value.x,
        '%y%': field.value.y
    }) }}
{% endblock %}
```

##### Modifier le type de contenu

Ajouter le nouveau format `(%x%, %y%)`dans le champ **Format**

<div style="width: 100%; text-align:center; margin: 10px auto">
<img src="change_format.png" style="width: 500px">
</div>

#### Ajouter une validation

On ajoute des contraintes de validation dans `Value.php` :
```php
/**
 * @var float|null
 *
 * @Assert\NotBlank()
 */
private $x;

/**
 * @var float|null
 *
 * @Assert\NotBlank()
 */
private $y;
```

#### Migration des données entre les versions de types de champs

L'ajout d'une migration de données vous permet de modifier le format de sortie des champs afin de répondre à vos besoins actuels. Ce processus est essentiel lorsqu'il est nécessaire de comparer des champs pour le tri et la recherche. La sérialisation permet de convertir des objets en tableaux en les normalisant, puis de les encoder au format sélectionné. Inversement, la désérialisation convertit différents formats en tableaux en les décodant, puis en les dénormalisant pour obtenir des objets.

#### Normalisation
Tout d'abord, vous devez ajouter la prise en charge de la normalisation dans un `src/Serializer/Point2D/ValueNormalizer.php`:

```php
<?php
declare(strict_types=1);

namespace App\Serializer\Point2D;

use App\FieldType\Point2D\Value;
use Symfony\Component\Serializer\Normalizer\NormalizerInterface;

final class ValueNormalizer implements NormalizerInterface
{
    public function normalize($object, string $format = null, array $context = [])
    {
        return [
            $object->getX(),
            $object->getY(),
        ];
    }

    public function supportsNormalization($data, string $format = null)
    {
        return $data instanceof Value;
    }
}
```

##### Ajouter la définition du normalisateur

Ensuite, ajouter la `ValueNormalizer`définition du service à l' `config/services.yaml`aide d'une `serializer.normalizer`balise :

```yaml
services:
    App\Serializer\Point2D\ValueNormalizer:
        tags:
            - { name: serializer.normalizer }
```

##### Compatibilité ascendante

Pour accepter les anciennes versions du type de champ, vous devez ajouter la prise en charge de la dénormalisation dans un `src/Serializer/Point2D/ValueDenormalizer.php`:

```php
<?php
declare(strict_types=1);

namespace App\Serializer\Point2D;

use App\FieldType\Point2D\Value;
use Symfony\Component\Serializer\Normalizer\DenormalizerInterface;

final class ValueDenormalizer implements DenormalizerInterface
{
    public function denormalize($data, string $class, string $format = null, array $context = [])
    {
        if (isset($data['x']) && isset($data['y'])) {
            // Support for old format
            $data = [$data['x'], $data['y']];
        }

        return new $class($data);
    }

    public function supportsDenormalization($data, string $type, string $format = null)
    {
        return $type === Value::class;
    }
}
```

##### Ajouter la définition du dénormalisateur

Ensuite, ajouter la `ValueNormalizer`définition du service à l' `config/services.yaml`aide d'une `serializer.normalizer`balise :

```yaml
services:
    App\Serializer\Point2D\ValueDenormalizer:
        tags:
            - { name: serializer.denormalizer }
```

##### Changer le format à la volée

Pour modifier le format à la volée, vous devez remplacer le constructeur dans `src/FieldType/Point2D/Value.php`:

```php
    public function __construct(array $coords = [])
    {
        if (!empty($coords)) {
            $this->x = $coords[0];
            $this->y = $coords[1];
        }
    }
```

On peut désormais modifier le format de représentation interne du type de champ Point 2D