# @ember-decorators/argument

[![Build Status](https://travis-ci.org/ember-decorators/argument.svg?branch=master)](https://travis-ci.org/ember-decorators/argument)
![Ember Version Badge](https://badgen.net/badge/ember/v3.6.0+/orange)

This addon provides a decorator that allows you to declaratively specify component arguments. Through it, you can have run-time type checking of component usage.

## Usage

```js
import Component from '@ember/component';
import { argument } from '@ember-decorators/argument';

export default class ExampleComponent extends Component {
  @argument('string')
  arg = 'default';
}
```

```hbs
{{example-component arg="value"}}
```

For each property that your component should be given, the `@argument` decorator should be applied to the property definition. It is passed a "type", which you can read more about below.

When rendering a component that uses `@argument`, the initial value of the property will be validated against the given type. Each time the property is changed, the new value will also be validated. If a mismatch is found, an error is thrown describing what went wrong.

In addition, any unexpected arguments to a component will also cause an error.

## Defining Types

The `@argument` decorator takes a definition for what kind of value the property should be set to. Usually, this will represent the type of the value.

### Primitive Types

For primitives types, the name should be provided as a string (as with `string` in the example above). The available types match those of Typescript, including:

- `any`
- `boolean`
- `null`
- `number`
- `object`
- `string`
- `symbol`
- `undefined`

There are also a number of helpers that can be used to validate against a more complex or specific type:

- `arrayOf`: Produces a type for an array of specific types
- `oneOf` : Produces a type that literally matches one of the given strings
- `optional`: Produces an optional / nullable type that, in addition to the type that was passed in,
  also allows `null` and `undefined`.
- `shapeOf`: Accepts an object of key -> type pairs, and checks the shape of the field to make sure it
  matches the object passed in. The validator only checks to make sure that the fields exist and are their
  proper types, so it is valid for all objects which fulfill the shape (structural typing)
- `unionOf`: Produces a union type from the specified types

```js
import Component from '@ember/component';
import { argument } from '@ember-decorators/argument';
import {
  arrayOf,
  oneOf,
  optional,
  shapeOf,
  unionOf
} from '@ember-decorators/argument/types';

export default class ExampleComponent extends Component {
  @argument(arrayOf('string'))
  stringArray;

  @argument(oneOf('red', 'blue', 'yellow'))
  primaryColor;

  @argument(optional(Date))
  optionalDate;

  @argument(shapeOf({ id: 'string' }))
  objectWithId;

  @argument(unionOf('number', 'string'))
  numberOrString;
}
```

In addition, this library includes several predefined types for convenience:

- `Action` - Type alias for `Function`, to be used when passing a ["closure action"][closure-action] into a component
- `ClassicAction` - Union of `string` and `Function`, to be used if your component uses `sendAction` to invoke a "classic action"
- `Element` - Fastboot safe type alias for `window.Element`
- `Node` - Fastboot safe type alias for `window.Node`

These types can also be imported from `@ember-decorators/argument/types`

### Class Instances

The `@argument` decorator can also take a class constructor to validate that the property value is an instance of that class

```js
import Component from '@ember/component';
import { argument } from '@ember-decorators/argument';

class Task {
  constructor() {
    this.complete = false;
  }
}

export default class TaskComponent extends Component {
  @argument(Task)
  task;
}
```

Passing a class works with all of the type helpers mentioned above.

## Installation

While `ember-decorators` is not a hard requirement to use this addon, it's recommended as it adds the base class field and decorator babel transforms

```bash
ember install ember-decorators
ember install @ember-decorators/argument
```

## Configuration

### Run-Time

#### `argumentWhitelist`

**Type**: `Array` or `Object` | **Default**: `[]`

Once `@argument` has been applied to a component, unexpected arguments passed to a component will result in an error. In order to work with some addons that expect you to provide additional arguments to your components, you can whitelist additional properties that should be ignored.

When provided an array of `string`, those exact properties will be added to the whitelist.

To provide a more flexible solution, you can also provide an object with the following structure:

```javascript
shapeOf({
  // Whitelist any string starting with one of these values
  startsWith: optional(arrayOf('string'))
  // Whitelist any string ending with one of these values
  endsWith: optional(arrayOf('string'))
  // Whitelist any string that include one of these values anywhere
  includes: optional(arrayOf('string'))
  // Whitelist any string that exactly matches one of these values
  matches: optional(arrayOf('string'))
})
```

##### Example

```javascript
// config/environment.js
module.exports = function(environment) {
  let ENV = {
    // ...
    '@ember-decorators/argument': {
      argumentWhitelist: {
        startsWith: ['hotReloadCUSTOM']
      }
    }
  };

  // ...

  return ENV;
};
```

### Build-Time

#### `enableCodeStripping`

**Type**: `Boolean` | **Default**: `true`

By default most of the code provided by this addon is removed in a Production build of your application. This way you can create a great development experience when writing your application, but prevent your users from paying the download or runtime cost of the validation. Both the runtime of the library, and any usage of the `@argument` decorator in your code, will be removed.

However, if the process seems buggy or you want the validation in production, setting this flag to `false` will prevent any code from being removed.

##### Example

```javascript
// ember-cli-build.js
const EmberApp = require('ember-cli/lib/broccoli/ember-app');

module.exports = function(defaults) {
  let app = new EmberApp(defaults, {
    '@ember-decorators/argument': {
      enableCodeStripping: false
    }
  });

  return app.toTree();
};
```

## Ember Compatibility

This addon works out-of-the-box with Ember `3.6` and higher. This is due to a dependency on the [changes to the native class constructor behavior][native-class-constructor-update] that landed in that Ember version.

Support can be polyfilled using [`ember-native-class-polyfill`][ember-native-class-polyfill] back to any version that it supports. Currently, that is Ember `3.4`.

## Running

- `ember serve`
- Visit your app at [http://localhost:4200](http://localhost:4200).

## Running Tests

- `npm test`
- `npm run test:all` (Runs `ember try:each` to test your addon against multiple Ember versions)
- `ember test --server`

## Building

- `ember build`

For more information on using ember-cli, visit [https://ember-cli.com/](https://ember-cli.com/).

[native-class-constructor-update]: https://github.com/emberjs/rfcs/blob/master/text/0337-native-class-constructor-update.md
[ember-native-class-polyfill]: https://www.npmjs.com/package/ember-native-class-polyfill
[closure-action]: https://alexdiliberto.com/posts/ember-closure-actions/
