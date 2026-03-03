# Séance 10 & 11 : Séance révision

Dans cette séance nous allons reprendre l'ensemble des concepts vu cette année sur Symfony et développer un mini forum.

* Installer un nouveau projet (réutiliser symfony.mmi-troyes.fr)
* Configurer une base de données (doctrine/maker/relations)
  * Les entités sont :
    * Categorie
      * titre, string 255
      * ordre, integer
      * couleur, string 6 caractères
    * Message
      * titre, string 255
      * message, text
      * datePosted, datetime
    * Tags
      * libelle
      * couleur
    * User (utiliser make:user et ajouter)
      * Nom, string 50
      * Email, string 255
      * Ville, string 50
    * Un user peut déposer plusieurs messages,
    * Un message est dans une seule catégorie
    * Une catégorie peut avoir plusieurs messages
    * Seul un admin peut créer des catégories,
    * un message pourra avoir un ou plusieurs tags
* Mettre en place une librairie CSS (assetmapper)
* Définir une page d'accueil (controler/vue/doctrine/repository)
  * Reprenant les tags des messages et les 3 dernières messages publiés
* Définir une page listant les catégories (avec la couleur) (controler/vue/doctrine/repository)
  * Chaque catégorie permet d'accéder à une page listant les messages, triés par date
* Mettre en place la sécurité (inscription, connexion, mot de passe perdu) (controler/vue/securité)
  * Et pour ceux qui veulent aller plus loin, mise en place des "voters"
* Chaque inscription doit envoyer un mail à l'utilisateur (controler/event/service)
* Dans la page d'une catégorie il sera possible d'ajouter un message (formulaire personnalisé et mis en page) (controler/vue/form)
* Un admin pourra ajouter une catégorie depuis une page de gestion dédiée (CRUD possible, mais mis en page) (controler/vue/CRUD)
* Une page de recherche permettra de rechercher un message dans toutes les catégories (texte libre) (controler/vue/form/repository)
* On pourra filtrer par tag (controler/vue/form/repository)
* Ajouter un événement déclenché à chaque ajout d'un message et diffuser aux utilisateurs par mail (controler/event/service)



{% hint style="info" %}
En fin de séance ! L'ensemble du projet sera à déposé sur un dépôt github privé, ajouter **dannebicque** aux contributeurs. Tout code identique sera sanctionné.&#x20;
{% endhint %}

