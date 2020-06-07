---
'xstate': major
---

**Breaking:** Activities are no longer a separate concept. Instead, activities are invoked. Internally, this is how activities worked. The API is consolidated so that `activities` are no longer a property of the state node or machine options:

```diff
import { createMachine } from 'xstate';
+import { invokeActivity } from 'xstate/invoke';

const machine = createMachine(
  {
    // ...
-   activities: 'someActivity',
+   invoke: {
+     src: 'someActivity'
+   }
  },
  {
-   activities: {
+   behaviors: {
-     someActivity: ((context, event) => {
+     someActivity: invokeActivity((context, event) => {
        // ... some continuous activity

        return () => {
          // dispose activity
        }
      })
    }
  }
);
```

**Breaking:** The `services` option passed as the second argument to `createMachine(config, options)` is renamed to `behaviors`. Each value in `behaviors`should be a function that takes in `context` and `event` and returns a [behavior](TODO: link). The provided invoke creators are:

- `invokeActivity`
- `invokePromise`
- `invokeCallback`
- `invokeObservable`
- `invokeMachine`

```diff
import { createMachine } from 'xstate';
+import { invokePromise } from 'xstate/invoke';

const machine = createMachine(
  {
    // ...
    invoke: {
      src: 'fetchFromAPI'
    }
  },
  {
-   services: {
+   behaviors: {
-     fetchFromAPI: ((context, event) => {
+     fetchFromAPI: invokePromise((context, event) => {
        // ... (return a promise)
      })
    }
  }
);
```

**Breaking:** The `state.children` property is now a mapping of invoked actor IDs to their `ActorRef` instances.

**Breaking:** The way that you interface with invoked/spawned actors is now through `ActorRef` instances. An `ActorRef` is an opaque reference to an `Actor`, which should be never referenced directly.

**Breaking:** The `spawn` function is no longer imported globally. Spawning actors is now done inside of `assign(...)`, as seen below:

```diff
-import { createMachine, spawn } from 'xstate';
+import { createMachine } from 'xstate';

const machine = createMachine({
  // ...
  entry: assign({
-   someRef: (context, event) => {
+   someRef: (context, event, { spawn }) => {
-     return spawn(somePromise);
+     return spawn.from(somePromise);
    }
  })
});

```

**Breaking:** The `src` of an `invoke` config is now either a string that references the machine's `options.behaviors`, or a `BehaviorCreator`, which is a function that takes in `context` and `event` and returns a `Behavior`:

```diff
import { createMachine } from 'xstate';
+import { invokePromise } from 'xstate/invoke';

const machine = createMachine({
  // ...
  invoke: {
-   src: (context, event) => somePromise
+   src: invokePromise((context, event) => somePromise)
  }
  // ...
});
```

**Breaking:** The `origin` of an `SCXML.Event` is no longer a string, but an `ActorRef` instance.