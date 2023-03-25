# Séance 10 : Validation des formulaires

## Validation des formulaires

La documentation officielle se trouve ici : [https://symfony.com/doc/current/validation.html](https://symfony.com/doc/current/validation.html)

Depuis le S3, nous écrivons systèmatiqument le code suivant pour enregistrer un formulaire :

```php
if ($form->isSubmitted() && $form->isValid()) {
   ...
}
```

Ce code fait deux choses. Il vérifie que le formulaire a été soumis, et qu'il est valide. Jusqu'a présent le coté "validation" n'est pas géré car nous n'avons pas mis de règles de validation sur nos formulaires. Nous allons donc voir comment ajouter des règles de validation sur nos formulaires.

### Validation des champs

Pour ajouter des règles de validation sur un champ, nous allons utiliser les contraintes de validation. Pour cela nous devons modifier notre formulaire `ArticleType` :

```php
<?php

namespace App\Form;

...
use Symfony\Component\Validator\Constraints\Length;
use Symfony\Component\Validator\Constraints\NotBlank;
use Symfony\Component\Validator\Constraints\Positive;
use Symfony\Component\Validator\Constraints\Type;

class ArticleType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('designation', TextType::class, [
                'label' => 'Désignation',
                'help' => 'La désignation de l\'article',
                'constraints' => [new Length(['min' => 5, 'max' => 5, 'minMessage' => 'La désignation doit faire au moins {{ limit }} caractères', 'maxMessage' => 'La désignation doit faire au plus {{ limit }} caractères'])],
            ])
            ->add('description', TextareaType::class, [
                'label' => 'Description',
                'help' => 'La description de l\'article',
                'constraints' => [new NotBlank(['message' => 'La description ne peut pas être vide'])
            ]])
            ...
        ;
    }

   ...
}
```

Dans cet exemple nous avons ajouté une contrainte de type `Length` sur le champ `designation`. Nous avons également ajouté une contrainte de type `NotBlank` sur le champ `description`. Nous pouvons ajouter autant de contraintes que nous le souhaitons sur un champ.

La liste des contraintes par défaut est disponible sur la [documentation de Symfony](https://symfony.com/doc/current/reference/constraints.html).

Pour chaque contrainte on peut définir un message. Ce message sera affiché sous le champs qui pose problème. (`form_errors` dans le thème de votre formulaire pour le personnaliser).

### Validation des entités

Cette syntaxe est intéressante, mais ne vérifie les données que si le formulaire est soumis. Dans les autres cas, les données ne sont pas forcéments validées. Selon votre fonctionnement, il peut être nécessaire de les valider (exemple envoie depuis une API, import de données, ...).

Pour valider les entités, nous allons utiliser les annotations de validation. Les possibilités sont les mêmes que pour les contraintes de validation sur les formulaires. La syntaxe utilise les attributs (depuis PHP 8.0) ou les annotations (PHP 7.4 et inférieur).

Pour cela nous pouvons modifier notre entité `Adresse` par exemple, pour le code postal :

```php
<?php

namespace App\Entity;

use App\Repository\AdresseRepository;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;

#[ORM\Entity(repositoryClass: AdresseRepository::class)]
class Adresse
{
    #[ORM\Column(length: 5)]
    #[Assert\Length(min: 5, max: 5)]
    private ?string $codePostal = null;
...
}
```

### Exercice

* Ajoutez les contraintes de validations sur vos formulaires. Testez les en soumettant des données invalides.
* Personnalisés l'affiche des erreurs dans vos formulaires
