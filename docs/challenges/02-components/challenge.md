# Components

## Your challenge
You are in this branch to start: `workshop/02-components-start`

I'm sorry, your project is still blank but I promise we will change this soon.

![Blank Project](blank-project.png)

Ensure your Application is still running under 4200 and that your specs are running too.

You will create several components in this challenge
+ `pages/gameplay` to hold our game stage
+ a card image
+ a card
+ and a pile

## Components all over
Create those component shells with the Angular CLI:
<details>
  <summary>Hint</summary>
  (`ng g c games/...`)
</details>

+ game/pages/gameplay
+ game/components/card
+ game/components/card-face
+ game/components/card-pile


All those components have been automatically added to the `GameModule` (take a look, field `declarations`). This means they can use each other — as all of them are inside the module. But if you want to use it outside you have to put the component manually in the export array.

## Outside vs. Inside
Let's try to use one of the component first outside then inside the module to see the differences. Add the following markup to your `app.component.html` (which is empty at the moment).

```html
<skipbo-gameplay></skipbo-gameplay>
```

Save it and watch what happens in your browser console.

> Uncaught Error: Template parse errors:
'skipbo-gameplay' is not a known element:<br>
> 1. If 'skipbo-gameplay' is an Angular component, then verify that it is part of this module.<br>
> 2. If 'skipbo-gameplay' is a Web Component then add 'CUSTOM_ELEMENTS_SCHEMA' to the '@NgModule.schemas' of this component to suppress this message. ("[ERROR ->]<skipbo-gameplay></skipbo-gameplay>")

Well that's what I said. It won't work. We can fix it in two ways — just like the error message tells us.

## Pretend to be a real component
Put the following line into your `app.module.ts`.

```typescript
@NgModule({
	//...
	schemas: [CUSTOM_ELEMENTS_SCHEMA]
})
export class AppModule { }
```

The error magically disappears, but you won't see any content from the the Gameplay Component you wanted to mount. We just told Angular that it should ignore all tags with dashes case (see [reference](https://angular.io/api/core/CUSTOM_ELEMENTS_SCHEMA)) which is required to use self-bootstrapping Angular Elements or any other Web Component. We are not using a Web Component here, so this is clearly not our solution.

Remember:
> You have to use CUSTOM_ELEMENTS_SCHEMA to suppress Angular errors regarding unknown html tags when using Web Components & Angular Elements.

## Export
Remove the schema entry from the application module so you see the error in the browser console again. Now do the following. Open file `game.module.ts` and provide `GameplayComponent` in the exports array:

```typescript
@NgModule({
  ///...
  exports: [
    GameplayComponent
  ]
})
export class GameModule { }

```

Did it work? Indeed, you should see this 💪
![gameplay-works](gameplay-works.png)

Why did this work? You just put the `GameplayComponent` in the export list of the GameModule. The module itself is imported in the `ApplicationModule`. By doing to you allowed any component defined in the ApplicationModule (like the ApplicationComponent) to use the `GameplayComponent` with its tag `<skipbo-gameplay></skipbo-gameplay>`.

## Private Components
Let's try to use a component inside that GameplayComponent. Put the following template content into `gameplay.component.html`

```html
<skipbo-card-pile></skipbo-card-pile>
<skipbo-card-pile></skipbo-card-pile>
```

Does it work ? Yes again!
![card-pile-works](card-pile-works.png)

We did not export `CardPileComponent` and still it works. That's because both components `GameplayComponent` and `CardPileComponent` are declared (`declarations:[]`) in the same module (`GameModule`). It's kind of a private/protected component limited to the scope of the module.

We will continue with a slightly updated branch to save you from writing some scss, html and also some code. Prepare to switch the branch.

---

## Cards & Piles

⏭  Switch to branch `workshop/02-components-progress-01` (_mandatory_) to continue.
I added some markup and scss to save you time and to display real cards finally.

![](two-piles.png)

### Start the game
The `GameService` will provide the necessary deck list data — but you have to start the game first by calling `start` on the GameService. Do this in the given `ngOnInit` callback in the `GameplayComponent`. Your screen should look like this if it's working and you can continue.

![](started.png)

### Fix the bug
There is a bug when you interact with the build and back button. Look at the height of the piles — they are supposed to change when growing or shrinking! Can you identify what's wrong and can you fix it ?

> **Tip:** Watch for two failing specs

<details>
  <summary>Hint</summary>
  someArray.pop() and someArray.push are mutating operations.
  If you don't want to mutate a source array you have to create a new one
  and insert the elements. Use ES6 [spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) or `array.concat`
</details>

Do not continue before fixing the bug.

## Inject Parent Example
Catch up branch to begin with: `workshop/02-components-progress-02`

A good programmer creates only loosely coupled systems. You learn this for a good reason as it makes your code more maintainable. The main method of doing so in React for example is by passing data, functions, references and even children with properties (which equals the composition pattern).

Angular is a different framework and we have different ways of doing so. Services, Events and Injection in general. You can not only inject Services but Components and Directives from your DOM tree. Let's try a quick example with no real usage yet. Throw in the following constructor into the `CardComponent`:

```typescript
constructor(
    @Host() @Optional() pile: CardPileComponent
  ) {

    if (pile) {
      console.log('Card says: My parent pile is', pile);
    } else {
      console.log('Card says: I Have no parent pile');
    }
  }
```

When you watch your console you will see plenty of those messages:
> Card says: My parent pile is **CardPileComponent**<br>
> Card says: My parent pile is **CardPileComponent**<br>
> Card says: My parent pile is **CardPileComponent**<br>
> Card says: My parent pile is **CardPileComponent**<br>
> ...

So that worked.  Now temporarily replace the whole gameplay template with the following html.

```html
<skipbo-card face="-1"></skipbo-card>
```

The log shrinks to this:

> Card says: I have no parent pile

That's exactly what should happen. There is no parent `CardPileComponent` anymore. The card can really detect if there is a parent pile.

That host injecting mechanic is kind of a tighter coupling but in reality not different to passing in properties — so you can absolutely use it when the classic methods are not working or if you would create too much boilercode by introducing services only for a single purpose that you could handle with such a host injection.  That being said: In most cases you shouldn't need such mechanics as you can simply pass in the information a card is interested in from the parent pile with `@Inputs`.

In other scenarios where you would call methods or bind events oof the parent component you could inject a service in both components to work as the message bus. Angular for example uses the `@Host()` pattern heavily in the forms package to get information about how the controls are grouped and nested.

You are done with this challenge. Congratulations 🏅🌟

----

You reached branch `workshop/02-components-end` by completing this lesson.


## Branches
+ workshop/02-components-start
+ workshop/02-components-progress-01 (_mandatory_)
+ workshop/02-components-progress-02 (_catch up_)
+ workshop/02-components-end