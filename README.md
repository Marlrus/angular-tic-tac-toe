# TicTacToeAngular

This is a quick learning experience using Angular just to stop hating on it without trying it.

## Angular CLI

To create an app, there is no npx command, you have to install the angular cli globally: `npm i -g @angular/cli` This gives us access to the **ng** command.

To create an app we run: ng new appName which will ask us a few questions about our app because Angular comes with configuration out of the box. Here I chose strict compiling, routing, and scss for styles. Since I'm not using VSCode, I will be using the commands for Angular in the CLI.

An angular app comes with a default tsconfig, testing using jasmine.

## SPA Setup

Angular is a declarative UI creator like React. By default, angular creates an app component **src/app**. If we open the **index.html** file we can see that it has a tag that uses **app-root** in it. This is very similar to the "root" id in CRA's index.html file.

If we go to the app.component.ts file we can see that this matches what we have under **selector**.

## Component Anatomy

Every Angular component has 3 main files:

1. component.ts

Where you define any TS logic and internal state of the component.

2. component.html

Contains the template and actual UI and allows the binding of the **ts** data to the template.

3. component.scss

Contains the styling for the component, this file is **scoped** to the component and does not require the .module.scss syntax that React requires.

We will delete all of the boilerplate in the component.html file. We delete the 500+ lines and just leave the **router-outlet** component on which allows us to map components to routes. This component can also handle _code splitting and lazy loading_ out of the bat, like NextJS and unlike CRA.

### VIM integration

I browsed for the LSP and found it: `npm install -g @angular/language-server` and adding it to the list of servers: `local servers = {'jsonls', 'tsserver', 'cssls', 'html', 'graphql', 'yamlls', 'bashls', 'vimls', 'ngserver' }`

## Creating a Component

We can create a component using the CLI command `ng generate component <name> [options]`, the shorthand for generate is **g**. We run: `ng g component square --inline-style --inline-template`

This command created a square directory with a square.component.ts file. The component with our optios cames with two imports from @angular/core **Component** and **OnInit**. The latter is a **lifecycle hook**. We delete in because we won't use it.

In the **class** we export we have a constructor and onInit but we remove it because we are not using it.

### Component Decorator

Every component starts with a **decorator** which allows us to determine how the component will work.

1. **selector** determines the name that the component will have.

2. **template** This part holds the HTML of our component, the JSX equivalent. How we do this is by using a `{{ varName }}` notation to access the state, logic that we have in the class.

```typescript
@Component({
  selector: "app-square",
  template: ` <p>{{ random }}</p> `,
  styles: [],
})
export class SquareComponent {
  random = Math.random();
}
```

This code will display a random number inside a p tag.

### Input decorator

We import Input and then set it inside the class. I had some errors, probably because of strict compilation options, and could only remove the error by making the value optional:

```typescript
export class SquareComponent {
  @Input() value?: "X" | "O";
}
```

### Value in Component

We can go to the place in the app html where we are using the component and add the proprety to the component as an **attribute**.

```html
<app-square [value]="'X'"></app-square> <router-outlet></router-outlet>
```

We can add the value straight away, or **dynamically** by using brackets allowing us to use expressions or dynamic value from a parent component.

At the moment we only have an @Input in our component, and it doesn't have a way to change its own state, making it a **dumb** component.

## Board Component

This component is our **smart** component, meaning that it has internal state that can change.

We create a new board component to manage the state that will be used in the SquareComponent. Here we will just use the default options in the creation.

Since we didn't set the flags we had before, we can see that this component has paths to the template and styles:

```typescript
import { Component, OnInit } from "@angular/core";

@Component({
  selector: "app-board",
  templateUrl: "./board.component.html",
  styleUrls: ["./board.component.scss"],
})
export class BoardComponent implements OnInit {
  constructor() {}

  ngOnInit(): void {}
}
```

Inside our class we add properties that we are going to use. Im getting tons of warnings and changing my tsconfig file does absolutely nothing to it unfortunately. One solution was to add ! or ?, for this case I will go with !.

```typescript
  squares!: any[];
  xIsNext!: boolean;
  winner!: string | null;
```

We will use ngOnInit this time, which is the equivalent of React's **useEffect** hook.

```typescript
  ngOnInit(): void {
    this.newGame();
  }

  newGame() {
    this.squares = Array(9).fill(null);
    this.winner = null;
    this.xIsNext = true;
  }
```

Here we will give the component its initial state.

### Computed properties

We can derive propeties from other state to use in our component. In this case we know what user is next depending on the xIsNext value but it is a boolean. We can compute the current user using that boolean and a TS getter:

```typescript
  get player() {
    return this.xIsNext ? 'X' : 'O';
  }
```

### Tic tac toe logic

We want to have a handler for the logic of the game, we call it **makeMove**. Here we will see if someone has won, if the square being touched has been chosen already and if the player changes.

