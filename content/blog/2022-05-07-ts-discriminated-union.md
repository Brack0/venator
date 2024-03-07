+++
title = "Leveraging Discriminated Unions in TypeScript"

[taxonomies]
tags = ["Typescript", "Dev"]
+++ 

## What are we talking about?

In TypeScript, it is possible to define types in several ways: interface, class, enum, the `type` keyword, `as const`, and more. In this article, we will focus on types constructed from a discriminated union and the advantages of such a practice. In TypeScript, unions are created using the `|` symbol (e.g., `type Union = A | B | C`). The term "disjoint" is not accidental because, unlike polymorphism, the types we will use may have nothing in common.

<!-- more -->

## Setting the stage

Let's take a very simple example: representing users in an application. These users can be guests, customers, or administrators. Logged-in users have an identifier, and administrators have specific permissions related to their administrative domain. Let's create a `User` interface to manage our users.

```ts
interface User {
  userType: "Guest" | "Customer" | "Admin";
  login?: string;
  accessRights?: string[]; // could be more specific
}
```

Some properties are optional because they do not exist for guest users (and we use TypeScript's `strict` mode).

This interface allows us to create valid users:

```ts
const johnDoe: User = {
  userType: "Admin",
  login: "JohnDoe",
  accessRights: ["database", "monitoring"],
};
const guest: User = {
  userType: "Guest",
};
```

But we can also create users that correspond to undesired cases:

```ts
const customerWithoutLogin: User = {
  userType: "Customer",
};
const guestWithAccessRights: User = {
  userType: "Guest",
  accessRights: ["fs"],
};
```

And we cannot guarantee whether, for example, the identifier exists or not:

```ts
johnDoe.login.toUpperCase(); // Object is possibly 'undefined'.
guest.login.toUpperCase(); // Object is possibly 'undefined'.
customerWithoutLogin.login.toUpperCase(); // Object is possibly 'undefined'.
guestWithAccessRights.login.toUpperCase(); // Object is possibly 'undefined'.
```

In general, we end up using conditional or optional chaining, which reinforces uncertainty about runtime behavior:

```ts
// Which one is really executed?
johnDoe.login?.toUpperCase();
guest.login?.toUpperCase();
customerWithoutLogin.login?.toUpperCase();
guestWithAccessRights.login?.toUpperCase();
```

To reduce this uncertainty, the only option left is to write unit tests, perform monitoring, debugging, and throw exceptions. Fortunately, we can avoid all of this with better typing.

## Discriminated Union

> Explicit is better than implicit

We have three distinct user types, and grouping them into a single interface or class is a common mistake. It's normal because we often hear DRY (Don't Repeat Yourself), and we want to factorize users into a single class or interface to apply common methods.

But what if we did the opposite? Three user types, so three interfaces.

```ts
interface GuestUser {
  userType: "Guest";
}

interface CustomerUser {
  userType: "Customer";
  login: string;
}

interface AdminUser {
  userType: "Admin";
  login: string;
  accessRights: string[];
}
```

Then, we only need to define a type that corresponds to the union of the three distinct interfaces:

```ts
type User = GuestUser | CustomerUser | AdminUser;
```

This time, the syntax still allows us to create valid users:

```ts
const johnDoe: User = {
  userType: "Admin",
  login: "JohnDoe",
  accessRights: ["database", "monitoring"],
};
const customer: User = {
  userType: "Customer",
  login: "JaneDoe",
};
const guest: User = {
  userType: "Guest",
};
```

But it prevents the creation of users that do not make sense:

```ts
/**
 * Type '{ userType: "Customer"; }' is not assignable to type 'User'.
 * Property 'login' is missing in type '{ userType: "Customer"; }'
 * but required in type 'CustomerUser'.
 */
const customerWithoutLogin: User = {
  userType: "Customer",
};

/**
 * Type '{ userType: "Guest"; accessRights: string[]; }'is not assignable to type 'User'.
 * Object literal may only specify known properties,
 * and 'accessRights' does not exist in type 'GuestUser'.
 */
const guestWithAccessRights: User = {
  userType: "Guest",
  accessRights: ["fs"],
};
```

Access to properties is also much more predictable:

```ts
johnDoe.login.toUpperCase(); // OK
customer.login.toUpperCase(); // OK
guest.login.toUpperCase(); // Property 'login' does not exist on type 'GuestUser'.
```

Type inference also works wonders:

```ts
// login is defined because GuestUser is excluded (Type guard)
const displayLogin = (user: User) =>
  user.userType === "Guest" ? "Guest" : user.login;
```

{% advice(type="tip", name="ProTip") %}

If type inference does not work, consider defining a field that will help TypeScript determine the correct type (`userType` in our example). You can also manually identify the type with the `is` keyword.

```ts
const isAdmin = (user: User): user is AdminUser =>
  (user as AdminUser).accessRights !== undefined;

const users: User[] = [johnDoe, customer, guest];

users.filter(isAdmin).forEach((admin) => console.log(admin.accessRights));
```

{% end %}

## Conclusion

You can now be more precise in typing data. There's nothing revolutionary here, but remember that typing is a good way to increase the predictability of your code.

## Further reading

I invite you to check out my article on [pattern matching in JS](./js-pattern-matching), which complements the disjoint unions we've just seen. By combining the two, you can quickly implement a strategy pattern.

```ts
const redirectToHomePage = () => (location.href = "/");
const redirectToAccountPage = () => (location.href = "/account");
const redirectToAdminDashboardPage = () => (location.href = "/admin/dashboard");

export const redirectToUserPage = (user: User) =>
  ({
    Guest: () => redirectToHomePage(),
    Customer: () => redirectToAccountPage(),
    Admin: () => redirectToAdminDashboardPage(),
  }[user.userType]());
```
