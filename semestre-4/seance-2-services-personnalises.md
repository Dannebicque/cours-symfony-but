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

