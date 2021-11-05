# Laravel-Relationship, Email-verifcation,Queue, Notification By Mial and Database, Ajax and Add More
## Date and Zip Create and Download and CURL

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

### Add More in Laravel
[https://itsolutionstuff.com/post/laravel-dynamically-add-or-remove-input-fields-using-jqueryexample.html](https://itsolutionstuff.com/post/laravel-dynamically-add-or-remove-input-fields-using-jqueryexample.html)

```php

public function addMorePost(Request $request)
    {
        $rules = [];
        foreach($request->input('name') as $key => $value) {
            $rules["name.{$key}"] = 'required';
            $rules["color.{$key}"] = 'required';
        }
        $validator = Validator::make($request->all(), $rules);
        if ($validator->passes()) 
        {
            foreach($request->input('name') as $key => $value) 
            {
                Tags::create([
                	'name'=>$value,
                	'color'=>$value,
                ]);
            }
            return response()->json(['success'=>'done']);
        }
        return response()->json(['error'=>$validator->errors()->all()]);
    }
```
### Add More Views
```php
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" />  

 
    <script src="//ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"></script>  
    <meta name="csrf-token" content="{{ csrf_token() }}">

<div class="container">
    <h2 align="center">Laravel - Dynamically Add or Remove input fields using JQuery</h2>  
    <div class="form-group">
         <form name="add_name" id="add_name">  


           <div class="alert alert-danger print-error-msg alert-dismissible" style="display:none">
              <a href="#" class="close" data-dismiss="alert" aria-label="close">&times;</a>
            <ul></ul>

            </div>


            <div class="alert alert-success print-success-msg" style="display:none">
            <ul></ul>
            </div>


            <div class="table-responsive">  
                <table class="table table-bordered" id="dynamic_field">  
                    <tr>  
                        <td>
                          <input type="text" name="name[]" placeholder="Enter your Name" class="form-control name_list" />
                        </td>
                        <td>
                          <input type="text" name="color[]" placeholder="Enter color" class="form-control color_list" />
                        </td>   
                        <td><button type="button" name="add" id="add" class="btn btn-success">Add More</button></td>  
                    </tr>  
                </table>  
                <input type="button" name="submit" id="submit" class="btn btn-info" value="Submit" />  
            </div>


         </form>  
    </div> 
</div>

```
### Add more js
```js
<script type="text/javascript">
    $(document).ready(function(){      
      var postURL = "<?php echo url('addmore'); ?>";
      var i=1;  


      $('#add').click(function(){  
           i++;  
           $('#dynamic_field').append('<tr id="row'+i+'" class="dynamic-added"><td><input type="text" name="name[]" placeholder="Enter your Name" class="form-control name_list" /></td><td><input type="text" name="color[]" placeholder="Enter color" class="form-control color_list" /></td><td><button type="button" name="remove" id="'+i+'" class="btn btn-danger btn_remove">X</button></td></tr>');   
      });  


      $(document).on('click', '.btn_remove', function(){  
           var button_id = $(this).attr("id");   
           $('#row'+button_id+'').remove();  
      });  


      $.ajaxSetup({
          headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
          }
      });


      $('#submit').click(function(){            
           $.ajax({  
                url:postURL,  
                method:"POST",  
                data:$('#add_name').serialize(),
                type:'json',
                success:function(data)  
                {
                    if(data.error){
                        printErrorMsg(data.error);
                    }else{
                        i=1;
                        $('.dynamic-added').remove();
                        $('#add_name')[0].reset();
                        $(".print-success-msg").find("ul").html('');
                        $(".print-success-msg").css('display','block');
                        $(".print-error-msg").css('display','none');
                        $(".print-success-msg").find("ul").append('<li>Record Inserted Successfully.</li>');
                    }
                }  
           });  
      });  


      function printErrorMsg (msg) {
         $(".print-error-msg").find("ul").html('');
         $(".print-error-msg").css('display','block');
         $(".print-success-msg").css('display','none');
         $.each( msg, function( key, value ) {
            $(".print-error-msg").find("ul").append('<li>'+value+'</li>');
         });
      }
    });  
</script>
```
### Add More in php
[https://itsolutionstuff.com/post/php-dynamically-add-remove-input-fields-using-jquery-ajax-example-with-demoexample.html](https://itsolutionstuff.com/post/php-dynamically-add-remove-input-fields-using-jquery-ajax-example-with-demoexample.html)
```php
CREATE TABLE IF NOT EXISTS `tagslist` (

  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,

  `name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,

  PRIMARY KEY (`id`)

) ENGINE=InnoDB  DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci AUTO_INCREMENT=24 ;
### index.php
<!DOCTYPE html>
<html>
<head>
    <title>PHP - Dynamically Add or Remove input fields using JQuery</title>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" />  
    <script src="//ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"></script>
</head>
<body>


<div class="container">
    <h2 align="center">PHP - Dynamically Add or Remove input fields using JQuery</h2>  
    <div class="form-group">
         <form name="add_name" id="add_name">


            <div class="table-responsive">  
                <table class="table table-bordered" id="dynamic_field">  
                    <tr>  
                        <td><input type="text" name="name[]" placeholder="Enter your Name" class="form-control name_list" required="" /></td>  
                        <td><button type="button" name="add" id="add" class="btn btn-success">Add More</button></td>  
                    </tr>  
                </table>  
                <input type="button" name="submit" id="submit" class="btn btn-info" value="Submit" />  
            </div>


         </form>  
    </div> 
</div>


<script type="text/javascript">
    $(document).ready(function(){      
      var postURL = "/addmore.php";
      var i=1;  


      $('#add').click(function(){  
           i++;  
           $('#dynamic_field').append('<tr id="row'+i+'" class="dynamic-added"><td><input type="text" name="name[]" placeholder="Enter your Name" class="form-control name_list" required /></td><td><button type="button" name="remove" id="'+i+'" class="btn btn-danger btn_remove">X</button></td></tr>');  
      });


      $(document).on('click', '.btn_remove', function(){  
           var button_id = $(this).attr("id");   
           $('#row'+button_id+'').remove();  
      });  


      $('#submit').click(function(){            
           $.ajax({  
                url:postURL,  
                method:"POST",  
                data:$('#add_name').serialize(),
                type:'json',
                success:function(data)  
                {
                  	i=1;
                  	$('.dynamic-added').remove();
                  	$('#add_name')[0].reset();
    				        alert('Record Inserted Successfully.');
                }  
           });  
      });


    });  
</script>
</body>
</html>

#### addmore.php
<?php


	define (DB_USER, "root");
	define (DB_PASSWORD, "root");
	define (DB_DATABASE, "test");
	define (DB_HOST, "localhost");
	$mysqli = new mysqli(DB_HOST, DB_USER, DB_PASSWORD, DB_DATABASE);


	if(!empty($_POST["name"])){


		foreach ($_POST["name"] as $key => $value) {
			$sql = "INSERT INTO tagslist(name) VALUES ('".$value."')";
			$mysqli->query($sql);
		}
		echo json_encode(['success'=>'Names Inserted successfully.']);
	}


?>

```
### View Counts Footers
```php

<div class="col-sm-5">
        <div class="dataTables_info" id="example1_info" role="status" aria-live="polite">
          Showing {{($employee->currentpage()-1)*$employee->perpage()+1}} to {{$employee->currentpage()*$employee->perpage()}}
    of  {{$employee->total()}} entries
        </div>
    </div>

```
```php
public function store(BlogRequest $request)
{
try 
{
    $file = $request->file('blog_image');
    $fileName = time().'.'.$file->getClientOriginalExtension();
    $destinationPath = public_path('admin/images/blogs');
    if (!is_dir($destinationPath)) {
	mkdir($destinationPath, 0777, true);
    }
    $file->move($destinationPath,$fileName);
    $blog_image = '/admin/images/blogs/'.$fileName;

    $blog = Blog::create([
	'title' => $request->title,
	'description' => $request->description,
	'image' => $blog_image,
    ]);
    return ['status'=>true, 'message'=>'New blog added successfully!'];
} 
catch (\Exception $e) 
{
   return ['status'=>true, 'message'=>$e->getMessage()];
}
}
```
# Request
```php

public function rules()
    {
        switch (request()->method()){
            case 'POST' :
                return [
                    'blog_image' => 'required|mimes:jpeg,png,jpg,gif,svg',
                    'title' => 'required',
                    'description' => 'required',
                ];
                break;
            case 'PUT' :
                return [
                    'title' => 'required',
                    'description' => 'required',
                ];
                break;
            default :
                return [];
        }
    }

    public function messages()
    {
        return [
            'blog_image.required' => 'Select image',
            'title.required' => 'Title is required',
            'description.required' => 'Description is required',
        ];
    }

```
# Display message
```html
<div>
    <div class="add-blog-success alert alert-success  alert-dismissible" style="display: none;">
    <a href="#" class="close" data-dismiss="alert" aria-label="close" style="color: black;">&times;</a>
</div>
```
# Html
```html
<div class="form-group">
<label class="col-sm-2"></label>
<div class="col-sm-10">
<img alt="Event Image" style="width: 250px; height: 163px;" 
class="img-md file-img1 model-add-image" src="{{asset('public/admin/')}}/images/404-Not-Found.jpg">
<p class="text-muted"></p>
<input type="file" id="fileElem" name="blog_image" class="file1" multiple accept="image/*" 
 style="display:none" onchange="handleFile(this.files)">
<button type="button" id="fileSelect" class="btn btn-primary mar-ver btn-img-file">Blog Image..</button>
<div id="image-error" class="btn-img-file validation-error"></div>
</div>

</div>

```
# Image Preview code on form
```js

     var fileSelect = document.getElementById("fileSelect"),
      fileElem = document.getElementById("fileElem"),
      fileList = document.getElementById("fileList");
      fileSelect.addEventListener("click", function (e) {
          if (fileElem) {
            fileElem.click();
        }
              e.preventDefault(); // prevent navigation to "#"
          }, false);
      function handleFile(files) {
       var preview = document.querySelector('.file-img1');
       var file    = document.querySelector('.file1').files[0];
       var reader  = new FileReader();

       reader.addEventListener("load", function () {
        preview.src = reader.result;
    }, false);

       if (file) {
        reader.readAsDataURL(file);
    }
}
```
# Insert
```js
    #store
    $('#create-blog-submit').submit(function(e) {
    e.preventDefault();
      var data = new FormData($("#create-blog-submit")[0]);
      $.ajax({
        method : 'POST',
        url : "{{route('blog.store')}}",
        data: data,
        contentType:false,
        cache: false,
        processData:false,
        success: function (response) {
            if(response.status)
            {
                $('.add-blog-success').show();
                setTimeout(function() {
                    window.location = "{{route('blog.index')}}";
                }, 3000);
            }
            if(response.status == false)
            {
                $('.add-blog-success').html(response.message);
            }
            else
            {
                $('.add-blog-success').html(response.message);
            }

        },
        error: function (errors) {
           //console.log(errors);
           var er = $.parseJSON(errors.responseText);
           var errors_list = '';
           $('#image-error').html(er.errors.blog_image);
           $('#title-error').html(er.errors.title);
           $('#description-error').html(er.errors.description);
           $.each(er.errors, function (fields, messages) {

               $.each(messages, function (index, msg) {
                   errors_list += '<li>' + msg + '</li>';
               })
           });
           console.log(errors_list);
          $('.alert-danger').html(errors_list);
           setTimeout(function() {
                $('#image-error').html('');
                $('#title-error').html('');
                $('#description-error').html('');
            }, 3000);
       }
   });
  });
</script>
```
# Edit and update
```js
<script type="text/javascript">
$('body').on('click',".edit_blog",function()
{
    var id = $(this).attr("data-id");
    var base_url = "edit-blog/"+id;
    //alert(cert_id);
    $.ajax({
        url: base_url,
        data: { id : id },
        success: function(data){
          console.log('success');
            var result = jQuery.parseJSON(data);
            $('.edit-blog-page').css('display','block');
            $('.list-blog-page').css('display','none');
            $('#blog_id').val(result.id);
            $('#blog_old_image').val(result.image);
            $('.blog-img').attr('src', '{{asset("public/")}}'+result.image);
            $('#blog-title').val(result.title);
            $('#blog-description').val(result.description);

        }
    });
});
</script>
<script type="text/javascript">
  $('#edit-blog-submit').submit(function(e) {
      e.preventDefault();
    var blog_id   = $("#blog_id").val();
      var data = new FormData($("#edit-blog-submit")[0]);
      $.ajax({
        method : 'POST',
        url : "update-blog/"+blog_id,
        data: data,
        contentType:false,
        cache: false,
        processData:false,
        success: function (response) {
            if(response.status)
            {
                $('.edit-blog-success').show();
                setTimeout(function() {
                    window.location = "{{route('blog.index')}}";
                }, 3000);
            }
            if(response.status == false)
            {
                $('.edit-blog-success').html(response.message);
            }
            else
            {
                $('.edit-blog-success').html(response.message);
            }

        },
        error: function (errors) {
          var er = $.parseJSON(errors.responseText);
           var errors_list = '';
           $('#image-error').html(er.errors.blog_image);
           $('#title-error').html(er.errors.title);
           $('#description-error').html(er.errors.description);
           $.each(er.errors, function (fields, messages) {

               $.each(messages, function (index, msg) {
                   errors_list += '<li>' + msg + '</li>';
               })
           });
           console.log(errors_list);
          $('.alert-danger').html(errors_list);
           setTimeout(function() {
                $('#image-error').html('');
                $('#title-error').html('');
                $('#description-error').html('');
            }, 3000);
       }
   });
  });
</script>
```
# Seeder
```php
<?php

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
	
    public function run()
    {
		
    	$user = \DB::table('users')->insert([
            'name' => 'Super Admin',
            'email' =>'admin@gmail.com',
            'password' => bcrypt('123456'),
        ]);

        //profile
        \DB::table('contacts')->insert([
            'phone' => '7493274932',
            'email' => 'abc@gmail.com',
            'address' => 'Hunza',
            'location' => 'Hunza',
            'country' => 'Pakistan',
            'latitude' => '759375349',
            'longitude' => '7239749327',
        ]);
        //project
        \DB::table('abouts')->insert([
            'title' => 'Saleem',
            'description' => 'lorum ipsum doller',
            'image' => '/admin/images/team/1538723992.jpg',
            'location' => 'Hunza',
        ]);
        \DB::table('services')->insert([
            'title' => 'tour',
            'service_image' => '/admin/images/service/service_1.jpg',
            'description' => 'lorum ipsum doller',
        ]);
        \DB::table('team_members')->insert([
            'name' => 'Saleem',
            'profile_pic' => '/admin/images/team/1538723992.jpg',
            'designation' => 'Developer',
            'phone_no' => '122233445',
            'bio' => 'Null',
        ]);
    }
	
}



