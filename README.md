# optics-ts

[![CircleCI](https://circleci.com/gh/akheron/optics-ts.svg?style=shield)](https://circleci.com/gh/akheron/optics-ts)

`optics-ts` provides type-safe, ergonomic, polymorphic optics for
TypeScript:

- **Optics** allow you to read or modify values from deeply nested
  data structures, while keeping all data immutable.
- **Ergonomic**: Optics are composed with method chaining, making it
  easy and fun!
- **Polymorpic**: When writing through the optics, you can change the
  data types in the nested structure.
- **Type-safe**: The compiler will type check all operations you do.
  No `any`, ever.

`optics-ts` supports equivalences, isomorphisms, lenses, prisms and
traversals.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

## Table of Contents

- [Installation](#installation)
- [Tutorial](#tutorial)
  - [Lens](#lens)
  - [Prism](#prism)
  - [Traversal](#traversal)
  - [Other types of optics](#other-types-of-optics)
  - [Polymorphism](#polymorphism)
- [API reference](#api-reference)
  - [Types of optics](#types-of-optics)
  - [Method chaining](#method-chaining)
  - [Type parameters](#type-parameters)
  - [Top-level functions](#top-level-functions)
    - [`optic<S>(): Equivalence<S, _, S>`](#optics-equivalences-_-s)
    - [`optic_<S>(): Equivalence<S, _, S>`](#optic_s-equivalences-_-s)
    - [`get<S, A>(optic: Optic<S, _, A>) => (source: S) => A`](#gets-aoptic-optics-_-a--source-s--a)
    - [`preview<S, A>(optic: Optic<S, _, A>) => (source: S) => A | undefined`](#previews-aoptic-optics-_-a--source-s--a--undefined)
    - [`collect<S, A>(optic: Optic<S, _, A>) => (source: S) => A[]`](#collects-aoptic-optics-_-a--source-s--a)
    - [`modify<S, T, A>(optic: Optic<S, T, A>) => <B>(f: (a: A) => B) => (source: S) => T<B>`](#modifys-t-aoptic-optics-t-a--bf-a-a--b--source-s--tb)
    - [`set<S, T, A>(optic: Optic<S, T, A>) => <B>(value: B) => (source: S) => T<B>`](#sets-t-aoptic-optics-t-a--bvalue-b--source-s--tb)
    - [`compose<S, A1, A2><optic1: Optic<S, _, A1>, optic2: Optic<A1, _, A2>): Optic<S, _, A2>`](#composes-a1-a2optic1-optics-_-a1-optic2-optica1-_-a2-optics-_-a2)
  - [Creating optics](#creating-optics)
  - [Isomorphisms](#isomorphisms)
    - [`iso<U>(there: (a: A) => U, back: (u: U) => A): Iso<S, _, U>`](#isouthere-a-a--u-back-u-u--a-isos-_-u)
  - [Lenses](#lenses)
    - [`prop<K extends keyof A>(key: K): Lens<S, _, A[K]>`](#propk-extends-keyof-akey-k-lenss-_-ak)
    - [`path<K1, K2, ...>(keys: [K1, K2, ...]): Lens<S, _, A[K1][K2]...>`](#pathk1-k2-keys-k1-k2--lenss-_-ak1k2)
    - [`pick<K extends keyof A>(keys: K[]): Lens<S, _, Pick<A, K>>`](#pickk-extends-keyof-akeys-k-lenss-_-picka-k)
  - [Prisms](#prisms)
    - [`optional(): Prism<S, _, Exclude<A, undefined>>`](#optional-prisms-_-excludea-undefined)
    - [`guard<U extends A>(g: (a: A) => a is U): Prism<S, _, U>`](#guardu-extends-ag-a-a--a-is-u-prisms-_-u)
    - [`guard_<F extends HKT>(): <U extends A>(g: (a: A) => a is U) => Prism<S, T · F, U>`](#guard_f-extends-hkt-u-extends-ag-a-a--a-is-u--prisms-t-%C2%B7-f-u)
    - [`index(i: number): Prism<S, _, ElemType<A>>`](#indexi-number-prisms-_-elemtypea)
    - [`find(p: (e: ElemType<A>) => boolean): Prism<S, _, ElemType<A>>`](#findp-e-elemtypea--boolean-prisms-_-elemtypea)
    - [`when(f: (a: A) => boolean): Prism<S, _, A>`](#whenf-a-a--boolean-prisms-_-a)
  - [Traversals](#traversals)
    - [`elems(): Traversal<S, _, ElemType<A>>`](#elems-traversals-_-elemtypea)
  - [Composing](#composing)
    - [`compose<B>(other: Optic<A, _, B>): Optic<S, _, B>`](#composebother-optica-_-b-optics-_-b)
- [Prior art](#prior-art)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Installation

```
npm install --save optics-ts
```

or

```
yarn add optics-ts
```

## Tutorial

The following import is assumed in all the examples below:

```typescript
import * as O from 'optics-ts'
```

### Lens

Lens is the most common optic you're going to use. You can create an
optic for a data structure by calling `O.optic()`, and turn in into a
lens that focuses on a property of an object with `.prop()`:

```typescript
type Data = {
  foo: { bar: number }
  other: string
}
const foo = O.optic<Data>().prop('foo')
```

`foo` is now a lens that focuses on `Data.foo`.

To dig deeper, just call `.prop()` again:

```typescript
const bar = O.optic<Data>()
  .prop('foo')
  .prop('bar')
// or from the `foo` lens we defined above
const bar = foo.prop('bar')
// or use .path() to compose multiple prop lenses with a single call
const bar = O.optic<Data>().path(['foo', 'bar'])
```

Use `get()` to read a value through the lens:

```typescript
const data: Data = {
  foo: { bar: 42 },
  other: 'stuff',
}

O.get(lens)(data)
// => 42
```

Use `set()` or `modify()` to write the focused value through the lens:

```typescript
O.set(lens)(99)(data)
// => {
//   foo: { bar: 99 },
//   other: 'stuff'
// }

O.modify(lens)(x => x * 100)(data)
// => {
//   foo: { bar: 4200 },
//   other: 'stuff'
// }
```

Writing through optics always creates a new data structure instead of
modifying the existing one. In other words, data is immutable.

### Prism

Lenses are great for focusing to a part of a larger structure. Prisms
are much like lenses, but they don't necessarily match anything, i.e.
they can have zero focuses.

A practical example isfocusing on a branch of a union type. Here, the
`User.age` field can be `number` or `undefined`. With the `.optional()`
prism we can focus only when the value is a `number`, and do nothing
when it's `undefined`:

```typescript
type User = {
  name: string
  age?: number | undefined
}

const age = O.optic<User>()
  .prop('age')
  .optional()
```

You read through a prism using the `preview()` function. When the prism
doesn't match, it returns `undefined`.

```typescript
const userWithAge: User = {
  name: 'Betty',
  age: 42,
}
O.preview(age)(userWithAge)
// ==> 42

const userWithoutAge: User = {
  name: 'Max',
  age: undefined,
}
O.preview(age)(userWithoutAge)
// ==> undefined
```

If the prism doesn't match, `preview()` returns undefined, as seen above.

You can write through a prism normally with `set()` and `modify()`. If
the prism doesn't match, the value is unchanged:

```typescript
O.modify(age)(n => n + 1)(userWithAge)
// ==> {
//   name: 'Betty',
//   age: 43,
// }

O.set(age)(60)(userWithoutAge)
// ==> {
//   name: 'Max',
//   age: undefined,
// }
```

`.guard()` is another method that creates a prism. It's a generalization
of `.optional()` in the sense that you can match on any branch of a
union type instead of just the non-`undefined` part:

```typescript
interface Square {
  kind: 'square'
  size: number
}
interface Rectangle {
  kind: 'rectangle'
  width: number
  height: number
}
type Shape = Square | Rectangle

function isRectangle(s: Shape): s is Rectangle {
  return s.kind === 'rectangle'
}

const rectWidth = O.optic<Shape>()
  .guard(isRectangle)
  .prop('width')

O.preview(rectWidth)({ kind: 'square', size: 10 })
// ==> undefined

O.preview(rectWidth)({ kind: 'rectangle', width: 5, height: 7 })
// ==> 5

O.modify(rectWidth)(w => w * 2)({ kind: 'rectangle', width: 5, height: 7 })
// ==> { kind: 'rectangle', width: 10, height: 7 })
```

Notice how above we used `.guard(...).prop(...)`, composing a prism with
a lens. This yields a prism, so we used `preview()` to read through it.
See [Types of optics](#types-of-optics) for the rules of composition.

### Traversal

The next optic type is the traversal. While lenses have 1 focus and
prisms have 0 or 1 focus (no match or match), traversals have 0 or
more focuses.

The simplest example of a traversal is to focus on the elements of an
array. To create such a traversal, use `.elems()`:

```typescript
type Person {
  name: string
  friends: Person[]
}

const friendsNames = O.optic<Person>()
  .prop('friends')
  .elems()
  .prop('name')
```

To read through a traversal, call `collect()` to collect all focused
elements into an array:

```typescript
const john = { name: 'John', friends: [] }
const bruce = { name: 'Bruce', friends: [] }
const amy = { name: 'Amy', friends: [john, bruce] }

O.collect(friendsNames)(amy)
// ==> [ 'John', 'Bruce' ]
```

Writing through a traversal writes to all focused values:

```typescript
O.modify(friendsNames)(name => `${name} Wayne`)(amy)
// ==> {
//   name: 'Amy',
//   friends: [
//     { name: 'John Wayne', friends: [] },
//     { name: 'Bruce Wayne', friends: [] },
//   ],
// }
```

Note again how we used `.prop(...).elems(...).prop(...)`, composing a
lens with a traversal, and then with a lens again. This yields a
traversal. See [Types of optics](#types-of-optics) for more info.

It's sometimes useful to further focus on certain elements of a
traversal. This can be done by composing a traversal with a prism like
`.when()` that skips items that don't match a predicate:

```typescript
const even = O.optic<number[]>()
  .elems()
  .when(n => n % 2 === 0)

O.modify(even)(n => -n)([1, 2, 3, 4, 5])
// ==> [1, -2, 3, -4, 5]
```

### Other types of optics

In fact, calling `O.optic()` also yields an optic, but instead of
being a lens, prism or traversal, it's an equivalence. As the name
suggests, equivalence keeps the value equal, in both reading and
writing directions:

```typescript
const str = O.optic<string>()

get(str)('original')
// ==> 'original'

set(str)('new')('original')
// ==> 'new' ('original' is discarded)
```

`optics-ts` also supports isomorphisms, which can be used to do 2-way
data transformations.

### Polymorphism

Optics can be polymorphic, which means you can change the type of the
focus when you write through an optic. Since this is a relatively rare
use case, and may be confusing if done by accident, polymorphic optics
are created with `optic_()` (note the underscore):

```typescript
type Data = {
  foo: { bar: string }
  other: boolean
}
const bar = O.optic_<Data>().path(['foo', 'bar'])
```

Let's modify `bar` to contain the length of the original string instead:

```typescript
const data: Data = {
  foo: { bar: 'hello there' },
  other: true,
}

const updated = O.modify(bar)(str => str.length)(data)
// ==> {
//   foo: { bar: 11 },
//   other: true
// }
```

This is a type-safe operation, i.e. the compiler knows that the type of
`updated.foo.bar` is `number`, editor autocomplete works correctly, etc.

If you ever see a `DisallowedTypeChange` type being returned from an
`optics-ts` function, it means that you tried to change the type when
writing through a monomorphic optic.

## API reference

### Types of optics

The supported optic classes are equivalence, isomorphism, lens, prism
and traversal. With this (incomplete) optics hierarchy, we can put the
optic classes in order:

```
Equivalence < Iso < Lens < Prism < Traversal
```

When you compose two optics, the result is the "greater" of the two,
i.e. the one that appears rightmost.

For example, composing an `Iso` with a `Prism` yields a `Prism`.
Composing an `Traversal` with a `Lens` yields a `Traversal`.

### Method chaining

Optics are composed with method chaining. This means that each optic
type has all the methods documented below, regardless of the type of the
optic that the method creates. The only difference is the return type,
which is determined by the composition rules above.

For example, assume we have a variable `myLens` that holds a `Lens`, and
call `.optional()` on it:

```typescript
const newOptic = myLens.optional()
```

`.optional()` creates a prism, so `newOptic` will be a composition of
lens and prism, i.e. a prism.

### Type parameters

All optics have 3 type parameters: `<S, T, A>`:

- `S` is the source on which the optic operates

- `A` is the type of the focus or focuses

- `T` is a "higher-kinded type" or a "partially applied type operator"
  that yields the output type when applied to some type `B`.

Conceptually, when you write a value of type `B`, the output type will
be `S` with `A` replaced by `B` at the focus(es) of the optic. `T` is
the mechanism that transforms `B` to the output type. This construct
makes it possible for the optics to be polymorphic on the type level.

In the following, we leave the exact definition of `T` for each optic
out for clarity, writing just `_` in its place. It's usually clear fom
how the optic works what will come out if you put a different type in.

In the documentation of functions that can be used to write through an
optic, the return type is denoted by `T<B>`. While not valid TypeScript
syntax (because `T` is a type parameter instead of a concrete type),
this captures the meaning quite well: `B` is applied to the
higher-kinded type `T`, yielding the output type.

Interested readers can refer to [hkt.ts](src/hkt.ts) to see how the
higher-kinded types / partially applied type operators are actually
implemented.

### Top-level functions

These functions are available as top level exports of the `optics-ts`
module.

Most functions have `Optic` in their signature. It means that multiple
optics work with the function. The optic classes that are actually
applicable are documented in the function description.

#### `optic<S>(): Equivalence<S, _, S>`

Create a monomorphic equivalence for `S`. If you ever see the type
`DisallowedTypeChange`, it means that you have attempted to change a
type with a monomorphic optic.

#### `optic_<S>(): Equivalence<S, _, S>`

Create a polymorphic equivalence for `S`.

#### `get<S, A>(optic: Optic<S, _, A>) => (source: S) => A`

Read a value through an `Equivalence`, `Iso` or `Lens`.

#### `preview<S, A>(optic: Optic<S, _, A>) => (source: S) => A | undefined`

Read a value through a `Prism` or `Traversal`. For `Prism`, return
`undefined` if the prism doesn't match. For `Traversal`, returns the
value of the first focus, or `undefined` if there are no focuses.

#### `collect<S, A>(optic: Optic<S, _, A>) => (source: S) => A[]`

Read all focused values through a `Prism` or `Traversal`. For `Prism`,
the return value is an array of 0 or 1 elements. For `Traversal`, the
return value is an array of zero or more elements.

#### `modify<S, T, A>(optic: Optic<S, T, A>) => <B>(f: (a: A) => B) => (source: S) => T<B>`

Modify the focused value(s) through an `Equivalence`, `Iso`, `Lens`,
`Prism` or `Traversal`. Returns an updated copy of `source` with all
focuses modified by mapping them through the function `f`.

#### `set<S, T, A>(optic: Optic<S, T, A>) => <B>(value: B) => (source: S) => T<B>`

Write a constant value through an `Equivalence`, `Iso`, `Lens`, `Prism`
or `Traversal`. Returns an updated copy of `source` with all focuses
replaced by `value`.

#### `compose<S, A1, A2><optic1: Optic<S, _, A1>, optic2: Optic<A1, _, A2>): Optic<S, _, A2>`

Compose two optics. If the first optic is from `S` to `A1`, and the
second optic is from `A1` to `A2`, the result is from `S` to `A2`.

See [Types of optics](#types-of-optics) for the rules of composition.

### Creating optics

The methods documented below are available on all optics types:
`Equivalence`, `Iso`, `Lens`, `Prism` and `Traversal`. The documented
return type is the type of the optic that these methods create. The
actual return type is the composition of the optic on which the method
is called and on the optic that the method creates.

### Isomorphisms

Isomorphisms have the type `Iso<S, T, A>`. In the following, we omit
the exact definition of `T` for clarity, and use `_` instead. See
[Type parameters](#type-parameters) for the meanings of type
parameters.

#### `iso<U>(there: (a: A) => U, back: (u: U) => A): Iso<S, _, U>`

Create an isomorphism from functions `there` and `back`. `there` takes
the focus and transforms it to another value. `back` is the inverse of
`there`.

Note that `iso` is monomorphic. There's no polymorphic alternative
(yet).

### Lenses

Lenses have the type `Lens<S, T, A>`. In the following, we omit the
exact definition of `T` for clarity, and use `_` instead. See [Type
parameters](#type-parameters) for the meanings of type parameters.

#### `prop<K extends keyof A>(key: K): Lens<S, _, A[K]>`

Create a lens that focuses on the property `K` of `A`.

**Note:** Only works for string properties, even though TypeScript's
type system also allows array's numeric indices when using `keyof`. Use
the `index()` prism to focus on an array element at a given index.

#### `path<K1, K2, ...>(keys: [K1, K2, ...]): Lens<S, _, A[K1][K2]...>`

A shortcut for focusing on chain of properties.

```typescript
foo.path(['a', 'b', 'c'])
```

is equal to

```typescript
foo
  .prop('a')
  .prop('b')
  .prop('c')
```

#### `pick<K extends keyof A>(keys: K[]): Lens<S, _, Pick<A, K>>`

Create a lens that focuses on a sub-object of `A` with the given
properties. When writing through a polymorphic `.pick()` lens, you can
add or remove properties.

Example:

```typescript
const data = {
  foo: 'something',
  bar: 42,
  baz: true,
}
const lens = O.optic_<typeof data>().pick(['foo', 'bar'])

O.get(lens)(data)
// ==> {
//  foo: 'something',
//  baz: true,
// }

O.set(lens)({ quux: null })(data)
// ==> {
//   quux: null,
//   baz: true,
// }

// monomorphic version of the same lens
const monoLens = O.optic<typeof data>().compose(lens)

O.set(monoLens)({ quux: null })(data)
// ==> DisallowedTypeChange
```

### Prisms

Prisms have the type `Prism<S, T, A>`. In the following, we omit the
exact definition of `T` for clarity, and use `_` instead. See [Type
parameters](#type-parameters) for the meanings of type parameters.

#### `optional(): Prism<S, _, Exclude<A, undefined>>`

Create a prism that focuses on the non-`undefined` subtype of `A`.

#### `guard<U extends A>(g: (a: A) => a is U): Prism<S, _, U>`

Create a prism that focuses on the subtype `U` of `A` that matches the
type guard `g`.

Note that `guard()` is monomorphic. Use `guard_` if you want a
polymorphic guard.

#### `guard_<F extends HKT>(): <U extends A>(g: (a: A) => a is U) => Prism<S, T · F, U>`

Create a prism that focuses on the subtype of `A` that matches the type
guard `g`. When written to, uses the higher-kinded type `F` to construct
the output type.

#### `index(i: number): Prism<S, _, ElemType<A>>`

Only works on array types. `ElemType<A>` is the element type of the
array type `A`.

Create a prism that focuses on index `i` of the focus array.

When a different type `B` is written through this optic, the resulting
array will have the type `Array<A | B>`.

#### `find(p: (e: ElemType<A>) => boolean): Prism<S, _, ElemType<A>>`

Only works on array types. `ElemType<A>` is the element type of the
array type `A`.

Like `.index()`, but the index to be focused on is determined by
finding the first element that matches the given predicate.

When a different type `B` is written through this optic, the resulting
array will have the type `Array<A | B>`.

#### `when(f: (a: A) => boolean): Prism<S, _, A>`

Create a prism that skips the focus if it doesn't match the given
predicate. Especially useful for filtering the focuses of a travesal.

When a different type `B` is written through this optic, the resulting
value will have the type `A | B`.

### Traversals

Traversals have the type `Traversal<S, T, A>`. In the following, we
omit the exact definition of `T` for clarity, and use `_` instead. See
[Type parameters](#type-parameters) for the meanings of type
parameters.

#### `elems(): Traversal<S, _, ElemType<A>>`

Only works on array types. `ElemType<A>` is the element type of the
array type `A`.

Create a traversal that focuses on all the elements of the array.

### Composing

#### `compose<B>(other: Optic<A, _, B>): Optic<S, _, B>`

`optic.compose(other)` is equivalent to `compose(optic, other)`.

## Prior art

There are many existing optics libraries of varying degree for
JavaScript, but only few for TypeScript. It's generally hard to create
good typings for optics in TypeScript, and the task becomes impossible
if one tried to retrofit types on an existing JavaScript implementation.

[monocle-ts](https://github.com/gcanti/monocle-ts) is probably the most
popular TypeScript optics library. It lacks polymorphism, and creating
optics is verbose and cumbersome. You first create optics for each level
of the data structure, and then compose them with separate function
calls. This also often requires you to declare many unnecessary
intermediate types for your data.

[@grammarly/focal](https://github.com/grammarly/focal) is not an optics
library per se, rather an UI framework. Its optics are very limited.
