# Refactorisation des requêtes HTTP et notion de services

Maintenant qu'on a entrevu le fonctionnement du système d'injection de dépendances au sein d'Angular, nous sommes prêts à comprendre la notion de services !

## Sommaire
  * [But de l'exercice](#but-de-l-exercice)
  * [Créer un service dédié à l'API](#créer-un-service-dédié-à-l-api)
  * [Créer un fournisseur pour le TasksService](#créer-un-fournisseur-pour-le-tasksservice)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)

## But de l'exercice
Nous allons refactoriser toutes les requêtes HTTP au sein d'une même classe que nous pourrons utiliser dans nos composants grâce à l'injection de dépendances.

## Créer un service dédié à l'API

Un composant Angular a pour spécialité d'afficher des informations et de récupérer les intéractions de l'utilisateur. Toute logique complexe ou métier ne devrait pas y apparaitre. On devrait donc refactoriser ce genre de traitements dans des classes maintenables et réutilisables !

Créons une classe *TasksService* dont le but serait de centraliser les méthodes d'accès à l'API qui sont pour l'instant codées dans le *AppComponent* :

```ts
// src/app/api/tasks.service.ts

import { HttpClient } from "@angular/common/http";
import { Injectable } from "@angular/core";
import { Observable } from "rxjs";
import { Tasks } from "../types/task";

// Commençons par factoriser sur 2 constantes les URLs et
// clé d'API de SUPABASE histoire de ne pas avoir à les 
// répéter pour chaque appel à l'API :
const SUPABASE_URL = 'https://IDENTIFIANT_SUPABASE.supabase.co/rest/v1/todos';
const SUPABASE_API_KEY = "CLE_API_SUPABASE";

// Nous créons un service, c'est un objet que l'on pourra utiliser
// au sein de nos composants ou même dans d'autres services ! Afin de ne pas avoir à instancier
// nous même ce service lorsqu'on en aura besoin, on peut s'appuyer sur Angular pour l'instancier
// pour nous à la demande : Merci Angular DI (Dependency Injection) System !
// Pour indiquer à Angular qu'il devra se charger de l'instanciation de cette classe, on utilise le
// décorateur @Injectable() :
@Injectable()
export class TasksService {

    /**
     * Dans ce service, nous allons envoyer des requêtes HTTP, nous demanderons donc
     * au système d'injection de dépendances d'Angular de nous injecter une instance du HttpClient
     */
    constructor(private http: HttpClient) { }

    /**
     * Récupère l'ensemble des lignes de l'API et retourne un tableau de tâches
     */
    findAll(): Observable<Tasks> {
        return this.http.get<Tasks>(SUPABASE_URL, {
            headers: {
                "Content-Type": "application/json",
                apiKey: SUPABASE_API_KEY
            }
        });
    }

    /**
     * Met à jour le statut d'une tâche et retourne la tâche mise à jour (dans un tableau contenant une tâche)
     */
    toggleDone(id: number, isDone: boolean): Observable<Tasks> {
        return this.http.patch<Tasks>(SUPABASE_URL + '?id=eq.' + id, {
            done: isDone
        }, {
            headers: {
                "Content-Type": "application/json",
                apiKey: SUPABASE_API_KEY,
                Prefer: "return=representation"
            }
        });
    }

    /**
     * Créé une tâche auprès de l'API qui nous retournera un tableau contenant la tâche
     * nouvellement créée
     */
    create(text: string): Observable<Tasks> {
        return this.http.post<Tasks>(SUPABASE_URL, {
            text: text,
            done: false
        }, {
            headers: {
                "Content-Type": "application/json",
                apiKey: SUPABASE_API_KEY,
                Prefer: "return=representation"
            }
        });
    }
}
```

Et voilà ! Faisons le test en utilisant ce service dans le *AppComponent*. Pour ce faire, nous allons nous reposer sur le système d'injection de dépendances d'Angular pour instancier notre service.

Commençons par tester en ne modifiant que la récupération des tâches :

```ts
// src/app/app.component.ts

import { TasksService } from './api/tasks.service';

// ...

export class AppComponent {
  tasks: Tasks = [];

  // On ajoute au constructeur une demande de recevoir une instance
  // du TasksService :
  constructor(
    private http: HttpClient, 
    private service: TasksService
  ) { }

  ngOnInit() {
    // On remplace le code de la requête HTTP par l'appel
    // à la méthode findAll() de notre service, qui renverra
    // exactement la même chose que ce que renvoyait le
    // HttpClient, on réagira donc de la même manière via la 
    // méthode subscribe()
    this.service
      .findAll()
      .subscribe((tasks) => this.tasks = tasks)
  }
  
  // ...
}
```

Si vous actualisez votre navigateur, vous verrez que rien ne fonctionne, et la console devrait vous afficher une belle erreur concernant l'injecteur de dépendances !

## Créer un fournisseur pour le TasksService

Le fait d'avoir créé un service `TasksService` ne suffit pas à ce que le système d'injection de dépendances soit capable de l'instancier et de l'injecter dans nos composants.

Pour ce faire, il faut désigner un `provider` (fournisseur). Il faut préciser à quel niveau l'instance du service devra être créée, cela peut-être fait au niveau d'un composant, ou d'un module.

Faisons en sorte que le *AppModule* soit doté d'une instance de *TasksService* qui pourra être injectée dans les composants et services du module :

```ts
// src/app/app.module.ts

import { TasksService } from './api/tasks.service';

@NgModule({
  declarations: [
    AppComponent,
    TodoListComponent,
    TaskFormComponent
  ],
  imports: [
    BrowserModule,
    ReactiveFormsModule,
    HttpClientModule
  ],
  providers: [TasksService],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Actualisez maintenant votre navigateur et normalement, tout fonctionne correctement !

On peut désormais faire le remplacement de tous les appels HTTP du *AppComponent* par les méthodes du service :

```ts
// src/app/app.component.ts

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
```

## Ce que vous avez appris
* Créer un service afin d'extraire les logiques métier complexes de vos composants ;
* Permettre à Angular d'instancier et d'injecter des services grâce au décorateur `@Injectable()` et aux `providers` ;

[Revenir au sommaire](../README.md) ou [Passer à la suite : Gérer la navigation grâce au RouterModule](routing.md)