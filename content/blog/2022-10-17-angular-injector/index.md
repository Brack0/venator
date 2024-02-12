+++
title = "The Concept of Injector in Angular"
description = "Become an expert in Angular: Understanding DI with Injectors and their usage"

[taxonomies]
tags = ["Angular", "Advanced", "Programming"]
+++

An `Injector` is a mechanism dedicated to dependency injection in Angular. Not all frameworks build their Dependency Injection (DI) the same way. For example, the Spring IoC Container can be mentioned for the famous Java framework. The Angular documentation has long emphasized `providers`, leaving aside other elements necessary for DI. `Injectors` are no exception, which is why I propose to see together how they work.

<!-- more -->

> _Until version 13, explanations on DI focus on [use cases](https://v13.angular.io/guide/dependency-injection-providers). It was only after that the internal workings were documented (some examples [here](https://angular.io/guide/dependency-injection) and [there](https://angular.io/guide/hierarchical-dependency-injection))._

## How an Injector Works

Simply put, it is nothing more or less than an object that will manage `providers`. It must provide dependencies and ensure that they remain singleton via an internal cache. This internal cache retains references to the `providers`, which allows returning existing instances. If necessary, the `Injector` takes care of instantiation and retains a reference for future calls (e.g., injection token of a class or service). `Injectors` are therefore positioned between the `providers` defined in the application and the components/services that consume these `providers`.

What can be noticed is that the operation of an `Injector` alone is fairly trivial. All the complexity comes from the fact that there are many of them in an Angular application and as an Angular developer, we do not manipulate them directly (very rarely).

## Injectors, Injectors everywhere

Okay, they are many, but how many? Each `NgModule` will automatically create an `Injector`, and each component or directive that has `providers` or `viewProviders` will also create an `Injector`. In the first case, it is a `ModuleInjector`, and in the second case, we talk about an `ElementInjector`. We therefore understand that the number of `Injectors` will quickly grow with our application (at a minimum, as many `Injectors` as `NgModule`).

{% advice(type="tip") %}
When an `Injector` is destroyed, the instances it had in its cache are also destroyed. `NgModules` are not often destroyed but components are a different story. So `ElementInjectors` are frequently created and destroyed, as are the associated services.
{% end %}

## Injector and Dependency Resolution

Each dependency is resolved by progressively climbing the `Injector` chain. For a service injected at the component level, the `ElementInjector` (if the component has `providers`) is successively interrogated, the `ModuleInjector` of the module that declares the component, then the `Injector` of the parent module, the `Injector` of the parent of the parent, etc., until reaching the _root_ `ModuleInjector`. Of course, the search stops as soon as an instance of the requested service is obtained.

At the top of the structure, it is the same operation for all Angular applications. There is a `PlatformModule` that is called in `main.ts` and everything that has not been found ends up in a `NullInjector` (cf. illustration below).

![Illustration of the root `Injector`, the platform `Injector` and the `NullInjector`](base-injectors.png)

{% advice(type="tip") %}
The list of `providers` for an `Injector` is not set in stone. It is possible to assign other services to the _root_ `Injector` after the application starts. This is what happens when you do `providedIn: "root"` on a service that is only used in a module with lazy-loading.
{% end %}

## Injector, Bundle & Instanciation

It's impossible to talk about `Injector` without discussing bundle management and instantiation because confusion is common. Just because a service is `providedIn: "root"` doesn't mean it will be in the main bundle. Similarly, this service will not necessarily be instantiated at the start of the application.

As for the bundle, _tree-shaking_ takes care of cleaning up and removing anything that is not necessary. So all services that are not used in the main bundle are excluded. However, be aware that services defined in the `providers` of a module do not benefit from this advantage. They will be included in the bundle along with the module! And what about services `providedIn: "root"` used in multiple modules with lazy-loading? That's where the [commonChunk](https://github.com/angular/angular-cli/blob/ce3f1bd6b9bd4f584fba9abe9bd7d6bb81670bda/packages/angular_devkit/build_angular/src/builders/browser/schema.json#L262) option comes in, which will determine whether the service is duplicated in each bundle or if a common bundle is made (default value).

Regarding instantiation, it is postponed until the last moment. As long as there are no consumers of the service, there is no reason to instantiate it. This means that the `providedIn: "root"` part only guarantees that the service will be a singleton available throughout the application. This same service may never be instantiated. So be careful with services because DI is magical but there are some gotchas.

## Conclusion

I hope I have helped you gain a better understanding of the role of `Injector` in Angular's operation. You now know that it is an object that manages instantiations at the root, in modules, and in certain components or directives. These instantiations are shared with child elements but they can also be overridden locally by a submodule or in a component.

When you do [dynamic component loading](https://angular.io/guide/dynamic-component-loader), you can define the `Injector` used yourself (see [`ViewContainerRef` documentation](https://angular.io/api/core/ViewContainerRef#createcomponent)). For the curious, I recommend checking out the impact of _standalone components_ on `Injectors`, more details [here](https://angular.io/guide/standalone-components#dependency-injection-and-injectors-hierarchy).

Go further : If you want a more details on `Injector` and Injection in Angular, I recommend checking this deep dive : [How Angular Dependency Injection works under the hood](https://medium.com/ngconf/how-angular-dependency-injection-works-under-the-hood-cc210f7040bd)
