---
title: Star Fox PS4 remake
description: A beloved franchise, recreated for the PlayStation 4 
        for educational purposes.
slug: 2017-07-lyat-wars
date: 2017-07-25
image: lyat-wars-ps4.png
categories:
    - computer-games-programming-bsc
tags:
    - C++
    - Game Development
    - Past Work
    - University
weight: 1
---

> This post was written 9th February 2025, as a retrospective post to port my portfolio from Adobe Portfolio to this blog.

One of the modules in my Computer Games Programming Degree at [Northumbria University](https://www.northumbria.ac.uk/) involved myself creating a game using the PlayStation 4 development kits. A friend of mine had the initiatice to recreate a video game from his childhood. The marking criteria for this module only involved gameplay elements such as loading `.obj` files, rendering textures and AI. This meant no marks were ever given for creativity and story telling; theoretically I could've produced unplayable rubbish, but if it had aspects of gameplay like AI or even 3D rendering, it would've scored marks. So I copied his thought process because if I recreated a game from my childhood as well, I would spend less time focusing on what to create and just focusing on gameplay mechanics.

With this in mind, I needed to pick something which wasn't too complicated, but also something I would be interested in recreating; if I thoroughly into recreating a game I loved, chances are I would also not mind spending hours creating the game as a passion project. In the end, I went with [Star Fox 64](https://en.wikipedia.org/wiki/Star_Fox_64) (I knew it as Lyat Wars growing up due to Nintendo wanting to avoid confusion with a German company named "StarVox").

To learn how to create games for the PlayStation 4, there was a very helpful tutorial given to the students within my course which went into how to load and render `.obj` files, to shader work and it's relativity to GLSL, to how to program the controller and it's behaviour. A lot of the work which was done within this tutorial was very much a tutorial on how to write a game in the same style as a lecturer, meaning everyone had the same game structure, but it was a very good start into the various aspects of creating a 3D game from scratch. Retrospectively, the PlayStation 4 actually did a lot of the rending for you with it's own SDK compared to OpenGL, but you still had to provide the SDK with your game data and logic. However with the basics understood, there was enough to make a start on the Star Fox 64 remake. Not too important, but games written to use the official PlayStation 4 development kits were written in [C++](https://en.wikipedia.org/wiki/C%2B%2B).

What I remembered most of Star Fox 64 from my childhood was how fun it was to control the player ship, and who can forget performing a [barrel roll](https://www.google.com/search?q=do+a+barrel+roll)! So I primarily focused on the handling of the player ship. I also knew I only had time to create a tech demo of sorts. The majority of the original game is actually an on the rails shooter, however I figured at the time it was easier to just program a free flying ship just like in the boss areas of the game. This way I could focus on the ship controls, and then later do the compilated rails shooter stuff if I had time; I didn't so good call!

In the end, I ended up creating something which consisted of a flat plane where enemies would constantly spawn in the middle, and the player could shoot them down. The Player also had health and when you died, it would be game over. It was quite a simple demo, but the work to get to that point was extensive. I have linked a clip of the game below. Unfortunately the clip doesn't show all the maneuvers you could do in the game and I no longer have access to the PlayStation 4 dev kits to record the game again.

<iframe width="560" height="315" src="https://www.youtube.com/embed/9XEttKAj2ak?si=iP7Pk1-EYiZJpoPU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

In the future, I would like to see if I could play the game in a PlayStation 4 emulator . This way I could look back on my code and perhaps have a retrospection on what I've learnt since working on this project.

This project was done under NDA. I still have the source code for it, but until there is a clear indication that it's okay to release the source code, it will remain private.
