# App Calling Web SDK Advanced Integration

This guide covers the full public web SDK surface that integrators can use.

## Current scope

- The built-in web overlay is always created during `initialize()`.
- Admin sessions support outbound calls and incoming calls while the browser tab is open in the foreground.

## Install and runtime requirements

- Install the package with `npm install app-calling-sdk` for module-based apps.
- Optional plain-browser integration: serve `node_modules/app-calling-sdk/dist/browser/app-calling.min.js` as a static asset and use the global `AppCalling` class.
- Browser support: WebRTC microphone access, `fetch`, `localStorage`, `crypto.randomUUID`, `AbortSignal.timeout`, Custom Elements, and Shadow DOM
- App requirement: a Calling JWT returned by your backend. See [AUTH_TOKEN.md](../AUTH_TOKEN.md) for how your backend should obtain it.

## Initialization

```ts
AppCalling.initialize(config, theme?)
```

Returns `Promise<AppCalling>`. The SDK creates or reuses the singleton facade and automatically creates the built-in web overlay.

| Argument | Type | Required | Notes |
| --- | --- | --- | --- |
| `config.environment` | `'Uat' | 'Live'` | Yes | Use `'Uat'` before go-live. The checked-in SDK currently only wires `Uat` to an API base URL. |
| `config.apiBaseUrl` | `string` | No | API URL provided for your integration; otherwise uses the configured environment endpoint |
| `config.userName` | `string` | No | Display name associated with the current SDK session |
| `config.designation` | `string` | No | Role/subtitle shown in the built-in call dialog |
| `config.photoUrl` | `string` | No | Avatar image URL shown in the built-in call dialog |
| `config.companyName` | `string` | No | Company label used by the built-in UI when shown |
| `theme` | `Partial<AppCallingTheme>` | No | Overrides the built-in overlay theme |

If you set `apiBaseUrl`, use an absolute HTTP(S) URL (HTTPS recommended) that does not end with `/api`. Your application backend must use the corresponding API URL when obtaining Calling JWTs.

```typescript
await AppCalling.initialize({
  environment: 'Uat',
  apiBaseUrl: 'https://api.example.com',
});
```

`AppCallingTheme` fields:

- `primaryColor`
- `dangerColor`
- `successColor`
- `backgroundColor`
- `textColor`
- `borderRadius`
- `fontFamily`

Example:

```ts
await AppCalling.initialize(
  {
    environment: 'Uat',
    userName: 'Aarav Mehta',
    designation: 'Relationship Manager',
    photoUrl: 'https://example.com/users/aarav-mehta.jpg',
    companyName: 'Ainxt Technovation Pvt. Ltd.',
  },
  {
    primaryColor: '#0057ff',
    borderRadius: '16px',
  },
);
```

Use the camelCase SDK config keys shown above. They map to the built-in dialog display fields `UserName`, `Designation`, `PhotoUrl`, and `CompanyName`.

All four display fields are optional. Pass only the ones you have, and leave the rest out of `config`.

Module example:

```ts
import { AppCalling } from 'app-calling-sdk';

const sdk = await AppCalling.initialize({ environment: 'Uat' });
```

Plain browser example:

```html
<script src="/vendor/app-calling.min.js"></script>
<script>
  AppCalling.initialize({ environment: 'Uat' })
    .then((sdk) => { window.callingSdk = sdk; })
    .catch(console.error);
</script>
```

## Activation and lifecycle

Public activation methods:

- `AppCalling.activateUser(jwt)`
- `AppCalling.activateAdmin(jwt)`

Public instance access:

- `AppCalling.instance`
- `AppCalling.maybeInstance`

Static convenience helpers:

- `AppCalling.subscribeReachableUsers(callback)`
- `AppCalling.isUserReachable(userId, role?)`
- `AppCalling.instance.onLifecycleStateChanged(callback)` returns `Unsubscribe`

Lifecycle states:

- `NotInitialized`
- `Initialized`
- `Active`
- `Disposed`

Integration rules:

