- Start Date: 2022-01-23
- Implementation PR: (leave this empty)
- Evans insane 3/4

# Summary
This RFC introduces a new feature to the `Astro` API: `Astro.compile`. It is designed to reduce javascript in built sites further by precompiling segments of external javascript, similar to frontmatter.
# Example
Evan is adding a really cool text to the bottom of his website. Inside a script tag, he writes
```js
let reallycooltext = "evan " + "is " + "really " + "cool!"
document.getElementById('footer').innerHTML = reallycooltext
```
Next, Evan refactors his code to do it the astro way
```html
---
let reallycooltext = "evan " + "is " + "really " + "cool!"
---
<script define:vars={{reallycooltext}}>
document.getElementById('footer').innerHTML = reallycooltext
</script>
```
This reduces his website size by getting rid of extra quotation marks and plus signs. The built HTML here looks like
```html
<script>
let reallycooltext="evan is really cool!"
document.getElementById('footer').innerHTML = reallycooltext
</script>
```
This is good progress, but Evan realizes that including this script tag on *every* page is excessive. He realizes it would be better to make another file, `script.js` and import it from the HTML. That way, the browser can cache it. He moves his code out of the astro component into it's own file.
```js
let reallycooltext = "evan " + "is " + "really " + "cool!"
document.getElementById('footer').innerHTML = reallycooltext
```
And imports it from the html. This is great improvement, but he is once again shipping the useless + symbols and " symbols.

This is where we are now, ideally Evan could have frontmatter inside his javascript file, but that's impossible. Right now, he could

1. Compile it manually, remove extra quotation marks and plus symbols
2. Accept the bigger js file size

Now, let's refactor the javascript file with our RFC.

```js
let reallycooltext = Astro.compile(function() {return "evan " + "is " + "really " + "cool!"})
document.getElementById('footer').innerHTML = reallycooltext
```
This gets built into
```js
let reallycooltext="evan is really cool!"
document.getElementById('footer').innerHTML = reallycooltext
```
Once again, while still being
1. Convenient for Evan, no manual conversion!
2. As small as possible, Convenient for the visitor.
3. Cached by the browser
Truly the best of both worlds.
# Motivation
As we all know, developers like convenience. Sometimes (example A: frameworks), they add extra overhead for their users so they can experience convenience. For scripts used on one page, Astro provides the best of both worlds, by prexecuting javascript inside of frontmatter.

For scripts used on many pages, developers want to make the script an external file to cache data and save space, but by doing so they lose out on the amazing functionality of prexecuting javascript, and end up shipping unnecessary setup code. 
# Detailed design
A new function is implemented that takes one parameter: a function.
```ts
function compile(func: Function) {
	return func() // return the value func returns
}

```
Whatever the inner function (`func`) returns is used to overwrite `Astro.compile(.*?)*`. For instance:
```js
let fiveplusfive = Astro.compile(function() {return 5 + 5})
```
is
```js
let fiveplusfive = 10
```
once built.
# Drawbacks
- implementation cost, both in term of code size and complexity
# Alternatives
Somehow cramming frontmatter into javascript
# Adoption strategy
First, a clear way to even achieve this must be specced out. Then, it can be implemented with little fanfare. 
# Unresolved questions
## How to code
It's actually pretty unclear how "overwriting a function" would even be done. Self modifying code is hard.
## Inner variables
Consider the following code:
```js
let n = 5
let result = Astro.compile(function() {return n + 5})
```
1. We could intelligently execute only parts of the file by looking at referenced variables inside `Astro.compile` ahead of time
2. We could execute the whole file
3. We could require referenced variables from inside `Astro.compile` to be defined inside the function signature (`Astro.compile(function(){return num+5},{num: n})`) to make analysis easier.

I think 1 is the best route, but not positive.