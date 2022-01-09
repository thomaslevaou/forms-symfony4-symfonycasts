# HTML5 & "Sanity" validation

Dans notre `ArticleAdminController`, la méthode `new()` va appeler la méthode
`handleRequest()` de `form` pour faire une première validation de la 
requête. C'est notamment `handleRequest()` qui permettra de déterminer 
la valeur de `isValid()` ensuite.  

Maintenant, on va ajouter des **règles de validation** pour que le formulaire
retourne des erreurs en rouge en cas d'erreurs lors de la saisie de
l'utilisateur.  

Déjà à l'heure actuelle, si on clique sur le bouton "Submit" du formulaire
de `http://localhost:8000/admin/article/new` sans rien remplir, alors le 
formulaire indique que le champ `Title` doit être rempli. En effet Symfony 
ajoute souvent un attribut `required="required"` dans les balises qu'il 
auto-génère (bien que ce champ ne soit pas indiqué comme obligatoire dans 
`ArticleFormType`), uniquement en HTML5 (rien n'est vérifié côté serveur).  

En fait, on doit préciser nous-même dans chaque champ quels champs du 
formulaire doivent être obligatoires et lesquels ne doivent pas l'être, 
sinon Symfony met par défaut tous les champs en obligatoires. Sauf dans 
les cas comme ici où le formulaire est lié à une entité (ici `Article`), 
auquel cas Symfony va se baser sur la valeur de `nullable` pour savoir si
le champ du formulaire correspondant est obligatoire ou non
(côté HTML5 client uniquement).  
Attention ce cas de Form Type Guessing ne se produit que si le deuxième 
argument de `add()` est non null : un `->add('content')` entraînera un
Form Type Guessing, `->add('content', TextAreaType::class)` n'en
entraînera pas (et on devra préciser nous-mêmes si on veut que le champ
soit obligatoire ou pas).  

Pour avoir une validation côté serveur, le mieux à faire est d'utiliser
le **Validator** Symfony. On va l'installer via la commande suivante :
`composer require validator`.  
On va distinguer deux types de validation côté serveur:
- Les **Sanity Validations**
- Les **Business-rules Validations**

Pour tester ces règles de validation côté serveur, précisions qu'on peut 
ajouter `novalidate` dans la balise `<form>` du formulaire pour éviter
les validations côté client (et ainsi n'avoir que des validations côté
serveur).

Les Sanity Validation sont là pour vérifier qu'une valeur n'est pas absurde.  
Par exemple, si on met un identifiant complètement démesuré dans le champ
associé à l'auteur sur le formulaire, une Sanity Validation dans l'entité
permet de vérifier que l'identifiant de l'auteur est bien un identifiant 
existant en base.  

Notons qu'on peut customiser le message d'erreur avec l'option
`invalid_message` :
```PHP

            ->add('author', EntityType::class, [
                'class' => User::class,
                ...
                'invalid_message' => 'Symfony is too smart for your hacking !'
            ])
```