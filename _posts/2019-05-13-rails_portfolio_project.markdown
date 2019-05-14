---
layout: post
title:      "Rails Portfolio Project"
date:       2019-05-14 00:36:08 +0000
permalink:  rails_portfolio_project
---


For my Rails portfolio project, I created Film Journal, a personalized list of films a user has seen. Users can sign up through Facebook thanks to OmniAuth or create a new account on the sign in page. Once signed in, the user may add a movie via a Rails form with fields for title, stars, director, year released, synopsis, and an image html as well as the users own review and rating(1-5).  For the views, I chose to learn how to incorporate Bootstrap HTML and CSS themes into the layout. 

Of the many challenges I faced creating the app, the most valuable was submitting the review and rating via the join table and associating them to a particualr instance of a movie through a custom writer on the Movie model (I wrote about the whole process in a two part blog [here](http://www.ryanmanchester.info/nested_forms_with_nested_routes_part_1) and [here](http://www.ryanmanchester.info/nested_forms_with_nested_routes_part_2)). The #user_movies_attributes= method took me quite some time to iron out the flow from the nested form, to the movie model, and to creating a new instance of a movie with the rating and review already associated, but once it was working, I was able to implement a unique destroy action. Rather than simply deleting a movie altogether, as that would delete it from any other users account it was associated with, I deleted the association between the movie instance and the join table. By doing this, the movie remains in the database, but is removed from the users account. 

Having now completed the app, I feel much more comfortable and confident with Ruby on Rails and with reading docs and other resources such as StackOverflow. 

I'm looking forward to adding more features witht the Rails with JavaScript project. 
