# Séance 7 : Requetes personnalisés (repository)

Dans cette partie nous allons revenir sur les notions de repository et de requêtes, afin de pouvoir écrire nos propres requêtes.

### Repository

Un repository est une classe qui permet de faire des requêtes sur une table (par l'intérmédiaire de l'eentité associée) de la base de données. Dans Symfony, lorsque vous ajoutez une entité, un repository est automatiquement créé.

Ce fichier va contenir nos requêtes spécifiques, en utilisant au choix du SQL ou du DQL (Doctrine Query Language), qui permet de construire des requêtes en notation objet sans utiliser la syntaxe SQL qui nous rendrait dépendant de notre SGBD.

Prenons par exemple une entité Adresse. Nous allons analyser le repository pour l'entité Adresse.

#### AdresseRepository.php

{% code lineNumbers="true" %}
```php
<?php

namespace App\Repository;

use App\Entity\Adresse;
use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
use Doctrine\Persistence\ManagerRegistry;

/**
 * @extends ServiceEntityRepository<Adresse>
 *
 * @method Adresse|null find($id, $lockMode = null, $lockVersion = null)
 * @method Adresse|null findOneBy(array $criteria, array $orderBy = null)
 * @method Adresse[]    findAll()
 * @method Adresse[]    findBy(array $criteria, array $orderBy = null, $limit = null, $offset = null)
 */
class AdresseRepository extends ServiceEntityRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, Adresse::class);
    }

//    /**
//     * @return Adresse[] Returns an array of Adresse objects
//     */
//    public function findByExampleField($value): array
//    {
//        return $this->createQueryBuilder('a')
//            ->andWhere('a.exampleField = :val')
//            ->setParameter('val', $value)
//            ->orderBy('a.id', 'ASC')
//            ->setMaxResults(10)
//            ->getQuery()
//            ->getResult()
//        ;
//    }

//    public function findOneBySomeField($value): ?Adresse
//    {
//        return $this->createQueryBuilder('a')
//            ->andWhere('a.exampleField = :val')
//            ->setParameter('val', $value)
//            ->getQuery()
//            ->getOneOrNullResult()
//        ;
//    }
}
```
{% endcode %}

Tout d'abord ce fichier contient une classe qui étend la classe ServiceEntityRepository, qui est une classe fournie par Doctrine. Cette classe permet de faire des requêtes sur une table de la base de données.

La classe contient une méthode constructeur qui prend en paramètre un objet de type ManagerRegistry, qui est une classe fournie par Doctrine. Cette classe permet de récupérer des informations sur la base de données, notamment le nom de la table associée à l'entité.

Enfin, notez les lignes de commentaires avant la classe. Elles indiquent les 4 méthodes nativement proposées par les repository avec Doctrine, à savoir (voir S3 pour plus de détails) :

* find : permet de récupérer une entité à partir de son identifiant
* findOneBy : permet de récupérer une entité à partir d'un tableau de critères
* findAll : permet de récupérer toutes les données d'une table et de les retourner sous forme de tableau
* findBy : permet de récupérer toutes les données d'une table et les retourner sous forme de tableau à partir d'un tableau de critères.

Vous avez ensuite deux exemples de requêtes, qui sont commentés. Ces requêtes sont écrites en DQL, et permettent de récupérer des données à partir de critères. Vous pouvez les utiliser comme modèle pour écrire vos propres requêtes.

#### Exemple 1

{% code lineNumbers="true" %}
```php
    public function findByExampleField($value): array
    {
        return $this->createQueryBuilder('a')
            ->andWhere('a.exampleField = :val')
            ->setParameter('val', $value)
            ->orderBy('a.id', 'ASC')
            ->setMaxResults(10)
            ->getQuery()
            ->getResult()
        ;
    }
```
{% endcode %}

La première requête permet de récupérer toutes les adresses dont le champ `exampleField` est égal à la valeur passée en paramètre. La seconde requête permet de récupérer une seule adresse dont le champ `exampleField` est égal à la valeur (`$value`) passée en paramètre. Pour que cet exemple fonctionne `exampleField` doit être un champ de la table `Adresse`.

Ce qui est à noter c'est que pour construire une requête on utilise la méthode `createQueryBuilder`, qui prend en paramètre le nom de l'alias de l'entité. Dans notre exemple, l'alias est `a`. Cet alias est utilisé pour construire la requête. Par exemple, si on veut récupérer les adresses dont le champ `exampleField` est égal à la valeur passée en paramètre, on écrit `a.exampleField = :val`. L'alias est donc utilisé pour indiquer le nom du champ de la table. Les alias doivent être unique si vous souhaitez faire des jointures.

Ensuite, on utilise la méthode andWhere pour ajouter une condition à la requête. Dans notre exemple, on ajoute une condition sur le champ `exampleField` qui doit être égal à `:val`. On utilise ce que l'on nomme des **requêtes parametrées**. Cela permet de sécuriser les requêtes, et d'éviter les injections SQL.