```
# Pass data to bootstrap model
```php
<a href="#" onclick="editClassTimeTable('{{$timetable->id}}','{{$timetable->description}}','{{$timetable->type}}','{{$timetable->teacher_id}}','{{$timetable->subject_id}}','{{$timetable->date}}','{{$timetable->day}}','{{$timetable->start_time}}','{{$timetable->end_time}}')" data-target="#update-class-timetable" data-toggle="modal" class="btn btn-icon demo-pli-pen-5 icon-lg add-tooltip" data-original-title="Edit Post" data-container="body"></a>

```
# js function editClassTimeTable()
```js
function editClassTimeTable(id, description, type, teacher_id, subject_id, date, day, start_time, end_time)
  {
    $("#id").val(id);
    $("#description").val(description);
    $('#type').val(type).change();
    $('#subject_id').val(subject_id).change();
    $('#teacher_id').val(teacher_id).change();
    $('#edit-class-time-table').val(date);
    $('#class-timetable-day').val(day).change();
    $('.class_start_time').val(start_time);
    $('.class_end_time').val(end_time);
}
```

### Update with model in ajax
```js
<!DOCTYPE html>
<html>
<head>
    <title>Ajax</title>
     <meta charset="utf-8">
     <meta name="csrf-token" content="{{ csrf_token() }}">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
