# `ts-reset`

Whenever you add TypeScript to a project, you're opting into **tens of thousands of lines of declaration files**. These files help describe JavaScript itself - `lib.dom.d.ts` describes the DOM, and `lib.d.ts` describes JavaScript itself.

These typings are not perfect.

- `.json` (in `fetch`) and `JSON.parse` both return `any`
- `.filter(Boolean)` doesn't behave how you expect
- `array.includes` often breaks on readonly arrays

`ts-reset` smooths over these hard edges, just like a CSS reset does in the browser. You'll be able to:

- Prevent `any` entering your codebase
- Make methods like `.filter` and `.includes` smarter

## Example

```ts
// Import in a single file, then across your whole project...
import "@total-typescript/ts-reset";

// .filter just got smarter!
const filteredArray = [1, 2, undefined].filter(Boolean); // number[]

// Get rid of the any's in JSON.parse and fetch
const result = JSON.parse("{}"); // unknown

fetch("/")
  .then((res) => res.json())
  .then((json) => {
    console.log(json); // unknown
  });
```

## Get Started

1. Install: `npm i @total-typescript/ts-reset`

2. Import **once** into any `.ts` or `.tsx` file:

```ts
import "@total-typescript/ts-reset";
```

3. Enjoy improved typings across your _entire_ project.

### Installing only certain rules

By importing from `@total-typescript/ts-reset`, you're bundling _all_ the recommended rules.

To only import the rules you want, you can import like so:

```ts
// Makes JSON.parse return unknown
import "@total-typescript/ts-reset/json-parse";

// Makes await fetch().then(res => res.json()) return unknown
import "@total-typescript/ts-reset/fetch";
```

Below is a full list of all the rules available.

## Caveats

### Use `ts-reset` in applications, not libraries

`ts-reset` is designed to be used in application code, not library code. Each rule you include will make changes to the global scope. That means that, simply by importing your library, your user will be unknowingly opting in to `ts-reset`.

## Rules

### Make `JSON.parse` return `unknown`

```ts
import "@total-typescript/ts-reset/json-parse";
```

`JSON.parse` returning `any` can cause nasty, subtle bugs. Frankly, any `any`'s can cause bugs because they disable typechecking on the values they describe.

```ts
// BEFORE
const result = JSON.parse("{}"); // any
```

By changing the result of `JSON.parse` to `unknown`, we're now forced to either validate the `unknown` to ensure it's the correct type (perhaps using [`zod`](https://github.com/colinhacks/zod)), or cast it with `as`.

```ts
// AFTER
import "@total-typescript/ts-reset/json-parse";

const result = JSON.parse("{}"); // unknown
```

### Make `.json()` return `unknown`

```ts
import "@total-typescript/ts-reset/fetch";
```

Just like `JSON.parse`, `.json()` returning `any` introduces unwanted `any`'s into your application code.

```ts
// BEFORE
fetch("/")
  .then((res) => res.json())
  .then((json) => {
    console.log(json); // any
  });
```

By forcing `res.json` to return `unknown`, we're encouraged to distrust its results, making us more likely to validate the results of `fetch`.

```ts
// AFTER
import "@total-typescript/ts-reset/fetch";

fetch("/")
  .then((res) => res.json())
  .then((json) => {
    console.log(json); // unknown
  });
```

### Make `.filter(Boolean)` filter out falsy values

```ts
import "@total-typescript/ts-reset/filter-boolean";
```

The default behaviour of `.filter` can feel pretty frustrating. Given the code below:

```ts
// BEFORE
const filteredArray = [1, 2, undefined].filter(Boolean); // (number | undefined)[]
```

It feels natural that TypeScript should understand that you've filtered out the `undefined` from `filteredArray`. You can make this work, but you need to mark it as a type predicate:

```ts
const filteredArray = [1, 2, undefined].filter((item): item is number => {
  return !!item;
}); // number[]
```

Using `.filter(Boolean)` is a really common shorthand for this. So, this rule makes it so `.filter(Boolean)` acts like a type predicate on the array passed in, removing any falsy values from the array member.

```ts
// AFTER
import "@total-typescript/ts-reset/filter-boolean";

const filteredArray = [1, 2, undefined].filter(Boolean); // number[]
```

### Make `.includes` on `as const` arrays less strict

```ts
import "@total-typescript/ts-reset/array-includes";
```

This rule improves on TypeScript's default `.includes` behaviour. Without this rule enabled, the argument passed to `.includes` MUST be a member of the array it's being tested against.

```ts
// BEFORE
const users = ["matt", "sofia", "waqas"] as const;

// Argument of type '"bryan"' is not assignable to
// parameter of type '"matt" | "sofia" | "waqas"'.
users.includes("bryan");
```

This can often feel extremely awkward. But with the rule enabled, `.includes` now takes a widened version of the literals in the `const` array.

```ts
// AFTER
import "@total-typescript/ts-reset/array-includes";

const users = ["matt", "sofia", "waqas"] as const;

// .includes now takes a string as the first parameter
users.includes("bryan");
```

This means you can test non-members of the array safely.

It also makes `.includes` a type predicate, meaning you can use it to narrow wider types to a set enum:

```ts
import "@total-typescript/ts-reset/array-includes";

const users = ["matt", "sofia", "waqas"] as const;

const isUser = (input: string) => {
  if (users.includes(input)) {
    // input is narrowed to "matt" | "sofia" | "waqas"
    console.log(input);
  }
};
```

### Make `Set.has()` less strict

```ts
import "@total-typescript/ts-reset/set-has";
```

Similar to `.includes`, `Set.has()` doesn't let you pass members that don't exist in the set:

```ts
// BEFORE
const userSet = new Set(["matt", "sofia", "waqas"] as const);

// Argument of type '"bryan"' is not assignable to
// parameter of type '"matt" | "sofia" | "waqas"'.
userSet.has("bryan");
```

With the rule enabled, `Set` is much smarter:

```ts
// AFTER
import "@total-typescript/ts-reset/set-has";

const userSet = new Set(["matt", "sofia", "waqas"] as const);

// .has now takes a string as the argument!
userSet.has("bryan");
```

### Removing `any[]` from `Array.isArray()`

```ts
import "@total-typescript/ts-reset/is-array";
```

When you're using `Array.isArray`, you can introduce subtle `any`'s into your app's code.

```ts
// BEFORE

const validate = (input: unknown) => {
  if (Array.isArray(input)) {
    console.log(input); // any[]
  }
};
```

With `is-array` enabled, this check will now mark the value as `unknown[]`:

```ts
// AFTER
import "@total-typescript/ts-reset/is-array";

const validate = (input: unknown) => {
  if (Array.isArray(input)) {
    console.log(input); // unknown[]
  }
};
```
