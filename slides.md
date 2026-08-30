---
theme: default
title: "dial9: Seeing What Tokio Is Actually Doing"
info: |
  dial9 - fine-grained runtime tracing for Tokio applications (RCN lightning talk)
layout: cover
transition: fade
duration: 5min
---

---
layout: image-right
image: /images/trace-hero.png
---

# dial9

WTF is Tokio actually doing?

<div class="my-4">
  <span class="text-gray-400">Russell Cohen · @rcoh</span>
</div>

<!--
TODO: 30 second version of the intro. Who I am, why I ended up building this.

Lightning talk budget: ~5 minutes. That's roughly 10 slides at 30s each. Cut ruthlessly.
-->

---
class: text-center
---

# Metrics exist.

<v-click>
But they don't tell the whole story.
</v-click>

<!--
TODO: the setup. Tokio gives you metrics, but they're aggregated and they update
on a timer, so anything short-lived is invisible.

[click]
-->

---
class: text-center
---

<div class="text-3xl leading-relaxed">

Their metrics updated every second.

The problem lasted **18 milliseconds**.

</div>

<!--
TODO: the concrete story. One customer, one number, one punchline. This is the
slide that makes the audience feel the problem.
-->

---
layout: statement
---

# What if you just recorded everything?

<!--
TODO: the hypothesis. Record every poll, park, unpark, wake — cheaply enough
that you can leave it on in prod.
-->

---

# What dial9 records

<div class="grid grid-cols-2 gap-6 mt-8 text-lg">
  <div v-click class="border border-gray-700 rounded p-4">
    <div class="text-blue-400 font-bold mb-2">Poll start / stop</div>
    <div class="text-gray-400">Every task, every poll, with duration</div>
  </div>
  <div v-click class="border border-gray-700 rounded p-4">
    <div class="text-blue-400 font-bold mb-2">Park / unpark</div>
    <div class="text-gray-400">When workers went to sleep and who woke them</div>
  </div>
  <div v-click class="border border-gray-700 rounded p-4">
    <div class="text-blue-400 font-bold mb-2">Wakes</div>
    <div class="text-gray-400">Which task woke which, across runtimes</div>
  </div>
  <div v-click class="border border-gray-700 rounded p-4">
    <div class="text-blue-400 font-bold mb-2">Stacks</div>
    <div class="text-gray-400">Sampled frames, attached to the poll</div>
  </div>
</div>

<!--
TODO: keep this to one breath per box, or cut boxes. Four clicks is already a
lot for a lightning talk.
-->

---
layout: image
image: /images/trace-hero.png
---

<!--
TODO: the money shot. Walk the trace: here's a long poll, here's the scheduling
delay behind it, here's the stack that caused it.

Replace this image with the screenshot that best tells the story in 45 seconds.
-->

---
class: text-center
---

<h1 class="text-6xl">~50ns per event</h1>

<!--
TODO: the "yes, but what does it cost" beat. Answer it before they ask it.
-->

---
class: text-center
---

<div class="flex flex-col items-center justify-center">
<div class="text-xl mb-8 text-gray-400">Typical poll: 10μs. dial9 overhead: ~100ns.</div>
<div class="w-120">
  <div class="flex items-center gap-3 mb-4">
    <div class="bg-blue-500 rounded h-12 w-full"></div>
    <span class="text-sm whitespace-nowrap">Application poll: 10μs</span>
  </div>
  <div class="flex items-center gap-3">
    <div class="bg-amber-500 rounded h-12" style="width:1%"></div>
    <span class="text-sm whitespace-nowrap text-amber-500">2 dial9 events: 100ns</span>
  </div>
</div>
<div class="mt-8 text-gray-400 text-lg">~1% overhead</div>
</div>

<!--
TODO: one sentence. The bar chart does the work.
-->

---

# Adding it to your app

````md magic-move
```rust
#[tokio::main]
async fn main() {
    // your async code here
}
```
```rust
use dial9_tokio_telemetry::{main, config::{Dial9Config, Dial9ConfigBuilder}};

fn dial9_config() -> Dial9Config {
    Dial9ConfigBuilder::new(
        "/tmp/my_traces/trace.bin",
        50 * 1024 * 1024,   // 50MB max file
        1000 * 1024 * 1024, // 1GB on disk
    )
    .build()
}

#[dial9_tokio_telemetry::main(config = dial9_config)]
async fn main() {
    // your async code here
}
```
````

<!--
TODO: "it's an attribute macro" is the whole slide. Don't read the code.
-->

---
layout: statement
---

# The idea is the point.

<div class="text-xl text-gray-400 mt-6">github.com/… · come find me after</div>

<!--
TODO: closer. The tool is just software; the idea that you can record all of
this and make these problems tractable is the takeaway.

Swap in the real link before the talk.
-->
