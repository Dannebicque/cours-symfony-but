# Séance 3 : Asset Mapper

{% hint style="info" %}
Depuis la version 7 de Symfony, il est conseillé d'utiliser une version sans webpack : [https://symfony.com/doc/current/frontend/asset_mapper.html](https://symfony.com/doc/current/frontend/asset_mapper.html)
{% endhint %}

## Introduction

Extrait de la documentation officielle de Symfony :

Le composant AssetMapper vous permet d'écrire du JavaScript et du CSS modernes sans la complexité d'utiliser un bundler. Les navigateurs prennent déjà en charge de nombreuses fonctionnalités modernes de JavaScript comme l'instruction import et les classes ES6. Et le protocole HTTP/2 signifie que combiner vos assets pour réduire les connexions HTTP n'est plus urgent. Ce composant est une couche légère qui aide à servir vos fichiers directement au navigateur.

Le composant a deux fonctionnalités principales :

**Mappage et versionnage des assets** : Tous les fichiers à l'intérieur du répertoire `assets/` sont rendus disponibles publiquement et versionnés. Vous pouvez référencer le fichier `assets/images/product.jpg` dans un template Twig avec `{{ asset('images/product.jpg') }}`. L'URL finale inclura un hash de version, comme `/assets/images/product-3c16d92m.jpg`.

**Importmaps** : Une fonctionnalité native du navigateur qui facilite l'utilisation de l'instruction `import` de JavaScript (par exemple, `import { Modal } from 'bootstrap'`) sans système de build. Elle est prise en charge par tous les navigateurs (grâce à un shim) et fait partie de la norme HTML.

## Installation

Pour installer le composant AssetMapper, vous devez exécuter la commande suivante :

```bash
composer require symfony/asset-mapper symfony/asset 
```

**Ces éléments sont déjà installés si vous avez installé le bundle webapp.**

Plusieurs fichiers ont été ajoutés à votre projet :

* `assets/app.js` Your main JavaScript file;
* `assets/styles/app.css` Your main CSS file;
* `config/packages/asset_mapper.yaml` Where you define your asset "paths";
* `importmap.php` Your importmap config file.

Et dans base.html.twig, vous avez maintenant :

```twig
{% block javascripts %}
    {% block importmap %}{{ importmap('app') }}{% endblock %}
{% endblock %}
```

## Réorganiser vos images

Il est recommandé de déplacer vos images (celles de votre site : logo, bannière, décoration, ...) dans le répertoire `assets/**` pour les rendre accessibles publiquement. Vous pouvez ensuite les référencer dans vos templates Twig avec `{{ asset('images/product.jpg') }}` si vous avez un répertoire `images` dans `assets`.

Les images uploadées par les utilisateurs (avatars, photos de profil, ...) ne doivent pas être déplacées dans `assets/`. Elles doivent être stockées dans un répertoire `public` (comme `public/uploads/`) et référencées directement dans vos templates Twig.

Si vous regardez la source de la page, vous verrez que les images sont maintenant servies avec un hash de version dans l'URL. Cela signifie que les navigateurs peuvent mettre en cache les images indéfiniment, mais que si vous modifiez une image, le navigateur téléchargera la nouvelle version.

En production il faudra "compiler" les assets avec la commande :

```bash
bin/console asset-map:compile
```

Il est possible de lister tous les assets à disposition avec la commande :

```bash
bin/console debug:asset-map
```

## Utiliser du JavaScript moderne

Le composant AssetMapper utilise Importmaps pour permettre l'utilisation de l'instruction `import` de JavaScript sans système de build. Cela signifie que vous pouvez écrire du JavaScript moderne sans avoir à vous soucier de la compilation.

Par exemple, en suivant la documentation de Symfony, nous allons créer un fichier javascript `assets/duck.js` :

```javascript
export default class {
    constructor(name) {
        this.name = name;
    }
    quack() {
        console.log(`${this.name} says: Quack!`);
    }
}
```

Ce code JavaScript crée une classe `Duck` avec une méthode `quack()` qui affiche un message dans la console.

Pour utiliser ce code dans un fichier `assets/app.js`, vous pouvez importer le fichier `duck.js` :

