# Career Overview


This is an attempt to capture highlights of my 16 years career as software engineer.

## Mozilla

Last 10 years I have spend at [Mozilla][] where I had an opportunity to work on some interesting projects. 

### [Libdweb][]

P2P / Decentralized / Distributed software had being suffering from high upfront costs. Creating a space to explore ideas within the open web platform seemed like a good idea, so I started a [libdweb][] where we surveyed existing implementations of Dat, IPFS, SSB to identify gaps in web platform and implemented those as experimental Firefox extensions APIs. This work has enabled and inspired community to build those systems into Firefox:



### [StralingJS][]

Concurrent JS Runtime aiming to achieve high-performance through modularity, parallelism & Rust interop. Runtime was formed by a components represented by an `async` functions communicating with other components through read/write stream pair forming a system resembling an Erlang [Actor Model][].
Components of this system could also be represented via Async Rust primitives which meant critical components could be written in Rust to achieve even higher performance. This system was designed to complement [servo][] rendering engine that was taking parallelism to the next level.

#### [Dominion][]

Building high performance UI for [Servo][] rendering engine using web technologies proved to be very challenging. Running complex application logic in JS in the UI thread meant that sooner or later frame would be dropped. [Servo][]s parallelize everything approach and raising popularity of JS libraries with Virtual DOM inspired project which moved all of the application logic into worker thread and provided Virtual DOM for view layer. To achieve high performance and avoid coping memory library encoded DOM tree and it's changes into compact [flatbuffer][] allowing transfers across threads without copying. This architecture enabled freeing UI thread of JS and also made possible to build applications that took advantage of multiple worker threads.

#### [Decoders][]

Even though [dominion][] moved most of the JS logic off the main thread event handling still had to be performed in the UI thread creating a challenging programing model. In an attempt to resolve this inspiration was taken from [parser combinator][] libraries popular in function languages. Combination of built-in basic parsers and several combinators formed a declarative interface for defining event serializers off the main thread that was encoded as byte array of instructions executed at the event site and encoded relevant information into [flatbuffer][] and transferred it back to worker thread.

### [Browser.html][]

At a time when Mozilla had bold ambitions small team was formed reimagining new generation web browser to complement a new generation web engine. It seemed obvious that new web engine had to be used to build it's own interface. This lead to the highlight of my carreer filled with challenges & rabbit-holes. Turns out innovative and performant user interfaces built with "work in progress" (and an established) web engines is hard but also very fun.


![browser.html](browserhtml.gif)

#### [Navigation Trails][]

Tabs, Browsing History, Bookmarks proved to be a poor tools with essentially a same task. User research lead us believe that we could upgrade existing metaphors and present those through a different lens that we called "navigation trails". Existing browsing sessions and browsing history got mapped onto expandable card stacks that also attempted to capture forks in navigation & blur lines between active sessions (tabs) and inactive ones (history) to better utilize visual space and computer resources.

Making snappy user interface in 3D space with infinitely large canvas at 60FPS using web platform was not the only challenge. Zooming in / out between web content and navigation trail meant web content had to remain alive for active trails, while older trails needed a way to be capturing such that they could be restored at will.

#### [Spacial User Interface][]

