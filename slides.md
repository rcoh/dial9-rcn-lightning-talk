---
theme: default
title: "dial9: a flight recorder for Rust"
info: |
  dial9 - a flight recorder for Rust (RCN lightning talk)
layout: cover
colorSchema: light
transition: fade
duration: 10min
class: text-center
---

<div class="flex flex-col items-center justify-center h-full">
  <img src="/images/dial9-logo.svg" class="h-40 mb-8" />
  <h1 class="text-5xl">dial9</h1>
  <div class="text-2xl text-gray-500 mt-2">a flight recorder for Rust</div>
</div>

<!--
Hello! I want to show folks dial9; dial9 is a flight recorder for Tokio (and Rust applications) in general. I get into what that means in a minute.
-->

---

# Me

<div class="mt-8 text-2xl">Russell Cohen</div>
<div class="text-xl text-gray-500 mt-1">AWS</div>

<div class="mt-10 text-xl leading-relaxed">

- AWS SDK for Rust
- `smithy-rs`
- Occasional Tokio contributor

</div>

<!--
First an extremely quick intro about myself for anyone who I don't know. I'm Russell, I've been working on Rust and Rust related things at AWS for the last 6 years, for the first 4 the AWS SDK for Rust and now I work on a team focused on making Rust in general successful at Amazon. It was on this team, getting pulled into pretty frequent escalations to help people with the performance of Rust code, that the need for a flight recorder really became apparent. So at the beginning of this year, I started working on dial9.
-->

---
layout: statement
---

# dial9: a flight recorder for Rust

<!--
dial9 is a flight recorder for Rust
-->

---

# Why do we need a flight recorder?

<div class="mt-10 text-2xl leading-loose">

- record a large number of <span class="text-blue-500">events</span>, compactly and efficiently
- easily get them off the host for analysis

</div>

<!--
What is a flight recorder and why do we need one? It's a way to record a large number of events, teams at Amazon are hitting 1 million events per second, on a production system and get them _off_ the host without degrading application performance too much. But the word "event" is doing a lot of heavy lifting here.
-->

---

<div class="text-2xl">
record a large number of <span class="bg-[#fec200]/70 px-1 -mx-1">events</span>, compactly and efficiently
</div>

<h1 class="mt-8">What's an "event"?</h1>

<div class="mt-8 space-y-6">
  <div class="flex items-center gap-8">
    <img src="/images/flamegraph.png" class="h-16 w-44 object-contain" />
    <span class="text-2xl">CPU stack samples</span>
  </div>
  <div v-click="1" class="flex items-center gap-8">
    <img src="/images/tracing-logo.png" class="h-16 w-44 object-contain" />
    <span class="text-2xl">application events / <code>tracing</code></span>
  </div>
  <div v-click="2" class="flex items-center gap-8">
    <img src="/images/tokio-logo.svg" class="h-12 w-44 object-contain invert" />
    <span class="text-2xl">Tokio events</span>
  </div>
  <div v-click="3" class="border border-dashed border-gray-400 rounded-lg py-3 -ml-4 pl-4 pr-6 inline-flex items-center gap-8">
    <img src="/images/nvidia.svg" class="h-12 w-40 object-contain" />
    <div>
      <div class="text-2xl">GPU telemetry</div>
      <div class="text-sm text-gray-500 mt-1">sources don't need to be part of dial9</div>
    </div>
  </div>
</div>

<!--
An event can be any useful information you can get out of your application;

One of the most useful is cpu samples like you might get from perf; the dial9 trace format unsurprisingly has specific support for stack frames, addresses, and symbols so that can be handled efficiently

[click]

then once you line those up with events from your application you can start to answer some really interesting questions; for example: show me a flamegraph of my requests but only requests that were slower than p99. Diff that with p50 requests. You can do that because you know the exact threads and moments in time these requests were running on.

[click]

And if you are working on an async application, you can pull in worker park/unpark and poll start/stop from Tokio. One interesting tidbit here: with Tokio events specifically, I've heard dial9 solved teams' problems not because it showed that there was some issue with how they were using tokio, but because it showed that there _was not_ a problem which gave them enough of an insight to go find the problem elsewhere.

[click]

And this is open ended, last week someone sent a PR to pull telemetry from NVIDIA GPUs. Sources don't need to be part of dial9, they can be local to your own application.
-->

---
clicks: 1
---

# Fast and compact event serialization

