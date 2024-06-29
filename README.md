
<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo"></a></p>

# auth-laravel-10-5-google-login

- Google Auth system
- Role based authentication system where we can give the user permission according to his role.

## Step 0
 - first you need to create a simple working login system 
 - you can take reference of : [auth-laravel-10-1](https://github.com/suraj-repositories/auth-laravel-10-1)


## Steps
- require the 'socialite' package
```sh
composer require laravel/socialite
```

- Now you need to create credentials on google cloud (steps may be changed in near feature)

    - on your browser search for 'google cloud' and visit to the website 
    - login yourself on the google cloud (don't worry this is free)
    - when you logged in the - on the top right corner you can see the 'console' Button - click on that button
    - on the top navbar click on "select a project" selector -> New Project -> fill the project name -> location may be empty -> click on create
    - once the project is created -> select the project from the selector 'select a project' (or it may be selected already)
    - On the sidebar go to API'S and services -> Credentials -> CONFIGURE CONCENT SCREEN -> select External radio btn -> click on create
    - In the 'Edit app registration' there are four section -
        - section 1 : Oauth consent screen
            - fill appname
            - fill user support email
            - fill developer conatact email
            - you can leave other fields empty if you want
            - click on save and continue
        - you can leave other sections empty
    - now go to the credentials on sidebar -> click on create credentials -> Oauth-client-id -> select the application type -> in our case the application type in web application
        - now it need some information
        - first is app name 
        - Authorized javascript origins : here you need to fill the url address or your website in my case it is http://localhost:8000
        - Authorized redirect URIs: here you need to fill the url where you want to send your user when login is successful in my case this is : http://localhost:8000/auth/google/callback
        - click on create 
    - It will create client id and client secret for you -> You need to paste the client id and client-secret into your env file
```sh
GOOGLE_CLIENT_ID=**********************************.apps.googleusercontent.com
GOOGLE_SECRET_ID=***********************************
```
- After the credential setup you need to open config\services.php in which you need to setup the following
```sh
 'google' => [
        'client_id' => env('GOOGLE_CLIENT_ID'),
        'client_secret' => env('GOOGLE_SECRET_ID'),
        'redirect' => "http://localhost:8000/auth/google/callback",
    ],
```
- on your login file create a link for login 
```php
<a href="{{ URL::to('googleLogin') }}">Google Login</a>
```
- setup the route for the URL and success-redirect-url
```php
Route::get('/googleLogin', [AuthController::class, 'googleLogin'])->middleware('guest')->name('googleLogin');

 Route::get('/auth/google/callback', [AuthController::class, 'googleHandler'])->middleware('guest')->name('googleHandler');
```
- setup a controller method to handle google login
```php
 public function googleLogin(){
    return Socialite::driver('google')->redirect();
 }
```
```php
public function googleHandler(){
    try{
        $user = Socialite::driver('google')->user();
        $findUser = User::where('email', $user->email)->first();
        if(!$findUser){
            $findUser = new User();
            $findUser->name = $user->name;
            $findUser->email = $user->email;
            $findUser->password = Hash::make("123");
            $findUser->role = 'USER';
            $findUser->save();
        }
        Auth::login($findUser);
        return redirect('/');
    }catch(Exception $e){
        dd($e->getMessage());
    }
}
```

