---
layout: post
title:  "Looking back at consoles and codecs"
date:   2017-11-23 11:28:12 +0200
---
# Classic consoles and codecs
## What was once mind blowing

Back in the 90's gaming was undergoing a radical change. Solid state cartridges had been the normal distribution method for decades and by the time of the Super Nintendo we had games starting to reach 10MB in size. The games were quite good too with graphical standouts like Donkey Kong Country

![Cheeky Monkey](https://i.imgur.com/LVRNLvA.png)  

and Chrono Trigger

![HDR Lighting... maybe](https://i.imgur.com/G5bUDab.jpg)

However, there was a new technology arriving on the scene. The compact disc. Used primarily for audio playback the CD was being adapted as a generic data storage medium and console makers were experimenting with it as a new format for game distribution. Sega jumped in early with the Sega CD and horrible low resolution video was being used as a gimmick to sell games. Most of these games were simply a collection of cutscenes being played one after another (See [Sewer Shark](https://en.wikipedia.org/wiki/Sewer_Shark)). Video playback and interactive movies were fine, but never really made it out of the novelty phase. It wasn't until the Sony launched Playstation that the potential of the CD was fully realised. If there's one game that truly showed off what the CD could add it was Final Fantasy 7 and just look at it.

![Look at those arms!](https://i.imgur.com/mqoBUu8.jpg)

Ok maybe the basic models haven't aged so well, but the the FMV sequences had better lighting, more detailed models, etc... and despite the low resolution they still look ok today.

![Just sleeping](https://i.imgur.com/THiW2gs.jpg)

FF7 blew my mind as a kid. Watching the cinematics blend into the game while listening to rich, complex score; it was something altogether different than what had come before. If you followed the marketing at the time you were to believe that games like FF7 simply would not fit on anything other than these massive CDs, and even then the game came on 3 of them! So then, what was on these discs? Well there was obviously the video and audio content mentioned before. There were character models, textures, menu sounds and of course game logic. As it turns out there's roughly four and a half hours of audio which is a bit longer than all four [Distant Worlds](https://www.youtube.com/watch?v=GYzPebzvOnw) albums played back to back. Compression was clearly key.

Audio had long been composed in MIDI formats or by writing programs for the consoles sound processor and in the case of FF7 it was no different. Character models were already pretty simple and many polygons didn't even have textures, but were rather just shaded with a single color. No the elephant on the discs is the video which was not only huge but also unique and the primary reason for the game to come on three discs at all.

## The Playstation

Sometimes you wander the web and find some absolute gems. One of the best documents I have ever stumbled across has to be  [Everything You Have Always Wanted to Know about the Playstation But Were Afraid to Ask.](https://gamehacking.org/faqs/PSX.pdf). A dense tome detailing the internals of the playstation, how it worked and what was in it. There's a section on something called the Motion Decoder which describes hardware used to decode "JPEG like images" in macroblocks (16x16 pixel segments) for display by the GPU. The motion decoder decodes at a maximal rate of 9000 macro blocks per second or about 30 frames per second at a resolution of 320x240. It's crazy what passed as high tech back in the day. So, were playstation FMVs motion jpeg videos? The hardware was there and who would write a software decoder for another codec? This was the early 90s, we didn't have the CPU power to do that! In fact a lot of work has been done in decoding playstation games and the videos in them. There's even a converter utility on github to take the playstation specific file formats and convert them into more standard files [jpsxdec](https://github.com/m35/jpsxdec). Motion JPEG is a notoriously bad video codec where each frame is spatially, but not temporally compressed. That is, each frame is simply compressed as a jpeg and displayed one after the other. This leads to big files which leads to lots of discs. So how much of a playstation game is just jpeg data?

## Analysis of disc files

So lets look at those files. Just so it's clear I own a copy of FF7 for the playstation. That said, I'm using iso images with the following hashes.
```
disc1 -
disc2 -
disc3 -
```
Right so, thankfully each disc is easy to mount and we see...


## Shared files

## So how big is Final Fantasy 7 really?

## Implications and context