<div class="text-lg text-gray-500 mt-1">{{ $clicks >= 1 ? 'Bytes per event, after gzip level 1' : 'Wall-clock time to turn one event into bytes' }}</div>

<div class="mt-10 space-y-8">
  <div class="grid grid-cols-[230px_1fr_130px] gap-5 items-center">
    <div class="text-right text-lg">dial9 trace format</div>
    <div class="bg-gray-200 rounded h-8"><div class="bg-blue-500 rounded h-8 transition-all duration-700 ease-out" :style="{ width: ($clicks >= 1 ? '16.2%' : '1.9%') }"></div></div>
    <div class="text-xl font-bold text-blue-500">{{ $clicks >= 1 ? '4.7 B' : '22 ns' }}</div>
  </div>
  <div class="grid grid-cols-[230px_1fr_130px] gap-5 items-center">
    <div class="text-right text-lg">OTLP protobuf<br><span class="text-sm text-gray-500">(LogRecord)</span></div>
    <div class="bg-gray-200 rounded h-8"><div class="bg-gray-400 rounded h-8 transition-all duration-700 ease-out" :style="{ width: ($clicks >= 1 ? '50.7%' : '30.8%') }"></div></div>
    <div class="text-xl font-bold text-gray-500">{{ $clicks >= 1 ? '14.7 B' : '345 ns' }}</div>
  </div>
  <div class="grid grid-cols-[230px_1fr_130px] gap-5 items-center">
    <div class="text-right text-lg">JSON<br><span class="text-sm text-gray-500">(tracing + JSON subscriber)</span></div>
    <div class="bg-gray-200 rounded h-8"><div class="bg-amber-500 rounded h-8 transition-all duration-700 ease-out" :style="{ width: ($clicks >= 1 ? '92.4%' : '93.8%') }"></div></div>
    <div class="text-xl font-bold text-amber-500">{{ $clicks >= 1 ? '26.8 B' : '1,050 ns' }}</div>
  </div>
</div>

<div class="mt-12 text-xs text-gray-400">1M-event scheduler-telemetry mix (poll start/end, park, wake, CPU samples w/ stacks) · Apple M2 · medians of 8 runs</div>

