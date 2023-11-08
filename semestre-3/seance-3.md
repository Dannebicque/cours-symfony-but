# Séance 3 et 4 : Controller, Routes et vues : les bases de notre fil rouge

## Objectifs

Tout au long des prochaines séances nous allons mettre en place un système de blog simple permettant de poster des articles, d'obtenir des commentaires et de gérer les accès avec des droits différenciés.

Ce projet va évoluer au fur et à mesure des connaissances de Symfony.

{% hint style="info" %}
Les concepts vont être vus progressivement selon nos besoins pour le projet. Ils ne seront donc jamais décris de manière exhaustive dans ce cours. Vous pouvez avoir l'ensemble des détails dans la documentation officielle de Symfony : [https://symfony.com/doc/current/index.html](https://symfony.com/doc/current/index.html)

Par ailleurs vous pouvez aussi trouver un exemple complètement décrit de manière progressive dans le livre de Fabien Pontetier : [https://symfony.com/book](https://symfony.com/book)
{% endhint %}

## Présentation des routes et des contrôleurs

**Une route permet de diriger une url (ou un pattern d'url) vers une méthode de controller appelée Action.**

[La documentation officielle de Symfony sur les routes](https://symfony.com/doc/current/routing.html) et [La documentation officielle de Symfony sur les controllers](https://symfony.com/doc/current/controller.html)

Il est possible de décrire des routes selon les formats de fichiers : XML, Classe PHP, en annotation ou avec les Attributes (php8 et +). Pour plus de commodité nous utiliserons **les Attributes**.

## Structures d'une route

Une route peut être **constante** : `/blog` ou **dynamique** : `/blog/{slug}`. Ici slug englobé de **{ }** devient une variable dynamique qui prend tous les caractères alphanumériques par exemple : `/blog/42`, `/blog/lorem-ipsum`, `/blog/titi-32\_tata` Ces 3 urls correspondent à la méthode ciblée par la route avec une variable `slug` différente. Cette variable peut être récupérée par le controller.

```php
#[Route('/blog/{slug}', name:'article_blog')]
public function article($slug)
{
    ...
}
```

* Une route est au minimum un **chemin (path)** et un **nom**;
* Ces variables peuvent être mises par défaut grâce à "defaults"
* Ces variables peuvent être soumises à une validation de format via "requirements"

```php
#[Route('/{id}', name:'index', requirements:['id'=>'\d+'], defaults:['id'=>1])]
public function index($id)
{
    ...
}
```

* On peut définir un path différent en fonction de la locale
* Vous pouvez cumuler plusieurs routes pour une méthode Action
* On peut spécifier le moyen d'accès à une route (GET, POST, PUT, ...), avec le paramètre "methods"

## Attributes

Pour pouvoir utiliser les Attributes il faut :

```php
use Symfony\Component\Routing\Annotation\Route;
```

à ajouter après le namespace dans votre Controller.

## Routes et paramètres

### Un paramêtre

On définit une variable d'url via des accolades {ma\_variable} :

```php
#[Route('/page/{page}', name:'blog_index')]
public function indexAction($page)
{
    echo $page;
}
```

Pour récupérer la variable dans le controller, il suffit de déclarer une variable dans la méthode avec le même nom que la variable d'url.

### Valeur par défaut

Pour définir une valeur par défaut, on peut préciser la valeur sur le paramêtre de la méthode :

```php
#[Route('/page/{page}', name:'blog_index')]
public function indexAction($page = 1)
{
    echo $page;
}
```

Si l'utilisateur utilise l'URL `/page/`, la variable `$page` sera égale à 1.

### plusieurs paramètres

On peut cumuler plusieurs variables :

```php
#[Route('/page/{page}/{subpage}', name:'blog_index')]
</strong>public function indexAction($page, $subpage)
{
    echo $page.' '.$subpage;
}
```

On pourrait utiliser un autre séparation que le slash, comme par exemple -, . ou \_. La seule condition étant que Symfony puisse être en mesure de différencier les paramètres.

## Génération d'url

Pour générer une url en PHP on utilise :

```php
$this->generateUrl('nom_de_la_route', [$variables]);
```

Ou sous TWIG on a deux fonctions :

```php
{{ path('nom_route', {'page': 'toto', 'vars2': 'titi'} ) }}

{{ url('nom_route', {'page': 'toto', 'vars2': 'titi'} ) }}
```

## Controller

Les routes redirigent vers une méthode d'un contrôleur (une action); un controller Symfony se nomme de la sorte : NomDuController où le suffixe **Controller** est obligatoire et le nom du fichier et de la classe est en **CamelCase**.

Les différentes méthodes se nomment de la sorte :

**nomDeLaMethode** est lui en **miniCamelCase**

Dans le cas idéal, le controller doit contenir le minimum de code possible (20 lignes maximum selon les standards de Symfony). Il ne doit que faire le lien entre les différents éléments de l'application et une réponse.

## Response

Une méthode d'un contrôleur renvoie toujours un type _Response_ ; il existe plusieurs type de Response : JsonResponse, RedirectResponse, HttpResponse, BinaryFileResponse etc ...

La plus utilisée est Response pour l'utiliser on va ajouter dans le "use" suivant :

```php
use Symfony\Component\HttpFoundation\Response;
```

et dans la méthode action on :

```php
public function index(){
    return new Response('Ma response');
}
```

Affiche à l'écran Ma response. Dans l'état cette réponse n'est pas du HTML, car rien n'est précisé dans le retour de la méthode.

Une méthode render() (définie quand la classe **AbstractController** dont votre controller doit hériter) permet aux méthodes de récupérer une vue et d'afficher le contenu de la vue compilée avec les différentes variables envoyées.

```php
#[Route("/", name:"page")]
public function index() 
{
    // votre code

    return $this->render('default/index.html.twig', 
        ['variable' => $variable]
    );
}
```

Ici on va récupérer le template présent dans templates/default/index.html.twig pour affecter la variable _variable_.

### Exercice 1

Les premières étapes consistent à mettre en place les bases de notre projet :

* Une nouvelle installation de Symfony, pour un projet nommé blog
  * On installera toutes les dépendances en une seule fois pour plus de confort : `composer require webapp`dans le dossier blog/
* Un contrôleur et la méthode pour la page d'accueil et la route "/"
* La vue associée à la page d'accueil
* Une page de contact avec une route "/contact" et une vue associée
* Testez vos routes et vos vues, ajoutez un contenu fictif simple pour le moment avec uniquement du HTML comme vous savez le faire depuis le S1.

## Les vues/templates

Un template ou une vue est le meilleur moyen d'organiser et de restituer le code HTML à partir de votre application, que vous deviez rendre le code HTML à partir d'un contrôleur ou générer le contenu d'un courrier électronique . Les templates dans Symfony sont créés avec Twig: un moteur de modèle flexible, rapide et sécurisé.

[Twig](https://twig.symfony.com/) est un moteur de rendu de template comme [Smarty](https://www.smarty.net/) (prestashop) ou [Blade](https://laravel.com/docs/5.8/blade) (laravel). Twig a cependant été développé pour Symfony à l'origine et peut être utilisé dans d'autres contextes.

Les vues sont des fichiers HTML qui sont compilés par Symfony pour être envoyés au navigateur. Elles sont stockées dans le dossier `templates` et sont généralement organisées par contrôleur.

{% hint style="info" %}
Un moteur de template permet de limiter les logiques complexes pour réaliser des templates simples à coder. Un moteur de template intègre généralement des fonctionnalités qui sont récurrentes dans le développement "front" et qui permettent de simplifier le code à écrire.
{% endhint %}

## Twig

Twig est le moteur de template par défaut de Symfony. Il permet de faire des boucles, des conditions, des inclusions, des héritages, des filtres, des fonctions, etc. Il est très complet et permet de faire des choses très puissantes. Twig est un langage à part entière, mais il est très simple à apprendre et à utiliser. Twig s'intègre dans le code HTML, il sera ensuite interprété par le moteur Symfony pour générer la page HTML finale.

Twig est un outil très puissant, mais il implique une courte phase d'apprentissage, car contrairement à d'autres moteurs de template il n'utilise pas une syntaxe PHP. Vous pouvez vous référer à la documentation officielle : [https://twig.symfony.com/doc/3.x/](https://twig.symfony.com/doc/3.x/)

### Exemple d'une vue avec twig

Exemple d'un code que vous pourriez écrire en PHP

```php
<body>
    <h1><?php echo $page_title ?></h1>
    <ul id="navigation">
        <?php foreach ($navigation as $item): ?>
        <li>
                <a href="<?php echo $item->getHref() ?>">
                    <?php echo $item->getCaption() ?>
                </a>
            </li>
        <?php endforeach ?>
    </ul>
</body>
```

Ce même code, écrit avec TWIG serait :

```twig
<body>
    <h1>{{ page_title }}</h1>

    <ul id="navigation">
        
{% raw %}
{% for item in navigation %}
            <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
        {% endfor %}
{% endraw %}

    </ul>
</body>
```

La syntaxe est plus "legére" et moins encombrées des balises PHP. Le code semble donc plus lisible et par conséquent plus facile à maintenir.

### Affichage

La syntaxe Twig est basée sur uniquement trois constructions:

* `{{ ... }}`, utilisé pour afficher le contenu d'une variable ou le résultat de l'évaluation d'une expression;
* `{% ... %}`, utilisé pour exécuter une logique, telle qu’une condition ou une boucle;
* `{# ... #}`, utilisé pour ajouter des commentaires au modèle (contrairement aux commentaires HTML, ces commentaires ne sont pas inclus dans la page rendue).

{% hint style="danger" %}
Vous ne pouvez pas exécuter de code PHP dans les modèles Twig, mais Twig fournit des utilitaires permettant d'exécuter une certaine logique dans les modèles. Par exemple, les **filtres** modifient le contenu avant le rendu, comme le `upper`filtre pour mettre le contenu en majuscule :

`{{ title|upper }}`

Twig intègre une [liste de filtre](https://twig.symfony.com/doc/2.x/) permettant de répondre aux usages courant. Mais il est très simple d'ajouter nos propres filtres afin de répondre parfaitement à nos besoins.
{% endhint %}

### Variables

TWIG est très puissant lorsqu'il s'agit de manipuler des variables. Pour lui, si c'est un tableau associatif ou un objet avec des propriétés cela est identique dans la syntaxe.

Par exemple si vous avez le tableau `$tab['param1']` vous écrirez en TWIG

```twig
{{ tab.param1 }}
```

Si vous avez un objet `$tab`, qui est une instance d'une classe ayant comme propriété `$param1`, la syntaxe en TWIG sera :

```twig
{{ tab.param1 }}
```

Si vous avez un objet `$tab`, qui est une instance d'une classe ayant comme propriété `$param1`, qui est un tableau associatif avec une clé `key1`, la syntaxe pourrait être :

```twig
{{ tab.param1.key1 }}
```

La manière dont TWIG inspecte votre code pour trouver la meilleure façon d'interpréter une variable est la suivante :

For convenience's sake `foo.bar` does the following things on the PHP layer:

* check if foo is an array and bar a valid element;
* if not, and if foo is an object, check that bar is a valid property;
* if not, and if foo is an object, check that bar is a valid method (even if bar is the constructor - use \_\_construct() instead);
* if not, and if foo is an object, check that getBar is a valid method;
* if not, and if foo is an object, check that isBar is a valid method;
* if not, and if foo is an object, check that hasBar is a valid method;
* if not, return a null value.

foo\['bar'] on the other hand only works with PHP arrays:

* check if foo is an array and bar a valid element;
* if not, return a null value.

### Héritage

TWIG permet l'héritage de template via un `extends` dans les templates enfants :

```twig
{% raw %}
{% extends 'base.html.twig' %}
{% endraw %}

```

Dans les templates mère on définit des "block" que l'on vient surcharger dans les templates enfants :

```twig
{% raw %}
{% block body %}
toto
{% endblock %}
{% endraw %}
```

On peut également reprendre le block parent via

```twig
{{ parent() }}
```

On crée donc des templates mère assez flexibles pour pouvoir en hériter et surcharger les différents blocks

## Exercice 2

En se basant sur les concept d'héritage, définir un menu avec nos deux pages (accueil et contact) dans le fichier base.html.twig et l'hériter dans les deux autres templates.

## Images et CSS/JavaScript

Pour inclure des images, des fichiers CSS ou JavaScript, il faut utiliser la fonction asset() :

```twig
<img src="{{ asset('images/logo.png') }}" alt="Symfony!" />
```

Cette fonction va chercher le fichier dans le dossier public/images/logo.png

Asset n'est pas installé par défaut, il faut donc l'installer :

```bash
composer require symfony/asset
```

Vous pouvez utiliser cette fonction pour inclure des fichiers CSS ou JavaScript :

```twig
<link rel="stylesheet" href="{{ asset('css/blog.css') }}" />
```

## Exercice 3

* Créer un dossier css dans le dossier public
* Créer un fichier style.css dans le dossier css, faites un peu de CSS pour mettre en forme votre page d'accueil
* Ajouter une balise link dans le fichier base.html.twig pour inclure le fichier css
* Ajouter une balise img dans le fichier index.html.twig de votre template de la page d'accueil pour inclure une image de votre choix&#x20;

## Logique

* `{% %}` permet d'utiliser des logiques tels que :
  * `{% if %} {% else %} {%endif %}` : condition
  * `{% for item in items %}{%endfor}` : foreach
  * `{% set foo='foo' %}`  : set des variables

### Tests

Il est possible d'écrire des tests, la syntaxe est très proche de celle de PHP. Attention toutefois, les opérateurs de comparaison ne s'écrive pas de manière identique.

Le && de PHP s'écrit "and" dans TWIG, et le || de PHP s'écrit "or" dans TWIG.

Il est possible d'avoir des _elseif_ (autant que nécessaire) et un bloc _else_ (1 au maximum)

```twig
{% raw %}
{% if condition %}

{% else if condition %}

{% else %}

{% endif %}
{% endraw %}
```

### Boucles

Il n'existe que la boucle for dans TWIG (elle est un peu l'équivalent d'un foreach).

Le code ci-dessous est une boucle qui varie de 1 à 10.

```twig
{% raw %}
{% for i in 1..10 %}

{% endfor %}
{% endraw %}
```

La boucle ci-dessous est une boucle qui parcours une "collection" users (l'équivalent du foreach). **Attention la syntaxe est inversée par rapport au PHP**

```twig
<ul>
    {% raw %}
{% for user in users %}
        <li>{{ user.username }}</li>
    {% endfor %}
{% endraw %}
</ul>
```

Dans le cadre d'une boucle TWIG propose une variable nommée loop qui permet d'avoir des informations sur la boucle :

| Variable       | Description                                                   |
| -------------- | ------------------------------------------------------------- |
| loop.index     | The current iteration of the loop. (1 indexed)                |
| loop.index0    | The current iteration of the loop. (0 indexed)                |
| loop.revindex  | The number of iterations from the end of the loop (1 indexed) |
| loop.revindex0 | The number of iterations from the end of the loop (0 indexed) |
| loop.first     | True if first iteration                                       |
| loop.last      | True if last iteration                                        |
| loop.length    | The number of items in the sequence                           |
| loop.parent    | The parent context                                            |

La boucle ci-dessous intègre un test. Ce qui simplifie l'écriture. Elle intègre également un else dans le cas ou la boucle ne ferait aucune itération. Il est possible d'utiliser le else sans le if et réciproquement.

```twig
<ul>
    {% raw %}
{% for user in users|filter(user => (user.active == true)) %}
        <li>{{ user.username }}</li>
    {% else %}
        <li>No users found</li>
    {% endfor %}
{% endraw %}
</ul>
```

Le code ci-dessus serait équivalent en PHP au code suivant:

```php
<?php
<ul>
if (count($users) > 0) {
    foreach ($users as $user) {
        if ($user->isActive() == true) {//isActive() est une méthode de mon objet $user
            echo '<li>'.$user->getUsername().'</li>';
        }
    }
}
else {
    echo '<li>No users found</li>';
}
</ul>
```

## Exercice 4

* Ajouter une page /articles
* Construire un tableau PHP contenant 3 articles, contenant chacun un titre, un texte et une date
* Passer le tableau à la vue de la page article.
* Afficher le tableau en twig

Exemple de tableau PHP



```php
<?php
$articles = [
1 => ['titre' => 'Mon premier article',
      'texte' => 'Un texte de mon article un peu long...',
      'date' => new DateTime('now')
      ],
 ...
 ];
 ?>      
```

## Exercice 5

* Ajoutez une nouvelle page sur l'url /recherche
* Ajouter une vue avec un formulaire avec une zone de saisie et un bouton submit. Le traitement se fera en poste sur l'url /recherche/resultat
* La vue résultat affichera le contenu de la zone de texte saisie dans la page précédente.
