# Firebase Auth API

As a community we will fill this out but for now it's just the Authentication module.

We are trying to keep things as separate as possible so we

## Project - Firebase

Exists only to add the correct cocoapod and gradle dependencies to the build.

Exposes no functionality to JS

## Project - Firebase.Authentication

This project is the meat of the Firebase Authentication API from a JS perspective. It provides all the properties, functions and events that work across all providers.

Firebase's client API allows only one user to be logged in at a time, so rather than have some kind of handle to the user that could expire we have a single static user that may or may not be authenticated.

Firebase's authentication api has a number of 'providers', we current support 3 of them: Google, Facebook & Email

Google and Facebook authentication involves using the facebook/google apis to authenticate the user, this then returns a token that is then used to authenticate against firebase. If you quit your app the token is maintain so that you will be automatically logged in when you next load the app.

Email authentication is provided by Firebase themselves and as such there is no 2 stage login or token. There is also no automatic reauthentication when returning to the app.

### Requiring the JS API

To get started we need to require the library

```js
var FirebaseUser = require("Firebase/Authentication/User");
```

We can now use the following on `FirebaseUser`

### Property - isSignedIn

This returns a boolean indicating whether the User is currently Authenticated with a provider or not.

#### Example
```js
var logUserIfAuthenticated = function() {
    if (FirebaseUser.isSignedIn) {
        console.log(FirebaseUser.name);
        console.log(FirebaseUser.email);
        console.log(FirebaseUser.photoUrl);
    }
};
```

### Event - signedInStateChanged

This event is called any time the state of whether the user is signed-in or not changes.

To clarify, this event will be called if:

- The User was not authenticated but now is
- The User was authenticated but now isn't

#### Example
```js
FirebaseUser.signedInStateChanged = function() {
    if (FirebaseUser.isSignedIn)
        console.log("I'm signed-in now!");
    else
        console.log("I'm not signed-in anymore");
};
```

### Function - signOut()

If the user is signed in then we sign them out from:

- Firebase
- The provider (unless the provider was 'Email' which is provided by firebase)

#### Example
```js
var signOutNow = function() {
    FirebaseUser.signOut();
};
```

### Function - reauthenticate() / reauthenticate(email, password)

From the firebase documentation:

> Some security-sensitive actions—such as deleting an account, setting a primary email address, and changing a password—require that the user has recently signed in. If you perform one of these actions, and the user signed in too long ago, the action fails and throws FirebaseAuthRecentLoginRequiredException. When this happens, re-authenticate the user

To do this from JS is simple but varies slightly if you logged in using the `Email` provider

- For Google/Facebook Providers simply call `reauthenticate()`
- For the Email provider call `reauthenticate(emailAddress, password)` where emailAddress & password are strings.

`reauthenticate` returns a promise of a `string`

#### Examples
```js
// For Google/Facebook
//
FirebaseUser.reauthenticate().then(function(message) {
    console.log("reauthenticated");
}).catch(function(e) {
    console.log("reauthentication failed:" + e);
});

// For Email
//
FirebaseUser.reauthenticate(userEmailAddress, userPassword).then(function(message) {
    console.log("reauthenticated");
}).catch(function(e) {
    console.log("reauthentication failed:" + e);
});

```


### Event - onError

The `onError` event is called when one of the `UX` Firebase provider login buttons. Throws an error

Usually we would prefer to use a `promise` to an `event` but as the login for Facebook/Google is initiated from the button then there is no place to return a `promise` to.

