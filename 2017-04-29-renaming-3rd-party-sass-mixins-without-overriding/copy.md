<!-- 
Renaming 3rd party SASS mixins without overriding
Posted on 29th April 2017
-->

Let's say in your project you regularly use your own simple custom mixin to make writing media queries easier. Something like this:

```scss
@include bp(mobile) {
	// Styles
}
```

And SASS compiles it into this:

```css
@media (min-width: 20em) {
	// Styles
}
```

For whatever reason, you decide to stop development on this mixin and instead move to [sass-mq](https://github.com/sass-mq/sass-mq), which is a similar but much more functional mixin developed in-house at the [Guardian](https://www.theguardian.com) newspaper. When using this mixin, the syntax is a little different:

```scss
@include mq(mobile) {
	// Styles
}
```

Instead of `bp` you're now using `mq`. In most cases this change of syntax wouldn't be a big deal. You would just use the new mixin from here on out, find/replace any references to your old custom mixin, and be on your merry way.

Or not.

A problem may be that you simply disagree with the new syntax, preferring your original because it's more semantic. Another could be if your project is something more than a standard website, such as a SASS framework, which is a dependency of other projects. In this instance making the change would break any of those dependent projects which use your original mixin. In the world of [semantic versioning](http://semver.org/) this would be an incompatible API change and so would require a major version bump.

Is their a way to get around these problems and retain our original syntax? Introducing...

## Wrapper mixins

Staying with our sass-mq example, it's possible to use this 3rd party mixin whilst retaining the original `bp` syntax from our original custom mixin. To do this, we'll create what I'll refer to as a "wrapper mixin".

```scss
// Import sass-mq
@import 'path/to/mq';

// Create wrapper mixin
@mixin bp(
	$from: false,
	$until: false,
	$and: false,
	$media-type: $mq-media-type,
	$breakpoints: $mq-breakpoints,
	$responsive: $mq-responsive,
	$static-breakpoint: $mq-static-breakpoint
) {
	// Include original sass-mq mixin
	@include mq(
		$from,
		$until,
		$and,
		$media-type,
		$breakpoints,
		$responsive,
		$static-breakpoint) {
			@content;
	}
}
```

What we've done here is the following:

1. Copied the original sass-mq mixin declaration and all of its parameters
2. Renamed it to match the original `bp` mixin
3. Instead of populating the mixin with any logic, we're just calling the original sass-mq mixin and passing our arguments, essentially placing our own wrapper around it (Hence the term "wrapper" mixin)

This means:

```scss
@include bp(mobile) {
	// Styles
}
```

Would still compile to:

```css
@media (min-width: 20em) {
	// Styles
}
```

But it also means that we can use any sass-mq functionality in our own wrapper mixin. Like so:

```scss
@include bp($until: mobile, $and: '(orientation: landscape)') {
	// Styles
}
```

This would compile to:

```css
@media (max-width: 19.99em) and (orientation: landscape) {
	// Styles
}
```

(Note: the slight variation in compiled `em` values is because sass-mq removes `1px` from any breakpoint value in a `max-width` media query to stop its content overlapping with any `min-width` media-queries at the same breakpoint.)

This approach would stop any dependent projects from breaking, meaning our framework would require only a minor version bump as per [semantic versioning](http://semver.org/). In addition because the sass-mq mixin isn't being modified directly, it's still encapsulated in `node_modules` and can be updated normally as updates are released.

## Other benefits

We can also use wrapper mixins to create "shortcuts" to call another mixin with certain arguments already set. This is useful if you find yourself passing the same set of arguments to a mixin over and over again.

By creating a shortcut wrapper mixin, you're not only sparing yourself from having to type the same snippet repeatedly, it's also much easier to edit as there's only one instance of the mixin call in the codebase.

Let's see some examples:

### Simple example

Coming back to our sass-mq example, a simple shortcut mixin to generate `max-width` media queries may look like:

```scss
// Create wrapper mixin
@mixin bp-until(
	$until: false,
	$and: false,
	$media-type: $mq-media-type,
	$breakpoints: $mq-breakpoints,
	$responsive: $mq-responsive,
	$static-breakpoint: $mq-static-breakpoint
) {
	// Include original sass-mq mixin
	@include mq(
		false,
		$until,
		$and,
		$media-type,
		$breakpoints,
		$responsive,
		$static-breakpoint) {
			@content;
	}
}
```

You would use it like:

```scss
@include bp-until(sm) {
	// Styles
}
```

And it would compile to:

```css
@media (max-width: 19.99em) {
	// Styles
}
```

### Advanced example

This highly specific shortcut mixin acts as a quick way of setting `max-width` media queries at a `landscape` orientation and with an `only screen` type:

```scss
// Create wrapper mixin
@mixin bp-max-portrait-screen(
	$until: false,
	$breakpoints: $mq-breakpoints,
	$responsive: $mq-responsive,
	$static-breakpoint: $mq-static-breakpoint
) {
	// Include original sass-mq mixin
	@include mq(
		false,
		$until,
		"(orientation: landscape)",
		"only screen",
		$breakpoints,
		$responsive,
		$static-breakpoint) {
			@content;
	}
}
```

You would use it like:

```scss
@include bp-max-portrait-screen(sm) {
	// Styles
}
```

This would compile to: 

```
@media only screen and (max-width: 19.99em) and (orientation: landscape) {
	// Styles
}
```

## Closing

Using wrapper mixins are a simple way of renaming 3rd party mixins. There's nothing particularly complex about them, but they can be useful.

In the first set of examples, we've used one to rename sass-mq's `mq` mixin into something more compatible. In the second set, we've also seen how wrapper mixins can be used to create helpful "shortcuts" for long winded mixin configurations. 

Have any examples of your own, or spotted a way of improving on what I've done? Let me know in the comments below.