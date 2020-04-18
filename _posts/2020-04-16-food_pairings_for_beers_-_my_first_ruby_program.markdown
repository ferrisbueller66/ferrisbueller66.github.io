---
layout: post
title:      "Beer-Pairings App for Food - My First Ruby Program"
date:       2020-04-16 22:40:55 -0400
permalink:  food_pairings_for_beers_-_my_first_ruby_program
---

25 days ago I didn't know the first thing about programming and really couldn't code anything. Almost 4 weeks into bootcamp, I am now amazed at what I've learned, particularly in Ruby. I understand methods, classes, class methods, and most importantly object orientation. Utilization of existing objects in a ruby program is efficient.

Our first bootcamp assignment was to demonstrate this knowledge of Ruby, and required either scraping a website or using an API, building a program that instantiated objects from the data we gathered from an outside source. I chose to use an API and spent the weekend before project week looking at various free APIs.

I found [PunkAPI](https://punkapi.com/) and immediately saw something I liked about their data. They included food-pairings as data associated with each beer in their database. We have so many tools  available to find food for wine-pairings, but I felt like beer hasn't received the same amount of attention. There are general descriptions of what kinds of beers to pair with certain styles of beer (i.e, pizza and stew go well with dark lagers), but no great way to pick a food and find specific beers based on that food. 

I had my project idea. I wanted to build a program that a user could type in a food and not only get specific beers that paired with that food, but also meal ideas as well based on that food.

I built this project in 4 stages. First,  I set up a model to successfully import data from PunkAPI. Once I established that I was receiving data from the API, I then built a user input-ouput model that would call data from the API. In that model, a user would input a food. That input would instatiate a food object so I then created a third model called Food. That new object would then then serve as the query value for calling the API. I then built a model for beer objects. Each beer that came into the program from the API would be instantiated into a beer object. These 4 models I titled Api, Cli, Food, and Beer, and I envisioned the initial workflow looking like this:


Cli Model (user input) -> Food Model (create new food object) -> Api Model (query data) -> Food (intantiate instances of Food for each returning result).


With food and beer objects created, the program could utilize the data from those objects going forward and minimize the times it hit the API for information. I then created helper methods in each model that allowed a user to find beers through food objects or foods through beer objects, as this method in the Beer class:

``` 
    def self.find_by_food(food)
        self.all.select{|beer| beer.foods.include?(food)}
    end
```

I believe what this project taught me most was how to think about instantiating objects in the workflow of the program.
My first design delayed instantiation, until after data was called from the API. Once the API returned a list of beers, I would have the API model instantiate each beer result as a beer object, and create a food object from the user's input that queried the API (a food word).

I realized that a user could input the same word repeatedly and it would then hit the API repeatedly (something I wanted to avoid), so I applied a helper method to find any object with the same name at time of user input, to avoid hitting the API unecessarily. That problem was solved, but the next question concerned when to instantiate a food object. At time of user input or after data was pulled from the API?

Feedback from my cohort and instructor was helpful at this point. If I instantiated an instance in the Cli model before running the API, I had the chance of creating food objects for foods that didn't exist in the API. Validation was needed. After a trying a few different ways to accomplish this, I landed on the final order of operation:

1. User input entered in the Cli method
2. The Cli method used a helper method to detect if a food object with that name was already created (if so, it simply returned the info on the object)
3. If no object was detected, the input went to the Api model to determine if the input was a viable query value
4. If not, it prompted the user to input again with a different value
5. Otherwise, the API successfully retrieved beers, which were then instantiated as Beer objects.
6. Last, Beer objects were associated with Food objects and vice versa to create reciprocity.

Steps 3 -6 of this validation process are contained in this portion of the Api model code:

```
beers = JSON.parse(response)
    if beers.length > 0
      new_food = Food.find_or_create_by_name(food)
        beers.each do |b| 
          new_beer = Beer.find_or_create_by_name(b["name"], b["description"], b["food_pairing"], b["abv"])
          new_food.beers << new_beer
          new_beer.foods << new_food
        end
    end
```


The results of the project were satisfying. I'm able to type in a single food and find a list of specific beers that pair with that food.  If I look at a description of a specific beer, I can see meal-pairings that go with that beer. Those meals can give me more food ideas to search with, and in seconds I can have a new list of beers based of that new food.

The night I finished this project, I had to make a grocery run. From the store, I called my wife:

"What are we having for dinner?

Pasta?

Go to my desk and type 'pasta' in the program. Tell me what beers it returns."



I had 8 beers to try to find at the store.



Try it yourself! You can download my [Beer-Pairing App for Food program](https://github.com/ferrisbueller66/beer_pairing_app) on my github page.


Special thanks to Robert Gu for help debugging an issue I was having with instantiation, and to Nancy Noyes for helping with line 2 of the Api model code above.
