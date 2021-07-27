# Form Type Class

Dans une partie précédente, on a mis un `die('todo')` dans la méthode `new()` de 
la classe `ArticleAdminController`. Il va être temps de permettre la création ou
l'édition d'un article, via un formulaire.  
Chaque formulaire Symfony dispose de sa propre classe `Form`. Ici, on va donc en
créer un, en commençant tout d'abord par créer le dossier `Form/` dans le dossier
`src/`. Dans ce dossier `Form/`, on va créer la classe `ArticleFormType`.  

Par convention, les classes de formulaire se terminent par "FormType".  
Toutes les classes de formulaire doivent hériter de la classe `AbstractType`.  

Pour que cette classe soit reconnue, on doit d'abord exécuter la commande
`composer require form`. Une fois la commande exécutée, PhpStorm peut trouver
la localisation de la classe `AbtractType`
(dans `use Symfony\Component\Form\AbstractType;`), et la suggérer.  

Une fois cela fait, on peut faire `Ctrl + O` sur la classe. On peut alors
voir que différentes méthodes de `AbstractType` peuvent être surchargées,
et celle qui nous intéresse est `buildForm()`, qu'on va surcharger ici.  
Le code complet de la classe va donc être le suivant :

```PHP
class ArticleFormType extends AbstractType
{

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('title')
            ->add('content')
        ;
    }
}
```

On peut alors changer le contenu de la méthode `new()` de
`ArticleAdminController` pour qu'il soit le suivant :
```PHP
public function new(EntityManagerInterface $em)
{
    $form = $this->createForm(ArticleFormType::class);
    return $this->render('article_admin/new.html.twig', [
       'articleForm' => $form->createView(),
    ]);
}
```

Notons le fait que dans le template Twig, on ne passe pas `$form`
directement mais `$form->createView()`, ce qui retourne un
objet bien plus sympa à afficher en Twig.  

Le code du template Twig peut alors devenir assez basique :
```HTML
{% extends 'content_base.html.twig' %}

{% block content_body %}
    <h1>Launch a new Article! </h1>

    {{ form_start(articleForm) }}
        {{ form_widget(articleForm) }}

        <button type="submit" class="btn btn-primary">Create!</button>
    {{ form_end(articleForm) }}
{% endblock %}
```

Notons que la méthode `form_start()` génère en gros un 
`<form method="POST">`, la méthode `form_end()` un 
`</form>` et d'autres trucs, tandis que `form_widget()` 
s'occupe du contenu du formulaire. Cette dernière méthode aurait
pu s'occuper du bouton submit, mais apparemment ici dans ce contexte
c'est plus simple de le coder manuellement.  

Les simples codes ci-dessus permettent d'afficher un formulaire 
à l'adresse `http://localhost:8000/admin/article/new`, qui reste pour le 
moment assez basique en style et qu'on va améliorer au fur et à mesure
du cours. 