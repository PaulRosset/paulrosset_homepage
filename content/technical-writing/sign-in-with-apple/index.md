---
title: "Boost User Security and Convenience with “Sign in with Apple“ Integration on Your Website"
date: 2022-11-18T15:00:00+02:00
draft: false
slug: "technical-writing/sign-in-with-apple"
description: "In this blog post, we will see how we can integrate Apple authentication to our web app only based on the client side with JSON Web Token integration passed to the Back-end."
keywords:
  [
    "javascript",
    "apple",
    "coding",
    "code",
    "programming",
    "web",
    "api",
    "authentification",
    "sign-in",
    "security",
  ]
toc: true
---

When creating a SaaS application, authentification plays a significant role in ensuring users create value that can be seen and associated with a user account.

Most of the time, users will have to provide email and password combinations that tend to be less and less secure over time, as hackers are constantly inventing new ways to breach users’ accounts by social engineering.

Moreover, integrating authentication flow inside an app is redundant and presents many efforts. That’s why major companies like Apple or Google, with which you have many chances to have an account, are creating ways to authenticate users for you. This method is also more secure as both companies have developed methods to ensure your account remains safe and because they most likely have more resources to keep it secure.

In this blog post, we will see how we can integrate Apple authentication to our web app only based on the client side with JSON Web Token integration passed to the Back-end.

## SDK Loading

We’ll use the SDK provided by Apple.

To use the SDK, we must first download it. It's a javascript library that will live on the page's `window` object once downloaded through a `<script/>` element.

You can either get it thanks to a script element on your HTML's `<head/>`, but you can also insert the script by creating a `<script/>` attribute programmatically.

I prefer the second solution, which is much more flexible, particularly in today's world, where most javascript frameworks are component-based.

So you can have a component that will be busy dealing with the SDK download, so the SDK will download only when necessary.

- HTML based loading

```
<script
  type="text/javascript"
  src="https://appleid.cdn-apple.com/appleauth/static/jsapi/appleid/1/en_US/appleid.auth.js"
></script>
```

- Javascript based loading

```
await loadScript(
  "https://appleid.cdn-apple.com/appleauth/static/jsapi/appleid/1/en_US/appleid.auth.js"
);
```

