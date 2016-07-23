# Tada Client JavaScript Library

Version 0.1.0

## Library Components

* [Analytics](#analytics): Track user actions.
* [Auth](#auth): Authenticate users.
* [Data](#data): Create and modify App data.

All library functions return ES6-compliant promises, and will alternatively respond to an idiomatic callback if one is provided as a final argument.

Example:

```js
const tada = new Tada('https://myapp.tadadb.com');

// Callback
tada.authWithPassword('admin@myapp.io', 'Password1!', function (err, auth) {
  if (err) {
    console.log(err);
  } else {
    console.log(auth)
  }
});

// Promise
try {
  const auth = await tada.authWithPassword('admin@myapp.io', 'Password1!');
  console.log(auth);
} catch (err) {
  console.log(err);
}
```

## Analytics

### track(event, attributes, [callback])

Tracks an analytics event.

Arguments:

* `event` **String**: The name of the event being tracked
* `attributes` **Object**: An object of key/value pairs being tracked with this event
* `[callback]` **Function** *Optional*: A function to call after tracking this event

Returns: The event tracking result.

```js
const tada = new Tada('https://myapp.tadadb.com');
const attributes = { path: "/profile", referrer: "https://www.referringsite.com", key1: 10, key2: true };

// Callback
tada.track('pageview', attributes, function (err, result) {
  if (err) {
    console.log(err);
  } else {
    console.log(result);
  }
});

// Promise
try {
  const result = await tada.track('pageview', attributes);
} catch (err) {
  console.log(err);
}
```

## Auth

### authAnonymously([callback])

Anonymously authenticates a user.

Arguments:

* `[callback]` **Function** *Optional*: A function to call after user authentication has completed

Returns: The user's auth object.

```js
const tada = new Tada('https://myapp.tadadb.com');

// Callback
tada.authAnonymously(function (err, auth) {
  if (err) {
    console.log(err);
  } else {
    console.log(auth);
  }
}

// Promise
try {
  const auth = await tada.authAnonymously();
  console.log(auth);
} catch (err) {
  console.log(err);
}
```

### authWithOAuthRedirect(provider, [callback])

Authenticates a user with OAuth2 redirect.

Arguments:

* `provider` **String**: The OAuth2 provider to use for authentication. Valid providers: `facebook`, `github`
* `[callback]` **Function** *Optional*: A function to call after user authentication has completed

Returns: The user's auth object.

```js
const tada = new Tada('https://myapp.tadadb.com');

// Callback
tada.authWithOAuthRedirect('facebook', function (err, auth) {
  if (err) {
    console.log(err);
  } else {
    console.log(auth);
  }
}

// Promise
try {
  const auth = await tada.authWithOAuthRedirect('facebook');
  console.log(auth); // Will never be called due to redirect
} catch (err) {
  console.log(err);
}
```

### authWithPassword(email, password, [callback])

Authenticates a user with email and password.

Arguments:

* `email` **String**: The user's email address
* `password` **String**: The user's password
* `[callback]` **Function** *Optional*: A function to call after user authentication has completed

Returns: The user's auth object.

```js
const tada = new Tada('https://myapp.tadadb.com');

// Callback
tada.authWithPassword('admin@myapp.io', 'Password1!', function (err, auth) {
  if (err) {
    console.log(err);
  } else {
    console.log(auth)
  }
});

// Promise
try {
  const auth = await tada.authWithPassword('admin@myapp.io', 'Password1!');
  console.log(auth);
} catch (err) {
  console.log(err);
}
```

### getAuth()

Gets the current user auth object.

Returns: The current user's auth object if the user is authenticated, `null` otherwise.

```js
const tada = new Tada('https://myapp.tadadb.com');
const auth = tada.getAuth();
console.log(auth);
```

**unauth()**
Unauthenticates the user, removing their locally-stored auth object.

Returns: **void**

```js
const tada = new Tada('https://myapp.tadadb.com');
tada.unauth();
console.log(tada.getAuth()); // null
```

## Data

### get([callback])

Gets App data from the specified database location.

Arguments:

* `[callback]` **Function** *Optional*: A function to call after data have been retrieved

Returns **Object**: App data from the specified database location

```js
const tada = new Tada(`https://myapp.tadadb.com/some/database/location`);

// Callback
tada.get(function (err, data) {
  if (err) {
    console.log(err);
  } else {
    console.log(data); // Prints data from /some/database/location
  }
});

// Promise
try {
  const data = await tada.get();
  console.log(data); // Prints data from /some/database/location
} catch (err) {
  console.log(err);
}
```

### set(datum, [callback])

Sets the datum at the specified location, i.e. sets the value for a single key.

Arguments:

* `datum` **Any**: The datum to set at the specified endpoint
* `[callback]` **Function** *Optional*: A function to call after the datum has been set

```js
const tada = new Tada(`https://myapp.tadadb.com/some/database/location/key`);

// Callback
tada.set('value', function (err, result) {
  if (err) {
    console.log(err);
  } else {
    console.log(result); // Prints the result of setting 'value' at /some/database/location/key
  }
});

// Promise
try {
  const result = await tada.set('value');
  console.log(result); // Prints the result of setting 'value' at /some/database/location/key
} catch (err) {
  console.log(err);
}
```

### update(data, [callback]

Updates data at the specified location, i.e. sets an object at the specified App database location.

Arguments:

* `data` **Object**: The data to set at the specified endpoint
* `[callback]` **Function** *Optional*: A function to call after the data have been set

```js
const tada = new Tada(`https://myapp.tadadb.com/some/database/location`);
const data = { key: "value" };

// Callback
tada.set(data, function (err, result) {
  if (err) {
    console.log(err);
  } else {
    console.log(result); // Prints the result of setting {data} at /some/database/location
  }
});

// Promise
try {
  const result = await tada.set(data);
  console.log(result); // Prints the result of setting {data} at /some/database/location
} catch (err) {
  console.log(err);
}
```