build-lists: true

![](../images/slides/intro.png)

---

+ [@deluxee](https://twitter.com/deluxee) (Twitter)
+ [@georgiee](https://github.com/georgiee/) (Github)
+ [Workshop Repository](https://github.com/georgiee/angular-workshop-skipbo)
+ [Workshop Project](https://github.com/georgiee/skipbo-angular)

---

# [fit] What's in the box?

---
Everything is here:
[github.com/georgiee/angular-workshop-skipbo](https://github.com/georgiee/angular-workshop-skipbo)

---
# [fit] 6 Chapters

---

# [fit] THEORY
# [fit] CHALLENGE
# [fit] RESULT

^ Mantra for our two days

^ That's how we work through all six chapters

---
# [fit] END RESULT

---

![](../images/preview.gif)

---
# [fit] Skip-Bo Core
### & Oscar 🐙

^ You will find a package `skipbo-core`.
^ That's the whole SkipBo game implemented and tested and it runs basically headless.
^ This will save you A LOT of time. Yes it's interesting implementing a set of game rules but you won't get many different challenges from it. That's why I did this for you!

---
# [fit] SCHEDULE

---
# _Day 2_
+ Chapter 04 — RxJS
+ Chapter 05 — Testing
+ Chapter 06 — Animation

^ That's tomorrow first

^ 04 RxJS: Debugging to RxJS in the wild & testing + AI 🐙 Autoplay V1

^ 05 Testing:  Async Code, Micro/Macro, ChangeDetection, Routing — Testing RxJS code from Oscar 🐙

^ 06 Animation: Basics to Router Animation and we flip many cards in the challenge

---

# _Day 1_
+ Chapter 01 — Modules
+ Chapter 02 — Components
+ Chapter 03 — Routing

^ That's what we are doing today

^ 01: Modules + GameService

^ 02: Components, Smart & Dumb Components, OnPush, Template and we will build our first components for the game

^ 03: Routing: Lazy Load, Guards, Resolver and we will mount many new pages and guard our game.

---
![original](../images/slides/bg.png)
# [fit] DAY 1

---
![original](../images/slides/bg.png)

#### __Chapter I__
# [fit] MODULES

---
> "What would you have ripped out of Angular if you had one breaking change for free ?"

---
> NgModules
-- Igor Minar, AngularConnect 2018

---
# [fit] THEORY

^ After motivating you by telling you that even the makers of Angular don't like modules

^ we are ready to start with the theory.

---
+ Providers
+ Declarations
+ Imports/Exports
+ EntryComponents
+ Bootstrap
+ Schema

^ we will go over all parts of a module

---

# Providers
+ provider array
+ `{ provide: Token, useClass: Value }`
+ InjectionToken
+ providedIn `'root'` or `OtherModule`
+ (Greeting Example)

^ provider array to provide services and values (config, static values)

^ provide a Token (string, InjectionToken, Any Reference) with a value, class or factory

^ nowadays you use provideIn `'root'` or `otherModules` by default

^ it's tree-shakable ( no references)

^ ReflectiveInjector example

---

# Declarations
+ components, pipes and directives
+ tell Angular about their existence — once!
+ SharedModules, FeatureModule, per Element Module

^ Things related to the DOM

^ Tell Angular about existence — but only once

^ With Modules you won't struggle to find a dedicated module for a declarations

---
# Imports/Exports
+ Export: What to use outside
+ Import: What to use inside
+ (ButtonModule Example)

^ Export can be Declarations & Modules (Reexport)

^ Import Modules only

---
# EntryComponents
+ For dynamic components
+ Routers do this this automatically for you
+ (Dynamic Example)

^ declarative (html) vs imperatively (dynamic, tag irrelevant)

^ You can still use dynamic components declarative as a tag

---
# Bootstrap
+ Define your app root component
+ There can be multiple app roots
+ (Example)

^ getAllAngularRootElements() to debug like `ng.probe`

---
# Schemas
+ CUSTOM_ELEMENTS_SCHEMA
+ `<my-component my-prop="lorem">`
+ NO_ERRORS_SCHEMA: anything
+ (Example)

^ Have you ever used this part of NgModules

^ CUSTOM_ELEMENTS_SCHEMA: dashes

^ NO_ERRORS_SCHEMA

---
# [fit] CHALLENGE

---
# _Your tasks_
+ Create our GameService
+ Provide expected interface (TDD)
+ Inject the GameService
+ Break the Injection and fix it
+ Answer a quick question


^ Go to your checked our workshop folder and open file docs/challenges/01-module/challenge.md

^ Or go to the github repository.

---
### [fit] RESULT

---
![original](../images/challenge-01-end.jpg)

---
![original](../images/slides/bg.png)

#### __Chapter II__
# [fit] COMPONENTS

---
# [fit] THEORY

---
+ Introduction
+ preserveWhitespaces
+ Selectors on existing elements
+ View Encapsulation
+ Smart & Dumb Components
+ OnPush
+ Template References

---
# Introduction

> A component controls a patch of screen called a view.

+ Meta, Component Class, Template
+ Directive vs. Components

^ cite from the official Angular

^ You will work a lot in components

^ Metadata (via Decorator), Component Class, Template

^ Components are Directives!

[.build-lists: false]

---
# preserveWhitespaces

```html
<button>1</button>
<button>2</button>
<button>3</button>
```
vs

```html
<button>1</button><button>2</button><button>3</button>
```

^ Show Live Example

---
# Selectors

```html
<my-custom-component></my-custom-component>
<some-component myDirective></some-component>
```

What about Components on existing Elements?

^ Show Live Example

^ Super Useful: Button & Input

^ template: '<ng-content></ng-content>'`

^ Never more than one component (Directive are ok) matched on this element.

---
# View Encapsulation
+ BEM anyone?
+ Scoped Styles in ShadowDOM standard 💪
+ Angular Native & Emulated
+ Native 0, ShadowDom 1
+ (Example)

^ BEM solution to the global namespace conflict

^ There is a standard for years with a solution: Shadow DOM with scoped styles

^ Angular always mimiced and supported the standard (ngcontent, slots, :host, shadow dom piercing ::deep)

^ V0 vs V1 ShadowDOM standard

^ Example to show DOM in browser

^ Don't use native, ShadowDom V0 is deprecated!

---
# Smart & Dumb
+ presentational not dumb!
+ presentational: rely on inputs only, no huge logic inside
+ smart: state, sometimes business logic, fetch data
+ (Example)

^ presentational is more polite

^ Other names: Fat & Skinny, Stateful & Pure, Screens & Components

^ presentational are: shareable, testable, understandable

^ smarts: focus on business logic and connection with the application

^ first presentation, then smart

---
# OnPush
+ Important Concept but CD makes it difficult
+ Rely only on inputs (presentational 👌)
+ Performance: [Website Example](https://hackernoon.com/angular-2-4-visualizing-change-detection-default-vs-onpush-3d7ed1f69f8e)
+ Still updating: UI events, Async Pipes
+ Live Example

^ Difficult because: Change Detection is involved

^ Promise to rely only on inputs

^ Nice website to show what it does

^ Still updating without input changesL UI, Async

^ Attention with object mutation

---
# Template References
+ exportAs
+ `#myComponent1='myAppOne'`
+ Live Example

^ Answers the question how to retrieve a reference to one of many directives

---
# [fit] CHALLENGE

^ Done with theory.

^ It's best to open the github page: Prepations -> Theory -> Challenge

^ Github because you can and anyone else finding it can work through theory & challenges.

^ always have the theory part or any other docs at hand

^ Challenge is taking your hand and tells a full story. I'm here only for further questions.

---
# _Your tasks_
+ Create Components
+ Use Gameplay Component
+ Use CardPile Component
+ Fix Bug in the CardPile
+ Inject parent component


---
### [fit] RESULT

---
![](../images/challenge-02-end.jpg)

---
![original](../images/slides/bg.png)

#### __Chapter III__
# [fit] ROUTING

---
# [fit] THEORY

---
+ Router Outlet
+ Lazy Load
+ Manual Loading a Module
+ Guards
+ Resolver

---
# Router Outlet
+ Anchor point for mounted components
+ Exposes reference `#myTemplateVar="outlet"`

^ Like ng-container, inserted as siblings

^ Use Directive Reference to get access to activatedRoute for exmaple

---
# Lazy Load
So easy to do.

```typescript
loadChildren: './lazy-load/lazy-load.module#LazyLoadModule'
```

+ Works by convention
+ Usually you use a empty route inside.
+ (Example)

---
# Manual Loading a Module
+ `lazyModules` key in angular.json
+ creates a chunk
+ load by convention url
+ Use it with `NgModuleFactoryLoader`
+ (Example)

---
# Guards
+ Protect your pages
+ Interfaces: CanActivate, CanDeactivate, ..
+ `ng g guard my-protector`
+ (Exmaple)

^ Help protecting Admin Area or from leaving or even load ing (CanLoad)

^ Implement through interfaces on a service like class

^ Also: CanDeactivate, CanActivateChild, CanDeactivate, CanLoad

^ Create with CLI

---
# Resolver
+ Ensure a component gets its data
+ Access via `route.snapshot.data`
+ Example

^ Can be put into the same service as guards. Different interface.

---
# [fit] CHALLENGE

---
# _Your tasks_

+ Route to the new pages
+ Make GameModule lazy load
+ Routing Guards: CanActivate
+ Routing Guards: CanDeactivate with prompt

---
### [fit] RESULT

---
![](../images/routing-03-end.jpg)

---
![original](../images/slides/bg.png)
### [fit] END 1
