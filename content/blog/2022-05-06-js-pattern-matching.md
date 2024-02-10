+++
title = "Pattern Matching in JS?"

[taxonomies]
tags = ["Javascript", "Tutorial", "Programming"]
+++

Unfortunately, it is not (yet) possible in JavaScript, nor in TypeScript. But we can try to get closer to it, especially by separating the identification of a scenario from its execution. The goal is to enhance readability and bring the code closer to the problem it solves. To illustrate this, we will use the example of processing a blog article (adding, deleting, publishing, etc.) based on the information contained in the article. Additionally, I suggest we take it step by step so that you can apply this refactoring in your code.

<!-- more -->

{% advice(type="tip") %}

The code examples presented below are in TypeScript and rely on functional programming. So no classes and no inheritance, but data, functions, and also functions of functions.

{% end %}

## Defining a working context

### Some modeling

Let's start by defining an interface that represents the payload to be processed, in this case, an article. This article has several attributes that indicate whether it should be deleted, unpublished, or created.

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

In addition, let's define two interfaces to improve the typing of our example: a repository and a logger.

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

### A (very) procedural first version

I propose the following implementation for article processing as a starting point for discussion.

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

Before we work on the code structure, let's go through the contents of the `processArticle()` function together to understand how our article is being processed.

To begin with, we can already exclude the logger whose role is to display the different steps in debug mode and anomalies in the article processing in warning mode. Next, we notice that the function handles 5 distinct use cases: deletion, unpublishing, publishing, creating/updating, and an error case. However, even though we understand the code written line by line, we can identify several major issues: the structural elements are drowned among the rest of the code, the function performs multiple actions on its own, and the identification of scenarios is coupled with the execution of these scenarios. This function has two responsibilities (identifying the scenario and the actions to be performed) and violates the Single Responsibility Principle (SRP).

If you're not convinced that this coupling is problematic, try to visualize the impact on the code of the following requirements:

- "Bug: At the time of publication, articles must have content."
- "Feature: Make it impossible to delete published articles."
- "Feature: Allow creation and publication in one go."

Without refactoring, we can see that the code will quickly become difficult to maintain, and it will be increasingly challenging to identify the intention behind the code.

## A short detour into Clean Code

To address some of the issues mentioned earlier, I suggest using a classic technique: decomposing into multiple functions (Clean Code: "Extract till you drop").

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

The first thing we notice is that readability is much better. We now have a healthier foundation to build a pattern matching in the `processArticle()` function. This will allow us to isolate the identification of the scenario so that we can easily add new use cases or change conditions without affecting the rest of the processing.

## Precisely and explicitly identifying the scenario

> Explicit is better than implicit

In our example, we have 5 distinct use cases, so let's explicitly bring out these 5 scenarios. To do this, we can use a simple Enum as follows.

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

Next, we can implement a method whose role is to identify an intention (named `Command` in our example) based on the information contained in the article. We just need to follow the structure of the previous code and return the appropriate use case.

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

## Getting closer to pattern matching

The main trick is to combine the concepts of [literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types#object_literals) and [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE). We will use an object literal as the structure of our pattern matching. The keys of the object correspond to the different use cases, and the associated values are the implementations of these use cases. Notice that we use arrow functions to avoid executing all scenarios when the object is created.

```typescript
const objectLiteral = {
  case1: () => fun1(),
  case2: () => fun2(),
  case3: () => fun3(),
};
```

From an object literal like the one above, the goal is to target the use case and execute the correct callback. I'll provide a very basic illustration.

```typescript
const callback = objectLiteral["case1"];

callback(); // Will call fun1()
```

In the previous step, we defined a function `getCommand()` whose role is to identify the use case. We also defined identifiers for our use cases (the values of `Command`). So, we can replace `case1`, `case2`, and `case3` with `Command` values, and `fun1()`, `fun2()`, and `fun3()` with the functions extracted earlier. This results in:

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

Finally, if we put ourselves back in the initial context and remove the intermediate variables, we get the following syntax, which represents the goal of this article.

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

We're almost there! Unfortunately, we are forced to go through an intermediate structure (which I called `Command` in this example) to build our pattern matching.

What do you think of this syntax? Should JavaScript include real pattern matching, like in C# for example?
