# Séance 2 : Formulaires

Pour initialiser cette séance, récupérez les fichiers ci-dessous et les mettre dans les bons répertoires (entity ou repository).  Configurer et mettre à jour votre base de données avec des 3 entités.&#x20;

{% file src="../.gitbook/assets/Adresse.php" %}

{% file src="../.gitbook/assets/AdresseRepository.php" %}

{% file src="../.gitbook/assets/Article.php" %}

{% file src="../.gitbook/assets/ArticleRepository.php" %}

{% file src="../.gitbook/assets/Fournisseur.php" %}

{% file src="../.gitbook/assets/FournisseurRepository.php" %}

## Mise en place d'un premier formulaire (rappel)

Pour se simplifier un peu la tâche, on peut commencer avec une base générée par le "maker" avec la commande ci-dessous :&#x20;

```
bin/console make:form
```

Répondez ensuite aux questions, la première est le nom du fichier qui va être créé, par exemple **AdresseType** dans cette partie. La seconde question concerne l'entité liée au formulaire (ou une classe). Indiquez ici **Adresse**.

Cela va donc créer le fichier AdresseType.php dans le répertoire Form.

Le fichier devrait ressembler à l'exemple ci-dessous :&#x20;

{% code title="AdresseType.php" lineNumbers="true" %}
```php
<?php

namespace App\Form;

use App\Entity\Adresse;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class AdresseType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('adresse1')
            ->add('adresse2')
            ->add('codePostal')
            ->add('ville')
        ;
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Adresse::class,
        ]);
    }
}
```
{% endcode %}

Par défaut, les lignes 15 à 18, sont les champs de votre entité (qui est précisée ligne 25). Ces champs sont automatiquement configurés selon leur type. On peut préciser le type de champ en paramètre 2, et le 3éme paramètre sont les options (cf [Séance 7 : Formulaires](../semestre-3/form.md)).

On peut par exemple préciser les éléments sur le champs adresse1 en modifiant la ligne 15 :

```php
->add('adresse1', TextType::class, [
    'label' => 'Adresse',
    'help' => 'Numéro, rue',
    'attr' => ['class' => 'form-control']
])
```

On ajoute le label, un texte d'aide (help), et une classe CSS.

{% hint style="warning" %}
Exercice :&#x20;

* Faites de même sur les autres champs.
* En consultant la documentation de Symfony, vos connaissances du HTML, configurez le champs "codePostal" avec une taille maximale de 5 caractères.
{% endhint %}
