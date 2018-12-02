# Challenge: Modules & Injection
Start with branch `workshop/01-modules-service-start`

## Your challenge
You start with the following blank screen. You will create a module, two services, tinker around with the Injection and reflect about how you make services accessible.

![Blank Project](blank-project.png)

---

## Task 1: Create & Implement
Use the CLI (`ng generate module`, `ng generate service` etc) to create the following parts of our Application. You should use the given string as the input for the generator. The CLI will create all missing subfolders for you.

**Your Task:**<br>
+ Create a new module `game`
+ Create a new service `game/services/game`
+ Create a new service `game/services/player`
+ Import the `GameModule` into your application module.

Now run the `npm run test` to start the unit tests. You will see plenty of failing tests.

![](failing-specs.png)

Fix all tests by providing the expected methods and properties. You don't have to write any logic is all about providing a work shell we can use later.

If you have to return a value double check the tests as they will tell you what to return.

> ⏱ Start Developing now and come back after ⏱

You completed the last task test driven. There are failing tests you have to turn green by implementing them. It's called TDD,
test-driven development and ensures that every bit you implement is covered by a test. I love that programming style and you will get some more opportunities soon.

## Task 2: Inject, Break & Fix
Catch up with branch `workshop/01-modules-service-progress-01`
Let's focus on the `GameService`.

### Inject
Inject `GameService` into the Application Component (`app.component.ts`) and try to output the content of the deck in the console with the following command.

```typescript
console.log('deck: ', this.gameService.deck);
```

### Break it
Have you noticed that little decorator on top of the your two services?

```typescript
@Injectable({
  providedIn: 'root'
})
export class GameService...
```

**Your Task:**<br>
Change it to the following reduced version — don't leave an empty object, remove it completely.

```typescript
@Injectable()
export class GameService...
```

You should see your shell turn red and also your browser will show similar errors.

> Error: StaticInjectorError[GameService]:<br>
>      NullInjectorError: No provider for GameService!

Remember, we talked about `providedIn: 'root'` and how a service will be magically included in the root injector? How can we fix it without reverting to the providedIn default ? Try the following.

### Fix it

Import the `GameService` explicitly into the GameModule you created and imported into the `ApplicationModule` earlier. Use the  `providers:` section. Does the frontend work again?

Prior to Angular 6 this was the standard way of registering a service. You always had to explicitly list your services in your modules. That prevented them from being tree-shaken as you reference them inside your `GameModule`. In most cases we don't want that and the global injector is just fine and more convenient. That being said, it's not black and white. You will definitely import services into modules especially in larger projects with lazy loading and the requirement to inject multiple instance of services — to prevent the sharing of the underlying resources.

In addition you will notice that your specs are not working at the moment. That's because Angular's TestBed also includes all root injectors by default. Your services aren't part of the root injector anymore. You would have to import your `GameModule` into your specs. In this case your own auto-generated `game.service.spec.ts` but also into the workshop specs I prepared `game-workshop.service.spec.ts` where I check for the methods & getter you should have created by now.

```typescript
TestBed.configureTestingModule({
  imports: [ GameModule ]
})
```

That's how you import other modules into your TestBed. That's okay to do, but you need to be aware what you are importing and you should override especially services with stubs otherwise you turn your unit tests into integrations tests because you are involving too many parties.

## Task 3: Question

> Why don't we need to put the service in the **exports** field in the module GameModule? We are now using it outside of the module. Didn't we talk about how exports provide everything from inside a module to another module ?

<details>
  <summary>Answer</summary>
  The field `exports:` is only for declarations (components, directive & pipes) and whole modules (which may export itself other declarations again). If you deal with services your only field to care about is `providers:` — if you don't use the default `provideIn` method.
</details>

## Completed
That's it. You completed the first challenge🏅 Was it too easy? I promise this will change 🤓☝️

You reached branch `workshop/01-modules-service-end` by completing the following tasks:

+ Task 1: Create & Implement ✅
+ Task 2: Inject, Break & Fix ✅
+ Task 3: Question ✅

Those are all branches involved in this challenge:

+ workshop/01-modules-service-start
+ workshop/01-modules-service-progress-01 (_catch up only_)
+ workshop/01-modules-service-end
