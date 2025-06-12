- A good transition time is about 0.15s.
- Using padding is better than width and height for "box" component. It helps keep the spacing even if the content inside the "box" change.
- Image inside div is overflowing by default.
- A typical characteristic of `inline` elements is that they respect the whitespace in the markup. Reference: https://stackoverflow.com/questions/19038799/why-is-there-an-unexplainable-gap-between-these-inline-block-div-elements
- The difference between a Flexbox and a Grid is that Flexbox has a more flexible layout where as the Grid has a more rigid layout.
- To set the shadow to inside the element, when using box-shadow property, put `inset` as the first value.
- If the border-radius value is half the value of height, it will make the element round.
- Shrink behaviour modification when using flex:
	- flex-shrink: 0; = don't shrink
	- width: 0; = shrink
- Follow transition property with name of another property to target it.
- To create an image which link to another website, make the image the content of the link element.
- display inline-flex is great for an element to have a dynamic width and height base on its child elements.
- To change a property of an element based on action performed on other element, use advance css selector with pseudo class:
	```
	."base-element":"pseudo-class" ."target-element"
	```
- For z-index to work properly, the element **must have position set to relative, absolute, fixed, or sticky**.
- CSS inheritance mostly work with text property.