```typescript
  makeMove(i: number) {
    if (this.squares[i]) return;

    // replace value at existing spot with current player
    this.squares.splice(i, 1, this.player);
    this.xIsNext = !this.xIsNext;

    this.winner = this.calculateWinner();
  }

  calculateWinner() {
    const lines = [
      [0, 1, 2],
      [3, 4, 5],
      [6, 7, 8],
      [0, 3, 6],
      [1, 4, 7],
      [2, 5, 8],
      [0, 4, 8],
      [2, 4, 6],
    ];
    for (let i = 0; i < lines.length; i++) {
      const [a, b, c] = lines[i];
      if (
        this.squares[a] &&
        this.squares[a] === this.squares[b] &&
        this.squares[a] === this.squares[c]
      ) {
        return this.squares[a];
      }
    }
    return null;
  }
```

## Binding data to HTML

We go into the board.component.html and here we create the HTML template, accessing the properties with the handlebar syntax. To execute methods from our class we can use events with parenthesis:

```html
<button (click)="newGame()">Start new Game</button>
```

This will execute newGame() when we click this button.

### Structural Directive

These handle things like conditional rendering. We will use **ngIf** to conditionally render the winner text, `*ngIf` displays if the condition on the right hand side is true:

```html
<h2 *ngIf="winner">Player {{ winner }} won the game!</h2>
```

This is the equivalent of using && or a ternary in React.

The other common directive is `*ngFor` which is equivalent to using **map** in React.

```html
<main>
  <app-square
    *ngFor="let val of squares; let i = index"
    [value]="val"
    (click)="makeMove(i)"
  >
  </app-square>
</main>
```

Here we declare in the directive that val will be the value of each square, and i will be the index. We access the value and use our handler using the index.

### Style

```scss
app-square {
  border: 1px gray solid;
  height: 200px;
}
```

Here we see that we can target our component directly as if it were a native html tag. We create a grid of 3 200px columns.

Once that is done I adjusted the square component and app.component.html to use board. This gives us the working app with _some_ styling to have the basics done.

## UI Library: Nebular

To install the ui kit from nebular we have to run `ng add @nebular/theme` which is like Angular's own package manager.

This command installs and configures nebular in our project, we are asked what theme we want and then if we want to add custom styling and animations. This is actually quite impressive and I've not seen something like this for React beyond the npx project creations for the different frameworks.

This command creates a **themes.scss** file in our environment/ directory. Here we can customize the variables used in the theme. We can also go to the **app.module.ts** and see that there are some new modules in the @NgModule decorator. This allows us to use angular in a _progressive_ way, especially using the parts of the framework that we actually need.

In this file we will import the NbButtonModule which we then need to add to the imports Array:

```typescript
import { NbThemeModule, NbLayoutModule, NbButtonModule } from "@nebular/theme";

@NgModule({
  declarations: [AppComponent, SquareComponent, BoardComponent],
  imports: [
    BrowserModule,
    AppRoutingModule,
    BrowserAnimationsModule,
    NbThemeModule.forRoot({ name: "cosmic" }),
    NbLayoutModule,
    NbEvaIconsModule,
    NbButtonModule,
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

We now have access to these components in our app.

#### Difference from React

This is actually quite interesting because in React we import components on a Component to Component basis, which greatly differs. This is probably possible in Angular but we might be missing out on the benefits of the @NgModule decorator.

### Changes in our app.component.html

This installation added some components into our app html component automatically, which is pretty nuts. If we open the app, it looks better just with having run that command.

## Changing Square style

We now have access to a new **directive** called nbButton which in itself gives us access to other **directives** (props) that will change the appearance and can be seen in the docs for nebular. We add these to the template we have in square:

```typescript
  template: `
    <button nbButton status="success" *ngIf="!value">{{ value }}</button>
    <button nbButton hero status="success" *ngIf="value == 'X'">
      {{ value }}
    </button>
    <button nbButton hero status="info" *ngIf="value == 'O'">
      {{ value }}
    </button>
  `,
```

I had to add the success status to get better styling because Nebular's default button is plain.

And to the board component:

```typescript
<button nbButton outline status="danger" (click)="newGame()">
  Start new Game
</button>
```

## PWA

We run: `ng add @angular/pwa` to install the package. This sets up the new module to the module file of our app:

```typescript
import { ServiceWorkerModule } from '@angular/service-worker';
import { environment } from '../environments/environment';

ServiceWorkerModule.register('ngsw-worker.js', { enabled: environment.production }),
```

And creates the **ngsw-config.json** file which allows us to configure the service worker. It also creates the manifest and adds the icons for the manifest and installation.

Since PWA is a production feature we need to bulid the app which is done with `ng build`

### Running locally

The tutorial suggested we host the app, I used the **lite-server** package to run the app. To do this, we must go to dist/tic-tac-toe-angular which was created at build and then run `lite-server`. The app worked correctly.

Unfortunately, since we are not hosting, the app is basically unfit for PWA installation due to HTTPS and not being able to load the service-worker. It seemed like a good idea, but the execution didn't work as expected.

### PWA problems

After running this command my app completely died. Nb components were not detected and the app just died. Restarting the server fixed it.
