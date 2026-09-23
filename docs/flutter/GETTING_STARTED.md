# Flutter Calling SDK Getting Started

Use this guide to add the Flutter SDK to a Flutter application. The documented integration path in this repository is Android.

## Before you start

- Use Flutter 3.x and Dart 3.x.
- Add Firebase to the Android app and place `google-services.json` at `android/app/google-services.json`.
- Enable the Google Services plugin in the Android build.
- Set Android `minSdk` to 24 or higher.
- Make sure the manifest includes `INTERNET`, `RECORD_AUDIO`, `VIBRATE`, `BLUETOOTH`, and `BLUETOOTH_CONNECT`.
- The SDK manifest declares `POST_NOTIFICATIONS`, `USE_FULL_SCREEN_INTENT`, and `WAKE_LOCK` for incoming-call presentation on Android.
- For reliable lock-screen wake with the built-in incoming-call UI, add `android:showWhenLocked="true"` and `android:turnScreenOn="true"` to the app activity that hosts Flutter, typically `MainActivity`.
- Make sure your backend can return a Calling JWT for the current user. See [AUTH_TOKEN.md](../AUTH_TOKEN.md) for how your backend should obtain it.

## 1. Add dependencies

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_calling_sdk:
  git:
    url: https://github.com/getseto-com/app-calling-sdk.git
    path: flutter
    ref: v1.0.0
  firebase_core: ^3.0.0
```

## 2. Register and initialize the SDK

If you use the built-in UI, connect a navigator key to your `MaterialApp`. `environment` is the only required config field.

If you want the built-in call dialog to show richer identity details, you can also pass these optional fields in `AppCallingConfig`:

In the SDK API, pass them as `userName`, `designation`, `photoUrl`, and `companyName`. These map to the display fields `UserName`, `Designation`, `PhotoUrl`, and `CompanyName` used by the built-in call dialog.

- `userName`: display name associated with the current SDK session.
- `designation`: subtitle/role text shown in the call dialog.
- `photoUrl`: avatar image URL shown in the call dialog.
- `companyName`: company label used by the built-in UI.

Pass only the fields you have. Omit any optional field that you do not want to provide.

```dart
import 'package:firebase_core/firebase_core.dart';
import 'package:flutter/material.dart';
import 'package:flutter_calling_sdk/flutter_calling_sdk.dart';

final appNavigatorKey = GlobalKey<NavigatorState>();

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  AppCalling.registerBackgroundHandlers();
  await Firebase.initializeApp();
  await AppCalling.initialize(
    const AppCallingConfig(
      environment: AppEnvironment.uat,
      userName: 'Aarav Mehta',
      designation: 'Relationship Manager',
      photoUrl: 'https://example.com/users/aarav-mehta.jpg',
      companyName: 'Ainxt Technovation Pvt. Ltd.',
    ),
    navigatorKey: appNavigatorKey,
  );
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      navigatorKey: appNavigatorKey,
      home: const Placeholder(),
    );
  }
}
```

If you do not already own a navigator key, you can use `AppCalling.navigatorKey`.

These fields are optional. This is also valid when you only want to pass one or two values:

```dart
await AppCalling.initialize(
  const AppCallingConfig(
    environment: AppEnvironment.uat,
    userName: 'Aarav Mehta',
    photoUrl: 'https://example.com/users/aarav-mehta.jpg',
  ),
  navigatorKey: appNavigatorKey,
);
```

Do not pass these values to `activateUser()`, `activateAdmin()`, or `startCall()`. Provide them in `AppCallingConfig` during `initialize()` so the SDK can use them for call-dialog display.

## 3. Activate the current user

Pass the Calling JWT returned by your backend. See [AUTH_TOKEN.md](../AUTH_TOKEN.md) for the backend token flow.

```dart
await AppCalling.activateUser(jwt);
// or
await AppCalling.activateAdmin(jwt);
```

Use `activateUser()` only with user tokens and `activateAdmin()` only with admin tokens.

If Android notification permission has not already been granted, the SDK requests it immediately during activation. If the user later disables notifications in system settings, the SDK re-checks that state on resume and before presenting incoming-call alerting.

## 4. Start a call

The current public outbound flow is for admin sessions. `toUserId` is required.

```dart
await AppCalling.instance.startCall(toUserId: 'user-1');
```

You can also pass an optional `message`.

## 5. Go offline on logout

```dart
await AppCalling.instance.deactivate();
```

Call `dispose()` only when you want to tear down the SDK instance completely.

## Notes

- `initialize()` can restore the last valid session if you did not call `deactivate()` before.
- Built-in UI is the default. Use `UiMode.headless` only when you plan to render your own UI.
- The current repository documents Android integration. iOS setup is not documented here.
- Do not add a separate Android notification-permission workflow for the calling SDK. The SDK owns that permission request and suppresses background incoming-call alerting if notifications are unavailable.

For headless mode, streams, theming, and the full public API surface, see [ADVANCED_INTEGRATION.md](./ADVANCED_INTEGRATION.md).
