# Tester les composants Angular et assurer la non régression
Notre application est désormais terminée : elle affiche une liste des tâches sauvegardées à distance et permet aussi de les modifier et d'en ajouter de nouvelles. Elle permet enfin de voir le détail d'une tâche donnée.

On veut désormais s'assurer que de futures modifications de notre code n'aillent pas à l'encontre de ce fonctionnement et pour ce faire nous allons écrire des .. Tests !

Comme tout langage de programmation, Javascript possède des utilitaires et frameworks de tests qui vont permettre d'écrire des tests unitaires afin de valider le bon fonctionnement du code.

Plus encore, le code Angular étant tout de même assez spécifique dans son fonctionnement, des librairies et utilitaires sont disponibles pour encore plus faciliter l'écriture des tests.

## Sommaire
  * [But de l'exercice](#but-de-l-exercice)
  * [Ecrire notre premier test](#ecrire-notre-premier-test)
  * [Testons le *TodoListComponent*](#testons-le--todolistcomponent-)
  * [Tester le composant TodoListPageComponent](#tester-le-composant-todolistpagecomponent)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)
## But de l'exercice
Nous allons découvrir ici les tests unitaires en utilisant les utilitaires livrés avec Angular à savoir `Jasmine` et le `TestBed` d'Angular.

**Ici, les notions importantes sur lesquelles se concentrer sont l'écriture de tests, la simulation des intéractions avec le DOM et les Composants Angular, ainsi que l'utilisation de fonctions simulées (mocks)**

## Ecrire notre premier test 

Dans toute application Angular, un framework de test est déjà prêt à l'utilisation : **Jasmine**.

Au niveau de la syntaxe, vous en serez pas perdu si vous avez déjà utilisé *Jest*. Jasmine apporte, en plus d'un *test runner*, la possibilité de jouer les tests au sein même d'un navigateur.

Créons donc un premier test dans le fichier *src/app/tests/todo-list.component.spec.ts* :

```ts
// src/app/tests/todo-list.component.spec.ts

describe("Une suite de tests", () => {
    it("should work fine", () => {
        expect(true).toBeTrue();
    });
});
```

Si vous tapez la commande `ng test`, vous verrez Jasmine se mettre en route et ouvrir une fenêtre de votre navigateur qui jouera les tests. Normalement ce premier test devrait se conclure avec succès.

## Testons le *TodoListComponent*

Tester un composant est évidemment un peu plus complexe que le test que nous venons d'écrire car **un composant ne peut pas vivre "seul", il a besoin d'un *module*** qui lui fourni des outils et un environnement. Angular nous fourni une classe `TestBed` (littéralement *lit de test*) qui permette de simuler l'existance d'un module :

```ts
// src/app/tests/todo-list.component.spec.ts

import { ComponentFixture, TestBed } from "@angular/core/testing";
import { By } from "@angular/platform-browser";

import { TodoListComponent } from "../todo-list.component"

// Ecrivons une suite de tests qui permettent de s'assurer que le
// TodoListComponent fonctionne correctement et fait ce que nous 
// attendons de lui
describe('TodoListComponent', () => {
    // Dans un test, la classe ComponentFixture nous permet de piloter et
    // d'analyser le composant et son rendu HTML. On peut notamment 
    // déclencher des événements (exactement comme si on avait le navigateur
    // dans les mains) mais aussi sélectionner des éléments HTML et les
    // analyser, et enfin on peut agir directement sur l'instance du composant
    let fixture: ComponentFixture<TodoListComponent>;

    // On voudra souvent agir directement sur l'instance du composant
    // qu'on stockera dans cette variable
    let component: TodoListComponent;

    // Avant chaque exécution d'un test dans cette suite
    beforeEach(async () => {
        // Nous configurons un "faux" module dont le seul but est d'envelopper
        // notre composant
        await TestBed.configureTestingModule({
            // Bien sur, le *faux* module que nous créons doit au moins
            // connaître le composant que nous souhaitons tester :
            declarations: [TodoListComponent],
            // Par ailleurs, notre composant faisant appel à la 
            // directive "routerLink" dans son template HTML, nous
            // devons aussi importer le RouterModule, même sans
            // configurer de Routes particulières, ne serait-ce que 
            // pour que Angular ne soit pas étonné de trouver un lien
            // portant un attribut "routerLink" :
            imports: [RouterModule.forRoot([])]
        }).compileComponents();

        // Une fois que le module a été "compilé", on va pouvoir
        // demander la création du composant exactement comme cela
        // se passerait dans le framework Angular :
        fixture = TestBed.createComponent(TodoListComponent);
        // Une fois que la fixture est prête, on peut accéder à l'instance
        // du composant qui a été instancié par le TestBed
        component = fixture.componentInstance;
    });

    // Le composant devrait afficher dans l'interface HTML autant de tâches
    // qu'on lui donne dans sa propriété tasks :
    it('should display print each tasks given in input on the screen', () => {
        // Créons un tableau de tâches tel qu'il serait si on le récupérait
        // depuis l'API :
        let MOCK_TASKS = [
            { id: 1, text: "MOCK_TASK_1", done: false },
            { id: 2, text: "MOCK_TASK_2", done: false },
        ]

        // On donne notre tableau à notre composant exactement comme si on 
        // avait écrit <app-todo-list [tasks]="MOCK_TASKS"></app-todo-list>
        component.tasks = MOCK_TASKS;

        // Attention : le fait de faire varier une propriété du composant
        // ne changera pas le HTML généré par ce même composant, car
        // ce changement n'aura lieu qu'une fois qu'Angular aura procédé
        // à une *détection de changement*
        // Pour simuler ce comportement du framework, on demande à la
        // fixture de procéder à la détection de changement
        fixture.detectChanges();

        // A partir de là, les changements ont été constaté et
        // projetés dans le template HTML. 

        // On peut donc vérifier grâce la fixture si tel ou tel élément
        // HTML existe.
        // Ici, on souhaite s'assurer que le composant a bien affiché
        // 2 élément <li>, puisque le tableau de tâches qu'on lui a donné
        // contient bien 2 tâches
        expect(fixture.debugElement.queryAll(By.css('ul li')).length).toBe(2);
    });
)};
```

Vous l'aurez compris, ici nous souhaitons nous assurer que lorsqu'on donne un tableau de tâches au composant *TodoListComponent*, alors il affiche ces tâches dans l'interface HTML.

Faisons un deuxième test :

```ts
// src/app/tests/todo-list.component.spec.ts

describe("TodoListComponent", () => {
    // ...

    // Nous souhaitons tester que lorsqu'une case à cocher sera
    // changée par l'utilisateur, alors le composant émettra
    // bien un événement via son @Output() "onToggle" et que celui-ci
    // propagera bien l'identifiant de la tâche qui a été touchée :
    it('should emit an event with onToggle output when a checkbox changes', () => {
        // On représente un nombre qui vaut 0 par défaut mais qui devra 
        // changer lorsque l'Output "onToggle" émettre un événément :
        let expectedEventId = 0;

        // Imaginons encore un tableau de tâches
        let MOCK_TASKS = [
            { id: 1, text: "MOCK_TASK_1", done: false },
            { id: 2, text: "MOCK_TASK_2", done: false },
        ];

        // Et confions le à notre composant afin qu'il le projète dans 
        // l'interface 
        component.tasks = MOCK_TASKS;

        // Faisons aussi en sorte que lorsque onToggle émettra une valeur
        // nous récupérions cette valeur et nous l'assignions à 
        // expectedEventId :
        component.onToggle
            .subscribe((id) => expectedEventId = id);

        // Lançons la détection de changement qui permettra de 
        // projeter les données des tâches dans l'interface
        fixture.detectChanges();

        // On récupère la checkbox de la première tâche grâce à son 
        // identifiant (qui devrait être décochée pour l'instant)
        const checkbox = fixture.debugElement.query(By.css('#item-1'));

        // Simulons le fait qu'un utilisateur change cette checkbox 
        checkbox.triggerEventHandler('change', {});

        // On s'attend à ce que le changement de la checkbox ait 
        // déclenché l'événement onToggle de notre composant et 
        // donc que notre expectedEventId ait pris la valeur de 
        // l'identifiant de la tâche qu'on a touché (coché ou décoché) :
        expect(expectedEventId).toBe(1);
    });
})
```

Normalement, ces tests devraient bien se passer lorsque vous lancez `ng test` !

Jusqu'ici, nous ne testons qu'un petit composant en isolation : pas d'injection de dépendance, pas de service à utiliser, pas d'appel HTTP, rien de spécial.

## Tester le composant TodoListPageComponent

Mais comment tester un composant un peu plus complexe qui fera appel à tous ces concepts (services, injection de dépendances, etc) ?

Nous souhaitons tester le composant *TodoListPageComponent* qui dépend et appel le service *TasksService*. Pourtant il est hors de question que lors de nos tests nous appelions l'API pour de bon. Il faudra donc **créer des fonctions simulées (appelées *mock functions*)** : de fausses fonctions qui vont se substituer aux méthodes du *TasksService*, et qu'on pourra contrôler et espionner !

```ts
// src/app/tests/todo-list-page.component.ts

import { HttpClientModule } from "@angular/common/http";
import { ComponentFixture, TestBed } from "@angular/core/testing";
import { ReactiveFormsModule } from "@angular/forms";
import { By } from "@angular/platform-browser";
import { RouterModule } from "@angular/router";
import { of } from "rxjs";
import { TasksService } from "../api/tasks.service";
import { TodoListPageComponent } from "../pages/todo-list-page.component";
import { TaskFormComponent } from "../task-form.component";
import { TodoListComponent } from "../todo-list.component";

// Créons une suite de tests qui validera le fonctionnement
// du TodoListPageComponent 
describe("TodoListPageComponent", () => {
    // La fixture nous permettra d'analyser le rendu HTML
    // et de le piloter (simuler des événements etc)
    let fixture: ComponentFixture<TodoListPageComponent>;

    // Avant chaque test de cette suite 
    beforeEach(async () => {
        // On configure un *faux* module qui enveloppera notre
        // composant et lui fournira les outils dont il a besoin
        await TestBed.configureTestingModule({
            // Déclarons les composants nécessaires :
            declarations: [
                // Le composant que l'on test
                TodoListPageComponent, 
                // Comme il fera appel dans son template à <app-todo-list>
                // nous avons besoin de déclarer ce composant
                TodoListComponent, 
                // Comme il fera appel dans son template à <app-task-form>
                // nous avons aussi besoin de le déclarer
                TaskFormComponent
            ],
            // Notre composant demande une injection du TasksService
            // il faut donc qu'Angular soit capable de l'instancier
            providers: [TasksService],
            // Enfin, comme on utilise des composants et directives
            // issues d'autres modules, il faut les importer :
            imports: [
                // Notre TasksService s'attend à se faire injecter
                // une instance du HttpClient, il faut donc importer
                // ce module :
                HttpClientModule, 
                // Le TaskFormComponent utilise différentes directives
                // de ce module, qu'on importe donc aussi :
                ReactiveFormsModule, 
                // Certains éléments <a> portent une directive "routerLink"
                // On a donc besoin d'importer le RouterModule, même sans
                // configurer de routes particulières :
                RouterModule.forRoot([])
            ],
        }).compileComponents();

        // Créons la fixture qui permettra de piloter notre composant
        fixture = TestBed.createComponent(TodoListPageComponent);
    });

    // Nous souhaitons nous assurer que le composant appellera bien
    // le TasksService et qu'il affichera les tâches que celui-ci
    // lui renverra
    it('should call TasksService and display returned tasks', () => {
        // Nous pouvons obtenir une instance du TasksService grâce
        // au TestBed
        const service = TestBed.inject(TasksService);

        // Nous souhaitons remplacer la méthode findAll() du service
        // par une fausse fonction (mock function) afin de : 
        // 1. S'assurer qu'on ne spamme pas l'API HTTP pendant nos tests
        // 2. Pouvoir contrôler la valeur de retour  de la méthode 
        // directement depuis nos tests
        //
        // Pour cela, on créé un "spy" (espion), qui va prendre la 
        // place de la véritable méthode :
        const findAllSpy = spyOn(service, "findAll");

        // On peut dire aussi que, lorsqu'elle sera appelée, la méthode 
        // findAll() retournera un Observable qui contiendra un tableau
        // d'une seule tâche, exactement comme si l'API avait répondu
        // ce même tableau
        findAllSpy.and.returnValue(of([
            { id: 1, text: "MOCK_TASK_1", done: false }
        ]));

        // On lance la détection de changements (la première), qui va
        // donc charger le composant et appeler ngOnInit()
        fixture.detectChanges();

        // On s'attend à ce qu'après le chargement (ngOnInit), le 
        // composant ait bien appelé la méthode findAll() qu'on a remplacé
        expect(findAllSpy).toHaveBeenCalled();

        // Et qu'il ait projeté dans le template une tâche sous la 
        // forme d'un <li>, vu que ce que findAll() a renvoyé est un 
        // tableau contenant une tâche
        expect(fixture.debugElement.queryAll(By.css('ul li')).length).toBe(1);
    });
)};
```

Ce qui est important dans ce test, c'est l'utilisation d'une *mock function* (fonction simulée) qui nous permet de **remplacer une méthode** du service afin de surcharger son comportement réel (faire un appel HTTP sur une API distante) par un comportement que l'on contrôle de A à Z.

**Notez : la méthode *findAll()* renvoie le résultat sous la forme d'un objet Observable, il est donc important que notre fausse fonction retourne un résultat similaire (un Observable donc), ce qu'on fait grâce à la fonction *of()* !**

Finissons de tester les comportements de la liste, en voyant ce qu'il se passe si l'on coche une case à cocher :

```ts
// src/app/tests/todo-list-page.component.ts

describe("TodoListPageComponent", () => {
    // ...

    // Nous souhaitons nous assurer que lorsqu'on change le statut
    // d'une tâche, alors la méthode toggleDone() du TasksService
    // sera bien appelée avec l'identifiant de la tâche en question
    // ainsi qu'un booléen représentant le statut
    it('should call TaskService on a checkbox change', () => {
        // On obtient l'instance du TasksService grâce au TestBed
        const service = TestBed.inject(TasksService);

        // On créé un mock pour la fonction findAll() afin de faire
        // apparaître une seule tâche dont l'identifiant est 1
        // et le statut est false :
        const findAllSpy = spyOn(service, "findAll");
        findAllSpy.and.returnValue(of([
            { id: 1, text: "MOCK_TASK_1", done: false }
        ]));

        // On créé un autre mock pour la fonction toggleDone
        const toggleSpy = spyOn(service, "toggleDone");

        // Le résultat de toggleDone ici importe peu, retournons
        // simplement un Observable d'un tableau vide :
        toggleSpy.and.returnValue(of([]));

        // On lance la première détection de changement qui devrait
        // initialiser le composant, faire appel à notre fausse findAll()
        // et projeter le résultat dans le template HTML 
        fixture.detectChanges();

        // On récupère la case à cocher correspondant à la tâche 1
        const checkbox = fixture.debugElement.query(By.css('#item-1'));
        // On déclenche le changement sur la checkbox comme si
        // l'utilisateur avait cliqué dessus
        checkbox.triggerEventHandler('change', {});

        // Et on s'attend à ce que la méthode toggleDone() du TasksService
        // soit appelée avec en premier paramètre l'identifiant de 
        // la tâche (1) et en second paramètre l'inverse de son statut
        // actuel (donc true) :
        expect(toggleSpy).toHaveBeenCalledWith(1, true);
    });
)};
```

Et voilà ! En lançant `ng test` vous devriez constater que tout se passe bien !

## Ce que vous avez appris 
* Utiliser le `TestBed` afin de fournir à votre composant l'environnement dont il a besoin pendant vos tests ;
* Décrire une suite de tests avec Jasmine ;
* Utiliser un `ComponentFixture` pour analyser le rendu du composant et simuler des intéractions de l'utilisateur ;
* Créer des `mock functions` grâce aux *espions* de Jasmine afin de remplacer certains comportements de vos classes au sein de vos tests ;