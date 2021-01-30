---
layout: post
title:      "StoreFront: a Rails API and React Frontend"
date:       2021-01-30 19:38:50 +0000
permalink:  storefront_a_rails_api_and_react_frontend
---

StoreFront is my final portfolio project featuring a Ruby on Rails API and React frontend. I'm also using Redux for state management. I've also included a healthy mix of React-Bootstrap and Styled Components for styling. 

The story behind StoreFront is creating a space for fashion designers, consignment shop owners, or just people with a good eye for vintage fashion to sell their clothing or accessories. Most of the action takes place in the Seller Portal where potential sellers can sign up and start selling, or returning sellers can log in. Once logged in, the seller can add to their inventory, update their items, or delete them altogether. 

On the user side, users can add the listed items to their cart, remove an item from the cart, remove all items from the cart, and place an order. Since each item is unique, once an item is added to a cart, the Add to Cart button is disabled. This prevents duplicate items appearing in the cart. 

In keeping with the Single Page Application convention, StoreFront does not redirect when navigating to different pages. It also does not refresh when perfoming the actions listed above. But what if something happens that causes a manual refresh? I've added a cart id to the Rails sessions hash in order to persist it even if the application is accidetally refreshed. This maintains the current cart the user has been building. If a user decides to clear their cart or complete an order, the cart id is removed from the sessions hash and an empty cart page is rendered. 

For now, the users can make purchases as guests and do not need to create accounts. I did this in order to roll out a MVP version of StoreFront. In the future, I'd like to add a user account to display past orders and a mock payment system. As it stands, once the user places their order, the items are deleted from the database and cleared from the cart. Once that happens, a Thank You For Your Order page is rendered. This mimics the functionality of ordering for my project app. 
