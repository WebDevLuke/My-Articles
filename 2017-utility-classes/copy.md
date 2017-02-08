# Component classes vs utility classes in scalable CSS
### Posted on XX XXXX 2017

Lately, I've been building the front-end for a large website which is likely going to stick around for a few years, so it was important that when it comes to CSS this thing is super scalable. The last thing I’d want is in a few years time the bulk of the stylesheet to be considered a CSS no-man’s land, with the only safe option being to add new styles to the bottom of the file to avoid any conflicts.

My path to scalable CSS was to use [CSSWizardry](https://csswizardry.com/)’s brilliant [ITCSS](https://www.xfive.co/blog/itcss-scalable-maintainable-css-architecture) methodology and naming conventions such as [BEM](http://getbem.com/). I have to say it worked rather nicely. However one element of this approach which had left me scratching my head on occasion was to what extent do I leverage utility classes (or ‘Trumps’ as they are called in ITCSS)? As components are built when should CSS properties be introduced via a utility class and when should they be included in the component class natively? Essentially, where should the line be drawn?

I started by looking at the types of styles it would make sense for a utility class to introduce. In my mind these are split into 2 sub-categories. Positional styles such as floats and margins, and cosmetic styles which change how a component looks in terms of colour, size etc. As I was applying the ITCSS methodology, I had to acknowledge that it was the role of the component layer to declare cosmetic styles, so therefore I would limit my utility classes to purely positional ones. 

On paper, this approach seemed great. I had encapsulated my UI component's styles within its component class but still had the flexibility to tweak its positioning or margins via utility classes. In practice however, I often found myself creating cosmetic utility classes to compliment simple UI components built with purely a combination of ITCSS objects and positional utility classes. My justification was that I felt it wouldn't make sense to create a brand new component class just to introduce `color:white` when I could create a utility class which would do the same thing but would be reusable across the whole stylesheet.

<iframe height='300' scrolling='no' title='ggdoov' src='//codepen.io/lukedidit/embed/ggdoov/?height=300&theme-id=5799&default-tab=css,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/lukedidit/pen/ggdoov/'>ggdoov</a> by Luke Harrison (<a href='http://codepen.io/lukedidit'>@lukedidit</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

In the example above, I have a very simple UI component which I've created using a combination of basic object patterns and some utility classes. Again, if I kept strictly to the rule that utility classes should only be used for positional styles, I would have to create a brand new component class just to introduce `background-color`. That seems like a waste, and places more importance on adhering to methodology over real-world practicality.

Coming back to the question, where should the line be drawn between component and utility classes? Is my answer that the component layer is useless and everything should be a combination of objects and utilities? No, I don't think so, as that would introduce too much bulk to the HTML. However I think there is a case for when objects and positional utility classes almost get the job done, cosmetic utility classes can be used to finish the job as long as those utility classes are simple enough to be reusable elsewhere. In most other cases however, the majority of a UI component's styles should still be placed within its component class, especially if those styles are highly specific to that particular UI component, as they simply wouldn't be reusable at the utility level. As per below:

#### Good
Simple and reusable. A perfectly fine cosmetic utility class.
````
.u-background-red {
	background-color:red !important;
}
`````

#### Bad
Probably too specific to be a utility class and not as reusable. Should probably be a part of a relevant component class.
````
.u-decorate {
	background-color:red !important;
	border-left:1px solid black !important;
	color:white !important;
	border-radius:3px;
}
`````

This approach has certainly helped to speed up development without being too detrimental to the readability of the HTML. The key is to be sensible and use cosmetic utility classes sparingly and make sure they are simple and reusable.