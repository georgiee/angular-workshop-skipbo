# Angular Workshop
Level Intermediate/Advanced.
You should have worked through Angular's excellent [Tour of Heroes](https://angular.io/tutorial) and some experience in working with Angular already.

We will handle the following topics in this workshop while creating a card game called SkipBo.

You will find many things thta you can use in your daily enterprise business while working on an actual game. This should be more fun and less abstract than working on another TodoList or an imaginary product.

---
## RxJs

## Animation
A game but also any other UI without animations is boring and not engaging. This being said it's still one of the most ignored aspects of web programming. Let's change this and let's look at Angular Animations to create some nice animations in our game.

+ Router Transition
+ Make flipping cards triggered by a state
+ Staggering Animations over multiple items. We will use this to create appearing animations of our cards in a group.


## Routing
1. How to lazy load and even preload feature modules.
2. Analyze results with webpack-bundle-analyzer.
3. Let us protect routes from being opened if a condition (like game running for examples) is not fulfilled.
4. Prefetch data so our component don't need to load data by itself.

## Modules

Compare two fresh CLI project, one with Ivy (`ng new test-ivy --experimental-ivy`)

Talk about export, import, bootstrap

## Testing
Some preparations about component testing and how to do it differently than Angular CLI tells you. Dumb components.

We will talk about micro and macros tasks to make you a confident tester who will finally know what's behind fixture.detectChanges, tick, flush & friends.

## Tips
### Opensource
Create a folder `~/opensource` and whenever you see an interesting project, don't stop with a start on github. Clone it and quickly browse through it. When you are working on a daily project, remember that project and if it fits in the overall complex browse through it again for inspiration about structure & code techniques.

Angular Material is for examples my goto project to look for ideas around RxJS & Testing.