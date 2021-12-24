# Créer une application Angular (Todo List)

Dans ce TP, vous allez créer une Single Page Application (SPA) avec Angular dans le but de découvrir différentes notions telles que les composants, les services, les appels HTTP et les Observables, le routage et bien sur les tests unitaires qui assureront la qualité et la non régression de vos codes.

## Mettre en place le projet en local
Clonez ce dépôt où bon vous semble :
```bash
git clone https://github.com/liorchamla/amu-tp-angular-todo.git
```
Ouvrez le dossier dans votre éditeur de code favoris !

## Travailler directement sur le navigateur
Plutôt que de cloner le dépôt, vous pouvez aussi travailler directement en ligne via Gitpod.

Pour cela il vous faudra *forker* le dépôt sur votre propre compte (bouton *Fork* en haut à droite de GitHub). Une fois sur la page du fork (la copie de ce dépôt sur votre propre compte), vous pourrez simplement ajouter `gitpod.io/#` juste devant l'URL du dépôt.

Par exemple, le lien suivant ouvrira **ce dépôt** dans Gitpod : https://gitpod.io/#https://github.com/liorchamla/amu-tp-angular-todo

**Attention : le travail ne sera pas sauvegardé d'une session de travail à l'autre, vous devrez systématiquement commit et push vos modifications si vous espérez les retrouver ensuite !**



## Attention au départ !

Une fois que le projet est mis en place, vous pouvez tout simplement suivre le TP étape par étape :

* [**Installation des outils de développement**](docs/setup.md)
  * [But de l'exercice](docs/setup.md#but-de-lexercice)
  * [Installation de la CLI](docs/setup.md#installation-de-la-cli)
  * [Création d'une nouvelle application Angular](docs/setup.md#création-dune-nouvelle-application-angular)
  * [Modifions notre application](docs/setup.md#modifions-notre-application)
  * [Ce que vous avez appris](docs/setup.md#ce-que-vous-avez-appris)
* [**Afficher une liste des tâches**](docs/display-list.md)
  * [But de l'exercice](docs/display-list.md#but-de-lexercice)
  * [Représenter des données dans un composant Angular](docs/display-list.md#représenter-des-données-dans-un-composant-angular)
  * [Projeter des données dans l'interface HTML](docs/display-list.md#projeter-des-données-dans-linterface-html)
  * [Ce que vous avez appris](docs/display-list.md#ce-que-vous-avez-appris)
* [**Refactoriser du code dans un composant**](docs/component.md)
  * [But de l'exercice](docs/component.md#but-de-lexercice)
  * [Création du TodoListComponent](docs/component.md#création-du-todolistcomponent)
  * [Déclarer le nouveau composant dans le AppModule](docs/component.md#déclarer-le-nouveau-composant-dans-le-appmodule)
  * [Ce que vous avez appris](docs/component.md#ce-que-vous-avez-appris)
* [**Créer des types et tirer profit de TypeScript**](docs/types.md)
  * [Créons un type qui représente la notion de tâche](docs/types.md#créons-un-type-qui-représente-la-notion-de-tâche)
  * [Créer un alias pour les tableaux de tâches](docs/types.md#créer-un-alias-pour-les-tableaux-de-tâches)
  * [Ce que vous avez appris](docs/types.md#ce-que-vous-avez-appris)
* [**Ajouter une tâche**](docs/add-item.md)
  * [But de l'exercice](docs/add-item.md#but-de-lexercice)
  * [Créons le composant TaskFormComponent](docs/add-item.md#créons-le-composant-taskformcomponent)
  * [Gérer un formulaire avec Angular](docs/add-item.md#gérer-un-formulaire-avec-angular)
    + [Importer le ReactiveFormsModule](docs/add-item.md#importer-le-reactiveformsmodule)
    + [Mettre en place un formulaire réactif](docs/add-item.md#mettre-en-place-un-formulaire-réactif)
    + [Utiliser les directives du ReactiveFormsModule](docs/add-item.md#utiliser-les-directives-du-reactiveformsmodule)
  * [Créer un événement dans notre composant](docs/add-item.md#créer-un-événement-dans-notre-composant)
  * [Ecouter l'événement *onNewTask* dans le *AppComponent*](docs/add-item.md#ecouter-lévénement-onnewtask-dans-le-appcomponent)
  * [Ce que vous avez appris](docs/add-item.md#ce-que-vous-avez-appris)
* [**Appels HTTP et API REST**](docs/http.md)
  * [But de l'exercice](docs/http.md#but-de-l-exercice)
  * [Créer un projet sur Supabase](docs/http.md#créer-un-projet-sur-supabase--)
  * [Comprendre comment fonctionne l'API de Supabase](docs/http.md#comprendre-comment-fonctionne-lapi-de-supabase)
  * [Requête HTTP avec Angular et le HttpClientModule](docs/http.md#requête-http-avec-angular-et-le-httpclientmodule)
  * [Obtenir une instance de HttpClient grâce à l'injection de dépendances](docs/http.md#obtenir-une-instance-de-httpclient-grâce-à-linjection-de-dépendances)
  * [Ne pas utiliser le constructeur pour des opérations complexes](docs/http.md#ne-pas-utiliser-le-constructeur-pour-des-opérations-complexes)
  * [Ajouter une tâche dans l'API](docs/http.md#ajouter-une-tâche-dans-l-api)
  * [Modifier le statut d'une tâche](docs/http.md#modifier-le-statut-dune-tâche)
  * [Ce que vous avez appris](docs/http.md#ce-que-vous-avez-appris)
* [**Refactoriser de la logique complexe dans un service**](docs/service.md)
  * [But de l'exercice](docs/service.md#but-de-lexercice)
  * [Créer un service dédié à l'API](docs/service.md#créer-un-service-dédié-à-lapi)
  * [Créer un fournisseur pour le TasksService](docs/service.md#créer-un-fournisseur-pour-le-tasksservice)
  * [Ce que vous avez appris](docs/service.md#ce-que-vous-avez-appris)
* [**Gérer la navigation (routing)**](docs/routing.md)
  * [But de l'exercice](docs/routing.md#but-de-lexercice)
  * [Refactorisation de la liste](docs/routing.md#refactorisation-de-la-liste)
  * [Créons un composant pour la page de détails](docs/routing.md#créons-un-composant-pour-la-page-de-détails)
  * [Mettre en place le routing grâce au RouterModule](docs/routing.md#mettre-en-place-le-routing-grâce-au-routermodule)
  * [Afficher le composant correspondant à l'URL](docs/routing.md#afficher-le-composant-correspondant-à-lurl)
  * [Récupérer un paramètre dans l'URL](docs/routing.md#récupérer-un-paramètre-dans-lurl)
  * [Navigation réactive sans rechargement de pages](docs/routing.md#navigation-réactive-sans-rechargement-de-pages)
  * [Ce que vous avez appris](docs/routing.md#ce-que-vous-avez-appris)
* [**Tester son code et éviter les régressions**](docs/tests.md)
  * [But de l'exercice](docs/tests.md#but-de-lexercice)
  * [Ecrire notre premier test](docs/tests.md#ecrire-notre-premier-test)
  * [Testons le *TodoListComponent*](docs/tests.md#testons-le-todolistcomponent)
  * [Tester le composant TodoListPageComponent](docs/tests.md#tester-le-composant-todolistpagecomponent)
  * [Ce que vous avez appris](docs/tests.md#ce-que-vous-avez-appris)

