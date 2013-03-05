---
layout: post
title: "Creating My Songkick-to-Calendar App"
date: 2013-02-23 14:58
comments: true
categories: 
---

Last week, myself and two other Flatiron students were assigned with building and presenting our own ruby command line interface (CLI) applications. The task at hand was to combine a data source with a messaging service in order to make something useful. Here's the process I went through to create <a href="https://github.com/dolin/songkick-to-cal/" target="_blank">my first useful ruby app.</a>

<h3><strong>1. Idea.</strong></h3>

Concert listing scraper that asks for user input and automatically adds events of your favorite artists to your calendar.

The final product: 

{% img http://s11.postimage.org/iav6sea2b/Screen_Shot_2013_02_26_at_4_59_01_PM.png %}

Songkick found two Beyonce shows on August 3rd and 4th at the Barclays Center in Brooklyn, NY. Both files were created in the current directory. Below I've opened the files with iCal. 

{% img http://s17.postimage.org/m48aa7s7j/Screen_Shot_2013_02_26_at_5_07_07_PM.png %}

<h3><strong>2. Oh shit.</strong></h3>

I actually have to build this thing and don't know where to start. It's Monday and the app needs to ready to roll by Thursday. 

<h3><strong>3. Break it down.</strong></h3>

My initial excitement to build a concert parser had quickly turned into a wall of stress and doubt. Avi could tell that I started to quit on the project and told me that I needed to both figuratively break down the wall and literally break down the problem. He was right. As a beginner dev, it's easy to be consumed by the idea of the final product without realizing that the overarching big problem is really just a set of interweaving easier problems. 

<h3><strong>4. Pseudocode.</strong></h3>

Pseudocode essentially solved this problem for me. Unfortunately, I must have written over my initial code, but it went something like this: 

```
# get artist from user

# get location from user

# use location to lookup the location id using Songkick API

# use songkickr gem with artist name and location id to fetch data for each show

# create hash to store event name and date from songkickr gem results

# create ical files from hash

```

<h3><stong>5. Build the parts individually.</strong></h3>

The two core pieces of functionality needed for my app to work hinged on creating ical files and fetching data from Songkick's API. While there would eventually have to be code that ties these two pieces together, it made the most sense to me to write them separately (again, breaking the problem down to smaller parts). Here's the inital code I wrote to create an ical file. Most of it comes directly from the Icalendar gem.

``` ruby

cal = Calendar.new
event = Event.new
event.start = Time.now
event.summary = "This is the event's summary"
cal.add_event(event)

File.open('testing'.ics", 'w+') do |f|
      f << cal.to_ical
    end

``` 

Eventually I would have to fill in event.start, event.summary, and the filename with variables, but all I wanted to do at this point was make sure that the creation of files worked properly. 

Similiarly, I hardcoded artist/location information into the songkickr gem to make sure it returned data for upcoming Beyonce shows in NYC. The gem is essentially a Songkick API wrapper that performs the API call and fetching of the concert data for you. 

``` ruby

remote = Songkickr::Remote.new 'Your Songkick API Key'
remote.events(:artist_name => "Beyonce", :location => "sk:7644", :min_date => "2013-02-20", :max_date => "2013-10-31")

```

Sidenote: Unless I am mistaken, it turned out that the Songkickr gem cannot accept a location such as 'NYC' and instead must have an 'sk:id' that points to a location (i.e. sk:7644 above points to NYC). Because of this, I needed to call Songkick's API myself in order to return a location id. That location id was then used in the songkickr gem. Here's the code: 

``` ruby

require 'open-uri'
require 'json'

doc = open("http://api.songkick.com/api/3.0/search/locations.json?query='#{interpolate user location}'&apikey='Your Songkick API Key').read
location_hash = JSON.parse(doc)
location_id = location_hash.flatten[1]["results"]["location"][0]["metroArea"]["id"]

```

<h3><strong>6. Methodize.</strong></h3>

After filling in my pseudocode with actual code, it became very clear that each piece of functionality needed to live within its own method. Also needed was a hash to store the event name and date. To accomplish this, I first created a 'shows' hash. Next, I iterated over the results from the Songkickr gem, creating key-value pairs of event name and date within the 'shows' hash. Finally, I created ical files by iterating over the hash, replacing previously hardcoded values with the new key value pairs of event name and date. See below for the complete gist.

```ruby

 def create_event_hash
  @event_results.each do |event|
    @shows[event.display_name] = event.start
  end
end

```

```ruby

def create_files
  @shows.each do |display_name, date|
    cal = Calendar.new
    event = Event.new
    event.start = date
    event.summary = display_name
    cal.add_event(event)

      File.open("'#{display_name}'.ics", 'w+') do |f|
          f << cal.to_ical
        end
  end
end

```

<h3><strong>7. Object-orient...ish.</strong></h3>

Being only a few weeks into really learning code, I'm still wrapping my head around object orientation. I understand the theory, but it's a challenge to implement a variety of classes that do the right things and play nicely with each other. So, for the sake of time, I decided to create one giant class called 'Result.' The Result class contains a method called 'run' which essentially fires off each method in order.

Moving forward, I plan to refactor the code into multiple classes and also integrate the app with Google Calendar. 

<h3><strong>8. Next Steps.</strong></h3>

I couldn't believe that by Thursday I had a tangible, presentable, functioning version of a product that was in my head three days previously. I've never been able to say that before and I think that's pretty cool. Stay tuned for the presentation video.  

If anyone has feedback on my code or suggestions for the type of classes to build, I'd be all ears. 

...now back to work. 

{% gist 5024843 %}











