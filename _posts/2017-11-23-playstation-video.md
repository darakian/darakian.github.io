---
layout: post
title:  "Looking back at consoles and codecs"
date:   2017-12-6 20:13:00 +0100
---
# Classic consoles and codecs
## What was once mind bending
Back in the 90's gaming was undergoing a radical change. Solid state cartridges had been the normal distribution method for decades and by the time of the Super Nintendo we had games starting to reach 10MB in size. The games were quite good too with graphical standouts like Donkey Kong Country

![Cheeky Monkey](https://i.imgur.com/LVRNLvA.png)  

and Chrono Trigger

![HDR Lighting... maybe](https://i.imgur.com/G5bUDab.jpg)

However, there was a new technology arriving on the scene. The compact disc. Used primarily for audio playback the CD was being adapted as a generic data storage medium and console makers were experimenting with it as a new format for game distribution. Sega jumped in early with the Sega CD and horrible low resolution video was being used as a gimmick to sell games. Most of these games were simply a collection of cutscenes being played one after another (See [Sewer Shark](https://en.wikipedia.org/wiki/Sewer_Shark)). Video playback and interactive movies were fine, but never really made it out of the novelty phase. It wasn't until the Sony launched Playstation that the potential of the CD was fully realised. If there's one game that truly showed off what the CD could add it was Final Fantasy 7 and just look at it.

![Look at those arms!](https://i.imgur.com/mqoBUu8.jpg)

Ok maybe the basic models haven't aged so well, but the the FMV sequences had better lighting, more detailed models, etc... and despite the low resolution they still look pretty good today.

![Just sleeping](https://i.imgur.com/THiW2gs.jpg)

FF7 blew my mind as a kid. Watching the cinematics blend into the game while listening to rich, complex score; it was something altogether different than what had come before. If you followed the marketing at the time you were to believe that games like FF7 simply would not fit on anything other than these massive CDs, and even then the game came on 3 of them! So then, what was on these discs? Well there was obviously the video and audio content mentioned before. There were character models, textures, menu sounds and of course game logic. As it turns out there's roughly four and a half hours of audio which is a bit longer than all four [Distant Worlds](https://www.youtube.com/watch?v=GYzPebzvOnw) albums played back to back. Compression was clearly key.

Audio had long been composed in MIDI formats i.e. by writing programs for the consoles sound processor and in the case of FF7 this is no different. Character models were already pretty simple and many polygons didn't even have textures, but were rather just shaded with a single color. No, the elephant on the discs is the video which was not only huge but also unique per disc and the primary reason for the game to come on three discs at all.

## Revisiting The Playstation
Sometimes you wander the web and find some absolute gems. One of the best documents I have ever stumbled across has to be  [Everything You Have Always Wanted to Know about the Playstation But Were Afraid to Ask.](https://gamehacking.org/faqs/PSX.pdf). A dense tome detailing the internals of the playstation, how it worked and what was in it. There's a section on something called the Motion Decoder which describes hardware used to decode "JPEG like images" in macroblocks (16x16 pixel segments) for display by the GPU. The motion decoder decodes at a maximal rate of 9000 macro blocks per second or about 30 frames per second at a resolution of 320x240. It's crazy what passed as high tech back in the day. So, were playstation FMVs motion jpeg videos? The hardware was there and who would write a software decoder for another codec? This was the early 90s, we didn't have the CPU power to do that! In fact a lot of work has been done in decoding playstation games and the videos in them. There are even converter utilities on the the net to take the playstation specific file formats and convert them into more standard files. None of these converters that I found actually run on my machine, but this one [jpsxdec](https://github.com/m35/jpsxdec) looked promising. Motion JPEG is a notoriously bad video codec where each frame is spatially, but not temporally compressed. That is, each frame is simply compressed as a jpeg and displayed one after the other. This leads to big files which leads to lots of discs. So how much of a playstation game is just jpeg data?

## Analysis of disc files

So lets look at those files. Just so it's clear I own a copy of FF7 for the playstation. That said, I'm using iso images with the following.
```
Sha256 hashes
disc1 - f35c22ffbdca9890a9edf88c4dad9c143227839f842b54801e5149d93a2966ec
disc2 - 728273d567f8f3bc8fad8705cc8660a0e0264b14ffcf4a4cd2d89b7e459023ce
disc3 - 627e6f0784e8fcbb72253dc6182f0e22eb5957b7bd625fa959b07197775870ca
```
Right so, thankfully each disc is easy to mount and we see a similar layout on each disc. (x={3, 4, 5} on disc {1, 2, 3})
```
battle      enemy2      enemy4      enemy6      init        menu        mint        scus_941.6x stage1      startup     world
enemy1      enemy3      enemy5      field       magic       mini        movie       sound       stage2      system.cnf
```
Wonderful! There's a folder for all the movies just sitting there! On disc three there's an extra folder named snova, but never the less it's pretty consistent and should be easy to analyse. I could have used a collection of unix utilities to look further into this, but instead I wrote a tool for this which traverses directories, hashes files and builds lists. [ddh](https://github.com/darakian/ddh/tree/master)

First lets look at the movies with ddh
```
199 Total files (with duplicates): 1025 Megabytes
123 Total files (without duplicates): 928 Megabytes
83 Single instance files: 867 Megabytes
40 Shared instance files: 61 Megabytes (116 instances)
```
As expected there's a little bit of overlap with just shy of a gigabyte of unique data spanning the three discs. That's two CDs worth of data right there!

Using ddh on the three mounted disc images, I found some interesting results.
```
10131 Total files (with duplicates): 1756 Megabytes
3134 Total files (without duplicates): 1168 Megabytes
144 Single instance files: 871 Megabytes
2990 Shared instance files: 297 Megabytes (9987 instances)
```
Using du and tree as sanity checks I verified that there are indeed 10131 files and that the disc usage is indeed 1756MB (which makes sense for a game of 3 CDs). What's immediately shocking to see is that the unique files found sum up to be less than what can fit in 2 CDs worth of storage space. I spent days pouring over this and something I ran into while writing this post is the fact that there are duplicates of the same file on the same disc. This seems to be the result of a tactic I recall some game developers talking about for older disc based consoles. The idea was to place the same file in multiple locations on disc to lower the seek time for accessing these files and thus to make the game play a little smoother. [RAM and Crash](https://www.gamasutra.com/view/news/310660/Memory_Matters_A_special_RAM_edition_of_Dirty_Coding_Tricks.php)

## Shared files
So what are the shared files? Well with 2990 of them in just under 10,000 places I won't go into all of them, but there are a few classes. Some shared files are pretty obvious such as these magic files from the magic directories
```
FF7Disc1/magic/kona.bin - e04de3eb68049869
FF7Disc2/magic/kona.bin - e04de3eb68049869
FF7Disc3/magic/kona.bin - e04de3eb68049869
```
Assuming Kona.bin is some magic spell (a coffee related spell maybe?) which was available to the player early in the game then it would need to be on all three discs simply because the player might cast it at any time. Other obvious files are videos like the opening demo which is stored on all three discs.

Some are less obvious such as magic files which have the same hash, but which are stored with a different name (these files were manually inspected after discovery)
```
FF7Disc1/magic/tupon1.lzs - fbd9bd6ca8089d13
FF7Disc1/magic/vaha0_5.lzs - fbd9bd6ca8089d13
FF7Disc2/magic/tupon1.lzs - fbd9bd6ca8089d13
FF7Disc2/magic/vaha0_5.lzs - fbd9bd6ca8089d13
FF7Disc3/magic/tupon1.lzs - fbd9bd6ca8089d13
FF7Disc3/magic/vaha0_5.lzs - fbd9bd6ca8089d13
```
I don't have an explanation for these renamed files, but they're quite frequent and I think it's well worth loading up your discs and taking a look at all the repeated files if you're interested in the game development that went into this duplication as there is a ton of it.

## So about those video files

Motion jpeg is a horribly inefficient video codec. As I was unable to get a playstation video converter working I had to think about this by proxy. I found some old low resolution video and encoded it to both mpeg1 and motion jpeg. Looking over a large selection of video I found that the mpeg1 video was about 65% the size of the motion jpeg. The implication here being that if all the video files in FF7 could be encoded in mpeg1 the total file size would drop to 603.2MB.

## So how big is Final Fantasy 7?
Well if you discount the video files and just look at the unique game files then FF7 is 240MB, but even with all the unique files the game looks like it could have easily fit into two discs. If history had taken a different path and mpeg1 support had been baked into the system the total game size would have been 843.2MB. So why was FF7 a three disc game? No idea. I started this article with the thought that I could point and laugh at an old game system about how bad the video codecs were, but that doesn't seem to even be a problem. The game itself seems like it could have been a two disc game without much work. I think that had there been a better video codec and had there been a few cuts this could have even been a single disc game.

I have a few additional thoughts as well
* Duplicate files may have been necessary for the game to play smoothly due to slow CD speeds and/or low ram capacity
* Mpeg decoding may have been far more expensive than motion jpeg and multiple CDs were probably still cheaper than cartridges (which were the alternative)
* 3 Discs may have just been for the wow factor

```
Thanks for reading
```
