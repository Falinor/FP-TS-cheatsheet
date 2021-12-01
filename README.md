# fp-ts cheatsheet

This cheatsheet is intended for any reader to understand and remember fp-ts' most used features.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [fp-ts cheatsheet](#fp-ts-cheatsheet)
  - [Summary](#summary)
  - [Combining functions](#combining-functions)
    - [Pipe](#pipe)
    - [Flow](#flow)
  - [Common constructors](#common-constructors)
    - [From a nullable value](#from-a-nullable-value)
    - [Using a predicate](#using-a-predicate)
    - [Using a refinement](#using-a-refinement)
  - [Either](#either)
    - [Useful functions](#useful-functions)
  - [Task](#task)
    - [Useful functions](#useful-functions-1)
  - [TaskEither](#taskeither)
    - [Useful functions](#useful-functions-2)
  - [Bind](#bind)
  - [What does `W` in `chainW` mean?](#what-does-w-in-chainw-mean)
  - [What does `K` in `fromNullableK` mean?](#what-does-k-in-fromnullablek-mean)
  - [Glossary](#glossary)
    - [Context value](#context-value)
    - [Predicate](#predicate)
    - [Refinement](#refinement)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Combining functions

### Pipe
Definition: transform an expression in a data-first pipeline of functions.
```ts
import { pipe } from 'fp-ts/function'

const masters = ['Yoda', 'Vador', 'Luke']
pipe(
  masters,
  map(master => master.length),
  filter(length => length > 4)
)
```

### Flow
Definition: like pipe but data-last.
```ts
import { flow } from 'fp-ts/function'

const value = ['Yoda', 'Vador', 'Luke']
flow(
  map(master => master.length),
  filter(length => length > 4),
)(masters)
```

## Common constructors

For all the monads that can fail below, these constructors exists

### From a nullable value
```ts
import { fromNullable } from 'fp-ts/Either'

// Without type signature
const mustExist = fromNullable(new CartMissingError())

// With explicit type signature
const mustExist: (cart: Cart | null) => Either<CartMissingError, Cart> = fromNullable(new CartMissingError())
```

### Using a [predicate](#predicate)
Test whether a predicate is true and return an error if not.
```ts
import { fromPredicate } from 'fp-ts/Either'

import { PropDifferentError } from '../errors'

const mustBeEqual: (obj: MyObject) => Either<PropDifferentError, MyObject> = fromPredicate(
  obj => obj.prop === 'value',
  obj => new PropDifferentError(obj.prop, 'value')
)
```

### Using a [refinement](#refinement)
Like a predicate but casts the given value into another type.
```ts
import { fromPredicate } from 'fp-ts/Either'

import { NotCartError } from '../errors'

const mustBeCart: (cartOrList: Cart | List) => Either<NotCartError, cartOrList is Cart> = fromPredicate(
  cartOrList => cartOrList.type === 'cart',
  cartOrList => new NotCartError()
)
```

## Either
Definition: an error (left) or a value (right).
```ts
interface Left<E> {
  readonly _tag: 'Left'
  readonly left: E
}

interface Right<A> {
  readonly _tag: 'Right'
  readonly right: A
}

type Either<E, A> = Left<E> | Right<A>
```

### Useful functions
Return error.
```ts
function left<E = never, A = never>(e: E) => Either<E, A>

left(new Error('error'))
```

Return success.
```ts
function right<E = never, A = never>(a: A) => Either<E, A>

right(42)
```

Map the inner type to another type.
```ts
function map<A, B>(f: (a: A) => B): <E>(fa: Either<E, A>) => Either<E, B>

const either = right('string')
map(str => str.length)(either)
```

Chain with another Either.
```ts
function chain<E, A, B>(f: (a: A) => Either<E, B>): (ma: Either<E, A>) => Either<E, B>

pipe(
  right(age),
  chain(age => age < 18 ? left(new TooYoungError(age)) : right(age)
)

// With `fromPredicate`
const isOldEnough = fromPredicate(
  age => age < 18,
  age => new TooYoungError(age)
)

pipe(
  right(age),
  chain(isOldEnough)
)
```
You may use `Either.chainW` to [_widen_ the error type](#what-does-w-in-chainw-mean).

## Task
Definition: an asynchronous computation that yields a value and **never fails**.
```ts
interface Task<A> {
  (): Promise<A>
}
```

Task is often used when one destroys a TaskEither e.g.
```ts
await pipe(
  useCase.execute(), // => TaskEither<E, A>
  fold(
    error => Task.of(error),
    value => Task.of(value)
  )
)()
```
### Useful functions

delay(millis: number) : Creates a task that will complete after a time delay.
```ts
  pipe(
      T.chain(() => doSomething()),
      T.chain(() => delay(1000)),
      T.map(() => 'done after 1000ms')
  )
```

## TaskEither
Definition: an asynchronous computation that **either yields a value** of type A or **fails, yielding an error** of type E
```ts
interface TaskEither<E, A> extends Task<Either<E, A>> {}
```

### Useful functions

TODO

## Bind
Definition: bind allows to store a read-only context value in the composition pipeline, so that it can be reused later.

```ts
pipe(
  bind('user', fromThunk(userRepository.save)),
  // Here is the ContextValue
  ({ user }) => {
    console.log(user)
  }
)

// This is inferred by typescript
interface ContextValue {
  readonly user: User
}
```
It forbids you to mutate the context value.

## What does `W` in `chainW` mean?
`W` means a function is able to aggregate errors into a union (for Either based data types) or environments into an intersection (for Reader based data types).
```ts
import * as E from 'fp-ts/Either'
import * as TE from 'fp-ts/TaskEither'
import { pipe } from 'fp-ts/function'

declare function parseString(s: string): E.Either<string, number>
declare function fetchUser(id: number): TE.TaskEither<Error, User>

// This raises an error because: Type 'string' is not assignable to type 'Error'
const program_ = (s: string) => pipe(s, TE.fromEitherK(parseString), TE.chain(fetchUser))

// const program: (s: string) => TE.TaskEither<string | Error, User>
const program = (s: string) => pipe(s, TE.fromEitherK(parseString), TE.chainW(fetchUser))
```

## What does `K` in `fromNullableK` mean?
`K` in `fromNullableK` is used to do a `fromNullable` with a transformation (by a function) apply to the given agurments `<A>`
And then return either `NonNullable<B>` or an error

```ts
export declare const fromNullableK: <E>(
  e: E
) => <A extends readonly unknown[], B>(f: (...a: A) => B | null | undefined) => (...a: A) => Either<E, NonNullable<B>>
```


`K` means Kleisli. A Kleisli arrow is a function with the following signature
```ts
(a: A) => F<B>
```
It's a sufix that we can find on other functions around the lib. ([more info on sufix](https://gcanti.github.io/fp-ts/guides/code-conventions.html))


## Glossary

### Context value
The value that transits throughout a pipe. See [bind](#bind).

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
