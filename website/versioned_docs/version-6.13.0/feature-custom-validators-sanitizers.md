---
id: version-6.13.0-custom-validators-sanitizers
title: Custom validators/sanitizers
original_id: custom-validators-sanitizers
---

Although express-validator offers plenty of handy validators and sanitizers through its underlying
dependency [validator.js](https://github.com/validatorjs/validator.js), it doesn't always suffice when
building your application.

For these cases, you may consider writing a custom validator or a custom sanitizer.

## Custom validator

A custom validator may be implemented by using the chain method [`.custom()`](api-validation-chain.md#customvalidator).
It takes a validator function.

Custom validators may return Promises to indicate an async validation (which will be awaited upon),
or `throw` any value/reject a promise to [use a custom error message](feature-error-messages.md#custom-validator-level).

> **Note:** if your custom validator returns a promise, it must reject to indicate that the field is invalid.

### Example: checking if e-mail is in use

<!--DOCUSAURUS_CODE_TABS-->
<!--JavaScript-->

```js
const { body } = require('express-validator');

app.post(
  '/user',
  body('email').custom(value => {
    return User.findUserByEmail(value).then(user => {
      if (user) {
        return Promise.reject('E-mail already in use');
      }
    });
  }),
  (req, res) => {
    // Handle the request
  },
);
```

<!--TypeScript-->

```js
import { body, CustomValidator } from 'express-validator';
// This allows you to reuse the validator
const isValidUser: CustomValidator = value => {
  return User.findUserByEmail(value).then(user => {
    if (user) {
      return Promise.reject('E-mail already in use');
    }
  });
};

app.post('/user', body('email').custom(isValidUser), (req, res) => {
  // Handle the request
});
```

<!--END_DOCUSAURUS_CODE_TABS-->

> **Note:** In the example above, validation might fail even due to issues with fetching User information. The implications of accessing the data layer during validation should be carefully considered.

### Example: checking if password confirmation matches password

```js
const { body } = require('express-validator');

app.post(
  '/user',
  body('password').isLength({ min: 5 }),
  body('passwordConfirmation').custom((value, { req }) => {
    if (value !== req.body.password) {
      throw new Error('Password confirmation does not match password');
    }

    // Indicates the success of this synchronous custom validator
    return true;
  }),
  (req, res) => {
    // Handle the request
  },
);
```

## Custom sanitizers

Custom sanitizers can be implemented by using the method `.customSanitizer()`, no matter if
the [validation chain one](api-validation-chain.md#customsanitizersanitizer) or
the [sanitization chain one](api-sanitization-chain.md#customsanitizersanitizer).  
Just like with the validators, you specify the sanitizer function, which _must_ be synchronous at the
moment.

### Example: converting to MongoDB's ObjectID

<!--DOCUSAURUS_CODE_TABS-->
<!--JavaScript-->

```js
const { param } = require('express-validator');

app.post(
  '/object/:id',
  param('id').customSanitizer(value => {
    return ObjectId(value);
  }),
  (req, res) => {
    // Handle the request
  },
);
```

<!--TypeScript-->

```typescript
import { param } from 'express-validator';
// This allows you to reuse the validator
const toObjectId: CustomSanitizer = value => {
  return ObjectId(value);
};

app.post('/object/:id', param('id').customSanitizer(toObjectId), (req, res) => {
  // Handle the request
});
```

<!--END_DOCUSAURUS_CODE_TABS-->
