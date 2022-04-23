# `Web Animation Performance Fundamentals`
**Resources:** https://www.freecodecamp.org/news/web-animation-performance-fundamentals/

## Summary:

- Web pages were interactive animations played back by your web browser
- The web browser has to be able to display sixty frames per second (60 fps).
- When you scroll through a page, the browser displays off-screen areas of the document as you scroll down (or up).
- When a page doesn't respond swiftly to user interactions or has jerky movements, something must be off.
And it's usually owing to the browser's main thread being so busy that it can't deliver frames on time (more on this below).

### Refresh Rate or Frame Rate?

- The average display device refreshes the screen sixty times per second (60Hz). To the human eyes, any frequency above 12Hz is perceived as motion.
- Refresh rate is the number of times a display device refreshes an image in one second. The frame rate is an arbitrary number of frames (in a filming system), captured or drawn in a second.

There's a Deadline to Produce Each Frame
- Displaying sixty frames per second means each frame must be screen-ready in 16.7ms (1 sec ÷ 60).
- Otherwise, the frame would be delayed or dropped. This issue is often referred to as jank on a web page.

### How a Frame is Produced

A web page changes when:

- The user interacts with the page. They scroll, pinch zoom, click, select a piece of text, and so on.
- A piece of JavaScript code changes the page. For instance, it adds a <div> element or changes a CSS style. 

And this is what it looks like from a high-level perspective:

- JavaScript Evaluate – the browser: oh, something changed! I need to generate a new frame.
- Style Calculate – the browser: now I must apply class some-class to to that <div> element).
- Layout (reflow) – the browser: I see some elements have new styles now. I need to calculate how much space they take on the screen and where they should be positioned based on these styles. Also, I need to calculate the geometry of every other element affected by this change!
- Paint – the browser: Now, I should group elements (that have an output) in multiple layers and convert each layer into a bitmap representation in the memory or the video RAM.
- Compositing – the browser: Now, I should combine these bitmaps in the defined order to form the final frame.

![The pixel pipeline](https://www.freecodecamp.org/news/content/images/size/w1600/2022/02/pipeline-1.png)
	
<p align="center">The pixel pipeline</p>

- Any change to an element's geometry (when you change the height, width, left, top, bottom, right, padding, margin, and so on) involves the whole pipeline.

- Sometime web browser skips the layout step this time and jumps to painting.

### Optimize paintwork

![](https://www.freecodecamp.org/news/content/images/size/w1600/2022/02/pipeline-paint.png)
	<center>Pixel pipeline without the layout step</center>

### Use composited-only transformations

![The pixel pipeline without Layout and Paint](https://www.freecodecamp.org/news/content/images/size/w1600/2022/02/pipeline-composite.png)
	<center>The pixel pipeline without Layout and Paint</center>

**Below is the list of changes the browser can do cheaply at compositing time:**
- Re-positioning with transform: translate(mpx, npx)
- Rotating with transform:rotate(xdeg)
- Scaling with transform: scale(x)
- Opacity with opacity(x)

So, depending on the change we make to the DOM, the process will be one of these three scenarios.

- JavaScript → Style → Layout → Paint → Composite
- JavaScript → Style → Paint → Composite
- JavaScript → Style →  Composite

**Try to reduce the main thread's workload**

![CSS properties and their initial step in the pixel pipeline](https://www.freecodecamp.org/news/content/images/size/w1600/2022/02/Twitter-post---55.png)
	<center>CSS properties and their initial step in the pixel pipeline</center>

**Make sure your JavaScript callbacks catch the train!**
- If you set the interval to repeat your code every 16.7ms, your code could run at any point during each 16.7ms slot.
- So if we have 16.7ms to make a change, we need to make sure our code executes right at the beginning of each 16.7ms slot.
Otherwise, it would require more than 16.7ms to complete, and it won't be ready for the current slot.
- `RequestAnimationFrame()` has been designed just for that. It makes sure your callbacks are executed right at the beginning of the next frame.
- Another benefit of using requestAnimationFrame is that the browser can run your animation more efficiently.
- For instance, if the user switches to another tab, the browser will pause the animation. This reduces the processing time and battery life.

So instead of:

```jsx
setInterval(
	() => {
    	// make some change
    },
    16.7
)
```
You can do:

```jsx
const animateSomething = function () {
	// make some change
    
    // Next call
    requestAnimationFrame(animateSomething)
}

// First manual call to start the animation
requestAnimationFrame(animateSomething)
```


### TL;DR:

To have smooth motions on your page, all you need to do is to make sure:

- Fames are delivered on time
- Frames are delivered on time consistently

**And here's a checklist to achieve it:**

- Make sure your JavaScript changes happen at the beginning of each frame by using `requestAnimationFrame`.
- When changing the dimension of an element, use `transform:scale()`  over `height` & `width`.
- To move the elements around, always use `transform: translate()` over coordinates (`top`, `right`, `bottom`, and `left`).
- Reduce paint complexity by using simple CSS styles over expensive ones. For instance, if possible, use solid colors over gradients or shadows.
- Normalize using the transitions on mobile versions. Even though the computing capacity of mobile phones is limited, mobile-version UX often contains more transitions/effects owing to their small screen.
- Use your browser's developer tools to diagnose animation performance. Use tools such as Paint Flashing and FPS meter to fine-tune your animations.
- Use DevTool's Performance panel to see how your code runs on lower-end devices.
You can apply these micro-optimizations when doing any type of change. Whether you're making JavaScript or CSS animation, or you're just making a one-off change with JavaScript.
