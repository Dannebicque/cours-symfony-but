# Séance 3 : Formulaires

## Formulaire pour article

Nous allons mettre en place le formulaire pour les articles. Dans cette entité nous avons une liaison vers fournisseur. Cette relation est de type 1..n (un article à un fournisseur, un fournisseur peut être dans plusieurs articles).

* Générez le formulaire pour article et configurez les champs "simple".

Pour la liaison entre les deux table, il faut un champ de type entity. Le code pourrait être celui ci-dessous :&#x20;

```php
->add('fournisseur', EntityType::class, [
    'class' => Fournisseur::class,
    'choice_label' => 'libelle',
])
```

Vous pouvez définir d'autres options.

N'oubliez pas d'ajouter les use qui vont bien ! (**ou utilisez PhpStorm avec le plugin Symfony**).

* Faite le contrôleur pour afficher et sauvegarder un article.
