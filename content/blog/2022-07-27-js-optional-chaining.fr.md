+++
title = "Optional chaining : meilleur ami ou pire ennemi ?"

[taxonomies]
tags = ["Javascript", "Opinion", "Programmation"]
+++

Intégré à ES2020, présent depuis 2018 dans Babel et depuis la version 3.7 de Typescript, l'optional chaining est un opérateur largement utilisé aujourd'hui. Même si l'intérêt d'un tel opérateur est indéniable, regardons ensemble les biais et les mauvaises pratiques qui peuvent émerger de son usage.

<!-- more -->

## Rappels sur l'opérateur de chaînage optionnel

L'opérateur de chaînage optionnel (`?.`) est un opérateur qui permet d'accéder aux propriétés en chaine alors que l'existence de ces propriétés n'est pas garantie. Il s'applique uniquement sur la propriété où il est apposé (opérande à gauche du symbole `?.`). Ainsi, on peut être amené à l'utiliser plusieurs fois dans une chaine de propriétés pour sécuriser l'accès à une valeur. Sous forme de code, on pourrait traduire `a?.b` par `(a === null || a === undefined) ? undefined : a.b`.

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

Cet opérateur est utilisable pour les objets, les tableaux et les fonctions. Et pour la petite histoire, l'usage systématique du point dans la syntaxe, même dans le cas des tableaux et des fonctions, est lié à une problématique du parser Javascript, qui pourrait confondre l'optional chaining avec un ternaire (exemple avec l'expression `obj?[expr].filter(fun):0`).

```ts
obj?.prop;
obj?.[expr];
arr?.[index];
func?.(args);
```

## L'abus de chaînage optionnel peut nuire à la santé de vos applications

Lorsque l'on utilise l'optional chaining, c'est pour gérer les propriétés optionnelles. L'usage des bons mots est important car on parle bien ici des propriétés dont leur absence a du sens dans l'application. Le fait que la propriété ne soit pas définie est non seulement un cas possible, mais également un cas prévu et qui a du sens ! Enoncé comme tel, on a l'impression d'enfoncer des portes ouvertes. Pourtant dans le code de nombreuses applications, les propriétés optionnelles ne correspondent pas à cette définition et traduisent une incertitude sur la donnée.

Si l'on part de ce constat, l'optional chaining représente avant tout la solution de facilité. En effet, on décide de ne pas résoudre l'incertitude en traitant l'information si elle existe et en ignorant le traitement si l'information est absente. Autant rajouter des `if` toutes les trois lignes nous paraît insensé, autant le recours à l'optional chaining (qui pourtant a le même rôle) est une solution étrangement plus acceptable dans notre code. Sachant que l'incertitude et l'imprédictibilité au sein du logiciel sont des métriques qui caractérisent un code legacy, on essaie autant que possible de limiter leur ampleur.

Mais arrêtons de taper sur les devs qui utilisent l'optional chaining et soyons plus constructif : que pouvons-nous faire pour éviter de transformer notre code en expérience de Schrödinger ?

## Quelles alternatives ?

### Fail fast

Il arrive que l'optional chaining soit utilisé pour du code défensif. Cela permet d'augmenter la résilience du code vis-à-vis de la donnée. Cependant le code défensif n'a pas forcément sa place quand on veut fiabiliser les données. Un des meilleurs moyens d'endiguer les données erronées et/ou incomplètes est le "fail fast". En refusant au plus vite les données invalides, on garantit que le reste de l'application manipulera des données fiables et produira donc moins de bugs.

### Typescript à la rescousse

Le pire scénario qui puisse vous arrivez est le typage "pauvre" (aka un typage pas assez précis pour représenter de la donnée). Dans ce cas, je vous renvoie vers mon article sur les [discriminated union en Typescript](./ts-discriminated-union). L'article montre comment mieux typer et éviter les propriétés optionnelles en remplaçant une interface "à tout faire" par une union d'interfaces plus précises. A titre d'exemple, vous pouvez facilement garantir que l'utilisateur courant a un `username` si vous savez que cet utilisateur est connecté et qu'il n'est pas un visiteur.

### Limiter le périmètre d'incertitude

Parmi les moyens efficaces pour combattre l'incertitude, vous pouvez également réduire l'espace concerné par cette incertitude. Il est nettement plus acceptable et gérable dans le temps de cantonner l'incertitude à quelques fonctions, voire quelques classes. Si vous exposez une interface avec des propriétés optionnelles, essayez de converger rapidement vers un typage plus strict et précis.

Typiquement pour la configuration d'une librairie (interface publique avec des propriétés optionnelles), vous pouvez soit :

- Remplacer les propriétés optionnelles par des propriétés obligatoires avec des valeurs par défaut.
- Définir une interface privée, sans propriétés optionnelles, qui sera utilisée dans code interne de la librairie à la place de l'interface publique.

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

Pensez à combiner l'optional chaining avec le nullish coalescing operator pour éviter les initialisations à undefined.

```ts
// will never be undefined
const animation = animationStyle?.toUpperCase() ?? "EASE-IN";
```

{% end %}

## Conclusion

Dans certains cas, l'usage de l'optional chaining masque un problème plus profond sur les données manipulées. Gardez votre esprit critique et prenez du recul face à cet opérateur. Est-ce pertinent de l'utiliser (comme la configuration d'une librairie qui propose beaucoup d'options, ex : [configuration Vite](https://github.com/vitejs/vite/blob/c78e4099e502876a2ab23fd8163455d8172ff5b7/packages/vite/src/node/config.ts#L103)) ou est-ce une solution palliative d'un autre problème (imprécision dans le typage, incertitude sur les données présentes, absence de valeurs par défaut) ?

Pour prolonger la réflexion sur la pertinence d'un bon typage, je vous invite à lire cet article : [Making Invalid State Unrepresentable](https://hugotunius.se/2020/05/16/making-invalid-state-unrepresentable.html).

Pour creuser les aspects "fail fast" et "périmètre de l'incertitude", je vous recommande la méthode [impure/pure/impure sandwitch](https://blog.ploeh.dk/2020/03/02/impureim-sandwich/).
