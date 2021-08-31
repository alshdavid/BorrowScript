# Borrow Checker

TypeScript Static includes a borrow checker using descriptive operators to indicate the movement of ownership 

- [Borrow Checker](#borrow-checker)
  - [Operators](#operators)
  - [Borrow rules](#borrow-rules)
    - [Example](#example)

## Operators 

A variable can be passed to various scopes using `move` `read` `write` `copy`

```typescript
let myVar: string = 'Hello World!'

// Read only access to the value of text
foo(read myVar)
let readText: string = read originalText

// Give the foo function write access to the value of the text reference
foo(write myVar)
let writeText: string = write originalText

// Create a new variable that is identical to the original value and move it to the function
foo(copy myVar)
let copiedText: string = copy originalText

// Move the value's ownership from the parent into the function
foo(move myVar)
let movedText: string = move originalText

// Default action is to move a value
foo(myVar)
let movedText: string = originalText
```

## Borrow rules

Like Rust, per value, you can give out an unlimited number of read borrows or only one write borrow.

Using `copy` creates a new variable with the same value as the variable being copied.

A borrow ends when the scope holding the variables terminates.

### Example

```typescript
import console from '@std/console'

function readText(read value: string): void {
  console.log(value)
}

function writeText(write value: string): void {
  value.concat('Mutated text')
}

function main() {
  // main owns text
  let text: string = 'Hello World!'
  
  // main still own text but lends read-only access to readText
  // main cannot modify text until the readText function ends
  readText(read text)
  
  // Give write access of text to writeText which will allow mutation
  // of the value while preventing read/write access until the writeText 
  // function completes
  writeText(write text)

  // Once writeText has ended, main regains ownership and it hands out a
  // read to readText again
  readText(read text)

  // One readText has completed, main has access again and 
  // it reassigns the value 
  text = 'Updated Text'

  // Main can mutate the reference
  text.concat(' Mutating the reference')

  let movedText: string = move text
  console.log(movedText) // "Updated Text Mutating the reference"

  // Cannot use the original value after the value has been moved to "movedText"
  console.log(text) // Error, value of "text" moved to "movedText"
}
```