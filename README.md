![HubSpot/general-store](https://cloud.githubusercontent.com/assets/478109/6376307/1d3c77dc-bceb-11e4-9a96-0a909810cc69.png)

[![Build Status](https://travis-ci.org/HubSpot/general-store.svg)](https://travis-ci.org/HubSpot/general-store)

**This is beta software. It's going to change.**

`general-store` aims to provide all the features of a [Flux](http://facebook.github.io/flux/) store without prescribing the implementation of that store's data or mutations.

Briefly, a store:

1. contains any arbitrary value
2. exposes that value via a get method
3. responds to specific events from the dispatcher
4. notifies subscribers when its value changes

That's it. All other features, like Immutability, data fetching, undo, etc. are implementation details.

Read more about the `general-store` rational [on the HubSpot Product Team Blog](http://product.hubspot.com/blog/keeping-flux-flexible-with-general-store).

## Install

```
# for node, browserify, etc
npm install general-store

# for bower
bower install general-store
```

## Create a store

GeneralStore uses functions to encapsulate private data.

```javascript
var dispatcher = new Flux.Dispatcher();
function defineUserStore() {
  // data is stored privately inside the store module's closure
  var users = {
    123: {
      id: 123,
      name: 'Mary'
    }
  };

  return GeneralStore.define()
    // the store's getter should return the public subset of its data
    .defineGet(function() {
      return users;
    })
    // handle actions received from the dispatcher
    .defineResponseTo('ADD_USER', function(user) {
      users[user.id] = user;
    })
    .defineResponseTo('REMOVE_USER', function(user) {
      delete users[user.id];
    })
    // after a store is "registered" its action handlers are bound
    // to the dispatcher
    .register(dispatcher);
}
```

If you use a singleton pattern for stores, simply use the result of `register` from a module.

```javascript
var Dispatcher = require('flux').Dispatcher;
var GeneralStore = require('general-store.js');

var dispatcher = new Dispatcher();
var users = {};

var UserStore = GeneralStore.define()
  .defineGet(function() {
    return users;
  })
  .register(dispatcher);

module.exports = UserStore;
```

## Dispatch to the Store

Sending a message to your stores via the dispatcher is easy.

```javascript
dispatcher.dispatch({
  actionType: 'ADD_USER', // required field
  data: { // optional field, passed to the store's response
    id: 12314,
    name: 'Colby Rabideau'
  }
});
```

## Using the Store API

A registered Store provides methods for "getting" its value and subscribing to changes to that value.

```javascript
UserStore.get() // returns {}
var subscription = UserStore.addOnChange(function() {
  // handle changes!
});
// addOnChange returns an object with a `remove` method.
// When you're ready to unsubscribe from a store's changes,
// simply call that method.
subscription.remove();
```

## React

GeneralStore provides a convenient mixin for binding stores to React components:

```javascript
var ProfileComponent = React.createClass({
  mixins: [
    GeneralStore.StoreDependencyMixin({
      // simple fields can be expressed in the form `key => store`
      subject: ProfileStore,
      // compound fields can depend on one or more stores
      // and specify a function to "dereference" the store's value
      friends: {
        stores: [ProfileStore, UsersStore],
        deref: (props, state) => {
          friendIds = ProfileStore.get().friendIds;
          users = UsersStore.get();
          return friendIds.map(id => users[id]);
        }
      }
    })
  ],

  render: function() {
    return (
      <div>
        <h1>{this.state.subject.name}</h1>
        {this.renderFriends()}
      </div>
    );
  },

  renderFriends: function() {
    var friends = this.state.friends;
    return (
      <div>
        <h3>Friends</h3>
        <ul>
          {Object.keys(friends).map(id => <li>{friends[id].name}</li>)}
        </ul>
      </div>
    );
  }
});
```

## Default Dispatcher Instance

The common Flux architecture has a single central dispatcher. As a convenience `GeneralStore` allows you to set a global dispatcher which will become the default when a store is registered.

```javascript
var dispatcher = new Flux.Dispatcher();
GeneralStore.DispatcherInstance.set(dispatcher);
```

Now you can register a store without explicitly passing a dispatcher:

```javascript
var users = {};

GeneralStore.define()
  .defineGet(() => users)
  .register(); // the dispatcher instance is set so no need to explicitly pass it
```

## Dispatcher Interface

At HubSpot we use the [Facebook Dispatcher](https://github.com/facebook/flux), but any object that conforms to the same interface (i.e. has register and unregister methods) should work just fine.

```javascript
type DispatcherPayload = {
  actionType: string;
  data: any;
};

type Dispatcher = {
  isDispatching: () => bool;
  register: (
    handleAction: (payload: DispatcherPayload) => void
  ) => number;
  unregister: (dispatchToken: number) => void;
  waitFor: (dispatchTokens: Array<number>) => void;
};
```

## Build and test

**Install Dependencies**

```
# pull in dependencies
npm install

# run the type checker and unit tests
npm test

# if all tests pass, run the dev and prod build
npm run build-and-test

# if all tests pass, run the dev and prod build then commit and push changes
npm run deploy
```

## Special Thanks

Logo design by [Chelsea Bathurst](http://www.chelseabathurst.com)
