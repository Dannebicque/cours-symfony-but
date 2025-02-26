# Séance 8 : Formulaires

## Formulaire pour Jeu

Nous allons mettre en place le formulaire pour les jeux. Dans cette entité nous avons une liaison vers éditeur. Cette relation est de type 1..n (un jeu à un éditeur, un éditeur peut être dans plusieurs jeux).

* Générez le formulaire pour jeu et configurez les champs "simple".

Pour la liaison entre les deux table, il faut un champ de type entity. Le code pourrait être celui ci-dessous :

{% code lineNumbers="true" %}
```php
->add('editeur', EntityType::class, [
    'class' => Editeur::class,
    'choice_label' => 'nom',
])
```
{% endcode %}

Vous pouvez définir d'autres options.

N'oubliez pas d'ajouter les use qui vont bien ! (**ou utilisez PhpStorm avec le plugin Symfony**).

* Faite le contrôleur pour afficher et sauvegarder un jeu.

## Passer des paramètres aux formulaires

Il est parfois nécessaire de passer des paramètres au formulaire (donc au fichier xxxType.php), pour par exemple afficher un champ selon des droits, activer ou désactiver une zone, pré-remplir une information ou encore filtrer les résultats d'une liste.

Comme vous pouvez le constater dans la ligne permettant de "créer" le formulaire, il n'est pas possible de passer des paramètres comme vous pourriez le faire avec une méthode ou un constructeur en POO.

```php
$form = $this->createForm(AdresseType::class, $adresse);
```

Cependant, la méthode `createForm`, autorise un troisième paramètre, qui est un tableau et qui va permettre de passer tout ce que l'on souhaite à note fichier de formulaire. Ce troisième paramètre, les options, est contrôler par Symfony (et ce que l'on nomme un `OptionResolver(`[`https://symfony.com/doc/current/components/options_resolver.html`](https://symfony.com/doc/current/components/options_resolver.html)`)`), qui va s'assurer que les paramètres obligatoires sont bien présent, et qu'ils ont les bonnes valeurs. Dans le cas du `createForm`, aucun paramètre n'est obligatoire.

Un ensemble de paramètre sont pré-définis (comme action pour modifier le lien de l'action à la validation, attr, ...). En fait ce troisième paramètre fonctionne comme celui des `add(...)` pour ajouter des champs (et au passage comme beaucoup de fonctionnalités dans Symfony).

Si on souhaite passer un paramètre qui n'est pas initialement prévu dans `l'OptionResolver`, il faut lui expliquer s'il est obligatoire, le type attendu, les valeurs possibles. Au dela de cette contraintes, vous pouvez passer autant de paramètres que nécessaires.

Configurons notre formulaire ArticleType pour recevoir un paramètre correspondant à un code postal que nous pourrons ensuite utiliser pour filtrer la liste des fournisseurs.

Tout d'abord, on modifie l'appel dans le contrôleur :

{% code lineNumbers="true" %}
```php
$form = $this->createForm(JeuType::class, $jeu,
[
    'codePostal' => '10000'
]);
```
{% endcode %}

Le troisième paramètre est un tableau, avec une clé (du texte), et une valeur. Ici la valeur est du texte, mais c'est bien souvent une variable dépendant de notre contexte.

Si vous essayez d'afficher votre formulaire, vous aurez une erreur vous indiquant que 'codePostal' n'est pas autorisé parmi les options définies dans l'OptionResolver.

<figure><img src="../.gitbook/assets/Capture d’écran 2023-02-10 à 08.24.26.png" alt=""><figcaption><p>Erreur si une option n'existe pas</p></figcaption></figure>

Notez au passage que vous avez ici la liste de toutes les options connues pour le formulaire.

Nous devons donc ajouter cette possibilité dans notre formulaire, pour cela, nous modifions le fichier JeuType.php, et plus particulièrement la méthode configureOptions, qui permet de gérer les paramètres autorisés.

Par défaut il contient les éléments ci-dessous

{% code lineNumbers="true" %}
```php
    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Jeu::class,
        ]);
    }
