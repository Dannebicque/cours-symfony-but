# Séance 1 : Eco-Système de Symfony

# Symfony - Eco-système

Symfony nécessite tout un environnement pour fonctionner. Symfony implique aussi différents "langages" et utilise un vocabulaire spécifique (souvent repris dans d'autres frameworks).

## Composer

[_Composer_](https://getcomposer.org/) est un logiciel **gestionnaire de dépendances** libre écrit en PHP. Il permet à ses utilisateurs de déclarer et d'installer les bibliothèques dont le projet principal a besoin. Développé, depuis 2011, par Nils Adermann et Jordi Boggiano  (qui continuent encore aujourd'hui à le maintenir), il est aujourd'hui en version 2.

Le logiciel _Composer_ trouve son équivalent pour le front avec _npm_ ou _Yarn_

## ENTITÉ (EQ. DU MODÈLE)

Une _entité_ est une classe PHP ! Elle peut faire le lien avec une base de données, on y déclare les différentes propriétés accessibles; Symfony utilise par défaut un outil de persistence de données : [_Doctrine_](https://www.doctrine-project.org/index.html) pour lier une entité à une table de base de données.

## ORM : OBJECT RELATIONNAL MAPPING

Système permettant de se libérer des requêtes pour la base de données. Il se charge de générer les requêtes à effectuer sur les Entités spécifiées. Il existe plusieurs ORM, dans Symfony, il s'agit de [_Doctrine_](https://www.doctrine-project.org/index.html).

## Repository

Classe PHP qui fait le pont entre une entité et l'ORM, il permet notamment de structurer des requêtes complexes.

## YAML

Format de structuration de données très utilisé dans Symfony, mais on peut utiliser du XML, du PHP ou encore les annotations (PHP 7) ou les attributs (php 8), les fichiers de configurations par défaut sont en YAML.

## Annotations

Commentaire PHP directement dans les classes utiles (controller, entité, ...) interprété par Symfony pour générer des fichiers de configuration ;

## Attributes

Les _Attributes_ sont la nouvelle version des annotations, intégrés nativement dans les versions 8 et supérieures de PHP. Les _Attributes_ sont une forme de données structurées, qui permet d'ajouter des métadonnées à nos déclarations de classes, propriétés, méthodes, fonctions, paramètres et constantes. Ils vont nous permettre de définir de la configuration au plus près du code, directement sur nos déclarations.

## Routes

Les routes permettent de faire un lien entre une URL et un contrôleur. 

## Bundles

Sorte de modules Symfony qui peuvent contenir tout et n'importe quoi ; C'est la force de Symfony les modules peuvent fonctionner indépendamment et même sur d'autres structures PHP, autre framework etc.

## ENVIRONNEMENTS

Symfony propose par défaut 2 environnements : _dev_ et _prod_ qui permettent de donner des configs différentes en fonction de l'environnement de travail ; 

* dev permet une utilisation sans cache avec des outils de dev comme le profiler ; 
* prod lui permet d'utiliser le site avec le cache et sans aucun message d'erreurs. 

De plus on peut configurer les différents environnements pour par exemple rediriger tous les mails vers toto@titi.com en dev et laisser le fonctionnement normal pour prod ; pratique pour les debugs.

## ENVIRONNEMENTS

Symfony propose également de définir autant d'environnement que nécessaire afin d'avoir différentes configurations. Le changement d'un environnement à un autre se faire en modifiant la ligne suivante dans le fichier ".env" (ou .env.local) :
   ```dotenv
   ###> symfony/framework-bundle ###
   APP_ENV=dev
   ```

## Profiler

Le profiler est un outil puissant (et indispensable) pour débuger une application. Par défaut le profiler n'est pas installé. Pour l'ajouter il faut exécuter la commande suivante :
```bash
composer require profiler --dev
```

Le profiler est toujours visible en bas de la page en mode développement.

## Maker

Le maker est un outil puissant pour générer du code, et des éléments dans notre projet. Par défaut le maker n'est pas installé. Pour l'ajouter il faut exécuter la commande suivante :

```bash
composer require maker --dev
```

Il est assez facile de modifier le maker pour que le code généré corresponde exactement à notre projet et nos attentes.

## Moteur de template

Un moteur de template est un "langage", un "outil" permettant d'écrire les vues (partie visible) de manière efficace et rapide. Symfony utilise [_Twig_](https://twig.symfony.com/doc/3.x/) par défaut.
