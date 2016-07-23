# Tada Services API

Version 1

The Tada Servies REST API is the main management and use interface for the Tada platform. All Tada client and server libraries and SDKs, as well as Tada web dashboards, are simple wrappers of this HTTP API.

## API Components

* [App Data](#app-data): `https://<appId>.tadadb.com`
* [Analytics](#analytics): `https://analytics.gettada.com/v1/<appId>`
* [Auth](#auth): `https://auth.gettada.com/v1/<appId>`
* [Management](#management): `https://api.gettada.com/v1`

## App Data

The **App Data** API allows for the creation, reading, updating, and deleting of application data. Your Tada app's data lives at `https://<appId>.tadadb.com`, where `<appId>` is your Tada app's **App ID**.

## App Data Overview

Data written to or read from your Tada app is stored as a single **JSON** document. Since each key in a JSON document is unique, your Tada app's data naturally maps to a URL, allowing for flexible data manipulation.

**Example**: Your soda can collection game, *Clash of Cans*, needs to store and modify each user's can collection sizes. Since **App ID**s are unique, lowercase, and cannot contain spaces, so we'll use `clashofcans`.

Data for a single user is structured as follows:

```JSON
{
  "users": {
    "coolSpot": {
      "canCounts": {
        "coca-cola": 32,
        "a&w": 13,
        "sprite": 25
      }
    }
  }
}
```

If we want to access user `coolSpot`'s can counts, we can perform an **HTTP GET** to `https://clashofcans.tadadb.com/users/coolSpot/canCounts`, giving us:

```JSON
{
  "a&w": 13,
  "coca-cola": 32,
  "sprite": 25
}
```

Likewise, if we only want user `coolSpot`'s coca-cola can count, we can perform an **HTTP GET** to `https://clashofcans.tadadb.com/users/coolSpot/canCounts/coca-cola`, giving us a single number:

```JSON
32
```

## App Data API Documentation

Service URL: `https://<appId>.tadadb.com`, where `<appId>` is your Tada app's **App ID**.

As described in the previous section, App Data can be written to **any path**.

To see this in action, consider an **HTTP POST** of `{ "hello": "world" }` to `https://<appId>.tadadb.com/this/is/a/really/long/example/url`

An **HTTP GET** to the database root, i.e. `https://<appId>.tadadb.com/`, now returns the following:

```json
{
  "this": {
    "is": {
      "a": {
        "really": {
          "long": {
            "example": {
              "url": {
                "hello": "world"
              }
            }
          }
        }
      }
    }
  }
}
```

### Create (insert)

| Route       | Any (e.g. `/any/database/location`) |
| ----------- | --- |
| Description | Inserts data at the specified location. This insert fails if data already exists at the specificed location. |
| HTTP method | POST |

Headers:

* **Authorization** - Specified in JWT format, i.e. `Bearer <token>`
* **Content-Type** - Must be `application/json`

Response Codes:

* 200: **Success** - The data was successfully inserted
* 400: **Bad Request** - A JSON payload was not provided
* 401: **Unauthorized** - Invalid or nonexistent Authorization header
* 409: **Conflict** - Data already exists at the specified location
* 500: **Internal Server Error** - A server error occured while processing your request

### Read (get)

| Route       | Any (e.g. `/any/database/location`) |
| ----------- | --- |
| Description | Retrieves data from the specified location. |
| HTTP method | GET |

Headers:

* **Authorization** - Specified in JWT format, i.e. `Bearer <token>`
* **Accept** *(Optional)* - Defaults to `application/json`

Response Codes:

* 200: **Success** - The data was successfully read
* 401: **Unauthorized** - Invalid or nonexistent Authorization header
* 500: **Internal Server Error** - A server error occured while processing your request

### Update (upsert)

| Route       | Any (e.g. `/any/database/location`) |
| ----------- | --- |
| Description | Updates data at the specified location, inserting data if none exist. |
| HTTP method | PATCH |

Headers:

* **Authorization** - Specified in JWT format, i.e. `Bearer <token>`
* **Content-Type** - Must be `application/json`

Response Codes:

* 200: **Success** - The data was successfully updated
* 400: **Bad Request** - A JSON payload was not provided
* 401: **Unauthorized** - Invalid or nonexistent Authorization header
* 500: **Internal Server Error** - A server error occured while processing your request

### Delete (remove)

| Route       | Any (e.g. `/any/database/location`) |
| ----------- | --- |
| Description | Removes data from the specified location. |
| HTTP method | DELETE |

Headers:

* **Authorization** - Specified in JWT format, i.e. `Bearer <token>`
* **Accept** *(Optional)* - Defaults to `application/json`

Response Codes:

* 200: **Success** - The data was successfully deleted
* 401: **Unauthorized** - Invalid or nonexistent Authorization header
* 500: **Internal Server Error** - A server error occured while processing your request

## Auth

The **Auth** API allows users to log into your apps using one or more authentication providers. Users are authenticated at `https://auth.gettada.com/v1/<appId>/<provider>`, where `<appId>` is your Tada app's **App ID**, and `<provider>` is the auth provider being used.

### Auth Overview

When your users log into any of your apps in the same **App Group**, a consistent, unique, cross-app user ID called a **xuid** is returned by the **Auth** API. This allows for your apps to frictionlessly support users logging in with the same account after a sequel or follow-up product is released.

Authentication providers can be enabled, configured, and disabled using the [Management API](#management).

### Auth API Documentation

Service URL: `https://auth.gettada.com/v1/<appId>/<provider>`, where `<appId>` is your Tada app's **App ID**, and `<provider>` is the auth provider being used.

### Anonymous Login

Allows for the creation of a user account without providing any user identifiers or credentials.

#### Login

| Route       | /v1/{appId}/anonymous |
| ----------- | --- |
| Description | Creates an anonymous user |
| {appId}     | Your Tada app's **App ID** |
| HTTP method | GET |

Response Codes:

* 200: **Success** - The anonymous user account was successfully created
* 500: **Internal Server Error** - A server error occured while processing your request

### Password Login (Email & Password)

Allows users to log in using their email address and a password of their choice.

#### Register

| Route       | /v1/{appId}/password/register |
| ----------- | --- |
| Description | Registers a new password user |
| {appId}     | Your Tada app's **App ID** |
| HTTP method | POST |

Headers:

* **Content-Type**: Must be `applciation/json`

Parameters:

* **email**: The user's email address
* **password**: The user's password

Response Codes:

* 200: **Success** - The user account was successfully created
* 400: **Bad Request** - Invalid or nonexistent email address or password
* 409: **Conflict** - The user account already exists
* 500: **Internal Server Error** - A server error occured while processing your request

#### Verify email address

| Route       | /v1/{appId}/password/verify |
| ----------- | --- |
| Description | Verifies a user's email address |
| {appId}     | Your Tada app's **App ID** |
| HTTP method | POST |

Parameters:

* **key**: The verificiation key emailed to the user

Response Codes:

* 200: **Success** - The user's email address was successfully verified
* 400: **Bad Request** - Invalid or nonexistent verification key
* 500: **Internal Server Error** - A server error occured while processing your request

### Password Login

| Route       | /v1/{appId}/password/login |
| ----------- | --- |
| Description | Logs a user into your app (i.e. returns a user's profile and access token) |
| {appId}     | Your Tada app's **App ID** |
| HTTP method | POST |

Headers:

* **Content-Type**: Must be `application/json`

Parameters:

* **email**: The user's email address
* **password**: The user's password

Response Codes:

* 200: **Success** - The user was successfully logged in
* 400: **Bad Request** - Invalid or nonexistent email address or password
* 500: **Internal Server Error** - A server error occured while processing your request

#### Send a "forgot password" email

| Route       | /v1/{appId}/password/forgot |
| ----------- | --- |
| Description | Emails a user a password reset key |
| {appId}     | Your Tada app's **App ID** |
| HTTP method | POST |

Headers:

* **Content-Type**: Must be `application/json`

Parameters:

* **email**: The user's email address

Response Codes:

* 200: **Success** - A password reset key was successfully sent
* 400: **Bad Request** - Invalid or nonexistent email address
* 404: **Not Found** - The user account does not exist
* 500: **Internal Server Error** - A server error occured while processing your request

#### Reset a password

| Route       | /v1/{appId}/password/reset |
| ----------- | --- |
| Description | Resets a user's password |
| {appId}     | Your Tada app's **App ID** |
| HTTP method | POST |

Headers:

* **Content-Type**: Must be `application/json`

Parameters:

* **key**: The password reset key emailed to the user
* **newPassword**: The user's new password

Response Codes:

* 200: **Success** - The user's password was successfully reset
* 400: **Bad Request** - Invalid or nonexistent password reset key
* 500: **Internal Server Error** - A server error occurred while processing your request

#### Change a password

| Route       | /v1/{appId}/password/change |
| ----------- | --- |
| Description | Changes a user's password |
| {appId}     | Your Tada app's **App ID** |
| HTTP method | POST |

Headers:

* **Content-Type**: Must be `application/json`

Parameters:

* **email**: The user's email address
* **password**: The user's password
* **newPassword**: The user's new password

Response Codes:

* 200: **Success** - The user's password was successfully changed
* 400: **Bad Request** - Invalid or nonexistent email address, password, or new password
* 500: **Internal Server Error** - A server error occured while processing your request

### Social Login (OAuth2)

Allows users to log in using their **Facebook** or **GitHub** accounts. Uses OAuth2 redirection to redirect back to your app after login.

#### Registration and Login

| Route       | /v1/{appId}/{provider} |
| ----------- | --- |
| Description | Changes a user's password |
| {appId}     | Your Tada app's **App ID** |
| {provider}  | One of [`facebook`, `github`] |
| HTTP method | GET |

Parameters:

**redirectUri**: The URI to redirect the browser or web view after logging in
**requestId**: A 32-character alphanumeric string that will be used to retrieve your session after logging in

Returns:
On successful redirection to **redirectUri**, a query parameter **requestKey** will be available for use in retrieving the user's session

### Session Data

Sessions contain user profile and token detail that's retrieved in multi-step login processes, such as in Social Login redirect flows.

#### Get session data

| Route       | /v1/{appId}/session |
| ----------- | --- |
| Description | Retrieves a user's session (i.e. their profile) |
| {appId}     | Your Tada app's **App ID** |
| HTTP method | GET |

Parameters:

**requestId**: The 32-character alphanumeric string that was used to log into the app
**requestKey**: The 32-character alphanumeric string that was provided in the redirect URI

Response Codes:

* 200: **Success** - The user's session was successfully retrieved
* 400: **Bad Request** - Invalid or nonexistent requestId or requestKey
* 500: **Internal Server Error** - A server error occured while processing your request

## Management

The **Management** API is the central location for management of all aspects of your Tada apps. Management actions are performed at `https://api.gettada.com`.

## Overview

All aspects of your Tada apps are managed through the Management API, including **App** and **App Group** creation, **Organization** and **Collaborator** settings, **Auth Provider** settings, and **User Profile** querying.

## API Documentation

Service URL: `https://api.gettada.com`

In order to access this API's routes you must first log into your Tada account at `https://auth.gettada.com/v1/tada/<provider>`, where `<provider>` is the auth provider you used when you registered for Tada.

### App creation

Create and modify your Tada apps.

#### Create an app

| Route       | /v1/apps |
| ----------- | --- |
| Description | Creates a new app |
| HTTP method | POST |

Headers:

 **Authorization** - Specified in JWT format, i.e. `Bearer <token>`

Parameters:

* **accountId**: The owning account ID. This can be your account ID or an organization ID.
* **appGroupId**: The containing app group ID
* **appId**: The app ID
* **createNewAppGroup**: `true` if the provided appGroupId is intended to be a new app group, `false` otherwise

Response Codes:

* 200: **Success** - The app was successfully created
* 400: **Bad Request** - Invalid or nonexistent accountId, appGroupId, or appId
* 401: **Unauthorized** - Invalid or nonexistent Authorization header
* 500: **Internal Server Error** - A server error occured while processing your request

#### Retrieve detail for all accessible apps

| Route       | /v1/apps |
| ----------- | --- |
| Description | Gets detail for all apps accessible by the logged in user |
| HTTP method | GET |

Headers:

* **Authorization** - Specified in JWT format, i.e. `Bearer <token>`
* **Accept** *(Optional)* - Defaults to `application/json`

Response Codes:

* 200: **Success** - Accessible apps were successfully retrieved
* 401: **Unauthorized** - Invalid or nonexistent Authorization header
* 500: **Internal Server Error** - A server error occured while processing your request

#### Retrieve app detail

| Route       | /v1/apps/{appId} |
| ----------- | --- |
| Description | Gets detail for a single app |
| {appId}     | Your Tada app's **App ID** |
| HTTP method | GET |

Headers:

* **Authorization** - Specified in JWT format, i.e. `Bearer <token>`

Response Codes:

* 200: **Success** - The specified app was successfully retrieved
* 401: **Unauthorized** - Invalid or nonexistent Authorization header
* 404: **Not Found** - The specified app was not found
* 500: **Internal Server Error** - A server error occured while processing your request

#### Update app detail

| Route       | /v1/apps/{appId}|
| ----------- | --- |
| Description | Updates the specified app's name and/or description |
| {appId}     | Your Tada app's **App ID** |
| HTTP method | POST |

Headers:

* **Authorization** - Specified in JWT format, i.e. `Bearer <token>`

Parameters:

* **name** *(Optional)* - An app name
* **description** *(Optional)* - An app description

Response Codes:

* 200: **Success** - The app was successfully updated
* 401: **Unauthorized** - Invalid or nonexistent Authorization header
* 404: **Not Found** - The specified app was not found
* 500: **Internal Server Error** - A server error occured while processing your request

### Auth Management

Manage how users log into your apps.

Since Apps in the same App Group share the same users, auth settings are managed at the app group level. App Groups are unique to a single account.

#### Get all auth provider settings

| Route       | /v1/auth |
| ----------- | --- |
| Description | Gets settings for all auth providers |
| HTTP method | GET |

Headers:

* **Authorization** - Specified in JWT format, i.e. `Bearer <token>`
* **Accept** *(Optional)* - Defaults to `application/json`

Parameters:

* **accountId**: The account that owns the app group
* **appGroupId**: The app group

Returns:

* A JSON array of auth provider settings.

Response Codes:

* 200: **Success** - The auth provider settings were successfully retrieved
* 401: **Unauthorized** - Invalid or nonexistent Authorization header
* 500: **Internal Server Error** - A server error occured while processing your request

#### Get auth provider settings for a specific provider

| Route       | /v1/auth/{provider}|
| ----------- | --- |
| Description | Gets settings for the specified auth provider |
| {provider}  | One of [`anonymous`, `facebook`, `github`, `password`] |
| HTTP method | GET |

Headers:

* **Authorization** - Specified in JWT format, i.e. `Bearer <token>`
* **Accept** *(Optional)* - Defaults to `application/json`

Parameters:

* **accountId**: The account that owns the app group
* **appGroupId**: The app group

Response Codes:

* 200: **Success** - The app was successfully updated
* 401: **Unauthorized** - Invalid or nonexistent Authorization header
* 404: **Not Found** - The specified app was not found
* 500: **Internal Server Error** - A server error occured while processing your request

#### Change settings for the *anonymous* auth provider

| Route       | /v1/auth/anonymous |
| ----------- | --- |
| Description | Updates settings for the *anonymous* auth provider |
| HTTP method | POST |

Headers:

* **Authorization** - Specified in JWT formaat, i.e. `Bearer <token>`
* **Content-Type** - Must be `application/json`

Parameters:

* **accountId**: The account that owns the app group
* **appGroupId**: The app group
* **enabled** *(Optional)*: Specifies whether this auth provider is available for use
* **canAccessAppData** *(Optional)*: Specifies whether *anonymous* users can access app data

Response Codes:

* 200: **Success** - The auth provider settings were successfully updated
* 400: **Bad Request** - Invalid or nonexistent parameter
* 401: **Unauthorized** - Invalid or nonexistent Authorization header
* 404: **Not Found** - The specified auth provider was not found
* 500: **Internal Server Error** - A server error occured while processing your request

#### Change settings for the *facebook* auth provider

| Route       | /v1/auth/facebook |
| ----------- | --- |
| Description | Updates settings for the *facebook* auth provider |
| HTTP method | POST |

Headers:

* **Authorization** - Specified in JWT formaat, i.e. `Bearer <token>`
* **Content-Type** - Must be `application/json`

Parameters:

* **accountId**: The account that owns the app group
* **appGroupId**: The app group
* **enabled** *(Optional)*: Specifies whether this auth provider is available for use
* **appId** *(Optional)*: Your Facebook AppID
* **appSecret** *(Optional)*: Your Facebook AppSecret

Response Codes:

* 200: **Success** - The auth provider settings were successfully updated
* 400: **Bad Request** - Invalid or nonexistent required parameter
* 401: **Unauthorized** - Invalid or nonexistent Authorization header
* 404: **Not Found** - The specified auth provider was not found
* 500: **Internal Server Error** - A server error occured while processing your request

#### Change settings for the *github* auth provider

| Route       | /v1/auth/github |
| ----------- | --- |
| Description | Updates settings for the *github* auth provider |
| HTTP method | POST |

Headers:

* **Authorization** - Specified in JWT formaat, i.e. `Bearer <token>`
* **Content-Type** - Must be `application/json`

Parameters:

* **accountId**: The account that owns the app group
* **appGroupId**: The app group
* **enabled** *(Optional)*: Specifies whether this auth provider is available for use
* **appId** *(Optional)*: Your GitHub AppID
* **appSecret** *(Optional)*: Your GitHub AppSecret

Response Codes:

* 200: **Success** - The auth provider settings were successfully updated
* 400: **Bad Request** - Invalid or nonexistent required parameter
* 401: **Unauthorized** - Invalid or nonexistent Authorization header
* 404: **Not Found** - The specified auth provider was not found
* 500: **Internal Server Error** - A server error occured while processing your request

#### Change settings for the *password* auth provider

| Route       | /v1/auth/password |
| ----------- | --- |
| Description | Updates settings for the *password* auth provider |
| HTTP method | POST |

Headers:

* **Authorization** - Specified in JWT formaat, i.e. `Bearer <token>`
* **Content-Type** - Must be `application/json`

Parameters:

* **accountId**: The account that owns the app group
* **appGroupId**: The app group
* **enabled** *(Optional)*: Specifies whether this auth provider is available for use
* **minPasswordLength** *(Optional)*: The minimum length of user passwords (8 or greater recommended)

Response Codes:

* 200: **Success** - The auth provider settings were successfully updated
* 400: **Bad Request** - Invalid or nonexistent required parameter
* 401: **Unauthorized** - Invalid or nonexistent Authorization header
* 404: **Not Found** - The specified auth provider was not found
* 500: **Internal Server Error** - A server error occured while processing your request