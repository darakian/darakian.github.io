---
layout: post
title:  "Looking back at consoles and codecs"
date:   2017-09-28 11:28:12 +0200
categories: musing update
---
# Looking back at consoles and codecs
## What was once mind blowing

Back in the 90's when the console wars were killing companies and rapidly evolving what a gaming console was; video play back was wowing everyone. Compact Disks were being added to consoles and horrible low resolution video was being played back for cutscenes, introductions, and occasionally for the entirety of the game (See [Sewer Shark](https://en.wikipedia.org/wiki/Sewer_Shark)). Playing back video had the benefit of making some nice screenshots to stick into a magazine, but generally made a game worse. It's worth remembering that video games were (and maybe still are) generally marketed toward children and a good looking screenshot is sometimes all it takes to make a sale.

One game that really made the case for these Full Motion Videos (FMVs) was Final Fantasy 7 which blew my mind when I was a kid. Final Fantasy 7 used FMVs for cinematic sequences: introductions, story heavy sequences, and generally anything where the hardware of the time was unable to render the desired effect. The playstation itself was quite limited in terms of hardware and any old FF7 screenshots will make these limits clear. For example

![Look at those arms!](https://i.imgur.com/mqoBUu8.jpg)

While the FMV sequences had better lighting, more detailed models, etc...

![Just sleeping](https://i.imgur.com/THiW2gs.jpg)

 As a way to sidestep the hardware limitations of the playstation FMVs were an interesting idea and quite pragmatic given the norms of the day. Mixing in pre-rendered content you could get something more cinematic and something that really felt futuristic. However, FMVs came at a cost; file size. Final Fantasy 7 shipped on three 660 MB disks. There's a good amount of overlap on these disks with the entire game engine, character models, common sounds, map data, etc... needing to be replicated, but the video files are mostly unique.

When you have free time and ponder the consoles of the past sometimes you wander the web and find some absolute gems. I did this while pondering the playstation and stumbled on [Everything You Have Always Wanted to Know about the Playstation But Were Afraid to Ask.](https://gamehacking.org/faqs/PSX.pdf). It's a dense read and mostly pretty boring unless you want to make your own software, but there's a section on something called the Motion Decoder which describes hardware used to decode "JPEG like images" in sections called macroblocks (16x16 pixel sections) for display by the GPU. The motion decoder apparently decodes at a rate of about 9000 macro blocks per second or about 30 frames per second at a resolution of 320x240. It's crazy what passed as high tech back in the day. Reading this I was left wondering; were playstation FMVs motion jpeg videos? The hardware was apparently there and who would write a software decoder for another codec? Doing a bit more reading I was able to find references to the motion decoder on wikipedia and in old gaming magazines, but I couldn't find any specifics. Surely someone must have looked into this by now. Right?

In fact a lot of work has been done in decoding playstation games. There's even a converter utility on github to take the playstation specific file formats and convert them into more standard files [jpsxdec](https://github.com/m35/jpsxdec). Motion JPEG is a notoriously bad video codec where each frame is spatially, but not temporally compressed. That is, each frame is simply compressed as a jpeg. This leads to big files which leads to lots of disks. So how much of a playstation game is just jpeg data?

## Analysis of files

### Other shared files

### How big is Final Fantasy 7?

### TODO


## Implications and context

You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
