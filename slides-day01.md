# [fit] Angular Workshop
### _Skip-Bo Edition_

---

![](images/preview.jpg)

---
# [fit] What's in the box

---
Everything is here:
[github.com/georgiee/angular-workshop-skipbo](https://github.com/georgiee/angular-workshop-skipbo)

---
# [fit] Theory

---
# [fit] Coding
# [fit] Challenges

^ Total of 6 Hours, Progress Branches, Starting Branch

---
# [fit] Schedule

---
[.build-lists: true]

# [fit] Schedule - Day 1
+ Chapter 01 — Modules & Injection
+ Chapter 02 — Components
+ Chapter 03 — Routing

---
# [fit] Schedule - Day 2
+ Chapter 04 — RxJS
+ Chapter 05 — Testing
+ Chapter 06 — Animation

---
# [fit] Day 1

---
### __Chapter 01__
# [fit] Modules
#### &
# [fit] Injection

---
> What would you have ripped out of Angular if you had one breaking change for free ?

---
> NgModules
-- Igor Minar, AngularConnect 2018

---

```typescript
export interface NgModule {
  providers?
  declarations?
  imports?
  exports?
  entryComponents?
  bootstrap?
  schemas?
}
```
[.footer: Chapter 01: Modules & Injection]

---

[.code-highlight: 2]
```typescript
export interface NgModule {
  providers?
  declarations?
  imports?
  exports?
  entryComponents?
  bootstrap?
  schemas?
}
```
[.footer: Chapter 01: Modules & Injection]

---


[.code-highlight: 3]
```typescript
export interface NgModule {
  providers?
  declarations?
  imports?
  exports?
  entryComponents?
  bootstrap?
  schemas?
}
```
[.footer: Chapter 01: Modules & Injection]

---



[.code-highlight: 4]
```typescript
export interface NgModule {
  providers?
  declarations?
  imports?
  exports?
  entryComponents?
  bootstrap?
  schemas?
}
```
[.footer: Chapter 01: Modules & Injection]

---



[.code-highlight: 5]
```typescript
export interface NgModule {
  providers?
  declarations?
  imports?
  exports?
  entryComponents?
  bootstrap?
  schemas?
}
```
[.footer: Chapter 01: Modules & Injection]

---




[.code-highlight: 6]
```typescript
export interface NgModule {
  providers?
  declarations?
  imports?
  exports?
  entryComponents?
  bootstrap?
  schemas?
}
```
[.footer: Chapter 01: Modules & Injection]

---


[.code-highlight: 7]
```typescript
export interface NgModule {
  providers?
  declarations?
  imports?
  exports?
  entryComponents?
  bootstrap?
  schemas?
}
```
[.footer: Chapter 01: Modules & Injection]

---


[.code-highlight: 8]
```typescript
export interface NgModule {
  providers?
  declarations?
  imports?
  exports?
  entryComponents?
  bootstrap?
  schemas?
}
```
[.footer: Chapter 01: Modules & Injection]

---

# [fit] Injection

---
# [fit] Challenge
Create & Break the GameService

^ Go to your checked our workshop folder and open file docs/challenges/01-module/challenge.md
^ Or go to the github repository.


---

## Resources
+ [Understanding Angular Modules (Cyrille Tuzi)](https://medium.com/@cyrilletuzi/understanding-angular-modules-ngmodule-and-their-scopes-81e4ed6f7407)
+ [Angular Dependency Injection tree (Alexey Zuev)](https://blog.angularindepth.com/angular-dependency-injection-and-tree-shakeable-tokens-4588a8f70d5d)
+ [A curious case of the @Host decorator (Max, ngWizard)](https://blog.angularindepth.com/a-curios-case-of-the-host-decorator-and-element-injectors-in-angular-582562abcf0a)
+ [Transclusion, Injection and Procrastination (Uri Shaked)](https://blog.angularindepth.com/transclusion-injection-and-procrastination-8e1581c7a34e)
+ [@Self or @Optional @Host?  (Tomek Sułkowski)](https://medium.com/frontend-coach/self-or-optional-host-the-visual-guide-to-angular-di-decorators-73fbbb5c8658)
+ [Angular DI Multi Option (Netanel Basal)](https://netbasal.com/better-code-organization-with-angular-di-multi-option-31f691918655)




---
### __Chapter 02__
# [fit] Components

---
# [fit] Theory
+ Directive vs. Components
+ Template Magic
+ ChangeDetection
+ Presentational vs. Smart Component

---
# [fit] Challenge
Build our first components: CardFace, Card, CardPile and our gaming stage

---
### __Chapter 03__
# [fit] Routing

---
# [fit] Theory
+ Directive vs. Components
+ Template Magic
+ ChangeDetection
+ Presentational vs. Smart Component

---
# [fit] Challenge
Map routes to our 5 pages, introduce lazy loading and routing guards
