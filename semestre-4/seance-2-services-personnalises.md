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

{% hint style="warning" %}
Exercice :&#x20;

* Faites de même sur les autres champs.
* En consultant la documentation de Symfony, vos connaissances du HTML, configurez le champs "codePostal" avec une taille maximale de 5 caractères.
{% endhint %}

### Affichons le formulaire

Pour cela, on va créer un contrôleur (`make:controller`) et ajouter la méthode new pour afficher le formulaire.

{% code title="AdresseController.php" lineNumbers="true" %}
```php
<?php

namespace App\Controller;

use App\Entity\Adresse;
use App\Form\AdresseType;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class AdresseController extends AbstractController
{
    #[Route('/adresse', name: 'app_adresse_new')]
    public function new(): Response
    {
        $adresse = new Adresse();
        $form = $this->createForm(AdresseType::class, $adresse);

        return $this->render('adresse/new.html.twig', [
            'form' => $form->createView(),
        ]);
    }
}

```
{% endcode %}

Et la vue qui va avec.

{% code lineNumbers="true" %}
```twig
{% raw %}
{% extends 'base.html.twig' %}

{% block title %}Hello DefaultController!{% endblock %}

{% block body %}
<h1>Formulaire</h1>

{{ form_start(form) }}
    {{ form_widget(form) }}
    <button type="submit" class="btn btn-primary">Submit</button>
{{ form_end(form) }}
{% endblock %}
{% endraw %}
```
{% endcode %}

Rendez-vous sur l'url associée à votre méthode pour voir s'afficher le formulaire. Testé que le code postal est bien limité à 5 caractères par exemple.

## Imbrication de formulaires

L'entité Fournisseur est liée à l'entité Adresse. On peut donc construire un formulaire pour Fournisseurs, puis sélectionner une adresse parmi une liste d'adresse.

Mais ! Dans notre cas, Fournisseur et Adresse ont une liaison 1 vers 1 (c'est à dire qu'un fournisseur à une adresse et une adresse un seul fournisseur). On utilise ce type de découpage pour éviter de surcharger l'entité, ou parce que l'on va manipuler plusieurs fois un même type de données.

Pour résumer, la saisie d'un fournisseur implique la saisie de son adresse. Toujours pour éviter de multiplier le code, on va pouvoir réutiliser le formulaire AdresseType créé dans la partie précédente dans notre formulaire Fournisseur.

### Exercice

* Créer le formulaire pour l'entitée Fournisseur. Paramétrez les champs, et définissez comme type de  champs pour la propriété Adresse : `AdresseType`.

```php
->add('adresse', AdresseType::class, [])
```

N'oubliez pas le use associé (ou utilisez **PhpStorm** !)

* Ajoutez le contrôleur associé pour la gestion des fournisseurs, et la méthode pour afficher **et** sauvegarder le formulaire du fournisseur.
* Testez votre code et constatez que l'adresse est stockée dans sa table, et que la clé étrangère d'adresse est dans la table de fournisseur.
