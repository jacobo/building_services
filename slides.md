!SLIDE
# Services

!SLIDE
(diagram of some EY services)

.notes Here's a picture of some of the services we have at EY and how they interact. You can see this talking to that, bla bla

!SLIDE
(diagram of Addons pieces)

.notes Let's zoom in on the Addons part and see some more detail. Notice that Addons actually has 4 APIs! 1. public HTTP API for partners 2. public mock mode Ruby API for partners 3. private API back to cloud dashboard. 4. private mock mode Ruby API with dashbaord

!SLIDE
# Lesson 0:
## Every API client needs a mock mode.

.notes there are lots of way to actually implement a mocked mode. One of the coolest examples is fog.

!SLIDE
#Fog
(picture of Wes)

!SLIDE
#Fog mock mode
(Mock Mode Code Example from fog)

.notes here's an example of using fog to call AWS web services in test. We ask amazon to boot an instance, and it happens immediately, and fog responds as if it worked, there's even a delay feature

!SLIDE
(diagram of Addons pieces)

.notes but back to the diagram. If we know this is what we want to build, then let's start here.

!SLIDE
# A quick demo
(series of screen shots)

.notes here's a quick demo of how the addons program works, so you can get an idea of what I was trying to build

!SLIDE
# Starting point requirements
* Write as little code in Cloud Dashboard as possible
* Use case: partners register their services with Addons service
* Use case: Addons service makes them available to users on Cloud Dashboard

.notes so we have this starting point of simple use case

!SLIDE
# Let's write some code

* Cloud Dashboard (awsm)
* Add-ons Service (tresfiestas)
* Fake Service (lisonja)
* Public API client (ey_services_api)
* Private API client (ey_services_api_internal)

.notes This was a reasonable mistake to make. At Engine Yard we always like to ship things to production as early as possible. We hide them behind a feature flag and so it doesn't matter if they are live.  So reasonably I thought to immediately make all of these things.

!SLIDE
#I wrote a rails app for Add-ons

!SLIDE
#I wrote an API client for the public API

!SLIDE
#I wrote a sinatra app for the fake service that used the API

!SLIDE
#I wrote an API client for the private API

!SLIDE
#I used the private API client in Cloud Dashboard

!SLIDE
(happy face)

.notes so I thought this was great right. I just wrote all this code, but only like 1/5 of it had to be in AWSM (the slow monolith), so for the most part I was writing greenfield code, and my TDD went fast.

!SLIDE
# Lesson 0.5:
## Bundler path

.notes and here's a little lesson for you... you can even run all this stuff locally with bundler path, and you can use bundler git so you don't have to publish your gems while you are still in development

!SLIDE
(500 page, sad face)

.notes so I got all this into production

!SLIDE
# Mistake 1
## Working in 5 repositories at once.

* No integration tests.
* No visibility into the live running system.

!SLIDE
# Lesson 1:
## Spike it!
(picture of Tim)

.notes the term spike generally refers to throwaway code that you write to try out an idea with the goal of figuring out all the questions you didn't know to ask. What will you stumble on. What assumptions can you discover that are wrong before you go too far down a given path.  Usually you don't write tests, or you write very minimal tests.

!SLIDE
# My Spike

* Step by step capybara interactions
* You can actually run it.

!SLIDE
# Demo

.notes show the directory structure
.notes show the running "spike"

!SLIDE
# Lesson 1.5
## Use artifice

!SLIDE
# Lesson 1.6
## Don't use artifice
## Use rack-client

!SLIDE
#More use cases

.notes So we went through all these various use cases with this basic framework of what I called the spike. But development could be easier. A big problem was that I was pairing and it was hard to make my pair understand everything that was going on, because my only tests were too high level.

!SLIDE
# Lesson: Visualize it

.notes so I wrote this little middleware that let me see a trace of the API traffic

!SLIDE
# Demo

.notes show running a live test with the visualizer on

!SLIDE
# Back to working in 5 repositories

!SLIDE
# Lesson: Verify your mock mode

.notes there are different approaches to doing this... In my case I wrote tests in my API client that could run against either the "mock" server OR the "real" server. I used 2 gemfiles and the "internal" one actually includes the full Addons project as a gem.

!SLIDE
# Lesson: Ship it!

.notes this is what I tried to do originally, but I needed to heed the other lessons first. With verifiable API clients and mock modes I could work in a single repository at a time, and be confident that if my tests pass, things will work when I deploy to production.

!SLIDE
# Shipping is just the start!

!SLIDE
# Iterate

!SLIDE
# Write Documentation
## And keep it up to date

.notes documentation has a price (just like code) the more you write, the more you need to maintain

!SLIDE
# Generate Documentation

!SLIDE
# Share your errors

.notes we actually show partners a debug console with exception traces for recent errors.

!SLIDE
# Don't use rails for API controllers, use sinatra

.notes because it's not as portable. Because you can re-use mini sinatra apps as your fake too.
.notes because nested resource routes are a pain and you end up not even providing every REST verb on every one of them.

!SLIDE
# Write your own presenter

.notes because it's hard to follow what as_json is doing with nested models

!SLIDE
# Your domain model should not match your database model

.notes because at some point you'll probably want to change one without changing the other
.notes because your API client doesn't need to know everything

!SLIDE
# The best identifier is a URL

.notes because then you don't need to construct URLs
.notes REST (and thus usefulness) will likely fall out.
.notes example of the URL you get identifying an account, you can GET to for the information again.
.notes the URL to POST to register a service is also a listing of services you've registered

!SLIDE
# Involve your designers early

.notes when your API is the backend for a web interface, changes to what the designers what to show on what screen, will dictate changes to what your various API endpoints need to return

!SLIDE
# One piece of data should have 1 authority

.notes delegate to that authority when you need that information

!SLIDE
# Follow conventions. Break conventions. Establish conventions.

.notes a of conventions I followed turned out to be mistakes: to_json, isolation testing. But I needed to build new conventions in their place: presenter, mock mode orchestrator.
.notes we use HMAC for auth on all our services, all of the TF data is represented with model name to model attributes, with URLS outside the inner hash. Everything is JSON post body and JSON response.

!SLIDE
# Documentation is hard

.notes demo of DDD









