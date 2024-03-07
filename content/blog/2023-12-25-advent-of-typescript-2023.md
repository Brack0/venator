+++
title = "Advent of TypeScript 2023"

[taxonomies]
tags = ["Typescript", "Dev"]
+++

Instead of doing the popular Advent of Code, this year I tried to complete the [Advent of TypeScript](https://typehero.dev/aot-2023). In this article, you'll find my solutions and some details on how they work or what I learned while solving the challenge.

<!-- more -->

_Titles are subjective. Based on one's experience, you may disagree on rankings._

## Straightforward problems

### Day 1

```ts
type SantasFavoriteCookies = "ginger-bread" | "chocolate-chip";
```

### Day 2

```ts
type CookieSurveyInput<T> = keyof T;
```

### Day 3

```ts
type GiftWrapper<T, U, V> = {
    present: T;
    from: U;
    to: V;
};
```

### Day 4

```ts
type Address = { address: string; city: string };
type PresentDeliveryList<T> = Record<keyof T, Address>;
```

### Day 5

```ts
type SantasList<T extends readonly unknown[], U extends readonly unknown[]> = [
    ...T,
    ...U,
];
```

### Day 6

```ts
type FilterChildrenBy<T, U> = Exclude<T, U>;
```

### Day 8

```ts
type RemoveNaughtyChildren<T> = Omit<T, `naughty_${string}`>;
```

### Day 10

```ts
type StreetSuffixTester<T, U> = T extends `${string}${U & string}`
    ? true
    : false;
```

### Day 17

I just handle all the cases. No extends, infer, generics, iterator, etc. Sometimes it's easier to do the thing rather than rely on some tricks or advanced keywords.

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

## Time to wake up

### Day 7

During this challenge, I learned Key Remapping (with `as`). First day of the Advent of TypeScript that made me go back to the documentation. A very nice reminder to check the documentation sometimes, despite using TypeScript almost everyday I missed some features in updates.

```ts
type AppendGood<T> = { [K in keyof T as `good_${K & string}`]: T[K] };
```

### Day 9

Hardest part of this challenge is to figure out how to technically solve it. First, recursive types got a lot better and I'm surprised that it can be used so easily. I remember having troubles with high number of recursion and kinda overflowing the capacity of TypeScript. This will really be useful later. Second, by using string literals type with infer you can create a iterator on characters in a string.

```ts
type Reverse<T extends string> = T extends `${infer U}${infer V}`
    ? `${Reverse<V>}${U}`
    : T;
```

### Day 11

I don't like my solution, mostly because I was expecting a combination of utility types (something like `type RecursiveReadonly<T, K = keyof T> = Readonly<Record<K, RecursiveReadonly<T[K]>>>`). I didn't manage to put the object part in a subtype to simplify this. I don't know. Skill issue ?

```ts
type SantaListProtector<T> = T extends Function
    ? T
    : T extends object
      ? {
            readonly [K in keyof T]: SantaListProtector<T[K]>;
        }
      : T;
```

### Day 12

First challenge where I used real name for generics. It gives me more understanding on what I'm manipulating. It's not a real `IndexOf` since I return `never` instead of `-1` (for the challenge purpose). By replacing string literals with arrays, I can now iterate over items in an array (basically this `type Iterate<T> = T extends [infer Item, ...infer Rest] ? Iterate<Rest> : never`). Calling `Arr["length"]` can either produce `number` when the array is unknown, or the real value if TypeScript can infer it (`['a', "b", "c"]["length"] /* 3 */`), which is pretty nice. Last important thing in this challenge, adding more generics with a default value can help insert values in recursions.

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

### Day 13

Based on the unit tests, I could just build an union with numbers from 0 to N, and then exclude 0. But since there is a starting value, I created an union of numbers between starting and ending values. Calling `Arr[number]` will produce all the possible values in the array. It can be a single type (`number`, `string`, `Foo`, etc) or an union type of all values depending on what TypeScript can infer from the array. In this challenge, I'm building a range in an array (i.e. in each cell, value equals array index) to get a union of all values in that range. Then I can exclude the first values (until `Start`) from all values (From 0 to `End`, End excluded). In the range I built, `End` value is excluded so we just need to append it to the union.

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

### Day 14

Pretty easy compared to previous days. It's not that simple but I solved it really fast as I get better and better in technics I wasn't using before, such as `infer`.

```ts
type DecipherNaughtyList<T> = T extends `${infer R}/${infer S}`
    ? R | DecipherNaughtyList<S>
    : T;
```

### Day 15

One feature of TypeScript reduce the amount of things to do in this challenge : [Distributive Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types). Giving an union of values will produce an union of arrays. So the solution is just a generator of array with n items.

```ts
type BoxToys<
    T extends string,
    U extends number,
    Acc extends string[] = [],
> = U extends Acc["length"] ? Acc : BoxToys<T, U, [...Acc, T]>;
```

### Day 16

Remember day 12 ? This is the next level : matrix coordinates. I go over each rows, and in each row I run the `IndexOf` of day 12.

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

### Day 18

Not that hard once I have done all previous challenges. I go over items and increment an external counter. At the end, I just return the length of the external counter.

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

## Things are getting serious

### Day 19

Time to add some TSDoc and split the answer in small parts. I made a RideGenerator that produce `"🛹"`, `"🚲"`, `"🛴"` or `"🏄"` based on a number. We would usually use a modulo here to loop over values, which is impossible in type definition (as far as I know). What I managed to do is to append values recursively until the index is in the array and that worked. I used 3 arrays in the main type (`Rebuild`) : the input, the output and an external counter. As said in the TSdoc, we need an index for RideGenerator and we cannot use input or output (input's length will decrement in each recursion, output will get N more items in each recursion), so I used the length of an external counter.

```ts
type Ride = ["🛹", "🚲", "🛴", "🏄"];

/**
 * Loop over Ride to choose one.
 *
 * Trick : If Acc[index] is out of bounds, append Ride items until it's defined
 */
type RideGenerator<
    N extends number,
    Acc extends unknown[] = Ride,
> = Acc[N] extends string ? Acc[N] : RideGenerator<N, [...Acc, ...Ride]>;

/**
 * Create an array of N items.
 *
 * Trick : If Acc["length"] is less than N, repeat with one more item in Acc.
 */
type NItems<Item, N, Acc extends unknown[] = []> = N extends Acc["length"]
    ? Acc
    : NItems<Item, N, [...Acc, Item]>;

/**
 * Tricks :
 * - For each number in array (type N), we add N generated items.
 * - Using a Counter since we cannot use T or Acc to track current index, which is used for RideGenerator.
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

### Day 20

Short explanation : split string in lines, split lines in characters, map character to ASCII art, append ASCII art character to a buffer. Long explanation : see TSDoc.

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
 * Build the ASCII art.
 *
 * Trick : "\n" is not a valid letter. Split the input into rows and delegate to ToAsciiLines.
 */
type ToAsciiArt<T extends string> = T extends `${infer Row}\n${infer Rest}`
    ? [...ToAsciiLines<Row>, ...ToAsciiArt<Rest>]
    : ToAsciiLines<T>;

/**
 * Build an ASCII art line from string input.
 *
 * Tricks :
 * - Make sure that every Char in string input is a valid Letter (case insensitive check)
 * - Create ASCII lines with ASCII Letters we got from Char (with AsciiLetterFromCaseInsensitiveChar<Char>)
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
 * Append an ASCII character to an ASCII art line with array containing 3 strings of the same length.
 *
 * Trick : Same as concatenation of two vectors of 3 coordinates but with strings instead of coordinates, and string concatenation instead of addition.
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

### Day 21

I'm pretty sure I can do better types for this challenge. I stopped at _"First, make it work."_. As many things should be done, I break this problem in three major parts : "Can we do this move ?", "Update the board" and "Update the game state". I moved types in `namespace` after I solved the challenge to make it more readable. I tried to use `never` with unions to simplify many expressions but since everything extends `never`, I got a lot of unwanted results (such as `"❌" | "⭕ Won"`). Time to use `true` and `false` types.

#### Forbidden moves

```ts
/**
 * Check if it's the end or if the cell is already filled
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
 * Map position to cell in board (easier than splitting string + resolve row and column)
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

#### Update board after a move

```ts
/**
 * Go over rows and update the relevant one.
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
 * Go over cells in row and update the relevant one.
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

#### Update state (based on new board)

```ts
/**
 * Return end state if the game ended, else return the next player
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
 * Check if either player has won or if it's a draw.
 */
type IsEnd<Board extends TicTacToeBoard> = Win<Board, "❌"> extends true
    ? "❌ Won"
    : Win<Board, "⭕"> extends true
      ? "⭕ Won"
      : Draw<Board>;

/**
 * A draw is defined by a full board. I assume that winning checks have been done before.
 *
 * Trivia : We can infer a draw without having a full board because sometimes, no matter what players will do, none of them can win.
 * It can be very complex to check in types and it's not in the challenge so I ignored this part.
 */
type Draw<Board extends TicTacToeBoard> = Board extends [
    [TicTacToeChip, TicTacToeChip, TicTacToeChip],
    [TicTacToeChip, TicTacToeChip, TicTacToeChip],
    [TicTacToeChip, TicTacToeChip, TicTacToeChip],
]
    ? "Draw"
    : false;

/**
 * Either a win with a column, a row or a diagonal.
 *
 * Call this type with either `"❌"` or `"⭕"` but not with the union `"❌" | "⭕"`.
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
 * Check all rows if one of them is a winning row.
 *
 * Call this type with either `"❌"` or `"⭕"` but not with the union `"❌" | "⭕"`.
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
 * Check all columns if one of them is a winning column. I could rebuild a column and then check
 * (ie. `[Board[0][0], Board[1][0], Board[2][0]] extends [Chip, Chip, Chip]`), but instead I relied on matrix stuff.
 * Column check is equivalent to row check on inverted matrix.
 *
 * Call this type with either `"❌"` or `"⭕"` but not with the union `"❌" | "⭕"`.
 */
type ColumnWin<
    Board extends TicTacToeBoard,
    Chip extends TicTacToeChip,
> = RowWin<Invert<Board>, Chip>;

/**
 * Matrix invertion on game board
 */
type Invert<Board extends TicTacToeBoard> = [
    [Board[0][0], Board[1][0], Board[2][0]],
    [Board[0][1], Board[1][1], Board[2][1]],
    [Board[0][2], Board[1][2], Board[2][2]],
];

/**
 * Build 2 arrays with diagonal values and check if one of them is a winning diagonal.
 *
 * Call this type with either `"❌"` or `"⭕"` but not with the union `"❌" | "⭕"`.
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

#### My solution

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
 * Can we do this move ? If so, update the board and update the state else, return the game state.
 */
type TicTacToe<
    Game extends TicTacToeGame,
    Positions extends TicTacToePositions,
> = Rules.ForbiddenMove<Game, Positions> extends false
    ? GameBoard.ComputeBoard<
          Game["board"],
          Positions,
          Game["state"]
      > extends infer GameBoardUpdated extends TicTacToeBoard
        ? GameState.ComputeState<
              GameBoardUpdated,
              Game["state"]
          > extends infer GameStateUpdated extends TicTacToeState
            ? {
                  board: GameBoardUpdated;
                  state: GameStateUpdated;
              }
            : never
        : never
    : Game;

namespace Utils {
    /**
     * Matrix invertion on game board
     */
    export type Invert<Board extends TicTacToeBoard> = [
        [Board[0][0], Board[1][0], Board[2][0]],
        [Board[0][1], Board[1][1], Board[2][1]],
        [Board[0][2], Board[1][2], Board[2][2]],
    ];

    /**
     * Map position to cell in board (easier than splitting string + resolve row and column)
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
     * Check if end state or if the cell is already filled
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
     * Return end state if the game ended, else return the next player
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
     * Check if either player has won or if it's a draw.
     */
    type IsEnd<Board extends TicTacToeBoard> = Win<Board, "❌"> extends true
        ? "❌ Won"
        : Win<Board, "⭕"> extends true
          ? "⭕ Won"
          : Draw<Board>;

    /**
     * A draw is defined by a full board. I assume that winning checks have been done before.
     *
     * Trivia : We can infer a draw without having a full board because no matter what players will do, none of them can win.
     * It can be very complex to check in types and it's not in the challenge => I ignored this part.
     */
    type Draw<Board extends TicTacToeBoard> = Board extends [
        [TicTacToeChip, TicTacToeChip, TicTacToeChip],
        [TicTacToeChip, TicTacToeChip, TicTacToeChip],
        [TicTacToeChip, TicTacToeChip, TicTacToeChip],
    ]
        ? "Draw"
        : false;

    /**
     * Either a win with a column, a row or a diagonal.
     *
     * Call this type with either `"❌"` or `"⭕"` but not with the union `"❌" | "⭕"`.
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
     * Check all rows if one of them is a winning row.
     *
     * Call this type with either `"❌"` or `"⭕"` but not with the union `"❌" | "⭕"`.
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
     * Check all columns if one of them is a winning column. I could rebuild a column and then check
     * (ie. `[Board[0][0], Board[1][0], Board[2][0]] extends [Chip, Chip, Chip]`), but instead I relied on matrix stuff.
     * Column check is equivalent to row check on inverted matrix.
     *
     * Call this type with either `"❌"` or `"⭕"` but not with the union `"❌" | "⭕"`.
     */
    type ColumnWin<
        Board extends TicTacToeBoard,
        Chip extends TicTacToeChip,
    > = RowWin<Utils.Invert<Board>, Chip>;

    /**
     * Build 2 arrays with diagonal values and check if one of them is a winning diagonal.
     *
     * Call this type with either `"❌"` or `"⭕"` but not with the union `"❌" | "⭕"`.
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
     * Go over rows and update the relevant one.
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
     * Go over cells in a row and update the relevant one.
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

## No time for these challenges

Sorry for you, fellow TypeScripters, but I didn't have time to complete the last days. And that's what I dislike about these kind of events (Advent of [X]). Provide harder and harder challenges to solve is really cool and you can feel your progression over days. What really grinds my gears is that hardest parts are when you have the least amount of time to do it. I expect spending hours on last challenges, and I definitly can't solve it the same day (personal agenda issue). I may give it a try after the holiday season. They look cool to solve by the way.

## Conclusion

It's a great journey and I loved this first Advent of TypeScript. I learned valuable things in TypeScript. However, most of the types I've written should remain for the challenge purpose only. Deriving types from other types and so one can add coupling at type level. Update one simple type (such a adding a value in an union) and everything can blow up. A better approach is to let TypeScript infer types based on the instructions (and giving him some help when he is not doing the job, looking at you `Object.keys()` and your friends). Anyway, I will definitly try the advent of TypeScript next year. It's worth noticing that most of challenges can be solved way faster than those from the advent of code, which was perfect for me this year.
