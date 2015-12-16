# redux-electron-store

This is a class which offers the same api as a [Redux](https://github.com/rackt/redux/) store, however it handles synchronizing stores initialized in the various processes of an Electron app.

## Installation
```bash
npm i redux-electron-store
```

## Usage

#### Browser Process

In the browser process `ReduxElectronStore` takes in a redux `createStore` function as well as a redux reducer, and once instantiated can act just like a normal redux store.

```javascript
let createReduxStore = compose(
  applyMiddleware(...middleware),
  DevTools.instrument()
)(createStore);
let reducer = (state, action) => { ... };
let store = new ReduxElectronStore({createReduxStore, reducer});
```

#### Renderer / Webview Process

In the renderer process, the store takes the same `createStore` and `reducer` properties, however it can also take a `filter`.  `filter` is a way of describing exactly what data this renderer process wishes to be notified of.

```javascript
let filter = {
  notifications: true,
  settings: true
}
this.store = new ReduxElectronStore({createReduxStore, reducer, filter});
```

##### Filters

A filter can be an `object`, a `function`, or `true`.

If the filter is `true`, the entire variable will pass through the filter.

If the filter is a `function`, the function will be called with the variable the filter is acting on as a parameter, and the return value of the function must itself be a filter (either an `object` or `true`)

If the filter is an `object`, its keys must be properties of the variable the filter is acting on, and its values are themselves filters which describe the value(s) of that property that will pass through the filter.

**Example Problem**: I am creating a Notifications window.  For this to work, I need to know the position to display the notifications, the notifications themselves, and the icons for each team to display as a thumbnail.  Any other data in my app has no bearing on this window, therefore it would be a waste for this window to have updates for any other data sent to it.

**Solution**:
```javascript
// Note: The Lodash library is being used here as _
let filter = {
  notifications: true,
  settings: {
    notifyPosition: true
  },
  teams: (teams) => {
    return _.mapValues(teams, (team) => _.pick(teams, 'icons'));
  }
}

this.store = new ReduxElectronStore({createReduxStore, reducer, filter});
```

A full electron example of this library in action is soon to come


## TODOs

1. Create working Electron App to serve as a complete example
2. Decide on a testing framework / library
3. Add unit tests for utility functions
4. Enable handling multiple stores
