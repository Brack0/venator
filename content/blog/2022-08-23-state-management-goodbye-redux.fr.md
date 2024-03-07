+++
title = "Goodbye Redux"

[taxonomies]
tags = ["Gestion d'état", "Opinion", "Dev"]
+++

Parlons de State Management au niveau d'une application. Les états locaux (React Hook, Vue Composable, Service Angular + RxJS Subject) ont un usage limité par définition. Avec l'ampleur de l'écosystème React sur le monde professionnel, Redux est devenu la réponse évidente à la problématique du State Management. Etudions ensemble le processus assez naturel qui nous amène à Redux et prenons du recul sur nos choix.

<!-- more -->

Disclaimer : Si vous vouliez avoir un plaidoyer sur GraphQL, c'est raté. Rassurez-vous, il existe plein d'autres articles qui en parle. Ici, nous allons nous concentrer sur la notion de State Management et pas sur les impacts que pourrait avoir une API comme GraphQL sur votre architecture et votre State Management.

## Getting started

Commençons en prenant un projet de build frontend. Ce projet peut utiliser les technologies que vous préférez (Vue, React, Angular, Svelte, etc). Pour la suite de l'article, nous nous baserons sur Angular.

Alors pourquoi Angular est un bon candidat me direz-vous ? Simplement parce que pleins de projet Angular démarrent et parfois restent sans solution de State Management. La combinaison de RxJS et des services en singleton permettent de résoudre la majorité des besoins (SSOT : Single Source Of Truth, données "réactives", dissociation lecture/modification des données). Donc le State Management arrive plus tard et pas toujours de la bonne façon, alors qu'il devient critique. Pour les autres frontends, la problématique reste identique, même si certains utilisent des gestions d'états locaux tel que React Hook et Vue composable.

Mais reprenons notre projet de build. La question du State Management arrive sur la table. En effet, l'application grossit et la gestion des données devient chaotique. Les données partagées sont modifiées à plusieurs endroits. Certains états sont dupliqués et des problèmes de synchronisation apparaissent. Finalement les outils fournis par le Framework ne suffisent plus pour traiter toutes les problématiques d'état au sein de l'application. Heureusement notre sauveur est là : le pattern Redux.

## Implementation

Etant sur un projet Angular, on se tourne assez naturellement vers NgRx qui semble répondre à nos attentes. En effet NgRx se présente lui-même comme solution à cette problématique :

> You might use NgRx when you build an application with a lot of user interactions and multiple data sources, or when managing state in services are no longer sufficient.

On s'engage donc dans la mise en place du pattern Redux, ce qui provoque l'ajout de beaucoup d'éléments (store, state, actions, selectors, reducers, parfois combineReducers, etc.). Même en utilisant des constructeurs d'action et des générateurs de reducer, on se retrouve rapidement avec une structure conséquente (boilerplate). Un coût qui semble être accepté, voire acceptable pour certains. Pourtant, vous avez aussi quelques développeurs juniors sur le projet et vous ne voulez pas leur infliger la fameuse JS Fatigue\* dont vous avez peut-être souffert.

{% advice(type="info", name="JS Fatigue") %}

Beaucoup de développeurs se sentent dépassés par l'évolution rapide et perpétuelle de l'écosystème Javascript moderne et ressentent le besoin d'apprendre la plupart des technologies émergentes pour effectuer leur travail correctement. Ce phénomène est entre-autre causé par la crainte de ne plus être dans le coup ou de choisir des solutions obsolètes et/ou non-optimales.

{% end %}

## Next step

Pour simplifier les usages au sein de l'application, vous vous lancez alors dans la mise en place de façades. Et c'est naturel, ce patron de conception est là pour traiter ces symptômes, en fournissant une interface simple qui permet de masquer la complexité d'un système/module. Cela nous permet de regrouper en une seule façade toute la gestion d'un concept métier (la session utilisateur par exemple). En plus de simplifier les usages, on apporte également plus de sens à l'application en se focalisant sur les concepts métiers plutôt que sur les éléments techniques sur lesquels ils reposent. Que du positif en somme, non ?

