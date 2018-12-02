# Routing
You should have used <router-outlet></router-outlet>
with different routes & components so far.

```html
<router-outlet></router-outlet>
```

This tag provides the anchor point to render any route and its component — they are being inserted after that point in the DOM. Any route that defines child components need to use a distinct router outlet. Don't forget about it — otherwise nothing will render and the application won't throw any error.

## Parameters
You can define a parameter:

```typescript
{ path: 'hero/:id', component: HeroDetailComponent }
```

and get hold of the current route and it's (changing) parameters by injecting ActivatedRoute.

```typescript
constructor(private route: ActivatedRoute){
  route.paramMap.pipe(
      tap(path => console.log('path--> ', path))
    ).subscribe();
}
```

## Lazy Load
There is no easy way of **manually** loading modules (but you can make it work — stay tuned). The lazy load a module you have to follow a specific syntax in your route in the router configuration.

```typescript
{
  path: 'lazy-module',
  loadChildren: './lazy-load/lazy-load.module#LazyLoadModule'
},
```

See the `loadChildren` with the value of? './lazy-load/lazy-load.module#LazyLoadModule'. That's the key to lazy loading. Angular will do the rest for you!

The module being loaded usually starts with an empty route. This way you give the consumer of your module to decide where module and all of its route should be mounted. Otherwise it would be hard coded in your module.

Of course you can start with any route you want if that's intended. The following module would be a valid module that you can lazy load.

```typescript

const routes: Routes = [{
  path: '',
  component: HelloComponent
}];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class LazyLoadRoutingModule { }

```

When you now hit the route you will see a file in the format filename-module-name so in this case `lazy-load-lazy-load-module.js`. That's the complete factory to use the module. Once it's loaded you get access to all components and services.

Lazy loaded modules create their own branch on the Dependency Injection tree and other parts of the application won't get access to provided values.

## Manual Loading a Module (modern)
Can we load a module manually? We would need to tell Angular somehow that we want to load a specific module later and that a factory needs to be compiled but not inlined.

Somehow the relative url we just used inside the router must be involved.

```typescript
./lazy-load/lazy-load.module#LazyLoadModule
```

You can use that string (but without the hash followed by the module name ☝️) in your `angular.json` config under the key `lazyModules`. The schema says about it:

> List of additional NgModule files that will be lazy loaded. Lazy router modules will be discovered automatically.


```json
"architect": {
  "build": {
    "lazyModules": [
      "src/app/lazy-load/lazy-load.module"
    ]
},
```

Remove the router entry referring to a lazy loaded module (if any) and place the lazy loading path into the `angular.json` file as just described. After restarting the app you should should see your module with a slightly different file name.

```
chunk {src-app-lazy-load-lazy-load-module}
src-app-lazy-load-lazy-load-module.js,
src-app-lazy-load-lazy-load-module.js.map
(src-app-lazy-load-lazy-load-module) 9.9 kB  [rendered]

```

You just told Angular CLI about the existence of the module so it can be prepared as a chunk to load later — just like a router would expect it. But this time no router is involved.

Let's load the prepared factory manually.


```typescript
// tag where we gonna inject the lazyload module and his default compononent "entry"
  @ViewChild('container', { read: ViewContainerRef }) viewRef: ViewContainerRef;

  constructor(
    private loader:     NgModuleFactoryLoader,
    private injector:   Injector,
    private moduleRef:  NgModuleRef<any>) {
  }

  ngOnInit(): void {
    this.load();
  }

  load() {
    const path = 'src/app/lazy-load/lazy-load.module#LazyLoadModule';

    this.loader.load(path).then((moduleFactory: NgModuleFactory<any>) => {
     // myEntryComponent is our property, not part of the framework
      const entryComponent = (<any>moduleFactory.moduleType).myEntryComponent;
      const moduleRef = moduleFactory.create(this.injector);
      const compFactory = moduleRef.componentFactoryResolver.resolveComponentFactory(entryComponent);

      this.viewRef.createComponent(compFactory);
    });
  }
```

We use the same module path as in the `angular.json` file but in addition we add the module name we want to load. The `NgModuleFactoryLoader` will take care of creating the correct url (by convention) to load the factory file.

Once it's loaded we can get the reference to the component we want to instantiate. Use the custom static property `myEntryComponent`. That way we prevent to include the component reference which would get included in the application module otherwise.

After we get our hands on the component factory we can create an instance inside our container.

## Manual Load Module (legacy)

Before Angular CLI provided us the `lazyModules` field we had to use internals of the framework to force Angular into pre-process a module. You used the token `ANALYZE_FOR_ENTRY_COMPONENTS` to provide any number of routes to modules. Those would get prepared but not available as a route.

```typescript
  {provide: ANALYZE_FOR_ENTRY_COMPONENTS, multi: true, useValue: routes},
```

## Guards

You can also guard routes — like protecting your admin area from loading without being authenticated  or ensure data being loaded before activating a route.

You have the following guard interfaces you can implement.

+ `CanActivate` to mediate navigation to a route.
+ `CanActivateChild` to mediate navigation to a child route.
+ `CanDeactivate` to mediate navigation away from the current route.
+ `Resolve` to perform route data retrieval before route activation.
+ `CanLoad` to mediate navigation to a feature module loaded asynchronously.

You can either return a static value or an observable (that must complete).

Example of a guard — you can put one at any route and implement multiple guards in one class.

1. Create a guard

```bash
ng g guard guarding/general
```


2. You get the following file scaffolded

```typescript
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class GeneralGuard implements CanActivate {
  canActivate(
    next: ActivatedRouteSnapshot,
    state: RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean {
    return true;
  }
}

```

Import it and assign it to a guard (here `canActivate`).

```typescript
const routes: Routes = [
  {
    path: 'guarding', redirectTo: 'guarding/one', pathMatch: 'full'},
  {
    path: 'guarding/one',
    component: OneComponent,
    canActivate: [GeneralGuard]
  },
```

If you navigate to `guarding/one` everything is fine but if you `return false` in the guard you will be redirected to the root.


## Resolver
Resolver are close relatives to guards. They are also involved in routing and ensure that data required for a route is guaranteed to be available when a route is activated.

Now let's add a resolve guard to fetch some data before accessing the route.


```typescript
{
    path: 'guarding/one',
    component: OneComponent,
    canActivate: [GeneralGuard],
    resolve: [GeneralGuard]
},

```

By specifying `resolve: [GeneralGuard]` you register the existing Guard. That Guard needs to implement the interface `Resolve<T>` correctly. T is the return value of the resolver.

Let's implement the require resolve method in the guard. This time we return an observable instead of a static value.

```typescript
resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): any | Observable<any> | Promise<any[]> {
    return of({
      hello: 'hello',
      items: [1, 2, 3, 6]
    }).pipe(delay(1000));
  }
```

You can get hold of the resolved data inside your routed component:

```typescript
constructor(private route: ActivatedRoute) { }

ngOnInit() {
  console.log('ngOnInit', this.route.snapshot.data);
}
```

If you now navigate to the url you will notice a delay of 1 second and then the data being loaded (for a simulated duration of 1s) is resolved and the component is loaded.

To display an application wide loading spinner you would implement a spinner component in the application template and control it through a service inside your guards by turning it on or off.

## Challenge
Continue with [Chapter 03 - Routing (Challenge)](../challenges/03-routing.md)