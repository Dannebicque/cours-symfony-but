---
description: Mettre en production un projet réalisé avec Symfony
---

# Mise en production

La mise en production d'un projet avec Symfony est simple, mais implique de suivre les différentes étapes.

## Mise en production avec un FTP

Pour effectuer une mise en ligne de votre projet avec un logiciel FTP, il faut télécharger sur le serveur les fichiers ou répertoires suivants.

* [ ] bin/
* [ ] config/
* [ ] public/ (normalement sans les fichiers uploadés, les fichiers générés, ...)
* [ ] src/
* [ ] templates/
* [ ] .env
* [ ] composer.json
* [ ] composer.lock
* [ ] symfony.lock

Pour une mise à jour d'un produit existant, l'upload de `src/` et `templates/` peut suffire, selon ce que vous avez modifié sur votre projet.

Il vous faut ensuite exécuter les différentes opérations de mise en ligne de la partie [Mise en production](mise-en-production.md#operation-a-realiser-apres-la-mise-en-ligne-des-fichiers)

## Mise en production avec Git

### Première mise en ligne

Pour une première mise en ligne vous pouvez faire un clone de votre dépôt.

```bash
git clone https://...
```

Cette commande va créer un nouveau répertoire et copier les fichiers de votre dépôt sur votre serveur.

Il vous faut ensuite exécuter les différentes opérations de mise en ligne de la partie [Mise en production](mise-en-production.md#operation-a-realiser-apres-la-mise-en-ligne-des-fichiers)

### Mise à jour d'un projet existant

Pour mettre à jour un projet déjà existant sur votre serveur, et géré avec Git, vous devez executer la commande suivante

```bash
git pull origin master
```

Où master est la branche qui contient la version de votre projet que vous souhaitez installer.

Il vous faut ensuite exécuter les différentes opérations de mise en ligne de la partie [Mise en production](mise-en-production.md#operation-a-realiser-apres-la-mise-en-ligne-des-fichiers)

## Opération à réaliser après la mise en ligne des fichiers

### 1) Configurer vos variables d'environnement

Symfony propose une gestion poussée des environnements, et vous pourriez avoir plusieurs fichiers en fonction du contexte :  [https://symfony.com/doc/current/configuration.html#configuration-environments](https://symfony.com/doc/current/configuration.html#configuration-environments)

Si vous n'avez qu'un fichier `.env`, vous devez le mettre à jour avec les informations de votre serveur de production (base de données, éventuellement serveur de mail, ...).

C'est également dans ce fichier d'environnement que vous devez définir que votre projet est en production et plus en développement :

```bash
APP_ENV=prod #au lieu de APP_ENV=dev
```

En production Symfony utilise un système de cache, n'affiche plus le profiler ou les messages d'erreur, mais utilise les pages 404, 500 que vous avez définies.

Les erreurs peuvent être sauvegardées dans des fichiers de log, selon votre configuration.

Toutes les configuration se trouvant dans `config/packages/dev` sont ignorées et celles se trouvant dans `config/packages/prod` sont utilisées.

### 2) Installer/Mettre à jour les Vendors[¶](https://symfony.com/doc/current/deployment.html#c-install-update-your-vendors)

Le répertoire vendor n'est jamais installé ou téléchargé, quelque soit la méthode que vous utilisez. Il faut donc soit le créer (une première installation), soit le mettre à jour (si vous avez ajouté des bundles ou que vous souhaitez obtenir la dernière version des bundles que vous utilisez)

```bash
composer install --no-dev --optimize-autoloader
```

L'information "--no-dev" permet de ne pas installer les bundles qui ne servent qu'en développement (make, profiler, ...). Si vous souhaitez être en développement sur ce serveur, vous devez retirer cette instruction.

### 3) Nettoyer le cache de Symfony[¶](https://symfony.com/doc/current/deployment.html#d-clear-your-symfony-cache)

En production Syfmony utilise un système de cache très performant. Mais cela peut empêcher de voir vos dernières modifications. Il faut donc s'assurer de nettoyer le cache de Symfony.

La commande suivante permet de supprimer le cache :

```bash
 APP_ENV=prod APP_DEBUG=0 php bin/console cache:clear
```

### 4) Mettre à jour votre base de données

Si vous avez ajouté des éléments dans vos entités, vous devez mettre à jour votre base de données. Pour cela vous pouvez executer la commande suivante :

```bash
bin/console doctrine:schema:update -f
```
