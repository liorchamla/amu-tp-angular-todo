```ts
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

```diff

declarations: [
    AppComponent,
    TodoListComponent,
    TaskFormComponent,
+   TodoListPageComponent
  ],
```

```ts
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

```ts
import { Component } from "@angular/core";

@Component({
    selector: 'app-todo-details-page',
    template: `
        <p>Ca fonctionne !</p>
    `
})
export class TodoDetailsPageComponent { }
```

```ts
import { NgModule } from '@angular/core';
import { ReactiveFormsModule } from '@angular/forms';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { TaskFormComponent } from './task-form.component';
import { TodoListComponent } from './todo-list.component';
import { HttpClientModule } from "@angular/common/http";
import { TasksService } from './api/tasks.service';
import { TodoListPageComponent } from './pages/todo-list-page.component';
import { RouterModule, Routes } from '@angular/router';
import { TodoDetailsPageComponent } from './pages/todo-details-page.component';

const routes: Routes = [
  { path: '', component: TodoListPageComponent },
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
    RouterModule.forRoot(routes)
  ],
  providers: [TasksService],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

```ts
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

```ts
import { Component } from "@angular/core";
import { ActivatedRoute } from "@angular/router";

@Component({
    selector: 'app-todo-details-page',
    template: `
        <p>Ca fonctionne !</p>
    `
})
export class TodoDetailsPageComponent {
    constructor(private route: ActivatedRoute) { }

    ngOnInit() {
        const id: number = Number(this.route.snapshot.paramMap.get('id'));

        console.log("Affichons ", id);
    }
}
```

```ts
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

```ts
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

```diff
<ng-container *ngIf="task">
    <h2>{{ task.text }}</h2>
    <strong>Statut : </strong>
    {{ task.done ? "Fait" : "Pas fait"}}
    <br />
-   <a href="/">Retour aux tâches</a>
+   <a routerLink="/">Retour aux tâches</a>
</ng-container>

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

Et voilà !