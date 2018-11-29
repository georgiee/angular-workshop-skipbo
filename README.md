# Angular Workshop
Welcome to this card game themed Angular Workshop. We will develop & play Skip-Bo, a famous and easy to learn card game. The making of this will be so much more fun than creating another todo list manager while having being even more challenging 💪

In six chapters you will learn about advanced techniques in Angular, gain deeper knowledge of well known parts of the framework and as if that were not enough, you will program a fully working Skip-Bo card game in parallel too!

![](images/preview.gif)

---

## What's in the box
The two days of our workshop are packed with six chapters. Each chapter includes a **theory part** and a **code challenge** where you have to code.

### Theory Part
Theory is boring and especially in the case of Angular many, many things can be found in their [outstanding documentation](https://angular.io/docs) already.  Personally I like to read about best practice, untold facts and surprising functionality that isn't part of any documentation.

Personally I like reading documentation and listening to people telling me some unknown facts about stuff I thought I knew already. If there is some room for discussion, great, couln't think of anything ! The theorey part should b

### Code Challenges
The challenges are six individual coding sessions with a total duration of roughly 6 hours over two days. In the end you will hold a fully working card game in the hand, with animations and AI-like CPU players. There is a lot of code involved and we couldn't programe everything from scratch in 6 hours. Some parts would be tedious and boring, others too complex and elaborated.

That's the reason why each session will start with a dedicated starting branch (`workshop/XX-chaptername-start`) — this allows me to provide you some code additions that brings the overall game forward without giving you everything. Trust me, you will have enough to code and think about.

There are also some progress branches (`*--progress-01`, `*--progress-02`) during a challenge  I use those progress branches to add some more parts during a challenge. You need to check them out to follow the challenge. Some people will be quicker than others, that's the second reason for having those progress branches: That's the perfect moment to catch up with the rest if you want.

## Schedule - Day 01
### Chapter 01 - Modules & Injection
We start slowly with a recap of what Modules are and their special role in Angular. The Injection System is tightly bound to the module world so it's a good moment to revise them in this chapter.

+ [Lession](docs/01-modules.md)
+ [Challenge](docs/challenges/01-modules/modules.md)

### Chapter 02: Components
Learn all about Directive/Components, things you can to in templates, ChangeDetection and the difference between presentational and smart components. In the challenge we will create our first components: CardFace, Card, CardPile and our gaming stage.

+ [Lession](docs/02-components.md)
+ [Challenge](docs/challenges/02-components/components.md)

### Chapter 03: Routing
We will get serious by providing a welcome page, instructions, a gameover page and a page to select our players. In the challenge you will map the routes, introduce lazy loading and routing guards.

+ [Lession](docs/03-routing.md)
+ [Challenge](docs/challenges/03-routing/challenge.md)


## Schedule - Day 02

### Chapter 04: RxJs
Learn the difference between cold & hot, some important rxjs operators and how to test. In the challenge we will Oscar 🐙 an card playing AI with the power of RxJs.

+ [Lession](docs/04-rxjs.md)
+ [Challenge](docs/challenges/04-rxjs/challenge.md)

### Chapter 05: Testing
How to use headless browsers, use hosting/wrapper components in tests and learn important details of the change detection system and how it impacts your testing (tick, fakeSync, micro, macro tasks). In the challenge we will fix a nasty component bug and test Oscar's 🐙 async rxjs stream.

+ [Lession](docs/05-testing.md)
+ [Challenge](docs/challenges/05-testing/challenge.md)

### Chapter 06: Animation
Learn about animations in Angular, how to apply and control them. In the challenge we will flip some cards 🙌

+ [Lession](docs/06-animation.md)
+ [Challenge](docs/challenges/06-animation/challenge.md)


## Let's get started

### Editor
Use whatever coding editor you are comfortable with. if you are unsure I recommend Visual Code Editor (VSCode) from Microsoft.

Make sure you have the following extension (or similar ones in other editors) installed:

+ *Angular Language Service*: to get language support in your html templates

+ *tslint*:  to be a good citizen in the programming world

+ Switch Typescript to the Workspace version (bottom right in VSCode). Otherwise you will have problems with Intellisense. That will hunt you in any Angular project.

### Code
Clone this repository. `--recurse-submodules` will include the Angular project itself which is included here only as a submodule.

```bash
git clone --recurse-submodules https://github.com/georgiee/angular-workshop-skipbo
```

Switch in branch `workshop/start` and install all dependencies with `npm install`.

→ The workshop will start with the first chapter: [01-modules](docs/01-modules.md)