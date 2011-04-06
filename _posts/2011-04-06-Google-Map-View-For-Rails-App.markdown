---
layout: post
title: Google Map View For Rails App
---

I recently built a view for a dynamic google map on a Rails application. Displaying the map then is as simple as dropping an iframe tag with parameters for latitude and longitude in the view. For this post I will be creating a personalized dynamic map for users.

When a user creates an account the app relies on the [geokit-rails3](https://github.com/jlecour/geokit-rails3) gem to geocode the user's location and store the latitude and longitude in the database.

{% highlight ruby linenos=inline %}
class User < ActiveRecord::Base
 before_validation :geocode_address
 
 def geocode_address
  geo = Geokit::Geocoders::MultiGeocoder.geocode(self.location)
  errors.add(:address, "Could not Geocode address") if !geo.success
  self.lat, self.lng, self.zip = geo.lat,geo.lng,geo.zip if geo.success
 end
end
{% endhighlight %}

In the maps controller the index action is set to pass the params for latitude and longitude. Also the layout is set to false, so the application layout won't show up in the iframe.

{% highlight ruby linenos=inline %}
class MapsController < ApplicationController
  layout false
  def index
    @lat = params[:lat]
    @lng = params[:lng]
  end
end
{% endhighlight %}

In order to really understand the code on the map index view reading the [google maps JavaScript API docs](http://code.google.com/apis/maps/documentation/javascript/basics.html) it's a must.

{% highlight html linenos=inline %}
<head>
 <style type="text/css">
  html { height: 100% }
  body { height: 100%; margin: 0px; padding: 0px }
  #map_canvas { height: 100% }
 </style>
 <script type="text/javascript" src="http://maps.google.com/maps/api/js?sensor=true"></script>
 <script type="text/javascript">
  var city = new google.maps.LatLng(<%= @lat -%>, <%= @lng -%> );
  var place = new google.maps.LatLng(<%= @lat -%>, <%= @lng -%> );
  var marker;
  var map;
  function initialize() {
   var mapOptions = {
    zoom: 15,
    mapTypeId: google.maps.MapTypeId.ROADMAP,
    center: city};
    map = new google.maps.Map(document.getElementById("map_canvas"), mapOptions);
    marker = new google.maps.Marker({
    map:map,
    draggable:true,
    position: place
   });}
  }
 </script>
</head>
<body onload="initialize()">
  <div id="map_canvas"></div>
</body>
{% endhighlight %}

Once the map index view is ready, it can be placed in other views with the code below. 

{% highlight html linenos=inline %}
<iframe src="/map/?lat=<%= @user.lat %>&lng=<%= @user.lng %>&zoom=15" width="400" height="400" frameborder="0"></iframe>
{% endhighlight %}


