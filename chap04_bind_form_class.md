# Bind Your Form to a Class

En fait on va voir comment on peut éviter de créer notre Article "à la main"
lors de la réception des données du formulaire.  

Pour ce faire, dans notre classe `ArticleFormType` on va surcharger la méthode 
`configureOptions()` en lui insérant le code suivant :
```PHP
public function configureOptions(OptionsResolver $resolver)
{
    $resolver->setDefaults([
       'data_class' => Article::class
    ]);
}
```

Rien qu'en faisant, ça on peut voir que le formulaire sur 
`http://localhost:8000/admin/article/new` est un textarea.  
En effet dans l'entité `Article`, l'attribut `content` est de type Doctrine `text`
(pas `string`), d'où le fait que le formulaire le reconnaisse en tant que 
`textarea`. Ceci est permis grâce au système de `form type guessing` qu'applique
`setDefaults()` en associant les données du formulaire à une classe.  

Et surtout, suite à cet ajout on peut bien vérifier par dump que les données 
entrées sont directement enregistrées dans une nouvelle instance de la classe
`Article`.  
Les reconnaissances sont faites à partir des chaînes de caractères envoyées 
dans `buildForm()` : si un élément de celui-ci s'appelle `title` alors ce sera
la méthode `setTitle()` qui sera appelée de la classe `Article` pour mettre à 
jour les données, etc.  

On peut alors supprimer les données envoyées dans `ArticleAdminController`:  
```PHP
if ($form->isSubmitted() && $form->isValid()) {
            /** @var Article $article */
            $article = $form->getData();
            $article->setAuthor($this->getUser());
            $em->persist($article);
            $em->flush();
            $this->addFlash('success', 'Article Created! Knowledge is power!');
            return $this->redirectToRoute('admin_article_list');
        }
```
Le commentaire `/** @var Article $article */` est juste là pour aider PhpStorm
à générer de la documentation automatique si besoin.  

Du coup voilà c'est pratique à utiliser pour relier rapidement des données 
entrées dans un formulaire à une classe. Mais si on doit développer un 
formulaire qui n'est pas directement lié à une classe, on peut tout à fait 
garder la manière plus manuelle employée dans les chapitres précédents.  

Bien que ce soit courant, les données n'ont aucune obligation d'être reliées 
obligatoirement à une entité. On peut mettre n'importe quelle classe PHP dans 
le `setDefaults()`. Parfois c'est même propre de créer des classes custom juste
pour certains formulaires.  

Sur les 3 méthodes utilisées dans notre template Twig de formulaire `new.html.twig`,
on peut appliquer des **form themes** pour modifier rapidement le CSS.  
Symfony dispose de form_themes en Bootstrap ou Foundation qu'on peut directement 
appliquer.  
Pour ce faire, on peut le préciser dans le fichier `config/packages/twig.yaml` de la
manière suivante (avec la clé `form_themes` à la fin):
```yaml
twig:
    paths: ['%kernel.project_dir%/templates']
    debug: '%kernel.debug%'
    strict_variables: '%kernel.debug%'
    form_themes:
        - 'bootstrap_4_layout.html.twig'
```

Et ça suffit pour avoir un formulaire Symfony sympa.

