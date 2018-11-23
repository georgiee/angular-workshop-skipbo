# Routing
You should have used <router-outlet></router-outlet>
with different routes & components so far.

```
<router-outlet></router-outlet>
```
provides the anchor point to render any route and its component. ANy route that define child components need to use a distinct router outlet. Don't forget about it otherwise nothing will render while the application won't throw an error.

## Parameters
You can define a parameter:
```
{ path: 'hero/:id', component: HeroDetailComponent }
```

and get hold of the current route and it's (changing) parameters by injecting ActivatedRoute.

```
constructor(private route: ActivatedRoute){
	route.paramMap.pipe(
      tap(path => console.log('path--> ', path))
    ).subscribe();
}
```


## Lazy Load
To lazy load a module you have to follow a specific syntax in the router configuration. There is no easy way of manually loading moduels (but there is!)

```
{
path: 'lazy-module',
loadChildren: './lazy-load/lazy-load.module#LazyLoadModule'
},
```

Where the module itself starts at an empty route:

```

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

That's it. When you now hit the route you will see a file in the format filename-module-name so in this case `lazy-load-lazy-load-module.js`. That's the complete factory to use the module. Once it's loaded you get access to all components and services.

Lazy loaded modules create their own branch on the Dependency Injection tree and other parts of the application won't get access to provided values.

## Manual Loading a Module (modern)
Can we load a module manually? We would need to tell Angular somehow that we want to load a specific module later and that a factory needs to be compiled but not inlined.

Somehow the relative url we just used inside the router must be involved.

```
./lazy-load/lazy-load.module#LazyLoadModule
```

you can use that string in your angular.json config, the schema says about it:

> List of additional NgModule files that will be lazy loaded. Lazy router modules with be discovered automatically.


```
"architect": {
"build": {
	//...
	"lazyModules": [
	  "src/app/lazy-load/lazy-load.module"
	]
},
```

Remove the router entry refering to that module (if any) and after restarting the app you should should see your module with a slightly different file name.

```
chunk {src-app-lazy-load-lazy-load-module} src-app-lazy-load-lazy-load-module.js, src-app-lazy-load-lazy-load-module.js.map (src-app-lazy-load-lazy-load-module) 9.9 kB  [rendered]
```

You just told Angular CLI about the existence of the module so it can be prepared. Let's load the prepared factory.


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

Once it's loaded we can get the reference to our comoponent we want to instantiate with our custom static property `myEntryComponent`. That way we prevent to include the component reference which would get included in the application module otherwise.

After we get our hands on the component factory we can create an instance inside our container.

## Manual Load Module (legacy)

Before Angular CLI provided us the `lazyModules` field we had to use internals of the framework to force Angular into pre-process a module. You used the token `ANALYZE_FOR_ENTRY_COMPONENTS` to provide any number of routes to modules. Those would get prepared but not available as a route.

```
	{provide: ANALYZE_FOR_ENTRY_COMPONENTS, multi: true, useValue: routes},
```

## Guards

You can also guard routes — like protecting your admin area from loading without being an authenticated admin or ensure data being loaded before activating a route.

You have the following guard interfaces you can implement:
+ CanActivate to mediate navigation to a route.
+ CanActivateChild to mediate navigation to a child route.
+ CanDeactivate to mediate navigation away from the current route.
+ Resolve to perform route data retrieval before route activation.
+ CanLoad to mediate navigation to a feature module loaded asynchronously.

You can either return a static value or an observable (that must complete).

Example of a guard — you can put one at any route and implement multiple guards in one class.

Create a guard

```
ng g guard guarding/general
```


You get the following file scaffolded
```
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

Import it and assign it to a guard (here canActivate).
```
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

Now let's add a resolve guard to fetch some data before accessing the route.


```
{
    path: 'guarding/one',
    component: OneComponent,
    canActivate: [GeneralGuard],
    resolve: [GeneralGuard]
},

```
By specifying `resolve: [GeneralGuard]` you register the existing Guard. That Guard needs to implement the interface `Resolve<T>` correctly. T is the return value of the resolver.

Let's implement the require resolve method in the guard. This time we return an observable instead of a static value.

```
resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): any | Observable<any> | Promise<any[]> {
    return of({
      hello: 'hello',
      items: [1, 2, 3, 6]
    }).pipe(delay(1000));
  }
```

You can get hold of the resolved data inside your routed component:

```
constructor(private route: ActivatedRoute) { }

ngOnInit() {
	console.log('ngOnInit', this.route.snapshot.data);
}
```

If you now navigate to the url you will notice a delay of 1 second and then the data being loaded (for a simulated duration of 1s) is resolved and the component is loaded.

To display an application wide loading spinner you would implement a spinner component in the application template and control it through a service inside your guards.

## Resolver
Resolver are close relatives to guards. They are also involved in routing and ensure that data required for a route is guaranteed to be available when a route is activated.