- Call `initialize()` before any other SDK API.
- The JWT role must match the activation method you call.
- `initialize()` can restore a previously saved valid session.
- Call `deactivate()` on logout to stop presence and prevent automatic restore.
- Call `dispose()` when your app is removing the SDK completely.
- The SDK will not replace the active session while a call is in progress.

## Reachability helpers

You can listen for reachable-user changes without polling:

```ts
const unsubscribe = AppCalling.subscribeReachableUsers((state) => {
  console.log(state.status, state.userIds, state.updatedAt);
});

const canCall = AppCalling.isUserReachable('user-1', 'user');

unsubscribe();
```

Notes:

- `subscribeReachableUsers()` requires an active admin session.
- `isUserReachable()` reads the in-memory reachability cache, so initialize and activate the SDK first.

## Call APIs

| API | What to use it for | Notes |
| --- | --- | --- |
| `startCall({ toUserId, message, metadata })` | Start an outbound call | Current public flow is for admin sessions; `toUserId` is required |
| `endCall()` | End the current call | Requires an active call |
| `acceptCall()` | Accept a ringing call | Available to active admin sessions with an incoming call |
| `rejectCall()` | Reject a ringing call | Available to active admin sessions with an incoming call |
| `toggleMute()` | Toggle microphone mute | Returns the new mute state |
| `isMuted()` | Read local mute state | Local state only |

Call operations require an active session created through `activateUser()` or `activateAdmin()`.

### Incoming calls and ringtone

An active admin session checks for incoming calls while its tab is open in the foreground. The built-in overlay displays a ringing call. Browser tabs do not receive background call delivery.

The SDK attempts to enable the ringtone during admin activation and again on the first user interaction. Browser audio policy may require a click or key press before sound plays. To enable it from your own button, call `enableRingtone()` directly in the button handler:

```ts
const sdk = AppCalling.instance;
const enabled = await sdk.enableRingtone(); // Promise<boolean>
sdk.setRingtoneVolume(0.75); // 0 (silent) to 1 (full); default 0.75
const ringtoneEnabled = sdk.isRingtoneEnabled(); // boolean
const unsubscribe = sdk.onRingtoneAvailabilityChanged((enabled: boolean) => {
  console.log('Ringtone available:', enabled);
}); // Unsubscribe
```

Call `unsubscribe()` when the listener is no longer needed. `setRingtoneVolume()` returns `void` and throws `RangeError` for values outside 0–1.

## Query APIs

| API | Return |
| --- | --- |
| `getActiveCall()` | `CallData | null` |
| `isReady()` | `boolean` |
| `isActive()` | `boolean` |
| `getLifecycleState()` | `SdkLifecycleState` |
| `getCallHistory({ limit, startDate, endDate })` | `Promise<CallData[]>` |
| `getCallRecording(callId)` | `Promise<string | null>` |
| `getPresentUsers(role?)` | `Promise<string[]>` |
| `getReachableUsers(role?)` | `Promise<string[]>` |
| `getUserStatus(userId)` | `Promise<UserPresenceStatus>` |

Query APIs require `initialize()` and an activated Calling JWT session.

## Useful public types

- `AppEnvironment`
- `AppCallingConfig`
- `AppCallingTheme`
- `StartCallOptions`
- `CallHistoryOptions`
- `CallData`
- `CallMetadata`
- `CallingSession`
- `ReachableUsersCallback`
- `ReachableUsersState`
- `ReachableUsersStatus`
- `Unsubscribe`
- `UserPresenceStatus`
- `DevicePresenceInfo`
- `DeviceType`
- `DevicePresenceState`
- `CallStatus`
- `SdkLifecycleState`

## Public errors

The SDK exports these error classes:

- `CallingException`
- `AuthenticationException`
- `PermissionDeniedException`
- `NetworkException`

## Integration notes

- Fetch Calling JWTs through your backend, not from the browser directly. See [AUTH_TOKEN.md](../AUTH_TOKEN.md).
- Keep logout wired to `deactivate()`.
- Keep an active admin tab in the foreground for incoming browser calls and allow a user gesture to enable ringtone audio.
