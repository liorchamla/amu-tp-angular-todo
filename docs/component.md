# Refactorisation et création d'un composant 

Le composant `AppComponent` est le *conteneur* de notre application, il va contenir toutes nos fonctionnalités et nos vues. Si on ne refactorise pas nos fonctionnalités dans des composants à part, il deviendra très vite énorme et ingérable.

Pour plus de maintenabilité, nous allons créer un composant `TodoListComponent` qui aura pour but d'afficher une liste de tâches.

## Sommaire

## But de l'exercice
Nous allons refactoriser le code actuellement présent dans *AppComponent* au sein d'un nouveau composant. 

**Ici, on se concentrera sur la possibiltié de scinder un composant en plusieurs composants plus petits et sur la possibilité de communiquer d'un composant parent à un composant enfant**.

## Création du TodoListComponent

Créons un nouveau fichier *app/todo-list.component.ts* qui contiendra notre nouveau composant. Nous ferons le choix de ne pas stocker la liste des tâches dans ce composant là. Il n'aura pour seul mission que d'afficher une liste de tâche qui, elle, restera dans le composant *AppComponent* :

```ts
// src/app/todo-list.component.ts

import { Component, Input } from "@angular/core";

@Component({
    // Ce composant sera affiché par Angular à chaque fois
    // qu'un élément <app-todo-list> sera rencontré dans
    // un template HTML
    selector: 'app-todo-list',
    // Le HTML reprend ici notre liste de tâches
    template: `
        <ul>
            <li *ngFor="let item of tasks">
                <label>
                <input 
                    type="checkbox" 
                    id="item-{{ item.id }}" 
                    [checked]="item.done" 
                />
                {{ item.text }}
                </label>
            </li>
        </ul>
    `
})
export class TodoListComponent {
    // Le décorateur Input permet de spécifier à Angular
    // que cette donnée tasks pourra être renseignée depuis
    // l'extérieur du composant. Par défaut, le tableau sera vide
    // mais il prendra la valeur qu'on lui donne depuis l'extérieur
    // si c'est le cas
    @Input()
    tasks = []; 
}
```

On a donc un composant dont le but est de récupérer depuis l'extérieur un tableau de tâches et de le projet dans le template HTML.

## Déclarer le nouveau composant dans le AppModule

On aimerait donc appeler ce composant depuis notre  *AppComponent*. Pour l'instant c'est encore impossible car notre application Angular ne connaît pas notre nouveau composant. 

Ce qui représente notre application de façon globale c'est le **module AppModule** ! Il déclare tous les composants qu'il contient, mais aussi les autres modules dont il souhaite importer les composants et services, et plein d'autres informations.

**On ne pourra pas utiliser le TodoListComponent dans notre code tant que le module ne l'aura pas identifié !**

```ts
// src/app/app.module.ts

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

Maintenant que le *AppModule* connaît notre *TodoListComponent*, on va pouvoir l'appeler dans le code du composant principal *AppComponent* :

```ts
// src/app/app.component.ts

import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div id="app">
        <h1>La Todo App</h1>

        <main>
          <app-todo-list 
            [tasks]="tasks"
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
}
```

Et voilà ! En rechargeant la page dans le navigateur, vous devriez constater que ça fonctionne correctement !

1. L'*AppComponent* possède une liste de tâches ;
2. Il appelle dans son template le *TodoListComponent* grâce à son sélecteur ;
3. Il fait le lien entre sa propriété *tasks* et l'attribut (Input) *tasks* du *TodoListComponent* ;
4. Le *TodoListComponent* se charge avec les tâches qu'il a reçu grâce à son *Input* ;

## Ce que vous avez appris 
* Créer un nouveau composant et le déclarer dans le *AppModule* ;
* Utiliser un composant dans le template d'un autre composant grâce à son *sélecteur* ;
* Permettre de passer des informations à un composant depuis l'extérieur grâce au décorateur `@Input()` ;