# EntityType: Custom Query

Dans notre liste déroulante d'auteurs, chaque élement de la liste a pour 
value son id en base. Ce qui veut dire qu'à partir de cet entier, le code
de Symfony s'est débrouillé pour aller récupérer un auteur dont l'id 
correspond à l'entier passé en paramètre.  
Jusqu'à maintenant ici, on n'a pas précisé le paramètre `choices` pour le 
champ `Author`, puisque la classe `EntityType` allait déjà chercher les
données en base pour nous. Mais maintenant, on va un peu customiser la
requête faite par `EntityType`, et pour ce faire une possibilité est 
d'appeler un `query_builder`. Mais ici, on va simplement surcharger le 
paramètre `choices`.  

On va déjà d'abord devoir appeler le `UserRepository` et l'insérer 
dans `ArticleFormType`. Comme `ArticleFormType` est un Service, on peut
l'appeler par injection de dépendance comme on a souvent fait jusqu'à 
présent:

```PHP
    private $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
    ...
            ->add('author', EntityType::class, [
                'class' => User::class,
                'choice_label' => function(User $user) {
                    return sprintf('(%d) %s', $user->getId(), $user->getEmail());
                },
                'placeholder' => 'Choose an author',
                'choices' => $this->userRepository
                    ->findAllEmailAlphabetical()
            ])
        ;
    }
```

Et dans la classe `UserRepository`, il suffit alors de définir la méthode
`findAllEmailAlphabetical()` :
```PHP
    public function findAllEmailAlphabetical()
    {
        return $this->createQueryBuilder('u')
            ->orderBy('u.email', 'ASC')
            ->getQuery()
            ->execute();
    }
```

Et comme ça sur notre formulaire d'ajout d'article, les e-mails des 
auteurs sont triés par ordre alphabétique croissant.  

En soi en regardant la méthode `buildForm` de `ArticleFormType`, on pourrait
penser que la classe `EntityType` n'est plus utile. Mais en fait on en 
a toujours besoin pour un dernier point qu'est la **Data Transformation**
(le fait que l'id doit être converti en un article d'une table en base).  
En général, c'est surtout sur ce dernier point que le `EntityType` est utile,
vu que toutes les autres apports de EntityType par défaut sont souvent
customisés comme on a pu le voir ici.