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

Within the developer community practice at Opencast software, once a month we are given a problem to solve. We call this
the Learn by Doing initiative. These problem can range in difficulty, however the premise is always the same - try
and solve the problem in a language you're not comfortable with or wish to learn more about. The idea behind this
initiative is that it's a great way to learn a new technology by just tackling a problem head on.

For July's Learn by Doing problem, we were tasked with creating a RESTful API which returns flood data from a SQLite3
database. We were also given an OpenAPI 3.1 contact to follow, and a test suite to ensure that the API we have built
works as expected.


r4rsfdddd


Here is some code <3

```c++
void loop()
{
  // Monitor heap memory usage
  LOG.debug_f("Free Heap: %d KB, Out of: %d KB", ESP.getFreeHeap() / 1024, ESP.getHeapSize() / 1024);

  flood_routes->loop();

  delay(1000); // Wait 5 seconds
}
```