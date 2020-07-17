---
layout: post
title:      "Structuring a Rails API/JavaScript Single Page App"
date:       2020-07-15 19:32:39 -0400
permalink:  structuring_a_rails_api_javascript_single_page_app
---

I recently worked on a small single-page application using a rails a Javascript frontend. The app is designed to track items a user plans to donate and deliver to a food pantry or non-profit (like Goodwill). Although I had high hopes about the app being practical and for a good cause, its first iteration feels like a glorified reverse grocery-shopping list. However, down the road, this code could be absorbed into the user account of a non-profit's website, allowing them to track what they've donated, and plan future donations. This could also integrate into the non-profit's accounting and help streamline calculations for tax deductions, particularly since the user will be executing some of the data input.

But for now, this single application remains simple. It renders all the past and pending visits to a non-profit along with all the items designated to be donated in that visit. "Visit" and "Item" are objects that user instantiates and consequently get persisted in Postgres. Rather than wallk through the sections of code or explain details of this project, I want to rather discuss some of the structuring behind this project, because code organization posed one of the more challenging aspects of this app.
## Thinking Through Outline and Structure
It didn't take long for the div and id references in the html to start getting complicated. With only 2 models behind the app, it started feeling like the references in the code were forming a hydra.

![](https://media3.giphy.com/media/ZFoTMBgPFwhcmWgtbp/source.gif)

The first 25% of the project was a bunch of trial-and-error. Should I put this reference in a footer? Nope. It now shows up everwhere. Should this other anchor go in the main body? Now it renders or clears out unexpectedly due to different javascript functions.

The next time I approach a similar project I'll whiteboard the design differently. I usually map out all the high level features like the models, their attributes and associations, along with the overall project file structure. But I think I'll add a second whiteboard in the future for my html and js files. This exercise will map out all the eventListeners I expect to make in the js files, and all the corresponding anchors I anticpate needed to make in the html as a result. This might help mitigate some of the unintentional div mania.
## Keeping track of classes and ids
As I built out the project, a common pattern emerged. I'd begin to try to isolate an element using querySelector, hook the element, but inadvertently grab some unwanted element with common features. To remedy this, I'd add a class or id to specify and ensure I could isolate elements. What I didn't take into account was how quickly the list of classes and IDs could compile and how easy it was to lose track of them.

Early in the project I again paused to think how to keep this under control. My first thought was to go back through the code using cmd+f and quickly track down all the ids and classes, and write them out as empty selectors in my styles.css. They may never get styled, but at least I have a running total typed out, in a place that I would eventually need to type them out if I needed to style. Going forward in the rest of the project, any new ID or class I created, I added to the css file.

But more importantly, I also thought about how many of those IDs *could* be repeated across js files. Here re-using the same ID or class names could help reduce code. I wouldn't have to write identical functions only differentiated by similar ID names. I also would have a better chance at refractoring some of those duplicates into a class or function.
## Hiccups
I had a few challenges in this project I feel are worth mentioning.
### Balancing Minimal Code, Speed, and User Experience
The first structural hiccup I ran into pertained to making new Visit objects. I had initially placed an a-tag in the main body of index.html to allow a user to click and create a new Visit. This would render a form and consequently instantiate a new object. Pretty much vanilla object-oriented programming. Since this link rendered on the equivalent of the Visits index page, I wanted to add the new object to the list of all Visits without hitting the database with a new get request. Javascript can accomplish this by simply adding the newly instantiated object to the list.

What if I'm only rendering one particular Visit in the HTML with Javascript (the equivalent of a Show page)? Do I include the link to create a new Visit object? It would provide convenience to the user to be able to create a new object from any "page" related to a visit?

If I recycle the "create" link I used above, it creates a new object successfully but then adds the next object to the current object's show page (because the original JS function I wrote is trying to avoid a second get request to the server, and thinks its adding to a comprehensive list of all Visits).

So do I write a second, modified create function? The first gets used on the "index" page and second gets used on a the "show" page? Or do I sacrifice server calls to keep the code more simple, and stick with one function to be used across the board? If I opt to write a second function, I need to also update my addEventListeners and possible write another set of those as well. This was only my first model and mapping this out was becoming a headache quickly.
### Deleting Child Objects with Their Parent
To round out all the CRUD actions on objects in my project, I added delete functionality to each visit object, so user could click and delete Visit object. This would send a fetch with a delete request to the server. I didn't want to leave an orphaned data, so I though about how I could remove any children objects associated with this Visit object. I would need to delete them first, otherwise, with no parent as a reference for their foreign key, the children would cause bugs.
My first attempt was to first iterate through each child and send a fetch with a delete request. This seemed really demanding on the server, but I wanted to get something up and running first, and then refractor.

```
function deleteVisit(){      
    event.preventDefault();
    let id = event.target.dataset.deleteId
    let items = event.target.parentElement.getElementsByClassName('item-li')
    if (items.length > 0){
        for (const item of items){
                fetch(BASE_URL+`/items/${item.dataset.itemId}`, {
                    method: "DELETE",
                    headers: {
                        "Content-Type": "application/json",
                        "Accept": "application/json"
                    }
                })
                    .then(item.parentElement.remove())
        }
        deleteVisit()
    }
    if (items.length == 0){
        fetch(BASE_URL+`/visits/${id}`, {
            method: "DELETE",
            headers: {
                "Content-Type": "application/json",
                "Accept": "application/json"
            }
        })
            .then(event.target.parentElement.remove())
   // }
}
```

This actually worked for the children. They deleted from the database fine, but it prompted several SQL errors when I moved on to delete the parent. Perhaps there were racing issues with the multiple fetch requests. I really didn't like all the hits to the server.

Fortunately, ActiveRecord has some magic for all this, all in 1 line of code!

```
has_many :items, :dependent => :destroy 
```
This solved everything: it quickly deletes the children and the parent with only 1 fetch request, without all the clunky iteration.

![](https://i.imgur.com/tS5Tv0L.png)

BIG BIG thank you to my colleague Arryn Boniface for tipping me off to this ActiveRecord feature. [Check out her blog here](https://ronniekram.github.io/).
## Fun Features
I had time to investigate and was able to implement a few features I felt were not only fun but enhanced the user experience.
### Less Work, More Play
 On my create Visit form, I initially put a placeholder value in to help guide and instruct the user. But once you clicked on the field, you had to manually erase the placeholder text.

![](https://i.imgur.com/9XWTamr.gif)

What a pain. 

I wonder if I could put an eventListener on that field and delete the placeholder text upon click?

Sure can.

![](https://i.imgur.com/Tb7gEfX.gif)

I love my life. This was accomplished by 3 lines of code: 
```
function clearPlaceHolderOnClick(textField){
    event.target.value = ""
}
```
and then I called this function as an argument for an eventListener.

I kept the name of the function generic, so that it can be called in other form fields I might have in other parts of the project and keep the code DRY.

### Elevator Ride
As a user keeps adding Visits and Items to the app, the default list will eventually extend beyond the fold of any screen. I had originally placed the link to create a new visit at the bottom of the page, but added a second one at the top for accessibility. I set a div-tag where the form would populate on the html. Upon clicking the "Create Visit" link, it auto-scrolled to the bottom of the page and rendered the form.

`<a id="newVisit" href="#createVisitForm">Create Next Visit</a>`

No more extra scrolling by the user.
### Want to see under the hood? 
You can [check out the repo here](https://github.com/ferrisbueller66/FoodShare).
