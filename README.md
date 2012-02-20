Manipulate the DOM and handle events of the browser from Haskell.

## Examples

* [Some simple buttons](http://chrisdone.com/ji/buttons/)
* [Missing dollars question](http://chrisdone.com/ji/missing-dollars/)
* [A chat app](http://chrisdone.com/ji/chat/)

## Difference to HJScript

HJScript is a library that generates JavaScript, which runs on the
client, with JavaScript semantics.

So, e.g. if you want to write expressions you write:

    x <- varWith (int 10)
    alert (mathMax (val 10 * x) 45)
    ajax (string "get-user") (10,"admin") $ \user -> do
       udiv <- j "<div></div>"
       setText (toString user)
       append body udiv

Ji is a set of Haskell functions that manipulate a remote DOM, which
runs on the server and sends code to the client, with Haskell
semantics.

So for the above in Ji you might write:

    let x = 10
    alert (max (10 * x) 45)
    user <- getUser 10 Admin
    udiv <- new
    setText (show user) udiv
    append body udiv

Although the HJScript can be abstracted somewhat to look more like the
below, by using Haskell like a macro language for JavaScript, it's
still JavaScript.

## More detail

Ji transparently runs on the server or the client depending on what
needs to be done, but most time is spent on the server. In this sense
one could write code like this:

     body <- getBody
     ul <- new ## addTo body
     forM_ [1..10] $ \i -> do
       new ## addTo ul ## setText (show i)
       threadDelay (1000*1000)

Which would add a new line with each number to the browser's body at
least every second, network latency allowing.

## Challenges

### Latency

The above example has a latency. If this is a problem, e.g. then using
Ji for this task is wrong. The solution is to produce some JavaScript
that will run on the client. Consider this approach similar to [a
shading language.](http://en.wikipedia.org/wiki/Shading_language)

Some means of producing JavaScript from Haskell might be:

* HJScript
* GHCJS
* UHC

It might be desirable in the future to write something like this:

    module Main where
    import Client.RenderLis
    main = runJiServer $ do
      body <- getBody
      ul <- new ## addTo body
      runJS renderLis

    module Client.RenderLis where
    renderLis = JS "render-lis" $ do -- This name could be generated by a TH macro.
      forM_ [1..10] $ \i -> do
        new ## addTo ul ## setText (show i)
        threadDelay (1000*1000)

where

    runJS :: (MonadJi,MonadJS m) => js () -> m ()
    runJS js = do
      id <- getId (toJiRepresentation js)
      callRemoteJS id

and then

    ghc --make Main.hs
    uhc -tjs Client.RenderLis.hs

Or you could just use HJScript with JavaScript semantics. It depends
how much you don't like JavaScript.

Either way, I think it strikes a good balance. In either case the
server remains in control. If you need user interaction with the model
and the business logic, use Ji. If you need some special rendering or
latency hotspots, optimize it with some custom JS. But use it
conservatively. Some examples where you might want to compile to JS
are:

* canvas rendering
* switching tabs/hover information
* timers that should be accurate

### Garbage collection

Every DOM element referenced from the server needs a number generated
for it per session in a map, and once the session timesout it's
deleted. It would be good to optimize this to an IntMap, for space
reasons. It we can attach finalizers to Element values then we can
automatically free up "reference" numbers to be used again.

## Approach

This is all assuming that this approach is a good approach, which has
yet to be confirmed, although the outlook is positive. Some public
applications which could seemingly be written in Ji:

* How much of GMail requires client-side code, not
  much, huh?
* How much of Google+?
* How much of YouTube?
* Hoogle?
* Hackage?

## Other ideas

Switch to HTML rendering mode — it might be nice in the case of search
engines to merely generate a DOM and render it, so that search engines
can read the pages.

## Expanding with a decent API

Inspiration can be taken from the following Obviously Decent UI
paradigms:

* [The Common Lisp Interface Manager](http://en.wikipedia.org/wiki/Common_Lisp_Interface_Manager)
* [The Windows Presentation Foundation](http://en.wikipedia.org/wiki/Windows_Presentation_Foundation)
* [Functional Reactive Programming](http://en.wikipedia.org/wiki/Functional_reactive_programming)

I will write more about why these are Obviously Decent and not crap
like typical UI application approaches.

## Other helpful libraries of interest

* [qooxdoo](http://qooxdoo.org/demo) — provides a feature-complete
  widget set. One could wrap this in a type-safe API from Ji and get a
  complete, stable UI framework for free. Most of the "immediate
  feedback" like dragging things here, switching tabs there, are taken
  care of by the framework. All that would be left would be to provide
  the domain configuration and business/presentation logic.

There are plenty more like this, but this is the first that springs to
mind that is good.
