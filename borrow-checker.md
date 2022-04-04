# Borrow Checker

Rust uses a compiler driven feature, kind of like a linter, that tracks the "ownership" of variables in a program.

A variable is owned by the scope it was declared in. The owner can then "move" the value to another scope or lend the value out for "reading" or "mutation". 

A variable can only have one owner and the owner can only lend out either one "mutable" borrow, or many "readable" borrows.

This allows the compiler to know when a variable is at risk of a data race (e.g. multiple scopes writing to it). It also allows the compiler to know when to make memory allocations and a memory deallocations at *compile time*.

This results in producing a binary that does not need a garbage collector and a ships a negligible runtime.

For example; we have three functions declared:
```javascript
// Asks for read only access to a string
const readLog = (read value: string) => console.log(read value)

// Asks for write access to a string
const writeLog = (write value: string) => value.push('bar')

// Asks for ownership transferal of a string
const moveLog = (move value: string) => console.log(read value)
```

If we declare a variable we can then give it to one of these functions to complete an operation. 

In the following example we create a variable and hand out one read borrow to the `readLog` function.
```typescript
let foo = 'foo'
readLog(read foo)
```
The read borrow given to `readLog` is temporary and is only present when that function is executing. 

It's helpful to imagine a sort of counter state tracking the borrows for `foo`.

|Owner|top level|
|-|-|
|reads|0|
|writes|0|

When given to `readLog`, the number of `reads` ticks up by one while the function executes, then ticks down when the function completes.

If we then hand a `write` borrow to `writeLog`, we will have one write borrow. Remember, only 1 write borrow **or** infinite read borrows.

```typescript
let foo = 'foo'

readLog(read foo)
writeLog(write foo)
```
In the above we tick up one `read`, tick down one `read`, tick up one `write`, tick down one `write`.

We can also change the owner of `foo` to another scope by using `move`.
```typescript
let foo = 'foo'
moveLog(move foo)
```
This now means that anything on the top level scope can no longer use `foo` as its owner is `moveLog`. This also means that the `foo` variable will be dropped from memory when the `moveLog` function completes.

So you can imagine the following code would fail
```typescript
let foo = 'foo'

moveLog(move foo)
readLog(read foo) // Error "tried to use 'foo' when you don't own 'foo'"
```

Tracking a variable's lifecycle like this means no need for a garbage collector.
Rust's idea is pretty clever, nice work Mozilla!

[Great short and simple video by YouCodeThings explaining the concept](https://www.youtube.com/watch?v=8M0QfLUDaaA)