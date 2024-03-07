+++
title = "Du pattern matching en JS ?"

[taxonomies]
tags = ["Javascript", "Dev"]
+++

Non malheureusement ce n'est pas (encore) possible en Javascript, ni en Typescript. Mais on peut essayer de s'en rapprocher, notamment en séparant l'identification d'un scénario de son exécution. L'objectif est de renforcer la lisibilité et rapprocher le code du problème à résoudre. Pour illustrer le propos, nous utiliserons le traitement d'un article de blog (ajout, suppression, publication, etc.) à partir des informations contenues dans cet article. De plus, je vous propose d'y aller étape par étape afin que vous puissiez appliquer ce refactoring dans votre code.

<!-- more -->

{% advice(type="note") %}

Les extraits de code présentés ci-dessous sont en Typescript et reposent sur la programmation fonctionnelle. Donc pas de classes et pas d'héritage, mais des data, des fonctions et aussi des fonctions de fonctions.

{% end %}

## Définition d'un contexte de travail

### Un peu de modélisation

Commençons par définir une interface qui représente la payload à traiter, dans le cas présent un article. Cet article possède plusieurs attributs qui indique s'il est à supprimer, à publier ou à créer.

{% codeblock(name="article.ts") %}

```typescript
export interface Article {
  delete: boolean;
  publishAction: PublishAction;
  id: string;
  content: string;
}

/**
 * Using Object instead of Enum here
 * @see https://www.typescriptlang.org/docs/handbook/enums.html#objects-vs-enums
 */
export const PUBLISH_ACTION = {
  none: 0,
  unpublish: 1,
  publish: 2,
} as const;

export type PublishAction = typeof PUBLISH_ACTION[keyof typeof PUBLISH_ACTION];
```

{% end %}

En complément, définissons deux interfaces pour améliorer le typage de notre exemple : un repository et un logger.

{% codeblock(name="dependencies.ts") %}

```typescript
export interface ArticleRepository {
  createOrUpdate: <T>(entity: T) => Promise<T>;
  delete: <T>(entity: T) => Promise<T>;
  publish: <T>(entity: T) => Promise<T>;
  unpublish: <T>(entity: T) => Promise<T>;
}

export interface Logger {
  debug: (...args: any[]) => void;
  info: (...args: any[]) => void;
  warn: (...args: any[]) => void;
  error: (...args: any[]) => void;
}
```

{% end %}

### Une première version (très) procédurale

Je vous propose l'implémentation suivante pour le traitement d'un article comme base de réflexion.

{% codeblock(name="index.ts") %}

```typescript
import { Article, PUBLISH_ACTION } from "./article";
import { ArticleRepository, Logger } from "./dependencies";

type ProcessArticle = (dependencies: {
  articleRepository: ArticleRepository;
  logger: Logger;
}) => (parameters: { article: Article }) => Promise<unknown>;
export const processArticle: ProcessArticle =
  ({ articleRepository, logger }) =>
  ({ article }) => {
    if (article.delete) {
      logger.debug(`Delete article with id : ${article.id}`);
      return articleRepository
        .delete(article)
        .then(() =>
          logger.debug(`Successfully deleted article with id : ${article.id}`)
        )
        .catch((err) =>
          logger.warn(`Cannot delete article with id : ${article.id}`, err)
        );
    }

    if (article.publishAction === PUBLISH_ACTION.unpublish) {
      logger.debug(`Unpublish article with id : ${article.id}`);
      return articleRepository
        .unpublish(article)
        .then(() =>
          logger.debug(
            `Successfully unpublished article with id : ${article.id}`
          )
        )
        .catch((err) =>
          logger.warn(`Cannot unpublish article with id : ${article.id}`, err)
        );
    }

    if (article.publishAction === PUBLISH_ACTION.publish) {
      logger.debug(`Publish article with id : ${article.id}`);
      return articleRepository
        .publish(article)
        .then(() =>
          logger.debug(`Successfully published article with id : ${article.id}`)
        )
        .catch((err) =>
          logger.warn(`Cannot unpublish article with id : ${article.id}`, err)
        );
    }

    if (article.content) {
      logger.debug(`Create or update article with id : ${article.id}`);
      return articleRepository
        .createOrUpdate(article)
        .then(() =>
          logger.debug(
            `Successfully created or updated article with id : ${article.id}`
          )
        )
        .catch((err) =>
          logger.warn(
            `Cannot create or update article with id : ${article.id}`,
            err
          )
        );
    }

    throw new Error(
      `Unexpected value for article : ${JSON.stringify(article)}`
    );
  };
```

{% end %}

Avant de travailler sur la structure du code, parcourons ensemble le contenu de la fonction `processArticle()` afin de comprendre comment notre article est traité.