</head>
<body>
<div class="container">
    <div class="row">
        <div id="student-list" style="margin-top: 20px;">
            
        </div>
        <div id="models"></div>
    </div>
</div>

<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js" integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa" crossorigin="anonymous"></script>
<script type="text/javascript">
    $(function() {
        $.ajaxSetup({
            headers: {
                'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
            }
        });
        $.get('{{url("student")}}', function(data){
            $('#student-list').empty().append(data);
        });
        $('#student-list').on('click','#add-student', function(){
            $.get('{{route("student.create")}}', function(data){
            $('#models').empty().append(data);
            $('#basicExampleModal').modal('show');
        });
        });

         $('#student-list').on('click','#std-edit', function(){
            var id = $(this).data('task');
            $.get('{{url("student/edit")}}/'+id, function(data){
            $('#models').empty().append(data);
            $('#editExampleModal').modal('show');
        });
        });

         $('#student-list').on('click','#std-delete', function(){
            var id = $(this).data('task');
            $.get('{{url("student/destroy")}}/'+id, function(data){
            $('#student-list').empty().append(data);
        });
        });

        $('#models').on('submit', '#stdForm', function(e){
            e.preventDefault();
            var formData = $(this).serialize();
            $.post('{{route("student.store")}}',formData, function(data, xhrStatus, xhr){
                $('#student-list').empty().append(data);
                $('#basicExampleModal').modal('hide');
            })
        });
        $('#models').on('submit', '#editStudent', function(e){
            e.preventDefault();
            var formData = $(this).serialize();
            $.post('{{route("student.update")}}',formData, function(data, xhrStatus, xhr){
                $('#student-list').empty().append(data);
                $('#editExampleModal').modal('hide');
            })
        });
    });
