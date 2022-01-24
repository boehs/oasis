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
Some data only becomes available after the site is built, like data in external javascript files and operations that depend on the output of another site. The ability to defer compilation of a file fixes this.
# Detailed design
Before the build of each page, frontmatter is checked for a variable with the EXACT signature of `const defer: number` (note: it's important for this to be constant. if it was `let`, the number could change and would require building the entire file to even determine if it should be built later).

First, astro searches each file for `defer`, if it has it, defer is it's value. If not, `defer = 0`. Then, it enters a for loop, building each file with `defer = 0`, than `defer = 1`, etc. Than 
# Drawbacks

Why should we *not* do this? Please consider:

- implementation cost, both in term of code size and complexity
- whether the proposed feature can be implemented in user space
- the impact on teaching people Astro
- integration of this feature with other existing and planned features
- cost of migrating existing Astro applications (is it a breaking change?)

There are tradeoffs to choosing any path. Attempt to identify them here.

# Alternatives

What other designs have been considered? What is the impact of not doing this?

# Adoption strategy

If we implement this proposal, how will existing Astro developers adopt it? Is
this a breaking change? Can we write a codemod? Can we provide a runtime adapter library for the original API it replaces? How will this affect other projects in the Astro ecosystem?

# Unresolved questions

Optional, but suggested for first drafts. What parts of the design are still
TBD?