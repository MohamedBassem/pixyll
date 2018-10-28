---
layout: post
title: The Crappy Game
summary : "Let me tell you about the crappiest thing I've ever built. A multiplayer 3D OpenGL game with a lot of hacks."
description : "Let me tell you about one of crappiest thing I've ever built. A multiplayer 3D OpenGL game with a lot of hacks."
date: '2018-10-28T19:00:00+0000'
author: Mohamed Bassem
image: /img/raafat-elhagan/game.png
tags:
  - graphics
  - opengl
  - game
  - hack
  - fun
categories :
---

Let me tell you the story of the crappiest thing I've ever built, which is at the same time one my favorite projects. It was someday in December 2014. My friend Farghal ([@medo](https://github.com/medo)) and I were in a team for the project of the computer graphics university course. The project was about building any 3D game using OpenGL. Sounds good? yeah, except that we procrastinated until it's 24 hours before the deadline, and ... we knew nothing about how OpenGL works. Well, actually we knew nothing about computer graphics in general.

We knew that if we want to get the full grade, we will have to build something impressive to cover up for the poor graphics aspect of the game. We decided to build a multiplayer first person shooter. Yes.

[![Game](/img/raafat-elhagan/game.png)](/img/raafat-elhagan/game.png){:: data-lightbox="img3"}

In this post I'll walk you through some of the hacks that we did to deliver this game in time. The game's code is [open source](https://github.com/medo/raafat-elhagan). Normally, I'd say go check it out, but in this specific case, please don't. I swear we both code way better than this right now, this was 4 years ago.

## Gameplay

You start the game on a rectangular plane with some walls, obstacles and the other player (represented as a cute yellow cuboid). You can't go through those obstacles except in a wide variety of corner cases that we didn't have time to handle. You can jump, gravity pulls you back down ... most of the time. Your goal is to shoot the other player multiple time until their health goes to zero, that's when you get a point and the game restarts.

How do we calculate whether the player got hit or not? I'm glad you asked.

[![Bullet Math](/img/raafat-elhagan/bullet.png)](/img/raafat-elhagan/bullet.png){:: data-lightbox="img3"}

We move the bullet one step each iteration in the direction of the shot for 1000 iterations. If the other player intersects with the bullet at some iteration, it's a hit. The bullet stops whenever it hits a wall. Well, the bullet can go through the wall if the bullet moves past the wall in a single iteration. A small bug, that nobody will notice.

[![Jump](/img/raafat-elhagan/jump.gif)](/img/raafat-elhagan/jump.gif){:: data-lightbox="img3"}

"***It's not a bug, it's a feature***" was our motto. We noticed that you can jump while you are still in the air. There was no quick way to prevent that, so we though it might be cool to allow players to fly. Totally intended.

[![Fall](/img/raafat-elhagan/fall.gif)](/img/raafat-elhagan/fall.gif){:: data-lightbox="img3"}

Did I mention that our world is infinite? Only when you are falling. You can fly back up though.

## Sound

Who plays a game without sound effects? The game plays a sound effect whenever a shot is fired, and whenever you are hit.

How to play mp3 files in c++? DON'T TRY THIS AT HOME (Although it's cross platform).

[![Shoot Sound](/img/raafat-elhagan/shot.png)](/img/raafat-elhagan/shot.png){:: data-lightbox="img3"}

What about background music?

[![Background Music](/img/raafat-elhagan/background_music.png)](/img/raafat-elhagan/background_music.png){:: data-lightbox="img3"}

Users must have a "rafat-elhagan.mp3" file on their desktop. We don't distribute it with the game. Copyrights. Or we might have missed moving it to the repo back then.

## Networking

Now comes what makes our game special. The networing part. Our game is a multiplayer game that is supposed to be played over local network. There was one single problem, we didn't know how to do networking in c++. We had to hack our way through.

[![Networking](/img/raafat-elhagan/bash.png)](/img/raafat-elhagan/bash.png){:: data-lightbox="img3"}

Let me explain what's going on in here. The game binary reads from stdin and writes to stdout. Each player starts a TCP server that dumps the data it gets to Stdin of the game. Stdout of the game gets periodically sent to the TCP server of the other player. So whatever one player's binary writes to stdout, will eventually be read by the other player's binary. Duplex communication channel established. Please don't judge, we only had 24 hours!

By now you've probably guessed what's being communicated. Whenever a binary is started, two threads are spawned. One thread that periodically dumps the game state (location, is shooting, a hit, etc.) to stdout. The other thread reads from stdin and applies this information to its local state (playing the sound effects). That's how the two instances are kept in sync.

[![Protocol](/img/raafat-elhagan/threads.png)](/img/raafat-elhagan/threads.png){:: data-lightbox="img3"}

## Demo

<iframe width="560" height="315" src="https://www.youtube.com/embed/2RfFhbd-tro" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>


Some of the clips in this demo are from an over-the-internet match. That's why some of them are a bit laggy, the game was designed only for local network matches 😌

## Grade

What we ended up with was actually a fun crappy game to play. We got the full grade with a bonus that overflowed and filled our lost quiz grades. IN 24 HOURS. SUCCESS.

## The Not So Happy Ending

We built this game. We wrote its logic. We knew how to cheat. We knew how to trigger certain bugs. But we lost the first real match against our friend [Rami](https://github.com/rami-khalil). Apparently, we suck at playing our own game.

