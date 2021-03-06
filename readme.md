<p align="center"><img src="https://laravel.com/assets/img/components/logo-laravel.svg"></p>

<p align="center">
<a href="https://travis-ci.org/laravel/framework"><img src="https://travis-ci.org/laravel/framework.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/d/total.svg" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/v/stable.svg" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/license.svg" alt="License"></a>
</p>

## About Laravel
`php artisan make:auth`

In <b>Config>auth.php</b> It is need to built admin guards
```php
'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'token',
            'provider' => 'users',
        ],

        'admin' => [
            'driver' => 'session',
            'provider' => 'admins',
        ],

        'admin-api' => [
            'driver' => 'token',
            'provider' => 'admins',
        ],

    ],
```
and then it is also need to built provider
```php
'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\User::class,
        ],
        'admins' => [
            'driver' => 'eloquent',
            'model' => App\Admin::class,
        ],
    ],
```
Then, specify the admin password reset time

```php
    'passwords' => [
        'users' => [
            'provider' => 'users',
            'table' => 'password_resets',
            'expire' => 60,
        ],

        'admins' => [
            'provider' => 'admins',
            'table' => 'password_resets',
            'expire' => 15,
        ],
    ],
```
Run `php artisan make:model Admin -m`

Database> Migrations > 2019_06_05_024213_create_admins_table.php
```php
    public function up()
    {
        Schema::create('admins', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('email')->unique();
            $table->string('job_title');
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }
```

Admin.php
```php
namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class Admin extends Authenticatable
{
    use Notifiable;

    protected $guard = 'admin';
```

Run `php artisan make:controller AdminController -r`

AdminController.php
```php
    public function __construct()
    {
        $this->middleware('auth:admin');
    }
```

Web.php
```php
    Route::prefix('admin')->group(function(){
	Route::get('/login', 'Auth\AdminLoginController@showLoginForm')->name('admin.login');
	Route::post('/login', 'Auth\AdminLoginController@login')->name('admin.login.submit');
	Route::get('/', 'AdminController@index')->name('admin.dashboard');
});
```
LoginController.php
```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Foundation\Auth\AuthenticatesUsers;

class LoginController extends Controller
{
    /*
    |--------------------------------------------------------------------------
    | Login Controller
    |--------------------------------------------------------------------------
    |
    | This controller handles authenticating users for the application and
    | redirecting them to your home screen. The controller uses a trait
    | to conveniently provide its functionality to your applications.
    |
    */

    use AuthenticatesUsers;

    /**
     * Where to redirect users after login.
     *
     * @var string
     */
    protected $redirectTo = '/home';

    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('guest')->except('logout');
        $this->middleware('guest:admin')->except('logout');
    }
    
}
```

Run `php artisan make:controller Auth\AdminLoginController -r`

AdminLoginController.php
```php
<?php

namespace App\Http\Controllers\Auth;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use Auth;

class AdminLoginController extends Controller
{
	public function __construct()
    {
        $this->middleware('guest:admin');
    }
    public function showLoginForm(){
    	return view('auth.admin-login');
    }

    public function login(Request $request)
    {
    	$this->validate($request,[
    		'email' => 'required|email',
    		'password' => 'required|min:6'
    	]);

    	if (Auth::guard('admin')->attempt(['email' => $request->email, 'password' => $request->password], $request->get('remember'))) {

            return redirect()->intended('admin');
        }
        return back()->withInput($request->only('email', 'remember'));

    	//Attempt to log the user in
    	

    }
}
```

admin.blade.php
```php

@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">ADMIN Dashboard</div>

                <div class="card-body">
                    @if (session('status'))
                        <div class="alert alert-success" role="alert">
                            {{ session('status') }}
                        </div>
                    @endif

                    You are logged in!<strong>ADMIN</strong>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```


`php artisan tinker`<br>

` $admin = new App\Admin`




Run `php artisan config:clear`

Run `php artisan cache:clear`

`php artisan serve`

For User
![image](https://user-images.githubusercontent.com/29988949/58376753-ad085500-7f26-11e9-931c-77f02be9d50c.png)
`localhost/`

For Admin
![image](https://user-images.githubusercontent.com/29988949/58376738-43884680-7f26-11e9-9ba2-17d0e53b9245.png)
`localhost/admin/login`

Hope this Help
