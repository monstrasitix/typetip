# TypeTip

Collection of ideas attempting to convey the importance of types. With each scenario there's a rundown of points which may ease one's experience with TypeScript.

Remember....

---
## Modules should remain cohesive

The task is to develop a browser variant of Poker. Game is played from two to ten players per session. Their deck and moves are persisted on the server via web sockets.

Through out the game your system must maintain a valid deck of cards, partially split between N number of players with random preconditions. Players may fold, bet all or even disconnect.

#### Sprint

Define your cards. We're aware of them being enumerable per suit and value. There're is finite amount of combinations under your control.

```TypeScript
type Deck = Card[];
type Card = [CardSuit, CardValue];

type CardSuit = "spade" | "club" | "heart" | "diamond";
type CardValue = "ace" | "two" | "three" | "..." | "hack" | "queen" | "king";
```

We've modeled a deck of cards. We must segregate our modules from the real world, web sockets, to guarantee cohesiveness and our business logic. With these types we can develop following functions.

```TypeScript
/**
 * Suits are exhaustive and we don't need a default case because we modeled
 * a function around the type. Return type "string" is optional.
 */
function suitToString(suit: CardSuit): string {
	switch (suit) {
		case "spade":   return "♠";
		case "club":    return "♣";
		case "heart":   return "♡";
		case "diamond": return "♢";
	}
}

// ---

/**
 * We can preform basic oeprations over types insde the module
 * and expose them when needed. It's easier to maintain features
 * inside few files than across few folders.
 */
function cardToString([suit, value]: Card): string {
	return `[ ${suitToString(suit)} ${value} ]`;
}

// --

/**
 * There needs to be a balance between types and real JavaScript values.
 * I proposed strings for types because they provide more constraint and
 * work without enums.
 */
function createDeck(): Deck {
	let deck: Deck = [];

	const values: CardValue[] = [
		"ace", /* .. */ "queen", "king",
	];

	for (const value of values) {
		deck = deck.concat([
			["spade",   value],
			["club",    value],
			["heart",   value],
			["diamond", value],
		]);
	}

	return deck;
}

// Execution
createDeck.map(cardToString);

const heartSuit: CardSuit = "heart";

cardToString(["heart", "ace"]);      // ✔
cardToString([heartSuit, "three"]);  // ✔
cardToString([suit.heart, 3]);       // x - 3 is not "three"
```

Notice how we're not dealing with external APIs or value absence like `null` or `undefined`. They may well be part of a data structure or a DTO object, but it's consolidated within a module that preforms and serves a purpose.

Likewise we would model a module for other instances including: chips, network requests, errors, users. etc.

All module objects can be simple structs and operations upon them. We should avoid modules like `thunkFnctions.ts`, `constants.ts` because they may never be cohesive or serve a specific purpose.

---
## Delegate unknown values to generics

At times it's acceptable to type values as  `any` when the type is `unknown`. You've probably came across undocumented schemas or loose APIs.

As a developer or consumer of such engineering, it's your responsibility to retain your intentions of the software you're modeling.

#### Sprint

Examine your payload and conclude what your software may need to consume. There may be a contract where some values may be missing or `null` by intent. Your types must be truthful to a scenario and we can always create more.

```TypeScript
// X
type UploadHandler = (event: Event, content: any) => void;

// ✔
// Letter "T" is a general convention for a type argument.
type UploadHandler<T> = (event: Event, content: T) => void;
```

IDE suggestions will be enabled and we can guarantee correct type upon use.
