# Ajouter une tâche via un formulaire

Angular nous propose une suite d'outils très simples pour nous aider à gérer les formulaires et en extraire les données !

## Sommaire
  * [But de l'exercice](#but-de-l-exercice)
  * [Créons le composant TaskFormComponent](#créons-le-composant-taskformcomponent)
  * [Gérer un formulaire avec Angular](#gérer-un-formulaire-avec-angular)
    + [Importer le ReactiveFormsModule](#importer-le-reactiveformsmodule)
    + [Mettre en place un formulaire réactif](#mettre-en-place-un-formulaire-réactif)
    + [Utiliser les directives du ReactiveFormsModule](#utiliser-les-directives-du-reactiveformsmodule)
  * [Créer un événement dans notre composant](#créer-un-événement-dans-notre-composant)
  * [Ecouter l'événement *onNewTask* dans le *AppComponent*](#ecouter-l-événement--onnewtask--dans-le--appcomponent-)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)
## But de l'exercice 

Nous allons créer un composant dont le but est de permettre au visiteur de créer une tâche via un formulaire.

**Ici, on se concentrera sur la gestion de formulaire et aussi sur la façon dont un composant peut faire savoir à son parent qu'il a une information à lui donner !**

## Créons le composant TaskFormComponent

On pourrait coder le formulaire directement dans le *TodoListComponent*, mais ce serait encore une fois prendre le risque d'un code moins maintenable et testable.

Partons donc du principe que l'on va donner cette responsabilité à un composant qui sera simple : le *TaskFormComponent* :

```ts
// src/app/task-form.component.ts

import { Component } from "@angular/core";

@Component({
    selector: "app-task-form",
    template: `
        <form>
            <input 
              type="text" 
              name="todo-text" 
              placeholder="Ajouter une tâche" 
            />
            <button>Ajouter</button>
        </form>
    `
})
export class TaskFormComponent {}
```

Vous le voyez, ce composant est extrêmement simple (pour l'instant) et pourra être appelé dans d'autres composants en utilisant le sélecteur `<app-task-form>`. Il nous faut maintenant le déclarer auprès du *AppModule* pour pouvoir l'utiliser :

```ts
// src/app/app.module.ts

import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { TaskFormComponent } from './task-form.component';
import { TodoListComponent } from './todo-list.component';

@NgModule({
  declarations: [
    AppComponent, 
    TodoListComponent, 
    TaskFormComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Et on pourra enfin venir afficher ce composant en dessous de la liste :

```ts
// src/app/app.component.ts

import { Component } from '@angular/core';
import { Tasks } from './types/task';

@Component({
  selector: 'app-root',
  template: `
    <div id="app">
        <h1>La Todo App</h1>

        <main>
          <app-todo-list 
            [tasks]="tasks" 
          ></app-todo-list>
          <app-task-form></app-task-form>
        </main>
    </div>
  `,
  styles: []
})
export class AppComponent {
  tasks: Tasks = [
    { id: 1, text: "Aller faire des courses", done: false },
    { id: 2, text: "Faire à manger", done: true },
  ];
}
```

Et voilà, notre formulaire apparait bel et bien sur la page, il va maintenant falloir le gérer !

## Gérer un formulaire avec Angular 

Dans une application Angular, il y'a deux façons principales de gérer un formulaire :
* Les formulaires Template Driven (dont la configuration et la mise en place se fait dans le template, et la gestion se fait dans le TypeScript) ;
* Les formulaires Reactif (dont la configuration, la mise en plage et la gestion se font tous dans le TypeScript) ;

Nous nous concentrerons ici sur l'approche Reactive.

### Importer le ReactiveFormsModule

Afin de bénéficier des fonctionnalités relatives aux formulaires Reactifs, nous allons **importer le ReactiveFormsModule** dans notre *AppModule*, ce n'est qu'une fois ce module importé que nous pourrons utiliser ses avantages :

```ts
// src/app/app.module.ts

import { NgModule } from '@angular/core';
import { ReactiveFormsModule } from '@angular/forms';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { TaskFormComponent } from './task-form.component';
import { TodoListComponent } from './todo-list.component';

@NgModule({
  declarations: [
    AppComponent,
    TodoListComponent,
    TaskFormComponent
  ],
  imports: [
    BrowserModule,
    // En important le ReactiveFormsModule, on importe des
    // composants, directives et services qu'il met à notre 
    // disposition !
    ReactiveFormsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### Mettre en place un formulaire réactif

Maintenant que tout est en place au niveau du *AppModule*, on va pouvoir créer une représentation du formulaire au sein de notre composant :

```ts
// src/app/task-form.component.ts

@Component({
  // ...
})
export class TaskFormComponent {
    // Notre formulaire sera représenté par la propriété "form"
    // Elle contient une instance d'un FormGroup (groupe de champs)
    // qui lui même ne contient qu'un seul champ appelé "text" et qui
    // sera représenté par une instance de FormControl
    // Ces deux classes (FormGroup et FormControl) vont nous donner 
    // toutes les fonctionnalités nécessaires à gérer des champs de 
    // formulaires et en extraire les données !
    form = new FormGroup({
        text: new FormControl()
    });
}
```

Bien sur, cette mise en place ne fera pas le lien *automatiquement* avec nos éléments HTML au sein du template.

### Utiliser les directives du ReactiveFormsModule

On va donc désormais travailler sur le HTML du formulaire afin de faire le lien entre nos éléments (`<form>` et `<input>`) et nos instances de `FormGroup` et `FormControl` dans notre classe TS :

```ts
// src/app/task-form.component.ts

import { Component } from "@angular/core";
import { FormControl, FormGroup } from "@angular/forms";

@Component({
    selector: "app-task-form",
    template: `
        <form (ngSubmit)="onSubmit()" [formGroup]="form">
            <input 
              formControlName="text"
              type="text" 
              name="todo-text" 
              placeholder="Ajouter une tâche" 
            />
            <button>Ajouter</button>
        </form>
    `
})
export class TaskFormComponent {
    form = new FormGroup({
        text: new FormControl()
    });

    onSubmit() {
        console.log(this.form.value);
    }
}
```

Vous remarquez plusieurs choses ici :
1. L'utilisation de la directive `formGroup` à qui on lie la donnée `form` de notre classe TS ;
2. L'utilisation de la directive `formControlName` à qui on assigne le nom du FormControl `text` qui se trouve dans le FormGroup ;
3. L'utilisation de l'événement `(ngSubmit)` sur la balise `<form>` qui permet d'appeler la méthode `onSubmit()` de notre classe TS lors de la soumission du formulaire ;

Si vous actualisez votre navigateur et que vous testez le formulaire, vous verrez dans votre console le résultat !

Il faut désormais faire en sorte que notre *TaskFormComponent* prévienne son parent que le formulaire a été soumis et lui passe le texte entré par l'utilisateur.

## Créer un événement dans notre composant

Cette communication depuis l'intérieur de notre composant vers le composant parent (celui qui appelle notre composant) se fait via un **événement** ! 

Un événement que le composant parent pourra écouter et qui déclenchera un traitement à l'intérieur du composant parent :

```ts
// src/app/task-form.component.ts

import { Component, Output, EventEmitter } from "@angular/core";

export class TaskFormComponent {

    // Le décorateur @Output permet de signaler à Angular 
    // que notre composant va pouvoir faire sortir une information
    // vers l'exéterieur sous a forme d'un événément !
    // Et pour émettre un événement, on utilise une instance
    // de la classe EventEmitter tout en précisant que l'information
    // qui sera émise sera une string (le texte tapé dans le formulaire !) :
    @Output()
    onNewTask = new EventEmitter<string>();

    form = new FormGroup({
        text: new FormControl()
    });

    onSubmit() {
        // Au moment de la soumission, on va déclencher un événement
        // Et la donnée que l'on va émettre sera la valeur du champ 
        // "text" qui se trouve dans notre formulaire !
        this.onNewTask.emit(this.form.value.text);

        // On pourra même réinitialiser la valeur du formulaire
        // une fois que le traitement sera terminé :
        this.form.setValue({
            text: ''
        });
    }
}
```

On vient de créer une propriété `onNewTask` dans notre composant *TaskFormComponent* et nous l'avons décoré avec `@Output()` qui veut dire que le composant parent pourra **écouter cette propriété comme un événement**.

## Ecouter l'événement *onNewTask* dans le *AppComponent*

Désormais, on veut être au courant à chaque fois que le visiteur ajoute une tâche dans le *TaskFormComponent*. Du coup, dans le *AppComponent* on va **écouter l'événement onNewTask* :

```ts
// src/app/app.component.ts 

import { Component } from '@angular/core';
import { Tasks } from './types/task';

@Component({
  selector: 'app-root',
  template: `
    <div id="app">
        <h1>La Todo App</h1>

        <main>
          <app-todo-list 
            [tasks]="tasks" 
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
  tasks: Tasks = [
    { id: 1, text: "Aller faire des courses", done: false },
    { id: 2, text: "Faire à manger", done: true },
  ];

  // La méthode addTask recevra une string
  addTask(text: string) {
    // Elle s'en servira pour créer une nouvelle tâche dans 
    // le tableau des tâches, et Angular mettra à jour 
    // l'affichage afin d'en tenir compte !
    this.tasks.push({
      id: Date.now(),
      text: text,
      done: false
    });
  }
}
```
Vous pouvez remarquer ici plusieurs choses :
1. On écoute l'événement `onNewTask` comme on écouterait n'importe quel autre événément (`click` ou `change`), la syntaxe est la même : `(eventQuOnEcoute)="MethodeQuonAppelle()"` ;
2. Lorsque l'on souhaite transmettre les données de l'événement, on fait référence à une variable `$event` qui représente l'événement ;


Si vous actualisez votre navigateur, vous verrez que désormais, en soumettant le formulaire, une nouvelle tâche va apparaitre dans la liste !

## Ce que vous avez appris
* Importer les fonctionnalités relatives à la gestion des formulaires grâce au `ReactiveFormsModule` ;
* Représenter un formulaire dans une classe TS avec les clases `FormGroup` et `FormControl` ;
* Lier des éléments HTML à vos groupes (via la directive `formGroup`) et contrôles (via la directive `formControlName`) ;
* Réagir à la soumission d'un formulaire grâce à la directive `(ngSubmit)` ;
* Emettre un événement depuis l'intérieur d'un composant vers l'extérieur grâce au décorateur `@Output()` et à la classe `EventEmitter` ;


[Revenir au sommaire](../README.md) ou [Passer à la suite : Appels HTTP et API REST](http.md)