# Laravel-Relationship, Email-verifcation,Queue, Notification By Mial and Database

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

### LINKS

[https://appdividend.com/2017/12/21/laravel-queues-tutorial-example-scratch/](https://appdividend.com/2017/12/21/laravel-queues-tutorial-example-scratch/)
### Run these artisan command
```php
php artisan queue:table

php artisan migrate

php artisan make:job SendEmailJob

```
### Before Submit Data open cmd and run
```php

php artisan queue:work --queue=your queue name --tries=3

```

### Check Logs
```php
cd storage/logs/
nano laravel-2019-05-15.log
```
### Add this in Controller
```php
use App\Jobs\NotificationJob;

 NotificationJob::dispatch($request->audience, $request->title, $request->description)
 ->onQueue('notification')->delay(Carbon::now()->addSeconds(3));
```

### App\Jobs

```php
 protected $type;
    protected $title;
    protected $description;
    public function __construct($type, $title, $description)
    {
        $this->type = $type;
        $this->title = $title;
        $this->description = $description;
    }


```

## First Create an account 
[https://pusher.com/](https://pusher.com/)

### BY Mail
### Run the artisan commond
```php

php artisan make:notification BookingNotification
```
### Past the keys in your env
```php
env
PUSHER_APP_ID=4234324324
PUSHER_APP_KEY=***************
PUSHER_APP_SECRET=*******************
```

# Model 
use this in your model from where you want to send notification
```php
use Illuminate\Notifications\Notifiable;
class User extends Authenticatable
{
    use Notifiable;
}
```
# App\Notification folder
```php
namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Notifications\Notification;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Messages\MailMessage;

class BookingNotification extends Notification
{
    use Queueable;

    /**
     * Create a new notification instance.
     *
     * @return void
     */
    private $user = '';
    protected $message; 
    public function __construct($user, $message)
    {
        $this->user = $user;
        $this->message = $message;
    }

    /**
     * Get the notification's delivery channels.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return ['mail','database'];
    }

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
	    ->line('Hi '.$this->user)
	    ->line($this->message)
	    ->action('Welcome to Fastinternetcable', url('http://dev.fastinternetcable.com'))
	    ->line('Thank you for using our application!');
    }

    /**
     * Get the array representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            'data' => 'New Booking has been created'
        ];
    }
}


```
# Controller

```php
use App\Notifications\BookingNotification;
public function index()
{
	$user = App\User::first();
	$message = "New Booking Created";
	$this->sendNotification($user, $message);
}

private function sendNotification($user, $message)
{
	try 
	{
	    $user->notify(new BookingNotification($user->name, $message));
	} 
	catch (Exception $exception) 
	{
	    echo $exception->getMessage();
	}
}

```
# Database Notifications
### Run artisan commond
```php
php artisan notifications:table

php artisan migrate
php artisan make:notification TaskCompletedNotification
```
# Routs
```php
use App\Notifications\TaskCompletedNotification;
Route::get('/', function () {

	$user = App\User::first();
	$user->notify(new TaskCompletedNotification);
    return view('welcome');
});

Route::get('/markAsRead', function(){
	auth()->user()->unreadNotifications->markAsRead();
	return redirect()->back();
})->name('markAsRead');

```

# Views

```html
<li class="dropdown">
<a  class="nav-link dropdown-toggle" href="#" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
<i class="fa fa-bell"></i>
@if(auth()->user()->unReadNotifications->count())
<span class="badge badge-light">{{ auth()->user()->unReadNotifications->count() }}</span>
@endif
</a>

<ul class="dropdown-menu">
<li><a style="margin-left: 27px;" href="{{route('markAsRead')}}">Mark all as read</a></li>
@foreach(auth()->user()->unReadNotifications as $notify)
<li class="dropdown-item" href="#">
<a href="#">{{$notify->data['data']}}</a>
</li>
@endforeach
</ul>
</li>

```
# App\Notification TaskCompletedNotification
```php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Notifications\Notification;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Messages\MailMessage;

class TaskCompletedNotification extends Notification
{
    use Queueable;

    /**
     * Create a new notification instance.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Get the notification's delivery channels.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return ['database'];
    }

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->line('The introduction to the notification.')
                    ->action('Notification Action', url('/'))
                    ->line('Thank you for using our application!');
    }

    /**
     * Get the array representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            'data' => 'This is my first notification',
        ];
    }
}


```
