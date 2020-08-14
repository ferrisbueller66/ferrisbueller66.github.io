---
layout: post
title:      "3 Specific Tips for a React/Redux Project"
date:       2020-08-14 17:34:48 +0000
permalink:  3_specific_tips_for_a_react_redux_project
---

React provides a very flexible, reactive framework due to its components, but, combined with Redux, can generate some complexity quickly. Rather than walkthough an entire project or give general advice, I wanted to address 3 specific issues I had faced in a recent React project. If you are using multiple reducers, plan to have a delete function, or are trying to redirect to a specific page after submiting a form, hopefully this blog will help.

First, if you set up more than one reducer, and use React-Redux' combined-reducer, you'll need to be conscious about how you call state in your app. Second, this will affect your delete action, not as much the fetch with a delete request to the database, but more so the re-rendering of your state after deleting. Third, if you build a form that constitutes its own container and route, you'll need to invoke useHistory to redirect to a new url path and component after submitting the form. But if you build that container as a class rather than a classless function, you'll have amazing access to window.history. Let's look at each of these three in order.

## Calling State from Redux with Multiple Reducers

In my project, the app catalogued different bourbons and whiskeys. I created a global state using Redux, but my initial fetch to the database wasn't populating Redux' state successfully.

I had originally written my state locally to the container components and then passed down state using props. 

```
state = {
  bourbons: []
}

const mapStateToProps = (state) => {
return { bourbons: this.state.bourbons}
}
```

Implementing Redux required me to connect the component to Redux and then write a mapStateToProps function in each component that required access to state in their props:

```
const mapStateToProps = (state) => {
  return {
    bourbons: ?,
    loading: ?
  }
}
```

None of this is unexpected. If the state was local, like I originally had it, this JS object would look like this:

```
bourbons: this.state.bourbons,
loading: this.state.loading
```

I thought simply calling the state through props would summon the data, now that we had connected to a Redux store:

```
const mapStateToProps = (state) => {
    
    return { 
        bourbons: state.bourbons,
        loading: state.loading
    }
}
```

Nothing. 

What if I tried calling it through my combined Reducer?
```
const mapStateToProps = (state) => {
    
    return { 
        bourbons: state.indexReducer.bourbons,
        loading: state.indexReducer.loading
    }
}
```

Definitely doesn't work but thought I'd give it a shot. 

Time to debug. Taking a look in my mapStateToProps function produced the following result:

![](https://i.imgur.com/RN6fwDJ.png)

Wait? What!?! I was looking for the bourbons and loading values to be directly under the state. I had forgotten I had created subsets of state named "bourbons" and "categories" when I had made my combined Reducer. As a result, I'll need to call state using state.bourbons.bourbons.

```
const mapStateToProps = (state) => {
    
    return { 
        bourbons: state.bourbons.bourbons,
        loading: state.bourbons.loading
    }
}
```

This worked. The take away from this is that creating multiple reducers and using a combined reducer creates subsets in your state. You'll need to account for this in your MapStateToProps. Remember this formula:

`state.nameOfReducer.nameOfStateValue`

## Updating State After Deleting an Object

I had a little trouble with the delete action for these objects. I had set a fetch with a DELETE request to the backend, and that fired off fine. The code looked like this:

```
const deleteBourbon = id => {
  return (dispatch) => {
    fetch(`http://localhost:3001/bourbons/${id}`, {
      method: "DELETE",
      headers: {
        "Content-Type": "application/json",
        "Accept": "application/json"
      }
    })
    .then(id => dispatch({type: 'DELETE_Bourbon', payload: id}))
  }
};
```

The data was being destroyed in the database without any issue. But the re-render in React wouldn't remove it from view. The problem then was not in the Action above but in my reducer:

```
case 'DELETE_BOURBON':
  idx = state.findIndex(bourbon => bourbon.id  === action.payload)
  return {...state, bourbons: [[...state.bourbons.slice(0, idx),
  ...state.bourbons.slice(idx + 1)]]};
