# Exceptions

Exceptions are expensive and require additional runtime to be shipped in the binary. It might be worth considering exceptions an opt-in feature which is included in the binary if used.

```
BorrowScript \
  --input main.tsb \
  --output main \
  --include-exceptions
```