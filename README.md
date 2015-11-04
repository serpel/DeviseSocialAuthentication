# DeviseSocialAuthentication
Devise authentification with Facebook and Google tested in rails 4.2.0

### Step 1 - create our new rails application
create our working space & our new app

```sh
rails new fb_auth
cd fb_auth
```

### Step 2 - Add the Required Gems to the Gemfile
edit this file `~/fb_auth/GemFile`.

```ruby
gem 'devise'
gem 'omniauth'
gem 'omniauth-facebook'
gem 'omniauth-google-oauth2'
bundle install
```

### Step 3 - Add Basic Functionality to the Application
Add some code to your application like this.

```ruby
rails g scaffold Product name:string price:integer description:text
rake db:migrate
```

Edit `~/fb_auth/config/routes.rb`, and add the line root `'products#index'`

```ruby
Rails.application.routes.draw do
  resources :products
  root 'products#index'
end
```

### Step 4 - Set Up Devise

```sh
rails generate devise:install
rails generate devise User
rails g migration AddColumnsToUsers provider uid
rake db:migrate
```

Our app now has a basic authentication system, where users can register themselves, and then log in. However, all the pages are still directly accessible. To change this, edit `~/fb_auth/app/controllers/application_controller.rb`.

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
  before_action :authenticate_user!
end
```

### Step 5 - Update the Devise Initializer

Edit `~/fb_auth/config/initializers/devise.rb` to add the client IDs and secrets at the bottom of the file, just before the end line.

```ruby
Devise.setup do |config|
  #Replace example.com with your own domain name
  config.mailer_sender = 'mailer@example.com'

  require 'devise/orm/active_record'
  config.case_insensitive_keys = [ :email ]
  config.strip_whitespace_keys = [ :email ]
  config.skip_session_storage = [:http_auth]
  config.stretches = Rails.env.test? ? 1 : 10
  config.reconfirmable = true
  config.expire_all_remember_me_on_sign_out = true
  config.password_length = 8..128
  config.reset_password_within = 6.hours
  config.sign_out_via = :delete

  #Add your ID and secret here
  #ID first, secret second
  config.omniauth :facebook, "11111dc9990be7e3bc42503d0", "111114c2722b65d29965f1a1df"
end
```

### Step 6 - Update the User Model
After editing it, your `~/fb_auth/app/models/user.rb` should look like this:

```ruby
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable,
         :omniauthable, :omniauth_providers => [:facebook]
  
  def self.from_omniauth(auth)
      where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
        user.provider = auth.provider
        user.uid = auth.uid
        user.email = auth.info.email
        user.password = Devise.friendly_token[0,20]
      end
  end
end
```

### Step 7 - Add a Controller to Handle the Callback URLs

First, edit `~/fb_auth/config/routes.rb` and update the devise_for line to specify the name of the controller that will be handling the callbacks.

```ruby
Rails.application.routes.draw do
  devise_for :users, :controllers => { :omniauth_callbacks => "callbacks" }
  resources :products
  root 'products#index'
end
```

Then, create a new file `~/rails_apps/myapp/app/controllers/callbacks_controller.rb`.

Add the following code to it:

```ruby
class CallbacksController < Devise::OmniauthCallbacksController
    def facebook
        @user = User.from_omniauth(request.env["omniauth.auth"])
        sign_in_and_redirect @user
    end
end
```

### Step 8 - Add login and sign out links whatever you want.
```
<% if !current_user %>
  <%= link_to "Sign in with Facebook", user_omniauth_authorize_path(:facebook) %>
<% else %>
  <%= link_to destroy_user_session_path, :method => :delete  do %>Sign out<%end%>
<%end%>
```

### Note: For adding others providers you must edit this files:
`~/fb_auth/GemFile`
`~/fb_auth/config/initializers/devise.rb` 
`~/fb_auth/app/models/user.rb` 
`~/fb_auth/app/controllers/callbacks_controller.rb`

If you have used more OAuth providers, you will need a separate method for each of them. The name of the method should match the name of the provider. For example, to add support for Facebook, your method will be defined using *def facebook*.

### Adding Google Authentication Support:

Edit `~/fb_auth/GemFile` and add this line:
```
gem 'omniauth-google-oauth2'
```

Run this rails command
```ruby
bundle install
```

Edit `~/fb_auth/config/initializers/devise.rb` and add this line:
```
config.omniauth :google_oauth2, "APP_ID", "APP_SECRET", { access_type: "offline", approval_prompt: "" }
```

Add new provider editing `~/fb_auth/app/models/user.rb`.
```
:omniauth_providers => [:facebook, google_oauth2]
```

After editing it, your `~/fb_auth/app/controllers/callbacks_controller.rb` should looks like:

```ruby
class CallbacksController < Devise::OmniauthCallbacksController
    def facebook
        @user = User.from_omniauth(request.env["omniauth.auth"])
        sign_in_and_redirect @user
    end
    
    def google_oauth2
        @user = User.from_omniauth(request.env["omniauth.auth"])
        sign_in_and_redirect @user
    end
end
```

Now you can put this link whatever you want:
```
<%= link_to "Sign in with Google", user_omniauth_authorize_path(:google_oauth2) %>
```

### Support

Sergio Peralta serpel.js@gmail.com

Please file issues [click here] at Github. 

Copyright (c) 2015 Sergio Peralta. This software is licensed under the MIT License.

Good luck!

[click here]:https://github.com/serpel/DeviseSocialAuthentication/issues

### Fork it

- Create your feature branch (git checkout -b my-new-feature)
- Commit your changes (git commit -am 'Add some feature')
- Push to the branch (git push origin my-new-feature)
- Create a new Pull Request
