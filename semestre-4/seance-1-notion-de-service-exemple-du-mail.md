# Séance 1 : Notion de service, exemple du mail

## Cours

Dans Symfony, tous les objets (au sens d'un ensemble de code qui apporte une fonctionnalité : mail, base de donnée, entitée, ...) sont des services. Dès l'instanciation d'une application symfony, de nombreux services sont lancés et accessible de partout dans l'application. Il est aussi possible de créer ses propres services.

La documentation officielle de Symfony sur les services : https://symfony.com/doc/current/service\_container.html

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

Il est possible de cumuler des services et des paramètres issus de nos routes. D'ailleurs, vous le faites déjà dans vos controllers. Par exemple, dans le controller `ArticleController`, on pourrait avoir :

```php
<?php

namespace App\Controller;

use App\Entity\Article;
use App\Form\ArticleType;
use App\Repository\ArticleRepository;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class ArticleController extends AbstractController
{
    #[Route('/article', name: 'article')]
    public function index(ArticleRepository $articleRepository): Response
    {
        $articles = $articleRepository->findAll();

        return $this->render('article/index.html.twig', [
            'articles' => $articles,
        ]);
    }

    #[Route('/article/{id}/edit', name: 'article_edit', methods: ['GET', 'POST'])]
    public function edit(Request $request, Article $article, EntityManagerInterface $entityManager): Response
    {
        $form = $this->createForm(ArticleType::class, $article);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $entityManager->flush();

            return $this->redirectToRoute('article');
        }

        return $this->render('article/edit.html.twig', [
            'article' => $article,
            'form' => $form->createView(),
        ]);
    }
}
```

La méthode `index` utilise le service `ArticleRepository` pour récupérer tous les articles. La méthode `edit` utilise les services `Request`et `EntityManagerInterface` pour récupérer les données du formulaire et sauvegarder les modifications d'un article, qui est passé en paramètre de la méthode et dans la route.

Notez au passage que la récupération de l'article déclenche aussi un mécanisme particulier qui vient executer une requete SQL pour récupérer l'article à partir de l'id passé en paramètre de la route. C'est le mécanisme de "ParamConverter".

### Création d'un service

Tout comme nous disposons de nombreux services déjà créés, il est possible de créer ses propres services. Pour cela, il faut créer une classe qui va contenir le code de notre service. Vous pouvez mettre ce fichier dans un repertoire dédié, par exemple `src/Service`. Par définition et sauf indication contraire cette classe sera considérée comme un service par Symfony.

Exemple, créons un service qui va nous permettre de récupérer le nombre d'articles dans la base de données :

```php
<?php

namespace App\Service;

use App\Repository\ArticleRepository;

class ArticleService
{
    private ArticleRepository $articleRepository;

    public function __construct(ArticleRepository $articleRepository)
    {
        $this->articleRepository = $articleRepository;
    }

    public function countArticles(): int
    {
        return $this->articleRepository->count([]);
    }
}
```

Dans cet exemple, on a créé une classe `ArticleService` qui contient une méthode `countArticles` qui va nous permettre de récupérer le nombre d'articles dans la base de données.

Notez que dans ce service, on utilise le service `ArticleRepository` qui est injecté dans le constructeur de notre classe.

Pour utiliser ce service, on peut l'injecter dans un controller :

```php
<?php

namespace App\Controller;

use App\Service\ArticleService;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class ArticleController extends AbstractController
{
    #[Route('/article', name: 'article')]
    public function index(ArticleService $articleService): Response
    {
        $count = $articleService->countArticles();

        return $this->render('article/index.html.twig', [
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

## Exercice de mise en pratique

### Reprenons quelques bases

Pour débuter ce nouveau semestre, nous allons repartir d'un projet Symfony vierge. Pour cela, nous allons utiliser le projet Symfony "skeleton" qui est un projet vierge. Pour créer ce projet, il faut utiliser la commande suivante :

```bash
composer create-project symfony/skeleton symfonyS4
```

ou

```bash
symfony new symfonyS4
```

N'oubliez pas de faire le nécessaire du côté de docker pour rendre ce projet accessible depuis votre navigateur sur l'adresse `http://localhost:8080/symfonyS4`. (Url à adapter en fonction de votre configuration).

Nous allons installer le bundle `MakerBundle` qui va nous permettre de créer des entités, des controllers, des services, etc. Pour cela, il faut lancer la commande suivante :

```bash
composer require profiler maker --dev
```

/!\ Attention, il faut que ces bundles soient installés en mode "dev" et non pas en mode "prod". Pour cela, il faut ajouter le paramètre `--dev` à la fin de la commande.

Nous installer également les dépendances suivantes :

```bash
composer require orm form validation annotation twig security mailer
```

### Utilisatation du mailer

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
