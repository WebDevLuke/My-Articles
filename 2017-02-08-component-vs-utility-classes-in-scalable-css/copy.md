<!--
Component vs utility classes in scalable CSS
Posted on 8th February 2017
-->

Lately, I've been building the front-end for a large website which is likely going to stick around for a few years, so it was important that when it comes to CSS this thing is super scalable. The last thing I’d want is in a few years time the bulk of the stylesheet to be considered a CSS no-man’s land, with the only safe option being to add new styles to the bottom of the file to avoid any conflicts.

My path to scalable CSS was to use [CSSWizardry](https://csswizardry.com/)’s brilliant [ITCSS](https://www.xfive.co/blog/itcss-scalable-maintainable-css-architecture) methodology and naming conventions such as [BEM](http://getbem.com/). I have to say it worked rather nicely. However one element of this approach which had left me scratching my head on occasion was to what extent do I leverage utility classes (or ‘Trumps’ as they are called in ITCSS)? As UI components are built when should CSS properties be introduced via a utility class and when should they be included in the component class natively? Essentially, where should the line be drawn?

I started by looking at the types of styles it would make sense for a utility class to introduce. In my mind these are split into 2 sub-categories. Positional styles such as floats and margins, and cosmetic styles which change how a component looks in terms of colour, size etc. As I was working to the ITCSS methodology, I had to acknowledge that it was the role of the component layer to declare cosmetic styles, so therefore I decided to limit my utility classes to purely positional ones. 

This approach seemed great and mostly worked fine. I had encapsulated my UI component's styles within its component class but still had the flexibility to tweak its positioning or margins via utility classes. However, I often found myself in a strange edgecase where I was creating cosmetic utility classes to supplement simple UI components built with purely a combination of ITCSS object patterns and positional utility classes. My justification was that I felt it wouldn't make sense to create a brand new component class just to introduce `color:white` when I could create a cosmetic utility class which would do the same thing but would be reusable across the whole stylesheet.

<iframe markdown height='300' scrolling='no' title='Utility Class' src='//codepen.io/lukedidit/embed/ggdoov/?height=300&theme-id=5799&default-tab=css,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/lukedidit/pen/ggdoov/'>Utility Class</a> by Luke Harrison (<a href='http://codepen.io/lukedidit'>@lukedidit</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

In the example above, I have a very simple UI component which I've created using a combination of basic object patterns and some utility classes. Again, if I kept strictly to the rule that utility classes should only be used for positional styles, I would have to create a brand new component class just to introduce `background-color`. That seems like a waste, and places more importance on adhering to methodology over real-world practicality.

So again, where should the line be drawn between component and utility classes? In most cases a component class complimented by a mix of positional utility classes will still be the way to go, however I do think there is a case for when objects and positional utility classes almost get the job done, cosmetic utility classes can be used to finish the job as long as those utility classes are simple enough to be reusable elsewhere.

This flexible approach has certainly helped to speed up development without having too much of an impact on the readability of the HTML.

Any questions? Feel free to comment below.