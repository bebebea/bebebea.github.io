---
layout: post
title: Pixel-perfect collisions (and movement) using Bresenham's algorithm (and implementing it in GameMaker)
tags: Games GameMaker
---

One of the first challenges I encountered trying to build a platformer are collisions. Now, collisions aren't necessarily hard to implement, and in theory it's just a few lines of code.
There are many different methods out there you can use, but many that I've either seen on the internet or created myself have always had a plethora of different issues.
  
The biggest issues I usually found were either that collisions were inaccurate or could not handle unusually fast or slow speeds. Another big issue would often be that the position we end up in after colliding is illogical or just off. This method addresses all of those issues.

The main reason I'm posting this is because I've never actually seen it done this way anywhere on the internet.
That's pretty surprising! So I figured I might as well share it with the world. I hope you'll find it useful.
  
Let's start with describing the method in theory before moving on to code. That way, you should be able to use it even outside of GameMaker.

## Theory
Let's first write down what it is we want to do.
We want a method to detect collisions. It should minimally affect movement and work no matter the direction or speed.
  
So, first, let's see how we can get that done the most efficiently. These are the steps we'd need to take:

1. Figure out where the entity is going
2. Find out if there is anything in the way
3. If there is, calculate how far to go before colliding

Now, what's important to understand is that this would be done every *step*, that is every frame of the game. And it's done *after* calculating speed and direction. That means we're only considering speed and direction of that particular frame. And even if it's 0.
  
Now, the tricky part is actually implementing it, because if this alone was all we wanted to do it'd be quite easy.
After all, GameMaker has built-in tools for this.
But there are a few additional requirements:

1. The final coordinates must integers
2. The final position must be as accurate as possible

Basically, the position we actually end up in must be realistic and pixel-perfect.
  
But how would we go about doing this as efficiently as possible?

### The solution
The solution is to draw a straight line from our current position to our next position - *the trajectory*.
Then, we would follow this trajectory until a collision happens.
  
But how can we draw a straight line on a pixel grid..?

### Bresenham's algorithm
[Bresenham's line algorithm](https://en.wikipedia.org/wiki/Bresenham's_line_algorithm) is what we will be using to achieve just that!
  
It's often used for drawing straight lines on raster images, because it is the closest thing you can get to a straight line when working with pixels.
This makes it perfect for calculating trajectories on a pixel grid accurately!

## Implementing it in practice
Before we start, please know that in order to achieve the best results, you need to have already implemented pixel-perfect movement in your game.
There is an excellent tutorial for that available, but I don't have the link...
  
Also, before calculating collisions, you need to have already calculated where it is the entity is going - that is, where it would end up if there was nothing at all in the way.
So, the X2 and Y2 coordinates.
This will then create our theoretical trajectory.
### Drawing the line
The first thing you'll need is a function for calculating the line itself. This really is just Bresenham's line algorithm specially implemented in GameMaker.
  
I'm not going to be covering it in detail since it's already been done before, in much better detail that I'd ever be capable of - 
you can read more about it in RefresherTowel's blog post [here](https://refreshertowelgames.wordpress.com/2021/01/23/procedural-generation-in-gms-4-connecting-the-dots-with-bresenhams/).
{% highlight js %}
function bresenhams_line(_x1, _y1, _x2, _y2)
{
	_x1 = int64(_x1); _y1 = int64(_y1);
	_x2 = int64(_x2); _y2 = int64(_y2);
	
	// Array creation
	var _points = [];
	
	// Delta
	var _dx = _x2-_x1;
	var _dy = _y2-_y1;
	
	// Angle
	var _sx = sign(_dx);
	var _sy = sign(_dy);

	// Segment Length
	_dx = abs(_dx);
	_dy = abs(_dy);
	var _d = max(_dx,_dy);
	var _r = _d/2;
	
	//Algorithm
	if (_dx > _dy) {
        for (var i=0;i<=_d;i++) {
			array_push(_points,[]);
			array_push(_points[i],_x1);
			array_push(_points[i],_y1);
            _x1 += _sx;
            _r += _dy;
            if (_r >= _dx) {
                _y1 += _sy;
                _r -= _dx;
            }
        }
    }
    else {
        for (var i=0;i<=_d;i++) {
			array_push(_points,[]);
			array_push(_points[i],_x1);
			array_push(_points[i],_y1);
            _y1 += _sy;
            _r += _dx;
            if (_r >= _dy) {
                _x1 += _sx;
                _r -= _dy;
            }
        }
    }
	
	// Value return
	return _points;
}
{% endhighlight %}
The end result is a 2D array of points the line includes. When accessing the array, know that every item is an array holding the X and Y coordinates respectively.
  
