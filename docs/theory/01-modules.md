# Theory: Modules & Injection

## Introduction
Let me begin with a question and an answer:

> Question: What would you have ripped out of Angular if you had one breaking change for free. ?
<details>
<summary>Answer</summary>
 It's NgModules 😬
</details>

That question got asked at the Angular Panel at the Angular Connect conference in London. Igor Minar himself answered this. Not in this shortness but in this determination. He talked about the modules thing being unfortunate.

But it's learnable and I guess everyone already has some kind of internal image already. I want to start slow with this workshop by going over all the parts of a module.

Heres the interface of NgModule.

```typescript
export interface NgModule {
  providers?: Provider[];
  declarations?: Array<Type<any> | any[]>;
  imports?: Array<Type<any> | ModuleWithProviders<{}> | any[]>;
  exports?: Array<Type<any> | any[]>;
  entryComponents?: Array<Type<any> | any[]>;
  bootstrap?: Array<Type<any> | any[]>;
  schemas?: Array<SchemaMetadata | any[]>;
  // skipped: id, jit
}
```

That's it. Nothing more. Let's look at each property with the following parts:

+ Module 1: Providers
+ Module 2: Declarations
+ Module 3: Imports/Exports
+ Module 4: EntryComponents
+ Module 5: Bootstrap
+ Module 6: Schema

## Module 1: Providers
Branch `modules/providers`

Before Angular 6 you would put your services here, nowadays it's fine to rely on `providedIn: 'root'` or `providedIn: YourTargetModule` inside the `@Injectable` decorator. That way you don't have to import the service anywhere, the DI dependency inside your consumers ensure that the service will be loaded and Angular detects that it should be globally provided. The service will be part of the root injector provided by `AppModule`.

You only use the provider array to override existing services
or provide other values (like custom factories, tokens, static values) in your dependency injection tree.

Services are tree-shakable by not having explicit references in the module.

You can inject any value within this form:

```
{ provide: SomeKeyWord, useClass: SomeClassToProvide },
```
1. There are some variations: Instead of providing a class you can provide a value or a factory function creating your values.
2. SomeKeyWord can be virtually anything that can be used to identify a dependency: a string, an abstract class or an InjectionToken (provided by Angular)

`InjectionToken` is a really important concept you use it like so:

Compare the two provided values here. I use the `ReflectiveInjector` for an easier demonstration. Angular uses it under the hood.

```typescript

class GreetNormal {
  greet() {
    console.log('Hello Sir!');
  }
}
class GreetLoud {
  greet() {
    console.log('HELLO SIR!');
  }
}

const GreetingService1 = new InjectionToken<string>('GreetingService');
const GreetingService2 = new InjectionToken<string>('GreetingService');

const myInjector = ReflectiveInjector.resolveAndCreate([
  { provide: GreetingService1, useClass: GreetNormal },
  { provide: GreetingService2, useClass: GreetLoud }
]);

const greeterA = myInjector.get(GreetingService1);
greeterA.greet();

const greeterB = myInjector.get(GreetingService2);
greeterB.greet();

```

> Hello Sir!
> HELLO SIR!

What's happening?
We have two tokens with the same name but we received — like requested — two different values depending on the given token. The involved string is only so we developers can identify them — internally they are unique references and as such
Angular can deliver different values. This is intended and prevents conflicts.

