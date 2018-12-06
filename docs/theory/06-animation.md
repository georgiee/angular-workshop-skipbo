# Theory: Animation

## Introduction
You can use animations for state transitions and router transitions.  Simple animation effects that can run standalone should be done with CSS only as you won't have any benefit plus a huge amount of boilerplate compared to the native CSS approach. Animations in Angular use the [Web Animations API](https://drafts.csswg.org/web-animations/) under the hood — just like the browser use it for CSS Animations.

This means: You can't animate what you can't animate in CSS.

+ Animation 1: Animation Basics
+ Animation 2: Appear & Disappear
+ Animation 3: Numeric Triggers
+ Animation 4: Disable
+ Animation 5: Router Animations
+ Animation 6: Animate Children

Excluded from the workshop
+ Animation 6: Keyframes
+ Animation 7: Complex Arrangements

## Animation 1: Animation Basics
Branch `animation/01`

### States & Transitions
Let's start with a simple usage of the animation system. Our goal is to give the component different backgrounds depending of the current state.
The state is reflected in a variables called `state` with values `on | off`. You can toggle between the values with the method `toggleState()`.

```typescript
import { HostBinding, Component, HostListener } from '@angular/core';
import { trigger, state, style } from '@angular/animations';

@Component({
  selector: 'app-my',
  template: 'hello animations',
  animations: [
    trigger('stateAnimation', [
      state('on', style({
        backgroundColor: 'red'
      })),

      state('off', style({
        backgroundColor: 'green'
      })),
    ])
  ]
})
export class MyComponent {
  @HostBinding('@stateAnimation')
  public state = 'on';

  @HostListener('click')
  toggleState() {
    this.state = this.state === 'on' ? 'off' : 'on';
  }
}

```

The result looks not so selling. There is a button and if you press it the background changes abruptly. There are no animations yet but you can easily add one.

Modify your animations array and append a transition method call.

```typescript
animations: [
  transition('on => off', [
     animate('1s')
  ]),
  transition('off => on', [
     animate('1s')
  ]),
]

```

