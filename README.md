# MEAN.IO (2/3)

Ce second article d'une série consacrée à MEAN.IO va nous permettre d'introduire les principaux concepts permettant de créer un nouveau module afin d'étendre le canevas. Le cas d'application que je vous propose d'étudier permettra également à la plupart d'entre vous de découvrir un autre domaine, celui de la visualisation de données géographiques sous forme de carte ou sous forme 3D. Nous souhaitons en effet créer une application permettant de visualiser un ou plusieurs itinéraires GPS (type randonnée VTT ou pédestre).

## Création d'un module

Tout d'abord un petit rappel, l'ensemble des modules d'une application MEAN.IO est stocké dans le dossier **packages**. Le premier sous-dossier **core** inclut les modules de base livrés avec le framework et le sous-dossier **custom** les modules spécifiques à la logique applicative, c'est donc ici qu'il faudra créer vos nouveaux modules. Grâce à l'outil en ligne de commande [mean-cli](https://www.npmjs.com/package/mean-cli) il est possible d'initialiser un module (scaffolding) via :
```
mean package package_name
```
Il est à tout moment possible de lister vos modules et d'effacer un module en particulier :
```
// Liste les modules présents
mean list
// Efface un module
mean uninstall package_name
```

Le dossier d'un module MEAN.IO présente la structure suivante (un module nouvellement créé peut ne pas contenir tous les dossiers néanmoins) :
```
Module folder
--- docs : contient la documentation de l'API (back-end)
--- public : partie publique du module sur le site
    --- assets : données statiques (images, css, dépendances front-end)
    --- controllers : controllers front-end (AngularJS)
    --- directives : directives front-end (AngularJS)
    --- routes : routing front-end (AngularUI Router)
    --- services : services front-end (AngularJS)
    --- tests : tests front-end (Jasmine)
    --- views : vues (AngularJS)
--- server : contient le code back-end
    --- controllers : controllers back-end (Express)
    --- routes : routing back-end (Express)
    --- models : modèle de données back-end (Mongoose)
    --- tests : tests back-end (Jasmine)
    --- views : vues HMTL (templating Swig)
--- node_modules : contient les dépendances Node.js du module (back-end)
```
> **Trucs & Astuces** : Tous les fichiers à l'intérieur du dossier **public** seront accessibles publiquement à l'URL */nom-module/chemin-relatif-fichier*. Par exemple pour accéder à une image nommé 'logo.png' dans le module nommé 'module' l'URL sera *module/assets/img/logo.png*.

A la racine on trouvera comme dans le cas de l'application les fichiers de configuration pour npm, bower et MEAN.IO (**mean.json**). Le plus important est le fichier **app.js** qui est le point d'entrée du module, à l'intérieur est réalisé l'enregistrement du module dans MEAN.IO. Créons un nouveau module nommé 'application' qui contiendra notre logique applicative, dans le fichier **app.js** généré par défaut nous trouverons ceci :
```javascript
var Module = require('meanio').Module;
// Création du module, ceci va automatiquement charger tous les modèles du sous-dossier models
var Application = new Module('application');
// Enregistrement du module
Application.register(function(app, auth, database) {
  // Déclaration automatique des routes du sous-dossier routes
  Application.routes(app, auth, database);
  // Aggrégation des dépendances front-end (voir article précédent)
  Application.aggregateAsset('css', 'application.css');

  return Application;
});
```
Si votre nouveau module dépend d'autres modules MEAN.IO ou AngularJS il vous faudra rajouter ceci lors du register :
```javascript
Application.angularDependencies(['mean.users']);
```
En effet l'arbre des dépendances est construit côté serveur au lancement de l'application et est récupéré au chargement de la page via une requête sur l'URL */_getModules* côté client. Ceci permet de déclarer alors dynamiquement la liste de tous les modules AngularJS dans le code JS (pour les curieux vois le fichier **bower_components/web-bootstrap/index.js**).

Par défaut MEAN.IO injecte trois dépendances en paramètre du register :

  - **app** : l'application Express,
  - **auth** : un module proposant des fonctions d'authenticatio basiques,
  - **database** : la connexion Mongoose à la base de données.
 
Si votre module dépend d'autres modules (e.g. module1 et module2) ils pourront être injectés à la suite :
```javascript
...
// Enregistrement du module
Application.register(function(app, auth, database, module1, module2) {
  ...
});
```

> **Trucs & Astuces** : par défaut, à l'intérieur de chaque module de base de MEAN.IO, les fichiers portent le même nom (celui du module comme par exemple *Module.js*). Vous verrez que dans la suite je préfère suffixer chaque fichier par le type d'objet qu'il contient (par exemple *ModuleController.js*, *ModuleRoutes.js*, etc.). En effet, même si le nom du dossier parent peut servir de discriminant, il est ainsi plus aisé de savoir à quel fichier l'on a affaire. Notamment lorsqu'ils sont ouverts simultanément sous forme d'onglets ne laissant apparaitre que le nom du fichier (et non le chemin complet) dans votre éditeur de texte favori. 

## Partie serveur (back-end)

### Création du modèle

Le premier travail lors de la création d'un module consiste à définir le modèle conceptuel du ou des objets qui seront manipulés par le module, ce modèle est ensuite exprimé au moyen d'un schéma Mongoose. Dans notre cas l'objet manipulé sera un itinéraire GPS ('track' en anglais). Un tel chemin est simplement décrit par une liste de positions GPS acquises par le capteur. Chaque point est repéré en coordonnées géographiques : longitude, latitude et altitue. A part l'utilisateur qui l'a créé et un descriptif le chemin contiendra donc un tableau de coordonnées. Pour déclarer le modèle il suffit de créer un fichier **TrackModel.js** dans le dossier **models** du module, il contiendra le schéma suivant :
```javascript
'use strict';

var mongoose = require('mongoose'),
    Schema = mongoose.Schema;

// Déclaration du schéma du modèle 'Track'
var TrackSchema = new mongoose.Schema({
  // Utilisateur ayant créé le chemin
  user : {
  	type: mongoose.Schema.Types.ObjectId,
  	ref: 'User' // Référence le modèle d'utilisateur de MEAN.IO
  },
  // Date de création
  created: {
    type: Date,
    default: Date.now
  },
  // Titre du chemin
  title: {
    type: String,
    required: true,
    trim: true
  },
  // Description associée au chemin
  description : {
  	type : String, required : true
  },
  // Liste des points de passage constituant le chemin (coordonnées géographiques)
  waypoints : {
  	type : [Number], required : false
  }
});
// Ajout d'une fonction de validation pour le titre et la description
TrackSchema.path('title').validate(function(title) {
  return !!title;
}, 'Title cannot be blank');
TrackSchema.path('description').validate(function(description) {
  return !!description;
}, 'Description cannot be blank');
// Méthode utilisée pour récupérer un chemin via son ID,
// va récupérer certaines des informations de l'utilisateur via populate
TrackSchema.statics.getById = function(id, cb) {
  this.findOne({
    _id: id
  }).populate('user', 'name username').exec(cb);
};

mongoose.model('Track', TrackSchema);
```

### Création des routes

Une fois le modèle créé, la seconde étape consiste à définir l'API REST qui permettra de le manipuler via les classiques opérations CRUD (création, lecture, mise à jour et destruction) et éventuellemnt d'autres opérations plus spécifiques à votre modèle. Une route consiste à associer un point d'entrée de l'API (i.e. une URL avec une méthode HTTP) à la fonction de traitement JavaScript associée au sein d'un contrôleur. Pour déclarer les routes il suffit donc de créer un fichier **TrackRoutes.js** dans le dossier **routes** du module, il contiendra le code suivant :
```javascript
'use strict';

// Récupération du contrôleur
var track = require('../controllers/TrackController');

// Fonction pour vérifier les droits d'accès à l'API
var hasAuthorization = function(req, res, next) {
  // Un utilisateur ne peut modifier que ses propres chemins 
  if (!req.track.user._id.equals(req.user._id)) {
    return res.status(401).send('User is not authorized');
  }
  next();
};

module.exports = function(Application, app, auth, database) {
    // Déclaration des routes
    app.route('/api/track')
        .get(auth.requiresLogin, track.list)
        .post(auth.requiresLogin, track.create);
    app.route('/api/track/count')
        .get(auth.requiresLogin, track.count);
    app.route('/api/track/:trackId')
        .get(auth.isMongoId, auth.requiresLogin, track.get)
        .put(auth.isMongoId, auth.requiresLogin, hasAuthorization, track.update)
        .delete(auth.isMongoId, auth.requiresLogin, hasAuthorization, track.destroy);

    app.param('trackId', track.findById);
};
```
On note l'ajout d'une fonction d'autorisation spécifique à notre modèle car seul l'utilisateur qui a créé un chemin est autorisé à le modifier. On notera également l'appel à des fonctions de vérification d'authentification de l'utilisateur ou de validité de l'identifiant d'un chemin qui sont fournies par MEAN.IO.

### Test de l'API

Afin de tester l'API j'utilise l'extension Chrome [Postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en). Cet outil permet d'envoyer des requêtes HTTP selon tout type de méthode (GET, POST, PUT, DELETE, etc.) et de créer des collections de templates de requête afin de pouvoir faire des tests rapides sur une API de type REST. Pour pouvoir l'utiliser avec MEAN.IO il faut tout d'abord récupérer un token JWT (voir article précédent) qui devra être adjoint à toutes les requêtes faites vers l'API. Pour cela le plus simple est de créer une requête de type POST vers l'URL */api/login* avec une charge utile (i.e. un body) contenant l'e-mail et le mot de passe de l'utilisateur de l'application préalablement créé (Figure 1).

![Figure 1](Figure1.png "Figure 1 : configuration de la requête de login à l'API dans Postman")

La réponse renvoyé contient votre token (Figure 2). Il faudra ensuite introduire ce token dans le header de vos prochaines requêtes en respectant le standard [Bearer](http://self-issued.info/docs/draft-ietf-oauth-v2-bearer.html).

![Figure 2](Figure2.png "Figure 2 : réponse de la requête de login à l'API contenant le token de l'utilisateur")

Ainsi le requête GET vers l'URL */api/track* à la Figure 3 va donner une réponse de status 200 ('OK') avec la charge utile `[]` (puisque nous n'avons créé aucun chemin à ce stade). Sans le token une réponse de status 401 ('User is not authorized') aurait été obtenue. Vous pouvez essayer de créer un chemin à la main en fournissant simplement un descriptif dans le corps de votre requête POST sur */api/track* puisque l'utilisateur sera automatiquement renseigné et que les coordonnées ne sont pas requises pour l'example.

![Figure 3](Figure3.png "Figure 3 : configuration d'une requête à l'API pour inclure le token dans Postman")

### Création du menu

Pour rajouter des entrées dans la barre de menu de MEAN.IO il suffit de les déclarer dans votre **app.js** en précisant pour chacune le titre, les rôles autorisés à la voir et l'état à activer côté client (voir ci-après dans la partie cliente). Dans notre cas nous aurons simplement deux entrées, une pour lister tous nos chemins et une pour créer un nouveau chemin :
```javascript
// Ajout des entrées de menu pour les utilisateurs authentifiés
Application.menus.add({
    'roles': ['authenticated'],
    'title': 'Tracks',
    'link': 'list tracks'
});
Application.menus.add({
    'roles': ['authenticated'],
    'title': 'New Track',
    'link': 'create track'
});
```

## Partie cliente (front-end)

### Service

La partie service est réellement la plus simple étant donné que le service [$resource](https://docs.angularjs.org/api/ngResource/service/$resource) d'AngularJS est spécifiquement fait pour encapsuler une API de type REST. Pour déclarer notre service d'accès à l'API des chemins il suffit de créer un fichier **TrackService.js** dans le dossier **services** de la partie publique, il contiendra la déclaration suivante :
```javascript
// Service utilisé pour accéder à l'API REST des chemins
angular.module('mean.application').factory('TrackService', ['$resource',
  function($resource) {
    return $resource('api/track/:trackId', {
      trackId: '@_id'
    }, {
      update: {
        method: 'PUT'
      }
    });
  }
]);
```
### Routes

Les routes côté front-end sont gérées via l'[AngularUI Router](https://github.com/angular-ui/ui-router). Ce module permet de d'organiser la navigation sous la forme d'une machine à état. Dans la version simple chaque à chaque état est associé une URL, une vue, un contrôleur. Dans la version plus complexe il est possible d'imbriquer les machines à état. Pour déclarer les routes il suffit de créer un fichier **ApplicationRoutes.js** dans le dossier **routes** de la partie publique, il contiendra dans notre cas les déclarations permettant d'accéder aux pages pour lister nos chemins, créer un nouveau chemin, éditer un chemin et le visualiser  :
```javascript
// Definition des routes pour les chemins
angular.module('mean.application').config(['$stateProvider',
  function($stateProvider) {

    // Page listant tous les chemins de l'utilisateur
    $stateProvider
      .state('list tracks', {
        url: '/tracks',
        templateUrl: '/application/views/ListTracks.html',
        controller: 'TrackController',
        resolve: {
          loggedin: function(MeanUser) {
            return MeanUser.checkLoggedin();
          }
        }
      })
      // Page permettant de créer un nouveau chemin
      .state('create track', {
        url: '/track/create',
        templateUrl: '/application/views/CreateTrack.html',
        controller: 'TrackController',
        resolve: {
          loggedin: function(MeanUser) {
            return MeanUser.checkLoggedin();
          }
        }
      })
      // Page permettant d'éditer un chemin
      .state('edit track', {
        url: '/track/:trackId/edit',
        templateUrl: '/application/views/EditTrack.html',
        controller: 'TrackController',
        resolve: {
          loggedin: function(MeanUser) {
            return MeanUser.checkLoggedin();
          }
        }
      })
      // Page permettant de voir un chemin
      .state('view track', {
        url: '/track/:trackId',
        templateUrl: '/application/views/ViewTrack.html',
        controller: 'TrackController',
        resolve: {
          loggedin: function(MeanUser) {
            return MeanUser.checkLoggedin();
          }
        }
      });
  }
]);
```
### Contrôleur

Notre contrôleur sera un contrôleur AngularJS classique mais nous allons faire en sorte qu'il gère les actions nécessaires à tous les états se rapportant aux chemins pour centraliser le comportement par type d'objet manipulé.
```javascript
// Contrôleur utilisé pour gérer les chemins
angular.module('mean.application').controller('TrackController', ['$scope', '$stateParams', '$location', 'TrackService', 
  function($scope, $stateParams, $location, TrackService) {
	// Créé un nouveau chemin
    $scope.create = function(isValid) {
      if (isValid) {
        var track = new TrackService({
          title: this.title,
          description: this.description
        });
        track.$save(function(response) {
          $location.path('track/' + response._id);
        });

        this.title = '';
        this.description = '';
      } else {
        $scope.submitted = true;
      }
    };
    // Détruit un chemin existant
    $scope.remove = function(track) {
      if (track) {
        track.$remove(function(response) {
          for (var i in $scope.tracks) {
            if ($scope.tracks[i] === track) {
	             $scope.tracks.splice(i,1);
            }
          }
          $location.path('tracks');
        });
      } else {
        $scope.track.$remove(function(response) {
          $location.path('tracks');
        });
      }
    };
    // Met à jour un chemin existant
    $scope.update = function(isValid) {
      if (isValid) {
        var track = $scope.track;
        if(!track.updated) {
          track.updated = [];
	}
        track.$update(function() {
          $location.path('track/' + track._id);
        });
      } else {
        $scope.submitted = true;
      }
    };
    // Liste tous les chemins de l'utilisateur
    $scope.find = function() {
      TrackService.query(function(tracks) {
        $scope.tracks = tracks;
      });
    };
    //Récupère un chemin via son ID
    $scope.findOne = function() {
      TrackService.get({
        trackId: $stateParams.trackId
      }, function(track) {
        $scope.track = track;
      });
    };
  }
]);
```

### Vues