<!--
under the hood, it has two interesting ideas: the first is that by amortizing the schema in a datastream, you can have a datastream that is as flexible as JSON while being extremely compact and fast to serialize. This means that at runtime you can define a totally new type of event and it is still very efficient to serialize (you don't have to codegen a serializer etc. etc.)

dial9 events take about 20-50ns to serialize

[click]

and they compress really well, you can end up with 10s of bytes per event uncompressed and <10B per event compressed. Each event also carries a nanosecond precision timestamp.
-->

---

# Thread-local buffers

<div class="text-lg text-gray-500 mt-1">avoid contention on the hot path</div>

<div class="text-xs text-gray-400 mt-5 text-center">1 MB each · events encoded in place · every push checks the global epoch counter</div>

<div class="mt-2 grid grid-cols-3 gap-5">
  <div class="border border-gray-300 rounded p-3">
    <div class="text-sm mb-2">thread 0</div>
    <div class="bg-gray-200 rounded h-3"><div class="bg-blue-500 rounded h-3" style="width:72%"></div></div>
  </div>
  <div class="border border-gray-300 rounded p-3">
    <div class="text-sm mb-2">thread 1</div>
    <div class="bg-gray-200 rounded h-3"><div class="bg-blue-500 rounded h-3" style="width:45%"></div></div>
  </div>
  <div class="border border-dashed border-gray-400 rounded p-3">
    <div class="text-sm mb-2">thread 2 <span class="text-gray-400">· idle</span></div>
    <div class="bg-gray-200 rounded h-3"><div class="bg-amber-500 rounded h-3" style="width:8%"></div></div>
  </div>
</div>

<svg viewBox="0 0 100 10" preserveAspectRatio="none" class="w-full h-10 text-gray-400 mt-3">
  <line x1="16.6" y1="0" x2="50" y2="9" stroke="currentColor" stroke-width="1.5" vector-effect="non-scaling-stroke"/>
  <line x1="50" y1="0" x2="50" y2="9" stroke="currentColor" stroke-width="1.5" vector-effect="non-scaling-stroke"/>
  <line x1="83.3" y1="0" x2="50" y2="9" stroke="#f59e0b" stroke-width="1.5" stroke-dasharray="4,3" vector-effect="non-scaling-stroke"/>
</svg>

<div class="flex justify-center">
  <div class="border border-blue-500 bg-blue-500/10 rounded px-8 py-2 text-center">
    <div class="text-lg font-bold text-blue-500">central buffer</div>
    <div class="text-sm text-gray-500">lock-free <code>ArrayQueue</code> · single reader</div>
  </div>
</div>

<div class="flex justify-center my-2 text-gray-400">
  <svg width="14" height="24" viewBox="0 0 14 24"><line x1="7" y1="0" x2="7" y2="16" stroke="currentColor" stroke-width="1.5"/><polygon points="2.5,15 11.5,15 7,24" fill="currentColor"/></svg>
</div>

<div class="flex h-8 border border-gray-300 rounded overflow-hidden text-xs">
  <div class="bg-gray-600 text-white flex items-center justify-center" style="width:8%">header</div>
  <div class="bg-gray-200 flex items-center justify-center border-r border-gray-300" style="width:38%">batch · thread 0</div>
  <div class="bg-gray-200 flex items-center justify-center border-r border-gray-300" style="width:24%">batch · thread 1</div>
  <div class="bg-amber-200 border-r border-gray-300" style="width:5%"></div>
  <div class="flex-1"></div>
</div>

<!--
The second idea is that by again, amortizing the cost of coordination, you can record events into large thread local buffers, which avoids lock contention when recording events. When the thread local buffer fills up, it flushes itself to the central buffer, but this is happening on the order of seconds so there effectively no contention. The central buffer writes the block into the main file.
-->

---
class: text-center
---

# Getting them off the host

<div class="flex items-center justify-center gap-10 mt-16">
  <img src="/images/dial9-logo.svg" class="h-32" />
  <svg width="90" height="120" viewBox="0 0 90 120" class="text-gray-400">
    <path d="M0 60h40M40 60L72 24m0 0h-10m10 0v10" stroke="currentColor" stroke-width="2" fill="none"/>
    <path d="M40 60h32m0 0l-8-6m8 6l-8 6" stroke="currentColor" stroke-width="2" fill="none"/>
    <path d="M40 60l32 36m0 0h-10m10 0v-10" stroke="currentColor" stroke-width="2" fill="none"/>
  </svg>
  <div class="flex flex-col gap-4 text-left">
    <div class="border border-gray-300 rounded px-6 py-2 text-xl w-52">S3</div>
    <div class="border border-gray-300 rounded px-6 py-2 text-xl w-52">Disk</div>
    <div class="border border-dashed border-gray-400 rounded px-6 py-2 text-xl w-52 text-gray-400">???</div>
  </div>
</div>

<!--
then once every 30 seconds or so, we symbolize all the addresses present in the file, compress it, and you can get it off the host.

this whole pipeline is configurable; you could upload to GCS, or save it to disk. etc.
-->

---
---

# Where are we now?

```bash
cargo add dial9
```

<div class="text-sm text-gray-500 mt-8 mb-1">Cargo.toml</div>

```toml
[dependencies]
dial9 = { version = "0.5", features = ["tokio", "cpu-profiling"] }
```

<div class="text-sm text-gray-500 mt-6 mb-1">main.rs</div>

```rust
#[dial9::main(config = dial9::recorder_from_env)]
async fn main() {
    // your async code here
}
```

<!--
Since then, dial9 has been used in production across AWS and beyond including in the rama proxy. Judging by who is sending PRs and bug reports, it's also being used by a bunch of other big Rust companies.
-->

---
layout: statement
---

# Demo

<!--
I want to give a very quick demo of what you can actually do once you have this data.
-->

---

# What's hard?

<div class="mt-10 text-xl leading-relaxed">

- Full Tokio instrumentation requires instrumenting every spawned task (and there is no good way to do this)
- <span class="text-blue-500">dial9</span> aims to be a "cafeteria profiler"; take what you want (more data, <span class="text-amber-500">more runtime overhead</span>). But this comes with complexity.

</div>

---

# What's next?

- Better analysis especially collaborative agent analysis
- Improved platform support (Android, iOS etc.)
- Squeezing out more performance
- File an issue!

---

# Some links

<div class="mt-8 text-xl leading-relaxed">

the dial9 blog:

- [dial9: a flight recorder for Rust](https://dial9-rs.github.io/blog/dial9-a-flight-recorder-for-rust/) (more details about internals)
- [what's new in dial9 0.5](https://dial9-rs.github.io/blog/whats-new-in-dial9-0-5/)

</div>

<div class="mt-6 text-xl leading-relaxed">

- [docs.rs](https://docs.rs/dial9/latest/dial9/)

</div>
