+++
title = "Deep dive Angular : Performance des templates"

[taxonomies]
tags = ["Angular", "Performance", "Programming"]
+++

La performance est un sujet récurrent quand on parle de frontend. Les principaux acteurs (librairies/frameworks Javascript) y font tous référence dès la page d'accueil. Angular est connu pour intégrer un bundle plus complet mais plus lourd que ses concurrents directs. Même si ces différentes technologies n'embarquent pas les mêmes fonctionnalités, il reste une problématique à résoudre pour tous : le rendu HTML. Nous allons analyser ensemble le fonctionnement d'Angular dans trois cas précis : la gestion des blocs statiques, la mise à jour du DOM et la mise en cache de valeurs.

![Cover](cover.webp)

<!-- more -->

Cet article se rapproche de ce qui a été fait par Grafikart en comparant Vue à React : <https://grafikart.fr/tutoriels/vuejs-perf-react-1941>. Certains exemples de code sont volontairement proches pour se donner des éléments de comparaison avec React et Vue.

Disclaimer : L'objectif de ce Deep dive est d'étudier la performance des templates Angular et de comparer leur fonctionnement avec ceux des concurrents directs. La performance d'un framework frontend ne peut et ne doit se résoudre à cette analyse. De même, elle ne peut s'en soustraire.

{% advice(type="caution", name="Précision technique") %}

La notion de template en Angular peut faire référence à la partie d'un composant écrite en HTML, mais aussi à un `<ng-template>`. Ce double sens peut parfois rendre le propos confus. Si tel est le cas, vous pouvez bien évidemment m'en faire part directement, cela n'en sera que bénéfique pour les prochains lecteurs.

{% end %}

## Les blocs statiques

Pour commencer, partons d'un simple template comme celui-ci et essayons de l'analyser :

```ts
import { Component } from "@angular/core";

@Component({
  selector: "app-root",
  template: `
    <h1>Hello world</h1>
    <div *ngIf="foo === 'bar'">Lorem ipsum dolor sit amet</div>
    <p>{{ value }}</p>
  `,
})
export class AppComponent {
  public foo = "";
  public value = "Value";
}
```

Le code produit par la compilation Angular est un peu plus fourni. Voici la partie qui concerne AppComponent avec quelques ajustements pour la lisibilité (build en mode développement, renommage des imports webpack, suppression des symboles 'ɵ').

```js
function AppComponent_div_2_Template(rf, ctx) {
  if (rf & 1) {
    angularCore["elementStart"](0, "div");
    angularCore["text"](1, "Lorem ipsum dolor sit amet");
    angularCore["elementEnd"]();
  }
}
class AppComponent {
  constructor() {
    this.foo = "";
    this.value = "Value";
  }
}
AppComponent.fac = function AppComponent_Factory(t) {
  return new (t || AppComponent)();
};
AppComponent.cmp = /*@__PURE__*/ angularCore["defineComponent"]({
  type: AppComponent,
  selectors: [["app-root"]],
  decls: 5,
  vars: 2,
  consts: [[4, "ngIf"]],
  template: function AppComponent_Template(rf, ctx) {
    if (rf & 1) {
      angularCore["elementStart"](0, "h1");
      angularCore["text"](1, "Hello world");
      angularCore["elementEnd"]();
      angularCore["template"](2, AppComponent_div_2_Template, 2, 0, "div", 0);
      angularCore["elementStart"](3, "p");
      angularCore["text"](4);
      angularCore["elementEnd"]();
    }
    if (rf & 2) {
      angularCore["advance"](2);
      angularCore["property"]("ngIf", ctx.foo === "bar");
      angularCore["advance"](2);
      angularCore["textInterpolate"](ctx.value);
    }
  },
  directives: [angularCommon.NgIf],
  encapsulation: 2,
});
```

Deux éléments importants sont à noter sur le code que l'on peut observer. Premièrement, on peut remarquer une fonction qui contient le contenu du \*ngIf (cf. `AppComponent_div_2_Template`). Ce n'est pas surprenant, souvenez-vous que l'astérisque sur les directives est un sucre syntaxique pour un bloc avec `<ng-template>` (pour rappel <https://angular.io/guide/structural-directives#structural-directive-shorthand>). En fait, une fonction de rendu sera créée pour chaque `<ng-template>` dans notre application. Cela veut dire que le rendu est non seulement découpé au niveau des composants, mais également en fonction des `<ng-template>` présents dans l'application.

Pour le deuxième aspect qui nous intéresse, concentrons-nous sur une portion de code que l'on n'a assez peu l'occasion de voir quand on fait du développement web : `(rf & 1)` et `(rf & 2)`. Oui il s'agit bien d'une opération bit à bit. Je vous rassure, nous n'entrerons pas dans les détails ici. Toutefois, selon vous, à quoi pourrait servir ces conditions dans les fonctions de rendu ? Regardons ensemble le code pour essayer d'en déduire les subtilités.

