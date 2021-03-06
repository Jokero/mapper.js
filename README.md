# mapper.js

Transform object from one structure to another by using schema. It's useful in API when you want to modify data before sending it to clients

[![NPM version](https://img.shields.io/npm/v/mapper.js.svg)](https://npmjs.org/package/mapper.js)
[![Build status](https://img.shields.io/travis/Jokero/mapper.js.svg)](https://travis-ci.org/Jokero/mapper.js)

**Note:** This module works in browsers and Node.js >= 6.0.

## Demo

Try [demo](https://runkit.com/npm/mapper.js) on RunKit.

## Installation

```sh
npm install mapper.js
```

### Node.js
```js
const mapper = require('mapper.js');
```

### Browser
```
<script src="node_modules/mapper.js/dist/mapper.js">
```
or minified version
```
<script src="node_modules/mapper.js/dist/mapper.min.js">
```

You can use the module with AMD/CommonJS or just use `window.mapper`.

## Usage

There are two ways to use module:

### 1. mapper(data, schema)

**Parameters**

* `data` (Object | Object[]) - Object or array of objects to map
* `schema` (Function | Object) - Schema which defines how to map object

**Return value**

(Object | Object[]) - Result of mapping (object or array of objects)

```js
const mapper = require('mapper.js');

const result = mapper(object, schema);
```

### 2. mapper(schema)

**Parameters**

* `schema` (Function | Object) - Schema which defines how to map object

**Return value**

(Function): Mapping function which can be called with `data` further

```js
const mapper = require('mapper.js');
const map    = mapper(schema);

const result = map(object);
```

### Dynamic mapping (schema as a function)

```js
const mapper = require('mapper.js');

const products = [
    {
        type: 'book',
        name: 'The Adventures of Tom Sawyer',
        description: 'Awesome book',
        count: 1
    },
    {
        type: 'sugar',
        description: 'Very tasty',
        weight: 3000
    }
];

const productsSchemas = {
    book: {
        name: true,
        count: true
    },

    sugar: {
        weight: true
    }
};

// function accepts an object and has to return a schema
const schema = product => Object.assign({}, productsSchemas[product.type], { type: true });

const result = mapper(products, schema);

```

### Example

Imagine we have collection of users and each user object has the following structure:

```js
const user = {
    id: 12345,

    firstName: 'Peter',
    lastName: 'Parker',

    company: {
        name: 'Daily Bugle',
        position: 'Photographer'
    },

    email: 'peter.parker@marvel.com',

    socialNetworks: {
        vk: 'https://vk.com/peter.parker',
        facebook: 'https://www.facebook.com/peter.parker'
    },

    friends: [
        {
            id: 12346,
            firstName: 'Mary Jane',
            lastName: 'Watson'
        },
        {
            id: 12347,
            firstName: 'Harold',
            lastName: 'Osborn'
        }
    ]
};
```

We want to send users data to clients in a different format:

```js
{
    id: 12345,
    fullName: 'Peter Parker',
    companyName: 'Daily Bugle',

    contacts: {
        email: 'peter.parker@marvel.com',
        vk: 'https://vk.com/peter.parker',
        facebook: 'https://www.facebook.com/peter.parker'
    },

    friends: [
        { id: 12346, fullName: 'Mary Jane Watson' },
        { id: 12347, fullName: 'Harold Osborn' }
    ]
}
```

We can do it by using the schema:

```js
const mapper = require('mapper.js');

const userSchema = {
    // take field without any changes
    id: true, // or '=',

    // you can specify function and return custom value
    fullName: (fullName, user) => user.firstName + ' ' + user.lastName,

    // take "name" field from "company" object
    companyName: 'company.name', // or '=company.name'

    // you can work with embedded objects
    contacts: {
        // by default property path is relative to current object,
        // so if you want to get value from top-level object ("user" in example) use "$"
        email: '$.email', // or '=$.email'

        // function takes 4 arguments:
        //   - current value of property = user.contacts.facebook
        //   - current object            = user.contacts
        //   - original object           = user
        //   - path as array             = ['contacts', 'facebook']
        facebook: (facebook, contacts, user, path) => user.socialNetworks.facebook,

        vk: '$.socialNetworks.vk'
    },

    // for arrays use array with schema for item
    friends: [{
        id: true,
        fullName: (fullName, friend) => friend.firstName + ' ' + friend.lastName
    }]
};

const result = mapper(user, userSchema); // or mapper(userSchema)(user)
```

## Build

```sh
npm install
npm run build
```

## Tests

```sh
npm install
npm test
```

## License

[MIT](LICENSE)