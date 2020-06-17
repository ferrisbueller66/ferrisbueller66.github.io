---
layout: post
title:      "Garden Helper Rails App"
date:       2020-06-17 17:27:57 +0000
permalink:  garden_helper_rails_app
---


I began by sketching the models and relationships I wanted to build out. If I have time, I'll add the fifth model "Seed," indicated in red as an optional resource.

![](https://i.imgur.com/4wJ9MHF.png)

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



