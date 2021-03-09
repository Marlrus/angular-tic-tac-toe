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

This component is our **smart** component, meaning that it has internal state that can change.

### VIM integration

I browsed for the LSP and found it: `npm install -g @angular/language-server` and adding it to the list of servers: `local servers = {'jsonls', 'tsserver', 'cssls', 'html', 'graphql', 'yamlls', 'bashls', 'vimls', 'ngserver' }`