</script>
</body>
</html>
```
### View
```html

<div class="panel panel-primary">
    <div class="panel-heading clearfix">
        <h4 class="panel-title pull-left" style="padding-top: 7.5px;">Student</h4>
        <div class="btn-group pull-right">
            <a href="#" class="btn btn-default btn-sm" data-toggle="modal" data-target="#basicExampleModal" id="add-student">Add Student</a>
        </div>
           
        </div>
        <div>
             <ul class="list-group">
                @if($students->all())
                <table class="table table-striped table-bordered">
                    <tr>
                        <th>Name</th>
                        <th>Email</th>
                        <th>Phone</th>
                        <th>Action</th>
                    </tr>
                    @foreach($students as $student)
                    <tr>
                        <td>{{$student->name}}</td>
                        <td>{{$student->email}}</td>
                        <td>{{$student->phone}}</td>
                        <td>
                            <p class="btn btn-success btn-xs" id="std-edit" data-task="{{$student->id}}">Edit</p>
                            <p class="btn btn-danger btn-xs" id="std-delete" data-task="{{$student->id}}">Delete</p>
                        </td>
                    </tr>
                    @endforeach
                </table>
                @else
                <li class="list-group-item">Record Not found</li>
                @endif
            </ul>
        </div>
    </div>
</div>
```
### Add Model from
```html
<!-- Modal -->
<div class="modal fade" id="basicExampleModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="exampleModalLabel">Student Model</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
       

        <form method="post" action="{{route('student.store')}}" id="stdForm">
          @csrf
            <div class="form-group">
              <label>Name</label>
              <input type="text" name="name" id="name" class="form-control" placeholder="Name">
              
            </div>
             <div class="form-group">
              <label>Email</label>
              <input type="text" name="email" id="email" class="form-control" placeholder="Email">
              
            </div>
             <div class="form-group">
              <label>Phone</label>
              <input type="text" name="phone" id="phone" class="form-control" placeholder="Phone">
              
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
              <input type="submit" class="btn btn-primary" value="Save Data">
            </div>
      </form>
    </div>
  </div>
