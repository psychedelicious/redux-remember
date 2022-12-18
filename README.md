[![NPM Version](https://img.shields.io/npm/v/redux-remember.svg?style=flat-square)](https://www.npmjs.com/package/redux-remember)
[![Build Status](https://github.com/zewish/redux-remember/workflows/build/badge.svg)](https://github.com/zewish/redux-remember/actions?query=workflow%3Abuild)
[![Coverage Status](https://coveralls.io/repos/github/zewish/redux-remember/badge.svg?branch=master)](https://coveralls.io/github/zewish/redux-remember?branch=master)
[![NPM Downloads](https://img.shields.io/npm/dm/redux-remember.svg?style=flat-square)](https://www.npmjs.com/package/redux-remember)

![Logo](https://raw.githubusercontent.com/zewish/redux-remember/master/logo.png)

Redux Remember saves and loads your redux state from a key-value store of your choice.

__Key features:__
- Saves (persists) and loads (rehydrates) only allowed keys and does not touch anything else.
- Completely unit and battle tested.
- Works on both web (any redux compatible app) and native (react-native).

__Works with any of the following:__
- AsyncStorage (react-native)
- LocalStorage (web)
- SessionStorage (web)
- Your own custom storage driver that implements `setItem(key, value)` and `getItem(key)`

### [__See demo!__](https://rawgit.com/zewish/redux-remember/master/demo-web/index.html)

Installation
------------
```bash
$ npm install --save redux-remember
# or
$ yarn add redux-remember
```

Usage - web
-----------

```ts
import { configureStore, createSlice, PayloadAction } from '@reduxjs/toolkit';
import { rememberReducer, rememberEnhancer } from 'redux-remember';

const myStateIsRemembered = createSlice({
  name: 'persisted-slice',
  initialState: {
    text: ''
  },
  reducers: {
    setPersistedText(state, action: PayloadAction<{ text: string }>) {
      state.text = action.payload;
    }
  }
});

const myStateIsForgotten = createSlice({
  name: 'forgotten-slice',
  initialState: {
    text: ''
  },
  reducers: {
    setForgottenText(state, action: PayloadAction<{ text: string }>) {
      state.text = action.payload;
    }
  }
});

const reducers = {
  myStateIsRemembered: myStateIsRemembered.reducer,
  myStateIsForgotten: myStateIsForgotten.reducer,
  someExtraData: (state = 'bla') => state
};

export const actions = {
  ...myStateIsRemembered.actions,
  ...myStateIsForgotten.actions
};

const rememberedKeys = [ 'myStateIsRemembered' ]; // 'myStateIsForgotten' will be forgotten, as it's not in this list

const store = configureStore({
  reducer: rememberReducer(
    reducers
  ),
  enhancers: [rememberEnhancer(
    window.localStorage,
    rememberedKeys,
    { persistWholeStore: true }
  )]
});

// Continue using the redux store as usual...
```

Usage - react-native
--------------------

```ts
import AsyncStorage from '@react-native-community/async-storage';
import { configureStore, createSlice, PayloadAction } from '@reduxjs/toolkit';
import { rememberReducer, rememberEnhancer } from 'redux-remember';

const myStateIsRemembered = createSlice({
  name: 'persisted-slice',
  initialState: {
    text: ''
  },
  reducers: {
    setPersistedText(state, action: PayloadAction<{ text: string }>) {
      state.text = action.payload;
    }
  }
});

const myStateIsForgotten = createSlice({
  name: 'forgotten-slice',
  initialState: {
    text: ''
  },
  reducers: {
    setForgottenText(state, action: PayloadAction<{ text: string }>) {
      state.text = action.payload;
    }
  }
});

const reducers = {
  myStateIsRemembered: myStateIsRemembered.reducer,
  myStateIsForgotten: myStateIsForgotten.reducer,
  someExtraData: (state = 'bla') => state
};

export const actions = {
  ...myStateIsRemembered.actions,
  ...myStateIsForgotten.actions
};

const rememberedKeys = [ 'myStateIsRemembered' ]; // 'myStateIsForgotten' will be forgotten, as it's not in this list

const store = configureStore({
  reducer: rememberReducer(
    reducers
  ),
  enhancers: [rememberEnhancer(
    AsyncStorage, // or your own custom storage driver
    rememberedKeys,
    { persistWholeStore: true }
  )]
});

// Continue using the redux store as usual...
```

Usage - inside a reducer
------------------------

```ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';
import { REMEMBER_REHYDRATED, REMEMBER_PERSISTED } from 'redux-remember';

type InitialState = {
  changeMe: any;
  rehydrated: boolean;
  persisted: boolean;
};

const initialState: InitialState = {
  changeMe: null,
  rehydrated: false,
  persisted: false
};

const usageInsideReducer = createSlice({
  name: 'usage-inside-reducer-slice',
  initialState,
  reducers: {
    someAction(state, action: PayloadAction<{ changeMe: any }>) {
      if (!state.rehydrated) {
        return;
      }

      state.changeMe = action.payload.changeMe;
    }
  },
  extraReducers: (builder) => builder
    .addCase(REMEMBER_REHYDRATED, (state, action) => {
      state.changeMe = (action as PayloadAction<InitialState>).payload.changeMe;
      state.rehydrated = true;
    })
    .addCase(REMEMBER_PERSISTED, (state) => {
      state.rehydrated = false;
      state.persisted = true;
    })
});
```

Usage - legacy apps (without redux toolkit)
-------------------------------------------

Examples here are using redux toolkit.<br>
If your application still isn't migrated to redux toolkit, [check the legacy usage documentation](./LEGACY-USAGE.md).


API reference
-------------
- **rememberReducer(reducers: Reducer | ReducersMapObject)**
    - Arguments:
        1. **reducers** *(required)* - takes the result of `combineReducers()` function or list of non-combined reducers to combine internally (same as redux toolkit);
    - Returns - a new root reducer to use as first argument for the `configureStore()` (redux toolkit) or the `createStore()` (plain redux) function;


- **rememberEnhancer(driver: Driver, rememberedKeys: string[], options?: Options)**
    - Arguments:
        1. **driver** *(required)* - storage driver instance, that implements the `setItem(key, value)` and `getItem(key)` functions;
        2. **rememberedKeys** *(required)* - an array of persistable keys - if an empty array is provided nothing will get persisted;
        3. **options** *(optional)* - plain object of extra options:
            - **prefix**: storage key prefix *(default: `'@@remember-'`)*;
            - **serialize** - a plain function that takes unserialized store state and its key (`serialize(state, stateKey)`) and returns serialized state to be persisted *(default: `JSON.stringify`)*;
            - **unserialize** - a plain function that takes serialized persisted state  and its key (`serialize(state, stateKey)`) and returns unserialized to be set in the store *(default: `JSON.parse`)*;
            - **persistThrottle** - how much time should the persistence be throttled in milliseconds *(default: `100`)*
            - **persistWholeStore** - a boolean which specifies if the whole store should be persisted at once. Generally only use this if you're using your own storage driver which has gigabytes of storage limits. Don't use this when using window.localStorage, window.sessionStorage or AsyncStorage as their limits are quite small. When using this option, key won't be passed to `serialize` nor `unserialize` functions - *(default: `false`)*;
    - Returns - an enhancer to be used with Redux
