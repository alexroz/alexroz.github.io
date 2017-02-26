---
layout: post
title: Change to string literal types and type narrowing in TypeScript 2.1
---

{{ page.title }}
================

I was caught out by a change between TypeScript 2.0 and 2.1 to do with how string literal types are narrowed.

I was using the following approach, taken from the ngrx example app at the time of TypeScript 2.0.10.

```typescript
export const ActionTypes = {
  SEARCH:           type('[Book] Search'),
  SEARCH_COMPLETE:  type('[Book] Search Complete'),
  LOAD:             type('[Book] Load'),
  SELECT:           type('[Book] Select'),
};

export class SearchAction implements Action {
  type = ActionTypes.SEARCH;

  constructor(public payload: string) { }
}

export class SearchCompleteAction implements Action {
  type = ActionTypes.SEARCH_COMPLETE;

  constructor(public payload: Book[]) { }
}

...
```

followed by the reducer code:

```typescript

 switch (action.type) {
    case book.ActionTypes.SEARCH_COMPLETE: {
      const books = action.payload;
      
        ...
    }
```

In TS 2.0, the action.payload type is narrowed to Book[] based on switch(action.type). However, in 2.1 the rules have changed.

Everything is well described here: ngrx app issue: https://github.com/ngrx/example-app/issues/93 and related TS issue: https://github.com/Microsoft/TypeScript/issues/2214#issuecomment-268855988

The TS 2.1 code requires a couple of changes:

1) The string literal types need to be declared as readonly - and since anonymous objects can't have readonly properties, we must use a class instead
2) The usage of these types must also be readonly, to preserve literals in TS 2.1 - I'm borrowing the phrase here, see the sources for a more insightful explanation.

```typescript
export class ActionTypes {
  static readonly SEARCH =            type('[Book] Search');
  static readonly SEARCH_COMPLETE =   type('[Book] Search Complete');

    ...
};

export class SearchAction implements Action {
  readonly type = ActionTypes.SEARCH;

  constructor(public payload: string) { }
}

export class SearchCompleteAction implements Action {
  readonly type = ActionTypes.SEARCH_COMPLETE;

  constructor(public payload: Book[]) { }
}

...
```
This seems like quite a drastic change for a point release.