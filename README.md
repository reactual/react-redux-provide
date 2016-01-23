# react-redux-provide

[![build status](https://img.shields.io/travis/loggur/react-redux-provide/master.svg?style=flat-square)](https://travis-ci.org/loggur/react-redux-provide) [![npm version](https://img.shields.io/npm/v/react-redux-provide.svg?style=flat-square)](https://www.npmjs.com/package/react-redux-provide)
[![npm downloads](https://img.shields.io/npm/dm/react-redux-provide.svg?style=flat-square)](https://www.npmjs.com/package/react-redux-provide)


This small library allows you to:

1. Build your entire app's view layer first - i.e., all your components become as "dumb" as possible.

2. Decorate your components with `@provide`, which allows you to specify - as `propTypes` - exactly the data and actions said components need from [`redux`](https://github.com/rackt/redux), using as many stores and/or combining providers as necessary.

  **Note: Providers are automatically assigned to the components that need them.**

3. Pass context as props at any level (though typically only the root component is necessary) to any component decorated with `@provide`.


## Pros

- Maximum separation of concerns.
- Use packaged providers for common use-cases or even build apps around specific providers.
- Quickly and easily switch out one provider for another.
- Very easily mix and match components from multiple applications whose components use `@provide`.
- Enforces clean and efficient design.
- Extremely predictable and easy to understand.
- Greatly reduces boilerplate.  Most apps should only need a `components` directory and a `providers` directory.
- No need for `react-redux`'s `<Provider>` component.  The higher-order component generated by `@provide` handles the contextual functionality (among a few other things!) for you at any level.
- Automatic stores, multiple stores, and/or combined stores.
- Looks good in [`react-devtools`](https://github.com/facebook/react-devtools)!

![See `ProvideBranches(theme,packageList,sources)`](https://cloud.githubusercontent.com/assets/7020411/9288123/3587858e-4305-11e5-8156-fe0392e6f7fd.png)


## Cons

- You tell me!


## Installation

```
npm install react-redux-provide --save
```


## Usage

The API surface area is [naturally tiny](https://github.com/loggur/react-redux-provide/blob/master/src/index.js).  You typically only need to concern yourself with the main export and occasionally a few utilities:

1.  `provide` - The decorator that passes the providers' `props` (state) and `actions` from `redux` to your components.  It returns a higher-order component wrapped around the decorated component and accepts the following 3 `props` which you can pass to any `@provide`d component at any level to render any `context` as necessary:

  - `providers` - Object containing all the providers you want your components to use.  The keys are the providers' names.

  - `providedState` - Optional object containing the combined state of each provider's `reducers`.  The decorator will automatically separate the keys into their providers' stores as necessary.  (This was originally called `initialState`, but `providedState` is semantically better, especially for server rendering.)

  - `combinedProviders` - Optional object or array of objects containing providers that share the same store(s).  See [`test/providers/someCombinedProvider.js`](https://github.com/loggur/react-redux-provide/blob/master/test/providers/someCombinedProvider.js) for an example of a provider (`someCombinedProvider`) that depends on another provider (`list`).  Then rendering the app typically looks like this:

    ```js
    import React from 'react';
    import ReactDOM from 'react-dom';
    import ReactDOMServer from 'react-dom/server';
    import { App } from './components/index';
    import providers, { list, someCombinedProvider } from './providers/index';

    const context = {
      providers,
      combinedProviders: [
        { list, someCombinedProvider }
      ],
      providedState: {
        list: [
          'easy!',
          'right?'
        ]
      }
    };

    ReactDOM.render(<App { ...context } />, document.getElementById('root'));
    
    // or server rendering
    export function renderToString(context) {
      return ReactDOMServer.renderToString(<App { ...context } />);
    }
    ```

2.  `pushMiddleware (Object providers, Array|Function middleware)` - Adds middleware(s) to the end of each provider's chain of middlewares.

3.  `unshiftMiddleware (Object providers, Array|Function middleware)` - Adds middleware(s) to the beginning of each provider's chain of middlewares.

4.  `pushEnhancer (Object providers, Array|Function enhancer)` - Adds enhancer(s) to the end of each provider's chain of enhancers.

5.  `unshiftEnhancer (Object providers, Array|Function enhancer)` - Adds enhancer(s) to the beginning of each provider's chain of enhancers.

6.  `createProviderStore (Object provider, Optional Object providedState)` - Creates and returns a store specifically for some provider.

7.  `createCombinedStore (Object providers, Optional Object providedState)` - Creates and returns a shared store based on the combination of each provider.  Especially useful when a provider's state depends on another provider's actions.


## Creating Providers

A provider is just an object with a few properties:

- `actions` - Your usual [`redux`](https://github.com/rackt/redux) `actions`.

- `reducers` - Your usual [`redux`](https://github.com/rackt/redux) `reducers`.

- `middleware` - Include whatever middleware is used by your provider.  This can be either an array of middlewares or a single middleware.

- `enhancer` - Include whatever enhancer is used by your provider's store.  This can be either an array of enhancers or a single enhancer.

- `merge (stateProps, dispatchProps, parentProps)` - This incredibly useful function should return an object, which typically adds, removes, or replaces certain provided properties based on whatever logic you deem necessary.  For example, in [`react-redux-provide-list`](https://github.com/loggur/react-redux-provide-list), if the component has an `index` prop passed to its parent and expects an `item` prop from the provider, the `merge` function will attempt to provide the `item` at that `index` within the `list` to the component.

- `store` - This is your typical `redux` store.  See the Caveats section above about automatically generated stores.

- `mapState` - Maps each reduced state to the provided `props`.  By default, it will map them all.  It's unlikely that you'll ever actually need to include this.

- `mapDispatch` - It's unlikely that you'll need to include this as well.  This defaults to `dispatch => bindActionCreators(actions, dispatch)` or if it's an object, it will use `redux`'s `wrapActionCreators`.


## Caveats

1.  Components can have multiple providers, but the provided `props` (`actions` and `reducers`) should be unique to each provider.

2.  When instantiating components and determining which providers are relevant to each component, it will automatically create a new store for you if you haven't explicitly included a `store` key within your `provider` object.  Said store is of course shared throughout the components.

3.  Specify *all* of your `propTypes`!  The `provide` decorator filters out any `props` not within your `propTypes`, which keeps things efficient and helps with avoiding unnecessary re-renders.  Plus, it's good design!


## Quick Example

Basically, create some component with only the view in mind, plus whatever `props` you'd expect to use for triggering actions.  For this quick example, we know [`react-redux-provide-list`](https://github.com/loggur/react-redux-provide-list) provides a `list` prop and a `pushItem` function, so after using the `@provide` decorator, all we have to do is specify `list` and `pushItem` within our component's `propTypes`.

From [examples/good-times/components/GoodTimes.js](https://github.com/loggur/react-redux-provide/blob/master/examples/good-times/components/GoodTimes.js):
```js
import React, { Component, PropTypes } from 'react';
import provide from 'react-redux-provide';

@provide
export default class GoodTimes extends Component {
  static propTypes = {
    list: PropTypes.arrayOf(PropTypes.object).isRequired,
    pushItem: PropTypes.func.isRequired
  };

  addTime() {
    this.props.pushItem({
      time: Date.now()
    });
  }

  render() {
    return (
      <div className="good-times">
        {this.renderButton()}
        {this.renderTimes()}
      </div>
    );
  }

  renderButton() {
    const style = {
      fontSize: '20px',
      marginBottom: '20px'
    };
    
    return (
      <input
        type="button"
        style={style}
        value="Let the good times roll"
        onClick={::this.addTime}
      />
    );
  }

  renderTimes() {
    return this.props.list.map(
      item => (
        <li key={item.time}>
          {new Date(item.time).toString()}
        </li>
      )
    );
  }
}
```

Then when rendering the app, all we need to do is pass the provider(s) and the state(s) of their reducers to the component(s):

```js
import { render } from 'react-dom';
import provideList from 'react-redux-provide-list';
import GoodStuff from './components/GoodStuff';

const list = provideList();

const context = {
  providers: { list },
  providedState: {
    list: [
      { fruit: 'apple' },
      { fruit: 'banana' }
      { vegetable: 'carrot' }
    ]
  }
};

render(<GoodStuff { ...context } />, document.getElementById('root'));
```


## Notes

You'll probably notice that many providers have everything in a single file.  It makes sense for most simple use-cases, but you can of course structure everything however you want since each provider is ultimately just a single object.

You don't have to use generic provider packages (e.g., [`list`](https://github.com/loggur/react-redux-provide-list), [`map`](https://github.com/loggur/react-redux-provide-map), etc.), as they only exist to help with really common use-cases.  For most apps, it works best to create a `providers` directory with an index that exports a `providers` object containing each provider, then we can simply import the object and pass it to the root component of the app when rendering.

**Protip:**  Avoid sharing constants.  Typically, the only place you should (occasionally) share constants is within providers.  See [`bloggur`'s `entries` provider](https://github.com/loggur/bloggur/blob/master/providers/entries.js) for a good example of shared constants, as its state depends on actions within [`react-redux-provide-page`](https://github.com/loggur/react-redux-provide-page).  Your components should have no knowledge of the constants used within your actions and reducers, which leads to a maximum separation of concerns and is always the best design.  Your components should care about only 2 things: what to render and which actions to call.
