# Séance 4 : Modèles - Entités - ORM

## Introduction

Dans Symfony la notion de modèle se retrouve sous la forme (entre autre) d'une **Entité**. Une entité est une **classe PHP**, qui **peut** être connectée à une table de votre base de données via l'ORM. Lorsqu'une entité est liée à une table, via l'ORM, il y a en général un fichier "repository" associé. Un repository permet la génération de requêtes simples ou complexes et dont le développeur peut modifier à volonté.

Un ORM (**Object Relation Mapper**) permet de gérer manipuler et de récupérer des tables de données de la même façon qu'un objet quelconque, donc en gardant le langage PHP. Plus besoin de requête MySQL, PostgresSQL ou autre.

Symfony utilise **Doctrine** comme ORM dans son système par défaut. Nous allons utiliser Doctrine mais vous pouvez utiliser d'autres systèmes si vous le souhaitez. Doctrine peut-être géré de plusieurs façon : XML, JSON, YAML, PHP et en Annotation nous allons utiliser ce dernier format de données.

[La documentation officielle de Symfony sur l'ORM Doctrine](https://symfony.com/doc/current/doctrine.html) [La document officielle de Doctrine](https://www.doctrine-project.org/projects/orm.html)

Vous êtes libre d'écrire le code qui permet le traitement métiers en dehors des entités et d'avoir votre propre logique d'organisation.

## Mise en application

### Configuration

Comme à chaque fois, il est d'abord nécessaire d'installer les bundles nécessaires pour manipuler la base de données avec un ORM. Il vous faut donc exécuter la commande ci\_dessous :

```bash
composer require symfony/orm-pack
```

On va également installer, si vous ne l'avez pas encore fait, le bundle "maker" qui contient des outils pour générer du code sous Symfony grâce à la console.

```bash
composer require symfony/maker-bundle --dev
```

Une fois ces deux éléments installés, il faut configurer la connexion à la base de données. Pour ce faire, il faut éditer le fichier `.env` à la racine de votre projet, qui doit normalement contenir une ligne d'exemple.

```
# .env

# customize this line!
DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/db_name"

# to use sqlite:
# DATABASE_URL="sqlite:///%kernel.project_dir%/var/app.db"
```

### Création de la base de données

Une fois le fichier à jour avec vos données, vous pouvez créer votre base de données depuis la console.

```bash
php bin/console doctrine:database:create
```

Les modifications de structure de votre base de données devront être réalisées avec la console pour que Symfony puisse faire le lien entre les tables et l'ORM.

### Création d'une entité liée à une table

Utilisez la commande `make:entity` (qui est dans le bundle maker) pour avoir une série de question vous permettant de créer votre entité avec l'utilisation de l'ORM Doctrine. Vous pouvez créer une nouvelle entité ou modifier (ajouter des champs) une entité déjà existante en saisissant son nom.

```bash
php bin/console make:entity
```

Vous allez devoir répondre à une suite de question avec le nom de l'entité (par défaut cela donnera le nom de la table), et les champs à créer. Dans Symfony une entité possède toujours un champs id, qui est la clé primaire et qui est auto-incrémenté. Vous ne devez donc pas l'ajouter dans la console.

Pour la création d'un champs, il vous faudra donner :

* son type
* sa taille le cas échéant
* si ce champs peut être null
* s'il doit être unique (index)

Vous pouvez obtenir la liste des types supportés en tapant "?" à la question du type.

Une fois terminé, le fichier d'Entité et le _repository_ associé sont générés.

Exemple dans la console :

```bash
php bin/console make:entity

Class name of the entity to create or update:
> Product

 to stop adding fields):
> name

Field type (enter ? to see all types) [string]:
> string

Field length [255]:
> 255

Can this field be null in the database (nullable) (yes/no) [no]:
> no

 to stop adding fields):
> price

Field type (enter ? to see all types) [string]:
> integer

Can this field be null in the database (nullable) (yes/no) [no]:
> no

 to stop adding fields):
>
(press enter again to finish)
```

Et le code de l'entité généré dans `src/Entity/Product.php` :

```php
// src/Entity/Product.php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Doctrine\DBAL\Types\Types;

#[ORM\Entity(repositoryClass:"App\Repository\ProductRepository")]
class Product
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private $id;

    #[ORM\Column(type:Types::STRING, length:255)]
    private $name;

    #[ORM\Column(type:Types::INTEGER)]
    private $price;

    public function getId()
    {
        return $this->id;
    }

    // ...Les getters et les setters sont automatiquement générés également. Par défaut il n'y a pas de constructeur. Vous pouvez en ajouter un si besoin.
}
```

A ce stade l'entité est créé, mais n'existe pas dans la base de données. Il reste deux étapes à exécuter.

#### Mettre à jour votre base de données : méthode 1

Sans générer de fichier de migration (qui contient toutes les instructions SQL à exécuter sur la base de données, notamment pour le déploiement d'une mise à jour)

```php
php bin/console doctrine:schema:update -f

//ou
php bin/console d:s:u -f
```

#### Mettre à jour votre base de données : méthode 2

La création d'un fichier de migration qui va contenir le code SQL a exécuter en fonction de votre SGBD.

```bash
 php bin/console make:migration
```

La mise à jour de votre base de données en fonction du fichier précédemment généré.

```bash
php bin/console doctrine:migrations:migrate
```

Si vous consultez votre PHPMyAdmin vous verrez la table apparaître.

## Modifications de champs et lien base de données

Pour modifier des champs vous pouvez éditer directement le code généré dans la partie annotation: nom (par défaut le nom de la variable), taille, type.

Pour ajouter des champs il vous faut relancer la commande `make:entity` en remettant le nom de votre entité.

Après chaque modification ou ajout il faut de nouveau générer le fichier de migration et mettre à jour la base de données. Vous pouvez bien sûr modifier ou créer plusieurs entités avant de faire une mise à jour de votre base de données.

## ORM

Une fois la base de données mise en place on va pouvoir insérer, modifier, supprimer et récupérer des informations de la base de données sans saisir de requêtes via des méthodes en initialisant l'entité fraichement créée :

<pre class="language-php"><code class="lang-php">use Doctrine\ORM\EntityManagerInterface;

...
<strong>
</strong><strong>#[Route("/test", name:"test")]
</strong>public function test(EntityManagerInterface $entityManager)
{
    $post = new Post(); // initialise l'entité
    $post->setTitle('Mon titre'); // on set les différents champs
    $post->setEnable(true);
    $post->setDateCreated(new \Datetime);

    $entityManager->persist( $post ); // on déclare une modification de type persist et la génération des différents liens entre entité
    $entityManager->flush(); // on effectue les différentes modifications sur la base de données 
    // réelle

    return new Response('Sauvegarde OK sur : ' . $post->getId() );
}
</code></pre>

Il existe à la place de `$em->persist, $em->remove($post);` qui permettra de faire une suppression.

#### Exercice

* Configurer votre base de données (dupliquer le `.env` en `.env.local` et modifier les informations dans le `.env.local`)

{% hint style="info" %}
Si vous utilisez limage docker du cours, la ligne devrait être

```
DATABASE_URL="mysql://root:123456@mariadb:3306/nomDeLaBDD?charset=utf8mb4"
```
{% endhint %}

* Créer la base de données (`bin/console doctrine:database:create`)
* Créer une entité (`bin/console make:entity`), nommée Post et ajoute les champs suivants
  * titre, string de 150 caractères
  * message, text
  * datePublication, DateTime
* L'id sera automatiquement ajouté
* Créer un nouveau contrôleur nommé "Post"
* Ajouter une route pour créer un nouvel enregistrement
  * Créer un objet Post et compléter les informations (titre, message, datePublication)
  * Récupérer la connexion à doctrine (`$em = $this->getDoctrine()->getManager()`)
  * Associer l'instance de Post avec l'ORM (`$em->persist($post))`
  * Enregistrer dans la base de données (`$em->flush()`)
* Appeler la route et vérifier que cela s'enregistre dans votre base de données
* Essayer d'appeler la route plusieurs fois.

## Recherche d'entité

Symfony et Doctrine proposes des requêtes prédéfinies, qui répondent aux usages les plus courant.

Si `$em` est le manager associé à une entité :

* `$em->find($id);` on récupère qu'un seul élément de l'entité avec l'id `$id`;
* `$em->findAll();` on récupère toutes les entrées de l'entité concernée
* `$em->findBy($where, $order, $limit, $offset);` on recherche avec le tableau `$where` on tri avec le tableau `$order` on récupère `$limit` éléments à partir de l'élément `$offset`.
* `$em->findOneBy($where, $order);` on récupère le premier élément respectant le tableau `$where` et trié avec le tableau `$order`;
* `$em->findByX($search);` requêtes magiques où X correspond à n'importe quel champs défini dans votre entité
*   `$em->findOneByX($search)` ; requêtes magiques où X correspond à n'importe quel champs défini dans votre entité

    Par exemple `findBySlug('home')`; ou `findByTitle('Bonjour);` génèrera des requêtes de recherche automatiquement. Pour les requêtes avec plusieurs éléments il faudra faire une itération (foreach) ou lister les différents éléments.

Exemple

```php
// Modifications multiples : 
#[Route("/est", name="test")]
public function test(
        EntityManagerInterface $entityManager,
        PostRepository $postRepository)
{
    // récupération de tous les posts
    $posts = $postRepository->findAll(); 
    //équivalent à SELECT * FROM post

    foreach($posts as $post)
    {
        $post->setTitle('Mon titre ' . $post->getId() ); // on set les différents champs
    }

    $entityManager->flush(); // on effectue les différentes modifications sur la base de données 
    // réelle

    return new Response('Sauvegarde OK ');
}
```

Si aucune requête prédéfinie ne correspond à vos besoin, vous pouvez bien sûr en créer une en passant par le _repository_.

Vous pouvez également générer vos requêtes manuellement pour avoir une requête complexe et précise directement dans le _controller_ mais idéalement il faudrait le placer dans le _repository_ dédié.

```php
// src/AppBundle/Repository/Post.php

public function maRequete( $where )
{
    // avec querybuilder
    $queryBuilder = $this->createQueryBuilder("p")
        ->where(' p.title like :w')
        ->setParameter(':w', '%'.$where.'%')
        ->getQuery(); // on récupère la requêtes 

   return $query->getResult(); // on renvoie le résultat
}
//OU
 public function maRequeteSQL( $where )
    {
        // avec requête SQL
        $em = $this->getEntityManager();
        $query = $em->createQuery('SELECT p from AppBundle:Post p 
    WHERE p.title like :w');

        $query->setParameter(':w', '%'.$where.'%');


        return $query->getResult(); // on renvoie le résultat
     }
}
```

Et l'utiliser dans votre _controller_

```php
// src/AppBundle/Controller/DefautController
$postRepository->maRequete('test');
```

## Modification

Ce dernier code effectue une création dans la base de données; pour une modification il suffit de modifier l'instanciation de l'entité de la sorte :

<pre class="language-php"><code class="lang-php"><strong>use App\Repository\PostRepositoy;
</strong>use Doctrine\ORM\EntityManagerInterface;

<strong>...
</strong><strong>
</strong><strong>#[Route("/test/modification", name:"test")]
</strong>public function testModification(
        EntityManagerInterface $entityManager,
        PostRepository $postRepository)
{
    // récupération du post avec id 1 
    $post = $postRepository->find(1); 
    //equivalent à SELECT * FROM post WHERE id=1
    
    $post->setTitle('Mon titre'); // on set les différents champs
    $post->setEnable(true);
    $post->setDateCreated(new \Datetime);

    $entityManager->flush(); // on effectue les différentes modifications sur la base de données 
    // réelle

    return new Response('Sauvegarde OK sur : ' . $post->getId() );
}
</code></pre>

ici on récupère le _repository_ de Post et on récupère l'id 1 ; tout le restant du code reste inchangé.

## Exercice

* Créer une entité "Categorie" avec :
  * titre string 255
  * ordre int
* Créer une entité "Article" avec :
  * titre string 255
  * texte text
  * datePublication datetime
  * auteur string 255
  * image string 255
* Une fois les deux tables créées ajoutez des données depuis phpMyAdmin dans la table Article ( 3 articles)
* Modifiez votre contrôleur et la page articles pour afficher tous les articles de votre table, par ordre décroissant (plus récent au plus ancien)
* Modifiez votre page d'accueil pour afficher le dernier article publié.
