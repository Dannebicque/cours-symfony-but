---
description: 'La documentation officielle : https://symfony.com/doc/current/security.html'
---

# Séance 10 : Sécurité

## Introduction

Deux notions majeures interviennent dans la conception de sécurité de Symfony :

* _Authentification_ : **Qui êtes vous ?** ; vous pouvez vous authentifier de plusieurs manières (HTTP authentification, certificat, formulaire de login, API, OAuth etc)
* _Authorization_ : **Avez vous accès à ?** ; permet d'autoriser de faire telle ou telle action ou accéder à telle page sans forcément savoir qui vous êtes, utilisateur anonyme par exemple.

Pour fonctionner, il est nécessaire d'ajouter le composant security à votre symfony.

```
composer require symfony/security-bundle
```

{% hint style="info" %}
Si vous avez installé le projet avec la version complète (webapp), cette ligne n'est pas nécessaire.
{% endhint %}

La sécurité dans symfony implique plusieurs éléments :

* Le firewall: qui est la porte d'entrée pour le système d'authentification, on définit différents firewall (au minimum 1 seul) qui va permettre de mettre en place le bon système de connexion pour l'url spécifiée via un pattern.
* Le provider : qui permet au firewall d'interroger une collection d'utilisateurs/mot de passe ; C'est une sorte de base de tous les utilisateurs avec les mots de passe. Il existe deux type par défaut :
  * in memory : directement dans le fichier security.yml mais du coup les hash des mots de passes sont disponible dans un fichier&#x20;
  *   Entity : N'importe quelle entité qui implémente à minima les deux interfaces

      ```
      use Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface;
      use Symfony\Component\Security\Core\User\UserInterface;
      ```
  * Enfin, plusieurs providers peuvent fonctionner en même temps par exemple in\_memory et entity voire plusieurs entités simultanément. [http://symfony.com/doc/current/security/entity\_provider.html](http://symfony.com/doc/current/security/entity\_provider.html)
* Un encoder : qui permet de générer des hashs/d'encoder des mots de passe ; le plus connu étant MD5 mais vous pouvez utiliser d'autres encoders tels que : sha1, bcrypt ou plaintext (qui n'encode rien c'est le mot de passe en clair) [http://symfony.com/doc/current/security/named\_encoders.html](http://symfony.com/doc/current/security/named\_encoders.html)
* Les rôles : qui permettent de définir le niveau d'accès des utilisateurs connectés (authentifiés) et de configurer le firewall en fonction de ces rôles. Les rôles peuvent être hierarchisées afin d'expliquer par exemple qu'un administrateur (ROLE\_ADMIN par exemple) et avant tout un utilisateur (ROLE\_USER).
* Le "guard"  ou "authenticator" qui va gérer l'authentification, au travers de "passport". Il va par exemple vérifier que le couple login/mot de passe existe dans l'un des provider.

## Configuration

A partir de la version 4, et avec le composant "maker", la gestion de la sécurité a été grandement facilitée. Là où sur les précédentes versions (la 2 notamment), il était d'usage de passer par un bundle tierce (FOSUserBundle par exemple), aujourd'hui cela n'est plus nécessaire.

### Créer sa classe User

Si vous ne disposez pas encore d'une classe permettant la gestion des utilisateurs, il est possible d'en créer une avec la console. Si vous disposez déjà d'une classe utilisateur (ou que vous souhaitez utiliser plusieurs entités, il faudra modifier votre code en implémentant les interfaces `UserInterface, PasswordAuthenticatedUserInterface` et en implémentant les méthodes imposées par ces interfaces).

L'instruction ci-dessous permet de lancer la console pour créer la table User.

```
bin/console make:user
```

Symfony va vous poser plusieurs questions afin de configurer les éléments (le nom de l'entité, si vous utilisez doctrine, le champ correspondant au login, et l'encodage du password.

```bash
bin/console make:user                                                                                                    davidannebicque@MacBook-Pro-de-David-2

 The name of the security user class (e.g. User) [User]:
 > 

 Do you want to store user data in the database (via Doctrine)? (yes/no) [yes]:
 > 

 Enter a property name that will be the unique "display" name for the user (e.g. email, username, uuid) [email]:
 > 

 Will this app need to hash/check user passwords? Choose No if passwords are not needed or will be checked/hashed by some other system (e.g. a single sign-on server).

 Does this app need to hash/check user passwords? (yes/no) [yes]:
 > 

 created: src/Entity/User.php
 created: src/Repository/UserRepository.php
 updated: src/Entity/User.php
 updated: config/packages/security.yaml

           
  Success! 
           

 Next Steps:
   - Review your new App\Entity\User class.
   - Use make:entity to add more fields to your User entity and then run make:migration.
   - Create a way to authenticate! See https://symfony.com/doc/current/security.html
```

Une fois cette commande exécutée vous avez un fichier d'entité de créé, un repository associé, et le fichier security.yaml (dans config) qui a été mis à jour.

### Le fichier entité

```php
<?php

namespace App\Entity;

use App\Repository\UserRepository;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface;
use Symfony\Component\Security\Core\User\UserInterface;

#[ORM\Entity(repositoryClass: UserRepository::class)]
class User implements UserInterface, PasswordAuthenticatedUserInterface
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 180, unique: true)]
    private ?string $email = null;

    #[ORM\Column]
    private array $roles = [];

    /**
     * @var string The hashed password
     */
    #[ORM\Column]
    private ?string $password = null;

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getEmail(): ?string
    {
        return $this->email;
    }

    public function setEmail(string $email): self
    {
        $this->email = $email;

        return $this;
    }

    /**
     * A visual identifier that represents this user.
     *
     * @see UserInterface
     */
    public function getUserIdentifier(): string
    {
        return (string) $this->email;
    }

    /**
     * @see UserInterface
     */
    public function getRoles(): array
    {
        $roles = $this->roles;
        // guarantee every user at least has ROLE_USER
        $roles[] = 'ROLE_USER';

        return array_unique($roles);
    }

    public function setRoles(array $roles): self
    {
        $this->roles = $roles;

        return $this;
    }

    /**
     * @see PasswordAuthenticatedUserInterface
     */
    public function getPassword(): string
    {
        return $this->password;
    }

    public function setPassword(string $password): self
    {
        $this->password = $password;

        return $this;
    }

    /**
     * @see UserInterface
     */
    public function eraseCredentials()
    {
        // If you store any temporary, sensitive data on the user, clear it here
        // $this->plainPassword = null;
    }
}

```

### Le fichier security.yaml

Ce fichier fait le lien avec l'entité User (le provider), le login retenu (ici un email), et l'encodage du mot de passe, par défaut "auto"

```yaml
security:
    # https://symfony.com/doc/current/security.html#registering-the-user-hashing-passwords
    password_hashers:
        Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface: 'auto'
    # https://symfony.com/doc/current/security.html#loading-the-user-the-user-provider
    providers:
        # used to reload user from session & other features (e.g. switch_user)
        app_user_provider:
            entity:
                class: App\Entity\User
                property: email
    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false
        main:
            lazy: true
            provider: app_user_provider

            # activate different ways to authenticate
            # https://symfony.com/doc/current/security.html#the-firewall

            # https://symfony.com/doc/current/security/impersonating_user.html
            # switch_user: true

    # Easy way to control access for large sections of your site
    # Note: Only the *first* access control that matches will be used
    access_control:
        # - { path: ^/admin, roles: ROLE_ADMIN }
        # - { path: ^/profile, roles: ROLE_USER }

when@test:
    security:
        password_hashers:
            # By default, password hashers are resource intensive and take time. This is
            # important to generate secure password hashes. In tests however, secure hashes
            # are not important, waste resources and increase test times. The following
            # reduces the work factor to the lowest possible values.
            Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface:
                algorithm: auto
                cost: 4 # Lowest possible value for bcrypt
                time_cost: 3 # Lowest possible value for argon
                memory_cost: 10 # Lowest possible value for argon

```

### Mise à jour de la BDD

Il faut ensuite mettre à jour votre base de données, avec les commandes suivantes:

```
bin/console d:s:u -f

#ou

bin/console make:migration
bin/console doctrine:migrations:migrate
```

## Créer la partie connexion

Une nouvelle fois la console va nous permettre de dégrossir le travail et produire le contrôleur, le fichier de configuration et le formulaire de connexion.

```
bin/console make:auth
```

Pour le résultat ci-dessous.

```bash
bin/console make:auth                                                                                                    davidannebicque@MacBook-Pro-de-David-2

 What style of authentication do you want? [Empty authenticator]:
  [0] Empty authenticator
  [1] Login form authenticator
 > 1

 The class name of the authenticator to create (e.g. AppCustomAuthenticator):
 > LoginAuthenticator

 Choose a name for the controller class (e.g. SecurityController) [SecurityController]:
 > 

 Do you want to generate a '/logout' URL? (yes/no) [yes]:
 > 

 created: src/Security/LoginAuthenticator.php
 updated: config/packages/security.yaml
 created: src/Controller/SecurityController.php
 created: templates/security/login.html.twig

           
  Success! 
           

 Next:
 - Customize your new authenticator.
 - Finish the redirect "TODO" in the App\Security\LoginAuthenticator::onAuthenticationSuccess() method.
 - Review & adapt the login template: templates/security/login.html.twig.
```

Cette commande, comme indiqué génère plusieurs fichiers :

* LoginAuthenticator.php : qui explique comment on authentifie un utilisateur (au travers de passeport)
* SecurityController.php : Car ici, nous avons choisi une connexion avec formulaire, ce contrôleur permettra d'afficher la page de login, de récupérer les informations et de gérer la déconnexion
* login.html.twig : qui contient le formulaire

Le fichier security.yaml est mis à jour pour faire le lien avec cet authenticator.

### LoginAuthenticator

```php
<?php

namespace App\Security;

use Symfony\Component\HttpFoundation\RedirectResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Generator\UrlGeneratorInterface;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Security;
use Symfony\Component\Security\Http\Authenticator\AbstractLoginFormAuthenticator;
use Symfony\Component\Security\Http\Authenticator\Passport\Badge\CsrfTokenBadge;
use Symfony\Component\Security\Http\Authenticator\Passport\Badge\UserBadge;
use Symfony\Component\Security\Http\Authenticator\Passport\Credentials\PasswordCredentials;
use Symfony\Component\Security\Http\Authenticator\Passport\Passport;
use Symfony\Component\Security\Http\Util\TargetPathTrait;

class LoginAuthenticator extends AbstractLoginFormAuthenticator
{
    use TargetPathTrait;

    public const LOGIN_ROUTE = 'app_login';

    public function __construct(private UrlGeneratorInterface $urlGenerator)
    {
    }

    public function authenticate(Request $request): Passport
    {
        $email = $request->request->get('email', '');

        $request->getSession()->set(Security::LAST_USERNAME, $email);

        return new Passport(
            new UserBadge($email),
            new PasswordCredentials($request->request->get('password', '')),
            [
                new CsrfTokenBadge('authenticate', $request->request->get('_csrf_token')),
            ]
        );
    }

    public function onAuthenticationSuccess(Request $request, TokenInterface $token, string $firewallName): ?Response
    {
        if ($targetPath = $this->getTargetPath($request->getSession(), $firewallName)) {
            return new RedirectResponse($targetPath);
        }

        // For example:
        // return new RedirectResponse($this->urlGenerator->generate('some_route'));
        throw new \Exception('TODO: provide a valid redirect inside '.__FILE__);
    }

    protected function getLoginUrl(Request $request): string
    {
        return $this->urlGenerator->generate(self::LOGIN_ROUTE);
    }
}

```

### SecurityController

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\Security\Http\Authentication\AuthenticationUtils;

class SecurityController extends AbstractController
{
    #[Route(path: '/login', name: 'app_login')]
    public function login(AuthenticationUtils $authenticationUtils): Response
    {
        // if ($this->getUser()) {
        //     return $this->redirectToRoute('target_path');
        // }

        // get the login error if there is one
        $error = $authenticationUtils->getLastAuthenticationError();
        // last username entered by the user
        $lastUsername = $authenticationUtils->getLastUsername();

        return $this->render('security/login.html.twig', ['last_username' => $lastUsername, 'error' => $error]);
    }

    #[Route(path: '/logout', name: 'app_logout')]
    public function logout(): void
    {
        throw new \LogicException('This method can be blank - it will be intercepted by the logout key on your firewall.');
    }
}

```

### login.html.twig

```twig
{% raw %}
{% extends 'base.html.twig' %}

{% block title %}Log in!{% endblock %}

{% block body %}
<form method="post">
    {% if error %}
        <div class="alert alert-danger">{{ error.messageKey|trans(error.messageData, 'security') }}</div>
    {% endif %}

    {% if app.user %}
        <div class="mb-3">
            You are logged in as {{ app.user.userIdentifier }}, <a href="{{ path('app_logout') }}">Logout</a>
        </div>
    {% endif %}

    <h1 class="h3 mb-3 font-weight-normal">Please sign in</h1>
    <label for="inputEmail">Email</label>
    <input type="email" value="{{ last_username }}" name="email" id="inputEmail" class="form-control" autocomplete="email" required autofocus>
    <label for="inputPassword">Password</label>
    <input type="password" name="password" id="inputPassword" class="form-control" autocomplete="current-password" required>

    <input type="hidden" name="_csrf_token"
           value="{{ csrf_token('authenticate') }}"
    >

    {#
        Uncomment this section and add a remember_me option below your firewall to activate remember me functionality.
        See https://symfony.com/doc/current/security/remember_me.html

        <div class="checkbox mb-3">
            <label>
                <input type="checkbox" name="_remember_me"> Remember me
            </label>
        </div>
    #}

    <button class="btn btn-lg btn-primary" type="submit">
        Sign in
    </button>
</form>
{% endblock %}
{% endraw %}

```

### Security.yaml mis à jour

```yaml
security:
    # https://symfony.com/doc/current/security.html#registering-the-user-hashing-passwords
    password_hashers:
        Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface: 'auto'
    # https://symfony.com/doc/current/security.html#loading-the-user-the-user-provider
    providers:
        # used to reload user from session & other features (e.g. switch_user)
        app_user_provider:
            entity:
                class: App\Entity\User
                property: email
    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false
        main:
            lazy: true
            provider: app_user_provider
            custom_authenticator: App\Security\LoginAuthenticator
            logout:
                path: app_logout
                # where to redirect after logout
                # target: app_any_route

            # activate different ways to authenticate
            # https://symfony.com/doc/current/security.html#the-firewall

            # https://symfony.com/doc/current/security/impersonating_user.html
            # switch_user: true

    # Easy way to control access for large sections of your site
    # Note: Only the *first* access control that matches will be used
    access_control:
        # - { path: ^/admin, roles: ROLE_ADMIN }
        # - { path: ^/profile, roles: ROLE_USER }

when@test:
    security:
        password_hashers:
            # By default, password hashers are resource intensive and take time. This is
            # important to generate secure password hashes. In tests however, secure hashes
            # are not important, waste resources and increase test times. The following
            # reduces the work factor to the lowest possible values.
            Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface:
                algorithm: auto
                cost: 4 # Lowest possible value for bcrypt
                time_cost: 3 # Lowest possible value for argon
                memory_cost: 10 # Lowest possible value for argon

```

La partie firewall est modifiée pour indiqué quel authenticator utiliser. On pourrait en avoir plusieurs.



Comme indiqué cette commande va créer plusieurs fichiers :

* src/Security/LoginAuthenticator.php : qui va contenir la logique de votre authentification. Que faire une fois l'authetification réussie, ou en cas d'échec. Comment récupérer les informations de l'utilisateur.
* src/Controller/SecurityController.php : qui va être le contrôleur gérant la partie sécurité et authentification. Par défaut la méthode login pour afficher le formulaire et traiter (avec l'aide de l'authenticator précédent), valider les données. C'est dans ce contraôleur que vous pouvez ajouter la déconnexion, l'enregistrement et le mot de passe perdu par exemple.
* templates/security/login.html.twig : la vue contenant le formulaire de connexion, que vous pouvez librement adapter.
* Mettre à jour le fichier security.yaml afin qu'il fasse le lien avec les différents éléments de sécurité.

Les étapes suivantes expliques ce qui a été créé.

## Analyse du fichier security.yaml

Quasiment toute la sécurité se joue dans le fichier security.yaml qui fait le lien entre les différents éléments et gère les accès.

### L'encodage du mot de passe (encoders) :

Tout est pré-confiuré, vous pouvez bien sûr adapter. L'encodage permet de définir le "format" de cryptage du mot de passe. Par défaut c'est "auto", c'est à dire que selon votre configuration Symfony choisira le niveau le plus élevé possible (bcrypt ou Argon2i)

```yaml
password_hashers:
    Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface: 'auto'
```

### Partie "User Provider" (providers) :

Le Provider permet de faire le lien avec une source de données contenant les couples login/mot de passe ou des clés d'API... Les providers peuvent être des entités, des données "in\_memory", ... Cette partie est configurée a été configurée suite à la création de l'entité User, avec la méthode de connection (login), ici l'email sera utilisé. Il est possible de coupler plusieurs provider tant que l'email est unique sur l'ensemble des sources.

```yaml
providers:
        # used to reload user from session & other features (e.g. switch_user)
        app_user_provider:
            entity:
                class: App\Entity\User
                property: email
```

### L'authentification et le firewall

C'est la partie essentielle du process de sécurisation. C'est lui qui permet de dire quand il faut vérifier et authentifier un utilisateur. Le firewall permet de déterminer pour un pattern d'url (une requête (request)), la méthode d'authentification à utiliser (une page de connexion, une clé d'API, une dépendance à un fournisseur OAuth, ...).

```yaml
    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false
        main:
            lazy: true
            provider: app_user_provider
            custom_authenticator: App\Security\LoginAuthenticator
            logout:
                path: app_logout
```

L'exemple ci-dessus permet de définir que pour les routes particulières _(les assets, le profiler)_, il n'y a pas de vérification. Pour toutes les autres routes (main), il faudra utiliser le provider contenant nos User et l'authenticator gérant le formulaire de Login. C'est ici que l'on pourrait proposer plusieurs méthodes de connexion en ajoutant les authenticator adaptés.

Symfony propose des exemples pour de nombreuses méthodes d'authentification (login, ldap, json, ...) que vous [trouverez sur la documentation officielle](https://symfony.com/doc/current/security/auth\_providers.html)

### La gestion des rôles et les autorisations

La gestion des roles se fait dans la partie "access\_control" du fichier security. Il permet de définir pour chaque pattern d'URL quel rôle peut y accèder. C'est là que l'on sécurise nos différentes parties. Il est donc important de construire et structurer nos URL correctement pour être efficace sur le filtrage. Il faut évidemment veiller à ce que les pages de connexion ne soient pas derrière une page sécurisée...

Exemple:

```yaml
    access_control:
        # - { path: ^/admin, roles: ROLE_ADMIN }
        # - { path: ^/profile, roles: ROLE_USER }
```

De cette manière les URL seront automatiquement bloquées si l'utilisateur ne dispose pas du bon rôle. Il est aussi possible de tester ce rôle directement dans un contrôleur ou dans une vue selon les besoins. [Voir la documentation pour plus d'éléments sur ce point](https://symfony.com/doc/current/security.html#securing-controllers-and-other-code)

## Récupérer l'utilisateur connecté

Enfin, il est souvent nécessaire de récupérer les informations sur l'utilisateur connecté. Pour cela, dans un contrô leur il est possible d'utiliser directement l'instruction :

```php
$user = $this->getUser();
```

### Gérer la déconnexion

Sur la même idée que pour la connexion, il est possible de gérer la déconnexion. Pour cela, dans le fichier security.yaml, il faut définir le path (la route) pour la méthode qui gére la déconnexion, et la cible (une route, optionnelle), une fois la déconnexion réussie.

```yaml
firewalls:
     main:
         # ...
         logout:
             path:   app_logout
             # where to redirect after logout
             # target: app_any_route
```

La méthode dans le contrôleur peut se résumer à :

```php
    #[Route(path: '/logout', name: 'app_logout')]
    public function logout(): void
    {
        throw new \LogicException('This method can be blank - it will be intercepted by the logout key on your firewall.');
    }
```

Cette méthode qui ne retourne rien, permet la déconnexion, et la redirection se fait via le target définit dans security.yaml.

#### Générer un mot de passe avec le bon encodage

Grâce à la console, il est possible de générer un mot de passe selon l'encodage utilisé par Symfony.

```bash
bin/console security:encode-password
```

## Exercice

Mettre en place une classe User, et créer le formulaire de connexion en suivant la documentation (une commande dans le maker existe).&#x20;

Ajouter une page qui ne sera accessible qu'a des utilisateurs Admin.

Vous pouvez ajouter une fonction pour récupérer le mot de passe perdu (une commande dans le maker existe...)