Pour commencer, on peut déjà écarter le logger dont le rôle est d'afficher en debug les différentes étapes et en warn les anomalies sur le traitement d'un article. Ensuite on remarque que la fonction gère 5 use-case distincts : suppression, dépublication, publication, création/mise à jour et un cas d'erreur. Cependant, même si on comprend le code écrit ligne par ligne, on peut identifier plusieurs problèmes majeurs : les éléments structurants sont noyés parmi le reste du code, la fonction réalise seule plusieurs actions, l'identification des scénarios est couplée à l'exécution de ces scénarios. Cette fonction a donc deux responsabilités (l'identification du scénario et les actions à réaliser) et viole le principe de responsabilité unique (SRP).

Si vous n'êtes pas convaincu que ce couplage est problématique, essayez de visualiser les impacts sur le code des besoins suivants :

- "Bug : au moment de la publication les articles doivent avoir un contenu"
- "Feature : Rendre impossible la suppression d'articles qui sont publiés"
- "Feature : Permettre la création et la publication en une seule fois"

Sans refactoring, on voit que le code va vite devenir difficile à maintenir et il sera de plus en plus compliqué d'identifier l'intention derrière le code.

## Petit détour par Clean Code

Afin de traiter certains des problèmes mentionnés précédemment, je vous propose d'utiliser une technique classique : la décomposition en plusieurs fonctions (Clean Code : "Extract till you drop").

{% codeblock(name="index.ts") %}

```typescript
import { Article, PUBLISH_ACTION } from "./article";
import { ArticleRepository, Logger } from "./dependencies";

type ProcessArticle = (dependencies: {
  articleRepository: ArticleRepository;
  logger: Logger;
}) => (parameters: { article: Article }) => Promise<unknown>;
export const processArticle: ProcessArticle =
  ({ articleRepository, logger }) =>
  ({ article }) => {
    if (article.delete) {
      return deleteArticle({ articleRepository, logger })({ article });
    }

    if (article.publishAction === PUBLISH_ACTION.unpublish) {
      return unpublishArticle({ articleRepository, logger })({ article });
    }

    if (article.publishAction === PUBLISH_ACTION.publish) {
      return publishArticle({ articleRepository, logger })({ article });
    }

    if (article.content) {
      return createOrUpdateArticle({ articleRepository, logger })({ article });
    }

    throw new Error(
      `Unexpected value for article : ${JSON.stringify(article)}`
    );
  };

type DeleteArticle = (dependencies: {
  articleRepository: ArticleRepository;
  logger: Logger;
}) => (parameters: { article: Article }) => Promise<unknown>;
export const deleteArticle: DeleteArticle =
  ({ articleRepository, logger }) =>
  ({ article }) => {
    logger.debug(`Delete article with id : ${article.id}`);

    return articleRepository
      .delete(article)
      .then(() =>
        logger.debug(`Successfully deleted article with id : ${article.id}`)
      )
      .catch((err) =>
        logger.warn(`Cannot delete article with id : ${article.id}`, err)
      );
  };

type UnpublishArticle = (dependencies: {
  articleRepository: ArticleRepository;
  logger: Logger;
}) => (parameters: { article: Article }) => Promise<unknown>;
export const unpublishArticle: UnpublishArticle =
  ({ articleRepository, logger }) =>
  ({ article }) => {
    logger.debug(`Unpublish article with id : ${article.id}`);

    return articleRepository
      .unpublish(article)
      .then(() =>
        logger.debug(`Successfully unpublished article with id : ${article.id}`)
      )
      .catch((err) =>
        logger.warn(`Cannot unpublish article with id : ${article.id}`, err)
      );
  };

type PublishArticle = (dependencies: {
  articleRepository: ArticleRepository;
  logger: Logger;
}) => (parameters: { article: Article }) => Promise<unknown>;
export const publishArticle: PublishArticle =
  ({ articleRepository, logger }) =>
  ({ article }) => {
    logger.debug(`Publish article with id : ${article.id}`);

    return articleRepository
      .publish(article)
      .then(() =>
        logger.debug(`Successfully published article with id : ${article.id}`)
      )
      .catch((err) =>
        logger.warn(`Cannot unpublish article with id : ${article.id}`, err)
      );
  };

type CreateOrUpdateArticle = (dependencies: {
  articleRepository: ArticleRepository;
  logger: Logger;
}) => (parameters: { article: Article }) => Promise<unknown>;
export const createOrUpdateArticle: CreateOrUpdateArticle =
  ({ articleRepository, logger }) =>
  ({ article }) => {
    logger.debug(`Create or update article with id : ${article.id}`);

    return articleRepository
      .createOrUpdate(article)
      .then(() =>
        logger.debug(
          `Successfully created or updated article with id : ${article.id}`
        )
      )
      .catch((err) =>
        logger.warn(
          `Cannot create or update article with id : ${article.id}`,
          err
        )
      );
  };
```

{% end %}

La première chose que l'on remarque, c'est que la lisibilité est bien meilleure. On a maintenant une base plus saine pour construire un pattern matching dans la fonction `processArticle()`. Cela va nous permettre d'isoler l'identification du scénario afin de pouvoir ajouter facilement des nouveaux use-case ou changer les conditions sans impacter le reste du traitement

