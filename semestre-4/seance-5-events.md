# Séance 5 : Event Dispatcher/Listener

Symfony (comme beaucoup de framework) émét des événements à chaque étape de son cycle de vie (préparation de la requête, traitement de la requête, génération de la réponse, etc.). Ces événements peuvent être écoutés par des listeners qui peuvent modifier le comportement de Symfony.

Les événements permettent aussi d'éviter de surcharger un contrôleur ou un service. Par exemple, un utilisateur s'inscrit sur votre site. Vous souhaitez lui envoyer un email, puis une notifaction push et notifier d'autres utilisateurs d'un nouvel inscrit par exemple.

1. Vous pourriez ajouter ces fonctionnalités dans le contrôleur qui gère l'inscription. Mais si vous avez besoin de faire la même chose dans un autre contrôleur, vous allez devoir dupliquer le code.
2. Si vous souhaitez retirer l'un des comportements, vous allez devoir le retirer dans tous les contrôleurs qui l'utilisent.
3. Si vous souhaitez ajouter un comportement vous allez devoir modifier tous les contrôleurs qui l'utilisent.

Les points 2 et 3 ne sont pas forcément problèmatique si vous êtes propriétaire du code. Mais si vous développez un bundle ou utilisez un bundle existant, vous ne pourrez peut etre pas modifier le code.

Dans ces trois exemples il est donc possible de définir nos événements qui pourront ensuite être écoutés par un ou plusieurs listeners, qui chacun pourra mener une action.

Nous allons donc voir dans cette partie comment écouter des événements, et comment définir nos propres événements.

## Ecouter des événements (_EventSubscriber_)

Symfony (ou votre application) émet des événements à chaque étape de son cycle de vie. Ces événements sont écoutés par des listeners qui peuvent modifier le comportement de Symfony.

Pour écouter un événement, il faut créer une classe qui implémente l'interface `Symfony\Component\EventDispatcher\EventSubscriberInterface`. Cette interface contient une méthode `getSubscribedEvents()` qui retourne un tableau associatif. La clé est le nom de l'événement et la valeur est la méthode à appeler.

Un exemple pourrait être :

{% code lineNumbers="true" %}
```php
<?php

namespace App\EventSubscriber;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;

class MyEventSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents()
    {
        return [
            'kernel.request' => 'onKernelRequest',
        ];
    }

    public function onKernelRequest()
    {
        // ...
    }
}

```
{% endcode %}

Dans cet exemple, la méthode `onKernelRequest()` sera appelée à chaque fois que l'événement `kernel.request` sera émis.

{% hint style="info" %}
Il existe aussi la possibilité de définir des listener avec une syntaxe différente qui ne va écouter qun' seul événement précis : [https://symfony.com/doc/current/event\_dispatcher.html#creating-an-event-listener](https://symfony.com/doc/current/event_dispatcher.html#creating-an-event-listener)
{% endhint %}

## Définir des événements (_Event_)

[https://symfony.com/doc/current/components/event\_dispatcher.html](https://symfony.com/doc/current/components/event_dispatcher.html)

Pour définir nos propres événements il faut deux choses :

1. Définir une classe qui hérite de `Symfony\Component\EventDispatcher\Event` qui va contenir la description et les données de notre événement.
2. "Emettre" l'événement avec la méthode `dispatch()` de l'objet `Symfony\Component\EventDispatcher\EventDispatcherInterface` quelque part dans notre application en lui passant un objet du type de la classe de l'événnement précédemment créée.

### Classe décrivant notre événement

Pour définir notre événement, il faut créer une classe qui hérite de `Symfony\Component\EventDispatcher\Event`. Cette classe doit contenir les données de l'événement. Par exemple, si nous voulons définir un événement qui sera émis à chaque fois qu'un utilisateur s'inscrit, nous pourrions avoir une classe comme celle-ci :

{% code lineNumbers="true" %}
```php
<?php

namespace App\Event;

use Symfony\Contracts\EventDispatcher\Event;

class UserRegisteredEvent extends Event
{
    const NAME = 'user.registered';

    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function getUser()
    {
        return $this->user;
    }
}

```
{% endcode %}

### Emettre l'événement

Pour émettre l'événement, il faut récupérer l'objet `Symfony\Component\EventDispatcher\EventDispatcherInterface` et appeler la méthode `dispatch()` en lui passant en premier paramètre le nom de l'événement et en second paramètre l'objet de l'événement.

{% code lineNumbers="true" %}
```php
<?php

namespace App\Controller;

use App\Event\UserRegisteredEvent;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\EventDispatcher\EventDispatcherInterface;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class UserController extends AbstractController
{
    /**
     * @Route("/register", name="user_register")
     */
    public function register(EventDispatcherInterface $eventDispatcher): Response
    {
        // ...

        $event = new UserRegisteredEvent($user);
        $eventDispatcher->dispatch($event, UserRegisteredEvent::NAME);

        // ...
    }
}

```
{% endcode %}

Et bien sûr définir un listener pour écouter l'événement, comme décrit dans la partie précédente, et qui pourrait être pour notre exemple :

{% code lineNumbers="true" %}
```php
<?php

namespace App\EventSubscriber;

use App\Event\UserRegisteredEvent;

class UserRegisteredSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents()
    {
        return [
            UserRegisteredEvent::NAME => 'onUserRegistered',
        ];
    }

    public function onUserRegistered(UserRegisteredEvent $event)
    {
        // ...
    }
}

```
{% endcode %}

## Exercices

### Exercice 1

Créer un événement `UserRegisteredEvent` qui sera émis à chaque fois qu'un utilisateur s'inscrit. Créer un listener qui enverra un email (en utilisant notre service) à l'utilisateur pour lui dire qu'il a bien été enregistré.

### Exercice 2

Créer deux événements `JeuAddedEvent` et `JeuUpdatedEvent` qui seront émis à chaque fois qu'un utilisateur ajoute ou modifie un jeu. Créer un listener qui enverra un email (en utilisant notre service) à l'administrateur du site (`admin@mmiple.fr)` pour lui indiquer qu'un jeu a été mis à jour ou ajouté avec les informations dans l'email.
