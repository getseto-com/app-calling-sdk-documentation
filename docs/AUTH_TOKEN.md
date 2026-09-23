# Obtaining a Calling JWT From Your Backend

Use this guide when you need a Calling JWT for `activateUser()` or `activateAdmin()`.

This flow is shared by both the Flutter SDK and the Web SDK.

## Why this is a backend flow

The Calling JWT is issued by the App Calling API `POST /auth/token` endpoint.
That endpoint requires your tenant `ApiKey` and `ApiSecret`.

Your backend must call this endpoint on behalf of your app because `ApiSecret` is sensitive.

Never expose `ApiSecret` in a frontend application, browser bundle, mobile app, desktop app, or any other client-side code.

## Required flow

1. Your frontend authenticates with your own backend.
2. Your backend decides which App Calling role the current user should use: `user` or `admin`.
3. Your backend calls the App Calling API `POST /auth/token` endpoint with `ApiKey`, `ApiSecret`, `UserId`, and `Role`.
4. The App Calling API returns a Calling JWT.
5. Your backend returns only the Calling JWT, and optionally its expiry, to the frontend.
6. Your frontend passes that JWT to the SDK using `activateUser(jwt)` or `activateAdmin(jwt)`.

## Endpoint

- Method: `POST`
- Path: `/auth/token`
- Caller: your backend only
- Content-Type: `application/json`

Use the App Calling API base URL for the same environment that your SDK is using.

## Request body

The auth endpoint expects this JSON shape:

```json
{
  "ApiKey": "your-tenant-api-key",
  "ApiSecret": "your-tenant-api-secret",
  "UserId": "user-123",
  "Role": "user"
}
```

### Fields

- `ApiKey`: Your tenant API key.
- `ApiSecret`: Your tenant API secret. Store this only on your backend.
- `UserId`: The unique user identity that should be embedded into the JWT.
- `Role`: Must be `user` or `admin`.

The token role must match the SDK activation method:

- Use a `user` token with `activateUser(jwt)`.
- Use an `admin` token with `activateAdmin(jwt)`.

## cURL example

This example is intentionally language-independent so you can implement the same backend call in any stack.

```bash
curl --request POST "${APP_CALLING_API_BASE_URL}/auth/token" \
  --header "Content-Type: application/json" \
  --data "{\"ApiKey\":\"your-tenant-api-key\",\"ApiSecret\":\"your-tenant-api-secret\",\"UserId\":\"user-123\",\"Role\":\"user\"}"
```

Example admin token request:

```bash
curl --request POST "${APP_CALLING_API_BASE_URL}/auth/token" \
  --header "Content-Type: application/json" \
  --data "{\"ApiKey\":\"your-tenant-api-key\",\"ApiSecret\":\"your-tenant-api-secret\",\"UserId\":\"admin-1\",\"Role\":\"admin\"}"
```

## Response

Successful responses return JSON like this:

```json
{
  "Token": "eyJhbGciOi...",
  "ExpiresIn": 3600
}
```

- `Token`: Pass this exact value to the SDK as `jwt`.
- `ExpiresIn`: Token lifetime in seconds.

Do not send the `ApiSecret` to the frontend. Only return the issued JWT and any metadata you want the client to know, such as the expiry time.

## Frontend usage

After your backend returns the token, use it in the SDK:

```ts
await AppCalling.activateUser(jwt);
```

```ts
await AppCalling.activateAdmin(jwt);
```

```dart
await AppCalling.activateUser(jwt);
```

```dart
await AppCalling.activateAdmin(jwt);
```

## Security notes

- Keep `ApiKey` and `ApiSecret` in secure server-side configuration.
- Do not generate Calling JWTs in browser or mobile code.
- Do not hardcode `ApiSecret` into app binaries, JavaScript bundles, or source code shipped to users.
- Your backend should validate the signed-in user before requesting a Calling JWT for that identity.
