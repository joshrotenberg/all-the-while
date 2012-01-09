# all-the-while

## Synopsis

This little library basically came out of a fairly specific need; I
have a set of tasks that a) need to run a long time (in a while()
loop), 2) need to run in parallel, and, last, are responsible for
their own per-loop runtime. The last part is where I needed something
different than ScheduledThreadPoolExecutor. The logic inside the
function will determine its run time, so, for example, iteration 1 may
sleep for 2 seconds, iteration 2 may not sleep at all, iteration 3 may
sleep for half a second, i.e.

```clojure
(while some-condition-is-true
   (let [foo (something)]
     (when (is-cool? foo)
        (Thread/sleep 2000))
    (when (is-lame? foo)
        (Thread/sleep 3000))
    ;; otherwise foo is meh, do nothing
   ))
```

This can be faked, for example, with [overtone.at-at](https://github.com/overtone/at-at)

```clojure

(every 10 (fn [] ;; everything but the while containter above 
```

but I don't think that's right. While I tried it out, and noted that
the queue never seemed to grow, it still felt wrong. I suppose you
could make that 10 a 1, but my fear is that some condition makes the
internal queue fill up and all hell breaks loose.

So I wrote this. It's probably broken in its own special ways.

## Usage

```clojure
(ns your.ns
    (:use all-the-while.core))

;; for now functions that run don't accept an argument
(defn do-some-stuff
      []
      (println "i do some stuff and then sleep")
      (Thread/sleep 2000))

(create-task "do-some-stuff" do-some-stuff)

;; if for some reason you want to emulate external scheduling, you can provide
;; a sleep time when creating the task. this will be done with Thread/sleep
;; inside the loop. if this is all you need, though, look at overtone.at-at

(create-task :do-other-stuff #(println "i always sleep") :sleep 3000)

;; you can now start either or both ...
(start-task :do-some-stuff)

;; or 

(start-all-tasks)

;; your functions are now running. when you want them to knock it off:

(stop-task "do-some-stuff")

;; or

(stop-all-tasks)

;; when you are sick of a task, you can remove it. if its still running, 
;; it will be safely stopped first.

(remove-task :do-some-stuff)

;; you can see if a task is running. returns true if it is, false otherwise.

(task-running? "do-other-stuff")

;; or you can find out the status of all of your tasks. returns a map, 
;; keys are the task names, values are either :stopped or :running

(task-status)

;; note that "string" names and :keyword names for tasks should be
;; interchangable across all functions

```

## License

Copyright (C) 2012 Josh Rotenberg

Distributed under the Eclipse Public License, the same as Clojure.
