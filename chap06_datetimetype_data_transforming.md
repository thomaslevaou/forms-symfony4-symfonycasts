# DateTimeType & Data "Transforming"

On va ajouter à notre formulaire d'ajout d'article un champ permettant de 
préciser sa date de publication désirée.  
Si on ne met qu'un simple `->add('publishedAt')` en laissant le form type
guessing deviner le type du champ, le champ généré sur le formulaire est une
liste de champ année, jour, mois, heure etc. bien dégueulasse.  

Si on jette un oeil dans le Profiler du formulaire, on peut y voir que
c'est le `DateTimeType` qui a été deviné par défaut par le form type 
guesser. [Sa documentation](https://symfony.com/doc/current/reference/forms/types/datetime.html)
énumère toute une liste d'options possibles pour un champ de ce type.  
Par mal d'options sont spécifiques à ce type de champ: on ne pourrait en 
effet pas avoir une option "seconds" pour un champ de type `TextType`.
On peut cependant remarquer que ce champ dispose d'une option "widget"
permettant de modifier l'affichage du champ. On peut préciser cette option
`widget` par un texte simple, comme ci-dessous:
```PHP
->add('publishedAt', null, [
    'widget' => 'single_text'
])
```
Ce qui permettra de n'avoir qu'un simple champ de texte dans lequel mettre
la date, au lieu de nos plusieurs champs dégueulasses. Le fait d'avoir mis
`null` en deuxième argument est juste un raccourci pour indiquer à Symfony
qu'il peut continuer le form type guessing sur ce champ.  

Ainsi cette option a permis de rendre le champ HTML associé sur le 
formulaire en type `datetime-local` (a l'air de fonctionner sur Chrome, 
pas sur FireFox).  

Pour avoir un date picker sympa sur notre formulaire compatible sur 
plusieurs navigateurs, on doit passer par du JavaScript comme j'ai déjà
pu faire par le passé.  

En tout cas notre input `datetime-local` va renvoyer maintenant une chaîne
de caractères au lieu d'envoyer une instance de `DateTimeInterface`. Mais 
ça ne va pas poser problème, car chaque type de champ (en paramètre de 
`add`) va se charger d'afficher le champ sur l'interface, et de récupérer
la chaîne de caractères associées et de la transformer dans le type attendu
(comme `DateTimeType` ici) si besoin.  
En fait ici le DateTimeType va se
charger d'appliquer un `DateTime()` sur la chaîne de caractère retournée
par l'input HTML, et c'est ce datetime qui sera appelé en paramètre de 
`setPublishedAt()` en interne grâce au contenu de la classe
`ArticleFormType`. Et à l'inverse, quand on développera un formulaire d'édition d'article,
le formulaire appellera `getPublishedAt()` pour récupérer un `DateTime`,
et le `DateTimeType` se chargera de transformer ce `DateTime` en `string`
pour être affiché correctement sur le formulaire HTML.