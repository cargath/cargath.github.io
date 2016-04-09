---
layout: post
title: "Thoughts on Entity / Component + Migrating an existing game to GameplayKit"
date: 2015-07-27
---

After WWDC15, i immediately wanted to try GameplayKit. However, a simple throw-away game for testing purposes would not have been enough: Never really satisfied with any architectural approach to my current project, i wondered whether Entity / Component might help clean things up. Of course, rewriting everything would take far too much time. Or would it?

My current approach was the inheritance-based one, [that the Apple documentation for GameplayKit uses as a bad example](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/EntityComponent.html), to make Entity / Component look good. Theirs isn’t entirely fair, though: Instead of moving the shooting-functionality to the base object, they could have created a shooting base object and have towers and enemies both extend that. I’m not saying these kind of architectures don’t tend to get confusing in larger projects, but it was perfectly fine for me.

What i wasn’t happy with were all the lengths i had to go to in order to keep my UI separate from the game logic. It would be so much easier if each object just had a SceneKit node to represent it, but that would mean putting UI code into the game object class. The first possible solution that comes to mind here are class extensions, but they can’t add any properties to a class, only functions. This is exactly where Entity / Component comes to the rescue: A way to add entirely new functionality, including new properties, to an existing object. This way i could add a SceneKit node component to each object without messing up the model code.

Doing this for my existing game was surprisingly easy. I just had my base game object extend `GKEntity`, which enabled me to add arbitrary `GKComponent`s. Now my existing inheritance-based architecture is still in place, but since i didn’t have any problems with it, that’s okay. I can now move existing code into components when i feel like it, and migrate slowly but steadily to GameplayKit without breaking things. I already did that for the SceneKit code mentioned before, which enables me to react to model changes more or less directly in a UI component, without implementing overly complex observer patterns. Before GameplayKit, i used to have model representing each event that happened during each cycle of the game loop, i.e. object X moved into direction Y, etc.
