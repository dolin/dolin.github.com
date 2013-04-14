---
layout: post
title: "Understanding Nested Attributes"
date: 2013-03-17 15:06
comments: true
categories: rails
---

Let's talk about rails' accepts_nested_attributes_for method.

<strong>What is it?</strong>

According to Rails API, the method allows you to 'save attributes on associated records through the parent.''

<strong>What does that mean and why should you use it?</strong>

Using an example from class, let's pretend you're creating an app built around horse races. In your app, you have a race model and a horse model. A race has a name and a horse has a name, number, and starting position. 

You also have a view to create a new race. When you create a new race you want to populate the race with horses. For simplicity sake, we'll only populate the race with one horse for now. 

You soon realize that you can't figure out a way to create a new horse within a race using the form_for @race helper.

<strong>Enter accepts_nested_attributes_for.</strong>

Essentially, the method allows you to create a new instance of a horse with it's respective attributes at the exact same time of creating the 'parent' race. 

Enter the following into your race model (I am assuming you have already setup the respective database associations):

<img src='images/horses1.png'>

Enter the following into your new race view:

``` erb

<%= form_for @race do |f| %>

  
  <p>Name <%= f.text_field :name %></p>
  <p>Description <%= f.text_field :description %></p>


   <%= f.fields_for :horses do |h| %>
    <p>Horse Name <%= h.text_field :name %></p>
    <p>Horse Number <%= h.number_field :number %></p>
    <p>Horse Position <%= h.number_field :position %></p>
  
  <% end %>

  <%= f.submit %>

<% end %>

```

Let's inspect one of the attribute fields for the horse.

<img src='images/horses2.png'>

If we click on 'Create Race' and raise params before the race is saved, we will get the following:

<img src='images/horses3.png'>


As you can see, the paramaters being passed through to the controller for a horse come in the form of race[horses_attributes][0][{attribute}]. horses_attributes is a hash nested within params[:race]

Make sure your create action in your race controller looks something like this:

``` ruby

  def create
      @race = Race.new(params[:race])

      if @race.save
        redirect_to @race, notice: 'Race was successfully created.'
      else
        render action: "new"
      end 
  end

``` 

Just like that you'll be able to create horses within your races thanks to mass-assignment. Now let's look at what the method is actually doing. 

####1st level of abstraction down:

Instead of using 'accepts_nested_attributes_for', we can substitute a writer method in your race model.

``` ruby

class Race < ActiveRecord::Base
  attr_accessible :description, :name
  
  has_many :horse_races
  has_many :horses, :through => :horse_races

  def horses_attributes=(hash)
    hash.each do |horse_index, horse_attributes|
      horse = self.horses.new
      horse_attributes.each do |horse_attribute, attribute_value|
        horse.send("#{horse_attribute}=", attribute_value)
      end
    end
  end

end

```

During mass-assignment from your race controller, the horses_attributes= method will be called and a new horse will be created from this iteration.

####2nd level of abstraction down: 

If you're like me when you first tried to figure out what's going on beneath the hood, you may have written the code in your race controller to handle the task. I initially iterated through the params for both race and horse attributes, assigning the attributes one by one.


``` ruby

class RacesController < ApplicationController

  def create
    @race = Race.new()
    @race.name = params[:race][:name]
    @race.description = params[:race][:description]


    params[:race][:horses_attributes].each do |horse_index, horse_data|
      @horse = Horse.new
      @horse.name = horse_data[:name]
      @horse.number = horse_data[:number]
      @horse.position = horse_data[:position]

      @race.horses << @horse
    end
 
    if @race.save
      redirect_to @race, notice: 'Race was successfully created.'
    else
      render action: "new"
    end
  end

end

```

This way works to get the job done, but it's best to take the code out of your controller and create a writer method as shown above in your race model. Or better yet, just use the 'accepts_nested_attributes_for' method and don't worry about creating the writer method at all. 

I hope this helps for anyone looking to better understand nested attributes. Feel free to comment below if there are any questions!


