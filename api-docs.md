![Dragon Ball World API v1](resources/logo.png?raw=true "API v1 Logo")
# API v1 Documentation (dbwots.pl)

## 1. Introduction
The API is integrated with the DBWOTS game database. It allows you to request various data related to the game, authenticate and manage your in-game account, fetch server details such as online count or news posts.

## 2. Prerequisites
API requires dotnet 6.0 runtime (also sdk for compiling).
### 2.1 Packages
All the neccessary packages can be installed via:
``` dotnet restore "./dbwots-api.csproj" ```
### 2.2 Publishing
Compiling and publishing app (for linux):
``` dotnet publish "./dbwots-api.csproj" -c release -r linux-x64```

## 3. Fundamentals
### 3.1 Authentication
All the private endpoints require API user to authorize. Authorization is done by including correct access token header named "Authorize".
To generate the access token, user has to authenticate first, using the **Authentication** endpoint. It returns a valid access token with a short lifespan.
When the access token expires, user has to either authenticate again, or use the **Refresh** endpoint. The endpoint does not require any account credentials. It verifies the user by a *Refresh Token* stored in cookies.
*Refresh action is only successful if stored Refresh token hasn't expired yet, the user has provided correct device identifier and client version, and the sender IP address matches.*
### 3.2 Rate Limiting
Rate limiting is used to decrease load on the database and API process in general. If the client sends too many requests in a specified amount of time, the request is cancelled and API responds with **429: Too Many Requests**. Rate limits are not shared between endpoints. The request count and timespan varies depending on endpoint importance.

## 4. API Reference
### 4.1 Creating new account
- **POST** /api/server/account
   - Input Form: *AccountName*, *Password*, *ConfirmPassword*, *Email*, *DisplayName*
   - Returns (200):
     ```json
     {
       "Message": "string",
       "Code": "int"
     }
     ```
### 4.2 Authentication
- **PUT** /api/server/authenticate
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
- **PUT** /api/server/refresh
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
### 4.3 Private Endpoints
#### Private endpoints require 'Authorize' header with correct authorization token. Otherwise it returns status code 401 Unauthorized with error code 0.
- **GET** /api/server/account
   - Returns (200):
     ```json
     {
       "AccountId": "int"
     }
     ```
- **GET** /api/server/account/characters
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
- **POST** /api/server/account/characters
   - Input Form: *Name*, *Vocation*, *Sex*
   - Returns(200):
     ```json
     {
       "Message": "string",
       "Code": "int"
     }
     ```
- **DELETE** /api/server/account/characters
   - Input Form: *CharacterId*, *CharacterName*, *Password*
   - Returns(200):
     ```json
     {
       "Message": "string",
       "Code": "int"
     }  
     ```
- **GET** /api/server/account/friends
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
- **GET** /api/server/highscores
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
- **GET** /api/server/characters
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
- **GET** /api/server/characters/online
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
- **GET** /api/server/killers
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
    
