# Afficher une liste de tâches

On peut maintenant commencer à se concentrer sur notre application et afficher de façon dynamique une liste de tâches.

Pour ça, on rappellera le flux des données dans une application front :
1. Les données et les traitements sont gérées via Javascript ;
2. Les affichages sont gérés dans le HTML ;
3. Le Javascript peut projeter des données dans l'interface HTML (le DOM) ;
4. Le HTML peut faire appel à des traitements positionnés dans le Javascript ;

## Sommaire
  * [But de l'exercice](#but-de-l-exercice)
  * [Représenter des données dans un composant Angular](#représenter-des-données-dans-un-composant-angular)
  * [Projeter des données dans l'interface HTML](#projeter-des-données-dans-l-interface-html)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)

## But de l'exercice
Le but ici est donc de comprendre comment des données (un tableau d'objets représentant les tâches de la liste) peut être projeté dans l'interface HTML.

## Représenter des données dans un composant Angular

Dans un composant Angular, la partie *données et traitements* est représentée par la classe TypeScript, alors que la partie *interface HTML* est représentée par le *template*.

Si l'on souhaite représenter des données, on va donc le faire sous la forme d'une propriété de notre classe TypeScript :

```ts
// app/app.component.ts 

export class AppComponent {
  tasks: any[] = [
    { id: 1, text: "Aller faire des courses", done: false },
    { id: 2, text: "Faire à manger", done: true },
  ];
}
```

Voilà, désormais, lorsque Angular voudra faire appel à notre composant, il instanciera cette classe et l'objet correspondant aura une propriété `tasks` qui sera un tableau d'objets.

## Projeter des données dans l'interface HTML

Désormais, il nous faut comprendre comment projeter ces tâches dans l'interface HTML. Dans un composant Angular, c'est le *template* qui exprime le visuel qui devra être rendu par le composant. Dans le template, on peut :
* faire appel à des éléments HTML classiques ;
* faire appel à d'autres composants Angular ; 
* décrire des comportements dynamiques (boucles, conditions) ;
* projeter des données exposées par la classe TS ;

Décrivons donc notre liste de tâches via une liste d'éléments `<li>` :

```ts
// app/app.component.ts 

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
  tasks: any[] = [
    { id: 1, text: "Aller faire des courses", done: false },
    { id: 2, text: "Faire à manger", done: true },
  ];
}
```

Plusieurs choses à remarquer ici :
1. Nous avons pu **exprimer un parcours de tableau** grâce à une directive spécifique `*ngFor="let task of tasks"` qui signifie que l'élément qui accueille cette directive sera affiché **pour chaque élément du tableau tasks** ;
2. Nous avons pu **projeter les données TS dans le HTML** grâce à la syntaxe d'interpolation `{{ task.text }}` qui dit à Angular : *affiche le contenu de cette variable* ;
3. Nous avons pu **lier à des attributs HTML des données de notre classe TS** grâce à la syntaxe de liaison de données (data binding) `[checked]="task.done"` qui dit à Angular : *la valeur de cet attribut est le résultat de l'évaluation de cette expression Javascript* ;

En actualisant la page dans votre navigateur, vous constatez qu'effectivement, les tâches remontent bien dans l'interface !

## Ce que vous avez appris
* Représenter des données dans un composant via les propriétés de la classe TS ;
* Projeter des données de la classe TS dans le HTML via la syntaxe d'interpolation `{{ data }}` ;
* Lier un attribut HTML à une donnée via la syntaxe de data binding `[attr]="data"` ;
* Exprimer une itération sur les élements d'un tableau grâce à la directive `*ngFor="let item of items"` ;

[Revenir au sommaire](../README.md) ou [Passer à la suite : Refactoriser du code dans un composant](component.md)
