# QuizQuizQuiz Windows Phone 7 Postmortem

**IMAGE MISSING**

## Introduction
Back in July, I got an email from Simon at Four Door Lemon confirming that I’d be working on porting QuizQuizQuiz, an established iPhone quiz game, to Windows Phone 7. At the time a platform that had yet to be finalised and released to consumers. The iPhone game features over 36,000 questions for a range of languages and locales. It supports both single player game modes and also multiplayer with up to 5 players, perhaps the strongest selling point of the game however is how it’s quiz engine has been built, catering the experience so that the user sees suitable difficulty questions and avoids repeating the same categories and questions. With Windows Phone 7 to be released within a few months, it seemed like a logical option to port QuizQuizQuiz to the platform and target the game as a launch title for WP7.

While I had plenty of experience developing in C# and XNA, porting the game to Windows Phone 7 was an interesting experience requiring me to think about structuring for a platform I’d never encountered before. There was plenty to learn from the experience.

## What went wrong?

### Dynamic allocation on an embedded system

One of the advantages of using XNA is how easy it is to deploy to other types of platforms, while the game was targeting WP7, two clicks and I had a project ready to deploy to PC. The code needed to support full use on PC ran at just 8 lines. While testing on both the PC and the emulator, load times were rather minimal, a couple of seconds. It wasn’t until we received a device from Microsoft however, that I noticed how hefty the load times were. At the time I was concentrating on getting the game working fully, so load times weren’t my main concern. A few weeks later and I tracked down the issue to our question loading.

With 36,000 questions, compressing them made sense to keep the installation size down. Porting the decompression algorithm from C++ to C# directly on a line by line basis wasn’t possible, I’d written my own C# implementation from scratch. Taking a look at it again and I saw the magic word “List” at the top. Oh dear, it was having to reallocate over and over again as the file decompressed. As the Italian captain says in ‘Allo ‘Allo – “What a mistake-a to make-a!” Since we already knew how big the decompressed file would be I replaced the list with a fixed length array, allocated it once (combined with a couple of other minor optimisations) and the result – an 18x faster decompression time!

| Version             | PC          | WP7 Device   |
| ------------------- | ----------- | ------------ |
| Original Algorithm  | 300 ms      | 9700 ms      |
| Optimised Algorithm | 50 ms       | 600 ms       |
|                     | 600% faster | 1800% faster |

### Tombstoning

During their usage of smartphones, users often need to stop what they’re doing in one app and use another, whether that’s checking a map, searching for bus times, or making a note of something they’ve just remembered. The first version of the Windows Phone 7 OS does not support multitasking, so this means that for the moment applications can’t be minimised to the background while another is active. Apparently, Microsoft is planning to implement multitasking in a future release. However, Microsoft present us with an alternative to having your game killed off, when the user needs to fire up a browser, called Tombstoning. The Windows Phone 7 OS uses a stack-like structure for open applications. If your game is top of the stack and the user hits start, the start application will open and an event will be triggered in your application, allowing you to save information in the background, before the OS forcibly terminates the application. When the user has finished searching and your application appears at the top of the stack again, it is reactivated. This gives you an opportunity to load your saved data and resume gameplay as though nothing had happened.

Tombstoning is something I ended up implementing in QuizQuizQuiz, towards the end. There’s a mixture of reasons for this, namely that I was having difficulty testing tombstoning on the emulator and didn’t have access to a device at the time and also I had prioritised other parts of code development. In hindsight, this wasn’t a good idea. As the user can effectively terminate your application at any time (with warning fortunately) and the need for gameplay to be able to be resumed from any point, it is important for tombstoning to be factored into the code structure and design early on. It’s both Microsoft’s advice and also something we felt important that the state of the game on termination and reactivation should be exactly the same, including things like temporary on screen messages. Implementing this retroactively, took a lot longer than I wanted and I feel that for any future WP7 applications I work on, I’ll factor in tombstoning right from the start.

### Emulator and device differences

Despite hearing at presentations that the emulator emulates the device exactly, this was not my experience. While working on network support, which unfortunately didn’t end up making it into the QuizQuizQuiz release, I noticed several implementation differences, which initially confused me significantly. Additionally, after spending weeks testing and using QuizQuizQuiz builds on Windows and the WP7 emulator, I encountered an exception as soon as I attempted to load the game data files on the device. This turned out to be the usage of multiple serializable dictionary objects in one class and I believe relates to a difference in the XML serialization on the .NET Compact Framework. Oddly, this runs fine on the Windows Phone 7 emulator and it was a day or two until I tracked down the issue and could run the game on the device. If you haven’t used the .net Compact Framework before, you’ll notice less functionality and missing classes. While developing the decompression algorithm described earlier, I needed the BitVector32 class which is missing from the Compact framework. Fortunately help is at hand in the form of the Mono project, who have made available alternate implementations of classes with source code.

### Underestimating scope

