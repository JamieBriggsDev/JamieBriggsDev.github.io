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

Within the developer community practice at [Opencast software](https://opencastsoftware.com/), once a month
we are given a problem to solve. We call this
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

## Planning the Environment and Tooling

Before diving into the Flood API implementation, I spent some time setting up my development
environment and selecting the correct tools which would enable me to effectively create the Flood API.
This section outlines the approach I took, the libraries I researched, and a couple of side tools I
built along the way that ended up being quite useful, but not essential.

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

#### Working remotely

Although not essential for microcontroller development itself, PlatformIO’s remote capabilities turned out to be
particularly useful for this project. By using [PlatformIO Core](https://docs.platformio.org/en/latest/core/index.html)
(their CLI tool), I was able to connect my ESP32-E to a host machine (an old 2012 iMac in my case) and start a remote
agent like this:

```shell
pui remote agent start
```

With the agent running, I could upload builds, monitor output, and even run tests remotely from any other machine using
the PlatformIO plugin. This setup was especially helpful since my main development machine is a MacBook Pro. It allowed
me to work from anywhere without needing to have the microcontroller physically connected at all times.

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
external storage. After some research, I
found [esp32_arduino_sqlite3_lib](https://github.com/siara-cc/esp32_arduino_sqlite3_lib),
a library developed by **siara-cc**. It supports
reading SQLite3 databases via various methods, including SPI and MicroSD cards. This library met both of my final
requirements. I did find that this library is not found within PlatformIOs library search. However, I could import it
into my project via [git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules).

### Other useful tools

Although it was not required for this project, I took the opportunity to build a couple of additional tools that proved
helpful during development of the Flood API.

First, I wanted to experiment with my
[LCD1602](https://thepihut.com/products/lcd1602-i2c-module?srsltid=AfmBOorLb8S0ax1tl3QRIeDpDyE0VDgfE2X7FBOmzy0fpz9NUIbVgin_)
module. Getting feedback on the LCD in response to hitting various Flood API endpoints proved to be quite handy. This
was
especially useful for showcasing important runtime messages such as IP addresses without having to dig through logs.
You'll
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
of the microcontroller environment. So I built a logger that chooses the appropriate logging approach, including log
levels
depending on the target platform:

```c++
LOG.debug_f("My log: %d", 125); 
```

Whilst these tools weren't part of the core Flood API functionality, they were valuable during development and may be
moved
into a common library for reuse in future projects. I won't be covering them in further detail here though.

## Implementation

> [!IMPORTANT]  
> Missing implementation paragraph

### Initial setup

> [!IMPORTANT]  
> Talk about connecting to WiFi, and connecting MicroSD SPI module to ESP32.

### Connecting to SQLite3

As [previously mentioned](#key-considerations), I found a crucial SQLite3 library specifically for ESP32
microcontrollers which supports accessing SQLite3 database files via SD cards. This library contains a good example of
how to read a file from an MicroSD card connected via the SPI module. Before actually reading the data from the
database file, some setup is required.

To begin, I needed to initialize the SPI bus, and SD library. I also make sure that the SD card is readable:

```c++
void FloodRepository::init()
{
  // Step 1: Initialize SPI and SD
  LOG.info("Initializing FloodRepository");

  LOG.debug("Beginning SPI");
  SPI.begin();
  LOG.debug("Beginning SD ");
  while (!SD.begin(config::MICRO_SD_CS_PIN))
  {
    LOG.error("Card Mount Failed");
  }

  uint8_t cardType = SD.cardType();
  if (cardType == CARD_NONE)
  {
    LOG.error("No SD card attached");
    throw std::runtime_error("No SD card attached");
  }

    ...
```

Next, I just make sure that the file I am going to be reading the flood data from is actually readable:

```c++
  // --- Continued: Still inside FloodRepository::init() ---
  
  // Step 2: Check the database file exists
  if (SD.exists(this->m_dbPath))
  {
    LOG.debug_f("Database file '%s' exists on SD card", this->m_dbPath);
    File file = SD.open(this->m_dbPath);
    LOG.debug_f("File size: %d", file.size());
    file.close();
  }
  else
  {
    LOG.error("Database file not found on SD card");
  }

    ...
```

Now that I know the SD card is mounted, and the database file is read, it's time to initialize the SQLite3 library:

```c++
  // --- Continued: Still inside FloodRepository::init() ---
  
  // Step 3: Initialize SQLite3
  LOG.info("Initializing SQLite3...");
  int initialize = sqlite3_initialize();
  if (initialize != SQLITE_OK)
  {
    LOG.error_f("Failed to initialize SQLite3: %s", initialize);
    throw std::runtime_error("Failed to initialize SQLite3");
  }
  
    ...
```

Once the library has been initialized, I open the database file:

```c++
  // --- Continued: Still inside FloodRepository::init() ---
  
  // Step 4: Open database file
  LOG.debug("Opening DB...");
  std::stringstream vfsPath;
  vfsPath << "/sd" << this->m_dbPath;
  if (openDb(vfsPath.str(), &m_floodDb) != SQLITE_OK)
  {
    LOG.error("Failed to open database");
    throw std::runtime_error("Failed to open database");
  }
  LOG.info_f("Connected to database!");
  
    ...
```

Once that is done, the database is actually in a usable state. there is one more thing I do during setup however to
better optimize my future calls to the database, and that is cache all station names found within the database to avoid
doing additional calls:

```c++
  // --- Continued: Still inside FloodRepository::init() ---
  
  // Step 5: Cache station names
  LOG.debug("Caching station names");
  auto stationNames = this->getAllStations();
  if (stationNames.empty())
  {
    LOG.error("Failed to get station names");
    throw std::runtime_error("Failed to get station names");
  }

  LOG.info("Completed initialization for FloodRepository");
}
```

### Mapping to something useful

Since my API needs to return JSON, I decided to create an intermediary service to map the objects returned
by my repository layer into JSON. Using [ArduinoJson](https://arduinojson.org/) made this process painless and
straightforward:

```c++
JsonDocument doc;
for (const auto& rainfallReading : rainfallReadings)
{
    JsonDocument reading;
    reading["timestamp"] = rainfallReading.timestamp;
    reading["level"] = rainfallReading.level;
    reading["station"] = rainfallReading.station;
    if (!doc["readings"].add(reading))
    {
      LOG.error("Failed to add reading");
      throw std::runtime_error("Failed to add reading");
    }
}
```

To convert this `JsonDocument` into an easy-to-read JSON output, ArduinoJson provides a handy function:

```c++
std::string json;
serializeJsonPretty(doc, json);
LOG.debug(json);
```

### Flood routes

#### Setting up WebServer

Within the Arduino core libraries for ESP32 is the
[WebServer](https://github.com/espressif/arduino-esp32/blob/master/libraries/WebServer/src/WebServer.h) class. This
class is a "dead simple we-server" which can support `GET` and `POST` HTTP requests; although limited, it is
perfect and lightweight for my use case of having a read-only API.

The setup for this web server is very simple; all you need is to register a handler to a URI:

```c++
FloodRoutes::FloodRoutes(common::display::IDisplay* display, db::IFloodRepository* flood_repository,
                         mapper::IFloodMapper* flood_mapper) : m_server(config::PORT), m_display(display),
                                                                 m_floodRepository(flood_repository),
                                                                 m_floodMapper(flood_mapper)
{
  // Setup routes
  LOG.debug("Setting up routes...");
  // GET: /river
  m_server.on("/river", HTTP_GET,
              [this]
              {
                // Handles the response
                this->river();
              });
    
    ...
```

For handling a URI with a path parameter, `UriBraces` becomes useful as a matcher, and then the path parameter
can be extracted via the `WebServer`:

```c++
// --- Continued: Still inside FloodRoutes::FloodRoutes() ---

// GET: /rainfall/{stationName}
  m_server.on(UriBraces("/rainfall/{}"), HTTP_GET,
              [this]
              {
                const auto pathArg = m_server.pathArg(0);
                const std::string stationName(pathArg.c_str(), pathArg.length());
                
                // Handles the response
                this->rainfallStation(stationName);
              });
    ...
```

Finally, with WebServer setup, the server can be started:

```c++
// --- Continued: Still inside FloodRoutes::FloodRoutes() ---

  // Begin server
  LOG.debug("Starting server in FloodRoutes");
  m_server.begin();
}
```

#### Returning content on WebServer

With `WebServer` now initialized, I needed to send a response to the client. As you may have seen in the
[previous section](#setting-up-webserver), as part of handling a URI, I called a function for both `/river` and
`/rainfall/{stationName}` which handles the response. Both functions are similar in functionality due to how much
logic is delegated to the [repository layer](#connecting-to-sqlite3) and [mappers](#mapping-to-something-useful).

The first step is to get any request parameters passed as part of the request. I created a handy function
which returns the value of a request parameter to help facilitate this. The `getQueryParameter`
function checks if a request param exists, and extracts it if it does. If the request
param does not exist, then return a default value:

```c++
std::string FloodRoutes::getQueryParameter(const std::string& param, const std::string& defaultValue)
{
  // Build string for LCD
  std::stringstream paramDisplay;
  paramDisplay << "Param " << param;
 
  // Check if request param is found on request
  const String paramName(param.c_str());
  if (!m_server.hasArg(paramName))
  {
    LOG.debug_f("Param %s not found", param.c_str());
    displayParamOnLCD(paramDisplay.str(), defaultValue.empty() ? "EMPTY" : defaultValue);
    // Return the default value if request param does not exist
    return defaultValue.empty() ? "" : defaultValue;
  }

  // Extract request param value
  const auto paramValue = m_server.arg(paramName);
  std::string result(paramValue.c_str(), paramValue.length());
  
  displayParamOnLCD(paramDisplay.str(), result);
  
  return result;
}
```

With `getQueryParameter()`, I could get the request parameters at the start of a request:

```c++
void FloodRoutes::river()
{
  LOG.info("/river requested");
  m_display->displayText("Calling", "/river", common::display::FLASH);
  
  // Get request parameters
  // Get the date parameter
  const std::string date = getQueryParameter("start");
  // Get limit parameter with default value, and convert to int
  const int limit = std::stoi(getQueryParameter("page", "1"));
  // Get page parameter with default value, and convert to int
  const int pagesize = std::stoi(getQueryParameter("pagesize", "12"));
  
    ...
```

Next, data needs to be fetched from the flood repository:

```c++
// --- Continued: Still inside FloodRoutes::river() ---

  const std::vector<db::RiverReading> readings 
    = m_floodRepository->getRiverReadings(date, limit, pagesize);
    
    ...
```

Finally, using my mapper, I could turn this data into JSON and return a response using `WebServer`:

```c++
// --- Continued: Still inside FloodRoutes::river() ---

  // Convert to JSON
  const JsonDocument doc = m_floodMapper->getFloodData(readings);
  std::string json;
  serializeJsonPretty(doc, json);

  const char* result = json.c_str();


  return m_server.send(200, "application/json", result);
}
```

The function for getting rainfalls for a station is very similar, only it has the addition of the path parameter
from the URI `/rainfall/{stationName}`, and a quick check to ensure that the station passed exists:

```c++
void FloodRoutes::rainfallStation(const std::string& stationName)
{
  // Get path param station name
  std::stringstream fullPath;
  fullPath << "/rainfall/" << stationName;
  LOG.info_f("/rainfall/{station} requested using %s", stationName);
  m_display->displayText("Calling", fullPath.str(), common::display::FLASH);

  // You can validate against your known stations
  if (!m_floodRepository->stationExists(stationName))
  {
    m_server.send(404, "application/json", R"({"error": "Invalid station name. Station not found."})");
    return;
  }

  // Get request parameters
  // Get the date parameter
  const std::string date = getQueryParameter("start", "2022-12-25");
  // Get limit parameter with default value
  const int limit = std::stoi(getQueryParameter("page", "1"));
  // Get page parameter with default value
  const int pagesize = std::stoi(getQueryParameter("pagesize", "12"));


  const std::vector<db::RainfallReading> rainfall_readings =
      m_floodRepository->getStationRainfallReadings(stationName, date, limit, pagesize);

  // Convert to JSON
  const JsonDocument doc = m_floodMapper->getRainfallReadings(rainfall_readings);
  std::string json;
  serializeJsonPretty(doc, json);

  return m_server.send(200, "application/json", json.c_str());
}
```

## What didn't work

> [!IMPORTANT]  
> Missing introduction paragraph

### SQLite3 library version conflicts

When I first set up the [repository layer](#connecting-to-sqlite3), I ran into an issue with
reading the SQLite3 database file. Initially, I suspected it was my own error, but testing with an example
database from the [SQLite3 library](https://github.com/siara-cc/esp32_arduino_sqlite3_lib) showed that the issue
was specific to the flood database file I was using.

I knew the file itself was fine - others working on the same challenge were using it without issues - so my next
suspicion was a compatibility problem with the library. Checking the database version confirmed the file was built with
SQLite `3.43.2`:

```shell
% file flood.db
flood.db: SQLite 3.x database, last written using SQLite version 3043002, writer version 2, read version 2, unused bytes 12, file counter 2378, database pages 6170, cookie 0x18, schema 4, UTF-8, version-valid-for 2378
```

The library I used was also based on this version, so I expected it to work. However my application logs showed that
this
was not the case:

```shell
[DEBUG] Beginning SPI
[DEBUG] Beginning SD
[DEBUG] Database file '/flood.db' exists on SD card
[DEBUG] File size: 25272320
[INFO] Initializing SQLite3...
[DEBUG] Opening DB...
[INFO] Opened database successfully: /sd/flood.db
[INFO] Connected to database!
[DEBUG] Caching station names
[DEBUG] Preparing query: SELECT * FROM StationNames
[ERROR] Failed to prepare statement: file is not a database
```

Locally, `flood.db` opened without issue. So I decided to recreate it myself. First, I dumped the database:

```shell
% sqlite3 flood.db .dump > flood_dump.sql 
```

I then downloaded [SQLite 3.43.2](https://github.com/sqlite/sqlite/releases/tag/version-3.43.2),
and built it from source:

```shell
% cd sqlite-version-3.43.2/
sqlite-version-3.43.2 % ./configure 
sqlite-version-3.43.2 % make 
```

Finally, I created a new database from the dump:

```shell
sqlite3 flood_recreated.db < flood_dump.sql
```

Using this newly created version of the flood database resolved the problem:

```shell
[INFO] Initializing SQLite3...
[DEBUG] Opening DB...
[INFO] Opened database successfully: /sd/flood_recreated.db
[INFO] Connected to database!
[DEBUG] Caching station names
[DEBUG] Preparing query: SELECT * FROM StationNames
[DEBUG] Stepping through statement
[DEBUG] Finalizing. Found 11 results.
[INFO] Completed initialization for FloodRepository
```

I'm still not so sure as to why the original file failed with this library, but recreating it allowed me to move
forward.

### Partition tables

> [!IMPORTANT]  
> For a while, the ESP32-E wouldn't work due to size restraints, couldn't run repository and route class same time.

## Final Solution

> Include .gif files showcasing this!

> [!IMPORTANT]  
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
