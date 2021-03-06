# Killer

[![Continuous Integration status](https://secure.travis-ci.org/JulesGosnell/killer.png)](http://travis-ci.org/JulesGosnell/killer)

Killer is [Functional](https://en.wikipedia.org/wiki/Functional_programming) [Event Stream Processing](https://en.wikipedia.org/wiki/Event_stream_processing) in [Clojure](https://clojure.org/) with [Transducers](http://clojure.org/reference/transducers).

Killer is a reiteration on
[DADA](https://github.com/JulesGosnell/dada "Event Stream Processing in Clojure").

Killer aims to build on everything that I learnt writing DADA whilst adding :

- ESP operations realised as Transducers
- Real-time charts and tables via a web UI
- Ability to drill into any values to see where they came from
- lots of other stuff...

I've only just started on Killer, but I'm already really enjoying
using transducers for ESP.

Let's look at a really simple example.

You have a sequence of symbols. You want to plot a pie-chart of the
frequency of each symbol.

In standard Clojure, you might try something like this:

```clojure
killer.transducers> (def s [:a :b :a :c :b :a])
#'killer.transducers/s
killer.transducers> s
[:a :b :a :c :b :a]
killer.transducers> (def f (frequencies s))
#'killer.transducers/f
killer.transducers> f
{:a 3, :b 2, :c 1}
killer.transducers> (pie-chart f 100)
{:a 50N, :b 100/3, :c 50/3}
killer.transducers> 
```

where pie-chart is a function that just happens to already be defined in Killer as:

```clojure
(defn- pie-chart
  "transform a hash-map of value:frequency to value:proportion of a given whole"
  [frequencies whole]
  (let [n (apply + (vals frequencies))]
    (into {} (map (fn [[k v]] [k (* (/ v n) whole)]) frequencies))))
```

Note that the final output of the pie-chart function is a data-model
that would bind nicely to a pie-chart rendering done by
percentage. (If you wanted it expressed in degrees you could use a
pie-chart 'whole' of 360 instead of 100.)

"Great" - I hear you say - "but what about this ESP thing that you
keep harping on about ? I'm not interested in static visualisation
of data, I want to see things moving around."

OK.

So now imagine that each of the functions that you used above could be
realised as a transducer which could be applied to a sequence and (in
this simple case) would generate a new output each time it consumed an
input value.

Then you might see something more like:

```clojure
killer.transducers> (def s [:a :b :a :c :b :a])
#'killer.transducers/s
killer.transducers> s
[:a :b :a :c :b :a]
killer.transducers> (def f (sequence (esp-frequencies) s))
#'killer.transducers/f
killer.transducers> f
({:a 1}
 {:a 1, :b 1}
 {:a 2, :b 1}
 {:a 2, :b 1, :c 1}
 {:a 2, :b 2, :c 1}
 {:a 3, :b 2, :c 1})
killer.transducers> (sequence (esp-pie-chart 100) f)
({:a 100}
 {:a 50N, :b 50N}
 {:a 200/3, :b 100/3}
 {:a 50N, :b 25N, :c 25N}
 {:a 40N, :b 40N, :c 20N}
 {:a 50N, :b 100/3, :c 50/3})
killer.transducers>
```
Here we imagine the same sequence of data arriving.

This time however, esp-frequencies will consume its input
element-by-element and output a complete new aggregate result as each
input element is consumed.

If we then imagine feeding the output of esp-frequencies into
esp-pie-chart we can then see that this does something similar - for
each frequencies value input it outputs a new pie-chart model.

Of course, as esp-frequencies and esp-pie-chart are transducers, they
can be composed, meaning that no intermediate sequence is created:

```clojure
killer.transducers> (def s [:a :b :a :c :b :a])
#'killer.transducers/s
killer.transducers> s
[:a :b :a :c :b :a]
killer.transducers> (sequence (comp (esp-frequencies)(esp-pie-chart 100)) s)
({:a 100}
 {:a 50N, :b 50N}
 {:a 200/3, :b 100/3}
 {:a 50N, :b 25N, :c 25N}
 {:a 40N, :b 40N, :c 20N}
 {:a 50N, :b 100/3, :c 50/3})
killer.transducers>
```

Now, if we were to replace the sequences above with core.async/channels
(a standard transducer capability) and had a nice dynamic HTML5 UI to
connect up, we would have real-time pie-chart-ing of our incoming data
in a single line of Clojure - each input value culminating in a frame
in a pie-chart movie showing the evolution of the data-set over
time.

I have now done this. If you clone this repo and type `lein start` at
the top-level, you should soon see a couple of live donut-charts in
your browser.

![Alt text](https://raw.github.com/JulesGosnell/killer/master/doc/images/donuts.gif)

Server side source code can be seen in src/killer/whales.clj and
trades.clj.

Client side src lives mainly in src/killer/donut.cljs which is a
translation of some JavaScript that I found online.

The Server Push infrastructure was borrowed from Sente.

Since our ESP domain is expressed very similarly to the familiar
domain of Clojure sequences, the learning curve to fully featured
Event Stream Processing should be very flat for anyone familiar with
Clojure or Functional Programming. Furthermore, since Killer is just a
Clojure DSL it can be extended in any direction by just plugging in
more Clojure functions (example to follow).

I hope this gives a flavour of my hopes for Killer.

Please get in touch, if you find this interesting.


Jules