> Note that the implementation of the **loadScript()** function can be found in the following [gist](https://gist.github.com/PaulRosset/4f51b1206ff368f3f0cb59a7ad3e28a3).

The Apple SDK is now loaded. Thanks to the `window` object, we can use it anywhere in our app.

## Implementation

Apple provides very detailed documentation, which is helpful, especially for knowing the data types that methods return.

The documentation is available at this address:

- [https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_js](https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_js)

The others pro of Apple SDK is that it provides a very flexible SDK that will allow anyone to design their button as their wish.

### Using SDK to authenticate users

The good news is that it’s straightforward to set up in terms of code.

### Initialization()

The first thing we need to do is to implement the `init` method.

```
window.AppleID.auth.init({
  clientId: "myapp.myapp.signin",
  scope: "email name",
  redirectURI: window.location.origin,
  state: "SignInUserAuthenticationRequest",
  usePopup: true,
});
```

Here, we passed several fields that have their importance.

- `clientId` → The `clientId` is the identifier that will help Apple identify the request to the Apple app you created. To get it from Apple, you must go through several steps we will define in the next section.
- `scope` → The `scope` is essential. You can ask Apple to return the name or/and email of the user that got authenticated. But it is also important to specify it as Apple will also insert it inside the payload of the **JsonWebToken** that we will use to auth the user. It’s handy for the Backend, which will create the user in the database with an email associated.
- `redirectURI` → Apple uses the `redirectURI` to redirect the user once he has finished the authentification process. You must pass the same URL as you will specify in the Apple console.
- `state` → The `state` is a property toward security. It’s a gentle way to identify the request. It allows the client to check that the request you get back from Apple is the one expected. For example, you can put whatever you want in that field, and when you get back the auth request from the user, the field will also be present. You can assert that the field is the same as you initially sent to ensure that someone has not tricked you.
- `usePopup`-> The `usePopup` feature is explicit. By using `true` here, it tells Apple to create a new browser window to authenticate the user. This property seems relatively straightforward but can lead to inconsistency when used in a Webview environment. We will get back to it.

As stated previously, you can find the interface and the method at the following URL:

- **[ClientConfigI](https://developer.apple.com/documentation/sign_in_with_apple/clientconfigi) interface**
- **[Init()](https://developer.apple.com/documentation/sign_in_with_apple/authi/3230945-init) method**

### Sign-in()

Once we initialize the SDK, we need to create a button that will react to a click event and start the Apple authentification flow.

```
button.addEventListener("click", async () => {
  await window.AppleID.auth.signIn();
});
```

Here, the **signIn()** method will trigger a popup where the user will be able to enter his credentials. During that phase, we no longer control anything. It's Apple's job.

- **[signIn()](https://developer.apple.com/documentation/sign_in_with_apple/authi/3261300-signin) method**

### Listening to success and error events to authenticate users

At the end of the flow, Apple needs to give back control with the **JsonWebToken** of the authenticated user. To achieve that, the SDK will emit a `CustomEvent` in the `DOM`.

It’s our job to listen to those events to react in consequence.

We need to handle two events: the **success** and the **error**.

- **Success**

```
document.addEventListener("AppleIDSignInOnSuccess", this._onAppleSignInOnSuccess);

_onAppleSignInOnSuccess(event) {
  const { state } = event.detail.authorization;
  const { email, name } = event.detail.user;
  // We are checking that the request we send matches the one we receive.
  if (state === "SignInUserAuthenticationRequest") {
    const { code, id_token } = event.detail.authorization;
    // Do something with id_token and code and email/name properties
  } else {
    this.onError(new Error('state property is not the one expected'));
  }
}
```

Here, we are listening to the `AppleIDSignInOnSuccess` event that Apple may trigger when the user successfully authenticates.

Apple gives back multiples elements:

- **[SignInResponseI](https://developer.apple.com/documentation/sign_in_with_apple/signinresponsei) Class**

  - `authorization` **[AuthorizationI](https://developer.apple.com/documentation/sign_in_with_apple/authorizationi) interface**
    - `state` → Do you remember? It’s the one we passed earlier in the **[ClientConfigI](https://developer.apple.com/documentation/sign_in_with_apple/clientconfigi) interface.** The condition makes sure that we receive the one we sent earlier.
    - `code` → A single-use authorization code.
    - `id_token` → This is the most precious piece of information we get back from Apple here. We will use the **JsonWebToken** to authenticate the user on the Back-end.
  - `user` **[UserI](https://developer.apple.com/documentation/sign_in_with_apple/useri) interface**
    - `email`
    - `name`

- **Error**

```
document.addEventListener("AppleIDSignInOnFailure", this._onAppleSignInOnFailure);

_onAppleSignInOnFailure(event) {
  let { error } = event.detail;
  this.onError(new Error(error));
}
```

We get back an error from Apple when, for some reason, the user did not successfully register. Errors rarely happen because if the user enters the wrong credentials, the error will be on Apple's side and not ours.
Note that when the user closes the popup explicitly, Apple triggers an error:

- `popup_closed_by_user`

In your sense, it may not be a proper error but a potential situation to leverage tracking for example.

### Setting up App through the Apple developer interface

Here comes the essential configuration we have to set up to correctly use the “Sign in with Apple” feature.

You first need an Apple developer account. To have it, you need to pay a subscription to Apple, which is 100$ per year.

The setup will happen on this page:

- [https://developer.apple.com/account/](https://developer.apple.com/account/)

You can follow the video below:

[https://youtu.be/9J5vgxspMhQ](https://youtu.be/9J5vgxspMhQ)

The crucial part is the identifier highlighted at the end of the video:

- `com.example.signin.test.with.apple.identifier`

That identifier is mandatory to have and use as the [`clientId`](https://developer.apple.com/documentation/sign_in_with_apple/clientconfigi/3230948-clientid).

The domains, subdomains, and return URLs are also essential, as your application won’t work if you don’t specify the URL it will run on. Indeed, many misconfigurations can lead to hours of debugging when the error is “configuration”.

### Experienced issues while developing

This section will state the issues you could encounter to gain you time.

In the previous chapter, we configured the App, so we made sure that Apple would know that the following clientId: `com.example.signin.test.with.apple.identifier` has the correct information to work correctly.

Again, that information is critical because your App could not work just because of this configuration.

When integrating the "Sign-in with Apple" feature, we needed to make sure that it will work inside a Webview. Since the App is misconfigured, the Apple SDK was returning an error from the `Error` event with the following payload:

- `popup_closed_by_browser`

The error was not too explicit, so we thought it was related to the property we specified: [`usePopup`](https://developer.apple.com/documentation/sign_in_with_apple/clientconfigi/3530376-usepopup).

So, we tried to set it to `false`, and it worked since the error was gone. But the misconfiguration triggered another error on the Apple side. The redirect URL parameter was misconfigured at that time, so we went to the developer tools interface and configured the correct redirects. Then, we thought that everything would work. But it wasn’t the case, as Apple was reloading the page cause of the misuse of the property `usePopup`, so we went back on this property to set it to `true` that time with the correct configuration, and finally, it worked.

To debug the Webview inside the App, we used Safari and the iOS simulator inside XCode.

Here are the steps to have it properly working:

- Open Safari  
  → Click on Safari on the top left bar  
  → Preferences  
  → Tick the checkbox “Show Develop menu in the menu bar.”
- Then, head to XCode, and create a new iOS project from Storyboard. Insert the following code into the ViewController file.
  - [Gist code file](https://gist.github.com/PaulRosset/cea982e09d09e609679b7e848e397674).
- Once you built and started the project inside the simulator, you can return to Safari. At the top of the screen in the top bar, click “Develop” you should see the simulator devices appearing, click on your simulator, and finally click on the Webview to spawn the Safari developer tool.

This hint for debugging is only toward iOS Webview usage. But having an environment where the developer is comfortable debugging is essential.

## Conclusion

Integrating a social provider into a front-end app is a pleasant experience for a developer. The development experience will likely be appreciated at work as integrating social providers is spread.

A social provider eases the authentification part, which is redundant for users as it's a mandatory step for using a service. Hence, easing it will contribute to reducing the friction by eliminating the need for users to enter email/password + email confirmation step. It also eliminates the need to remember passwords and removes the associated challenges, improving the signup and sign-in user experience.