```
	
I put a debugger in the reducer. Made an object, tried to delete it and… nothing. Didn't hit the debugger. So wasn't getting to the reducer. 

Time to take a step back.

I didn't have the action.type fulling capitalized in my action above. 

So picky. 

Changing to 

`type: 'DELETE_Bourbon', > type: 'DELETE_BOURBON' `

got me to the reducer, but I still didn't reach my debugger before breaking the app with the error "The method call "state.findIndex is not a function." 

Right. 

This piggy-backed on my first issue. Now that I identified the nested state that combined reducer created, I needed to reflect that in my reducers. "State.bourbons" not simply "state" is housing the array of data. After changing that line of code in the reducer to:

`idx = state.bourbons.findIndex(bourbon => bourbon.id  === action.payload)`

That did the trick and I then hit my debugger.

### Bonus Round

That should have done it, but I had found one other bug preventing proper re-rendering, and this was caused by the boilerplate method call .slice I had first written for my reducer. If this isn't of interest, skip down to my third point below on useHistrory().

At this point, when I hit the delete button, I expected the page to render the list without it. But hitting delete cleared everything on the page after re-render. The database was fine - everything was still there and only the objects I hit delete on were being deleted on the backend. I'm just not re-rendering state accurately. Back to debugging in the reducer. Again, the code at this point read:

```
case 'DELETE_BOURBON': 
  idx = state.bourbons.findIndex(bourbon => bourbon.id  === action.payload)
  debugger
  return {...state, bourbons: [[...state.bourbons.slice(0, idx), ...state.bourbons.slice(idx + 1)]]};
	```
	
In debugger, I typed "idx", expecting the variable's assigned value to be 13. I got -1. 

Ok, that definitely isn't right. 

How about "action.payload?" 

Nothing. Interesting. 

The ID wasn't coming over in the payload value of the action.

Back to the action then. The argument for the final anonymous function that executed dispatch was the problem:

```
return (dispatch) => {
    fetch(`http://localhost:3001/bourbons/${id}`, {
      method: "DELETE",
      headers: {
        "Content-Type": "application/json",
        "Accept": "application/json"
      }
    })
    .then( () => dispatch({type: 'DELETE_Bourbon', payload: id}))
  }
};
```

Changing the final dispatch to the following did the trick:
`.then(id => dispatch({type: 'DELETE_Bourbon', payload: id}))`

We now have a payload with an id number carrying over to the reducer. But when I click delete, everything still wiped from the render. Time to take one more walk through the reducer:

```
case 'DELETE_BOURBON': 
  idx = state.bourbons.findIndex(bourbon => bourbon.id  === action.payload)
  debugger
  return {...state, bourbons: [[...state.bourbons.slice(0, idx), ...state.bourbons.slice(idx + 1)]]};
	```
	
The variable idx was still returning an empty array at the debugger. The method call .findIndex wasn't getting the job done. And I just realized why. The index number that React assigned to the array during .map was entirely different from the primary key the database table assigned an object. The .findIndex was doomed to fail from the start with this approach. Rather than fiddle with .findIndex to pull the primary keys, I substituted .filter in and consequrly decreased the code by a few lines:
```
case 'DELETE_BOURBON':
   return {...state, bourbons: state.bourbons.filter(bourbon => bourbon.id  != action.payload)};
```

I was curious to see the slice would compare . [Justin Tulk has a very quick summation here](https://medium.com/@justintulk/javascript-performance-array-slice-vs-array-filter-4573d726aacb), concluding that slice generally was less costly to employ than .filter until you were processing 1M+ items in an array.

Implementing the .filter worked fine and got everything up and running. I would have liked to have utilized .findIndex and this is still possible later, after I have the minimum requirements of the project up and running.

## Re-Rendering after Submitting a Standalone Form

There was one other issue that I found interesting in this project that I want to share. I had a sizeable form to render and rather than build it as a simple component, I made it a container and gave it its own route. It warranted it. The form legitimately looked like its own page. If you're used to having a form featured within a page, upon submission, React notices an update of state and re-renders. But in this case, the standalone page I created for the form didn't have anything to re-render. It new global state was being updated, but the component taking up the whole page didn't render an a list or index of data...just the form.

Upon hitting submit, it just hung out there.

I was going to have to call useHistory to redirect to a different route onSubmit in order to allow the re-rendering I wanted. This sounded simple enough. React-Router provides this function, which we can call at the top of the container:

`import { useHistory } from 'react-router-dom';`

However, utilizing the function quickly became unwieldy for my purposes. A boilerplate example, [provided by React Training](https://reacttraining.com/blog/react-router-v5-1/), looks like so:

```
function BackButton({ children }) {
  let history = useHistory()
  return (
    <button type="button" onClick={() => history.goBack()}>
      {children}
    </button>
  )
}
```

Creating a separate function for the Submit button became messy, particularly because I wrote the form as a class (as container), not as a classless function (component).

Because I had written the entire form as a class, the class has access to window.history. History automatically was passed to a class in props.

At the end of my onSubmit function, I simply had to call:

`this.props.history.push('/bourbons')`

This effectively re-rendered in 1 line of code. There was no need for a separate useHistory function and I could also remove the `import { useHistory } from 'react-router-dom';`.

If you find you need to redirect to a new route FROM a class, remember that React powerfully passes window.history to the class through props. This is an amazing little feature that I learned about from this project.
