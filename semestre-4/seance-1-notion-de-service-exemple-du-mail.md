# Séance 1 : Notion de service, exemple du mail

**Le concept de service a déjà été évoqué en S3, il s'agit ici d'un rappel et d'une mise en pratique.**

## Cours

Dans Symfony, tous les objets (au sens d'un ensemble de code qui apporte une fonctionnalité : mail, base de données, entitée, ...) sont des services. Dès l'instanciation d'une application symfony, de nombreux services sont lancés et accessible de partout dans l'application. Il est aussi possible de créer ses propres services.

La documentation officielle de Symfony sur les services : [https://symfony.com/doc/current/service\_container.html](https://symfony.com/doc/current/service_container.html)

### Utilisation d'un service existant (rappel sur l'usage des mails)

Dans un controller, on peut utiliser un service existant en l'injectant dans la méthode de notre classe "controller". Par exemple, pour envoyer un mail, on peut utiliser le service `mailer` :

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\Mime\Email;
use Symfony\Component\Mailer\MailerInterface;

class MailController extends AbstractController
{
    #[Route("/mail", name:"mail")]
    public function index(MailerInterface $mailer): Response
    {
        $email = (new Email())
            ->from('mail@mail.com')
            ->to('destinataire@mail.com')
            ->subject('Hello Email')
            ->text('Sending emails is fun again!')
            ->html('<p>See Twig integration for better HTML integration!</p>');

        $mailer->send($email);

        return $this->render('mail/index.html.twig', []);

    }
}
```

Dans cet exemple, on injecte le service `mailer` (par l'intermédiaire de son interface `MailerInterface`) dans la méthode `index` de notre controller. On peut ensuite utiliser ce service pour envoyer un mail.

Grâce au mécanisme d'injection de dépendance de Symfony, le service `mailer` est automatiquement instancié et injecté dans notre controller. On peut donc l'utiliser dans notre méthode `index`. Il n'est pas nécessaire de l'instancier nous même et de le passer à la méthode. Cela fonctionne parce que tout est considéré comme un service dans Symfony.

Il est possible de cumuler des services et des paramètres issus de nos routes. D'ailleurs, vous le faites déjà dans vos controllers. Par exemple, dans le controller `EditeurController`, on pourrait avoir :

```php
<?php

namespace App\Controller;

use App\Entity\Editeur;
use App\Form\EditeurType;
use App\Repository\EditeurRepository;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class EditeurController extends AbstractController
{
    #[Route('/editeur', name: 'editeur')]
    public function index(EditeurRepository $editeurRepository): Response
    {
        $editeurs = $editeurRepository->findAll();

        return $this->render('editeur/index.html.twig', [
            'editeurs' => $editeurs,
        ]);
    }

    #[Route('/editeur/{id}/edit', name: 'editeur_edit', methods: ['GET', 'POST'])]
    public function edit(Request $request, Editeur $editeur, EntityManagerInterface $entityManager): Response
    {
        $form = $this->createForm(EditeurType::class, $editeur);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $entityManager->flush();

            return $this->redirectToRoute('editeur');
        }

        return $this->render('editeur/edit.html.twig', [
            'editeur' => $editeur,
            'form' => $form->createView(),
        ]);
    }
}
```

La méthode `index` utilise le service `EditeurRepository` pour récupérer tous les éditeurs. La méthode `edit` utilise les services `Request`et `EntityManagerInterface` pour récupérer les données du formulaire et sauvegarder les modifications d'un article, qui est passé en paramètre de la méthode et dans la route.

Notez au passage que la récupération de l'éditeur déclenche aussi un mécanisme particulier qui vient executer une requete SQL pour récupérer l'éditeur à partir de l'id passé en paramètre de la route. C'est le mécanisme de "ParamConverter".

### Création d'un service

Tout comme nous disposons de nombreux services déjà créés, il est possible de créer ses propres services. Pour cela, il faut créer une classe qui va contenir le code de notre service. Vous pouvez mettre ce fichier dans un repertoire dédié, par exemple `src/Service`. Par définition et sauf indication contraire cette classe sera considérée comme un service par Symfony.

Exemple, créons un service qui va nous permettre de récupérer le nombre d'éditeurs dans la base de données :

```php
<?php

namespace App\Service;

use App\Repository\EditeurRepository;

class EditeurService
{
    public function __construct(private EditeurRepository $editeurRepository)
    {
    }

    public function countEditeurs(): int
    {
        return $this->editeurRepository->count([]);
    }
}
```

Dans cet exemple, on a créé une classe `EditeurService` qui contient une méthode `countEditeurs` qui va nous permettre de récupérer le nombre d'éditeur dans la base de données.

Notez que dans ce service, on utilise le service `EditeurRepository` qui est injecté dans le constructeur de notre classe.

Pour utiliser ce service, on peut l'injecter dans un controller :

```php
<?php

namespace App\Controller;

use App\Service\EditeurService;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class EditeurController extends AbstractController
{
    #[Route('/editeur', name: 'editeur')]
    public function index(EditeurService $editeurService): Response
    {
        $count = $editeurService->countEditeurs();

        return $this->render('editeur/index.html.twig', [
            'count' => $count,
        ]);
    }
}
```

### A quoi ca sert ?

Les services sont très utiles pour plusieurs raisons :

* Ils permettent de découper notre code en plusieurs parties. Par exemple, on peut avoir un service qui va nous permettre de récupérer les articles, un autre qui va nous permettre de récupérer les utilisateurs, un autre qui va nous permettre de récupérer les commentaires, etc.
* On peut ainsi avoir un service par entité. Cela permet de découper notre code en plusieurs parties et de rendre notre code plus lisible et plus maintenable.

Il n'y a pas de règle pour savoir quand créer un service. Cela dépend de votre code et de vos besoins. Par exemple, si vous avez un controller qui fait trop de choses, vous pouvez créer un service qui va vous permettre de déplacer une partie du code dans ce service.

L'objectif est de garder vos contrôleurs et les méthodes qu'ils contiennent relativement légers. Les services permettent aussi de réduire le code qui serait dupliqué dans plusieurs contrôleurs en mettant à un seul endroit les méthodes qui sont communes à plusieurs contrôleurs.

## Exercice de (re)découverte

### Reprenons quelques bases

**Pour faciliter le point de départ, vous avez une correction du S3 à votre disposition dans le dossier ci-dessous. Cette correction peut être incomplète par rapport au S3 et peu contenir des différences de nomages ou d'organisation.**

{% file src="../.gitbook/assets/mmiple-base.zip" %}
Projet de départ
{% endfile %}

Un fois le dossier récupéré il faut lancer les commandes suivantes :

* Installer les dépendances :

```bash
composer install
```

* Créer et mettre à jour le fichier `.env.local` avec les informations de connexion à votre base de données.
* puis créer la base de données :

```bash
php bin/console doctrine:database:create
```

* puis créer les tables :

```bash
php bin/console doctrine:schema:update --force
```

_On pourrait utiliser une migration comme vu en S3, mais pour simplifier nous allons utiliser cette commande qu va ici reconstruire toutes les tables de la base de données_

* Charger les données du fichier sql qui se trouve dans le dossier.
* N'oubliez pas de faire le nécessaire du côté de docker pour rendre ce projet accessible depuis votre navigateur sur l'adresse `http://mmiple.mmi-troyes:8319/`. (Url à adapter en fonction de votre configuration).

### Utilisation du mailer

Dans un premier temps vous allez créer un contrôleur et une méthode, associée à une route permettant d'envoyer un mail avec avec des informations pré-remplies (n'hésitez pas à reprendre les séances du S3 sur le mailer).