```
{% endcode %}

C'est le code minimal, obligatoire pour fonctionner et qui permet de lier une classe à notre formulaire (ligne 4).

Nous allons ajouter entre la ligne 4 et 5, notre nouvelle option, et lui donner une valeur par défaut, le code devient :

{% code lineNumbers="true" %}
```php
   public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Jeu::class,
            'codePostal' => null
        ]);
    }
```
{% endcode %}

Par défaut, on considère qu'il n'y a pas de valeur (null), on pourrait aussi fixer autre chose, cela dépend de votre projet. On pourrait également préciser le type, une plage de valeur possible, ...

Si vous actualisez votre page, vous n'avez plus d'erreurs.

Mais à ce stade, votre nouveau paramètre n'est pas utilisé dans votre code, la valeur que vous avez passé même pas récupérée.

Dans la méthode `buildForm`, qui construit le formulaire, nous allons donc récupérer ce paramètre. Pour cela, nous avons le second paramètre $options, qui contient tout ce qui a été passé au formulaire, avec notamment, la classe (et les éventuelles données pré-remplies), et nos options.

Un "`dump`" de cette variable vous donnerai un affichage comme ci-dessous :

<figure><img src="../.gitbook/assets/Capture d’écran 2023-02-10 à 08.31.26.png" alt=""><figcaption><p>dump($options)</p></figcaption></figure>

Vous pouvez constatez sur l'avant dernière ligne que nous avons notre codePostal. Pour le récupérer nous pourrons écrire la ligne 4 ci-dessous :

{% code lineNumbers="true" %}
```php
public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        //dump($options);
        $codePostal = $options['codePostal'];
        $builder
            ->add('...')
            ...
        ;
    }
```
{% endcode %}

Maintenant nous pouvons utiliser notre variable $codePostal comme nous le souhaitons dans notre formulaire. Par exemple, le code ci-dessous permettrait de filtrer nos adresses de fournisseurs ayant uniquement code postal donné.

{% code lineNumbers="true" %}
```php
->add('editeur', EntityType::class, [
                'class' => Editeur::class,
                'choice_label' => 'libelle',
                'query_builder' => function (EditeurRepository $fr) use ($codePostal) {
                    return $fr->createQueryBuilder('f')
                        ->join('f.adresse', 'a')
                        ->where('a.codePostal = :codePostal')
                        ->setParameter('codePostal', $codePostal)
                    ;
                },
            ])
```
{% endcode %}

Quelques explications :

* Les lignes 2 et 3 permettent de faire le lien avec l'entité Editeur (voir S3)
* La ligne 4 (jusque 10) permet d'exécuter une requête pour filtrer les résultats sur l'entité.
  * Ligne 4 nous passons le repository qui permet de manipuler nos données, et on indique avec le use que l'on souhaite utiliser notre variable codePostal.
  * Les lignes 5 à 8 sont la requête qui permet de filtrer. Vous reconnaissez vaguement du SQL, mais en notamment objet, on appelle cela le DQL (Doctrine Query Language). On y reviendra en détail sur la partie repository.

N'oubliez pas d'ajouter les use qui vont bien ! (**ou utilisez PhpStorm avec le plugin Symfony**).

### Exercice

* Modifiez vos fichier pour permettre de filtrer sur un code postal le éditeur. Ajoutez des adresses et des éditeurs (avec le formulaire que vous avez créé sur la séance précédente). Vérifiez que tout fonctionne et que vous filtrez bien les éditeurs selon leur code postal.
* Mise en forme du formulaire.
  * En reprenant les éléments de la séance 7 du S3, et en installant Bootstrap, affichez le formulaire article sur 2 colonnes avec le thème de formulaire adapté . Vous devez obtenir le résultat ci-dessous :

<figure><img src="../.gitbook/assets/Capture d’écran 2023-02-10 à 08.46.43.png" alt=""><figcaption></figcaption></figure>
