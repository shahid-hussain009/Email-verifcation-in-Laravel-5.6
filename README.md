# Email-verifcation-in-Laravel-5.6
# Route
```php
Route::get('verify/{email}/{token}', 'Auth\RegisterController@verifyUser')->name('verify');
```
# Register Contrller 
```php
use Mail;
use App\Mail\verifyUserByEmail;
protected function create(array $data)
    {
        $user = User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => bcrypt($data['password']),
            'verifyToken' => str_random(25),
            'role' => 'organizer',
        ]);
        Mail::to($user->email)->send(new verifyUserByEmail($user));
        return $user;
    }

    public function register(Request $request)
    {
        $this->validator($request->all())->validate();

        event(new Registered($user = $this->create($request->all())));

        //$this->guard()->login($user);
        Auth::logout();


        return redirect('login')->with('message', 'check your email to verify your account');
    }
    public function verifyUser($email, $token){
        $user = User::where(['email'=>$email, 'verifyToken'=>$token])->first();
        if($user){
            $user->verifyToken = '';
            $user->status = 1;
            if($user->save()){
                return redirect('login')->with('message', 'Your Accont successfully activated');
            }else{
                return redirect('err_message', 'Activate Your Account First');
            }

        }else{
            return redirect('err_message', 'Invalid Email and Password');
        }
    }

```
# Login Controller 
```php
protected function authenticated(Request $request, $user)
    {
        $user = User::find(Auth::id());
        if($user->status == 0){
            Auth::logout();
            return redirect('login')->with('err_message', 'Activate Your Account First');
        }
    }

```
# Mail verifyUserByEmail
```php
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Contracts\Queue\ShouldQueue;

class verifyUserByEmail extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Create a new message instance.
     *
     * @return void
     */
    public $user;
    public function __construct($user)
    {
        $this->user = $user;
    }

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('no-reply@admin.com')->from('Admin of ItTeach')->view('auth.verifyUser');
    }
}

```
# Mail view
```php
<a href='{{url("verify/$user->email/$user->verifyToken")}}'>Click here</a> To verify Your account
```
