+++
title = "Optional Chaining: Best Friend or Worst Enemy?"

[taxonomies]
tags = ["Javascript", "Opinion", "Programming"]
+++

Integrated into ES2020, present since 2018 in Babel, and since version 3.7 of TypeScript, optional chaining is an operator widely used today. Although the usefulness of such an operator is undeniable, let's explore together the biases and bad practices that can emerge from its usage.

<!-- more -->

## A reminder about the optional chaining operator

The optional chaining operator (`?.`) is an operator that allows accessing properties in a chain when the existence of these properties is not guaranteed. It only applies to the property where it is placed (the operand to the left of the `?.` symbol). Thus, you may need to use it multiple times in a property chain to secure access to a value. In code form, `a?.b` could be translated as `(a === null || a === undefined) ? undefined : a.b`.

```ts
const foo = {
  bar: {
    baz: "Defined",
  },
  x: {},
};

foo.bar.baz; // "Defined"
foo.bar.waldo; // undefined
foo.qux.quux; // TypeError
foo.qux?.quux; // undefined
foo.qux?.quux.quuz; //  undefined
foo.qux?.quux?.quuz; // undefined
foo.x.y; // undefined
foo.x.y.z; // TypeError
foo.x.y?.z; // undefined
foo.x?.y.z; // TypeError (optional chaining on 'x' won't prevent error on 'y')
```

This operator can be used for objects, arrays, and functions. And for the record, the systematic use of the dot in the syntax, even in the case of arrays and functions, is related to a problem with the JavaScript parser, which could confuse optional chaining with a ternary (example with the expression `obj?[expr].filter(fun):0`).

```ts
obj?.prop;
obj?.[expr];
arr?.[index];
func?.(args);
```

## Excessive optional chaining can harm your application's health

When using optional chaining, it's to manage optional properties. The use of the right words is essential because we are talking about properties whose absence makes sense in the application. The fact that the property is not defined is not only a possible case but also a planned case that makes sense! Stated as such, it may seem like stating the obvious. However, in the code of many applications, optional properties do not fit this definition and reflect uncertainty about the data.

Starting from this observation, optional chaining primarily represents the path of least resistance. Indeed, we decide not to resolve uncertainty by processing the information if it exists and ignoring the processing if the information is absent. As much as adding `if` statements every three lines seems absurd, the use of optional chaining (which has the same role) is strangely more acceptable in our code. Knowing that uncertainty and unpredictability within software are metrics that characterize legacy code, we try to limit their scope as much as possible.

But let's stop criticizing developers who use optional chaining and be more constructive: what can we do to avoid turning our code into a Schrödinger's experiment?

## What are the alternatives?

### Fail fast

Sometimes optional chaining is used for defensive coding. This increases code resilience in the face of data. However, defensive coding does not necessarily have a place when you want to make data reliable. One of the best ways to contain erroneous and/or incomplete data is "fail fast." By rejecting invalid data as quickly as possible, we ensure that the rest of the application will handle reliable data and produce fewer bugs.

### TypeScript to the rescue

The worst-case scenario you can encounter is "poor" typing (i.e., not precise enough to represent data). In this case, I refer you to my article on [discriminated unions in TypeScript](./ts-discriminated-union). The article explains how to better type and avoid optional properties by replacing a "one-size-fits-all" interface with a union of more precise interfaces. For example, you can easily ensure that the current user has a `username` if you know that this user is logged in and not a visitor.

### Limit the scope of uncertainty

Among the effective ways to combat uncertainty, you can also reduce the space affected by this uncertainty. It is much more acceptable and manageable over time to confine uncertainty to a few functions or even a few classes. If you expose an interface with optional properties, try to quickly converge toward a stricter and more precise typing.

Typically, for configuring a library (public interface with optional properties), you can either:

- Replace optional properties with mandatory properties with default values.
- Define a private interface without optional properties, which will be used internally in the library code instead of the public interface.

```ts
export interface PublicLibConfig {
  /**
   * If not provided, animations are disabled
   */
  animationStyle?: "ease-in" | "ease-out";
}

type Animation =
  | { status: "disabled" }
  | { status: "enabled"; animationStyle: "ease-in" | "ease-out" };
interface PrivateLibConfig {
  animation: Animation;
}
```

{% advice(type="tip", name="ProTip") %}

Consider combining optional chaining with the nullish coalescing operator to avoid initializations to `undefined`.

```ts
// will never be undefined
const animation = animationStyle?.toUpperCase() ?? "EASE-IN";
```

{% end %}

## Conclusion

In some cases, the use of optional chaining hides a deeper problem with the manipulated data. Keep your critical thinking and take a step back from this operator. Is it relevant to use it (such as configuring a library that offers many options, e.g., [Vite configuration](https://github.com/vitejs/vite/blob/c78e4099e502876a2ab23fd8163455d8172ff5b7/packages/vite/src/node/config.ts#L103)), or is it a palliative solution to another problem (imprecise typing, uncertainty about the available data, lack of default values)?

To further reflect on the relevance of good typing, I invite you to read this article: [Making Invalid State Unrepresentable](https://hugotunius.se/2020/05/16/making-invalid-state-unrepresentable.html).

To delve into "fail fast" and the scope of uncertainty, I recommend the [impure/pure/impure sandwich method](https://blog.ploeh.dk/2020/03/02/impureim-sandwich/).
