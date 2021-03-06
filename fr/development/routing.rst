Routing
#######

.. php:namespace:: Cake\Routing

.. php:class:: Router

Le Routing est une fonctionnalité qui fait correspondre les URLs aux actions du
controller. En définissant des routes, vous pouvez séparer la façon dont votre
application est intégré de la façon dont ses URLs sont structurées.

Le Routing dans CakePHP englobe aussi l'idée de routing inversé, où un tableau
de paramètres peut être transformé en une URL. En utilisant le routing
inversé, vous pouvez facilement reconstruire la structure d'URL de
votre application sans mettre à jour tous vos codes.

.. index:: routes.php

Tour Rapide
===========

Cette section va vous apprendre les utilisations les plus
habituelles du Router de CakePHP. Typiquement si vous voulez afficher quelque
chose en page d'accueil, vous ajoutez ceci au fichier **routes.php**::

    use Cake\Routing\Router;

    Router::connect('/', ['controller' => 'Articles', 'action' => 'index']);

Ceci va exécuter la méthode ``index`` dans ``ArticlesController`` quand la page
d'accueil de votre site est visitée. Parfois vous avez besoin de routes
dynamiques qui vont accepter plusieurs paramètres, ce sera par exemple le cas
d'une route pour voir le contenu d'un article::

    Router::connect('/articles/*', ['controller' => 'Articles', 'action' => 'view']);

La route ci-dessus accepte toute url qui ressemble à ``/articles/15`` et appelle
la méthode ``view(15)`` dans ``ArticlesController``. En revanche, ceci ne va pas
empêcher les visiteurs d'accéder à une URLs ressemblant à
``/articles/foobar``. Si vous le souhaitez, vous pouvez restreindre certains
paramètres grâce à une expression régulière::

    Router::connect(
        '/articles/:id',
        ['controller' => 'Articles', 'action' => 'view'],
        ['id' => '\d+', 'pass' => ['id']]
    );

Dans l'exemple précédent, le caractère jocker ``*`` est remplacé par un placeholder ``:id``.
Utiliser les placeholders nous permet de valider les parties de l'url, dans ce
cas, nous utilisons l'expression régulière ``\d+`` pour que seuls les chiffres
fonctionnent. Finalement, nous disons au Router de traiter le placeholder
``id`` comme un argument de fonction pour la fonction ``view()`` en spécifiant
l'option ``pass``. Vous pourrez en voir plus sur leur utilisation plus tard.

Le Router de CakePHP peut aussi faire correspondre les routes en reverse. Cela
signifie qu'à partir d'un tableau contenant des paramètres similaires, il est
capable de générer une chaîne URL::

    use Cake\Routing\Router;

    echo Router::url(['controller' => 'Articles', 'action' => 'view', 'id' => 15]);
    // Va afficher
    /articles/15

Les routes peuvent aussi être labellisées avec un nom unique, cela vous permet
de rapidement leur faire référence lors de la construction des liens plutôt
que de spécifier chacun des paramètres de routing::

    use Cake\Routing\Router;

    Router::connect(
        '/login',
        ['controller' => 'Users', 'action' => 'login'],
        ['_name' => 'login']
    );

    echo Router::url(['_name' => 'login']);
    // Va afficher
    /login

Pour aider à garder votre code de router "DRY", le router apporte le concept
de 'scopes'. Un scope (étendue) défini un segment de chemin commun, et
optionnellement des routes par défaut. Toute route connectée à l'intérieur d'un
scope héritera du chemin et des routes par défaut du scope qui la contient::

    Router::scope('/blog', ['plugin' => 'Blog'], function ($routes) {
        $routes->connect('/', ['controller' => 'Articles']);
    });

Le route ci-dessus matchera ``/blog/`` et renverra
``Blog\Controller\ArticlesController::index()``.

Le squelette d'application contient quelques routes pour vous aider à commencer.
Une fois que vous avez ajouté vos propres routes, vous pouvez retirer les routes
par défaut si vous n'en avez pas besoin.

.. index:: :controller, :action, :plugin
.. index:: greedy star, trailing star
.. _connecting-routes:
.. _routes-configuration:

Connecter les Routes
====================

.. php:staticmethod:: connect($route, $defaults = [], $options = [])

Pour garder votre code :term:`DRY`, vous pouvez utiliser les 'routing scopes'.
Les scopes de Routing permettent non seulement de garder votre code DRY mais
aident aussi le Router à optimiser son opération. Comme vous l'avez vu
précédemment, vous pouvez aussi utiliser ``Router::connect()`` pour connecter
les routes. Cette méthode va par défaut vers le scope ``/``. Pour créer un
scope et connecter certaines routes, nous allons utiliser la méthode
``scope()``::

    // Dans config/routes.php
    Router::scope('/', function ($routes) {
        $routes->fallbacks('InflectedRoute');
    });

La méthode ``connect()`` prend trois paramètres: l'URL que vous souhaitez
faire correspondre, les valeurs par défaut pour les éléments de votre
route, et les règles d'expression régulière pour aider le router à
faire correspondre les éléments dans l'URL.

Le format basique pour une définition de route est::

    $routes->connect(
        'URL template',
        ['default' => 'defaultValue'],
        ['option' => 'matchingRegex']
    );

