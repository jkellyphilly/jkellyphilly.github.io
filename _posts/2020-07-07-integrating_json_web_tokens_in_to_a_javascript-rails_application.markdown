---
layout: post
title:      "Integrating JSON Web Tokens in to a JavaScript-Rails application"
date:       2020-07-07 19:06:42 +0000
permalink:  integrating_json_web_tokens_in_to_a_javascript-rails_application
---


### To check out an example of a project where I've implemented JWT, check out my project "The Allyship Corner" (Github repo [here](https://github.com/jkellyphilly/allyship-corner)). 

First of all: what *is* a JSON Web Token? 

I'll give you the high level: it's essentially a "secrete signature" that gets included in requests to your Rails API that tell the server, "Hey, this request is coming from an authorized party. Proceed with getting them that information, please!!" When I was implementing JWT for my JS-Rails project, I actually thought of it as a secret handshake. 

My front end JS code would interpret what my end user wanted to do: as an example, let's say my user wanted to create a new social justice event (for background on The Allyship Corner application, watch [this video demo](https://www.youtube.com/watch?v=Gcph4zjZFCQ&t=5s)). 

This means my front-end JS walks over to the Rails server (via a fetch call), gives Railsy a firm handshake, and says: "Hey there, Railsy, how's it going? Can you create a new event with this information for me, please? Let me know what the result is." However, Railsy won't do a single thing, because in order for her to go inside the Base of Data and fetch things, she told JS to do a fist pound instead of a handshake to communicate that this request was coming from a valid user and not someone trying to hack into the Data Base. 

"I don't think so, JavaScripter. Here's an unauthorized status response instead."

## High level flow

Basically, this is what happens: when an application boots up, usually there is a place for users to log in or sign up. When the user logs in successfully or gives valid data to create a new profile, the back-end encodes a token and sends it along with the successful response of having created a new user in the database/authenticating a log-in. 

Then, the front-end code takes that token and stores it somewhere secure. On every subsequent request to the back-end for the rest of the session, that token will be sent along with each request, and the back-end will decode the token and thus ensure the request is valid. When a user signs up or logs in for a new session, the process begins again.

[Read more about the makeup of JWTs here](https://jwt.io/introduction/).

## Implementation

For authentication of users *within* the database and storing secure passwords, I used a combination of [ActiveModel's has_secure_password](https://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html) and the [BCrypt gem](https://github.com/codahale/bcrypt-ruby); however, feel free to use a different method such as [Devise](https://github.com/heartcombo/devise) for your own implementation. 

Enable [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) in your app by uncommenting the required code in `config/initializers/cors.rb`, and change the origins from `example.com` to `*`. **Make sure to change these settings before deploying your app to the internet!!**

Next, let's install the JWT gem to our backend repo. Run `bundle add jwt` and then `bundle install`. 

### Now we're ready for sending a JWT with a successful response

The JWT#encode method takes two arguments: the "payload" (information to encode into the token), and a secret key that is used to hash the header and payload of the JWT together. Once again, [read more about the makeup of JWTs here](https://jwt.io/introduction/). 

**Note: it is very important that you store your secret key securely within your application. Personally, I used a gem called [Figaro](https://github.com/laserlemon/figaro#getting-started), but there are many other good options as well. The secret key should never be displayed and should never be pushed up to a remote repository!**

Look below at how we will set up our #encode_token method. 

SCREENSHOT HERE

Next, let's implement that method in our `UsersController#create` (for signing up a new user) and `AuthController#create` (for logging in an existing user). 

SCREENSHOT HERE OF USERSCONTROLLER
SCREENSHOT HERE OF AUTHCONTROLLER

### Store the JWT on the front-end for use during the session

There are a few options about where to store JWTs - read [this article](https://stormpath.com/blog/where-to-store-your-jwts-cookies-vs-html5-web-storage) for a good understanding of your options. 

I personally went with using `sessionStorage` to store the token. There are definitely concerns for cross-site scripting (XSS) attacks stealing information, but one way to minimize this risk is by ensuring that your application is running on HTTPS. Read [this article](https://stackoverflow.com/questions/35291573/csrf-protection-with-json-web-tokens/35347022#35347022) for a more nuanced conversation on the risks of XSS for local storage vs. cookie storage. 

Using my approach, the bottom line is that you can pull out the `jwt` key from the response and store that in `sessionStorage.accessToken`. Now, when we have any future request to the server, we must include an `Authorization` key in the header that looks like this:

SCREENSHOT OF EXAMPLE REQUEST HERE

#### Let's recap: we now have the ability to, upon successful user creation/authentication, send a token to our front-end. We have stored that token in our front-end, and we are sending it along with every request to our server. Now, let's set up a decoding strategy to authenticate requests in our back-end. 

### Set up the back-end decoding

The approach I took involved creating an `#authorized` method that was called *before* every call to the database. It is important to note that this `before_action` should be *skipped* for the actions relating to logging in or signing up. 

See the screenshot below and work your way up. The status of the response to the client will be **unauthorized** unless someone is `logged_in`; for someone to be logged in, there must be a `decoded_token` present. If there is a `decoded_token`, then find the current user given the payload.

SCREENSHOT OF CURRENT USER SECTION

Now, how do we go about getting the `decoded_token`? First, we need to parse the response and get the token out of it. Next, we use the JWT#decode method, passing in the token, our application secret, and an optional hashing algorithm to come up with our decoded token! 

OTHER SCREENSHOT

### A note on error handling

The `rescue JWT::DecodeError` portion of our code in the screenshot above is designed to save our application from crashing if the token isn't decoded properly or if the token is invalid. If we didn't have the rescue statement there, our server would crash as a result of this error. 

Instead, we can catch this error and return `nil` for the `#decoded_token` method, which keeps our server running. This will render an unauthorized response in `#authorize`, since there will be no `current_user` and thus `logged_in?` will equate to false.


Still confused? Check out [this walkthrough from Flatiron](https://learn.co/lessons/jwt-auth-rails) as well.

Thanks for reading!!
