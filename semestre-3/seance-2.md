# Séance 2 : Architecture de Symfony & première page

Depuis la version 4, Symfony à grandement simplifié la structure de ses répertoires et à normalisé les appellations pour coïncider avec la pratique de la majorité des framework.

Symfony à également abandonné la notion de Bundle, qui était nécessaire pour développer son projet.

## Architecture globale

![Arborescence de base d'un projet Symfony](../.gitbook/assets/arbo1.png)

* **bin/** : contient la console
* **config/** : contient les configurations des différents bundles, les routes, la sécurité et les services
* **public/** : c'est le répertoire public et accessible du site. Contient le contrôleur frontal "index.php" et les "assets" (css, js, images, ...)
* **src/** : contient votre projet (**M et C** du MVC)
* **templates/** : contient les vues (**V** du MVC)
* **var/** : contient les logs, le cache
* **vendor/** : contient les sources de bundles tiers et de Symfony

## Configuration

![Arborescence du répertoire de configuration](../.gitbook/assets/arbo-config.png)

Vous remarques des fichiers au format yaml, le format par défaut pour la configuration de Symfony. Vous pouvez aussi remarquer les répertoires dev, prod et test. Ces répertoires permettent de facilement différencier une configuration pour un environnement de développement, pour le site en ligne, ou encore pour un contexte de test.

Par exemple, lorsque l'on développe, on ne souhaite pas réellement envoyer les mails aux usagers, par contre, on souhaite les visualiser. On peut donc configurer ce comportement dans le répertoire dev, pour le bundle de gestion des emails. Un autre exemple est la gestion des logs. En dev, on ne souhaite pas les conserver, mais les afficher, en production, on ne souhaite surtout pas les afficher, mais les stocker dans des fichiers.

Le concept est donc de définir le comportement global quelque soit l'environnement dans le fichier à la racine de config/packages, et de spécifier un comportement en fonction de l'environnement dans les répertoire dev, prod ou test.

## Src

![Configuration classique du répertoire src/](../.gitbook/assets/arbo-src.png)

Dans ce répertoire se trouve toute la logique de l'application, les contrôleurs, les modèles, nos classes spécifiques, les services, ...

* Controller/ : Tous les contrôleurs de l'application, accessibles par des routes
* Entity/ : Les classes spécifiques pour la gestion de l'application, des données et des API
* Migrations/ : Contient les fichiers permettant la mise à jour de la base de données pour chaque modification effectuée sur la structure.
* Repository/ : Est très généralement lié à une entité, et permet d'ajouter des requêtes spécifiques.

Tous ces points seront détaillés dans les parties suivantes.

## Principe général de fonctionnement de Symfony

<figure><img src="../.gitbook/assets/Capture d’écran 2023-09-05 à 21.54.00.png" alt=""><figcaption><p>Principe général de fonctionnement de Symfony (source : <a href="https://symfony.com/doc/current/introduction/http_fundamentals.html#the-symfony-application-flow">https://symfony.com/doc/current/introduction/http_fundamentals.html#the-symfony-application-flow</a>)</p></figcaption></figure>

## Première page avec Symfony

Pour créer une page dans Symfony, il faut, au minimum :

* Une route : pour faire le lien entre une URL et une méthode d'un contrôleur
* Un contrôleur : qui contient des méthodes, chacune, en générale, associée à une route
* Une méthode : permet l'execution d'une action précise, généralement en lien avec une route

### Premier controller

```php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;

class LuckyController
{
    public function number(): Response
    {
        $number = random_int(0, 100);

        return new Response(
            'Lucky number: '.$number
        );
    }
}
```

Ce code est votre premier contrôleur (à déposer dans `src/Controller`). Ce contrôleur est composé d'une méthode qui calcul un nombre aléatoire et retourne une réponse. Ce code n'utilise pas directement les vues de Symfony, et ne fonctionne pas (en tout cas il n'est pas possible de l'appeler), car il n'est pas lié à une route.

### Définir une route

Nous allons utiliser les Attributes, qui permettent une syntaxe plus simple, et une proximité entre la définition de la route et la définition de la méthode (il existe d'autres façons de faire, qui peuvent avoir leurs avantages et leurs inconvenients et dépendre des pratiques de l'entreprise).

Les Attributes, contrairement aux annotations, ne nécessitent pas de dépendances supplémentaires. En effet les Attributes sont maintenant natifs en PHP 8.0, et sont supportés par Symfony depuis la version 5.1.

Modifions le contrôleur précédent en intégrant directement la route sous forme d'Attribute.

```php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class LuckyController
{
    #[Route("/lucky/number", name:"app_lucky_number")]
    public function number(): Response
    {
        $number = random_int(0, 100);

        return new Response(
            'Lucky number: '.$number
        );
    }
}
```

Pour accéder a cette route, il faut maintenant taper l'URL suivante : `http://localhost:8000/lucky/number` si vous executez le serveur de développement de Symfony. Sinon, il faut taper l'URL suivante : `http://localhost:8000/public/index.php/lucky/number`. **La partie locahost:8000 est à adapter en fonction de votre configuration.**

Si vous testez cette URL, vous devriez voir affiché le nombre aléatoire. Cependant, regardez attentivement et vous verrez que ce qui est renvoyé n'est pas du HTML, mais du texte brut. C'est parce que nous n'utilisons pas encore les vues de Symfony, et que nous retournons directement une réponse sans plus d'information que cela sur le format retourné.

### Ajout d'une vue

Cette solution fonctionne, mais écrire tout le code "HTML" dans la méthode n'est pas très pratique. Nous devons donc écrire des vues. Par défaut, Symfony utilise [**Twig**](https://twig.symfony.com/). Pour cela, il faut l'installer

```bash
composer require twig
```

<pre data-overflow="wrap"><code><strong># Uniquement si vous n'avez pas installé le projet avec toutes les dépendances --webapp
</strong></code></pre>

Il faut ensuite modifier le contrôleur pour utiliser les vues.

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class LuckyController extends AbstractController
{
    #[Route("/lucky/number", name:"app_lucky_number")]
    public function number(): Response
    {
        $number = random_int(0, 100);

        return $this->render(
            'lucky/number.html.twig',
            [
                'number' => $number
            ]
        );
    }
}

```

Il faut maintenant écrire la vue.

```twig
{# templates/lucky/number.html.twig #}
<h1>Your lucky number is {{ number }}</h1>
```

**Et Voilà !**

### **Création avec la console**

Symfony propose un outil qui se nomme "maker" qui permet de générer du code pour nous. Le code produit est très générique, ne correspond pas forcément exactement à vos besoin, mais permet d'avoir une première base de travail et une structure pour vos différents fichiers.

Pour installer le maker (si vous n'avez pas installé un projet avec toutes les dépendances --wepabb), en étant dans votre projet :

`composer require maker --dev`

**ou**

`symfony composer require maker --dev`

Ensuite pour utiliser le maker, dans le projet saisir la commande suivante

`bin/console make:controller NomDuController`

Cette commande va générer un controller nommé "NomDuController" (dans src/controller) et la vue associée dans le repertoire templates/NomDuController, avec une méthode index.

{% hint style="info" %}
Pour ajouter d'autres méthodes, vous devrez le faire directement dans le controller, le maker ne permet pas d'en ajouter dans un fichier existant.
{% endhint %}
