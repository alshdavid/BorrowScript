# Classes and Inheritance

This section is subject to the whims of my personal programming style and will require additional input from external sources.

I prefer composition to inherence and seldom use it. I would be find with excluding class inherence from the BorrowScript specification.

```typescript
// Illegal
class A {}

class B extends A {}
```

```typescript
// Legal
class A {}

class B {
  private a: A

  constructor() {
    this.a = new A()
  }
}
```