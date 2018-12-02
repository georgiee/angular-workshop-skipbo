# Challenge: Modules & Injection
Start with branch `workshop/01-modules-service-start`

## Your challenge
You start with the following blank screen. You will create a module, two services, tinker around with the Injection and reflect about how you make services accessible.

![Blank Project](blank-project.png)

## Task 1: Create & Implement
Use the CLI (`ng generate module`, `ng generate service` etc) to create the following parts of our Application.

**Your Task:**<br>
+ Create a new feature module `game`
+ Create a new service `game/services/game`
+ Create a new service `game/services/player`
+ Import the `GameModule` into your application module.

Now run the `npm run test` to start the unit tests. You will see plenty of failing tests, fix them by implementing them.

![](failing-specs.png)

*Watch out:☝️* <br> You may see `✔ 0 tests completed` on the first run. Just make a change and save one of your services.

If you have to return a value double check the tests as they will tell you what to return.

**Your Task:**<br>
Can you provide the required functions?
Continue when you have turned all tests green.

## 2. Inject
> Branch to catch up: `workshop/01-modules-service-progress-01`

Let's focus on the `GameService`.

**Your Task:**<br>
Inject it into the Application Component (`app.component.ts`) and try to output the content of the deck in the console with the following command.

```typescript
console.log('deck: ', this.gameService.deck);
```

## 3. Break it
Yes 2) was pretty easy. Let's continue. Have you noticed that little decorator on top of the your two services?

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

You should see your shell turned red and also your browser will show similar errors.

> Error: StaticInjectorError[GameService]:<br>
>      NullInjectorError: No provider for GameService!

Remember, we talked about `providedIn: 'root'` and how it's magically included in the root injector. But how can we fix it here without reverting to the providedIn default ? Try the following.

## 4. Fix it
**Your Task:**<br>
Import the `GameService` explicitly into the GameModule you created and imported into the `ApplicationModule` earlier. Use the  `providers:` section. Does the frontend work again?

Prior to Angular 6 this was the standard way of registering a service. You always had to explicitly list your services in your modules. That prevented them from being tree-shaken as you reference them inside your `GameModule`. In most cases we don't want that and the global injector is just fine and more convenient. That being said, it's not black and white. You will definitely import services into modules especially in larger projects with lazy loading and the requirement to inject multiple instance of services — to prevent the sharing of the underlying resources.

In addition you will notice that your specs are not working at the moment. That's because Angular's Testbed also includes all root injectors by default. Now you would have to import your `GameModule` into your specs. In this case your own auto-generated `game.service.spec.ts` but also into `game-workshop.service.spec.ts` where I check for the methods & getter you should have created by now.

```typescript
TestBed.configureTestingModule({
  imports: [ GameModule ]
})
```

That's how you import other modules into your TestBed. That's okay to do, but you need to be aware what you are importing and you should override especially services with stubs otherwise you turn your unit tests into integrations tests because you are involving too many parties.

## 5. Question

> Why don't wee need to put the service in the **exports** field in the module GameModule? We are now using it outside of the module. Didn't we talk about how exports provide everything from inside a module to another module ?

<details>
  <summary>Answer</summary>
  The field `exports:` is only for declarations (components, directive & pipes) and whole modules (which may export itself other declarations again). If you deal with services your only field to care about is `providers:` — if you don't use the default `provideIn` method.
</details>

## Finish
That's it. You completed the first challenge🏅<br>

----
You reached branch `workshop/01-modules-service-end` by completing this lesson.

## Branches of this challenge
+ workshop/01-modules-service-start
+ workshop/01-modules-service-progress-01 (_catch up only_)
+ workshop/01-modules-service-end
