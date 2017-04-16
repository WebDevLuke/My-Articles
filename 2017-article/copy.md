<!-- 
Managing state in front-end with reusable JavaScript functions
Posted on XXXXXX
-->

# Managing state in front-end with reusable JavaScript functions

Determining the most efficient way of managing state is a common issue in HTML/CSS, but thankfully there are many methodologies out there which provide some good solutions. My preferred solution comes from [SMACSS (Scalable and modular architecture for CSS)](https://smacss.com/) and involves stateful classes. To quote SMACSS's [own documentation](https://smacss.com/book/type-state), stateful classes are:

> A state is something that augments and overrides all other styles. For example, an accordion section may be in a collapsed or expanded state. A message may be in a success or error state.
> 
> States are generally applied to the same element as a layout rule or applied to the same element as a base module class.

One of my most used stateful classes is `is-active`. I typically use this to apply styles to components to reflect user interactions such as a click. Taking the accordion example from the prior quote, `is-active` would be added to denote an expanded state and would apply all the relevant CSS to reflect this change. As seen in the example below:

<iframe height='300' scrolling='no' title='Accordion Component w Stateful Class' src='//codepen.io/lukedidit/embed/mmJRQP/?height=300&theme-id=5799&default-tab=html,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/lukedidit/pen/mmJRQP/'>Accordion Component w Stateful Class</a> by Luke Harrison (<a href='http://codepen.io/lukedidit'>@lukedidit</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>


You will notice that there's some javascript in play which essentially toggles the `is-active` class on the component when a click event is detected:

```javascript
var accordion = document.querySelectorAll(".c-accordion"),
		accordionHeader;

for(var i = 0; i < accordion.length; i++) {
	accordionHeader = accordion[i].querySelector(".c-accordion__header");
	accordionCurrent = accordion[i];
	
	accordionHeader.addEventListener("click", function(){
		accordionCurrent.classList.toggle("is-active");
	});
}
```

Whilst valid, this would have to be repeated again and again for any other components which leverage the `is-active` stateful class, leading to many duplicates of what is essentially the same function. The only difference between any of these would be the component the change of state to is being applied to.

Not very efficient and certainly not very DRY.

A better approach would be instead to write a single function which performs the same task and can be reused over and over again. Let's do that.

## Creating a simple reusuable function

Let's start off by building a simple function which accepts an element as a parameter and toggles `is-active`:


```javascript
var makeActive = function(elem){
	elem.classList.toggle("is-active");
}
```

This works fine, but if we slot it into our accordion javascript there's a problem:

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

Although the `makeActive` function is reusable, we still need to write code to grab our component and any of its inner elements, so there's certainly room for improvement.

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

I've attached a `data-active` attribute to the element which previously triggered the `is-active` toggle when clicked. This attribute's value represents the element where the `is-active` toggle should take place, which as before is the top-level `c-accordion` element. Note the addition of a new `js-accordion` class rather than hooking into the existing `c-accordion` class. This is so we can decouple functional aspects of the component from it's styling.

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

      // Grab linked element
      var linkedElement = document.querySelector("." + this.getAttribute('data-active'));

      // Toggle linked element if present
      if(linkedElement) {
        linkedElement.classList.toggle("is-active");
      }
      
    });    
  }
}
```

This has certainly improved things as we no longer have to write code to set up the toggle, just attach a data-active attribute to our trigger element and specify a target element. As it stands, this function can be used for any other component where a click-based `is-active` class is required. Full example below:

<iframe height='300' scrolling='no' title='Accordion Component w Reusable is-active function' src='//codepen.io/lukedidit/embed/zwGdqV/?height=300&theme-id=5799&default-tab=html,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/lukedidit/pen/zwGdqV/'>Accordion Component w Reusable is-active function</a> by Luke Harrison (<a href='http://codepen.io/lukedidit'>@lukedidit</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

## Improving our reusable function

Whilst the reusable function works, when scaled I have to take care to make sure my trigger and target element classes don't conflict with one another. In the example below, when I add a few more accordion elements, without the added number suffixes clicking one accordian would trigger `is-active` on all of them. 

```html
<div class="c-accordion js-accordion">
	<div class="c-accordion__header" data-active="js-accordion">First Accordion</div>
	[...]
</div>

<div class="c-accordion js-accordion-2">
	<div class="c-accordion__header" data-active="js-accordion-2">Second Accordion</div>
	[...]
</div>

<div class="c-accordion js-accordion-3">
	<div class="c-accordion__header" data-active="js-accordion-3">Third Accordion</div>
	[...]
</div>
```

Adding number suffixes does solve the problem, but it's a little bit of extra work which we can do without. One way to get around this would be to implement "keywords" such as `parent` which when passed as a target element in `data-active` would trigger `is-active` on the trigger element's parent. For example:

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

      // If data-active is set to parent
      if(this.getAttribute('data-active') === "parent") {
        // Grab parent element
        var linkedElement = this.parentElement;
      }
      else {
        // Otherwise grab linked element
        var linkedElement = document.querySelector("." + this.getAttribute('data-active'));
      }

      // Toggle linked element if present
      if(linkedElement) {
        linkedElement.classList.toggle("is-active");
      }
      
    });    
  }
}
```

<iframe height='300' scrolling='no' title='#3) Accordion Component w Improved reusable is-active function' src='//codepen.io/lukedidit/embed/YVXEMy/?height=300&theme-id=5799&default-tab=html,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/lukedidit/pen/YVXEMy/'>#3) Accordion Component w Improved reusable is-active function</a> by Luke Harrison (<a href='http://codepen.io/lukedidit'>@lukedidit</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

Other possibilities for keywords could be:

- `this` - Toggles `is-active` on the trigger element
- `nextSibling` - Toggles `is-active` on the element following the trigger element
- `previousSibling` - Toggles `is-active` on the element preceding the trigger element

## Moving beyond is-active
Our reusable function is now working nicely and is an efficient way of setting up `is-active` toggles on all kinds of components. However what if I need to set up a similar toggle for another stateful class? As it stands I would have to duplicate my function and change all references of `is-active` to my new stateful class. Not very efficient.

We should improve our reusuble function to accept any class. To do so we'll have to refactor our data-attributes:







	- Manages multiple classes
	- Implement different behaviours (add, remove, toggle)
	- General ideas for improvement (eg swipe), reference OrionJS