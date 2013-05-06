---
layout: post
title: "Building API Wrappers For Your Application"
date: 2013-04-24 18:02
comments: true
categories: APIs rails ruby wrappers
---

First, the boiler plate for <a href="http://strboard.com">Starboard</a>, one of my Flatiron School projects:

<h4>What is it?</h4>

A modern-take on an old fashioned gold star board.

<h4>Why does it matter?</h4>

The Flatiron School is hard. Throughout the semester students are learning new material, working on github projects, writing blog posts, completing online coding tutorials, and much more. Without having much time to realize how much has been accomplished, Starboard becomes the hub to track all of their achievements. From completing Codeschool courses, to committing to an open source project, to giving someone else a star for help with a bug fix, Starboard is an app to collectively celebrate learning to code.

One of my favorite problems to solve when building the application was how to normalize data (creating user achievements) for actions taken on other websites such as completing a Codeschool course or making a commit to an opensource project. The complete gist is below.

Each service we used would return a uniquely formatted set of data. Treehouse and Codeschool also have 0 documentation for their APIs. Figuring out how to manipulate these different data sets was the engine driving our application. 
Here are the steps I suggest you take when building API wrappers:

<h4>Explore the API in your browser</h4>

Without writing a line of code, read the documentation (if there is any) and ask for json objects back. For Codeschool, all that was required was a json get request for a user profile's information: http://www.codeschool.com/users/dolinsky.json

<img src="/images/codeschool.png">

<h4>Play with the data in console</h4>

Using the json and open-uri gems, test manipulating the data in console. How is the json structured? What keys do you need to chain together in order to get to the values you are looking for? In our case, we were looking for the 'title' of the 'completed' 'courses'. 

<h4>Decide on your output structure</h4>

Once you have a solid understanding of what you can get back from each API, begin to think through how you can normalize the data to use in your own application and save to your database. For instance, would a hash work best with sets of key/value pairs or a simpler array? We decided that an array would be best for us. At this point, all we knew was that each wrapper (Codeschool, Treehouse, Github, Blog) would need to return an array of strings. As an example, Codeschool could return ["Rails for Zombies", "Ruby Bits"], while Github could return ["Committed to an open source project", "Had a repo forked"]. The individual wrappers will contain their own methods for getting us to the final array, but in the end, an array will be needed. 


<h4>Normalize methods</h4>

Before an array is returned from a wrapper, a method needs to be called to ask for data from an API. As you can see in the gist below, each of our wrappers has the 'get_data' method. So, we could call things like Treehouse.get_data(username) or Blog.get_data(blog_url). Test the method by itself. Write additional methods needed within the wrapper's class to help you manipulate the data for your application. For example, Codeschool.get_data creates a hash from the json returned and then calls the get_completed_courses method to iterate through the hash and return an array of completed courses. 

Here's our get_data method for Codeschool:

``` ruby

def get_data(username)
    cs_json = self.class.get("http://codeschool.com/users/#{username}.json")
    case cs_json.code
      when 200
        self.get_completed_courses(cs_json)
      when 404
        p "Codeschool username not found - #{username}"
        return nil
      when 500..600
        p "ERROR Pulling from Codeschool #{response.code}"
        return nil
    end
end

```

<h4>Create services hash</h4>

After creation of a user, we call user.get_external_data in ther user model. This method contains a hash of service => identifier

``` ruby

 def get_external_data
    external_services = 
    {
      'Treehouse' => self.treehouse_username,
      'Codeschool' => self.codeschool_username,
      'Github' => self.github_username,
      'Blog' => self.id
    }
 end

```

<h4>Call individual wrappers</h4>

 Once you have your hash, iterate through it, calling your predefined get_data methods.

``` ruby

  def get_external_data
    external_services = 
    {
      'Treehouse' => self.treehouse_username,
      'Codeschool' => self.codeschool_username,
      'Github' => self.github_username,
      'Blog' => self.id
    }

    external_services.each do |service, identifier|
      array = service.classify.constantize.get_data(identifier)
    end
 end

```

<h4>Save your data</h4>

 Now comes the fun part. We have the data needed stored in arrays for each service and we can begin storing the data how we see fit in our application. In our case, we created a method called check_achievement_by_array.  

``` ruby

def get_external_data
  ...

  external_services.each do |service, identifier|
    array = service.classify.constantize.get_data(identifier)
    check_achievements_by_array(array)
  end
end 

```

 You'll see that the code looks slightly different in the gist below, but don't worry about that. We are using Sidekiq to offload these API calls to a background service (another blog post to come). You can make these API calls work without Sidekiq.

 For each element in the returned array, we then call check_achievement_by_string. We are essentially checking to see if we have the given star in our database already, if that user already has that star, and if not, create the 'achievement' for that user.

``` ruby

def check_achievement_by_string(string, source_string)
  source = Source.where(:name => source_string).first
  star = Star.where(:name => string, :source_id => source.id).first_or_create
  if source.name == 'Blog'
    self.achievements.create(:star_id => star.id)
  else
    unless self.star_ids.include? star.id
      self.achievements.create(:star_id => star.id)
    end
  end
end
 
def check_achievements_by_array(array, source_string)
  if array
    array.each do |item|
      self.check_achievement_by_string(item, source_string)
    end
  end
end

```

<h4>Check your database</h4>

 Does your data look the way you want it to look? Is it there? It may not be. We fixed many bugs along the way to actually getting our wrappers to work. Whether it's the data itself, the way you are manipulating it, or an error from the API, you will need to work through these bugs to eventually store the data you need. But, it's worth it. 

I hope this helps as an example for those who are depending on mulitiple API's to drive their own application. Feel free to comment below with other suggestions!

{% gist 5454674 %}