The method call `transition('* => *', [animate('1s')])` consists of two parts. When should it happen (`* => *`) and what should happen. The _when_ part is a custom DSL and you use the values from your state (here _on_ or _off_ or the special value `*` for any) to tell what should happen for which change. In case of what should happen we only tell to animate it over 1 seconds. The animation call takes a plain number which gets interpreted as milliseconds our you pass in a string. That string can contain timing (like 1000, 1000ms, 1s) and [easing](https://developer.mozilla.org/en-US/docs/Web/API/EffectTiming/easing) like in the css animation property.

That's nice and easy but it's no more than defining a transition like:

```css
:host {
  transition: background-color 1s;
}
```

No worry, that's part of the story as both CSS transitions and Angular Animations use the [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API/Using_the_Web_Animations_API) under the hood. It's good that we can match concepts that easily while we have the door wide open to more powerful techniques than we could ever do in CSS only.

Let's continue.

### Target Element
You might have wondered where the element is being targeted to animate. That's happening through the trigger Name prefixed with an `@` sign (hidden in the `@HostBinding('@stateAnimation')`). That's a special annotation for animations only and you get an error if you refer to a trigger that is not existing.

You can do this in your template

```html
<div [@stateAnimation]='valueOrState'></div>
```

or you can directly target the component itself with a host binding

```typescript
@Component({...})
class MyComponent {
  @HostBinding('@triggerName')
  public valueOrState: 'on'|'off' = 'on'
}
```

You can also reuse it across multiple elements if you want of course — the animation will be spawned independently.

```html
<div [@triggerName]='valueOrState'>content A</div>
<div [@triggerName]='valueOrState'>content B</div>
```

## Animation 2: Appear & Disappear
Branch `animation/02`

We can also animate an element that is being created for example by using a structural directive like `*ngIf` or `*ngFor`. Try the following.

```typescript
<button (click)='showElement = true'>show element</button>
<div [@stateAnimation]="state" *ngIf='showElement'>
  animate me
</div>
```

and add another transition to your existing trigger `stateAnimation`.

```typescript
transition('void => *', [
  style({
    height: 0,
    overflow: 'hidden'
  }),
  animate('1s',
    style({
      height: '*',
      overflow: 'auto'
    })
  )
]),
```

Click on the button `show element`. You have created a transition from height 0 to the actual height of the element. Did you ever try this in pure CSS? It's simply not possible and we created it just by dropping in that transition 💪.

A few things are new here. We have another style object added directly into the transition (where we set `height` to 0) and also into the animate property. That's really powerful because we can set initial properties of a transition where no state is (void has no state but we want to ensure to start with zero height). We can also set the target values of a transition without declaring a state — here we tell the animation system that we want the actual height of the element (`height:'*'`).

Noticed the special keyword `void => *` in the transition call? That's one of a few reserved keywords in the DSL. `void`, `*`, `:enter` & `:leave` (also `:increment`, `:decrement` when your trigger is numeric)
`void` stands for any element not yet in the view, either leaving or entering the view. `:enter` & `:leave` are syntactic sugar for `void  => *` & `* => void` so you can target those states more semantically.

Let's create a new trigger to showcase `:enter` and `:leave`. This time we won't have any state involved.

```typescript
trigger('boxAddRemoveAnimation', [
  transition(':enter', [
    style({
      height: 0,
      transform: 'translateX(-100%)'
    }),
    animate('0.4s', style({
      height: '*',
      transform: 'translateX(0)',
    })),
  ]),
  transition(':leave', [
    animate('0.4s', style({
      height: 0,
    }))
  ])
])
```

Also define a template using that trigger multiple times:

```html
<button (click)='showAll=!showAll'>show</button>
<div @boxAddRemoveAnimation class="box"></div>
<div @boxAddRemoveAnimation class="box" *ngIf='showAll'></div>
<div @boxAddRemoveAnimation class="box"></div>
<div @boxAddRemoveAnimation class="box" *ngIf='showAll'></div>
<div @boxAddRemoveAnimation class="box"></div>
<div @boxAddRemoveAnimation class="box" *ngIf='showAll'></div>
<div @boxAddRemoveAnimation class="box"></div>
```

So what's happening?

The trigger defines zero height and and horizontal offset as the initial styles for the entering elements. Then the elements are transitioned to full height and zero translation. THis yields an appearing effect from the left with growing larger in height. The leaving animation is easier. We simply shrink the elements down to zero height. After that Angular will remove them.

Pretty, isn't it ?

## Animation 4: Disable
When you want to disable animations depending on an expression you can use the special binding `@.disabled`. This will prevent all animations on the element all children.

```html
<div [@boxCounter]="boxCounter" [@.disabled]="true">
    <div *ngFor="let items of boxes" class="box"></div>
</div>
```

There are some exceptions:
In the following example we move the disabled from the parent to the children. This will still animate because the animation is on the parent and using :enter to target the entering children. Disable can only disable animations bound to the specific element not effects down from parents.

```html
<div [@boxCounter]="boxCounter">
    <div *ngFor="let items of boxes" class="box" [@.disabled]="true"></div>
</div>
```

To disable all animations in a component you can go to the host element and bind the disable flag there.

```typescript
@HostBinding('@.disabled')
public animationsDisabled = true;
```

You can even go as high as the bootstrapping app component to disable all animations (or use `NoopAnimationsModule` instead of the `BrowserAnimationsModule`) to disable it with no possibility to enable it during runtime.

With trigger, state, transition and animations, we already have a nice bag of tools to work with animations. What's missing is the possibility to create complex ensembles of elements and animations.

There are:

+ query to target other elements
+ stagger to play delayed animation on a group of elements
+ group  for parallel animations
+ and sequence for sequential animations

You can already target elements with the animation trigger but there is no possibility to involve children yet. You use query to do so. Here an example.

Create a new component (`ng g component animations/complex-query`) and place 6 of the following elements your template.

```html
<div class="panel">
  <h1>Headline 2</h1>
  <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Sed dignissimos ut sit numquam veritatis sapiente error, quas soluta? Illum sequi dolorem cumque incidunt illo deleniti magnam porro consequuntur vero sint!</p>
  <button>press me</button>
</div>
```

Create a `3*n` grid of boxes in your component.

```css
:host {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 10px;
}
```

Now you can animate them once the host component enters the view. By using query('.panel') we can select each panel in the order of their appearance in the DOM and apply an animation. The animation timing is offseted as we use stagger.

```typescript
@Component({
  animations: [
    trigger('showBoxesOnLoad', [
      transition(':enter', [
        query('.panel', [
          style({ opacity: 0, transform: 'translateY(-100px)'}),
          stagger(-30, [
            animate('500ms cubic-bezier(0.35, 0, 0.25, 1)', style({ opacity: 1, transform: 'none' }))
          ])
        ])
      ])
    ])
  ]
})
class ComplexQueryComponent {
  @HostBinding('@showBoxesOnLoad')
  public showBoxes = true;
```

That looks nice already. We also want to animate the background color of the host element itself.  By default what you pass into transition is considered as a sequence. You can use a group to animate in parall as you want the background to fade in in parallel:

```typescript
transition(':enter', [
  sequence([
    animate('3s', style({
      backgroundColor: 'hotpink'
    })),
    query('.panel', [
      style({ opacity: 0, transform: 'translateY(-100px)'}),
      stagger(-30, [
        animate('500ms cubic-bezier(0.35, 0, 0.25, 1)', style({ opacity: 1, transform: 'none' }))
      ])
    ])
  ])
])
```

If you want to persist the color (remember nothing after a transition is persisted) you need to provide a state or give the element an explicit background. Prepend your state definition to the transition.

```typescript
state('true', style({
  backgroundColor: 'hotpink'
})),
transition(':enter', [])
```

Let's try another animation.
Headline, Copy and Button should appear one after another and it should be staggered across all panels. So first create a sequenced animation to show one element and stagger that for each panel.
The difficult thing here is, we stagger the panel animations so they run pretty late. The animation system won't set any initial css values in beforehand, that's your duty. So pick whatever initial value you want, group them if necessary and put them before the actual query with the staggering animations. This way all initial css values are set and wait to be animated.

```typescript
group([
    query('h1', [
      style({ opacity: 0, transform: 'translateX(-100px)'}),
    ]),
    query('p', [
        style({ opacity: 0, transform: 'translateX(100px)'}),
    ]),
    query('button', [
        style({ opacity: 0, transform: 'translateX(-100px)'}),
    ]),
  ]),
  query('.panel', [
    stagger(30, [
      sequence([
        query('h1', [
          animate('250ms cubic-bezier(0.35, 0, 0.25, 1)', style({ opacity: 1, transform: 'none' }))
        ]),
        query('p', [
          animate('250ms cubic-bezier(0.35, 0, 0.25, 1)', style({ opacity: 1, transform: 'none' }))
        ]),
        query('button', [
          animate('250ms cubic-bezier(0.35, 0, 0.25, 1)', style({ opacity: 1, transform: 'none' }))
        ]),
      ])
  ])
```

animations: [ trigger, trigger ]
trigger: [ state ]

## Animation 5: Router Animations
That sound very special but after everything you learned around animations this is going to be pretty easy. There is only one thing to remember: Your state is the current router path and you read it from your outlet by exposing the outlet state through the outlet template reference accessor.

```typescript
<router-outlet #yourOutlet="outlet"></router-outlet>
```
Prepare a getter function to extract the current route which you can use as your state.

```typescript
@ViewChild('yourOutlet') outlet: RouterOutlet;
get routerState() {
  return this.outlet.isActivated && this.outlet.activatedRoute.routeConfig.path;
}
```

You can test if it's working inside your template:

```typescript
{{routerState}}
```

It will output the current route as a string that you will use as your trigger to get a different state per route. Crate a trigger and bit it to your current route.

```typescript

@Component({
 //...
  animations: [
    trigger('routeAnimations', [

    ])
  ]
})
export class AppComponent {
  @HostBinding('@routeAnimations')
  get routerState() {
  ...
  }
}

```

You can now create transitions between your router urls.

```typescript
animations: [
    trigger('routeAnimations', [
      // * <=> * can also seen as: whenever the binding changes, I don't care for the actual value.
      transition('* <=> *',[
        query('.content',[
          style({
            opacity: 0
          }),
          animate('1s', style({
            opacity: 1
          }))
        ])

      ])
    ])
  ]

```

Once you get the hang on nesting trigger, transition, animate, query, style and so on you can quickly create nice animations but it can get crowded. That's why you can read in animations from external files and also reuse animations with parameters.

## Animation 6: Animate Children
If you mount the created examples in a router and you apply a routing animation on the router outlet you will see that neither of the children will animate.

You have to use `animateChild()` together with `query` to target the elements that contain your animation (full component tag name, nested class names or target the animation itself with the trigger Name `@animationName`. You can also target all animations if you want with the following query. It's important to pass `optional` as not every element with an animation is ready at that point of time.

```typescript
query('@*', [
    animateChild()
], { optional: true })
```

The query + animateChild is basically the start signal for the child animation. Use it in combination with group or sequence if you animate anything else in your parent animation.

## Completed
We are done 🙌  We tackled the following topics and can continue with the challenge which will be amazing!

+ Animation 1: Animation Basics
+ Animation 2: Appear & Disappear
+ Animation 3: Numeric Triggers
+ Animation 4: Disable
+ Animation 5: Router Animations
+ Animation 6: Animate Children

## Challenge
Continue with [Chapter 06 - Animation (Challenge)](../challenges/06-animation.md)

# Excluded Content

## Animation 6: Keyframes
Instead of defining start and end values we can also describe an animation more granularly with keyframes just like in CSS.

```typescript
trigger('keyframeTest', [
  transition(':enter', [
    animate('1s', keyframes([
      style({ backgroundColor: 'red' }), // offset = 0
      style({ backgroundColor: 'blue' }), // offset = 0.33
      style({ backgroundColor: 'orange' }), // offset = 0.66
      style({ backgroundColor: 'black' }) // offset = 1
    ]))
  ])
])
```

```html
<div *ngFor="let items of boxes" @keyframeTest>some content</div>
```

## Animation 7: Complex Arrangements

## Animation 10: Reuse Animations with Params
Let's look how we can reuse animations and also use params to customize them.

A trigger value has an undocumented second format. Instead of being a string you can use an object with the type {value:string, params: {}}. Params will be passed through to the animation and you can interpolate the values.

```typescript

@HostBinding('@myTrigger')
public triggerState = {
  value: 'left',
  params: {
    speed: '250ms',
    backgroundColor: 'yellow'
  }
}

trigger('myTrigger', [
  transition('* => *', [
  animate('{{speed}}', style({
       backgroundColor: '{{backgroundColor}}'
  }))
  ])
]),

stateOne() {
  this.triggerState = {
    value: 'left',
    params: {
      speed: '250ms',
      backgroundColor: 'red'
    }
  };
}
stateOne() {
  this.triggerState = {
    value: 'right',
    params: {
      speed: '2000ms',
      backgroundColor: 'green'
    }
  };
}

```

Each time you assign a new value to `triggerState` you trigger the animation system because the value of the trigger changes (here explicitly given with the value fields).
When the animation is built you can use params provided in the trigger value to interpolate values like we do:

```typescript
animate('{{speed}}', style({
  backgroundColor: '{{backgroundColor}}'
}))
```

There is no default value that you can give here (see Animation Package: interpolateParams) so you have to ensure a value all the time. You can also use observables as a trigger value, looks like this:

```typescript
private triggerSignalSubject = new BehaviorSubject({
    value: 'left',
    params: {
      speed: '1s',
      backgroundColor: 'red'
    }
  });

public triggerSignalObservable = this.triggerSignalSubject.asObservable();
```

In your template or host binding use the async operator:

```typescript
<div [@myTrigger]="triggerSignalObservable | async">
  I will be changed with the power of a stream
</div>
```

You can now feed in values through your stream:

```typescript
this.moveSignalSubject.next({
  value: 'left',
  params: {
    speed: '250ms',
    backgroundColor: 'red'
  }
});
```

There is a nice example what you can do with it:
https://medium.com/frontend-coach/angular-router-animations-what-they-dont-tell-you-3d2737a7f20b

See the left/right example where the transition offset is switched depending on the current route being before or after.

## Animation 11: Other things

### Event Callbacks
You can listen for start and done events to know about the state of the animation.

```typescript
<div [@trigger]="someState"
    (@trigger.start)="onAnimationEvent($event)"
    (@trigger.done)="onAnimationEvent($event)">
</div>
```

### Animate same router component
when you have `{path: ':index', component: ViewComponent}`
your route component won't change so you do not get an `:enter` transition.

You can use customized `RouteReuseStrategy` via https://medium.com/frontend-coach/angular-router-animations-what-they-dont-tell-you-3d2737a7f20b

```typescript
{
  path: 'router-example', pathMatch: 'full',
  redirectTo: 'router-example/1'
},
{
  path: 'router-example/:index',
  component: RoutingComponent,
},
```

This will reuse a component and if you have a simple enter opacity effect on the router you will the animation only the first time this component is activated. Any change the only involves the given index param won't recreate the component but instead use the observable of the params observable to communicate changes on the params.

The animation system won't get triggered. You can try to override `RouteReuseStrategy`. But did not work for me in a quick example.