</div>
```
 ### Edit Model form
 ```html
 <!-- Modal -->
<div class="modal fade" id="editExampleModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="exampleModalLabel">Student Model</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
       

        <form method="post" action="{{route('student.update')}}" id="editStudent">
          @csrf
          <input type="hidden" name="id" value="{{$student->id}}">
            <div class="form-group">
              <label>Name</label>
              <input type="text" name="name" class="form-control" placeholder="Name" value="{{$student->name}}">
              
            </div>
             <div class="form-group">
              <label>Email</label>
              <input type="text" name="email" class="form-control" placeholder="Email" value="{{$student->email}}">
              
            </div>
             <div class="form-group">
              <label>Phone</label>
              <input type="text" name="phone" class="form-control" placeholder="Phone" value="{{$student->phone}}">
            </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
          <button type="submit" class="btn btn-primary">Save changes</button>
        </div>
      </form>
    </div>
  </div>
</div>
 ```
 
 ### Show date in laravel bladed
```php

{{date('l jS F Y ', strtotime($user->created_at))}}
```
```php
  $time_ago = strtotime($notify->created_at);
  $cur_time   = time();
  $time_elapsed   = $cur_time - $time_ago;
  $seconds    = $time_elapsed ;
  $minutes    = round($time_elapsed / 60 );
  $hours      = round($time_elapsed / 3600);
  $days       = round($time_elapsed / 86400 );
  $weeks      = round($time_elapsed / 604800);
  $months     = round($time_elapsed / 2600640 );
  $years      = round($time_elapsed / 31207680 );
  // Seconds
  if($seconds <= 60)
  {
      echo "just now";
  }
  //Minutes
  else if($minutes <=60)
  {
      if($minutes==1)
      {
          echo "one minute ago";
      }
      else
      {
          echo "$minutes minutes ago";
      }
  }
  //Hours
  else if($hours <=24)
  {
      if($hours==1)
      {
          echo "an hour ago";
      }
      else
      {
          echo "$hours hrs ago";
      }
  }
  //Days
  else if($days <= 7)
  {
      if($days==1)
      {
          echo "yesterday";
      }
      else
      {
          echo "$days days ago";
      }
  }
  //Weeks
  else if($weeks <= 4.3)
  {
      if($weeks==1)
      {
          echo "a week ago";
      }
      else
      {
          echo "$weeks weeks ago";
      }
  }
  //Months
  else if($months <=12)
  {
      if($months==1)
      {
          echo "a month ago";
      }
      else
      {
          echo "$months months ago";
      }
  }
  //Years
  else{
      if($years==1)
      {
          echo "one year ago";
      }
      else
      {
          echo "$years years ago";
      }
  }
```

## Last day of the month
```php
public function get_date()
    {
    	//Get Last Day Of The Current Month:
    	$lastDay = date('t',strtotime('today'));
    	print_r($lastDay);

    	//Get Last Day Of The Current Month 2:
    	$lastDateOfThisMonth =strtotime('last day of this month') ;
    	$lastDay = date('d/m/Y', $lastDateOfThisMonth);
    	print_r($lastDay);

    	//Get Last Day Of The Next Month:
    	$lastDay = date('t',strtotime('next month'));
    	print_r($lastDay);

    	//Get Last Day Of The Next Month 2:
    	$lastDateOfNextMonth =strtotime('last day of next month') ;
    	$lastDay = date('d/m/Y', $lastDateOfNextMonth);
    	print_r($lastDay);

    	//Get Last Day Of The Previous Month:	
    	$lastDay = date('t',strtotime('last month'));
    	print_r($lastDay);


    	//Get Last Day Of The Previous Month 2:
    	$lastDateOfLastMonth =strtotime('last day of last month') ;
    	$lastDay = date('d/m/Y', $lastDateOfLastMonth);
    	print_r($lastDay);

    	//Get Last Day Of The Specific Month:
    	$lastDay = date('t',strtotime('1/1/2018'));
    	print_r($lastDay);
    }