Il faut ensuite définir une valeur pour notre paramètre :val. Pour cela on utilise la méthode setParameter, qui prend en paramètre le nom du paramètre (ici `:val`) et la valeur à lui attribuer (ici `$value`). (il existe une méthode setParameters qui permet de définir plusieurs paramètres en même temps dans un tableau associatif).

On peut donc avoir plusieurs conditions et autant de paramètres que nécessaires.

Ensuite, on peut ajouter des ordres de tri sur les résultats de la requête. Pour cela on utilise la méthode `orderBy`, qui prend en paramètre le nom du champ sur lequel on souhaite trier, et le sens du tri (`ASC` ou `DESC`). Si on souhaite ajouter d'autres tris, on peut ajouter des méthodes `addOrderBy` qui prennent les mêmes paramètres. Le "add" permet de dire que l'on souhaite ajouter un autre ordre de tri.

Ensuite, on peut limiter le nombre de résultats retournés par la requête. Pour cela on utilise la méthode `setMaxResults`, qui prend en paramètre le nombre de résultats maximum. Dans notre exemple, on limite à 10 résultats.

A ce stade la requête est prête mais non fonctionnelle. Pour l'exécuter, on utilise la méthode `getQuery`, qui permet de récupérer l'objet Query qui représente la requête. Enfin, on utilise la méthode `getResult` pour récupérer les résultats de la requête. La réponse est un tableau dans ce cas.

Pour tester cette requête, vous pouvez l'appeler depuis un contrôleur, un service, une méthode... par l'intermédiaire du repository. Par exemple, si vous avez un contrôleur qui s'appelle `AdresseController`, vous pouvez écrire :

{% code lineNumbers="true" %}
```php
    public function index(AdresseRepository $adresseRepository): Response
    {
        $adresses = $adresseRepository->findByExampleField('test');
        return $this->render('adresse/index.html.twig', [
            'adresses' => $adresses,
        ]);
    }
```
{% endcode %}

#### Exemple 2

{% code lineNumbers="true" %}
```php
    public function findOneBySomeField($value): ?Adresse
    {
        return $this->createQueryBuilder('a')
            ->andWhere('a.exampleField = :val')
            ->setParameter('val', $value)
            ->getQuery()
            ->getOneOrNullResult()
        ;
    }
```
{% endcode %}

Cette requête est très similaire à la précédente, sauf qu'elle **ne doit retourner qu'un seul résultat ou rien** (`getOneOrNullResult`). Dans ce cas, la réponse est un objet de type Adresse, ou null si aucun résultat n'est trouvé. Si la requête retour plus d'un résultat, alors une erreur est levée.

### Exercice

* Proposer une requête qui permet de récupérer tous les éditeurs dont le champ `code_postal` est égal à 33000.
* Proposer une requête qui permet de récupérer tous les éditeurs et trier les réponses par nom.
* Testez ces deux méthodes dans un contrôleur pour voir les résultats.

**Ajoutez des données dans votre base de données pour effectuer vos tests.**

### Quelques instructions DQL

#### Les jointures

Pour faire des jointures, il faut utiliser la méthode `join` de la requête. Par exemple, si on souhaite récupérer les adresses et les villes associées (qui serait dans une autre entité), on peut écrire :

{% code lineNumbers="true" %}
```php
    public function findAllWithVille(): array
    {
        return $this->createQueryBuilder('a')
            ->join('a.ville', 'v')
            ->orderBy('a.id', 'ASC')
            ->getQuery()
            ->getResult()
        ;
    }
```
{% endcode %}

Dans cet exemple, on utilise la méthode `join` pour faire une jointure sur la table `ville`. On utilise l'alias `v` pour la table `ville`. On peut ensuite utiliser cet alias pour faire des requêtes sur la table `ville`. Par exemple, si on souhaite récupérer les adresses dont la ville est `Paris`, on peut écrire :

{% code lineNumbers="true" %}
```php
    public function findAllWithVilleParis(): array
    {
        return $this->createQueryBuilder('a')
            ->join('a.ville', 'v')
            ->andWhere('v.nom = :val')
            ->setParameter('val', 'Paris')
            ->orderBy('a.id', 'ASC')
            ->getQuery()
            ->getResult()
        ;
    }
```
{% endcode %}

Les autres jointures sont :

* `leftJoin` : jointure gauche
* `rightJoin` : jointure droite
* `innerJoin` : jointure interne

Exemple de jointure gauche :

{% code lineNumbers="true" %}
```php
    public function findAllWithVilleParis(): array
    {
        return $this->createQueryBuilder('a')
            ->leftJoin('a.ville', 'v')
            ->andWhere('v.nom = :val')
            ->setParameter('val', 'Paris')
            ->orderBy('a.id', 'ASC')
            ->getQuery()
            ->getResult()
        ;
    }
```
{% endcode %}

#### Les opérateurs

Il existe plusieurs opérateurs pour faire des requêtes. Par exemple, pour récupérer les adresses dont le champ `code_postal` est supérieur à 10000, on peut écrire :

