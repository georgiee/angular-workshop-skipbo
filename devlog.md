# Log
I started end of October around 30/31 with the project, as a project ended the plan was to have full 4 weeks to develop the workshop. Luckliy (or unluckily) we got landed a project — starting on 21.11 and removing 1.5 weeks from my schedule 😑 This means workign on the workshop weekends, before work and after work. But it's a fun challenge and totally worth.

Did I mention I'm also 7 days in London for AngularConnect beginning of Novemeber ? Great planinng by me. See me working in London in some shared workspaces.

+ Research, Research, Research. There must be a Skip-Bo Game in JavaScript — which framework doesn't matter. But there isn't. Most games are only rough starters never completed.
+ Before doing nothing, I start with building a Typescript Only, TDD-driven  version of [Skipbo Core](https://github.com/georgiee/skipbo-typescript-jest). This took my around 5 days.
+ TDD with implementing the Game Rules was a beautiful fit. I will use this to show people how powerful TDD can be!
+ Continue with an Angular version. But let's
Integrate the Skipbo Core... no not yet. Search for some game artwork. And there is nothing. Search more. Still nothing. Search again — OK I give up, there is nothing I could use so I build myself a set of cards in Sketch

+ Research again for completed implementations. But nothing, there is really nothing.
+ Found the official App of Skip-Bo and I thought I could unpack the Android pkg somehow to get ANY insights. No luck.
+ Back to my Sketch design. I update some designs and layouts — as I got soem nice inspiration from the official app.
+ Let's start with implementing the Angular version. It's awesome how good my skipbo-core works as a "backend". This was not planned but its code only nature leaded to maybe.
+ Steady progress with the UI. I also created plenty of other Angular projects called `skipbo`, `skipbo-workshop`, `skipbo-workshop- scratchpad`, `scratchpad`, `playind-draft-no-ui` or simply `lessions`. For what? Because I really need to get my lessions started. There are only three weeks left.
+ I pivoted and started with some lists what I want to teach. I already have a list from the client but it's too huge. So I write a list, shrink it, switch it, delete it to end up with 6 items: modules, components, testing, aniamtion, rxjs & routing. Looks solid.
+ After having such a list I filled six markdown documents with whatever I felt is interesting about the topic. This was a great reflection of what I knew and what I should read about again.
+ Back to the projects, I continue working on building the final prototype and I spent far too much time with animations. But it looked so neat.
+ I run in circles and don't have a clear way of creating the lessions yet. One day the pressure (less than two weeks) is too much: I just create a first lession file. Write down some things I would want to learn about modules. Created a starter project which had all assets and the skipbo-core integrated and started coding. Step by step. Creatign branches for my milestones. That felt good!
+ It looks like I found my working mode. Chapter by chapter (the six topics) I review the theory files I created, and thought about nice things to let people build. I also got the brilliant idea to fast forward code to give the people a feeling of real progress so we can land at the final game that I prototyped. That helped me a lot as I was really blocked on how to make progress over the six chapters. It was important to me that the people can hold a finished game in the hands after six chapters. After figuring out that this was more important than people created every bit of code I was really confident that this will work.
+ I discovered the details & summary html5 tag for my purposes in the workshop to show Hints that are hidden initially.
<details>
<summary>Hint</summary>
So cool and useful for workshops 🙌
</details>

+ Generate a list of all branches (git branch -a) 49 (🧐) and manually check if all are pushed or force push if necessary)
+ git cherry pick changes across all branches
Found this in skipbo-core after creating ALL branches

```
+import { PlatformRef } from '@angular/core';
+import { executeInitAndContentHooks } from '@angular/core/src/render3/instructions';
```

Cherry picking to the resuce.
+ Here the [completed workshop](https://skipbo-angular-workshop.netlify.com/game/play) branch.

+ Tinkerign around with gifs and videos.
+ Started creating the presentation (less than 7 days) while working on the theory & challenge markdown files.
+ I use Deckset for the presentation so now technlogy gets in my way and I can just use markdown.
+ Refactor structure (separate theory + challenge folder, images moved)
+ Create Slides for Day 1 and Day 2 with Deckset so I can use markdown.
+ Should I do housekeeping across all branches or not 🤔
+ Decided to create previews of all completed challenges with netlify. Again, cherry picking the commit where I provide the necessary _redirect file (to allow pushState).


+ Forgot to create the theory file for components so I'm doign this now after I already finished everything else. Well "finished" we will see.