```
## PHP CURL 
use this code in helper in your project
```php
function callMailchimpAPI($method, $url, $data=null)
{
    $curl = curl_init();
    /*$info = curl_getinfo($curl);
    echo "<pre>";
    print_r( $info );*/
    switch ($method)
    {
        case "POST":
            curl_setopt($curl, CURLOPT_POST, 1);
            curl_setopt($curl, CURLOPT_POSTFIELDS, $data);
         break;
        case "PATCH":
            curl_setopt($curl, CURLOPT_CUSTOMREQUEST, "PATCH");
        if ($data)
            curl_setopt($curl, CURLOPT_POSTFIELDS, $data);                              
        break;
        case "DELETE":
            curl_setopt($curl, CURLOPT_CUSTOMREQUEST, "DELETE"); 
            curl_setopt($curl, CURLOPT_POSTFIELDS, $data);
        default:
        if($data)
            $url = sprintf("%s?%s", $url, http_build_query($data));
   }
    
    $headers = array(  
        "--user: 93353d176c81e3391109a76455f50045-us7",
        "Authorization: Basic b3dhaXNfdGFhcnVmZjo5MzM1M2QxNzZjODFlMzM5MTEwOWE3NjQ1NWY1MDA0NS11czc=",
        "Content-Type: application/json",
        "Postman-Token: 163f7134-68ca-45bb-9420-ebf2bef7f447",
        "cache-control: no-cache"
     );
    curl_setopt_array($curl, 
    [
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_SSL_VERIFYPEER => false,
        CURLOPT_SSL_VERIFYHOST => false,
        CURLOPT_HTTPHEADER => $headers, 
        CURLOPT_HEADER => false,
        CURLOPT_URL => $url.'?apikey=93353d176c81e3391109a76455f50045-us7'
    ]);

    $response = curl_exec($curl);
    curl_close($curl);
    return $response;
}

function createMailChimpStore($id)
{
    $curl = curl_init();
    $store_name = "store_";
    curl_setopt_array($curl, array(
    CURLOPT_URL => "https://us7.api.mailchimp.com/3.0//ecommerce/stores/",
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_ENCODING => "",
    CURLOPT_MAXREDIRS => 10,
    CURLOPT_TIMEOUT => 30,
    CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
    CURLOPT_CUSTOMREQUEST => "POST",
    CURLOPT_POSTFIELDS => '{
      "id" : "'.$store_name.$id.'",
      "list_id" : "317fbacff8",
      "name" : "'.$store_name.$id.'",
      "domain" : "http://boksha.eshmar.com/storew/inspiration",
      "email_address" : "inspiratiossn@boksha.com",
      "currency_code" : "AED"
    }',
     CURLOPT_HTTPHEADER => array(
        "--user: 93353d176c81e3391109a76455f50045-us7",
        "Authorization: Basic b3dhaXNfdGFhcnVmZjo5MzM1M2QxNzZjODFlMzM5MTEwOWE3NjQ1NWY1MDA0NS11czc=",
        "Content-Type: application/json",
        "Postman-Token: 8621048d-e026-4135-b456-400b3f3ec523",
        "cache-control: no-cache"
      ),
    ));

    $response = curl_exec($curl);
    $err = curl_error($curl);

    curl_close($curl);
}

/* Store Product */
function curlMailchimpStore($data_array, $product)
{
    $url = "https://us7.api.mailchimp.com/3.0/ecommerce/stores";
    $data = callMailchimpAPI('GET', $url, false);
    $result = json_decode($data);

    $store_ids = array();
    foreach ($result->stores as $store) 
    {
         $store_ids[] =  $store->id;            
    }
    /* Distinct Srore */
    if (in_array("store_".$product->title, $store_ids))
    {
        $url = "https://us7.api.mailchimp.com/3.0/ecommerce/stores/store_$product->title/products";
        $data = callMailchimpAPI('POST', $url, $data_array);
    }
    else
    {
        createMailChimpStore($product->title);
        $url = "https://us7.api.mailchimp.com/3.0/ecommerce/stores/store_$product->title/products";
        $data = callMailchimpAPI('POST', $url, $data_array);
    }
    /* Common store */
    if (in_array("store_all_products", $store_ids))
    {
        $url = "https://us7.api.mailchimp.com/3.0/ecommerce/stores/store_all_products/products";
        $data = callMailchimpAPI('POST', $url, $data_array);
    }
    else
    {
        $id = 'all_products';
        createMailChimpStore($id);
        $url = "https://us7.api.mailchimp.com/3.0/ecommerce/stores/store_all_products/products";
        $data = callMailchimpAPI('POST', $url, $data_array);
    }
}

