# App Calling SDK Integration Guides

This folder contains the public integration guides for the two SDKs in this repository:

- Web SDK for browser applications
- Flutter SDK for Flutter applications

## Common concepts

- Your app must obtain a Calling JWT from your own backend. The SDKs do not create tokens on the client. See [AUTH_TOKEN.md](./docs/AUTH_TOKEN.md) for the backend token flow.
- The usual lifecycle is `initialize -> activate -> use calling APIs -> deactivate on logout -> dispose when you no longer need the SDK`.
- On web, the public surface is the `AppCalling` singleton facade: call `initialize()`, then `activateUser()` or `activateAdmin()`, then use `AppCalling.instance` or the returned SDK object.
- Use the user activation method for user tokens and the admin activation method for admin tokens.
- Presence means the SDK is currently active for that user.
- Reachability means the platform can still route a call to one of that user's devices.

## Shared backend guide

- [AUTH_TOKEN.md](./docs/AUTH_TOKEN.md): explains how your backend should exchange `ApiKey` and `ApiSecret` for a Calling JWT, then return that JWT to the frontend for `activateUser()` or `activateAdmin()`.


## Choose a guide

| SDK | Best for | Guides |
| --- | --- | --- |
| Web | Browser applications that use the built-in web calling UI through the npm package or browser bundle | [Getting Started](./docs/web/GETTING_STARTED.md), [Advanced Integration](./docs/web/ADVANCED_INTEGRATION.md) |
| Flutter | Flutter applications. The documented path in this repository is Android. | [Getting Started](./docs/flutter/GETTING_STARTED.md), [Advanced Integration](./docs/flutter/ADVANCED_INTEGRATION.md) |

## Main differences

- The web SDK always creates its built-in UI during `initialize()`. There is no public headless mode.
- The Flutter SDK supports built-in UI and headless mode, and it exposes streams for custom UI.
- The Flutter SDK includes Android incoming-call integration with Firebase Cloud Messaging.
- The current public web integration path is centered on activation, presence, reachability helpers, and admin outbound calling.
