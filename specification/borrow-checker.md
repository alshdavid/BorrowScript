# Borrow Checker

TypeScript BC includes a borrow checker using descriptive operators to indicate the movement of ownership 

## Operators 

A variable can be passed to various scopes using the `Mut`, `Borrow` and `BorrowMut` generic types.

```typescript

// borrow can always be implied, so it's optional
function borrow(myFoo: Borrow<string>): void {
  console.log(myFoo);
}


// for mutable borrows, the BorrowMut<T> generic type should be used
function borrowMutable(myBar: BorrowMut<string>): void {
  // do something mutable with an immutable string somehow
}

let fooMut: Mut<string> = "a mutable string";
let foo: string = "an immutable string";

// fine
borrow(fooMut);
borrow(foo);

// fails borrow checker
borrowMutable(foo);
```

## Borrow rules

Like Rust, per value, you can give out an unlimited number of read borrows or only one write borrow.

Using the `copy()` function creates a new variable with the same value as the variable being copied.

A borrow ends when the scope holding the variables terminates.

### Example

```typescript
import console from '@std/console'

// borrows immutably
function readText(value: string): void {
  console.log(value)
}

// borrows mutably, and mutates value
function writeText(value: BorrowMut<string>): void {
  value.concat('Mutated text')
}

function main() {
  // main owns text
  let text: Mut<string> = 'Hello World!'
  
  // main still own text but lends read-only access to readText
  // main cannot modify text until the readText function ends
  readText(text)
  
  // Give write access of text to writeText which will allow mutation
  // of the value while preventing read/write access until the writeText 
  // function completes
  writeText(text)

  // Once writeText has ended, main regains ownership and it hands out a
  // read to readText again
  readText(text)

  // One readText has completed, main has access again and 
  // it reassigns the value 
  text = 'Updated Text'

  // Main can mutate the reference
  text.concat(' Mutating the reference')

  let movedText: string = text as Move<string> // move semantics can use a Move generic type
  console.log(movedText) // "Updated Text Mutating the reference"

  // Cannot use the original value after the value has been moved to "movedText"
  console.log(text) // Error, value of "text" moved to "movedText"
}
```
