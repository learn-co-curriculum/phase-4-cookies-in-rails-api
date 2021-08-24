# Cookies in APIs

## Learning Goals

- Configure a Rails API to use cookies
- Use the developer tools to inspect cookies

## Configuring Cookies in Rails APIs

Since cookies are such an important part of most web applications, Rails has
excellent support for cookies and sessions baked in. Unfortunately for us, when
you create a new application in API mode with `rails new appname --api`, the
code needed for working with sessions and cookies in the controller is
excluded by default.

To add session and cookie support back in, we need to update our application's
configuration in the `config/application.rb` file:

```rb
# config/application.rb
module MyApp
  class Application < Rails::Application
    config.load_defaults 6.1
    # This is set in apps generated with the --api flag, and removes session/cookie middleware
    config.api_only = true

    # Must add these lines!
    # Adding back cookies and session middleware
    config.middleware.use ActionDispatch::Cookies
    config.middleware.use ActionDispatch::Session::CookieStore

    # Use SameSite=Strict for all cookies to help protect against CSRF
    config.action_dispatch.cookies_same_site_protection = :strict
  end
end
```

This will add in the necessary [middleware][] for working with sessions and cookies
in our application.

The last line adds some additional security to our cookies by also
configuring the `SameSite` policy for our cookies as `strict`, which means
that the browser will only send these cookies in requests to websites that are
on the same domain. This is a relatively new feature, but an important one for
security! You can read more about [`SameSite` cookies here][same site cookies].

To access the `cookies` hash in our controllers, we also need to include the
`ActionController::Cookies` module in our `ApplicationController`:

```rb
# app/controllers/application_controller.rb
class ApplicationController < ActionController::API
  include ActionController::Cookies
end
```

Since all of our controllers inherit from `ApplicationController`, adding this
module here means all of our controllers will be able to work with cookies.

## Working With Sessions and Cookies

We've included some starter code for a Rails API application with this lesson so
you can see a basic example of working with sessions and cookies. The
configuration is already done, so we can work on inspecting sessions and cookies
in the controller and see how we can interact with them in our code.

To set up and run the Rails application, run:

```console
$ bundle install
$ rails s
```

Then, in the browser, make a request to `http://localhost:3000/sessions`. This
will run the code in our `SessionsController#index` method:

```rb
def index
  session[:session_hello] ||= "World"
  cookies[:cookies_hello] ||= "World"
  render json: { session: session, cookies: cookies.to_hash }
end
```

In this method, we're setting values on the `session` hash and the `cookies`
hash, and serializing them in the response so we can view their values in the
browser.

> If you haven't encountered [`||=`][ruby or equals] syntax in Ruby, it's a
> shorthand way to assign a value if the current value is `nil` or `false`. So
> if `session[:session_hello]` has not already been assigned a value, it will be
> assigned a value of "World". Otherwise, it won't get assigned a new value.

The first time a user makes a request to this controller, Rails will include the
`Set-Cookie` **response header** with our sessions and cookies values, which
will instruct the browser to store these values in memory and send them with any
future requests on this domain.

![set-cookie headers](https://curriculum-content.s3.amazonaws.com/phase-4/cookies-in-rails-api/set-cookie-headers.png)

After making the request, you should see something like this in the browser:

```json
{
  "session": {
    "session_id": "2ed452b4e28ca49ce32749fc67571ced",
    "session_hello": "World"
  },
  "cookies": {
    "cookies_hello": "World",
    "_session_id": "AT26hlXMDW5EroI89/piWHiTDRF4SQvtuvoeNZYBYNaApyLvl8a1MvhnTsLfTK57QeJCMM6YkyFqaSWguqVMWljwl+ZmELmT/wHXfFJiGL0kvadecPhyXup+p7kO66HAFVBSTOKefbkhDtQz8Ex5pHW+UBAhFfoDnDZ9/4QgST3LPyGHKf4Pgix+JwOFU9MqeFQqXZTITRW7DFi+aGDdrb1hUeIGZLuezO2QN3+TEu2xHMc=--HJwJL83oJZqcaIL1--snxu+v1esfT9YLOXUGxLYw=="
  }
}
```

From this, we can see that the session and cookies hashes can both be used to
store key-value pairs of data. The entire session hash is actually stored in
that `_session_id` cookie, in a signed and encrypted format, which makes it
impossible for users to tamper with.

You can view cookie information directly in the browser as well. In the
developer tools, find the **Application** tab, and go to the **Cookies** section
(under "Storage" in the pane on the left). There, you'll find all the cookies
for our domain (`http://localhost:3000`):

![cookies in dev tools](https://curriculum-content.s3.amazonaws.com/phase-4/cookies-in-rails-api/cookies-devtools.png)

Cookies can be edited directly in the dev tools. Try changing the value of the
`cookies_hello` key to something new. Then refresh the page in the browser to
make another request. If you try to edit the `_session_id` cookie, on the other
hand, it will have no effect thanks to Rails security features like signing and
encryption.

Finally, you can also view cookies by looking at the **request headers** (under
the Network tab, click "sessions" then "Headers"):

![cookies in headers](https://curriculum-content.s3.amazonaws.com/phase-4/cookies-in-rails-api/cookies-headers.png)

## Explore

Try adding a `byebug` at the top of the `SessionsController#index` method:

```rb
def index
  byebug
  session[:session_hello] ||= "World"
  cookies[:cookies_hello] ||= "World"
  render json: { session: session, cookies: cookies.to_hash }
end
```

Experiment in the browser by changing the cookie values and making more requests
to the server. Use the `byebug` to see how changing these values in the browser
affects what is available in the session and cookies hashes.

## Conclusion

Rails has a lot of great functionality built in to work with cookies and sessions.
When working with Rails in API mode, we need to add some additional configuration
to get them working again.

Cookies are an integral part of modern web applications; they help keep track of
**stateful** information in an inherently **stateless** protocol by
automatically passing additional data with each request using the headers. We
can use the developer tools to get a better sense of how cookies are being used
by websites by either inspecting the request/response headers under the Network
tab or looking at storage under the Application tab.

## Resources

- [Rails Middleware Stack][middleware]
- [Rails Session Security](https://guides.rubyonrails.org/security.html#sessions)
- [Rails Session Cookie Configuration](https://api.rubyonrails.org/classes/ActionDispatch/Session/CookieStore.html)
- [SameSite Cookies Explained][same site cookies]
- [Chrome DevTools: Working With Cookies](https://developer.chrome.com/docs/devtools/storage/cookies/)

[middleware]: https://guides.rubyonrails.org/rails_on_rack.html#action-dispatcher-middleware-stack
[same site cookies]: https://web.dev/samesite-cookies-explained/
[ruby or equals]: http://www.rubyinside.com/what-rubys-double-pipe-or-equals-really-does-5488.html
