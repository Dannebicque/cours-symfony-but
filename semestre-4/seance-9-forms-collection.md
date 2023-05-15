# Séance 9 : Form Collection

Dans cette partie nous allons revenir sur les formulaires, et sur un cas particulier des collections de formulaires. La documentation officielle : [https://symfony.com/doc/current/form/form\_collections.html](https://symfony.com/doc/current/form/form\_collections.html)

Nous avons vu dans une séance précédente comment intégrer un formulaire dans un autre. C'est un cas d'usage pratique, mais si nous souhaitons avoir une collection de formulaires, c'est à dire pouvoir ajouter plusieurs éléments en lien avec un autre, nous allons devoir faire un peu plus de travail.

Prenons par exemple le cas où nous aimerions ajouter des "tags" à nos articles. Nous allons donc avoir une collection de tags, et chaque tag sera un formulaire (basique, ne contenant que le libellé).

## Premières étapes

* Créer une entité `Tag` avec un champ `label` de type `string`.
* Créer une relation `ManyToMany` entre `Article` et `Tag`.
* Créer un formulaire `TagType` avec un champ `label` de type `TextType` en lien avec l'entité `Tag`.

## Configuration du formulaire

Nous allons maintenant modifier notre formulaire `ArticleType` qui va contenir une collection de tags. Pour cela, nous allons utiliser la classe `CollectionType` de Symfony. La méthode `buildForm`de notre formulaire `ArticleType` va donc ressembler à ceci :

```php
public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        //dump($options);
        $codePsotal = $options['codePostal'];
        $builder
            ...les autres champs...
            ->add('tags', CollectionType::class, [
                'entry_type' => TagType::class,
                'entry_options' => ['label' => false],
            ])
        ;
    }
```

Il faut ajouter le champs dans la vue (si vous avez détaillez l'affichage du formulaire), ou alors il est déjà pris en compte.

### Exercice

* Modifiez le formulaire `ArticleType` pour ajouter une collection de tags.
* Vérifiez que tout fonctionne dans la vue (à ce stade vous ne devriez vous que l'entrée "tags" apparaître)

## Ajout d'un tag

Nous allons maintenant ajouter un tag à notre collection. Pour cela, nous allons autoriser l'ajout dans notre formulaire en mettant `allow_add`à vraie dans le champs `tags`.

```php
 ->add('tags', CollectionType::class, [
                'entry_type' => TagType::class,
                'entry_options' => ['label' => false],
                'allow_add' => true,
                'by_reference' => false,
            ])
```

Autoriser l'ajoute, ca ajouter un attribut sur notre formulaire (nommé data-prototype) qui va contenir le code HTML du formulaire de notre tag. L'idée c'est ensuite d'ajouter du JavaScript qui va récupérer ce code et le dupliquer à chaque fois que nous allons ajouter un tag.

Pour que ca fonctionne, nous devons légérement modifier l'affichage dans notre formulaire.

{% code lineNumbers="true" %}
```twig
<ul class="tags"
    data-index="{{ form.tags|length > 0 ? form.tags|last.vars.name + 1 : 0 }}"
    data-prototype="{{ form_widget(form.tags.vars.prototype)|e('html_attr') }}"
>
    <div>
        {% raw %}
{% for tag in form.tags %}
            <li>{{ form_row(tag.libelle) }}</li>
        {% endfor %}
{% endraw %}
    </div>
</ul>
```
{% endcode %}

L'usage des balises ul est un choix pour l'exercice, vous pourriez le faire sous n'importe qu'elle forme.

* data-index : permet de récupérer le dernier index de notre collection de tags, en fonction de ce qui serait déjà saisie (sur une modification par exemple).
* data-prototype : contient le code HTML du formulaire de notre tag.

Nous devons ensuite ajouter le bouton permettant d'ajouter un tag. Par exemple le code ci-dessous.

```twig
<button type="button" class="add_item_link" data-collection-holder-class="tags">Add a tag</button>
```

Ensuite, nous avons besoin du code JavaScript qui va détecter l'appui sur le bouton et ajouter un nouveau tag.

Pour cela, et comme nous avons installé webpack-encore, nous allons ajouter notre code dans le fichier `assets/app.js`.

Tout d'abord on écoute un événement "click" sur le bouton "add a tag".

{% code lineNumbers="true" %}
```javascript
document
  .querySelectorAll('.add_item_link')
  .forEach(btn => {
      btn.addEventListener("click", addFormToCollection)
  });
```
{% endcode %}

Ensuite, on récupère le code HTML du prototype, et on l'ajoute à notre collection de tags, avec la méthode `addFormToCollection`.

{% code lineNumbers="true" %}
```javascript
const addFormToCollection = (e) => {
  const collectionHolder = document.querySelector('.' + e.currentTarget.dataset.collectionHolderClass);

  const item = document.createElement('li');

  item.innerHTML = collectionHolder
    .dataset
    .prototype
    .replace(
      /__name__/g,
      collectionHolder.dataset.index
    );

  collectionHolder.appendChild(item);

  collectionHolder.dataset.index++;
};
```
{% endcode %}

Cette méthode récupère le contenu de data-prototype, remplace `/__name__/` par l'index de notre collection (le numéro du tag que nous ajoutons), cela pour avoir des formulaires valides avec des champs ayant un nom différent à chaque fois. Ensuite, on ajoute le code HTML à la suite (`appendChild`) de notre formulaire déjà existant.

Petite particularité, le code JavaScript va s'instancier avant notre HTML, donc l'événement ne sera pas associé au bouton. Il faut donc dire au JavaScript, d'attendre que le dom soit chargé pour pouvoir ajouter l'événement, l'équivalent du `document.ready`avec jQuery.

```javascript
window.addEventListener('load', () => { // le dom est chargé
    .. votre code pour ajouter des écouteurs d'événements
})
```

### Exercice

* Ajoutez le code JavaScript pour ajouter un tag à votre formulaire.
* Testez votre formulaire et l'ajout des tags.
* Mettez en place une page permettant de voir l'ensemble des articles avec leurs tags (attention, c'est une collection (i.e. un tableau)).

{% hint style="info" %}
Lorsque vous allez soumettre le formulaire avec les tags, vous allez avoir une erreur de base de données. En effet, lorsque nous ajoutons un articles, nous essayons également d'ajouter un tag. Ce tag doit s'ajouter dans une autre entité, liée à l'article. Pour cela, nous allons devoir ajouter une méthode `persist` sur notre relation afin que Doctrine puisse ajouter le tag dans la table. Modifiez la relation de votre entité `Article` pour ajouter la méthode `persist`.

```php
    #[ORM\ManyToMany(targetEntity: Tag::class, inversedBy: 'articles', cascade: ['persist', 'remove'])]
    private Collection $tags;
```

On ajoute également remove, pour permettre la suppression d'un tag si un article est supprimé.
{% endhint %}

## Suppression d'un tag

Nous allons maintenant ajouter la possibilité de supprimer un tag. Pour cela, nous allons devoir ajouter un bouton "supprimer" à chaque tag. Tout d'abord, il faut autoriser la suppression dans notre formulaire en mettant `allow_delete`à vraie dans le champs `tags` de notre formulaire `ArticleType`.

{% code lineNumbers="true" %}
```php
 ->add('tags', CollectionType::class, [
                'entry_type' => TagType::class,
                'entry_options' => ['label' => false],
                'allow_add' => true,
                'by_reference' => false,
                'allow_delete' => true,
            ])
```
{% endcode %}

Ensuite, nous allons ajouter un bouton "supprimer" à chaque tag. Mais comme les tags n'existent pas dans notre vue, nous devons les gérer en JavaScript également, pour que l'ajout du bouton soit dynamique.

{% code lineNumbers="true" %}
```javascript
document
    .querySelectorAll('ul.tags li')
    .forEach((tag) => {
        addTagFormDeleteLink(tag)
    })

// ... the rest of the block from above

const addFormToCollection = (e) => {
    // ...

    // add a delete link to the new form
    addTagFormDeleteLink(item);
}
```
{% endcode %}

Cet extrait de code ajoute un bouton "supprimer" à chaque tag existant (en détectant le li des il, il faudrait adapter si vous affichez le formulaire autrement), et à chaque nouveau tag ajouté.

La méthode `addTagFormDeleteLink` est la suivante, et elle permet de supprimer la ligne "li" du formulaire.

{% code lineNumbers="true" %}
```javascript
const addTagFormDeleteLink = (item) => {
    const removeFormButton = document.createElement('button');
    
    item.append(removeFormButton);

    removeFormButton.addEventListener('click', (e) => {
        e.preventDefault();
        // remove the li for the tag form
        item.remove();
    });
}
```
{% endcode %}

### Exercice

* Ajoutez le code JavaScript pour ajouter un bouton "supprimer" à chaque tag.
* Testez votre formulaire et la suppression des tags.
* Ajouter une méthode permettant de modifier un article.
