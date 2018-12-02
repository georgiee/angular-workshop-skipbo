# Modules & Injection
Let me begin with a question and an answer:

> Question: What would you have ripped out of Angular if you had one breaking change for free. ?
> Answer: NgModules

That question got asked at the Angular Panel at the end of the two day Angular Connect conference in London.

Igor Minar himself answered this. Not in this shortness nor in this determination but he talked about the modules thing being unfortunate.

But it's learnable. I expect everybody to have already a basic knowledge to let's start with a quick overview by looking at all fields of a module.

## Module Recap
Here the interface.

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

That's it. Nothing more and we now look at each property.

### providers
Before Angular 6 you would put your services here, nowadays it's fine to rely on `providedIn: 'root'` inside the `@Injectable` decorator. That way you don't have to import the service anywhere, the DI dependency inside your consumers ensure that the service will be loaded and Angular detects that it should be globally provided. The service will be part of the root injector provided by `AppModule`.

You only use the provider array to override existing services
or provide other values (like custom factories, tokens, static values) in your dependency injection tree.

Services are tree-shakable by not having explicit references in the module.

## declarations
You list here components, pipes and directive. Those are the things related to the DOM/template. You need to tell Angular about their existence as their presence in the template would not include the classes itself.
You already know the number one rule: You can declare a class only once. If you have problems organizing your code you might run into this problem often.

You usually create Feature Modules, UI Modules or Shared Modules and improt them often while the declaration of elements are done once in those modules.

## exports
That's tightly bound to declarations and it can be confusing because you only list here declarations you want to use outside of the module. Never services.

**Example 1:** You have a UI module to share components you will list ALL components in declarations as well as in exports.

**Example 2:** If you have a feature module where the components are only used internally you don't want to expose them so you never list them in exports.

A typical UI shared module will look like the following code snippet:

```typescript
const myComponentList = [
  ComponentA, ComponentB, ComponentC
];

@NgModule({
  declarations: [
    ...myComponentList
  ],
  exports: [
    ...myComponentList
  ],

  //...
})
export class UISharedModule { }
```

You can also export modules you imported to provide their functionality to every consumer of your module. The CLI does this if you create a module with routing support;

```bash
$: ng create m ThisModuleWillDoRouting --routing
```

The accompanying routing module (`this-module-will-do-routing-routing.module.ts`) contains the following code to ensure that the actual module can use routing functionality (like routerLink) while providing the same functionality to anyone importing the module itself.

```
@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
```

## imports
That's the place to import other modules to use their components/directives/pipes (only if they are listed in its `exports: []` list). If you have decided to bind a service to a module (by explicitly list the service in their provider array) you can provide access to the service also by importing the module.

## entryComponents
Any component you create without using its template tag in a template (that's called `declarative`) is created imperatively. That's the case for components you use in routers.

```typescript
const routes =[
  {route: 'some-path', component: MyComponent}
]l
```
MyComponent will be displayed without using the tag `<my-component></my-component>` in any of your templates. In most cases and Angular will automatically add those components to the list of entryComponents. That's why you usually not touch the `entryComponents` list.

Technically we need to tell Angular about it so it can create a component factory which is not automatically generated if a component is not used anywhere.

**Example:**<br>
You can use that factory yourself: Create a simple component, put it in the `declarations` list and the `entryComponents`. _Don't use its html tag (selector) nor put it in any router._

Go into any other component and provide the following additions to the class and template.

Template, create a place to create stuff. We use an ng-container to prevent Angular from creating any element in the DOM but you could use actual html elements too.

1. Minimal Unicorn component:

```typescript
@Component({ template: '🦄' })
export class UnicornComponent { }
```

2. Template in any other component to show some unicorns:

```typescript
<ng-container #unicorns></ng-container>
```

Reference the container and dynamically create the unicorns

```typescript
@ViewChild('unicorns', {read: ViewContainerRef}) unicornsContainer: ViewContainerRef;
constructor(
    private componentFactoryResolver: ComponentFactoryResolver
) {}
ngAfterViewInit() {
    const unicornComponent = this.componentFactoryResolver.resolveComponentFactory(UnicornComponent);
    this.unicornsContainer.createComponent(unicornComponent);
}
```

This won't run any change detection inside the generated components (because you have not created any input bindings).

### bootstrap
From the Angular Docs:

> Among other things, the bootstrapping process creates the component(s) listed in the bootstrap array and inserts each one into the browser DOM.

Usually one is fine.

By the way you can access all root elements with `getAllAngularRootElements()` in the console

### schemas
Place a random tag name in your main application template you find in Angular.
<not-existing></not-existing>

You will get an error:
> 'not-existing' is not a known element:

Now place this in your app module:
```
schemas: [
  CUSTOM_ELEMENTS_SCHEMA
]
```
This will tell Angular to ignore elements not known to the framework. That way you can inser [custom elements](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements). The schema is only valid in the current module. If your component is declared in another module you have to unlock the schema in its module.


## Injection
Injection is a beautiful part of Angular and supports your engineering efforts by providing a clear way of implement features by composition.

You have injected services before I guess. But did you also inject other components, custom tokens, abstract classes to derive actual different classes. There is a lot you can do.

The Injection Hierarchy is a difficult topic and I suggest you read dedicated resources about it.

The official Angular Docs are great on the topic:

+ [hierarchical-dependency-injection](https://angular.io/guide/hierarchical-dependency-injection)
+ [ngmodule-faq](https://angular.io/guide/ngmodule-faq)

And there are some interesting deep dives from the community too.

+ [angular-dependency-injection-and-tree-shakeable-tokens](https://blog.angularindepth.com/angular-dependency-injection-and-tree-shakeable-tokens-4588a8f70d5d)
+ [a-curios-case-of-the-host-decorator-and-element-injectors-in-angular](https://blog.angularindepth.com/a-curios-case-of-the-host-decorator-and-element-injectors-in-angular-582562abcf0a)
+ [transclusion-injection-and-procrastination](https://blog.angularindepth.com/transclusion-injection-and-procrastination-8e1581c7a34e)
+ [self-or-optional-host-the-visual-guide-to-angular-di-decorators](https://medium.com/frontend-coach/self-or-optional-host-the-visual-guide-to-angular-di-decorators-73fbbb5c8658)
+ [better-code-organization-with-angular-di-multi-option](https://netbasal.com/better-code-organization-with-angular-di-multi-option-31f691918655)

We don't look at details instead I recommend reading the articles and check if you know about the following things

+ Injection Hierarchy
+ @Host @Optional @Self @Skip-Self
+ InjectionTokens()
+ multi providers

Injector Types:
+ element injectors (dom elements, ElementDef) `ng.probe($0).injector.elDef.element`
+ component injector (components)
+ module injectors
+ merge injectors (module/component), `ng.probe($0).injector`


## Resource
+ [Great post by Cyrille Tuzi](https://medium.com/@cyrilletuzi/understanding-angular-modules-ngmodule-and-their-scopes-81e4ed6f7407)
+ [Before ngAfterViewInit](https://blog.angularindepth.com/here-is-how-to-get-viewcontainerref-before-viewchild-query-is-evaluated-f649e51315fb)

## Challenge
Continue with [Chapter 01 - Modules & Injection (Challenge)](../challenges/01-modules.md)