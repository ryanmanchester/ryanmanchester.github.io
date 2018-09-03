---
layout: post
title:      "Rack-Flash"
date:       2018-09-03 01:00:57 +0000
permalink:  rack-flash
---


I recently completed my Sinatra portfolio project and taught myself how to use the rack-flash3 gem to give a clear user experience. 

By wrapping the embedded Ruby code in the layout.erb file before the yield, we are able to create custom flash messages in the controllers. 
```
   <% if flash.has?(:message) %>
  <%= flash[:message] %>
<% end %>
    <%= yield %>
```
In the controller, we can display a customized error message:
```

  get '/records/:id' do
    if logged_in?
      @user = User.find_by(id: session[:user_id])
      @record = Record.find_by(id: params["id"])

      erb :'/records/show'
    else
      flash[:message] = "Please log in or sign up"
      redirect '/records'
    end
  end
```
This is requiring a user be logged in before displaying that page. 

Or we can use the messages to confirm actions have completed successfully:
```
  patch '/records/:id' do
    @user = User.find_by(id: session[:user_id])
    @record = Record.find_by(id: params["id"])
    @record.update(name: params["name"], artist: params["artist"])

    flash[:message] = "Record sucessfully updated"
    redirect "/users/#{@user.id}"
  end
```
Here, we have a message that confirms the object has been updated via a patch method. This is very helpful in driving the experience at every turn. Some set up is required after installing the 'rake-flash3' gem in the gem file:
```
require './config/environment'
require 'rack-flash'

class ApplicationController < Sinatra::Base
  use Rack::Flash

  configure do
    set :public_folder, 'public'
    set :views, 'app/views'
    enable :sessions
    set :session_secret, "secret"

  end
```
It has to be required in the controller and mounted in the body of the controller class. I've required and mounted all in ApplicationController because the other controllers I create will simply inherit from here. 

