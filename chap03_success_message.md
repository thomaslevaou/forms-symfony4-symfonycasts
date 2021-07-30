# Success (Flash) Messages

Du coup c'est cool d'avoir un cas passant pour notre formulaire, mais on a deux soucis:  
1. Le formulaire redirige vers la page d'accueil (pas la vue de liste qu'on a 
   faite à la fin du chapitre précédent);
2. Le formulaire n'a pas de message "Flash" disant que les données ont bien été
   envoyées (le formulaire n'est pas super intuitif du coup).  

Le premier point est rapidement corrigé en ajoutant le nom
"admin_article_list" à notre fonction `list()`, puis en changeant 
la route en cas de succès d'envoi du formulaire pour ce nouveau nom.

Pour le deuxième point, on peut appliquer directement la fonction `addFlash()` de
Symfony: 
```PHP
if ($form->isSubmitted() && $form->isValid()) {
            ...
            $em->persist($article);
            $em->flush();

            $this->addFlash('success', 'Article Created! Knowledge is power');

            return $this->redirectToRoute('admin_article_list');
        }
```

Grâce à cette fonction, le message est mis en variable de session. Mais à la
différence des autres variables de session, les messages Flash sont gardés en 
session jusqu'à être lus pour la première fois, avant d'être enlevés de la 
variable de session ensuite. Comme en Yii du coup.  

Afin de ne pas mettre le dawa dans l'organisation des affichages, le plus
simplie ici va être de mettre nos messages de succès dans `base.html.twig`.  
Comme ça le flash message va être affiché quelque soit la page redirigée après 
envoi du formulaire. Dans ce fichier Twig, on le met juste au dessus de l'appel
au bloc Twig body, et de la manière suivante :
```HTML
{% for message in app.flashes('success') %}
    <div class="alert alert-success">
        {{ message }}
    </div>
{% endfor %}

{% block body %}{% endblock %}
```

Ainsi, le message est affiché comme prévu après un envoi réussi des données du
formulaire. Et si on recharge la page, le message disparaît aussitôt.

On peut aussi adapter notre template Twig pour qu'il affiche ou non certains
éléments HTML/CSS s'il y a des messages Flash. Mais on ne peut pas faire ça 
en lisant juste le nombre d'éléments dans `app.flashes`, parce qu'il suffit
que cette variable soit lue une fois pour supprimer son contenu (donc on n'aurait
plus de messages flash après).  
A la place, on va faire appel à la fonction `peek` :
```HTML
<nav class="navbar navbar-expand-lg navbar-dark navbar-bg {{ app.session.flashbag.peek('success')|length > 0 ? '' : 'mb-5' }}">
```

On rappelle que `app` en Twig fait référence à la classe Symfony `AppVariable`.  
Quand on regarde dans le code de cette classe, on peut voir sa méthode `getFlashes()` qui
a été utilisée pour afficher les `flashes` en Twig.  
Dans notre cas où on a vérifié l'affichage de la marge en fonction de s'il y a des flashes 
ou non, on passe par l'attribut `session` via la méthode `getSession()` de `AppVariable`.  
En appelant `peek` de `flashbag` au lieu de son `get()` comme le fait la fonction 
`getFlashes()`, le message flash n'est pas supprimé à la lecture.
