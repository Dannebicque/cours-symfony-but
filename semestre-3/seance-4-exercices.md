# Séance 4 : Exercices

## Exercice 1

Créer un nouveau projet Symfony et installer les dépendances nécessaires.

Pour rappel, en ligne de commande depuis Docker/serveur et dans le repertoire où vous souhaitez mettre votre projet :

`symfony new nomDuProjet --webapp`

{% hint style="info" %}
Comme vous utilisez docker, pensez à créer un nouvel alias dans votre fichier de conf (symfony.conf) et redémarrer apache2
{% endhint %}

En utilisant la notion d'héritage des templates créez un menu avec 3 items :

* Accueil (qui mènera vers /)
* Articles (qui mènera vers /articles)
* Recherche (qui mènera vers /recherche)

Chaque page sera associée à une méthode dans un controller différent.

En utilisant les assets installer Bootstrap.

Sur la page accueil insérez une image de votre choix

Sur la page articles, ajouter des articles dans votre méthode de controller (dans un tableau associatif, ajouter au moins 3 articles).

Affichez ces 3 articles dans votre page articles.

{% hint style="info" %}
Un article est composé d'un titre, d'un texte (on peut utiliser du loreum ipsun), d'une date de publication (on utilisera un objet datetime), et d'un auteur.&#x20;
{% endhint %}

Dans la page recherche, ajoutez simplement un formulaire de recherche comme vous l'auriez fait en BUT1.

Soignez un minimum la mise en page.



Pour la page recherche, on va simuler la récupération de la recherche utilisateur. Pour le moment, on a pas de source de donner pour effectuer la recherche.

Récupéré la recherche de l'utilisateur soumise avec un formulaire en method post ($request peut vous aider), et afficher dans une nouvelle page le mot recherché.



Vous veillerez à ce que le bon item soit "actif" dans le menu.&#x20;
