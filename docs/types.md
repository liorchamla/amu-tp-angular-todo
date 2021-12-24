# Créer des types pour tirer profit de TypeScript

L'avantage de TypeScript est de représenter un code à typage faible (le Javascript) comme si c'était un langage à typage fort !

Les avantages sont multiples :
1. Des erreurs qu'on n'aurait détecté que lors de l'exécution seront détectée dès la compilation ;
2. Nos IDE peuvent plus facilement deviner une autocomplétion adéquate en fonction du type d'une variable ;
3. On peut créer des *types customs*, et donc représenter dans le code des concepts métiers ;

## Sommaire
  * [Créons un type qui représente la notion de tâche](#créons-un-type-qui-représente-la-notion-de-tâche)
  * [Créer un alias pour les tableaux de tâches](#créer-un-alias-pour-les-tableaux-de-tâches)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)
## Créons un type qui représente la notion de tâche

Comme nous allons traiter un peu partout dans notre code de la notion de tâche, pourquoi ne pas avoir un type qui représente cette notion ? Cela nous aiderait tant pendant l'écriture du code (via l'autocomplétion de l'IDE) que pendant la phase de compilation (détection d'erreurs) :

```ts
// src/app/types/task.ts

// Représentons une tâche par tout objet qui aurait :
export type Task = {
    // Une propriété id numérique
    id: number;
    // Une propriété text de type string
    text: string;
    // Une propriété done de type booléen
    done: boolean;
}
```

Grâce à ce type, nous pouvons déclarer une variable donnée comme étant une `Task`. TypeScript sera capable de nous dire si quelque chose ne va pas, par exemple :

```ts
// Cette ligne causera une erreur car l'objet que l'on créé se dit
// de type Task, mais ne contient aucune des propriétés nécessaires
const t: Task = {};

// Idem ici, car il manque une propriété done :
const t: Task = {
    id: 1,
    text: "Hello World"
};

// Idem ici, car id n'a pas le bon type :
const t: Task = {
    id: "1", // ERREUR ! On devait avoir un <number> et on a un <string>
    text: "Hello World",
    done: true
}
```

## Créer un alias pour les tableaux de tâches
Pour l'instant, on peut déjà déclarer qu'une variable est un tableau de tâches :
```ts
const tableau: Task[] = [];
```

Néanmoins l'écriture peut parfois paraître fastidieuse, et TypeScript nous donne un moyen de faire mieux :

```ts
// src/app/types/task.ts
export type Tasks = Task[];
```

Et voilà ! Désormais, toute variable déclarée comme étant de type `Tasks` représentera en réalité un tableau d'objets de type `Task` !

```ts
// Ecrire :
const tasks: Tasks = [];

// Correspond exactement à :
const tasks: Task[] = [];
```

## Utilisons ces types dans notre code
On peut maintenant renforcer le code du composant *TodoListComponent* grâce à nos nouveaux types :

```ts
// src/app/app.component.ts 

import { Component } from '@angular/core';
import { Tasks } from './types/task.ts';

@Component({
  // ...
})
export class AppComponent {
  tasks: Tasks = [
    { id: 1, text: "Aller faire des courses", done: false },
    { id: 2, text: "Faire à manger", done: true },
  ];
}

// src/app/todo-list.component.ts

import { Component, Input } from "@angular/core";
import { Tasks } from './types/task.ts';

@Component({
    // ...
})
export class TodoListComponent {
    @Input()
    tasks: Tasks = []; 
}
```

Et voilà, on a créé des types qui pourront être utilisés dans notre code pour en renforcer la cohérence !

## Ce que vous avez appris
* Créer un type pour représenter une notion métier ;
* Créer un type qui réutilise un autre type (un tableau de X, un objet dont une propriété est un X, etc) ;


[Revenir au sommaire](../README.md) ou [Passer à la suite : Ajouter une tâche](add-item.md)