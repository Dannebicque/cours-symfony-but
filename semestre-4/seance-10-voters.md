# Séance 12 : Sécurité et voters

### Objectif du cours

* quand utiliser un **Voter** plutôt qu’un `ROLE_*` en dur
* comment **créer** un voter
* comment l’utiliser :
  * dans un contrôleur (`denyAccessUnlessGranted`)
  * dans Twig (`is_granted`)
* comment débugger rapidement (le “pourquoi c’est refusé ?”)

***

### Rappel : pourquoi un Voter ?

Les rôles (`ROLE_ADMIN`, `ROLE_USER`) répondent à : **“qui es-tu ?”**\
Un voter répond à : **“as-tu le droit de faire X sur CET objet précis ?”**

Exemples :

* “un message est éditable **seulement** par son auteur”
* “un modérateur peut supprimer n’importe quel message”
* “un message verrouillé ne peut plus être modifié (même par l’auteur)”

👉 Donc dès qu’il y a **un objet (Message, Post, Document…) + une règle métier**, Voter.

***

### Modèle d’exemple : Message de forum

On imagine une entité `Message` :

* `content`
* `author` (User)
* `createdAt`
* `locked` (bool) : message verrouillé par un modo

Règles métiers (exemple simple) :

* **VIEW** : tout le monde connecté peut voir
* **EDIT** :
  * auteur peut éditer **si pas locked**
  * modérateur / admin peut éditer même si locked
* **DELETE** :
  * auteur peut supprimer **dans les 15 minutes** après création et si pas locked
  * modérateur / admin peut supprimer à tout moment

***

### Étape 1 — Créer le Voter

Créer `src/Security/Voter/MessageVoter.php`

```php
<?php

namespace App\Security\Voter;

use App\Entity\Message;
use App\Entity\User;
use Symfony\Bundle\SecurityBundle\Security;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authorization\Voter\Voter;

class MessageVoter extends Voter
{
    public const VIEW = 'MESSAGE_VIEW';
    public const EDIT = 'MESSAGE_EDIT';
    public const DELETE = 'MESSAGE_DELETE';

    public function __construct(
        private readonly Security $security,
    ) {}

    protected function supports(string $attribute, mixed $subject): bool
    {
        // 1) le voter ne gère QUE ces actions
        if (!in_array($attribute, [self::VIEW, self::EDIT, self::DELETE], true)) {
            return false;
        }

        // 2) le voter ne s’applique QUE sur Message
        return $subject instanceof Message;
    }

    /**
     * @param Message $subject
     */
    protected function voteOnAttribute(string $attribute, mixed $subject, TokenInterface $token): bool
    {
        $user = $token->getUser();

        // pas connecté => pas de droits (dans notre exemple)
        if (!$user instanceof User) {
            return false;
        }

        // règle "super pouvoirs"
        if ($this->security->isGranted('ROLE_ADMIN') || $this->security->isGranted('ROLE_MODERATOR')) {
            return true;
        }

        return match ($attribute) {
            self::VIEW => $this->canView($subject, $user),
            self::EDIT => $this->canEdit($subject, $user),
            self::DELETE => $this->canDelete($subject, $user),
            default => false,
        };
    }

    private function canView(Message $message, User $user): bool
    {
        // ici : tout utilisateur connecté peut lire
        return true;
    }

    private function canEdit(Message $message, User $user): bool
    {
        // auteur seulement + pas verrouillé
        return $message->getAuthor() === $user && !$message->isLocked();
    }

    private function canDelete(Message $message, User $user): bool
    {
        // auteur, pas locked, et suppression limitée dans le temps (15 min)
        if ($message->getAuthor() !== $user) {
            return false;
        }

        if ($message->isLocked()) {
            return false;
        }

        $limit = (clone $message->getCreatedAt())->modify('+15 minutes');
        return new \DateTimeImmutable() <= \DateTimeImmutable::createFromMutable($limit);
    }
}
```

#### Points importants

* `supports()` : “est-ce que je suis concerné ?”
* `voteOnAttribute()` : “j’applique mes règles”
* `Security $security` permet de réutiliser `isGranted()` pour un rôle “global”.

***

### Étape 2 — Utiliser le voter dans un contrôleur

Exemple d’édition :

```
#[Route('/message/{id}/edit', name: 'message_edit')]
public function edit(Message $message, Request $request): Response
{
    $this->denyAccessUnlessGranted('MESSAGE_EDIT', $message);

    // ... formulaire, save, etc.
}
```

Exemple suppression :

```
#[Route('/message/{id}/delete', name: 'message_delete', methods: ['POST'])]
public function delete(Message $message, Request $request): Response
{
    $this->denyAccessUnlessGranted('MESSAGE_DELETE', $message);

    // ... CSRF + suppression
}
```

***

### Étape 3 — Utiliser dans Twig

Afficher les boutons selon les droits :

```
{% if is_granted('MESSAGE_EDIT', message) %}
  <a href="{{ path('message_edit', {id: message.id}) }}">Éditer</a>
{% endif %}

{% if is_granted('MESSAGE_DELETE', message) %}
  <form method="post" action="{{ path('message_delete', {id: message.id}) }}">
    <button type="submit">Supprimer</button>
  </form>
{% endif %}
```

***

### Étape 4 — Bonnes pratiques

* **Constantes** pour les attributs (`MESSAGE_EDIT`) : évite les fautes de frappe.
* Ne pas mettre “trop” de règles dans le contrôleur : le contrôleur appelle `denyAccessUnlessGranted`.
* Le voter doit rester **lisible** : extraire des méthodes (`canEdit`, `canDelete`).
* Si les règles deviennent grosses : créer un petit service métier `MessagePermissionChecker` et l’utiliser dans le voter.

***

### Débogage rapide

Si “ça marche pas” :

1. Vérifier que `supports()` renvoie bien `true` (bon attribut + bon objet)
2. Vérifier que le `$subject` est bien une entité `Message` (pas `null`, pas un id)
3. Vérifier que l’utilisateur est bien connecté (`$user instanceof User`)

Commande utile :

```
php bin/console debug:container --tag=security.voter
```

***
