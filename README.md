# Email-verifcation-in-Laravel-5.6

```php
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
