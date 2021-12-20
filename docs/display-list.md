```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div id="app">
        <h1>La Todo App</h1>

        <main>
          <ul>
            <li *ngFor="let item of tasks">
              <label>
                <input type="checkbox" id="item-{{ item.id }}" [checked]="item.done" />
                {{ item.text }}
              </label>
            </li>
          </ul>
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
}
```

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div id="app">
        <h1>La Todo App</h1>

        <main>
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