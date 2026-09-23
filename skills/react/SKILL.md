---
name: react
description: Writing React with TypeScript and Vite — effects that do not lie, data fetching that handles its own failure, consuming Server-Sent Events, and error boundaries so one broken screen does not blank the app. Use when adding a component or screen, wiring a component to an API, debugging a stale value or a double-fired effect, or deciding where state belongs.
---

# React with TypeScript

This covers React itself. Where code goes — layers, slices, public APIs — is the
`feature-sliced-design` skill's question, not this one.

## An error boundary under every screen

Put this first because the failure it prevents is the worst one a React app has.
**An exception thrown during render unmounts the whole tree.** The reader gets a
blank document: no message, no console they are looking at, nothing to go back
to. One malformed field in one row takes down the entire application.

```tsx
import { Component, type ErrorInfo, type ReactNode } from 'react'

interface Props { what: string; children: ReactNode; escape?: ReactNode }
interface State { error: Error | null }

export class Boundary extends Component<Props, State> {
  state: State = { error: null }
  static getDerivedStateFromError(error: Error) { return { error } }
  componentDidCatch(error: Error, info: ErrorInfo) {
    console.error(`[${this.props.what}] failed to render`, error, info)
  }
  render() {
    if (!this.state.error) return this.props.children
    return (
      <section>
        <h2>{this.props.what} could not be rendered</h2>
        <p>{this.state.error.name}: {this.state.error.message}</p>
        {this.props.escape}
      </section>
    )
  }
}
```

Three things that are not obvious:

- **It must be a class.** There is no hook equivalent; this is the one place
  classes are still required.
- **Key it**, `<Boundary key={`${id}-${tab}`}>`. A boundary that has caught stays
  caught: without a key, navigating to a different record shows the previous
  one's error.
- **One per screen, not one around everything.** Screens read different data;
  losing all of them to one is the failure being prevented, in miniature.

It does not catch errors in event handlers, in `setTimeout`, or in async code —
only render, lifecycle and constructors. Handle those where they happen.

## Effects

Most `useEffect` calls are a mistake. Before writing one, check:

- **Deriving a value from props or state?** Compute it during render. An effect
  that sets state from other state renders twice and can go stale.
- **Responding to a user action?** Put it in the handler. An effect that watches
  a value to detect a click is guessing at intent it was already told.
- **Effects are for synchronising with something outside React**: a subscription,
  a timer, an event source, a fetch. That is the whole list.

```tsx
useEffect(() => {
  let live = true
  fetchRun(id)
    .then((data) => { if (live) setRun(data) })
    .catch((err) => { if (live) setError(err) })
  return () => { live = false }        // the response for a previous id must not win
}, [id])
```

**Always handle the unmount and the change of dependency.** Without the `live`
flag, switching quickly between two records can leave the first response
arriving last and overwriting the second — a race that is invisible on a fast
machine and reported as "sometimes it shows the wrong one".

**Effects run twice in development** under StrictMode, deliberately, to surface
missing cleanup. Do not disable it; fix the cleanup.

**Never lie about dependencies.** Removing a dependency to stop a loop hides the
loop rather than fixing it; the fix is usually a stable callback (`useCallback`
with honest deps) or moving the work out of the effect entirely.

## Consuming Server-Sent Events

```tsx
useEffect(() => {
  if (!runId) return
  const es = new EventSource(`/api/runs/${runId}/events`)
  es.addEventListener('progress', (e) => setProgress(JSON.parse(e.data)))
  es.addEventListener('done', () => { es.close(); onFinished() })
  es.onerror = () => { /* it reconnects on its own; do not close here */ }
  return () => es.close()
}, [runId, onFinished])
```

- **`EventSource` reconnects by itself.** Closing it in `onerror` turns a
  recoverable blip into a permanent failure.
- **The stream is a view, not the record.** When it ends, refetch the real state
  from the API rather than trusting what accumulated in memory — a missed event
  otherwise becomes a wrong screen.
- Fold accumulating updates in a local variable inside the effect when the next
  value is needed in the same tick; a state update is not readable that soon.

## State: where it belongs

- **Server data is not state.** It is a cache of something that lives elsewhere.
  Keep it with what it describes, refetch it rather than syncing it.
- **Lift only as far as the nearest common parent.** State at the top that one
  leaf uses re-renders everything between.
- **`useState` for values, `useRef` for things that must not cause a render** —
  a timer id, a "have we already done this" flag, a DOM node.
- **Derived values are computed, never stored.** Two sources for one fact will
  disagree, and the bug is reported as the display being wrong.

## Types that catch real bugs

```tsx
// The union makes the impossible state unrepresentable
type Load<T> =
  | { status: 'loading' }
  | { status: 'error'; error: string }
  | { status: 'ready'; data: T }
```

This beats `{ data?: T; loading: boolean; error?: string }`, which permits
`loading: true` with an error and data at the same time — and every consumer then
re-derives which combination means what.

**Do not trust data from the network at its type.** `await res.json()` is `any`
wearing an interface. A field the API stopped sending is `undefined` at runtime
and typed as present, and the render throws. Guard what you index into:
`critique.iterations ?? []`, `rows?.map(...)`.

## Rendering lists and text

- **Keys are identity, not position.** `key={item.id}`. `key={index}` on a list
  that reorders or filters gives components to the wrong data.
- **Do not build HTML strings.** `dangerouslySetInnerHTML` is named that way for
  a reason; render model-written or user-written text as text.
- Conditional rendering with `&&` on a number prints the number: `{count && <X/>}`
  renders `0`. Use a real ternary or `count > 0 &&`.

## Vite

- `import.meta.env.VITE_*` for build-time values. **Anything in the bundle is
  public** — no secret belongs there.
- `npm run build` runs the type check; a `tsc` error must fail the build, not be
  warned about.
- Dev-only server plugins take `apply: 'serve'`, so they can never end up in a
  production build.
