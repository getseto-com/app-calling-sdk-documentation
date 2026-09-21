# Flutter Calling SDK Advanced Integration

This guide covers the full public Flutter SDK surface that integrators can use.

## Current scope

- The documented integration path in this repository is Android.
- Built-in UI is the default, and `UiMode.headless` is available for custom UI.
- The current public outbound flow is for admin sessions.

## Required app setup

Your Flutter app should provide:

- `flutter_calling_sdk` and `firebase_core`
- Firebase initialization during startup
- `SipCalling.registerBackgroundHandlers()` before `runApp()`
- Android Firebase configuration and Google Services plugin
- Android `minSdk` 24 or higher
- A `NavigatorKey` for built-in UI, or `SipCalling.navigatorKey`
- A Calling JWT from your backend. See [AUTH_TOKEN.md](../AUTH_TOKEN.md) for how your backend should obtain it.
- No separate Android notification-permission workflow for calling; the SDK requests notification permission during activation when needed.

## Initialization

```dart
SipCalling.initialize(
  config,
  fcmToken: fcmToken,
  navigatorKey: appNavigatorKey,
  uiMode: UiMode.builtIn,
  theme: theme,
)
```

| Argument | Type | Required | Notes |
| --- | --- | --- | --- |
| `config.environment` | `SipEnvironment` | Yes | Use `SipEnvironment.uat` before go-live |
| `fcmToken` | `String?` | No | Pass it if your app already owns the token |
| `navigatorKey` | `GlobalKey<NavigatorState>?` | No | Recommended for built-in UI; falls back to `SipCalling.navigatorKey` |
| `uiMode` | `UiMode` | No | Defaults to `UiMode.builtIn` |
| `theme` | `SipCallingTheme?` | No | Applies to the built-in UI |

`SipCallingTheme` fields:

- `primaryColor`
- `dangerColor`
- `successColor`
- `backgroundColor`
- `textStyle`
- `borderRadius`

### Built-in UI example

```dart
await SipCalling.initialize(
  const SipCallingConfig(environment: SipEnvironment.uat),
  navigatorKey: appNavigatorKey,
);
```

### Headless example

```dart
await SipCalling.initialize(
  const SipCallingConfig(environment: SipEnvironment.uat),
  uiMode: UiMode.headless,
);
```

## Activation and lifecycle

Public activation methods:

- `SipCalling.activateUser(jwt, { fcmToken })`
- `SipCalling.activateAdmin(jwt, { fcmToken })`

Public instance access:

- `SipCalling.instance`
- `SipCalling.maybeInstance`

Lifecycle states:

- `notInitialized`
- `initialized`
- `active`
- `disposed`

Integration rules:

- Call `initialize()` before any other SDK API.
- The JWT role must match the activation method you call.
- `initialize()` can restore a previously saved valid session.
- Android notification permission is re-checked from current OS state during activation and on resume before incoming-call alerting is presented.
- Call `deactivate()` on logout to stop presence and prevent automatic restore.
- Call `dispose()` when your app is removing the SDK completely.
- The SDK will not replace the active session while a call is in progress.

## Call APIs

| API | What to use it for | Notes |
| --- | --- | --- |
| `startCall({ toUserId, message })` | Start an outbound call | Current public flow is for admin sessions; `toUserId` is required |
| `acceptCall()` | Accept a ringing call | Public inbound flow is available on the Flutter side |
| `rejectCall()` | Reject a ringing call | Use when the current call is ringing |
| `endCall()` | End the current call | Requires an active call |
| `toggleMute()` | Toggle microphone mute | Returns the current mute state |
| `isMuted()` | Read local mute state | Local state only |
| `toggleSpeaker(bool speakerOn)` | Switch speaker output | Flutter-specific |

## Streams

These streams are available from `SipCalling.instance`:

| Stream | Emits |
| --- | --- |
| `lifecycleState$` | SDK lifecycle changes |
| `incomingCall$` | New inbound calls |
| `activeCall$` | Active call changes, including `null` when a call ends |
| `isMuted$` | Mute state changes |
| `noAgentAvailable$` | Backend no-agent signal |
| `error$` | SDK error messages and clear events |

## Query and utility APIs

| API | Return |
| --- | --- |
| `getActiveCall()` | `CallData?` |
| `isReady()` | `bool` |
| `isActive()` | `bool` |
| `getLifecycleState()` | `SdkLifecycleState` |
| `getPresentUsers({ role })` | `Future<List<String>>` |
| `getReachableUsers({ role })` | `Future<List<String>>` |
| `getUserStatus(userId)` | `Future<UserPresenceStatus>` |
| `getCallHistory({ limit, startDate, endDate })` | `Future<List<CallHistoryItem>>` |
| `loadCallHistory({ limit, offset, startDate, endDate })` | `Future<List<CallHistoryItem>>` |
| `getCallRecording(callId)` | `Future<String?>` |
| `hasMicPermission()` | `Future<bool>` |
| `requestMicPermission()` | `Future<bool>` |

## Optional public helpers

- `SipCallingOverlay`: convenience widget that initializes the SDK and activates a user or admin from the widget tree.
- `SipRole`: enum used by `SipCallingOverlay` with `SipRole.user` and `SipRole.admin`.
- `RingtoneService`: Android-only helper for manual ringtone control when you build custom incoming-call handling.

## Useful public types

- `SipCallingConfig`
- `UiMode`
- `SipCallingTheme`
- `SdkLifecycleState`
- `CallData`
- `CallHistoryItem`
- `CallingSession`
- `UserPresenceStatus`
- `DevicePresenceInfo`
- `CallStatus`

## Public errors

The SDK exports these error classes:

- `CallingException`
- `AuthenticationException`
- `PermissionDeniedException`
- `NetworkException`

## Integration notes

- Fetch Calling JWTs through your backend, not from the app directly. See [AUTH_TOKEN.md](../AUTH_TOKEN.md).
- Keep logout wired to `deactivate()`.
- If notifications are denied or later revoked, the SDK suppresses background incoming-call ringtone/vibration instead of starting an alert with no actionable notification.
- If you build your own UI, switch to `UiMode.headless` and drive your screens from the public streams.
