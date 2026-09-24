# App Calling Web SDK Advanced Integration

This guide covers the full public web SDK surface that integrators can use.

## Current scope

- The built-in web overlay is always created during `initialize()`.
- The current public calling flow is strongest for activation, presence, and admin outbound calling.
- The SDK exposes `acceptCall()` and `rejectCall()`, but it does not expose a complete public inbound browser delivery path.

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
| `config.userName` | `string` | No | Display name associated with the current SDK session |
| `config.designation` | `string` | No | Role/subtitle shown in the built-in call dialog |
| `config.photoUrl` | `string` | No | Avatar image URL shown in the built-in call dialog |
| `config.companyName` | `string` | No | Company label used by the built-in UI when shown |
| `theme` | `Partial<AppCallingTheme>` | No | Overrides the built-in overlay theme |

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
  const sdk = await AppCalling.initialize({ environment: 'Uat' });
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
| `acceptCall()` | Accept a ringing call | Public API exists, but inbound browser delivery is not publicly wired end to end |
| `rejectCall()` | Reject a ringing call | Same limitation as `acceptCall()` |
| `toggleMute()` | Toggle microphone mute | Returns the new mute state |
| `isMuted()` | Read local mute state | Local state only |

Call operations require an active session created through `activateUser()` or `activateAdmin()`.

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
- Plan your web integration around the current public capabilities: activation, built-in UI, presence, reachability helpers, and admin outbound calling.