```javascript
import Duck from './duck.js';

const duck = new Duck('Waddles');
duck.quack();
```

Comme nous avons déjà ajouté le fichier `app.js` dans le fichier `base.html.twig` avec l'instruction `{{ importmap('app') }}`, vous pouvez maintenant ouvrir la console de votre navigateur pour voir le message `Waddles says: Quack!`. **Et voilà !**

## Importer des dépendances externes

Pour ajouter un package externe, vous pouvez utiliser la commande suivante :

```bash
bin/console importmap:require bootstrap
```

Pour ajouter une dépendance à Bootrstrap. Vous pouvez ajouter tous les packages disponibles sur Npm (https://www.npmjs.com/) avec cette commande.

Cela ajoutera une entrée dans le fichier `importmap.php` :

```php
return [
    'app' => [
        'path' => './assets/app.js',
        'entrypoint' => true,
    ],
    'bootstrap' => [
        'version' => '5.3.0',
    ],
];
```

Cette commande peut installer plusieurs lignes si la librairie a des dépendances. Automatiquement les fichiers sont télécargés et ajoutés dans le répertoire `assets/vendor/`. **Ce dossier ne doit pas être modifié ni ajouté dans Git**

Vous pouvez maintenant utiliser Bootstrap dans votre fichier `app.js` :

```javascript
import 'bootstrap';
// ou
import { Alert } from 'bootstrap'; //pour n'importer que les Alertes par exemple
```

## Importer des fichiers CSS

### Pour des fichiers CSS internes

Il faut ajouter dans app.js votre fichier CSS :

```javascript
import '../styles/app.css';
```

Si styles est un dossier dans assets.

### Pour des fichiers CSS externes

Pour importer des fichiers CSS, vous pouvez utiliser la commande suivante :

```bash
bin/console importmap:require bootstrap/dist/css/bootstrap.min.css
```

Puis l'importer dans votre fichier `app.js` :

```javascript
import 'bootstrap/dist/css/bootstrap.min.css';
```

## Exercices

## Exercice 1

Tester l'exemple de duck.js et app.js. Modifier le message de la console.

## Exercice 2

Ajouter Bootstrap à votre projet. Créer un fichier `assets/styles/app.css` et ajouter une couleur de fond à la page. Ajouter une couleur de texte.

## Exercice 3

Ajoutez une dépendance à une librairie d'icône (par exemple Font Awesome) et ajoutez un icône à votre page dans le menu.

Modifiez le filtre Twig créé sur la séance d'avant pour ajouter des étoiles à la place des * ou -.

## Précédemment dans Symfony, Webpack Encore et CSS

Dans les versions 4 à 6 la recommandation était d'utiliser Webpack Encore pour gérer les assets de votre projet Symfony.

Ci-dessous le cours sur Webpack Encore, si besoin.

Webpack ([https://webpack.js.org/](https://webpack.js.org/)) est un outil logiciel open-source de type « module bundler » (littéralement, « groupeur de modules »), conçu pour faciliter le développement et la gestion de sites et d'applications web modernes. (source Wikipedia).

Webpack va permettre de gérer vos fichiers CSS/SCSS, JS, images, etc. Il va permettre de les regrouper, de les minifier, de les compiler, etc. Il permet aussi d'utiliser des langages comme VueJS, React, etc.

Utiliser Webpack dans un projet Symfony va vous permettre de gérer vos fichiers CSS/SCSS, JS, images, etc. comme dans un projet classique, puis de les regrouper, de les minifier, de les compiler, etc. et de les mettre dans le répertoire public pour qu'ils soient accessibles de vos vues.

## Installation

Pour installer et utiliser Webpack, vous devez installer NodeJS ([https://nodejs.org/en/](https://nodejs.org/en/)), pour disposer de npm ([https://www.npmjs.com/](https://www.npmjs.com/)) ou de yarn ([https://yarnpkg.com/](https://yarnpkg.com/)).

Dans le contexte de Symfony, nous n'allons pas utiliser directement Webpack, mais une version adaptée de Webpack appelée Encore ([https://symfony.com/doc/current/frontend.html](https://symfony.com/doc/current/frontend.html)). Ce package permet de simplifier l'utilisation de Webpack dans un projet Symfony, avec une gestion dans le projet et dans twig.

Pour installer Encore, il faut utiliser la commande suivante :

```bash
composer require webpack-encore
```

puis lancer la commande suivante :

```bash
yarn install ou npm install
```

Pour installer les dépendances "front".

Ces installations ont eu plusieurs effets :

* Le fichier `package.json` a été créé. Ce fichier contient les dépendances "front" de votre projet. Il est important de ne pas le supprimer.
* Le fichier `webpack.config.js` a été créé. Ce fichier contient la configuration de Webpack, et la liste de toutes les tâches qu'il doit réaliser. Il est important de ne pas le supprimer.
* Le répertoire assets a été créé. Ce répertoire contient tous les fichiers "front" de votre projet (SCSS, CSS, JS, ...), dans leur version non compilée.

Regardons en détail ces fichiers.

### package.json

### webpack.config.js

### Le répertoire assets/

### base.html.twig

## Utilisation

Nous avons quelques données d'exemples dans les fichiers du repertoire assets. Mais si vous essayez de les utiliser, vous allez avoir une erreur. Cela est lié au fait que Webpack n'a pas encore été lancé, et n'a donc pas généré les fichiers compilés dans le répertoire public/build.

Donc, pour utiliser les fichiers "front" de votre projet, vous devez lancer la commande suivante :

```bash
yarn dev ou npm run dev
```

Cette commande va lancer Webpack, et va générer les fichiers compilés dans le répertoire public/build. Un fichier entrypoints.json a été créé dans le répertoire public/build. Ce fichier contient la liste des fichiers compilés, et les chemins vers ces fichiers. C'est grâce à ce fichier que Twig va pouvoir charger les fichiers compilés.

Retournez dans votre navigateur pour réessayer d'afficher votre page, elle devrait fonctionner.

**Essayez de modifier le fichier css dans le repertoire assets pour voir les changements. N'oublier pas de relancer la commande `yarn encore dev` pour voir les changements.**

Comme il peut être fastidieux de relancer la commande à chaque fois que vous modifiez un fichier, vous pouvez utiliser la commande suivante :

```bash
yarn watch (qui est une version courte de yarn dev --watch) ou npm run watch
```

Cette commande va lancer Webpack, et va générer les fichiers compilés dans le répertoire public/build. En plus, elle va surveiller les fichiers du répertoire assets, et va relancer Webpack à chaque fois qu'un fichier est modifié, tant que vous ne fermez pas la fenêtre de votre terminal.

### Les fichiers SCSS

Si vous souhaitez utiliser des fichiers SCSS, vous devez créer ces fichiers, et les importer dans le fichier `app.scss` (le fichier app.css doit être renommé en app.scss) du répertoire assets. Par exemple, si vous souhaitez créer un fichier `style.scss` dans le répertoire assets/scss, vous devez ajouter la ligne suivante dans le fichier `app.scss` :

```scss
@import 'scss/style.scss';
```

Vous devez ensuite expliquer à webpack comment compiler ce fichier SCSS. Pour cela, vous devez ajouter (en fait décommenter) la ligne suivante dans le fichier `webpack.config.js` :

```js
.enableSassLoader()
```

Comme vous avez modifié le fichier `webpack.config.js`, vous devez relancer la commande `yarn watch` ou `npm run watch` pour que les changements soient pris en compte.

Vous allez obtenir une erreur, car il manque quelques dépendances. Suivez les instructions du terminal et installez les dépendances manquantes.

Relancez la commande `yarn watch` ou `npm run watch` pour voir les changements.

Vous pouvez maintenant mettre en pratique ce que vous savez faire en SCSS/SASS dans un projet Symfony, sans avoir besoin de compiler vos fichiers SCSS/SASS à la main.

## Exercices

### Exercice 1

Créer un fichier `style.scss` dans le répertoire assets/scss, et ajouter une couleur de fond à la page. Ajoutez aussi une couleur de texte.

Créer et modifier des contrôleurs et des vues pour tester votre style.

### Exercice 2

En vous appuyant sur la documentation de Boostrap et de Symfony, ajouter Bootstrap à votre projet (ni en CDN, ni en téléchargement, mais en l'installant avec npm ou yarn).
