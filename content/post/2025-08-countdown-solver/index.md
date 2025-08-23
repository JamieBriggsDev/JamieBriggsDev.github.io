---
title: A brute force Countdown solution
draft: true
description: Solving the Countdown number game using brute force, and Rust.
slug: countdown-numbers-game-rust
date: 2025-08-19 09:00:00+0000
image: cover.jpeg
categories:
  - opencast
tags:
  - Showcase
  - General
weight: 1
links:
  - title: Countdown number game
    description: GitHub
    image: https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png
    url: https://github.com/JamieBriggsDev/Countdown-Numbers-Game-Solver
---

# Introduction

- Introduce the problem
- Why was the problem introduced? (Was it for fun? Was there an actual issue at work?)
- Expectations of what I'll cover, and what the reader will learn.

The [Countdown game show](https://en.wikipedia.org/wiki/Countdown_(game_show)) has been on
air since 1982. One of the challenges within the show is to solve a number game. The rules are
simple:

- Pick randomly from two pools of cards **six** numbers, one pool featuring small numbers (**1-10**), and the other pool
  featuring large numbers (**25**, **50**, **75**, **100**).
- After the numbers are chosen, a number is picked from **100**-**999**.
- Using only addition, subtraction, multiplication, and division, try and make the resulting number using the six
  numbers chosen.

For example (and using the cover image of this post as the example), if the six numbers were **5,2,3,10,6,4**
and the number to make was **609**, you could solve it like this:

1. $(((4 + 10) * 6) + 3) * (2 + 5) = 609$
2. $((14 * 6) + 3) * 7 = 609$
3. $(84 + 3) * 7 = 609$
4. $87 * 7 = 609$

## The problem

For August's Learn by Doing challenge, I was tasked with implementing a Countdown number game solver. There was a
tweak to the original rules—Instead of multiplication and division, we could instead do both a bitshift left, and
bitshift right. Like the previous [Learn by Doing challenge](../esp32-flood-api), this could be done in any language I
wanted.

## Where to start

Before I dove into this challenge, I needed to understand the problem and then come up with an implementation.

In my plan, I decided that I was going to implement a brute force solution. This meant I needed to use a language that
could efficiently go through all possibilities efficiently. With this in mind, I decided to
use [Rust](https://www.rust-lang.org/). I went with Rust
as it is designed to be both fast and efficient. It's also a language I've been wanting to learn more about as the first
language
I learned was C++ during my Computer Games Programming degree, and I still use C++ to this day for various games
programming style projects or Advent of Code. From what I understand, it's also just as efficient as C++ and has a
much more modern syntax.

Next I needed to come up with a way to solve the problem. What I did know is that you do not need to solve the number 
game with all the numbers, for example:

> With numbers **1,11,50,75,2,3** and the number to make being `12`: $1 + 11 = 12$

With this in mind, I decided that it would be easier to implement a solution which could solve
the number game with just two numbers, then grow from there.

## Implementation

- Step through how you solved the problem
- Include code snippets, terminal outputs, screenshots, or visuals.
- Mention challenges or false starts, and how you handled them.

If this is getting long, or there's lots to talk about, break this down into sub sections.

## What didn't work

- Share bugs, dead ends, or approaches that failed
- This helps humanize your post and is super valuable to readers.

> At first, I tried [method], but it turned out to be a dead end because...

## Final Solution

- Present your final working solution
- Show the result/ output
- Compare it to your initial idea if it evolved
- Link to GitHub repo or code sandbox

## What I learned

- Highlight key takeaways, new skills, or surprising things.
- Mention any tools, libraries, or concepts you explored.

## What I'd do differently

- If you revisted the problem, what would you change?
- Are there better solutions you discovered later?

## Wrap up

- Sum up the experience
- Invite others to share their solutions or ask questions
- (Optional) suggest similar porblems or resources for readers.

