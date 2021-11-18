# fp-ts cheatsheet

This cheatsheet is intended for any reader to understand and remember fp-ts' most used features.

### Error or Value (Either) {#either}

The type `Either` represents an error or a value. 

```ts
interface Left<E> {
  value: E
}

interface Right<A> {
  value: A
}

type Either<E, A> = Left<E> | Right<A>
```

### Using a [predicate](#predicate)

When you want to test whether a predicate

## Glossary

### Predicate
A predicate with the following signature:
```ts
<A>(a: A) => boolean
```

### Refinement
