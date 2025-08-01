---
draft: true
title: ESP32-E Flood API!
description: A RESTful API running on a ESP32-E which reads data from a SQLite3 database.
slug: esp32-flood-api
date: 2025-07-31 18:48:00+0000
image: cover.jpeg
categories:
  - opencast
tags:
  - Showcase
  - General
weight: 1
links:
  - title: GitHub
    description: If you wish to look at some of my work, have a look at my GitHub account.
    website: https://github.com/JamieBriggsDev
---

# Introduction

Within the developer community practice at Opencast software, once a month we are given a problem to solve. We call this
the Learn by Doing initiative. These problems can range in difficulty, however, the premise is always the same: try
and solve the problem in a language you're not comfortable with or wish to learn more about. The idea behind this
initiative is that it's a great way to learn a new technology by just tackling a problem head on.

For July's Learn by Doing problem, we were tasked with creating a RESTful API which returns river levels and rainfall
levels. In this post, I'll explore why I went with a Firebeetle 2 ESP32-E microcontroller, how did I implement this
solution, and what I learnt from it.

## The problem

As previously mentioned, the problem revolved around creating a RESTful API. To satisfy this, we were also provided an
[OpenAPI 3.1](https://github.com/JamieBriggsDev/FloodApi/blob/main/openapi.yaml) contact to follow, and a test suite to
ensure that the API we have built works as expected. As for data for this API, I was also provided with a SQLite3 DB
containing data captured
from [Defra's flood monitoring API](https://environment.data.gov.uk/flood-monitoring/doc/reference) that was captured
every day over two years.

To solve this problem, I was allowed to tackle this in any language and technology I wished. The only constraint was
that a test suite should pass when pointed to the RESTful API I have created.

Given I was allowed to use any technology I wished, I decided to tackle this solution using a Firebeetle 2 ESP32-E IoT
microcontroller. While I've owned this microcontroller for a couple of years with the idea to incorporate it into a
future [Home Assistant](https://www.home-assistant.io/) setup, I hesitated due to not having a 3D printer to create
protective enclosures for home deployment. This Learn by Doing challenge
presented the perfect opportunity to work with the ESP32-E as the requirements of the problem (i.e. networking and
database operations) aligned well with my future home automation plans.

This was an incredibly interesting challenge due to the nature of working with a Firebeetle 2 ESP32-E. Although this
microcontroller is low-powered and slow compared to your usual server, it is also very portable. The scope of the
problem was to read river and flood levels from a DB, yet it could easily be extended to write new flood information
to the DB. This could be done either from the
existing [Defra flood monitoring API](https://environment.data.gov.uk/flood-monitoring/doc/reference),
or even contribute new data by being deployed to a real river site and record data in real time.

- [X]  Explain the problem clearly.
- [X]  Include any constraints or rules.
- [X]  Describe why it was interesting or difficult.

## Initial thoughts & my plan (This could be a better title)

### The development environment

Before I started writing any code, I needed to decide what development environment I should use. The usual go-to
option is the [Arduino IDE](https://www.arduino.cc/en/software/)—a beginner-friendly platform primarily designed for
Arduino microcontrollers. Whilst it's great for simple projects, it becomes quite limited
when it comes to dependency management or more complex projects.

As an alternative, I opted for PlatformIO which builds what Arduino IDE can do, but more. PlatformIO IDE
tends to come as a plugin for [CLion](https://www.jetbrains.com/clion/)
or [Visual Studio Code](https://code.visualstudio.com/)
rather than standalone software. Since I am comfortable with the JetBrains suite of IDEs, being able to use
[CLion](https://www.jetbrains.com/clion/) was perfect for me. The PlatformIO IDE also comes with tools for
dependency management, unit testing via [GoogleTest](https://github.com/google/googletest), and remote
upload functionality.

### Key considerations

The next step was to think about libraries I would need to. I had four main considerations:

1. How do I expose a web app?
2. How can I serialize an object into JSON?
3. How can I read an SQLite3 database?
4. How can I read a **22mb** file on a microcontroller with only 4MB of flash memory?

[PlatformIO Home](https://docs.platformio.org/en/latest/home/index.html) is a local user interface provided by the
PlatformIO plugin that helps configure projects. The user interface includes a library search
feature, which I used to address my first two considerations.

For exposing HTTP endpoints (consideration #1), I quickly found a few libraries that provide web server functionality.
For JSON serialization (consideration #2), I came across ArduinoJson, which is widely regarded as the standard library
for handling JSON on microcontrollers.

Addressing considerations #3 and #4 required more effort. I knew I’d be reading my SQLite3 database from a MicroSD card,
so I needed both a compatible [MicroSD SPI module](https://amzn.eu/d/8WP2FOO) and a library capable of reading SQLite
databases directly from
external storage. After some research, I found esp32_arduino_sqlite3_lib, a library developed by siara-cc. It supports
reading SQLite3 databases via various methods, including SPI and MicroSD cards. This library met both of my final
requirements.

### Other useful tools

Although it was not required for this project, I took the opportunity to build a couple of additional tools that proved 
helpful during development of the Flood API.

First, I wanted to experiment with my 
[LCD1602](https://thepihut.com/products/lcd1602-i2c-module?srsltid=AfmBOorLb8S0ax1tl3QRIeDpDyE0VDgfE2X7FBOmzy0fpz9NUIbVgin_)
module. Getting feedback on the LCD in response to hitting various Flood API endpoints proved to be quite handy. This was 
especially useful for showcasing important runtime messages such as IP addresses without having to dig through logs. You'll 
see it in various portions of my code with command like this:

```c++
// Display IP and PORT number
std::ostringstream portMessage;
portMessage << "Port: " << std::to_string(flood::config::PORT);
display->displayText(WiFi.localIP().toString().c_str(), portMessage.str(), common::display::STICKY);
```

I also created a cross-platform logger library. Usually, logging on a microcontroller looks like this:

```c++
Serial.println("My log!");
```

But since I wanted to be run tests natively on my MacBook, I knew this default logging approach wouldn't work outside
of the microcontroller environment. So I built a logger that chooses the appropriate logging approach, including log levels 
depending on the target platform:

```c++
LOG.debug_f("My log: %d", 125); 
```

Whilst these tools weren't part of the core Flood API functionality, they were valuable during development and may be moved
into a common library for reuse in future projects. I won't be covering them in further detail here though.

- [X]  Walk through how you initially thought about the problem.
- [X]  Did you make any assumptions?
- [x]  What was your plan of attack?

## Implementation

> Introduce the implementation paragraph

### Setting up my environment

> Talk about how I set up PlatformIO to flash onto ESP32-E, installing PlatformIO core. Talk about PlatformIO Core
> allows remote upload and test allowing me to develop without being next to the chip.

### The display and logger

> Talk about how this was not required, but was fundamental in helping debug issues; having a display lets me know that
> the ESP32-E is responding to HTTP requests including what params are passed. Logging is also useful for more verbose
> logs.

### Connecting to SQLite3

> Talk about how initially I started implementing a method to read an SD card. Then I found an SQLite3 library which
> handles reading from an SD card directly meaning I could stream data off the MicroSD card.

### Mapping to something useful

> Although it's possible to just map the SQLite3 output to a response, I thought it would be nicer to make a mapper
> which
> serializes the structs returned from the repository. Just makes testing easier.

### Flood routes

> Talk about putting it together using built in Arduino libaries.

- [ ]  Step through how you solved the problem
- [ ]  Include code snippets, terminal outputs, screenshots, or visuals.
- [ ]  Mention challenges or false starts, and how you handled them.

> If this is getting long, or there's lots to talk about, break this down into sub sections.

## What didn't work

### SQLite3 library uses previous version of SQLite3

> Had to create a new SQLite3 file using previous version of SQLite3 by dumping the original file.

### Partition tables

> For a while, the ESP32-E wouldn't work due to size restraints, couldn't run repository and route class same time.

### aWot library and using built in tools instead

> At first I used aWot library, but I found Arduino had stuff built in. Don't slate it.

- [ ]  Share bugs, dead ends, or approaches that failed
- [ ]  This helps humanize your post and is super valuable to readers.

> At first, I tried [method], but it turned out to be a dead end because...

## Final Solution

> Include .gif files showcasing this!

> Include console of test suite.

- [ ]  Present your final working solution
- [ ]  Show the result/ output
- [ ]  Compare it to your initial idea if it evolved
- [ ]  Link to GitHub repo or code sandbox

## What I learned

- [ ]  Highlight key takeaways, new skills, or surprising things.
- [ ]  Mention any tools, libraries, or concepts you explored.

## What I'd do differently

- [ ]  If you revisted the problem, what would you change?
- [ ]  Are there better solutions you discovered later?

## Wrap up

- [ ]  Sum up the experience
- [ ]  Invite others to share their solutions or ask questions
- [ ]  (Optional) suggest similar porblems or resources for readers.