Le premier paramètre est utilisé pour dire au router quelle sorte d'URL
vous essayez de contrôler. L'URL est une chaîne normale délimitée par
des slashes, mais peut aussi contenir une wildcard (\*) ou
:ref:`route-elements`. Utiliser une wildcard dit au router que vous êtes prêt
à accepter tout argument supplémentaire fourni. Les Routes sans un \* ne
matchent que le pattern template exact fourni.

Une fois que vous spécifiez une URL, vous utilisez les deux derniers paramètres
de ``connect()`` pour dire à CakePHP quoi faire avec une requête une fois
qu'elle a été matchée. Le deuxième paramètre est un tableau associatif. Les
clés du tableau devraient être appelées après les éléments de route dans l'URL,
ou les éléments par défaut: ``:controller``, ``:action``, et ``:plugin``.
Les valeurs dans le tableau sont les valeurs par défaut pour ces clés.
Regardons quelques exemples simples avant que nous commencions à voir l'utilisation
du troisième paramètre de connect()::

    $routes->connect(
        '/pages/*',
        ['controller' => 'Pages', 'action' => 'display']
    );

Cette route est trouvée dans le fichier routes.php distribué avec CakePHP.
Cette route matche toute URL commençant par ``/pages/`` et il tend vers
l'action ``display()`` de ``PagesController`` La requête ``/pages/products``
serait mappé vers ``PagesController->display('products')``.

