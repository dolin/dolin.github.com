---
layout: post
title: "Understanding Rails' Routes"
date: 2013-03-04 22:03
comments: true
categories: rails, routes
---

We began our foray into Rails last week. As expected, a ton of questions immediately came to mind once diving into a few tutorials. This post will address the routes that Rails automatically builds upon generation of resources and scaffolds.

As a Ruby framework, Rails essentially creates your application's skeleton so that the programmer can get right to building the muscle of the app (not sure if that analogy makes sense, but sounded good at the time).

In creating the skeleton, Rails abstracts out many parts of your application into files that it has already created. For a Ruby newb, this is both incredible and confusing. It's an awesome feeling to be able to startup the rails server, edit the controller, create a view or two with some forms, and automagically see your code in action. But, the hidden logic which lies within a new Rails' application begs the question, "How does all of this actually work?"

While there are a ton of books to learn about the ins and outs of Rails, let's take a look at one small, but extremely important piece: <strong>routes</strong>. 

Let's pretend you have created a new rails app for RSVPing to an event. To kick things off, you run the following in terminal to generate a User resource:

```
$ rails generate scaffold User

```

Along with creating a controller, model, migration, and other files, Rails also creates routes for you. Take a look within your config directory and you'll see a 'routes.rb' file. Within the first few lines, you should see something like this: 

``` ruby

RsvpApp::Application.routes.draw do
  
  resources :users

```

This one line of code blew my mind. All of a sudden (if you created the respective views and controller actions), you could navigate to paths such as http://localhost:4000/users, http://localhost:4000/user/new, or even fill out a form and have your data be stored to a database. 

Let's take a closer look at what happens. Run the following within your app's directory in terminal: 

```
$ rake routes

```
You should get output similar to this:

```
        users GET    /users(.:format)          users#index
              POST   /users(.:format)          users#create
     new_user GET    /users/new(.:format)      users#new
    edit_user GET    /users/:id/edit(.:format) users#edit
         user GET    /users/:id(.:format)      users#show
              PUT    /users/:id(.:format)      users#update
              DELETE /users/:id(.:format)      users#destroy
         rsvp GET    /rsvp(.:format)           users#new
         root        /                         users#home

```

Rails has created all of these routes for you with just that one line of code. 

Now, do the following:

<strong>1. Comment out 'resources :users' in routes.rb.</strong><br></br>
<strong>2. Insert this code into routes.rb:</strong>

``` ruby

  get '/users' => 'users#index'
  post '/users' => 'users#create' 
  get '/users/new' => 'users#new', :as => :new_user
  get '/users/:id/edit' => 'users#edit', :as => :edit_user
  get '/users/:id' => 'users#show', :as => :user
  
  put '/users/:id' => 'users#update'
  delete '/users/:id' => 'users#destroy'

```

<strong>3. Run 'rake routes' again within terminal. Your output should be the same as before.</strong>

<strong>4. Think about what Rails just did. Anytime you generate a new scaffold/resource, the above routes are automatically created so you don't have to go through the trouble of writing them yourself. Thanks Rails!</strong>

If anyone has other helpful tidbits on rails routes, please link to them in the comments section below. 










