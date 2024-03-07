+++
title = "Exploiter les Discriminated Union en Typescript"

[taxonomies]
tags = ["Typescript", "Dev"]
+++ 

## De quoi parle-t-on ?

En Typescript, il est possible de définir des types de plusieurs manières : interface, classe, enum, mot-clé `type`, `as const`, etc. Dans cet article, nous allons nous concentrer sur les types construits à partir d'une union disjointe et les avantages d'une telle pratique. L'union en Typescript se fait via le symbole `|` (ex : `type Union = A | B | C`). Le terme disjoint n'est pas anodin car contrairement au polymorphisme, les types que l'on va utiliser peuvent ne rien avoir en commun.

<!-- more -->

## Mise en situation

Prenons un exemple très simple, la représentation des utilisateurs dans une application. Ces utilisateurs peuvent être des invités (guest), des clients (customer) ou des administrateurs (admin). Les utilisateurs connectés ont un identifiant et les administrateurs ont des permissions spécifiques relatives à leur domaine d'administration. Construisons donc une interface `User` pour gérer nos utilisateurs.

```ts
interface User {
  userType: "Guest" | "Customer" | "Admin";
  login?: string;
  accessRights?: string[]; // could be more specific
}
```

Certaines propriétés sont optionnelles car elles n'existent pas pour l'utilisateur invité (et qu'on utilise le mode `strict` du compilateur Typescript).

Cette interface nous permet de créer des utilisateurs valides :

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

Mais nous pouvons également créer des utilisateurs qui correspondent à des cas non souhaités :

```ts
const customerWithoutLogin: User = {
  userType: "Customer",
};
const guestWithAccessRights: User = {
  userType: "Guest",
  accessRights: ["fs"],
};
```

Et on ne peut pas garantir si par exemple l'identifiant existe ou pas :

```ts
johnDoe.login.toUpperCase(); // Object is possibly 'undefined'.
guest.login.toUpperCase(); // Object is possibly 'undefined'.
customerWithoutLogin.login.toUpperCase(); // Object is possibly 'undefined'.
guestWithAccessRights.login.toUpperCase(); // Object is possibly 'undefined'.
```

En général on finit par utiliser une condition ou du chainage optionnel, ce qui renforce l'incertitude sur le fonctionnement au runtime :

```ts
// Which one is really executed?
johnDoe.login?.toUpperCase();
guest.login?.toUpperCase();
customerWithoutLogin.login?.toUpperCase();
guestWithAccessRights.login?.toUpperCase();
```

Pour réduire cette incertitude, il ne nous reste plus qu'à écrire des tests unitaires, faire du monitoring, du debug et lever des exceptions. Heureusement, nous pouvons éviter tout ça avec un meilleur typage.

## Discriminated Union

> Explicit is better than implicit

Nous avons trois types d'utilisateurs distincts et les regrouper dans une même interface/classe est une erreur commune. Et c'est normal, on nous répète souvent DRY (Don't Repeat Yourself) et on a envie de factoriser les utilisateurs dans une même classe ou dans une même interface pour y appliquer des méthodes communes.

Et si on faisait le contraire ? Trois types d'utilisateurs, donc trois interfaces.

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

Ensuite, il nous suffit de définir un type qui correspond à l'union des trois interfaces distinctes :

```ts
type User = GuestUser | CustomerUser | AdminUser;
```

Cette fois, la syntaxe nous permet toujours de créer des utilisateurs valides :

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

Mais interdit la création d'utilisateurs qui n'ont pas de sens :

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

L'accès aux propriétés est également bien plus prédictible :

```ts
johnDoe.login.toUpperCase(); // OK
customer.login.toUpperCase(); // OK
guest.login.toUpperCase(); // Property 'login' does not exist on type 'GuestUser'.
```

L'inférence de type fait également des merveilles :

```ts
// login is defined because GuestUser is excluded (Type guard)
const displayLogin = (user: User) =>
  user.userType === "Guest" ? "Guest" : user.login;
```

{% advice(type="tip", name="ProTip") %}

Si l'inférence ne fonctionne pas, pensez à définir un champ qui va aider Typescript à déterminer le bon type (`userType` dans notre exemple). Vous pouvez aussi identifier le type manuellement avec le mot-clé `is`.

```ts
const isAdmin = (user: User): user is AdminUser =>
  (user as AdminUser).accessRights !== undefined;

const users: User[] = [johnDoe, customer, guest];

users.filter(isAdmin).forEach((admin) => console.log(admin.accessRights));
```

{% end %}

## Conclusion

Vous pouvez maintenant être plus précis sur le typage des données. Rien de révolutionnaire ici mais rappelez-vous que le typage est un bon moyen d'augmenter la prédictibilité de votre code.

## Pour aller plus loin

Je vous invite à aller voir mon article sur le [pattern matching en JS](./js-pattern-matching) qui complète assez bien les unions disjointes que l'on vient de voir. En combinant les deux, vous pouvez rapidement faire un pattern stratégie.

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