Imagine this scenario. We use the string `GreetingService` now. There is no chance to know what the user wanted. This could have happened if two parts of your application are developed in two different teams. `InjectionToken` save us from such a conflict. (In this case, the last one just wins so it's the loud one).

```typescript
const myInjector = ReflectiveInjector.resolveAndCreate([
  { provide: 'GreetingService', useClass: GreetNormal },
  { provide: 'GreetingService', useClass: GreetLoud }
]);
```

## Module 2: Declarations
You list here components, pipes and directives. Those are the things related to the DOM/template. You need to tell Angular about their existence as their presence in the template would not include the classes itself.

You already know the number one rule: You can declare a class only once. If you have problems organizing your code you might run into this problem often. You usually create Feature Modules, UI Modules or Shared Modules and import them often while the declaration of elements are done once in those modules.

In UI Libraries you even go as far as one module per component — this ensures that the consumer can import your component module that already defines all dependencies (like other UI components) correctly.

## Module 3: Imports/Exports
Branch `modules/exports`

Exports are tightly bound to declarations and it can be confusing because you only list here declarations you want to use outside of the module. You add here components but also other modules so you can reexport them — to ensure that your own consumer benefits from the other module too.

Examples:

+ UI Module: You have a UI module to share components you will list ALL components in declarations as well as in exports.
+ Shared Module: If you have a feature module where the components are only used internally you don't want to expose them so you never list them in exports. (private vs public components).

+ A module can also reexport what any other modules exports.

Take a single UI element with its own module:

```typescript
@NgModule({
  declarations: [
    ButtonComponent
  ],
  imports: [
    CommonModule
  ],
  exports: [
    ButtonComponent
  ]
})
export class ButtonModule { }
```

The module ensure that the component is declared and that it can be used by any module that imports this module.
Let's do this in our UIModule.

```typescript
const uiComponents = [AbcComponent, DefComponent, FooComponent, BarComponent];

@NgModule({
  declarations: [
    ...uiComponents
  ],
  exports: [
    ...uiComponents,
    ButtonModule // reexport what ButtonModule already exports.
  ],
  imports: [
    CommonModule,
    ButtonModule // only required if any other component defined here needs a button
  ],
})
export class UiModule { }
```

1. We add the `ButtonModule` to the exports list. Whoever imports the `UiModule` will now be able to use the button inn its templates too.

You can also see imports in action:<br>
That's the place to import other modules to use their components/directives/pipes (only if they are listed in its `exports: []`).

## Module 4: EntryComponents
Branch: `modules/entry-components`

When you create a component by using its tag it's called `declarative`. If you don't use the tag but create it dynamically you build it `imperatively`. (via [angular docs](https://angular.io/guide/entry-components))

If you want to create a component dynamically you have to add it to the `entryComponents` list of your module.
Any component that is mounted by a router will build imperatively. You might wonder why this is working. The router adds the component to the `entryComponents` list automatically during compilation. So Angular follow its own rules — no magic!

A component can be build `declarative` and `imperatively` and the same time, just ensure the component is in the entryComponents list as said. Let's try this.

We have a template in `app.component.html`

```html

<h1>Declarative Created Components</h1>
<app-unicorn></app-unicorn>
<app-usual-component></app-usual-component>

<h1>Dynamically Created Components</h1>
<ng-container #unicorns></ng-container>
```

and the following unicorn component:<br>

```typescript
@Component({ template: '🦄' })
export class UnicornComponent { }
```

The app module looks like this — Unicorn is part of the entryComponents. Good read to build it.

```typescript
@NgModule({
  declarations: [
    AppComponent,
    UsualComponentComponent,
    UnicornComponent
  ],
  entryComponents: [
    UnicornComponent
  ],
  // ...
```

This is already working and you will see the usual component as well as the unicorn. The part _Dynamically Created Components_ is still empty. Let's fill it.

We will reference the ng-container and dynamically create the unicorns

```typescript
@ViewChild('unicorns', {read: ViewContainerRef})
unicornsContainer: ViewContainerRef;

constructor(
  private componentFactoryResolver: ComponentFactoryResolver
) {}

ngAfterViewInit() {
  const unicornComponent = this.componentFactoryResolver.resolveComponentFactory(UnicornComponent);
  this.unicornsContainer.createComponent(unicornComponent);
}
```

**Notice:** This won't run any change detection inside the generated unicorn components (because you have not created any input bindings).

## Module 5: Bootstrap
Branch `modules/bootstrap`

That is the place to put in any component you want to use as an app root. Usually you have only a single app root but actually you could create multiple whereas each component will be a separate DOM tree

Here a quick example. You can create an inline component and add it to the bootstrap list

```typescript
@Component({
  selector: `other-app-root`,
  template: `I'm a full Angular application 🤪`
})
class OtherAppComponent {
}

@NgModule({
  declarations: [
    AppComponent, OtherAppComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
  bootstrap: [AppComponent, OtherAppComponent]
})
export class AppModule { }
```

Now list it in your `index.html` together with the default app root tag.

```html
<body>
  <app-root></app-root>
  <other-app-root></other-app-root>
</body>
```

You can access all root elements with `getAllAngularRootElements()` in the console

## Module 6: Schemas
Branch `modules/schemas`

Have you ever used this part of NgModules ? Probably if you tinkered around with Angular Elements or if you have some advanced tests.
I tell you why.

I bet you know the following error message:
> Uncaught Error: Template parse errors: 'your-tag-name' is not a known element: <br>
> 1. If 'your-tag-name' is an Angular component, then verify that it is part of this module. <br>
> 2. If 'your-tag-name' is a Web Component then add 'CUSTOM_ELEMENTS_SCHEMA' to the '@NgModule.schemas'  <br>

Did you spot that `CUSTOM_ELEMENTS_SCHEMA` ? Have you ever tried it?
Let's try.

```
ng g c some
```
This will create SomeComponent and also add it to the declaration array in `app.module`.
Use the component in the app component template.

```html
<app-some></app-some>
```

If we remove the declaration you will get the error mentioned. But what happens when we provide the `CUSTOM_ELEMENTS_SCHEMA` schema ?
No more errors 🙌 But also no content coming from the component. Well we only told Angular it should not complain about elements unknown to Angular. That's a good thing we can do — otherwise yu could not use any web component.

This also means you need to be careful when you use that schema — Angular won't tell you anymore if some component is not found.

There is a second schema called `NO_ERRORS_SCHEMA`. That's a little bit more telling but what's the difference?
We can ask the [documentation](https://angular.io/api/core/NO_ERRORS_SCHEMA).

> Defines a schema that allows any property on any element.

vs [CUSTOM_ELEMENTS_SCHEMA](https://angular.io/api/core/CUSTOM_ELEMENTS_SCHEMA)
> Defines a schema that allows an NgModule to contain the following:
> Non-Angular elements named with dash case (-).
> Element properties named with dash case (-). Dash case is the naming convention for custom elements.


Ah! `NO_ERRORS_SCHEMA` suppresses all errors while `CUSTOM_ELEMENTS_SCHEMA` is only holding back Angular for components that match Angular Elements.
We can quickly test it by using an element without dashes (while keeping the CUSTOM_ELEMENTS_SCHEMA)

```
<someCoolTagName></someCoolTagName>
```
You get an error with the following message:
> To allow any element add 'NO_ERRORS_SCHEMA' to the '@NgModule.schemas'

We can fix it with `NO_ERRORS_SCHEMA` instead of `CUSTOM_ELEMENTS_SCHEMA`.

And for what?
1. To allow Web Components (Custom Element are part of the standard)
2. You can use it for your tests to ignore elements instead of importing or mocking them.

## Completed
Yes we successfully worked through those parts of a module:

+ Module 1: Providers
+ Module 2: Declarations
+ Module 3: Imports/Exports
+ Module 4: EntryComponents
+ Module 5: Bootstrap
+ Module 6: Schema

We are ready for the first challenge!

## Challenge
Continue with [Chapter 01 - Modules & Injection (Challenge)](../challenges/01-modules.md)

## Resources
+ [Great post by Cyrille Tuzi about Modules in general](https://medium.com/@cyrilletuzi/understanding-angular-modules-ngmodule-and-their-scopes-81e4ed6f7407)


### Injection
We covered Injection partially already in _Module 1: Providers_.

The Injection Hierarchy is a difficult topic and I suggest you read dedicated resources about it to understand it beyond providing values occasionally. The official Angular Docs are great on the topic:

+ [Hierarchical Dependency Injection](https://angular.io/guide/hierarchical-dependency-injection)
+ [ngModule FAQ](https://angular.io/guide/ngmodule-faq)

And there are some interesting deep dives from the community too.

+ [Angular Dependency Injection and Tree Shakeable Tokens](https://blog.angularindepth.com/angular-dependency-injection-and-tree-shakeable-tokens-4588a8f70d5d)
+ [A curios case of the host decorator and element injectors in angular](https://blog.angularindepth.com/a-curios-case-of-the-host-decorator-and-element-injectors-in-angular-582562abcf0a)
+ [Transclusion Injection and Procrastination](https://blog.angularindepth.com/transclusion-injection-and-procrastination-8e1581c7a34e)
+ [Self or optional host the visual guide to angular di decorators](https://medium.com/frontend-coach/self-or-optional-host-the-visual-guide-to-angular-di-decorators-73fbbb5c8658)
+ [Better code organization with angular di multi option](https://netbasal.com/better-code-organization-with-angular-di-multi-option-31f691918655)
