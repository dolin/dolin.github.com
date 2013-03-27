---
layout: post
title: "Making a Simple Rake Task"
date: 2013-03-26 19:29
comments: true
categories: rails, rake
---

Last week, we began our group projects at the school. From here on out we'll be building everyday, allday. My group is working on Starboard, an online coding activity tracker. At its core, you earn achievements for different activities such as writing blog posts, commiting to open source projects on Github, and earning badges on Codeschool/Treehouse. To picture the concept, think of the charts when you were a kid at school and your teachers gave you those gold star stickers for doing good things. You were able to not only see your stars, but also how you compared to your fellow students. Similiarly, future Flatiron students will be able to track their coding activities and see how they stack up to the rest of the class. 

<strong>Why is a rake task needed?</strong>

The main data engine driving Starboard is the wrapper we've built to extract information from our sources (currently Github, Treehouse, Codeschool, your blog). Stay tuned for another blog post on how we did this. When you first sign-up, you'll be prompted to enter in your usernames/urls, so that we can go out and populate your profile with current achievements. We'll be vastly improving the interface, but here's our V1 for the profile page to give you a better idea:

<img src= /images/starboard-profile.png>

After we populate your profile, we'll also have to check for new achievements at scheduled intervals. To do this, we first need to implement a rake task. This rake task will be called by a cron job eventually, but for now we can call it manually to update all users' achievements. 

Here's the code for our rake task, found at app/lib/tasks/achievements.rake:

``` ruby

namespace :achievements do
  desc "Check for new user achievements"
  task :update => :environment do
    User.all.each do |user|
      user.save
    end
  end
end

```

'namespace' and 'task' let us define the name of our task. In this case, we would run <strong>rake achievements:update</strong> in our console to run the task. The meat of this task is querying our database for all of our users and then calling rails' save method for each user. To update each user's data, we have a couple callbacks in the User model that are run before the user is actually saved. 

``` ruby

class User < ActiveRecord::Base
  before_save :check_blog,
              :get_external_data

...

end

```

For a quick example, here are the number of stars I had before writing this post (we give one star for writing one blog post).

<img src= /images/blogstars.png>

The rake task.

<img src= /images/raketask.png>

And finally, my updated stars.

<img src= /images/blogstars2.png>

Just like that we can update all of our users' data with one command. Pretty cool. 







