![Dragon Ball World API v1](resources/logo.png?raw=true "API v1 Logo")
# API v1 Documentation (dbwots.pl)

## 1. Introduction
Built in ASP.NET Core Web API<br>
Project started **May 2023**<br>
Released to production in **June 2023**<br>
Most recent major update **Feb 2024** (*2FA TOTP support*)<br>

The API is integrated with the [DBWOTS Game](https://dbwots.pl/) database. It allows users to request various data related to the game, authenticate and manage their in-game account, fetch server details such as online count or news posts.

## 2. Core Features
### 2.1 Authentication
All the private endpoints require API user to authorize. Authorization is done by including correct access token as a header titled "Authorize".
To generate the access token, user has to authenticate first, using the **Authentication** endpoint. It returns a valid access token with a short lifespan.
When the access token expires, user has to either authenticate again, or use the **Refresh** endpoint. The endpoint does not require any account credentials. It verifies the user by a *Refresh Token* stored in cookies.
*Refresh action is only successful if stored Refresh token hasn't expired yet, the user has provided correct device identifier and client version, and the sender IP address matches.*
### 2.2 Rate Limiting
Rate limiting is used to decrease load on the database and API process in general. If the client sends too many requests in a specified amount of time, the request is cancelled and API responds with **429: Too Many Requests**. Rate limits are not shared between endpoints. The request count and timespan varies depending on endpoint importance.
### 2.3 Email features
Some endpoints are crucial for account's security and require a valid JWT token sent as an additional request parameter. These tokens are generated and sent only through SMTP email server to the real account owner. API considers emails as the top of security hierarchy, so most of security options may be modified, enabled or disabled via valid and confirmed email address connected to the account.
### 2.4 Response Caching
To avoid unneccessary load on the database, API uses in-memory cache that saves the most recent public records. If the record has not expired yet, it is sent directly as an endpoint response, and saves not only database usage, but also some computing power used to process the endpoint.
### 2.5 Response Pagination
There is some data that doesn't need to be loaded at once, and may be cut into chunks, transferred over multiple requests. Endpoints that support Pagination usually include *'index'* and *'count'* parameters to control how much data, and at what offset should be loaded. Paginated results use their own cache too, so if similar requests requested the same chunk of data, it can be reused directly from cache.
### 2.6 Two-Factor Authentication
#### This feature is new and not released to the production yet.
Two-factor authentication uses TOTP concept to link user's account with an external trusted application on their phone. Each user has their own secret key (QR Code) that is fetched by authenticator app like *Google Authenticator, Microsoft Authenticator, Authy* and is used to generate time-based codes. User has a short time window (30 seconds, but expanded by one window both ways - so 90 seconds) to enter the code during any request that requires 2FA.

## 3. API Reference
### 3.1 Creating new account
- **POST** /api/auth/account
  - Creates new user account
   - Input Form: *AccountName*, *Password*, *ConfirmPassword*, *Email*, *DisplayName*
   - Returns (200):
     ```json
     {
       "Message": "string",
       "Code": "int"
     }
     ```
### 3.2 Authentication
- **PUT** /api/auth/authenticate
  - Authenticates user and creates new session
   - Input Form: *AccountName*, *Password*, *DeviceIdentifier*, *ClientVersion*
   - Returns (200):
    ```json
    {
      "Result": "bool",
      "AccessToken": "string",
      "AccessExpiration": "date",
      "RefreshExpiration": "date",
      "DisplayName": "string",
      "Access": "int",
      "Message": "string",
      "Code": "int"
    }
    ```
- **PUT** /api/auth/refresh
  - Requires valid refresh token stored in the browser as http-only cookie
   - Input Form: *DeviceIdentifier*, *ClientVersion*
   - Returns (200):
    ```json
    {
      "Result": "bool",
      "AccessToken": "string",
      "AccessExpiration": "date",
      "DisplayName": "string",
      "Access": "int",
      "Message": "string",
      "Code": "int"
    }
    ```
### 3.3 Private Endpoints
#### Private endpoints require 'Authorize' header with correct authorization token. Otherwise it returns status code 401 Unauthorized with error code 0.
- **GET** /api/auth/account
   - Returns (200):
     ```json
     {
       "Id": "int",
       "Name": "string",
       "Email": "string",
       "Confirmed": "bool",
       "TwoFactorEnabled": "bool",
       "LastChange": "date",
       "DisplayName": "string",
       "Created": "date",
       "Access": "int",
       "Shards": "int",
       "PremiumDays": "int",
       "IsPremium": "bool"
     }
     ```
- **PATCH** /api/auth/account/displayname
   - Input Form: *DisplayName*
   - Returns (200):
    ```json
    {
      "Result": 0,
      "Message": "string"
    }
    ```
- **PUT** /api/auth/account/email/verify
  - Requests Email verification
   - Input Form: *RedirectUrl*
   - Returns (200):
    ```json
    {
      "Result": 0,
      "Message": "string"
    }
    ```
- **GET** /api/auth/account/email
  - Verifies email with token
   - Input Form: *Verify (from email)*
   - Returns (200):
    ```json
    {
      "Result": 0,
      "Message": "string"
    }
    ```
- **PUT** /api/auth/account/password/request
    - Requests Password Change
   - Input Form: *RedirectUrl*
   - Returns (200):
    ```json
    {
      "Result": 0,
      "Message": "string"
    }
    ```
- **PATCH** /api/auth/account/password
    - Changes password
   - Input Form: *AccountName*, *Password*, *ClientVersion*, *NewPassword*, *Verify (from email)*
   - Returns (200):
    ```json
    {
      "Result": 0,
      "Message": "string"
    }
    ```
- **PUT** /api/auth/account/email/request
    - Requests Email Change
   - Input Form: *RedirectUrl*
   - Returns (200):
    ```json
    {
      "Result": 0,
      "Message": "string"
    }
    ```
- **PATCH** /api/auth/account/email
    - Changes email connected to the account (requires old email)
   - Input Form: *NewEmail*, *Verify (from old email)*
   - Returns (200):
    ```json
    {
      "Result": 0,
      "Message": "string"
    }
    ```
- **PUT** /api/auth/account/recovery/request
    - Request account recovery
    - Input Query: *email*, *name*
   - Input Form: *RedirectUrl*
   - Returns (200):
    ```json
    {
      "Result": 0,
      "Message": "string"
    }
    ```
- **PATCH** /api/auth/account/recovery
    - Executes account recovery
   - Input Form: *AccountName*, *NewPassword*, *Verify (from old email)*
   - Returns (200):
    ```json
    {
      "Result": 0,
      "Message": "string"
    }
    ```
- **GET** /api/auth/account/twofactor
    - Requests two factor authentication (generates key for qr)
   - Returns (200):
    ```json
    {
      "Result": 0,
      "Message": "string",
      "TwoFactorSecret": "string",
      "Issuer": "string",
      "DisplayName": "string"
    }
    ```
- **PUT** /api/auth/account/twofactor
    - Enables two factor authentication
    - Input Form: *Code*
   - Returns (200):
    ```json
    {
      "Result": 0,
      "Message": "string",
    }
    ```
- **GET** /api/game/account/characters
  - Fetches all game characters from currently authorized account
   - Returns (200):
     ```json
     [
      {
        "Id": "int",
        "Name": "string",
        "Level": "int",
        "ImageSource": "string",
        "Title": "string",
        "Balance": "int",
        "Premium": "int"
      },
      ...
     ]
     ```
- **POST** /api/game/account/characters
  - Creates new game character
   - Input Form: *Name*, *Vocation*, *Sex*
   - Returns(200):
     ```json
     {
       "Message": "string",
       "Code": "int"
     }
     ```
- **DELETE** /api/game/account/characters
  - Deletes game character
   - Input Form: *CharacterId*, *CharacterName*, *Password*
   - Returns(200):
     ```json
     {
       "Message": "string",
       "Code": "int"
     }  
     ```
- **GET** /api/game/account/friends
   - Returns(200):
     ```json
     [
      {
        "AccountId": "int",
        "DisplayName": "string",
        "CharacterName": "string"
      },
      '''
     ]
     ```
- **POST** /api/home/news
    - Input Form: *Title*, *Content*, *Tags*
    - Returns(200):
    ```json
    {
      "Code": -1,
      "Message": "string"
    }
    ```
- **PATCH** /api/home/news
    - Query: *id*
    - Input Form: *Title*, *Content*, *Tags*
    - Returns(200):
    ```json
    {
      "Code": -1,
      "Message": "string"
    }
    ```
- **DELETE** /api/home/news
    - Query: *id*
    - Returns(200):
    ```json
    {
      "Code": -1,
      "Message": "string"
    }
    ```
### 4.4 Public Endpoints
- **GET** /api/home/updater
   - Returns(200):
    ```json
   {
     "LocalTime": "datetime",
     "HomeUrl": "string",
     "FetchUrl": "string"
   }
    ```
- **GET** /api/home/hello
   - Returns(200):
    ```json
   {
     "LocalTime": "datetime",
     "Uptime": "timespan",
     "Name": "string",
     "NameShort": "string",
     "Version": "string",
     "VersionDate": "string",
     "AuthenticationMethod": "string"
   }
    ```
- **GET** /api/game/highscores
   - Query: *type*
   - Returns(200):
    ```json
    [
      {
        "Id": "int",
        "Name": "string",
        "Level": "int",
        "ImageSource": "string",
        "Title": "string"
      },
      '''
    ]
    ```
- **GET** /api/game/characters
    - Query: *id*
    - Returns(200):
     ```json
     {
       "Id": "int",
       "Name": "string",
       "Level": "int",
       "Sex": "int",
       "ImageSource": "string",
       "Title": "string",
       "Vocation": "string",
       "LastLogin": "datetime",
       "Created": "datetime",
       "IsOnline": "bool",
       "LevelExperience": "long",
       "Experience": "long",
       "NeedExperience": "long",
       "Health": "int",
       "MaxHealth": "int",
       "Ki": "int",
       "MaxKi": "int",
       "KiLevel": "int",
       "TrainPoints": "int",
       "AttackSpeed": "int",
       "Strength": "int",
       "SwordFighting": "int",
       "KiBlasting": "int",
       "Defense": "int",
       "Energy": "int",
       "Deaths": [
          {
            "Date": "datetime",
            "KillerId": "int",
            "KillerName": "string",
            "KillerImage": "string",
            "Level": "int"
          }
       ],
       "Account": [
          {
            "Id": "int",
            "Name": "string",
            "Level": "int",
            "ImageSource": "string",
            "Title": "string"
          }
       ]
     }
     ```
- **GET** /api/game/characters/online
    - Returns(200):
    ```json
    [
      {
        "Id": "int",
        "Name": "string",
        "Level": "int",
        "ImageSource": "string",
        "Title": "string"
      },
      '''
    ]
    ```
- **GET** /api/game/killers
    - Returns(200):
    ```json
    [
      {
        "Date": "datetime",
        "Victim": {
          "Id": "int",
          "Name": "string",
          "Level": "int",
          "ImageSource": "string",
          "Title": "string"
        },
        "Killer": {
          "Id": "int",    
          "Name": "string",
          "Level": "int",
          "ImageSource": "string",
          "Title": "string"
        },
        "Level": "int",
        "IsUnjustified": "bool",
        "IsInWar": "bool",
        "Assists": [
          {
            "Id": "int",
            "Name": "string",
            "Level": "int",
            "ImageSource": "string",
            "Title": "string"
          },
          '''
        ]
      },
      '''
    ]
    ```
- **GET** /api/home/news
    - Query: *id*
    - Returns(200):
    ```json
    {
      "Id": "int",
      "Title": "string",
      "Content": "string",
      "AuthorName": "string",
      "EditorName": "string",
      "Created": "datetime",
      "Edited": "datetime",
      "Tags": "string"
    }
    ```
- **GET** /api/home/news/multi
    - Query: *index*, *count*
    - Returns(200):
    ```json
    [
      {
        "Id": "int",
        "Title": "string",
        "Content": "string",
        "AuthorName": "string",
        "EditorName": "string",
        "Created": "datetime",
        "Edited": "datetime",
        "Tags": "string"
      },
      '''
    ]
    ```
    
