# Séance 11-12 : TP Révision

Dans cette séance nous allons reprendre l'ensemble des concepts vu cette année sur Symfony et développer un mini forum.

* Installer un nouveau projet (réutiliser symfony.mmi-troyes.fr)
* Configurer une base de données
  * Les entités sont :&#x20;
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
    * Un user peut déposer plusieurs messages,&#x20;
    * Un message est dans une seule catégorie
    * Une catégorie peut avoir plusieurs messages
    * Seul un admin peut créer des catégories,
    * un message pourra avoir un ou plusieurs tags
* Définir une page d'accueil
  * Reprenant les tags des messages et les 3 dernières messages publiés
* Définir une page listant les catégories (avec la couleur)
  * Chaque catégorie permet d'accéder à une page listant les messages, triés par date
* Mettre en place la sécurité
* Chaque inscription doit envoyer un mail à l'utilisateur
* Dans la page d'une catégorie il sera possible d'ajouter un message (formulaire personnalisé et mis en page)
* Un admin pourra ajouter une catégorie depuis une page de gestion dédiée (CRUD possible, mais mis en page)
* Une page de recherche permettra de rechercher un message dans toutes les catégories (texte libre)
* On pourra filtrer par tag
