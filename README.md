# Duckr

> Note for the course [Redux](https://learn.tylermcginnis.com/courses/51210)

## redux schema

```js
{
  users: {
    isAuthed,
    isFetching,
    error,
    authedId,
    [uid]: {
      lastUpdated,
      info: {
        name,
        uid,
        avatar,
      }
    }
  },
  modal: {
    duck,
    isOpen
  },
  ducks: {
    [duckId]: {
      lastUpdated,
      info: {
        avatar,
        duckId,
        name,
        text,
        timestamp,
        uid,
      }
    }
  },
  likeCount: {
    [duckId]: 0
  },
  usersDucks: {
    isFetching,
    error,
    [uid]: {
      lastUpdated,
      duckIds: [duckId, duckId, duckId]
    }
  },
  usersLikes: {
    duckid: true,
  }
  feed: {
    isFetching,
    error,
    newDucksAvailable,
    duckIdsToAdd: [duckId, duckId],
    duckIds: [duckid, duckId, duckId]
  }
  replies: {
    isFetching,
    error,
    [duckId]: {
      lastUpdated,
      replies: {
        [replyId]: {
          name,
          reply,
          uid,
          timestamp,
          avatar
        }
      }
    }
  },
  listeners: {
    [listenerId]: true
  }
}
```

## actions & reducers

> Actions describe the fact that some action occurred in your application but don’t concern themselves with how the actual state change is implemented in response to that action. That’s the job of the reducer.

```js
// action
// NOTE: dispatch come from react-redux
dispatch(someAction(...))

// reducer
const initialState = {
  isFetching: true
}

function (state = initialState, action) {
  switch (action.type) {
    type "FETCH_SUCCESS":
      return {
        ...state,
        isFetching: false
      }
    default:
      return state
  }
}
```

## Improving your Developer Experience with Redux

* Code lint & style

  * eslint
  * prettier

* Webpack

```js
import webpack from "webpack";
import path from "path";
import HtmlWebpackPlugin from "html-webpack-plugin";

// NOTE: npm scripts -> npm_lifecycle_event
// "start": "webpack-dev-server" -> "start"
// "production": "webpack -p" -> "production"
const LAUNCH_COMMAND = process.env.npm_lifecycle_event;

const isProduction = LAUNCH_COMMAND === "production";
process.env.BABEL_ENV = LAUNCH_COMMAND;

const PATHS = {
  app: path.join(__dirname, "app"),
  build: path.join(__dirname, "dist")
};

const HTMLWebpackPluginConfig = new HtmlWebpackPlugin({
  template: PATHS.app + "/index.html",
  filename: "index.html",
  inject: "body"
});

const productionPlugin = new webpack.DefinePlugin({
  "process.env": {
    NODE_ENV: JSON.stringify("production")
  }
});

const base = {
  entry: [PATHS.app],
  output: {
    path: PATHS.build,
    filename: "index_bundle.js"
  },
  module: {
    loaders: [
      // NOTE: exclude node_modules is important
      { test: /\.js$/, exclude: /node_modules/, loader: "babel-loader" },
      {
        test: /\.css$/,
        loader:
          "style!css?sourceMap&modules&localIdentName=[name]__[local]___[hash:base64:5]"
      }
    ]
  },
  resolve: {
    // define src root path
    root: path.resolve("./app")
  }
};

const developmentConfig = {
  devtool: "cheap-module-inline-source-map",
  devServer: {
    contentBase: PATHS.build,
    hot: true,
    inline: true,
    progress: true
  },
  plugins: [HTMLWebpackPluginConfig, new webpack.HotModuleReplacementPlugin()]
};

const productionConfig = {
  devtool: "cheap-module-source-map",
  plugins: [HTMLWebpackPluginConfig, productionPlugin]
};

export default Object.assign(
  {},
  base,
  isProduction === true ? productionConfig : developmentConfig
);
```

## CSS Modules

```js
import styles from "styles.css";
function Button() {
  return <div className={styles.container}>Yo</div>;
}
// ->
<div class="styles__container__eEoJi">Yo</div>;
```

```css
/* colors.css */
.primary {
  background-color: #ff000;
}
.button {
  height: 50px;
  width: 100px;
}
.dark {
  composes: button;
  background: #000;
}
// NOTE: compose
// like less `.button` syntax, but lost other less/sass features
.light {
  composes: button;
  composes: primary from "../shared/colors.css";
}
```

## Connect redux to react

```js
// ref: https://github.com/reactjs/redux
https: import { createStore } from "redux";

/**
 * This is a reducer, a pure function with (state, action) => state signature.
 * It describes how an action transforms the state into the next state.
 *
 * The shape of the state is up to you: it can be a primitive, an array, an object,
 * or even an Immutable.js data structure. The only important part is that you should
 * not mutate the state object, but return a new object if the state changes.
 *
 * In this example, we use a `switch` statement and strings, but you can use a helper that
 * follows a different convention (such as function maps) if it makes sense for your
 * project.
 */
function counter(state = 0, action) {
  switch (action.type) {
    case "INCREMENT":
      return state + 1;
    case "DECREMENT":
      return state - 1;
    default:
      return state;
  }
}

// Create a Redux store holding the state of your app.
// Its API is { subscribe, dispatch, getState }.
let store = createStore(counter);

// You can use subscribe() to update the UI in response to state changes.
// Normally you'd use a view binding library (e.g. React Redux) rather than subscribe() directly.
// However it can also be handy to persist the current state in the localStorage.

store.subscribe(() => console.log(store.getState()));

// The only way to mutate the internal state is to dispatch an action.
// The actions can be serialized, logged or stored and later replayed.
store.dispatch({ type: "INCREMENT" });
// 1
store.dispatch({ type: "INCREMENT" });
// 2
store.dispatch({ type: "DECREMENT" });
// 1
```

```js
// ref: https://github.com/reactjs/react-redux/blob/master/docs/api.md

/**
 * <Provider store>
 * Makes the Redux store available to the connect() calls in the component hierarchy below.
 * Normally, you can’t use connect() without wrapping a parent or ancestor component in <Provider>.
 */
// Vanilla React
ReactDOM.render(
  <Provider store={store}>
    <MyRootComponent />
  </Provider>,
  rootEl
);
// React Router
ReactDOM.render(
  <Provider store={store}>
    <Router history={history}>
      <Route path="/" component={App}>
        <Route path="foo" component={Foo} />
        <Route path="bar" component={Bar} />
      </Route>
    </Router>
  </Provider>,
  document.getElementById("root")
);

// connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])
// TODO
```

## Redux Thunks

```js
// ref: https://github.com/gaearon/redux-thunk
import { createStore, applyMiddleware } from "redux";
import thunk from "redux-thunk";
import rootReducer from "./reducers";

// Note: this API requires redux@>=3.1.0
const store = createStore(rootReducer, applyMiddleware(thunk));

function fetchSecretSauce() {
  return fetch("https://www.google.com/search?q=secret+sauce");
}

// These are the normal action creators you have seen so far.
// The actions they return can be dispatched without any middleware.
// However, they only express “facts” and not the “async flow”.

function makeASandwich(forPerson, secretSauce) {
  return {
    type: "MAKE_SANDWICH",
    forPerson,
    secretSauce
  };
}

function apologize(fromPerson, toPerson, error) {
  return {
    type: "APOLOGIZE",
    fromPerson,
    toPerson,
    error
  };
}

function withdrawMoney(amount) {
  return {
    type: "WITHDRAW",
    amount
  };
}

// Even without middleware, you can dispatch an action:
store.dispatch(withdrawMoney(100));

// But what do you do when you need to start an asynchronous action,
// such as an API call, or a router transition?

// Meet thunks.
// A thunk is a function that returns a function.
// This is a thunk.

function makeASandwichWithSecretSauce(forPerson) {
  // Invert control!
  // Return a function that accepts `dispatch` so we can dispatch later.
  // Thunk middleware knows how to turn thunk async actions into actions.

  return function(dispatch) {
    return fetchSecretSauce().then(
      sauce => dispatch(makeASandwich(forPerson, sauce)),
      error => dispatch(apologize("The Sandwich Shop", forPerson, error))
    );
  };
}

// Thunk middleware lets me dispatch thunk async actions
// as if they were actions!

store.dispatch(makeASandwichWithSecretSauce("Me"));

// It even takes care to return the thunk’s return value
// from the dispatch, so I can chain Promises as long as I return them.

store.dispatch(makeASandwichWithSecretSauce("My wife")).then(() => {
  console.log("Done!");
});

// In fact I can write action creators that dispatch
// actions and async actions from other action creators,
// and I can build my control flow with Promises.

function makeSandwichesForEverybody() {
  return function(dispatch, getState) {
    if (!getState().sandwiches.isShopOpen) {
      // You don’t have to return Promises, but it’s a handy convention
      // so the caller can always call .then() on async dispatch result.

      return Promise.resolve();
    }

    // We can dispatch both plain object actions and other thunks,
    // which lets us compose the asynchronous actions in a single flow.

    return dispatch(makeASandwichWithSecretSauce("My Grandma"))
      .then(() =>
        Promise.all([
          dispatch(makeASandwichWithSecretSauce("Me")),
          dispatch(makeASandwichWithSecretSauce("My wife"))
        ])
      )
      .then(() => dispatch(makeASandwichWithSecretSauce("Our kids")))
      .then(() =>
        dispatch(
          getState().myMoney > 42
            ? withdrawMoney(42)
            : apologize("Me", "The Sandwich Shop")
        )
      );
  };
}

// This is very useful for server side rendering, because I can wait
// until data is available, then synchronously render the app.

store
  .dispatch(makeSandwichesForEverybody())
  .then(() =>
    response.send(ReactDOMServer.renderToString(<MyApp store={store} />))
  );

// I can also dispatch a thunk async action from a component
// any time its props change to load the missing data.

import { connect } from "react-redux";
import { Component } from "react";

class SandwichShop extends Component {
  componentDidMount() {
    this.props.dispatch(makeASandwichWithSecretSauce(this.props.forPerson));
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.forPerson !== this.props.forPerson) {
      this.props.dispatch(makeASandwichWithSecretSauce(nextProps.forPerson));
    }
  }

  render() {
    return <p>{this.props.sandwiches.join("mustard")}</p>;
  }
}

export default connect(state => ({
  sandwiches: state.sandwiches
}))(SandwichShop);
```

```js
// Since 2.1.0, Redux Thunk supports injecting a custom argument using the
// withExtraArgument function:
const store = createStore(
  reducer,
  applyMiddleware(thunk.withExtraArgument({ api, whatever }))
);

// later
function fetchUser(id) {
  return (dispatch, getState, { api, whatever }) => {
    // you can use api and something else here here
  };
}
```

## Route Protection

```js
<Router history={hashHistory}>
  <Route path="/" component={MainContainer}>
    <Route path="feed" component={FeedContainer} onEnter={checkAuth} />
    <Route path="logout" component={LogoutContainer} />
    <IndexRoute component={HomeContainer} />
  </Route>
</Router>;

function checkAuth(nextState, replace) {
  if (nextState.location.pathname === "/feed") {
    if (isAuthed() !== true) {
      replace("/");
    }
  }
}
```

## react-router-redux

```js
// ref: https://github.com/reactjs/react-router-redux
import React from "react";
import ReactDOM from "react-dom";
import { createStore, combineReducers } from "redux";
import { Provider } from "react-redux";
import { Router, Route, browserHistory } from "react-router";
import { syncHistoryWithStore, routerReducer } from "react-router-redux";

import reducers from "<project-path>/reducers";

// Add the reducer to your store on the `routing` key
const store = createStore(
  combineReducers({
    ...reducers,
    routing: routerReducer
  })
);

// Create an enhanced history that syncs navigation events with the store
const history = syncHistoryWithStore(browserHistory, store);

ReactDOM.render(
  <Provider store={store}>
    {/* Tell the Router to use our enhanced history */}
    <Router history={history}>
      <Route path="/" component={App}>
        <Route path="foo" component={Foo} />
        <Route path="bar" component={Bar} />
      </Route>
    </Router>
  </Provider>,
  document.getElementById("mount")
);
```
