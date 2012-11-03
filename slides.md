!SLIDE[bg=pictures/surfpier.jpg]
### Jacob Burkhart
<br/><br/><br/><br/>
<br/><br/><br/><br/>
<br/><br/><br/><br/><br/>
## github.com/jacobo/building_services
## @beanstalksurf

.notes Hi there. This is me. I'm Jacob Burkhart. You can tweet at me there or download this presentation.  

!SLIDE[bg=pictures/whatamidoing.jpg]
### What do I know?

.notes This is the first time I've given a talk in Israel. It's also the first time I've given a technical talk. Last time I gave a talk at a ruby conference I talked about Surfing. This talk is going to be very different, but there's one key point that remains the same. This is a story of my experiences and what has worked for me. I'm not here to give you instructions on what to do. I only hope to inspire you. Maybe you'll face some problems similar to the ones I'm going to describe. Hopefully sharing my experience will help you to solve them better than I did.

!SLIDE
<br/>
### Engine Yard
(picture of a train in the clouds from J-Bird)

.notes Engine Yard. This is where I work. Thank you to them for sponsoring this conference and sending me here. And this is what this talk is about. Because at Engine Yard we have a fairly large and complicated product...

!SLIDE[bg=diagrams/ey_soa_overview.png]

.notes and it consists of a lot of services.  We DO have the monolithic app syndrome. (that's our cloud dashboard in the middle). But we've been gradually been adding new features. And whenever we do we look for ways they can be added as separate services.  By the way, I colored those 3 boxes because those are the services that I'm going to talk about today.

!SLIDE[bg=screenshots/terminal_colors.png]
<br/>
### Side Note: Use colors

.notes And it can actually help to think of things in terms of colors. This is a typical screenshot of my terminal, the colors help me to remember what I was doing (and on which app).

!SLIDE[bg=diagrams/just_addons.png] bullets rightside-bullets incremental
* Minimize additions to Cloud Dashboard
* Iterate quickly
* Decouple

.notes Back to addons. In the higher level diagram I drew this. So why? Why not just connect our 3rd party services directly to Cloud Dashboard. There are some design goals here.

!SLIDE[bg=pictures/distributed_objects.jpg]
### Distributed Objects

.notes Let me tell you about distributed objects. This is about relationships that go across systems.

!SLIDE[bg=pictures/solowave.jpg] align-left shadowed
# Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.

!SLIDE[bg=diagrams/datamodel_simple.png]

.notes this is a simple example of the data model in our case.

!SLIDE[bg=diagrams/datamodel.png]

.notes this is the diagram I actually created and maintained during development

!SLIDE[bg=pictures/pointing.jpg]
### APIs

.notes The keyword in API is interface. When I talk about APIs I'm talking about interfaces across systems. But was is a system. When you write a single rails app you don't think about your APIs as much but you do have objects interacting. When your system spans codebases and run on separate machines, you have to formalize your APIS more. But they are there even in a single rails app.

!SLIDE[bg=diagrams/just_addons.png]

.notes back to the diagram of the 3 systems

!SLIDE[bg=diagrams/just_addons_really.png]

.notes Here's what it looks like with the cross-app APIs defined. Notice that Addons actually has 2 distinct APIs for public and private, and then each API has two parties. For convenience we call one the server and one the client, but they do more than that.

!SLIDE[bg=diagrams/addons_workflow.png]

.notes here's a workflow of a user using the cloud dashboard to enable a service for their app. This one UI action triggers a chain of calls and responses that create new data in each system.

!SLIDE[bg=diagrams/addons_workflow.png]
<br/><br/><br/><br/><br/><br/>
### Where To Start?

!SLIDE[bg=pictures/complexity.jpg]
### How do you solve complexity?

!SLIDE bullets incremental
# Isolation!
![thing](pictures/isolation.png)

* Solve simple problems in isolation
* Combine the simple solutions into a complex solution

!SLIDE[bg=diagrams/addons_workflow.png] black align-left
## from this

.notes we we'll go from this

!SLIDE[bg=diagrams/addons_workflow_mock.png] black align-left
## to this

.notes so we use a mock

!SLIDE
<br/><br/><br/>
<br/><br/><br/>
# Every API Client needs a mocked mode.

.notes there are lots of way to actually implement a mocked mode. One of the coolest examples is fog. Shai is going to go into deeper details in his talk later about the different ways you can setup a mocked mode. I just want to introduce the concept.

!SLIDE[bg=pictures/wes.jpg]
### Wes

!SLIDE smaller_p
![](pictures/fog.png)

    @@@ ruby
    
    require 'fog'
    creds = {:provider => 'AWS',
      :aws_access_key_id => 'a123',
      :aws_secret_access_key => 'b456'}
    fog = Fog::Compute.new(creds)
    fog.servers
    Fog.mock!
    fog = Fog::Compute.new(creds)
    fog.servers
    fog.servers.create
    fog.servers
    fog.servers.first.destroy
    fog.servers
    fog.servers

.notes Demo. here's an example of using fog to call AWS web services in test. We ask amazon to boot an instance, and it happens immediately, and fog responds as if it worked, there's even a delay feature.

!SLIDE bullets incremental align-left
# Approach so far
* Distributed Objects
* APIs
* Isolation
* Mock-Mode

!SLIDE bullets incremental align-left
# Let's write some code

* Cloud Dashboard (awsm)
* Add-ons Service (tresfiestas)
* Fake Service (lisonja)
* Public API client (ey\_services\_api)
* Private API client (ey\_services\_api\_internal)

.notes This was a reasonable mistake to make. At Engine Yard we always like to ship things to production as early as possible. We hide them behind a feature flag and so it doesn't matter if they are live.  So reasonably I thought to immediately make all of these things.

!SLIDE
(green tests, happy face)

.notes so I thought this was great right. I just wrote all this code, but only like 1/5 of it had to be in AWSM (the slow monolith), so for the most part I was writing greenfield code, and my TDD went fast.

!SLIDE
(500 page, sad face)

.notes so I got all this into production

!SLIDE bullets incremental align-left
# What went wrong

* Working in 5 repositories at once sucks.
* No integration tests.
* No visibility of interactions in live system.

!SLIDE
# Side Note: Bundler Path

    @@@ ruby
    
    gem 'ey_services_api'
    
    gem 'ey_services_api', 
      :git => "git@github.com:engineyard/ey_services_api.git"
    
    gem 'ey_services_api', 
      :path => "../ey_services_api"

.notes and here's a little lesson for you... you can even run all this stuff locally with bundler path, and you can use bundler git so you don't have to publish your gems while you are still in development

!SLIDE[bg=pictures/tim.jpg]
### Tim

.notes the term spike generally refers to throwaway code that you write to try out an idea with the goal of figuring out all the questions you didn't know to ask. What will you stumble on. What assumptions can you discover that are wrong before you go too far down a given path.  Usually you don't write tests, or you write very minimal tests.

!SLIDE[bg=pictures/tim.jpg]
<br/>
### That's crazyness.
### Write a Spike.

!SLIDE
# Spike

* Step by step capybara interactions
* You can actually run it.

!SLIDE
# Spike Demo

.notes show the directory structure
.notes show the running "spike"

!SLIDE
### Side Note: Use artifice

!SLIDE
### Side Note: Use <s><div></div>artifice</s> rack-client

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