## Identifier précisément et explicitement le scénario

> Explicit is better than implicit

Dans notre exemple nous avons 5 use-case distincts, donc faisons ressortir explicitement ces 5 scénarios. Pour ce faire, on peut baser sur un simple Enum comme ci-dessous.

{% codeblock(name="command.ts") %}

```typescript
export const COMMAND = {
  delete: 0,
  unpublish: 1,
  publish: 2,
  createOrUpdate: 3,
  unknown: 4,
} as const;

export type Command = typeof COMMAND[keyof typeof COMMAND];
```

{% end %}

Ensuite, nous pouvons implémenter une méthode dont le rôle est d'identifier une intention (nommée `Command` dans notre exemple) à partir des informations contenues dans l'article. Il nous suffit de reprendre l'articulation du code précédent et de renvoyer le bon use-case.

{% codeblock(name="command.ts") %}

```typescript
import { Article, PUBLISH_ACTION } from "./article";

type GetCommand = (parameters: { article: Article }) => Command;
export const getCommand: GetCommand = ({ article }) => {
  if (article.delete) {
    return COMMAND.delete;
  }

  if (article.publishAction === PUBLISH_ACTION.unpublish) {
    return COMMAND.unpublish;
  }

  if (article.publishAction === PUBLISH_ACTION.publish) {
    return COMMAND.publish;
  }

  if (article.content) {
    return COMMAND.createOrUpdate;
  }

  return COMMAND.unknown;
};
```

{% end %}

## S'approcher du pattern matching

L'astuce principale est de mêler les concepts de [literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types#object_literals) et d'[IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE). On va utiliser un objet litéral comme structure de notre pattern matching. Les clés de l'objet correspondent aux différents use-case et les valeurs associées sont les implémentations de ces use-case. Point important, on utilise des arrow functions pour éviter d'exécuter tous les scénarios à la création de l'objet.

```typescript
const objectLiteral = {
  case1: () => fun1(),
  case2: () => fun2(),
  case3: () => fun3(),
};
```

A partir d'un objet litéral comme ci-dessus, l'objectif est de cibler le use-case et d'exécuter la bonne callback. Je vous donne une illustration très basique.

```typescript
const callback = objectLiteral["case1"];

callback(); // Will call fun1()
```

A l'étape précédente, nous avons défini une fonction `getCommand()` dont le rôle est d'identifier le use-case. Nous avons également défini des identifiants pour nos use-cases (les valeurs de `Command`). Donc, nous pouvons remplacer les `case1`, `case2` et `case3` par des valeurs de `Command`, et les `fun1()`, `fun2()` et `fun3()` par les fonctions extraites plus haut. Ce qui donne :

```typescript
const objectLiteral = {
  [COMMAND.delete]: () =>
    deleteArticle({ articleRepository, logger })({ article }),
  [COMMAND.unpublish]: () =>
    unpublishArticle({ articleRepository, logger })({ article }),
  [COMMAND.publish]: () =>
    publishArticle({ articleRepository, logger })({ article }),
  [COMMAND.createOrUpdate]: () =>
    createOrUpdateArticle({ articleRepository, logger })({ article }),
  [COMMAND.unknown]: () => {
    throw new Error(
      `Unexpected value for article : ${JSON.stringify(article)}`
    );
  },
};

const callback = objectLiteral[getCommand({ article })];

callback();
```

Enfin si on se remet dans le contexte initial et que l'on retire les variables intermédiaires, on obtient la syntaxe ci-dessous qui représente l'objectif de cet article.

{% codeblock(name="article.ts") %}

```typescript
type ProcessArticle = (dependencies: {
  articleRepository: ArticleRepository;
  logger: Logger;
}) => (parameters: { article: Article }) => Promise<unknown>;
export const processArticle: ProcessArticle =
  ({ articleRepository, logger }) =>
  ({ article }) =>
    ({
      [COMMAND.delete]: () =>
        deleteArticle({ articleRepository, logger })({ article }),
      [COMMAND.unpublish]: () =>
        unpublishArticle({ articleRepository, logger })({ article }),
      [COMMAND.publish]: () =>
        publishArticle({ articleRepository, logger })({ article }),
      [COMMAND.createOrUpdate]: () =>
        createOrUpdateArticle({ articleRepository, logger })({ article }),
      [COMMAND.unknown]: () => {
        throw new Error(
          `Unexpected value for article : ${JSON.stringify(article)}`
        );
      },
    }[getCommand({ article })]());
```

{% end %}

## Conclusion

On y est presque ! Malheureusement, on est contraint de passer par une structure intermédiaire (que j'ai appelé `Command` dans cet exemple) afin de construire notre pattern matching.

Que pensez-vous de cette syntaxe ? Est-ce que Javascript devrait inclure un vrai pattern matching, comme en C# par exemple ?
