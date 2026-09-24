# App Calling Web SDK Getting Started

Use this guide to add the web SDK to a browser application.

## Before you start

- Choose one integration path:
  - install `app-calling-sdk` from npm and import `AppCalling` in your app
  - or serve the browser bundle and use the global `AppCalling` class on a script page
- Use a browser that supports WebRTC microphone access, `fetch`, `localStorage`, `crypto.randomUUID`, and `AbortSignal.timeout`.
- Make sure your backend can return a Calling JWT for the current user. See [AUTH_TOKEN.md](../AUTH_TOKEN.md) for how your backend should obtain it.

## 1. Install the SDK

Recommended for Angular, React, Vue, Vite, Webpack, and other bundler-based apps:

```sh
npm install app-calling-sdk
```

```ts
import { AppCalling } from 'app-calling-sdk';
```

If your app does not use a bundler, serve `node_modules/app-calling-sdk/dist/browser/app-calling.min.js`
as a static asset in your app and use the global `AppCalling` class:

```html
<script src="/vendor/app-calling.min.js"></script>
<script>
  await AppCalling.initialize({ environment: 'Uat' });
</script>
```

## 2. Initialize the SDK

`environment` is the only required initialization field.

If you want the built-in call dialog to show richer identity details, you can also pass these optional fields during initialization:

In the SDK API, pass them as `userName`, `designation`, `photoUrl`, and `companyName`. These map to the display fields `UserName`, `Designation`, `PhotoUrl`, and `CompanyName` used by the built-in call dialog.

- `userName`: display name associated with the current SDK session.
- `designation`: subtitle/role text shown in the call dialog.
- `photoUrl`: avatar image URL shown in the call dialog.
- `companyName`: company label used by the built-in UI.

Pass only the fields you have. Omit any optional field that you do not want to provide.

```ts
import { AppCalling } from 'app-calling-sdk';

await AppCalling.initialize({
  environment: 'Uat',
  userName: 'Aarav Mehta',
  designation: 'Relationship Manager',
  photoUrl: 'https://example.com/users/aarav-mehta.jpg',
  companyName: 'Ainxt Technovation Pvt. Ltd.',
});
```

The web SDK creates its built-in call UI during `initialize()`.

These fields are optional. This is also valid when you only want to pass one or two values:

```ts
await AppCalling.initialize({
  environment: 'Uat',
  userName: 'Aarav Mehta',
  photoUrl: 'https://example.com/users/aarav-mehta.jpg',
});
```

Do not pass these values to `activateUser()`, `activateAdmin()`, or `startCall()`. Provide them in `initialize()` so the SDK can use them for call-dialog display.

## 3. Activate the current user

Pass the Calling JWT returned by your backend. See [AUTH_TOKEN.md](../AUTH_TOKEN.md) for the backend token flow.

```ts
await AppCalling.activateUser(jwt);
// or
await AppCalling.activateAdmin(jwt);
```

Use `activateUser()` only with user tokens and `activateAdmin()` only with admin tokens.

## 4. Start a call

The current public outbound flow is for admin sessions. `toUserId` is required.
User-initiated support requests are not implemented in the current web SDK.

```ts
await AppCalling.instance.startCall({ toUserId: 'user-1' });
```

You can also pass an optional `message` and `metadata`.

## 5. Read presence and reachability

After activation, you can query user availability or subscribe to reachability changes:

```ts
const users = await AppCalling.instance.getReachableUsers('user');

const unsubscribe = AppCalling.subscribeReachableUsers((state) => {
  console.log(state.status, state.userIds);
});

const online = AppCalling.isUserReachable('user-1', 'user');

unsubscribe();
```

`subscribeReachableUsers()` is intended for admin presence views and requires an active admin session.

## 6. Go offline on logout

```ts
await AppCalling.instance.deactivate();
```

Call `dispose()` only when you want to tear down the SDK instance completely.

## Minimal integration example

```ts
import { Injectable } from '@angular/core';
import { AppCalling } from 'app-calling-sdk';

@Injectable({ providedIn: 'root' })
export class CallingService {
  async initialize(): Promise<void> {
    await AppCalling.initialize({ environment: 'Uat' });
  }

  async goOnline(jwt: string, role: 'user' | 'admin'): Promise<void> {
    if (role === 'admin') {
      await AppCalling.activateAdmin(jwt);
      return;
    }

    await AppCalling.activateUser(jwt);
  }

  async startAdminCall(toUserId: string): Promise<void> {
    await AppCalling.instance.startCall({ toUserId });
  }

  async goOffline(): Promise<void> {
    await AppCalling.instance.deactivate();
  }
}
```

## Notes

- `initialize()` can restore the last valid session if you did not call `deactivate()` before.
- The built-in UI is created automatically during `initialize()`. There is no public headless mode in the current web SDK.
- `acceptCall()` and `rejectCall()` are public methods, but the current web SDK does not expose a complete public inbound browser delivery flow.

For theme options and the full public API surface, see [ADVANCED_INTEGRATION.md](./ADVANCED_INTEGRATION.md).
