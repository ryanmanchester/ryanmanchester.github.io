---
layout: post
title:      "Nested Forms with Nested Routes Part 1"
date:       2019-05-09 13:40:38 +0000
permalink:  nested_forms_with_nested_routes_part_1
---

This is a two part installment covering how I used a nested form to CRUD with a nested resource. These two posts will go in depth, step by step on how I was able to get this complex relationship working for my Rails portfolio project, Film Journal. 
The concept of Film Journal is simple: it is a personalized web app used to keep track of the movies the user watches. In addition to simply adding the movie to the account, the user is also able to rate and review the movies watched. 
Sounds easy and straightforward, right? Well...it is and it isn't. See, what we as users take for granted is the complex relationship required to execute a simple action, such as rating and reviewing a movie. On top of that, Rails has its own conventions to create some magic with less code, which is awesome, but takes a little getting used to. 

Enough talk, lets look at some code. 

After creating your new Rails project and migrating the database, now we can start to customize the routes. I'm going to only be focusing on nested routes, hence the title of this post. 
```
Rails.application.routes.draw do
  get 'static/welcome'
  get 'signup', to: 'users#new'
  get 'signin', to: 'sessions#new'
  post 'signin', to: 'sessions#create'
  get '/auth/facebook/callback', to: 'sessions#create'
  delete 'signout', to: 'sessions#destroy'
	  resources :users do
    resources :movies
  end
  root 'sessions#new'
end
```

So we see the typical sign/sign out protocol, then we get to the nested route:
```
  resources :users do
    resources :movies
  end
```

I've chosen to use resources because every CRUD action in this app takes place as /users/:user_id/movies or some other route depending on the action. One difference you've probably already noticed is by nesting the routes, the route changed from /users/:id to users/:user_id. This is important to note because it informs how we will build our form to add a movie to the user's account. 

**Using form-for**

The rails form_for builder is an example of the power of Rails. Here, Rails is building a form AND posting (or patching) it to the appropriate action. It works by accepting an argument of a newly instaniated object, which it attaches the builder to, and then uses helpers to generate the HTML for the form. Something like this:
```
*film-journal/app/controllers/movies_controller.rb*
def new
  @movie = Movie.new
	end
```

Which creates a new instance of a movie to pass to form_for:
```
*film-journal/app/views/movies/_form.html.erb*
<%= form_for @movie do |f| <-- *f is the builder* %>
         <%=  code for the form %>
				 <% f.submit %>
			<% end%>
```
Normally, this would suffice. BUT we are using nested routes! Rails knows where to post (or patch) forms from the instance variable. In this case, Rails wants to post to /movies, and again...nested forms. So, Rails needs a little more information about where it's posting. The convention is that form_for can accept a nested argument (an array), and those arguments read a lot like the routes already drawn. 

```
*film-journal/app/views/movies/_form.html.erb*
  <%= form_for ([@user, @movie]) do |f| %>
	         <%= code for form %>
					 <%= f.submit %>
				<% end %>
```
After finding the user that's logged in and storing that instance in the @user instance variable, we can pass that in with the new movie instance as an array to form_for. Once we submit, the form will post to /users/:user_id/movies. That's it! Seems simple enough, but it took me a while of reading many docs on form_for and debugging to get there. There is more than one way to achieve this post behavior, but I find that this one works the best and is the cleanest for Film Journal. 

In Part 2, I'll go over submitting a rating and review through this same form using nesting, and creating a custom writer method on the Movie model to associate the review and rating with the user and movie. HInt: it involves a join table :)

      
