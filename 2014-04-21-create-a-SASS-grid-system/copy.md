<!--
Create a SASS grid system
Posted on 21st April 2014
-->

![SASS Grid System](https://raw.githubusercontent.com/WebDevLuke/My-Articles/master/2014-04-21-create-a-SASS-grid-system/blog-pic.jpg)

In this post I'll be showing you how to create a responsive grid system where you simply define a few variables and let SASS worry about the rest. This tutorial assumes you have some knowledge of both CSS grids and SASS, so if you're unsure about either I would definitely recommend reading up before continuing.

Right, let's get started.

## 1) Set the variables

The first thing we need to do is define the SASS variables which will allow you to specify the type of grid system you want to generate.

These should hopefully be pretty self explanatory.

```scss
// Set Grid
$columns: 12;
$column-width: 70px;
$gutter: 30px;
$padding: $gutter / 2;
```

`$columns` will govern the total number of columns.

`$column-width` will govern the width of each individual column in pixels.

`$gutter` will govern the width of the margin in between columns in pixels.

`$padding` will govern the outer padding on either side of the grid in pixels. By default I've set this figure to half of the gutter, but feel free to change this around.

We'll also add another variable which will govern if or not the container should be responsive.

```scss
// Should it be responsive?
$responsive: true;
```

## 2) Create the container

Before we create the container itself, we first need to work out how wide it should be based on the variables we've defined.

We do this by defining a new variable called $width as the sum of the width of all columns plus the width of all gutters.

```scss
// Set container width relative to grid size
$width: ($column-width * $columns) + ($gutter * ($columns - 1));
```

![SASS Grid System](https://raw.githubusercontent.com/WebDevLuke/My-Articles/master/2014-04-21-create-a-SASS-grid-system/grid.jpg)
Assuming the figures from step 1 remain unchanged, adding these together would return a figure of 1170.

We can then use this to create the grid container. Notice the use of an IF statement and the responsive variable here to determine how we set the width.


```scss
// Set Base Container
.container
{
// Set container width if responsive
@if $responsive == true
   {
	max-width:$width;
   }
@else
   {
	width:$width;
   }
margin:0px auto;
padding: 0 $padding 0 $padding;
}
```

## 3) Create the grid classes

This is where SASS really comes into it's own. Using a FOR statement we can procedurally generate the grid classes based on the $columns variable we've defined.

```scss
// Define Individual Grid Classes
@for $i from 1 to $columns
{
	// Calculate grid size in pixels
	$grid: ($column-width * $i) + ($gutter * ($i - 1));
	
	// Calculate responsive grid sizes if required
	@if $responsive == true
	{
		// Convert Pixels to Percentages
		$grid: ($grid / $width) * 100%;

		// Convert Gutter Pixels to Percentages
		$gutter-percentage: ($gutter / $width) * 100%;

		.grid#{$i}
			{
			width: $grid;
			float: left;
			margin-left: $gutter-percentage;
			}
	}
	@else
	{
		$grid: $grid;
		.grid#{$i}
			{
			width: $grid;
			float: left;
			margin-left: $gutter;
			}
	}
}

.clear
{
margin-left: 0px;
clear: left;
}
```

To help with this process there is another new variable called $grid which is used to calculate the width of each grid class based on combining the total width of columns and gutters relative to the progress of the FOR statement (represented as $i).

![SASS Grid System](https://raw.githubusercontent.com/WebDevLuke/My-Articles/master/2014-04-21-create-a-SASS-grid-system/grid2.jpg)
For example on the 7th loop of the FOR statement $i would be 7, so therefore $grid would return 670.

We've also created additional variables within the responsive IF statement to convert pixel values for both grid and gutter to percentages aswell as an additional clear class used to start new rows.

## 4) The Finished Code

So here we have the finished code and how it would compile into regular CSS. It's up to you on how you organise it. Personally I like to stick everything apart from the initial 5 user defined variables within a SASS partial.

#### SASS

```scss
// Set Grid (1170gs default)
$column-width: 70px;
$gutter: 30px;
$columns: 12;
$padding: $gutter / 2;

// Should it be responsive?
$responsive: true;

// Set container width relative to user variables
$width: ($column-width * $columns) + ($gutter * ($columns - 1));

// Set Base Container
.container
{
// Set container width if responsive
@if $responsive == true
   {
	max-width:$width;
   }
@else
   {
	width:$width;
   }
margin:0px auto;
padding: 0 $padding 0 $padding;
}

// Define Individual Grid Classes
@for $i from 1 to $columns
{
	// Calculate grid size in pixels
	$grid: ($column-width * $i) + ($gutter * ($i - 1));
	
	// Calculate responsive grid sizes if required
	@if $responsive == true
	{
		// Calculate grid size in pixels
		$grid: ($grid / $width) * 100%;

		// Get Gutter Percentage for Grids
		$gutter-percentage: ($gutter / $width) * 100%;

		.grid#{$i}
			{
			width: $grid;
			float: left;
			margin-left:$gutter-percentage;
			}
	}
	@else
	{
		$grid: $grid;
		.grid#{$i}
			{
			width: $grid;
			float: left;
			margin-left:$gutter;
			}
	}
}

.clear
{
margin-left: 0px;
clear: left;
}
```

#### CSS
`$responsive` set to `true`

```scss
.container {
  max-width: 1170px;
  margin: 0px auto;
  padding: 0 15px 0 15px; }

.grid1 {
  width: 5.98291%;
  float: left;
  margin-left: 2.5641%; }

.grid2 {
  width: 14.52991%;
  float: left;
  margin-left: 2.5641%; }

.grid3 {
  width: 23.07692%;
  float: left;
  margin-left: 2.5641%; }

.grid4 {
  width: 31.62393%;
  float: left;
  margin-left: 2.5641%; }

.grid5 {
  width: 40.17094%;
  float: left;
  margin-left: 2.5641%; }

.grid6 {
  width: 48.71795%;
  float: left;
  margin-left: 2.5641%; }

.grid7 {
  width: 57.26496%;
  float: left;
  margin-left: 2.5641%; }

.grid8 {
  width: 65.81197%;
  float: left;
  margin-left: 2.5641%; }

.grid9 {
  width: 74.35897%;
  float: left;
  margin-left: 2.5641%; }

.grid10 {
  width: 82.90598%;
  float: left;
  margin-left: 2.5641%; }

.grid11 {
  width: 91.45299%;
  float: left;
  margin-left: 2.5641%; }

.clear {
  margin-left: 0px;
  clear: left; }
```

#### CSS
`$responsive` set to `false`

```scss
.container {
  width: 1170px;
  margin: 0px auto;
  padding: 0 15px 0 15px; }

.grid1 {
  width: 70px;
  float: left;
  margin-left: 30px; }

.grid2 {
  width: 170px;
  float: left;
  margin-left: 30px; }

.grid3 {
  width: 270px;
  float: left;
  margin-left: 30px; }

.grid4 {
  width: 370px;
  float: left;
  margin-left: 30px; }

.grid5 {
  width: 470px;
  float: left;
  margin-left: 30px; }

.grid6 {
  width: 570px;
  float: left;
  margin-left: 30px; }

.grid7 {
  width: 670px;
  float: left;
  margin-left: 30px; }

.grid8 {
  width: 770px;
  float: left;
  margin-left: 30px; }

.grid9 {
  width: 870px;
  float: left;
  margin-left: 30px; }

.grid10 {
  width: 970px;
  float: left;
  margin-left: 30px; }

.grid11 {
  width: 1070px;
  float: left;
  margin-left: 30px; }

.clear {
  margin-left: 0px;
  clear: left; }
 ```

#### Example HTML
```html
<div class="container">
	<div class="clear grid4">Grid 4</div>
	<div class="grid4">Grid 4</div>
	<div class="grid4">Grid 4</div>
</div>
```

## 5) See it in action

<iframe height='300' scrolling='no' title='SASS Grid System' src='//codepen.io/lukedidit/embed/AoCfK/?height=300&theme-id=5799&default-tab=css,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/lukedidit/pen/AoCfK/'>SASS Grid System</a> by Luke Harrison (<a href='http://codepen.io/lukedidit'>@lukedidit</a>) on <a href='http://codepen.io'>CodePen</a>. </iframe>
