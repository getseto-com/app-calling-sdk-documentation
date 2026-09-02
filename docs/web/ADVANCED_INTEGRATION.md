# App Calling Web SDK Advanced Integration

This guide covers the full public web SDK surface that integrators can use.

## Current scope

- The built-in web overlay is always created during `initialize()`.
- The current public calling flow is strongest for activation, presence, and admin outbound calling.
- The SDK exposes `acceptCall()` and `rejectCall()`, but it does not expose a complete public inbound browser delivery path.

## Install and runtime requirements

- Package: `app-calling-sdk`
- Browser support: WebRTC microphone access, `fetch`, `localStorage`, `crypto.randomUUID`, `AbortSignal.timeout`, Custom Elements, and Shadow DOM
- App requirement: a Calling JWT returned by your backend. See [AUTH_TOKEN.md](../AUTH_TOKEN.md) for how your backend should obtain it.

## Initialization

```ts
AppCalling.initialize(config, theme?)
```

| Argument | Type | Required | Notes |
| --- | --- | --- | --- |
| `config.environment` | `'Uat' | 'Live'` | Yes | Use `'Uat'` before go-live |
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
  { environment: 'Uat' },
  {
    primaryColor: '#0057ff',
    borderRadius: '16px',
  },
);
```

## Activation and lifecycle

Public activation methods:

- `AppCalling.activateUser(jwt)`
- `AppCalling.activateAdmin(jwt)`

Public instance access:

- `AppCalling.instance`
- `AppCalling.maybeInstance`

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

## Call APIs

| API | What to use it for | Notes |
| --- | --- | --- |
| `startCall({ toUserId, message, metadata })` | Start an outbound call | Current public flow is for admin sessions; `toUserId` is required |
| `endCall()` | End the current call | Requires an active call |
| `acceptCall()` | Accept a ringing call | Public API exists, but inbound browser delivery is not publicly wired end to end |
| `rejectCall()` | Reject a ringing call | Same limitation as `acceptCall()` |
| `toggleMute()` | Toggle microphone mute | Returns the new mute state |
| `isMuted()` | Read local mute state | Local state only |

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

## Useful public types

- `AppCallingConfig`
- `AppCallingTheme`
- `StartCallOptions`
- `CallHistoryOptions`
- `CallData`
- `CallingSession`
- `UserPresenceStatus`
- `DevicePresenceInfo`
- `CallStatus`
- `SdkLifecycleState`

## Public errors

The SDK exports these error classes:

- `CallingException`
- `AuthenticationException`
- `PermissionDeniedException`
- `NetworkException`

## Integration notes

- Fetch Calling JWTs through your backend, not from the browser directly. See [AUTH_TOKEN.md](../auht-token/AUTH_TOKEN.md).
- Keep logout wired to `deactivate()`.
- Plan your web integration around the current public capabilities: activation, built-in UI, presence, and admin outbound calling.
