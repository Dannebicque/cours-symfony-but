# Séance 4 : Webpack Encore et CSS

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
