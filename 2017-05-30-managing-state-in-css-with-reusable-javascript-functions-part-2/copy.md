<!-- 
Managing State in CSS with Reusable JavaScript Functions - Part 2
Posted on 30th May 2017

Originally posted at:
https://css-tricks.com/managing-state-css-reusable-javascript-functions-part-2/
-->

In my previous article, which shall now retroactively be known as [Managing State in CSS with Reusable JavaScript Functions - Part 1](https://css-tricks.com/managing-state-css-reusable-javascript-functions/), we created a powerful reusable function which allows us to quickly add, remove and toggle stateful classes via click. 

One of the reasons I wanted to share this approach was to see what kind of response it would generate. Since then I've received some interesting feedback from other developers, with some raising valid shortcomings about this approach that would have never otherwise occurred to me.

In this article, I'll be providing some solutions to these shortcomings, as well as baking in more features and general improvements to make our reusable function even more powerful.

For reference, here's the JavaScript from [Part 1](https://css-tricks.com/managing-state-css-reusable-javascript-functions/) for our reusable function as it stands:

```javascript
// Grab all elements with required attributes
var elems = document.querySelectorAll("[data-class][data-class-element]");

// closestParent helper function
closestParent = function(child, match) {
  if (!child || child == document) {
    return null;
  }
  if (child.classList.contains(match) || child.nodeName.toLowerCase() == match) {
    return child;
  }
  else {
    return closestParent(child.parentNode, match);
  }
}

// Loop through if any are found
for(var i = 0; i < elems.length; i++){
  // Add event listeners to each one
  elems[i].addEventListener("click", function(e){

    // Prevent default action of element
    e.preventDefault();

    // Grab classes list and convert to array
    var dataClass = this.getAttribute('data-class');
    dataClass = dataClass.split(", ");

    // Grab linked elements list and convert to array
    var dataClassElement = this.getAttribute('data-class-element');
    dataClassElement = dataClassElement.split(", ");

    // Grab data-class-behaviour list if present and convert to array
    if(this.getAttribute("data-class-behaviour")) {
      var dataClassBehaviour = this.getAttribute("data-class-behaviour");
      dataClassBehaviour = dataClassBehaviour.split(", ");
    }

    // Grab data-scope list if present and convert to array
    if(this.getAttribute("data-class-scope")) {
      var dataClassScope = this.getAttribute("data-class-scope");
      dataClassScope = dataClassScope.split(", ");
    }

    // Loop through all our dataClassElement items
    for(var b = 0; b < dataClassElement.length; b++) {
      // Grab elem references, apply scope if found
      if(dataClassScope && dataClassScope[b] !== "false") {
        // Grab parent
        var elemParent = closestParent(this, dataClassScope[b]),

        // Grab all matching child elements of parent
        elemRef = elemParent.querySelectorAll("." + dataClassElement[b]);

        // Convert to array
        elemRef = Array.prototype.slice.call(elemRef);

        // Add parent if it matches the data-class-element and fits within scope
        if(dataClassScope[b] === dataClassElement[b] && elemParent.classList.contains(dataClassElement[b])) {
          elemRef.unshift(elemParent);
        }
      }
      else {
        var elemRef = document.querySelectorAll("." + dataClassElement[b]);
      }
      // Grab class we will add
      var elemClass = dataClass[b];
      // Grab behaviour if any exists
      if(dataClassBehaviour) {
        var elemBehaviour = dataClassBehaviour[b];
      }
      // Do
      for(var c = 0; c < elemRef.length; c++) {
        if(elemBehaviour === "add") {
          if(!elemRef[c].classList.contains(elemClass)) {
            elemRef[c].classList.add(elemClass);
          }
        }
        else if(elemBehaviour === "remove") {
          if(elemRef[c].classList.contains(elemClass)) {
            elemRef[c].classList.remove(elemClass);
          }
        }
        else {
          elemRef[c].classList.toggle(elemClass);
        }
      }
    }

  });    
}
```

Going forward, this is going to serve as the base for our improvements.

Let's get started!


## Accessibility

The most common piece of feedback I've received from other developers in response to [Part 1](https://css-tricks.com/managing-state-css-reusable-javascript-functions/) was the approach's lack of consideration for accessibility. More specifically, it's lack of support for ARIA attributes (or ARIA states if you prefer) and its failure to provide keyboard events for triggering our reusable function.

Let's see how we can integrate both.

### ARIA attributes

ARIA attributes form part of the [WAI-ARIA specification](https://www.w3.org/TR/wai-aria/). In the words of the specification they...

> ...are used to support platform accessibility APIs on various operating system platforms. Assistive technologies may access this information through an exposed user agent DOM or through a mapping to the platform accessibility API. When combined with roles, the user agent can supply the assistive technologies with user interface information to convey to the user at any time. Changes in states or properties will result in a notification to assistive technologies, which could alert the user that a change has occurred.

Revisiting the accordion example from [Part 1](https://css-tricks.com/managing-state-css-reusable-javascript-functions/), an `aria-expanded` attribute set to `true` when the component is expanded, and vice versa when at its default state, would allow assistive technologies such as screen readers to better assess the component.

In addition to providing these benefits, as Ben Frain explores in his [article](https://benfrain.com/dealing-with-state-in-ecss-web-applications-using-aria/), we can drop stateful classes and instead rely on ARIA attributes as our CSS hooks for styling some component state:

> Adopting this approach results in what is (cringingly) referred to as a â€˜Win Winâ€™; situation. We get to improve the accessibility of our web application, while also gaining a clearly defined, well considered lexicon for communicating the states we need in our application logic.

For example, instead of:

```css
.c-accordion.is-active .c-accordion__content {
  [...]
}
```

We would have:

```css
.c-accordion[aria-expanded="true"] .c-accordion__content {
  [...]
}
```

Coming back to our reusable function, we'll build in support so the `data-class` attribute can also accept an ARIA attribute reference. Since we're now manipulating attributes rather than just classes, it would make sense semantically to rename `data-class` and all of its associated attributes to `data-state`:

```html
<div class="c-mycomponent" data-state="aria-expanded" data-state-element="c-mycomponent" aria-expanded="false" tabindex="0">
```

In the above example, clicking `c-mycomponent` should toggle `aria-expanded` on itself. Whilst in the below example, in addition to the previous behaviour `my-class` would be removed from `c-myothercomponent`.

```html
<div class="c-mycomponent" data-state="aria-expanded, my-class" data-state-element="c-mycomponent, c-myothercomponent" data-state-behaviour="toggle, remove" aria-expanded="false" tabindex="0">
```

In addition to `aria-expanded`, other examples of how ARIA attributes could be used instead of stateful classes are:

- `aria-disabled="true"` instead of `is-disabled`
- `aria-checked="true"` instead of `is-checked`
- `aria-pressed="true"` or `aria-selected="true"` instead of `is-active`

Here's a [handy ARIA cheatsheet](http://karlgroves-sandbox.com/CheatSheets/ARIA-Cheatsheet.html) which came in handy whilst researching this article.

#### Implementation
Our reusable function currently assumes that everything passed to it via our newly renamed `data-state` attribute is a class. It then acts accordingly based on either what's defined in `data-state-behaviour` or its default `toggle` behaviour:

```javascript
// Cycle through target elements
for(var c = 0; c < elemRef.length; c++) {
  if(elemBehaviour === "add") {
    if(!elemRef[c].classList.contains(elemClass)) {
      elemRef[c].classList.add(elemClass);
    }
  }
  else if(elemBehaviour === "remove") {
    if(elemRef[c].classList.contains(elemClass)) {
      elemRef[c].classList.remove(elemClass);
    }
  }
  else {
    elemRef[c].classList.toggle(elemClass);
  }
}
```

Let's tweak this slightly:

```javascript
// Cycle through target elements
for(var c = 0; c < elemRef.length; c++) {
    // Find out if we're manipulating aria-attributes or classes
    var toggleAttr;
    if(elemRef[c].getAttribute(elemState)) {
        toggleAttr = true;
    }
    else {
        toggleAttr = false;
    }
    if(elemBehaviour === "add") {
        if(toggleAttr) {
            elemRef[c].setAttribute(elemState, true);
        }
        else {
            elemRef[c].classList.add(elemState);
        }
    }
    else if(elemBehaviour === "remove") {
        if(toggleAttr) {
            elemRef[c].setAttribute(elemState, false);
        }
        else {
            elemRef[c].classList.remove(elemState);
        }
    }
    else {
        if(toggleAttr) {
            if(elemRef[c].getAttribute(elemState) === "true") {
                elemRef[c].setAttribute(elemState, false);
            }
            else {
                elemRef[c].setAttribute(elemState, true);
            }
        }
        else {
            elemRef[c].classList.toggle(elemState);
        }
    }
}
```

To support ARIA attributes, we've simply added a check to first see if the given ARIA attribute exists on the element, and if not, assume it's a class and process it as before. This way we can support both ARIA attributes and classes to cover all eventualities. Also removed are the `classList.contains()` checks, as in current spec `classList.add()` and `classList.remove()` are smart enough to account for this.


### Keyboard Events

For a website to be considered accessible, it's important that it can be easily navigated and interacted with through just the use of a keyboard. As far as the developer is concerned, this often involves the usage of the `tabindex` attribute and leveraging keyboard events.

In most browsers, elements such as the anchor already have these properties by default. You can tab to them and when in focus they can be activated on the press of the enter key. However for many components built with a combination of semantic elements and divs, this is not the case.

Let's make our reusable function pick up this slack by writing logic to automatically add a keyboard event to the trigger element so it can be activated - like an anchor - with a press of the enter key.


#### Implementation
At the moment, as the function logic is triggered by clicking an element with `data-state` and `data-state-element` attributes, everything is wrapped in a `click` event listener:

```javascript
elems[i].addEventListener("click", function(e){
  // Function logic
});
```

As a press of the enter key is going to need to trigger the same function logic as a click, it makes sense to seperate out this logic into it's own function so it can be triggered from either. We'll call it `processChange()`:

```javascript
// Assign click event
elem.addEventListener("click", function(e){
    // Prevent default action of element
    e.preventDefault(); 
    // Run state function
    processChange(this);
});
// Add keyboard event for enter key to mimic anchor functionality
elem.addEventListener("keypress", function(e){
    // e.which refers to the key pressed, 13 being the enter key.
    if(e.which === 13) {
        // Prevent default action of element
        e.preventDefault();
        // Run state function
        processChange(this);
    }
});
```

In addition to the existing `click` event listener, we've added an extra listener to react when the enter key is pressed. When a matching `keypress` event occurs on a focused trigger element, it's just a matter of running our new `processChange()` function and passing the element. 

You will also notice there's no logic to automatically add a `tabIndex` attribute. This is because it may conflict with any `tabIndex` hierarchy already defined on the page and interfere with developer intent.

### Example
Here's a modified version of the accordion example from [Part 1](https://css-tricks.com/managing-state-css-reusable-javascript-functions/), but fully updated to leverage ARIA attributes and keyboard events to make it a more accessible component. You can see the full reusable function as it now stands in the JavaScript panel.

<p data-height="350" data-theme-id="5799" data-slug-hash="KmRwBP" data-default-tab="result" data-user="lukedidit" data-embed-version="2" data-pen-title="#7) Accessibility example" class="codepen">See the Pen <a href="http://codepen.io/lukedidit/pen/KmRwBP/">#7) Accessibility example</a> by Luke Harrison (<a href="http://codepen.io/lukedidit">@lukedidit</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

## Accounting for elements added to DOM later
In [Part 1](https://css-tricks.com/managing-state-css-reusable-javascript-functions/), an issue was raised in the comments section:

> I think this would have some issues for elements added to the DOM at a later point. In that case you would need to repeat assigning the click eventâ€¦ Am I correct?

That is correct! Any elements with `data-state` and `data-state-element` attributes added after the initial render of the DOM won't have any event listeners assigned to them. So when they are clicked or swiped, nothing will happen.

Why? This is because in our Javascript, once the initial round of assigning event listeners to elements with `data-state` and `data-state-element` is complete, there's no functionality in place to say "Hey! Watch out for any new elements with`data-state` and `data-state-element` attributes and make them work."

### Implementation
To fix this, we'll leverage something called MutationObservers. Whilst they can be explained much better in [David Walsh's great overview of the API](https://davidwalsh.name/mutationobserver-api), MutationObservers essentially allow us to track any nodes added or removed from the DOM (also known as DOM mutations). 

We can setup one up like so:

```javascript
// Setup mutation observer to track changes for matching elements added after initial DOM render
var observer = new MutationObserver(function(mutations) {
    mutations.forEach(function(mutation) {
        for(var d = 0; d < mutation.addedNodes.length; d++) {
            // Check if we're dealing with an element node
            if(typeof mutation.addedNodes[d].getAttribute === 'function') {
                if(mutation.addedNodes[d].getAttribute("data-state") && mutation.addedNodes[d].getAttribute("data-state-element")) {
                     // Create click and keyboard event listeners etc
                }
            }
        }
    });  
});

// Define type of change our observer will watch out for
observer.observe(document.body, {
  childList: true,
  subtree: true
});
```

This is what our MutationObserver is doing:

1. Recording any DOM mutation to the body element which are immediate children `childList: true` or its decendents `subtree: true`
2. Checking if that DOM mutation is a new element node, rather than a text node
3. If true, then check if the new element node has `data-state` and `data-state-element` attributes

The next step, assuming these 3 checks pass, would be to setup our `click` and `keypress` event listeners. Like with the implementation of keyboard events, let's seperate out this setup logic into its own function so we can reuse it both on page load and when an element with `data-state` and `data-state-element` attributes is detected by our MutationObserver. 

We'll call this new function `initDataState()`.

```javascript
// Init function
initDataState = function(elem){
  // Add event listeners to each one
  elems.addEventListener("click", function(e){
    // Prevent default action of element
    e.preventDefault();
    // Run state function
    processChange(this);
  });    
  // Add keyboard event for enter key to mimic anchor functionality
  elems.addEventListener("keypress", function(e){
      if(e.which === 13) {
          // Prevent default action of element
          e.preventDefault();
          // Run state function
          processChange(this);
      }
  });
}
```
Then it's just a matter of hooking everything up correctly:

```javascript
// Run when DOM has finished loading
document.addEventListener("DOMContentLoaded", function() {

  // Grab all elements with required attributes
  var elems = document.querySelectorAll("[data-state][data-state-element]");

  // Loop through if any are found
  for(var a = 0; a < elems.length; b++){
    initDataState(elems[a]);
  }

  // Setup mutation observer to track changes for matching elements added after initial DOM render
  var observer = new MutationObserver(function(mutations) {
      mutations.forEach(function(mutation) {
          for(var d = 0; d < mutation.addedNodes.length; d++) {
              // Check if we're dealing with an element node
              if(typeof mutation.addedNodes[d].getAttribute === 'function') {
                  if(mutation.addedNodes[d].getAttribute("data-state")) {
                       initDataState(mutation.addedNodes[d]);
                  }
              }
          }
      });  
  });

  // Define type of change our observer will watch out for
  observer.observe(document.body, {
      childList: true,
      subtree: true
  });
});
```
### Example
Click the "Add" button to insert more elements into the page (example Pen below):

<p data-height="350" data-theme-id="5799" data-slug-hash="jmzLob" data-default-tab="result" data-user="lukedidit" data-embed-version="2" data-pen-title="#8) Correctly set up new data-class elements when they are added to the DOM" class="codepen">See the Pen <a href="http://codepen.io/lukedidit/pen/jmzLob/">#8) Correctly set up new data-class elements when they are added to the DOM</a> by Luke Harrison (<a href="http://codepen.io/lukedidit">@lukedidit</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

## Swipe support
At the moment, our reusable function is using click and keyboard events to trigger state logic. This is fine at desktop level, but on touch devices for some UI components (Such as closing a sliding navigation menu for instance) it's often more useful to have this logic trigger on detection of a swipe.

Let's build in optional swipe support for our reusable function. This will require adding a new data attribute to compliment our existing set:

### `data-state-swipe`
The purpose of this new attribute is to allow us to define the swipe direction which should trigger our state logic. These directions should be:

- `up`
- `right`
- `down`
- `left`

Let's also build in the option to specify if or not the swipe event should replace the click event, or if both should coexist. We can add a comma separated boolean to `data-state-swipe` to trigger this behaviour:

- `true` - Swipe event listener replaces click event listener
- `false` - Swipe event listener and click event listener are both added (Default)

So for example, when the `div` below detects a left swipe, the `aria-expanded` attribute on `js-elem` would be changed to `true`. The swipe event listener would also in this instance replace the click event listener, as we're passing `true` in `data-state-swipe`:

```html
<div data-state="aria-expanded" data-state-element="js-elem" data-state-swipe="left, true" data-state-behaviour="add">
```

Now let's make the changes. 


### Implementation
Swipe is managed in just the same way as you would manage clicks and keyboard input - via event listeners. To keep the article focused on our reusable function, I'll be using a helper function called `swipeDetect()` which will handle all the calculations required for accurate swipe detection. However feel free to use your own preferred method of detecting swipe direction in place of it.

We're building swipe into our reusable function as another way of triggering function logic, so it makes sense that it should sit with the click and keyboard event listeners in `initDateState()` and then trigger `processChange()` once our requirements for a desired swipe direction are met.

Though we also have to account for the behaviour flag passed in `data-state-swipe` that determines if swipe should replace click. Let's refactor `initDataState()` to add some scaffolding to properly support all of this:

```javascript
// Init function
initDataState = function(elem){
    // Detect data-swipe attribute before we do anything, as its optional
    // If not present, assign click event like before
    if(elem.getAttribute("data-state-swipe")){
        // Grab swipe specific data from data-state-swipe   
        var elemSwipe = elem.getAttribute("data-state-swipe"),
              elemSwipe = elemSwipe.split(", "),
              swipeDirection = elemSwipe[0],
              elemSwipeBool = elemSwipe[1],
              currentElem = elem;

        // If the behaviour flag is set to "false", or not set at all, then assign our click event
        if(elemSwipeBool === "false" || !elemSwipeBool) {
            // Assign click event
            elem.addEventListener("click", function(e){
                // Prevent default action of element
                e.preventDefault(); 
                // Run state function
                processChange(this);
            });
        }
        // Use our swipeDetect helper function to determine if the swipe direction matches our desired direction
        swipeDetect(elem, function(swipedir){
            if(swipedir === swipeDirection) {
                // Run state function
                processChange(currentElem);
            }
        })
    }
    else {
        // Assign click event
        elem.addEventListener("click", function(e){
            // Prevent default action of element
            e.preventDefault(); 
            // Run state function
            processChange(this);
        });
    }
    // Add keyboard event for enter key to mimic anchor functionality
    elem.addEventListener("keypress", function(e){
        if(e.which === 13) {
            // Prevent default action of element
            e.preventDefault();
            // Run state function
            processChange(this);
        }
    });
};
```

These amends to `initDataState` now give it 3 different outcomes:

1. If there's a `data-state-swipe` attribute on the trigger element, and its behaviour boolean is set to `true`, then only swipe and keyboard events are assigned.
2. If there's a `data-state-swipe` attribute on the trigger element, but its behaviour boolean is set to `false`, then swipe, click and keyboard events are all assigned.
3. If there's **no** `data-state-swipe` attribute on the trigger element altogether, only click and keyboard event listeners are assigned.

### Example
Here's a very barebones example of the new swipe functionality in practice. Click the button to toggle the menu and then swipe right on the menu whilst on a touch device (or your preferred browser inspector) to close it. Simple.

<p data-height="350" data-theme-id="5799" data-slug-hash="GmQyyz" data-default-tab="result" data-user="lukedidit" data-embed-version="2" data-pen-title="#9) Adding swipe support to our reusable function" class="codepen">See the Pen <a href="http://codepen.io/lukedidit/pen/GmQyyz/">#9) Adding swipe support to our reusable function</a> by Luke Harrison (<a href="http://codepen.io/lukedidit">@lukedidit</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

## Function Refinements
Finally, we'll be looking into ways we can refine our reusable function to make it more efficient and easier to use.

### Targeting the trigger element
Say for example I have an element named `c-btn` which on click would need to toggle `aria-pressed` on itself. With our reusable function as it stands, the HTML would look something like this:

```html
<button class="c-btn" data-state="aria-pressed" data-state-element="c-btn" aria-pressed="false">
```

The problem here is that on click, `aria-pressed` would be toggled on all instances of `c-btn` everywhere, which isn't the behaviour we're looking for.

This was the problem which `data-state-scope` was created to resolve. By scoping our `data-state` instance to the nearest `c-btn` (which in this case would be itself) then we are creating the desired toggle behaviour.

```html
<button class="c-btn" data-state="aria-pressed" data-state-element="c-btn" data-class-scope="c-btn" aria-pressed="false">
```

Whilst the above snippet works fine, it's a bit jarring having all these attributes all referencing the same `c-btn` element. Ideally if `data-state-element` and `data-state-scope` aren't defined, then the function should default to the element triggering it. This would allow easy targeting of our trigger element. Like so:

```html
<button class="c-btn" data-state="aria-pressed" aria-pressed="false">
```

#### Implementation
`data-scope-element` is currently a required attribute. If it isn't present, the function will not be able to assign any event listeners. This is because in our reusable function as it stands, the initial scan of the document is looking for elements with both the `data-scope` **and** `data-scope-element` attributes:

```javascript
// Grab all elements with required attributes
var elems = document.querySelectorAll("[data-state][data-state-element]");
```

We need to tweak this so we're only looking for elements with `data-state`, as `data-state-element` will shortly be relegated to an optional attribute.

```javascript
// Grab all elements with required attributes
var elems = document.querySelectorAll("[data-state]");
```

In addition, we need to add an if-statement to `processChange()` which wraps around the retrieval of the `data-state-element` value, as if it's not present, the function will throw an error when trying to call `getAttribute()` on something which doesn't exist.

```javascript
// Grab data-state-element list and convert to array
if(elem.getAttribute("data-state-element")) {
  var dataStateElement = elem.getAttribute("data-state-element");
  dataStateElement = dataStateElement.split(", ");
}
```

Next, let's implement the logic which makes `data-state-element` and `data-state-scope` default to the trigger element if they are not explicitly defined. We can build on our previous amends to `processChange()` and add an `else` block to our `data-state-element` check to manually declare our target element and scope.

```javascript
// Grab data-state-element list and convert to array
// If data-state-element isn't found, pass self, set scope to self if none is present, essentially replicating "this"
if(elem.getAttribute("data-state-element")) {
  var dataStateElement = elem.getAttribute("data-state-element");
  dataStateElement = dataStateElement.split(", ");
}
else {
  var dataStateElement = [];
  dataStateElement.push(elem.classList[0]);
  if(!dataStateScope) {
    var dataStateScope = dataStateElement;
  }
}
```

Another consequence of making `data-state-element` no longer required is that in `processChange()`, it's length is used in the `for` loop to make sure all the elements defined in `data-state-element` receive their changes of state.  This is the loop as it stands:

```javascipt
// Loop through all our dataStateElement items
for(var b = 0; b < dataStateElement.length; b++) {
  [...]
}
```

Thankfully, all we need to do here is swap out our now optional `data-state-element` attribute for our still required `data-state` element as the base for this loop. 

```javascipt
// Loop through all our dataStateElement items
for(var b = 0; b < dataState.length; b++) {
  [...]
}
```

This is because in instances where multiple values have been passed to each `data-state` attribute (For example: `<div data-state="is-active, is-disabled" data-state-element="my-elem, my-elem-2" data-state-behaviour="add, remove">`) the length of the array which is derived from each of these is always going to match, so we're always going to get the same amount of loops in the `for` block.

### Simplifying repeated values

Another improvement which we could make relates to assigning similar types of logic in a single `data-state` use. Consider the below example:

```html
<a data-state="my-state, my-state-2, my-state-3" data-state-element="c-btn, c-btn, c-btn" data-state-behaviour="remove, remove, remove" data-state-scope="o-mycomponent, o-mycomponent, o-mycomponent">
```

Whilst this a legitimate use of our reusable function, one thing you will notice is that we have a lot of repeated values in many of the `data-state` attributes. Ideally, if we want to assign similar types of logic many times over, we should be able to just write a value once and have the function interpret that as repeat values.

For example, the below HTML snippet should perform the same action as the one above.

```html
<a data-state="my-state, my-state-2, my-state-3" data-state-element="c-btn" data-state-behaviour="remove" data-state-scope="o-mycomponent">
```

Here is another example of what should be considered a valid `data-state` use:

```html
<a data-state="aria-expanded" data-state-element="c-menu, c-other-menu, c-final-menu" data-state-behaviour="remove, add">
```


#### Implementation
The first thing we need to consider is the `for` loop in `processChange()` which we last amended in the previous section. As it uses the length of `data-state` as a base for it's loop amount, implementing these changes would expose a bug in scenarios where we have one class being applied to many elements.

Consider the following:

```html
<a data-state="my-state" data-state-element="c-btn, c-btn-2" data-state-behaviour="remove" data-state-scope="o-mycomponent">
```

What would happen here is because `data-state` only has a single value, the `for` loop in `processChange()` would only loop a single time, meaning our intended logic for `c-btn-2` would never be assigned.

To fix this, we need to compare `data-state` and `data-state-element`. Whichever has the most values then becomes the base for our loop. Like so:

```javascript
// Find out which has the biggest length between states and elements and use that length as loop number
// This is to make sure situations where we have one data-state-element value and many data-state values are correctly setup
var dataLength = Math.max(dataStateElement.length, dataState.length);

// Loop
for(var b = 0; b < dataLength; b++) {
  [...]
}
```

As for the rest of the implementation, it's now a matter of adding logic in the `for` loop for each attribute which says "If a matching value can't be found, use the last valid one".

Let's use `data-state` value as an example. Currently, the code in the `for` loop which grabs the state value looks like:

```javascript
// Grab state we will add
var elemState = dataState[b];
```

The problem now is if we have 3 `data-state-element` values, but only 1 `data-state` value, on loops 2 and 3 `elemState` would be `undefined`. 

What we need to do is only redefine `elemState` if we have a value to give it. Like so:

```javascript
// Grab state we will add
// If one isn't found, keep last valid one
if(dataState[b] !== undefined) {
  var elemState = dataState[b];
}
```

This would ensure `elemState` would always have a value, including inheriting any previous values if one can't initially be found.

### Example
Here's a final example showing all of our function refinements:

<p data-height="350" data-theme-id="5799" data-slug-hash="RVQJZw" data-default-tab="result" data-user="lukedidit" data-embed-version="2" data-pen-title="#10) Allow easier targeting of self & general improvements" class="codepen">See the Pen <a href="http://codepen.io/lukedidit/pen/RVQJZw/">#10) Allow easier targeting of self & general improvements</a> by Luke Harrison (<a href="http://codepen.io/lukedidit">@lukedidit</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

## Closing

In this article, we've covered how to build upon the reusable function created in [Part 1](https://css-tricks.com/managing-state-css-reusable-javascript-functions/) to make it more accessible and easier to use.

In addition, we've also added swipe support for trigger elements and have made sure any `data-state` elements added after the initial render of the DOM are no longer ignored.

As before, any comments or constructive feedback are welcome. I'll leave you will the full reusable function which we've developed over the last 2 articles:

```javascript
(function(){

  // SWIPE DETECT HELPER
  //----------------------------------------------

  var swipeDetect = function(el, callback){ 
    var touchsurface = el,
    swipedir,
    startX,
    startY,
    dist,
    distX,
    distY,
    threshold = 100, //required min distance traveled to be considered swipe
    restraint = 100, // maximum distance allowed at the same time in perpendicular direction
    allowedTime = 300, // maximum time allowed to travel that distance
    elapsedTime,
    startTime,
    eventObj,
    handleswipe = callback || function(swipedir, eventObj){}

    touchsurface.addEventListener('touchstart', function(e){
      var touchobj = e.changedTouches[0]
      swipedir = 'none'
      dist = 0
      startX = touchobj.pageX
      startY = touchobj.pageY
      startTime = new Date().getTime() // record time when finger first makes contact with surface
      eventObj = e;
    }, false)

    touchsurface.addEventListener('touchend', function(e){
      var touchobj = e.changedTouches[0]
      distX = touchobj.pageX - startX // get horizontal dist traveled by finger while in contact with surface
      distY = touchobj.pageY - startY // get vertical dist traveled by finger while in contact with surface
      elapsedTime = new Date().getTime() - startTime // get time elapsed
      if (elapsedTime <= allowedTime){ // first condition for awipe met
        if (Math.abs(distX) >= threshold && Math.abs(distY) <= restraint){ // 2nd condition for horizontal swipe met
          swipedir = (distX < 0)? 'left' : 'right' // if dist traveled is negative, it indicates left swipe
        }
        else if (Math.abs(distY) >= threshold && Math.abs(distX) <= restraint){ // 2nd condition for vertical swipe met
          swipedir = (distY < 0)? 'up' : 'down' // if dist traveled is negative, it indicates up swipe
        }
      }
      handleswipe(swipedir, eventObj)
    }, false)
  }


  // CLOSEST PARENT HELPER FUNCTION
  //----------------------------------------------

  closestParent = function(child, match) {
    if (!child || child == document) {
      return null;
    }
    if (child.classList.contains(match) || child.nodeName.toLowerCase() == match) {
      return child;
    }
    else {
      return closestParent(child.parentNode, match);
    }
  }


  // REUSABLE FUNCTION
  //----------------------------------------------

  // Change function
  processChange = function(elem){

    // Grab data-state list and convert to array
    var dataState = elem.getAttribute("data-state");
    dataState = dataState.split(", ");

    // Grab data-state-behaviour list if present and convert to array
    if(elem.getAttribute("data-state-behaviour")) {
      var dataStateBehaviour = elem.getAttribute("data-state-behaviour");
      dataStateBehaviour = dataStateBehaviour.split(", ");
    }

    // Grab data-scope list if present and convert to array
    if(elem.getAttribute("data-state-scope")) {
      var dataStateScope = elem.getAttribute("data-state-scope");
      dataStateScope = dataStateScope.split(", ");
    }

    // Grab data-state-element list and convert to array
    // If data-state-element isn't found, pass self, set scope to self if none is present, essentially replicating "this"
    if(elem.getAttribute("data-state-element")) {
      var dataStateElement = elem.getAttribute("data-state-element");
      dataStateElement = dataStateElement.split(", ");
    }
    else {
      var dataStateElement = [];
      dataStateElement.push(elem.classList[0]);
      if(!dataStateScope) {
        var dataStateScope = dataStateElement;
      }
    }

    // Find out which has the biggest length between states and elements and use that length as loop number
    // This is to make sure situations where we have one data-state-element value and many data-state values are correctly setup
    var dataLength = Math.max(dataStateElement.length, dataState.length);

    // Loop
    for(var b = 0; b < dataLength; b++) {

      // If a data-state-element value isn't found, use last valid one
      if(dataStateElement[b] !== undefined) {
        var dataStateElementValue = dataStateElement[b];
      } 

      // If scope isn't found, use last valid one
      if(dataStateScope && dataStateScope[b] !== undefined) {
        var cachedScope = dataStateScope[b];
      }
      else if(cachedScope) {
        dataStateScope[b] = cachedScope;
      }

      // Grab elem references, apply scope if found
      if(dataStateScope && dataStateScope[b] !== "false") {

        // Grab parent
        var elemParent = closestParent(elem, dataStateScope[b]);

        // Grab all matching child elements of parent
        var elemRef = elemParent.querySelectorAll("." + dataStateElementValue);

        // Convert to array
        elemRef = Array.prototype.slice.call(elemRef);

        // Add parent if it matches the data-state-element and fits within scope
        if(elemParent.classList.contains(dataStateElementValue)) {
          elemRef.unshift(elemParent);
        }
      }
      else {
        var elemRef = document.querySelectorAll("." + dataStateElementValue);
      }
      // Grab state we will add
      // If one isn't found, keep last valid one
      if(dataState[b] !== undefined) {
        var elemState = dataState[b];
      }   
      // Grab behaviour if any exists
      // If one isn't found, keep last valid one
      if(dataStateBehaviour) {
        if(dataStateBehaviour[b] !== undefined) {
          var elemBehaviour = dataStateBehaviour[b];
        }
      }
      // Do
      for(var c = 0; c < elemRef.length; c++) {
        // Find out if we're manipulating aria-attributes or classes
        var toggleAttr;
        if(elemRef[c].getAttribute(elemState)) {
          toggleAttr = true;
        }
        else {
          toggleAttr = false;
        }
        if(elemBehaviour === "add") {
          if(toggleAttr) {
            elemRef[c].setAttribute(elemState, true);
          }
          else {
            elemRef[c].classList.add(elemState);
          }
        }
        else if(elemBehaviour === "remove") {
          if(toggleAttr) {
            elemRef[c].setAttribute(elemState, false);
          }
          else {
            elemRef[c].classList.remove(elemState);
          }
        }
        else {
          if(toggleAttr) {
            if(elemRef[c].getAttribute(elemState) === "true") {
              elemRef[c].setAttribute(elemState, false);
            }
            else {
              elemRef[c].setAttribute(elemState, true);
            }
          }
          else {
            elemRef[c].classList.toggle(elemState);
          }
        }
      }

    }

  },
    // Init function
    initDataState = function(elem){
    // Detect data-swipe attribute before we do anything, as its optional
    // If not present, assign click event like before
    if(elem.getAttribute("data-state-swipe")){
      // Grab swipe specific data from data-state-swipe
      var elemSwipe = elem.getAttribute("data-state-swipe"),
          elemSwipe = elemSwipe.split(", "),
          direction = elemSwipe[0],
          elemSwipeBool = elemSwipe[1],
          currentElem = elem;

      // If the behaviour flag is set to "false", or not set at all, then assign our click event
      if(elemSwipeBool === "false" || !elemSwipeBool) {
        // Assign click event
        elem.addEventListener("click", function(e){
          // Prevent default action of element
          e.preventDefault(); 
          // Run state function
          processChange(this);
        });
      }
      // Use our swipeDetect helper function to determine if the swipe direction matches our desired direction
      swipeDetect(elem, function(swipedir){
        if(swipedir === direction) {
          // Run state function
          processChange(currentElem);
        }
      })
    }
    else {
      // Assign click event
      elem.addEventListener("click", function(e){
        // Prevent default action of element
        e.preventDefault(); 
        // Run state function
        processChange(this);
      });
    }
    // Add keyboard event for enter key to mimic anchor functionality
    elem.addEventListener("keypress", function(e){
      if(e.which === 13) {
        // Prevent default action of element
        e.preventDefault();
        // Run state function
        processChange(this);
      }
    });
  };

  // Run when DOM has finished loading
  document.addEventListener("DOMContentLoaded", function() {

    // Grab all elements with required attributes
    var elems = document.querySelectorAll("[data-state]");

    // Loop through our matches and add click events
    for(var a = 0; a < elems.length; a++){
      initDataState(elems[a]);
    }

    // Setup mutation observer to track changes for matching elements added after initial DOM render
    var observer = new MutationObserver(function(mutations) {
      mutations.forEach(function(mutation) {
        for(var d = 0; d < mutation.addedNodes.length; d++) {
          // Check if we're dealing with an element node
          if(typeof mutation.addedNodes[d].getAttribute === 'function') {
            if(mutation.addedNodes[d].getAttribute("data-state")) {
              initDataState(mutation.addedNodes[d]);
            }
          }
        }
      });    
    });

    // Define type of change our observer will watch out for
    observer.observe(document.body, {
      childList: true,
      subtree: true
    });
  });
}());
```