/* Update Product */
function curlMailchimpUpdate($data_array, $product)
{

    $url = "https://us7.api.mailchimp.com/3.0/ecommerce/stores/store_$product->title/products/$product->id";
    $data = callMailchimpAPI('PATCH', $url, $data_array);

    $url = "https://us7.api.mailchimp.com/3.0/ecommerce/stores/store_all_products/products/$product->id";
    $data = callMailchimpAPI('PATCH', $url, $data_array);
}
    /*Delete Product */
function curlMailchimpDelete($id)
{
    $delete_id = curl_product_query($id);
    $url = "https://us7.api.mailchimp.com/3.0/ecommerce/stores/store_$delete_id->title/products/$id";
    $data = callMailchimpAPI('DELETE', $url, false);

    $url = "https://us7.api.mailchimp.com/3.0/ecommerce/stores/store_all_products/products/$id";
    $data = callMailchimpAPI('DELETE', $url, false);
}

function curl_data_array($product)
{
    $product_url = "http://boksha.com/product/$product->slug";
    return  $data_array = '
    {
            "id": "'.$product->id.'", 
            "title": "'.$product->name.'", 
            "handle": "'.$product->slug.'", 
            "url": "'.$product_url.'", 
            "description": "'.$product->description.'", 
            "type": "'.$product->slug.'",
            "vendor": "'.$product->title.'", 
            "image_url": "'.$product->image_path.'", 
            "variants": [
                { 
                    "id": "'.$product->id.'",
                    "title": "'.$product->name.'",
                    "url": "'.$product_url.'",
                    "sku": "",
                    "price": "'.$product->price.'",
                    "inventory_quantity": 0,
                    "image_url": "'.$product->image_path.'",
                    "backorders": "0",
                    "visibility": "visible",
                    "created_at": "'.$product->created_at.'",
                    "updated_at": "'.$product->updated_at.'"
                }
            ]
     }';
}

function curl_product_query($id)
{
    return $product = DB::table('products')
        ->join('stores', 'stores.id', '=', 'products.store_id')
        ->leftJoin('product_images', 'products.id', 'product_images.product_id')
        ->select('products.*', 'stores.title','product_images.image_path')
        ->whereNull('products.deleted_at')
        ->groupBy("products.id")
        ->where('products.id',  $id)
        ->first();
}

```
### use this in method
```php
public function store(ProductsRequest $request) {
    $last_inserted_id = $product->id;
    $product = curl_product_query($last_inserted_id);
    $data_array = curl_data_array($product);
    curlMailchimpStore($data_array, $product);
}
```
### Zoom Image
click here to read zoom image 

(http://mark-rolich.github.io/Magnifier.js/)[http://mark-rolich.github.io/Magnifier.js/]

## Multiple images download and delete in Laravel/Codeigniter
```js
$(document).ready(function () 
{
    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

    $('#download-master').on('click', function(e) 
    {
        if($(this).is(':checked',true))  
        {
            $(".multiple_download").prop('checked', true);  
        } else {  
            $(".multiple_download").prop('checked',false);  
        }  
    });

    $('.download-all').on('click', function(e) {

        var allVals = [];  
        $(".multiple_download:checked").each(function() {  
            allVals.push($(this).attr('data-id'));
        });  

        if(allVals.length <=0)  
        {  
            alert("Please select row to download.");  
        }  
        else 
        {  
            var join_selected_values = allVals.join(",");
            $.ajax({
                url: $(this).data('url'),
                type: 'POST',
                data:'_token = <?php echo csrf_token() ?>',
                data: 'ids='+join_selected_values,
                success: function (data) 
                {
                    //data.fileurl,
                    //window.location = response;
                    alert('Seccussfully Downloaded');
                },
                error: function (data) {
                    alert(data.responseText);
                }
            }); 
        }  
    });

    $('.download-all').on('click', function(e){
        e.preventDefault();
        var allVals = [];  
        $(".multiple_download:checked").each(function() {  
            allVals.push($(this).attr('data-id'));
        });  

        if(allVals.length <=0)  
        {  
            console.log("Please select row to download.");  
        }
        else
        {
            $('.zip-file').multiDownload();
        }
        
    });
});

