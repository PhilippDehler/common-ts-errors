# How to workaround the max recursion depth in TypeScript

Have you already seen this error message?

```
Type instantiation is excessively deep and possibly infinite.
```

If you're deeply involved with TypeScript 🧙‍♂️, chances are you've come across this error, causing frustration and leaving you unsure of how to resolve it. Fortunately, I have multiple solutions for you. However, bear in in mind that while these solutions might assist you in gaining a deeper understanding of the problem, they can slow down the compiler.
Let's delve into some magic tricks.

## The Issue

TypeScript's type system imposes a strict maximum recursion depth of 1000, an unchangeable hard limit. When dealing with intricate types, such as attempting complex tasks like solving Advent of Code challenges using only types or developing TypeScript libraries that may lead to intricate type structures, you might encounter this limit.

Consider a `Replace` type that substitutes all values in an array with a given value:

```ts
type Replace<T extends any[], ReplaceValue> = T extends [any, ...infer Tail]
  ? [ReplaceValue, ...Replace<Tail, ReplaceValue>]
  : [];
```

Calling this type with an array of 49 elements triggers the error. When dealing with recursive types like this, what options do we have?

The first obvious solution involves transforming the type to be tail-recursive:

```ts
//prettier-ignore
type Replace<T extends any[], ReplaceValue, Agg extends ReplaceValue[] = []> = 
T extends [any, ...infer Tail]
  ? Replace<Tail, [ReplaceValue, ...Agg]>
  : Agg;
```

Here, we introduce an aggregator to collect the recursion results, following a common pattern in functional programming. Yet, this modification alone doesn't resolve the issue; the recursion limit is now reached after 999 elements. If your type still isn't functioning, it's time to either consider your life choices or try to optimize other types in your code. If both doesn't work you could apply some magic.

There are two possible ways of work arround the recursion limit.

# The first 🧙‍♂️-trick

The initial workaround involves unrolling the loop or recursion. Similar to optimizing loops in languages like C/C++, we can apply a similar technique in TypeScript:

```ts
//prettier-ignore
type Replace<T extends any[],ReplaceValue,Agg extends any[] = []> = 
T extends [any, any, ...infer Tail]
  ? Replace<Tail, [ReplaceValue, ReplaceValue, ...Agg]>
  : T["length"] extends 1 ? [ReplaceValue, ...Agg] : Agg;
```

By unrolling the recursion like this we can increase the max recursion depth by a factor of 2. So we can now handle arrays with 1998 elements. You can also do 11 or steps at a time then you will run into new error message. You can work around this aswell but that's another topic.

```ts
Type produces a tuple type that is too large to represent.(2799)
```

Note: This also works with strings.

```ts
//prettier-ignore
type Replace<T extends string, ReplaceValue, Agg extends string = ""> =
  T extends `${infer _}${infer _}${infer Tail}`
    ? Replace<Tail, `${ReplaceValue}${ReplaceValue}${Agg}`>
    : T["length"] extends 1 ? `${ReplaceValue}${Agg}` : Agg;
```

# The second 🧙‍♂️-trick

Internally, TypeScript keeps track of the recursion limit using a counter. Resetting this counter can be achieved by utilizing an intersection type, effectively resetting the counter to zero. This workaround can be used multiple times, but keep in mind that it can slow down the compiler:

```ts
//prettier-ignore
type Replace<T extends any[], ReplaceValue, Agg extends any[] = [], RecursionCount extends any[] = []> = RecursionCount["length"] extends 500
  ? Replace<T, ReplaceValue, Agg, []> & {} // reset the counter
  : T extends [any, ...infer Tail]
    ? Replace<Tail, ReplaceValue, [ReplaceValue, ...Agg], [...RecursionCount, unknown]>
    : Agg;
```

While not as swift as the first method, this approach extends the limit to approximately ~3000 elements before encountering the "Type Instantiation Depth (TS2589)" error. Further information about this method can be found [here](https://herringtondarkholme.github.io/2023/04/30/typescript-magic/).

# Conclusion

These two workarounds offer methods to tackle TypeScript's maximum recursion depth limitation. Whether by unrolling the recursion or resetting the counter, both approaches can aid in understanding the problem better. If you have alternative ideas on surpassing this limitation, I'm eager to hear about them. Feel free to share your thoughts and write [me on Twitter](https://twitter.com/KingPhipps).
