+++
title = "La notion d'Injector en Angular"
description = "Devenez un expert en Angular : comprendre la DI avec les Injectors et leur utilisation"

[taxonomies]
tags = ["Angular", "Avancé", "Programmation"]
+++

Un `Injector` est une mécanique dédiée à l'injection de dépendances en Angular. Les différents frameworks n'ont pas tous la même approche pour la DI. Par exemple, le fameux framework Java utilise le Spring IoC Container. La documentation d'Angular a longtemps mis en avant les `providers`, tout en négligeant d'autres éléments importants pour la DI. Les `Injector` ne font pas exception à cette règle. C'est pourquoi je vous propose de découvrir ensemble comment ils fonctionnent.

<!-- more -->

> _Jusqu'à la version 13, les explications sur la DI se concentrent sur les [cas d'usages](https://v13.angular.io/guide/dependency-injection-providers). Ce n'est qu'après que les rouages internes ont été documentés (quelques exemples [ici](https://angular.io/guide/dependency-injection) et [là](https://angular.io/guide/hierarchical-dependency-injection))._

## Comprendre le rôle de l'Injector en Angular

L'`Injector` est simplement un objet qui gère les `providers`. Il doit fournir les dépendances et s'assurer qu'elles sont des singletons grâce à un cache interne. Ce cache interne conserve les références aux `providers`, ce qui permet de renvoyer les instances existantes. Si nécessaire, l'`Injector` se charge de l'instanciation et conserve une référence pour les prochaines demandes, comme un jeton d'injection pour une classe ou un service. L'`Injector` joue donc un rôle clé entre les `providers` définis dans l'application et les composants/services qui utilisent ces `providers`.

Il est à noter que le fonctionnement d'un seul `Injector` est relativement simple. Toutefois, la complexité provient du fait qu'il en existe un grand nombre dans une application Angular et que, en tant que développeur Angular, on ne les manipule pas directement (sauf dans des cas exceptionnels).

## Des Injector partout

Ils sont nombreux, mais combien sont-ils exactement ? Chaque `NgModule` crée automatiquement un `Injector`, tandis que chaque composant ou directive qui possède des `providers` ou des `viewProviders` en crée également un. Dans le premier cas, il s'agit d'un `ModuleInjector`, et dans le deuxième, on parle d'`ElementInjector`. Il est donc évident que le nombre d'`Injector` augmentera rapidement avec l'application, avec au minimum autant d'`Injector` que de `NgModule`.

{% advice(type="tip", name="Astuce") %}

Lorsqu'un `Injector` est détruit, les instances qu'il gérait dans son cache sont également détruites. Les `NgModule` ne sont pas souvent détruits, mais ce n'est pas le cas pour les composants. Les `ElementInjector` sont donc fréquemment créés et détruits, tout comme les services associés.

{% end %}

## Gestion de dépendances et Injector

Chaque demande de dépendance est résolue en parcourant la hiérarchie des `Injector`. Pour un service injecté dans un composant, l'`ElementInjector` du composant est interrogé en premier (s'il existe), puis le `ModuleInjector` du module déclarant le composant, ensuite l'`Injector` du module parent, et ainsi de suite jusqu'au `ModuleInjector` _root_. La recherche s'interrompt dès qu'une instance du service est trouvée.

Pour le sommet de la structure,le fonctionnement est similaire pour toutes les applications Angular. Le `PlatformModule` est invoqué dans `main.ts` et tout ce qui n'a pas été trouvé est finalement géré par un `NullInjector` (cf. illustration ci-dessous).

![Illustration du root `Injector`, du platform `Injector` et du `NullInjector`](./base-injectors.png)

{% advice(type="tip", name="Astuce") %}

Le contenu des `providers` d'un `Injector` n'est pas immuable. Il est tout à fait possible d'ajouter des services supplémentaires à l'`Injector` _root_ après le démarrage de l'application. C'est ce qui se produit lorsque vous spécifiez `providedIn: "root"` pour un service qui n'est utilisé que dans un module en mode lazy-loading.

{% end %}

## Injector, Bundle et Instanciation

Il est impératif d'aborder la gestion du bundle et de l'instanciation en parlant d'`Injector`, car les erreurs de compréhension sont fréquentes. Il ne faut pas croire que le simple fait d'avoir un service déclaré avec `providedIn: "root"` signifie qu'il sera inclus dans le bundle principal, ni qu'il sera automatiquement instancié lors du démarrage de l'application.

Concernant le bundle, le processus de _tree-shaking_ se charge de ne garder que ce qui est vraiment nécessaire. Les services qui ne sont pas utilisés dans le bundle principal sont donc exclus. Cependant, les services définis dans les `providers` d'un module ne bénéficient pas de cet avantage et seront inclus dans le bundle en même temps que le module. Quant aux services `providedIn: "root"` utilisés dans plusieurs modules avec lazy-loading, l'option [commonChunk](https://github.com/angular/angular-cli/blob/ce3f1bd6b9bd4f584fba9abe9bd7d6bb81670bda/packages/angular_devkit/build_angular/src/builders/browser/schema.json#L262) détermine s'ils seront dupliqués dans chaque bundle ou s'ils seront regroupés dans un seul bundle commun (valeur par défaut).

Quand à l'instanciation, elle est reportée jusqu'au dernier moment. Si aucun consommateur n'utilise le service, il n'y a pas besoin de l'instancier. La déclaration `providedIn: "root"` garantit simplement que le service sera un singleton accessible dans toute l'application, mais il peut ne jamais être instancié. Soyez donc attentif lors de l'utilisation de services, car la DI peut être magique, mais il y a certaines nuances à prendre en compte.

## Conclusion

J'espère avoir amélioré votre compréhension du rôle des `Injector` dans Angular. Vous êtes à présent au courant que c'est un objet responsable de la gestion des instanciations à la racine, dans les modules et dans certains composants ou directives. Ces instanciations sont partagées avec les éléments enfants, mais peuvent également être remplacées localement par un sous-module ou un composant.

Lorsque vous effectuez un [chargement de composant dynamique](https://angular.io/guide/dynamic-component-loader), vous pouvez spécifier l'`Injector` à utiliser (voir [documentation sur `ViewContainerRef`](https://angular.io/api/core/ViewContainerRef#createcomponent)). Pour en savoir plus, je vous recommande de vérifier l'impact des _standalone components_ sur les `Injector`, disponible [ici](https://angular.io/guide/standalone-components#dependency-injection-and-injectors-hierarchy).

Aller plus loin : Si vous voulez plus de détails sur les `Injector` et l'injection en Angular, je vous recommande cet article (en anglais) qui vous apprendra tous les rouages : [How Angular Dependency Injection works under the hood](https://medium.com/ngconf/how-angular-dependency-injection-works-under-the-hood-cc210f7040bd)
