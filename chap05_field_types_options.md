# Field Types & Options

On cherche à présent à préciser nous même les types des champs du
formulaire au lieu de laisser le form type guessing de Symfony le faire.  
Histoire d'avoir un peu plus de contrôle si besoin surtout.  

La méthode `add()` de notre `FormBuilderInterface` dispose de trois 
arguments: le nom du champ (comme "field"), son type et des options.  
On peut donc préciser le type dans le deuxième argument de la manière 
suivante :
```PHP
        $builder
            ->add('title', TextType::class);
```

On peut voir dans [la doc de Symfony sur les champs de formulaire
prédéfinis](https://symfony.com/doc/current/reference/forms/types.html#text-fields)
qu'il y a pas mal de type de champs prédéfinis par Symfony.
Il y en a un pour chaque input HTML5 existant. 
Chaque champ prédéfini dispose d'options configurables.  
Certaines options peuvent être appliquées à plusieurs champs. Par exemple,
on peut appliquer l'option `label` à tous les types de champs prédéfinis.  

On peut également appliquer une option `help` affichant un message d'aide
pour n'importe quel type de champ prédéfini. Pour ajouter des options, 
on va utiliser la méthode `add()` du `FormBuilderInterface` de la manière
suivante :
```PHP
$builder
            ->add('title', TextType::class, [
                'help' => 'Choose something catchy!'
            ])
```
Ce qui permet de voir notre message d'aide sur le formulaire.

Dans le Profiler, une icône en forme de formulaire papier est accessible.  
Cette icône permet de voir différents détails sur chacun des champs du 
formulaire, en cliquant sur ceux-ci. En cliquant sur un des champs du
formulaire, on peut voir ses détails dans la partie "Passed Options".  
Dans la partie "Resolved Options", on peut voir différentes infos bas
niveau qui ont permis d'afficher le formulaire et ses champs, ainsi que
toutes les options disponibles pour un champ donné (bien que la doc 
Symfony affiche plus lisiblement les options disponibles pour un champ).
