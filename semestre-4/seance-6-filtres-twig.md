# Séance 6 : Filtres twig

Twig propose de nombreux filtres permettant d'interagir avec vos variables. Ces filtres sont très utiles pour formater vos données, les transformer, les comparer, etc. Une longue liste est disponible par défaut [https://twig.symfony.com/doc/3.x/](https://twig.symfony.com/doc/3.x/), mais vous pouvez également créer vos propres filtres [https://symfony.com/doc/current/templates.html#writing-a-twig-extension](https://symfony.com/doc/current/templates.html#writing-a-twig-extension).

## Création de votre premier filtre

### Première version

Nous souhaitons formater les prix de notre application dans le format monétaire en euros. Nous pourrions le faire avec le filtre `number_format` de twig, mais nous allons créer notre propre filtre, car nous aurons toujours la même configuration. Cela nous permettra également d'ajouter le symfony euro.

Pour créer un filtre nous devons créer une classe qui va étendre Twig\Extension\AbstractExtension. Cette classe peut se mettre n'importe où dans le dossier src, mais nous allons la mettre dans le dossier src/Twig.

```php
<?php

namespace App\Twig;

use Twig\Extension\AbstractExtension;
use Twig\TwigFilter;

class AppExtension extends AbstractExtension
{
    public function getFilters()
    {
        return [
            new TwigFilter('price', [$this, 'formatPrice']),
        ];
    }

    public function formatPrice($number)
    {
        $price = number_format($number, 2);
        $price = $price.' €';

        return $price;
    }
}
```

Dans cette classe, nous avons créé une méthode `getFilters` qui va nous permettre de définir les filtres que nous souhaitons créer. Cette méthode doit retourner un tableau d'objets `TwigFilter`. Pour chaque filtre, nous devons définir un nom (qui est utilisé dans Twig) et une méthode qui va être appelée lorsque le filtre sera utilisé.

Dans notre exemple, nous avons créé un filtre `price` qui va formater un nombre en ajoutant le symbole euro.

Pour l'utiliser dans Twig, il suffit d'utiliser le nom du filtre :

```twig
{{ 10|price }}
```

### Deuxième version

Nous pouvons améliorer notre filtre afin de pouvoir recevoir des paramètres si nous ne souhaitons pas avoir un format en euro, mais par exemple en dollar. Dans ce cas, le symbole est différent, le séparateur de décimales et de milliers également. Cependant notre cas le plus classique restant l'euro, nous allons mettre ces paramètres en option.

```php
<?php

namespace App\Twig;

use Twig\Extension\AbstractExtension;
use Twig\TwigFilter;

class AppExtension extends AbstractExtension
{
    public function getFilters()
    {
        return [
            new TwigFilter('price', [$this, 'formatPrice']),
        ];
    }

        public function formatPrice($number, $symbol = '€', $decimals = 0, $decPoint = '.', $thousandsSep = ',')
    {
        $price = number_format($number, $decimals, $decPoint, $thousandsSep);
        $price = $price. ' €'; //il faudrait améliorer encore ce code car le symbole ne se met pas à la fin si le nombre est en dollar par exemple.

        return $price;
    }
}
```

Et l'usage dans twig pourrait être

```twig
{{ 10|price }}
{{ 10|price('€', 2, ',', ' ') }}
{{ 10|price('$', 2, '.', ',') }}
```

## Un filtre qui retourne du HTML

Nous allons créer un filtre qui va retourner du HTML. Nous allons créer un filtre qui va permettre d'afficher un nombre d'étoiles en fonction d'une note. Par exemple, si la note est 3, nous allons afficher 3 étoiles pleines, 2 étoiles vides.

```php
<?php

namespace App\Twig;

use Twig\Extension\AbstractExtension;

class AppExtension extends AbstractExtension
{
    public function getFilters()
    {
        return [
            new TwigFilter('stars', [$this, 'stars']),
        ];
    }

    public function stars($note)
    {
        $html = '';
        for ($i = 0; $i < $note; $i++) {
            $html .= '<i class="fas fa-star"></i>';
        }
        for ($i = 0; $i < 5 - $note; $i++) {
            $html .= '<i class="far fa-star"></i>';
        }

        return $html;
    }
}
```

Et l'usage dans twig pourrait être

```twig
{{ 3|stars }}
```

Mais vous aurez une erreur, et un résultat affiché en HTML (en fait Symfony à empéché l'interpréation du HTML car il ne sait pas s'il peut le faire. Cela pourrait potentiellement être dangeureux). Pour cela, nous devons utiliser le filtre `raw` de twig pour forcer l'affichage du HTML.

```twig
{{ 3|stars|raw }}
```

Comme ce filtre va toujours renvoyer un contenu HTML que nous allons vouloir afficher, nous pouvons configurer notre filtre pour qu'il retourne directement du HTML "affiché".

```php
<?php

namespace App\Twig;

use Twig\Extension\AbstractExtension;

class AppExtension extends AbstractExtension
{
    public function getFilters()
    {
        return [
            new TwigFilter('stars', [$this, 'stars'], ['is_safe' => ['html']]),
        ];
    }

    public function stars($note)
    {
        $html = '';
        for ($i = 0; $i < $note; $i++) {
            $html .= '<i class="fas fa-star"></i>';
        }
        for ($i = 0; $i < 5 - $note; $i++) {
            $html .= '<i class="far fa-star"></i>';
        }

        return $html;
    }
}
```

Nous ajoutons sur la déclaration du filtre un tableau avec la clé `is_safe` et la valeur `html`. Cela va permettre à twig de savoir que le filtre retourne du HTML et qu'il ne doit pas l'encoder (autrement dit l'afficher en texte).

## Exercices

### Exercice 1

Créer un filtre qui va permettre d'afficher la date du jour au format français.
L'appel du filtre dans twig pourrait être :

```twig
{{ 'now'|dateFr }}
```

### Exercice 2

Créer un filtre qui va formatter un numéro de téléphone en ajoutant des espaces entre les groupes de chiffres.
L'appel du filtre dans twig pourrait être :

```twig
{{ '0606060606'|formatPhone }}
```

