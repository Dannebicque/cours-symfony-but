# Séance 2 : Première page avec Symfony

Pour créer une page dans Symfony, il faut, au minimum :

* Une route : pour faire le lien entre une URL et une méthode d'un contrôleur
* Un contrôleur : qui contient des méthodes, chacune, en générale, associée à une route
* Une méthode : permet l'execution d'une action précise, généralement en lien avec une route

## Premier controller

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

## Définir une route

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

## Ajout d'une vue

Cette solution fonctionne, mais écrire tout le code "HTML" dans la méthode n'est pas très pratique. Nous devons donc écrire des vues. Par défaut, Symfony utilise [**Twig**](https://twig.symfony.com/). Pour cela, il faut l'installer

```bash
composer require twig
```

Il faut ensuite modifier le contrôleur pour utiliser les vues.

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class LuckyController extends AbstractController
{
    #[Route("/lucky/number", name:"app_lucky_number")]
    public function number()
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

## **Création avec la console**

Symfony propose un outil qui se nomme "maker" qui permet de générer du code pour nous. Le code produit est très générique, ne correspond pas forcément exactement à vos besoin, mais permet d'avoir une première base de travail et une structure pour vos différents fichiers.

Pour installer le maker, en étant dans votre projet :

`composer require maker --dev`

**ou**

`symfony composer require maker --dev`

Ensuite pour utiliser le maker, dans le projet saisir la commande suivante

`bin/console make:controller NomDuController`

Cette commande va générer un controller nommé "NomDuController" (dans src/controller) et la vue associée dans le repertoire templates/NomDuController, avec une méthode index.

{% hint style="info" %}
Pour ajouter d'autres méthodes, vous devrez le faire directement dans le controller, le maker ne permet pas d'en ajouter dans un fichier existant.
{% endhint %}