En plus de l'étoile greedy ``/*`` il y aussi la syntaxe de l'étoile trailing
``/**``. Utiliser une étoile double trailing, va capturer le reste de l'URL
en tant qu'argument unique passé. Ceci est utile quand vous voulez utilisez
un argument qui incluait un ``/`` dedans::

    $routes->connect(
        '/pages/**',
        ['controller' => 'Pages', 'action' => 'show']
    );

L'URL entrante de ``/pages/the-example-/-and-proof`` résulterait en un argument
unique passé de ``the-example-/-and-proof``.

Vous pouvez utiliser le deuxième paramètre de ``connect()`` pour fournir tout
paramètre de routing qui est composé des valeurs par défaut de la route::

    $routes->connect(
        '/government',
        ['controller' => 'Pages', 'action' => 'display', 5]
    );

Cet exemple montre comment vous pouvez utilisez le deuxième paramètre de
``connect()`` pour définir les paramètres par défaut. Si vous construisez un
site qui propose des produits pour différentes catégories de clients, vous
pourriez considérer la création d'une route. Cela vous permet de vous lier
à ``/government`` plutôt qu'à ``/pages/display/5``.

Une autre utilisation ordinaire pour le Router est de définir un "alias" pour
un controller. Disons qu'au lieu d'accéder à notre URL régulière à
``/users/some_action/5``, nous aimerions être capable de l'accéder avec
``/cooks/some_action/5``. La route suivante s'occupe facilement de cela::

    $routes->connect(
        '/cooks/:action/*', ['controller' => 'Users']
    );

Cela dit au Router que toute URL commençant par ``/cooks/`` devrait être
envoyée au controller users. L'action appelée dépendra de la valeur du
paramètre ``:action``. En utilisant :ref:`route-elements`, vous pouvez
créer des routes variables, qui acceptent les entrées utilisateur ou les
variables. La route ci-dessus utilise aussi l'étoile greedy.
L'étoile greedy indique au :php:class:`Router` que cette route devrait
accepter tout argument de position supplémentaire donné. Ces arguments
seront rendus disponibles dans le tableau :ref:`passed-arguments`.

Quand on génère les URLs, les routes sont aussi utilisées. Utiliser
``['controller' => 'Users', 'action' => 'some_action', 5]`` en
URL va sortir /cooks/some_action/5 si la route ci-dessus est la
première correspondante trouvée.

.. _route-elements:

Les Eléments de Route
---------------------

Vous pouvez spécifier vos propres éléments de route et ce faisant
cela vous donne le pouvoir de définir des places dans l'URL où les
paramètres pour les actions du controller doivent se trouver. Quand
une requête est faite, les valeurs pour ces éléments de route se
trouvent dans ``$this->request->params`` dans le controller. Quand vous
définissez un element de route personnalisé, vous pouvez spécifier en option
une expression régulière - ceci dit à CakePHP comment savoir si l'URL est
correctement formée ou non. Si vous choisissez de ne pas fournir une expression
régulière, toute expression non ``/`` sera traitée comme une partie du
paramètre::

    $routes->connect(
        '/:controller/:id',
        ['action' => 'view'],
        ['id' => '[0-9]+']
    );

Cet exemple simple montre comment créer une manière rapide de voir les models
à partir de tout controller en élaborant une URL qui ressemble à
``/controllername/:id``. L'URL fournie à connect() spécifie deux éléments de
route: ``:controller`` et ``:id``. L'élément ``:controller`` est l'élément de
route par défaut de CakePHP, donc le router sait comment matcher et identifier
les noms de controller dans les URLs. L'élément ``:id`` est un élément de route
personnalisé, et doit être clarifié plus loin en spécifiant une expression
régulière correspondante dans le troisième paramètre de connect().

CakePHP ne produit pas automatiquement d'urls en minuscule quand vous utilisez
le paramètre ``:controller``. Si vous avez besoin de ceci, l'exemple ci-dessus
peut être réécrit en::

    $routes->connect(
        '/:controller/:id',
        ['action' => 'view'],
        ['id' => '[0-9]+', 'routeClass' => 'InflectedRoute']
    );

La classe spéciale ``InflectedRoute`` va s'assurer que les paramètres
``:controller`` et ``:plugin`` sont correctement mis en minuscule.

.. note::

    Les Patrons utilisés pour les éléments de route ne doivent pas contenir
    de groupes capturés. S'ils le font, le Router ne va pas fonctionner
    correctement.

Une fois que cette route a été définie, la requête ``/apples/5`` est la même
que celle requêtant ``/apples/view/5``. Les deux appelleraient la méthode view()
de ApplesController. A l'intérieur de la méthode view(), vous aurez besoin
d'accéder à l'ID passé à ``$this->request->params['id']``.

Si vous avez un unique controller dans votre application et que vous ne
voulez pas que le nom du controller apparaisse dans l'URL, vous pouvez mapper
toutes les URLs aux actions dans votre controller. Par exemple, pour mapper
toutes les URLs aux actions du controller ``home``, par ex avoir des URLs
comme ``/demo`` à la place de ``/home/demo``, vous pouvez faire ce qui suit::

    $routes->connect('/:action', ['controller' => 'Home']);

Si vous souhaitez fournir une URL non sensible à la casse, vous pouvez utiliser
les modificateurs en ligne d'expression régulière::

    $routes->connect(
        '/:userShortcut',
        ['controller' => 'Teachers', 'action' => 'profile', 1],
        ['userShortcut' => '(?i:principal)']
    );

Un exemple de plus, et vous serez un pro du routing::

    $routes->connect(
        '/:controller/:year/:month/:day',
        ['action' => 'index'],
        [
            'year' => '[12][0-9]{3}',
            'month' => '0[1-9]|1[012]',
            'day' => '0[1-9]|[12][0-9]|3[01]'
        ]
    );

C'est assez complexe, mais montre comme les routes peuvent vraiment
devenir puissantes. L'URL fourni a quatre éléments de route. Le premier
nous est familier: c'est une route par défaut qui dit à CakePHP d'attendre
un nom de controller.

Ensuite, nous spécifions quelques valeurs par défaut. Quel que soit le
controller, nous voulons que l'action index() soit appelée.

Finalement, nous spécifions quelques expressions régulières qui vont
matcher les années, mois et jours sous forme numérique. Notez que les
parenthèses (le groupement) ne sont pas supportées dans les expressions
régulières. Vous pouvez toujours spécifier des alternatives, comme
dessus, mais ne pas grouper avec les parenthèses.

Une fois définie, cette route va matcher ``/articles/2007/02/01``,
``/posts/2004/11/16``, gérant les requêtes
pour les actions index() de ses controllers respectifs, avec les paramètres de
date dans ``$this->request->params``.

Il y a plusieurs éléments de route qui ont une signification spéciale dans
CakePHP, et ne devraient pas être utilisés à moins que vous souhaitiez
spécifiquement utiliser leur signification.

* ``controller`` Utilisé pour nommer le controller pour une route.
* ``action`` Utilisé pour nommer l'action de controller pour une route.
* ``plugin`` Utilisé pour nommer le plugin dans lequel un controller est
  localisé.
* ``prefix`` Utilisé pour :ref:`prefix-routing`.
* ``_ext`` Utilisé pour le routing des :ref:`file-extensions`.
* ``_base`` Défini à ``false`` pour retirer le chemin de base de l'URL générée.
  Si votre application n'est pas dans le répertoire racine, cette option peut
  être utilisée pour générer les URLs qui sont 'liées à cake'.
  Les URLs liées à cake sont nécessaires pour utiliser requestAction.
* ``_scheme`` Défini pour créer les liens sur les schémas différents comme
  `webcal` ou `ftp`. Par défaut, au schéma courant.
* ``_host`` Définit l'hôte à utiliser pour le lien. Par défaut à l'hôte courant.
* ``_port`` Définit le port si vous avez besoin de créer les liens sur des ports
  non-standards.
* ``_full`` Si à ``true``, la constante `FULL_BASE_URL` va être ajoutée devant
  les URLS générées.
* ``#`` Vous permet de définir les fragments de hash d'URL.
* ``_ssl`` Défini à ``true`` pour convertir l'URL générée à https, ou ``false``
  pour forcer http.
* ``_method`` Defini la méthode HTTP à utiliser. utile si vous travaillez avec
  :ref:`resource-routes`.
* ``_name`` Nom de route. Si vous avez configuré les routes nommées, vous
  pouvez utiliser cette clé pour les spécifier.

Passer des Paramètres à l'Action
--------------------------------

Quand vous connectez les routes en utilisant
:ref:`route-elements` vous voudrez peut-être que des éléments routés
soient passés aux arguments à la place. En utilisant le 3ème argument de
:php:meth:`Router::connect()`, vous pouvez définir quels éléments de route
doivent aussi être rendus disponibles en arguments passés::

    // SomeController.php
    public function view($articleId = null, $slug = null)
    {
        // du code ici...
    }

    // routes.php
    Router::connect(
        '/blog/:id-:slug', // E.g. /blog/3-CakePHP_Rocks
        ['controller' => 'Blog', 'action' => 'view'],
        [
            // order matters since this will simply map ":id" to $articleId in your action
            'pass' => ['id', 'slug'],
            'id' => '[0-9]+'
        ]
    );

Maintenant, grâce aux possibilités de routing inversé, vous pouvez passer
dans le tableau d'URL comme ci-dessous et CakePHP sait comment former l'URL
comme définie dans les routes::

    // view.ctp
    // ceci va retourner un lien vers /blog/3-CakePHP_Rocks
    echo $this->Html->link('CakePHP Rocks', [
        'controller' => 'Blog',
        'action' => 'view',
        'id' => 3,
        'slug' => 'CakePHP_Rocks'
    ]);

    // Vous pouvez aussi utiliser des paramètres indexés numériquement.
    echo $this->Html->link('CakePHP Rocks', [
        'controller' => 'Blog',
        'action' => 'view',
        3,
        'CakePHP_Rocks'
    ]);

.. _named-routes:

Utiliser les Routes Nommées
---------------------------

Parfois vous trouvez que taper tous les paramètres de l'URL pour une route est
trop verbeux, ou bien vous souhaitez tirer avantage des améliorations de la
performance que les routes nommées permettent. Lorque vous connectez les routes,
vous pouvez spécifier une option ``_name``, cette option peut être utilisée
pour le routing inversé pour identifier la route que vous souhaitez utiliser::

    // Connecter une route avec un nom.
    $routes->connect(
        '/login',
        ['controller' => 'Users', 'action' => 'login'],
        ['_name' => 'login']
    );

    // Génère une URL en utilisant une route nommée.
    $url = Router::url(['_name' => 'login']);

    // Génère une URL en utilisant une route nommée,
    // avec certains args query string
    $url = Router::url(['_name' => 'login', 'username' => 'jimmy']);

Si votre template de route contient des elements de route comme ``:controller``,
vous aurez besoin de fournir ceux-ci comme options de ``Router::url()``.

.. index:: admin routing, prefix routing
.. _prefix-routing:

Prefix de Routage
-----------------

.. php:staticmethod:: prefix($name, $callback)

De nombreuses applications nécessitent une section d'administration dans
laquelle les utilisateurs privilégiés peuvent faire des modifications.
Ceci est souvent réalisé grâce à une URL spéciale telle que
``/admin/users/edit/5``. Dans CakePHP, les préfixes de routage peuvent être
activés depuis le fichier de configuration du cœur en configurant les
préfixes avec Routing.prefixes. Les Prefixes peuvent être soit activés en
utilisant la valeur de configuration ``Routing.prefixes``, soit en définissant
la clé ``prefix`` avec un appel de ``Router::connect()``::

    Router::prefix('admin', function ($routes) {
        // Toutes les routes ici seront préfixées avec `/admin` et auront
        // l'élément de route prefix => admin ajouté.
        $routes->fallbacks('InflectedRoute');
    });

Les préfixes sont mappés aux sous-espaces de noms dans l'espace de nom
``Controller`` de votre application. En ayant des préfixes en tant que
controller séparés, vous pouvez créer de plus petits et/ou de plus simples
controllers. Les comportements communs aux controllers préfixés et non-préfixés
peuvent être encapsulés via héritage :doc:`/controllers/components`, ou traits.
En utilisant notre exemple des utilisateurs, accéder à l'url
``/admin/users/edit/5`` devrait appeler la méthode ``edit`` de notre
``App\Controller\Admin\UsersController`` en passant 5 comme premier paramètre.
Le fichier de vue utilisé serait ``src/Template/Admin/Users/edit.ctp``.

Vous pouvez faire correspondre l'URL /admin à votre action ``index``
du controller Pages en utilisant la route suivante::

    Router::prefix('admin', function ($routes) {
        // Parce que vous êtes dans le scope admin, vous n'avez pas besoin
        // d'inclure le prefix /admin ou l'élément de route admin.
        $routes->connect('/', ['controller' => 'Pages', 'action' => 'index']);
    });

Vous pouvez aussi définir les préfixes dans les scopes de plugin::

    Router::plugin('DebugKit', function ($routes) {
        $routes->prefix('admin', function ($routes) {
            $routes->connect('/:controller');
        });
    });

Ce qui est au-dessus va créer un template de route de type
``/debug_kit/admin/:controller``. La route connectée aura les éléments de
route ``plugin`` et ``prefix`` définis.

Quand vous définissez des préfixes, vous pouvez imbriquer plusieurs préfixes
si besoin::

    Router::prefix('manager', function ($routes) {
        $routes->prefix('admin', function ($routes) {
            $routes->connect('/:controller');
        });
    });

Ce qui est au-dessus va créer un template de route de type
``/manager/admin/:controller``. La route connectée aura l'élément de
route ``prefix`` défini à ``manager/admin``.

Le préfixe actuel sera disponible à partir des méthodes du controller avec
``$this->request->params['prefix']``

Quand vous utilisez les routes préfixées, il est important de définir l'option
prefix. Voici comment construire ce lien en utilisant le helper HTML::

    // Aller vers une route préfixée.
    echo $this->Html->link(
        'Manage articles',
        ['prefix' => 'manager', 'controller' => 'Articles', 'action' => 'add']
    );

    // Enlever un prefix
    echo $this->Html->link(
        'View Post',
        ['prefix' => false, 'controller' => 'Articles', 'action' => 'view', 5]
    );

.. note::

    Vous devez connecter les routes préfixées *avant* de connecter les routes
    fallback.

.. index:: plugin routing

Routing des Plugins
-------------------

.. php:staticmethod:: plugin($name, $options = [], $callback)

Les routes des plugins sont plus faciles à créer en utilisant la méthode
``plugin()``. Cette méthode crée un nouveau scope pour les routes de plugin::

    Router::plugin('DebugKit', function ($routes) {
        // Les routes connectées ici sont préfixées par '/debug_kit' et ont
        // l'élément de route plugin défini à 'DebugKit'.
        $routes->connect('/:controller');
    });

Lors de la création des scopes de plugin, vous pouvez personnaliser le chemin de
l'élément avec l'option ``path``::

    Router::plugin('DebugKit', ['path' => '/debugger'], function ($routes) {
        // Les routes connectées ici sont préfixées par '/debugger' et ont
        // l'élément de route plugin défini à 'DebugKit'.
        $routes->connect('/:controller');
    });

Lors de l'utilisation des scopes, vous pouvez imbriquer un scope de plugin dans
un scope de prefix::

    Router::prefix('admin', function ($routes) {
        $routes->plugin('DebugKit', function ($routes) {
            $routes->connect('/:controller');
        });
    });

Le code ci-dessus devrait créer une route similaire à
``/admin/debug_kit/:controller``. Elle devrait avoir les éléments de route
``prefix`` et ``plugin`` définis.

Vous pouvez créer des liens qui pointent vers un plugin, en ajoutant la clé
``plugin`` au tableau de l'URL::

    echo $this->Html->link(
        'New todo',
        ['plugin' => 'Todo', 'controller' => 'TodoItems', 'action' => 'create']
    );

Inversement, si la requête active est une requête de plugin et que vous
souhaitez créer un lien qui n'a pas de plugin, vous pouvez faire ceci::

    echo $this->Html->link(
        'New todo',
        ['plugin' => null, 'controller' => 'Users', 'action' => 'profile']
    );

En définissant ``plugin => null``, vous dites au Router que vous souhaitez
créer un lien qui n'appartient pas à un plugin.

Routing Favorisant le SEO
-------------------------

Certains développeurs préfèrent utiliser des tirets dans les URLs, car cela
semble donner un meilleur classement dans les moteurs de recherche.
La classe ``DashedRoute`` fournit à votre application la possibilité de créer des
URLs avec des tirets pour vos plugins, controllers, et les noms d'action en
``camelCase``.

Par exemple, si nous avons un plugin ``ToDo`` avec un controller ``TodoItems``
et une action ``showItems``, la route générée sera
``/to-do/todo-items/show-items`` avec le code qui suit::

    Router::plugin('ToDo', ['path' => 'to-do'], function ($routes) {
        $routes->fallbacks('DashedRoute');
    });

.. index:: file extensions
.. _file-extensions:

Routing des Extensions de Fichier
---------------------------------

.. php:staticmethod:: parseExtensions($extensions, $merge = true)

Pour manipuler différentes extensions de fichier avec vos routes, vous avez
besoin d'une ligne supplémentaire dans votre fichier de config des routes::

    Router::parseExtensions(['html', 'rss']);

Cela activera les extensions de nom pour toutes les routes déclarées **après**
l'appel de cette méthode. Par défaut, les extensions que vous avez déclarées
seront fusionnées avec la liste des extensions existantes. Vous pouvez passer
``false`` en second argument pour remplacer la liste d'extensions déjà
existantes. Si vous appelez la méthode sans arguments, elle retournera la liste
des extensions existantes. Vous pouvez définir des extensions pour chaque
scope::

    Router::scope('/api', function ($routes) {
        $routes->extensions(['json', 'xml']);
    });

.. note::

    Le réglage des extensions devrait être la première chose que vous devriez
    faire dans un scope, car les extensions seront appliquées uniquement aux
    routes qui sont définies **après** la déclaration des extensions.

En utilisant des extensions, vous dites au router de supprimer toutes les
extensions de fichiers correspondant, puis d'analyser le reste. Si vous
souhaitez créer une URL comme ``/page/title-of-page.html`` vous devriez créer
un scope comme ceci::

    Router::scope('/api', function ($routes) {
        $routes->extensions(['json', 'xml']);
        $routes->connect(
            '/page/:title',
            ['controller' => 'Pages', 'action' => 'view'],
            [
                'pass' => ['title']
            ]
        );
    });

Ensuite, pour créer des liens, utilisez simplement::

    $this->Html->link(
        'Link title',
        ['controller' => 'Pages', 'action' => 'view', 'title' => 'super-article', '_ext' => 'html']
    );

Les extensions de fichier sont utilisées par le
:doc:`/controllers/components/request-handling` qui fait la commutation des
vues automatiquement en se basant sur les types de contenu.

.. _resource-routes:

Créer des Routes RESTful
========================

.. php:staticmethod:: mapResources($controller, $options)

Avec le router, il est facile de générer des routes RESTful pour vos
controllers. Si nous voulions permettre l'accès à une base de données REST,
nous ferions quelque chose comme ceci::

    //Dans config/routes.php

    Router::scope('/', function ($routes) {
        $routes->extensions(['json']);
        $routes->resources('recipes');
    });

La première ligne définit un certain nombre de routes par défaut pour l'accès
REST où la méthode spécifie le format du résultat souhaité (par exemple, xml,
json, rss). Ces routes sont sensibles aux méthodes de requêtes HTTP.

=========== ===================== ==============================
HTTP format URL.format            Controller action invoked
=========== ===================== ==============================
GET         /recipes.format       RecipesController::index()
----------- --------------------- ------------------------------
GET         /recipes/123.format   RecipesController::view(123)
----------- --------------------- ------------------------------
POST        /recipes.format       RecipesController::add()
----------- --------------------- ------------------------------
PUT         /recipes/123.format   RecipesController::edit(123)
----------- --------------------- ------------------------------
PATCH       /recipes/123.format   RecipesController::edit(123)
----------- --------------------- ------------------------------
DELETE      /recipes/123.format   RecipesController::delete(123)
=========== ===================== ==============================

La classe Router de CakePHP utilise un nombre différent d'indicateurs pour
détecter la méthode HTTP utilisée. Voici la liste dans l'ordre de préférence:

#. La variable POST \_method
#. Le X\_HTTP\_METHOD\_OVERRIDE
#. Le header REQUEST\_METHOD

La variable POST \_method est utile dans l'utilisation d'un navigateur comme un
client REST (ou tout ce qui peut faire facilement du POST). Il suffit de
configurer la valeur de \_method avec le nom de la méthode de requête HTTP que
vous souhaitez émuler.

Créer des Ressources Imbriquées
-------------------------------

Une fois que vous avez connecté une ressource dans un scope, vous pouvez aussi
connecter des routes pour des sous-ressources. Les routes de sous-ressources
seront préfixées par le nom de la ressource originale et par son paramètre id.
Par exemple::

    Router::scope('/api', function ($routes) {
        $routes->resources('Articles', function ($routes) {
            $routes->resources('Comments');
        });
    });

Le code ci-dessus va générer une ressource de routes pour ``articles`` et
``comments``. Les routes des ``comments`` vont ressembler à ceci::

    /api/articles/:article_id/comments
    /api/articles/:article_id/comments/:id

.. note::

    Vous pouvez imbriquer autant de ressources que vous le souhaitez, mais il
    n'est pas recommandé d'imbriquer plus de 2 ressources ensembles.

Limiter la Création des Routes
------------------------------

Par défaut, CakePHP va connecter 6 routes pour chaque ressource. Si vous
souhaitez connecter uniquement des routes spécifiques à une ressource, vous
pouvez utilisez l'option ``only``::

    $routes->resources('Articles', [
        'only' => ['index', 'view']
    ]);

Le code ci-dessus devrait créer uniquement les routes de ressource ``lecture``.
Les noms de route sont ``create``, ``update``, ``view``, ``index`` et ``delete``.

Changer les Actions du Controller
---------------------------------

Vous devrez peut-être modifier le nom des actions du controller qui sont
utilisés lors de la connexion des routes. Par exemple, si votre action ``edit``
est nommée ``update``, vous pouvez utiliser la clé ``actions`` pour renommer
vos actions::

    $routes->resources('Articles', [
        'actions' => ['update' => 'update', 'add' => 'create']
    ]);

Le code ci-dessus va utiliser ``update`` pour l'action update, et ``create`` au
lieu de ``add``.

.. _custom-rest-routing:

Classes de Route Personnalisée pour les Ressources
--------------------------------------------------

Vous pouvez spécifier la clé ``connectOptions`` dans le tableau ``$options`` de
la fonction ``resources()`` pour fournir une configuration personnalisée
utilisée par ``connect()``::

    Router::scope('/', function ($routes) {
        $routes->resources('books', [
            'connectOptions' => [
                'routeClass' => 'ApiRoute',
            ]
        ];
    });

.. index:: passed arguments
.. _passed-arguments:

Arguments Passés
================

Les arguments passés sont des arguments supplémentaires ou des segments
du chemin qui sont utilisés lors d'une requête. Ils sont souvent utilisés
pour transmettre des paramètres aux méthodes de vos controllers. ::

    http://localhost/calendars/view/recent/mark

Dans l'exemple ci-dessus, ``recent`` et ``mark`` sont tous deux des arguments
passés à ``CalendarsController::view()``. Les arguments passés sont transmis aux
contrôleurs de trois manières. D'abord comme arguments de la méthode de
l'action appelée, deuxièmement en étant accessibles dans
``$this->request->params['pass']`` sous la forme d'un tableau indexé
numériquement. Enfin, il y a ``$this->passedArgs`` disponible de la même
façon que par ``$this->request->params['pass']``. Lorsque vous utilisez des
routes personnalisées il est possible de forcer des paramètres particuliers
comme étant des paramètres passés également.

Si vous alliez visiter l'URL mentionné précédemment, et que vous aviez une
action de controller qui ressemblait à cela::

    class CalendarsController extends AppController
    {
        public function view($arg1, $arg2)
        {
            debug(func_get_args());
        }
    }

Vous auriez le résultat suivant::

    Array
    (
        [0] => recent
        [1] => mark
    )

La même donnée est aussi disponible dans ``$this->request->params['pass']``
et dans ``$this->passedArgs`` dans vos controllers, vues, et helpers.
Les valeurs dans le tableau pass sont indicées numériquement basé sur l'ordre
dans lequel elles apparaissent dans l'URL appelé::

    debug($this->request->params['pass']);
    debug($this->passedArgs);

Le résultat des 2 debug() du dessus serait::

    Array
    (
        [0] => recent
        [1] => mark
    )

Quand vous générez des URLs, en utilisant un :term:`tableau de routing`, vous
ajoutez des arguments passés en valeurs sans clés de type chaîne dans le
tableau::

    ['controller' => 'Articles', 'action' => 'view', 5]

Comme ``5`` a une clé numérique, il est traité comme un argument passé.

Générer des URLs
================

.. php:staticmethod:: url($url = null, $full = false)

Le routing inversé est une fonctionnalité dans CakePHP qui est utilisée pour
vous permettre de changer facilement votre structure d'URL sans avoir à
modifier tout votre code. En utilisant des
:term:`tableaux de routing <tableau de routing>` pour définir vos URLs, vous
pouvez configurer les routes plus tard et les URLs générés vont automatiquement
être mises à jour.

Si vous créez des URLs en utilisant des chaînes de caractères comme::

    $this->Html->link('View', '/articles/view/' + $id);

Et ensuite plus tard, vous décidez que ``/posts`` devrait vraiment être
appelé 'articles' à la place, vous devrez aller dans toute votre application
en renommant les URLs. Cependant, si vous définissiez votre lien comme::

    $this->Html->link(
        'View',
        ['controller' => 'Articles', 'action' => 'view', $id]
    );

Ensuite quand vous décidez de changer vos URLs, vous pouvez le faire en
définissant une route. Cela changerait à la fois le mapping d'URL entrant,
ainsi que les URLs générés.

Quand vous utilisez les URLs en tableau, vous pouvez définir les paramètres
chaîne de la requête et les fragments de document en utilisant les clés
spéciales::

    Router::url([
        'controller' => 'Articles',
        'action' => 'index',
        '?' => ['page' => 1],
        '#' => 'top'
    ]);

    // Cela générera une URL comme:
    /articles/index?page=1#top

Router will also convert any unknown parameters in a routing array to
querystring parameters.  The ``?`` is offered for backwards compatibility with
older versions of CakePHP.

You can also use any of the special route elements when generating URLs:

* ``_ext`` Used for :ref:`file-extensions` routing.
* ``_base`` Set to ``false`` to remove the base path from the generated URL. If
  your application is not in the root directory, this can be used to generate
  URLs that are 'cake relative'. cake relative URLs are required when using
  requestAction.
* ``_scheme``  Set to create links on different schemes like `webcal` or `ftp`.
  Defaults to the current scheme.
* ``_host`` Set the host to use for the link.  Defaults to the current host.
* ``_port`` Set the port if you need to create links on non-standard ports.
* ``_full``  If ``true`` the `FULL_BASE_URL` constant will be prepended to
  generated URLs.
* ``_ssl`` Set to ``true`` to convert the generated URL to https, or ``false``
  to force http.
* ``_name`` Name of route. If you have setup named routes, you can use this key
  to specify it.

.. _redirect-routing:

Redirect Routing
================

.. php:staticmethod:: redirect($route, $url, $options = [])

Redirect routing allows you to issue HTTP status 30x redirects for
incoming routes, and point them at different URLs. This is useful
when you want to inform client applications that a resource has moved
and you don't want to expose two URLs for the same content

Redirection routes are different from normal routes as they perform an actual
header redirection if a match is found. The redirection can occur to
a destination within your application or an outside location::

    $routes->redirect(
        '/home/*',
        ['controller' => 'Articles', 'action' => 'view'],
        ['persist' => true]
        // or ['persist'=>['id']] for default routing where the
        // view action expects $id as an argument
    );

Redirects ``/home/*`` to ``/articles/view`` and passes the parameters to
``/articles/view``. Using an array as the redirect destination allows
you to use other routes to define where a URL string should be
redirected to. You can redirect to external locations using
string URLs as the destination::

    $routes->redirect('/articles/*', 'http://google.com', ['status' => 302]);

This would redirect ``/articles/*`` to ``http://google.com`` with a
HTTP status of 302.

.. _custom-route-classes:

Custom Route Classes
====================

Custom route classes allow you to extend and change how individual routes parse
requests and handle reverse routing. Route classes have a few conventions:

* Route classes are expected to be found in the ``Routing\\Route`` namespace of
  your application or plugin.
* Route classes should extend :php:class:`Cake\\Routing\\Route`.
* Route classes should implement one or both of ``match()`` and/or ``parse()``.

The ``parse()`` method is used to parse an incoming URL. It should generate an
array of request parameters that can be resolved into a controller & action.
Return ``false`` from this method to indicate a match failure.

The ``match()`` method is used to match an array of URL parameters and create a
string URL. If the URL parameters do not match the route ``false`` should be
returned.

You can use a custom route class when making a route by using the ``routeClass``
option::

    Router::connect(
         '/:slug',
         ['controller' => 'Articles', 'action' => 'view'],
         ['routeClass' => 'SlugRoute']
    );

This route would create an instance of ``SlugRoute`` and allow you
to implement custom parameter handling. You can use plugin route classes using
standard :term:`syntaxe de plugin`.

Classe de Route par Défaut
--------------------------

.. php:staticmethod:: defaultRouteClass($routeClass = null)

Si vous voulez utiliser une autre classe de route pour toutes vos routes
en plus de la ``Route`` par défaut, vous pouvez faire ceci en appelant
``Router::defaultRouteClass()`` avant de définir la moindre route et éviter
de spécifier l'option ``routeClass`` pour chaque route. Par exemple en
utilisant::

    Router::defaultRouteClass('DashedRoute');

Cela provoquera l'utilisation de la classe ``DashedRoute`` pour toutes les
routes suivantes.
Appeler la méthode sans argument va retourner la classe de route courante par
défaut.

Fallbacks method
----------------

.. php:method:: fallbacks($routeClass = null)

The fallbacks method is a simple shortcut for defining default routes. The
method uses the passed routing class for the defined rules or if no class is
provided the class returned by ``Router::defaultRouteClass()`` is used.

Calling fallbacks like so::

    $routes->fallbacks('InflectedRoute');

Is equivalent to the following explicit calls::

    $routes->connect('/:controller', ['action' => 'index'], ['routeClass' => 'InflectedRoute']);
    $routes->connect('/:controller/:action/*', [], , ['routeClass' => 'InflectedRoute']);

.. note::

    Using the default route class (``Route``) with fallbacks, or any route
    with ``:plugin`` and/or ``:controller`` route elements will result in
    inconsistent URL case.

Handling Named Parameters in URLs
=================================

Although named parameters were removed in CakePHP 3.0, applications may have
published URLs containing them.  You can continue to accept URLs containing
named parameters.

In your controller's ``beforeFilter()`` method you can call
``parseNamedParams()`` to extract any named parameters from the passed
arguments::

    public function beforeFilter()
    {
        parent::beforeFilter();
        Router::parseNamedParams($this->request);
    }

This will populate ``$this->request->params['named']`` with any named parameters
found in the passed arguments.  Any passed argument that was interpreted as a
named parameter, will be removed from the list of passed arguments.


RequestActionTrait
==================

.. php:trait:: RequestActionTrait

    This trait allows classes which include it to create sub-requests or
    request actions.

.. php:method:: requestAction(string $url, array $options)

    This function calls a controller's action from any location and
    returns data from the action. The ``$url`` passed is a
    CakePHP-relative URL (/controllername/actionname/params). To pass
    extra data to the receiving controller action add to the $options
    array.

    .. note::

        You can use ``requestAction()`` to retrieve a fully rendered view
        by passing 'return' in the options:
        ``requestAction($url, ['return']);``. It is important to note
        that making a requestAction using 'return' from a controller method
        can cause script and css tags to not work correctly.

    .. warning::

        If used without caching ``requestAction`` can lead to poor
        performance. It is seldom appropriate to use in a controller.

    ``requestAction`` is best used in conjunction with (cached)
    elements – as a way to fetch data for an element before rendering.
    Let's use the example of putting a "latest comments" element in the
    layout. First we need to create a controller function that will
    return the data::

        // Controller/CommentsController.php
        class CommentsController extends AppController
        {
            public function latest()
            {
                if (!$this->request->is('requested')) {
                    throw new ForbiddenException();
                }
                return $this->Comments->find('all', [
                    'order' => 'Comment.created DESC',
                    'limit' => 10
               ]);
            }
        }

    You should always include checks to make sure your requestAction methods are
    actually originating from ``requestAction``.  Failing to do so will allow
    requestAction methods to be directly accessible from a URL, which is
    generally undesirable.

    If we now create a simple element to call that function::

        // View/Element/latest_comments.ctp

        $comments = $this->requestAction('/comments/latest');
        foreach ($comments as $comment) {
            echo $comment->title;
        }

    We can then place that element anywhere to get the output
    using::

        echo $this->element('latest_comments');

    Written in this way, whenever the element is rendered, a request
    will be made to the controller to get the data, the data will be
    processed, and returned. However in accordance with the warning
    above it's best to make use of element caching to prevent needless
    processing. By modifying the call to element to look like this::

        echo $this->element('latest_comments', [], ['cache' => '+1 hour']);

    The ``requestAction`` call will not be made while the cached
    element view file exists and is valid.

    In addition, requestAction now takes array based cake style URLs::

        echo $this->requestAction(
            ['controller' => 'Articles', 'action' => 'featured'],
            ['return']
        );

    The URL based array are the same as the ones that
    :php:meth:`HtmlHelper::link()` uses with one difference - if you are using
    passed parameters, you must put them in a second array and wrap them with
    the correct key. This is because requestAction merges the extra parameters
    (requestAction's 2nd parameter) with the ``request->params`` member array
    and does not explicitly place them under the ``pass`` key. Any additional
    keys in the ``$option`` array will be made available in the requested
    action's ``request->params`` property::

        echo $this->requestAction('/articles/view/5');

    As an array in the requestAction would then be::

        echo $this->requestAction(
            ['controller' => 'Articles', 'action' => 'view', 5],
        );

    You can also pass querystring arguments, post data or cookies using the
    appropriate keys. Cookies can be passed using the ``cookies`` key.
    Get parameters can be set with ``query`` and post data can be sent
    using the ``post`` key::

        $vars = $this->requestAction('/articles/popular', [
          'query' => ['page' = > 1],
          'cookies' => ['remember_me' => 1],
        ]);

    .. note::

        Unlike other places where array URLs are analogous to string URLs,
        requestAction treats them differently.

    When using an array URL in conjunction with requestAction() you
    must specify **all** parameters that you will need in the requested
    action. This includes parameters like ``$this->request->data``.  In addition
    to passing all required parameters, passed arguments must be done
    in the second array as seen above.

.. toctree::
    :glob:
    :maxdepth: 1

    /development/dispatch-filters

.. meta::
    :title lang=fr: Routing
    :keywords lang=fr: controller actions,default routes,mod rewrite,code index,string url,php class,incoming requests,dispatcher,url url,meth,maps,match,parameters,array,config,cakephp,apache,routeur,router
