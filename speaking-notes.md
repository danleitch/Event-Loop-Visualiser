# Speaking notes: what happens to your code in the browser (30 min)

## Intro (opening)
Good day everyone, good to have you here. Today I want to talk mainly about the event loop, which is how your code gets run. I'm hoping to make this as practical as possible, with real-world examples.

We'll take a small journey from your IDE to the finished web page.

Whatever JavaScript framework you use, the code is compiled and bundled at build time into plain JavaScript that the browser can run. A bundler packages it up. (List of bundlers: https://dev.to/shadowtime2000/list-of-js-bundlers-14ee. Check this link is still current, it's an older article.)

Browsers support JavaScript through the ECMAScript standard.

Let's stop here for a second on ECMAScript (https://en.wikipedia.org/wiki/ECMAScript_version_history#ES.Next).

Ecma International is the organisation that standardises JavaScript, through a committee called TC39.

Usually we don't worry about versions of JavaScript, because browser teams keep up and projects tend to wait for wide support. It only matters when a newer feature hasn't reached the browser yet.

Once the browser has the JavaScript, its engine (V8 in Chrome) parses it, runs it, and compiles the busy parts to faster code while the page runs. Everything then runs on one call stack, and that's where the event loop comes in.

## Engine basics (keep it short)
- Parse: the text becomes a structure the engine understands.
- Run: the code is turned into bytecode, and an interpreter runs it one step at a time as the program reaches it. Functions are often only turned into bytecode the first time they're called.
- Contrast with a C compiler: that turns the whole program into machine code before it runs. V8 does the minimum up front and the rest as the program goes.
- JIT: the engine notices which functions run often and compiles those to faster machine code while the page is running. "Just in time" means it happens during the run, not before.
- Syntax vs methods: new syntax (like `?.` in older Chrome) is a parse error, so build tools rewrite it. A new method (like `Array.fromAsync`) only fails at runtime with "is not a function", so you can polyfill it.

## Core idea
JavaScript has one call stack. The browser around it has many threads. Many tasks wait in the task queue to take turns on that one stack.

## Why it's worth learning
Understanding the event loop is vital for writing responsive, non-blocking JavaScript code.

Let's look at this visualised (open the event loop visualiser).

## The five pieces (overview slide)
1. **Call stack:** tracks the functions running right now. A function is added when called and removed when it finishes. Last in, first out (LIFO).
2. **Web APIs:** provided by the runtime (the browser), not the language. Examples: setTimeout, the Fetch API, DOM events. Calling them starts the work and returns immediately, so the main thread carries on.
3. **Task queue:** when an async operation finishes or an event happens, its callback is placed here to wait.
4. **Microtask queue:** a separate, higher-priority queue for small jobs, mostly promises and a few JavaScript APIs.
5. **Event loop:** the process that keeps checking all of the above and decides what runs next.

Names you'll see online: task queue = callback queue = macrotask queue. Same thing, different labels.

**One turn of the event loop:**
1. Is the call stack empty? If not, wait.
2. Run all microtasks until that queue is empty (including any added along the way).
3. Render if a frame is due.
4. Take ONE task from the task queue and run it. Then back to step 2.

## Language vs host environment
setTimeout isn't part of the language. You call it from JavaScript and it feels like part of the language, but it isn't defined by it. The browser provides it.
- **The language (ECMAScript)** defines the core: variables, functions, Array, Math, JSON, Promise and so on. It has no timers, no console, no DOM and no fetch.
- **The host environment** is whatever runs the language, such as a browser or Node. It adds its own extras on top. A browser adds document, fetch and setTimeout. Node adds fs, process and its own setTimeout.

## setTimeout and the task queue
- The delay is a minimum, not a schedule.
- When the timer expires, the callback goes to the back of the task queue (not onto the call stack), behind anything already waiting.
- It runs only when: the timer has expired, it has reached the front of the queue, and the call stack is empty. Microtasks (promises) go first.
- Busy code makes it late. One long task delays every timer and click behind it.
- A 5 minute `setTimeout` doesn't freeze anything. 5 minutes of synchronous work does.
- One-liner: setTimeout doesn't say when your code runs, it says when your code becomes eligible to run.
- If asked: browsers keep several task queues (timers, input, network) and may prioritise between them. The single queue is a teaching model. Timers also get clamped when nested and throttled in background tabs.

## Microtask queue
The rule: when a task finishes and the call stack is empty, the browser runs every microtask until the queue is empty. That includes any microtasks added while it's draining. Only then does it render or take the next task from the task queue.

- Goes in the microtask queue: promise callbacks (`.then`, `.catch`, `.finally`), the code after an `await`, and `queueMicrotask(fn)`.
- Why it exists: a promise callback should run as soon as the current code finishes, but not in the middle of it.
- Starvation: a microtask that keeps queueing another microtask means the queue never empties, so the page never renders or takes the next task. It freezes.
- `console.log` is an ordinary call: it runs on the call stack and never goes in a queue. Only callbacks get queued.
- Rule of thumb: a normal call runs on the stack, a callback handed to the browser waits in a queue.
- Example (prints 1, 2, 3, 4):
  ```js
  console.log('1');
  setTimeout(() => console.log('4'), 0);
  Promise.resolve().then(() => console.log('3'));
  console.log('2');
  ```

### Starvation example (visualiser scenario 8)
```js
setTimeout(function timer() { console.log('timer: my turn at last'); }, 0);
let n = 0;
function loop() {
  console.log('microtask ' + ++n);
  Promise.resolve().then(loop);   // queues ANOTHER microtask
}
loop();
```
- The 0 ms timer expires straight away and joins the task queue, but it never runs. Each microtask queues the next one, so the microtask queue is never empty.
- The loop never reaches the task queue or rendering. In a real browser the page freezes: no painting, no clicks.
- The demo stops after 5 rounds so the animation can finish. In a real browser it would not stop.
- Contrast: a chain of `setTimeout`s doesn't starve. Each callback is a separate task, so the loop can paint between them (the "Chain of setTimeouts" button in event-loop-live.html).

## What goes on the call stack?
- Defining a function puts nothing on the stack. Calling it does.
- Valid version of the example: `const hello = () => { console.log('hello'); };` then `hello();`
- (`const hello = { console.log('hello') }` isn't valid JavaScript. Curly braces after `=` make an object literal, so you need `() =>` or `function`.)
- Calling `hello()` puts `hello` on the stack and runs it straight away. Inside it, `console.log` goes on top, runs, and pops off. Then `hello` pops off.
- Nothing goes in a queue. Only callbacks handed to the browser (timers, clicks, promises) wait in queues.

---

## Kept from the earlier version (not in your edited text, delete what you don't want)

### What is single-threaded, and what isn't
- Each tab has one main thread. Your JavaScript, the DOM, style, layout and most input handling share it.
- Chrome is multi-process: browser process, a renderer process per tab or site, GPU process, network service, extensions and utilities. Shift+Esc opens Chrome's own Task Manager to show this live.
- The Web APIs box runs outside the JS thread (timers, fetch, network). That is why a timer keeps counting while the stack is busy.
- Web Workers are extra JavaScript threads. Each has its own stack and event loop, and they talk to the main thread by passing messages. They can't touch the DOM.

### Don't mix up two kinds of "compile"
The build-time compile (your tools turning source into JavaScript) happens in your project, before anything ships. V8's JIT happens in the browser while the page runs.

### Plain-English labels (Explain like I'm 16 toggle)
- Web APIs = waiting room ⏱️
- Microtask queue = higher-priority queue 🚨
- Task queue = normal queue
- Call stack = what's running right now
- Event loop = the traffic cop

### Demo flow (event-loop-visualiser.html)
1. setTimeout basics
2. setTimeout 0 still waits
3. Promise beats setTimeout
4. Microtasks run between tasks
5. Blocking the loop
6. Events pile up behind a busy stack
7. A 3s timer joins the back of a busy queue (the main one)
8. Starve the loop with microtasks
9. Practical: setTimeout 0 so the page can paint
10. Audience quiz: guess the output (1, 4, 3, 2)

Also: event-loop-live.html runs real code and shows jank on a frame graph.

### Parked for later
- Compositor thread and raster threads: they handle scrolling and CSS transform and opacity animations off the main thread, so those can keep running while JavaScript is blocked. Worth a deeper look.
