- Start Date: 2022-01-23
- Implementation PR: (leave this empty)
- Insane 4/4

# Summary
This RFC introduces a new variable that is checked in each page during build, `defer`
# Example
Let's say we have a setup like this
`layouts/main`
```js
import Stf from '$scripts/title'
const {title} = Astro.props;
if (title) Stf.add(title)
```
`pages/titles`
```jsx
    {[...Stf].map((item) => (
        <li>{item}</li>
    ))}
``` 
`$scripts/title`
```js
let titles = new Set([])
export default titles;
```
In this setup, the user wants a page with a list of all the titles on their website. The issue is there is no way to guarantee that `pages/titles` will be built last. Let's introduce `defer`
`pages/titles`
```jsx
---
const defer = 1
---
```
This page will be built after all the pages in the site are built, because of it's number.
# Motivation
Some data only becomes available after the site is built, like data in external javascript files and operations that depend on the output of another page. The ability to defer compilation of a file fixes this.
# Detailed design
Before the build of each page, frontmatter is checked for a variable with the EXACT signature of `const defer: number` (note: it's important for this to be constant. if it was `let`, the number could change and would require building the entire file to even determine if it should be built later).

If the file has that variable, defer is it's value. If not, `defer = 0`. Then, it enters a for loop, building each file with `defer = 0`, than `defer = 1`, etc.

A reference implementation is [here](https://i.boehs.org/astro-defer-basic-ref1.js)
# Drawbacks
* extra build time, likely can be negated by
	* instead of indexing each file ahead of time, only bother if there is actually the variable defer in the file, otherwise build normally. Then the order becomes `pages without defer & defer = 0 (on sight of file)`, `defer = 1`, etc
* Defer variable is unusable
# Alternatives
Same thing but with a function.