# Email-verifcation
# Laravel-Relationship

# Controller 
```php

public function index()
{
	$profiles = Profile::with(['user', 'level'])->where('is_bd_partner', 'Yes')->get();
}

```
# Profile Model
```php
class Profile extends Model
{
    public function user()
    {
    	return $this->belongsTo(User::class, 'user_id');
    }
    public function level()
    {
    	return $this->belongsTo(Level::class, 'level_id');
    }
}

```
# User Model
```php
class User extends Authenticatable
{
    use Notifiable;
    use HasRoles;

    public function setPasswordAttribute($password)
    {   
        $this->attributes['password'] = bcrypt($password);
    }
    public function userName()
    {   
        return $this->profile->first_name.' '.$this->profile->last_name;
    }

    public function profile()
    {
        return $this->hasOne(Profile::class, 'id');
    }
}
```
# Level Model
```php
class Level extends Model
{
    protected $guarded = [];

    public function profile()
    {
    	return $this->hasOne(Profile::class, 'id');
    }
}
```
# Render data to view
```php
@php $sn = 1; @endphp
@foreach ($profiles as $profile)
<tr>
	<td>{{$sn++}}</td>
	<td>{{ ucfirst($profile->first_name)}}</td>
	<td>{{ucfirst($profile->last_name)}}</td>
	<td>{{$profile->user->email}}</td>
	<td>{{$profile->cnic}}</td>
	<td>{{$profile->phone_no}}</td>
	<td>{{$profile->level->level}}</td>
	<td>{{$profile->limit}}</td>
	<td>{{date('l jS F Y ', strtotime($profile->created_at))}}</td>
	<td>
	<a href="{{ URL::to('dbd-partner/edit/'.$profile->id) }}" class="btn btn-info btn-xs pull-left" style="margin-right: 3px;">Edit</a>
	<a href="{{ URL::to('dbd-partner/destroy/'.$profile->id) }}" class="btn btn-danger btn-xs">Delete</a>

	</td>
</tr>
@endforeach
@else
<td>{{date(' F jS, Y ', strtotime($timetable->date))}}</td>
@endif
<td>{{$timetable->day}}</td>
<td>{{date('h:i:s a', strtotime($timetable->start_time))}}</td>
<td>{{date('h:i:s a', strtotime($timetable->end_time))}}</td>

</tr>
@endforeach

```

# First Trun on less secure app by following below link
```php
https://myaccount.google.com/security#connectedapps

https://accounts.google.com/UnlockCaptcha 

```
# .env set up
```php

MAIL_DRIVER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=test_hello@gmail.com
MAIL_PASSWORD=yourpassword
MAIL_ENCRYPTION=tls

```
# Route
```php
Route::get('verify/{email}/{token}', 'Auth\RegisterController@verifyUser')->name('verify');
```
# Make Mail
### php artisan make:mail verifyUserByEmail
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
