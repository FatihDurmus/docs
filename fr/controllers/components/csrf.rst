Cross Site Request Forgery
##########################

En activant le component CSRFComponent vous bénéficiez d'une protection contre
les attaques `CSRF <http://fr.wikipedia.org/wiki/Cross-Site_Request_Forgery>`_
ou "Cross Site Request Forgery" qui est une vulnérabilité habituelle dans les
applications web. Cela permet à un attaquant de capturer et rejouer une requête
précédente, et parfois soumettre des données en utilisant des balises images ou
des ressources sur d'autres domaines.

Le CsrfComponent fonctionne en installant un cookie sur le navigateur de
l'utilisateur. Quand des formulaires sont créés à l'aide du
:php:class:`Cake\\View\\Helper\\FormHelper`, un champ caché contenant un jeton
CSRF est ajouté. Au cours de l'événement ``Controller.startup``, si la requête
est de type POST, PUT, DELETE, PATCH, le component va comparer les données de
la requête et la valeur du cookie. Si l'une des deux est manquantes ou que les
deux valeurs ne correspondent pas, le component lancera une
:php:class:`Cake\Network\Exception\ForbiddenException`.

Utiliser le CsrfComponent
=========================

En ajoutant simplement le ``CsrfComponent`` à votre tableau de components,
vous pouvez profiter de la protection CSRF fournie::

    public function initialize()
    {
        parent::initialize();
        $this->loadComponent('Csrf', [
            'secure' => true
        ]);
    }

Des réglages peuvent être transmis au composant par l'intermédiaire des
paramètres de votre composant.
Les options de configuration disponibles sont les suivants:

- ``cookieName`` Le nom du cookie à envoyer. Par défaut ``csrfToken``.
- ``expiry`` Durée avant l'expiration du jeton CSRF. Session du navigateur par
  défaut.
- ``secure`` Si le cookie doit être créé avec Secure flag ou pas.
  Par défaut à ``false``.
- ``field`` Le champ de formulaire à vérifier. Par défaut ``_csrfToken``.
  Changer cette valeur nécéssite également de configurer le FormHelper.

Lorsqu'il est activé, vous pouvez accéder au jeton CSRF actuel sur l'objet
request::

    $token = $this->request->param('_csrfToken');

Intégration avec le FormHelper
==============================

Le CsrfComponent s'intègre de façon transparente avec `` FormHelper``. Chaque
fois que vous créer un formulaire avec FormHelper, il va insérer un champ caché
contenant le jeton CSRF.

.. note::

    Lorsque vous utilisez le CsrfComponent vous devez toujours commencer vos
    formulaires avec le FormHelper. Si vous ne le faites pas, vous devrez créer
    manuellement les champs cachées dans chacun de vos formulaires.

Protection CSRF et Requêtes AJAX
================================

En plus des paramètres de données de requête, les jetons CSRF peuvent être
soumis par le biais d'un en-tête spécial ``X-CSRF-Token``. Utiliser un en-tête
rend souvent plus simple l'intégration des jetons CSRF avec de lourdes
applications Javascript, ou des API basées sur XML/JSON.

Désactiver le Component CSRF pour des Actions Spécifiques
=========================================================

Bien que non recommandé, vous pouvez désactiver le CsrfComponent pour cetaines
requêtes. Vous pouvez réalisez ceci en utilisant le dispatcheur d'événement du
controller, au cours de la méthode ``beforeFilter``::

    public function beforeFilter(Event $event)
    {
        $this->eventManager()->off($this->Csrf);
    }

.. meta::
    :title lang=fr: Csrf
    :keywords lang=fr: configurable parameters,security component,configuration parameters,invalid request,csrf,submission
