# Les Bundles

Symfony dispose d'une large communauté très active qui contribue à développer des "bundles" afin de venir enrichir le framework avec des fonctionnalités récurrentes : Back office, gestion d'upload, ...

Il existe un site regroupant les bundles [https://flex.symfony.com/](https://flex.symfony.com/) (ou pour les versions précédentes de Symfony [http://knpbundles.com/](http://knpbundles.com/)).

Parmis quelques bundles intéressants on peut citer :

* EasyAdminBundle (bundle reconnu par Symfony pour la génération automatique de backoffice)
* SonataAdminBundle
* StofDoctrineExtensionsBundle, qui permet d'enrichir Doctrine (gestion de date created/updated, de traduction, d'upload, ...)
* KnpSnappyBundle, pour la génération de PDF à partir de WKHtmlToPdf

et beaucoup d'autres pour la sérialization, l'authentification, la génération de pagination, ...

## Installation

Avec Symfony 4 l'installation d'un Bundle est devenue très simple (notamment si le bundle dispose d'une recette _recipe_ pour l'utilsisation optimale de flex).

Liste des recipes officielles pour Flex et Symfony [https://github.com/symfony/recipes](https://github.com/symfony/recipes), et le dépôt des recipes "non-officielles" [https://github.com/symfony/recipes-contrib](https://github.com/symfony/recipes-contrib)

Il suffit, en général, d'installer le bundle avec Composer pour que ce dernier installe et configure les éléments.

## Cas de VichUploadBundle

Ce bundle permet la gestion de l'upload en lien avec Doctrine.

[https://github.com/dustin10/VichUploaderBundle](https://github.com/dustin10/VichUploaderBundle)