Dans la partie `rf & 1`, on peut identifier la création d'un `<h1>` avec son contenu `"Hello world"`, puis un template et enfin un `<p>`. Ces éléments ressemblent beaucoup à ce qu'on a déclaré dans notre composant. Dans le deuxième bloc (`rf & 2`), si on met de côté l'instruction opaque `"advance"`, il ne reste que le `ngIf` et l'interpolation `{{ value }}`.

Si maintenant je vous dis que la variable `rf` vient de RenderFlag, vous devriez avoir une bonne idée de ce qu'il se passe. En fait, en Angular les fonctions de rendu contiennent deux blocs d'instructions, un premier pour la création du template et le deuxième pour les mises à jour dudit template.

Que dire de tout ça ? Tout d'abord, on peut voir que les blocs statiques sont définis dans la partie création (cf. `rf & 1` => Partie "création" de la fonction de rendu) et qu'ils ne sont pas modifiés lors des mises à jour du template (cf. `rf & 2`). C'est plutôt un bon point pour Angular, qui bénéficie comme VueJS d'une détection automatique des contenus statiques, contrairement à React qui requiert l'usage de `React.memo()` et d'un composant dédié. Demi-point bonus pour Angular par rapport à VueJS, les contenus statiques ne sont créés que s'ils sont visibles, là où en VueJS tous ces contenus sont générés dès la création du composant même s'ils sont masqués par un `v-if`. La seconde conclusion que l'on peut tirer concerne les rerendu ou plutôt l'absence de rerendu mais je vous propose de traiter ça plus en détails dans le chapitre suivant.

## Les mises à jour de template

NB : Etant donné que les illustrations de code à partir de maintenant peuvent être conséquentes, un commit avec les composants et un extrait du build en mode développement sera fourni en guise d'exemple.

Avec un découpage des composants à partir des `<ng-template>`, Angular isole les problématiques de création et de mise à jour très finement. Si bien que les optimisations faites au niveau des composants sont aussi valables pour les templates. C'est notamment le cas de la différenciation entre les propriétés qui provoquent une mise à jour du template et celles qui sont externes. Ainsi, comme VueJS et React (via memo), Angular ne va pas faire de rendu (ou plutôt d'update si on se fie à l'analyse du chapitre précédent) pour les composants enfants dont les entrées n'ont pas été modifiées. Cependant, comme nous l'avons vu auparavant, Angular est également capable de limiter les mises à jour aux éléments pertinents parmi le template parent et chaque `<ng-template>`.

Pas vraiment convaincu par ces explications ? Vérifions ensemble avec un exemple :

