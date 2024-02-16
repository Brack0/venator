+++
title = "Advent of TypeScript 2023"

[taxonomies]
tags = ["Typescript", "Tutoriel", "Programmation"]
+++

Au lieu de participer au populaire Advent of Code, cette année j'ai décidé de relever le défi de l'[Advent of TypeScript](https://typehero.dev/aot-2023). Dans cet article, vous trouverez mes solutions ainsi que quelques détails sur leur fonctionnement ou ce que j'ai appris en résolvant les défis.

<!-- more -->

_Les titres sont subjectifs. Selon l'expérience de chacun, vous pourriez ne pas être d'accord sur la catégorie._

## Problèmes simples

### Jour 1

```ts
type SantasFavoriteCookies = "ginger-bread" | "chocolate-chip";
```

### Jour 2

```ts
type CookieSurveyInput<T> = keyof T;
```

### Jour 3

```ts
type GiftWrapper<T, U, V> = {
    present: T;
    from: U;
    to: V;
};
```

### Jour 4

```ts
type Address = { address: string; city: string };
type PresentDeliveryList<T> = Record<keyof T, Address>;
```

### Jour 5

```ts
type SantasList<T extends readonly unknown[], U extends readonly unknown[]> = [
    ...T,
    ...U,
];
```

### Jour 6

```ts
type FilterChildrenBy<T, U> = Exclude<T, U>;
```

### Jour 8

```ts
type RemoveNaughtyChildren<T> = Omit<T, `naughty_${string}`>;
```

### Jour 10

```ts
type StreetSuffixTester<T, U> = T extends `${string}${U & string}`
    ? true
    : false;
```

### Jour 17

Je gère simplement tous les cas. Pas d'extends, d'inférence, de génériques, d'itérateurs, etc. Parfois, il est plus facile de faire les choses plutôt que de s'appuyer sur des astuces ou des mots-clés avancés.

```ts
type RockPaperScissors = "👊🏻" | "🖐🏾" | "✌🏽";

type WhoWins<Opponent, You> = Opponent extends "👊🏻"
    ? You extends "👊🏻"
        ? "draw"
        : You extends "🖐🏾"
          ? "win"
          : "lose"
    : Opponent extends "🖐🏾"
      ? You extends "👊🏻"
          ? "lose"
          : You extends "🖐🏾"
            ? "draw"
            : "win"
      : You extends "👊🏻"
        ? "win"
        : You extends "🖐🏾"
          ? "lose"
          : "draw";
```

## Il est temps de se réveiller

### Jour 7

Au cours de ce défi, j'ai appris le remappage de clés (avec `as`). Premier jour de l'Advent of TypeScript qui m'a fait retourner à la documentation. Un rappel très utile de consulter parfois la documentation, car malgré l'utilisation quasi quotidienne de TypeScript, j'ai raté certaines nouvelles fonctionnalités.

```ts
type AppendGood<T> = { [K in keyof T as `good_${K & string}`]: T[K] };
```

### Jour 9

La partie la plus difficile de ce défi est de comprendre comment le résoudre techniquement. Tout d'abord, les types récursifs se sont beaucoup améliorés et je suis surpris qu'ils puissent être utilisés si facilement. Je me souviens avoir eu des problèmes avec un grand nombre de récursions et dépassant un peu la capacité de TypeScript. Cela sera vraiment utile plus tard. Deuxième point important, en combinant string literals et `infer`, on peut créer un itérateur sur les caractères d'un string.

```ts
type Reverse<T extends string> = T extends `${infer U}${infer V}`
    ? `${Reverse<V>}${U}`
    : T;
```

### Jour 11

Je n'aime pas totalement ma solution, surtout parce que je m'attendais à une combinaison de types utilitaires (quelque chose comme `type RecursiveReadonly<T, K = keyof T> = Readonly<Record<K, RecursiveReadonly<T[K]>>>`). Je n'ai pas réussi à mettre la partie objet dans un sous-type pour simplifier cela. Je ne sais pas. _Skill issue ?_

```ts
type SantaListProtector<T> = T extends Function
    ? T
    : T extends object
      ? {
            readonly [K in keyof T]: SantaListProtector<T[K]>;
        }
      : T;
```

### Jour 12

Premier défi où j'ai utilisé des vrais noms pour les génériques. Cela me donne une meilleure compréhension de ce que je manipule. Ce n'est pas un vrai `IndexOf` puisque je retourne `never` au lieu de `-1` (pour l'objectif du défi). En remplaçant les string literals par des tableaux, je peux maintenant itérer sur les éléments d'un tableau (en gros `type Iterate<T> = T extends [infer Item, ...infer Rest] ? Iterate<Rest> : never`). Appeler `Arr["length"]` peut produire soit `number` lorsque le tableau est inconnu, soit la vraie valeur si TypeScript peut l'inférer (`['a', "b", 'c']["length"] /* 3 */`), ce qui est plutôt pratique. Dernière chose importante dans ce défi, ajouter des génériques avec une valeur par défaut peut aider à insérer des valeurs dans les récursions.

```ts
type IndexOf<Arr, Item, Checked extends unknown[] = []> = Arr extends [
    infer First,
    ...infer Rest,
]
    ? Item extends First
        ? Checked["length"]
        : IndexOf<Rest, Item, [...Checked, First]>
    : never;

type FindSanta<T> = IndexOf<T, "🎅🏼">;
```

### Jour 13

Sur la base des tests unitaires du défi, je pourrais simplement construire une union avec des nombres de 0 à N, puis exclure 0. Mais comme il y a une valeur de départ, j'ai créé une union de nombres entre les valeurs de début et de fin. Appeler `Arr[number]` produira toutes les valeurs possibles dans le tableau. Cela peut être un seul type (`number`, `string`, `Foo`, etc.) ou une union de toutes les valeurs en fonction de ce que TypeScript peut déduire du tableau. Dans ce défi, je construis une série dans un tableau (chaque case est égale à son indice dans le tableau) pour obtenir une union de toutes les valeurs contenu dans la série. Je peux ensuite exclure les premières valeurs (jusqu'à `Start`) de toutes les valeurs (De 0 à `End`, `End` exclu). Pour finir, on doit rajouter la valeur `End` dans l'union car ma construction via des séries ne l'inclut pas.

```ts
type Enumerate<
    N extends number,
    Acc extends number[] = [],
> = Acc["length"] extends N
    ? Acc[number]
    : Enumerate<N, [...Acc, Acc["length"]]>;

type DayCounter<Start extends number, End extends number> =
    | Exclude<Enumerate<End>, Enumerate<Start>>
    | End;
```

### Jour 14

Assez facile par rapport aux jours précédents. Ce n'est pas si simple, mais je l'ai résolu très rapidement car je suis de plus en plus à l'aise avec des techniques que je n'utilisais pas auparavant, comme `infer`.

```ts
type DecipherNaughtyList<T> = T extends `${infer R}/${infer S}`
    ? R | DecipherNaughtyList<S>
    : T;
```

### Jour 15

Une fonctionnalité de TypeScript réduit la quantité de choses à faire pour ce défi : [les types conditionnels distributifs](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types). Fournir une union de valeurs produira une union de tableaux. Donc la solution est simplement un générateur de tableaux avec n éléments.

```ts
type BoxToys<
    T extends string,
    U extends number,
    Acc extends string[] = [],
> = U extends Acc["length"] ? Acc : BoxToys<T, U, [...Acc, T]>;
```

### Jour 16

Vous vous souvenez du jour 12 ? Là, c'est la suite logique : trouver des coordonnées dans une matrice. Je passe en revue chaque ligne, et dans chaque ligne, je lance l'`IndexOf` du jour 12.

```ts
type FindSanta<T> = MatrixCoordinatesOf<T, "🎅🏼">;

type MatrixCoordinatesOf<T, Item, Acc extends unknown[] = []> = T extends [
    infer Row,
    ...infer Rest,
]
    ? IndexOf<Row, Item> extends -1
        ? MatrixCoordinatesOf<Rest, Item, [...Acc, Row]>
        : [Acc["length"], IndexOf<Row, Item>]
    : never;

type IndexOf<Row, Item, Checked extends unknown[] = []> = Row extends [
    infer Column,
    ...infer Rest,
]
    ? Item extends Column
        ? Checked["length"]
        : IndexOf<Rest, Item, [...Checked, Column]>
    : -1;
```

### Jour 18

Pas si difficile après avoir passé les défis précédents. Je parcours les éléments et j'incrémente un compteur externe sous forme de tableau. À la fin, je retourne simplement la longueur du compteur externe.

```ts
type Count<T, U, Counter extends unknown[] = []> = T extends [
    infer First,
    ...infer Rest,
]
    ? U extends First
        ? Count<Rest, U, [...Counter, First]>
        : Count<Rest, U, Counter>
    : Counter["length"];
```

## Les choses deviennent sérieuses

### Jour 19

Il est temps d'ajouter quelques commentaires en TSDoc et de diviser la réponse en sous parties. J'ai créé un générateur de véhicule (`RideGenerator`) qui produit `"🛹"`, `"🚲"`, `"🛴"` ou `"🏄"` en fonction d'un nombre. Habituellement, on utiliserait un modulo ici pour parcourir les valeurs, ce qui est impossible avec des types (autant que je sache). Ce que j'ai réussi à faire c'est d'ajouter des valeurs de manière récursive jusqu'à ce que l'index demandé soit dans le tableau, et cela a fonctionné. J'ai utilisé 3 tableaux dans le type principal (`Rebuild`) : l'entrée, la sortie et un compteur externe. Comme indiqué dans la TSDoc, nous avons besoin d'un index pour `RideGenerator` et nous ne pouvons pas utiliser l'entrée ou la sortie (la longueur de l'entrée diminuera à chaque récursion, la sortie aura N éléments de plus à chaque récursion), c'est pour ça qu'on utilise un compteur externe.

```ts
type Ride = ["🛹", "🚲", "🛴", "🏄"];

/**
 * Parcours de Ride pour en choisir un.
 *
 * Astuce : Si Acc[index] n'est pas défini, ajoutez des éléments de Ride jusqu'à ce qu'il existe.
 */
type RideGenerator<
    N extends number,
    Acc extends unknown[] = Ride,
> = Acc[N] extends string ? Acc[N] : RideGenerator<N, [...Acc, ...Ride]>;

/**
 * Crée un tableau de N éléments.
 *
 * Astuce : Si Acc["length"] est inférieur à N, répétez avec un élément de plus dans Acc.
 */
type NItems<Item, N, Acc extends unknown[] = []> = N extends Acc["length"]
    ? Acc
    : NItems<Item, N, [...Acc, Item]>;

/**
 * Astuces :
 * - Pour chaque nombre dans le tableau (type N), nous ajoutons N éléments générés.
 * - Utilisation d'un compteur car nous ne pouvons pas utiliser T ou Acc pour suivre l'index actuel, qui est utilisé pour RideGenerator.
 */
type Rebuild<
    T,
    Acc extends unknown[] = [],
    Counter extends unknown[] = [],
> = T extends [infer N, ...infer Rest]
    ? Rebuild<
          Rest,
          [...Acc, ...NItems<RideGenerator<Counter["length"]>, N>],
          [...Counter, N]
      >
    : Acc;
```

### Jour 20

Version courte : découpez l'expression en entrée en lignes, découpez les lignes en caractères, mappez chaque caractère en ASCII art, ajoutez le caractère ASCII à un buffer. Version longue : voir la TSDoc.

```ts
type Letters = {
    A: ["█▀█ ", "█▀█ ", "▀ ▀ "];
    B: ["█▀▄ ", "█▀▄ ", "▀▀  "];
    C: ["█▀▀ ", "█ ░░", "▀▀▀ "];
    E: ["█▀▀ ", "█▀▀ ", "▀▀▀ "];
    H: ["█ █ ", "█▀█ ", "▀ ▀ "];
    I: ["█ ", "█ ", "▀ "];
    M: ["█▄░▄█ ", "█ ▀ █ ", "▀ ░░▀ "];
    N: ["█▄░█ ", "█ ▀█ ", "▀ ░▀ "];
    P: ["█▀█ ", "█▀▀ ", "▀ ░░"];
    R: ["█▀█ ", "██▀ ", "▀ ▀ "];
    S: ["█▀▀ ", "▀▀█ ", "▀▀▀ "];
    T: ["▀█▀ ", "░█ ░", "░▀ ░"];
    Y: ["█ █ ", "▀█▀ ", "░▀ ░"];
    W: ["█ ░░█ ", "█▄▀▄█ ", "▀ ░ ▀ "];
    " ": ["░", "░", "░"];
    ":": ["#", "░", "#"];
    "*": ["░", "#", "░"];
};

type Letter = keyof Letters;
type CaseInsensitive<T extends string> = T | Lowercase<T> | Uppercase<T>;
type AsciiLetterFromCaseInsensitiveChar<Char extends CaseInsensitive<Letter>> =
    Letters[Uppercase<Char>];

/**
 * Construit l'ASCII art.
 *
 * Astuce : "\n" n'est pas une lettre valide donc on découpe l'entrée en lignes et on appelle ToAsciiLines.
 */
type ToAsciiArt<T extends string> = T extends `${infer Row}\n${infer Rest}`
    ? [...ToAsciiLines<Row>, ...ToAsciiArt<Rest>]
    : ToAsciiLines<T>;

/**
 * Construit une ligne d'ASCII art à partir d'une string.
 *
 * Astuces :
 * - On s'assure que chaque caractère dans l'input est une lettre valide (vérification insensible à la casse)
 * - On crée des lignes ASCII avec les lettres ASCII que nous avons obtenues à partir du caractère (avec AsciiLetterFromCaseInsensitiveChar<Char>)
 */
type ToAsciiLines<
    Str extends string,
    AsciiLines extends [string, string, string] = ["", "", ""],
> = Str extends `${infer Char extends CaseInsensitive<Letter>}${infer Rest}`
    ? ToAsciiLines<
          Rest,
          AppendChar<AsciiLetterFromCaseInsensitiveChar<Char>, AsciiLines>
      >
    : AsciiLines;

/**
 * Ajoute un caractère ASCII à une ligne d'ASCII art avec un tableau contenant 3 string de même longueur.
 *
 * Astuce : Identique à la concaténation de deux vecteurs de 3 coordonnées mais avec des string au lieu de coordonnées, et concaténation de string au lieu d'addition.
 */
type AppendChar<
    AsciiChar extends [string, string, string],
    AsciiLines extends [string, string, string] = ["", "", ""],
> = AsciiChar extends [
    `${infer Char0}${infer Rest0}`,
    `${infer Char1}${infer Rest1}`,
    `${infer Char2}${infer Rest2}`,
]
    ? AppendChar<
          [Rest0, Rest1, Rest2],
          [
              `${AsciiLines[0]}${Char0}`,
              `${AsciiLines[1]}${Char1}`,
              `${AsciiLines[2]}${Char2}`,
          ]
      >
    : AsciiLines;
```

### Jour 21

Je pense pouvoir faire mieux pour ce défi. Je me suis arrêté à _"First, make it work."_. Comme il y a beaucoup de types à définir, j'ai divisé ce problème en trois parties majeures : "Peut-on jouer ce coup ?", "Mise à jour du plateau" et "Mise à jour de l'état du jeu". J'ai déplacé les types dans des `namespace` après avoir résolu le défi pour le rendre plus lisible. J'ai essayé d'utiliser `never` avec des unions pour simplifier de nombreuses expressions, mais comme _tout_ étend `never`, j'ai obtenu beaucoup de résultats indésirables (comme `"❌" | "⭕ Won"`). Il est temps d'utiliser les types `true` et `false`.

#### Coups interdits

```ts
/**
 * Vérifie si c'est la fin du jeu ou si la cellule est déjà remplie
 */
type ForbiddenMove<
    Game extends TicTacToeGame,
    Position extends TicTacToePositions,
> = Game["state"] extends TicTacToeEndState
    ? true
    : BoardValueAtPosition<Game["board"], Position> extends TicTacToeChip
      ? true
      : false;

/**
 * Mapper la position à la cellule dans le tableau (plus facile que de diviser le string + résoudre la ligne et la colonne)
 */
type BoardValueAtPosition<
    Board extends TicTacToeBoard,
    Position extends TicTacToePositions,
> = {
    "top-center": Board[0][1];
    "top-left": Board[0][0];
    "top-right": Board[0][2];
    "middle-center": Board[1][1];
    "middle-left": Board[1][0];
    "middle-right": Board[1][2];
    "bottom-center": Board[2][1];
    "bottom-left": Board[2][0];
    "bottom-right": Board[2][2];
}[Position];
```

#### Mise à jour du plateau après un coup

```ts
/**
 * Parcours des lignes et mise à jour de celle qui est pertinente.
 */
type ComputeBoard<
    Board extends TicTacToeBoard,
    positions extends TicTacToePositions,
    Value extends TicTacToeState,
> = [
    positions extends `top-${infer XPosition extends TicTacToeXPositions}`
        ? UpdateCell<Board[0], XPosition, Value>
        : Board[0],
    positions extends `middle-${infer XPosition extends TicTacToeXPositions}`
        ? UpdateCell<Board[1], XPosition, Value>
        : Board[1],
    positions extends `bottom-${infer XPosition extends TicTacToeXPositions}`
        ? UpdateCell<Board[2], XPosition, Value>
        : Board[2],
];

/**
 * Parcours des cellules de la ligne et mise à jour de celle qui est pertinente.
 */
type UpdateCell<
    Row extends TicTacToeCell[],
    XPosition extends TicTacToeXPositions,
    Value extends TicTacToeState,
> = [
    XPosition extends `left` ? Value : Row[0],
    XPosition extends `center` ? Value : Row[1],
    XPosition extends `right` ? Value : Row[2],
];
```

#### Mise à jour de l'état (en se basant sur le nouveau plateau)

```ts
/**
 * Renvoie l'état final si le jeu est terminé, sinon renvoie le joueur suivant
 */
type ComputeState<
    Board extends TicTacToeBoard,
    State extends TicTacToeState,
> = IsEnd<Board> extends string
    ? IsEnd<Board>
    : State extends "❌"
      ? "⭕"
      : "❌";

/**
 * Vérifie si l'un des joueurs a gagné ou si c'est une égalité.
 */
type IsEnd<Board extends TicTacToeBoard> = Win<Board, "❌"> extends true
    ? "❌ Won"
    : Win<Board, "⭕"> extends true
      ? "⭕ Won"
      : Draw<Board>;

/**
 * Un match nul est défini par un plateau plein. Je pars du principe que les vérifications de victoire ont été effectuées auparavant.
 *
 * Anecdote : On peut déduire un match nul sans avoir un plateau complet car il existe des cas où, quoi que fassent les joueurs, aucun d'entre eux ne peut gagner.
 * Cela peut être très complexe à vérifier avec des types et ce n'est pas dans le défi, j'ai donc ignoré cette partie.
 */
type Draw<Board extends TicTacToeBoard> = Board extends [
    [TicTacToeChip, TicTacToeChip, TicTacToeChip],
    [TicTacToeChip, TicTacToeChip, TicTacToeChip],
    [TicTacToeChip, TicTacToeChip, TicTacToeChip],
]
    ? "Draw"
    : false;

/**
 * Soit une victoire sur une colonne, une ligne ou une diagonale.
 *
 * A utiliser avec soit `"❌"` ou `"⭕"` mais pas avec l'union `"❌" | "⭕"`.
 */
type Win<Board extends TicTacToeBoard, Chip extends TicTacToeChip> = ColumnWin<
    Board,
    Chip
> &
    RowWin<Board, Chip> &
    DiagonalWin<Board, Chip> extends true
    ? true
    : false;

/**
 * Vérifie si une des lignes est gagnante.
 *
 * A utiliser avec soit `"❌"` ou `"⭕"` mais pas avec l'union `"❌" | "⭕"`.
 */
type RowWin<
    Board extends TicTacToeBoard,
    Chip extends TicTacToeChip,
> = Board[0] extends [Chip, Chip, Chip]
    ? true
    : Board[1] extends [Chip, Chip, Chip]
      ? true
      : Board[2] extends [Chip, Chip, Chip]
        ? true
        : false;

/**
 * Vérifie si une des colonnes est gagnante. Je pourrais reconstruire une colonne et ensuite vérifier si elle est gagnante
 * (c'est-à-dire `[Board[0][0], Board[1][0], Board[2][0]] extends [Chip, Chip, Chip]`), mais j'ai plutôt utilisé des techniques de matrice.
 * La vérification d'une colonne est équivalente à la vérification d'une sur une matrice inversée.
 *
 * A utiliser avec soit `"❌"` ou `"⭕"` mais pas avec l'union `"❌" | "⭕"`.
 */
type ColumnWin<
    Board extends TicTacToeBoard,
    Chip extends TicTacToeChip,
> = RowWin<Invert<Board>, Chip>;

/**
 * Inversion du plateau de jeu
 */
type Invert<Board extends TicTacToeBoard> = [
    [Board[0][0], Board[1][0], Board[2][0]],
    [Board[0][1], Board[1][1], Board[2][1]],
    [Board[0][2], Board[1][2], Board[2][2]],
];

/**
 * Construit 2 tableaux avec les valeurs diagonales et vérifie si l'une d'elles est une diagonale gagnante.
 *
 * A utiliser avec soit `"❌"` ou `"⭕"` mais pas avec l'union `"❌" | "⭕"`.
 */
type DiagonalWin<Board extends TicTacToeBoard, Chip extends TicTacToeChip> = [
    Board[0][0],
    Board[1][1],
    Board[2][2],
] extends [Chip, Chip, Chip]
    ? true
    : [Board[0][2], Board[1][1], Board[2][0]] extends [Chip, Chip, Chip]
      ? true
      : false;
```

#### Ma solution

```ts
type TicTacToeChip = "❌" | "⭕";
type TicTacToeEndState = "❌ Won" | "⭕ Won" | "Draw";
type TicTacToeState = TicTacToeChip | TicTacToeEndState;
type TicTacToeEmptyCell = "  ";
type TicTacToeCell = TicTacToeChip | TicTacToeEmptyCell;
type TicTacToeYPositions = "top" | "middle" | "bottom";
type TicTacToeXPositions = "left" | "center" | "right";
type TicTacToePositions = `${TicTacToeYPositions}-${TicTacToeXPositions}`;
type TicTacToeBoard = TicTacToeCell[][];
type TicTacToeGame = {
    board: TicTacToeBoard;
    state: TicTacToeState;
};

type EmptyBoard = [["  ", "  ", "  "], ["  ", "  ", "  "], ["  ", "  ", "  "]];

type NewGame = {
    board: EmptyBoard;
    state: "❌";
};

/**
 * Peut-on jouer ce coup ? Si oui, mise à jour du plateau et mise à jour de l'état. Sinon, on retourne le jeu tel quel.
 */
type TicTacToe<
    Game extends TicTacToeGame,
    Positions extends TicTacToePositions,
> = Rules.ForbiddenMove<Game, Positions> extends false
    ? GameBoard.ComputeBoard<
          Game["board"],
          Positions,
          Game["state"]
      > extends infer GameBoardUpdated
        ? GameState.ComputeState<
              GameBoardUpdated,
              Game["state"]
          > extends infer GameStateUpdated
            ? {
                  board: GameBoardUpdated;
                  state: GameStateUpdated;
              }
            : never
        : never
    : Game;

namespace Utils {
    /**
     * Inversion du plateau de jeu
     */
    export type Invert<Board extends TicTacToeBoard> = [
        [Board[0][0], Board[1][0], Board[2][0]],
        [Board[0][1], Board[1][1], Board[2][1]],
        [Board[0][2], Board[1][2], Board[2][2]],
    ];

    /**
     * Mapper la position à la cellule dans le tableau (plus facile que de diviser le string + résoudre la ligne et la colonne)
     */
    export type BoardValueAtPosition<
        Board extends TicTacToeBoard,
        Position extends TicTacToePositions,
    > = {
        "top-center": Board[0][1];
        "top-left": Board[0][0];
        "top-right": Board[0][2];
        "middle-center": Board[1][1];
        "middle-left": Board[1][0];
        "middle-right": Board[1][2];
        "bottom-center": Board[2][1];
        "bottom-left": Board[2][0];
        "bottom-right": Board[2][2];
    }[Position];
}

namespace Rules {
    /**
     * Vérifie si c'est la fin du jeu ou si la cellule est déjà remplie
     */
    export type ForbiddenMove<
        Game extends TicTacToeGame,
        Position extends TicTacToePositions,
    > = Game["state"] extends TicTacToeEndState
        ? true
        : Utils.BoardValueAtPosition<
                Game["board"],
                Position
            > extends TicTacToeChip
          ? true
          : false;
}

namespace GameState {
    /**
     * Renvoie l'état final si le jeu est terminé, sinon renvoie le joueur suivant
     */
    export type ComputeState<
        Board extends TicTacToeBoard,
        State extends TicTacToeState,
    > = IsEnd<Board> extends string
        ? IsEnd<Board>
        : State extends "❌"
          ? "⭕"
          : "❌";

    /**
     * Vérifie si l'un des joueurs a gagné ou si c'est une égalité.
     */
    type IsEnd<Board extends TicTacToeBoard> = Win<Board, "❌"> extends true
        ? "❌ Won"
        : Win<Board, "⭕"> extends true
          ? "⭕ Won"
          : Draw<Board>;

    /**
     * Un match nul est défini par un plateau plein. Je pars du principe que les vérifications de victoire ont été effectuées auparavant.
     *
     * Anecdote : On peut déduire un match nul sans avoir un plateau complet car il existe des cas où, quoi que fassent les joueurs, aucun d'entre eux ne peut gagner.
     * Cela peut être très complexe à vérifier avec des types et ce n'est pas dans le défi, j'ai donc ignoré cette partie.
     */
    type Draw<Board extends TicTacToeBoard> = Board extends [
        [TicTacToeChip, TicTacToeChip, TicTacToeChip],
        [TicTacToeChip, TicTacToeChip, TicTacToeChip],
        [TicTacToeChip, TicTacToeChip, TicTacToeChip],
    ]
        ? "Draw"
        : false;

    /**
     * Soit une victoire sur une colonne, une ligne ou une diagonale.
     *
     * A utiliser avec soit `"❌"` ou `"⭕"` mais pas avec l'union `"❌" | "⭕"`.
     */
    type Win<
        Board extends TicTacToeBoard,
        Chip extends TicTacToeChip,
    > = ColumnWin<Board, Chip> &
        RowWin<Board, Chip> &
        DiagonalWin<Board, Chip> extends true
        ? true
        : false;

    /**
     * Vérifie si une des lignes est gagnante.
     *
     * A utiliser avec soit `"❌"` ou `"⭕"` mais pas avec l'union `"❌" | "⭕"`.
     */
    type RowWin<
        Board extends TicTacToeBoard,
        Chip extends TicTacToeChip,
    > = Board[0] extends [Chip, Chip, Chip]
        ? true
        : Board[1] extends [Chip, Chip, Chip]
          ? true
          : Board[2] extends [Chip, Chip, Chip]
            ? true
            : false;

    /**
     * Vérifie si une des colonnes est gagnante. Je pourrais reconstruire une colonne et ensuite vérifier si elle est gagnante
     * (c'est-à-dire `[Board[0][0], Board[1][0], Board[2][0]] extends [Chip, Chip, Chip]`), mais j'ai plutôt utilisé des techniques de matrice.
     * La vérification d'une colonne est équivalente à la vérification d'une sur une matrice inversée.
     *
     * A utiliser avec soit `"❌"` ou `"⭕"` mais pas avec l'union `"❌" | "⭕"`.
     */
    type ColumnWin<
        Board extends TicTacToeBoard,
        Chip extends TicTacToeChip,
    > = RowWin<Utils.Invert<Board>, Chip>;

    /**
     * Construit 2 tableaux avec les valeurs diagonales et vérifie si l'une d'elles est une diagonale gagnante.
     *
     * A utiliser avec soit `"❌"` ou `"⭕"` mais pas avec l'union `"❌" | "⭕"`.
     */
    type DiagonalWin<
        Board extends TicTacToeBoard,
        Chip extends TicTacToeChip,
    > = [Board[0][0], Board[1][1], Board[2][2]] extends [Chip, Chip, Chip]
        ? true
        : [Board[0][2], Board[1][1], Board[2][0]] extends [Chip, Chip, Chip]
          ? true
          : false;
}

namespace GameBoard {
    /**
     * Parcours des lignes et mise à jour de celle qui est pertinente.
     */
    export type ComputeBoard<
        Board extends TicTacToeBoard,
        positions extends TicTacToePositions,
        Value extends TicTacToeState,
    > = [
        positions extends `top-${infer XPosition extends TicTacToeXPositions}`
            ? UpdateCell<Board[0], XPosition, Value>
            : Board[0],
        positions extends `middle-${infer XPosition extends
            TicTacToeXPositions}`
            ? UpdateCell<Board[1], XPosition, Value>
            : Board[1],
        positions extends `bottom-${infer XPosition extends
            TicTacToeXPositions}`
            ? UpdateCell<Board[2], XPosition, Value>
            : Board[2],
    ];

    /**
     * Parcours des cellules de la ligne et mise à jour de celle qui est pertinente.
     */
    type UpdateCell<
        Row extends TicTacToeCell[],
        XPosition extends TicTacToeXPositions,
        Value extends TicTacToeState,
    > = [
        XPosition extends `left` ? Value : Row[0],
        XPosition extends `center` ? Value : Row[1],
        XPosition extends `right` ? Value : Row[2],
    ];
}
```

## Pas le temps pour les derniers défis

Désolé pour vous, chers adeptes de TypeScript, mais je n'ai pas eu le temps de terminer les derniers jours. Et c'est ce que je n'aime pas dans ce type d'événements (Advent of [X]). Proposer des défis de plus en plus difficiles à résoudre est vraiment cool et on peut ressentir sa progression au fil des jours. Ce qui me dérange vraiment, c'est que les parties les plus difficiles surviennent lorsque l'on a le moins de temps pour les résoudre. Je m'attends à passer des heures sur les derniers défis, et je ne peux certainement pas les résoudre le jour même (problème d'agenda personnel). Je tenterai peut-être après la période des fêtes car ils ont l'air cool à résoudre.

## Conclusion

C'était une super aventure, j'ai adoré ce premier Advent of TypeScript. J'ai appris des choses importantes. Cependant, la plupart des types que j'ai écrits devraient rester à des fins de défi uniquement. Dériver des types à partir d'autres types et ainsi de suite peut ajouter un couplage au niveau typage. Une simple mise à jour d'un type (comme ajouter une valeur dans une union) et tout peut exploser. Une meilleure approche est de laisser TypeScript inférer les types en fonction des instructions (et de lui donner un peu d'aide quand il ne fait pas le travail, je pense à `Object.keys()` et ses amis). Quoi qu'il en soit, je vais certainement essayer l'Advent of TypeScript l'année prochaine. Il est à noter que la plupart des défis peuvent être résolus beaucoup plus rapidement que ceux de l'Advent of Code, ce qui m'arrangeait bien cette année.
