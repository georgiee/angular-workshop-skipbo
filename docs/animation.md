# Animation
Doing Animation in Angular not not only with CSS is useful as you can bind your animations easily to states and also wait for animations being completed.

You can use animations for state transitions and router transitions. Simple animation effects that can run standalone should be done with CSS only as you won't have any benefit plus a huge amount of boilerplate compared to the native CSS approach.

Animations in Angular are strictly bound to CSS animations. You can't animate what's not animateable with CSS too. Let's start.

Animations are always triggered declarative. You don't call any explicit function.

->  Web Animations API
https://drafts.csswg.org/web-animations/



https://medium.com/google-developer-experts/angular-supercharge-your-router-transitions-using-new-animation-features-v4-3-3eb341ede6c8


http://slides.yearofmoo.com/ng-japan-2017-slides/#/13/0/


## States & Transitions

Let's start with a simple usage of the animation system. Our goal is to give the component different backgrounds depending of the current state.

The state is reflected in a variables called `state` with values `on | off`. You can toggle between the values with the method `toggleState()`.

```typescript
@Component({
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

	toggleState() {
		this.state = this.state === 'on' ? 'off' : 'on';
	}
}
```

The result looks not so selling. There is a button and if you press it the background changes abruptly. There are no animations yet but you can easily add one.

Modify your animations array and append a transition method call.

```typescript
animations: [
	trigger('stateAnimation', [
	  ...
	]),
	transition('* => *', [
	   animate('1s')
	]),
]

```

The method call `transition('* => *', [animate('1s')])` consists of two parts. When should it happend (`* => *`) and what should happen.

The when is a custom DSL and you use the values from your state (here on/off or `*` for any) to tell what should happen for which change.

In case of what should happen we only tell to animate it over 1 seconds. The animation call takes a plain number which gets interpreted as milliseconds our you pass in a string. That string can contain timing (like 1000, 1000ms, 1s) and [easing](https://developer.mozilla.org/en-US/docs/Web/API/EffectTiming/easing) like in the css animation property.

That's nice and easy but it's no more than defining a transition like:

```css
:host {
	transition: background-color 1s;
}
```

But no worry, that's part of the story as both CSS transitions and Angular Animations use the [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API/Using_the_Web_Animations_API) under the hood. It's good that we can match concepts that easily while we have the door wide open to more powerful techniques than we could ever do in CSS only.

Let's continue.

## Target Element
You might have wondered where the element is being targeted to animate. That's happening through the trigger Name prefixed with an `@` sign. That's a special annotation for animations only and you get an error if you refer to a trigger that is not existing.

You can do this in your template:

```html
<div [@triggerName]='valueOrState'></div>
```

or you can directly target the component itself with a host binding:

```typescript
@Component({...})
class MyComponent {
  @HostBinding('@triggerName')
  public valueOrState: 'on'|'off' = 'on'
}
```

and you can reuse it across multiple elements if you want of course — the animation will be spawned independently.

```html
<div [@triggerName]='valueOrState'>content A</div>
<div [@triggerName]='valueOrState'>content B</div>
```

## Appearing & Disappearing Elements
We can also animate an element that is being created for examples by using a structural directive like `*ngIf` or `*ngFor`. Try the following.


```
<button (click)='showElement = true'>show element</button>
<div [@stateAnimation]="state" *ngIf='showElement'>
  animate me
</div>
```

and add another transition to your existing trigger `stateAnimation`.

```
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

A few things are new here.
We have another style object added directly into the transition (where we height to 0) and also into the animate property. That's really powerful because we can set initial properties of a transition where no state is (void has no state but we want to ensure to start with zero height). We can also set the target values of a transition without declaring a state — here want to tell the animation system that we want the actual height of the element (height:'*').


Noticed the special keyword `void => *` in the transition call? That's one of a few reserved keywords in the DSL.

void, *, :enter & :leave (also :increment, :decrement when your trigger is numeric)

void stands for any element not yet in the view, either leaving or entering the view.

:enter & :leave are syntactic sugar for `void  => *` & `* => void` so you can target those states more semantically.

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

The trigger defines zero height and and horziontal offset as the initial styles for the entering elements. Then the elements are transitioned to full height and zero translation. THis yields an appearing effect from the left with growing larger in height.

The leaving animation is easier. We simply shrink the elements down to zero height. After that Angular will remove them.

Pretty, isn't it ?

## Numeric Triggers
Assume our trigger is not a state with values on & off but a numeric value. 

```
public boxCounter = 0;
// transform our number into an array to use with ngFor
get boxes() {
	return Array.from(Array(this.boxCounter));
}
```

```
<button (click)='boxCounter = boxCounter + 1 '>add</button>
<div *ngFor="let items of boxes" class="box"></div>
```

Now you have a button that can add boxes. We want to animate those boxes so we create a new trigger referring to boxCounter.

## Disable Animations
When you want to disable animations depending on an expression you can use the special binding `@.disabled`. This will prevent all animations on the element all children.

```
<div [@boxCounter]="boxCounter" [@.disabled]="true">
    <div *ngFor="let items of boxes" class="box"></div>
</div>
```


There are some exceptions:
In the following ecxample we move the disabled from the parent to the children. This will still animate because the animation is on the parent and using :enter to target the entering children. Disablde can only disable animations bound to the specific element not effects down from parents.

```
<div [@boxCounter]="boxCounter">
    <div *ngFor="let items of boxes" class="box" [@.disabled]="true"></div>
</div>
```

To disable all animations in a component you can go to the host element and bind the disable flag there.

```
@HostBinding('@.disabled')
public animationsDisabled = true;
```

you can even go as high as the bootstrapping app component to disable all animations (or use `NoopAnimationsModule` instead of the `BrowserAnimationsModule`) to disable it with no possibility to enable it during runtime.

## Keyframes
Instead of defining start and end values we can also describe an animation more granulary with keyframes just like in CSS.

```
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
```
<div *ngFor="let items of boxes" @keyframeTest>some content</div>
```

## Complex Animations


## Other things

### Event Callbacks
You can listen for start and done events to know about the state of the animation.
```
<div [@trigger]="someState"
    (@trigger.start)="onAnimationEvent($event)"
    (@trigger.done)="onAnimationEvent($event)">
</div>
```

