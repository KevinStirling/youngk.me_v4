---
title: "Godot Wild Jam 64"
date: 2023-12-31T15:46:28-05:00
description: 'The making of CandleRun'
image: images/posts/candlerun_cover.png
tags: ["godot4", "gamejam"]
draft: false
---

Godot Wild Jam 64 happened this past month, with the theme Illumination, and I decided to give it a go! This was my 3rd game jam attempt, although this would be my first submitted entry. It's called CandleRun, and you can check it out [here](https://necrokev.itch.io/candlerun).

Along with the main theme, the 3 option "wild card" themes were "white out/ freezing out", "blood is fuel", and "trash everywhere".

## Design Process
I originally planned on a completely different game. Aside from many of the ideas I had during my brainstorming session, which included a souls-like style game about climbing a freezing mountain, a game that involved writing medieval illuminated manuscripts, and even a game about lighting dumpsters on fire...

What I eventually landed on was a colony sim game, where you manage a community living through a post apocalyptic eternal winter. At the center, a garbage fire keeps their village warm. Your job would be keep their fire burning bright by sending colonists out to scavenge for trash to burn, hunt to keep the colony satiated, and eventually accrue enough weapons and numbers to take back the junk yard. I sketched up some rough concept art and called it a day.

The next day I woke up to get to work. I sat down with my design doc open, fired up Godot, and started to slap some basic components together. However it was during this process that I had the realization that there would be no way I could finish this in time with a proper game loop. I tried to strip it down to the most basic game loop possible, and considered building the same idea out into an idle game format. But even after reworking the doc in various ways, I was losing sight of one of my main goal for the week: whatever I make, just make sure its fun. If you can't find the fun, move on and try something else.

I put this initial concept aside and started to think of something else to make that was more realistic for my skill level. By this point the day was half over, so I decided to do something unconventional for me, which was to just start by drawing something fun. This is how I ended up with the character design seen in the final version. What I ended up with was this little man right here.

![anim_test](/images/posts/anim_test.gif "animation_test") 


At this point there was no design doc aside from a couple bullet points I had written down. 

```
CANDLE RUNNER
- You play as a lit candle guy
- As you run, the candle burns out
- You cannot burn out fully
- You can pick up wax by standing under big melting candles
- You illuminate checkpoints by standing under a wick that lights a flame
- The check point forms you into a fresh tall candle
- Some areas require you to be shorter

```

It was pretty minimal, but it was enough to improvise off of. This moment kind of set the tone for my development process over the rest of the week as well, since it became more of a rapid prototyping session, which allowed me to become more comfortable throwing away ideas that didn't work quite right.

## Rapid and Reactive Prototyping

So, it was settled then. A puzzle game where the "wind" caused by the candle running around would make the flame burn brighter and the wax run out faster. I got to work on creating the mechanics necessary, and had it worked out by the end of Tuesday. 

![anim_test_3](/images/posts/anim_test_3.gif "animation_test")

Over the next couple days I had made all of the other components necessary to build some puzzles out. I built the wax candle spawner that allows you to grow the candle size, the lanterns you need to light, and I even added a bonus spider web obstacle, that you could get past by lighting it on fire with your flame!

There was one small problem that I had realized on Wednesday night though... while the mechanics were coming together to form some semblance of a game... it _still_ didn't _feel_ fun. I didn't want to lose sight of this core goal I gave myself, so I decided to just sit and play around with game feel for the rest of the night. The first thing I realized was, the movement just didn't feel smooth or natural enough. So that would be the first thing to fix. The other key takeaway was, I really didn't like the wax running out based on the player movement. It almost felt like the game was de-incentivizing the player... playing the game? "Don't move around too much, or you just die!" It sounds so obvious now but when I was designing the mechanic on paper, it just simply didn't occur to me. 

It was because of this short late night play test that I decided to instead give the player the choice of what size the candle should be, and base the puzzle on this aspect. Need to fit into a tight area? Shrink down! Need to reach a lantern up high? Find a wax spawner to grow! This was the moment the game went from puzzle game, to puzzle platformer. I decided to focus on the movement aspect more, and make it feel more like a platformer, which is now fully realized the in the final release.


![movement_test](/images/posts/movement_test1.gif "movement_test")

## Polishing it up
For the movement changes, I really wanted the animations and physics to work with each other, to make the player feel like the movement felt natural and believable, all while being somewhat whimsical. I added a slight skewing effect based on the direction and speed of the player, which kind of gives the effect that the candle is a little bit top heavy. 

I also wanted to add some momentum to the movement, so it was here that I was starting to tweak the jump physics, and the amount the player would slide around when changing directions. It was now Thursday night, and I was getting a little concerned that I would not have enough time to create enough levels for the game.


You see, I had held off making levels entirely up until this point. Not out of procrastination, but because I was afraid of wasting time making levels, and then having to either fix them or throw them away if I made any movement tweaks that ended up breaking the levels. I was able to finish the movement tweaks by the end of Thursday, and now I had all of Friday, Saturday, and whatever would be left of Sunday to design levels.

Designing the levels was a massive learning process. I had gone into it thinking "Ah finally, this will be easy! I've been working towards this moment and now all I have to do is the most fun part!"....no it was not easy. 3 Days resulted in 6 levels. Not terrible but I was hoping to reach an even 10 levels. 

I knew I wanted to have some easy levels dedicated to introducing new mechanics. They hard part was filling in the gaps between those introductions, and combining the mechanics learned in creative ways. Puzzles are hard to make! And because of this, I am glad that I ended up dedicating more time to making the movement feel good, because it allowed me to lean on more skill based levels than thinking based levels.

After making all 6 levels, I ended up having an hour left, and that's where I added a basic pause function, sound effects, and some graphics for the itch page.

## Results
My game ended up taking **13th place overall out of 120 submissions**! What was even more exiting to me, I placed **5th in the fun category**! I couldn't believe that my goal I had set for myself for the week actually paid off. 

My take aways from this experience were: 
- Don't spend the time creating a design doc, until you've prototyped enough to determine that the design will actually be fun
- Always be testing your game, always make sure it feels right, don't let an un-fun game become such a big undertaking that you don't even know why you are making it anymore. Figure out how to make it fun as early as possible.

Things I would change based off of user feedback:
- Make the controls a little tighter. I went a little hard on the slidey-ness of the momentum and made the game a little too slippery.
- Levels are going to be harder than you think, because you designed them 