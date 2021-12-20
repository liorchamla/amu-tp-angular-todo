```ts
import { Component, Input, Output, EventEmitter } from "@angular/core";

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
            </li>
        </ul>
    `
})
export class TodoListComponent {
    @Input()
    tasks: any[] = [];

    @Output()
    onToggle = new EventEmitter<number>();

    toggle(id: number) {
        this.onToggle.emit(id);
    }
}
```

```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { TodoListComponent } from './todo-list.component';

@NgModule({
  declarations: [
    AppComponent, TodoListComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
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
          <app-todo-list 
            [tasks]="tasks" 
            (onToggle)="toggle($event)"
          ></app-todo-list>
        </main>
    </div>
  `,
  styles: []
})
export class AppComponent {
  tasks = [
    { id: 1, text: "Aller faire des courses", done: false },
    { id: 2, text: "Faire à manger", done: true },
  ];

  toggle(id: number) {
    const task = this.tasks.find(task => task.id === id);

    if (task) {
      task.done = !task?.done;
    }
  }
}
```