Please see [Issue #1](https://github.com/cbaggers/Fuse.Firebase/issues/1) for a potential fix for this.

#### Example
```js
FirebaseUser.onError = function(errorMsg) {
    console.log("ERROR(" + errorCode + "): " + errorMsg);
    statusText.value = "Error: " + errorMsg;
};
```


### Property - name

This property returns the Firebase name of the authenticated user

#### Example
```js
console.log(FirebaseUser.name);
```


### Property - email

This property returns the Firebase email address of the authenticated user

#### Example
```js
console.log(FirebaseUser.email);
```


### Property - photoUrl

This property returns the url on the photo Firebase associates with the authenticated user

#### Example
```js
console.log(FirebaseUser.photoUrl);
```


### Function - updateProfile(newDisplayNameString, newPhotoUrlString)

This function attempts to update the `displayName` and `photoUrl` of the authenticated user.

It returns a promise of a `string`

#### Example
```js
FirebaseUser.updateProfile(FirebaseUser.name, newPhotoUrl);
```


### Function - updateEmail(newEmailAddressString)

This function attempts to update the primary email address that Firebase associates with the authenticated user.

It returns a promise of a `string`

#### Example
```js
FirebaseUser.updateProfile(newEmailAddress);
```


### Function - updateEmail(newEmailAddressString)

This function attempts to delete the currently authenticated user from Firebase.

It returns a promise of a `string`

#### Example
```js
FirebaseUser.delete();
```


## Project - Firebase.Authentication.Email

This project initializes the Email provider and registers it with `Firebase.Authentication`.

It can be required from JS with:

```js
var EmailAuth = require("Firebase/Authentication/Email");
```

It exposes 2 functions to JS:

### Function - createWithEmailAndPassword(email, password)

This function attempts to create a new firebase users with the given email address and password.

It returns a promise of a `string`

If it succeeds it will also sign-in that user.

#### Example
```js
var createUser = function() {
    var email = userEmailInput.value;
    var password = userPasswordInput.value;
    EmailAuth.createWithEmailAndPassword(email, password).then(function(user) {
        signedIn();
    }).catch(function(e) {
        console.log("Signup failed: " + e);
        FirebaseUser.onError(e, -1);
    });
};
```

### Function - signInWithEmailAndPassword(email, password)

This function attempts to sign-in the firebase user with the given email address and password.

It returns a promise of a `string`

#### Example
```js
var signInWithEmail = function() {
    var email = userEmailInput.value;
    var password = userPasswordInput.value;
    EmailAuth.signInWithEmailAndPassword(email, password).then(function(user) {
        signedIn();
    }).catch(function(e) {
        console.log("SignIn failed: " + e);
        FirebaseUser.onError(e, -1);
    });
};
```

### Function - updateEmail(newEmail)

This function updates the currently signed in user's email

> Important. To set a user's email, the user must have signed in recently.

#### Example
```js
var updateEmail = function() {
    EmailAuth.updateEmail(newEmail)
    .then(function() {
        console.log("Success!")
    }).catch(function(e) {
        console.log("Error: " + e);
    });
};
```

### Function - updatePassword(newPassword)

This function updates the currently signed in user's password

> Important. To set a user's password, the user must have signed in recently.

#### Example
```js
var updatePassword = function() {
    EmailAuth.updatePassword(newPassword)
    .then(function() {
        console.log("Success!")
    }).catch(function(e) {
        console.log("Error: " + e);
    });
};
```

### Function - sendVerificationEmail()

This function sends the current signed in user an email to verify the account.

#### Example
```js
var sendVerificationEmail = function() {
    EmailAuth.sendVerificationEmail()
    .then(function(res) {
        console.log("Success! " +res)
    }).catch(function(e) {
        console.log("Error: " + e);
    });
};
```

### Function - sendPasswordResetEmail(email)

This function sends an email to allow the user to reset their password if they've forgotton it.

#### Example
```js
var sendPasswordResetEmail = function() {
    EmailAuth.sendPasswordResetEmail(email)
    .then(function(res) {
        console.log("Succes! " + res);
    }).catch(function(e) {
        console.log("Error: " + e);
    });
}
```

### Property - isEmailVerified

This property returns a boolean representing whether the user has verified their email or not by following the link sent by `sendVerificationEmail()`

> Important. The user must be reauthenticated before trying to access this property to have the latest values.

#### Example
```js
console.log(FirebaseUser.isEmailVerified);
```

## Project - Firebase.Authentication.Google

This project exists to initialize the Google provider and register it with `Firebase.Authentication`.

It exposes no functionality to JS itself, however it does provide the `Firebase.Authentication.Google.GoogleButton` that can be used in UX to initiate login


## Project - Firebase.Authentication.Facebook

This project exists to initialize the Google provider and register it with `Firebase.Authentication`.

It exposes no functionality to JS itself, however it does provide the `Firebase.Authentication.Facebook.FacebookButton` that can be used in UX to initiate login
