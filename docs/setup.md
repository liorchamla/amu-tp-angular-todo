# Installation et utilisation de la CLI d'Angular

Créons notre application Angular en utilisant la CLI !

## Sommaire
  * [But de l'exercice](#but-de-l-exercice)
  * [Installation de la CLI](#installation-de-la-cli)
  * [Création d'une nouvelle application Angular](#création-d-une-nouvelle-application-angular)
  * [Modifions notre application](#modifions-notre-application)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)
  
## But de l'exercice

Nous allons installer l'outil principal de gestion d'une application Angular : la CLI. Puis nous verrons comment créer une application et la lancer dans notre navigateur, avant de modifier le composant principal.

## Installation de la CLI

Contrairement à un projet classique où l'on pourrait créer notre application web puis y ajouter quelques bouts de codes (comme on le fait en Vanilla JS ou même avec React ou encore Vue), Angular est un framework complet et notre application doit donc être créée comme une application Angular dès le départ.

La structure des dosssiers et les fichiers de configuration sont tels que créer une telle application "à la main" relèverait de l'exploit. Angular nous propose donc un outil qui va non seulement nous aider à créer des applications mais aussi à les faire évoluer pendant le processus de développement : **Angular CLI**

Une CLI (Command Line Interface) est un outil qu'on appelle en ligne de commandes afin qu'il performe des tâches automatisées pour nous. Pour l'installer de façon globale (afin qu'elle soit utilisable où que nous nous trouvions sur notre disque), nous utiliserons NPM :
```
npm install -g @angular/cli
```

Vous pouvez vous assurer que cela a fonctionné en tapant la commande `ng` dans votre terminal.

## Création d'une nouvelle application Angular

Pour ce TP, nous allons créer une application Angular très simple et nous allons donc préciser un certain nombre de paramètres à la CLI :
```bash
ng new todoapp --skip-tests --inline-style --inline-template --routing false --style css
```

Ce qu'il faut comprendre :
1. Nous créons une application appelée `todoapp` (la CLI va donc créer le dossier au même nom) ;
2. Nous ne voulons pas pour l'instant créer de tests unitaires (--skip-tests) ;
3. Nous ne souhaitons pas que les fichiers CSS soient séparés par défaut (--inline-style) ;
4. Nous ne souhaitons pas que les fichiers HTML soient séparés par défaut (--inline-template) ;
5. Nous ne souhaitons pas commencer tout de suite avec la gestion du routing ;
6. Nous ne souhaitons pas utiliser de pré-processeurs CSS (--style css) ;

Une fois que l'application a été créée, on va pouvoir la lancer dans le navigateur pour voir ce qui a été fait pour l'instant :

```bash
cd todoapp
ng serve
```

**Attention** : si vous utilisez **gitpod.io**, la commande a lancer sera plutôt `ng serve --public-host 0.0.0.0:4200`

## Modifions notre application

La page que vous voyez à l'écran est entièrement codée dans un fichier précis : le fichier *app/app.component.ts* ! C'est le composant principal qui contiendra toute notre application.

On peut d'ores et déjà la modifier afin qu'elle affiche notre squelette d'application TodoList :

```ts
// src/app/app.component.ts

// Le décorateur Component permet de donner à Angular des informations
// supplémentaires sur une classe afin d'expliquer que :
// 1. C'est un composant ;
// 2. Il devra afficher un template HTML donné ;
// 3. Il aura des styles scopés ;
// Et beaucoup d'autres choses encore
import { Component } from '@angular/core';

@Component({
  // Le sélecteur du composant permet de dire à Angular
  // "Quand tu vois une balise <app-root>, remplaces la
  // par le rendu de ce composant
  selector: 'app-root',
  // Le template représente le HTML qui sera rendu par ce composant
  // Il peut contenir des balises HTML classiques comme des
  // appels à d'autres composants Angular
  template: `
    <div id="app">
        <h1>La Todo App</h1>

        <main></main>
    </div>
  `,
  // Les styles nous permettent de créer des styles CSS *scopés*
  // C'est à dire que les règles définies ici ne s'appliqueront que sur
  // le template de ce composant, et pas en dehors
  styles: []
})
export class AppComponent { }
```

Et enfin, afin qu'elle ressemble à quelque chose, vous pouvez copier le contenu du dossier *docs/styles.css* de ce TP dans le fichier *src/styles.css* !

## Ce que vous avez appris
* Installer la CLI d'Angular (ng) via NPM ;
* Utiliser la CLI pour créer une nouvelle application ;
* Utiliser la CLI pour lancer un serveur de développement local ;
* Le composant `AppComponent` représente notre application ;

[Revenir au sommaire](../README.md) ou [Passer à la suite : Afficher une liste des tâches](display-list.md)