Before working on QuizQuizQuiz, I had no mobile development experience or experience with Objective-C, previously having only worked on Windows based games and tools. My initial estimation that it would only take a few weeks to port was therefore ambitious to say the least. The scope of the project was something I underestimated and the number of bug fixes required was something I also didn’t expect. After 6 weeks, the game was fully playable on our Windows Phone 7 device and was more or less finished after 8 weeks, besides the final testing and bug fixing. I feel that having learnt from the experience of porting QuizQuizQuiz, if I were to do it again I’d be able to shave a couple of weeks off that and improve the gameplay structure.

## What went right?

### Using EasyStorage

Nick Gravelyn, formerly a Microsoft MVP and now part of the XNA Framework Team, released a library back in 2009 that manages data storage across multiple platforms. The library allows you to save and load data from local storage by writing your storage interaction code once, rather than writing all the platform specific permutations. Nick has since updated EasyStorage to run on XNA 4.0 and Windows Phone 7.

The rationale for this may not be immediately apparent, since this game was targeting Windows Phone 7 only. Although I found using the Windows version of the build quicker to test on (especially when you’re doing something which requires iterative testing), as it didn’t require the time needed to deploy and start-up on an emulator or device, the main importance of building on a cross platform storage system was that it gave us the possibility of deploying the game to Windows PC and Xbox Live Arcade Indie Games for the future, without needing to rewrite code that interacted with the storage systems. The difficulty in using EasyStorage is comparable to building on top of the storage device provided by XNA, so I feel it was a good idea to leverage EasyStorage just in case.

### Building our own GUI system

On previous projects that I’d worked on in XNA, I’d been involved in the development of XNA based GUI systems. This presented a number of challenging areas, particularly representing layouts on varying aspect ratios. I investigated the possibility of using an existing GUI system, in particular looking at those listed in the XNA Developer Survival Kit (which has now been superseeded for XNA 4 by XDSK2.)

However I decided, to achieve the intended interface controls it would be easier to roll my own GUI system. This went much more smoothly than I’d imagined, by defining a base GUI control class and some interfaces (user interaction for example), I found that it was very easy to create derived GUI controls and their functionality. Flagging a single control as active when the user interacted with it prevented the user from accidentally interacting with other controls and seems quite effective. While rolling your own GUI system could become very complex with advanced text rendering requirements, the basic system required and implemented for QuizQuizQuiz is something that I feel went right.

### Using existing data files

The iOS version of QuizQuizQuiz already had a substantial amount of data, from complete localisation tables to quiz categories and thousands of questions. Leveraging the existing data files and formats used by data formats and files was therefore a good idea as it meant that the existing file authoring systems could be used, saving time and also that the loading code would be straightforward to implement, which ended up being more or less a direct port of the Lemon Engine based code to System.IO. This was quick to implement, but importantly also quick to run and propagate the data structures in the game at runtime.

Supporting the existing authoring tools has important implications as it means both the iOS and Windows Phone 7 versions of the game use the same file format. New content then only needs to be generated once and will work perfectly on both platforms, making it quick and easy to update multi-platform versions.

### The hardware back button

Many touchscreen mobile devices also feature hardware buttons, previously I’d found the mixture of software and hardware buttons to be an irritation, particularly with using a touchscreen keyboard with a hardware backspace button. This was something which broke the flow of writing a message and slowed me down. Using the hardware back button on a Windows Phone 7 device however was something I very quickly got used to and appreciated.

I feel the stack-like navigation in WP7 is very effective and the usage of a hardware back button feels a lot more natural and logical to the touchscreen buttons employed on iOS. While it seems that I shouldn’t say usage of the hardware back button is something that went well, as supporting it is a technical requirement for publishing, I’m going to anyway! The presence and usage of a an ever present back button, speeds up day-to-day navigation on the phone and also helps standardise the user’s movement from screen to screen. It’s definitely a good idea to build your application or game flow and screen layouts with the hardware back button in mind.

## Stats:

**Software Used:**
Microsoft Visual Studio 2010 Express for Windows Phone
Notepad++
Adobe Photoshop
TortoiseSVN

**Platform:** Windows Phone 7
**Time in development:** 8 weeks (280 hours)
**Source Lines of Code:** 5869
**Development Machine:**
Windows Vista 32 bit
Intel Core i7
6 GB RAM
NVIDIA GeForce GTX 260

## Links:
- [QuizQuizQuiz on Windows Phone 7](https://www.windowsphone.com/en-gb/store/app/quizquizquiz/17f2ce36-7ed5-df11-a844-00237de2db9e)
- [QuizQuizQuiz WP7 Trailer](https://www.youtube.com/watch?v=sfUwzRIT2Fc&noredirect=1)
- [Original iOS Game](https://itunes.apple.com/gb/app/quizquizquiz/id329015885?mt=8)
- [Four Door Lemon](http://www.fourdoorlemon.com/)
