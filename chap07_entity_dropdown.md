# EntityType: Drop-downs from the Database

Ca faisait longtemps que je ne m'étais pas remis dans ces tutos, alors je
rappelle qu'il y a 3 admins dans nos fixtures :
admin1@thespacebar.com, admin2@thespacebar.com, admin3@thespacebar.com
Chacun de ces admin a pour mot de passe `engage`.

Dans notre fichier `ArticleAdminController`, on a supposé que l'utilisateur
actuellement connecté est également l'auteur de l'article, comme écrit dans
la méthode `new()` :
```PHP
            $article->setAuthor($this->getUser());
```

Sauf que ce n'est pas systématiquement vrai. On va à présent faire
en sorte que l'utilisateur puisse choisir l'auteur de l'article.  

Pour ce faire, on va appliquer le **champ ChoiceType** qui permet de faire
plein de trucs en Symfony. On l'a déjà vu dans Trëmma, ce champ permet de
faire des boutons radio, des cases à cocher, des listes select déroulantes
(dropdown). Bref, le champ ChoiceType permet d'afficher tout champ de
formulaire demandant un choix à l'utilisateur. Par défaut, le champ 
retourne une liste déroulante select avec les options renseignées en 
paramètre.  

La particularité de notre situation ici est qu'on va vouloir une liste 
déroulante dont les contenus proviennent de la base de données. Pour ce 
faire, le plus simple va être d'utiliser le champ `EntityType` au lieu 
de `ChoiceType`. `EntityType` hérite de `ChoiceType`.  

On rappelle que via la méthode `configureOptions()` de la classe
`ArticleFormType`, on a un système de Form Type Guessing qui permet 
d'ajouter simplement la ligne suivante dans la méthode `build()` :
```PHP
...
            ->add('author')
...
```
Et on a directement un champ `author` qui va être enregistré dans le 
formulaire, et à partir des accesseurs en lecture et écriture, proposer
une liste déroulante à partir de données en base (voir le chap 4 si besoin
de rappels/détails sur le sujet).  
Comme dans l'Entity `Article` le champ `author` concerne tous les 
utilisateurs en base, donc tous les utilisateurs en base sont affichés 
dans la liste déroulante. Et on rappelle que dans l'Entity `User`, on a une
méthode `__toString` qui demande à afficher le prénom de l'utilisateur
quand on affiche sa donnée, d'où le fait qu'il n'y a que des prénoms dans 
la liste déroulante. Le code n'aurait pas su comment afficher un `User` 
sans cette méthode, et aurait donc planté sans celle-ci.


Si le Form Type Guessing avait échoué, on aurait pu préciser spécifiquement
qu'on avait besoin d'une liste déroulante basée sur des utilisateurs de 
la manière suivante :
```PHP
...   
    ->add('author', EntityType::class, [
                'class' => User::class,
            ])
...
```
 
Si on a besoin d'afficher un champ spécifique qui n'est pas celui retourné
par le `__toString()` de la classe, on peut appliquer un champ spécifique
en utilisant le paramètre `choice_label`:
```PHP
...   
    ->add('author', EntityType::class, [
                'class' => User::class,
                'choice_label' => 'email'
            ])
...
```

Et ceci suffit pour que la méthode `getEmail()` soit appelée pour 
l'affichage de chaque utilisateur, à la place du `getFirstName()` induit
par `__toString()`.  


On peut même créer une fonction custom qui retournera une chaîne de 
caractères à chaque appel de User, comme ci-dessous par exemple :
```PHP
           'choice_label' => function(User $user) {
                    return sprintf('(%d) %s', $user->getId(), $user->getEmail());
            }
```

Et il suffit d'ajouter le paramètre `placeholder` pour avoir un 
placeholder dans la liste :
```PHP
->add('author', EntityType::class, [
                'class' => User::class,
                'choice_label' => function(User $user) {
                    return sprintf('(%d) %s', $user->getId(), $user->getEmail());
                },
                'placeholder' => 'Choose an author'
            ])
```

Notre formulaire est à présent fonctionnel, on peut donc retirer la ligne
`$article->setAuthor($this->getUser());` citée au début.