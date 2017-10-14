# ESNext Proposal: Monadic chain function

This proposal introduces a syntactic sugar for the monad "bind" operator (sometimes called beefed up function composition or programmable semicolon).

## Introduction

The `chain/from` function syntax is esentially an elegant way to express a composition of monadic functions, similar to how `async/await` syntax is sort of a consice expression of a `Promise` chain. In fact `chain/from` can be thought of as a generalization of `async/await` that does not restrict users to the built-in `Promise`, you can `chain` any type that implements the [`Monad` protocol](#monad-protocol).

```js
import S from 'sanctuary';

const users = [
  { id: 1, firstName: 'Alice', lastName: 'Cooper' },
  { id: 2, firstName: 'Bob',   lastName: 'Marley' }
];

// Number -> Maybe String
const findFullName = chain (id_) => {
  const { firstName, lastName } = from S.find(({ id }) => id === id_, users);
  return `${firstName} ${lastName}`;
};

findFullName(2);
// -> Just('Bob Marley')

findFullName(3);
// -> Nothing
```

## Monad protocol

The actual definition is TBD, but surely it will just be a subset or entirety of [Fantasy Land Monad](https://github.com/fantasyland/fantasy-land#monad) (thus every such monad will be automatically supported).

## Desugaring

TBD

Should basically turns it into a chain of `.chain(x => { ... })` calls.

```js
chain function (n, f) {
  const m = f(n);
  const o = from S.Just(f(m));
  from S.Nothing;
  const p = from S.Just(f(o));
  return f(p);
}

↓↓↓↓↓↓↓↓

// This is very naive but correct.
function (n, f) {
  const m = f(n);
  return S.Just(f(m))
    .chain(_fm => {
      const o = _fm;
      return S.Nothing
        .chain(() => {
          return S.Just(f(o))
            .chain(_fo => {
              const p = _fo;
              return S.Maybe.of(p);
            })
        });
    });
}
```

## Motivating examples

### Custom `Promise`

Result of an async function is always an instance of builtin `Promise`, that's a shame.

```js
import Promise from 'bluebird';

const doStuff = async () {
  await Promise.delay(100);
};

doStuff() instanceof Promise
// -> false

doStuff().delay // bluebird-specific API
// -> undefined
```

Chain function, on the other hand, preserves the type that is being chained (assuming it's a proper [Monad protocol](#monad-protocol) implementation).

```js
import Promise from 'bluebird';

const doStuff = chain () {
  from Promise.delay(100);
};

doStuff() instanceof Promise
// -> true

doStuff().delay // bluebird-specific API
// -> function ...
```

### Custom Monads

TBD

[Sanctuary](https://github.com/sanctuary-js), [Folktale](http://folktale.origamitower.com/), [Fluture](https://github.com/fluture-js), and [Fantasy-Land](https://github.com/fantasyland).
* State: [fantasyland/fantasy-states](https://github.com/fantasyland/fantasy-states)
* Tuple: [fantasyland/fantasy-tuples](https://github.com/fantasyland/fantasy-tuples)
* Reader: [fantasyland/fantasy-readers](https://github.com/fantasyland/fantasy-readers)
* IO: [fantasyland/fantasy-io](https://github.com/fantasyland/fantasy-io)
* Identity: [fantasyland/fantasy-identities](https://github.com/fantasyland/fantasy-identities)
