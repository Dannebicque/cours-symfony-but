# Séances 10 & 11 : Séances révision

Dans cette séance nous allons reprendre l'ensemble des concepts vu cette année sur Symfony et développer un mini forum.

* Installer un nouveau projet (réutiliser symfony.mmi-troyes.fr)
* Configurer une base de données _(doctrine/maker/relations)_
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
    * User _(utiliser make:user et ajouter)_
      * Nom, string 50
      * Email, string 255
      * Ville, string 50
    * Un user peut déposer plusieurs messages,
    * Un message est dans une seule catégorie
    * Une catégorie peut avoir plusieurs messages
    * Seul un admin peut créer des catégories,
    * un message pourra avoir un ou plusieurs tags
* Mettre en place une librairie CSS _(assetmapper)_
* Le site sera disponible en deux langues _(translations)_
* Définir une page d'accueil _(controler/vue/doctrine/repository)_
  * Reprenant les tags des messages et les 3 dernières messages publiés
* Définir une page listant les catégories (avec la couleur) _(controler/vue/doctrine/repository)_
  * Chaque catégorie permet d'accéder à une page listant les messages, triés par date
* Mettre en place la sécurité (inscription, connexion, mot de passe perdu) _(controler/vue/securité)_
  * Et pour ceux qui veulent aller plus loin, mise en place des "voters"
* Chaque inscription doit envoyer un mail à l'utilisateur _(controler/event/service)_
* Dans la page d'une catégorie il sera possible d'ajouter un message (formulaire personnalisé et mis en page) _(controler/vue/form)_
* Un admin pourra ajouter une catégorie depuis une page de gestion dédiée, personnalisé et mis en page _(controler/vue/CRUD)_
* Une page de recherche permettra de rechercher un message dans toutes les catégories (texte libre) _(controler/vue/form/repository)_
* On pourra filtrer par tag _(controler/vue/form/repository)_
* Ajouter un événement déclenché à chaque ajout d'un message et diffuser aux utilisateurs par mail _(controler/event/service)_



{% hint style="info" %}
En fin de séance ! L'ensemble du projet sera à déposé sur un dépôt github privé, ajouter **dannebicque** aux contributeurs. Tout code identique sera sanctionné.&#x20;
{% endhint %}

