# Placeholders in Games

Creation and use of placeholders, whether levels, 3d models, sprites or sounds, has a number of advantages when developing computer games. In this article I discuss some aspects of the use of placeholders in games.

## Decouple programming and art schedules

Placeholders are an excellent way to support code development without waiting on asset completion. Using placeholder graphics decouples programming and art schedules, allowing them to progress independently of each other. By using quick and effective placeholders, the coders can get to work on what’s important without being held back waiting on final assets to be finished. The coders can then put in revised assets when they become available – just make sure you don’t leave any placeholders behind. An industry professional once told me how they “borrowed” some models from a very similar game to the one they were making at the time and accidentally shipped the demo with the hijacked assets!

## Speed of build turnaround – Proof of concept

High poly models or detailed assets can take a long time to create and when you’re trying to prove the feasibility of a project you’re often more interested in the technical side. When trying to decide if what you might attempt is technically feasible in your environment and time constraints, you may find that when building a proof of concept you just want some quick assets to put in so that you can continue to focus on building up the code. In terms of pre-production, placeholder art can be very important just to get a working build. I was recently at the Microsoft X48 competition, where we coded a game in 7 and a half hours. When developing, our artist mercilessly took Google images and made graphics by tweaking them in Photoshop. Our focus here was not to make a pretty game, but to get a working build as fast as possible.

## Identify the tolerances

Placeholders also are a great way to carry out tolerance testing when trying to work out how high poly you can make scenes for example. It would waste a lot of time for the artists to not know how detailed to make their 3d models and what limitations there are on the type of material properties they can use. By using placeholders that simulate the sort of poly counts and shaders that you want to support you can figure out the tolerances of your game engine, what you need to optimise and what the artists need to be limited to.

## Issues with placeholders

While placeholders mean that you can get the basic look and feel of your target environment without the full development time, they can also work to your detriment. It can be demotivating to see your project littered with ugly placeholders. Most games go the extra mile in creating the fun factor by rewarding visuals and involving graphical finesse. Likewise placeholders can create issues for the team, the designers may not know how the physics of a vehicle may work when the vehicle has not yet gone beyond being implemented as a placeholder, which of course must be understood for a more balanced and smoother level.

## Some examples in the context of level design

When Half Life 2 was being developed, the developers took a new approach. Separating art from design in the level creation, the designers would flesh out the gameplay before the team of artists made the levels look pretty. The initial stage in level production was to create an “orange map”, where the entire level’s textures were orange. The result of this meant that the team could test and perfect gameplay, script in any necessary elements and let the programmers work on what would become finalised levels without waiting for the art pipeline to be completed (or indeed necessitate the start of art development.) The result was a much more rapid and smoother development process, as designers involved in play-testing, tweaking and working out where those big, beasty enemies should jump out can do their work equally well in either a Sunny-D environment or a fancy multitextured and materialed environment.

In the context of level design for example, I discussed this with a friend who is a level designer using UE3. He stated that he believes using placeholders cuts development time in half. “With level design (my choice examples, as it is my area of knowledge) placeholders are easier to throw around. Gameplay isn’t fun? Put a block there to cut the view distance as opposed to putting a table there and finding out later that it is ineffective. If designers are too busy worrying about “But a table doesn’t make sense” then the actual fun will never be developed.”

## So who makes it?

Depending on your development environment (and in the case of studios), you may have either the programmer developing the placeholders or the 3D artist. I know a few triple A studios where they won’t even let their programmers have 3ds max installed in case they start making placeholder graphics, leaving this task to be done by the 3d artists.However, depending on your environment this may not be feasible or desirable. Depending on the organisation, it may be the 3d artists or the programmer who creates the placeholder, this is more of a managerial decision and each has its pros and cons.
