npm install -g @angular/cli

ng new todoapp --skip-tests --inline-style --inline-template --routing false --style css --skip-git

cd todoapp

ng serve

styles.css à copier

```ts
// src/app/app.component.ts

import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div id="app">
        <h1>La Todo App</h1>

        <main></main>
    </div>
  `,
  styles: []
})
export class AppComponent {
  title = 'todoapp';
}

```