---
tag: Reverse Engineering
---

# Core Concepts Of Reverse Engineering: Part 1 – Introduction

<img src="Images/ReverseEngineeringPart1Image1.png" style="float: right; margin: 5px 0 5px 20px;" />
**IMAGE MISSING**

This is the first in a series of mini articles on reverse engineering binary files, particularly those in computer games but the majority of what I will discuss has applications in all areas of computer software. While I may touch on reverse engineering executable code, memory hacking and plain text formats, the majority of the focus is on actual binary files.

Since 2009 I’ve been reverse engineering and modifying the classic Westwood RTS Dune 2000. Despite being released in 1998, the modding community was held back by a limited number of tools. For example while it was possible to edit and make new multiplayer maps for the game, it was not possible to make campaign maps and missions. A couple of guys had been exploring the mission files and had made small, but significant, discoveries. This was when I joined the modding scene and since then I’ve reverse engineered a whole range of different files in the game and released a bunch of tools based on my research.

When I started, I had no reverse engineering experience. I’d never used a hex editor and had no idea about the binary representation of data. If you’re in the same boat, then don’t worry, in this series of articles I’m going to talk about what I’ve learned in terms of both conceptual theory and also practical techniques that I employ. This series of articles is particularly aimed at computer programmers with no reverse engineering experience and who would like to get started.

As with programming, the initiative is all on you. You will apply the theory and techniques to a different set of problems and formats than I did. As such you will need to extend and modify them to fit your needs and make a few leaps on your own to successfully reverse engineer a complex format. But if you start off small and work your way up, once you understand the basic theory you’ll find reverse engineering is actually very straight forward.
