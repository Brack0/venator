+++
title = "Qu'est qu'on pourrait améliorer sur Angular ?"

[taxonomies]
tags = ["Angular", "Opinion", "Programming"]
+++

Je vous propose quelques fonctionnalités qui pourraient améliorer la Developer eXperience et le framework dans sa globalité.

<!-- more -->

## Build

Le temps de build est sûrement l'élément que je trouve le plus problématique pour Angular (actuellement en version 14). Des améliorations sont en cours ou prévues mais Angular est en retrait par rapport aux autres frameworks frontend.

Parmi les derniers travaux menés, on peut citer l'utilisation de [cache](https://angular.io/cli/cache) pour certaines étapes du build. Cependant la feature n'est disponible que depuis la version 13. On peut également se tourner vers [NX](https://nx.dev/) pour profiter de plusieurs optimisations complémentaires : [rebuild des éléments affectés par une modification](https://nx.dev/using-nx/affected) et [build incrémental](https://nx.dev/ci/setup-incremental-builds-angular) par exemple.

En parallèle, il existe les bundlers "modernes" comme [Vite](https://vitejs.dev/) qui exploitent la puissance des modules ECMAScript. Malheureusement, ils ne sont pas encore disponibles en Angular. Néanmoins, on peut espérer des avancées sur le sujet bientôt car un support expérimental d'[ESBuild](https://esbuild.github.io/) est présent en Angular 14. Plus d'informations sur Github : <https://github.com/angular/angular-cli/pull/22995>

## Gestion des bundles

Angular est connu pour ne pas être le plus exemplaire sur le bundle du framework en lui-même (`@angular/core`, `@angular/browser`, `@angular/router`, etc.). Même si Ivy et les constantes améliorations réduisent progressivement le bundle minimal, on se souvient des fameux Hello World ou Todo MVC où la place du framework dans le bundle était bien supérieure aux concurrents.

En guise de comparaison, je vous propose d'analyser deux applications similaires ayant pour but de montrer les fondamentaux d'un framework frontend :

- [Heroes Angular](https://github.com/johnpapa/heroes-angular), Angular 11
- [Heroes React](https://github.com/johnpapa/heroes-react), React 16

Sur la première page des deux applications (<https://papa-heroes-angular.azurewebsites.net> et <https://papa-heroes-react.azurewebsites.net>), on observe un écart prononcé entre les assets.

### Angular 11

![Assets téléchargés en Angular : 926 Ko](heroes-angular.png)

### React 16

![Assets téléchargés en React : 754 Ko](heroes-react.png)

Cela représente presque 20% de moins pour l'application en React. Même si l'écart se réduit (en pourcentage) quand l'application prend de l'ampleur, la taille et la performance de nos applications sont un élément crucial de l'expérience utilisateur. La performance et les optimisations sont souvent mentionnées dans la [roadmap](https://angular.io/guide/roadmap), espérons que cela continue !

{% advice(type="note") %}

Il existe également une application Vue dans le même thème, cependant il semble ne pas y avoir de code-splitting au niveau des routes. Etant donné que l'on ne mesure pas la même chose, la comparaison n'a pas de sens (il y a plus de 3 Mo d'assets pour l'application Heroes Vue).

{% end %}

{% advice(type="note") %}

Au moment de l'écriture de cet article, les applications ont quelques versions de retard (Angular 14, React 18). Les chiffres mentionnés précédemment sont potentiellement obsolètes.

{% end %}

## Server-side rendering (SSR)

Lorsque l'on fait de la génération de page côté client, les frameworks Javascript vont jouer un certain nombre d'actions au niveau du client (requêtes HTTP, construction du state, etc.). Pourtant, quand on regarde le fonctionnement de [Qwik](https://qwik.builder.io/), on remarque que la majorité des frameworks frontend pourrait également réduire ces opérations faites par le navigateur quand on fait du SSR. Sans forcément aller aussi loin que Qwik, on pourrait faciliter la transmission des états depuis le serveur (en utilisant systématiquement le [TransferState](https://angular.io/api/platform-browser/TransferState) par exemple). Cela nous permettrait d'avoir automatiquement en cache les requêtes HTTP et le state de l'application sans configuration supplémentaire.

{% advice(type="note") %}

Aujourd'hui, c'est à vous de configurer le TransferState pour l'utiliser. A titre d'exemple, si vous voulez récupérer les requêtes HTTP utilisées pour la génération de la page serveur, vous devez écrire un intercepteur HTTP qui va insérer dans le TransferState lors du contexte serveur, et lire depuis le TransferState quand on est au niveau du navigateur.

{% end %}

De plus, on pourrait également alléger la partie _browser_ car le serveur a déjà traité la génération de la page. En effet, si on récupère la page générée et les différents états, nous n'avons plus besoin d'avoir le code Javascript permettant de (re)générer la page actuelle. C'est l'un des arguments principaux de Qwik et il est pertinent car ni Angular, ni d'autres frameworks actuels sont matures à ce sujet.

## Conclusion

Certains frameworks comme Angular ou Next sont vraiment complets. Pourtant, je ne vous apprends rien en disant que le framework parfait n'existe pas. Cela ne les empêche pas d'avoir des défauts et des imperfections.

Je pense enrichir cet article au fur et à mesure que je trouve d'autres pistes d'améliorations pour Angular. Et vous, vous avez des propositions pour révolutionner Angular ?