```
### MultiDownload js
```js
(function ($) {

    var methods = {
        _download: function (options) {
            var triggerDelay = (options && options.delay) || 100;
            var cleaningDelay = (options && options.cleaningDelay) || 1000;

            this.each(function (index, item) {
                methods._createIFrame(item, index * triggerDelay, cleaningDelay);
            });
            return this;
        },

        _createIFrame: function (item, triggerDelay, cleaningDelay) {
            setTimeout(function () {
                var frame = $('<iframe style="display: none;" class="multi-download-frame"></iframe>');
                frame.attr('src', $(item).attr('href') || $(item).attr('src'));
                $(item).after(frame);
                setTimeout(function () { frame.remove(); }, cleaningDelay);
            }, triggerDelay);
        }
    };

    $.fn.multiDownload = function(options) {
        return methods._download.apply(this, arguments);
    };

})(jQuery);
```

### Code
```php
public function downloads_all(Request $request) 
     {
        $public_dir = '';
        $$zipFileName = '';
        $zip = '';
        $product_images =  DB::table('product_images')
                    ->where('deleted_at', null)
                    ->whereIn('product_id',  explode(',', $request->ids))
                    ->get();
        if ($product_images) 
        {
            foreach ($product_images as $value) 
            {
                $zipFileName = 'product_' . $value->product_id . '_images.zip';
                $zip = new ZipArchive;
                $public_dir = public_path('uploads/download-images');
                if (!is_dir($public_dir)) 
                {
                    mkdir($public_dir, 0777, true);
                }
                if ($zip->open($public_dir . '/' . $zipFileName, ZipArchive::CREATE) === TRUE) 
                {
                    foreach ($product_images as $image) 
                    {
                        $directory_path = base_path() . '/public/uploads/products/' . $image->product_id . '/images/';
                        $image_path = $image->image_path;
                        $file_name = basename($image_path);
                        $zip->addFile($directory_path . $file_name, $file_name);
                    }
                    $zip->close();
                }
            }
            $files = glob(public_path('uploads/download-images'));
            \Zipper::make(public_path('uploads/product_images.zip'))->add($files)->close();


            $fileurl =  public_path('uploads/product_images.zip');
    
            
            if (file_exists($fileurl)) 
            {
                return 'success';
                //return response()->download($fileurl);
            }
            else
            {
                return back();
            }
            
           /* $headers = [
                'Access-Control-Allow-Origin' => '*',
                'Access-Control-Allow-Methods' => 'POST, GET, OPTIONS, PUT, PATCH, DELETE',
                'Access-Control-Allow-Headers' => 'Access-Control-Allow-Headers, Origin,Accept, X-Requested-With, Content-Type, Access-Control-Request-Method, Authorization , Access-Control-Request-Headers',
                'Content-Type' => 'application/zip'
                ];
            return response()->download(public_path('uploads/'), 'product_images.zip', $headers);*/
        }
        else
        {
            return back();
        }
    }
```
### views
```html
<div class="col-md-2">
  <input type="button" id="download-button" value="Download" class="btn sbold green float-right download-all" data-url="{{url('admin/product/download-all')}}" style="margin-left: 12px !important;">
</div>
<a class="zip-file" href="{{ url('public/uploads/product_images.zip') }}"></a>

<td>
  <label class="mt-checkbox mt-checkbox-single mt-checkbox-outline">
    <input type="checkbox" class="checkboxes bulk-checkbox multiple_download" value=" {{$product->id}}" name="product_ids[]" data-id="    {{$product->id}}"/>
<span></span>
  </label>
</td>

````
### Read file from folder in laravl
```php
$files = File::files(public_path('uploads/download-images'));
if ($files !== false) 
{
    $filecount = count($files);
   foreach ($filecount as $file) {
        return response()->download($filetopath, $zipFileName, $headers);
   }
}
```
### Multiple rows update and insert in laravel
 ```php
 function update_financial_report(Request $request)
    {
        $order_item_id = $request->order_item_id;
        $financial_notes = $request->financial_notes;
        $financial_comments = $request->financial_comments;
        $transfer_amount = $request->transfer_amount;
        $transfer_date = $request->transfer_date;

        foreach($order_item_id as $k => $id){
          $values = array(
                'financial_notes'=> $financial_notes[$k],
                'financial_comments'=> $financial_comments[$k],
                'transfer_amount'=> $transfer_amount[$k],
                'transfer_date'=> $transfer_date[$k]
            );
          OrderToStore::where('id',$id)->update($values);

        }
        return redirect()->back();
    }
 ````
### Html 
```html
<input type="hidden" value="<?php echo $order->order_item_id; ?>" name="order_item_id[]">
 <input type="text" name="transfer_amount[]" value="<?php echo $net_transfer_store; ?>">
```

# Multiple Image upload
```php
public function vehicle_verification(Request $request)
    {

        $files=[];
        $destinationPath = public_path('/uploads/vehicle_verification');
        if (!is_dir($destinationPath)) 
        {
            mkdir($destinationPath, 0777, true);
        }
        foreach ($request->file('dv_vehicle_img') as $image) 
        {
            if (!empty($image)) 
            {
                $fileName = $image->getClientOriginalName();
                $image->move($destinationPath,$fileName);
                //dd($request->all());
                $files[] = '/uploads/vehicle_verification/'.$fileName;
            }

        }

        
        DB::table('tms_vehicle_detail')
            ->where('dv_vehicle_id', $request->dv_vehicle_id)
            ->update([
                'dv_vehicle_img' =>  implode(',',$files),
                'remarks' =>  $request->remarks,
                'is_verify' =>  $request->is_verify,
            ]);
        return redirect()->back();
    }

```


