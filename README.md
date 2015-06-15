# MEAN.IO (2/3)

Ce second article d'une série consacrée à MEAN.IO va nous permettre d'introduire les principaux concepts permettant de créer un nouveau module afin d'étendre le canevas. Le cas d'application que je vous propose d'étudier permettra également à la plupart d'entre vous de découvrir un autre domaine, celui de la visualisation de données géographiques sous forme de carte ou sous forme 3D. 

## Création d'un module

Tout d'abord un petit rappel, l'ensemble des modules d'une application MEAN.IO est stocké dans le dossier **packages**. Le premier sous-dossier **core** inclut les modules de base livrés avec le framework et le sous-dossier **custom** les modules spécifiques à la logique applicative, c'est donc ici qu'il faudra créer vos nouveaux modules. Grâce à l'outil en ligne de commande [mean-cli](https://www.npmjs.com/package/mean-cli) il est possible d'initialiser un package (scaffolding) via :
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

le dossier d'un package ou module MEAN.IO présente la structure suivante :
```
Module folder
--- docs : contient la documentation de l'API (back-end)
--- public : partie publique du module sur le site
    --- assets : données statiques (images, css, dépendances front-end)
    --- controllers : controllers front-end (AngularJS)
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

A la racine on trouve comme dans le cas de l'application les fichiers de configuration pour npm, bower et MEAN.IO (**mean.json**). Le plus important est le fichier **app.js** qui est le point d'entrée du module. Tous les fichiers à l'intérieur du dossier **public** seront accessibles publiquement à l'URL */nom-module/chemin-relatif-fichier*. Par exemple pour accéder à un contrôleur AngularJS nommé 'controller' dans le module nommé 'module' l'URL sera *module/controllers/controller.js*.

Pour rajouter un module au canevas il suffit de lui donner un nom unique dans l'application et de le copier dans le répertoire **packages/custom** de l'application, MEAN.IO se charge du reste !

## Partie serveur (back-end)

### Création du modèle

### Création des routes

### Création du menu

## Partie cliente (front-end)

### Service

### Contrôleur

### Vues

## Contribuer

La société qui a lancé MEAN.IO (Linnovate) a également déployé une infrastructure afin de permettre à la communauté de partager et de mettre à disposition des modules, il s'agit du [MEANetwork](https://network.mean.io). La première chose pour utiliser cette infrastructure est de s'enregistrer via l'outil en ligne de commande :
```
mean register
```
Une fois enregistré vous pouvez publier un module en vous positionnant dans le dossier du module et en exécutant la commande `publish` :
```
cd packages/custom/module
mean publish
```
Lors de la publication d'un module, son code source sera en fait publié sur le service [GitLab](http://git.mean.io) propre au MEANetwork. Pour l'utilisateur nommé 'user' et le module nommé 'package' le dépôt associé sera accessible à l'URL http://git.mean.io/user/package. [GitLab](https://gitlab.com/) est un équivalenet Open Source du service [GitHub]
(https://github.com/) qui peut être déployé de façon interne.

> **Trucs & Astuces** : par défaut, à l'intérieur de chaque module de base de MEAN.IO, les fichiers portent le même nom (celui du module comme par exemple *Module.js*). Je préfère suffixer chaque fichier par le type d'objet qu'il contient (par exemple *ModuleController.js*, *ModuleRoutes.js*, etc.). En effet, même si le nom du dossier parent peut servir de discriminant, il est ainsi plus aisé de savoir à quel fichier l'on a affaire. Notamment lorsqu'ils sont ouverts simultanément sous forme d'onglets ne laissant apparaitre que le nom du fichier (et non le chemin complet) dans votre éditeur de texte favori. 