- Commençons par lancer l'[application préparée pour l'occasion](https://github.com/Brack0/angular-template-performance/tree/5113f29f796c232c5e41d4788ffe97d58ca1b8d6), puis saisissons '_compteur_' dans le champ de recherche pour activer la condition du `*ngIf`.
- Deux boutons s'affichent comme prévu : '_Incrémenter_' et '_Ajouter un item_'
- En cliquant sur le bouton '_Incrémenter_', on déclenche la fonction `AppComponent_div_7_Template_button_click_3_listener()` (d'après le fichier `main.js` reporté dans les [assets](https://github.com/Brack0/angular-template-performance/blob/5113f29f796c232c5e41d4788ffe97d58ca1b8d6/src/assets/main.js))
- Remarquer le contenu du `*ngIf` est dans la fonction `AppComponent_div_7_Template()` et que celui du `*ngFor` est dans `AppComponent_tr_16_Template()`.

Voici ce que l'on obtient en regardant le Flamegraph associé à notre clic :

![Flamegraph when updating a template](https://raw.githubusercontent.com/Brack0/angular-template-performance/5113f29f796c232c5e41d4788ffe97d58ca1b8d6/src/assets/update-template-flamegraph.png)

En y regardant de plus près, on peut effectivement distinguer les étapes dans le fonctionnement d'Angular (cycle de vie, étapes de rafraichissement, détections de différences, validations, etc). De plus, on retrouve des éléments connus comme la fonction `AppComponent_div_7_Template_button_click_3_listener()` associée au clic sur le bouton, mais aussi des fonctions de rendu comme `AppComponent_Template()` et `AppComponent_div_7_Template()`. Pourtant, il n'y a aucune trace de la fonction `AppComponent_tr_16_Template()`. Même en cherchant bien, il nous est impossible de trouver un appel de la fonction qui fait le rendu du contenu du `*ngFor` ! Ce qui veut dire que le contenu du `*ngFor` n'est pas impacté par les actions satellites. Pour être exact, la fonction `AppComponent_tr_16_Template()` ne s'est pas déclenchée car il y a eu un contrôle sur le tableau `items` qui est en paramètre du `*ngFor`. Dans notre cas, pas de changements sur `items` donc pas d'appel à la fonction. A l'inverse, la mutation, l'ajout ou la suppression d'éléments aurait provoqué un appel à `AppComponent_tr_16_Template()` et une mise à jour du template.

Donc ça voudrait dire qu'à chaque mise à jour des templates Angular va vérifier un par un chaque élément de chaque tableau pour détecter d'éventuels changements, ce n'est pas terrible pour les performances non ? Non effectivement et on peut le constater rapidement si on utilise beaucoup de `*ngFor` sans précaution. Mais rassurez-vous, je vous liste ci-dessous trois méthodes que vous connaissez peut-être déjà pour réduire efficacement les détections de changements sur les tableaux :

- Utiliser la fonction trackBy pour simplifier les comparaisons entre les éléments
- Isoler la boucle `*ngFor` dans un composant utilisant la stratégie _OnPush_ avec le tableau en `@Input()`, seuls les changements de référence du tableau déclencheront un rendu par défaut (libre à vous ensuite de forcer d'autres rendus si besoin)
- Sortir de zone.js quand vous risquez de provoquer beaucoup de mise à jour des templates sur une courte période (<https://angular.io/api/core/NgZone#runOutsideAngular>)

Avant de finir cette section sur ~~le rerendu~~ la mise à jour des templates Angular, vous pouvez retrouver [ici](https://github.com/Brack0/angular-template-performance/tree/9719bb1f0a4c6322672028d1e31a6cc8ef02e722) un exemple qui met en avant la stratégie _OnPush_.

En analysant le comportement d'Angular, on constate que le Framework répond à la problématique initiale : éviter les rendus et les rafraichissements inutiles. Néanmoins, il est difficile de dire si la solution est plus efficace que celle proposée par React et VueJS. D'un côté, on a un découpage fin et beaucoup d'efforts sur la détection de changement ; de l'autre, un peu moins de vérifications et l'utilisation du VirtualDOM pour limiter les mises à jour du DOM. Quelques pistes de réponse sur ce fameux benchmark : <https://krausest.github.io/js-framework-benchmark/index.html>

## Mise en cache des valeurs calculées dans les templates

Si vous avez déjà fait un peu d'Angular, vous savez que les optimisations que j'ai mentionnées précédemment ne s'applique pas dans un cas précis : les fonctions dans les templates. Qu'elles soient explicites (`*ngIf="isValid()`) ou implicites (`{{ a * b + c }}`), les fonctions peuvent également causer des problèmes de performance. A chaque raffraichissement de l'application, toutes les fonctions présentes dans les composants affichés sont réévaluées. Dans certains cas cela peut être désastreux. Imaginez un tableau de données avec 500 lignes et des colonnes contenant des dates (date de début, date de fin, date de sortie, date de création, etc). Les performances s'écroulent quand chaque évènement de scroll provoque un formatage de toutes les dates du tableau.

Vous pouvez constater par vous-même, en reprenant [le code du chapitre précédent](https://github.com/Brack0/angular-template-performance/tree/5113f29f796c232c5e41d4788ffe97d58ca1b8d6), que l'ajout d'un item dans le tableau provoque un recalcul de `{{ count * 2 }}` (constater l'appel à `ɵɵtextInterpolate2`, `textBindingInternal`, `updateTextNode` puis `setValue` dans le Flamegraph).

Alors comment faire pour traiter les besoins de valeurs calculées sans faire exploser les performances, le nombre d'attributs et le nombre de fonctions utilitaires dans nos composants ? La réponse d'Angular s'appelle un `Pipe` et se base sur deux concepts : les références (souvenez-vous, la stratégie _OnPush_ aime bien ça également) et la mise en cache. En prenant [le dernier commit qui nous intéresse](https://github.com/Brack0/angular-template-performance/tree/ff5b6a6962341e562aa61810658148824080debe), vous devriez maintenant constater que l'ajout d'un élément dans le tableau ne provoque plus de calcul de `{{ count * 2 }}`.

Ni Angular, ni React, ni VueJS ne se démarque sur cet aspect. Les trois Frameworks permettent d'utiliser des méthodes directement dans les templates, avec les défauts de performance mentionnés plus haut. De plus, chacun propose une solution de mise en cache des valeurs : `Pipe` pour Angular, `useMemo()` pour React et `computed()` pour VueJS

## Angular est sous-estimé ?

Résumons. Angular est capable d'isoler les contenus statiques pour éviter de les regénérer. De plus, au lieu de regénérer des morceaux plus ou moins conséquents en utilisant un Virtual DOM, il va analyser finement les templates à mettre à jour. Même si les méthodes diffèrent, l'objectif est identique : limiter les modifications du DOM au strict minimum car elles peuvent s'avérer couteuses. Enfin, pour la gestion des valeurs calculées, tout le monde est à la même enseigne en proposant une méthode directe mais peu performante et une méthode optimisée avec de la mise en cache.

Quelle surprise de découvrir qu'Angular soit aussi pointu et précis sur la gestion des templates. Pour être honnête, je m'attendais à avoir un système complexe et lourd. Même si cela ne fait pas d'Angular le meilleur Framework car il a toujours ses défauts et il ne convient pas à tous, le cœur du Framework, à savoir le rendu d'élément HTML, a des atouts face aux stars du moment, React et VueJS. De quoi peut-être vous (re)donner envie de l'utiliser ?
