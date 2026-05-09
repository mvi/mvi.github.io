# Lessons Learned From Tools Development

Recently I’ve been doing tools development, something I’ve done a lot of in the past when reverse engineering Dune 2000 (an old RTS.) I’ve learnt some interesting lessons in tools development, which I thought I’d share with you.

## Understand your user

Firstly, the most important part of tools development is seeing how the end user will use it and making it as easy as possible for them. When developing a series of tools for our artists, I’d frequently speak to them, going through the work process and how they feel it should flow. Make sure the tools you develop are suited to the intended target.

## Keep things straightforward

Everything should be kept straightforward, easy and quick (and obvious). Too many programs have massively complicated user interfaces which can be completely baffling. It’s worth spending time working on the design of the program and the structure whether that’s on paper or on a graphics package. The last tool I wrote I mocked up quickly in Photoshop what it would look like and drew the class structures on paper, before touching a line of code. This helps you think about how to make the program well structured and come across potential problems before they become an issue.

## Make everything fluid

It’s important to make everything fluid, support dragging and dropping of files rather than forcing a rather more cumbersome process of being locked in to the windows open file dialog box. If your user is applying a texture to a shader for example, let them drag it straight in from Windows Explorer, it makes things quick and easy. Auto focus helps keep the flow going with tool usage, having to keep reselecting the same option because the program forgets gets frustrating quickly. In a similar fashion if you have a list of items in a menu, let them be dragged up and down rather than requiring a button to be pressed each time you want to move the item up one.

## Keep clicking to a minimum

Following on from this idea, a good rule of thumb is to keep functionality down to as few clicks as possible, this way your end users will be able to work more productively. Shortcuts are also good at speeding things up, but should never be the exclusive way to carry out an action. Most modelling packages for example have many ways to achieve the same task. Less clicks is a fundamental improvement and goal for tools, people spend half their lives clicking through menu after menu. The objective of a good tool is to cut this down to a minimum.

## Clean UI

Keep the user interface clean and clear of clutter. Icons can be very helpful, but what they represent need to be easily recognisable at a glance. Adopting common icon designs is definitely a great idea, using a floppy disk icon for saving and an open folder for opening a file for example. Everyone knows what they mean, resulting in a quicker learning curve.

## Redesign and reiterate when necessary

When development of the tool becomes held back by poor or a changed design, it’s time to have a think, work out the optimal structure and recode the tool. Frequently I’ve wasted more time trying to make code work which is broken from poor design, than it would have taken me to change the structure and reimplement the code. I find an iterative approach to this development very helpful, the first design is rarely optimal and you’ll learn from each iteration.

## Target your tool

I also believe in the Unix ethos of developing small targeted use applications as opposed to über programs which attempt to be a jack of all trades but result in being a master of none. My experience of the asset production pipeline is that dedicated tools which each accomplish a job is a much more favourable approach, for example creating a mesh in Lightwave, shipping it off to Zbrush to be sculpted, then taking it into Topogun to create a lower poly version, baking the high poly to normal maps, tweaking in Lightwave again, generating maps with XNormal, then exporting to FBX via 3ds max (Lightwave’s FBX exported is supposedly pants).

## Flexible interfaces

This is more an issue of personal taste and indeed there are a lot of different systems out there for the way different parts of an application are presented. Recently I’ve gotten into using floating windows, if I want to bring up the properties of a mesh for example, I find it far more useful having that as a separate window. Windows can then be resized and moved about, your preview window can be maximized on a second monitor for example. Then the layout can be saved and preserved, allowing a lot of flexibility for the end user.

## Allow extension

Make tools that are easily extendible, with readable and editable configuration files. Custom user configuration helps makes people life’s easier, having the capability to change their preferences to something they find more suitable. We don’t all like the same layouts or colours, so supporting user changes will make people’s time less painful.

## Existing formats make life easier

Usage of existing and established file formats can be very useful. I like to use XML where appropriate (although I change the extension) as it makes serialization and deserialization very easy, tends to generate readable files and allows quick editing in a program like Notepad++ (or just Notepad). As a standardised system, it also means someone else can pick up your files and understand how they’re structured – which is certainly more involved with a custom or binary format. An alternative to XML is JSON, which provides a lightweight alternative that my friend swears by. It can also give a lot more flexibility when adhering to an established standard when storing other data, such as images and models.

While using established standards for storing any kind of data can be extremely helpful and effective, it does carry with it a potential inherent problem. By adhering to standards, you are limiting what you can do to what the standard supports. In the case of XML this may not be such an issue as its a very versatile markup language, however locking yourself in to an image or 3d model format can be extremely limiting. To get past this, you could extend or encapsulate the format or simply see if there is an alternative which is better suited.

## Summary

To sum up, planning and design with an iterative approach allows for speedy development of tools. Keeping things extendible and configurable means the user can keep things the way they prefer, while constantly looking to streamline the flow of usage speeds up tool use and improves productivity.

## Further Reading

- Shawn Hargreaves on [Tool Postmortem: Climax Brighton’s Supertools](http://www.gamasutra.com/view/feature/2987/tool_postmortem_climax_brightons_.php)
- Noor Khawaja on [Making Your Game Tools Fast And Efficient](https://www.gamasutra.com/view/feature/3798/making_your_game_tools_fast_and_.php)
- Dan Goodman on [Game Tools Tune-Up: Optimize Your Pipeline Through Usability](http://www.gamasutra.com/view/feature/4016/game_tools_tuneup_optimize_your_.php)
- Adams Greenwood-Ericksen, Eric Preisz, Shawn Stafford on [Usability Breakthroughs: Four Techniques To Improve Your Game](https://www.gamasutra.com/view/feature/6130/usability_breakthroughs_four_.php)
