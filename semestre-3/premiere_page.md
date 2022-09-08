# Séance XX : Première page avec Symfony

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
    public function number()
    {
        $number = random_int(0, 100);

        return new Response(
            'Lucky number: '.$number
        );
    }
}
```

Ce code est votre premier contrôleur (à déposer dans `src/Controller`). Ce contrôleur est composé d'une méthode qui calcul un nombre aléatoire et retourne une réponse qui est du code HTML. Ce code n'utilise pas directement les vues de Symfony, et ne fonctionne pas (en tout cas il n'est pas possible de l'appeler), car il n'est pas lié à une route.

## Route

Il faut définir les routes. Il existe de nombreuses méthodes (yaml, xml, php, annotations, attributs (php8)) Pour information voici la syntaxe en YAML, à mettre dans le fichier **routes.yaml**.

```yaml
# config/routes.yaml

# the "app_lucky_number" route name is not important yet
app_lucky_number:
    path: /lucky/number
    controller: App\Controller\LuckyController::number
```

Le site sera accessible à cette adresse sur votre serveur local

[http://localhost/NomDuProjet/public/index.php/lucky/number](http://localhost:8000/lucky/number) (WampServeur - Windows)

[http://localhost:8888/NomDuProjet/public/index.php/lucky/number](http://localhost:8000/lucky/number) (Mamp - MacOs)

[http://localhost:8000/lucky/number](http://localhost:8000/lucky/number) (Serveur embarqué avec Symfony CLI)

{% hint style="danger" %}
**Nous n'utiliserons pas cette solution pour la gestion des routes, pour des raisons de confort.**
{% endhint %}

## Autre solution

Nous allons utiliser les annotations, qui permettent une syntaxe plus simple, et une proximité entre la définition de la route et la définition de la méthode. Pour cela, il faut installer les annotations à Symfony avec la commande suivante :

```
composer require annotations
```

Et modifier le contrôleur précédent en intégrant directement la route sous forme d'une annotation.

```php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class LuckyController
{
    #[Route("/lucky/number", name:"app_lucky_number")]
    public function number()
    {
        $number = random_int(0, 100);

        return new Response(
            'Lucky number: '.$number
        );
    }
}

```

{% hint style="info" %}
La syntaxe avec "#" se nomme un attribut au sens de PHP8 et supérieur. Cette syntaxe ne fonctionne pas avec une version PHP inférieure à PHP8. Auparavant vous devez utiliser la syntaxe suivante (dite annotations) :&#x20;

```
/**
* @Route("/lucky/number", name="app_lucky_number")
*/
```
{% endhint %}

## Ajout d'une vue

Cette solution fonctionne, mais écrire tout le code HTML dans la méthode n'est pas très pratique. Nous devons donc écrire des vues. Par défaut, Symfony utilise [**Twig**](https://twig.symfony.com/). Pour cela, il faut l'installer

```
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
    /**
     * @Route("/lucky/number", name="app_lucky_number")
     */
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