{% code lineNumbers="true" %}
```php
    public function findAllWithCodePostalSup10000(): array
    {
        return $this->createQueryBuilder('a')
            ->andWhere('a.code_postal > :val')
            ->setParameter('val', 10000)
            ->orderBy('a.id', 'ASC')
            ->getQuery()
            ->getResult()
        ;
    }
```
{% endcode %}

Il existe plusieurs opérateurs :

* `=` : égal
* `!=` : différent
* `>` : supérieur
* `<` : inférieur
* `>=` : supérieur ou égal
* `<=` : inférieur ou égal
* `LIKE` : contient. Le paramètre doit être une chaîne de caractères, et on peut utiliser le caractère `%` pour remplacer un nombre quelconque de caractères.
* `NOT LIKE` : ne contient pas. Le paramètre doit être une chaîne de caractères, et on peut utiliser le caractère `%` pour remplacer un nombre quelconque de caractères.
* `IN` : dans une liste. Le paramètre doit être un tableau.
* `NOT IN` : pas dans une liste. Le paramètre doit être un tableau.
* `BETWEEN` : entre deux valeurs. Le paramètre doit être un tableau de deux valeurs.
* `IS NULL` : est null. Dans ce cas il n'y a pas de paramètre attendu.
* `IS NOT NULL` : n'est pas null. Dans ce cas il n'y a pas de paramètre attendu.
* `EXISTS` : existe
* `NOT EXISTS` : n'existe pas
* `IS EMPTY` : est vide
* `IS NOT EMPTY` : n'est pas vide

#### Les clauses

Il existe plusieurs clauses pour faire des requêtes. Par exemple, pour récupérer les adresses dont le champ `code_postal` est supérieur à 10000 et dont le champ `rue` contient `rue`, on peut écrire :

{% code lineNumbers="true" %}
```php
    public function findAllWithCodePostalSup10000AndRueContainsRue(): array
    {
        return $this->createQueryBuilder('a')
            ->andWhere('a.code_postal > :val')
            ->setParameter('val', 10000)
            ->andWhere('a.rue LIKE :val2')
            ->setParameter('val2', '%rue%')
            ->orderBy('a.id', 'ASC')
            ->getQuery()
            ->getResult()
        ;
    }
```
{% endcode %}

Il existe plusieurs clauses :

* `andWhere` : et
* `orWhere` : ou
* `andHaving` : et
* `orHaving` : ou
* `groupBy` : groupe par
* `orderBy` : tri par
* `setMaxResults` : nombre maximum de résultats
* `setFirstResult` : nombre de résultats à sauter
* `addSelect` : ajouter un champ à sélectionner
* `addOrderBy` : ajouter un champ à trier
* `addGroupBy` : ajouter un champ à grouper

L'ordre et les priorités sont les mês que pour SQL.

#### Les fonctions

Comme en SQL il existe des fonctions qui peuvent s'appliquer sur la requête pour faire des sommes, moyennes, etc. Par exemple, pour récupérer la somme des codes postaux (ce qui ne sert à rien bien sûr !), on peut écrire :

{% code lineNumbers="true" %}
```php
    public function findAllWithSumCodePostal(): array
    {
        return $this->createQueryBuilder('a')
            ->select('SUM(a.code_postal) as somme')
            ->getQuery()
            ->getResult()
        ;
    }
```
{% endcode %}

Il existe plusieurs fonctions :

* `COUNT` : nombre d'éléments
* `SUM` : somme
* `AVG` : moyenne
* `MIN` : minimum
* `MAX` : maximum
* ...

### Exercice 2

* Proposer une requête qui permet de récupérer tous les jeux dont le prix est supérieur à une valeur passée en paramètre.
* Proposer une requête qui permet de récupérer tous les jeux dont le prix est supérieur à une valeur passée en paramètre et dont le nom contient une chaîne de caractères passée en paramètre.
* Proposer une requête qui permet de récupérer tous les jeux dont le prix est supérieur à une valeur passée en paramètre et dont le nom contient une chaîne de caractères passée en paramètre, et qui sont triés par prix décroissant.
* Proposer une requête qui permet de récupérer tous les jeux dont le prix est supérieur à une valeur passée en paramètre et dont le nom contient une chaîne de caractères passée en paramètre, et qui sont triés par prix décroissant, et qui est proposé par un éditeur dont le code postal est 33000.
* Ecrire les méthodes correspondantes dans le repository `JeuRepository`.
* Utiliser les méthodes dans le contrôleur `JeuxRequetesController` pour afficher les résultats dans une vue. Chaque methode utilisera une requête (donc 4 méthodes/routes). Mais la même vue pourra être utilisée.

### Exercice 3

* Affichez un formulaire avec les champs prix, texte (nom) et code postal.
* Récupérez ces données et les passer dans une requête qui filtre les produits inférieurs au prix saisie et dont le code postal de l'éditeur est celui saisi ou pour lesquels le nom contient le "texte" du champs associé.. Triez par prix décroissant.
