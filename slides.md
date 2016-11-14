% title: The Looker API
% subtitle: Practical Knowledge Building Session
% author: Maxie Corbin

---
title: Setup
subtitle: Get Credentials

Before you can make requests to the API, you have to have API credentials on a user in that instance.

Do this by either attaching API credentials to an existing user, or by creating a new user and generating credentials for them.

API requests use the user's permissions, access filters, and content access controls, so choose sensibly.

---
title: Setup
class: img-top-center

<img height=450 src=figures/api-credentials.png />

---
title: API Secrets Are Not Meant to be Shared
subtitle: They are basically passwords. Don't share them.
class: segue dark nobackground

---
title: cURL
subtitle: What is cURL?

cURL is a command line utility for sending HTTP requests and receiving HTTP responses.

Since the API works off of receiving HTTPS requests and sending back data in response, we can use this to get data from the API.

---
title: cURL
subtitle: How to use cURL

At its simplest, using cURL can be as easy as opening up Terminal and typing:

<pre class="prettyprint" data-lang="shell">
curl www.google.com
</pre>

This will return the HTML for Google's front page, which your browser would normally render into the familiar logo and search bar.

---
title: cURL
subtitle: How to use cURL

Normally, you will want to send along some headers, and maybe a data payload, and possibly some URL parameters.

You can send headers using the -H option, data with the -d option, and URL parameters can be added directly to the URL.

Here are some examples of what cURL can look like.

<pre class="prettyprint" data-lang="shell">
curl -H "Accept-Charset: utf-8" www.google.com
curl https://www.google.com/#q=how+to+be+smart
curl -d "This is data in the form of text." https://localhost:9999
</pre>

---
title: cURL
subtitle: Trying it out

It might be useful to be able to check on-the-fly if a Looker instance is taking API requests before you start sending them.

Luckily, Looker has an endpoint for just that. Try this from your terminal:

<pre class="prettyprint" data-lang="shell">
curl -i https://learn.looker.com/alive
</pre>

The -i flag just gives you a more informative output when the response comes back. You should see something like this:

---
title: cURL
subtitle: Trying it out

<pre class="prettyprint">
HTTP/1.1 200 OK
Server: nginx/1.10.0
Date: Fri, 11 Nov 2016 09:41:45 GMT
Content-Type: text/html;charset=utf-8
Content-Length: 0
Connection: keep-alive
Set-Cookie: XSRF-TOKEN=; max-age=0; expires=Thu, 01 Jan 1970 00:00:00 -0000; secure
Set-Cookie: CSRF-TOKEN=2tdoL3wqZ7QXdEU0agvBuIYtJE9AoBHJ%2BihTArRVk0s%3D; path=/; secure
Set-Cookie: rack.session=BAh7B0kiD3Nlc3Npb25faWQGOgZFRiJFODU4MjJhZWM5OGU0YWQxODUxMGQy%0AZDE5MmIyYjNhMDVhNzc3NDUyYTQwNjI1MWM3NWJhNTViMjQ1OWM1MmE4M0ki%0AD2NzcmYudG9rZW4GOwBGSSIxMnRkb0wzd3FaN1FYZEVVMGFndkJ1SVl0SkU5%0AQW9CSEoraWhUQXJSVmswcz0GOwBG%0A--2d1f35413063b0c8ac39039ecdfb105e589dd951; path=/; expires=Sun, 11 Dec 2016 09:41:45 -0000; HttpOnly; secure
X-UA-Compatible: IE=edge,chrome=1
P3P: CP="This is not a P3P policy!"
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000; includeSubDomains
Vary: Accept-Encoding
</pre>

---
title: cURL
subtitle: Trying it out

That's a lot of information, but the important part is that first line: HTTP/1.1 200 OK

This means that the instance at this address is ready to accept API requests.

The first step is to authenticate, using our client ID and secret.

---
title: Next
subtitle: We'll use cURL to authenticate to the Looker API
class: segue dark nobackground

---
title: Authentication
subtitle: Use /login, not api/3.0/login