### Création d'un service

Dans un deuxième temps nous allons utiliser notre propre service de mailer pour gérer l'envoi des mails.

L'intérêt de faire cela est que nous pourrons ainsi définir un ensemble de paramètre par défaut pour nos mails (expéditeur, destinataire, sujet, etc.) et ainsi simplifier l'utilisation de notre service, sans repeter à chaque fois que nécessaire ces paramètres.

L'autre intéret est que nous pourrons ainsi facilement changer les paramètres de notre service de mailer, voire changer de bibliothèque, sans avoir à modifier le code de nos contrôleurs.

Cela évite donc que nos contrôleurs et notre code ne soit trop dépendant des bibliothèques que nous utilisons.

1. Créer une classe `MailerService` dans le dossier `src/Service` et ajouter les méthodes suivantes :
   * le constructeur qui prend en paramètre l'objet `MailerInterface` et qui le stocke dans une propriété
   * `sendMail` : qui prend en paramètre le destinataire, le sujet et le corps du mail et qui envoie le mail
2. Modifier le contrôleur pour utiliser le service de mailer
3. Pour aller plus loin
   1. Ajoutez une méthode qui permet d'envoyer un email en utilisant "twig"
      1. Quelles sont les propriétés nécessaires ?
      2. Comment modifier le code ?

## Exercice de mise en pratique

* Créer un formulaire de contact.
* Créer un nouveau contrôleur nommé ContactController
* Ajouter une méthode index qui va se charger d'afficher le formulaire de contact (adresse de l'expéditeur, sujet, message), vous serez le destinataire par défaut..
* Ajouter une méthode send qui va se charger de récupérer les informations du formulaire et de les envoyer par mail en utilisant le service de mailer.