## What's wrong ?

Alors qu'est-ce qui ne va pas dans notre solution ? La réponse est finalement assez simple quand on prend du recul sur le scénario exposé, la solution est disproportionnée par rapport aux usages qui en sont faits. La puissance de Redux est apportée par un découplage extrême entre les éléments. Sans pour autant être problématique, ajouter des façades réintroduit un peu de couplage. En conclusion, la solution est lourde à implémenter (pattern Redux), mais sans en tirer tous les avantages (couplage apporté par les façades). Une situation qui est résumée par Mike Ryan (co-fondateur de NgRx) dans un tweet :

> The Redux pattern has a high code cost to achieve indirection. Turning around and removing the indirection with facades makes me wonder why you are paying the Redux cost in the first place.

Donc on peut légitimement se poser la question de pourquoi on a choisi Redux.

## One fits all approach

Est-ce qu'il existe mieux ? Objectivement il est difficile de répondre à cette question. En revanche, il est tout à fait possible de répondre à la question "Est-ce qu'il existe mieux pour notre application ?". Et c'est finalement la première question à se poser. Quelle solution de State management sert le mieux nos besoins ? Car même si Redux répond à la plupart des besoins, sommes-nous capables d'absorber le coût de développement induit ? Avons-nous réellement besoin de tout ce que Redux fournit ? Si ce n'est pas le cas, il est intéressant de se pencher sur des solutions plus simples. Personnellement j'ai retenu deux solutions qui sortent du lot : Undux et Akita.

## Simplify your State management

Undux se montre comme une alternative simplifiée de Redux et Flux. Les concepts sont conservés mais l'implémentation est plus légère. La partie réactive est assurée par RxJS. Je vous renvoie vers le [Quick Start](https://undux.org/#quick-start) pour avoir des exemples. Les méthodes "get", "set" et "on" remplacent les selectors, les reducers et les effects sans avoir de code à écrire.

Cependant, mon coup de cœur est pour [Akita](https://github.com/salesforce/akita), également construit avec RxJS. Ici, on reprend les recettes qui fonctionnent ailleurs (pattern Flux et Redux, immutabilité, les notions de flux de données/programmation réactive, etc) et on construit une solution axée sur l'expérience développeur (concepts issus de la POO, courbe d'apprentissage modérée, boilerplate réduit).

La simplification, c'est aussi le choix de Vue, précédemment avec VueX et maintenant avec Pinia. Même si Vue laisse la porte ouverte à Redux, cela fait plusieurs années qu'un State Management plus simple est proposé, voire recommandé dans la documentation.

## Choices have been made

En tant que développeurs, nous devons faire des choix efficaces et c'est ce que nous faisons souvent finalement. Mais ces choix sont-ils efficients ? D'un autre côté, une solution qui répond parfaitement aux besoins actuels peut se montrer limitée pour les besoins futurs. Alors qu'est-ce qu'on fait ?

Premièrement, dans la très grande majorité des projets existants, vous n'avez pas besoin de toutes les fonctionnalités de Redux, le fameux [YAGNI](https://martinfowler.com/bliki/Yagni.html). Et la seconde réponse que j'aimerais apporter nous vient du monde agile via deux principes fondamentaux :

> Simplicity (the art of maximizing the amount of work not done) is essential.
<!-- -->
> The best architectures, requirements, and designs emerge from self-organizing teams.

C'est pour cela que la compréhension du besoin et du contexte de votre projet est primordiale. En ayant toutes les cartes en main et la possibilité d'agir\*, vous construirez une solution qui sert au mieux votre produit.

De votre côté, avez-vous déjà été tenté par d'autres solutions que Redux dans vos projets front ? Avez-vous franchi le pas, même sur des fronts assez conséquents ?

{% advice(type="tip", name="Inability to act") %}

Si vous ou votre équipe êtes pieds et poings liés sur l'ensemble des solutions techniques, fuyez !

{% end %}
