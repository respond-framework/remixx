# Implementation Step 1: Tracking Reducer Usage

> This is based on rewriting `connect/index.js` in our codesandbox: https://codesandbox.io/s/jn8l8ly85?module=%2Fsrc%2Fconnect%2FProvider.js

## `connect` implementation

```js
import React from 'react'
import { createPathTracker } from 'remixx'
import Context from './Context'
import shallowEqual from './shallowEqual'

export default function connect(componentFunc) {
  class Inner extends React.Component {
    constructor(props) {
      super(props)
      this.state = createPathTracker(props.state)
    }

    shouldComponentUpdate(nextProps) {
      const stateChanged = this.state._hasAccessedPathsChanged(nextProps.state)

      if (stateChanged) {
        this.state = createPathTracker(nextProps.state) // we need a new tracker that contains next state
      }

      return stateChanged || !shallowEqual(nextProps.props, this.props.props)
    }

    render() {
      const { props, actions } = this.props
      return componentFunc(props, this.state, actions)
    }
  }

  return function Outter(props) {
    return (
      <Context.Consumer>
        {({ state, actions }) => (
          <Inner props={props} state={state} actions={actions} />
        )}
      </Context.Consumer>
    )
  }
}
```

## `createPathTracker` implementation

```js
export default function createPathTracker(obj) {
  const proxy = createProxy(obj) // a general infinite nested key access tracker

  // the passed function here determines the return values when our proxy is accessed (just a rough idea)
  proxy.onAccess((path) => {
    return obj[path] // this obviously needs to be generalized to operate recursively on nested paths
  })

  proxy._hasAccessedPathsChanged = obj2 => {
    return !!proxy._accessedPaths.find(path => {
      return obj[path] !== obj2[path] // again, this needs to be generalized for nested paths
    })
  }

  return proxy
}
```


## `createProxy` implementation

```js
export default function createProxy(obj) {
  // the idea is this function creates an infinite proxy access recorder
  // where the following throws no errors:
  // obj.foo.bar.more.etc
  // obj.foo.bar.anotherBranch.etc
  return proxy
}


// key access produces the following tree data structure:
proxy._accessedPaths = {
  foo: {
    bar: {
      more: { etc: {} },
      anotherBranch: { etc: {} }
    }
  }
}
```

So this is just a hypothetical idea. It may not be feasable or smart. But like infinite numbers in math, they are impossible to reach,
but help us think about a concept. The concept here is the ultimate generalization of *nested key access recording*. And more
importantly the overall concept is that this allows us functions/abstractions higher up (namely `createPathTracker`) to not have
to think about these details. It's a nice primitive to help us better express the code that is truly unique to our needs. Again,
that code lives in `createPathTracker`. 

Perhaps, this only serves as a way to communicate ideas from me to you. I'm in no way religious about the implementation. That's
totally up to you based on the real details of how proxies work. I think it would also be nice to have a file that showcases
the raw proxy magic in the most amount of simplicity possible. That way it's easier for me and others to understand.


## Better Traversing Algo

> Kashey: this operation should be centralized to allow bulk optimizations, as long store updates are centralized.

```js
class AccessProxy {
 constructor(state) {
  this._oldState = state
 }
 
 traverse(accessedPaths, obj, obj2) {
  return Object.keys(accessedPaths).find(key => {
    return obj[key] !== obj2[key] || this.traverse(accessedPaths[key], obj[key], obj2[key])
  })
 }

 hasAccessedPathsChanged(nextState) {
   return this.traverse(this._accessedPaths, this._oldState, nextState)
 }
}

const stateProxy = new AccessProxy(state)

// user performs:
// stateProxy.foo.bar.more.etc
// stateProxy.foo.bar.anotherBranch.etc

// now we have:
stateProxy._accessedPaths: {
  foo: {
    bar: {
      more: { etc: {} },
      anotherBranch: { etc: {} }
    }
  }
}

// usage:
stateProxy.hasAccessedPathsChanged(nextState)
```



## FINAL NOTES:

We build this in relation to ONLY REDUCERS at first. Then when it works, we add selectors. 

We also don't change `redux`. We only make our own `react-redux`. 

This isolates our task and minimizes the # of things we have to worry about. Let's just get those reducers reactively trackable/subscribable (note: we should come up with a good name for this connection).

## PS:
All this looking quite similar to https://github.com/solkimicreb/react-easy-state, the only key difference - "easy" magic is not applicable here,
as long we could not afford "setters" it relies on.