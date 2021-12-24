# Routing et affichage dynamique
Dans toute application web, le protocole HTTP étant primordial, l'accès à une ressource (que ce soit une page, des données, un fichier etc) se fait via un URL.

Le système de routage est donc lui aussi essentiel : le *routing* c'est le fait de décrire l'ensemble des *routes* disponibles dans votre application. Alors qu'est-ce qu'une *route* ? C'est **l'association entre une URL et un comportement** !

Le routing est donc l'ensemble des associations URL / Comportements de votre application.

Dans une Single Page Application, on chercher à faire croire au visiteur qu'il navigue dans différentes pages (et donc différentes URL) alors qu'en réalité il reste toujours sur la même page (index.html) et c'est Javascript qui donne l'impression que la page change lorsque l'URL change.

## Sommaire
  * [But de l'exercice](#but-de-l-exercice)
  * [Refactorisation de la liste](#refactorisation-de-la-liste)
  * [Créons un composant pour la page de détails](#créons-un-composant-pour-la-page-de-détails)
  * [Mettre en place le routing grâce au RouterModule](#mettre-en-place-le-routing-grâce-au-routermodule)
  * [Afficher le composant correspondant à l'URL](#afficher-le-composant-correspondant-à-l-url)
  * [Récupérer un paramètre dans l'URL](#récupérer-un-paramètre-dans-l-url)
  * [Navigation réactive sans rechargement de pages](#navigation-réactive-sans-rechargement-de-pages)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)

## But de l'exercice
Nous allons voir comment gérer le routage dynamique (sans rechargement de page) dans une application Angular grâce au module **RouterModule**.

Il nous donne un ensemble de composants et de classes qui vont permettre de configurer et gérer le routage sur notre application.

## Refactorisation de la liste

On voudrait désormais avoir deux pages dans notre application :
* La liste des tâches ;
* Le détail d'une tâche ;

Chaque page devra être définie sous la forme d'un composant Angular.

Créons le composant *TodoListPageComponent* qui représentera la page qui affichera la liste des tâches. Il reprendra l'ensemble des logiques qui sont pour l'instant dans notre *AppComponent* :

```ts
// src/app/pages/todo-list-page.component.ts

import { Component } from "@angular/core";
import { TasksService } from "../api/tasks.service";
import { Tasks } from "../types/task";

@Component({
    selector: "app-todo-list-page",
    template: `
        <app-todo-list 
            [tasks]="tasks" 
            (onToggle)="toggle($event)"
        >
        </app-todo-list>
        <app-task-form (onNewTask)="addTask($event)"></app-task-form>
    `
})
export class TodoListPageComponent {
    tasks: Tasks = [];

    constructor(private service: TasksService) { }

    ngOnInit() {
        this.service
            .findAll()
            .subscribe((tasks) => this.tasks = tasks)
    }

    toggle(id: number) {
        const task = this.tasks.find(task => task.id === id);

        if (task) {
            const isDone = !task.done;

            this.service
                .toggleDone(id, isDone)
                .subscribe(() => task.done = isDone);
        }
    }

    addTask(text: string) {
        this.service
            .create(text)
            .subscribe((tasks) => this.tasks.push(tasks[0]));
    }
}
```

N'oublions pas d'aller déclarer ce nouveau composant dans notre module *AppModule* :

```diff
// src/app/app.module.ts

declarations: [
    AppComponent,
    TodoListComponent,
    TaskFormComponent,
+   TodoListPageComponent
  ],
```

Et bien sur, supprimons toutes ces logiques de notre composant *AppComponent* :

```ts
// src/app/app.component.ts

import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div id="app">
        <h1>La Todo App</h1>

        <main>
        </main>
    </div>
  `,
  styles: []
})
export class AppComponent {}
```

## Créons un composant pour la page de détails
Comme on souhaite avoir une page qui montre le détail d'une tâche, nous allons la représenter elle aussi grâce à un composant *TodoDetailsPageComponent* :

```ts
// src/app/pages/todo-details-page.component.ts

import { Component } from "@angular/core";

@Component({
    selector: 'app-todo-details-page',
    template: `
        <p>Ca fonctionne !</p>
    `
})
export class TodoDetailsPageComponent { }
```

Pour l'instant, il n'est pas question de mettre en place de logique trop complexe, on veut avant tout s'assurer que notre futur système de routing fonctionne.

Là encore, on va le déclarer auprès du *AppModule* :
```diff
// src/app/app.module.ts

declarations: [
    AppComponent,
    TodoListComponent,
    TaskFormComponent,
    TodoListPageComponent
+   TodoDetailsPageComponent
],
```

## Mettre en place le routing grâce au RouterModule

Le framework Angular nous offre là encore des outils précis pour gérer cet aspect de nos applications. Le *RouterModule* contient des composants et des services qui vont nous permettre de gérer le routing dans notre application front.

Les Routes représentent l'ensemble des associations qui existent entre des URLs et les composants qu'il faudra afficher.

Mettons donc en place deux routes :
* `/` qui affichera la liste des tâches ;
* `/:id/details` qui affichera le détail d'une tâche ;

```ts
// src/app/app.module.ts

import { RouterModule, Routes } from '@angular/router';

// Ici, nous représentons les Routes, c'est une liste d'associations
// entre URLs et composants. Chaque URL donnera lieu à l'affichage 
// du composant qui lui est associé
const routes: Routes = [
  // La page d'accueil affichera la liste des tâches
  { path: '', component: TodoListPageComponent },
  // Ici on utilise une URL paramétrée
  { path: ':id/details', component: TodoDetailsPageComponent }
]

@NgModule({
  declarations: [
    AppComponent,
    TodoListComponent,
    TaskFormComponent,
    TodoListPageComponent,
    TodoDetailsPageComponent
  ],
  imports: [
    BrowserModule,
    ReactiveFormsModule,
    HttpClientModule,
    // On importe le RouterModule tout en lui donnant la configuration 
    // nécessaire (nos routes)
    RouterModule.forRoot(routes)
  ],
  providers: [TasksService],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Le *RouterModule* est désormais importé et configuré avec les Routes que nous lui avons soumis.

## Afficher le composant correspondant à l'URL 

Pour autant, notre application n'est pas encore prête à afficher le composant correspondant à l'URL, il nous faut lui indiquer à quel endroit Angular devra afficher le contenu adéquat. Pour cela, nous allons utiliser un **composant offert par le RouterModule : le `<router-outlet>`**


```ts
// src/app/app.component.ts

import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div id="app">
        <h1>La Todo App</h1>

        <main>
          <router-outlet></router-outlet>
        </main>
    </div>
  `,
  styles: []
})
export class AppComponent { }
```

Comme vous le remarquez, nous décidons d'afficher nos composants dans l'élément `<main>`.

Si vous naviguez sur la page d'accueil, vous retrouverez la liste des tâches. Si vous naviguez désormais sur une URL qui correspond à `/:id/details` (par exemple /12/details ou même /40/details), vous verrez s'afficher le composant de détails.

## Récupérer un paramètre dans l'URL

Le routage fonctionne bien, on voit bien que chaque URL programmée affiche bien le composant prévu. 

Néanmoins, l'URL qui affiche le composant *TodoDetailsPageComponent* contient un paramètre `:id` qui représente l'identifiant de la tâche dont on veut afficher le détai. 

On aimerait donc pouvoir récupérer la valeur donnée à ce paramètre dans l'URL. Par exemple, si le visiteur tape `/12/details`, nous aimerions retrouver l'information `12` qui prend la place du paramètre `:id` dans l'URL.

Pour ce faire, Angular peut nous injecter une instance de la classe `ActivatedRoute`, qui représente la route actuellement reconnue :

```ts
// src/app/todo-details-page.component.ts

import { Component } from "@angular/core";
import { ActivatedRoute } from "@angular/router";

@Component({
    selector: 'app-todo-details-page',
    template: `
        <p>Ca fonctionne !</p>
    `
})
export class TodoDetailsPageComponent {
    // On demande à Angular de nous injecter une instance
    // de la classe ActivatedRoute qui représente la route
    // actuelle et tous ses détails 
    constructor(private route: ActivatedRoute) { }

    ngOnInit() {
        // On peut récupérer le paramètre "id" qui se trouve
        // dans l'URL, et le transformer en nombre :
        const id: number = Number(this.route.snapshot.paramMap.get('id'));

        // Affichons l'id dans la console pour s'assurer que cela fonctionne
        console.log("Affichons ", id);
    }
}
```

Désormais, si vous vous rendez sur `/12/details`, vous devriez retrouver l'information `12` dans la console, idem si vous essayez avec `/440/details` et toutes les autres URLs qui représentent ce pattern.

On peut donc terminer cette page en allant réellement chercher au sein de l'API la tâche correspondante à l'identifiant tapé dans l'URL.

Commençons par ajouter une méthode dans notre *TasksService* :

```ts
// src/app/api/tasks.service.ts 

    /**
     * Récupère les tâches dont l'identifiant correspond
     */
    findOne(id: number): Observable<Tasks> {
        return this.http.get<Tasks>(SUPABASE_URL + '?id=eq.' + id, {
            headers: {
                "Content-Type": "application/json",
                apiKey: SUPABASE_API_KEY,
                Prefer: "return=representation"
            }
        });
    }
```

Et faisons appel à cette méthode dans notre composant *TodoDetailsPageComponent* afin de mettre en place les informations de la tâche dans le template :

```ts
// src/app/todo-details-page.component.ts

import { Component } from "@angular/core";
import { ActivatedRoute } from "@angular/router";
import { TasksService } from "../api/tasks.service";
import { Task } from "../types/task";

@Component({
    selector: 'app-todo-details-page',
    template: `
        <ng-container *ngIf="task">
            <h2>{{ task.text }}</h2>
            <strong>Statut : </strong>
            {{ task.done ? "Fait" : "Pas fait"}}
            <br />
            <a href="/">Retour aux tâches</a>
        </ng-container>

        <p *ngIf="!task">En cours de chargement</p>
    `
})
export class TodoDetailsPageComponent {
    task?: Task;

    constructor(private route: ActivatedRoute, private service: TasksService) { }

    ngOnInit() {
        const id: number = Number(this.route.snapshot.paramMap.get('id'));

        this.service
            .findOne(id)
            .subscribe(tasks => this.task = tasks[0]);
    }
}
```

Vous remarquez dans le code ci-dessus que nous avons deux affichages possibles dans le template : nous tenons compte du fait que la requête HTTP qui ira chercher les informations de la tâche est **asynchrone**. Ce qui veut dire que pendant les toutes premières secondes lors de l'affichage du composant, nous n'aurons pas encore les informations de la tâche et il nous faudra afficher autre chose.

Testez dans votre navigateur, en principe tout devrait fonctionner correctement.

## Navigation réactive sans rechargement de pages

Pour l'instant, on constate hélas que lorsque l'on clique sur un lien, le navigateur recharge la page, ce que l'on ne souhaite absolument pas voir dans une Single Page Application.

Pour gérer cet aspect des choses, le *RouterModule* nous offre une directive que l'on peut placer sur les éléments `<a>` : la directive `routerLink` qui permet de mitiger le comportement des liens :

```diff
// src/app/todo-details-page.component.ts

<ng-container *ngIf="task">
    <h2>{{ task.text }}</h2>
    <strong>Statut : </strong>
    {{ task.done ? "Fait" : "Pas fait"}}
    <br />
-   <a href="/">Retour aux tâches</a>
+   <a routerLink="/">Retour aux tâches</a>
</ng-container>


// src/app/todo-list-page.component.ts

<label>
    <input 
        type="checkbox" 
        id="item-{{ item.id }}" 
        [checked]="item.done" 
        (change)="toggle(item.id)" 
    />
    {{ item.text }}
</label>
- <a href="/{{ item.id }}/details">Details</a>
+ <a routerLink="/{{ item.id }}/details">Details</a>

```

A partir de maintenant vous devriez pouvoir naviguer de page en page sans que le navigateur ne recharge jamais la page.

Et voilà !

## Ce que vous avez appris 
* Configurer les Routes de votre application et les confier au `RouterModule` lors de son importation ;
* Afficher où vous souhaitez le composant correspondant à l'URL actuel grâce au composant `<router-outlet>` ;
* Créer des routes paramétrées grâce à la notation `:paramètre` dans une URL ;
* Récupérer des informations sur la route actuelle grâce au service `ActivatedRoute` ;
* Transformer les liens classiques en liens "réactifs" grâce à la directive `routerLink` ;


[Revenir au sommaire](../README.md) ou [Passer à la suite : Tester son code et éviter les régressions](tests.md)