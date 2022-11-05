---
description: Mise en place d'une page de contact
---

# Séance 9 : Mails

## Installation du composant de mail

```
composer require symfony/mailer
```

{% hint style="info" %}
Cette commande n'est pas nécessaire si vous avez installé le projet Symfony avec "webapp"
{% endhint %}

Il faut ensuite configurer le serveur de mail qui sera utilisé.

Pour cela modifier le fichier `.env.local` (rappel: on ne modifie jamais le `.env` avec vos données personnelles).

Modifier la ligne :

```
MAILER_DSN=smtp://user:pass@smtp.example.com:port
```

{% hint style="info" %}
Symfony propose par défaut l'usage du SMTP, mais vous pouvez ajouter d'autres gestionnaires de mail (gmail, mailChimp...). La liste ici : [https://symfony.com/doc/current/mailer.html#using-a-3rd-party-transport](https://symfony.com/doc/current/mailer.html#using-a-3rd-party-transport)
{% endhint %}

## Envoyer un mail simple

Prenons l'exemple de Symfony

{% code title="" lineNumbers="true" %}
```php
// src/Controller/MailerController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Mime\Email;
use Symfony\Component\Routing\Annotation\Route;

class MailerController extends AbstractController
{
    #[Route('/email')]
    public function sendEmail(MailerInterface $mailer): Response
    {
        $email = (new Email())
            ->from('hello@example.com')
            ->to('you@example.com')
            //->cc('cc@example.com')
            //->addCc('deuxiemecc@example.com')
            //->addTo('deuxiemeto@example.com')
            //->bcc('bcc@example.com')
            //->replyTo('fabien@example.com')
            //->priority(Email::PRIORITY_HIGH)
            ->subject('Time for Symfony Mailer!')
            ->text('Sending emails is fun again!')
            ->html('<p>See Twig integration for better HTML integration!</p>');

        $mailer->send($email);

        // ...
    }
}
```
{% endcode %}

Cet exemple ajoute une route (email), qui est associé à une méthode (sendEmail) dans le contrôleur (MailerController).&#x20;

Modifiez les adresses ligne 16 (l'expéditeur) et ligne 17 (le destinataire).

Vous pourriez ajouter un ensemble d'option (cc, bcc, reply...)

* Ligne 22 : C'es le sujet de votre mail
* Ligne 23 : C'est votre email au format texte brut (sans HTML donc)
* Ligne 24 : C'est le contenu du "body" de votre mail au format HTML

{% hint style="info" %}
Dans cette construction, le mail est envoyé au format HTML et au format texte, c'est votre outil de mail qui choisira le plus approprié à afficher. Dans l'exemple les contenus sont différents selon le format, ca n'est qu'un exemple bien sûr.
{% endhint %}

### A faire

Saisir le code, configurer le serveur de mail, modifier les emails et le texte.

### Profiler

<figure><img src="../.gitbook/assets/Capture d’écran 2022-11-05 à 10.41.47.png" alt=""><figcaption><p>Profiler</p></figcaption></figure>

En mode développement, on interdit la distribution des mails réellement (on définit MAILER\_DSN=null). Pour voir les mails, on peut donc utiliser le profiler (le dernier icône de la barre).

{% hint style="info" %}
On peut aussi utiliser un outil type MailTrap, mailDev, ... qui va simuler un SMTP et récupérer l'ensemble des mails envoyés par Symfony.&#x20;
{% endhint %}

On peut voir l'ensemble des éléments du mail, le format HTML, Text, brut, l'en-tête...

<figure><img src="../.gitbook/assets/Capture d’écran 2022-11-05 à 10.44.18.png" alt=""><figcaption></figcaption></figure>

## Envoyer un mail au format HTML

La solution précédente est fonctionnelle, mais pas très pratique si vous souhaitez faire un long mail, ou un mail complexe, ou encore gérer de nombreuses données.

Il est donc possible de construire un email en utilisant la puissance de Twig et des templates.

{% code lineNumbers="true" %}
```php
// src/Controller/MailerController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Mailer\MailerInterface;
use Symfony\Bridge\Twig\Mime\TemplatedEmail;
use Symfony\Component\Routing\Annotation\Route;

class MailerController extends AbstractController
{
    #[Route('/email2')]
    public function sendEmail2(MailerInterface $mailer): Response
    {
      $email = (new TemplatedEmail())
              ->from('fabien@example.com')
              ->to(new Address('ryan@example.com'))
              ->subject('Thanks for signing up!')
          
              // path of the Twig template to render
              ->htmlTemplate('emails/signup.html.twig')
          
              // pass variables (name => value) to the template
              ->context([
                  'expiration_date' => new \DateTime('+7 days'),
                  'username' => 'foo',
              ])
          ;

        // ...
        $mailer->send($email);
    }
}

```
{% endcode %}

Dans ce deuxième exemple on créé un objet TemplatedEmail qui permet de manipuler des templates Twig.

La structure de base ne change pas (destinataires,... sujet).&#x20;

* Ligne 21 : on associe un template (dans le répertoire templates, puis emails/signup.html.twig)
* Ligne 24-27 : on passe les paramètres nécessaires à notre vue (comme on le ferait depuis un contrôleur).

Le template de mail pourrait ressembler à :&#x20;

```twig
{# templates/emails/signup.html.twig #}
<h1>Welcome {{ email.toName }}!</h1>

<p>
    You signed up as {{ username }} the following email:
</p>
<p><code>{{ email.to[0].address }}</code></p>

<p>
    <a href="#">Click here to activate your account</a>
    (this link is valid until {{ expiration_date|date('F jS') }})
</p>
```

{% hint style="info" %}
Dans cet exemple seul le format HTML est proposé. SI l'outil de mail ne supporte pas le format HTML il va afficher un format texte brut que Symfony aussi essayé de construire automatiquement à partir de votre format HTML
{% endhint %}

Pour ajouter le format texte du mail on peut passer un template spécifique pour le format texte, qui va utiliser les mêmes données fournies par context.

```
->textTemplate('emails/signup.txt.twig')
```

Ce deuxième template (txt) ne devra pas contenir de balises HTML.

### Tester

Tester ce second envoi d'email en format HTML et texte

## Gestion des adresses emails

Dans Symfony il existe plusieurs façon de gérer les adresses emails. Vous trouverez ci-après quelques exemples.

```php
// ...
use Symfony\Component\Mime\Address;

$email = (new Email())
    // email address as a simple string
    ->from('fabien@example.com')

    // email address as an object
    ->from(new Address('fabien@example.com'))

    // defining the email address and name as an object
    // (email clients will display the name)
    ->from(new Address('fabien@example.com', 'Fabien'))

    // defining the email address and name as a string
    // (the format must match: 'Name <email@example.com>')
    ->from(Address::create('Fabien Potencier <fabien@example.com>'))

    // ...
;
```

## Faire un formulaire de contact

Exercice :&#x20;

Créer un formulaire de contact.

* Créer un nouveau contrôleur nommé ContactController
* Ajouter une méthode index qui va se charger d'afficher un formulaire (adresse de l'expéditeur, sujet, message), vous serez le destinataire par défaut..
  * Vous pouvez créer le formulaire directement dans le template en HTML ou avec l'objet FormBuilder à votre convenance.
* Ajouter une méthode sendMail qui va se charger d'envoyer l'email en fonction des données du formulaire
  * On utilisera Request pour récupérer les données du formulaire
  * L'email sera envoyé à vous et en copie à la personne ayant complété le formulaire
  * Le champ reply sera configuré pour répondre à l'expéditeur
  * Le mail sera au format texte et HTML
