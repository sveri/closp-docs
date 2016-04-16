{:title "Authentication"
 :layout :page
 :page-index 700}


As you may have noticed, authentication is already part of closp. With every project comes a set of
code that takes care of it.


### SQL file

There are two files in `resources/migrators/[h2|sqlite]` that take care of creation and deletion of the user
table during `lein migrate`.
It also creates an admin user during that process that can be used to login to the application and gain acces to
admin privileged actions.

> Username: _admin@localhost.de_
> Password: _admin_

Please change that password or initial user to your liking, but never deploy it like that anywhere in production
as this password is not a secret.



### Admin interface

If you browse to: <http://localhost:3000/admin/users> after login you will see a screen similar to this:

![Admin Interface](/img/authentication-01.png)

This interface currently supports the following features:

- Create a new user
- Edit an existing user (setting role / active status)
- Delete an existing user
- Filter for users


### Self Registration

Closp supports the ability for users to register themselves. To enable that feature open _closp.edn_ and
set :registration-allowed? to true.
Afterwards reload the configuration by executing `(reload)` in the REPL.

Self registration follows this process:

1. Click on the _Sign Up_ button
2. Fill out the form (Email / Password / Confirmation)
3. Afterwards an email is sent to the user with an activation link
4. User clicks on that link to get activated
5. Only then he is allowed to login

Of course all of that can be changed as all the code is there and no magic is involved.

There are more options in the configuration that are related to self registration:

- A whole set of email options needed to send the activation email. Please see the chapter about email
configuration for more information
- :captcha-enabled? - if set to true users will have to pass the captcha test
- :captcha-public-key - the public captcha key
- :private-recaptcha-key - the private captcha key
- :recaptcha-domain - the captcha domain

Please be aware that the captcha implementation only work with googles
[recaptcha](https://www.google.com/recaptcha/intro/index.html). If you want to use a different provider
you have to implement it yourself.