If you ever have logged in to the API docs in an instance, you may have noticed this.

<img height=300 src=figures/wrong-login.png />

---
title: Authentication
subtitle: Use /login, not api/3,0/login

This is a special end-point used to log in _only_ from the API docs hosted on the instance.

This is not a general purpose login endpoint. You are supposed to login from plain old /login.

This is a very common mistake.

---
title: Authentication
subtitle: What do authentication requests look like?

When you authenticate to the API, you're sending a POST HTTP request to the /login endpoint.

POST requests always have a data payload, and supplying a data payload with -d automatically makes a request a POST request.

<pre class="prettyprint">
curl -d "client_id=< id >&client_secret=< secret > https://learn.looker.com:19999/login
</pre>

---
title: Authentication
subtitle:What do authentication *responses* look like?

Once you've successfully authenticated to the API, the /login endpoint will respond with a JSON string containing your access token:

<pre class="prettyprint">
curl -i -d "client_id=5vCNBh7Xt648jxBd9Dnp&client_secret=< REDACTED >" https://learn.looker.c
HTTP/1.1 200 OK
Server: nginx/1.10.0
Date: Sat, 12 Nov 2016 00:52:41 GMT
Content-Type: application/json;charset=utf-8
Content-Length: 99
Connection: keep-alive
Vary: Accept-Encoding
X-Content-Type-Options: nosniff

{"access_token":"BGbkvjdGSkMNJphXFPTT82PHTxSPVtXWCRyfYfDy","token_type":"Bearer","expires_in":3600}
</pre>

"token_type" never changes really, and "expires_in" tells you that the token expires in 3600 seconds (60 minutes, or 1 hour).

---
title: Getting Data
subtitle: Access tokens go in the headers

Once you've got your access token (more on how to use that without copyin/pasting in a minute) you can actually get real data out of the instance now.

Let's get some data about me. My user ID in Learn is 3801, so you can put that in the URL.

---
title: Getting Data
subtitle: You'll notice this looks a lot like when we supplied our client ID and secret, but in the header, not data payload

<pre class="prettyprint">
curl -H "Authorization: token BGbkvjdGSkMNJphXFPTT82PHTxSPVtXWCRyfYfDy" https://learn.looker.com:19999/api/3.0/users/3801
</pre>

---
title: Getting Data
subtitle: This is what a typical response might look like

<pre class="prettyprint">
curl -H "Authorization: token BGbkvjdGSkMNJphXFPTT82PHTxSPVtXWCRyfYfDy" https://learn.looker.com:19999/api/3.0/users/3801
{"id":3801,"first_name":"Max","last_name":"Corbin","email":"max@looker.com","is_disabled":false,"avatar_url":"https://gravatar.lookercdn.com/avatar/5788fec97dc68ae345b1002701f20466?s=156\u0026d=blank","home_space_id":"2766","credentials_email":{"email":"max@looker.com","created_at":"2016-04-11T17:08:02.708+00:00","logged_in_at":"2016-11-11T20:39:09.954+00:00","type":"email","is_disabled":false,"password_reset_url":null,"url":"https://localhost:19999/api/3.0/users/3801/credentials_email","user_url":"https://localhost:19999/api/3.0/users/3801","can":{"show_password_reset_url":true}},"credentials_totp":null,"credentials_ldap":null,"credentials_google":null,"credentials_saml":null,"credentials_api":null,"credentials_api3":[{"id":32,"client_id":"5vCNBh7Xt648jxBd9Dnp","created_at":"2016-06-08T18:34:57.875+00:00","type":"api3","is_disabled":false,"url":"https://localhost:19999/api/3.0/users/3801/credentials_api3/32","can":{}}],"credentials_embed":[],"credentials_looker_openid":null,"looker_versions":["3.54.0","3.56.10","3.56.2","3.56.3","3.56.5","3.56.7","3.56.9","4.0.0","4.0.1","4.0.10","4.0.11","4.0.7"],"ui_state":{"dismissedDialogs":["VIEW_LOOK_INTRO_DIALOG"]},"sessions":[{"id":17809,"ip_address":"71.146.65.122","browser":"Chrome","operating_system":"OS X 10.12.0","city":"Scotts Valley","state":"CA","country":"US","credentials_type":"email","extended_count":null,"extended_at":null,"sudo_user_id":null,"created_at":"2016-10-24T04:25:31.910+00:00","expires_at":"2016-11-23T04:25:31.907+00:00","url":"https://localhost:19999/api/3.0/users/3801/sessions/17809","can":{}},{"id":18269,"ip_address":"63.249.67.206","browser":"Chrome","operating_system":"OS X 10.11.6","city":"Santa Cruz","state":"CA","country":"US","credentials_type":"email","extended_count":null,"extended_at":null,"sudo_user_id":null,"created_at":"2016-11-11T20:39:09.960+00:00","expires_at":"2016-12-11T20:39:09.960+00:00","url":"https://localhost:19999/api/3.0/users/3801/sessions/18269","can":{}}],"presumed_looker_employee":true,"verified_looker_employee":false,"access_filters":[],"display_name":"Max Corbin","role_ids":[2],"group_ids":[1,2],"url":"https://localhost:19999/api/3.0/users/3801","can":{"show":true,"index":true,"show_details":true,"index_details":true,"sudo":false}}
</pre>