That means reading **points[13][0]** would give you the **X** coordinate of the **13th point** of the line.
  
This line is *inclusive* of starting points and end points.
  
We can use this to check every point in our trajectory for collision.

### Checking for collisions
For this script I will go step by step. Create a new function and follow along!
  
Make sure the function takes at least 4 arguments, which will be our current coordinates and our will-be coordinates. So, X1, Y1, X2, Y2. You might also be able to get away with only taking the last two coords. If you feel like it, I'd recommend adding checks and conversions first to make sure all coordinates are integers.
  
Then, we call our previous function for drawing lines. It will return an array of points which is our trajectory.

{% highlight js %}
var positions = bresenhams_line(x, y, x2, y2);
{% endhighlight %}

This is when we start doing stuff.
First, we simply create a for loop, repeating the same collision checks for every point in our trajectory.
This will basically make us start moving towards the end of the line.

{% highlight js %}
for (var i = 0; i < array_length(positions); ++i) {
{% endhighlight %}

Now we check if there are any collisions at our current position in the trajectory.
  
If not, we move to that position.
I coded it by directly changing the calling instance's coordinates, though you can do it via variables if the calling instance isn't the one that's moving.
We'll return the coordinate values anyway.

{% highlight js %}
if (place_meeting(positions[i][0],positions[i][1],object)) {
  // There is a collision at this point
}
else {
  // There is no collision, we move to this point.
  x = positions[i][0];
  y = positions[i][1];
}
{% endhighlight %}

Now, here's what happens when a collision is detected - meaning we are now *inside* something we're not supposed to be.
  
First, we check whether we're at the start of the line, meaning the position where we started.
This may seem random, but it's really useful as it essentially tells us that we're *stuck*. You can put code here that tries to make the entity unstuck. I might write about that one day. (It also makes sure the code after that doesn't break.)
  
Then, we check whether our *previous* position is free. It absolutely should be, but I added a check anyway since we can't really assume these things.
  
The rest is simple: we move back to the previous free position and exit the loop. At this point we are the farthest we could have gone without colliding.

{% highlight js %}
// Collision is detected
if (i == 0) {
  // We are stuck at the point we began.
}
else if not (place_meeting(positions[i-1][0],positions[i-1][1],object)) {
  // We are stuck now but weren't before, so we move back.
  x = positions[i-1][0];
  y = positions[i-1][1];
  // We don't want to move anymore, so we exit the loop.
  break;
}

{% endhighlight %}

Finally, at the end of it all, we return the coordinate values. In this case I'm returning the distance we moved on each axis.

{% highlight js %}
return [x-x1,y-y1];
{% endhighlight %}

That's it!

### Putting it together
This is what your function should look like. This isn't the best it can be - I intentionally stripped it down to only what really matters. Feel free to make edits to make it suit your game better.

{% highlight js %}
var positions = bresenhams_line(x, y, x2, y2);

for (var i = 0; i < array_length(positions); ++i)
{
  if (place_meeting(positions[i][0],positions[i][1],object))
  {
    if (i == 0)
    {
      // Insert "I'm stuck" code here
      break;
    }
    else if not (place_meeting(positions[i-1][0],positions[i-1][1],object))
    {
      x = positions[i-1][0];
      y = positions[i-1][1];
      break;
    }
    else
    {
      // Decide what your game should do in case the script can't find a free position.
    }
  }
  else
  {
    x = positions[i][0];
    y = positions[i][1];
  }
}
return [x, y];
{% endhighlight %}

And that is essentially it. I hope you found this useful!