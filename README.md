# Immutable State Stores  
_... and why using immutable state stores is a good idea_

## Imperative

Take the following imperative program...

```javascript

let x = 3, y = 5;

x = x + 1;

y = y + x;

// x is now 4
// y is now 9

```

This seems normal and probably comfortable to most of us, but it also causes a lot of bugs in code. 
As more variables are introduced and more interactions occur, it becomes very difficult to follow the changes in state. 
When there is an issue of how something got into a particular state, the only way to trace the cause is to run the program with break points.

## Immutable

If instead we take an immutable approach we can now follow the changes in state.


```javascript

const x = 3, y = 5;

const x1 = x + 1;

const y1 = y + x1;

// the x stack
// x = 3
// x1 = 4

// the y stack
// y = 5
// y1 = 9

```

The problem here is that the variables holding the state are changing (eg. x to x1). This makes it very difficult to keep track of which variable even contains the current state.

## Immutable State Stores

If instead we store each stack as an array of state history, we can easily keep track of changes in state AND easily retrieve the current state.

```javascript

const x = [3], y = [5];

x.unshift(x[0] + 1);

y.unshift(y[0] + x[0]);

// x = [4, 3]
// y = [9, 5]

```

`x[0]` is always the current state of the x stack, while `x[1]` is the previous state, and so on. This is an **Immutable State Store**.

## State in a Single Store

If we store all state in a single place, we can keep track of how the whole program-state changes over time.

```javascript

const store = [{x: 3, y: 5}];

store.unshift(Object.assign({}, getState(), { x: getState().x + 1 }));

store.unshift(Object.assign({}, getState(), { y: getState().y + getState().x }));

function getState() { return store[0]; }

/* 
store = [
  {"x":4,"y":9},
  {"x":4,"y":5},
  {"x":3,"y":5}
]
*/

```

The only down-side is that immutably updating objects can be very verbose and fiddly. The more deeply in an object that you make the update, the more cumbersome it gets.

For example, if we were to immutably update 

```
{
  deep: {
    nested: "object"
  }
}
```

to 

```
{
  deep: {
    nested: "monkey"
  }
}
```

We would need to do the following...

```javascript

const a = {
  deep: {
    nested: "object"
  }
};

const a1 = Object.assign({}, a, {
  deep: Object.assign({}, a.deep, {
    nested: "monkey"
  })
});

```

## Zeron

Zeron is a functional frontend framework and provides the `iu` (immutable update) utility function for immutably updating objects. To do the above with Zeron's `iu` function looks like...

```javascript

const a = {
  deep: {
    nested: "object"
  }
};

const a1 = iu(a, 'deep/nested', 'monkey');

```

`iu` takes the starting object, the node to update, and the value to update the node to.

In Zeron we can directly access the current state with `getState()` and unshift a new state onto the Store with `state().set()`.

Zeron also has a debug mode that is turned on with `debug().on()`. Zeron's Debugger `console.logs` any state changes when set to on.

Put all this together and it makes it extremely easy to monitor state changes, and the functions that cause them.

```javascript

debug().on();

state().set({x: 3, y: 5});

state().set(
  iu(getState(), 'x', getState().x + 1)
);

state().set(
  iu(getState(), 'y', getState().y + getState().x)
)

/*
console.log...

Unshifting new state into Store...
Store: [
  {"x":4,"y":5},
  {"x":3,"y":5}
]

Unshifting new state into Store...
Store: [
  {"x":4,"y":9},
  {"x":4,"y":5},
  {"x":3,"y":5}
]

*/

```

By also debug-logging on the functions that cause the state change - we can make it extemely easy to debug state related issues.

```javascript

debug().on();

state().set({x: 3, y: 5});

// on button click
on($('#button'), 'click', () => {
  
  // log where the event is happening
  debug().log('Button click in myComponent');
  
  // Update the state-stack and log
  state().set(
    iu(getState(), 'x', getState().x + 1)
  );

  // Update the state-stack again and log
  state().set(
    iu(getState(), 'y', getState().y + getState().x)
  );
});

```

To turn off the debugging, just comment out or remove the `debug().on()` statement.
