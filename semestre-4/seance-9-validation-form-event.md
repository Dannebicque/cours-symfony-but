# Séance 9 : Formulaires et validations

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

Pour ajouter des règles de validation sur un champ, nous allons utiliser les contraintes de validation. Pour cela nous devons modifier notre formulaire `JeuType` :

```php
<?php

namespace App\Form;

...
use Symfony\Component\Validator\Constraints\Length;
use Symfony\Component\Validator\Constraints\NotBlank;
use Symfony\Component\Validator\Constraints\Positive;
use Symfony\Component\Validator\Constraints\Type;

class JeuType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('nom', TextType::class, [
                'label' => 'Nom du jeu',
                'help' => 'Le nom du jeu',
                'constraints' => [new Length(['min' => 2, 'max' => 100, 'minMessage' => 'Le nom doit faire au moins {{ limit }} caractères', 'maxMessage' => 'Le nom doit faire au plus {{ limit }} caractères'])],
            ])
            ->add('duree', TextareaType::class, [
                'label' => 'Durée d\'une partie',
                'help' => 'La durée en moyenne d\'une partie',
                'constraints' => [new NotBlank(['message' => 'La durée moyenne d\'une partie ne peut pas être vide'])
            ]])
            ...
        ;
    }

   ...
}
```

Dans cet exemple nous avons ajouté une contrainte de type `Length` sur le champ `nom`. Nous avons également ajouté une contrainte de type `NotBlank` sur le champ `duree`. Nous pouvons ajouter autant de contraintes que nous le souhaitons sur un champ.

La liste des contraintes par défaut est disponible sur la [documentation de Symfony](https://symfony.com/doc/current/reference/constraints.html).

Pour chaque contrainte on peut définir un message. Ce message sera affiché sous le champs qui pose problème. (`form_errors` dans le thème de votre formulaire pour le personnaliser).

### Validation des entités

Cette syntaxe est intéressante, mais ne vérifie les données que si le formulaire est soumis. Dans les autres cas, les données ne sont pas forcément validées. Selon votre fonctionnement, il peut être nécessaire de les valider (exemple envoie depuis une API, import de données, ...).

Pour valider les entités, nous allons utiliser les attributs de validation. Les possibilités sont les mêmes que pour les contraintes de validation sur les formulaires. La syntaxe utilise les attributs (depuis PHP 8.0).

Pour cela nous pouvons modifier notre entité `Editeur` par exemple, pour le code postal :

```php
<?php

namespace App\Entity;

use App\Repository\AdresseRepository;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;

#[ORM\Entity(repositoryClass: EditeurRepository::class)]
class Editeur
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
