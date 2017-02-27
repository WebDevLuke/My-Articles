<!-- 
Naming Conventions for Sizes in Scalable CSS
Posted on 27th February 2017

Originally posted at:
https://speckyboy.com/naming-conventions-sizes-scalable-css/
-->

In this article, I’ll be exploring how naming conventions can be used to represent sizes in scalable CSS.

So, what do I mean by this? One aspect of scalable CSS commonly involves creating patterns, components and utilities which are reusable across the codebase. Below is a very simple object pattern which boxes off content, similar to the island object, as detailed by [Harry Roberts](https://csswizardry.com/2011/10/the-island-object/):-

```css
.o-box {
	display:block;
}

.o-box--spacing-small {
	padding:10px;
}

.o-box--spacing-medium {
	padding:20px;
}

.o-box--spacing-large {
	padding:30px;
}
```

What makes this pattern especially reusable are the modifiers (The 'M' in [BEM](https://en.bem.info/)), which provide a range of spacing options to choose from.

These modifiers use a naming convention which represents a clear incrementation from small to large. This means that a developer reading the HTML would notice these classes, and therefore, that one box implementation has more spacing than another, without having to refer back to the stylesheet.

```css
.o-box--spacing-small
.o-box--spacing-medium
.o-box--spacing-large
```

This small -> medium -> large pattern is a very basic example of a naming convention in CSS. It can be repeated to similar effect across a whole range of scenarios.

## What makes the perfect naming convention?

Ideally the perfect naming convention for sizes should:-

- Be semantic, in that it uses names representing a clear incrementation from small to large - otherwise it’s impossible to identify which class represents what, without having to check the stylesheet.
- Be easy to remember - otherwise again, I'll need to refer back to the stylesheet.
- Scale - this one is the important one as it needs to be able to adapt to change as my stylesheet grows in size.

Below is a media object with three modifiers using the same simple naming convention as before - small, medium and large. I'll apply the three principles and see what happens.

```css
.o-media {
	box-sizing:border-box;
}

.o-media__image {
	float:left;
}

.o-media__body {
	float:left;
}

.o-media--spacing-small > .o-media__image {
	margin-right:5px;
}

.o-media--spacing-medium > .o-media__image {
	margin-right:15px;
}

.o-media--spacing-large > .o-media__image {
	margin-right:30px;
}
```

It’s definitely semantic in that it implies incrementation. If I see `.o-media--spacing-large` attached to one element and then I see `.o-media--spacing-small` attached to another, I'll know for certain that the former is going to have more spacing than the latter. This naming convention is also simple enough to remember, without having to refer back to the stylesheet. So far so good.

As mentioned above, scalability is the one of the most important aspects a naming convention needs. 

Say a year after the initial build is complete, I’m making amends and I require another spacing modifier with `margin-right: 50px;`. I decide to call this `.o-media--spacing-extra-large` as it’s already larger than my largest modifier, so should reflect this semantically in its name. The situation repeats itself a few more times, as I require larger and larger abstractions, and suddenly I have this:-

```css
.o-media--spacing-small > .o-media__image {
	margin-right:5px;
}

.o-media--spacing-medium > .o-media__image {
	margin-right:15px;
}

.o-media--spacing-large > .o-media__image {
	margin-right:30px;
}

.o-media--spacing-extra-large > .o-media__image {
	margin-right:50px;
}

.o-media--spacing-extra-extra-large > .o-media__image {
	margin-right:100px;
}

.o-media--spacing-extra-extra-extra-large > .o-media__image {
	margin-right:150px;
}
```

The problem here is that the medium class `.o-media--spacing-medium` is now no longer techically medium as suddenly most of the modifiers are larger than it. It’s still ok in that it’s working, but in my opinion, is a bit messy.

Another issue arises if something in the design changes which means, for example, a `25px` modifier is now required. To preserve the structure of the modifiers, this needs to sit in between `.o-media--spacing-medium` and `.o-media--spacing-large`. Previously new modifiers have been added using an ‘extra’ prefix, however in this case ‘extra-medium’ doesn’t make any sense at all:-

```css
.o-media--spacing-medium > .o-media__image {
	margin-right:15px;
}

.o-media--spacing-extra-medium > .o-media__image {
	margin-right:25px;
}

.o-media--spacing-large > .o-media__image {
	margin-right:30px;
}
```

I could replace `.o-media--spacing-large`’s value with `25px`, `.o-media--spacing-extra-large`’s value with `.o-media--spacing-large`’s, and so on until it’s all consistent again. However, I would then have to go through all the instances of each class in the HTML and update accordingly every time I add another modifier, which will become more and more time consuming as the project grows larger.
 
So what may have seemed like a reasonable naming system for sizings when the project started, suddenly begins to strain as things change.

## Making the names for sizes as scalable as the CSS

Coming back to the initial list of what makes the perfect naming convention, is it possible to have a naming convention which is semantic, easy to remember AND scalable? 

I'm yet to find a perfect solution! But here are three approaches I've found, and regularly use, which may work for you, depending on the situation:-

### 1) Sacrifice semantics for pure scalability

In one project, I was building a website based on PSD designs and was using an incremental naming convention to declare type modifiers (Think `c-type--small`, `c-type--medium` etc). As I got drip fed more and more designs, new type sizes were being introduced at an alarming rate, which meant I often spent quite a bit of time renaming all my modifiers in the CSS and then their references in the HTML.

Eventually I decided enough was enough and sacrificed semantics for pure scalability with this solution based on the [phonetic alphabet](https://en.wikipedia.org/wiki/NATO_phonetic_alphabet):

```css
.c-type--alpha {
	font-size:1.5rem;
}

.c-type--bravo {
	font-size:0.5rem;
}

.c-type--charlie {
	font-size:3.5rem;
}

.c-type--delta {
	font-size:6rem;
}

@media only screen and (min-width: 768px) {
	.c-type--charlie {
		font-size:5rem;
	}
}
```

It’s by no measure a perfect solution, because it’s not clear which class represents which font size without having to refer back to the stylesheet. But what I do get in return, is a massive boost to scalability as I no longer have to spend time enforcing the incremental nature of the previous convention. If I need another class, I just simply pick the next available letter and I’m on my way! 

When you have many instances of classes sharing a naming convention, I feel scalability should take priority over semantics. This is because it takes less effort for a developer to refer to the stylesheet for more information than it does to restructure an entire set of classes every time a new one is added.


### 2) Using numbers as suffixes instead of “extra” prefix

Below is a naming convention very similar to the pattern demonstrated at the start of the article. What differs here is that instead of naming the second instance of a size “extra-large” for example, I’m incrementing a number suffix so I end up with “large-2”. 

```scss
$breakpoints: (
	"small": 370px,
	"medium": 768px,
	"medium-2": 840px,
	"large": 1024px,
	"large-2": 1280px
)
```

How does this help? Well, it removes the problem of bloated names, such as “extra-extra-extra-large”, and also works when applied to “medium” when previously, “extra-medium” made no sense whatsoever.

This compatibility across small, medium and large names also means it fares a little better when scaled up:

```scss
$breakpoints: (
	"small": 370px,
	"small-2": 425px,
	"small-3": 500px,
	"medium": 768px,
	"medium-2": 840px,
	"medium-3": 925px,
	"large": 1024px,
	"large-2": 1280px,
	"large-3": 1440px
)
```

Though, it’s still not perfect. For example, if I need to add a `medium-4`, there may be a clash with the first instance of `large` due to the narrow variation in the pixel values. I would then have to move each `large` instance up a notch to make way for my new value. So again, the same old problem of having to restructure things.


### 3) Declare speculative sizes at the start of the project

Finally, what I could do is essentially declare so many sizes upfront that chances are I’ll have already covered all potential eventualities. I call this the nuclear option!

```scss
$spacing: (
	"small": 5px,
	"small-2": 10px,
	"small-3": 15px,
	"small-4": 20px,
	"small-5": 25px,
	"medium": 30px,
	"medium-2": 35px,
	"medium-3": 40px,
	"medium-4": 45px,
	"medium-5": 50px,
	"large": 55px,
	"large-2": 60px,
	"large-3" 65px,
	"large-4": 70px,
	"large-5": 75px,
	"huge": 80px,
	"huge-2": 85px,
	"huge-3": 90px,
	"huge-4": 95px,
	"huge-5": 100px
);

@each $name, $value in $spacing {
	.c-mycomponent--spacing-#{$name} {
		padding:$value;
	}
}
```

This could sidestep the issue of having to restructure things if new values are introduced later down the line, as chances are I may have already included it.

Though of course this is a bit heavy handed as it would generate a ton of CSS which may never get used. Though if using tools such as Gulp, Grunt or PostCSS, this problem is solved by running everything through [UNCSS](https://github.com/giakki/uncss) - which removes unused CSS from a stylesheet.

### Naming conventions glossary
There’s so many different ways to approach naming conventions for sizes in scalable CSS. To finish off I’ve collected a bunch together (admittedly for the reasons discussed some are better than others) for you to use in your own projects. Enjoy!

#### Size
```scss
$myVar: (
	"atomic": 10px,
	"micro": 20px,
	"tiny": 30px,
	"small": 40px,
	"reduced": 50px,
	"regular": 60px,
	"increased": 70px,
	"large": 80px,
	"huge": 90px,
	"giant": 100px
)
```

#### Size + Number
```scss
$myVar: (
	"tiny": 10px,
	"tiny-2": 20px,
	"tiny-3": 30px,
	"small": 40px,
	"small-2": 50px,
	"small-3": 60px,
	"medium": 70px,
	"medium-2": 80px,
	"medium-3": 90px,
	"large": 100px,
	"large-2": 110px,
	"large-3": 120px
)
```
#### Location
```scss
$myVar: (
	"room": 10px,
	"house": 20px,
	"street": 30px,
	"town": 40px,
	"city": 50px,
	"state": 60px,
	"country": 70px,
	"continent": 80px,
	"planet": 90px,
	"system": 100px,
	"cluster": 110px,
	"galaxy": 120px
)
```

#### Phonetic Alphabet (Non-incremental)
You get 26 variants to choose from. After that, you could combine variants to form new ones. (alpha-bravo for example)
```scss
$myVar: (
	"alpha": 10px,
	"bravo": 20px,
	"charlie": 30px,
	"delta": 40px,
	"echo": 50px,
	"foxtrot": 60px,
	"golf": 70px,
	"hotel": 80px,
	"india": 90px,
	"juliett": 100px,
	"kilo": 110px,
	"lima": 120px
	/* and so on up to 26... */
)
```

#### Military Ranks
```scss
$myVar: (
	"recruit": 10px,
	"private": 20px,
	"corporal": 30px,
	"sergeant": 40px,
	"captain": 50px,
	"major": 60px,
	"lieutenant": 70px,
	"continent": 80px,
	"colonel": 90px,
	"general": 100px
)
```

#### Planets 
```scss
$myVar: (
	"mercury": 10px,
	"venus": 20px,
	"earth": 30px,
	"mars": 40px,
	"jupitor": 50px,
	"saturn": 60px,
	"uranus": 70px,
	"neptune": 80px,
	"pluto": 90px"
)
```

#### Size Abbreviations 
```scss
$myVar: (
	"xxs": 10px,
	"xs": 20px,
	"s": 30px,
	"m": 40px,
	"l": 50px,
	"xl": 60px,
	"xxl": 70px
)
```