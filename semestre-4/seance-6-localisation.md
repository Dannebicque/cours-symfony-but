# Séance 6 : Localisation de l'application

Le terme "internationalisation" (souvent abrégé [i18n](https://fr.wikipedia.org/wiki/Internationalisation_\(informatique\)) ) fait référence au processus d'extraction de chaînes et d'autres éléments spécifiques aux paramètres régionaux de votre application dans une couche où ils peuvent être traduits et convertis en fonction des paramètres régionaux de l'utilisateur (c'est-à-dire la langue et le pays). Pour le texte, cela signifie envelopper chacun avec une fonction capable de traduire le texte (ou "message") dans la langue de l'utilisateur :

```php
// text will *always* print out in English
echo 'Hello World';

// text can be translated into the end-user's language or
// default to English
echo $translator->trans('Hello World');
```

Le processus de traduction dans Symfony comporte plusieurs étapes :

1. Activer et configurer le service de traduction de Symfony ;
2. définir les chaînes (c'est-à-dire "messages") à localiser/traduire en les enveloppant dans des appels au traducteur ("Translations");
3. Créer des ressources/fichiers de traduction pour chaque langue prise en charge qui traduisent chaque message dans l'application ;
4. Déterminez, définissez et gérez les paramètres régionaux de l'utilisateur sur l'ensemble de la session de l'utilisateur.

## Activer et configurer le service de traduction de Symfony

Il faut installer le bundle `symfony/translation` :

```bash
composer require symfony/translation
```

Après l'installation, le service de traduction de Symfony est activé par défaut dans une application Symfony. Pour le configurer, il faut modifier le fichier `config/packages/translation.yaml` :

```yaml
# config/packages/translation.yaml
framework:
    default_locale: 'en'
    translator:
        default_path: '%kernel.project_dir%/translations'
```

Le paramètre `default_locale` définit la langue par défaut de l'application. Le paramètre `default_path` définit le chemin vers lequel Symfony doit rechercher les fichiers de traduction.

## Définir les chaînes à localiser/traduire

### Dans le code PHP

Pour définir les chaînes à localiser/traduire, il faut les envelopper dans des appels au traducteur :

```php
// src/Controller/DefaultController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Contracts\Translation\TranslatorInterface;
//...

class DefaultController extends AbstractController
{
    public function index(TranslatorInterface $translator)
    {
        $translated = $translator->trans('Symfony is great');
        //....
        return $this->render('default/index.html.twig', [
            'translated' => $translated,
        ]);
    }
}
```

Pour ce premier exemple, la traduction est gérée dans une méthode d'un de nos contrôleurs qui utilise le service `translator` injecté dans la méthode. La chaîne de caractères à traduire est passée en paramètre de la méthode `trans()` du service `translator`.

Notez que nous avons ici la chaîne parfaitement compréhensible par l'utilisateur, mais nous pourrions aussi avoir une clé de traduction, par exemple `home.welcome_message` qui serait plus facile à gérer pour les développeurs.

L'avantage de cette solution est que si la traduction n'existe pas, la chaîne de caractères passée en paramètre est retournée, et le texte reste donc lisible.

### Dans Twig

{% embed url="https://symfony.com/doc/current/reference/twig_reference.html#trans" %}

```twig
// La forme la plus simple
{{ 'symfony is great'|trans }}

// avec un domaine
{{ 'symfony is great'|trans({},'message') }}

// avec des paramètres
{{ 'Symfony %version% is great'|trans({'%version%': 7}) }}
```

## Extraire les chaînes à traduire

Il est possible de copier/coller les chaînes à traduire manuellement, mais Symfony propose une commande pour extraires automatiquement les textes à traduire depuis vos vues ou vos contrôlleurs (pour les autres, il faudra les ajouter à la main).

```
bin/console translation:extract en --format yaml --force
```

Cette commande est à exécuter pour chaque locale de votre site (ici en). Le format permet de définir le type de fichier de sortie.

Le `--force` va écraser les données existantes pour ajouter les clés manquantes.

## Créer des ressources/fichiers de traduction

Pour créer des ressources/fichiers de traduction, il faut créer un fichier de traduction pour chaque langue prise en charge. Par exemple, pour la langue française, nous pouvons créer un fichier `messages.fr.yaml` dans le dossier `translations` :

```yaml
# translations/messages.fr.yaml
Symfony is great: Symfony est génial
```

Le nom de fichier des fichiers de traduction est important : chaque fichier de message doit être nommé selon le chemin suivant domain.locale.loader:

* domain (ici message) : Le domaine de traduction ;
* locale (ici fr) : La locale à laquelle les traductions sont destinées (par exemple en\_GB, en, etc.) ;
* loader (ici yaml) : Comment Symfony doit charger et analyser le fichier (par exemple xlf, php, yaml, etc).

Par défaut, Symfony fournit de nombreux "loader"" qui sont sélectionnés en fonction des extensions de fichiers suivantes :

* .yaml: fichier YAML (vous pouvez également utiliser l'extension de fichier .yml) ;
* .xlf: fichier XLIFF (vous pouvez également utiliser l'extension de fichier .xliff) ;
* .php: un fichier PHP qui renvoie un tableau avec les traductions ;
* .csv: fichier CSV ;
* .json: fichier JSON ;
* .ini: fichier INI ;
* ... [https://symfony.com/doc/current/translation.html#translation-resource-file-names-and-locations](https://symfony.com/doc/current/translation.html#translation-resource-file-names-and-locations)

## Déterminer, définir et gérer les paramètres régionaux de l'utilisateur

La traduction se produit en fonction des paramètres régionaux de l'utilisateur. La locale de l'utilisateur courant est stockée dans l'objet `Request` :

```php
use Symfony\Component\HttpFoundation\Request;

public function index(Request $request)
{
    $locale = $request->getLocale();
}
```

Pour définir la locale de l'utilisateur, il faut utiliser la méthode `setLocale()` de l'objet `Request` :

```php
use Symfony\Component\HttpFoundation\Request;

public function index(Request $request)
{
    $request->setLocale('fr'); //on pourrait associer une route pour changer de langue et sauvegarder la langue de l'utilisateur dans la session
}
```

Symfony récupère automatiquement la locale pour choisir le bon fichier de traduction.

Il est aussi possible de gérer la locale en utilisant les URL :

```php
// src/Controller/ContactController.php
namespace App\Controller;

// ...
class ContactController extends AbstractController
{
    #[Route(
        path: '/{_locale}/contact',
        name: 'contact',
        requirements: [
            '_locale' => 'en|fr|de',
        ],
    )]
    public function contact()
    {
    }
}
```

La variable `_locale` est automatiquement injectée dans le contrôleur et peut être utilisée pour définir la locale de l'utilisateur.

## Exercices

1. Traduire le menu et la page jeu (qui liste les jeux) de mmiple
2. Créer un lien qui permet de changer de langue.

## Localisation des heures, des dates, ...

Twig permet de faciliter la localisation des dates, des heures, ...

Vous pouvez suivre les documentations selon vos besoins :

* Les dates : [https://twig.symfony.com/doc/3.x/filters/format\_date.html](https://twig.symfony.com/doc/3.x/filters/format_date.html)
* Les dates et heures : [https://twig.symfony.com/doc/3.x/filters/format\_datetime.html](https://twig.symfony.com/doc/3.x/filters/format_datetime.html)
* Les heures : [https://twig.symfony.com/doc/3.x/filters/format\_time.html](https://twig.symfony.com/doc/3.x/filters/format_time.html)
* Les monnaies : [https://twig.symfony.com/doc/3.x/filters/format\_currency.html](https://twig.symfony.com/doc/3.x/filters/format_currency.html)

Dans tous les cas deux bundles sont nécessaires

```bash
composer require twig/intl-extra
composer require twig/extra-bundle
```

Ces deux dépendances nécessites [php-intl](https://www.php.net/manual/fr/book.intl.php) d'installé sur le serveur.

### Exercices

* Installez les éléments et testés différents cas de figure&#x20;
  * avec les dates (afficher la date du jour sur la page d'accueil).
  * avec les monnaies à la place du filtre que nous avions écrit dans les séances précédentes.
