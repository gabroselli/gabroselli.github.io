---
layout: post
title: Using UTC Offset in a Rails app.
---

Time zone math is no pleasure cruise, especially if, like me, you are new to the matter. Having users scattered all around the globe, apps need to be intelligent enough to be aware of the correct time for each user - particularly when there are scheduled events and notifications that need to be triggered at a specific time for a specific user.

I pretty much exhausted the internet recently looking for articles about it, and I certainly could have used some more good ones.

The UTC* offset is without a doubt the one indispensable thing when dealing with time zone calculations, and more specifically the UTC offset expressed in hours. For those not familiar with the concept, the UTC offset is the time offset of a specific time zone from the UTC. In my case I first derived the latitude and longitude from the user's address using the [geokit-rails3](https://github.com/jlecour/geokit-rails3) gem, then I used [earthtools.org](http://www.earthtools.org/webservices.htm) to get the <span class="cursive">utc_offset</span>. I then saved it in the user table.

In the user model:
{% highlight ruby %}
class User < ActiveRecord::Base
  def utc_offset
    location = "#{self.address}, #{self.city} #{self.state} #{self.zip}"
    geo = Geokit::Geocoders::MultiGeocoder.geocode(location)
    url = "http://www.earthtools.org/timezone/#{geo.lat}/#{geo.lng}"
    response = Array(HTTParty.get(url).parsed_response)[0][1]
    offset = Array(response)[9][1]
    self.update_attributes(:utc_offset => offset)
  end
end
{% endhighlight %}

I picked [Earthtools.org](http://www.earthtools.org/webservices.htm) because it's free, quick and dirty, and I was really eager to dive into the somewhat frustrating subject that is time zone math.

Rails stores all dates in the database in UTC, so that really helps. Also to avoid any other problems I also set up my app default time to UTC.

In the config/application.rb:
	
{% highlight ruby %}
config.time_zone = 'UTC'
{% endhighlight %}

Finally, since we are using the <span class="cursive">DateTime</span> new_offset method to "localize" the date/time needed for our actions, it's good practice also to store the dates and time in the database as <span class="cursive">datetime</span> data type.

The "localization" happens like so:

{% highlight ruby %}
t = User.find(n).date
offset = Rational(user.utc_offset,24)
localize_datetime = t.new_offset(offset)
{% endhighlight %}

The <span class="cursive">new_offset</span> method creates a new <span class="cursive">DateTime</span> object, identical to the current one, except with a new time zone offset (where offset is the new offset from UTC as a fraction of a day). That is why I used Rational to convert the offset into a fraction.

<span class="cursive">*UTC (Coordinated Universal Time), differently from GMT (Greenwich Mean Time), is based on atomic time and includes leap seconds as they are added to our clock every so often.</span>