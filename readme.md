<p>Passport</p>

<ul>
	<li>composer create-project --prefer-dist laravel/laravel api</li>
	<li>composer require laravel/passport</li>
	<li>php artisan migrate</li>
	<li>php arisan passport:install</li>
<ul>

	Langka Selanjtutn Setiting 3 file

app/User.php di model
app/providers/AuthServiceProvider di provider
confgi/auth di auth

tambhakan 

use Laravel\Passport\HasApiToken;
lali use traitnya 
use HasApiTokens,Notifiable

lalu ke provider folder
use Laravel\Passport\Passport;
hilangkan komentar 'App\Model'
	 protected $policies = [
        'App\Model' => 'App\Policies\ModelPolicy',
    ];

passang routes passport di boot
$this->registerPolicies();
Passport::routes();

lalu ke congi auth.php

ganti 
'guards' => [
   'web' => [
      'driver' => 'session',
      'provider' => 'users',
   ],

   'api' => [
       'driver' => 'passport', /* yang ini yang di ganti */
       'provider' => 'users',
   ],
]


lalu selanjunta buat routes nya di routes/api.php

Route::post('login', 'API\UserController@login');
Route::post('register', 'API\UserController@register');

Route::group(['middleware' => 'auth:api'], function(){
    Route::post('details', 'API\UserController@details');
});

buat user controller 
php artisan make:controller API/UserController

langkah akhir 

<?php
namespace App\Http\Controllers\API;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use App\User;
use Illuminate\Support\Facades\Auth;
use Validator;

class UserController extends Controller
{

    public $successStatus = 200;

    public function login(){
        if(Auth::attempt(['email' => request('email'), 'password' => request('password')])){
            $user = Auth::user();
            $success['token'] =  $user->createToken('nApp')->accessToken;
            return response()->json(['success' => $success], $this->successStatus);
        }
        else{
            return response()->json(['error'=>'Unauthorised'], 401);
        }
    }

    public function register(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'name' => 'required',
            'email' => 'required|email',
            'password' => 'required',
            'c_password' => 'required|same:password',
        ]);

        if ($validator->fails()) {
            return response()->json(['error'=>$validator->errors()], 401);            
        }

        $input = $request->all();
        $input['password'] = bcrypt($input['password']);
        $user = User::create($input);
        $success['token'] =  $user->createToken('nApp')->accessToken;
        $success['name'] =  $user->name;

        return response()->json(['success'=>$success], $this->successStatus);
    }

    public function details()
    {
        $user = Auth::user();
        return response()->json(['success' => $user], $this->successStatus);
    }
}


lalu php artisan serve dan uji di postman

http://localhost:8000/api/login
http://localhost:8000/api/register
http://localhost:8000/api/details

register dulu baru login 

link login
method => post 
ke body ===> x-www-form-urlencode 
key => email | value => terserah
key => password | value => terserah

jika berhasil token akan muncul
ke headers untuk masukan token jika ingin lihat details
key => Authorization | value = Bearer Token_yang_di_dapat 

jika blm register maka unauth 

link register
method => post
ke body ===> x-www=form-urlencode
key => email | value => terserah
key => password | value => terserah
key => name | value => terserah
key => c_password | value => terserah

link detiail 
masukan token di header 
lalu arahkan link dengan method post