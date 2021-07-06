# Iterator Helpers
A proposal for several interfaces that will help with general usage and
consumption of iterators in ECMAScript.

## Status
Authors: Gus Caplan, Michael Ficcara, Adam Vandolder, Jason Orendorff

Champions: Michael Ficcara, Yulia Startsev

This proposal is at Stage 2 of [The TC39 Process](https://tc39.es/process-document/).

## Motivation

Iterators are a useful way to represent large or possible infinite enumerable data sets. However,
they lack helpers which make them as easy to use as Arrays and other finite data structures, which
results in certain problems that could be better represented by iterators being expressed in Arrays,
or using libraries to introduce the necessary helpers. Many [libraries and languages](#prior-art) already provide these interfaces.

## Proposal

The proposal introduces a collection of new methods on the Iterator prototype, to allow general
ussage and consumption of iterators. For specifics on the implemented methods, please refer to the
specification.

See [DETAILS.md](./DETAILS.md) for details on semantics decisions.

See this proposal rendered [here](https://tc39.es/proposal-iterator-helpers)

## Added Methods

For Iterators and AsyncIterators we add the following methods:

### `.map(mapperFn)`

`map` takes a function as an argument. It allows users to apply a funtion to every element returned from an iterator.

Returns an iterator of the values with the map function applied.

#### Example

```JavaScript
function* naturals() {
  let i = 0;
  while (true) {
    yield i;
    i += 1;
  }
}

const result = naturals()
  .map(value => {
    return value * value;
  });
result.next(); //  {value: 0, done: false};
result.next(); //  {value: 1, done: false};
result.next(); //  {value: 4, done: false};
```

### `.filter(filtererFn)`

`filter` takes a function as an argument. It allows users to skip values from an iterator which do not pass a filter function.

Returns an iterator of values from the original iterator that pass the filter.

#### Example

```JavaScript
function* naturals() {
  let i = 0;
  while (true) {
    yield i;
    i += 1;
  }
}

const result = naturals()
  .filter(value => {
    return value % 2 == 0;
  });
result.next(); //  {value: 0, done: false};
result.next(); //  {value: 2, done: false};
result.next(); //  {value: 4, done: false};
```

### `.take(limit)`

`take` takes an integer as an argument. It takes a group of the specified size and returns an
iterator with only those elements.

Returns an iterator with items from the original iterator from 0 until the limit.

#### Example

```JavaScript
function* naturals() {
  let i = 0;
  while (true) {
    yield i;
    i += 1;
  }
}

const result = naturals()
  .take(3);
result.next(); //  {value: 0, done: false};
result.next(); //  {value: 1, done: false};
result.next(); //  {value: 2, done: false};
result.next(); //  {value: undefined, done: true};
```

### `.drop(limit)`

`drop` takes an integer as an argument. It skips a set of value in the iterator determined by the
`limit` argument.

Returns an iterator of items after the limit.

#### Example

```JavaScript
function* naturals() {
  let i = 0;
  while (true) {
    yield i;
    i += 1;
  }
}

const result = naturals()
  .drop(3);
result.next(); //  {value: 3, done: false};
result.next(); //  {value: 4, done: false};
result.next(); //  {value: 5, done: false};
```

### `.asIndexedPairs()`

`.asInexedPairs` takes no arguments. It returns an iterator where each value is paired with its
index.

Returns an iterator of pairs.

#### Example

```JavaScript
const abc = ["a", "b", "c"].values();

const result = abc
  .asIndexedPairs();
result.next(); //  {value: [0, "a"], done: false};
result.next(); //  {value: [1, "b"], done: false};
result.next(); //  {value: [2, "c"], done: false};
```

### `.flatMap(mapperFn)`

`.flatMap` takes a mapping function as an argument. It returns an iterator where each value, if it
is an iterator, is flattened into a single iterator.

Returns an iterator of flat values.

#### Example

```JavaScript
const sunny = ["It's Sunny in", "", "California"].values();

const result = sunny
  .flatMap(value => value.split(" ").values());
result.next(); //  "it's";
result.next(); //  "Sunny";
result.next(); //  {value: "It's", done: false};
result.next(); //  {value: "Sunny", done: false};
result.next(); //  {value: "in", done: false};
result.next(); //  {value: "", done: false};
result.next(); //  {value: "California", done: false};
result.next(); //  {value: undefined, done: true};
```

### `.reduce(reducer [, initialValue ])`

`reduce` takes a function and an optional initial value as an argument. It allows users to apply a funtion to every element returned from an iterator.

Returns a value (in the example, a number) of the type returned to the reducer function.

#### Example

```JavaScript
function* naturals() {
  let i = 0;
  while (true) {
    yield i;
    i += 1;
  }
}

const result = naturals()
  .take(5)
  .reduce((value, sum) => {
    return sum + value;
  }, 3);

result // 13
```

### `.toArray()`

When you have a non-infinite iterator which you wish to transform into an array, you can do so with
the builtin `toArray` method.

Returns an Array containing the values from the iterator.

#### Example

```JavaScript
function* naturals() {
  let i = 0;
  while (true) {
    yield i;
    i += 1;
  }
}

const result = naturals()
  .take(5)
  .toArray();

result // [0, 1, 2, 3, 4]
```

### `.forEach(fn)`

For using side effects with an iterator, you can use the `.forEach` builtin method, which takes as
an argument a function.

Returns undefined.

#### Example

```JavaScript
const log = [];
const fn = (value) => log.push(value);
const iter = [1, 2, 3].values();

iter.forEach(fn);
console.log(log.join(", ")) // "1, 2, 3"
```

### `some(fn)`

To check if any value in the iterator matches a paramter, `.some` can be used. It takes as an
argument a function which returns true or false.


Returns a boolean which is true if any element returned true when `fn` was called on it. The
iterator is consumed when some is called.

#### Example

```JavaScript
const iter = [1, 2, 3].values();

iter.some(v => v > 1); // true
iter.some(v => v  == 1) // false, iterator is already consumed.

function* naturals() {
  let i = 0;
  while (true) {
    yield i;
    i += 1;
  }
}

naturals().take(4).some(v => v > 1); // true
naturals().take(4).some(v => v == 1); // true, acting on a new iterator
```

### `.every(fn)`

`.every` takes a function which returns a boolean as an argument. It is used to check if every
value generated by the iterator passes the test function.

Returns a boolean.

```JavaScript
const iter = [1, 2, 3].values();

iter.every(v => v >= 1); // true
iter.every(v => v >= 1) // false, iterator is already consumed.

function* naturals() {
  let i = 0;
  while (true) {
    yield i;
    i += 1;
  }
}

naturals().take(4).every(v => v > 1); // false, first value is 1
naturals().take(4).every(v => v >= 1); // true, acting on a new iterator
```

### `.find(fn)`

`.find` takes a function as an argument. It is used to find the first element in an iterator that matches.

Can be used without `take` on infinite iterators.

Returns the found element.

```JavaScript
function* naturals() {
  let i = 0;
  while (true) {
    yield i;
    i += 1;
  }
}

naturals().find(v => v > 1); // 2
```

## More Example Usage

### Lazy Iteration over sets

Iterating over a set of URLs, asynchronously fetching each, and returning an array of their
JSON Output.

```js
const responses = await AsyncIterator.from(urls)
  .map(async (url) => {
    const response = await fetch(url);
    return response.json();
  })
  .toArray();
```

Example of iterating over a potentially infinite iterator and transforming it to an array in groups
of 5.

```js
class ObligatoryCryptocurrencyReference extends Component {
  componentWillMount() {
    const items = ticker() // returns async iterator
      .map((c) => createElement('h2', null, `${c.name}: ${c.price}`))
      .take(5) // only consume 5 items of a potentially infinite iterator
      .toArray() // greedily transform async iterator into array
      .then((data) => this.setState({ data }));
  }

  render() {
    return createElement('div', null, this.state.data);
  }
}
```

#### Extending Iterator Prototype

With this proposal, it will be easier to extend the IteatorPrototype for a custom class. See the
below example for the previous implementation compared to the new one.

```js
const MyIteratorPrototype = {
  next() {},
  throw() {},
  return() {},

  // but we don't properly implement %IteratorPrototype%!!!
};

// Previously...
// Object.setPrototypeOf(MyIteratorPrototype,
//   Object.getPrototypeOf(Object.getPrototypeOf([][Symbol.iterator]())));

Object.setPrototypeOf(MyIteratorPrototype, Iterator.prototype);
```
## Implementations

Implementation tracking of Iterator Helpers

- Browsers:
  * [] V8
  * [] [SpiderMonkey](https://bugzilla.mozilla.org/show_bug.cgi?id=1568906) (feature-flagged on Nightly
      only)
  * [] JavaScriptCore

## Q & A

### Why not use Array.from + Array.prototype methods?

All of these methods (except for reduce and toArray) are **lazy**. They will
only consume the iterator when they need the next item from it. Especially
for iterators that never end, this is key. Without generic support for
any form of iterator, different iterators have to be handled differently.

## Prior Art

- https://www.npmjs.com/package/itertools
- https://www.npmjs.com/package/lodash
- https://docs.python.org/3/library/itertools.html
- https://github.com/more-itertools/more-itertools
- https://docs.rs/itertools/
- https://doc.rust-lang.org/std/iter/trait.Iterator.html
- https://www.boost.org/doc/libs/1_66_0/libs/iterator/doc/index.html
- https://docs.microsoft.com/en-us/dotnet/api/system.linq.enumerable
- https://github.com/ReactiveX/IxJS
- https://www.npmjs.com/package/ballvalve

| Method                      | Rust | Python | npm Itertools | C# |
| --------------------------- | ---- | ------ | --------------| -- |
| all                         | ☑    | ☑      | ☑             | ☑  |
| any                         | ☑    | ☑      | ☑             | ☑  |
| chain                       | ☑    | ☑      | ☑             | ☑  |
| collect                     | ☑    | ☐      | ☐             | ☐  |
| count                       | ☑    | ☑      | ☑             | ☑  |
| cycle                       | ☑    | ☑      | ☑             | ☐  |
| enumerate                   | ☑    | ☑      | ☑             | ☐  |
| filter                      | ☑    | ☑      | ☑             | ☑  |
| filterMap                   | ☑    | ☐      | ☐             | ☐  |
| find                        | ☑    | ☐      | ☑             | ☑  |
| findMap                     | ☑    | ☐      | ☐             | ☐  |
| flatMap                     | ☑    | ☐      | ☑             | ☑  |
| flatten                     | ☑    | ☐      | ☐             | ☐  |
| forEach                     | ☑    | ☐      | ☐             | ☐  |
| last                        | ☑    | ☐      | ☐             | ☑  |
| map                         | ☑    | ☑      | ☑             | ☑  |
| max                         | ☑    | ☑      | ☑             | ☑  |
| min                         | ☑    | ☑      | ☑             | ☑  |
| nth                         | ☑    | ☐      | ☐             | ☑  |
| partition                   | ☑    | ☐      | ☐             | ☑  |
| peekable                    | ☑    | ☐      | ☐             | ☐  |
| position                    | ☑    | ☐      | ☐             | ☐  |
| product                     | ☑    | ☑      | ☐             | ☐  |
| reverse                     | ☑    | ☐      | ☐             | ☑  |
| scan                        | ☑    | ☐      | ☐             | ☐  |
| skip                        | ☑    | ☐      | ☐             | ☑  |
| skipWhile                   | ☑    | ☑      | ☐             | ☑  |
| stepBy                      | ☑    | ☐      | ☐             | ☐  |
| sum                         | ☑    | ☑      | ☑             | ☑  |
| take                        | ☑    | ☐      | ☑             | ☑  |
| takeWhile                   | ☑    | ☑      | ☐             | ☑  |
| unzip                       | ☑    | ☐      | ☐             | ☐  |
| zip                         | ☑    | ☑      | ☑             | ☑  |
| compress                    | ☐    | ☑      | ☑             | ☐  |
| permutations                | ☐    | ☑      | ☑             | ☐  |
| repeat                      | ☑    | ☑      | ☑             | ☑  |
| slice                       | ☐    | ☑      | ☑             | ☐  |
| starmap                     | ☐    | ☑      | ☐             | ☐  |
| tee                         | ☐    | ☑      | ☐             | ☐  |
| compact                     | ☐    | ☐      | ☑             | ☐  |
| contains                    | ☐    | ☑      | ☑             | ☑  |
| range                       | ☑    | ☑      | ☑             | ☑  |
| reduce                      | ☑    | ☑      | ☑             | ☑  |
| sorted                      | ☐    | ☑      | ☑             | ☐  |
| unique                      | ☐    | ☐      | ☑             | ☑  |
| average                     | ☐    | ☐      | ☐             | ☑  |
| empty                       | ☑    | ☐      | ☐             | ☑  |
| except                      | ☐    | ☐      | ☐             | ☑  |
| intersect                   | ☐    | ☐      | ☐             | ☑  |
| prepend                     | ☐    | ☐      | ☐             | ☑  |
| append                      | ☐    | ☐      | ☐             | ☑  |

Note: The method names are combined, such as `toArray` and `collect`.