---
title: Next
subtitle: Using the Ruby SDK to do cool things
class: segue dark nobackground

---
title: The Ruby SDK
subtitle: First download it

To download the Ruby SDK and get it ready for action, clone the repository and install it by following these steps:

<pre class="prettyprint">
git clone https://github.com/looker/looker-sdk-ruby.git looker-sdk
cd looker-sdk
gem install bundle
bundle install
rake install
</pre>

This will download the repository, navigate to the directory, and install the necessary libraries to start.

---
title: The Ruby SDK
subtitle: Logging in

Now we should open your favorite text editor and start writing a new Ruby script.

The first thing you need is a line telling Ruby you want to use the code in looker-sdk and tell the script what credentials you want to use.

(This is not normally recommended, and typically you would have an end user specify these credentials, or put them in a separate file. But this is just for funsies)

---
title: The Ruby SDK
subtitle: Logging in

<pre class="prettyprint">
#!/usr/bin/env ruby
require 'looker-sdk'

sdk = LookerSDK::Client.new(
  :client_id => "<your ID here>",
  :client_secret => "<your secret here>",
  :api_endpoint => "https://learn.looker.com:19999/api/3.0"
)
</pre>

This creates an object in Ruby that represents our connection to the instance. We can use functions on this object to do anything we would normally be able to do in curl without having to write HTTP requests from scratch. So, let's check if the instance is alive and login.

---
title: The Ruby SDK
subtitle: Logging in

<pre class="prettyprint">
#!/usr/bin/env ruby
require 'looker-sdk'

sdk = LookerSDK::Client.new(
  :client_id => "<your ID here>",
  :client_secret => "<your secret here>",
  :api_endpoint => "https://learn.looker.com:19999/api/3.0"
)

puts sdk.alive
</pre>

If we did everything right, we'll get a 200 status, indicating that we're good to run queries. Let's add a line to get the first name and email of user 3801 (me, Max Corbin).

---
title: The Ruby SDK
subtitle: Getting data

<pre class="prettyprint">
#!/usr/bin/env ruby
require 'looker-sdk'

sdk = LookerSDK::Client.new(
  :client_id => "<your ID here>",
  :client_secret => "<your secret here>",
  :api_endpoint => "https://learn.looker.com:19999/api/3.0"
)

puts sdk.alive # we'll leave this in here because it throws an error if it fails

puts sdk.user(3801)[:first_name]
puts sdk.user(3801)[:email]

---
title: The Ruby SDK
subtitle: Getting data

<pre class="prettyprint">
./first_script.rb
200
Max
max@looker.com
</pre>

---
title: That's it.
subtitle: This is where the API ends and just regular programming begins
class: segue dark nobackground
