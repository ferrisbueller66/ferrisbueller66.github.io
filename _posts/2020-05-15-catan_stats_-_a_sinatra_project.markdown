---
layout: post
title:      "Catan_Stats - A Sinatra Project"
date:       2020-05-15 13:55:47 +0000
permalink:  catan_stats_-_a_sinatra_project
---


This is week 8 of full-time software engineering bootcamp and I've gone from zero knowledge of coding and web development to building out a full stack in 3 days. Our project requirements were to write in Ruby, using a SQLite databas,  ActiveRecord as the interface for the models, and Sinatra as the framework. This project was incredibly fulfilling since it allowed me to showcase everything I've learned in 2 months as well as visusally see the results of my coding in a web browser for the first time.

One other thing made this project so fun. We had freedom to create virtually any website we wanted, provided we could demonstrate building at least a couple models that maintained a has-many, belongs-to relationship. I decided to use Settlers of Catan as the inspiration for the project. As a family, we keep track of our dice rolls when we play Catan. Although the most common dice roll is generally "7", each game seems to have one or two other dominant rolls, and tracking that in-game helps with strategy. Although my personal motivation was to use this for Catan, I expanded the project to allow for other games that use 2 dice, such as Monopoly.

![](https://img.buzzfeed.com/buzzfeed-static/static/2019-10/8/15/asset/e9fec4db735b/anigif_sub-buzz-325-1570547000-1.gif?crop=230%3A230%3B7%2C0&resize=475%3A%2A)

I created 3 models for the project that would create the following objects: users, games, and turns. This resulted in a nested has-many relationship.  A turn belonged to a game, and a game belonged to a user. Conversely, a user had many games, and many turns through games. This structure helped my start the project quickly. I built migration ruby files and class models before creating a sqlite database. After migrating seed data and testing that ActiveRecord was making accurate associations, I began building out the controllers that would redirect routes and render views.

Each controller had basic CRUD actions, and generally demonstrated a corresponding erb file providing a view roughly for every controller action. For creating a new *game* [instance] in the turns controller, a basic request/flow would look something like this. A get request would be received at a route, which Sinatra would identify and redirect to a view that contained a form. The user's input would provide the parameters for creating a new *turn* instance, which would fire off a post request. The *create* controller action would pass the parameters to ActiveRecord to instantiate and persist a new instance, and then finally render a *new* view, presumably showing the newly created object (a game).

However, as I progressed through the project, I was able to experiment. I bypassed rendering a view for a *new* turn in a game. Rather than create an erb file to provide a view of a form (to create a new turn in a game), I embedded the form in a game's *show* erb. The form precluded any need for a *new* controller or *new* view for a turn, and routed straight to the *create* action in the turns controller. I also used drop-down options in the form to maintain uniform data being persisted in the database:

``` 
<form method="POST" action='/games/<%=@game.id%>/turns'>
    <label>Roll/Spin Result:</label> <br>
    <select name="result">
        <option>1</option>
        <option>2</option>
        <option>3</option>
        <option>4</option>
        <option>5</option>
        <option>6</option>
        <option>7</option>
        <option>8</option>
        <option>9</option>
        <option>10</option>
        <option>11</option>
        <option>12</option>
    </select>
    <br><br>

    <input type="submit" value="submit" id="submit">
  </form>
```


I wanted to maintain RESTful routes with my nested has-many relationships. *Turns* in a game were the most nested, belonging both to a game, and to a user through a game. Editing a turn object needed a route that reflected the object was accessed through a game, so the *edit* route in the turns controller ended up like this:

```
   patch '/games/:game_id/turns/:id' do
        if logged_in?
            @game = current_user.games.find_by_id(params[:game_id])
            @turn = @game.turns.find_by_id(params[:id])
            @turn.update(result: params[:result])
            redirect "/games/#{@game.id}"
        else
            redirect '/login'
        end                       
    end
```


With the controllers redirecting successfully to routes, I had a functional website in place. I now could query SQLite and present data to the user. The first piece of this that I constructed presented a user the most commonly occurring roll result in a current game: 
``` 
games = current_user.games.count
                count = @game.turns.count
                
                @length = @game.turns.count
                     if @length >= 1
                         x = @game.turns.all.group(:result).count
                         @top_roll = x.sort_by{|k, v| -v}.first[0]
                         @times = x.sort_by{|k, v| -v}.first[1]
                        
                    else
                       @top_roll = "No Data"
                       @times = 0
                     end
                erb :"/games/show
```

This project was extremely fulfilling, allowing me to develop a full-stack from start to finish, work on my sql and activerecord querying skills, and begin to practice working on some CSS and front-end design skills.

If you want to see Catan_Stats, [Check out the repo here](https://github.com/ferrisbueller66/Catan_stats).


