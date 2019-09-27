---
layout: post
title:      "Rails with JS Front End Portfolio Project "
date:       2019-09-27 19:35:36 +0000
permalink:  rails_with_js_front_end_portfolio_project
---


With adding some JavaScript functionality to the front end of my Rails project, I was able to use AJAX get and post requests to render an index and show page as well as submit a form all without redirecting! This not only makes the web app behave faster since we don't have to wait for a new request, but gives a more dynamic, interactive experience for the user. One extremely useful thing about AJAX is how visual the language is. If you can see the property in the browser JS viewier, you can access it with an AJAX query. For example, to use the post request, I needed to acces the route that the Rails form helper was sending the form to. In the Elements tab of the JS viewer, that route was stored in the 'action' part of the form. 
```
$(document).ready( () => {
  $('form#new_movie').on('submit',function (e){
    e.preventDefault();
    let postUrl = $(this).attr('action');
    let values = $(this).serialize();
    $.post(postUrl, values)
```
So we can see that inside the click event of the form with an ID of 'new_movie,' the *this* keyword refers to the actual Rails generated form object. By wrapping *this* in an AJAX query, we can use the attr method to return the value of 'action,' which is the route the form is submitted to. Then we send the route and the serialized form to the AJAX post request. We now have access to that data and can clear the DOM and paint it anyway we wish with the data we have. AJAX makes a complex Rails post and redirect simple through serialization and the ability to paint the DOM in a format of our choosing. 
