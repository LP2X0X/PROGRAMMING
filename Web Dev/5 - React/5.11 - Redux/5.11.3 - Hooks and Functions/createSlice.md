---
tags: 
 - react
 - redux
 - function
---

- A function that accepts an initial state, an object of reducer functions, and a "slice name", and automatically generates action creators and action types that correspond to the reducers and state.
  
```js
function createSlice({
    // A name, used in action types
    name: string,
    // The initial state for the reducer
    initialState: State,
    // An object of "case reducers". Key names will be used to generate actions.
    reducers: Record<string, ReducerFunction | ReducerAndPrepareObject>,
    // A "builder callback" function used to add more reducers
    extraReducers?: (builder: ActionReducerMapBuilder<State>) => void,
    // A preference for the slice reducer's location, used by `combineSlices` and `slice.selectors`. Defaults to `name`.
    reducerPath?: string,
    // An object of selectors, which receive the slice's state as their first parameter.
    selectors?: Record<string, (sliceState: State, ...args: any[]) => any>,
})
```