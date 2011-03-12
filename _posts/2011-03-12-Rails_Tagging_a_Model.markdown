---
layout: post
title: Using UTC Offset in a Rails app.
---
Tagging is a very simple concept. Wikipedia describes a tag as "a non-hierarchical keyword or term assigned to a piece of information".

There are few gems out there about tagging such as [acts-as-taggable-on][1], [is_taggable][2] and so on. I do value the convenience and power of gems. And if I can I’d rather build my own logic. Mostly because it is the easiest way to deepen my knowledge on the subject.

If you are dealing only with one model that needs tagging, developing such a system in Rails is quite simple. For this article I will use tagging a blog post as example. 

I usually establish the model associations right away. In this case it’s a many to many association between tags and posts like so:

In the post model:
{% highlight ruby linenos=inline%}
class Post < ActiveRecord::Base
 has_many :tags, :through => :posts_tags
 has_many :posts_tags 
end
{% endhighlight %}

In the tag model:
{% highlight ruby linenos=inline%}
class Tag < ActiveRecord::Base
 has_many :posts_tags
end 
{% endhighlight %}

In the join post tag model:
{% highlight ruby linenos=inline%}
class PostsTag < ActiveRecord::Base
 belongs_to :post belongs_to :tag
end
{% endhighlight %}

Assuming that your post table is already in place, the next step is to generate the migrations to create the tags and the join posts_tags tables.

{% highlight text %}
$ rails g migration CreateTags
$ rails g migration CreatePostsTags
{% endhighlight %}

{% highlight ruby linenos=inline%}
class CreateTags < ActiveRecord::Migration
 def self.up 
  create_table :tags do |t| 
   t.string :name 
   t.timestamps
  end
 end

 def self.down
  drop_table :tags 
 end 
end 
{% endhighlight %}

{% highlight ruby linenos=inline%}
class CreatePostsTags < ActiveRecord::Migration
 def self.up
  create_table :posts_tags do |t| 
  t.references :post
  t.references :tag
  t.timestamps
 end 
end

 def self.down
  drop_table :posts_tags
 end
end
{% endhighlight %}

Let’s not forget to migrate.
{% highlight text %}
 $ rake db:migrate
{% endhighlight %}

After the migration, controller’s actions are next so that we can find, create and associate tags to posts when posts are created or edited.

{% highlight ruby linenos=inline %}
class PostsController < ApplicationController
 def new
  @post = Post.new
  @tags = Tag.all

  respond_to do |format|
   format.html # new.html.erb
   format.xml { render :xml => @post }
  end
 end

 def edit
  @post = Post.find(params[:id])
  @tags = Tag.all
 end

 def create
  @post = Post.new(params[:post])
  unless params[:tags].nil?
   params[:tags] = params[:tags].collect{|tag| Tag.find_or_create_by_name(tag)} 
  end

  respond_to do |format|
   if @post.save
    @post.create_new_tags(params[:new_tags][0]) unless params[:new_tags].nil?
    @post.tags << params[:tags] unless params[:tags]nil?

    format.html { redirect_to(@post, :notice => 'Post was successfully created.') }
    format.xml  { render :xml => @post, :status => :created, :location => @post }
   else
    format.html { render :action => "new" } 
    format.xml { render :xml => @post.errors, :status => :unprocessable_entity }
   end
  end
 end

 def update
  @post = Post.find(params[:id])
   unless params[:tags].nil?
    params[:tags] = params[:tags].collect{|tag| Tag.find_or_create_by_name(tag)}
   end
  respond_to do |format|
   if @post.update_attributes(params[:post])
    @post.create_new_tags(params[:new_tags][0]) unless params[:new_tags].nil?
    @post.tags << params[:tags] unless params[:tags].nil?
    format.html { redirect_to(vendors_path, :notice => "Vendor was successfully updated.") }
    format.xml { head :ok }
   else format.html { render :action => "edit" }
    format.xml { render :xml => @vendor.errors, :status => :unprocessable_entity }
   end
  end
 end
end
{% endhighlight %}

On line 25 and 45 a “create_new_tags(params\[:new_tags\])” method is called for @post. This method allows the user the option to create new tags by providing a list of comma separated values. The method in the post model looks like this:

{% highlight ruby linenos=inline%}
class Post < ActiveRecord::Base
 def create_new_tags(tags) 
  new_tags = tags.split(%r{,\s*}) 
  new_tags.each do |tag|
   self.tags.create(:name => tag)
  end
 end
end
{% endhighlight %}

Finally the post form view brings the above logic to the surface. The text field starting on line 1 is add new tags as a list of comma separated values. The check boxes assign existing tags to the post.

{% highlight erb linenos=inline%}
<div>
 <%= label_tag :tags %>
 <%= text_field_tag('new_tags[]')%> 
</div>
<div>
 <% @tags.each do |tag| %>
</div>
 <%= check_box_tag('tags[]', tag.name) %>
 <%= label_tag(tag.name)%>
<% end %>
{% endhighlight %}

And that’s it. You can now tag the post model.

[1]: https://github.com/mbleigh/acts-as-taggable-on
[2]: https://github.com/jamesgolick/is_taggable

