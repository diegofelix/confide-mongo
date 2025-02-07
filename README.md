# Confide Mongo (Laravel5 Package)

![Confide Poster](https://dl.dropboxusercontent.com/u/12506137/libs_bundles/confide_mongo.png)

[![Build Status](https://api.travis-ci.org/Zizaco/confide-mongo.png)](https://travis-ci.org/Zizaco/confide-mongo)
[![ProjectStatus](http://stillmaintained.com/Zizaco/confide-mongo.png)](http://stillmaintained.com/Zizaco/confide-mongo)

Confide is a authentication solution for **Laravel5** using [MongoLid](https://github.com/leroy-merlin-br/mongolid-laravel) made to eliminate repetitive tasks involving the management of users: Account creation, login, logout, confirmation by e-mail, password reset, etc.

Confide aims to be simple to use, quick to configure and flexible.

> Note: If you are **NOT** using MongoDB check [Confide](https://github.com/Zizaco/confide).

## Features

**Current:**
- Account confirmation (through confirmation link).
- Password reset (sending email with a change password link).
- Easily render forms for login, signup and password reset.
- Generate customizable routes for login, signup, password reset, confirmation, etc.
- Generate a customizable controller that handles the basic user account actions.
- Contains a set of methods to help basic user features.
- Integrated with the Laravel Auth component/configs.
- Field/model validation.
- Login throttling.
- Redirecting to previous route after authentication.

If you are looking for user roles and permissions see [Entrust](https://github.com/Zizaco/entrust)

**Planned:**
- Captcha in user signup and password reset.
- General improvements.

## Quick start

### Required setup

**Note**: This package includes Mongolid Laravel. Use the correct tag according to your Laravel version:

- Laravel 5.1: `^0.14`
- Laravel 5.2: `^0.15`
- Laravel 5.3: `^0.15`
- Laravel 5.4: `^0.16`

Run composer require command to download the package.

```shell
composer require zizaco/confide-mongo
```

In your `config/app.php` add `'Zizaco\ConfideMongo\ConfideMongoServiceProvider'` to the end of the `$providers` array

    'providers' => array(

        'Illuminate\Foundation\Providers\ArtisanServiceProvider',
        'Illuminate\Auth\AuthServiceProvider',
        ...
        'Zizaco\ConfideMongo\ConfideMongoServiceProvider',

    ),

At the end of `config/app.php` add `'Confide'    => 'Zizaco\Confide\ConfideFacade'` to the `$aliases` array

    'aliases' => array(

        'App'        => 'Illuminate\Support\Facades\App',
        'Artisan'    => 'Illuminate\Support\Facades\Artisan',
        ...
        'Confide'    => 'Zizaco\Confide\ConfideFacade',

    ),

### Configuration

Set the user provider driver to _"mongolid"_ in `config/auth.php` as stated in [MongoLid Authentication](https://github.com/leroy-merlin-br/mongolid-laravel#authentication):

```php

    'providers' => [

        // ...

        'users' => [
            'driver' => 'mongolid',
            'model' => \App\User::class
        ],

        // ...

    ],

```

This values contained in `config/auth.php` will be used by Confide Mongo to generate the controllers and routes.

Set the `address` and `name` from the `from` array in `config/mail.php`. Those will be used to send account confirmation and password reset emails to the users.

### User model

Change your User model in `app/models/User.php` to:

    <?php

    use Zizaco\ConfideMongo\ConfideMongoUser;

    class User extends ConfideMongoUser {

    }

`ConfideMongoUser` class will take care of some behaviors of the user model.

### Dump the default acessors

Least, you can dump a default controller and the default routes for Confide.

    $ php artisan confide:controller
    $ php artisan confide:routes

Don't forget to dump composer autoload

    $ composer dump-autoload

**And you are ready to go.**
Access `http://yourapp/user/create` to create your first user. Check the `app/routes.php` to see the available routes.

## Usage in detail

**Basic setup:**

1. Mongo database connection in `config/database.php` running properly.
2. Set the auth driver to _"mongoLid"_ in `config/auth.php`.
3. Check and correct the model and collection names in `config/auth.php`. They will be used by Confide all the time.
4. `from` configuration in `config/mail.php`.

**Configuration:**

1. `ConfideMongoServiceProvider` and `ConfideFacade` entry in `config/app.php` `'providers'` and `'aliases'` respectively.
2. User model (with the same name as in `config/auth.php`) should extend `ConfideMongoUser` class. This will cause to methods like `resetPassword()`, `confirm()` and a overloaded `save()` to be available.

**Optional steps:**

1. Use `Confide` facade to dump login and signup forms easly with `makeLoginForm()` and `makeSignupForm()`. You can render the forms within your views by doing `{{ Confide::makeLoginForm()->render() }}`.
2. Generate a controller with the template contained in Confide throught the artisan command `$ php artisan confide:controller`. If a controller with the same name exists it will **NOT** be overwritten.
3. Generate routes matching the controller template throught the artisan command `$ php artisan confide:routes`. Your `routes.php` will **NOT** be overwritten.

### Advanced

#### Using custom collection / model name

You can change the model name that will be authenticated in the `config/auth.php` file.
Confide uses the values present in that configuration file.

To change the controller name when dumping the default controller template you can use the --name option.

    $ php artisan confide:controller --name Employee

Will result in `EmployeeController`

Then, when dumping the routes, you should use the --controller option to match the existing controller.

    $ php artisan confide:routes --controller Employee

#### Using custom form or emails

First, publish the config files:

    $ php artisan config:publish zizaco/confide

Then edit the view names in `app/config/packages/zizaco/confide/config.php`.

#### Update an User

To update an user already in the database you'll want to either pass in an different rule set or use the amend function.

    $user = new User;
    $user->username = 'newUserName';

    // Save
    $user->save($this->getUpdateRules());

#### Validate model fields

To change the validation rules of the User model you can take a look at [Laravel 5 Validations](http://laravel.com/docs/validation "Laravel Validation Rules"). For example:

    <?php

    use Zizaco\ConfideMongo\ConfideMongoUser;

    class User extends ConfideMongoUser {

        /**
         * Validation rules
         */
        public $rules = array(
            'email' => 'required|email',
            'password' => 'required|between:4,11|confirmed',
        );

    }

Feel free to add more fields to your collection and to the validation array. Then you should build your own sign-up form with the additional fields.

#### Passing additional information to the make methods

If you want to pass additional parameters to the forms, you can use an alternate syntax to achieve this.

Instead of using the make method:

    Confide::makeResetPasswordForm( $token ):

You would use:

    View::make(Config::get('confide::reset_password_form'))
        ->with('token', $token);

It produces the same output, but you would be able to add more inputs using 'with' just like any other view.

#### RESTful controller

If you want to generate a [RESTful controller](https://github.com/laravel/docs/blob/master/controllers.md#restful-controllers) you can use the aditional `--restful` or `-r` option.

    $ php artisan confide:controller --restful

Will result in a [RESTful controller](https://github.com/laravel/docs/blob/master/controllers.md#restful-controllers)

Then, when dumping the routes, you should use the --restful option to match the existing controller.

    $ php artisan confide:routes --restful

#### User roles and permissions

In order not to bloat Confide with not related features, the role and permission was developed as another package: [Entrust](https://github.com/Zizaco/entrust). This package couples very well with Confide.

See [Entrust](https://github.com/Zizaco/entrust)

> Note: Entrust is not yet available for MongoLid / MongoDB

#### Redirecting to previous route after login

When defining your filter you should set the `'loginRedirect'` session variable. For example:

    // filters.php

    Route::filter('auth', function()
    {
        if (Auth::guest()) // If the user is not logged in
        {
            // Set the loginRedirect session variable
            Session::put('loginRedirect', Request::url());

            // Redirect back to user login
            return Redirect::to('user/login');
        }
    });

    // Only authenticated users will be able to access routes that begins with
    // 'admin'. Ex: 'admin/posts', 'admin/categories'.
    Route::when('admin*', 'auth');

or, if you are using [Entrust](https://github.com/Zizaco/entrust) ;)

    // filters.php

    Entrust::routeNeedsRole( 'admin*', 'Admin', function(){
        Session::put( 'loginRedirect', Request::url() );
        return Redirect::to( 'user/login' );
    } );

#### Validating a route

If you want to validate whether a route exists, the `Confide::checkAction` function is what you are looking for.

Currently it is used within the views to determine Non-RESTful vs RESTful routes.

## Troubleshooting

__"'confirmation_code' required" when creating a user__

If you overwrite the `save()` method in your model, make sure to call `parent::save()`:

    public function save( $forced = false ){

        parent::save( $forced) // Don't forget this

        // Your stuff
    }

__Confirmation link is not sent when user signup__

If you overwrite the `afterSave()` method in your model, make sure to call `parent::afterSave()`

__Users are able to login without confirming account__

If you want only confirmed users to login, in your `UserController`, instead of simply calling `logAttempt( $input )`, call `logAttempt( $input, true )`. The second parameter stands for _"confirmed_only"_.


## License

Confide is free software distributed under the terms of the MIT license

## Aditional information

Any questions, feel free to contact me or ask [here](http://forums.laravel.io/viewtopic.php?id=4658)

Any issues, please [report here](https://github.com/Zizaco/confide/issues)
