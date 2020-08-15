<p align="center">
  <img src="bird.png">  
</p>
<p align="center">
&nbsp;
        <img src="https://img.shields.io/packagist/v/robsontenorio/laravel-keycloak-guard.svg" />
        <img src="https://img.shields.io/packagist/dt/robsontenorio/laravel-keycloak-guard.svg" />

</p>

# Simple Keycloak Guard for Laravel / Lumen

This package helps you authenticate users on a Laravel API based on JWT tokens generated from  **Keycloak Server**.


# Requirements

✔️ I`m building an API with Laravel.

✔️ I will not use Laravel Passport for authentication, because Keycloak Server will do the job.

✔️ The frontend is a separated project.

✔️ The frontend users authenticate **directly on Keycloak Server** to obtain a JWT token. This process have nothing to do with the Laravel API.

✔️ The frontend keep the JWT token from Keycloak Server.

✔️ The frontend make requests to the Laravel API, with that token.


💔 If your app does not match requirements, probably you are looking for https://github.com/SocialiteProviders

# The flow

<p align="center">
  <img src="flow.png">  
</p>


1. The frontend user authenticates on Keycloak Server

1. The frontend user obtains a JWT token.

1. In another moment, the frontend user makes a request to some protected endpoint on a Laravel API, with that token.

1. The Laravel API (through `Keycloak Guard`) handle it.
   - Verify token signature.
   - Verify token structure.
   - Verify token expiration time.
   - Verify if my API allows `resource access` from token.

1. If everything is ok, find the user on database and authenticate it on my API.

1. Return response

# Install

Require the package

```
composer require saldanhaascs/laravel-keycloak-guard
```

Publish the config file

```
php artisan vendor:publish  --provider="KeycloakGuard\KeycloakGuardServiceProvider"

```

### Lumen

Register the provider in your boostrap app file ```boostrap/app.php```

Add the following line in the "Register Service Providers"  section at the bottom of the file. 

```php
$app->register(\KeycloakGuard\KeycloakGuardServiceProvider::class);
```
For facades, uncomment ```$app->withFacades();``` in your boostrap app file ```boostrap/app.php```

# Configuration

## Keycloak Guard

The Keycloak Guard configuration can be handled from Laravel `.env` file. ⚠️ Be sure all strings **are trimmed.**

```php
<?php

return [
  'realm_public_key' => env('KEYCLOAK_REALM_PUBLIC_KEY', null),

  'load_user_from_database' => env('KEYCLOAK_LOAD_USER_FROM_DATABASE', true),

  'user_provider_credential' => env('KEYCLOAK_USER_PROVIDER_CREDENTIAL', 'email'),

  'token_principal_attribute' => env('KEYCLOAK_TOKEN_PRINCIPAL_ATTRIBUTE', 'email'),

  'append_decoded_token' => env('KEYCLOAK_APPEND_DECODED_TOKEN', true),

  'allowed_resources' => env('KEYCLOAK_ALLOWED_RESOURCES', account)
];

```

✔️  **realm_public_key**

*Required.*

The Keycloak Server realm public key (string). Click on your REALM-> TAB KEYS-> Public Key.

✔️ **load_user_from_database**

*Required. Default is `true`.*

If you do not have an `users` table you must disable this.

It fetchs user from database and fill values into authenticated user object. If enabled, it will work together with `user_provider_credential` and `token_principal_attribute`.

✔️ **user_provider_credential**

*Required. Default is `email`.*


The field from "users" table that contains the user unique identifier (eg.  username, email, nickname). This will be confronted against  `token_principal_attribute` attribute, while authenticating.

✔️ **token_principal_attribute**

*Required. Default is `email`.*

The property from JWT token that contains the user identifier.
This will be confronted against  `user_provider_credential` attribute, while authenticating.

✔️ **append_decoded_token**

*Default is `false`.*

Appends to the authenticated user the full decoded JWT token. Useful if you need to know roles, groups and another user info holded by JWT token. Even choosing `false`, you can also get it using `Auth::token()`, see API section.

✔️ **allowed_resources**

*Required*

Usually you API should handle one *resource_access*. But, if you handle multiples, just use a comma separated list of allowed resources accepted by API. This attribute will be confronted against `resource_access` attribute from JWT token, while authenticating. Example: account,client_id_keycloak_server. To know your resource_access that comes in the token use https://jwt.io/#debugger-io.

*Default is `account`.*

The Default is `account` so that users who do not have roles associated with their client_id can be registered in the Laravel database.

## Laravel Auth

Changes on `config/auth.php`
```php
'defaults' => [
        'guard' => 'api', # <-- For sure, i`m building an API
        'passwords' => 'users',
    ],

    'guards' => [
        'api' => [
            'driver' => 'keycloak', # <-- Set the API guard driver to "keycloak"
            'provider' => 'users',
        ],
    ],
```

## Laravel Routes
Just protect some endpoints on `routes/api.php` and you are done!

```php
// public endpoints
Route::get('/hello', function () {
    return ':)';
});

// protected endpoints
Route::group(['middleware' => 'auth:api'], function () {
    Route::get('/protected-endpoint', 'SecretController@index');
    // more endpoints ...
});
```


## Lumen Routes
Just protect some endpoints on `routes/web.php` and you are done!

```php
// public endpoints
$router->get('/hello', function () {
    return ':)';
});

// protected endpoints
$router->group(['middleware' => 'auth'], function () {
    $router->get('/protected-endpoint', 'SecretController@index');
    // more endpoints ...
});
```

# API

Simple Keycloak Guard implements `Illuminate\Contracts\Auth\Guard`. So, all Laravel default methods will be available. Ex: `Auth::user()` returns the authenticated user.

### Default methods:

- check()
- guest()
- user()
- id()
- validate()
- setUser()


### Keycloak Guard methods:

- token()

Ex: `Auth::token()` returns full decoded JWT token from authenticated user

- hasRole('some-resource', 'some-role'):  Check if the authenticated user has especific role into a resource.

Ex:
Whit this payload:

```
'resource_access' => [
  'myapp-backend' => [
      'roles' => [
        'myapp-backend-role1',
        'myapp-backend-role2'
      ]
  ],
  'myapp-frontend' => [
    'roles' => [
      'myapp-frontend-role1',
      'myapp-frontend-role2'
    ]
  ]
]
```
```
Auth::hasRole('myapp-backend', 'myapp-backend-role1') => true
Auth::hasRole('myapp-frontend', 'myapp-frontend-role1') => true
Auth::hasRole('myapp-backend', 'myapp-frontend-role1') => false
```

# Contact

Twitter [@SaBaLAkU](https://twitter.com/sabalaku)