[Navigation trails][#Navigation_Trails] gave us an insight that (unlike traditional tabbed user interface) spacial interface enhanced our cognitive process, instead of increasing our cognitive load.

This lead us to creating a consistency in every interaction. In order for the mental model to work it needed to be applied to all views.

### Rocket Bar

![](BaGX4Sr.png)

### [Jetpack / Addon-SDK][jetpack]

Flexible work culture, appreciation for open source and personal interest also lead me to get involved in projects that are not linked to Mozilla in any way.

### [Wisp][]

- [lispyscript][]

### [Interactivate][]



#### [Interactivate-dat][]

#### [Allusion][]



### Functional Reactive Programing (FRP)

- [signalize][]
- [transducer][]
- [Elm Signals][]
- [reflex][]

## TomTom

### Webkit based UI tookit for TomTom PDAs

~I remember having [iPhone 3G][] at a time that had a far superiour hardware. Same (but unoptimized) software was not dropping even a frame, which seemed like an impossible challenge to overcome given hardware constraints.

To get there I have designed and implemented high performance JS framework with reactive data bindings and template engine with primary goal of reducing memory allocations as [GC][] was creating a significant overhead for device with a very limited RAM capacity.

~All the communication with device occurred through message passing channel. This gave me an idea to create a record/reply tool for bug reports that captured interaction session that could be later replayed for investigation and be saved as a test case to prevent regressions.

#### [TomTom Go Live 1000](https://www.engadget.com/2010/04/27/tomtom-go-1000-live-to-offer-capacitive-touchscreen-webkit-brow/)

![](./MXYbKlE.png)



### On PDA server as app replacement

In those days most devices were not internet connected. That meant users had to download software onto their computer and use it to manage content on the device, maps in case of PDAs. This was not a great user experience and required development and maintenance of cross-platform software.

It seemed like web platform could have replaced a need for extra software. Me and my collogue took up that challange to create a proof of this concept. We put an [embedded HTTP server](https://en.wikipedia.org/wiki/Mongoose_(web_server)?wprov=sfti1) on device exposing REST API for managing content on device. Then we used this REST API from a web application to recreate functionality of existing software.

Result was that users no longer needed to install anything to manage maps on the device instead they had to connect device and purchase / install maps from TomTom website.

### TomTom Home

TomTom home was content management software for TomTom devices written on top of XULRunner cross platform application toolkit used in Firefox. 

### [NarvalJS][]

NarvalJS was a predecessor to nodejs that used [Rhino JS engine][]. Idea of writing server software in JS was both inspiring and intriguing. Given experience writing server software in Java I was able to contribute to it's development.

### [Narval XULRunner][]

Developing server software in JS was becoming a thing & I was part of this shift. Given that I was writing desktop application in JS for TomTom in XULRunner it was obvious to me that we would want to use the same code to write cross platform desktop application as well. So I have worked on a predecessor of [ElectorJS](https://www.electronjs.org) it was both fun & challenging to implement and expose API compatible with a rest of the ecosystem through [XPCOM](https://en.wikipedia.org/wiki/XPCOM) bindings.

This work was eventually got recognized by some at Mozilla where I then was hired to work on Firefox Extensions which was also was interested in interop with growing server JS ecosystem.

### [Actors in JS][Actor]

In the days before JS had promises or `async` / `await` managing code complexity in large applications was a nightmare a.k.a callback hell. TomTom Home and Firefox extensions later that I was employed to work during those days suffered with that problem as well. However Spidermonkey JS engine even back then had generators and my exposure to Erlang inspired me to utilize thouse to design a better async programming model.

At a time I have successfully deployed this solution across organizations and teams to overcome challenges of async programing.

### [Ubiquity][]

### [Skywritter][] (formally Bespin)

## UFC

### Oracle

### Java Services

### Visa / Mastercard

[Mozilla]:https://www.mozilla.org/
[Browser.html]:https://github.com/browserhtml/browserhtml
[StralingJS]:https://github.com/starlingjs
[Dominion]:https://github.com/gozala/dominion
[Decoders]:https://github.com/gozala/decoder.flow
[Libdweb]:https://github.com/mozilla/libdweb
[Wisp]:http://github.com/gozala/wisp
[Interactivate]:https://gozala.io/interactivate-wisp/
[Interactivate-dat]:https://github.com/gozala/interactivate-dat
[Allusion]:https://github.com/gozala/allusion
[Lunet]:https://lunet.link
[IPDF]:https://github.com/gozala/ipdf/
[IPDF Talk]:https://www.youtube.com/watch?v=KBwR0I7i4Wg&feature=youtu.be
[Textile Threads V2]:https://blog.textile.io/introducing-textiles-threads-protocol/
[signalize]:https://github.com/Gozala/signalize
[transducer]:https://github.com/Gozala/transducer
[Elm Signals]:https://package.elm-lang.org/packages/elm-lang/core/3.0.0/Signal
[reflex]:https://github.com/gozala/reflex/
[NarvalJS]:https://github.com/280north/narwhal
[iPhone 3G]:https://en.wikipedia.org/wiki/IPhone_3G

[Jetpack]:https://wiki.mozilla.org/Jetpack
[Narval XULRunner]:https://github.com/Gozala/narwhal-xulrunner
[lispyscript]:https://github.com/Gozala/lispyscript
[streamer]:https://github.com/Gozala/streamer
[Ubiquity]:https://en.wikipedia.org/wiki/Ubiquity_(Firefox)
[Skywritter]:https://en.wikipedia.org/wiki/Mozilla_Skywriter
[Actor]:https://github.com/Gozala/actor
[reducible-dom]:https://github.com/Gozala/dom-reduce
[servo]:http://servo.org
[Actor Model]:https://en.wikipedia.org/wiki/Actor_model?wprov=sfti1
[flatbuffer]:https://google.github.io/flatbuffers/
[parser combinator]: https://en.wikipedia.org/wiki/Parser_combinator?wprov=sfti1
[Navigation Trails]:https://www.freecodecamp.org/news/lossless-web-navigation-with-trails-9cd48c0abb56/
[Spacial User Interface]:https://www.freecodecamp.org/news/lossless-web-navigation-spatial-model-37f83438201d/
[Rethinking browser]:https://patrykadas.com/browser.html
[GC]:https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)?wprov=sfti1
[RAM]:https://en.wikipedia.org/wiki/Random-access_memory?wprov=sfti1
[Rhino JS Engine]: https://en.wikipedia.org/wiki/Rhino_(JavaScript_engine)?wprov=sfti1

[#Navigation Trails]:

[#Navigation_Trails]: