# fp-ts cheatsheet

This cheatsheet is intended for any reader to understand and remember fp-ts' most used features.

### Error or Value (Either)

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

### From a nullable value
When your original value is nullable, you can use `Either.fromNullable`:
```ts
import { fromNullable } from 'fp-ts/either'

// Without type signature
const mustExist = fromNullable(new CartMissingError())

// With explicit type signature
const mustExist: (cart: Cart | null) => Either<CartMissingError, Cart> = fromNullable(new CartMissingError())
```

### Using a [predicate](#predicate)
When you want to test whether a predicate is true and return an error if not, you can use `Either.fromPredicate`:
```ts
import { fromPredicate } from 'fp-ts/either'

import { PropDifferentError } from '../errors'

const mustBeEqual: (obj: MyObject) => Either<PropDifferentError, MyObject> = fromPredicate(
  obj => obj.prop === 'value',
  obj => new PropDifferentError(obj.prop, 'value')
)
```

### Using a [refinement](#refinement)
When you want to cast a value into another type, you can use the same function `Either.fromPredicate` with a refinement:
```ts
import { fromPredicate } from 'fp-ts/either'

import { NotCartError } from '../errors'

const mustBeCart: (cartOrList: Cart | List) => Either<NotCartError, cartOrList is Cart> = fromPredicate(
  cartOrList => cartOrList.type === 'cart',
  cartOrList => new NotCartError()
)
```

## Glossary

### Predicate
A predicate is a function with the following signature:
```ts
<A>(a: A) => boolean
```

### Refinement
A refinement is a special type of predicate that casts the given value in another type.
```ts
<A, B>(a: A) => a is B
```
