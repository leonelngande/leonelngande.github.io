---
layout: post
hidden: false
title: >-
  Using Laravel's Default Password Reset Functionality from an API - Sending the
  Notification
tags:
  - Laravel
  - Password Reset
  - API
---
This assumes you've already [setup API authentication](https://medium.com/modulr/create-api-authentication-with-passport-of-laravel-5-6-1dc2d400a7f) in your Laravel application.

I've put together the following steps mainly as some sort of future reference, so please pardon the brevity 🙏.

1. Make sure your user model implements the [`CanResetPassword` interface](https://laravel.com/docs/5.7/passwords#resetting-database).

```php
use Illuminate\Contracts\Auth\CanResetPassword;
use Laravel\Passport\HasApiTokens;
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements CanResetPassword
{
    use HasApiTokens, Notifiable;
    //
}
```

2. Customize your  ResetPasswordController to use the guard of your choice by [overriding](https://laravel.com/docs/5.7/passwords#password-customization) the guard method on the controller. Since we're dealing with an API (like in my case), we customize to use the api guard defined in config/auth.php.

```php
namespace App\Http\Controllers\API\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Foundation\Auth\ResetsPasswords;
use Illuminate\Support\Facades\Auth;

class ResetPasswordController extends Controller
{
    /*
    |--------------------------------------------------------------------------
    | Password Reset Controller
    |--------------------------------------------------------------------------
    |
    | This controller is responsible for handling password reset requests
    | and uses a simple trait to include this behavior. You're free to
    | explore this trait and override any methods you wish to tweak.
    |
    */

    use ResetsPasswords;

    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('guest');
    }

    protected function guard()
    {
        return Auth::guard('web');
    }
}
```

FYI: Notice my use of `namespace App\Http\Controllers\API\Auth;` above, I have a custom namespace for all API routes, that is `App\Http\Controllers\API,` and my authentication controllers are part of this namespace as above.

Also make sure you point your routes to the correct namespaced controllers. Here's how mine are defined:

```php
Route::group([
    'namespace' => 'Auth',
    'middleware' => 'api',
    'prefix' => 'password'
], function () {
    Route::post('forgot', 'ForgotPasswordController');
    Route::post('reset', 'ResetPasswordController');
});
```

3. If you'd like to customize the notification email, and more likely the action url in the email sent to the user, then overide the sendPasswordResetNotification method of the CanResetPassword trait as below.

```php
use App\Notifications\PasswordResetRequest;
use Illuminate\Contracts\Auth\CanResetPassword;
use Laravel\Passport\HasApiTokens;
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements CanResetPassword
{
    use HasApiTokens, Notifiable;
    //

    /**
     * Send the password reset notification.
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new PasswordResetRequest($token));
    }
}
```

4. Update your ForgotPasswordController to look something like this:

```php
use App\Http\Controllers\Controller;
use Illuminate\Foundation\Auth\SendsPasswordResetEmails;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Password;

class ForgotPasswordController extends Controller
{
    /*
    |--------------------------------------------------------------------------
    | Password Reset Controller
    |--------------------------------------------------------------------------
    |
    | This controller is responsible for handling password reset emails and
    | includes a trait which assists in sending these notifications from
    | your application to your users. Feel free to explore this trait.
    |
    */

    use SendsPasswordResetEmails;

    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('guest');
    }

    public function __invoke(Request $request)
    {
        $this->validateEmail($request);
        // We will send the password reset link to this user. Once we have attempted
        // to send the link, we will examine the response then see the message we
        // need to show to the user. Finally, we'll send out a proper response.
        $response = $this->broker()->sendResetLink(
            $request->only('email')
        );
        // return ['received' => true];
        return $response == Password::RESET_LINK_SENT
            ? response()->json(['message' => 'Reset link sent to your email.', 'success' => true], 201)
            : response()->json(['message' => 'Unable to send reset link', 'success' => false], 401);
    }
}
```

What changes is the use of the __invoke function on the controller, turning it into a single action controller. This change is courtesy of [this post](https://blog.nasrulhazim.com/2018/04/laravel-reset-password-from-an-api/) by Nasrul Hazim Bin Mohamad.

Now Laravel will take care of sending the reset password link to your users. Will link this to the next article which will detail the steps after the user clicks on the email link.

Happy coding!
