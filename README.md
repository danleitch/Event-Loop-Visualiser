# Event Loop Visualiser

An animated, step-by-step way to see how the JavaScript event loop works in the browser. Built for a talk, so it's made to be stepped through in front of an audience.

No install, no build step, no dependencies. Download the files and open the HTML in any modern browser.

## What's in here

| File | What it is |
| --- | --- |
| `event-loop-visualiser.html` | The main animated walkthrough: code panel, console, call stack, Web APIs, microtask queue, task queue and the event loop. |
| `event-loop-live.html` | Runs real code in your browser and shows what actually happens, including a frame-time graph that shows jank when the main thread is blocked. |
| `speaking-notes.md` | Talk notes: how code gets from your editor to the page, the engine basics, and the event loop explained. |

## The visualiser

Pick a scenario from the dropdown and press **Run**.

- **Auto-play** steps through on its own. Turn it off and use **Next** (or the right arrow / space bar) to go at your own pace.
- **Speed** goes from 0.5x to 2x.
- **Explain like I'm 16** adds plain-English labels to each box and simpler narration, plus a list of things that go in the microtask queue with links to learn more.

Scenarios:

1. `setTimeout` basics
2. `setTimeout` 0 still waits
3. Promise beats `setTimeout`
4. Microtasks run between tasks
5. Blocking the loop
6. Events pile up behind a busy stack
7. A 3s timer joins the back of a busy queue
8. Starve the loop with microtasks
9. Practical: `setTimeout` 0 so the page can paint
10. Audience quiz: guess the output

## Things to know

- The visualiser is a **simulation**. Each scenario is a small script whose steps are animated in the order a browser would run them. Timer durations are sped up so you don't have to wait.
- It uses one task queue as a teaching model. Real browsers keep several task queues and may prioritise between them.
- Painting is shown as a simple marker in scenario 9. Real painting depends on the browser's frame timing.
- `event-loop-live.html` is the opposite: it runs real code, so results can vary slightly between runs and browsers.

## Running it

Open either HTML file directly in your browser (double-click, or drag it into a browser window).

To serve it locally instead:

```sh
python3 -m http.server 8000
```

Then visit `http://localhost:8000/event-loop-visualiser.html`.

## Credits

Inspired by Philip Roberts' talk [What the heck is the event loop anyway?](https://www.youtube.com/watch?v=8aGhZQkoFbQ) (JSConf EU 2014) and his Loupe visualiser.
