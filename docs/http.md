# Appels HTTP et API REST avec Supabase

Pour l'instant, les données de notre applications sont **particulièrement éphémères** ! En effet, à chaque réactualisation du navigateur, les tâches reviennent au départ. Il existe plusieurs possibilités pour pallier à ce problème :

1. Le stockage local : on peut tout à fait décider de stocker les données de notre application sur le navigateur via le *LocalStorage* (dont la durée de vie est illimitée sauf intervention de l'utilisateur) ou le *SessionStorage* (dont la durée de vie est relativement courte) ;
2. Le stockage distant : on peut décider au contraire de stocker les données à distance, de telle sorte que la durée de vie du stockage soit réellement illimitée et **surtout que les données restent accessibles y compris sur d'autres navigateurs** ;


**Notez que Javascript n'a aucun moyen de se connecter à une base de données, qu'elle soit distante ou locale !**

Par contre, ce que Javascript sait très bien faire, c'est appeler un serveur web distant via des requêtes HTTP.

Il faut donc créer une application web backend qui aura accès à la base de données, et que l'on pourra appeler depuis notre javascript via des requêtes HTTP (le concept même d'API ;-)).

## Sommaire 
  * [But de l'exercice](#but-de-l-exercice)
  * [Créer un projet sur Supabase](#créer-un-projet-sur-supabase--)
  * [Comprendre comment fonctionne l'API de Supabase](#comprendre-comment-fonctionne-l-api-de-supabase)
  * [Requête HTTP avec Angular et le HttpClientModule](#requête-http-avec-angular-et-le-httpclientmodule)
  * [Obtenir une instance de HttpClient grâce à l'injection de dépendances](#obtenir-une-instance-de-httpclient-grâce-à-l-injection-de-dépendances)
  * [Ne pas utiliser le constructeur pour des opérations complexes](#ne-pas-utiliser-le-constructeur-pour-des-opérations-complexes)
  * [Ajouter une tâche dans l'API](#ajouter-une-tâche-dans-l-api)
  * [Modifier le statut d'une tâche](#modifier-le-statut-d-une-tâche)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)

## But de l'exercice
Nous allons créer une base de données PostgreSQL sur un serveur distant avec une table qui permettra de stocker les données de nos tâches. 

Pour ce faire nous utiliserons un service sur mesure : Supabase (une alternative aux services Firebase de Google).

Cette solution nous permet non seulement de créer des bases de données, mais intègre aussi par défaut une API qui permet d'intéroger ces bases de données via des requêtes HTTP.

Nous allons donc :
1. Créer un compte sur le service Supabase (nécessite un compte GitHub)
2. Créer un projet et une base de données adéquate
3. Connecter notre application front à Supabase afin de stocker les données à distance

## Créer un projet sur Supabase :
[Rendez vous sur le service Supabase](https://supabase.com) et cliquez sur ***"Start your project"*** afin de vous y connecter.

Une fois authentifié via GitHub, vous pourrez créer un projet en cliquant sur ***New Project***.

La création du projet peut durer quelques minutes et vous pourrez ensuite aller sur le ***Table Editor*** de votre base de données et cliquer sur ***New Table*** afin de créer une table que vous appellerez ***todos*** et dont les champs seront :

| Nom du champ | Type | Defaut |
| --- | --- | --- |
| id | Laissez tel quel
| created_at | Laissez tel quel
| text | varchar | null |
| done | bool | false |

Vous pouvez enfin insérer des lignes selon vos choix afin d'avoir déjà des données d'exemple.

## Comprendre comment fonctionne l'API de Supabase

Avant de continuer, vous devrez tout de même essayer de comprendre l'API en vous rendant dans la documentation (icône de fichier sur la gauche) et en sélectionnant ***Bash*** sur l'onglet de droite pour véritablement voir les requêtes HTTP qui sont acceptables ainsi que les options.

Une fois dans la documentation de l'API, vous pourrez même sélectionner le menu "***todos***" dans la barre de gauche et voir les requêtes HTTP spécifiques à la table *todos*.

Vous allez avoir besoin de votre identifiant de projet ainsi que de la clé de sécurité. Vous trouverez ces informations dans la partie *Authentication* de la documentation de l'API.

## Requête HTTP avec Angular et le HttpClientModule

Angular nous offre un module qui contient tous les outils nécessaires à faire des requêtes HTTP dans son *HttpClientModule*.

Commençons donc par importer ce module dans notre *AppModule* :

```ts
// src/app/app.module.ts

import { HttpClientModule } from "@angular/common/http";

@NgModule({
  declarations: [
    AppComponent,
    TodoListComponent,
    TaskFormComponent
  ],
  imports: [
    BrowserModule,
    ReactiveFormsModule,
    // En important le HttpClientModule, on rend disponible dans notre 
    // application un service crucial, une instance de la classe HttpClient
    // On pourra utiliser cet objet dans nos composants pour effectuer
    // des requêtes HTTP :
    HttpClientModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Le fait d'importer le *HttpClientModule* nous permet d'utiliser dans le reste de notre application un ***service*** : le **HttpClient**.

## Obtenir une instance de HttpClient grâce à l'injection de dépendances

Comme nous l'avons vu, il existe dans le *HttpClientModule* une classe appelée *HttpClient*, néanmoins elle serait beaucoup trop complexe à instancier à la main en faisant par exemple :
```ts
const http = new HttpClient(....);
```

C'est pour ce genre de situations qu'Angular met à notre disposition un système complexe (et complet) d'***Injection de dépendances***. C'est un système qui permet d'instancier automatiquement des objets lorsqu'on les demande en paramètre du constructeur d'une de nos classes.

Mettons en pratique au sein du *AppComponent* :
```ts
// src/app/app.component.ts

import { HttpClient } from "@angular/common/http";
// ...

export class AppComponent {
  // Supprimons les tasks qui étaient en dur dans le tableau
  // d'objets et remplaçons cela par un tableau vide au départ
  tasks: Tasks = [];

  // Grâce au constructor, on peut indiquer à Angular que notre
  // composant aura besoin d'une instance de la classe HttpClient
  constructor(private http: HttpClient) {

    // Ayant obtenu l'instance de HttpClient, on peut l'utiliser
    // pour appeler Supabase en méthode GET. On peut tout de suite
    // indiquer à la méthode GET qu'elle doit s'attendre à recevoir un json
    // correspondant à un tableau de tâches (le fameux type Tasks).
    // On n'oubliera pas aussi de préciser pour cette requête HTTP
    // les entêtes importantes comme le Content-Type ou la clé API
    this.http.get<Tasks>('https://IDENTIFIANT_SUPABASE.supabase.co/rest/v1/todos', {
      headers: {
        "Content-Type": "application/json",
        apiKey: "CLE_API_SUPABASE"
      }
    })
    // Lorsque la requête aura terminé son travail et que le serveur
    // aura répondu, nous recevrons une liste de tâches que
    // nous pourrons alors assigner à notre propriété "tasks"
    .subscribe((tasks) => this.tasks = tasks)
  }

  // ...
}
```

Et tadaaa ! Ca marche :

1. Angular voit qu'un composant *AppComponent* doit être rendu ;
2. Angular veut donc instancier un objet de la classe *AppComponent* ;
3. Il inspecte le constructeur de la classe *AppComponent* et se rend compte qu'il ne pourra pas l'instancier sans lui passer une instance de la classe *HttpClient* ;
4. Il instancie la classe *HttpClient* ;
5. Il instancie la classe *AppComponent* en passant en paramètre l'instance du *HttpClient* ;

Voilà en gros comment fonctionne le système d'injections de dépendances (qu'on appellera à partir de maintenant le ***DI***).     

## Ne pas utiliser le constructeur pour des opérations complexes

Pour l'instant, nous exécutons la requête HTTP dans le constructeur du composant, ce n'est pas une bonne pratique car le constructeur devrait être réservé à des opérations simples d'initialisation des propriétés de la classe.

Pour palier à ce problème, Angular nous offre des **méthodes de cycle de vie** du composant qui permettent de spécifier quoi faire lorsque le composant se charge réellement, lorsqu'il est détruit, et d'autres étapes de sa vie.

Utilisons la méthode `ngOnInit()` qui sera appelée lors du chargement du composant *AppComponent* :

```ts
// src/app/app.component.ts

export class AppComponent {
  tasks: Tasks = [];

  // Le constructeur nous permet toujours d'obtenir une instance
  // du HttpClient, mais on ne l'utilise plus du tout dans 
  // le constructeur
  constructor(private http: HttpClient) { }

  // La méthode ngOnInit sera appelée par Angular lors du chargement 
  // du composant. C'est typiquement ici que l'on placera nos comportements
  // complexes à exécuter au départ :
  ngOnInit() {
    this.http.get<Tasks>('https://IDENTIFIANT_SUPABASE.supabase.co/rest/v1/todos', {
      headers: {
        "Content-Type": "application/json",
        apiKey: "CLE_API_SUPABASE"
      }
    }).subscribe((tasks) => this.tasks = tasks)
  }

  // ...
}
```

En rechargeant la page, vous devriez constater que tout fonctionne correctement !

## Ajouter une tâche dans l'API
Maintenant qu'on a entrevu le fonctionnement du client Http d'Angular, on va pouvoir continuer à l'utiliser pour gérer l'ajout d'une tâche !

Modifions le comportement de la méthod `addTask()` du *AppComponent :

```ts
// src/app/app.component.ts

addTask(text: string) {
    // Appelons l'API en POST et signalons que nous recevrons
    // en retour un JSON représentant un tableau de tâches
    this.http.post<Tasks>(
      'https://IDENTIFIANT_SUPABASE.supabase.co/rest/v1/todos', 
      // Passons à l'API un objet à insérer dans la base de données
      // qui ne comporte que le text et le statut (vu que l'id sera
      // assigné automatiquement par la base de données)
      {
        text: text,
        done: false
      }, 
      // N'oublions pas les entêtes permettant d'informer le 
      // serveur sur ce qu'on envoie et ce que l'on souhaite
      // recevoir
      {
        headers: {
          "Content-Type": "application/json",
          apiKey: "CLE_API_SUPABASE",
          Prefer: "return=representation"
        }
      }
    )
    // Lorsque la requête sera terminée et qu'on aura reçu la réponse du serveur
    // nous recevrons un tableau de tasks, dont la première
    // (et la seule) sera celle qu'on vient d'ajouter.
    // Il nous suffira donc de la pousser dans le tableau, et Angular
    // réaffichera le tableau modifié dans l'interface
    .subscribe((tasks) => this.tasks.push(tasks[0]));
}
```

Faites le test dans votre navigateur, tout devrait fonctionner correctement.

## Modifier le statut d'une tâche 
Il ne nous reste plus qu'à permettre à l'utilisateur de changer le statut d'une tâche lors d'un click sur une case à cocher.

Pour ce faire, modifions tout d'abord le composant *TodoListComponent*, de telle sorte qu'il puisse tenir compte d'un changement sur une checkbox, et prévenir l'*AppComponent* qu'une tâche a changé :

```ts
// src/app/todo-list.component.ts

import { Component, Input, Output, EventEmitter } from "@angular/core";
import { Tasks } from "./types/task";

@Component({
    selector: 'app-todo-list',
    template: `
        <ul>
            <li *ngFor="let item of tasks">
                <label>
                    <input 
                        type="checkbox" 
                        id="item-{{ item.id }}" 
                        [checked]="item.done" 
                        (change)="toggle(item.id)" 
                    />
                    {{ item.text }}
                </label>
                <a href="/{{ item.id }}/details">Details</a>
            </li>
        </ul>
    `
})
export class TodoListComponent {
    @Input()
    tasks: Tasks = [];

    // On utilise encore une fois le décorateur @Output()
    // pour exprimer le fait que onToggle sera un événement
    // qui pourra être écouté par le composant parent, et on
    // utilisera pour cela un EventEmitter de type number
    // car l'information passée lors de cet événement sera
    // l'identifiant de la tâche qui doit changer (un numérique donc)
    @Output()
    onToggle = new EventEmitter<number>();

    // Cette méthode sera appelée lorsqu'une checkbox change
    toggle(id: number) {
      // On se sert de l'EventEmitter pour émettre l'identifiant 
      // d'une tâche qui change afin que le composant parent
      // soit au courant que cette tâche doit changer de statut
      this.onToggle.emit(id);
    }
}
```

Au sein de l'*AppComponent* on va pouvoir désormais écouter un événement `onToggle` sur le composant `<app-todo-list>`:


```ts
// src/app/app.component.ts

@Component({
  selector: 'app-root',
  template: `
    <div id="app">
        <h1>La Todo App</h1>

        <main>
          <app-todo-list 
            [tasks]="tasks" 
            (onToggle)="toggle($event)"
          ></app-todo-list>
          <app-task-form 
            (onNewTask)="addTask($event)"
          ></app-task-form>
        </main>
    </div>
  `,
  styles: []
})
export class AppComponent {
  // ...

  // Lorsque le composant TodoListComponent va emettre l'événement
  // (onToggle), on va l'écouter et appeler cette méthode on passant
  // l'événement (qui est un identifiant numérique d'une tâche)
  toggle(id: number) {
    // On retrouve la tâche qui correspond à l'identifiant
    const task = this.tasks.find(task => task.id === id);

    // Si la tâche existe
    if (task) {
      // On récupère l'inverse de son statut
      const isDone = !task.done;
      
      // On appelle l'API en PATCH (pour modifier une tâche)
      this.http.patch<Task>(
        'https://IDENTIFIANT_SUPABASE.supabase.co/rest/v1/todos?id=eq.' + id, 
        // On ne passe que la donnée qui doit changer
        {
          done: isDone
        }, 
        // Et toujours les entêtes importantes
        {
          headers: {
            "Content-Type": "application/json",
            apiKey: "CLE_API_SUPABASE",
            Prefer: "return=representation"
          }
        }
      )
      // Lorsque la réponse revient, il nous suffit simplement
      // de faire évoluer la tâche locale, et Angular actualisera
      // l'interface HTML
      .subscribe(() => task.done = isDone);
    }
  }
}
```

Testez le changement de statut dans votre navigateur en modifiant une tâche et en actualisant ensuite la page, la tâche devrait avoir conservé son statut.

## Ce que vous avez appris
* Utiliser le `HttpClient` en important le `HttpClientModule` ;
* Utiliser **l'injection de dépendances** pour obtenir dans une classe une instance d'un service qui vous intéresse ;
* Envoyer des requêtes HTTP avec différentes méthodes grâce au `HttpClient` ;
* Spécifier un comportement asynchrone à exécuter lors du retour de la requête HTTP grâce à la méthode `subscribe()` ;


[Revenir au sommaire](../README.md) ou [Passer à la suite : Refactoriser de la logique complexe dans des services](service.md)
