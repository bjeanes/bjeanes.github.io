+++
aliases = ["2012/call-clojure-function-on-a-timer", "2012/09/call-clojure-function-on-a-timer"]
date = 2012-09-20T16:07:00Z
slug = "call-clojure-function-on-a-timer"
title = "Call Clojure function on a timer"
updated = 2012-09-20T16:28:53Z
+++

In Clojure, I didn't see a nice way to simply call a function on a timer (e.g. to poll for changes in another service). 

I didn't find something in `clojure.core` to achieve this readily (but `clojure.core` is quite big, so I may have missed something obvious — let me know), so I whipped up the following to put in my project's `util.clj` file:

``` clojure
(defn tick
  "Call f with args every ms. First call will be after ms"
  [ms f & args]

  (future
    (doseq [f (repeatedly #(apply f args))]
      (Thread/sleep ms)
      (f))))

(defn tick-now
  "Call f with args every ms. First call will be immediately (and blocking)"
  [ms f & args]

  (apply f args)
  (apply tick ms f args)
```

There are two variants. `tick` waits `ms` milliseconds and then calls `f` with `args` and repeats indefinitely. `tick-now` does the same thing except it calls `f` with `args` *before* starting the timer.

They are simple to use:

``` clojure
user=> (tick 500 #(println "hi"))
; 500ms delay
hi
; 500ms delay
hi
; 500ms delay
hi
...
```

In my project, I'm using them like so:

``` clojure
(defn start-fetchers
  [api-token]

  (future
    (let [minutes (partial * 60 1000)]
      (tick-now (minutes 60)
                update-project-list!
                api-token)

      (tick-now (minutes 5)
                fetch-milestones!
                api-token
                projects-to-fetch
                milestones-by-project))))
```