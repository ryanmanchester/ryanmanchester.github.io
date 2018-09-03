---
layout: post
title:      "Portfolio Project: Sinatra"
date:       2018-09-03 00:44:22 +0000
permalink:  portfolio_project_sinatra
---


As I have been making my way through the Sinatra curriculum, I started to see its strength (and conveniece!) in creating instances of a class from user generated information. User genrated information being the parameters the user inputs into a form in their browser and having Sinatra create and save that information into a database using ActiveRecord. It was such a relief to learn that ActiveRecord comes with so many helpful methods that allow the developer to write code that creates, reads, updates, and destroys (CRUD). 

For my portfolio project, I chose to keep with the music motif of my last project, and develop a web app that organizes a persons vinyl record music collection. 

To begin using this applicatoin, the user must first sign up with a username of his or her choice, an email, and a password. I used the input type of password to hide the actual password in the form as well as the has_secure_password method from the Bcrypt gem. 
```

class User < ActiveRecord::Base
  has_secure_password
  has_many :records


end
```

One of the many amazing things about ActiveRecord in Sinatra is its associations. The developer only has to create a has_many and belongs_to relationship in each model class as well as having a foreing key id in the belongs_to database table. 
```
class Record < ActiveRecord::Base
  belongs_to :user
	end
```

Once the users profile has been created, he or she is able to start adding records to their collection. 
Through the form in the new.erb file in the records folder, the user is able to enter the information and have it persisted to the database and either retrieve, edit, or delete at any time. 

BUT...in order to do any of that, the user needs to be logged into their account. How awful would it be if someone without an account could not only enter unsubstantiated information, but also edit or delete other users' records?!? Well, thanks to my helper methods in the ApplicationController, we can make sure everyone has an account AND is logged in before interacting with the application. 

```
  helpers do
    def current_user
      @user = User.find_by(id: session[:user_id])
    end

    def logged_in?
      !!current_user
    end

    def logout
      session.clear
    end
  end

```

And since the RecordController and the UserController both inherit from ApplicationController, our logged_in? helper method is available in both worlds. 

Here is a small example from the UserController:
```
 get '/users/:id' do
    if logged_in?
      @user = User.find_by(id: session[:user_id])

      erb :'users/show'
  else
    redirect '/records'
  end
end
```
We only want a user who is logged in to access the profile, so once logged_in? returns true, ActiveRecord finds that user by matching their primary key id to the value of the key user_id in the session hash. 

ActiveRecord is great for implemetning the CRUD experience, but what I find to be the most helpful in driving the overall user experience are the flash messages available from the rack-flash3 gem. These messages will display in the browser using embedded Ruby, and are customizable in the controllers. From the RecordController:
```
  get '/records/:id/edit' do
    if logged_in?
      @record = Record.find_by(id: params["id"])
      erb :'/records/edit'
    else
      flash[:message] = "You must be logged in to edit your record collection"
      redirect '/records'
    end
  end
```
This flash messages is flagging an error (the user is trying to edit without being logged in), so the message that displays in the browser lets he or she know they must be logged in. Flash messages can also tell the user an actoin was successful: 
```
  delete '/records/:id/delete' do
    @user = User.find_by(id: session[:user_id])
    @record = Record.find_by(id: params["id"])
    @record.destroy

    flash[:message] = "Record sucessfully deleted"
    redirect "/users/#{@user.id}"
  end
```
Here the message is confirming that the record was deleted from the users collection. Learning to use rack-flash coupled with input validation (making sure the correct fields have information and flagging errors with messages if not) brought my application from a simple, yet vaugely straight-forward experience to a simple and clear experience. At each step of CRUD, the flash messages in the browser are flagging errors and pointing the user to the correct action, and providing piece of mind by confirming the action worked. 

