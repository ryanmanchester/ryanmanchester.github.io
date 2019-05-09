---
layout: post
title:      "Nested Forms with Nested Routes Part 2"
date:       2019-05-09 19:17:14 +0000
permalink:  nested_forms_with_nested_routes_part_2
---


Now that we have the form submitting via a post method to the correct route (/users/:user_id/movies), let's take a look at how Rails can associate an instance of @user and @movie in a join table. We know we have a users_table and a movies_table with the specific attributes we've assigned, and we want to create an association between the two using a join table of user_movies. Here we will have the foreign keys of :user_id and :movie_id. In addition, we have the :review and :rating attributes, so that a user may submit a review and a rating(1-5) via the join table. 

Starting with associations, we have:
```
*film-journal/app/models/movie.rb*
  has_many :user_movies
  has_many :users, through: :user_movies
```
A Movie has many :user_movies and many :users, through :user_movies. The has_many_through macro allows an instance of movie to directly access its collection of users. 

For user:
```
*film-journal/app/models/user.rb*
has_many :user_movies
  has_many :movies, through: :user_movies
```
A similar syntax. And finally, user_movies:
```
*film-journal/app/models/user_movie.rb*
belongs_to :user
  belongs_to :movie
```
Now that our associations are complete, we know we will need to use a nested form to create an instance of :user_movie (a new review and rating associated with a new or existing movie) and create a movie at the same time. Taking the form from Part 1 of this post, we can add a fields_for helper to nest the user_movie attributes inside the movie form:
```
*film-journal/app/views/_form.html.erb*
<%= form_for ([@user, @movie]) do |f| %>
     <%= code for movie attributes %>
		 <%= f.fields_for :user_movies do |user_movie| %>
                    <%= user_movie.hidden_field(:user_id, {:value => @user.id}) %>
                    <%= user_movie.label :review, "Your Review" %><br>
                    <%= user_movie.text_area :review %><br>
                    <%= user_movie.label :rating, "Your Rating (1-5)" %>
                    <%= user_movie.select :rating, ["1", "2", "3", "4", "5"]  %><br>
                  <% end %>
```
**Special Note:** The hidden_field value of @user.id is a Rails convention to associate the :user_id from the join table with the current user.

Note that we've attached the original form_builder (f) to the fields_for helper and passed in the :user_movies object. This is essentially accessing the user_movies collection through the movie form builder. We have access to the user_movie attributes in the body of the fields_for helper from the new action in the movies_controller:
```
*film-journal/app/controllers/movies_controller.rb*
def new
  @movie = Movie.new
	@movie.user_movies.build
end
```
This view is all well and good, but it won't actually do anything with the user_movies object until we implement the Rails macro accepts_nested_attributes_for on the Movie model:
```
*film-journal/app/models/movie.rb*
accepts_nested_attributes_for :user_movies
```
This macro creates a new key in the params[:movie] hash of user_movies_attributes=> {"0" =>{user_id: , review: , rating: }}.
You maybe asking yourself, "So what? We get this new stuff in the raw, scary params. We can't even use it!" True...for now. We just have to permit them in the strong params (movie_params) we created in Part 1. 
```
*film-journal/app/controllers/movies_controller.rb*
  private
  def movie_params
    params.require(:movie).permit(:title, :release_year, :director_name, :starring, :synopsis, :image, user_movies_attributes: [:review, :rating, :user_id])
  end
```
All we have to do is pass in the user_movies_attributes we with to permit and let Rails take care of the rest! But now we have to actually use these new parameters. 

Since we are using the Rails macro, accepts_nested_attributes_for, we can use a Rails convention of a custom writer method on the Movie model to create the proper associations: 
```
*film-journal/app/models/movie.rb*
def user_movies_attributes=(movie_attributes)
    movie_attributes.values.each do |movie_attribute|
      if user_movie = UserMovie.find_by(movie_id: self.id)
        user_movie.update(movie_attribute)
      else
        user_movie = UserMovie.create(movie_attribute)
        self.user_movies << user_movie
      end
    end
  end
```
The user_movies_attributes= method comes with the macro and is customizable. Here, we pass it an argument of movie_attributes(the hash of {"0" => {user_id:, rating:, review: }). Calling .values on the hash gives us access to :user_id, :rating, and :review and assigns those values to movie_attribute, which we can customize how the create action will handle the information. 

**Special Note:** Once the user hits the submit button on the form, the params are passed to the create action, and through the argument of movie_params, we are associating user_movie with :user and :movie in the Movie model. That is, the create action is still in progress, and therefore nothing saved just yet, i.e. the new movie's id (self.id) will be nil. That's ok because through @movie.user_movies.build, we have set up a temporary association that will save after the create action is complete. 

So this custom writer isn't too complicated (yes, it did take days of debugging to get here, though), simply finding an existing movie in the database and updating the rating and review through mass assingment, or creating a new instance if self.id is nil. Now lets check out the create action:
```
*film-journal/app/controllers/movies_controller.rb*
  def create
    if movie = Movie.find_by(title: movie_params[:title])
      movie.update(movie_params)
      redirect_to user_movies_path
    else
      movie = Movie.create(movie_params)
      redirect_to user_movies_path
     end
  end
```

Again, seems basic enough. Finding an exising movie, updating it with those new params, or simply creating a new one. One tricky thing here to note is some Rails magic happening. Notice the create action isn't actually associating anything. That is because the temporary association between a user, a movie, and the user_movie that was accessed through movie_params. Remember that hidden_field :user_id? That carried an association to the Movie model, where the association is completed. So further association in the controller is redundant and will create multiple instances of UserMovie (thus will have a messy view with repeating data, ask me how I know :) ). After understanding this Rails abstraction, my controller slimmed down quite a bit and the associations from the nested form are working! 
