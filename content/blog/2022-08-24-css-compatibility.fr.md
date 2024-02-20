+++
title = "Comment gérer sa compatibilité CSS ?"

[taxonomies]
tags = ["CSS", "Tutoriel", "Programmation"]
+++

Avec l'omniprésence de Babel et Typescript dans l'univers Frontend, la compatibilité Javascript de nos applications n'est (presque) plus un sujet. Pour le HTML, seulement quelques attributs sont ajoutés depuis le HTML5 pour des fonctionnalités _bonus_ (lazyload, prefetch, etc.). Ils sont en général ignorés par les navigateurs incompatibles. Et pour le CSS ? Pas de polyfills à proprement parler, un support très hétérogène et la moindre esquive d'un `display:grid` est désastreux pour le visuel. Alors comment fait-on pour rationaliser notre CSS ? Voyons ça ensemble.

<!-- more -->

## Browserslist

[Browserslist](https://github.com/browserslist/browserslist) est une library utilisée par beaucoup de frontend. Il y a de fortes chances que dans vos projets actuels vous ayez un fichier `.browserslistrc` ou une configuration dans votre `package.json`. Que vous en soyez conscients ou non, Browserslist est présent dans les applications **React** (via create-react-app), **Angular**, **Vue** et plein d'autres (j'ai vérifié que les trois plus connus 😬).

Je vous donne un rapide exemple de comment fonctionne Browserslist en donnant une configuration et en regardant ensuite quels sont les navigateurs associés à cette configuration :

{% codeblock(name="package.json") %}

```json
{
  "browserslist": [
    "last 1 version",
    "> 5% and not dead",
    "BlackBerry 10",
    "not ie 11"
  ]
}
```

{% end %}

{% codeblock(name="$ npx browserslist") %}

```sh
and_chr 103
and_ff 101
and_qq 10.4
and_uc 12.12
android 103
bb 10
chrome 103
chrome 102
edge 103
firefox 103
ios_saf 15.5
kaios 2.5
op_mini all
op_mob 64
opera 89
safari 15.6
samsung 17.0
```

{% end %}

Dans notre cas, Browserslist (avec l'aide de [Can I Use](https://caniuse.com/)) va nous aider à configurer les deux prochains outils que je présente ci-dessous. En effet, nous allons l'utiliser en tant que référentiel des navigateurs à supporter. L'avantage est que Babel et [ESLint](https://github.com/amilajack/eslint-plugin-compat) utilise également ce référentiel pour le support Javascript.

{% advice(type="tip") %}

Vous pouvez également exploiter la puissance de Browserslist pour vos backends NodeJS. Plus d'infos sur le dépôt de [Browserslist](https://github.com/browserslist/browserslist).

{% end %}

## Le plugin PostCSS Autoprefixer

Si vous avez des projets Angular, React ou VueCLI, vous exploitez déjà [Autoprefixer](https://github.com/postcss/autoprefixer). Quoi qu'il en soit, son fonctionnement est très simple : inspecter votre CSS et rajouter les préfixes nécessaires pour certains navigateurs (`-webkit-`, `-moz-`, etc.). Vous l'aurez deviné, Autoprefixer va se baser sur Browserslist (+ Can I Use) pour savoir quand et quel type de préfixe ajouter.

Bref, ce plugin [PostCSS](https://github.com/postcss/postcss) vous permet de vous concentrer sur les règles CSS principales en vous déchargeant de la gestion d'une partie de la compatibilité de ces règles. Je vous renvoie vers la documentation pour sa mise en place s'il n'est pas déjà présent.

## Stylelint en tant que garde-fou

[Stylelint](https://stylelint.io/) est un linter de code CSS/SASS/Less/etc. A l'instar d'ESLint pour le Javascript, Stylelint va mettre en évidence, en fonction de règles configurables, des anomalies dans vos feuilles de styles. L'objectif est simple, éviter les erreurs et définir des conventions pour les développeurs.

En complément des best practices, vous pouvez également faire ressortir les règles non supportées par votre cible (les navigateurs à supporter). Tout comme Autoprefixer, on va se reposer sur Browserslist (+ Can I Use) pour faire le lien entre votre cible et les règles CSS utilisées. C'est exactement ce que fait le plugin Stylelint [no-unsupported-browser-features](https://github.com/ismay/stylelint-no-unsupported-browser-features) que je vous invite à essayer. N'oubliez pas vos IDE pour réduire la boucle de feedback.

{% advice(type="tip") %}

Quand vous combinez Stylelint et Autoprefixer (ce que je vous recommande chaudement), pensez à activer les règles de type `xxx-no-vendor-prefix` car Autoprefixer se charge du support à votre place. Nul besoin de polluer vos fichiers de styles.

{% end %}

## Tweaking

Une fois tous ces éléments en place, vous pouvez ajuster la liste des navigateurs à supporter en fonction de vos besoins. Les réglages par défaut de Browserslist conviennent pour la majorité des projets n'ayant pas de besoins spécifiques (`> 0.5%, last 2 versions, Firefox ESR, not dead`). Si cela n'est pas suffisant, à vous de définir un ensemble de règles plus ou moins contraignantes qui correspond à votre audience.

Enfin, j'attire votre attention sur une approche plus ciblée sur votre produit. Via des petits utilitaires ([Browserslist-GA](https://github.com/browserslist/browserslist-ga) et [browserslist-new-relic](https://github.com/syntactic-salt/browserslist-new-relic)), vous pouvez recueillir **vos** statistiques et les exploiter dans Browserslist (à l'aide des règles `> Y% in my stats`). Cela peut avoir du sens quand les statistiques publiques ne sont pas pertinentes, comme sur des marchés très ciblés (exemple : la Chine) ou des plateformes privées.

## Conclusion

En utilisant ces outils, vous pouvez sécuriser votre production de CSS. Stylelint nous permet de voir certaines subtilités dans nos usages. Par exemple, la propriété `gap` combinée avec un affichage `flex` est disponible depuis mi-2020 seulement ! En comparaison, utilisée avec `grid` cette propriété existe depuis mi-2018, et même depuis début 2017 _[NDLR: c'est à dire en même temps que CSS Grid]_ via son ancien nom (`grid-gap`). Il existe plein d'autres exemples de ce genre et à moins de passer son temps sur MDN vous allez en rater plus d'un dans vos projets. En complément, Autoprefixer se charge des alias, préfixes et autres sophistications requis par nos navigateurs préférés. De quoi s'éviter des bugs tordus et des maux de têtes.
