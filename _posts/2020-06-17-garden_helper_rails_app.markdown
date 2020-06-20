---
layout: post
title:      "Garden Helper - A Rails App"
date:       2020-06-17 13:27:58 -0400
permalink:  garden_helper_rails_app
---

My most recent project involved building a website with Ruby on Rails. I designed an web app that allows gardeners to track the grow beds they have, the plants in them, and the harvests that each plant produces. I began by sketching the models and relationships I wanted to build out. If I have time, I'll add the fifth model "Seed," indicated in red as an optional resource.

![](https://i.imgur.com/4wJ9MHF.png)

For object orientation, a plant belongs to a user and belongs to a grow bed in this app. A User can have many beds through a plant as a result. A Harvest belongs to a plant. Consequently, a User or a Grow Bed can have many harvests through a Plant.

I began this project by utilizing the devise gem in rails to build out a User Model, and then integrate OmniAuth to allow 3rd Party Authentication through Github. From there I built index and show controller actions and views for the Bed model, then subsequently the same for the Plant model. After theses views were rendering successfully, I worked on creating new, create, edit, and update controller actions and respective views for both of these Models. Once I had these rendering successfully, and confirmed objects could be instantiated or updated successfully, I began nesting routes and creating a few nested forms in the "new" views. I finished by refractoring by using partials in a couple of the views and create a few private methods in the controllers to cut down on redundant. Perhaps your process is different, but the final result should be fairly boilerplate, RESTful rails. What I want to focus on in this blog is three of the features that I enjoyed the most in development.
## date_field
I wanted to use date rather than datetime for some of the attributes in my Plant model, but immediately found some issue instantiating objects. My migration file looked like this:

```
def change
    create_table :plants do |t|
      t.date :germination_date
      t.date :transplant_date
      t.string :nickname
      t.string :variety
      t.string :species
      t.text :description
      t.integer :user_id
      t.integer :bed_id

      t.timestamps
    end
```

and my initial workaround in rails console to create a Plant with a transplant date was:

```
c.transplant_date = Date.parse("May 16 2020")
```

Rails form builder allows for date_field, which automatically parses the date in params, allowing the Plant object to be instantiated without a hitch. Additionally, the date_field renders a nice calendar drop-down for the user to select a date from...minimizing their typing and mitigating against typos that would lead to persisting bad data.
## Iterating a has_many_through

Grow beds have many plants.  A plant belongs to a grow bed. The index view of a grow bed showed all grow beds associated with a user. Since a user has many plants, and several of those different plants can share the same bed, in my initial iteration, the same bed was being duplicated in the view due to the iteration:

```
<% @beds.each do |bed|  %>
<ul>
<li> <%= link_to bed.name, bed_path(bed) %></li>
</ul>
<%end  %>
```

I had recently used .select in a form, so since that was fresh in my mind I tried calling @beds.select(:id).uniq instead of @beds.each provides only one of each ID#, but .select returns an array with only the object's instance number and the ID number, it doesn't return the full object. I would then have to iterate across the results of calling .select, find_by_id, and then return the object's attributes I want to render. This would hit the database a second time, something I want to avoid.

Additionally, .select with .uniq was redundant. Dot uniq should be able to handle this. After a refresher on syntax from Stack Overflow, I wrote out 
```

<%@beds = @beds.uniq {|b| b.id}%>
<% @beds.each do |bed|  %>
<ul>
<li> <%= link_to bed.name, bed_path(bed) %></li>
</ul>
<%end  %>
```

This worked, but the view shouldn't be interacting directly with the database, so I moved the logic to the controller's index action like so:

```
def index
    @beds = current_user.beds.uniq {|b| b.id}
 end
```

## Booleans Tripping

In the Plant model, I created a boolean database table column called "Harvested" to help track if a plant had been harvested or not. To utilize this attribute, I first wrote a method in the Plant model:
```
  def harvest_status
        unless self.harvests.count == 0
        self.harvested = true
				self.save
        end
    end
		```
		
		then called that method in the Harvest controller create action:
		
```
		  @harvest = Harvest.create(harvest_params)
    if @harvest.valid?
      @harvest.plant.harvest_status
```

When a Harvest instance was instantiated, the method would be called to trip "harvested" attribute of the Plant it belonged to, from false to true.
	
This came in handy, since I eventually created a scope that queried all harvested plants:

```
scope :been_harvested, -> { where(harvested: true) }
```

Want to see more?

Check out my repo of [the Garden Helper at Github](https://github.com/ferrisbueller66/garden_helper).

