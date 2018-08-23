---
layout: post
title: Chemotaxis
---

Here's a little bacteria simulation I've built in ClojureScript.
I took a lazy approach and instead of starting from scratch I modified
[Julian Gamble's ClojureScript port](https://github.com/juliangamble/clojure-conj-2014-paradigms-of-core-async/tree/950964320bbff17cdd3da7bbcae00ac85dbcd388/9.Ants-CLJS-Array-Optimised/ants-cljs){:target="_blank"} of Rich Hickey's ants colony demo.

One common behaviour of bacteria is movement in response to chemical stimuli
([chemotaxis](https://en.wikipedia.org/wiki/Chemotaxis){:target="_blank"}).
Wikipedia lists [many types of taxis](https://en.wikipedia.org/wiki/Taxis){:target="_blank"}
or movement towards and away from stimuli such as phototaxis (stimulation by
light) or thermotaxis (stimulation by heat), but I decided to simulate
chemotaxis only.

The organisms in question obviously cannot acquire and process a lot
of information about the environment. Some bacteria (e.g. E. coli)
observe the concentration of chemicals as they move and continue moving
forward if the concentration of attracting chemicals increases (or repelling
ones decreases). Otherwise they change direction randomly by flipping over.

The simulation is probably not very realistic because of scale,
speed etc. and it lacks negative stimuli (poison), but it nicely
demonstrates the effectiveness of the very simple movement strategy.

Click the image below to see a working demo:

[![chemotaxis](/images/posts/chemotaxis.png)](https://cdn.rawgit.com/replomancer/chemotaxis-cljs/master/resources/public/index.html){:target="_blank"}


## Starting point: ants colony in ClojureScript

Rich Hickey's ants colony simulation is a demo of Clojure's concurrency
features from more than 10 years ago. I decided to use a ClojureScript port
to have something runnable in a browser. The ported version I found on github
differs a lot from the original. In particular:

1. CLJS has no agents or refs. Concurrency in the CLJS version is introduced
   with Communicating Sequential Processes (CSP).

2. For performance reasons the CLJS version makes heavy use of JavaScript
   mutable arrays in place of Clojure persistent data structures. This makes
   the code far from idiomatic. I believe it was [Lau Jensen's blog post](https://www.bestinclass.dk/da/posts/brians-brain-optimized-clojurescript-html5){:target="_blank"}
   that inspired the array-based solution.

CSP on the front end is a nice change from typical callbacks-heavy
JavaScript code. E.g. instead of calling setTimeout you can read from timeout
channels. Fun fact: ClojureScript's core.async library has been [ported to JavaScript](https://github.com/ubolonton/js-csp){:target="_blank"} so you can use CSP channels with ES6 generators.


## Chemotaxis-cljs:

I've cleaned the code, added figwheel and replaced all ant-specific
things with bacteria and chemotaxis. Changes include closing the world
borders for more realism (the world is no longer a torus).

The main part is in this function:

{% highlight clojure %}

(defn behave [bacterium]
  (let [coord (get-bacterium-coord bacterium)
        occupied-cell (place coord)
        curr-food-amount (get-cell-food occupied-cell)
        prev-food-amount (get-bacterium-food bacterium)
        ahead-coord (delta-loc coord (get-bacterium-dir bacterium))]
    ;; When food is less abundant, the bacterium flips and chooses
    ;; a new dir randomly (sometimes it's the same as the old dir).
    ;; It also flips when the way is blocked (by world borders or another
    ;; bacteria). Otherwise it continues moving forward.
    (when (or (< curr-food-amount prev-food-amount)
              (outside-world? ahead-coord)
              (get-cell-bacterium (place ahead-coord)))
      (set-bacterium-dir bacterium (rand-int 8)))
    ;; remember how much food there was
    (set-bacterium-food bacterium curr-food-amount)
    ;; eat some of the available food
    (set-cell-food occupied-cell (int (Math.floor (* 0.9 curr-food-amount))))
    ;; move
    (move bacterium)))
{% endhighlight %}

It's amazing how such a simple strategy can work so well (see the demo above).


Source: <https://github.com/replomancer/chemotaxis-cljs>{:target="_blank"}
