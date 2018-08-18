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



