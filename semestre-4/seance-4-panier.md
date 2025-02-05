# Séance 4 : Mise en place d'un panier

## Objectif

L'objectif de cette séance est de mettre en place un panier pour notre application. Nous allons le gérer en JavaScript et en PHP.

* Ajouter un contrôleur, un lien dans le menu et une page afin d'afficher le panier (PanierController)
* Ajouter un bouton sur la liste des jeux pour ajouter un jeu au panier. On va traiter cette action en JavaScript.
* Afficher le contenu du panier sur la page panier.

## Préparation

### Création du contrôleur

Créez un contrôleur `PanierController` avec une méthode `index` qui affiche le contenu du panier.

### Création de la vue

Créez une vue `panier/index.html.twig` qui affiche le contenu du panier.

### Ajout d'un lien dans le menu

Ajoutez un lien dans le menu pour accéder à la page du panier.

### Ajout d'un bouton pour ajouter un jeu au panier

Ajoutez un bouton sur la page de la liste des jeux pour ajouter un jeu au panier. Ce bouton doit être traité en JavaScript.

Exemple de code HTML pour le bouton :

```html
<button class="btn btn-primary ajoutJeu" data-id="{{ jeu.id }}" data-nom="{{ jeu.nom }}" data-prix="{{ jeu.prix }}">Ajouter au panier</button>
```

### Ajout d'un script JavaScript

Ajoutez un fichier `panier.js` dans le dossier `assets/` pour gérer les actions sur le panier. N'oubliez pas de l'importer dans `app.js`.

#### Détection du clic sur le bouton

```javascript
document.querySelectorAll('.ajoutJeu').forEach(function(button) {
    button.addEventListener('click', function() {
        // Récupération des données du jeu
        let id = button.getAttribute('data-id');
        let nom = button.getAttribute('data-nom');
        let prix = button.getAttribute('data-prix');

        // Ajout du jeu au panier
        ajouterJeu(id, nom, prix);
    });
});
```

#### Ajout du jeu au panier

```javascript
function ajouterJeu(id, nom, prix) {
    // Récupération du panier
    let panier = JSON.parse(localStorage.getItem('panier')) || [];

    // Vérification si le jeu est déjà dans le panier
    let jeu = panier.find(j => j.id == id);
    if (jeu) {
        jeu.quantite++;
    } else {
        panier.push({ id: id, nom: nom, prix: prix, quantite: 1 });
    }

    // Enregistrement du panier
    localStorage.setItem('panier', JSON.stringify(panier));
}
```

On utilise ici le Local Storage pour stocker les données du panier. Cela permet de garder les données même si l'utilisateur recharge la page. Ces données sont dans le navigateur de l'utilisateur et ne sont pas envoyées au serveur. Regardez l'outil de développement de votre navigateur pour voir le contenu du Local Storage (application).

### Affichage du panier

Affichez le contenu du panier sur la page `panier/index.html.twig`. Le contenu du panier est stocké dans le Local Storage et n'est donc pas accèssible directement en PHP. Il faut donc passer par JavaScript pour récupérer les données du panier.

#### La partie Twig

```twig

<div data-gb-custom-block data-tag="block">

    <h1>Panier</h1>

    <table class="table">
        <thead>
            <tr>
                <th>Nom</th>
                <th>Prix</th>
                <th>Quantité</th>
                <th>Total</th>
            </tr>
        </thead>
        <tbody id="panier">
        </tbody>
    </table>

</div>
```

#### La partie JavaScript

```javascript
// dans panier.js
document.addEventListener('DOMContentLoaded', function() {
    // Récupération du panier
    // détecter si un élément avec l'id panier existe, si oui appeler la fonction afficherPanier
    if (document.getElementById('panier')) {
        afficherPanier();
    }
});


function afficherPanier() {
    // Récupération du panier
    let panier = JSON.parse(localStorage.getItem('panier')) || [];

    // Affichage du panier
    let panierElement = document.getElementById('panier');
    panierElement.innerHTML = '';
    panier.forEach(function(jeu) {
        let tr = document.createElement('tr');
        tr.innerHTML = `
            <td>${jeu.nom}</td>
            <td>${jeu.prix} €</td>
            <td>${jeu.quantite}</td>
            <td>${jeu.prix * jeu.quantite} €</td>
        `;
        panierElement.appendChild(tr);
    });
}
```

## Pour aller plus loin

* Ajouter un bouton pour supprimer un jeu du panier.
* Ajouter un bouton pour vider le panier.
* Permettre de gérer la quantité d'un produit.
