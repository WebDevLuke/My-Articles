<!-- 
Managing state in OOCSS with reusable JavaScript functions
Posted on XXXXXX
-->

# Managing state in OOCSS with reusable JavaScript functions

Determining the most efficient way of managing component state is a common issue in [OOCSS](https://www.smashingmagazine.com/2011/12/an-introduction-to-object-oriented-css-oocss/), but thankfully there are many methodologies out there which provide some good solutions. My preferred solution comes from [SMACSS (Scalable and modular architecture for CSS)](https://smacss.com/) and involves stateful classes. To quote SMACSS's [own documentation](https://smacss.com/book/type-state), stateful classes are:

> A state is something that augments and overrides all other styles. For example, an accordion section may be in a collapsed or expanded state. A message may be in a success or error state.
> 
> States are generally applied to the same element as a layout rule or applied to the same element as a base module class.

One of my most used stateful classes is `is-active`. Taking the accordion example from the prior quote, `is-active` would apply all the required CSS styles to represent an expanded state. As seen in the example below:

<iframe height='300' scrolling='no' title='Accordion Component w Stateful Class' src='//codepen.io/lukedidit/embed/mmJRQP/?height=300&theme-id=5799&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/lukedidit/pen/mmJRQP/'>Accordion Component w Stateful Class</a> by Luke Harrison (<a href='http://codepen.io/lukedidit'>@lukedidit</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>


You will notice there's some JavaScript which toggles the `is-active` class on the component when a click event is detected:

```javascript
var accordion = document.querySelectorAll(".c-accordion");

for(var i = 0; i < accordion.length; i++) {
	var accordionHeader = accordion[i].querySelector(".c-accordion__header"),
	accordionCurrent = accordion[i];
	
	accordionHeader.addEventListener("click", function(){
		accordionCurrent.classList.toggle("is-active");
	});
}
```

Whilst valid, this would have to be repeated again and again for any other components which leverage the `is-active` stateful class via a click event, leading to many duplicates of what is essentially the same code snippet.

Not very efficient and certainly not very DRY.

A better approach would be instead to write a single function which performs the same task and can be reused over and over again. Let's do that.

## Creating a simple reusuable function

Let's start off by building a simple function which accepts an element as a parameter and toggles `is-active`:


```javascript
var makeActive = function(elem){
	elem.classList.toggle("is-active");
}
```

This works fine, but if we slot it into our accordion JavaScript there's a problem:

```javascript
var accordion = document.querySelectorAll(".c-accordion"),
		accordionHeader,
		makeActive = function(elem){
			elem.classList.toggle("is-active");
		}

for(var i = 0; i < accordion.length; i++) {
	accordionHeader = accordion[i].querySelector(".c-accordion__header");
	accordionCurrent = accordion[i];
	
	accordionHeader.addEventListener("click", function(){
		makeActive(accordionCurrent);
	});
}
```

Although the `makeActive` function is reusable, we still need to first write code to grab our component and any of its inner elements, so there's certainly lots of room for improvement.

To make these improvements, we can leverage [HTML5 custom data attributes](http://html5doctor.com/html5-custom-data-attributes/):

```html
<div class="c-accordion js-accordion">
	<div class="c-accordion__header" data-active="js-accordion">My Accordion Component</div>
	<div class="c-accordion__content-wrapper">
		<div class="c-accordion__content">
			Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce laoreet ultricies risus, sit amet congue nulla mollis et. Suspendisse bibendum eros sed sem facilisis ornare. Donec sit amet erat vel dui semper pretium facilisis eget nisi. Fusce consectetur vehicula libero vitae faucibus. Nullam sed orci leo. Fusce dapibus est velit, at maximus turpis iaculis in. Pellentesque ultricies ultrices nisl, eu consequat est molestie sit amet. Phasellus laoreet magna felis, ut vulputate justo tempor eu. Nam commodo aliquam vulputate.
		</div>
	</div>
</div>
```

A `data-active` attribute has been added to the element which previously triggered the `is-active` toggle when clicked. This attribute's value represents the element where the `is-active` toggle should take place, which as before is the top-level `c-accordion` element. Note the addition of a new `js-accordion` class rather than hooking into the existing `c-accordion` class. This is generally regarded as good practice as we are decoupling functional aspects of the component from it's styling.

Let's take a look at the JavaScript:

```javascript
// Grab all elements with data-active attribute
var elems = document.querySelectorAll("[data-active]");

// Loop through if any are found
if(elems.length){
  for(var i = 0; i < elems.length; i++){
    // Add event listeners to each one
    elems[i].addEventListener("click", function(e){

      // Prevent default action of element
      e.preventDefault();

      // Grab linked elements
      var linkedElement = document.querySelectorAll("." + this.getAttribute("data-active"));

      // Toggle linked element if present
      if(linkedElement.length) {
        for(var i = 0; i < linkedElement.length; i++) {
          linkedElement[i].classList.toggle("is-active");
        }
      }
      
    });    
  }
}
```

This has certainly improved things as we no longer have to write code to grab any required elements like before, just attach a data-active attribute to our trigger element and specify a target element. As it stands, this function can be used for any other component where a click-based `is-active` class is required without any additional coding. Full example below:

<iframe height='300' scrolling='no' title='Accordion Component w Reusable is-active function' src='//codepen.io/lukedidit/embed/zwGdqV/?height=300&theme-id=5799&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/lukedidit/pen/zwGdqV/'>Accordion Component w Reusable is-active function</a> by Luke Harrison (<a href='http://codepen.io/lukedidit'>@lukedidit</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

## Improving our reusable function

Whilst the reusable function works, when scaled we have to take care to make sure trigger and target element classes don't conflict with one another. In the example below, clicking one accordian would trigger `is-active` on all of them. 

```html
<div class="c-accordion js-accordion">
	<div class="c-accordion__header" data-active="js-accordion">First Accordion</div>
	[...]
</div>

<div class="c-accordion js-accordion">
	<div class="c-accordion__header" data-active="js-accordion">Second Accordion</div>
	[...]
</div>

<div class="c-accordion js-accordion">
	<div class="c-accordion__header" data-active="js-accordion">Third Accordion</div>
	[...]
</div>
```

Adding number suffixes to each `js-accordion` reference does solve the problem, but it's a hassle which we can do without. A good solution would be to instead implement  scoping to our reusable function which would enable us to encapulate our toggles so they only effect the elements we want.

To implement scoping, we'll need to create a seperate custom attribute called `data-active-scope`. It's value should represent the element which the toggle should be encapculated within, which in this instance is the parent `js-accordion` element.

```html
<div class="c-accordion js-accordion">
	<div class="c-accordion__header" data-active="js-accordion" data-active-scope="js-accordion">First Accordion</div>
	[...]
</div>

<div class="c-accordion js-accordion">
	<div class="c-accordion__header" data-active="js-accordion">Second Accordion</div>
	[...]
</div>
```

Using the above HTML, the following behaviour should happen:

1. When you click the first accordion, because it has a scope set to `js-accordion`, only elements which match or are children of that instance should be modified.
2. When you click the second accordion, which doesn't have a scope, `is-active` would be toggled on all instances of `js-accordion`.

Here's the modified Javascript and a working example:

```javascript
// Grab all elements with data-active attribute
var elems = document.querySelectorAll("[data-active]"),
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
if(elems.length){
	for(var i = 0; i < elems.length; i++){
		// Add event listeners to each one
		elems[i].addEventListener("click", function(e){

			// Prevent default action of element
			e.preventDefault();

			// Grab scope if defined
			if(this.getAttribute("data-active-scope")) {
				var scopeElement = closestParent(this, this.getAttribute("data-active-scope"));
			}

			if(scopeElement) {
				// Grab scoped linked element
				var linkedElement = scopeElement.querySelectorAll("." + this.getAttribute("data-active"));
				// Convert to array
				linkedElement = Array.prototype.slice.call(linkedElement);
				// Check if our scope matches our target element and add to array if true.
				// This is to make sure everything works when data-active matches data-active-scope.
				if(scopeElement.classList.contains(this.getAttribute("data-active"))) {
					linkedElement.unshift(scopeElement);
				}
			}
			else {
				// Grab linked element
				var linkedElement = document.querySelectorAll("." + this.getAttribute("data-active"));
			}

			// Toggle linked element if present
			if(linkedElement.length) {
				for(var i = 0; i < linkedElement.length; i++) {
					linkedElement[i].classList.toggle("is-active");
				}
			}
			
		});    
	}
}
```

<iframe height='300' scrolling='no' title='#3) Accordion Component w Improved reusable is-active function' src='//codepen.io/lukedidit/embed/YVXEMy/?height=300&theme-id=5799&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/lukedidit/pen/YVXEMy/'>#3) Accordion Component w Improved reusable is-active function</a> by Luke Harrison (<a href='http://codepen.io/lukedidit'>@lukedidit</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>


## Moving beyond is-active
Our reusable function is now working nicely and is an efficient way of setting up `is-active` toggles on all kinds of components. However what if we need to set up a similar toggle for another stateful class? As it stands we would have to duplicate the function and change all references of `is-active` to the new stateful class. Not very efficient.

We should improve our reusuble function to accept any class, and to do so we'll have to refactor our data-attributes. Instead of attaching the `data-active` attribute to our trigger element, let's replace it with the following:

- `data-class` - The class we wish to add.
- `data-class-element` - The element we wish to add the class to.
- `data-class-scope` - The scope attributre performs the same function, but has been renamed for consistency.

This requires a few minor tweaks to our JavaScript:

```javascript
// Grab all elements with data-active attribute
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
if(elems.length){
    for(var i = 0; i < elems.length; i++){
        // Add event listeners to each one
        elems[i].addEventListener("click", function(e){

            // Prevent default action of element
            e.preventDefault();

            // Grab scope if defined
            if(this.getAttribute("data-class-scope")) {
                var scopeElement = closestParent(this, this.getAttribute("data-class-scope"));
            }

            if(scopeElement) {
                // Grab scoped linked element
                var linkedElement = scopeElement.querySelectorAll("." + this.getAttribute("data-class-element"));
                // Convert to array
                linkedElement = Array.prototype.slice.call(linkedElement);
                // Check if our scope matches our target element and add to array if true.
                // This is to make sure everything works when data-active matches data-active-scope.
                if(scopeElement.classList.contains(this.getAttribute("data-class-element"))) {
                    linkedElement.unshift(scopeElement);
                }
            }
            else {
                // Grab linked element
                var linkedElement = document.querySelectorAll("." + this.getAttribute("data-class-element"));
            }

            // Toggle linked element if present
            if(linkedElement.length) {
                for(var i = 0; i < linkedElement.length; i++) {
                    linkedElement[i].classList.toggle(this.getAttribute("data-class"));
                }
            }

        });    
    }
}
```

It would be set up in the HTML like so:

```html
<button class="c-button" data-class="is-loading" data-class-element="js-form-area">Submit</button>
```

In the example below, clicking the `c-button` component toggles the `is-loading` class on the `js-form-area` component:

<iframe height='300' scrolling='no' title='#4) Form Component w Improved reusable any class function' src='//codepen.io/lukedidit/embed/oWXowO/?height=300&theme-id=5799&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/lukedidit/pen/oWXowO/'>#4) Form Component w Improved reusable any class function</a> by Luke Harrison (<a href='http://codepen.io/lukedidit'>@lukedidit</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

## Handling multiple toggles

So we have a reusuable function which toggles any class on any element. These click events can be set up without having to write any additional JavaScript through the use of custom data attributes. However there's still ways to make this reusable function even more useful.

Coming back to our previous example of the login form component, what if when the `c-button` element is clicked, in addition to it toggling `is-loading` on `js-form-area` we also want to toggle `is-disabled` on all the input elements? At the moment this isn't possible as our custom attributes only accept a single value each.

Let's modify our function, so instead of each custom data attribute only accepting a single value, it accepts a comma seperated list of values - with each item value in `data-class` linking with the value of a matching index in `data-class-element` and `data-class-scope`. Like so:

```html
<button class="c-button" data-class="is-loading, is-disabled" data-class-element="js-form-area, js-input" data-class-scope="false, js-form-area">Submit</button>
```

Assuming the above is used, the following would happen once `c-button` is clicked:

1. `is-loading` would be toggled on `js-form-area`.
2. `is-disabled` would be toggled on `js-input` and be scoped within the parent `js-form-area` element.

This requires more changes to our JavaScript:

```javascript
// Grab all elements with data-active attribute
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
if(elems.length){
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
				// Do
				for(var c = 0; c < elemRef.length; c++) {
					elemRef[c].classList.toggle(elemClass);
				}
			}

		});    
	}
}
```

And here's another working example:

<iframe height='300' scrolling='no' title='#5) Form Component w Improved reusable + multiple any class function' src='//codepen.io/lukedidit/embed/oWjZej/?height=300&theme-id=5799&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/lukedidit/pen/oWjZej/'>#5) Form Component w Improved reusable + multiple any class function</a> by Luke Harrison (<a href='http://codepen.io/lukedidit'>@lukedidit</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

## Moving beyond toggle
Our reusable function is quite useful now, but it makes a presumption that toggling classes is the desired behavour. What if when clicked you want the trigger to remove a class if it's present and do nothing otherwise? Currently that's not possible.

To round the function off let's integrate a bit of extra logic to allow for this behaviour. We'll introduce an optional data-attribute called `data-class-behaviour` which accepts the following options:

- `toggle` - Toggles `data-class` on `data-class-element`. This should also be the default behaviour which happens if `data-class-behaviour` isn't present.
- `add` - Adds `data-class` on `data-class-element` if it isn't already present. If it is, nothing happens.
- `remove` - Removes `data-class` on `data-class-element` if it's already present. If it isn't, nothing happens.

As with previous data attributes, this new optional attribute will be a comma-seperated list to allow for different behaviours for each action. Like so:

```html
<button class="c-button" data-class="is-loading, is-disabled" data-class-element="js-form-area, js-input" data-class-behaviour="toggle, remove">Submit</button>
```

Assuming the above is used, the following would happen once `c-button` is clicked:

1. `is-loading` would be toggled on `js-form-area`
2. `is-disabled` would be removed from `js-input` if present.

Let's make the necessary JavaScript changes:

```javascript
// Grab all elements with data-active attribute
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
if(elems.length){
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
}
```

And finally, a working example:

<iframe height='300' scrolling='no' title='#6) Form Component w Improved reusable + multiple any class + behaviours function' src='//codepen.io/lukedidit/embed/zwvdeY/?height=300&theme-id=5799&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/lukedidit/pen/zwvdeY/'>#6) Form Component w Improved reusable + multiple any class + behaviours function</a> by Luke Harrison (<a href='http://codepen.io/lukedidit'>@lukedidit</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

## Further improvements
There's many ways in which this reusable function could be improved even further. Including but not limited to:

- Support for using different events other than click.
- Swipe support for touch devices.

In the meantime, if you have any ideas for your own improvements or have a completely different method of managing stateful classes altogether then be sure to let me know in the comments below.