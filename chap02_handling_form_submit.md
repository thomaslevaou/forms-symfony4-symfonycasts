# Handling the Form Submit

Maintenant qu'on a créé le visuel d'un formulaire au chapitre précédent, on va voir
comment gérer l'envoi des données de celui-ci.  
Comme on n'a pas précisé d'URL d'arrivée du formulaire, celui-ci redirige vers sa
page d'origine, c'est-à-dire `admin/article/new`. En effet, par défaut la fonction
`form_start()` dans le formulaire Twig génère un `<form>` HTML sans attribut 
`action`.  

Comme bien souvent en Web, le même contrôleur va être chargé d'afficher le
formulaire en GET, et de récupérer ses données en POST.  
Pour ce faire, le code de notre méthode `new()` dans le contrôleur va être
le suivant :
```PHP
public function new(EntityManagerInterface $em, Request $request)
{
    $form = $this->createForm(ArticleFormType::class);

    $form->handleRequest($request);
    if ($form->isSubmitted() && $form->isValid()) {
        dd($form->getData());
    }
    
    return $this->render('article_admin/new.html.twig', [
       'articleForm' => $form->createView(),
    ]);
}
```

La fonction `handleRequest()` ne va analyser les données que si elles sont 
envoyées en POST. En cas d'appel à `new()` en GET, la fonction `handleRequest()` 
ne fait rien, puis `$form->isSubmitted()` retourne `false`, et ainsi le formulaire
peut être affiché en Twig.  
Mais lors d'un appel à `new()` en POST, le `$form` récupère les données de la
requête introduisibles dans le formulaire, et `isSubmitted()` retourne `true`.  
Comme son nom l'indique `isValid()` retourne `true` si le formulaire a été 
rempli correctement, `false` sinon (dans ce dernier cas ici, la vue Twig est
de nouveau retournée, mais avec des erreurs).  


Pour le moment, lorsque ce formulaire est rempli correctement, on va insérer
"à la main" ces données dans une nouvelle instance de `Article` en base.  
Pour ce faire, il suffit de remplacer le `dd($form->getData())` par le code 
suivant:
```PHP
        if ($form->isSubmitted() && $form->isValid()) {
//            dd($form->getData());
            $data = $form->getData();
            $article = new Article();
            $article->setTitle($data['title']);
            $article->setContent($data['content']);
            $article->setAuthor($this->getUser());

            $em->persist($article);
            $em->flush();

            return $this->redirectToRoute('app_homepage');
        }
```

Ainsi, on peut insérer un article, mais on ne le voit pas encore sur la page
d'accueil car il n'a pas encore été publié.  

Pour afficher la liste de tous les articles non publiés, on va juste développer
une fonction `list()` assez simple dans `ArticleAdminController` :
```PHP
/**
 * @Route("/admin/article")
 */
public function list(ArticleRepository $articleRepo)
{
    $articles = $articleRepo->findAll();
    return $this->render('article_admin/list.html.twig', [
        'articles' => $articles
    ]);
}
```
Et la vue Twig sera juste un code Twig simple fourni par le cours.

Pour le faire fonctionner, on va juste ajouter une méthode `isPublished()` dans
notre entité `Article` : 
```PHP
public function isPublished(): bool
{
    return $this->publishedAt !== null;
}
```
On aurait pu vérifier que le publishedAt est postérieur à aujourd'hui, mais osef.  

Les articles créés sont alors visibles sur cette nouvelle page.  
Bizarrement j'ai du faire un `./bin/console cache:clear` pour que ça marche,
mais tant mieux que ça fonctionne maintenant...