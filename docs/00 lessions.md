# Lessons
Themenkomplexe: 
Tag 1: Module/Services, Components, Testing
Tag 2: Routing, Animations, RxJS

## Schedule
### Intro
Spielanleitung durchsprechen
Wörter etc.

### Module & Services
Topics: Igor, Module Recap (Decorator Fields), Feature Modules/ Architecture

**Übung**: Modul anlegen, Game Service anlegen

Game Service + Player Service and test method existence.
-> only small set, expand with next lesson
-> inject in application
-> force failure, remove provide in
-> revert
-> next example

### Components
Components Recap (Decorator Fields), Directive vs. Components,
Dumb vs Smart, Higher Order, Template Recap (references), Inject @Host

**Übung**: CardFace, Card (higher), CardPile + Service

### Testing
Configuration, Component with Abstract Test Wrapper, TDD or not.
Tests give you confidence that nothing breaks.

Excourse: Renovate Bot
Asyn Testing: fakeAsync & tick, ngZone
Change Detection & OnPush

Excurse: Micro vs Macro Tasks

**Übung**: TDD driven new component: Player oder Opponent ?

### Routing
Lazy Loading, Guards, 

**Übung**: Create new pages: Welcome, Instructions, GameOver
Create Guard for Playing

### RxJS
**Übung**: Extend GameService or use Router Streams.

### Animations
**Übung**: Flip Card, Router Transition

## Ablauf
Komplettes Projekt zeigen, dann zurück zum Start
und die Anwesenden das Musterprojekt auschecken lassen.

Das Projekt enthält Branches für alle Aufgaben.
Die Zwischenschritte dienen dem aufholen und starten mit dem gleichen Stand. Ich bekomme damit auch die Möglichkeit weitere Anpassungen durchzuführen.


## Ideas
1. [Routing] Lazy Loading
2. [Routing] Guards (Already Playing?)
3. [Component/Injection] Injecton vs @Host/@Optional Injection for Inter Component Communication
4. [Modules/Injection] Configuration with Injection Tokens
5. [Component] Dumb/Peresenbtional vs Smart Components
6. [Component] Architecture (Folder Structure)
7. [Component] Dyanmic Component Creation (entryComponent)
8. [RxJs] Cold vs Hot 
9. [RxJs] Destroy Component
10. [Testing] General (Abstract)
11. [Testing] RxJS example
12. [Animation] Card Flip
13. [Animation] Router
14. [CDK] Overview
15. [CDK] Drag & Drop
16. [CDK] Overlay


## Module
Start your specs with `npm test`. You should see two failing specs.


1. Create Game Module

```
ng g module Game
```

Import in ApplicationModule

2. Create Game Service

```
ng g service game/game
```
provide a hello function

```
hello() {
    return 'hello workshop!';
}
```
inject it into Application Component and use it in the template:
```
constructor(public gameService: GameService) {
	console.log(gameService.hello());
}
```

app.component.html
```
{{gameService.hello()}}
```

and test for the expected result in the service spec


3. Now Break it
(Also: Problems with existing specs when breaking)

Remove provideIn Option from the decorator
You should get an error now.

> Error: StaticInjectorError(AppModule)[AppComponent -> GameService]:
> Also your te

4. Fix it the clasic way
import game module

> **Question: **: Why don't wee need to put the service in the exports field — we ware using it outside of the module.



---

## Workshop Project
Folder Structure Konvention

Card + Card Face (Dumb vs Smart)
Card Flip
-> Animation
Card Pile (Stock, Deck, Discard, Building)

CDK Drag & Drop
Card Group
Pages (Start, Gameplay, Gameover, Instructions)
-> Lazy Loading, Guards, Animation

Automata: RxJs

## Lazy Loading
Create a lazy loaded module (`GameModule`) together with a first page (`GameCompomnent`). Load it with your application router. If you see the page it works. Use the CLI.

Challenges:
+ Create a production build and compare before and after.
+ Use webpack analyzer to see the difference

```
ng g module skipbo --routing
ng g c skipbo/skipbo
```

## Injection vs Services
Inter Component Communicatipn
@Host Component Injection

## Dumb/Presentational vs Smart
Card vs CardFace


