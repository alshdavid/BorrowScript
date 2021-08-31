# Decorators

TODO

Decorators are available on class declaration, class methods, class properties, function declarations and variable declarations.

```typescript
@FooClass
class Foo {
  @FooClassProp 
  private _value: string = ''

  @FooClassMethod
  foo() {

  }
}

@FooFunc
function foo() {

}

@FooProp
const foobar: string = ''
```

Decorators are syntax sugar for functions that wrap around the initialization of a variable. 

```typescript
function Decorator(assignedValue: string) {
  return `Decorated - ${assignedValue}`
}

@Decorator
const foobar: string = 'Hello World!'

console.log(foobar) // "Decorated - Hello World!"

```